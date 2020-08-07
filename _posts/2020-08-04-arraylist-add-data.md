---
title: "Arraylist에 addAll을 사용하여 값을 넣어보기"
layout: single    
read_time: true    
comments: true   
categories: 
 -  java
toc: true    
toc_sticky: true    
toc_label: contents  
description: Arraylist에서 값이 들어가는 과정
last_modified_at: 2020-08-04   
---

Arraylist에서 `addAll` 이라는 메서드를 사용하여 아래와 같이 데이터를 
입력한다고 생각해보자
<br>
<br>

```java
public void checkArrayList3() {
  ArrayList<String> list = new ArrayList<String>();
  
  list.add("A");
  list.add("B");
  list.add("C");
  list.add("D");
  list.add("E");
  list.add(1, "A1");
  
   ArrayList<String> list = new ArrayList<String>();
   list.add("0 ");
   list.addAll(1, "A1");
   
   for(String e : list2) {
    System.out.println("List2 " + e);
   }
}
```
<br>

```
List2 0
List2 A
List2 A1
List2 B
List2 C
List2 D
List2 E
```
<br>

출력결과를 보면, list에 있는 내용들이 모두 list2에 복사가 되었고 
`addAll()` 메서드를 호출하기 전에 추가한 "0" 이후에 저장이 되었다. 
복사를 할 일이 있다면 아래와 같은 방법을 써도 괜찮다.
<br>
<br>

```java
ArrayList<String> list2 = new ArrayList<String>(list);
```
<br>

Arraylist의 API문서를 잘 살펴보면 Collections 인터페이스를 구현한 어떠한 
클래스도 포함시킬 수 있도록 생성자가 `ArrayList(Collection<? extends E> c)` 로 정의되어 있기 때문에 
해당 생성자를 활용하여 list를 복사할 수 있다. 
<br>
<br>
<br>

```java
public void checkArrayList4() {
  ArrayList<String> list = new ArrayList<String>();
  list.add("A");
  
  ArrayList<String> list2 = list;
  list.add("oops!");
  
  for(String e : list2) {
    System.out.println("List2 " + e);
  }
}
```
<br>

이런식으로 list2 값을 바로 list로 치환을 할 수도 있다. 
그리고 중간 줄에 보면 list2가 아니라 list에 값도 추가를 해주었다. 
<br>
<br>

```
출력결과

List2 A
List2 oops!
```
<br>

코드를 자세히 보면, list2에 "oops!" 라는 값을 넣어준 적이 없다. 
그럼에도 list2는 list에 저장했던 oops를 출력했다. 
<br>
<br>

```java
list2 = list;
```
<br>

위의 문장의 의미는 단순히 list2가 list값을 사용하겠다는 의미가 아니라. list라는 객체가 생성되어 
**참조되고 있는 주소** 까지도 사용을 하겠다는 의미이다. 
자바에서는 모든 객체가 생성되면 JVM이 알아서 객체가 위치하는 주소를 부여해주기 때문에 
개발자가 직접 주소를 알아야 할 필요가 없다. 
<br>

`list2 = list` 와 같이 코드를 입력하게 되면 두 변수명을 다르지만 하나의 객체가 변경되면 
다른 이름의 변수를 갖는 객체의 내용도 바뀌게 된다. 
<br>

하나의 Collections 관련 객체를 복사할 땐, 생성자 또든 `addAll()` 메서드를 사용하는 것이 더 낫다. 
<br>
<br>
<br>
<br>
<br>
<br>
<br>












