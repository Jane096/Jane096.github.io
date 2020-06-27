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

## Java로 Heap 구현하기 
```java
//최대힙 정렬

public static void main(String[] args) {
  int[] arr = {3, 5, 1, 7, 35, 24};
  heapSort(arr);
}

public static void heapSort(int[] a) {
 int max = a.length-1;
 
 //마지막 노드의 부모 노드에서 시작
 int lastNode = a.length;
 for(int i= lastNode / 2; i>=0; i--){
  int temp = a[0];
  a[0] = a[i];
  a[i] = temp; //swap
  lastNode -= 1;
  maxHeapify(a, 0);
 }
}

public static void maxHeapify(int[] a, int parent) {
  int length = a.length;
  int leftChild = i*2;
  int rightChild = i*2+1;
  int child = 0;
  
  if(leftChild > length || rightChild > length) {
    return; //자식노드가 없다면 종료
  }
  
  //왼쪽 오른쪽 중에서 더 큰 값을 n에 저장한다.
  if(a[leftChild] > arr[rightChild]) {
    child = leftChild;
  }else {
    child = rightChild;
  }
  
  //부모가 자식보다 큰 경우 종료
  if(a[parent] >= arr[child]) {
    return;
  }
  
  //swap
  int temp = a[parent];
  a[parent] = a[child];
  a[child] = temp; 
  
  //재귀적 호출로 더이상 바뀌지 않을 때까지 반복
  maxHeapify(arr, child);
}
```
<br>

**<참고>**   

>부모 노드 구하기: a[parent/2]    
>부모의 왼쪽 자식 노드 구하기: a[parent x2]   
>부모의 오른쪽 자식 노드 구하기: a[parent*2+1]   

<br>
<br>
<br>
<br>
<br>



















