---
title: "실행계획을 분석해서 SQL 성능튜닝을 해보자(feat.MySQL)"    
layout: single    
read_time: true    
comments: true   
categories: 
 - project  
toc: true    
toc_sticky: true    
toc_label: contents    
description: MySQL의 실행계획을 분석해 비효율적인 부분을 어떻게 개선해 나갔는지 정리한 포스팅
last_modified_at: 2021-04-21   
---

<br>
<br>

## Overview

최근 몇달 간 MySQL에 대해 공부하기 위해서 [Real MySQL](https://book.naver.com/bookdb/book_detail.nhn?bid=6886962) 이라는 책을 읽고 있습니다. 
6번째 장에는 **"실행계획"** 이라는 내용이 소개가 되는데요, `EXPLAIN` 이라는 예약어를 활용하여 현재 작성한 쿼리문이 어떤 원리로 돌아가고 있는지를 설명해주는 부분입니다.

6장을 거의 다 읽어갈 때 쯤, **"뭔가 내가 작성한 쿼리의 실행계획을 살펴보면 고칠 부분이 많겠다"** 라는 생각이 들어 현재 진행 중인 [festa 
프로젝트](https://github.com/f-lab-edu/event-recommender-festa)에 한번 적용해보기로 하였습니다.

어떤 쿼리를 우선적으로 튜닝해볼까 고민하다가 **이벤트 목록조회** 쿼리문을 튜닝해보기로 결정했습니다!

<br>
<br>

## 왜 이벤트 목록조회를 제일 먼저 튜닝했나요?

다른 쿼리문도 많은데 **이벤트 목록조회** 를 선택한 이유는, 우선 **내 주변지역의 이벤트와 행사** 를 추천하는 프로젝트이고 저는 이 서비스가
수 많은 유저들이 이용한다는 가정하에 진행하고 있습니다. 

목록조회 쿼리는 `members` 라는 사용자 테이블을 조인하게 되는데, 서비스가 점점 커진다면 유저는
끝없이 늘어나게 됩니다. 그만큼 수 많은 유저들이 가장 많이 이용하는 기능이 바로 **목록조회** 이기 때문에 성능에 대한 이슈가 빈번히 발생할 여지가 많았습니다.

그래서 [이전 포스팅](https://jane096.github.io/project/redis-caching/)에도 작성한 적이 있지만, 캐싱기능을 적용한 것도 이러한 문제 소지가 있었기 때문이죠.

하지만 SQL 자체가 비효율적으로 작동되고 있다면 캐싱을 적용한다고 완전 해결될 문제는 아니었습니다. 그래서 다른 것보다도 우선적으로 튜닝해보게 
되었습니다.

<br>
<br>

## 기존 SQL을 먼저 분석해보자

그럼 본격적으로 실행계획 분석을 시작해보겠습니다. 내용이 길 수도 있으니 천천히 읽어주시기 바랍니다! 

<br>

```sql
SELECT
      A.userNo,
      A.eventNo,
      A.eventTitle,
      A.eventContent,
      A.startDate,
      A.endDate,
      A.categoryCode,
      A.participantLimit,
      A.noOfParticipants,
      B.registerDate,
      B.cityName,
      B.districtName,
      B.streetName,
      B.detail,
      B.writer
  FROM events AS A
  JOIN
      (SELECT
          E.userNo,
          E.eventNo,
          E.eventTitle,
          E.eventContent,
          E.startDate,
          E.endDate,
          E.categoryCode,
          E.participantLimit,
          E.noOfParticipants,
          E.registerDate,
          EA.cityName,
          EA.districtName,
          EA.streetCode,
          EA.streetName,
          EA.detail,
          M.userName AS writer
      FROM events E
      LEFT OUTER JOIN event_address EA
      ON EA.eventNo = E.eventNo
      LEFT OUTER JOIN category C
      ON C.categoryCode = E.categoryCode
      INNER JOIN members M
      ON E.userNo = M.userNo
      WHERE E.userNo < 1001
      AND E.categoryCode = 1
      ORDER BY registerDate DESC
      LIMIT 10
      ) AS B
  WHERE A.eventNo = B.eventNo;
```

<br>

지금 다시보니 정말 복잡하게 생기고 분량도 꽤 되는 것 같습니다. 이 쿼리를 `EXPLAIN` 으로 실행계획을 살펴보면 어떻게 나올까요?

<br>

### 변경 이전 SQL 실행계획 결과

![image](https://user-images.githubusercontent.com/58355531/115554759-c91af980-a2e9-11eb-8c95-b3baef0903ef.png){: .align-center}

<br>

위의 쿼리는 이런 실행계획으로 돌아가게 되는데요 여기서 중요한 점만 찝어 현재 대략적으로 어떻게 돌아가고 있고, 비효율적인 부분이
어느부분인지 정리해보겠습니다.

<br>

### select_type

`SELECT` 쿼리가 어떤 타입의 쿼리인지를 나타냅니다. 제 쿼리의 경우 **DERIVED** 라는 표시가 많이 보이는데 이 의미는 현재 임시테이블을 만들었다는 뜻이며 
보통 `FROM` 절에 서브쿼리를 만들었을 경우 `DERIVED` 가 뜨게 됩니다. 

하지만 임시테이블이 생성된 경우 **인덱스가 전혀없기 때문에** 성능저하의 우려가 있는 부분입니다. 
아직 MySQL 에서 `FROM` 절에 서브쿼리는 최적화가 많이 개선되진 않았기에 이미 많은 최적화가 되어있는 조인이나 아니면 제거하는 쪽으로 변경할 예정입니다.

<br>

### type

MySQL이 각 레코드를 어떤 방식으로 읽었는지를 의미하는데 인덱스를 사용했는지 아니면 테이블 풀스캔을 했는지를 이 컬럼을 통해 알 수 있습니다.
`ALL` 의 경우 테이블 접근 방식 중에서 가장 성능이 낮은 걸로, 인덱스를 전혀타지 않고 테이블 풀스캔을 했다는 의미입니다. 
위의 쿼리 경우 `events` 테이블을  `ALL` 로 풀스캔을 하고 있고, `FROM` 절 다음의 서브쿼리 또한 테이블 풀스캔 중이기 때문에 이 후 데이터가 많아지면 상당히 느려질 여지가 많았습니다. 

<br>

### Extra

이 부분에서는 `Using temporary` 나 `Using filesort` 가 뜨고있습니다. 전자는 임시테이블을 생성했다는 뜻(임시테이블에는 인덱스 없음, 
인덱스 사용 안하고 있음)이고 후자는 인덱스를 사용하지 못해 MySQL 서버가 조회된 레코드를 다시정렬을 했다는 의미입니다.

<br>

### row

총 몇개의 레코드를 읽었는지에 대한 정보입니다. 테이블 `E(events)` 테이블은 테이블 풀스캔을 하고 있으니 `events` 안의 데이터를 전부다 읽고 있습니다. 
(인덱스를 통한 스캔을 원했으나 그게 아니니.. 개선해야할 부분이겠죠)

<br>
<br>

## 이제 튜닝을 시작해봅시다!

이제 실행계획을 살펴보고 개선해야할 부분을 찾았으니 차근차근 해쳐나가보도록 합시다~!

<br>

### DERIVED 제거

```sql
SELECT
      E.userNo,
      E.eventNo,
      E.eventTitle,
      E.eventContent,
      E.startDate,
      E.endDate,
      E.categoryCode,
      E.participantLimit,
      E.noOfParticipants,
      E.registerDate,
      EA.cityName,
      EA.districtName,
      EA.streetCode,
      EA.streetName,
      EA.detail,
      M.userName AS writer
  FROM events E
   LEFT OUTER JOIN event_address EA
   ON EA.eventNo = E.eventNo
   LEFT OUTER JOIN category C
   ON C.categoryCode = E.categoryCode
   INNER JOIN members M
   ON E.userNo = M.userNo
  WHERE E.userNo < 1001
   AND E.categoryCode = 1
  ORDER BY registerDate DESC
  LIMIT 10;
```

<br>

![image](https://user-images.githubusercontent.com/58355531/115982457-31155c80-a5d6-11eb-8abb-fff4c9913eac.png){: .align-center}

`DERIVED` 의 원인이 되는 `FROM` 절 다음 서브쿼리 부분을 없애주어 `SIMPLE` 이 뜨도록 변경했습니다.
이렇게 하니 `events_address` 테이블 `EA` 가 여전히 테이블 풀스캔과 `Using temporary` , `Using filesort` 를 하고 있었고 나머지는 `eq_ref` 가 뜨게 되었습니다.

<br>

### LEFT OUTER JOIN 수정

```sql
SELECT
    E.userNo,
    E.eventNo,
    E.eventTitle,
    E.eventContent,
    E.startDate,
    E.endDate,
    E.categoryCode,
    E.participantLimit,
    E.noOfParticipants,
    E.registerDate,
    EA.cityName,
    EA.districtName,
    EA.streetCode,
    EA.streetName,
    EA.detail,
    M.userName AS writer
FROM events E
 INNER JOIN event_address EA
 ON EA.eventNo = E.eventNo
 INNER JOIN members M
 ON E.userNo = M.userNo
WHERE E.userNo < 1001
 AND E.categoryCode = 1
ORDER BY registerDate DESC
LIMIT 10;
```

테스트 데이터라 `event_address` 테이블이 추가 전 입력된 데이터들은 주소지에 대한 정보가 없었습니다. 주소지가 있는 것만 불러와야 하는데 필요없는 데이터까지 조회할 수 있겠다 싶어 
`LEFT OUTER` 를 제거하고 `INNER JOIN` 으로 바꾸었습니다. 

> 지금 근무하는 회사에서는 거의 모든(99%) SQL이 `LEFT OUTER JOIN` 으로만 조인을 걸고 있습니다.(거짓말이 아니라 정말로 그렇습니다.)
그렇다 보니 저도 모르게 아무 이유없이 쓰게 되는 버릇이 보였던 것 같아 이 기회에 개선해야겠다는 생각이 많이 들었습니다.

<br>

`category` 테이블은 카테고리코드가 `events` 테이블에 존재하고 있습니다. 카테고리명을 뽑아내기 위해 조인을 걸었지만 이름을 명시해 주는 것이 
굳이 조회 쿼리에서 뽑아낼 필요가 있을까 싶어 제거하게 되었습니다.(조인을 거니 `event_address` 이 테이블 풀스캔을 하게 되는 추가적인 문제도 있어 제거하였습니다.)

<br>

### order by 수정

```sql
SELECT
    E.userNo,
    E.eventNo,
    E.eventTitle,
    E.eventContent,
    E.startDate,
    E.endDate,
    E.categoryCode,
    E.participantLimit,
    E.noOfParticipants,
    E.registerDate,
    EA.cityName,
    EA.districtName,
    EA.streetCode,
    EA.streetName,
    EA.detail,
    M.userName AS writer
FROM events E
 INNER JOIN event_address EA
 ON EA.eventNo = E.eventNo
 INNER JOIN members M
 ON E.userNo = M.userNo
WHERE E.userNo < 1001
 AND E.categoryCode = 1
ORDER BY E.eventNo DESC
LIMIT 10;
```

`events` 테이블 내에 존재하는  `registerDate` 는 인덱스가 적용된 컬럼이 아닙니다. 인덱스를 사용하지 못한 정렬작업 때문에 
당연히 `Using temporary` , `Using filesort` 가 발생할 수 밖에 없었습니다.

그래서 인덱스가 걸려있는 `eventNo` 로 변경을 해주었습니다. 이 컬럼은 `auto_increment` 가 걸려있기 때문에 등록된 순서대로 부여가 됩니다. 
즉 굳이 `registerDate` 기준이 아니더라도 `eventNo` 만으로 최신등록 순 부터 보여줄 수 있을 것이라고 생각해 변경하게 되었습니다.

<br>

![image](https://user-images.githubusercontent.com/58355531/115982660-d11fb580-a5d7-11eb-86b4-2e826e19c82f.png){: .align-center}

그럼에도 `event_address` 테이블은 계속 테이블 풀스캔을 하고 있었는데 `where E.userNo < 5 and E.categoryCode = 1` 명령문이 임시테이블을 생성하게 하여 
인덱스를 사용하지 못하고 정렬해 뜨는 문제로 보였습니다.

<br>

### WHERE절 수정, 그리고 최종 변경

```xml
<!-- 현재 프로젝트 상에 최종 적용된 SQL 입니다! -->

SELECT
      E.userNo,
      E.eventNo,
      E.eventTitle,
      E.eventContent,
      E.startDate,
      E.endDate,
      E.categoryCode,
      E.participantLimit,
      E.noOfParticipants,
      E.registerDate,
      EA.cityName,
      EA.districtName,
      EA.streetCode,
      EA.streetName,
      EA.detail,
      M.userName AS writer
  FROM events E
   INNER JOIN event_address EA
   ON EA.eventNo = E.eventNo

  <!-- 카테고리별 조회-->
  <if test="categoryCode != NULL">
    AND E.categoryCode = 1
  </if>

   INNER JOIN members M
   ON E.userNo = M.userNo 
  WHERE E.userNo < 1001
  ORDER BY EA.eventNo DESC
  LIMIT 10;
```

그래서 `WHERE` 이하에 있던 카테고리별 조회 코드를 `event_address` 와 조인할 때 걸러질 수 있도록 수정을 하여 임시테이블 생성을 차단하였습니다.

`ORDER BY` 에서 `events` 테이블의 `eventNo` 기준으로 내림차순 정렬을 하게 되면 `event_address` 의 `eventNo` 가 인덱스를 활용하지 못하는 문제점이 있어 
`event_address` 기준으로 정렬하도록 교체해 `event_address` 가 `eventNo` 를 통해 인덱스 스캔하여 조금 더 빠른 읽기가 가능하도록 최종 변경하였습니다.

<br>
<br>

## 최종 변경 쿼리 실행계획

![image](https://user-images.githubusercontent.com/58355531/115556338-8bb76b80-a2eb-11eb-8434-850f832b880d.png){: .align-center}

<br>

이렇게 실행계획 분석을 통해서 SQL 수정이 완료되었습니다! 처음 해보는 거라 제가 미처 발견하지 못한 부분이 있을 수도 있겠지만 크게 성능저하가
올 수 있는 부분은 어느정도 해결이 된 것 같습니다. 현재는 모두 필요한 레코드만 가져다 **인덱스 스캔** 을 하고 있어 이전에 테이블 풀스캔을 하던 부분은 
많은 개선이 이루어졌습니다.

실제로 돌아간다고 거기서 끝이 아닌 직접 실행계획을 조회해보고 분석해보고 개선해 나가는 작업이 저에겐 소중한 기회가 되었던 것 같습니다.
조만간 다른 성능튜닝 이야기로 다시 돌아오겠습니다!! 

긴 글 읽어 주셔서 감사합니다~!

<br>
<br>

## Project Github URL 

[![오구리이미지](https://user-images.githubusercontent.com/58355531/99896015-085c0480-2cd0-11eb-998d-8b8faeb43e17.gif)](https://github.com/f-lab-edu/event-recommender-festa "이미지를 클릭하면 프로젝트 깃허브를 볼 수 있습니다 :)")

[FESTA 프로젝트 Github 보러가기 Click! (또는 위의 이미지 Click!)](https://github.com/f-lab-edu/event-recommender-festa)

<br>
<br>
<br>

## Referenced by

- Real MySQL: 이성욱 지음      
  <https://book.naver.com/bookdb/book_detail.nhn?bid=6886962>
  
<br>
<br>
<br>
<br>
<br>
<br>

