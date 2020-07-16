---
title: "기본 자료형의 형변환"
layout: single    
read_time: true    
comments: true   
categories: 
 -  java
toc: true    
toc_sticky: true    
toc_label: contents    
description: 형변환에 대한 이해 정리
last_modified_at: 2020-07-16       
---

**형변환(Type casting)** 이란 서로 다른 타입 사이에 변환하는 작업을 수행한다. 
형변환을 할 때는 기본자료형, 참조자료형 모두 괄호로 묶어주기만 하면 된다. 
<br>

여기서 boolean형은 형변환이 절대 불가능이다. boolean은 숫자로 변환할 수가 없기 때문이다.
또한 기본자료형 <-> 참조자료형 간의 형변환도 불가능하다. 
<br>
<br>
<br>

### 형변환을 사용하는 방법

밑의 2가지를 예시로 들어 정리했다.
- byte 타입 변수를 short타입 변수로 지정할 때
- short타입 변수를 byte타입 변수로 지정할 때

<br>

byte에서 short로 변환되는 것은 1바이트에서 2바이트 짜리의 타입으로 바이트 크기가 커지는 것이기 때문에 
형변환을 할 경우 따로 지정해 줄 것은 없다. 
그러나 그 반대라면 작은 바이트로 들어가기 때문에 확인이 필요하다.
<br>
<br>

```java
public class OperatorCasting() {
  public static void main(String[] args) {
    OperatorCasting operator = new OperatorCasting();
    operator.casting();
  }
  
  public void casting() {
    byte byteValue = 127; //byte 최대값을 할당
    short shortValue = byteValue; //범위가 큰 타입에 넣어주고 있어 명시하지 않아도 됨
    
    shortValue++; //byte최대값은 넘어가나 short는 괜찮음
    System.out.println(shortValue);
    byteValue = (byte) shortValue; //범위가 작은 타입이기 때문에 ()안에 명시해주어야함
    System.out.println(byteValue);
  }
}
```
<br>
<br>

```bash
출력결과

128
-128
```
<br>

첫번째는 너무나 당연한 결과이나 두번째는 -128이 나왔다. 여기서 헷갈릴 수 있을 것 같다. 
<br>
<br>

![short이미지]({{ site.url }}{{ site.baseurl }}/assets/connection/short.PNG){: .align-center}
<br>

128은 2의 7승에 해당한다. 비트로 표시하면 위에 처럼 나올 것이다. short에서 byte로 변환할 때는 short앞에 있는 
1바이트 (8비트)를 그냥 다 버려버리기 때문에 byte로 형변환 한다면 아래 처럼 나오게 된다
<br>

![byte 이미지]({{ site.url }}{{ site.baseurl }}/assets/connection/byte.PNG){: .align-center}
<br>

byte에서 첫번째로 시작하는 값이 1이라면 그것은 음수의 값을 의미하게 된다. 때문에 결과는 -128이 나오게 된다. 
<br>
<br>

![short이미지]({{ site.url }}{{ site.baseurl }}/assets/connection/short2.PNG){: .align-center}
<br>

short안에 2의 8승 자리에 1이 있는 상황이다. 여기서 byte로 형변환 하게 된다면 1바이트(8비트)는 역시 똑같이 버리게 된다. 
그러면 형변환 된 바이트 안에는 0만 남게 되어 출력은 0이 될것이다. 
<br>

```java
public class OperatorCasting() {
  public static void main(String[] args) {
    OperatorCasting operator = new OperatorCasting();
    operator.casting2();
  }
  
  public void casting2() {
    short shortValue = 256; 
    byteValue = (byte) shortValue;
    System.out.println(byteValue);
    
    shortValue = 255;
    byteValue = (byte) shortValue;
    System.out.println(shortValue);
  }
}
```
<br>

```bash
출력결과

0
-1
```
<br>

이렇게 0과 -1이라는 예상치 못한 결과가 나온다. 이렇게 범위가 작은 타입으로 형변환 할 때는 우리가 생각한 대로 
안나올 수도 있으니 주의가 필요하다. 
<br>
<br>
<br>
<br>
<br>
<br>













