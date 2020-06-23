---
title: "[자료구조]배열과 리스트의 관계"
layout: single    
read_time: true    
comments: true   
categories: 
 - Data structure  
toc: true    
toc_sticky: true    
toc_label: contents    
description: 배열과 리스트의 대한 차이점과 어떻게 동작하는지
last_modified_at: 2020-06-23   
---   
List는 특정 타입 값들이 순차적으로 정렬 된 Collection이다. 자바에서 가장 대표적인 기능이 바로 LinkedList와 ArrayList이다.   
ArrayList는 데이터를 검색할 때, LinkedList는 데이터를 삽입/삭제 할 때 편한 자료구조 라는 것은 흔히 알 수 있는 사실이다.   
<br>
<br>

또한 List를 사용하면 상황에 따라 타입을 변경할 수 있다는 장점이 있다.   
그렇다면 일반 배열과 List는 정확하게 뭐가 다른 것인지    
List의 구현 방법의 차이를 알기 전, 어떻게 동작하는지 정확히 이해를 할 필요가 있었다.   
<br>
<br>

## 배열(Array)
```java
public void array() {

  //배열 정의
  final int[] integers = new int[3]; 
  final boolean[] bools = {false, true, false};
  final String[] str = new String[] {"one", "two"};
  
  final Random r = new Random();
  final String[] randomArrayLength = new String[r.nextInt(100)]; //계산된 값을 이용해 배열 생성
}
```
<br>

배열은 정의할 때 크기를 반드시 지정해야 하고 이미 정해진 크기를 밑에서 임의적으로 바꿀 수가 없다.    
또한 배열의 원소에는 인덱스 값이 지정되어 있어 직접 접근할 수 있는데 이를 **랜덤접근** 이라고 한다.   
<br>
<br>

만약 배열의 모든 인덱스를 사용 중인데 원소를 추가할려면 크기를 늘려야 한다.   
실제로는 더 큰 배열을 새롭게 만들고 현재 배열의 모든 원소를 JVM이 한꺼번에 복사해 새로운 배열에 넣어준다. 
그리고 새로운 배열이 원본 배열의 주소를 가리키도록 재할당을 한다. 
<br>
<br>

System의 arrayCopy라는 정적메서드가 배열의 일부나 전부를 새로운 배열로 복사를 가능하게 해준다.
<br>
```java
//배열의 크기 확장
public void arrayCopy() {
  int[] integers = {0, 1, 2, 3};
  
  int[] newIntegersArray = new int[integers.length+1];
  System.arrayCopy(integers, 0, newIntegersArray, 0, integers.length);
  integers = newIntegersArray;
  integers[5] = 5;
  
  assertEqual(5, integers[5]);
}
```
<br>

값을 재할당 하기 때문에 정수 타입의 배열에는 final 키워드를 사용할 수 없으며 배열 대신 
List인터페이스를 이용할 수도 있다. 
<br>
<br>

### ArrayList
ArrayList 클래스는 배열의 초기 크기를 지정할 수 있고, 동시에 새로운 원소를 추가할 때 마다 자동으로 더 큰 배열을 할당한다.    
새로운 원소를 추가할 때 마다 재할당이 일어나기 때문에 시간이 소요되고 메모리의 용량을 많이 소모한다. 
이러한 이유로 크기가 큰 컬렉션을 이용한다면 애초에 크게 잡아주는게 좋다. 
<br>
<br>

만약 ArrayList의 시작 또는 중간에 데이터를 삽입하게 된다면 그 다음의 모든 데이터들은 공간을 만들기 위해 이동이 불가피하다. 
배열의 크기가 크면 그만큼 연산량이 어마어마할 수 밖에 없다.(특히 시작위치에 넣는다면 더더욱-뒤의 모든 데이터들의 이동을 위한 연산때문)     
_배열의 크기는 단방향성이라는 사실!_
<br>
<br>

그렇다고 원소를 지운다 해도 배열 자체의 크기는 줄어들지 않는다. 이런 특성 때문에 ArrayList는 삽입/삭제에 비효율적이라는 것이다.    
이럴 경우 LinkedList가 더 효율적이라는 말을 많이 들었는데 왜인지 코드를 보며 이해했다.   
<br>

## LinkedList
```java
//LinkedList 클래스로 리스트 구현
public class SimpleLinkedList<E> {
  private static class Element<E> {
    E value;
    Element<E> next;
  }
  private Element<E> head;
}
```
<br>

LinkedList의 인스턴스는 Element라는 리스트의 첫부분을 가리키는 head필드만 참조한다. 
Element 클래스의 내부에는 또다시 재귀적 타입 next필드를 볼 수 있는데, 이걸 이용하면 리스트 사이를
쉽게 이동할 수 있고 순서대로 각 원소를 찾아 처리할 수 있다.   
<br>
<br>

단, LinkedList로 원소를 검색할 때는 해당 인덱스를 찾을 때까지 계속 순회해야 한다. 순회도 단방향성이며 
ArrayList가 랜덤접근을 통해 찾는 것과는 매우 대조적이다.    
<br>
<br>

LinkedList는 양방향 포인터 구조로 인접해있는 원소를 참조하여 체인처럼 연결해 관리되기 때문에 역방향으로의 검색이 쉽다.   
그리고 ArrayList처럼 배열 재할당 과정으로 인한 손실이 일어나지 않고 데이터 삭제 시 리스트의 크기가 작아져
리스트의 처음이든 중간이든 상관없이 중간에 원소를 삽입/삭제 동작에 사용하면 좋다.    
(Stack같이 특수한 자료구조 사용에도 LinkedList를 사용하는 것이 바람직하다.)
<br>
<br>

## Queue 와 Deque?

### Queue
first in first out(선입선출) 자료구조로, 새 원소 추가는 **add()**, 오래된 원소 제거는 **remove()**, 
가장 오래 된 원소를 반환하지만 삭제는 하지 않는 **peek()**이 대표적인 메서드이다. 
<br>

```java
//LinkedList에 Queue를 구현
public void queueInsert() {
  final Queue<String> queue = new LinkedList<>();
  queue.add("first");
  queue.add("second");
  
  assertEquals("first", queue.remove());
  assertEquals("second", queue.peek());
}
```
<br>

위와 같은 코드로 LinkedList를 통해 빈번한 삽입 삭제가 일어날 시 유용하게 쓰일 수 있다.    

**Deque?:** 'deck'이라 발음하며 Queue interface의 확장이고 양 끝에 원소를 추가하고 삭제할 수 있다 
{: .notice--info}   

<br>
<br>
<br>
<br>
<br>













