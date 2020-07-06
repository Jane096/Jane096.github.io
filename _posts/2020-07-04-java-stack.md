---
title: "[자료구조] 자바로 스택 구현하기 "
layout: single    
read_time: true    
comments: true   
categories: 
 - Data Structure  
toc: true    
toc_sticky: true    
toc_label: contents    
description: 스택 자료구조의 직접구현과 관련된 메서드의 동작 원리 
last_modified_at: 2020-07-04       
---
**스택(Stack)** 이란 데이터를 일시적으로 저장하기 위한 자료구조로, 가장 나중에 넣은 데이터를 가장 먼저 꺼내는 
후입선출(Last in First out)의 입출력 순서를 가진다. 
<br>
<br>

## 주요 method

- push(): 스택에서 데이터를 넣는 작업
- pop(): 스택에서 데이터를 꺼내는 작업
- peek(): 스택의 꼭대기의 데이터를 몰래 엿보는 기능, 비어있다면 EmptyIntStackException 발생
- indexOf(): 스택 본체 배열에 원하는 값이 포함되어있다면 배열의 어디에 들어있는지 조사함
- clear(): 스택의 모든 요소를 삭제
- capacity(): 스택의 용량(max)을 확인함
- size(): 현재 스택에 쌓여있는 데이터의 값 확인
- isEmpty(): 스택이 비어있는지 확인
- isFull(): 스택이 가득찼는지 검사하는 메서드(boolean 반환)
- dump(): 스택에 쌓여있는 모든 데이터를 바닥에서 꼭대기 순으로 표시

<br>
<br>
<br>

## 호출과 실행 과정

```java
//예시

void x() {

}
void y() {

}

void z() {

}

int main() {
  z();
}
```
<br>

1. 가장먼저 main 메서드르 찾아가 실행하여 z()를 호출한다. 
2. 호출된 메서드 z()는 x()와 y()를 순서대로 호출한다. 
3. 메서드를 호출할 때 마다 하나씩 푸시한다. 
4. 모든 실행을 종료하고 호출한 원래의 메서드로 돌아갈 때 종료할 메서드를 팝한다. 

<br>
<br>
<br>

## 스택 만들기

```java
public class IntStack {
  private int max; //스택의 용량
  private int ptr; //스택 포인터: 스택에 쌓여있는 데이터 수
  private int[] stk; //스택 본체
  
  //스택이 비어있는 경우 예외발생
  public class EmptyIntStackException extends RuntimeException {
    public EmptyIntStackException() {}
  }
  
  //스택이 꽉 차있는 경우 예외발생
  public class OverFlowIntStackException extends RuntimeException {
    public OverFlowIntStackException() {}
  }
  
  public IntStack(int capacity) {
    ptr = 0;
    max = capacity;
    try {
      stk = new int[max]; //스택 본체를 위한 배열 생성
    }catch(OutOfMemoryError e) { //배열 생성 예외발생
      max = 0;
    }
  }
  
  public int push(int x) throws OverFlowIntStackException {
    if(ptr >= max) throw new OverFlowIntStackException(); //스택 가득찼음
    return stk[ptr++] = x;
  }
  
  public int pop() throws EmptyIntStackException {
    if(ptr <= 0) throw new EmptyIntStackException(); //스택 비어있음 
    return stk[--ptr];
  }
  
  public int peek() throws EmptyIntStackException {
    if(ptr <= 0) throw new EmptyIntStackException();
    return stk[ptr-1];
  }
  
  public int indexOf(int x) {
    for(int i=ptr-1; i>=0; i--) {
      if(stk[i] == x) return i; // 검색성공
    } 
    return -1; //검색 실패
  }
  
  //스택 비우기
  public void clear() {
    ptr = 0;
  }
  
  //스택 용량
  public int capacity() {
    return max;
  }
  
  //스택 안의 데이터 수
  public int size() {
    return ptr;
  }
  
  public boolean isEmpty() {
    return ptr <= 0;
  }
  
  public boolean isFull() {
    return ptr >= max;
  }
  
  //스택안에 있는 모든데이터 표시하기 
  public void dump() {
    if(ptr <= 0) 
      System.out.println("Stack is empty");
    else {
      for(int i=0; i<ptr; i++) {
        System.out.println(stk[i] + " ");
      }
      System.out.println();
     }
  }
}
```
<br>
<br>

### 생성자 IntStack

생성자는 스택 본체용 배열을 생성하고 준비 작업을 수행한다. 
생성할 때 스택은 비어있기 때문에 ptr은 0으로 초기화한다. 매개변수로 capacity
로 전달받은 값을 스택 용량을 나타내는 max에 복사한다. max값을 기준으로 배열 stk의 
본체를 생성한다. 
<br>
<br>

### push()

스택에 데이터를 푸시하는 메서드이다. 스택이 가득차있는 경우 예외발생하도록 구성한다. 
전달받은 데이터 x를 배열요소 stk[prt]에 저장하고 인덱스를 증가시켜 다음칸에 입력할 수 있도록 한다. 
<br>
<br>

### pop()

스택의 꼭대기에서 데이터를 제거하고 그 값을 반환하는 메서드이다. 스택이 비어있다면 
예외를 발생시킨다. 먼저 ptr의 값을 감소시키고 그때 str[ptr]에 저장되어 있는 값을 반환한다. 
<br>
<br>

### peek()

스택의 꼭대기에 있는 데이터를 확인하는 메서드이다. 스택이 비어있지 않는다면 
꼭대기의 요소 stk[ptr - 1]의 값을 반환한다. 이때 데이터의 입력과 출입이 없어 스택포인터는 변하지 않는다. 
<br>
<br>

### 검색 메서드 indexOf()

배열 stk에 x와 같은 값이 들어있는지, 들어있다면 어느 인덱스에 들어있는지 조사하는 메서드이다. 
검색은 꼭대기부터 바닥까지 선형검색을 수행하며 배열 인덱스가 큰쪽에서 작은 쪽으로 스캔한다. 성공한다면 
해당 인덱스를 반환하고 못찾았다면 -1을 반환한다. 
<br>
<br>

### Others

>clear(): 스택의 모든 요소를 삭제    
>capacity(): 스택의 용량(max)을 확인함     
>size(): 현재 스택에 쌓여있는 데이터의 값 확인    
>isEmpty(): 스택이 비어있는지 확인     
>isFull(): 스택이 가득찼는지 검사하는 메서드(boolean 반환)    
>dump(): 스택에 쌓여있는 모든 데이터를 바닥에서 꼭대기 순으로 표시      

위의 설명과 동일하다 :)
<br>
<br>

## 스택이 사용되는 경우

스택은 깊이 우선 탐색(DFS)에서 루트 노드에서 다음 분기(branch)로 넘어가기 전에 해당 분기를 
완벽하게 탐색할 때 같이 사용하게 된다. 
<br>
<br>
<br>

## Codility Brackets로 스택 문제 풀이

>A string S consisting of N characters is considered to be properly nested if any of the following conditions is true:
><br>
>- S is empty;
>- S has the form "(U)" or "\[U]" or "{U}" where U is a properly nested string;    
>- S has the form "VW" where V and W are properly nested strings.   
>
>For example, the string "{\[()()]}" is properly nested but "(\[)()]" is not.
<br>
<br>

```java
import java.util.Stack;

class Solution {
     public int solution(String S) {
       Stack<Character> stack = new Stack<>();
       
       for(char start : S.toCharArray()){
           if(start == '{' || start == '[' || start == '(') {
            stack.push(start);    
           }else{
            if(stack.isEmpty()) {
               return 0;
           }
           char end = stack.pop();
           
           if(start == ')' && end  != '(') return 0;
           if(start == '}' && end  != '{') return 0;
           if(start == ']' && end  != '[') return 0;    
           } 
       }
       if(!stack.isEmpty()) return 0;
       return 1;
    }
}
```
<br>

util패키지 안에 Stack을 import 한 후, 입력 받은 String을 for-each문을 이용해 한 글자씩 읽어오도록 한다. 
그 중에서 Stack안에 push할 글자들은 '{' '\[' '(' 이다. 즉, Stack 안에는 괄호의 시작 부분만 들어가게 되고 닫힌 괄호들은 
그대로 String안에 남게 된다. 
<br>

괄호 시작 부분만 들어있는 Stack에 pop을 하게 되면 end는 열린괄호 부분만을 가리키게 되고 이 후 조건문 안에 start는 
String에 남아있는 닫힌 괄호를 가리켜 두가지를 비교하게 된다.(변수명 때문에 헷갈릴 여지가 있을 듯...) 
<br>

'(' 다음에 ')'가 오지 않으면 0을 리턴하게 해서 nested인지 아닌지를 확인하면 된다. 
Stack이 pop이 모두 완료되면 최종 Stack은 비어있어야 하는데 비어있지 않다면 0을 리턴하고 비어있다면 1을 리턴하면 된다.

<br>
<br>
<br>
<br>
<br>
<br>












