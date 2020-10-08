---
title: "대용량 트래픽 로그인 기술을 위해서 서버의 부하를 어떻게 해결할까? Part 2"    
layout: single    
read_time: true    
comments: true   
categories: 
 - project  
toc: true    
toc_sticky: true    
toc_label: contents    
description: 서버 부하를 해결하기 위해 고려한 점은 자세하게 정리하기 위해 생성한 포스팅 - Sticky Session, Session-Clustering, in-memory Database
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

![현재상황](https://user-images.githubusercontent.com/58355531/95462384-6946ac00-09b2-11eb-8666-82413f5224ed.PNG)

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
라는 기술이 수행하게 됩니다. (로드 밸런서도 유형에 따라 3가지로 구분한다고 합니다. 제가 링크로 건 유형은 Classic 버전입니다.)

로드 밸런서는 요청에 따라 보내야 할 인스턴스를 찾기 위해서 **AWSELB** 라는 특별한 쿠키를 사용합니다. 로드 밸런서가 한 요청을 받게 되면 
우선적으로 요청에 쿠키정보가 있는지 부터 확인합니다. 쿠키의 정보를 확인했다면, 해당 요청은 로드 밸런서에 의해 해당 쿠키가 생성되어 있는
인스턴스로 보내지게 됩니다. 만약 존재하지 않는 쿠키라면 로드 밸런서의 알고리즘에 의해 선택된 다른 인스턴스에 쿠키가 생성되어 
다음에 똑같은 요청이 오면 같은 경로로 맵핑시켜 줄 수 있도록 합니다. 

<br>

이렇게 보자면 **Sticky Session** 으로 데이터 불일치 문제를 해결할 수 있을 것 같지만 단점도 존재합니다. 

<br>

### 하나의 서버로 요청이 몰릴 수 있다!

**Sticky Session** 은 요청받은 쿠키정보를 통해 특정 서버로 맵핑을 한다고 언급했습니다. 이렇게 된다면 `서버1`로 맵핑 된 요청들이 
갑자기 많은 요청을 한꺼번에 보내 `서버1` 이 다운되는 상황이 발생할 수도 있습니다. 

이렇게 `서버1` 이 다운되어 버리면 해당 서버에 맵핑 된 요청들은 아무런 동작을 수행할 수 없게 됩니다. 서버의 장애 발생으로
저장되어 있던 모든 데이터 정보 또한 사라지게 됩니다. 

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




<br>
<br>
<br>

#### Referenced by 

- What is a Classic Load Balancer?  
<https://docs.aws.amazon.com/elasticloadbalancing/latest/classic/introduction.html>

- Configure sticky sessions for your Classic Load Balancer   
<https://docs.aws.amazon.com/elasticloadbalancing/latest/classic/elb-sticky-sessions.html>

- Why you should never use Sticky sessions   
<https://dev.to/gkoniaris/why-you-should-never-use-sticky-sessions-2pkj>

<br>
<br>
<br>
<br>
<br>
<br>
