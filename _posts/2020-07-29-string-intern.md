---
title: "intern()을 사용을 지양해야 하는 이유"
layout: single    
read_time: true    
comments: true   
categories: 
 -  java
toc: true    
toc_sticky: true    
toc_label: contents    
description: intern 메서드가 성능저하를 불러오는 이유
last_modified_at: 2020-07-29       
---

간혹 초보들이 String클래스를 활용하면서 많이 실수할 수 있다는 부분이 있다.
바로 **intern()** 이라는 메서드 이다. 
<br>
<br>

이 메서드는 자바로 구현되어있지 않고 C언어로 구현되어 있는 native 메서드 중 하나이다. 
무조건 쓰지 말라는 것은 아니지만 남용하게 되면 심각한 성능저하를 불러일으킨다. 

<br>
<br>

```java
public void internCheck() {
  String text1 = "Java Class";
  String text2 = "Java Class";
  String text3 = new String("Java Class");
  
  System.out.println(text1 == text2);
  System.out.println(text1 == text3);
  System.out.println(text1.equals(text3));
}
```
<br>
<br>

```bash
출력결과

true
false
true
```
<br>

text1과 text2의 객체를 같이 생성하면 String클래스에서 관리하는 문자열 풀(pool)에 해당값이 있는지 확인하고 
존재한다면 기존의 객체를 참고하고, text3과 같이 String객체를 생성해버리면 같은 문자열의 존재 여부에 상관 없이 
새로운 객체를 생성을 한다(pool에 존재여부 체크 안함)
<br>
<br>

```java
public void internCheck() {
  String text1 = "Java Class";
  String text2 = "Java Class";
  String text3 = new String("Java Class");
  
  //intern 메서드 추가
  text3 = text3.intern();
  
  System.out.println(text1 == text2);
  System.out.println(text1 == text3);
  System.out.println(text1.equals(text3));
}
```
<br>
<br>

```bash
출력결과

true
true
true
```
<br>

intern()의 추가 후, 위의 결과와 다른 것을 확인할 수 있다. 
new String으로 문자열 객체를 생성해주었지만, 풀에 해당 값이 존재한다면 
풀에 있는 값을 참조하는 객체를 리턴하게 되어있다. 
<br>

반대로 풀에 동일한 객체가 없다면 풀에 새로운 값을 추가한다. 
intern()을 사용한 후 문자열은 equals()가 아닌 == 으로 비교연산이 가능하다. 
<br>

성능면에서 equals()보다는 == 가 훨씬 빠르다. 
intern()은 더 효율적인 연산 사용을 가능하게 해주지만 
새로운 문자열을 끊임없이 추가하는 프로그램이라면 억지로 문자열 풀에 값을 
할당하도록 만들어버리게 된다. 
<br>

저장되는 영역은 한계가 존재하는데 그 영역에 대해서 별도로 메모리를 청소하는 단계를 거치게 된다. 
이러한 동작방식 때문에 작은 연산 하나를 위해서 필요없는 동작이 수행되기에 전체 자바 시스템 성능에 악영향을 끼치는 것이다. 
<br>
<br>
<br>

정리하자면, 우리가 만드는 애플리케이션에 생성하는 문자열이 정해져 있고 그 문자열에 대해서만 intern()메서드를 
사용하는 것이라면 문제가 없다. 그러나 이런 경우는 거의 존재할 수 없기 때문에 intern()의 사용을 지양하는 것이다. 
<br>
<br>
<br>
<br>
<br>
<br>





















