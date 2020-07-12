---
title: "자바 원시타입의 이름을 지정하고 JVM에서 처리되는 과정 "
layout: single    
read_time: true    
comments: true   
categories: 
 -  java
toc: true    
toc_sticky: true    
toc_label: contents    
description: 원시타입의 개념 정리
last_modified_at: 2020-07-13       
---

**Primitive type(원시타입)** 이란, boolean, int, double, float 같은 각각의 기본타입을 말한다. 
흔히 "객체" 라고 알려진 참조 타입과는 다른 방식으로 JVM에서 동작을 하게 된다. 
<br>
<br>
<br>

원시타입의 가장 큰 차이점은 항상 값이 있어야 하며 **null**이 될 수 없다.
<br>
<br>

int와 long 타입의 변수를 정의하면 컴파일러는 두 타입을 구분지을 수 있어야한다. 
주로 값 뒤에 L이 오면 long타입으로, 없다면 자동으로int타입이라고 인식하게 된다. 

**notice** L은 대소문자를 지킬 필요는 없지만 헷갈릴 여지를 없애기 위해 주로 대문자를 사용한다고 한다.
{: .notice--info}

<br>

float와 double의 관계에서도 마찬가지이다. (float는 F, double은 D로 적용 가능하나, D를 빼먹는다면 자동으로 double로 인식한다.)
<br>
<br>

### 원시타입의 종류와 크기

> boolean : 1    
> short : 16   
> int : 32    
> long : 64     
> float : 32     
> double: 64     
> char : 16     


**char 타입**은 unsigned라는 점을 기억해야한다. char값은 유니코드 값을 기반으로 동작하기 때문에 
0 ~ 65.535까지 넣을 수 있다. 
<br>
<br>
<br>

원시타입에서 값을 정의하지 않는다면 기본값으로 지정이 되는데, boolean은 false로, 다른 타입은 0으로 표시를 하게 된다.
(int는 0, float는 0.0으로 표현됨)
<br>
<br>

### 상위 개념으로 암시적으로 타입캐스팅하기

```java
int value = Integer.MAX_VALUE;
long biggerValue = value + 1;
```
<br>

char 타입을 제외하면 컴파일러는 해당 값을 저장하기 위해 상위 타입을 자동으로 사용할 수 있다. 
상위 타입으로 사용하더라고 정확도를 잃지 않기 때문에 가능하다고 한다. 
<br>

예를 들어, int타입에서 long을 사용하거나 float타입에서 double을 사용할 수 있다는 의미이다. 
그러나 그 반대는 성립되지 않는다. 만약 그래야 한다면, 해당 값의 타입을 () 안에 명시를 해주어야 한다. 
<br>
<br>
<br>

### 하위 타입으로 명시적 타입캐스팅

```java
long num = Long.MAX_VALUE;
int LongToInt = (int) num;
```
<br>

그러나 하위 타입으로 자주 변환한다는 것은 적절한 타입을 사용하지 않는다는 것을 잘 기억해두자
<br>
<br>
<br>
<br>
<br>
<br>

















