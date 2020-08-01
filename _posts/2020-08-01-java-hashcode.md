---
title: "java에서 객체의 고유값을 나타내는 hashcode"
layout: single    
read_time: true    
comments: true   
categories: 
 - java 
description: hashcode와 equals가 같이 오버라이딩 되는 이유와 hashcode를 사용하지 않으면 어떻게 되는지에 대한 정리
last_modified_at: 2020-08-01   
--- 

자바의 가장 최상위 클래스인 **Object** 클래스에서 가장 많이 사용하는 메서드 중 하나는 
**toString()** 메서드 이다. **hashCode()** 도 많이 사용하긴 하지만 우리가 직접 구현을 할 필요는 없다. 
<br>

아래의 코드를 참고해보자
<br>

```java
public void equalMethod() {
  MemberDTO obj1 = new MemberDTO("Samgmin");
  MemberDTO obj2 = new MemberDTO("Samgmin");
  
  if(obj1.equals(obj2)) {
    System.out.println("obj1 and obj2 is same");
  }else {
    System.out.println("obj1 and obj2 is different");
  }
}
```
<br>

```
obj1 and obj2 is different
```
<br>

equals()로 비교했기 때문에 same이 나올줄 알았지만 다르다는 결과가 나왔다. 
MemberDTO 라는 클래스에는 해당 equals() 메서드가 오버라이딩 되어있지 않아 
위의 equals() 메서드에서는 hashCode() 값을 비교하게 되어있다. hashCode()는 해당 객체의 
주소값을 리턴하기 때문에 클래스의 값이 같더라도 해시코드가 다르니 당연히 different가 나오는 것이다. 
<br>
<br>

equals()로 비교를 했을 때 만약 어떤 두개의 객체가 서로 동일하다면, hashCode() 값도 동일하게 리턴이 되어야 한다. 
이렇기 때문에 IDE에서는 equals()와 hashCode()를 같이 override하게 하도록 하고 있다. 
<br>
<br>
<br>

### hashCode 구현 시 따라야하는 조건
- 어떤 객체에 대해서 이 메서드가 호출될 때에는 항상 동일한 int값을 리터해야한다. 그러나, 실행할 때마다 반드시 같은 필요는 없다
- 어떤 두개의 객체에 대하여 equals() 메서드를 사용하거 비교한 결과가 true라면, 두 객체의 hashcode도 동일한 int 값이어야 한다.
- 두 객체의 값이 equals로 false가 나왔다고 해서 hashcode도 무조건 달라야 할 필요는 없다. 하지만 이 경우 서로 다른 int 값을 제공하면 hashtable의 성능이 올라간다.
<br>

이런 많은 제약들 때문에 대부분 직접 구현하여 사용하지는 않는다. 개발툴에서 자동으로 두 개의 메서드를 동시에 override 하게 되어있으니 최대한 활용하는게 좋다. 
<br>
<br>
<br>

### hashCode()를 오버라이딩 하지 않는다면?

equals()를 사용할 때 hashCode()도 동시에 overriding을 시켜준다고 했었다. 
그러나 만약에 equals()만 override하고 hashCode()는 안한다면 어떻게 동작을 하게 될까?
<br>

만약 hashCode()를 오버라이드 하지 않는다면 default로 Object클래스가 implements하도록 되어있다. 
이 경우 equals()로 true가 나왔다고 할지라도 그 값이 같다고 그 객체의 주소값은 다르기 때문에 결과는 위에 처럼
different가 나오게 되는 것이다. 
<br>

hash기반의 자료구조인 HashSet, HashMap, HashTable은 hashcode를 이용하여 데이터를 저장하거나 참조하게 되어있다. 
만약 이 자료구조를 이용하는데 hashCode()와 equals()를 override하지 않는다면 제대로 작동을 할 수가 없다. 
<br>

추가로, 프로그래밍을 할 때 equals()로 false가 나온 경우, 꼭 hashcode를 부여하지는 않아도 된다. 하지만
hashcode로 특정한 숫자를 부여해 주는 것이 hashtable의 성능을 향상시킬 수 있다고 한다.
<br>
<br>
<br>


**Referenced by** <br>

<https://stackoverflow.com/questions/14608683/java-what-happens-if-hashcode-is-not-overriden> <br>
<https://www.geeksforgeeks.org/override-equalsobject-hashcode-method/>
<br>
<br>
<br>
<br>
<br>
<br>





