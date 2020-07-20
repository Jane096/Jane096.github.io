---
title: "static 메서드와 일반 메서드의 차이"
layout: single    
read_time: true    
comments: true   
categories: 
 -  java
toc: true    
toc_sticky: true    
toc_label: contents    
description: static 메서드와 일반메서드의 동작방식 정리
last_modified_at: 2020-07-20       
---


**static method** 란 객체를 생성하지 않아도 메서드를 호출할 수 있는 예약어 이다. 
대표적인 예로 System.out.println()이 객체를 생성하지 않고도 쓸 수 있는 이유가 바로 static 키워드 때문이다.
<br>
<br>

```java
public class ReferenceStatic {
  public static void main(String[] args) {
    ReferenceStatic.staticMethod();
  }
  public static void staticMethod() {
    System.out.println("This is a staticMethod");
  }
}
```
<br>

현재 method앞에 static 키워드가 붙어 해당 메서드는 static 메서드가 되었다. 당연히 객체를 생성하지 않고도 
바로 불러서 사용이 가능하다.
<br>

```bash
컴파일 결과

This is a staticMethod
```
<br>
<br>


static 메서드의 가장 큰 특징은 **클래스 변수** 만 사용할 수 있다는 점이다. 
<br>

```java
public class ReferenceStatic {
  public static void main(String[] args) {
    String name = "Min";
    
    ReferenceStatic.staticMethod();
  }
  public static void staticMethod() {
    System.out.println("This is a staticMethod");
  }
  
  public static void staticMethodCallVariable() {
    System.out.println(name);
  }
}
```
<br>

위의 staticMethodCallVariable 메서드는 컴파일 에러를 발생시킨다. 
static의 큰 특징이자 단점은 클래스 변수만을 사용할 수 있다는 점이다. 즉, name변수는 static이 붙지않은
인스턴스 변수 이기 때문에 "static이 아닌 변수 이름은 static context에서 참조할 수 없다" 는 컴파일 에러를 발생시킨다. 
<br>
<br>

static을 붙여 클래스 변수로 만들게 되면 모든 객체에서 하나의 값만을 바라보게 된다. 즉, 인스턴스 변수에 
아무렇게나 static을 붙이게 되면 생각했던 결과가 나오지 않을 수도 있다. 
<br>
<br>

```java
public class ReferenceStaticVariable {
  static String name; //클래스변수
  
  public ReferenceStaticVariable() {}
  public ReferenceStaticVariable(String name) {
    this.name = name;
  }
  
  public static void main(String[] args) {
    ReferenceStaticVariable reference = new ReferenceStaticVariable();
    reference.checkName();
    
  }
  public void checkName() {
    ReferenceStaticVariable reference1 = new ReferenceStaticVariable("Sangmin");
    System.out.println(reference1.name);
    ReferenceStaticVariable reference2 = new ReferenceStaticVariable("Sungchoon");
    System.out.println(reference1.name);
  }
}
```
<br>

```bash
Sangmin
Sunchoon
```
<br>

그냥 생각했을 때는 당연이 reference1만을 출력해주고 있으니 sangmin이라는 같은 값이 나올 것이라 생각하지만 
실제로 출력해보면 다른 값이 나온다. 왜냐하면 name이 클래스 변수이기 때문이다. 만약 원래 생각했던 대로 출력을 
하고 싶다면 static키워드를 제거하면 된다!
<br>
<br>
<br>
<br>
<br>
<br>

























