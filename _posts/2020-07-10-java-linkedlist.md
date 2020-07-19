---
title: "[자료구조] LinkedList 자바로 직접 구현해보기"
layout: single    
read_time: true    
comments: true   
categories: 
 - Data structure  
toc: true    
toc_sticky: true    
toc_label: contents    
description: 구현하기 까다로운 리스트 중 하나로 연결리스트의 구조 파악해보기
last_modified_at: 2020-07-10       
---

## LinkedList

- 한국어로는 연결리스트라고 불림
- 배열과 달리 순차적으로 연결된 공간에 데이터를 나열하는 데이터 구조
- 떨어진 곳에 존재하는 데이터를 화살표로 연결해 관리 

<br>
<br>
<br>

### 기본 구조와 용어

- 노드(Node) : 데이터 저장단위(데이터, 포인터로 구성)
- 포인터(Pointer) : 각 노드 안에서 다음이나 이전의 노드와 연결정보를 가지고 있는 공간

<br>
<br>
<br>


### 장단점

- 장점
  + 미리 데이터 공간을 할당하지 않아도 됨
  + 배열은 미리 할당해야 한다는 차이가 있음
  
- 단점
  - 연결을 위한 별도의 공간이 필요해 저장공간 효율 낮음
  - 찾을 때는 선형탐색을 수행해 속도가 느림
  - 중간 데이터 삭제 시, 앞 뒤 데이터 연결 재구성 필요

<br>
<br>
<br>



### 직접 구현해보기

<br>

```java
public class LinkedList<E> {
    //노드
    class Node<E> {
        private E data;//입력한 데이터(오로지 참조용임, 주의!)
        private Node<E> next;//포인터(다음 노드 참조용)
    	
        //constructor
    	Node(E data, Node<E> next) {
        	this.data = data; 
	        this.next = next; 
    	}
    }
    
    private Node<E> head; //머리노드(오로지 참조용, 머리 노드 그 값 자체가 아님을 주의)
    private Node<E> current; //선택된 노드(검색하거나 삭제되는 노드)
	
    //constructor: 초기화, 어떠한 값도 가리키지 않도록 설정
    public LinkedList() {
        head = current = null; //머리노드는 null -> 노드가 하나도 없는 비어있는 연결리스트 생성
    }
    
    public E search(E obj, Comparator<? super E> c) {
        Node<E> ptr = head; //현재 스캔 중인 노드
        
        while(ptr!=null) {
            if(c.compare(obj, ptr.data) == 0) { // 검색 성공(compare가 반환하는 0은 성공)
                current = ptr;
                return ptr.data;
            }
            ptr = ptr.next; //다음 노드 선택
        }
        return null; //검색 실패 
    }
    
    //머리노드에 삽입
    public void addFirst(E obj) {
        Node<E> ptr = head; // 삽입 전 머리노드
        head = current = new Node<E>(obj, ptr)
    }
    
    //꼬리에 삽입
    public void addLast(E obj) {
        if(head == null) {
            addFirst(obj);
        }else {
            Node<E> ptr = head;
            while(ptr.next != null) { 
                ptr= ptr.next; //while문이 끝까지 돌아 다음 노드가 존재하지 않는다면 ptr은 맨 마지막 노드를 가리키게됨
            }
            ptr.next = current = new Node<E>(obj, null);//마지막은 next 값이 null이어야함(참조하는 값이 없어서)
            //ptr.next는 새로 삽입한 obj를 참조하도록 업데이트 해야함
        }
    }
    
    //머리노드일 경우 삭제
    public void removeFirst() {
        if(head != null) { //리스트가 비어있지 않다면 
            head = current = head.next;
        }
    }
    
    //임의의 노드 삭제
    public void remove(Node p) {
        if(head != null) {
            if(p == head) {
                removeFirst();
            }else {
                Node<E> ptr = head;
                
                while(ptr.next != p) {//머리노드부터 p(찾으려는 값) 까지 
                    ptr = ptr.next; //삭제할려는 값 바로 앞쪽 노드를 반환하게 함
                    if(ptr == null) return; //못찾음
                }
                ptr.next = p.next; //p값이 삭제 된 후, ptr의 next가 p의 next값을 참조하도록 연결 구조 변경
                current = ptr;
            }
        }
    }
    
    //모든 노드 삭제
    public void clear() {
        while(head != null) { //처음부터 끝(null 나올때까지) 모든 요소 삭제
            removeFirst();
        }
        current = null;
    }
    
    //리스트가 비어있지 않고 선택노드(current) 뒤에 노드가 있을 때만 뒤로 이동
    public boolean next() {
        if(current == null || current.next == null) {
            return false;
        }
        current = current.next; 
        return true;
    }
    
    //선택노드(current) 출력
    public void printCurrentNode() {
        if(current == null) {
            System.out.println("없어요..");
        }else {
            System.out.println(current.data);
        }
    }
    
    public void dump() {
        Node<E> ptr = head; //머리부터
        
        while(ptr != null) { //꼬리까지 스캔
            System.out.println(ptr.data);
            ptr = ptr.next; //다음을 가리키도록 설정
        }
    }
}


```
<br>
<br>
<br>
<br>
<br>
<br>



