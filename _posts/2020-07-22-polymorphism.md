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
<br>
<br>
<br>

























