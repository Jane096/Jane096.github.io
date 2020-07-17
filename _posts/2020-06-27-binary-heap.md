---
title: "[자료구조]이진 힙(binary heap)"
layout: single    
read_time: true    
comments: true   
categories: 
 - Data structure  
toc: true    
toc_sticky: true    
toc_label: contents    
description: 이진 힙 자료구조의 동작 방법과 구현 코드  
last_modified_at: 2020-06-27   
---

앞서 올렸던 이진 트리는 검색의 용도로도 사용되지만, 다른 응용방법도 존재하는데 
그것이 바로 **이진 힙(binary Heap)**이다. 이진 힙은 완전이진트리(균형잡힌)를 기본으로 한 
자료구조이며, 시간복잡도는 **O(log N)**을 가진다.    
<br>
<br>

## 힙(Heap)의 속성(heap property)

힙은 트리에서 부모노드와 자식노드 간의 대소관계가 성립되어야 한다는 속성이 있다. 이러한 속성으로 인해 
순서를 가진 Queue나 List의 가장 작은 원소 또는 가장 큰 원소를 빠르게 접근해야 하는 경우 특히 유용하다.  
<br>

힙의 속성의 따라 두 가지의 종류가 존재하게 된다. 
- 최대 힙: 부모노드의 값이 자식노드의 값보다 항상 큰 힙
- 최소 힙: 부모노드의 값이 자식노드의 값보다 항상 작은 힙  
<br>

왼쪽과 오른쪽으로 형제 사이에 대소관계를 비교해 구성하는 이진 검색 트리와의 차이점이 바로 이 부분이다. 
형제 관계의 대소관계가 일정하지 않으나 부모와 자식 요소의 관계만 일정하다면 힙이라고 할 수 있다. 
<br>
<br>
<br>   

## Heap & BST
Stack Overflow에서 heap과 binary serarch tree(이하 BST)의 findmin/findmax 기능에 대해 이야기가 오고 
간 것을 보았는데, BST에서 가장 큰 원소를 top에 배치하는 것 자체가 진부한 방법이기도 하고 
언제든 원소가 바뀔 수 있어 흔히 잘못알고 있는 사실이라고 한다. 결론은 heap에서도 O(1)의 시간을 가지고 BST에서도 O(log(n))이 아닌 
O(1)의 시간을 가질 수 있다고 한다.  
<br>

BST는 임의의 값 찾기에 O(log(n))을 가져 평균적으로 O(n)시간을 가지는 heap보다 성능적으로 이점을 가지는 부분이라고 한다. 
<br>

반면에 heap은 삽입 동작에서 O(1)의 시간을 가져 O(log(n))의 시간을 가지는 BST보다 성능이 좋다고 한다.   
<br>
<br>

**Referenced by**   
<https://stackoverflow.com/questions/6147242/heap-vs-binary-search-tree-bst/27074221#27074221>
<br>
<br>
<br>

## 힙 정렬

선택 정렬을 응용한 것으로, **가장 큰 값을 루트에 위치** 하는 특징을 이용하는 정렬 알고리즘 이다. 힙에서 가장 큰 값인 루트를 꺼내는 작업을 반복하고 
그 값을 늘어놓으면 배열은 정렬을 마친다. 힙정렬은 다른 정렬과 다르게 **불안정한(unstable)** 정렬이라고도 부른다. 그 이유는 밑의 출력결과를 보면 알 수 있다. 

## Java로 Heap 정렬 구현하기 
```java

public static void main(String[] args) {
  int[] arr = {3, 5, 1, 7, 35, 24};
  heapSort(arr);
}

public static void heapSort(int[] a) {
 int max = a.length;
 
 //build Heap: 힙인지 아닌지 판별 여부(heapify 연산 가능 여부) - 초기상태의 배열이 힙이 아닐 수도 있기 
 for(int i=max/2 - 1; i >= 0; i--) {
  heapify(a, max, i);
 }
 
 //max 노드와 -1을 한 마지막 값(i)을 swap해서 자리를 바꾸고
 //배열의 크기를 1 줄여 마지막은 힙의 일부가 아니도록 수정함
 for(int i=max - 1; i > 0; i--){
  int temp = a[0];
  a[0] = a[i];
  a[i] = temp; //swap
  heapify(a, i, 0);
 }
}

//이진트리로 구성하기 
public static void heapify(int[] a, int length, int i) {
  int parent = i;
  int leftChild = i*2+1;
  int rightChild = i*2+2;
  
  if(leftchild < length && a[parent] < a[leftchild]) {
    parent = leftChild; 
    
  }else if(rightChild < length && a[parent] < a[rightChild]) {
    parent = rightChild;
  }

  //swap
  if(i != parent) {
    int temp = a[parent];
    a[parent] = a[i];
    a[i] = temp;
    
    //재귀호출
    heapify(a, length, parent); 
  }
 }
  
```
<br>

**참고**   

>부모 노드 구하기: a\[parent/2]    
>부모의 왼쪽 자식 노드 구하기: a\[parent x2]   
>부모의 오른쪽 자식 노드 구하기: a\[parent*2+1]  

<br>

백준 알고리즘 2751번을 힙 정렬로 푼 분이 계셔서 참고했는데 left, right값에 +1씩 해줘 계산을 하는 걸 보았다. 음...무슨 말인지 확실하게 이해할 필요가 있을 것 같다. 
<br>

## 추가
for문 돌 때(build heap 부분) 인덱스를 0부터 돌때는 왼쪽 자식노드는 i*2 + 1, 오른쪽은 i*2+2 라고 한다. 
<br>
<br>
<br>

## 최대힙 정렬(heap sort)에서 최소값 찾아보기

```java
package backjoon_test;

import java.util.Arrays;
import java.util.Scanner;

class heap2959 {
	 public static void heapSort(int[] a) {
		 int max = a.length;
         
         //builf heap
         for(int i=max / 2 - 1; i>=0; i--) {
             heapify(a, max, i);
         }
        
         //크기를 줄여가며 반복적으로 힙을 구성 
         for(int i=max - 1; i>0; i--) {
             int temp = a[0];
             a[0] = a[i];
             a[i] = temp;
             heapify(a, i, 0);
         }
      }
    
     public static void heapify(int[] a, int length, int i) {
         int parent = i;
         int leftchild = i*2 + 1;
         int rightchild = i*2+2;
         
         if(leftchild < length && a[parent] > a[leftchild]) {
             parent = leftchild;
         }else if(rightchild < length && a[parent] > a[rightchild]) {
             parent = rightchild;
         }
         
         if(i != parent) {
             int temp = a[parent];
             a[parent] = a[i];
             a[i] = temp;
             heapify(a,length, parent);
             System.out.println(Arrays.toString(a));
         }
     }
     
     //추가된 method
     public static int findMin(int[] a, int size) {
    	 int s = size;
    	 int min = a[s / 2]; //리프노드 인것만 확인하면 됨(narrow down)
    	 
    	 for(int i = s / 2 + 1; i<s; i++) {
    		 min = Math.min(min, a[i]);
    	 }
    	 return min;
     }
     
     public static void main(String[] args) {
         Scanner sc = new Scanner(System.in);
         System.out.println("배열크기: ");
         int size = sc.nextInt();
         
         int[] arr = new int[size];
         
         System.out.println("배열 숫자입력: ");
         for(int i=0; i<arr.length; i++) {
             arr[i] = sc.nextInt();
         }
         
         heapSort(arr);
         int answer = findMin(arr, size);
         
         System.out.println("최종정렬: " + Arrays.toString(arr));
         System.out.println("최소값: " + answer);
     }
    }

```
<br>

```
output
배열크기: 
5
배열 숫자입력: 
5
2
3
4
1
[5, 1, 3, 4, 2]
[1, 4, 3, 5, 2]
[1, 4, 3, 5, 2]
[4, 5, 3, 2, 1]
최종정렬: [5, 3, 4, 2, 1] -> 완벽히 정렬되진 않음
최소값: 1
```
<br>
원래 최대힙으로 정렬했을 때는 최대값을 찾을 때 **O(1)** 의 상수의 시간밖에 걸리지 않는다. 하지만 **최대힙에서 최소값을 찾을 땐** 얼마나 걸리게될까?  
최소값을 찾을 때, 리프노드가 있는 노드는 대상에서 제외하고 보는 것이 더 빠르다.   

**Leaf Node?** 자식노드가 없는 가장 하위의 노드를 의미한다
{: .notice--info}

때문에 배열을 절반으로 분리하여 모든 노드를 확인할 필요가 없게 한다. 그러나 확인할 대상이 줄어들어도 
시간복잡도는 여전히 O(n)의 복잡도를 갖는다고 한다(점근적 복잡도에 영향을 받지 않음!)
<br>

**Referenced by**    
<https://www.geeksforgeeks.org/minimum-element-in-a-max-heap/>
<br>
<br>
<br>
<br>
<br>



















