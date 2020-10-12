---
title: "대용량 트래픽을 고려한 서버 분산 처리 환경에서 데이터의 불일치를 어떻게 해결할까? Part 1"    
layout: single    
read_time: true    
comments: true   
categories: 
 - project  
toc: true    
toc_sticky: true    
toc_label: contents    
description: 서버 부하를 해결하기 위해 고려한 점은 자세하게 정리하기 위해 생성한 포스팅 - Sticky Session 비교분석
last_modified_at: 2020-10-07 
---

<br>
<br>

지난 포스팅에서는 대용량 트래픽 환경에서 로그인 기능을 개발하며 부딪혔던 문제점과 그에 따른 해결방법으로 
**Scale-up** 과 **Scale-out** 에 대해 비교해보았습니다.

최종적으로 **Scale-out** 을 선택했지만 여전히 **데이터 불일치** 의 이슈가 그대로 남아있었습니다. 

현재 Scale-out을 그대로 적용한다면 아래의 그림과 같은 상황이 됩니다. 

<br>
<br>

![현재상황](https://user-images.githubusercontent.com/58355531/95462384-6946ac00-09b2-11eb-8666-82413f5224ed.PNG){: .align-center}

<br>

> 데이터베이스 아이콘을 세션이라고 생각해주시면 됩니다..^^

<br>

보통 WAS 1개에는 세션1개만이 생성되도록 설계되어있습니다. 세션은 위의 그림과 같이 독립적으로 작동을 하며
서로의 데이터를 공유받을려면 정해진 세션으로만 요청이 가게 하는 **고정세션(Sticky Session)** 이나 **클러스터링 기술(Session-Clustering)** 
처럼 여러 세션을 묶어 하나의 세션처럼 동작하게 하는 방법을 이용해야 합니다. 

<br>
<br>
<br>

## 고정 세션(Sticky Session) 이란?

클라이언트로 받은 요청을 항상 고정된 세션으로만 보낸다는 개념입니다. 예를 들어, `username1` 이라는 사람의 요청을 
`서버1` 이 받았다면 앞으로 `username1` 의 대한 모든 요청이 `서버1` 에게로만 가게 설정하는 것입니다. 

<br>

![고정세션](https://user-images.githubusercontent.com/58355531/95469513-b464bd00-09ba-11eb-8fc7-fc69d9f4f107.PNG)

<br>

이렇게 고정된 세션으로 요청이 갈 수 있도록 중간에 이정표와 같은 역할이 필요한데요, 그것을 [로드 밸런서](https://docs.aws.amazon.com/ko_kr/elasticloadbalancing/latest/classic/introduction.html)
라는 기술이 수행하게 됩니다. 

   > 로드 밸런서도 유형에 따라 3가지로 구분한다고 합니다. 제가 링크로 건 유형은 Classic 버전입니다.

<br>

로드 밸런서는 요청에 따라 보내야 할 인스턴스를 찾기 위해서 쿠키정보를 활용합니다. 로드 밸런서가 한 요청을 받게 되면 
우선적으로 요청에 쿠키정보가 있는지 부터 확인하고 쿠키의 정보를 확인했다면, 해당 요청은 로드 밸런서에 의해 해당 쿠키가 생성되어 있는
인스턴스로 보내지게 됩니다. 만약 존재하지 않는 쿠키라면 로드 밸런서의 알고리즘에 의해 선택된 다른 인스턴스에 쿠키가 생성되어 
다음에 똑같은 요청이 오면 같은 경로로 맵핑시켜 줄 수 있도록 합니다. 

<br>

이렇게 보자면 **Sticky Session** 으로 데이터 불일치 문제를 해결할 수 있을 것 같지만 단점도 존재합니다. 

<br>

### 하나의 서버로 요청이 몰릴 수 있다!

**Sticky Session** 은 요청받은 쿠키정보를 통해 특정 서버로 맵핑을 한다고 언급했습니다. 만약 `서버1`로 맵핑 된 사용자들이 
다른 사용자들과 비교해서 왕성한 활동으로 갑자기 한꺼번에 요청을 보내게 되면 `서버1` 이 다운되는 상황이 발생할 수도 있습니다. 

이렇게 `서버1` 이 다운되어 버리면 해당 서버에 맵핑 된 사용자들은 우리 서비스를 이용할 수 없게 됩니다. 서버의 장애 발생으로
세션에 저장되어 있던 모든 데이터 정보 또한 사라지게 됩니다. 

<br>
<br>

### 분산 환경을 완벽히 사용할 수 없다!

**Sticky Session** 특성 상 다른 인스턴스로 요청을 보낼 수 없기 때문에 다른 서버가 받아서 수행할 수 있는 능력을 활용할 수 없습니다. 
즉, `서버1` 에 요청이 몰려도 `서버1` 이 수행하도록 강제할 수 밖에 없다는 의미가 됩니다.
이렇게 된다면 분산 환경을 설계한 의미가 다소 퇴색될 수 밖에 없습니다.

> 그렇다면 다른 대안으로 각기 다른 세션을 하나의 세션으로 묶어주는 클러스터링 기술을 어떨까요?

<br>
<br>
<br>

## Session-Clustering 이란?

![클러스터링](https://user-images.githubusercontent.com/58355531/95583792-a32db600-0a77-11eb-9a45-0c99ff0ff110.PNG)

<br>

**Session Clustering** 이란 각각의 WAS에 있는 세션을 묶어 동일한 세션으로 관리하는 방법입니다. **Sticky-Session** 에서 언급되었던 
분산 환경 활용을 제대로 못하는 단점을 클러스터링 기술에서는 어느 정도 해결이 가능합니다. 세션 클러스터링에서는 하나의 서버가 
다운되어 버리면 해당 WAS의 데이터를 다른 WAS가 대신 관리할 수 있기 때문에 데이터의 유실이나 전면 장애의 가능성을 낮출 수 있습니다. 

**Session Clustering** 의 경우 사용하는 WAS 마다 클러스터링 하는 방법이 다르기 때문에 현재 사용하고 있는 WAS의 공식 문서를 
참고하시는게 좋습니다.

<br>
<br>

![tomcat버전](https://user-images.githubusercontent.com/58355531/95581888-bc813300-0a74-11eb-9e14-ca71a1834cee.PNG){: .align-center}

저는 **스프링 부트**로 개발을 진행하고 있고 스프링 부트에는 Tomcat이 내장되어 있기 때문에 Tomcat 공식문서를 참고하여 작성하였습니다.
위의 사진과 같이 제 프로젝트에는 Tomcat 9가 이미 들어와있네요. 이 글에서는 [Tomcat 9 Clustering](https://tomcat.apache.org/tomcat-9.0-doc/cluster-howto.html)을 기준으로 설명하겠습니다.

<br>
<br>

### All-to-all Replication

![alltoallreplication](https://user-images.githubusercontent.com/58355531/95587119-9bbcdb80-0a7c-11eb-9d8b-1cdbeb099f24.PNG){: .align-center}

<br>

톰캣은 여러 Session manager 중에서 `DeltaManager` 와 `BackupManager`를 이용한 클러스터링 방법을 소개하고 있습니다. 먼저 `DeltaManager` 를 이용하여 한 세션의 상태를 모두 다른 세션으로 복사하는 **All-to-all Replication** 을 먼저 설명하도록 하겠습니다. 

**All-to-all** 방식은 모든 세션들이 동일한 데이터를 가질 수 있도록 모든 데이터를 복사합니다. 때문에 데이터의 불일치 문제를 해결할 수 있고 하나의 서버가 다운되어도 다른 서버가 대신 작동하기 때문에 전면 장애 우려도 해결됩니다. 그러나 공식문서에서는 적은 갯수의 세션을 클러스터링 했을 땐 괜찮지만 **4개 이상의 세션** 을 클러스터링 했을 경우 추천하지 않는 방식이라고 합니다. 

왜냐하면 **"모든"** 데이터를 각각의 Tomcat 노드에게 전달해야 해야 하고 만약 배포하는 노드가 아닐 경우에도 복사를 진행하기 때문에 
불필요하게 메모리를 차지하게 되고, 결국은 성능이 떨어질 수 있기 때문입니다. 

<br>
<br>

### Primary-Secondary Session Replication

이러한 문제점을 극복하기 위해 다른 방법으로 `BackupManager`를 이용한 세션 복사 방법이 있습니다.
이 방식은 오직 배포된 어플리케이션의 노드에만 복사를 진행하고, 하나의 백업 노드에 세션 데이터를 복사하기 때문에 불필요한 
작업을 줄일 수 있습니다. 사용하는 세션의 갯수가 늘어날 수록 `BackupManager`를 사용하는 것은 필연적일 수 밖에 없습니다. 

<br>

![백업매니저](https://user-images.githubusercontent.com/58355531/95598032-61f2d180-0a8a-11eb-905c-9b05e1283aa7.PNG)

<br>

클라이언트로 부터 요청을 로드 밸런서에 의해 받은 서버에 세션을 생성하고 해당 인스턴스는 **Primary node**로 선정됩니다. 그리고 다른 서버들 중에 하나가 **Backup node(=Secondary node)** 로 선정이 되고 나머지는 **Proxy node** 가 됩니다. 

`Primary node`의 세션 데이터는 오직 `Backup node`에만 복사가 되며 나머지 `Proxy`에는 세선 아이디와 `Primary`, `Seconday node`의 주소값만을 가지고 있게 됩니다. 만약 `Proxy`로 요청이 들어온다면 `Proxy`는 `Primary`에게 해당 세션 아이디의 데이터를 요청하여 받아오게 되고
최종적으로 클라이언트는 모든 세션으로 부터 동일한 값을 불러올 수 있어 **데이터 불일치**의 문제점이 해결됩니다. 

**하지만** `Primary` 노드가 다운이 되어 버리고 로드 밸런서에 의해 선정된 다른 노드에 세션 정보가 없다고 가정을 해봅시다.
해당 노드는 데이터를 불러오기 위해 다른 서버에게 세션 데이터가 있는지 물어보고 데이터가 존재하는 서버에게 응답을 받습니다. 이 후 모든 데이터를 복사하고 해당 노드는 `Primary` 로써 수행을 하게 됩니다. 

만약 대용량 트래픽의 상황에서 수 많은 서버를 사용하게 되면 빈번하게 복사가 일어나게 되고 결국은 **성능의 저하**로 이어질 수 있습니다. 

<br>

> 이렇게 불필요하게 복제가 많이 발생한다면, 결국은 서버 확장의 한계에 도달하게 되고 Scale-out의 무한대로 서버를 늘릴 수 있는 장점을 활용하지 못하게 됩니다. Session-Clustering 처럼 독립된 세션을 하나로 묶어 관리할 수 있으면서 불필요한 복사가 일어나지 않는 **in-memory Database** 는 어떤 장점과 단점을 가지고 있을지 다음 포스팅에서 자세히 소개하도록 하겠습니다! 


<br>
<br>
<br>

### Referenced by 

- What is a Classic Load Balancer?  
  <https://docs.aws.amazon.com/elasticloadbalancing/latest/classic/introduction.html>

- Configure sticky sessions for your Classic Load Balancer   
  <https://docs.aws.amazon.com/elasticloadbalancing/latest/classic/elb-sticky-sessions.html>

- Why you should never use Sticky sessions   
  <https://dev.to/gkoniaris/why-you-should-never-use-sticky-sessions-2pkj>

- Apache Tomcat 9 Clustering/Session Replication How-To Official documentation   
  <https://tomcat.apache.org/tomcat-9.0-doc/cluster-howto.html>
  
- About Tomcat BackupManager   
  <https://tkstone.blog/2018/09/19/about-tomcat-backupmanager/>
  
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
