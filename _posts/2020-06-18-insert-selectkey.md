---
title: "Spring 게시판에서 insert와 insertSelectKey"
layout: single    
read_time: true    
comments: true   
categories: 
 - spring  
toc: true    
toc_sticky: true    
toc_label: contents    
description: 그냥 insert할 때와 insertSelectKey를 적용했을 때의 차이점
last_modified_at: 2020-06-18   
---   

<br>  
게시판 기능에서 insert를 할 때 2가지의 처리방식이 있다.     
- insert만 처리되고 생성된 PK값을 몰라도 되는 경우
- insert가 처리되고 생성된 PK값을 알아야 하는 경우<br> 

<br>  
만약 2번째와 같은 상황이 자주 발생이 되는 경우 Mapper interface로 구현된 파일에 method를 하나 추가해주도록 한다.    
<br>
<br> 

## Insert  
### Mapper Interface  

```java
public interface BoardMapper { 
	public void insert(BoardVO board);
	public void insertSelectKey(BoardVO board); //추가된 method
}
```

<br>
### Mapper XML   

```xml
<insert id="insert">
		insert into tbl_board (bno, title, content, writer)
		values ((select nvl(MAX(bno+1), 1) from tbl_board), #{title}, #{content}, #{writer})
	</insert>
  
	<insert id="insertSelectKey"> //추가된 sql
		<selectKey keyProperty="bno" order="BEFORE" resultType="long">
			select seq_board.nextval from dual
		</selectKey>
			insert into tbl_board (bno, title, content, writer)
			values (#{bno}, #{title}, #{content}, #{writer})
	</insert>
```  
<br>   
method가 하나 추가되었으니 그에 맞는 sql쿼리문을 xml파일에도 하나 추가해 준다.   
**SelectKey**라는 기능이 하나 추가된 insert문을 볼 수 있다.   
그냥 insert랑 selectkey가 추가된 insert의 차이를 알기 위해서 test를 해보았다.    
<br>
### Mapper Insert 단위테스트   

```java
//BoardMapperTest파일
@Test
public void testInsert() {
  BoardVO vo = new BoardVO();
  vo.setTitle("하하");
  vo.setContent("내용");
  vo.setWriter("tester");
  
  mapper.insert(vo);
  log.info(vo);
}
```
<br>  
우선 일반 insert를 실행했을 때 console화면에 뜨는 로그를 확인해보면
```java
INFO: xxx.xxx.mapper.BoardMapperTest - BoardVO(bno=null, title=하하, content=내용.....)
```
<br>  
(나는 bno라는 글 게시물 번호에 seq값을 넣어주어 pk값으로 설정해두었다)   
bno의 값이 null로 설정되어 입력하기 전에 bno값이 어떤 값으로 설정되어있는지 전혀 알 수가 없다.   
DB로 입력되었을 때 자동으로 생성되기 때문에 게시물의 데이터를 입력하기 전 bno값을 알고 싶다면 SelectKey를 사용하면 된다.   
<br>
<br>
### Mapper InsertSelectKey 단위테스트   

```java
//BoardMapperTest파일
@Test
public void testInsert() {
  BoardVO vo = new BoardVO();
  vo.setTitle("하하");
  vo.setContent("내용");
  vo.setWriter("tester");
  
  mapper.insertSelectKey(vo);
  log.info(vo);
}
```
<br>  
```java
//Console
INFO: jdbc.sqlonly - select seq_board.nextval from dual
....//(skip)
INFO : 1. Connection.preparedStatement(insert into board.........)
....//(skip)
INFO: xxx.xxx.mapper.BoardMapperTest - BoardVO(bno=1, title=하하, content=내용.....)
```
<br>  
실행결과를 살펴보면 **selectKey**에 order를 **BEFORE**로 설정해놨기에 insert보다 먼저 쿼리문이 실행이 되어진다.   
bno값이 먼저 실행이 되고 insert가 실행이 되기 때문에 console에서 3번째 info를 확인해본다면 bno값이 어떤 값으로 설정되었는지 확인할 수 있다.   
<br>
<br>
물론 sql문이 두번이나 실행되어지기 때문에 약간의 부담이 될 수도 있지만, pk값이 sequence로 설정되어 있어 자동으로 생성되는 값을 확인하고 싶다면 유용하다고 할 수 있다.   
<br>
<br>
<br>
<br>


