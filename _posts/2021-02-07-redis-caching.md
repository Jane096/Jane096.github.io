---
title: "Redis Cache 기능을 활용한 성능 개선 이야기 part 1"    
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

서비스의 점차 규모가 커지고 그에 따라 사용자의 수는 늘어나게 됩니다. 증가한 사용자 만큼 수 천, 또는 수 만개의 이벤트 목록을 불러올 때 
매번 DB까지 도달하여 읽어오게 된다면 성능적인 이슈가 발생할 수 밖에 없었습니다. 

<br>
<br>

## 캐싱을 적용해보게 된 이유

**"이벤트 목록 조회"** 는 위에서 언급했던 대로 **가장 자주 사용하는 기능** 입니다. 
또한 업데이트가 아주 빈번하게 일어나는 것이 아니라서 반복적으로 동일한 결과를 출력하는 경우가 많습니다. 

<br>

이러한 특징을 가진 경우, 한번 읽어온 조회결과를 복사하여 메모리에 저장해둔다면 매번 DB로부터 호출하지 않고 메모리로 부터 값을 가져오기 때문에 DB의 부하를 줄이며동시에 속도 향상의 결과를 볼 수 있기 때문에 조회 기능에 캐싱을 적용해보기로 결정하게 되었습니다!

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

**그러나** 각각의 was 마다 캐시를 저장하기 때문에 하나의 장비에만 저장이 됩니다. 그리고 서버에 저장하는 것이 
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

사용자의 세션이 어떤 서버에 저장이 되어도 항상 같은 데이터를 조회할 수 있어야 합니다. 만약 **Local Cache** 를 이용하게 된다면, 해당 서버에
저장된 이벤트 목록만 조회됩니다.     

예를 들어 `이벤트1` 이라는 게시글이 `서버1` 에 저장이 되어있는 상황에서 `서버2` 를 통해 접속한 사용자는
`이벤트1` 이라는 목록을 볼 수 없게 된다는 의미입니다.      

캐시 데이터 또한 이러한 다중 서버 환경에서 데이터의 동기화가 필요하기 때문에 Local cache가 아닌 **Global cache** 를 
최종적으로 선택하게 되었습니다. 

<br>
<br>
<br>

## 캐시 히트율(Cache Hit ratio)

**캐시 히트율** 이란, 캐시된 데이터를 요청할 때 해당 키 값이 메모리에 존재하여 얼마만큼의 비율로 잘 찾았는지에 대한 여부를 말합니다. 캐시 데이터를 잘 찾았다면 
`Cache hits` 이라고 하며 반대로 캐시가 존재하지 않아 찾지 못했을 경우 `Cache Misses` 라고 합니다. 

**캐시 히트율** 을 계산할 때에는 `cmd` 창에서 `redis-cli` 로  Redis에 접속하여 `info stats` 명령어를 입력하면 아래와 같은 사진처럼 정보가 뜨게 됩니다.

<br>

![image](https://user-images.githubusercontent.com/58355531/107664713-0eaabd00-6cd0-11eb-8107-ad1886440ef7.png){: .align-center}

<br>

이 때, `Keyspace-hits` 가 `Cache-hits` 을 의미하며 `Keyspace-misses` 가 `Cache Misses` 를 의미하여 캐시 히트율을 구하는 산술식은

> 캐시 히트율 = Keyspace-hits / (Keyspace-hits + Keyspace-misses)

이며, 해당 공식으로 캐시 히트율을 구할 수 있습니다.

<br>

만약, 캐시 히트율이 현저히 낮다면 그 의미는 **캐시된 데이터를 제대로 찾지 못하고 있다** 는 뜻이며 그만큼 디스크에서 데이터를 읽어오는 비율이 높다는 것을 의미합니다. 디스크에서 데이터를 읽어올 경우, I/O가 많이 발생하기 때문에 그만큼 성능에 큰 영향을 끼칠 수 밖에 없습니다. 

반대로, 캐시 히트율이 높다는 것은 데이터베이스에서 읽어오는 비율보다 메모리에 캐시된 데이터를 읽어오는 비율이 높다는 의미이기 때문에 효율적으로 캐시를 사용하고 있다는 반증이 됩니다. 

<br>
<br>

### 캐시 evict 와 entryTtl 

위에서 언급했던 바와 같이 캐시 히트율이 높다면 캐시 데이터를 효율적으로 사용하고 있다는 증거가 된다고 했습니다. 캐시 히트율을 높일려면 
항상 캐시 데이터를 메모리에 저장을 해두면 될 것 같지만, 그렇게 된다면 메모리에 리소스를 계속해서 차지하기 때문에 그것 또한 문제가 될 수 있습니다. 

캐시된 데이터는 적절한 때에 `Cache 지속시간(entryTtl)` 을 설정하여 만료 되도록 하고, `Cache evict`를 하여 메모리 리소스가 차지 않도록 적절한 조절이 필요합니다.

> 그렇다면 얼마만큼의 지속시간이 가장 이상적이며 어떨 때 캐시데이터를 삭제해야할까요?

<br>

**정답은 없습니다..!**

각각의 어플리케이션 마다 사용하는 메모리 크기나 키 값의 수가 똑같을 수가 없고 같은 어플리케이션 내에서도 매번 같을 것을 보장할 수 없습니다.
메모리의 크기와 키 값은 `Cache hits` 이나 `Cache misses` 에 영향을 주는 요소들이기 때문에 정확하게 지속시간을 몇 초로 설정해야 하고, 
언제 캐시를 지워야한다 라는 공식은 없습니다. 자신의 어플리케이션에서 설정한 메모리와 키의 갯수, 그리고 사용자의 패턴을 분석하여 적절한 튜닝이 필요합니다.

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

위의 이미지 처럼 임의로 정한 메모리 크기(현재 100mb)와 키의 갯수(현재 1000만 개) 를 설정해둔다면 매 1초마다 `Cache hits` 와 `Cache misses` 를 수학적 공식에 기반하여 히트율을 분석적으로 보여줍니다. 

**중요한 점은** 만약 메모리 크기를 지정하지 않고 캐시 키 값을 사용 한다면 모든 키 값을 메모리에 저장할 수 있기 때문에 캐시히트율은 항상 **100%** 가 나올 수 있습니다. 또한, 메모리 크기를 지정하지 않고 많은 갯수의 캐시 키를 사용한다면 컴퓨터 내의 모든 RAM 을 사용하게 되기 때문에 메모리 크기에 대한 설정이 꼭 필요하다고 합니다.     

<br>
<br>

## 정리하며

이렇게 캐싱을 적용해보게 된 이유, 캐시를 적용하기 전 Local Cache/Global Cache에 대해 알아보고 캐시 만료시간과 키 삭제, 그리고 캐시 히트율을 높이기 위한 방안 등을 정리해보았습니다. 이제 정리한 글을 바탕으로 실제 프로젝트에선 어떻게 적용이 되었는지는 다음편 part 2에서 자세히 소개해보도록 하겠습니다!

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
  



