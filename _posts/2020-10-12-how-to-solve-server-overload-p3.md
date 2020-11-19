---
title: "서버 분산 처리 환경에서 세션 불일치를 어떻게 해결할까? Part 2"    
layout: single    
read_time: true    
comments: true   
categories: 
 - project  
toc: true    
toc_sticky: true    
toc_label: contents    
description: 서버 부하를 해결하기 위해 고려한 점은 자세하게 정리하기 위해 생성한 포스팅 - 인메모리 데이터베이스
last_modified_at: 2020-10-12 
---

<br>

지난 ["Part 1"](https://jane096.github.io/project/how-to-solve-server-overload-p2/) 포스팅에서 분산 처리 환경에서 
데이터 불일치 문제를 해결할 Sticky Session, Session-Clustering에 대해서 살펴보았습니다. 

**하지만** Sticky Session은 특정 서버에만 요청이 몰리는 경우 서버의 다운과 데이터의 유실 위험이, Session-Clustering은 
확실하게 분산 처리 환경을 이용할 수 있었지만 클러스터링의 구조상 불필요한 데이터의 복사로 인한 성능의 저하와 그로인해 서버를 늘리기에
한계점이 존재했습니다.

모든 세션이 하나의 DB를 바라보게 함으로써 불필요한 복제가 일어나지 않고, 서버를 계속 추가시켜도 성능의 저하 우려를 해결할 수 있는 **In-memory Database**는 어떤지 한번 살펴보겠습니다.

<br>
<br>
<br>

## In-memory Database 란?

![인메모리이미지](https://user-images.githubusercontent.com/58355531/97765291-dafabb80-1b54-11eb-8869-4de698a061f8.PNG){: .align-center}

<br>

인 메모리 데이터베이스란 디스크나 SSD에 저장되는 데이터베이스와는 다르게 **"메모리"** 에 데이터 저장 목적으로 사용되는 데이터베이스를 말합니다.
보통 디스크에 저장이 되면 읽고 쓰기가 느리기 때문에 시간이 걸리게 되는데 이러한 응답 시간을 줄이고자 설계된 것이 인 메모리 데이터베이스 
입니다. 

하지만 메모리에 저장되기 때문에 서버가 다운되거나 동작 과정 중에 데이터가 유실될 수 있는 가능성이 큽니다. 그래서 **Redis**의 경우 이럴 때를  대비하여 동작 중에 로그나 스냅샷(snapshots)를 이용하여 디스크에 데이터를 저장합니다. 

**인 메모리 데이터베이스** 의 사용이 적합한 경우는 microsecond 단위의 응답시간을 요구하는 애플리케이션이나 실시간 분석, 세션 저장, 게임 순위 선정 등 트래픽이 비교적 많이 몰리는 상황에 적절하다고 합니다. 

또한 분산 처리 환경에서 여러 세션이 하나의 DB를 바라보게 설계가 가능하기 때문에 Session-clustering에서 단점으로 꼽히던 **데이터의 복사로 인한 성능 저하** 가 해결될 수 있습니다.

<br>
<br>
<br>

## In-memory Database의 종류

![db종류](https://user-images.githubusercontent.com/58355531/95645677-6ce64a00-0afc-11eb-99a6-32dda4260bdb.PNG){: .align-center}
  > 출처: 위키피디아 List of in-memory Database <https://en.wikipedia.org/wiki/List_of_in-memory_databases>


위키피디아에 의하면 in-memory Database는 이렇게나 많은 종류가 존재한다고 합니다. 조금 더 검색을 해본 결과, 저는 **Redis** 와 **Memcached** 를 고려해보기로 하는데 그 이유는 다음과 같습니다. 

1. 무료 오픈소스 데이터베이스 이기 때문에 라이센스 비용이 대폭 절약된다.
2. sub-milisecond 단위의 높은 응답 속도를 보여주기 때문에 대용량 트래픽을 고려하는 우리 프로젝트에 적절하다.
3. Key-Value 형태로 저장되기 때문에 같은 Key-Value 로 저장되는 세션 데이터를 다루는데 적합하다.

<br>

> 이러한 공통점을 가진 **Redis**와 **Memcached** 중에 어떤 것을 사용하는데 더 좋을지 차이점을 분석해보았습니다.

<br>
<br>
<br>

## Redis VS Memcached

인 메모리 데이터베이스 이면서, 세션을 저장하는데 적합한 key-value 형태로 관리되고 속도도 빠른 이 두 데이터베이스의 차이점은 무엇일까요?

<br>
<br>

### Multi-Thread VS Single Thread

Memcached는 멀티쓰레드를 지원하지만 Redis는 싱글스레드를 지원합니다. Memcached는 쓰레드를 이용하여 CPU를 통해 Scaling을 진행하며 각각의 쓰레드마다 **concurrent connection(동시에 TCP 커넥션이 가능한 수)** 을 관리하고 있습니다. 또한, **[libevent](https://en.wikipedia.org/wiki/Libevent)** 라는 비동기 이벤트 알림 라이브러리를 사용하여 비교적 손쉽게 scaling을 가능하게 하기 때문에 각각의 쓰레드는 많은 클라이언트를 관리할 수 있습니다. 

Memcached에서는 기본적으로 4개의 쓰레드를 기본으로 할당을 해주며 너무 많은 쓰레드(예를 들어 80개 이상)를 사용할 경우 쓰레드를 적게 쓰는 것 보다 오히려 성능은 크게 저하될 수 있으니 주의가 필요하다고 합니다. Memcached를 사용할 경우, 성능 향상 목적으로 쓰레드를 계속 늘려도 오히려 성능이 크게 저하 될 수 있기 때문에 많은 고민이 필요해 보입니다. 

Redis는 싱글스레드를 지원하며, 싱글스레드를 지원함으로써 많은 사용자들이 **"그럼 병목현상을 일으키는 것 아닌가?""** 하는 의문이 [Common FAQ](https://redis.io/topics/faq#redis-is-single-threaded-how-can-i-exploit-multiple-cpu--cores) 질문 중 하나로 올라와있습니다. 공식문서의 답은 병목현상 해결 목적으로 CPU를 멀티 코어로 바꿔도 Redis에게 크게 영향이 미치지 않고 오히려 메모리나 네트워크에 의해 영향을 더 많이 받는다고 합니다. 그리고 **파이프라인(Piplining)** 이 구축된 Redis는 Linux 위에서 1초에 1백만 건 이상을 다룰 수 있기에 애플리케이션의 속도가 O(N), O(log N) 될 때 CPU를 크게 잡아먹지 않는다고 합니다. 

<br>
<br>

### 장애 극복

<br>

![replication](https://user-images.githubusercontent.com/58355531/95860918-adafbe80-0d9b-11eb-9e87-25d17e8c9894.PNG){: .align-center}

<br>

Redis는 Replication을 전혀 지원하지않는 Memcached와 달리 **master-slave** 관계의 replication을 지원합니다. 이러한 복제 방식으로 Redis는 스스로 장애 극복이 가능합니다. 반면에 Memcached는 복제를 지원하지 않고 노드 분산을 통한 장애 완화 방식을 지원합니다. 

Redis는 `master` 라는 인스턴스에 모든 데이터를 저장하고 `slave` 인스턴스에 같은 데이터를 복제해 저장합니다. 만약 `master`가 다운이 되어버리면 자동으로 알고리즘에 의해 다른 `slave`가 `master`로 지정되기 때문에 사용자들은 **서비스의 중단 없이** 이용할 수 있습니다. 

**그러나** Redis의 경우 Master-Slave 복사과정이 모두 비동기 방식으로 이루어져 높은 퍼포먼스를 보여줄 수 있지만, Master는 Slave로 복사하는 시간을 기다려주지 않기 때문에 데이터가 완벽히 복사되지 않을 수도 있습니다. 그래서 **"WAIT"** 이란 명령어를 통해 필요하다면 동기방식으로 복사를 진행할 수 있지만 성능이 떨어질 수 있으며 완벽한 일관성을 유지하지는 못한다고 합니다. 

<br>

**Memcached**의 경우 Redis와 같은 복제 방식을 지원하지는 않지만 데이터를 여러 노드로 분산하는 방식으로 장애를 최소화 합니다. 노드를 많이 생성하여 데이터를 더 잘게 분할하면 하나의 노드에 장애가 발생하더라도 그 손실을 최소화 시켜줍니다. 예를 들어, 10개의 노드에 분산하였다면 하나의 노드에 약 10% 가량의 데이터를 저장합니다. 그렇다면 장애가 발생했을 경우 그 손실은 10%가 된다는 의미입니다. 같은 양의 데이터를 3개의 노드에 분할했을 경우, 노드 하나가 다운된다면 그 손실은 33% 가량 증가하게 됩니다. 그래서 공식문서에서는 장애를 완화시킬려면 더 많은 노드로 분산하도록 권장하고 있습니다. 

**그러나** 데이터의 손실을 완벽히 해결할 수 없다는 단점이 존재하며 서비스의 중단과 데이터의 손실을 **최소화** 한다는 개념이기 때문에 안정적인 장애극복의 방식을 채택하고 있지 않습니다. 

<br>
<br>
<br>

## 나의 선택은 Redis로!

위의 비교를 통해서 FESTA 프로젝트에 어울리는 인 메모리 데이터베이스는 뭔지 고민해보았습니다. 최종적으로 저는 **Redis** 를 이용하기로 합니다. 

그 이유는 우선 `Master-Slave Replication`을 통한 장애극복으로 **서비스의 중단** 을 피할 수 있다는 점입니다. 데이터의 손실을 완전히 막을 수 없지만 Memcached 보다 확연히 적습니다. 

**AWS**의 공식문서에도 Redis는 대용량 트래픽을 위해 설계되었고, Redis의 대한 장점을 Memcached 보다 자세히 서술해두었습니다. 우리 프로젝트는 계속 대용량 트래픽에 대해 고민해왔고 데이터의 일관성을 위해 완벽하게 동작하지 않는다는 것을 감안해도 더 좋은 선택이라고 생각했습니다.

이 선택을 내리는데 결정적이었던 것은 **Spring** 은 Redis에 대해서만 지원한다는 점이었습니다. Spring 공식문서에도 왜 써야하는지에 대해 자세히 기술되어 있을 정도로 **스프링 부트**로 개발 중인 우리 프로젝트에 제격이라고 판단하였습니다. 

> Redis를 선택함으로써 서버 분산 처리 환경에서 세션을 효율적으로 관리할 수 있게 되었습니다! 3편으로 나뉜 긴 글을 읽어주셔서 감사합니다~!

<br>
<br>
<br>

## 번외 

> 제가 **"번외"** 라고 한 이유는 **데이터 타입** 이 메모리 가용성에 영향을 미치는 경우는 캐싱 솔루션으로 이용했을 때 입니다. **데이터의 영속성** 또한 세션 스토리지 용도로 사용한다면 우리 프로젝트를 개발하는데 고려하지 않아도 되는 부분이기에 이 부분은 따로 번외로 빼두었습니다.  

<br>

### 데이터의 영속성 관리

Memcached는 데이터의 영속성을 관리하기 위해 지원되는 기능이 없습니다. 반면에 Redis는 어느 한 시점마다 **스냅샷(Snapshots)** 을 이용해 데이터를 디스크 저장소에 보관합니다. 이렇게 보관된 데이터는 추후 데이터의 이상이 왔을 때 복구용으로 사용할 수 있습니다. 

또한, 스냅샷 기능과 별도로 **AOF log** 기능도 지원합니다. **AOF log** 란 서버로 부터 받은 커맨드 명령어들을 로그 파일로 수집하는 방식입니다. 만약 로그의 양이 방대해지면 백그라운드에서 다시 쓰는 것이 가능합니다. 

**그러나** 영속성 관리에 탁월한 스냅샷과 AOF 기능에도 단점이 존재합니다. 스냅샷의 경우, 일반적으로 5분마다 스냅샷을 생성하는데 어떠한 이유로든 올바르게 종료되지 않고 Redis가 중지되었다면 최신 데이터 분량이 유실될 수 있다고 합니다. 

**AOF** 의 경우, 동기식 함수(fsync)를 사용하기 때문에 스냅샷보다 오래 지속되지만 느릴 수 있다고 합니다. 그리고 AOF는 다시 로딩할 때 완전히 똑같은 데이터셋을 불러들이지 못할 때가 있어 가끔씩 특정 커맨드([BRPOPLPUSH](https://redis.io/commands/brpoplpush) 같은) 에 대해 버그를  합니다. 이 버그는 매우 드문 경우이고 Redis 측에서도 여러번의 테스트 후 괜찮다는 판단을 내렸다고 합니다만, 그래도 단점으로 기술되어 있는 만큼 생각해볼 여지가 있습니다. 

<br>
<br>

### Redis는 데이터 타입을 지원한다

String만 지원하는 Memcached와 달리 Redis는 Hashes, Lists, Sets 등 자료구조를 지원합니다. 이렇게 데이터 타입을 지원함으로써 Memcached 보다 메모리를 효율적으로 관리할 수 있습니다. 어떻게 그럴 수 있을까요?

Key-Value의 구조를 지원하는 Redis는 메모리의 효율을 위해 **해시**를 이용하여 그 값을 추상화 시킵니다. 

예를 들어 1234 이라는 숫자의 객체를 저장할려고 한다면 Redis는 이를 "12"라는 key 이름과 와 "34" 라는 필드이름 으로 분할합니다. 그리고 모든 해시는 100개의 필드를 포함할 수 있기 때문에 CPU와 Memory의 공간절약을 직관적으로 확인이 가능합니다. 
Redis 공식문서에 의하면 Memcached는 String만을 지원하기 때문에 메모리 가용성 측면에서는 Memcached 보다 훨씬 앞서있다고 합니다.

<br>
<br>
<br>

### Referenced by

- What is an In-Memory Database?    
  <https://aws.amazon.com/ko/nosql/in-memory>
  
- Comparing Redis and Memcached     
  <https://aws.amazon.com/ko/elasticache/redis-vs-memcached/>

- Redis Official documentation   
  <https://redis.io/documentation>

- Spring docs: Why Spring Data Redis?   
  <https://docs.spring.io/spring-data/data-redis/docs/current/reference/html/#reference>
  
- Mitigating Failures    
  <https://docs.aws.amazon.com/AmazonElastiCache/latest/mem-ug/FaultTolerance.html>
  
- Memcached Github WIKI         
  <https://github.com/memcached/memcached/wiki/ConfiguringServer#threading>
  
<br>
<br>

### Project Github URL
[FESTA 프로젝트 Github 보러가기 Click!](https://github.com/f-lab-edu/event-recommender-festa)

<br>
<br>
<br>
<br>
<br>
<br>
