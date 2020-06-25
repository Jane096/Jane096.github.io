---
title: "[자료구조] 프로그래머스 해시 - 전화번호목록(Java)"
layout: single    
read_time: true    
comments: true   
categories: 
 - algorithm 
 - Data structure
description: 프로그래머스 전화번호목록(level 2) 문제해결 하며 정리한 내용
last_modified_at: 2020-06-20   
---   

<br>  
프로그래머스에서 해시를 활용한 문제 '전화번호목록' 문제를 풀고 정리한 내용이다. 
해시에 포함되어 있었지만 간단하게 for문과 조건절만 이용해도 문제를 풀 수 있어 해시 알고리즘은 쓰지 않았다.  
<br>
[프로그래머스 해시 - 전화번호목록](https://programmers.co.kr/learn/courses/30/lessons/42577){: target="_blank"}   
<br>
<br>

문제의 내용은 한 번호가 다른 번호의 **접두어**인 경우가 있는지 확인하는 문제이다.  
<br>
나 같은 경우 startsWith() 대신 contains()를 써도 되지 않을까 착각을 했었는데, 명시되어 있는대로 
"접두어"에 유의해야 한다. contains는 앞에 오든 뒤에 오든 해당 글자나 숫자가 있는 경우 true를 반환하기 때문이다.    
<br>

입력받은 배열을 우선 오름차순으로 정렬을 한 후 0번째 index 값이 뒤에 있는 index의 시작부분과 일치하는지 
startsWith() method로 간단하게 알 수 있다. 만약 해당 숫자로 시작한다면 false를 반환하게 하고 아니라면 true를 반환하면 된다. 
<br>
<br>

```java
import java.util.*;

class Solution {
    public boolean solution(String[] phone_book) {
        boolean answer = true;
        Arrays.sort(phone_book);
        
        for(int i=1; i<=phone_book.length-1; i++) {
            if(phone_book[i].startsWith(phone_book[0])){
                return answer=false;
            }
        }
        return answer;
    }
}
```

<br>
<br>
<br>
<br>





