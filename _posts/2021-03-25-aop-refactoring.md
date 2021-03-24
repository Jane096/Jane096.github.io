---
title: "AOP를 활용한 중복코드 리팩토링"    
layout: single    
read_time: true    
comments: true   
categories: 
 - project  
toc: true    
toc_sticky: true    
toc_label: contents    
description: 로그인 확인 중복코드를 AOP로 리팩토링하여 중복코드를 해결한 포스팅
last_modified_at: 2021-01-10   
---

<br>
<br>

## Overview

사용자에게 [행사추천 서비스](https://github.com/f-lab-edu/event-recommender-festa)를 진행하면서 우리 프로젝트의 대부분의 기능은 **로그인 여부 확인** 이 되어야만 이용할 수 있는 
서비스가 대부분이었습니다. 

<br>

그로 인해, 회원의 정보수정/로그아웃, 이벤트 등록/수정/삭제 등 서비스의 주요 메서드에서 아래의 코드가 중복으로 보여지게 되었습니다. 

<br>

```java

boolean isLoginUser = loginService.isLoginUser();

  if(!isLoginUser) {
     throw new HttpStatusCodeException(HttpStatus.UNAUTHORIZED, "user is not authorized") {};
  }
  
```

<br>

동일한 코드가 여러 메서드에서 사용되면 하나하나 일일히 찾아가며 디버깅을 해야하기 때문에 **유지보수성** 이 매우 떨어질 수 있습니다.
**또한**, 회원정보 조회를 하는 메서드의 경우, 회원을 조회하는 핵심로직만 있어야 하는데 로그인 체크 여부까지 
확인해야 하는 짐을 지고 있었습니다. 

<br>

현재 로그인 여부 확인의 코드를 간단하게 작성했지만 코드가 복잡해질 가능성도 배재할 수 없고, 그렇게 된다면 코드를 파악하는 것 조차
어려울 수 있습니다. 

그리하여, 로그인 확인여부 로직을 따로 분리하여 한 곳에서만 관리를 하도록 리팩토링을 진행하게 되었습니다.

<br>
<br>
<br>

## 리팩토링 진행과정

처음에는 기존 객체지향의 관점대로 중복되는 부분을 클래스로 분리하여 DI로 주입하는 방식을 고려해보았습니다. 
그러나 로그인 확인 여부 코드가 해당 클래스 내 각각의 메서드마다 또다시 전반적으로 쓰여지고, 그렇다고 로그인 확인 여부를 지울 수도 없었습니다.결국, 지금과 같은 케이스에서 기존 클래스 분리 방법은 OOP의 모듈화와 핵심 로직의 가독성, 확장성을 방해할 수 있었습니다.   

이러한 경우를 위해서 스프링에서는 **AOP** 라고 하는 기능을 제공합니다. 
**AOP** 는 어플리케이션 전반적으로 퍼져있는 같은 용도의 중복코드를 한 곳에 모아 핵심로직으로부터 분리하고 메서드 별로 세심한 조정이나
메서드 전후로 자유롭게 설정이 가능합니다. 더 나아가 주소, 파라미터, 커스텀 어노테이션 등의 다양한 방식으로 
대상을 지정할 수 있도록 기능을 제공합니다.

이렇게 **AOP** 의 커스텀 어노테이션의 기능을 활용하여 코드를 핵심로직으로부터 분리하여 메서드가 실행 되기 전 수행하도록
로직을 변경하였습니다. 

<br>
<br>

### 의존성 추가 

```xml
<!--AOP 관련 의존성-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-aop</artifactId>
        </dependency>
```

<br>

**스프링 부트** 를 사용할 경우 위의 의존성만 추가해주면 됩니다!

<br>
<br>
<br>

### @EnableAspectJAutoProxy 어노테이션 추가 

```java

@SpringBootApplication
@EnableAspectJAutoProxy
public class FestaApplication {

    public static void main(String[] args) {
        SpringApplication.run(FestaApplication.class, args);
    }
}
```

<br>

프로그램이 실행될 때 `main()` 이 AOP가 적용된 클래스를 읽어올 수 있도록 `@EnableAspectJAutoProxy` 어노테이션을 추가해줍니다. 

<br>
<br>
<br>

### @CheckLoginStatus 인터페이스 생성(커스텀 어노테이션)

```java

@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface CheckLoginStatus {

    UserLevel auth() default UserLevel.USER;
}

```

<br>

`Runtime` 시에 위빙이 되며 메서드 별로 설정이 되도록 어노테이션을 추가하였습니다. 

우리 프로젝트에는 **일반사용자** 와 **주최자** 의 권한이 다르게 설정되어있어야 하기 때문에 
`UserLevel` 의 권한을 체크하는 부분을 추가로 작성하였습니다. 

<br>
<br>
<br>

### CheckLoginStatusAop 클래스 생성

실제로 `@CheckLoginStatus` 어노테이션이 동작할 `@Aspect` 클래스를 작성해줍니다. 그리고 `@Component` 를 추가하여
반드시 빈으로 등록해주어야 합니다. 

<br>

```java

@Aspect
@Component
@RequiredArgsConstructor
@Log4j2
public class CheckLoginStatusAop {

    private final LoginService loginService;
    private final MemberService memberService;

    /**
     * 권한에 따른 분기처리를 위한 메서드
     * No Param
     * No return
    */
    @Before(value = "@annotation(CheckLoginStatus) && @annotation(checkLoginStatus)")
    public void checkStatus(CheckLoginStatus checkLoginStatus) {
        UserLevel auth = checkLoginStatus.auth();

        switch(auth) {
            case USER:
                allUserLoginStatus();
                break;

            case HOST:
                hostLoginStatus();
                break;

            default:
                break;
        }
    }

    /**
     * 모든 사용자 로그인 여부 확인
     * No param
     * No return
     * @throws HttpStatusCodeException
    */
    public void allUserLoginStatus() {
        boolean isLoginUser = loginService.isLoginUser();

        if(!isLoginUser) {
            throw new HttpStatusCodeException(HttpStatus.UNAUTHORIZED, "user is not authorized") {};
        }
    }

    /**
     * 주최자 권한 사용자의 로그인 여부 확인
     * No param
     * @return {@literal ResponseEntity<HttpStatus>}
     * @throws HttpStatusCodeException
    */
    public ResponseEntity<HttpStatus> hostLoginStatus() {
        allUserLoginStatus();

        long userNo = loginService.getUserNo();
        MemberDTO memberInfo = memberService.getUser(userNo);

        log.debug(userNo + ": Started to check Host-user authentication");

        if(memberInfo.getUserLevel() != UserLevel.HOST) {
            throw new HttpStatusCodeException(HttpStatus.UNAUTHORIZED, userNo + " is not a Host") {};
        }

        return RESPONSE_ENTITY_OK;
    }
}

```

<br>

`@CheckLoginStatus` 가 달려있는 메서드의 경우 실행된다면 `@Before` 포인트컷으로 인해 메서드 시작 전, 로그인 여부를 체크하게 됩니다. 만약 어노테이션의 속성이 `auth = UserLevel.USER` 일 경우 권한에 따른 분기처리 부분에서 `USER` 로 확인이 되어 `allUserLoginStatus()` 가 실행되고
로그인이 되지 않은 상태라면 `~~ is not Authorized` 의 메세지를 출력하여 **401 status code** 응답을 날려줍니다. 

주최자의 경우 현재 요청이 들어온 사람의 `UserLevel` 을 확인하여 `HOST` 로 동일하지 않다면 똑같이 **401 status code** 를 보내주고
`HOST` 의 권한을 가진 사용자라면 **200 status code** 를 보내줍니다. 

만약 **주최자인지 확인** 해야 하는 경우 일반사용자 권한 체크보다 긴 코드가 모든 메서드에 추가가 되니 로그인 확인 여부에 오류가 생긴다면 상당히 곤란하게 될 수 있습니다. 또한 보는 사람 입장에서도 코드가 지저분하게 보여 핵심로직이 어디에 있는지 찾는데 시간을 낭비할 수도 있습니다.

<br>
<br>

## Before Refactoring 

회원정보 조회 기능을 예시로 AOP 적용 전과 후를 비교해보았습니다. 

<br>

```java
    
   /**
     * 사용자 회원정보 조회 기능
     * @param userNo
     * @return {@literal ResponseEntity<MemberDTO>}
     */
    @GetMapping("/{userNo}")
    public ResponseEntity<HttpStatus> getUser(@RequestParam long userNo) {
        boolean isLoginUser = loginService.isLoginUser();

        if(!isLoginUser) {
            throw new HttpStatusCodeException(HttpStatus.UNAUTHORIZED, "user is not authorized") {};
        }
        
        MemberDTO memberInfo = memberService.getUser(userNo);

        if(memberInfo == null) {
            return RESPONSE_ENTITY_MEMBER_NULL;
        }
        return RESPONSE_ENTITY_OK;
    }
```

<br>

회원의 정보를 조회를 하는 메서드인데 사용자의 로그인 여부를 먼저 체크합니다. 만약 주최자를 체크해야하는 경우 핵심로직은 단 3줄인데
주최자 권한까지 확인하며 핵심로직보다 로그인 확인여부의 코드가 더 많은 양을 차지하게 됩니다.

<br>
<br>

## After Refactoring

```java
    
   /**
     * 사용자 회원정보 조회 기능
     * @param userNo
     * @return {@literal ResponseEntity<MemberDTO>}
     */
    @CheckLoginStatus(auth = UserLevel.USER)
    @GetMapping("/{userNo}")
    public ResponseEntity<HttpStatus> getUser(@RequestParam long userNo) {
        MemberDTO memberInfo = memberService.getUser(userNo);

        if(memberInfo == null) {
            return RESPONSE_ENTITY_MEMBER_NULL;
        }
        return RESPONSE_ENTITY_OK;
    }
```

<br>

회원정보 조회 메서드는 이제 로그인 체크 여부 임무에서 벗어나 오로지 자기 자신의 역할에 충실할 수 있게 되었습니다.
또한 코드의 양도 확연히 줄어들어 다른 사람이 봐도 파악하기 더욱 쉬운 코드가 되었습니다.

<br>
<br>
<br>

## Project Github URL

<br>

![오구리이미지](https://user-images.githubusercontent.com/58355531/99896015-085c0480-2cd0-11eb-998d-8b8faeb43e17.gif)

[FESTA 프로젝트 Github 보러가기 Click!](https://github.com/f-lab-edu/event-recommender-festa)

<br>
<br>

## Referenced by

- 토비의 스프링 3.1 vol.1 : 이일민 지음      
  <https://book.naver.com/bookdb/book_detail.nhn?bid=7006514>      
  
- Spring Framework Official Document: Spring AOP APIs     
  <https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#aop-api>
  
<br>
<br>
<br>
<br>
<br>
<br>


