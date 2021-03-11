---
title: "MySQL Master/Slave Replication으로 이중화를 구성해 본 이야기"    
layout: single    
read_time: true    
comments: true   
categories: 
 - project  
toc: true    
toc_sticky: true    
toc_label: contents    
description: 젠킨스 CI를 통한 파이프라인 구성 전 이론을 정리한 포스팅
last_modified_at: 2021-03-02     
---

<br>

현재 진행하고 있는 프로젝트에서는 **MySQL** 이라는 데이터베이스를 사용하고 있습니다. 얼마 전, **MySQL Master/Slave Replication** 을 구성해
반영을 했는데요, 공부한 내용을 정리할 겸 복제구성이 **왜 필요한지, 어떻게 적용하는지** 에 대해 글로 정리하고자 합니다.

<br>
<br>

## Replication은 왜 필요한가요?

데이터베이스에 데이터를 잘 저장하고 잘 불러오고 있는데 굳이 서버 하나를 늘려서 이중화를 시켜야 하는 이유가 무엇입니까? 라고 당연히 물어볼 수 있을 것 같습니다. 
Replication이 필요한 이유는 다음과 같습니다.

<br>

### Scale-out Solutions

일종의 부하 분산을 의미합니다. 제 개인 프로젝트를 포함하여 모든 프로젝트가 그렇지만, DB에 접근해서 처리해야 하는 것들이 대부분일 겁니다. 
읽기, 쓰기, 수정의 모든 연산이 하나의 DB에서 일어난다면 **트래픽이 늘어남에 따라** 자연스럽게 **병목 현상** 이 생길 수 밖에 없습니다.

쓰기는 **원본 서버** 에서만 수행하게 하고 읽기 기능은 **원본의 복제 서버** 에서 읽어오게 한다면 
쓰기의 기능과 읽기의 기능을 병목 없이 모두 향상시킬 수 있게 됩니다. 

<br>

### 데이터의 보안

Replication 을 구성하게 되면 항상 복제를 진행하는게 아닌, **일시중지** 가 가능합니다. 그럼으로써 원천 데이터를 손상시키지 않고 
복제본에서 백업 서비스를 작동시키는게 가능하게 됩니다.

<br>

### 장애 극복

만약 **Master** DB가 모종의 이유로 장애가 발생해 사용이 중지 되었다면 바로 **Slave**를 Master로 지정해 데이터의 대한 복구를 
빠르게 진행할 수 있습니다. **그러나** MySQL Replication은 **비동기** 방식으로 진행하기 때문에 Slave로 복사 되는 시간을 온전히
기다려 주지 않습니다. 때문에 데이터가 완벽하게 복사가 되지 않을 수도 있습니다.

<br>

> Replication의 필요성을 이렇게 알아볼 수 있었는데요, 그러면 Replication은 어떤 원리로 동작을 하게 될까요?

<br>
<br>

## Replication의 동작 원리

위의 그림을 차례차례 설명을 해보겠습니다 :)

1. 클라이언트가 **Commit** 을 누르면 먼저 Master 서버에 존재하는 **Binary log** 에 변경사항을 모두 기록합니다.
2. **Master Thread** 는 **비동기적** 으로(복사되는 시간을 기다려주지 않습니다.) Binary log를 읽어 Slave 서버로 전송합니다.
3. Slave 의 **I/O Thread** 는 Master로 부터 받은 변경 데이터들을 **Relay log** 에 기록을 합니다.
4. Slave의 **SQL Thread** 는 Replay log의 기록들을 읽어 자신의 스토리지 엔진에 최종 적용합니다.

<br>

> Replication 의 동작 방식을 알아보았으니, 이제 실제로 적용을 해봐야 할 것 같은데 어떻게 해야할까요?

<br>
<br>

## Replication을 구성해 내 프로젝트에 적용해보자

시작하기에 앞서, 제 프로젝트의 환경 구성을

> - 네이버 클라우드 플랫폼 Compact g1 1vCPU, 2GB Memory, 50GB Disk(HDD) 
> - MySQL 5.7.33 버전 
> - Linux ubuntu 16.04-64-server

이렇게 설정하였으니 참고 바랍니다!!


<br>

### 서버 생성하기

우선, 원본 서버 이외에 한 개의 서버를 생성합니다. (`festa` 라는 이름이 마스터 서버, `festa-slave` 가 슬레이브 서버로 지정하였습니다.
슬레이브 서버에도 **MySQL** 을 설치해주도록 합니다. 이때 중요한 점은 **슬레이브 서버는 미스터 서버의 DB와 버전이 같거나 또는 더 높아야 합니다.**

<br>

### 슬레이브 서버에 Replication 용 새로운 계정 생성 

```bash
mysql> create user '아이디'@'%' identified by '비밀번호'; 
```

그 다음, 마스터 서버 `festa` 에 접속하여 **MySQL** 에 로그인 한 다음, 새로운 계정을 하나 생성해줍니다.

<br>

### 슬레이브에 새로운 DB 생성

```bash
mysql> create database festa;
```

마스터 서버에 있는 같은 이름의 DB를 슬레이브 서버에도 하나 생성해주도록 합니다.

<br>

### 새로 생성한 계정에 권한 부여

```bash
mysql> grant replication slave on *.* to '아이디'@'%' identified by '비밀번호';
mysql> flush privileges;
```

Replication 용 계정에 슬레이브 권한을 부여하고 최종 적용을 합니다. 

<br>

### 마스터 DB my.cnf 수정

```bash
# sudo nano /etc/mysql/my.cnf
```

마스터 서버에 다시 접속하여 MySQL의 `my.cnf` 파일을 열어 마스터 서버에 대한 수정을 진행합니다.
아래에 해당하는 내용을 파일에 붙여넣으시면 됩니다. 편집 완료 후, MySQL을 한번 **재기동** 합니다.

```
[mysqld]
log-bin=mysql-bin    
server-id=1          # 서버 구분값         
binlog-do-db=festa   # 복제할 DB명
```

<br>

### 슬레이브 DB my.cnf 수정

마스터와 동일하게 슬레이브 서버에도 `my.cnf` 파일 편집이 필요한데, 하나 다른 점은
`server-id` 를 꼭 **마스터 server-id 와 다르게 입력** 해야합니다. 편집 완료 후, 마스터와 같이
한번 **재기동** 을 해주도록 합니다.

```
[mysqld]
log-bin=mysql-bin    
server-id=2        
```

<br>

### 슬레이브로 log file 정보 전송

```bash
mysql> show master status
```

이 때 조회되는 파일명과 `Position` 값이 중요합니다. 이 정보를 바탕으로 Slave 가 Master로 부터 정보를 가져오기 때문입니다.
그리고 Database의 덤프파일을 생성합니다.

<br>

```bash
# mysqldump -u root -p festa > festa.sql
# scp festa.sql '아이디'@'슬레이브 IP주소':/root/
```

처음 수행하는 거라면, 권한이 필요하다는 메세지가 출력될텐데 `yes` 라고 클릭해주시고 파일 비밀번호를 입력합니다.
그리고 `scp` 명령어를 통해 슬레이브 서버로 해당 파일을 전송합니다.

<br>

### 슬레이브에서 mysqldump 파일 받기

```bash
mysql -u root -p festa < festa.sql
```

<br>

이제 마스터에서 전송해 준 DB 정보파일을 위의 명령어를 입력해서 받도록합니다.
이때 **비밀번호** 를 입력하라고 하는데 마스터에서 `.sql` 파일을 만들 때 입력한 비밀번호를 치면
파일 전송을 완료하게 됩니다. 저는 마스터와 같은 이름의 DB `festa` 에 넣어주었습니다. 만약 같은 이름의 DB가 없을 경우
신규 생성을 꼭 하셔야 전송이 됩니다.
