---
title: "[자료구조] 자바 해시를 이용한 프로그래머스 위장(level 2)"
layout: single    
read_time: true    
comments: true   
categories: 
 - algorithm 
 - Data structure
description: 프로그래머스 위장 문제에서 헷갈렸던 점 정리
last_modified_at: 2020-06-19   
---   

<br>
프로그래머스에서 해시를 응용한 문제 '위장'을 풀면서 이해하기 어려웠던 내용을 정리해보았다.   
<br>
<br>
[Programmers 위장 문제](https://programmers.co.kr/learn/courses/30/lessons/42578#){: target="_blank"}
<br>
<br>

문제의 조건은 스파이가 가진 의상에서 상의 하의가 겹치지 않게 입어야 하고 같은 옷을 다음날 또 입을 수 없다.   
그리고 스파이는 옷을 무조건 1개 이상 입어야 한다.(즉, 아무것도 안 입는 경우는 없다)   
<br>
<br>

>[["yellow_hat", "headgear"], ["blue_sunglasses", "eyewear"], ["green_turban, headgear"]]	
><br>
>return: 5
><br>
><br>
>[["crow_mask", "face"], ["blue_sunglasses", "face"], ["smoky_makeup", "face"]]	
><br>
>return: 3

<br>
HashMap을 써서 상의나 하의(headgear, face)의 정보를 key값으로 넣고 getOrDefault를 이용해 key값이 중복이 된다면 
value 값에 2를 입력하고 중복 된 key가 없다면 default 값 0에서 +1해서 해당 key값에 value를 1로 설정해 
해당 key가 몇개 들어 있는지 확인을 한다.  
<br>
<br>

```java
import java.util.*;

class Solution {
    public int solution(String[][] clothes) {
        int answer = 1;
        HashMap<String, Integer> map = new HashMap<>();
        
        for(int i=0; i<clothes.length; i++) {
            map.put(clothes[i][1], map.getOrDefault(clothes[i][1], 0) + 1);   
        }
        for(String key : map.keySet()) {
            answer *= (map.get(key) + 1);
        }
        return answer-1;
    }
}
```  

<br>

HashMap은 같은 key가 있다면 가장 최근에 들어온 value로 덮어쓰기를 한다.  
<br>
<br>
getOrDefault를 이용해서 중복 된 키에 대해 값을 조정하도록 한다. 
<br>
<br>
예를 들어 첫 배열의 headgear는 중복된 값이 없기 때문에 [headgear, 1]로 들어가지만 다시 들어온 배열에
headgear가 있다면 현재 headgear의 value에서 1을 더해 2로 덮어씌워지게 된다([headgear, 2] <- 이렇게])  
<br>
<br>
문제에는 추가적인 headgear가 없지만 혹시나 또 다른 배열에 headgear가 포함되어 있다면 
getOrDefault가 해당 키의 value(2)에 +1을 해서 최종 ["headgear", 3]으로 덮어씌워진다.   
<br>
<br>

두번째 for문에서 key값만을 가져와서 해당 key의 value를 가져와 +1을 한다.(headgear가 3이 된다)     
answer값에 곱하기를 누적하면 된다. eyewear는 테스트케이스에서 1개만 있기 때문에 ["eyewear", 1]이고 
answer에 3 x 2 = 6이 들어가게 된다.   
<br>
<br>

아무것도 입지 않는다는 조건을 빼야 하기 때문에 최종 answer에서는 -1을 해주어 5를 리턴하도록 한다.   
<br>
<br>

다른 분들의 코드를 참조한 것이지만 getOrDefault라는 기능을 많이 사용한 것을 볼 수 있었고, 이해하는 것이 관건이었다.    .   
<br>
<br>
<br>
<br>








