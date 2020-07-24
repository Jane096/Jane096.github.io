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


우리가 알고있는 일반 메서드는 자신의 클래스나 다른 클래스에 있는 경우, 항상 객체를 생성 후 불러올 수 있었다. 
그러나 **static method**  객체를 생성하지 않아도 메서드를 호출할 수 있는 예약어 이다. 
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


static 메서드의 가장 큰 특징은 **클래스 변수** 만 사용할 수 있다는 점이다. (static이 붙은 변수!)
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

위의 staticMethodCallVariable 메서드는 name을 출력하는 부분에서 컴파일 에러를 발생시킨다. 
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

### static을 특이하게 사용해보기

아래 처럼 어떤 클래스의 객체가 생성되면서 딱 한번만 불러야 하는 코드가 있다.
<br>

```java
public class StaticBlock {
  static int data = 1;
  
  public StaticBlock() {
    System.out.println("StaticBlock Constructor");
  }
  
  static {
    System.out.println("First static Constructor");
  }
  
  static {
    System.out.println("Second static Constructor");
  }
  
  public static int getData() {
    return data;
  }
}
```
<br>

**static{}** 형태의 static 블록은 객체 생성 전 딱 한번만 호출이 된다. 
클래스 내에 선언되어 있어야 하며, 메서드 내에서는 선언이 불가능 하다는 특징이 있다. 
<br>

위의 코드를 봤을 때 static블록은 2개가 선언이 되어 있다. static 블록은 꼭 하나가 아니라 여러개 선언이 가능하다. 
선언할 때 순서대로 호출이 되기 때문에 선언되어있는 순서가 매우 중요하다. 
<br>
<br>

```java
public class StaticBlockCheck {
  public static void main(String[] args) {
    StaticBlockCheck check = new StaticBlockCheck();
    check.makeStaticBlockObject();
  }
  
  public void makeStaticBlockObject() {
    System.out.println("Creating block1");
    
    StaticBlock block1 = new StaticBlock();
    System.out.println("Created block1");
    System.out.println("-------------------------");
    System.out.println("Creating block2");
    
    StaticBlock block2 = new StaticBlock();
    System.out.println("Created block2");
  }
}
```
<br>

```bash
<출력결과>

Creating block1
First static Constructor //1번만 호출
Second static Constructor //1번만 호출
StaticBlock Constructor
Created block1
----------------------
Creating block2
StaticBlock Constructor //first, second static constructor 호출 안함
Created block2
```
<br>

두개의 StaticBlock 객체를 만들었지만 결과를 보면 static 블록은 딱 1번만 호출이 되었다. 
그리고 호출된 순서를 보자면 생성자보다 먼저 호출된 것을 확인할 수 있다. 이처럼, static블록은 
클래스를 초기화 할 때 꼭 수행되어야 하는 작업이 있을 경우 유용하게 사용할 수 있다. 
<br>
<br>
<br>
<br>
<br>
<br>

























