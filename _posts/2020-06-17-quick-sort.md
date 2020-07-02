---
title: "퀵 정렬 알고리즘(Quick sort)"
layout: single    
read_time: true    
comments: true   
categories: 
 - algorithm  
toc: true    
toc_sticky: true    
toc_label: contents    
description: 분할정복 알고리즘과 퀵 정렬 알고리즘(자바 base)
last_modified_at: 2020-06-17   
---   
<br>  
분할 정복 전략 중의 한가지 방법인 퀵 정렬(Quick Sort)에 대해 정리한 포스팅이다. 선택 정렬에 비해서 훨씬 빠르고 실제로 많이 사용되는 방법이다.  
버블 정렬이나 삽입 정렬보다도 더 빠른 성능을 보이며 시간 복잡도는 O(n) 이거나 O(n log n)이 나오게 된다.
(이미 정렬되어 있거나 pivot의 선택에 따라 최악의 경우 O(n2)도 나올 수 있다는 것 주의...)   

<br>
시간 복잡도 때문에 pivot을 아예 첫번째 원소로만 고르는 경우도 있다고 한다.   

<br>  
<br>

## 동작 방법
퀵 정렬은 기본적으로 **분할(partitioning) 정복 전략**을 사용한다. 즉, 기본단계가 될 때까지 나눠야 한다는 의미이다.   
우선 주어진 배열에서 임의의 값 pivot(원소)를 선택한다.   
<br>
```python
[3,5,4,1,6,2] #pivot을 4로 지정
```
<br>
기준 값을 임의의 숫자로 정해준다. 
<br>
```python
[1,3,2] < 4 < [6,5] 
```
<br>
그리고 나서 pivot을 기준으로 크다 작다를 비교해 pivot보다 작은 부분과 큰 부분으로 분할을 한다.
<br>
```python
[1] < 2 < [3]  // pivot을 2로 지정
```
<br>  
왼쪽으로 분할 된 배열 안에서 또다시 pivot을 선정해 똑같이 크다 작다를 비교하여 하위 배열을 정렬해준다.  
이런 수행을 할 때 재귀적 호출을 한다고 말할 수 있겠다.   
<br>
재귀 알고리즘을 사용할 때에는 실행 종료를 보장하는지 확인을 해야한다. 
<br>
<br>

_재귀호출 할 때 무한루프에 빠지지 않도록 주의하자_
<br>
<br>
이렇게 분할 된 부분에 대하여 퀵 정렬을 호출하여 모두 수행이 완료 되었다면 다시 합쳐주면 된다!
<br>
<br>

![퀵정렬 동작순서]({{ site.url }}{{ site.baseurl }}/assets/image/quicksort.PNG){: .align-center}
<br>
그림으로 표현했을 때 이렇게 볼 수 있을 것 같다. 
<br>
<br>

자바로 구현을 해본다면...
<br>
```java
  public void quickSort(int[] arr, s, e) {
    int start = s; //index 0을 보내야함
    int end = e; //arr.length - 1로 가장 큰 index값 
    int pivot = arr[(start+end)/2];
    
    while(start <= end) { //start는 계속 증가시키고 end는 계속 감소시켜 서로 교차하는 지점까지 반복
       while(arr[start] < pivot) start++; //start가 가리키는 값과 pivot 값을 비교해서 더 작은 경우 시작 인덱스 값을 증가시켜 큰값이 좌측에 있는 것을 찾는다.
       while(arr[end] > pivot) end--; //end값과 pivot값을 비교해 end가 더 크다면 인덱스 값을 감소 시켜 작은값이 오른쪽에 있는 것을 찾아낸다.  
       if(start <= end) { //swap기능으로, 아직 교차지점에 오지 않았다면, start인덱스와 end인덱스를 상호 교대 시켜준다(잘못된 위치의 있는 두 값을 고치기 위해서)
         int temp = arr[start];
         arr[start] = arr[end];
         arr[end] = temp;
         start++; //swap후에 각자의 다음 값을 가리키기 위해 start는 증가, end는 위에와 같이 감소시켜 준다.  
         end--;
       }
    }
    if(s < end) quickSort(arr, s, end); //start와 end의 교차 지점에 와서 while을 빠져 나왔다면 다시 재귀호출해 정렬이 완료될 때까지 돈다.   
    if(e > start) quickSort(arr, start, e);
  }
```
<br>
코드로 직접 써보니 start와 end값을 swap해주고 다시 재귀호출하는 로직이 조금 어려운 것 같다.   
시간을 들여 찬찬히 이해해 보도록 해야겠다!  
<br>
<br>
<br>
<br>









