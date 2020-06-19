---
title: "Codility MaxProductOfThree 자바"
layout: single    
read_time: true    
comments: true   
categories: 
 - algorithm  
description: codility 해당 문제에서 경우의 수를 잘못 고려한 점 정리
last_modified_at: 2020-06-19   
---   

<br>  

[codility Sorting문제-MaxProductOfThree](https://app.codility.com/programmers/lessons/6-sorting/){: target="_blank"}

>  A[0] = -3  
>  A[1] = 1  
>  A[2] = 2  
>  A[3] = -2  
>  A[4] = 5  
>  A[5] = 6  
><br>
>contains the following example triplets:  
><br>
><br>
>(0, 1, 2), product is −3 * 1 * 2 = −6    
>(1, 2, 4), product is 1 * 2 * 5 = 10    
>(2, 4, 5), product is 2 * 5 * 6 = 60    
><br>
>Your goal is to find the maximal product of any triplet.   

<br>
<br>
해당 문제에서는 주어진 배열에서 임의의 3개의 숫자를 골라 가장 큰 수를 만들어 내야 한다.   
<br>
Arrays.sort를 사용해 오름차순 정렬 후 오른쪽에서 3개를 고르면 될 것 같지만 0이 있거나 3개 중 하나가 음수라면 더 작은 수가 나올 수 있기 때문에 경우의 수를 잘 생각을 해야한다.   
<br>
<br>
예를 들자면, {-100, -10, -1, 5, 7}이 주어졌을 때 -1x5x7 = -35이지만 다른 경우로 보면 -100x-10x7 = 7000 이기 때문에 주의해야한다.   
<br>
<br>
또한 고른 숫자 들 중에서 0이 포함되어있다면 값이 무조건 0이기 때문에 0은 제외를 시켜야 한다.    
숫자의 시작이 0이거나 모든 숫자가 1이라면 0을 리턴하게 한다.   
<br>
<br>
```java

import java.util.*;

class Solution {
    public int solution(int[] A) {
        Arrays.sort(A);
        int index = A.length-1;
        
        if(A[index] < 0) {
            return A[index] * A[index-1] * A[index-2];
        }else if(A[index-1] < 0 || A[index-2] < 0) {
            return A[index] * A[0] * A[1];
        }
        int answer1 = A[index] * A[index-1] * A[index-2];
        int answer2 = A[index] * A[0] * A[1];
        
        return answer1 < answer2 ? answer2 : answer1;
    }
}
```

<br>

경우의 수에 대해 충분히 생각을 못해 처음에 절반 정도 밖에 못맞추고 다른 사람들의 코드를 봤는데   
<br>

제일 큰 수가 음수일 경우엔 오른쪽에서 3개를 골라 곱하면 되고 오른쪽 값 3개 중 하나라도 음수라면 왼쪽에 2개의 값을 가져와 곱한다.   
<br>

두개 중 어느 것도 속하지 않는다면 오른쪽 값 3개를 곱한 값과 왼쪽 2개 값을 가져와 곱한 것을 비교해 제일 큰 쪽으로 리턴한다.    
<br>
<br>
<br>
<br>






