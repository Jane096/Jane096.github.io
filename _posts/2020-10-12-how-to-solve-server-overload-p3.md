---
title: "대용량 트래픽을 고려한 서버 분산 처리 환경에서 데이터의 불일치를 어떻게 해결할까? Part 2"    
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

인 메모리 데이터베이스란 디스크나 SSD에 저장되는 데이터베이스와는 다르게 **"메모리"** 에 데이터 저장 목적으로 사용되는 데이터베이스를 말합니다.
보통 디스크에 저장이 되면 읽고 쓰기가 느리기 때문에 시간이 걸리게 되는데 이러한 응답 시간을 줄이고자 설계된 것이 인 메모리 데이터베이스 
입니다. 

하지만 메모리에 저장되기 때문에 서버가 다운되거나 동작 과정 중에 데이터가 유실될 수 있는 가능성이 큽니다. 그래서 인 메모리 데이터베이스는
이러한 단점을 커버하기 위해서 동작 중에 로그나 스냅샷(snapshots)를 이용하여 디스크에 데이터를 저장합니다. 

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
3. **AWS** 라는 거대 클라우드 서비스에서 지원한다. 
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

### Redis는 데이터 타입을 지원한다

String만 지원하는 Memcached와 달리 Redis는 Hashes, Lists, Sets 등 자료구조를 지원합니다. 이렇게 데이터 타입을 지원함으로써 Memcached 보다 메모리를 효율적으로 관리할 수 있습니다. 어떻게 그럴 수 있을까요?

Key-Value의 구조를 지원하는 Redis는 메모리의 효율을 위해 **해시**를 이용하여 그 값을 추상화 시킵니다. 

예를 들어 1234 이라는 숫자의 객체를 저장할려고 한다면 Redis는 이를 "12"라는 key 이름과 와 "34" 라는 필드이름 으로 분할합니다. 그리고 모든 해시는 100개의 필드를 포함할 수 있기 때문에 CPU와 Memory의 공간절약을 직관적으로 확인이 가능합니다. 
Redis 공식문서에 의하면 Memcached는 String만을 지원하기 때문에 메모리 가용성 측면에서는 Memcached 보다 훨씬 앞서있다고 합니다.

<br>
<br>

### Multi-Thread VS Single Thread

Memcached는 멀티쓰레드를 지원하지만 Redis는 싱글스레드를 지원합니다. Memcached가 멀티쓰레드 구현이 가능함으로써 Redis보다는 손 쉽게 Scaling이 가능합니다. 

<br>
<br>

### Replication

Redis는 Replication을 전혀 지원하지않는 Memcached와 달리 **master-slave** 관계의 replication을 지원합니다. 자세히 설명하자면, Redis는 `master` 라는 인스턴스에 모든 데이터를 저장하고 `slave` 인스턴스에 같은 데이터를 복제해 저장합니다. 만약 `master`가 다운이 되어버리면 알고리즘에 의해 다른 `slave`가 `master`로 지정되기 때문에 사용자들은 서비스의 중단 없이 이용할 수 있습니다. 

**그러나** Redis의 경우 Master-Slave 복사과정이 모두 비동기 방식으로 이루어져 높은 퍼포먼스를 보여줄 수 있지만 Master는 복사하는 그 시간을 기다려주지 않기 때문에 데이터가 완벽히 복사되지 않을 수도 있습니다. 그래서 **"WAIT"** 이란 명령어를 통해 필요하다면 동기방식으로 복사를 진행할 수 있지만 완벽한 일관성을 유지하지는 못한다고 합니다. 


- What is an In-Memory Database?    
  <https://aws.amazon.com/ko/nosql/in-memory>
  
- Comparing Redis and Memcached     
  <https://aws.amazon.com/ko/elasticache/redis-vs-memcached/>

- Redis Official documentation   
  <https://redis.io/documentation>

- Spring docs: Why Spring Data Redis?   
  <https://docs.spring.io/spring-data/data-redis/docs/current/reference/html/#reference>
