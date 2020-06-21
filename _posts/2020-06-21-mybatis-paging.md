---
title: "Mybatis-Spring에서 페이징 처리하기"
layout: single    
read_time: true    
comments: true   
categories: 
 - spring  
toc: true    
toc_sticky: true    
toc_label: contents    
description: mybatis 페이징 처리와 해당 sql 쿼리문에 대한 정리
last_modified_at: 2020-06-21   
---   

<br> 

게시판에서 한 페이지 당 정해진 갯수의 게시물을 보여주고 싶을 때 페이징 처리를 하게 되는데, 
이 때 실행되는 SQL에서 몇 가지 파라미터가 필요하다.  
<br>

- 페이지 번호(pageNum)
- 한 페이지 당 보여줄 게시물의 갯수(amount) <br>
<br>

우선 xxx.xxx.domain 패키지에 Criteria 라는 클래스를 하나 만들어주도록 한다. 
<br>

## 페이징 처리 클래스 추가

### domain / Criteria 생성
```java
package org.zerock.domain;

import lombok.Getter;
import lombok.Setter;
import lombok.ToString;

@Getter
@Setter
@ToString
public class Criteria {
	private int pageNum;
	private int amount;
	
	public Criteria() {
		this(1, 10); // 생성자를 통해 기본값을 정해놔야 한다
	}
	
	public Criteria(int pageNum, int amount) {
		this.pageNum = pageNum;
		this.amount = amount;
	}
}

```
<br>
<br>
<br>

### mapper.java method 추가
mapper 패키지에서 게시물에 해당하는 Mapper 클래스에 Criteria 타입을 파라미터로 사용하는 method를 추가해주도록 한다.

**Criteria?** 페이지 검색의 기준을 위한 클래스로 편의를 위해 하나의 객체로 묶어서 전달하기 위함 
{: .notice--info}

<br>

```java
package org.zerock.mapper;

import java.util.List;

import org.apache.ibatis.annotations.Param;
import org.zerock.domain.BoardVO;
import org.zerock.domain.Criteria;

public interface BoardMapper { 
	.
  .
  .
  . //생략
	public void getListPage(Criteria cri);
}
```
<br>
<br>
<br>

### mapper.xml에 method에 해당하는 sql 쿼리문 추가
sql 쿼리문을 수행할 mapper.xml에 추가한 메서드에 맞는 태그를 추가하도록 한다.   
<br>

```xml
<select id=getListPage resultType="xxx.xxx.domain.BoardVO">
		<![CDATA[
			select bno, title, content, writer, regdate, updatedate from 
			(select /*+INDEX_DESC(tbl_board pk_board)*/ 
			rownum rn, bno, title, content, writer, regdate, updatedate from tbl_board 
			where rownum <= 20) 
			where rn > 10
		]]>
	</select>
```
<br>
<br>
<br>

### 주의할점(+인라인뷰)
where 절에서 만약에 10에서 20에 해당하는 rownum을 가져올 때     
**"select /*+INDEX_DESC(tbl_board pk_board)*/ rownum rn, bno, title, content from board where rownum > 10 and rownum <= 20"** 하면 될 것 같지만 
그렇게 하면 아무런 결과가 나오지 않게 된다. (대체 왜...?)
<br>
<br>

데이터베이스의 실행 계획을 잘 살펴보도록 하자(sql developer에 계획설명을 클릭하면 나온다!)   
실행 계획은 안쪽에서 바깥쪽으로, 위에서 아래로 보게 되어 제일 먼저 찾는 부분은 rownum > 10에 해당하는 데이터들을 찾는다. 
<br>
<br>

그런데 문제는 board에서 처음으로 나오는 rownum은 무조건 1부터 나오게 되어있고 없어지는 과정을 반복하기 때문에
아무리 찾아도 매번 where 조건에 의해 무효화 되어 출력이 되지 않는 것이다.    
<br>
<br>

이러한 이유로 sql을 작성할 때 조건은 항상 1을 포함하고 있어야 하며 where절 이하는 일단 rownum <= 20으로만 써두도록 한다.   
이렇게 되면 rownum 1부터 20까지의 데이터가 나오게 되는데 이럴 때 in-line view 처리를 하여 1~20에서 10~20만 빼서 출력해주도록 한다.   
(코드블럭 마지막 where절의 변화를 살펴보자)
<br>
<br>
<br>

## Test
10~20에 해당하는 데이터가 제대로 출력이 되는지 mapperTest에서 실행해보도록 한다. 
<br>

```java
@Test
public void testPage() {
  Criteria cri = new Criteria();
  List<BoardVO> list = mapper.getListPage(cri);
  list.forEach(board -> log,info(board));
}
```
<br>

console에 로그가 제대로 찍힌다면 성공이다!
이제 10과 20이라는 숫자를 pageNum 과 amount라는 변수로 변경하는 작업을 하도록 한다. 
<br>

```xml
<select id=getListPage resultType="xxx.xxx.domain.BoardVO">
		<![CDATA[
			select bno, title, content, writer, regdate, updatedate from 
			(select /*+INDEX_DESC(tbl_board pk_board)*/ 
			rownum rn, bno, title, content, writer, regdate, updatedate from tbl_board 
			where rownum <= #{pageNum} * #{amount}) 
			where rn > (#{pageNum} - 1) * #{amount}
		]]>
	</select>
```

```java
//테스트 해보기

@Test
public void testPage() {
  Criteria cri = new Criteria();
  
  //10개씩 2페이지 - 2페이지에 해당하는 내용 출력
  cri.setPageNum(2);
  cri.setAmount(10);
  List<BoardVO> list = mapper.getListPage(cri);
  list.forEach(board -> log,info(board.getBno()));
}
```
<br>

## Service와 ServiceImplemented 변경하기

브라우저에서 들어오는 정보로 처리를 하기 때문에 controller와 service, serviceImplemented 모두 변경이 필요하다.   
<br>
<br>

### Service method 변경

```java
public interface BoardService() {
  .
  .
  . //skip
  public List<BoardVO> getList(Criteria cri);
```
<br>
<br>

### ServiceImplemented 변경

```java
@Override
public List<BoardVO> getList(Criteria cri) {
  log.info("List w/ page: " + cri);
  return mapper.getListPage(cri);
}
```
<br>
<br>

### Controller 변경

```java
@GetMapping("/list")
public void list(Criteria cri, Model model) {
  log.info("list: " + cri);
  model.addAttribute("list", service.getList(cri));
}
```
<br>
<br>

### Controller test

```java
@Test
public void testPage() {
  log.info(mockMvc.perform(
    MockMvcRequestBuilders.get("board/list")
    .param("pageNum", "2")
    .param("amount", "50"))
    .andReturn().getModelAndView().getModelMap());
}
```
<br>

페이징 화면 처리에 대한 내용은 없지만 우선 뒷단을 이렇게 해서 test했을 때 나온다면 로직에 문제가 없다는 의미이다.   
<br>
<br>
<br>

## DTO 클래스 생성

마지막으로 DTO클래스를 하나 정의를 해주도록 하자(화면 페이징 처리를 위해 여러 정보가 필요한데 하나의 객체로 구성하면 아주 편하다)    
<br>

```java

import lombok.Getter;
import lombok.ToString;

@Getter
@ToString
public class PageDTO {
	private int startPage;
	private int endPage;
	private boolean prev, next;
	private int total;
	private Criteria cri;
	
	public PageDTO(Criteria cri, int total) {
		this.cri = cri;
		this.total = total;
		
		this.endPage = (int) (Math.ceil(cri.getPageNum() / 10.0)) * 10; // 1. 페이지 끝번호(먼저 계산해두면 시작번호는 -9만 하면 됨
		this.startPage = this.endPage - 9; // 2. 페이지 시작번호 계산
		
		int realEnd = (int) (Math.ceil((total * 1.0) / cri.getAmount())); // 3. 끝번호 다시 계산
		
		if(realEnd < this.endPage) {
			this.endPage = realEnd;
		}
		
		this.prev = this.startPage > 1; // 이전페이지
		this.next = this.endPage < realEnd; //다음페이지
	}
}
```
<br>

### Math.ceil()을 이용한 끝번호 계산 방법

- 1페이지 : Math.ceil(0.1)*10 = 10
- 10페이지 : Math.ceil(1)*10 = 10
- 11페이지 : Math.ceil(1.1)*10 = 20 
<br>
<br>

### realEnd 

endPage의 경우 전체 데이터 수에 따라 영향을 받기 때문에 전체 데이터 수가 70이라면 7이 끝번호가 되어 
7페이지 까지만 보여주어야 한다. 그래서 endPage와 한 페이지 당 출력되는 amount의 곱이 전체 데이터 수(total)보다 크다면 
endPage는 다시 total을 이용하여 재 계산을 해주도록 한다.
<br>
<br>

전체 데이터 수(total)를 이용해서 realEnd가 몇 번까지 되는지를 계산하고 endPage가 realEnd보다 작다면 끝 번호는 작은 값으로 가도록 해야한다.
<br>
<br>
.
.
.
.
.
진짜 어렵다..ㅠ
<br>
<br>
<br>
<br>
<br>








