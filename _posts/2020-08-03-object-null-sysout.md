---
title: "객체에 null을 넣고 System.out.println을 해보면?"
layout: single    
read_time: true    
comments: true   
categories: 
 -  java
toc: true    
toc_sticky: true    
toc_label: contents  
description: toString 과 valueOf의 차이
last_modified_at: 2020-08-03   
---

밑의 코드와 같이 `null`을 출력하는 경우가 있다고 생각해보자
<br>
<br>

```java
public void printNull() {
  Object obj = null;
  System.out.println(obj);
  System.out.println(obj+" is object`s value");
}
```
<br>

```
출력결과

null
null is object`s value
```
<br>
<br>

나는 이 부분에 대해서 "당연히 되겠지..?" 라는 생각밖에 못했다. 
위의 상황은 `null`인 obj가 `toString()`을 호출을 하는 경우이다. 
이전에도 봤었지만, `null`인 obj는 아무것도 할당이 되어있는 상태가 아니기 때문에 
`toString()`을 호출을 할 수가 없다. 
<br>
<br>

출력결과를 또 한번 자세히 보면, null은 "null이니까 출력해주겠지"할 수 있어도, 밑에는 
null + is object value 를 합친 값을 출력하고 있다. 이런 결과가 가능한 이유는 
`print()`와 `println()` 메서드는 단순이 toString()을 호출하지 않기 때문이다. 
<br>

String 클래스를 조금 뒤지다 보면 `valueOf()`라는 메서드를 찾을 수 있다. static 메서드라서 
바로 가져다 쓸 수 있다. 저렇게 + 연산으로 합쳐진 경우에는 **String.valueOf(obj)** 가 호출된 상황이다.
<br>
<br>

```java
//Throw Exception

public void printNull() {
  Object obj = null;
  System.out.println(obj.toString());
}
```
<br>

직접 눈으로 확인해보면, 위의 코드는 `NullPointerException`을 발생시키며 종료된다. 
null인 객체를 toString()으로 불러버리면 이렇게 예외를 발생시키기 때문에 객체를 출력할 때는 
`valueOf()`를 사용하는 것을 권장한다고 한다. 
<br>
<br>

```java
public void printNull() {
  Object obj = null;
  //System.out.println(obj.toString()); 주석처리
  System.out.println(obj + " is object`s value"); // 예외발생 안함
}
```
<br>

아까 했던 문장을 주석처리하고 null을 더해주던 두 번째 출력문으로 컴파일을 시도해보면 
신기하게도 에러를 발생시키지 않는다. 그 이유는, 위의 + 연산을 `StringBuilder`로 변환하기 때문이다. 
<br>

```
new StringBuilder().append(obj).append(" is object`s value");
```
<br>

위의 코드블럭 처럼 StringBuilder를 이용해 append하는 방식으로 실행이 되고 StringBuilder의 `append()` 메서드에는 
null을 문제없이 넣을 수 있다. 
<br>
<br>
<br>
<br>
<br>
<br>
<br>

















