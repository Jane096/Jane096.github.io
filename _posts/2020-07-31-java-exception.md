---
title: "try-catch를 이용할때의 변수선언"
layout: single    
read_time: true    
comments: true   
categories: 
 -  java
toc: true    
toc_sticky: true    
toc_label: contents    
description: try-catch에서 변수선언하는 법
last_modified_at: 2020-07-31       
---

try-catch에서 많이 할 수 있는 실수가 바로 **변수선언** 이다. 
try 블록은 중괄호에 싸여있으며 try내의 선언한 지역변수는 
catch에서 당연히 사용이 불가하다.
<br>
<br>

```java
public class ExceptionVariable {
  public static void main(String[] args) {
    ExceptionVariable sample = new ExceptionVariable();
    sample.checkVariable();
  }

  public void checkVariable() {
    int[] intArray = new int[5];
    try{
      System.out.println(intArray[5]);
    }catch (Exception e) {
      System.out.println(intArray.length);
    }
    System.out.println("This code must run");
  }
}
```
<br>

```
출력결과
5
This code must run
```
<br>

이 메서드는 문제없이 예외가 발생하여 intArray의 길이를 출력하고 
try-catch 이후의 출력문이 실행된다. 하지만 밑에 처럼 바꾸면 
결과가 달라진다.
<br>
<br>

```java
public class ExceptionVariable {
  public static void main(String[] args) {
    ExceptionVariable sample = new ExceptionVariable();
    sample.checkVariable();
  }

  public void checkVariable() {
    //int[] intArray = new int[5]; 주석처리

    try{
      int[] intArray = new int[5]; //try블록안에 선언
      System.out.println(intArray[5]);
    }catch (Exception e) {
      System.out.println(intArray.length);
    }
    System.out.println("This code must run");
  }
}
```
<br>

위의 메서드의 경우 컴파일 에러를 발생시키게 된다. catch에서는 
intArray가 누군지 모르기 때문에 **"cannot find symbol"** 
이라는 에러메세지를 발생시킨다.
<br>
<br>

이런 탓에 변수선언의 경우 try블록 바로 위에 선언을 해야한다.
<br>
<br>

```java
public class ExceptionVariable {
  public static void main(String[] args) {
    ExceptionVariable sample = new ExceptionVariable();
    sample.checkVariable();
  }

  public void checkVariable() {
    int[] intArray = null; //null로 선언

    try{
      intArray = new int[5]; 
      System.out.println(intArray[5]);
    }catch (Exception e) {
      System.out.println(intArray.length);
    }
    System.out.println("This code must run");
  }
}
```
<br>

```
출력결과
5
This code must run
```
<br>

이런식으로 변수만 미리 설정을 해 놓는다면 null이라도 
try블록 내의 실행될 모든 문장이 무시되는게 아니기 때문에 
정상적으로 컴파일이 된다. catch에서 사용할 변수는 
꼭 try블록 앞에 선언해야 된다는 점을 잊지말아야한다.
