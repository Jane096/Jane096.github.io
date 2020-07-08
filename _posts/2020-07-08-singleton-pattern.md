---
title: "[디자인패턴] 싱글톤 패턴은 어떻게 사용할까? "
layout: single    
read_time: true    
comments: true   
categories: 
 - Design pattern  
toc: true    
toc_sticky: true    
toc_label: contents    
description: 오직 하나의 인스턴스만을 생성하는 싱글톤 패턴의 특징
last_modified_at: 2020-07-08       
---

**싱글톤 패턴(Singleton Pattern)** 이란 클래스가 오직 하나의 인스턴스만 생성한다는 것을 보장하는 
패턴이다. 주로 데이터베이스, 웹 서비스 같은 여러 서드파티에 해당 인스턴스에 접근할 수 있는 
유일한 지점을 만드는 데 사용한다. 
<br>
<br>

**장점? ** 다수 서비스에서 보내는 연결 요청을 한 곳에서 쉽게 관리하고 설정이 가능하다
{: .notice--info}

<br>
<br>
<br>

## Singleton 패턴의 잘못된 예시

```java
public class Singleton {
  private static Singleton instance;
  
  public static Singleton getInstance() {
    if(instance == null) {
      instance = new Singleton();
    }
    return instance;
  }
  
  public void singletonMethod() {
    //여기서 연산 시작
  }
}
```
<br>

위와 같은 접근 방법을 **지연 초기화(lazy Initialization)** 라고 한다. 
Singleton 인스턴스는 제일 처음 필요로 하는 경우에 생성이 된다. 위의 코드를 보면 
누가 getInstance() 호출하든 같은 인스턴스를 반환할 것처럼 보일 수 있다. 
<br>
<br>

그러나 인스턴스의 값이 null 이거나 if문 때문에 인스턴스가 초기화되기 전에 스레드가 변경되면 
두 번째(혹은 더 많은) 스레드가 getInstance() 메서드를 호출하게 되고 if문은 true와 새로운 객체를 반환한다. 
이럴 경우, 결과가 이상하거나 오작동이 발생하고 최악의 경우 메모리 누수발생으로 JVM이 동작을 멈출 수도 있다. 
<br>
<br>
<br>

## 보완책

이럴 때 **Enum** 타입을 사용하는 것이 좋다. 
Singleton 인스턴스를 하나의 원소를 가진 Enum 타입으로 생성하면 JVM은 밑의 코드 처럼 하나의 
인스턴스만 생성하는 것을 보장해준다.
<br>

```java
public enum SingletonEnum {
  instance;
  
  public void singletonMethod() {
    //연산수행 시작
  }
}
```
<br>
<br>
<br>

### 기억해야할 점

Singleton 인스턴스로 데이터베이스 저장 같이 무거운 연산을 실행할 경우, 
작은 코드 부분 단위로 분리해 테스트하기가 매우 어렵게 된다고 한다. 
클래스의 의존성을 유지하려면 **'싱글톤 같은'** 객체를 이용하는 것이 관리를 단순화 시켜
더 유용하다. 
<br>

그리고 하나의 객체만 생성하려면 의존성 주입 프레임워크를 이용하도록 하자
<br>
<br>
<br>

### 싱글톤 패턴의 사용

보통 데스크탑, 모바일 기기의 GUI 어플리케이션, 동시사용자가 많지 않은 어플리케이션 구현에 유리하다. 
그러나 대규모 확장 가능한 서버 어플리케이션에는 병목현상의 원인이 되니 주의하도록 하자
<br>
<br>
<br>
<br>
<br>
<br>






















