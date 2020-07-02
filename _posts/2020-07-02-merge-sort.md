---
title: "[자료구조] 병합정렬(Merge Sort) "
layout: single    
read_time: true    
comments: true   
categories: 
 - Data Structure  
toc: true    
toc_sticky: true    
toc_label: contents    
description: 정렬에서 좋은 시간복잡도를 가진 병합 정렬에 대해 정리한 글
last_modified_at: 2020-07-02       
---

O(N ^ 2)의 시간복잡도를 가지는 선택정렬, 버블정렬, 삽입정렬과 평균 시간복잡도 O(N * logN)인 퀵 정렬 이외에 
**병합 정렬(Merge Sort)** 이라는 기능이 있다. 
<br>

## 병합 정렬

병합정렬은 배열을 두 개의 작은 문제로 분할한 뒤 각자 정렬을 수행 후 나중에 합치는 방식을 채택한다. 
즉, **일단 정확히 반을 나누고 나중에 정렬** 하는 것이다.
<br>  

퀵 정렬과 동일하게 **분할정복** 방식을 그대로 사용하는 알고리즘이다. 그런 이유로 퀵 정렬과 동일하게 O(N * logN)의 시간복잡도를 
가지게 된다. 
<br>

퀵 정렬과 병합 정렬의 차이점은 퀵 정렬의 경우 pivot값에 따라 편향되게 나눠질 수 있고 정해진 pivot값에 따라 최악의 상황에서 O(N ^ 2)가 
걸릴 수 있다는 단점이 있다. 

그러나 병합정렬의 경우에는 항상 middle값으로 분할하기 때문에 퀵 정렬보단 느리지만 **최악의 경우에도 O(N * logN)을 보장** 한다는 
큰 장점이 존재한다. 
<br>
<br>

### 병합 정렬의 분할 방식
<br>

![병합정렬 분할]({{ site.url }}{{ site.baseurl }}/assets/image/mergesort.PNG){: .align-center}
<br>

첫번째 시작 부분을 본다면 들어온 배열이 모두 1인 배열 상태로 시작을 하게 된다.   
그 다음 1단계에서 1개였던 배열을 두개씩 묶어서 합치게 된다.    
2단계 에서는 크기 2의 배열을 합쳐 크기 4의 배열로 만든다.    
<br>
<br>
<br>

### 병합정렬 시간복잡도
<br>

![병합정렬 시간복잡도]({{ site.url }}{{ site.baseurl }}/assets/image/mergesort4.PNG){: .align-center}

**합치는 순간에 정렬을 수행** 하는 것을 확인할 수 있다. 합치는 단계는 3단계에서만 진행한다. 
합치는 갯수가 2배씩 증가하기에 2 ^ 3 = 8로 총 3개 단계만 필요하게 되는 것이다. 
데이터의 갯수가 N개 일 때, logN을 유지하게 되고 정렬 자체에 N만큼의 수행시간만이 필요(데이터 갯수만큼만 연산되기때문)하기 때문에 
결과적으로 O(N * logN) 인 것이다.
<br>
<br>
<br>

### 병합 정렬 동작 방법
<br>

![병합정렬 동작1]({{ site.url }}{{ site.baseurl }}/assets/image/mergesort2.PNG){: .align-center}
<br>

초기 상태의 모습이다. 왼쪽 배열에서 i가 첫번째 원소를, 오른쪽 배열에서는 j가 두번째 원소를 가리킨다.    
그리고 임시배열은 비어있는 상태이다. **부분 집합은 이미 정렬 되어 있는 상태** 라고 가정하고 합치기 때문에 
이미 정렬되어 있는 것 두개를 합치는 것은 시간복잡도 **O(N)** 이면 충분하다. 
<br>
먼저 5와 6을 비교해 작은 수 5를 먼저 배열에 입력한다. 처리한 인덱스를 1씩 더해 k와 j가 한칸 이동하도록 한다.  
<br>
<br>

![병합정렬 동작2]({{ site.url }}{{ site.baseurl }}/assets/image/mergesort3.PNG){: .align-center}
<br>
  
i는 아직 6을 가리키고 j는 다음 값인 8을 가리키게 된다. 6이 작기 때문에 2번째 칸에는 6을 입력한다.     
처리 후, i와 k가 한칸 이동을 해 일련의 과정을 반복한다.   
이런식으로 동작하기 때문에 정확히 **N번**만 처리하면 되는 것이다. 
<br>
<br>
<br>

## 소스코드(자바)

```java

import java.util.Arrays;
import java.util.Scanner;

class Test {
	 public static void merge(int[] a, int m, int middle, int length, int n) {
		 int sorted[] = new int[n]; //정렬은 반드시 전역변수
         int start = m;
         int end = middle + 1;
         int k = m;
        
         //작은 순서대로 배열에 입력하는 기능, start와 end를 비교해 작은 값을 k에 입력 
         //start도 한칸씩, end도 한칸씩 이동
         while(start <= middle && end <= length) {
             if(a[start] <= a[end]) {
            	 sorted[k] = a[start];
            	 start++;
             }else { //start보다 end가 작다면 end를 넣어주고 인덱스 한칸 이동
            	 sorted[k] = a[end];
            	 end++;
             }
             k++; //다음 빈칸 채우기 위해 한칸이동
         }
         
         //end가 아직 끝까지 돌지 못했는데 start가 끝난 경우가 있을 수 있어 나머지 값을 넣어줘야함
         if(start > middle) { //start가 먼저 끝나버렸음
        	 System.out.println("start:" + start); //2
        	 System.out.println("middle: " + middle);
        	 for(int t=end; t<=length; t++) {
        		 sorted[k] = a[t];
        		 k++;
        	 }
         }else {
        	 for(int t=start; t<=middle; t++) {
            	 sorted[k] = a[t];
            	 k++;
             }
         }
         
         //정렬된 배열을 실제 배열로 삽입함(m=0, 즉 처음부터 끝까지)
         for(int t=m; t<=length; t++) {
        	 a[t] = sorted[t];
         }       
	 }
   
   //두가지로 나뉜다는 점에서 재귀함수로 구현하는 것이 가장 좋음 
	 //크기가 1이상이라는 것만 다룰 수 있도록 m은 length보다 작게 조건문 생성해 분할
	 public static void mergeSort(int[] a, int m, int length, int n) {
		 if(m < length) {
			 int middle = (m+length) / 2;
			 mergeSort(a, m, middle, n); //중간 이전 배열 병합정렬
			 mergeSort(a, middle + 1, length, n); //중간 이후 병합정렬 수행
			 merge(a, m, middle, length, n); //위의 2개 배열을 나중에 합쳐줌
		 }
	 }
     
	 public static void main(String[] args) {
		 Scanner sc = new Scanner(System.in);
		 int num = sc.nextInt();
     
		 int[] arr = new int[num];
     
		 for(int i=0; i<arr.length; i++) {
			 arr[i] = sc.nextInt();
		 }
     
		 mergeSort(arr, 0, arr.length-1, num);    
		 System.out.println(Arrays.toString(arr));
 	}
}

```
<br>
<br>
<br>
<br>
<br>





















