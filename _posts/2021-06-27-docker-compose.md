---
title: "도커 컴포즈(docker-compose)로 개발환경 구축해보기(feat.SpringBoot, MySQL, Mybatis)"    
layout: single    
read_time: true    
comments: true   
categories: 
 - project  
toc: true    
toc_sticky: true    
toc_label: contents    
description: 각각의 도커 컨테이너를 컴포즈로 묶어 본 경험을 정리한 글
last_modified_at: 2021-06-26   
---

<br>
<br>


## Overview

![image](https://user-images.githubusercontent.com/58355531/123458361-ed63c100-d61f-11eb-9669-d53f63e62edd.png){: .align-center}


최근 진행하는 사이드 프로젝트에도 그렇고 여러 기업의 취업을 준비하면서 자주 접하게 된 기술이 있었는데요, 바로 **도커(Docker)** 입니다.
저는 사이드 프로젝트에선 **Redis Cache(6379포트)**, **MySQL(3307포트)** 을 도커로 띄워서 사용하고 있습니다. 로컬에는 이미 6378 포트의
Redis와 3306포트의 MySQL이 설치되어 있기 때문이죠. 도커는 이렇게 여러 대의 프로그램이 필요할 때 유용하게 사용하고 있었습니다.

<br>

![image](https://user-images.githubusercontent.com/58355531/123459290-0f117800-d621-11eb-908a-f6d7c52ea8e8.png){: .align-center}

도커는 원하는 기술의 이미지를 다운로드 받아 컨테이너 형식으로 각각 올려 구동시키는 방식으로 되어있습니다. 저 같은 경우 윈도우 운영체제에서
**Docker for Desktop** 을 다운로드 받아서 사용 중이라 cmd 창에서 올린 컨테이너를 다시 내리고 필요할 때 올리고의 작업이 그렇게(?) 복잡하지는 않았습니다.
UI는 깔끔했기 때문이죠. 하지만 어플리케이션이 올라갈 때 같이 실행되지 않아 일일히 확인해야하는 수고로움이 있었고, 당연히 자주 잊어버리게 되니
갑작스럽게 에러를 본 적이 많았습니다. 

**어플리케이션을 구동할 때 한번에 할 수 있는게 없을까?** 하고 찾아보다가 **docker-compose** 라는 기술을 알게 되었고 꽤나 복잡해 보였지만
한번 구축해 둔다면 도커 환경에서 편안하게 사용할 수 있을 것 같아 적용을 해보게 되었습니다!

<br>
<br>

## 작업환경

> 저는 Spring boot 2.x 버전을 사용 중이고 이외 MySQL, Maven, Mybatis 프로젝트를 기준으로 소개하겠습니다.

- Windows 10
- Spring Boot 2.x
- MySQL 8.0 +
- Redis
- Maven
- Mybatis

이렇게 구성된 프로젝트이니 참고 부탁드립니다 :)

<br>
<br>

## docker-compose.yml

우선 로컬환경에 도커가 모두 설치되어 있다는 가정하에 진행하겠습니다. 여러대의 도커 컨테이너를 묶기 위해서는 **docker-compose.yml** 이라는 
파일을 반드시 작성해야합니다. 여기서 작성된 코드로 여러대의 도커 컨테이너가 묶여 올라가게 되어있기 때문입니다.

```yml
version: '3'

services:
  redis:
    image: redis
    container_name: redis
    ports:
      - 6379:6379
      
  mysql:
    container_name: mysql3308
    image: mysql:8.0.13
    environment:
      MYSQL_DATABASE: demoProject
      MYSQL_ROOT_PASSWORD: test123
      MYSQL_ROOT_HOST: '%'
    ports:
      - "3308:3306"
    restart: always

  app:
    restart: always
    build: ""
    working_dir: /demo
    volumes:
      - ./demo:/demo
      - ~/.m2:/root/.m2
    ports:
      - "8080:8080"
    expose:
      - "8080"
    depends_on:
      - redis
      - mysql
    command: mvn clean spring-boot:run
```

<br>

저는 이런식으로 필요한 컨테이너인 Redis, MySQL, Spring Boot 프로젝트를 작성해주었습니다. 

<br>

### services

사용할 프로그램의 리스트를 `services:` 를 선언한 후 작성해줍니다. 이하에 작성된 프로그램들을 읽어오게 됩니다.

### image

도커에 올려서 쓸려면 우선 사용할 프로그램의 이미지를 다운로드 받아야 합니다. 작성을 안한다면 알아서 가장 최신의 이미지(latest)를 다운로드 받게 되어있습니다.
하지만 만약 특정 버전을 원하는 경우 반드시 명시를 해주어야 합니다.

### container_name

도커에 컨테이너 명을 지정해줄 수 있는 옵션입니다. 안한다면 임의의 이름으로 지정되는데 이름을 지어주는게 더 보기는 좋았던 것 같습니다.

### environment

MySQL의 경우에만 작성을 해주었는데요, 여기에 명시를 해주고 기동하게 되면 최초로 DB가 생성될 때 자동으로 제가 지정한 Root 비밀번호로 계정을 생성하고 `MYSQL_DATABASE` 는 해당 DB 내에
지정한 이름의 데이터베이스를 생성한 채로 컨테이너를 올려줍니다. 모든 IP에서 외부접근을 해야한다면 `MYSQL_ROOT_HOST` 를 **%** 로 지정해주도록합니다.

### ports 

포트번호 맵핑의 경우 저는 현재 3306포트가 로컬에 설치된 MySQL에서 사용중이기 때문에 `3306:3306` 으로 작성하면 에러가 납니다. 이럴경우 로컬에서는 다른 포트에서 접근할 수 있도록 설정이 필요한데 
`:` 왼쪽의 포트번호만 로컬에서 사용중이지 않은 포트로 바꿔주시고 오른쪽은 해당 프로그램의 기본 포트번호`(예: Redis는 6379, MySQL은 3306)` 를 넣어서 연결되도록 합니다. 
만약 오른쪽 포트에 다른 임의의 포트를 넣는다면 도커 내에서 제대로 인식을 못해 외부접근이 되지 않습니다. 


> ... To be Continue
