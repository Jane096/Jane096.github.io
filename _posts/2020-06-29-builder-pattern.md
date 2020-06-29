---
title: "[디자인패턴] Builder pattern은 얼마나 유용할까"
layout: single    
read_time: true    
comments: true   
categories: 
 - Design pattern  
toc: true    
toc_sticky: true    
toc_label: contents    
description: 필드가 많은 객체를 생성할 때 유용하게 쓰이는 빌더 패턴에 대한 정리  
last_modified_at: 2020-06-29   
---   

밑에 코드블럭 처럼 필드가 많은 객체를 생성할 경우, 생성자를 이용하면 다루기 어렵고 헷갈릴 
여지가 있다. 
<br>
<br>
<br>

## 빌더 패턴이란

```java
public class Pet {
  private final Animal animal; //필수
  private final String petName; //필수
  private final String ownerName; //필수
  private final String address; //필수
  private final String telephone; //필수
  private final String dateOfBirth; //선택적 정보
  private final String emailAddress; //선택적 정보
}
```
<br>

유효한 Pet객체를 위해서는 필수적 필드와 조합할 수 있는 선택적 필드를 포함하도록 되도록 많은 매개변수를 
갖는 생성자가 최소 4개 이상 이어야 한다. 더 많은 선택적 필드를 추가한다면 
그만큼 관리도 어려워지고 코드의 이해도도 떨어지게 된다. 
<br>

이러한 문제를 해결하는 방법은 불필요하게 생성자를 사용하지 않고, 필드의 final 키워드를 제거해 
Pet객체에 setter 메서드를 이용하는 것이다. 
<br>

```java
final Pet p = new Pet();
p.setEmailAddress("owners@address.com");
```
<br>

단, 조건이 붙게 되는데 필수적 필드는 모두 채워져 있어야 한다는 것이다. 또한 유효하지 않은 
Pet 객체를 생성할 수 있다는 것에 주의해야 한다. 
<br>

이러한 상황일 때 Builder()라는 도메인에 적합한 객체를 생성하는 **빌더 패턴**을 사용하면 도움이 된다. 
빌더 패턴이란 복합 객체의 생성 과정에서 표현 방법을 분리하여 동일한 생성 절차에서 서로 다른 
표현 결과를 만들 수 있게 하는 패턴이다.
<br>
<br>
<br>

## 빌더 패턴으로 객체 생성하기

```java
//빌더 패턴을 통해 객체 생성하는 방법

@Test
public void legalBuild() {
  final Pet.Builder builder = new Pet.Builder();
  final Pet pet = builder
    .withAnimal(Animal.PUPPY)
    .withpetName("sam")
    .withOwnerName("jane")
    .withAddress("seoul")
    .withTelephone("010")
    .withEmailAddress("abc@abc.com")
    .build(); //예외발생 없이 잘 동작함
}

@Test(expected = IllegalStateException.class)
public void illegalBuild() {
  final Pet.Builder builder = new Pet.Builder();
  final Pet pet = builder
    .withAnimal(Animal.PUPPY)
    .withpetName("sam")
    .withOwnerName("jane")
    .build(); //예외발생
}
```
<br>
<br>
<br>

## Builder 클래스

Builder 클래스는 Pet 클래스의 일부이며 Pet 객체를 생성하는 권한이 있다. 
생성자를 사용할 때는 매개변수를 어떤 걸 사용할지를 매개변수의 순서가 결정을 했다. 
그러나, 각 매개변수에 명시적으로 메서드가 존재한다면 값을 어떻게 사용할지 이해하기 쉽고 
**순서에 구애받지 않고** 마음대로 호출이 가능하다. 
<br>
<br>

즉, Pet객체의 실제 생성자를 호출해 Pet객체를 반환한다. 생성자는 이제 final 키워드를 뺀 
private로 선언이 가능하다. 만약 호출해야하는 builder 클래스의 메서드들을 더 간결하게 
정의할 수 있도록 **builder.withAA().withBB()** 해서 선언할 수 있다. 
<br>
<br>

```java
//빌더 클래스 구현 
public class Pet {
  public static class Builder {
    private final Animal animal; 
    private final String petName; 
    private final String ownerName; 
    private final String address; 
    private final String telephone; 
    private final Date dateOfBirth; 
    private final String emailAddress; 
    
    public Builder withAnimal(final Animal animal) {
      this.animal = animal;
      return this;
    }
     
    public Builder withPetName(final String petName) {
      this.petName = petName;
      return this;
    }
    
    public Builder withAnimal(final String ownerName) {
      this.ownerName = ownerName;
      return this;
    }
    
    public Builder withAnimal(final String address) {
      this.address = address;
      return this;
    }
    
    public Builder withAnimal(final String telephone) {
      this.telephone = telephone;
      return this;
    }
    
    public Builder withAnimal(final Date dateOfBirth) {
      this.dateOfBirth = dateOfBirth;
      return this;
    }
    
    public Builder withAnimal(final String emailAddress) {
      this.emailAddress = emailAddress;
      return this;
    }
    
    //명시적 메서드의 역할 
    public Pet build() {
      if(animal == null ||
        petName == null ||
        ownerName == null ||
        address == null ||
        telephone == null) {
        throw new IllegalStateException("Cannot create Pet Object");
      }
      
      return new Pet(animal,petName,ownerName,address,telephone,dateOfBirth,emailAddress);
    }
  }
  
    private final Animal animal;
    private final String petName;
    private final String ownerName;
    private final String address;
    private final String telephone;
    private final String dateOfBirth;
    private final String emailAddress;
    
    //생성자(private로 선언)
    private Pet(final Animal animal,
      final String petName,
      final String ownerName;
      final String address,
      final String telephone,
      final Date dateOfBirth,
      final String emailAddress) {
     this.animal = animal;
     this.petName = petName;
     this.ownerName = ownerName;
     this.address = address;
     this.telephone = telephone;
     this.dateOfBirth = dateOfBirth;
     this.emailAddress = emailAddress
   }
}
```
<br>
<br>
<br>
<br>
<br>
<br>



























