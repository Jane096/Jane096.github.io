---
title: "Redis Cache 기능을 활용한 성능 개선 이야기"    
layout: single    
read_time: true    
comments: true   
categories: 
 - project  
toc: true    
toc_sticky: true    
toc_label: contents    
description: Redis caching 기능을 활용하여 조회속도를 높여 본 이야기를 정리한 포스팅
last_modified_at: 2021-02-07     
---

<br>
<br>

## Overview 

저희 프로젝트에서 회원이 로그인 후 보여지는 메인 화면은 **내 주변 지역의 이벤트 목록/카테고리별 이벤트 목록** 입니다. 
즉, 모든 사용자가 저희 서비스에 접속하고 가장 많이 이용하게 되는 기능 중에 하나 입니다. 

<br>

서비스의 점차 규모가 커지고 그에 따라 사용자의 수는 늘어나게 됩니다. 증가한 사용자 만큼 수 천, 또는 수 만개의 이벤트 목록을 불러올 때 
매번 DB까지 도달하여 읽어오게 된다면 성능적인 이슈가 발생할 수 밖에 없었습니다. 

<br>
<br>
<br>

## 캐싱을 적용해보게 된 이유

**"이벤트 목록 조회"** 는 위에서 언급했던 대로 **가장 자주 사용하고 가장 오래 걸릴 수 있는 기능** 입니다. 
또한 업데이트가 아주 빈번하게 일어나는 것이 아니라서 반복적으로 동일한 결과를 출력하는 경우가 많습니다. 

<br>

이러한 특징을 가진 경우, 한번 읽어온 조회결과를 복사하여 메모리에 저장해둔다면 매번 DB로부터 호출하지 않고 메모리로 부터 값을 가져오기 때문에 DB의 부하를 줄이며
동시에 속도 향상의 결과를 볼 수 있습니다. 데이터를 메모리에 저장해둔 후 지워진다 해도 크게 문제가 되지 않기 때문에 
조회 기능에 캐싱을 적용해보기로 결정하게 되었습니다!

<br>
<br>
<br>

## Local Cache VS Global Cache

캐싱을 적용할 때 가장 먼저 고려하게 되는 요소 중 하나입니다. 서버를 기준으로 캐시를 분류하게 된다면 
Local cache 와 Global cache로 나누어 볼 수 있습니다.

<br>
<br>

### Local Cache

<br>

![image](https://user-images.githubusercontent.com/58355531/107146758-7dfd7580-698d-11eb-9ff0-b1826c979b06.png){: .align-center}

<br>

**Local Cache** 란 캐시 전략 중에 하나이며 서버에 저장하는게 아닌 클라이언트 로컬 영역에 캐시 데이터를 저장하는 것을 의미합니다. 
Local cache 를 이용하게 된다면 서버로 데이터 통신을 위한 대기 시간이 없기 때문에 응답속도가 빠릅니다. 또한 데이터의 쓰기가 한번씩만
발생하기 때문에 네트워크 트래픽 또한 줄일 수 있는 장점이 있습니다.

**그러나** 각각의 was 마다 캐시를 저장하기 때문에 하나의 장비에서만 데이터 동기화가 이루어지게 됩니다. 그리고 서버에 저장하는 것이 
아니기 때문에 다른 서버에서 참조하기가 어렵습니다. 

<br>
<br>

### Global Cache

<br>

![image](https://user-images.githubusercontent.com/58355531/107146827-dfbddf80-698d-11eb-8d19-fccd6fb39056.png){: .align-center}

<br>

**Global Cache** 란 Local Cache와 다르게 서버에 캐시 데이터를 저장합니다. 따라서 여러 대의 서버를 이용할 때 서버 간의 
데이터 동기화가 용이하며 트래픽의 분산 또한 가능합니다. 

**그러나** 서버와 네트워크 통신이 이루어지기 때문에 Local Cache보다 속도가 느리다는 단점이 존재합니다. 

<br>
<br>

### 최종선택은 Global Cache로!

[지난 포스팅](https://jane096.github.io/project/how-to-solve-server-overload-p3/)을 보셨다시피, 현재 저희 [FESTA 프로젝트](https://github.com/f-lab-edu/event-recommender-festa)에서는 급증하는 트래픽에 대응하고자 다중 서버 환경을 구축하였습니다. 
캐시 데이터 또한 이러한 다중 서버 환경에서 데이터의 동기화가 필요하기 때문에 Local cache가 아닌 **Global cache** 를 
최종적으로 선택하게 되었습니다. 

<br>
<br>
<br>

## 캐시 히트율(Cache Hit ratio)

**캐시 히트율** 이란, 캐시된 데이터를 요청할 때 해당 키 값이 메모리에 존재하여 얼마만큼의 비율로 잘 찾았는지에 대한 여부를 말합니다. 캐시 데이터를 잘 찾았다면 
`Cache hit` 이라고 하며 반대로 캐시가 존재하지 않아 찾지 못했을 경우 `Cache Miss` 라고 합니다. 

**캐시 히트율** 을 계산할 때에는 `cmd` 창에서 `redis-cli` 로  Redis에 접속하여 `info stats` 명령어를 입력하면 아래와 같은 사진처럼 정보가 뜨게 됩니다.

<br>

![image](https://user-images.githubusercontent.com/58355531/107664713-0eaabd00-6cd0-11eb-8107-ad1886440ef7.png){: .align-center}

<br>

이 때, `Keyspace-hits` 가 `Cache-hit` 을 의미하며 `Keyspace-misses` 가 `Cache Miss` 를 의미하여 캐시 히트율을 구하는 산술식은

`캐시 히트율 = Keyspace-hits / (Keyspace-hits + Keyspace-misses)`{: .text-center }      

이며, 해당 공식으로 캐시 히트율을 구할 수 있습니다.

<br>

만약, 캐시 히트율이 현저히 낮다면 그 의미는 **캐시된 데이터를 제대로 찾지 못하고 있다** 는 뜻이며 그만큼 디스크에서 데이터를 읽어오는 비율이 높다는 
것을 의미합니다. 디스크에서 데이터를 읽어올 경우, I/O가 많이 발생하기 때문에 그만큼 성능에 큰 영향을 끼칠 수 밖에 없습니다. 

반대로, 캐시 히트율이 높다는 것은 데이터베이스에서 읽어오는 비율보다 메모리에 캐시된 데이터를 읽어오는 비율이 높다는 의미이기 때문에 효율적으로 
캐시를 사용하고 있다는 반증이 됩니다. 

<br>
<br>
<br>

### 캐시 evict 와 entryTtl 

위에서 언급했던 바와 같이 캐시 히트율이 높다면 캐시 데이터를 효율적으로 사용하고 있다는 증거가 된다고 했습니다. 캐시 히트율을 높일려면 
항상 캐시 데이터를 메모리에 저장을 해두면 될 것 같지만, 그렇게 된다면 메모리에 리소스를 계속해서 차지하기 때문에 그것 또한 문제가 될 수 있습니다. 

캐시된 데이터는 적절한 때에 `Cache 지속시간(entryTtl)` 을 설정하여 만료 되도록 하고, `Cache evict`를 하여 메모리 리소스가 차지 않도록 적절한 조절이 필요합니다.

<br>

> 그렇다면 얼마만큼의 지속시간이 가장 이상적이며 어떨 때 캐시데이터를 삭제해야할까요?

<br>

**정답은 없습니다..!**

각각의 어플리케이션 마다 사용하는 메모리 크기나 키 값의 수가 똑같을 수가 없고 같은 어플리케이션 내에서도 매번 같을 것을 보장할 수 없습니다.
메모리의 크기와 키 값은 `Cache hit` 이나 `Cache misses` 에 영향을 주는 요소들이기 때문에 정확하게 지속시간을 몇 초로 설정해야 하고, 
언제 캐시를 지워야한다 라는 공식은 없습니다. 자신의 어플리케이션에서 설정한 메모리와 키의 갯수, 그리고 사용자의 패턴을 분석하여 적절한 튜닝이 필요합니다.

<br>
<br>
<br>

## LRU Simulation

그렇다면 메모리의 크기와 캐시 키 갯수가 매번 달라지는 환경에서 적절한 튜닝을 위해 Redis에선 어떤 기능을 제공하고 있을까요?

Redis 에서는 `redis-cli` 라는 **command line interface** 를 통해 테스트할 수 있는 아주 유용한 기능인 [LRU Simulation](https://redis.io/topics/rediscli#performing-an-lru-simulation)
이라는 것을 지원합니다. 이 뿐만 아니라 특정 버전의 배포를 하기 전 다양한 버전으로 미리 시뮬레이션을 할 수 있다는 장점이 있는 도구입니다.

시뮬레이션을 진행할 때 20%의 키가 80% 요청을 수행한다는 **80-20% Power law(멱 법칙)** 을 기반으로 하며 캐싱 시나리오에서 가장 흔한 분포 비율 중 하나라고 합니다. 

<br>

![image](https://user-images.githubusercontent.com/58355531/107776016-66a2fb80-6d84-11eb-80b3-0afc328cff9a.png){: .align-center}

<br>

> 출처: <https://redis.io/topics/rediscli#performing-an-lru-simulation>

<br>

위의 이미지 처럼 임의로 정한 메모리 크기(현재 100mb)와 키의 갯수(현재 1000만 개) 를 설정해둔다면 매 1초마다 `Cache hits` 와 `Cache misses` 를 
수학적 공식에 기반하여 히트율을 분석적으로 보여줍니다. 

**중요한 점은** 만약 메모리 크기를 지정하지 않고 캐시 키 값을 사용 한다면 모든 키 값을 메모리에 저장할 수 있기 때문에 캐시히트율은 항상 **100%** 가 나올 수 있습니다. 
또한, 메모리 크기를 지정하지 않고 많은 갯수의 캐시 키를 사용한다면 컴퓨터 내의 모든 RAM 을 사용하게 되기 때문에 메모리 크기에 대한 설정이 꼭 필요하다고 합니다. 

<br>
<br>
<br>

## 캐싱 적용해보기

### @EnableCaching

<br>

```java
@EnableCaching
@SpringBootApplication
public class FestaApplication {

    public static void main(String[] args) {
        SpringApplication.run(FestaApplication.class, args);
    }
}
```

<br>

맨 처음으로 어플리케이션에서 `main()` 메서드가 있는 `SpringBootApplication` 파일에 `@EnableCaching`을 추가하여 어플리케이션 내에
캐싱을 이용하겠다는 명시를 하도록 합니다.

<br>
<br>

### Redis Configuration

프로젝트 내에서 Redis Configuration을 작성한 파일에 `Cache Manager` 를 구현하여 캐싱 사용을 위한 설정코드를 입력합니다.

<br>

```java
@Configuration
public class RedisConfig {

    @Value("${spring.redis.host}")
    private String redisHost;

    @Value("${spring.redis.port}")
    private int redisPort;


    /*
        Lettuce: Multi-Thread 에서 Thread-Safe한 Redis 클라이언트로 netty에 의해 관리된다.
                 Sentinel, Cluster, Redis data model 같은 고급 기능들을 지원하며
                 비동기 방식으로 요청하기에 TPS/CPU/Connection 개수와 응답속도 등 전 분야에서 Jedis 보다 뛰어나다.
                 스프링 부트의 기본 의존성은 현재 Lettuce로 되어있다.

        Jedis  : Multi-Thread 에서 Thread-unsafe 하며 Connection pool을 이용해 멀티쓰레드 환경을 구성한다.
                 Jedis 인스턴스와 연결할 때마다 Connection pool을 불러오고 스레드 갯수가
                 늘어난다면 시간이 상당히 소요될 수 있다.
     */
    @Bean
    public RedisConnectionFactory redisConnectionFactory() {
        return new LettuceConnectionFactory(new RedisStandaloneConfiguration(redisHost, redisPort));
    }


    /*
        RedisTemplate: Redis data access code를 간소화 하기 위해 제공되는 클래스이다.
                       주어진 객체들을 자동으로 직렬화/역직렬화 하며 binary 데이터를 Redis에 저장한다.
                       기본설정은 JdkSerializationRedisSerializer 이다.

        StringRedisSerializer: binary 데이터로 저장되기 때문에 이를 String 으로 변환시켜주며(반대로도 가능) UTF-8 인코딩 방식을 사용한다.

        GenericJackson2JsonRedisSerializer: 객체를 json타입으로 직렬화/역직렬화를 수행한다.
     */
    @Bean
    public RedisTemplate<String, Object> redisTemplate() {
        RedisTemplate<String, Object> redisTemplate = new RedisTemplate<>();
        redisTemplate.setConnectionFactory(redisConnectionFactory());
        redisTemplate.setKeySerializer(new StringRedisSerializer());
        redisTemplate.setValueSerializer(new GenericJackson2JsonRedisSerializer());

        return redisTemplate;
    }

    /*
        Redis Cache 적용을 위한 RedisCacheManager 설정
     */
    @Bean
    public RedisCacheManager redisCacheManager(RedisConnectionFactory redisConnectionFactory) {
        RedisCacheConfiguration redisCacheConfiguration = RedisCacheConfiguration.defaultCacheConfig()
                .serializeKeysWith(RedisSerializationContext
                                    .SerializationPair
                                    .fromSerializer(new StringRedisSerializer()))
                .serializeValuesWith(RedisSerializationContext
                                    .SerializationPair
                                    .fromSerializer(new GenericJackson2JsonRedisSerializer()));

        Map<String, RedisCacheConfiguration> cacheConfiguration = new HashMap<>();
        cacheConfiguration.put(RedisCacheKey.CATEGORY_LIST, redisCacheConfiguration.entryTtl(Duration.ofSeconds(180L)));


        return RedisCacheManager
                .RedisCacheManagerBuilder
                .fromConnectionFactory(redisConnectionFactory)
                .cacheDefaults(redisCacheConfiguration)
                .build();
    }
}
```

<br>

`redisCacheManager` 라는 메서드 안에 Map을 이용하여 캐시 키를 등록해주었고 `entryTtl`은 모든 키에 대한 설정이 아닌 각각의 키 별로 지정할 수 있게
`categoryList` 키 하나에 한정해서 180초로 설정을 해두었습니다. 

<br>
<br>

### Redis Cache Key

Redis 는 데이터는 Key-Value 값으로 저장하기 때문에 키에 대한 이름을 설정해야합니다. 키의 이름은 따로 파일을 만들어 상수화 하여 사용을 했습니다.

<br>

```java
public class RedisCacheKey {

    public static final String CATEGORY_LIST = "categoryList";
}
```

<br>
<br>

### @Cachable

이제 설정은 모두 완료되었으니 비로소 캐싱을 적용하고 싶은 메서드에 `@Cachable` 을 선언하여 해당 메서드에 요청이 오면 
데이터를 캐시할 수 있도록 명시를 합니다. 제 프로젝트에서는 **이벤트 목록/이벤트 카테고리별 조회** 기능에 추가를 했습니다 :)

<br>

```java
@Cacheable(key = "#categoryCode", value = CATEGORY_LIST, cacheManager = "redisCacheManager")
public List<EventDTO> getListOfEvents(PageInfo pageInfo, int categoryCode) {
    return eventDAO.getListOfEvents(pageInfo, categoryCode);
}
```

<br>

이벤트 목록조회/카테고리별 조회를 할 경우 `categoryList` 라는 키의 이름으로 데이터가 캐시됩니다. 

<br>
<br>

### @CacheEvict

캐시 키 값은 `@CacheEvict` 로 지워주지 않는 한, 메모리에 계속 남아있게 됩니다. 저는 새로운 이벤트가 등록이 되면 
refresh를 위해서 **이벤트 등록** 기능에 캐시 데이터를 삭제할 수 있도록 설정하였습니다.

내용 업데이트 기능에도 `@CacheEvict` 를 선언하는 경우도 많이 봤었는데, 개개인의 어플리케이션에서 
많은 테스트를 거쳐 적절한 곳에 선언을 해주면 될 것 같습니다.

<br>
<br>

### 정말 캐싱이 되는지 확인해보자

저는 `Talend API Tester` 라는 툴을 chrome 에서 다운로드 하여 사용했습니다.

<br>

![URL](https://user-images.githubusercontent.com/58355531/107786134-cef7da00-6d90-11eb-8b91-a99a0dca80be.PNG){: .align-center}

<br>

요청할 URL과 파라미터 값을 입력하여 send를 누르면 

<br>

![image](https://user-images.githubusercontent.com/58355531/107779307-82a89c00-6d88-11eb-9ea5-c3a7c56732fd.png){: .align-center}

<br>

캐싱이 적용되기 전에는 **137ms** 의 속도가 출력됩니다. 현재 목록이 많은 것은 아니지만 실제 서비스에선 수 천, 수 만개의 목록을 불러들이기 때문에
이보다 몇 배 이상으로 걸릴 수 있습니다. 

<br>

![image](https://user-images.githubusercontent.com/58355531/107779584-e92dba00-6d88-11eb-823b-aad0ef512df7.png){: .align-center}

캐싱을 적용해 본 후의 속도입니다. 이전엔 137ms 가 출력되었으나 현재는 **23ms** 정도로 대폭 감소한 것을 확인할 수 있었습니다.

<br>

![image](https://user-images.githubusercontent.com/58355531/107779767-285c0b00-6d89-11eb-994c-c5708fd45ba5.png){: .align-center}

<br>

Redis 에서도 `categoryList` 라는 키 이름으로 지정된 캐시 데이터가 잘 저장된 것을 확인할 수 있었습니다! 해당 내용은 커맨트창에서 `keys *` 를 입력하면
현재 캐시된 키 들의 리스트를 확인해볼 수 있습니다.

<br>
<br>

캐시 히트율도 같이 계산을 해보면 더 의미가 있었을 것 같지만, 현재 서버 배포하기 전 단계라 **LRU Simulation** 은 확인할 수 없었습니다.
이 후 배포까지 완료를 한다면 실제 테스트를 해보며 유의미한 캐시 히트율과 그에 따른 `entryTtl` , `Cache evict` 부분을 튜닝 해보면 좋겠다는 생각이 듭니다:)

<br>

끝으로 캐싱에 대한 저의 긴 글 끝까지 읽어주셔서 감사합니다~!

<br>
<br>
<br>

## Project Github URL 

![오구리이미지](https://user-images.githubusercontent.com/58355531/99896015-085c0480-2cd0-11eb-998d-8b8faeb43e17.gif)

[FESTA 프로젝트 Github 보러가기 Click!](https://github.com/f-lab-edu/event-recommender-festa)

<br>
<br>
<br>


## Referenced by

- 캐싱 개요 Amazon Official Document      
  <https://aws.amazon.com/ko/caching/>      
  
- 캐싱 전략 Amazon Official Document     
  <https://docs.aws.amazon.com/ko_kr/AmazonElastiCache/latest/red-ug/Strategies.html>     

- Local Caching: Microsoft Official Document      
  <https://docs.microsoft.com/en-us/windows/win32/fileio/local-caching>     
  
- How to Monitor Redis Performance metrics      
  <https://www.datadoghq.com/blog/how-to-monitor-redis-performance-metrics/>       
  
- Performing an LRU simulation     
  <https://redis.io/topics/rediscli#performing-an-lru-simulation>
  
- Using Redis as an LRU cache      
  <https://redis.io/topics/lru-cache>
  
<br>
<br>
<br>
<br>
<br>
<br>
  



