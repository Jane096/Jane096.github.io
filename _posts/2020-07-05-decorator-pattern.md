---
title: "[디자인패턴] 테코레이터 패턴과 예시 "
layout: single    
read_time: true    
comments: true   
categories: 
 - Design pattern  
toc: true    
toc_sticky: true    
toc_label: contents    
description: 객체에 동적으로 새로운 책임을 추가하는 테코레이터 패턴에 대한 정리
last_modified_at: 2020-07-05       
---

**데코레이터 패턴(Decorator Pattern)** 이란, 특정 객체의 기능을 설정하거나 변경할 수 있게 해주는 
디자인 패턴이다. 실생활로 예시를 들어보자면 버튼을 추가하거나 스크롤 바에 기능을 추가하거나 같은 
재료로 만드는 샌드위치를 두 명의 고객에게 요청받았을 때 어떻게 처리할지를 들 수 있다. 
<br>
<br>

대표적으로 InputStream, OutputStream이 데이터를 읽고 저장하는데 데코레이터 패턴을 사용하는 클래스들 이다. 
외부 소스를 읽고 다시 저장할 때 효율적으로 사용하기 위함이다. 
<br>
<br>
<br>

## OutputStream의 write()

<br>
```java
public abstract void write(int b) throws IOException;
```
<br>

데코레이터 패턴 구현의 핵심이 되는 OutputStream 클래스의 정의를 보자.
바이트를 어떻게 저장할지를 정하는 write 메서드를 볼 수 있다. 
<br>

대량으로 바이트를 저장할 때 다른 메서드를 사용할 수 있지만, write메서드는 각 바이트에 대응하던 
이전 메서드를 우선적으로 호출한다. 하지만 구현방법에 따라서 각 바이트에 대응하던 메서드들을 
오버라이드 하는 것이 더 효율적인 경우가 있다. 
<br>

이런 경우 OutputStream 클래스에서 구현한 메서드들은 필요 동작을 수행 후, 다른 OutputStream클래스에 
수행 권한을 위임한다. 이 때 또다른 OutputStream 객체를 생성자 매개변수로 받게 되는 것이다. 
<br>
<br>

반면 FileOutputStream, SocketOutputStream의 실제 데이터를 저장하는 OutputStream이 write메서드의 수행 
권한을 다른 클래스에 위임하지 않는다. 
<br>
<br>
<br>

## 디스크에 바이트 배열을 저장하는 데커레이터 패턴 

```java
public void decoratorPattern() throws IOException {
  File file = new File("target", "out.bin");
  FileOutputStream fs = new FileOutputStream(file);
  BufferedOutputStream bs = new BufferedOutputStream(fs);
  ObjectOutputStream os = new ObjectOutputStream(fs);
  
  os.writeBoolean(true);
  os.writeInt(42);
  os.writeObject(new ArrayList<Integer>());
  
  os.flush();
  os.close();
  bs.close();
  fs.close();
}
```
<br>

File 선언부터 ObjectOutputStream 까지는 저장하려는 파일 이름과 파일을 어떻게 저장할지 정의한 것이다. 
BufferedOutputStream이 파일을 저장하는데 필요한 호출들을 캐시하고 한번에 여러바이트씩 저장을 한다. 
이렇게 해야 파일을 저장할 때 효율이 좋아진다. 
<br>

이 상황에서 OutputStream은 파일이 어디에 기록되는지 알 수 없다. 다른 OutputStream 클래스에 간단히 저장 권한을 위임할 뿐이다. 
이러한 권한 위임이 데코레이터 패턴의 가장 큰 장점이다. 새로운 OutputStream을 클래스 구현을 제공하더라도 기존 코드와 함께 
정상적으로 작동되기 때문이다. 
<br>
<br>

## 그 외

만약에 BufferedOutputStream을 호출할 때마다 콘솔에 로그를 찍어보고 싶다면 단순이 로그를 작성하는 부분을 
구현해 체인의 앞부분에 적용만 한다면 쉽게 구현이 가능하다. 
<br>

대신 OutputStream은 체인을 할 때 정의하는 순서가 매우 중요하므로 기억하고 있자
{: .notice--info}
<br>
<br>
<br>
<br>
<br>
<br>

























