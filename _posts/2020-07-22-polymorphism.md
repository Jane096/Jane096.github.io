---
title: "자바의 다형성(Polymorphism)"
layout: single    
read_time: true    
comments: true   
categories: 
 -  java
toc: true    
toc_sticky: true    
toc_label: contents    
description: 상속에서 성립되는 다형성에 대한 정리
last_modified_at: 2020-07-22       
---

```java
public class Parent {
  public Parent() {
    System.out.println("Parent Constructor");
  }
  
  public void printName() {
    System.out.println("Parent - printName()");
  }
}

public class Child extends Parent {
  public ChildOther() {
    System.out.println("Child Constructor");
  }
  
  public void printName() {
    System.out.println("Child - printName()");
  }
}
```
<br>

위와 같이 parent를 상속하는 Child라는 자식클래스가 있다고 생각해보자. 
(부모클래스를 상속하는 자식클래스는 여러개 만들 수 있다!)
<br>
<br>

```java
public class InheritancePoly {
  public static void main(String[] args) {
    InheritancePoly inheritance = new InheritancePoly();
    inheritance.callPrintName();
  }
  
  public void callPrintName() {
    Parent parent 1 = new Parent();
    Parent parent 2 = new Child();
    
    parent1.printName();
    parent2.printName();
  }
}
```
<br>

메인메서드에서 위에 정의한 ChildOther와 부모클래스에 대한 객체를 생성해주고 각각의 참조변수를 가지고 
printName()메서드를 각각 호출해본다. (하나는 parent, 하나는 ChildOther이다)
<br>
<br>

```bash
Parent Constructor
Parent Constructor
Child Constructor

Parent printName()
Child printName()

```
<br>

두 객체 모두 Parent타입으로 선언해줬음에도 불구하고 printName()을 불러오는 객체가 다르다. 
선언시에는 모두 Parent타입으로 선언했지만, 실제로 호출된 메서드는 생성자를 사용한 클래스에 있는 것이 
호출되었다. 왜냐하면 각 객체의 실제타입은 다르기 때문이다.
<br>

즉, 참조자료형을 형변환 하더라도 실제 호출되는 것은 자기 자신의 객체에 있는 메서드 이다. 
이것이 **다형성** 이다. 
<br>
<br>

### 정리

- 생성자
  + 자식클래스의 생성자가 호출되면 자동으로 부모클래스의 매개변수가 없는 기본생성자를 호출하게 되어있다.(매개변수를 받는 생성자를 만들경우 기본생성자를 자동으로 호출하지 않으니 주의)
  + super()는 부모클래스 생성자의 명시적 호출 방법이다. 

- 변수
  + 부모클래스의 private 변수를 제외하고 모든 변수를 자식 클래스에 있는 것처럼 사용할 수 있다. 
  + 부모클래스의 변수와 자식클래스의 변수명은 웬만해선 같은 이름으로 지정하지 않도록 한다. 
  + 부모클래스에 선언된 변수 이외에 새로운 변수를 자식클래스에서 가능하다.

- 메서드
  + 변수와 마찬가지로 자식클래스도 부모클래스의 메서드를 그대로 이어받았기 때문에 사용이 가능하다.
  + 부모클래스의 선언된 메서드로 메서드 Overriding이 가능하다.
  + 자식클래스에서 추가로 새로운 메서드 선언은 얼마든지 가능하다.
  
<br>
<br>
<br>
<br>

## final 키워드

### class에 final을 사용하는 이유

```java
public final class FinalClass {
  //skip
}
```
<br>

**"final"** 이라는 키워드는 영어 해석 그대로 "마지막" 이라는 의미이다. 
상속에서 이 키워드는 Overriding을 해주거나 상속해주는 행위 자체를 할 수가 없다. 

<br>
<br>

만약 FinalClass를 다른 클래스에 extends한다면 상속을 해줄 수 없다는 컴파일 에러가 발생한다. 
이렇게 클래스에 final을 사용하는 이유는 어떠한 클래스의 기능이 매우 중요해서(ex. String) 조금이라도 
변경이 되거나 더이상의 확장을 해서는 안되는 클래스일 경우 final 키워드를 사용한다. 
<br>
<br>
<br>

### method에 final 키워드를 사용하는 이유

```java
public abstract class finalMethodClass {
  public final void printLog(String data) {
    //기능정의
  }
}
```
<br>

메서드에 final을 선언하게 되면 해당 클래스를 상속할 때 Overriding이 불가능 하다는 컴파일 에러를 발생시킨다. 
해당 메서드를 변경하지 못하도록 막을 수 있는 기능이 final 이다. 하지만 많이 쓰는일은 없다고 한다. 
<br>
<br>
<br>

### 변수에 final을 사용하는 이유

```java
public class FinalVariable {
  final int instanceVariable;
}
```
<br>

위의 클래스는 인스턴스 변수 부분에서 에러를 발생시킨다. 인스턴스 변수에 final이 붙은 경우 
저렇게 선언해서는 안된다. 
<br>
<br>

```java
final int instanceVariable = 1;
```
<br>

final 키워드를 변수에 사용할 경우 기억해야 할 것은 반드시 초기값을 지정해주어야 한다는 것이다. 
생성자나 메서드에서 초기화 하는 것은 final의 의도에서 벗어나기 때문에 인스턴스 변수나 클래스 변수에 final을 붙이는 경우 
초기화 하는 것을 잊지말아야한다. 
<br>
<br>

매개변수에도 final 키워드를 사용할 수 있는데, 매개변수의 경우 이미 초기화되어 값이 넘어오기 때문에 
**public void method(final int test)..."** 이런식으로 원래 쓰던 방식처럼 쓸 수 있다. 
<br>
<br>
<br>

### 주의해야할 점 

```java
public void method(final int parameter) {
  final int instanceVariable;
  instanceVariable = 2;
  
  instanceVariable = 3;
  parameter = 4;
}
```
<br>

2로 초기화한 instanceVariable 부분은 컴파일에러를 발생시키지 않는다. 하지만 해당 변수는 final로 선언되어 있기 떄문에 
3으로 값을 변경하여 사용해서는 안된다. parameter 변수도 마찬가지로 이미 초기화 된 값을 넘겨 줬기 때문에 
새로운 값을 할당해서는 안된다. 
<br>
<br>
<br>


<br>
<br>
<br>

























