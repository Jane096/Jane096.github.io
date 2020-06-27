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
 int max = a.length;
 
 //build Heap 
 for(int i=max/2 - 1; i >= 0; i--) {
  maxheapify(a, max, i);
 }
 
 //루트 노드와 마지막 값을 swap해서 자리를 바꾸고
 //배열의 크기를 1 줄여 마지막은 힙의 일부가 아니도록 수정함
 for(int i=max - 1; i > 0; i--){
  int temp = a[0];
  a[0] = a[i];
  a[i] = temp; //swap
  lastNode -= 1;
  maxHeapify(a, 0);
 }
}

//이진트리로 구성하기 
public static void maxHeapify(int[] a, int parent, int i) {
  int parent = i;
  int leftChild = i*2+1;
  int rightChild = i*2+2;
  
  if(leftchild < max && a[parent] < a[leftchild]) {
    parent = leftChild; 
    
  }else if(rightChild < max && a[parent] < a[rightChild]) {
    parent = rightChild;
  }

  //swap
  if(i != parent) {
    int temp = a[parent];
    a[parent] = a[i];
    a[i] = temp;
    
    //재귀호출
    maxheapify(a, max, parent); 
  }
 }
  
```
<br>

**참고**   

>부모 노드 구하기: a[parent/2]    
>부모의 왼쪽 자식 노드 구하기: a[parent x2]   
>부모의 오른쪽 자식 노드 구하기: a[parent*2+1]
<br>

백준 알고리즘 2751번을 힙 정렬로 푼 분이 계셔서 참고했는데 left, right값에 +1씩 해줘 계산을 하는 걸 보았다. 음...무슨 말인지 확실하게 이해할 필요가 있을 것 같다. 

<br>
<br>
<br>
<br>
<br>



















