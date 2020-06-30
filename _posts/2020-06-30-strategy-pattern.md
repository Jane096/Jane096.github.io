---
title: "[디자인패턴] 스트래티지 패턴과 예시"
layout: single    
read_time: true    
comments: true   
categories: 
 - Design pattern  
toc: true    
toc_sticky: true    
toc_label: contents    
description: 의존성 주입에 많이 사용하는 패턴인 스트래티지 패턴에 대한 정리
last_modified_at: 2020-06-29   
---   

**스트래티지 패턴** 이란 지정된 알고리즘의 세부 구현을 변경할 필요 없이 쉽게 교환할 수 있게 
해주는 디자인 패턴이다. 행위를 클래스로 캡슐화 하여 동적으로 행위를 자유롭게 바꿀 수 있게 해준다. 
<br>
<br>

대표적으로 스프링 프레임워크의 **의존성 주입(dependency Injection)** 이 이 스트래티지 패턴을 
따라가는 개념 중의 하나이다. 의존성 주입은 테스트 코드에서 구현된 알고리즘을 교환하거나 목(Mock) 
구현을 가능하게 해준다. 
<br>
<br>

## 스트래티지 패턴으로 구현한 로거

```java
public interface Logging {
  void write(String message);
}

public class ConsoleLogging implements Logging {
  @Override
  public void write(final String message) {
    System.out.println(message);
  }
}

public class FileLogging implements Logging {
  private final File toWrite;
  
  public FileLogging(final File toWrite) {
    this.toWrite = toWrite;
  }
  
  @Override
  public void write(final String message) {
    try{
      final FileWriter fos = new FileWriter(toWrite);
      fos.writer(message);
      fos.close();
    }catch(IOException e) {
      //예외처리 구문 입력하기
    }
  }
}
```
<br>

위의 코드는 스트래티지 패턴을 이용해 Logger를 만들어본 코드이다. 
Logging 인터페이스를 사용하면 콘솔에 출력하는지 파일에 기록하는지 신경쓰지 않고 
개발할 수 있게 해주는 기능이다.
<br>

특정 구현을 활용하지 않고 테스트 할 때는 consoleLogging을, 실제로 사용할 땐 fileLogging클래스를 
사용하도록 작성되어있다. 
<br>

이후에 해당 인터페이스에 추가 구현이 있을 경우 작성한 코드는 수정하지 않고 새로운 구현을 활용할 수 있는 
토대가 된다. 
<br>
<br>
<br>

## Client 코드에서 Logging interface 사용 예

```java
public class Client {
  private final Logging logging;
  
  public Client(Logging logging) {
    this.logging = logging;
  }
  
  public void doWork(final int count) {
    if(count % 2 == 0) {
      logging.write("Even number: "+count);
    }
  }
}

public class ClientTest {
  //Console Logging이용
  @Test
  public void useConsoleLogging() {
    final Client c = new Client(new ConsoleLogging());
    c.doWork(32);
  }
  
  //FileLogging 이용
  @Test
  public void useFileLogging() throws IOException {
    final File tempFile = File.createTempFile("test", "log");
    final Client c = new Client(new FileLogging(tempFile));
    c.doWork(41);
    c.doWork(42);
    c.doWork(43);
    
    final BufferedReader reader = new BufferedReader(new FileReader(tempFile));
    assertEquals("Even number: 42", reader.readLine());
    assertEquals(-1, reader.read());
  }
  
  //Mockito(Mock) 이용
  @Test
  public void useMockLogging() {
    final Logging mockLogging = mock(Logging.class);
    final Client c = new Client(mockLogging);
    c.doWork(1);
    c.doWork(2);
    
    verify(mockLogging).write("Even number:2");
  }
}
```
<br>

스트래티지 패턴의 장점은 사용했을 때 실행하기 전까지 어떤 구현을 사용할지 결정을 미룰 수 있다는 점이다. 
로깅 말고도 가장 좋은 예시가 바로 스프링 프레임워크이다. 객체나 의존성을 생성할 때 XML(또는 Java configure)fmf 
사용하며, 웹 어플리케이션을 실행할 때 먼저 읽어들인다. 
<br>

사용자는 파일의 내용을 변경하면 다시 컴파일할 필요없이 다른 구현 방법을 이용할 수 있다. 
<br>

**자바 어플리케이션에서 로깅** 위의 코드 2가지는 스트래티지 패턴을 사용하는 기본적인 방법의 예시이다. 
실제로 로그는 Log4J, SLF4J 같은 로그 출력 전용 라이브러리를 사용하지 위의 패턴으로 로그를 남기지는 않는다!
{: .notice--info}
<br>
<br>
<br>
<br>




























