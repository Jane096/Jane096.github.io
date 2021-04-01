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
last_modified_at: 2021-03-25   
---

<br>
<br>

## Overview

사용자에게 [이벤트나 행사를 추천해주는 서비스](https://github.com/f-lab-edu/event-recommender-festa)를 진행하면서, 저희 서비스는 **로그인이 완료 되어야만** 이용할 수 있는 기능이 대부분을 차지하고 있습니다.


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

동일한 코드가 여러 메서드에서 사용되면 이 후 기능의 추가나 변경이 있을 시, 하나하나 일일히 찾아가며 디버깅을 해야하기 때문에 **유지보수성** 이 매우 떨어질 수 있습니다.
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
그러나 로그인 확인 여부 코드가 해당 클래스 내 각각의 메서드마다 또다시 전반적으로 쓰여지고, 그렇다고 로그인 확인 여부를 지울 수도 없었습니다. 결국, 지금과 같은 케이스에서 기존 클래스 분리 방법은 OOP의 모듈화와 핵심 로직의 가독성, 확장성을 방해할 수 있었습니다.   

<br>

### AOP

이러한 경우를 위해서 스프링에서는 **AOP** 라고 하는 기능을 제공합니다. **AOP** 는 로깅, 트랜잭션, 권한체크와 같이 어떠한 클래스나 메서드가 관심가지지 않아도 될 사항이지만 없으면 안되는 로직이면서 어플리케이션 전반적으로 동일하게 사용될 수 있는 코드를 **횡단 관심** 으로 보아 흩어져 있는 같은 관심사를 하나로 모듈화 하여 이를 여러 곳에서 재사용이 가능하게 합니다. 

> 조금 더 덧붙여서 **AOP** 는 어드바이스에 상관없이 포인트컷의 재사용을 가능하게 한다는 개념을 가지고 있습니다. 쉽게 설명하자면 `어드바이스 == 로그인 확인 여부 코드` 가 되고 `포인트컷 == 로그인 여부 코드가 언제 적용될지` 를 의미합니다. 즉, 적용시점이 같아도 여러 개의 모듈화 된 코드를 적용할 수 있습니다. 

<br>

어플리케이션 전반적으로 퍼져있는 같은 용도의 중복코드를 한 곳에 모아 핵심로직으로부터 분리하고 메서드 별로 세심한 조정이나 메서드 전후로 자유롭게 설정이 가능합니다. 더 나아가 주소, 파라미터, 커스텀 어노테이션 등의 다양한 방식으로 대상을 지정할 수 있도록 기능을 제공합니다.

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

프로그램이 실행될 때 `@Aspect` 가 명시된 클래스를 읽어 Bean으로 등록될 수 있도록 `@EnableAspectJAutoProxy` 어노테이션을 추가해줍니다. 

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

여기서 `@Target` 이란 핵심로직을 의미합니다. 즉, **어드바이스(로그인 확인여부 코드)를 받을 대상** 이라고 보면 됩니다. 저희는 모듈화 된 
로그인 확인여부 코드를 메서드 별로 지정할 예정이기에 `METHOD` 라고 지정해주었습니다.

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

`@Aspect`은 이 클래스가 **흩어진 관심사를 모듈화 했다** 라는 것을 명시하는 어노테이션 입니다. 이런 Aspect 클래스가 스프링에서 관리될 수 있도록 `@Component` 어노테이션을 이용해 Bean으로 등록합니다. 

`@Before` Pointcut으로 인해 `@CheckLoginStatus` 가 달려있는 메서드가 실행된다면 메서드 시작 전, 로그인 여부를 체크하게 됩니다. 만약 어노테이션의 속성이 `auth = UserLevel.USER` 일 경우 권한에 따른 분기처리 부분에서 `USER` 로 확인이 되어 `allUserLoginStatus()` 가 실행되고 로그인이 되지 않은 상태라면 `~~ is not Authorized` 의 메세지를 출력하여 **401 status code** 응답을 날려줍니다. 

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

## HandlerMethodArgumentResolver 를 이용한 AOP 적용하기

지금까지는 메서드 별로 적용가능한 AOP에 대해 정리해보았습니다. 제 프로젝트 전체적으로 추가로 적용할 곳이 없나 찾아보던 중 
또 다른 중복코드가 보여지게 되었는데 바로 **현재 로그인 중인 사용자의 회원코드인 userNo** 를 세션에서 가져오는 부분이었습니다. 

<br>

#### example

```java
/**
     * 회원 탈퇴 기능
     * @Param userNo
     * @return {@literal ResponseEntity<HttpStatus>}
     */
    @CheckLoginStatus(auth = UserLevel.USER)
    @DeleteMapping("/")
    public void memberWithdraw(String userId) {
        long userNo = loginService.getUserNo(userId) //사용자의 아이디명을 통해서 세션에 저장된 userNo를 가져오는 부분
        memberService.memberWithdraw(userNo);
    }
```

<br>

이 부분에 대해서도 여러 메서드 마다 중복으로 나타나게 되었고 **userNo를 가져오는 로직** 또한 9의 관심대상이 아니었습니다.
이러한 이유로 이전처럼 어노테이션을 이용한 AOP를 파라미터별로 적용할 수 있도록 설계를 변경해보기로 합니다. 

<br>

### HandlerMethodArgumentResolver 란?

Method 별로 특정조건에 맞는 파라미터가 있을 때 원하는 값을 알아서 바인딩 시켜주는 인터페이스를 스프링에서 제공을 해주는데
그 인터페이스는 바로 [HandlerMethodArgumentResolver](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/web/method/support/HandlerMethodArgumentResolver.html) 입니다. 

우리는 `Controller` 에 값을 받아오는 부분을 지정할 때 `@RequestBody` 나 `@PathVariable` 을 사용하신 경험이 있을겁니다. 이 때 이 어노테이션 내부에는
`HandlerMethodArgumentResolver` 인터페이스가 있으며 이를 통해 값을 받아옵니다. 

저는 이 인터페이스를 커스텀하게 지정하여 세션에 저장 된 `userNo` 값을 받아오는 코드를 모듈화 할 예정입니다.

<br>
<br>

### 어노테이션 인터페이스 작성

어노테이션에 적용하기 위해서는 당연히 어노테이션 인터페이스가 작성이 먼저 되어있어야 합니다. 저는 **현재 로그인 된 userNo** 라는 의미로 
`@CurrentLoginUserNo` 라고 명칭을 지어주었습니다.

```java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.PARAMETER)
public @interface CurrentLoginUserNo {
}
```

<br>

이전과 다른점은 메서드별이 아닌 파라미터별로 지정할 것이기 때문에 `@Target` 은 **PARAMETER** 로 선언해주도록 합니다.

<br>

### Resolver 작성하기

실제로 `@CurrentLoginUserNo` 가 수행해야할 부분을 작성해주도록 합니다. 클래스를 하나 새로 생성하여 그 클래스는 **HandlerMethodArgumentResolver**
인터페이스를 implements 받도록 합니다. 그렇다면 자동으로 오버라이드 해야할 두개의 메서드가 존재합니다.

```java
@Component
@RequiredArgsConstructor
public class LoginUserNoResolver implements HandlerMethodArgumentResolver {

    private final LoginService loginService;

    @Override
    public boolean supportsParameter(MethodParameter methodParameter) {
        return methodParameter.hasParameterAnnotation(CurrentLoginUserNo.class);
    }

    @Override
    public Object resolveArgument(@Nullable MethodParameter methodParameter, ModelAndViewContainer modelAndViewContainer, @Nullable NativeWebRequest nativeWebRequest, WebDataBinderFactory webDataBinderFactory) {
        return loginService.getUserNo();
    }
}
```

<br>

첫번째 `supportsParameter` 에는 특정 어노테이션이 선언 된 부분에만 동작을 할 수 있도록 설정할 수 있습니다. 동작이 가능하다면 **True** 를 반환하게 됩니다.    

두번째 `resolveArgument` 는 실제 어노테이션이 선언 된 부분이 수행해야할 기능을 설정할 수 있습니다. 저희는 세션의 저장된 사용자 회원번호를 가져와야 하기 때문에
`loginService.getUserNo()` 동작을 이 곳 하나에만 지정하도록 합니다. 

<br>
<br>

### WebConfig로 스프링에 등록하기

이제 커스텀하게 만든 `LoginUserNoResolver` 를 스프링에 등록하여 스프링에서 관리될 수 있도록 합니다. 이 때 `WebConfig` 는 `WebMvcConfigurer` 를 implements 받도록 합니다.
그래야 개발자가 새로 커스텀하게 만들어 등록한 `loginUserNoResolver` 를 이용할 수 있게 됩니다. 

```java
@Configuration
@RequiredArgsConstructor
public class WebConfig implements WebMvcConfigurer {

    private final LoginUserNoResolver loginUserNoResolver;

    @Override
    public void addArgumentResolvers(List<HandlerMethodArgumentResolver> resolvers) {
        resolvers.add(loginUserNoResolver);
    }
}
```

<br>
<br>

## After refactoring

```java
/**
     * 회원 탈퇴 기능
     * @Param userNo
     * @return {@literal ResponseEntity<HttpStatus>}
     */
    @CheckLoginStatus(auth = UserLevel.USER)
    @DeleteMapping("/")
    public void memberWithdraw(@CurrentLoginUserNo long userNo) {
        memberService.memberWithdraw(userNo);
    }
    
```

<br>

이로써 커스텀하게 설정한 `@CurrentLoginUserNo` 를 파라미터별로 설정하여 핵심로직의 관심대상을 분리하면서 어플리케이션 전반적으로 중복되어 나타나는 코드를 
모듈화 할 수 있었습니다! 

현재 올려진 예시 이외의 소스코드는 아래 첨부된 깃허브 레파지토리를 통해 참고 부탁드립니다 :)

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


