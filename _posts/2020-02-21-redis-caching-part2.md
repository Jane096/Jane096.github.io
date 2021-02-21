---
title: "Redis Cache 기능을 활용한 성능 개선 이야기 Part 2"    
layout: single    
read_time: true    
comments: true   
categories: 
 - project  
toc: true    
toc_sticky: true    
toc_label: contents    
description: 실제로 프로젝트에 적용해보고 어떻게 성능이 개선되었는지 정리한 포스팅입니다.
last_modified_at: 2020-10-12 
---

<br>

지난 [part 1 포스팅](https://jane096.github.io/project/redis-caching/) 에서 캐싱을 적용하기 전 어떤 점을 알아보고 고려해봤는지 정리를
해보았습니다. 이제 실제 프로젝트에 적용을 해보고 어떻게 성능이 개선되었는지 자세히 정리해보도록 하겠습니다 :)

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

`@EnableCaching` 을 적용하게 되면 `@Cacheable` 이라는 어노테이션이 명시된 메서드가 실행될 때 내부적으로 Proxy, AspectJ 기반 어드바이스를
`CacheInterceptor` 와 연결하여 Spring에서 캐시 관리에 필요한 구성요소로 등록을 하게 됩니다.

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

`redisCacheManager` 라는 메서드 안에 Map을 이용하여 캐시 키를 등록해주었고 `entryTtl`은 모든 키에 대한 설정이 아닌 각각의 키 별로 지정할 수 있게 `categoryList` 키 하나에 한정해서 180초로 설정을 해두었습니다. 

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

![image](https://user-images.githubusercontent.com/58355531/107779767-285c0b00-6d89-11eb-994c-c5708fd45ba5.png){: .align-center}

아래에 다시 한번 서술하겠지만, 이벤트 목록조회/카테고리별 조회를 할 경우 위와 같이 `categoryList::(실제 카테고리 코드)` 라는 우리가 지정한 키의 이름으로 데이터가 캐시됩니다. 

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

Redis 에서도 `categoryList` 라는 키 이름으로 지정된 캐시 데이터가 잘 저장된 것을 확인할 수 있었습니다! 해당 내용은 커맨트창에서 `keys *` 를 입력하면 현재 캐시된 키 들의 리스트를 확인해볼 수 있습니다.

<br>
<br>

## 정리하며

캐시 히트율도 같이 계산을 해보면 더 의미가 있었을 것 같지만, 현재 서버 배포하기 전 단계라 **LRU Simulation** 은 확인할 수 없었습니다.
이 후 배포까지 완료를 한다면 실제 테스트를 해보며 유의미한 캐시 히트율과 그에 따른 `entryTtl` , `Cache evict` 부분을 튜닝 해보면 좋겠다는 생각이 듭니다:)

끝으로 캐싱에 대한 저의 긴 글 끝까지 읽어주셔서 감사합니다~!

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
  
- Performing an LRU simulation: Redis Official Documentation         
  <https://redis.io/topics/rediscli#performing-an-lru-simulation>
  
- Using Redis as an LRU cache: Redis Official Documentation      
  <https://redis.io/topics/lru-cache>
  
- Annotation Type EnableCaching: Spring Framework Official Documentation      
  <https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/cache/annotation/EnableCaching.html>
  
<br>
<br>
<br>
<br>
<br>
<br>
