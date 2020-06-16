---
title: "log4jdbc-log4j2 설정 시 발생됐던 오류와 해결"
layout: single    
read_time: true    
comments: true   
categories: 
 - spring  
toc: true    
toc_sticky: true    
toc_label: contents    
description: hikari 커넥션 풀과 log4jdbc 설정할 때 발생했던 오류와 해당 문제점 해결방법  
last_modified_at: 2020-06-16     
---
<br>
Spring 커넥션 풀을 설정할 때 자주 사용하는 Hikari CP와 로그 확인용인 log4jdbc 라이브러리 구축할 때 책이나 구글에서도 흔하게 발생하지 않는(?) 에러가 나에게 발견되었어서...   
한번 정리해본다.      
<br>
<br>
Hikari CP를 빌드 하고 난 뒤 PreparedStatement의 ?값에 대한 로그를 제대로 보기 위해 
log4jdbc 라이브러리를 설치 후 기존 RootConfig의 코드를 아래와 같이 변경을 하게 될 것이다.      
<br><br> 

**Please Note:** oracle database 11g xe버전 기준
{: .notice--info}
<br>
```java
@Bean
public DataSource dataSource() {
  HikariConfig hikariConfig = new HikariConfig();
  hikariConfig.setDriverClassName("net.sf.log4jdbc.sql.jdbcapi.DriverSpy");
  hikariConfig.setJdbcUrl("jdbc:log4jdbc:oracle:thin:@localhost:1521:XE");
  hikariConfig.setUsername("username");
  hikariConfig.setPassword("password");
}
```
<br>
<br>
RootConfig를 수정하고 로그 설정 파일을 추가하는 작업과 JDBC의 연결 정보를 추가 해야 하는데 이는 src/main/resources 밑에 properties파일을 추가해 이를 수행하도록 한다.    
<br>
<br>
```java
log4jdbc.spylogdelegator.name=net.sf.log4jdbc.log.slf4j.Slf4jSpyLogDelegator
```
<br>
<br>
그런데 console 창을 보면...??       
<br>
<br>  

**Console** "Cannot create JDBC driver of class 'net.sf.log4jdbc.sql.jdbcapi.DriverSpy'....."
{: .notice--warning}
<br><br>
보통 책이나 구글링을 하다보면 대부분의 사람들이 properties에 저 코드만 넣어도 동작이 된다고 하는데 나는 이상하게 동작하지 않았다.   
그리고 이에 대한 설명을 해놓은 부분도 찾기가 힘들어 고치는데 상당히 애를 먹었다...ㅠ    
<br>
<br>
그래서 겨우 찾은 해결 방법은 properties파일에 내가 사용하고 있는 데이터베이스의 Drivers정보를 추가해주는 것이었다.   
console창에 뜨는 에러를 읽어봐도 드라이버 정보가 명시되어있지 않아 JDBC 드라이버를 생성할 수 없다고 나와있었다.   
(에러메세지를 잘 읽어보도록 하자....)   
<br>
<br>
```java
log4jdbc.spylogdelegator.name=net.sf.log4jdbc.log.slf4j.Slf4jSpyLogDelegator
log4jdbc.drivers=org.oracledb.jdbc.Driver
```
<br>
혹시나 oracle이 아닌 mySQL, mariaDB같은 경우도 드라이브 정보가 따로 있는 것으로 알고 있다.   
오라클을 사용하고 있다면 위의 코드를 넣어준다면 해결될 것이다.    





