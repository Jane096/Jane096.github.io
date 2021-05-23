---
title: "Datadog APM으로 내 프로젝트 모니터링 하기"    
layout: single    
read_time: true    
comments: true   
categories: 
 - project  
toc: true    
toc_sticky: true    
toc_label: contents    
description: Datadog APM을 연결해서 모니터링한 과정을 정리한 포스팅
last_modified_at: 2021-05-24   
---

<br>
<br>

## Overview

[지난 포스팅]() 과 마찬가지로 최근에도 성능테스트 및 개선작업을 진행 중이었습니다. 리눅스 우분투에서 `htop` 명령어로 cpu나 memory 등을 모니터링 하고 있었지만
이런 화면은 저에겐 정확한 병목의 원인을 파악하기가 매우 힘들었습니다.

![image](https://user-images.githubusercontent.com/58355531/116382749-33cfb600-a851-11eb-81e5-7455f0c44c72.png){: .align-center}

<br>

모니터링 현황에는 redis, mysql의 사용 내역도 쭉쭉 올라오지만 성능저하의 원인을 쉽게 파악하지 못했습니다.
그리하여 **Datadog APM** 을 연결하여 복잡하게 얽혀있을 수 있는 성능 문제를 자세하게 모니터링 하기로 결정했습니다.

<br>
<br>

## APM 이란?

**Application Performance Monitoring** 의 약자로 구동 중인 애플리케이션의 대한 성능측정과 에러탐지 등의 정보를 수집해 모니터링하는 툴입니다.
보다 편리성을 위해서 시각화한 **Metrics** 도 지원을 해줍니다.

여러 대의 애플리케이션에 설치가 가능하며 이를 한꺼번에 같은 UI 상에 보여주기 때문에 마이크로서비스 아키텍처 에도 유용하게 사용될 수 있다고 합니다.

<br>
<br>

## APM 연결해보기

### Agent 설치

저는 모든 서버 환경이 Google cloud platform(이하 GCP) 구성되어있습니다. GCP를 사용할 경우 [Datadog](https://www.datadoghq.com/)에서 빠르게 연동할 수 있도록 서비스를 지원하기 때문에
저와 같은 GCP를 사용하신다면 추천드립니다! 2주 동안 무료 Trial이 가능하네요!

처음 Trial을 누르면 웹페이지에서 안내하는 순서에 따라 진행하시면 됩니다. 애플리케이션의 성능을 측정해줄 도구가 필요한데요, 이것을 **Agent** 라고 합니다.
위에서 언급했듯이 Agent는 여러개 설치가 가능합니다. 저는 아직 측정할 서버가 1대 뿐이라 1개만 설치해주었습니다.

![image](https://user-images.githubusercontent.com/58355531/119266383-8ec4b500-bc25-11eb-8e5c-0b89b06a6f80.png){: .align-center}

여러 환경에서의 설치방법을 아주 자세하게 설명해주기 때문에 이 순서만 잘 따라해주신다면 문제없이 Agent를 설치하실 수 있습니다. 저의 애플리케이션 환경은 우분투 이기 때문에
옆에서 알려준 명령어를 그대로 입력해 설치를 진행했습니다.

<br>

### GCP 인스턴스 읽어오기

Agent 설치가 끝났으니 이젠 우리가 구성하고 있는 GCP의 compute engine을 불러와야 합니다. 왼쪽 메뉴에 **Integrations** 에서 Google cloud~ 를 치면 저렇게 
API가 나오는데 내 GCP의 서비스 계정 private key를 넣어 연결시켜주면 됩니다. 

![image](https://user-images.githubusercontent.com/58355531/119266536-29bd8f00-bc26-11eb-9ac1-158b92c718ae.png){: .align-center}

> 서비스 계정 private key를 가져오는 방법은 아래 **Instructions for adding Google cloud project**를 참고해주시면 됩니다. 간단한 과정이기에 이 블로그에선 생략하겠습니다.

<br>

![image](https://user-images.githubusercontent.com/58355531/119266617-7608cf00-bc26-11eb-87b1-eac6834fc6b9.png){: .align-center}

성공한다면 위의 사진처럼 별다른 경고음 없이 추가가 되며 **Install Configuration/Update Configuration** 을 클릭해주면 자동으로 연계된 모든 API까지 
설치를 완료시켜 줍니다. 

<br>

### 연결 확인해보기

![image](https://user-images.githubusercontent.com/58355531/119266812-31316800-bc27-11eb-8460-3db1367a2b47.png){: .align-center}

여기까지 문제가 없었다면 **Infrastructure List** 에서 내 서버를 제대로 모니터링을 해 리스트에 이렇게 보여주게 될겁니다. 

<br>

![image](https://user-images.githubusercontent.com/58355531/119266888-879ea680-bc27-11eb-8beb-042ab8246337.png){: .align-center}

또한 Agent가 해당 서버의 cpu, memory, disk의 정보를 읽어 한눈에 확인할 수 있도록 시각화를 해줍니다. 이전에 `htop` 과는 확실히 다르죠?
하지만 이렇게 한다고 APM 기능까지 보여주지 않습니다. APM 기능을 불러올려면 간단하게 설정파일만 업데이트 해주면 됩니다.

<br>

### datadog.yaml 수정하기

Agent의 모든 설정은 제목의 파일명에서 확인할 수 있습니다. 각각의 설치환경 별로 해당파일의 경로를 모두 소개해주고 있으니 APM Service setup docs를 
꼼꼼히 확인해주시기 바랍니다. 우분투의 경우 `/etc/datadog-agent/datadog.yaml` 에 위치해 있습니다!

`sudo vi /etc/datadog-agent/datadog.yaml` 으로 파일을 열어 쭈욱 밑으로 내린다면 **Traces Configuration** 이라는 부분이 있을 겁니다.

```
***********************************
Traces Configuration
***********************************
#
...중략....
#
# apm_config:
# enable: true
```
<br>

위의 두 줄의 명령어가 보이시나요? 첫 설치라면 이 부분은 모두 주석처리가 되어있을 겁니다. 이 두개의 부분만 주석을 해제해 Agent를 restart 해주도록 합니다.

```bash
systemctl restart datadog-agent
```

<br>

### Tracer 설치하기

[Datadog docs](https://docs.datadoghq.com/tracing/setup_overview/setup/java/?tab=containers) 에 따라서 진행했을 때, 마지막으론 Tracer를 설치해주어야 했습니다.
문서에 소개된대로 `wget -O dd-java-agent.jar https://dtdg.co/latest-java-tracer` 를 입력해 `dd-java-agent.jar` 라는 파일을 받아둡니다. 저는 / 디렉토리에
받아두었습니다.

이제 이 Tracer를 실행해 내 애플리케이션의 jvm을 모니터링 할 수 있도록 실행시켜야 합니다.

![image](https://user-images.githubusercontent.com/58355531/119267458-aa31bf00-bc29-11eb-9005-25d1cc8965b1.png){: .align-center}

이렇게 친절하게 `dd-java-agent.jar` 파일명과 경로, 그리고 모니터링할 내 앱의 jar 파일과 파일경로로 바꾸어 서버 커맨드 창에서 실행시켜주면 됩니다.

```bash
java -javaagent:dd-java-agent.jar -Ddd.logs.injection=true -jar /festa-0.0.1-SNAPSHOT.jar
```
<br>

저는 이렇게 실행시켜 주었습니다 :)

<br>
<br>

## APM을 통해 jvm 모니터링 확인해보기

![image](https://user-images.githubusercontent.com/58355531/119267654-8b7ff800-bc2a-11eb-9121-27616d05d010.png){: .align-center}

이렇게 제 애플리케이션의 jvm 상태, 그리고 모든 프로그램들을 Live로 모니터링 해주는 것을 확인하실 수 있습니다!! 너무 꼼꼼하고 방대한 양을 보여주고 있어
이것 또한 어떤 것을 의미하는지 빠르게 공부가 필요할 것 같습니다..

<br>

![image](https://user-images.githubusercontent.com/58355531/119267678-a05c8b80-bc2a-11eb-9e29-c17e5986dcce.png){: .align-center}

Redis에 대한 것까지 모두 꼼꼼히 모니터링 되고 있습니다.(사진엔 없지만 MySQL도 같이 모니터링 중입니다!)

<br>

![image](https://user-images.githubusercontent.com/58355531/119267729-ce41d000-bc2a-11eb-8da5-b2005c2e9cbb.png){: .align-center}

현재 실행 중인 앱에서 어떤 것이 가장 많은 요청을 받고 있는지, 어떤 동작을 얼마나 수행하는지 아주 세세하게 나오고 있습니다. 이 지표를 보고서
이전 보다 더 명확하게 원인을 파악할 수 있게 되었습니다! 생각보다 오래걸리지 않았고 간단한 과정이니 한번쯤 연결시켜 내 프로젝트의 상태를
측정해보는 것도 좋을 것 같습니다!

글이 길었는데 끝까지 읽어주셔서 감사합니다~!

<br>
<br>

## Project Github URL 

[![오구리이미지](https://user-images.githubusercontent.com/58355531/99896015-085c0480-2cd0-11eb-998d-8b8faeb43e17.gif)](https://github.com/f-lab-edu/event-recommender-festa "이미지를 클릭하면 프로젝트 깃허브를 볼 수 있습니다 :)")

[FESTA 프로젝트 Github 보러가기 Click! (또는 위의 이미지 Click!)](https://github.com/f-lab-edu/event-recommender-festa)

<br>
<br>
<br>

## Referenced by

- Datadog Official documentation : <https://docs.datadoghq.com/getting_started/>
  
<br>
<br>
<br>
<br>
<br>
<br>




