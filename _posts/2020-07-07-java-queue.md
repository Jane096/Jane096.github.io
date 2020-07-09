---
title: "[자료구조] 자바로 구현해보는 큐(Queue)"
layout: single    
read_time: true    
comments: true   
categories: 
 - Data structure  
toc: true    
toc_sticky: true    
toc_label: contents    
description: 선입선출의 구조를 가지는 큐를 직접 구현한 게시물
last_modified_at: 2020-07-07       
---

큐(Queue) 자료구조는 스택과 마찬가지고 데이터를 일시적으로 쌓아놓는다. 하지만 스택과 달리, 
**선입선출(First in First Out)** 의 구조를 가진다는 것이 다른점이다. 
<br>

큐에 데이터를 넣는 작업을 **enqueue**, 데이터를 꺼내는 작업을 **dequeue** 라고 정의한다. 
데이터를 꺼내는 쪽은 **front** , 데이터를 넣는 쪽을 **rear** 라고 한다. 
<br>

큐는 스택과 마찬가지로 배열을 사용하여 구현할 수 있다. front부터 차곡차곡 숫자를 인큐 한다. 
처리의 복잡도는 O(1)로 적은 비용으로 구현이 가능하다.
<br>

그러나 반대로 dequeue할 때는 front의 값을 꺼내고 한칸씩 앞으로 이동을 수행하기 때문에 O(n)이라는 
시간복잡도가 나온다. 데이터를 꺼낼때마다 이런 처리를 하게 된다면 상당히 비효율적이다. 
<br>
<br>
<br>

## 링 버퍼로 큐 만들기

링 버퍼 큐는 배열요소를 앞쪽으로 옮기지 않는 큐를 의미한다. 배열의 처음과 끝을 연결해 어떤 요소가 첫번째이고 어떤 요소가 
마지막인지 식별하기 위한 변수로 front와 rear를 사용한다. 
<br>
<br>
<br>

### 링버퍼 enqueue & dequeue

임의의 숫자를 enqueue한다면 rear가 가리키는 공간에 데이터를 입력하고 rear는 그 다음칸을 가리키도록 인덱스를 증가시킨다. 
반대로 dequeue를 하게 되면 front가 가리키는 값이 삭제가 되고 front값은 그 다음 인덱스를 가리키게 되어있다. 
<br>
<br>
<br>

## 링 버퍼 큐 구현해보기

```java
public class IntQueue {
  private int max;
  private int front;
  private int rear;
  private int num;
  private int[] que;
  
  public class EmptyIntQueueException extends RuntimeException {
    public EmptyIntQueueExceptioin() {}
  }
  
  public class OverflowIntQueueException extends RuntimeException {
    public OverflowIntQueueException() {}
  }
  
  public IntQueue(int capacity) {
    num = front = rear = 0;
    max = capacity;
    try {
      que = new int[max];
    }catch(OutOfMemoryError e) {
      max = 0;
    }
  }
  
  public int enque(int x) throws OverflowIntQueueException {
    if(num >= max) {
      throw new OverflowIntQueueException(); //현재 꽉참
    }
    que[rear++] = x;
    num++;
    
    if(rear == max) {
      rear = 0;
    }
    return x;
  }
  
  public int deque() throws EmptyIntQueueException {
    if(num <= 0) {
      throw new EmptyIntQueueException();
    }
    int x = que[front++];
    num--;
    
    if(front == max) {
      front = 0;
    }
    return x;
  }
  
  public int peek() throws EmptyIntQueueException {
    if(num <= 0) {
      throw new EmptyIntQueueException();
    }
    return que[front];
  }
  
  public int indexOf(int x) {
    for(int i=0; i<num; i++) {
      int idx = (i+front) % max;
      if(que[idx] == x) {
        return idx;
      }
    }
    return -1; //검색실패
  }
  
  //모든데이터 삭제
  public void clear() {
    num = front = rear = 0;
  }
  
  //최대 용량 확인
  public int capacity() {
    return max;
  }
  
  //데이터의 수 확인
  public int size() {
    return num;
  }
  
  //Queue가 비어있는지 확인
  public boolean isEmpty() {
    return num <= 0;
  }
  
  //Queue가 꽉 찼는지 확인
  public boolean isFull() {
    return num >= max;
  }
  
  //모든데이터를 
  public void dump() {
    if(num <= 0) {
      System.out.println("Queue is empty");
    }else {
      for(int i=0; i<num; i++) {
        System.out.println(que[(i+front) % max] + " ");
      }
      System.out.println();
    }
  }
}
```
<br>

### Queue 클래스 IntQueue

> - que: 인큐하는 데이터를 저장하기 위한 Queue 본체용 배열    
> - max: Queue의 최대 용량을 저장하는 필드, que배열에 저장할 수 있는 최대 요소의 갯수를 의미    
> - front: enqueue하는 데이터 가운데 첫 번째 요소의 인덱스를 저장하는 필드     
> - rear: enqueue한 데이터 가운데 맨 나중에 넣은 요소의 하나 뒤의 인덱스를 저장하는 필드   
> - num: 현재 Queue의 데이터 수를 나타낸 필드, front와 rear의 값이 같은 경우 Queue가 가득 찼는지 여부를 위해 필요, 가득찼을 때는 num과 max값 같음(반대는 0)     

<br>
<br>
<br>

### 생성자 IntQueue

생성자는 큐 본채용 배열을 생성하는 준비 작업을 수행한다. 생성 시에 num, front, rear는 모두 0으로 초기화해 비어있게 만든다. 
<br>
<br>
<br>

### enqueue()

Queue에 데이터를 삽입하는 메서드이다. 인큐에 성공하면 해당 값을 그대로 반환한다. 큐가 가득차있다면 설정한 예외를 
발생시키도록 한다. 
<br>

만약 front와 rear값이 같다면 더이상 입력할 수 있는 공간이 없다는 뜻이니 최대용량 max와 같다면 배열의 처음인 0으로 변경을 해야한다. 
<br>
<br>
<br>

### dequeue()

Queue에서 데이터를 디큐하고 그 값을 반환하는 메서드이다. 큐가 비어있다면(num <= 0) 예외를 발생시킨다. 
처음부터 차례대로 디큐를 수행하며 front값을 1증가 시키고, 데이터 갯수를 나타내는 num 값은 -1을 수행해준다. 
front값이 큐의 용량인 max와 같아진다면 front값을 처음인 0으로 변경해야한다. 
<br>
<br>
<br>

### peek() 

맨 앞의 데이터(dequeue하면서 꺼내질 데이터)를 몰래 엿보는 메서드이다. 데이터를 꺼내지 않고 조사만 하기 때문에 
front, rear, num의 값은 변동이 없다. (비어있을 때는 예외를 발생시키도록 한다.)
<br>
<br>
<br>

### indexOf()

큐의 배열에서 내가 원하는 데이터 x값이 저장되어 있는지 인덱스 위치를 알아내는 메서드이다. front에서 rear쪽으로 선형 검색을 수행하며
스캔의 시작은 front라는 것을 명심하자. 그래서 인덱스 계산이 **(i + front) % max** 로 수행이 된다.
<br>
<br>
<br>

## 프로그래머스 - 기능개발

```java

import java.util.*;

class Solution {
    public int[] solution(int[] progresses, int[] speeds) {
        Queue<Integer> works = new LinkedList<>();
        
        for(int i=0; i<progresses.length; i++) {
            int leftover = 100 - progresses[i];
            if(leftover % speeds[i] != 0) {
                works.offer(leftover / speeds[i] + 1);    
            }else{
                works.offer(leftover / speeds[i]);
            }
        }
        
        List<Integer> list = new ArrayList<>();
        int prev = works.poll();
        int count = 1;
        
        while(!works.isEmpty()) {
            int current = works.poll();
            if(prev >= current) {
                count++;
            }else{
                list.add(count);
                count = 1;
                prev = current;
            }
        }
        list.add(count);
        
        int[] answer = new int[list.size()];
        for(int i=0; i<list.size(); i++) {
            answer[i] = list.get(i);
        }
        return answer;
    }
}
```
<br>

**기능개발 문제**  
<https://programmers.co.kr/learn/courses/30/lessons/42586>

프로그래머스에서 Queue를 활용하여 푸는 문제 중 기능개발 문제에 적용해서 풀어보았다. 
100% 에서 현재 진행된 만큼의 수를 빼고, 그 남은 양을 각자의 속도로 며칠 안에 할 수 있는지 갯수를 세어보는 문제이다. 
<br>

첫번째 반복문에서는 남은 일 수가 확실하게 떨어지지 않을 때만 1일을 더해줘서 일 수를 체크하면 된다.
while문이 시작되기 전에 첫번째 값을 poll해서 미리 변수 prev에 저장해두고 그 다음 반복문 안에서 두번째 값을 poll해 current 변수에 담아준다. 
prev와 current를 비교해 prev가 크다면 count를 누적하고 그렇지 않다면 그대로 count 값을 list에 담아준다. 
<br>
<br>

list에 담고 나서는 다른 숫자를 비교해야 하기 때문에 1로 다시 초기화 한 후, Queue가 모두 poll 될 때 까지 반복문을 돌리면 모든 값들이 확인이 된다. 
list에 최종적으로 다 담기게 되면 그 사이즈 만큼 answer배열 크기를 지정하고 간단히 list의 모든 값을 읽어오기만 하면 된다. 
<br>
<br>
<br>

### 놓친 점

스택에 남은 일 수를 계산해 넣는 부분은 금방 해결을 했지만 여전히 요소 간 비교해야 하는 부분에서 많이 막히는 것 같다. 
어떻게 하면 효율적으로 값을 비교할 수 있는지 확실하게 그림이 그려지지 않는 것 같다. 
또한 else에서 count를 1로 초기화 시켜주어야 하는 점도 놓치기 쉬운 부분일거라고 생각한다. 
<br>
<br>
<br>
<br>
<br>
<br>





















