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

데이터베이스에 데이터를 잘 저장하고 잘 불러오고 있는데 **굳이 서버 하나를 늘려서 이중화를 시켜야 하는 이유가 무엇입니까?** 라고 당연히 물어볼 수 있을 것 같습니다. 
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

![image](https://user-images.githubusercontent.com/58355531/110962962-deb90d00-8394-11eb-942d-9eb5690daffa.png){: .align-center} 

<br>

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

![image](https://user-images.githubusercontent.com/58355531/110955295-ca711200-838c-11eb-802c-74f789fcba3f.png){: .align-center}       

우선, 원본 서버 이외에 한 개의 서버를 생성합니다. `festa` 라는 이름이 마스터 서버, `festa-slave` 가 슬레이브 서버로 지정하였습니다.
슬레이브 서버에도 **MySQL** 을 설치해주도록 합니다. 이때 중요한 점은 **슬레이브 서버는 마스터 서버의 DB와 버전이 같거나 또는 더 높아야 합니다.**

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

### 슬레이브로 보낼 덤프파일 생성하기 

```bash
# flush tables with read lock;
# mysqldump -u root -p festa > festa.sql
# scp festa.sql '아이디'@'슬레이브 IP주소':/root/
# unlock tables;
```

덤프파일을 생성 전, DB의 `lock`을 걸어 임시로 읽기 쓰기를 차단하고 진행합니다. 이 후, `mysqldump` 명령어를 이용해 DB 정보가 담긴
덤프파일을 생성합니다.

처음 수행하는 거라면, 권한이 필요하다는 메세지가 출력될텐데 `yes` 라고 클릭해주시고 파일 비밀번호를 입력합니다.
그리고 `scp` 명령어를 통해 슬레이브 서버로 해당 파일을 전송합니다.

전송 후, 락을 걸었던 것을 해제 시켜주면 됩니다:)

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

<br>

### SHOW MASTER STATUS

이제 기본적인 설정은 끝났습니다. 본격적으로 두 서버를 연결해야 하는데요, 우선 마스터의 DB 정보를 조회해야합니다.
헤더 제목과 같이 아래의 명령어를 입력하면 캡처화면과 같은 정보가 뜨게 됩니다.

```bash
mysql> show master status;
```

<br>

![image](https://user-images.githubusercontent.com/58355531/110955659-2b004f00-838d-11eb-8aae-572ea5ffe2b8.png){: .align-center} 

<br>

위의 캡처에서 `File` 명과 `Position` 이 나중에 슬레이브에게 전달해 줄 중요 정보들입니다. 그래서 따로 메모를 해두는 것이 좋습니다!
`Binlog-Do-DB` 는 복제할 데이터베이스명을 의미합니다.

<br>

### 슬레이브에 마스터 정보 입력하기

```bash
CHANGE MASTER TO MASTER_HOST='27.96.135.160', MASTER_PORT=3306, MASTER_USER='아이디', MASTER_PASSWORD='비밀번호', MASTER_LOG_FILE='mysql-bin.000016', MASTER_LOG_POS=154;  
```

아까 마스터에서 `show master status` 때 보였던 정보를 `MASTER_LOG_FILE` 과 `MASTER_LOG_POS` 에 각각 넣어주고 마스터의 **공인IP** 주소, MySQL 포트번호인 **3306** 그리고
Replication 용으로 만든 계정의 정보를 입력해주고 엔터를 누릅니다. 

아무런 에러가 뜨지 않았다면 정상적으로 입력된 것이기에 `start slave;` 명령어를 날려 Replication을 시작합니다.

<br>

### 연동 확인해보기 

**첫번째** 기동 된 슬레이브의 **Status**를 확인해보는 작업입니다.

```bash
mysql> show slave status\G;
```

이렇게 입력 후 정상적으로 기동이 되었다면 아래와 같이 **Waiting for master to send event** 라는 상태 정보가 뜨면서 
**Slave_IO_Running: Yes**, **Slave_SQL_Running: Yes** 가 뜬다면 연동이 성공적으로 이루어진 것입니다.

```bash
# 전체 로그 

 Slave_IO_State: Waiting for master to send event
                  Master_Host: 27.96.135.160
                  Master_User: repl
                  Master_Port: 3306
                Connect_Retry: 60
              Master_Log_File: mysql-bin.000022
          Read_Master_Log_Pos: 154
               Relay_Log_File: festa-slave-relay-bin.000022
                Relay_Log_Pos: 367
        Relay_Master_Log_File: mysql-bin.000022
             Slave_IO_Running: Yes
            Slave_SQL_Running: Yes
              Replicate_Do_DB:
          Replicate_Ignore_DB:
           Replicate_Do_Table:
       Replicate_Ignore_Table:
      Replicate_Wild_Do_Table:
  Replicate_Wild_Ignore_Table:
                   Last_Errno: 0
                   Last_Error:
                 Skip_Counter: 0
          Exec_Master_Log_Pos: 154
              Relay_Log_Space: 627
              Until_Condition: None
               Until_Log_File:
                Until_Log_Pos: 0
           Master_SSL_Allowed: No
           Master_SSL_CA_File:
           Master_SSL_CA_Path:
              Master_SSL_Cert:
            Master_SSL_Cipher:
               Master_SSL_Key:
        Seconds_Behind_Master: 0
Master_SSL_Verify_Server_Cert: No
                Last_IO_Errno: 0
                Last_IO_Error:
               Last_SQL_Errno: 0
               Last_SQL_Error:
  Replicate_Ignore_Server_Ids:
             Master_Server_Id: 1
                  Master_UUID: 375dddd-1111-11gg-gggg-f2230geksja
             Master_Info_File: /var/lib/mysql/master.info
                    SQL_Delay: 0
          SQL_Remaining_Delay: NULL
      Slave_SQL_Running_State: Slave has read all relay log; waiting for more up               dates
           Master_Retry_Count: 86400
                  Master_Bind:
      Last_IO_Error_Timestamp:
     Last_SQL_Error_Timestamp:
               Master_SSL_Crl:
           Master_SSL_Crlpath:
           Retrieved_Gtid_Set:
            Executed_Gtid_Set:
                Auto_Position: 0
         Replicate_Rewrite_DB:
                 Channel_Name:
           Master_TLS_Version:
1 row in set (0.00 sec)
```

만약 에러가 발생했다면 `Last_IO_Errno: ` 부분에서 에러코드와 에러메세지가 출력되면서 연동에 실패하는 걸 보실 수 있을 겁니다.

<br>

**두번째** 는 마스터에서 현재 프로세스 리스트를 확인하는 방법입니다. 

```bash
mysql> show processlist\G;
```

<br>

![image](https://user-images.githubusercontent.com/58355531/110958096-ab27b400-838f-11eb-9b07-19ac0b7c035d.png){: .align-center}

해당 명령어를 입력한다면 위와 같은 로그가 출력이 됩니다. 맨 위의 보이는 것이 슬레이브의 접속 정보를 의미합니다. 맨 처음 Replication으로 생성해준
계정이 보이고 `master has sent all binlog to slave; waiting for more updates` 라는 메세지가 보인다면 제대로 마스터로 접속하고 있다는 것을 의미합니다.

<br>

> 제가 진행했을 때 3306 포트를 열어주지 않아서 슬레이브에서 마스터로 전혀 접속을 못한 적이 있습니다. 3306 포트번호가 열려있지 않다면
> 방화벽 설정에서 확인해 꼭!!! 열어주시기 바랍니다!

<br>

### 테이블에 값을 넣어 직접 확인해보기

임시로 `test` 라는 파일을 생성해 더미 데이터를 입력해주었습니다.

<br>

**MASTER**

![image](https://user-images.githubusercontent.com/58355531/110957708-39e80100-838f-11eb-9239-7d512e40bddd.png){: .align-center}

`insert` 구문을 이용해 해당 데이터를 입력한 후 슬레이브로 접속하여 확인해보겠습니다!

<br>

**SLAVE**

![image](https://user-images.githubusercontent.com/58355531/110958183-c4c8fb80-838f-11eb-9be3-5b84a17f305d.png){: .align-center}

`select * from test;` 를 했을 때, Master 와 같은 데이터를 불러옵니다. 이렇게 모든 **MySQL Master/Slave Replication** 구성이 완료되었습니다!!

<br>

> 초기 설정들은 비교적 간단하나, 방화벽이나 계정 권한 문제로 안되는 경우가 종종 발생을 많이 하는 것 같습니다. 연동을 확인해보기 전,
> 3306 포트가 열려있는지, 계정엔 Slave 권한이 있는지 미리 확인해두고 진행하는 것이 좋을 것 같습니다!
> 긴 글 읽어주셔서 감사합니다! 모든 반영사항은 저의 깃허브 레파지토리 참고 바랍니다 :)

<br>
<br>

## Project Github URL 

![오구리이미지](https://user-images.githubusercontent.com/58355531/99896015-085c0480-2cd0-11eb-998d-8b8faeb43e17.gif)

[FESTA 프로젝트 Github 보러가기 Click!](https://github.com/f-lab-edu/event-recommender-festa)

<br>
<br>
<br>

## Referenced by

- MySQL Official Document: Chapter 17 Replication     
  <https://dev.mysql.com/doc/refman/8.0/en/replication.html>      
  
- Real MySQL: 이성욱 지음      
  <https://book.naver.com/bookdb/book_detail.nhn?bid=6886962>
  
<br>
<br>
<br>
<br>
<br>
<br>

