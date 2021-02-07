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

**"이벤트 목록 조회"** 는 위에서 언급했던 대로 **가장 자주 사용히고 가장 오래 걸릴 수 있는 기능** 입니다. 
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

![image](https://user-images.githubusercontent.com/58355531/107146758-7dfd7580-698d-11eb-9ff0-b1826c979b06.png)

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

![image](https://user-images.githubusercontent.com/58355531/107146827-dfbddf80-698d-11eb-8d19-fccd6fb39056.png)

<br>

**Global Cache** 란 Local Cache와 다르게 서버에 캐시 데이터를 저장합니다. 따라서 여러 대의 서버를 이용할 때 서버 간의 
데이터 동기화가 용이하며 트래픽의 분산 또한 가능합니다. 

**그러나** 서버와 네트워크 통신이 이루어지기 때문에 Local Cache보다 속도가 느리다는 단점이 존재합니다. 

<br>
<br>

### 최종선택은 Global Cache로!

지난 포스팅을 보셨다시피, 현재 저희 FESTA 프로젝트에서는 급증하는 트래픽에 대응하고자 다중 서버 환경을 구축하였습니다. 
캐시 데이터 또한 이러한 다중 서버 환경에서 데이터의 동기화가 필요하기 때문에 Local cache가 아닌 **Global cache** 를 
최종적으로 선택하게 되었습니다. 

<br>
<br>
<br>

## 캐시 히트율(Cache Hit ratio)






## Referenced by

- 캐싱 개요 Amazon Official Document      
  <https://aws.amazon.com/ko/caching/>      
  
- 캐싱 전략 Amazodn Official Document     
  <https://docs.aws.amazon.com/ko_kr/AmazonElastiCache/latest/red-ug/Strategies.html>     

- Local Caching: Microsoft Official Document      
  <https://docs.microsoft.com/en-us/windows/win32/fileio/local-caching>     






