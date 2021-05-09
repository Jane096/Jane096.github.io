---
title: "nGrinder 성능테스트 결과를 통한 로그인 성능개선 과정을 알아보자"    
layout: single    
read_time: true    
comments: true   
categories: 
 - project  
toc: true    
toc_sticky: true    
toc_label: contents    
description: 지난 로그인 기능의 성능테스트 결과를 토대로 병목지점을 찾아 리팩토링한 과정을 정리한 포스팅입니다.
last_modified_at: 2021-05-09   
---

<br>
<br>

## Overview

[지난 포스팅](https://jane096.github.io/project/ngrinder-performance-test/) 에서 처음으로 `nGrinder` 를 서버에 연결하여 한번의 테스트를 진행해보았습니다.
하지만 가상 유저의 수 200명일 때, **초당 트랜잭션 수(이하 TPS)** 는 62.7, Errors **약 2000건 이상** 으로 상당히 불안정한 지표를 받게 되었습니다.

<br>

![image](https://user-images.githubusercontent.com/58355531/116384191-94abbe00-a852-11eb-9bcb-e850cdb31ec2.png){: .align-center}

이미 그래프의 모양새부터 들쭉날쭉 하며 최대 TPS 역시 164로 200명의 유저도 감당하지 못하는 수준입니다. 크나큰 병목 지점이 있는 것으로 파악되기 때문에 
로그인 기능의 리팩토링을 진행하고자 합니다.

<br>
<br>

## 문제점1: Firebase 토큰 가져오기

**Firebase** 에서는 사용자가 로그인에 성공하면 개개인을 식별할 토큰값을 생성해줍니다. Firebase 서버에서 생성을 해주는데 
리팩토링을 하기 전, 저희 로그인 기능은 로그인을 할 때 파일시스템을 이용하여 각자의 유니크한 토큰값을 생성해주고 있었습니다. 

<br>

```java
/**
     * 사용자 로그인 기능
     * Firebase Token 생성 후 로그인한 회원에게 보내야 할 알림여부를 응답을 보냄
     * @param memberLogin
     * @return {@literal List<AlertResponse>}
     */
    @PostMapping("/login")
    public List<AlertResponse> login(@RequestBody MemberLogin memberLogin) {
        String userId = memberLogin.getUserId();
        String password = memberLogin.getPassword();

        memberService.isUserIdExist(userId, password);
        loginService.setUserNo(memberLogin.getUserNo());
        
        //토큰을 생성하는 부분!
        firebaseTokenManager.makeAccessToken(memberLogin.getUserNo());

        List<AlertResponse> loginResponses = alertService.eventStartNotice(memberLogin.getUserNo(), LocalDate.now());
        AlertResponse isUserNeedToChangePw = alertService.changePasswordNotice(memberLogin.getUserNo());

        loginResponses.add(isUserNeedToChangePw);

        return loginResponses;
    }
```

<br>

토큰을 생성하는 `makeAccessToken()` 은 아래와 같이 작성되어 있었습니다.

<br>

```java
/**
     * 접근을 위한 Token을 얻어온 후 Map에 저장하는 메서드
     */
    public void makeAccessToken(long userNo) {

        GoogleCredentials googleCredentials = null;

        try {
            //이 부분을 주목해주세요
            googleCredentials = GoogleCredentials
                    .fromStream(new ClassPathResource(firebaseConfigPath).getInputStream())
                    .createScoped(Arrays.asList("https://www.googleapis.com/auth/firebase.remoteconfig"));


            googleCredentials.refreshIfExpired();
        } catch (IOException e) {
            throw new FcmTokenException("Token 생성에 실패하였습니다.");
        }
        String token = googleCredentials.getAccessToken().getTokenValue();

        register(String.valueOf(userNo), token);
    }
```

<br>

제가 위에 `이 부분을 주목해주세요` 라는 주석문을 단 코드가 보이시나요? 현재 Firebase 토큰은 이전에 언급한 바와 같이 **파일시스템** 에서 
읽어오도록 설정되어 있었습니다. 운영체제에서는 파일시스템을 읽어올 때 커널시스템까지 다녀와야 하기에 **I/O** 가 발생한다고 하며 `getInputStream()` 은 1바이트를 로딩 할때마다
**I/O** 가 발생하기 때문에 주의하라는 내용이 많았습니다. 그 말은 즉슨, 현재 로그인 로직에서 가장 큰 병목현상을 일으키는 부분이라는 것을 의미했습니다.

<br>

### makeAccessToken을 클라이언트에서 처리하도록 하자

그리하여 제가 선택한 방식은 토큰 생성을 백엔드에서 하지말고 클라이언트쪽에서 생성 후, 이미 생성 된 토큰을 넘겨주는 방식을 택했습니다. 그렇기 때문에 
로그인 시 `json`으로 받는 객체에 새로히 `String token` 을 추가하였습니다. 그리고, 로그인 할 때 받아온 토큰을 그대로 Redis에 저장하도록 `register()` 메서드를 
사용했습니다.

```java
/**
     * 사용자 로그인 기능
     * Firebase Token 생성 후 로그인한 회원에게 보내야 할 알림여부를 응답을 보냄
     * @param memberLogin
     * @return {@literal List<AlertResponse>}
     */
    @PostMapping("/login")
    public List<AlertResponse> login(@RequestBody MemberLogin memberLogin) {
        String userNo = String.valueOf(memberLogin.getUserNo());
        String userId = memberLogin.getUserId();
        String password = memberLogin.getPassword();

        memberService.isUserIdExist(userId, password);
        loginService.setUserNo(memberLogin.getUserNo());

        //firebaseTokenManager.makeAccessToken(memberLogin.getUserNo()); 토큰생성 로직삭제
        firebaseTokenManager.register(userNo, memberLogin.getToken());  //토큰 저장 로직을 이쪽으로 옮기게 됨

        List<AlertResponse> loginResponses = alertService.eventStartNotice(memberLogin.getUserNo(), LocalDate.now());
        AlertResponse isUserNeedToChangePw = alertService.changePasswordNotice(memberLogin.getUserNo());
        loginResponses.add(isUserNeedToChangePw);
        
        return loginResponses;
    }

```

<br>

이렇게 코드를 변경하고 다시 프로젝트를 서버에 재배포 하였습니다. 성능테스트를 다시 수행했을 때 어떤 결과가 나오게 될까요?

<br>

### 리팩토링 후 성능테스트 결과

![image](https://user-images.githubusercontent.com/58355531/117577647-9fedcc00-b125-11eb-8b55-12aa6cf38a48.png)

사용자 수도 300명으로 늘렸을 때, 평균 **TPS 465** , **Errors 약 1000건** , **최대 TPS 978** 로 이전에 비해 월등하게 나아진 성능 결과를 보여주었습니다!!
파일시스템에서 읽어오던 방식이 이렇게나 성능에 막대한 영향을 미친다는 것을 알게 해주었습니다.

> 하지만 여전히 병목지점은 있었습니다. 2분 18초 쯤에 갑자기 TPS가 크게 떨어져 0을 가리키는 현상이 있었는데
> 이는 또다른 병목지점이 있다는 의미로 보였습니다.

<br>
<br>

## 문제점2: 비밀번호 변경알림, 이벤트 시작알림의 응답

```java
/**
     * 사용자 로그인 기능
     * Firebase Token 생성 후 로그인한 회원에게 보내야 할 알림여부를 응답을 보냄
     * @param memberLogin
     * @return {@literal List<AlertResponse>}
     */
    @PostMapping("/login")
    public List<AlertResponse> login(@RequestBody MemberLogin memberLogin) {
        String userNo = String.valueOf(memberLogin.getUserNo());
        String userId = memberLogin.getUserId();
        String password = memberLogin.getPassword();

        memberService.isUserIdExist(userId, password);
        loginService.setUserNo(memberLogin.getUserNo());

        firebaseTokenManager.register(userNo, memberLogin.getToken());
        
        //로그인 후 응답으로 보내주는 비밀번호 변경알림, 이벤트 시작알림
        List<AlertResponse> loginResponses = alertService.eventStartNotice(memberLogin.getUserNo(), LocalDate.now());
        AlertResponse isUserNeedToChangePw = alertService.changePasswordNotice(memberLogin.getUserNo());
        loginResponses.add(isUserNeedToChangePw);
        
        return loginResponses;
    }

```

<br>

`alertService` 는 비밀번호 변경알림, 이벤트 시작알림의 알림 여부를 보내야하는지 말아야하는지에 대한 true, false 값을 리턴해주도록 구성하였습니다.
그리하여 클라이언트에서 true면 알림을 보내주도록 하는 것이 기존의 설계였습니다.

하지만, 매번 응답을 꼬박꼬박 받게 된다면 그것은 동기방식에 속하게 됩니다. 동기방식은 항상 응답으로 받아야하기 때문에 비동기 방식보다 상당히 느릴 수밖에 없습니다. 

다행히도, 현재 Firebase 에서 알림 메세지를 전송하는 부분은 비동기 처리가 이미 완료되어있었습니다. 단지 편의를 위해 응답을 매번 받게 해두었는데 성능에 큰 영향을 받게 된다면 
다시 한번 고민해봐야 하는 부분이었습니다. 

<br>

### 알림여부를 보내주는 응답을 삭제하자

```java
/**
     * 사용자 로그인 기능
     * Firebase Token 생성 후 로그인한 회원에게 보내야 할 알림여부를 응답을 보냄
     * @param memberLogin
     * @return {@literal List<AlertResponse>}
     * @return {@literal CompletableFuture<List<AlertResponse>>}
     */
    @PostMapping("/login")
    public ResponseEntity<HttpStatus> login(@RequestBody MemberLogin memberLogin) {
        String userNo = String.valueOf(memberLogin.getUserNo());
        String userId = memberLogin.getUserId();
        String password = memberLogin.getPassword();
        
        memberService.isUserIdExist(userId, password);
        loginService.setUserNo(memberLogin.getUserNo());

        firebaseTokenManager.register(userNo, memberLogin.getToken());

        alertService.eventStartNotice(memberLogin.getUserNo(), LocalDate.now());
        alertService.changePasswordNotice(memberLogin.getUserNo());

        return RESPONSE_ENTITY_OK;
    }
```

<br>

이미 Firebase에 알림 메세지를 비동기로 전송해주는 `sendAsync()` 가 있기 때문에 알림을 비동기로 전송하고 있는 와중에 이중으로 클라이언트에 true, false 알림여부를 
보내지 않아도 된다고 판단하였습니다. 그리하여 과감히 이 부분을 삭제하고 로그인에 성공한다면 `200 Status code` 를 리턴하도록 변경하였습니다. 

<br>


### @Async의 활용

```java
@EnableAsync
@Configuration
public class AsyncConfig implements AsyncConfigurer {

    @Override
    public Executor getAsyncExecutor() {
        ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor();
        executor.setThreadNamePrefix("festa-async-");
        executor.setCorePoolSize(5);
        executor.setMaxPoolSize(10);
        executor.setQueueCapacity(100);
        executor.setRejectedExecutionHandler(new ThreadPoolExecutor.CallerRunsPolicy());
        executor.initialize();

        return executor;
    }
```

추가로, Service단에서 `eventStartNotice` , `changePasswordNotice` 메서드에 `@Async` 를 선언하여 비동기 처리 될 수 있도록
구성하였습니다. `@Async` 는 Thread Pool을 활용하기 때문에 따로 `AsyncConfig` configuration 파일을 생성하였습니다.

<br>
<br>

### 리팩토링 후 성능테스트 결과

![image](https://user-images.githubusercontent.com/58355531/117578218-4a66ee80-b128-11eb-81ad-b71252d3e13c.png)

<br>

아쉽게도 TPS의 차이는 크게 나타나지 않았습니다. 하지만 의미있는 점은, Errors 가 약 200건으로 전보다 5배 가량 크게 감소했으며 
기존에 최대 TPS 정점을 찍으면 성능이 갑자기 수직낙하하여 아예 0이 뜨는 현상은 없어지게 되었습니다. 낙하하는 비율 또한 많이 좁힐 수 있었습니다.

> 하지만 여전히 의문이 남는 것은, 2~3분 사이 최대 정점을 찍은 후 성능의 저하는 계속 발생하고 있다는 점이었습니다. 이제는 서버의 사양 아니면
> 로드밸런싱, 힙 상태 모니터링 등 다방면의 트레이드 오프의 고려가 필요해보였습니다. 추가 리팩토링을 진행한 과정은 다음 포스팅에 이어서 올려보도록 하겠습니다!

<br>
<br>
<br>

## Project Github URL 

[![오구리이미지](https://user-images.githubusercontent.com/58355531/99896015-085c0480-2cd0-11eb-998d-8b8faeb43e17.gif)](https://github.com/f-lab-edu/event-recommender-festa "이미지를 클릭하면 프로젝트 깃허브를 볼 수 있습니다 :)")

[FESTA 프로젝트 Github 보러가기 Click! (또는 위의 이미지 Click!)](https://github.com/f-lab-edu/event-recommender-festa)

<br>
<br>
<br>

## Referenced by

- nGrinder 공식 Github 주소: <https://github.com/naver/ngrinder>
  
<br>
<br>
<br>
<br>
<br>
<br>
