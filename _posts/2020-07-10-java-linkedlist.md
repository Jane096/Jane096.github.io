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
ArrayList나 Vector 같은 자료형들은 각 위치가 정해져 있고 그 위치로 데이터를 찾는 특징이 있다. 
그러나 `LinkedList`는 마치 열차처럼 데이터끼리 연결이 되어 있다. 
<br>
<br>

> LinkedList에서는 해당 데이터가 자신의 바로 앞과 뒤의 정보만을 알고있다.
> 다르게 이야기를 하면, 한 다리 건너 있는 앞이나 뒤의 데이터의 대한 정보를 
> 알고 있을 필요가 없다는 뜻이다. 
> LinkedList에서는 배열처럼 데이터를 담아 순차적으로 뺄 경우에는 필요가 없을 수 있겠지만, 
> 데이터가 지속적으로 삭제, 추가 될 경우 메모리 공간측면에서 효율성이 아주 높다. 

<br>
<br>

ArrayList나 vector는 위치가 정해져 있는 탓에 맨 앞이나 중간에 값을 삽입하게 된다면 
뒤의 데이터를 모두 한칸씩 옮겨주어야 하기 때문에 중간삽입, 삭제 시 효율성이 매우 떨어지게 된다. 
<br>

LinkedList에서는 각 데이터끼리 체인처럼 관리되어 있고 데이터를 삭제할 때 앞에 있는 데이터와 
뒤의 데이터의 연결고리를 끊어주기만 하면 되기 때문에 위치를 맞추기 위해 일괄적으로 이동을 하는 
수고를 덜 수 있다. 

<br>
<br>
<br>

### 기본 구조와 용어

- 노드(Node) : 데이터 저장단위(데이터, 포인터로 구성)
- 포인터(Pointer) : 각 노드 안에서 다음이나 이전의 노드와 연결정보를 가지고 있는 공간

<br>
<br>
<br>


### LinkedList 파헤쳐 보기

LinkedList는 일반 배열 타입의 클래스와는 다르게 생성자로 객체를 생성할 때 데이터들이 앞 뒤로 
연결되는 구조이기 때문에 미리 공간을 만들어 놓을 필요가 없다(선언시에 미리 크기를 지정할 필요가 없다)

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



