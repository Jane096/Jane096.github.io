---
title: "Arraylist 객체에 값을 꺼내보기"
layout: single    
read_time: true    
comments: true   
categories: 
 -  java
toc: true    
toc_sticky: true    
toc_label: contents  
description: Arraylist에서 값이 꺼내오는 과정
last_modified_at: 2020-08-08   
---

Arraylist 에서 값을 꺼내오는 방법을 알기 전에 미리 알아야 하는 기능이 있다. 
`size()` 라고 하는 Arraylist 객체에 들어있는 데이터 갯수를 가져오는 메서드이다. 
일반 배열에서는 배열.length 를 이용하지만 Collections에서는 공통으로 `size()` 라는 기능을 쓰니 
차이점을 잘 알아두어야 한다. 
<br>
<br>
<br>

```java
public void checkArrayList() {
  ArrayList<String> list = new Arraylist<String>();
  list.add("A");
  list.add("B");
  int listSize = list.size();
  
  for(int loop=0; loop<listSize; loop++) {
    System.out.println("list.get("+loop+")="+list.get(loop));
  }
}
```
<br>

for 루프 부분을 보면 해당하는 loop위치의 데이터를 꺼내는 `get()` 사용했다. 
추가로 Arraylist 에서는 `indexOf()` 와 `lastIndexOf()` 라는 메서드도 같이 제공한다. 
<br>
<br>
<br>

### indexOf()와 lastIndexOf()의 차이?

List의 특징이자 List인터페이스 확장하는 Arraylist에서는 데이터의 중복을 허용한다. 
0번째에 1을 넣든, 1번째에 1을 넣든 데이터는 해당위치에 잘 저장이 된다. 
앞에서부터 찾을 때는 `indexOf()`를, 뒤에서부터 찾을 때는 `lastIndexOf()`를 사용하면 된다.
<br>
<br>
<br>

### 배열로 뽑아내는 toArray() 

자바 API문서로 toArray()기능을 자세히 살펴보면 이 메서드는 매개변수가 없고 Object 타입의 
배열로만 리턴을 하게 되어있다. Generic 타입을 매개변수로 받는 toArray()도 같이 존재하는데, 
제너릭으로 선언한 Arraylist 객체를 배열로 생성할 때는 Object타입으로 리턴하는 toArray()는 
사용하지 않는 것이 좋다. 
<br>
<br>

```java
public void checkArrayList() {
  ArrayList<String> list = new ArrayList<String>();
  
  list.add("A");
  String[] strList = list.toArray(new String[0]);
  System.out.println(strList[0]);
}
```
<br>

`toArray()`를 입력한 부분처럼, 이 메서드의 매개변수로 변환하려는 배열의 타입을 지정해주면 된다. 
유심히 살펴보면, **new String[0]** 으로 매개변수를 지정해주었는데, 매개변수로 넘기는 배열은 
그냥 이렇게 의미없이 타입만을 지정하기 위해 사용할 수도 있지만 실제로는 매개변수로 넘긴 
객체에 값을 담아주게 되어있다. 
<br>

하지만 Arraylist 객체의 데이터 크기가 매개변수로 넘어간 배열 객체의 크기보다 클 경우에는 매개변수로 
배열의 모든 값이 `null`로 채워지므로 이렇게 0으로 초기화된 값을 넣어주는게 좋다. 
<br>
<br>

```java
public void checkArrayList() {
  ArrayList<String> list = new ArrayList<String>();
  
  list.add("A");
  list.add("B");
  list.add("C");
  
  String[] tmpArr = new String[5];
  String[] strList = list.toArray(tmpArr);
  System.out.println(strList[0]);
  
  for(String tmp : strList) {
    System.out.println(tmp);
  }
}
```
<br>

```
출력결과

A
B
C
null
null
```
<br>

예시로 하나 출력해보면 정말로 null로 채워주고 출력하는 것을 알 수 있다. 
이런 방식 때문에 꼭 0으로 초기화된 값을 넘겨주어야 한다. 
<br>
<br>
<br>
<br>
<br>
<br>
<br>
























