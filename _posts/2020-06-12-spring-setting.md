---
title: "Spring 개발 환경에 대한 정리"
layout: single    
read_time: true    
comments: true   
categories: 
 - spring  
toc: true    
toc_sticky: true    
toc_label: contents    
description: spring 개발환경(part1) 정리('코드로 배우는 스프링 웹 프로젝트' 참고)  
last_modified_at: 2020-06-12  
---
<br>
<br>
STS(이클립스 플러그인)와 Tomcat, Oracle DB 설치 방법은 생략하고 진행함.
<br>

## chapter 1. 스프링 프로젝트 생성하기 
File -> New -> Spring Legacy Project 에서 Spring MVC Project 이용해 생성함. 
<br>
<br>

![m2 이미지1]({{ site.url }}{{ site.baseurl }}/assets/ready-spring/m2file.PNG){: .align-center}  <br>
Maven 등에서 스프링 관련 파일을 다운로드 받는데 이러한 라이브러리를 보관하고 있는 폴더라 .m2 폴더
<br>

![m2이미지2]({{ site.url }}{{ site.baseurl }}/assets/ready-spring/m2filedetail.PNG){: .align-center}  <br>
이렇게 구성되어있고 보면 pom.xml의 내용과 상당수 일치함. pom.xml파일을 고치고 Alt+f5를 누르면 build maven project를 실시하는데 그 관련 파일들이 해당 폴더에 다 보이게 됨.
<br>

![스프링 생성에러 이미지]({{ site.url }}{{ site.baseurl }}/assets/ready-spring/springdefault.PNG){: .align-center}   <br>
이클립스에서 새로운 패키지를 생성하다 보면 이렇게 에러가 종종 뜨는데(정상일때도?) 환경설정 하다 안되면 maven jar파일을 제대로 다운을 못한 뜻이다.  
그럴 경우, .m2/repository 하위의 모든 폴더를 삭제 후, Alt+F5를 눌러 다시 maven project를 build 해주면 된다.   
<br>

### 스프링 프로젝트 구조
![스프링 생성에러 이미지]({{ site.url }}{{ site.baseurl }}/assets/ready-spring/springdefault.PNG){: .align-center} 
<br>
- src/main/java : 작성되는 java코드 경로 (controller, config, service, mapper java파일 관리함)
- src/main/resources : 실행할 때 참고하는 기본 경로 (Mapper의 xml파일, properties파일 등이 여기서 관리됨)
- src/test/java : junit의 테스트 코드들의 경로 (server로 돌리기 전에 제대로 비즈니스를 처리하는지 테스트를 함)
- src/test/resources : 테스트 관련 설정 파일 보관 경로 (한번도 파일을 생성한 적이 없다..)
<br>
- servlet-context.xml(servlet config) : 웹과 관련된 스프링 설정 파일, java 설정으로 할 경우 src/main/java에서 servletConfig.java가 대신하게 된다.
- root-context.xml(RootConfig) : 스프링 설정 파일, java 설정은 RootConfig가 대신함
- web.xml(WebConfig) : Tomcat의 web.xml파일 
- pom.xml : maven 관련 설정 파일 목록

### Java version으로 변경할 경우
![스프링 생성에러 이미지]({{ site.url }}{{ site.baseurl }}/assets/ready-spring/javawarplugin.PNG){: .align-center} 
<br>
pom.xml하단에 해당 코드를 입력하고 위의 maven-compiler-plugin과 java version은 모두 1.8로 맞춰주기(스프링 5.x 버전 에서는 jdk 1.8이 가장 안정적이라고 함)
그리고 WEB-INF 하위 spring 파일을 삭제하고 web.xml도 삭제하기.
**pom.xml을 변경하면 꼭 Alt+F5 해서 maven project 업데이트 하도록!**

### @Configuration Annotation (RootConfig)
![어노테이션 이미지]({{ site.url }}{{ site.baseurl }}/assets/ready-spring/configuration.PNG){: .align-center} 
<br>
src/main/java 하위에 xxx.xxx.config 패키지 생성 후, RootConfig/WebConfig/ServletConfig라는 이름의 파일 생성하기.  
@Configuration은 xml설정 파일을 이 어노테이션이 대신한다는 의미(우리는 설정파일을 작성할 필요가 없다!)  
다른 어노테이션 정리는 이후에..  
<br>
<br>

### WebConfig와 AbstractAnnotationConfigDispatcherServletInitializer
![인터페이스 오버라이딩]({{ site.url }}{{ site.baseurl }}/assets/ready-spring/webconfiginterface.PNG){: .align-center}
<br>
이름이 엄청 긴 이 인터페이스는 spring 3.x이상에서 작동하며 web.xml을 대체할 수 있도록 해주는 기능이다. 
<br>
<br>
![오버라이딩 기능]({{ site.url }}{{ site.baseurl }}/assets/ready-spring/webconfigmvc.PNG){: .align-center}
<br>
오버라이딩을 하면 자동으로 생기는 메서드들인데, 해당 메서드로 xml파일들을 config파일로 최종 대체하며 spring mvc를 구축하게 된다.  
파일을 저장하면 아마 console창에 mvc를 구축하는 로그가 뜨게 될 것이다.  
<br>
<br>

## chapter 2. 스프링의 특징과 의존성 주입(spring DI)
### 스프링의 주요 특징
- POJO 기반의 구성
- DI를 통한 객체 간의 구성
- AOP를 통한 중복 코드 최소화<br>
크게는 이렇게 3가지가 있다.  
<br>

### POJO???
Plain Old Java Object의 약자로 특정 라이브러리나 컨테이너에 기술이 종속적이지 않다는 것(즉, 재사용성이 높음!)
무슨말이냐면
<br>
```java
@Component
public class Example {
  @AListener
  public void process(String msg) {
    System.out.println(msg);
  }
}
```
<br>
이런 식으로 AListener라는 환경에 종속되지 않고 필요에 의해 BListener로 바꿔주기만 하면 어디든 쓸 수 있도록 
코드와 라이브러리의 결합성을 줄이는 것을 의미한다.  
<br>
<br>

### 의존성 주입(Dependency Injection = DI) 
DI란 하나의 객체가 다른 객체 없이 제대로 된 역할을 할 수 없다는 것을 의미한다.  
어떠한 객체를 외부에서 주입한다는 것인데, 주입받는 입장에서는 어떤 객체인지 신경 쓸 필요가 없고, 자신의 역할만을
충실하게 수행을 하게 된다.  
<br>

```java
@Component
@Data //setter, getter method 
@AllArgsConstructor //Spring 4.3 이상에서만 가능
public class Restaurant {  
   private Chef chef;
}
```
<br>
이런식으로 Restaurant이라는 객체는 Chef가 필요하다고 주입을 해주는 것이다.  
Restaurant객체는 Chef말고도 여러가지를 주입받을 수 있으며 어떠한 것을 주입받아도 Restaurant의 역할은
변하지 않는다.  
<br>
<br>

### 의존성 주입 테스트 해보기 
src/test/java 에 ~~test.java 라는 파일을 하나 생성하고 Rootconfig파일에 @ComponentScan을 추가해
간단하게 의존성 주입을 테스트 해보면
<br>
```java
//RootConfig.java파일
@Configuration
@ComponentScan(basePackages = {xxx.xxx.sample}) //@Component어노테이션을 찾으면 해당 클래스 인스턴스 생성
public class RootConfig {

}
```
```java
//src/test/java 의 테스트파일

@RunWith(SpringJUnit4ClassRunner.class) //현재 코드가 스프링을 실행하는 역할을 한다는 뜻
@ContextConfiguration(classes = {RootConfig.class}) //지정된 클래스나 문자열을 이용해 필요한 객체를 스프링 내에 객체로 등록함
@AllArgsConstructor
@Log4j //lombok을 이용해 로그를 기록하는 Logger변수 생성
public class testDI {
  private Restaurant restaurant;
  
  @Test //JUnit의 단위테스트 대상임을 알려줌
  public void test() {
    assertNull(restaurant);
    log.info(restaurant);
    log.info(restaurant.getChef());
  }
}
```
<br>
Restaurant클래스에 @Component가 있으므로 해당 인스턴스를 생성하고 Chef라는 객체가 필요하다는 설정이 있으므로
최종 주입을 한다. 
<br>
<br>
<br>
스프링이 동작하는 과정을 정리해보자면, 
- 스프링이 시작되면 스프링이 사용하는 메모리 영역인 Context를 만들게 되고 해당 메모리 영역을 ApplicationContext라는 이름으로 생성한다.
- RootConfig를 통해 스프링이 객체를 생성하고 관리해야 하는 객체들의 설정 정보를 읽는다.
- @ComponentScan에 지정된 패키지를 읽고 해당 패키지 내에 @Component 어노테이션이 달린 클래스를 찾는다.
- 해당 클래스의 인스턴스를 생성하고 필요한 객체(@AllArgsConstructor 또는 @Autowired)를 주입해준다.
<br>
<br>
<br>
여기서 주목할 점은  
- Restaurant이 new 연산자 없이 객체가 생성되었다는 것
- @Data 어노테이션으로 getter/setter method를 자동 생성해준 것
- @AllArgsConstructor(또는 @Autowired)로 chef의 인스턴스 변수가 Restaurant에 주입되어 자동으로 객체가 관리되는 것<br>
<br>
<br>
요 3가지로 정리할 수 있을 듯 하다 :) 
<br>
<br>
<br>
계속 쓰다 보면 정리하는 스킬도 늘겠지?? ㅎㅎㅎㅎ (처음이라 뭔가 부족해보임..ㅠㅠ)
<br>
<br>
<br>
<br>







