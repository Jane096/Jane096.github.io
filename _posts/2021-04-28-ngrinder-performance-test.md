---
title: "nGrinder를 이용한 api 성능테스트 후기"    
layout: single    
read_time: true    
comments: true   
categories: 
 - project  
toc: true    
toc_sticky: true    
toc_label: contents    
description: 로그인 기능으로 성능테스트 후 작성한 후기글
last_modified_at: 2021-04-28   
---

<br>
<br>

## Overview

최근 [프로젝트](https://github.com/f-lab-edu/event-recommender-festa)를 서버에 배포하는 작업과 젠킨스 CD 까지 적용이 완료되어 본격적으로 **nGrinder** 의 사용법을 익혀 성능테스트 준비를 진행하고 있었습니다. 최근 로그인 기능에 대하여 성능테스트 결과를 얻을 수 있었는데요, 그 과정을 정리하고자 블로그 글을 쓰게 되었습니다. 

<br>
<br>

## nGrinder 란?

네이버에서 제작한 성능테스트 솔루션으로 오픈소스 입니다. 누구나 [nGrinder 공식 Github 페이지](https://github.com/naver/ngrinder/releases)에서 다운로드 받아 사용할 수 있습니다!
스크립트만 만들어 둔다면 불필요한 작업 없이 바로 테스트를 수행해 모니터링까지 가능한 플랫폼입니다.

<br>
<br>

## nGrinder Architecture

![image](https://user-images.githubusercontent.com/58355531/116376659-70001800-a84b-11eb-88f7-8afc1b0968e0.png){: .align-center}

**nGrinder** 는 Grinder 라는 내부 엔진을 사용하며 nGrinder 는 이 엔진을 **controller** 와 **agent**로
감싸 다수의 테스트를 병렬처리할 수 있도록 합니다. **controller** 와 **agent** 는 nGrinder 의 주요 요소들 인데요, 어떤 역할을 하는지 알아보도록 하겠습니다.

<br>

### controller

성능테스트를 위한 웹 인터페이스를 제공하는 역할을 합니다. **agent** 를 관리하는 것이 controller 이며
테스트 통계를 보여주거나 사용자가 스크립트를 생성 또는 변경하는 것을 가능하게 합니다. 

<br>

### agent

스크립트를 기반으로 프로세스를 실행하고 지정한 타겟에 실제로 트래픽을 발생시켜 스레드를 동작시키게 합니다.
Monitoring 모드에서 타겟 시스템의 성능을 모니터링 하는 것도 agent의 역할입니다. 

<br>
<br>

## Script 작성하기 

저는 Google cloud platform에 nGrinder 용 서버를 따로 받아 설치했습니다. 
서버의 스펙은 아래와 같습니다.

> - Google cloud platform e2-medium(vCPU 2개, 4GB Memory)

<br>

controller, agent의 설치 후 해당 url에 접속하면 nGrinder 의 UI를 보실 수 있는데요, 
맨 위에 Script 라는 메뉴를 클릭해 스크립트를 작성해주면 됩니다. 

![image](https://user-images.githubusercontent.com/58355531/116378791-68da0980-a84d-11eb-88d3-b1b4f355e2a5.png){: .align-center}

script 파일명을 입력해주고 테스트할 REST api의 URL을 작성해줍니다. `POST` 요청일 경우 보통 `json`을 Body로 
보내게 되는데요, Headers 에 `application/json` 으로 변경 해준 후, 원하는 값을 입력해주면 됩니다.

<br>
<br>

```java
import static net.grinder.script.Grinder.grinder
import static org.junit.Assert.*
import static org.hamcrest.Matchers.*
import net.grinder.script.GTest
import net.grinder.script.Grinder
import net.grinder.scriptengine.groovy.junit.GrinderRunner
import net.grinder.scriptengine.groovy.junit.annotation.BeforeProcess
import net.grinder.scriptengine.groovy.junit.annotation.BeforeThread
// import static net.grinder.util.GrinderUtils.* // You can use this if you're using nGrinder after 3.2.3
import org.junit.Before
import org.junit.BeforeClass
import org.junit.Test
import org.junit.runner.RunWith

import java.util.Date
import java.util.List
import java.util.ArrayList

import net.grinder.plugin.http.HTTPRequest
import net.grinder.plugin.http.HTTPPluginControl

import HTTPClient.Cookie
import HTTPClient.CookieModule
import HTTPClient.HTTPResponse
import HTTPClient.NVPair

// Uncomment this to use new experimental HTTP client.
// import org.ngrinder.http.HTTPRequest
// import org.ngrinder.http.HTTPResponse
// import org.ngrinder.http.cookie.Cookie
// import org.ngrinder.http.cookie.CookieManager

/**
 * A simple example using the HTTP plugin that shows the retrieval of a
 * single page via HTTP.
 *
 * This script is automatically generated by ngrinder.
 *
 * @author admin
 */
@RunWith(GrinderRunner)
class TestRunner {

	public static GTest test
	public static HTTPRequest request
	public static NVPair[] headers = []
	public static String body = "{\"userNo\":2,\n \"userId\":\"아이디\",\n \"password\":\"비밀번호\"\n }"
	public static Cookie[] cookies = []

	@BeforeProcess
	public static void beforeProcess() {
		HTTPPluginControl.getConnectionDefaults().timeout = 6000
		test = new GTest(1, "34.64.133.153")
		request = new HTTPRequest()
		// Set header datas
		List<NVPair> headerList = new ArrayList<>()
		headerList.add(new NVPair("Content-Type", "application/json"))
		headers = headerList.toArray()
		grinder.logger.info("before process.");
	}

	@BeforeThread
	public void beforeThread() {
		test.record(this, "test")
		grinder.statistics.delayReports=true;
		grinder.logger.info("before thread.");
	}

	@Before
	public void before() {
		request.setHeaders(headers)
		cookies.each { CookieModule.addCookie(it, HTTPPluginControl.getThreadHTTPClientContext()) }
		grinder.logger.info("before. init headers and cookies");
	}

	@Test
	public void test(){
		HTTPResponse result = request.POST("http://34.64.133.153:8080/members/login", body.getBytes())

		if (result.statusCode == 301 || result.statusCode == 302) {
			grinder.logger.warn("Warning. The response may not be correct. The response code was {}.", result.statusCode);
		} else {
			assertThat(result.statusCode, is(200));
		}
	}
}

```

`create` 버튼을 눌러주면 자동으로 이런 스크립트를 작성해주는데요, 오른쪽 위에 `Validation` 이라는 파란색 버튼을 눌러 
스크립트가 오류 없이 돌아가는지를 확인한 후, 파일을 저장해줍니다. 

<br>
<br>

## Performance Test

![image](https://user-images.githubusercontent.com/58355531/116379706-3f6dad80-a84e-11eb-9574-ff002804b6c7.png){: .align-center}

`Performance Test` 메뉴로 이동해서 새로운 테스트를 하나 입력해주도록 합니다. 
이 화면에서 사용자 수를 정해볼 수도 있고, 스레드, 프로세스의 갯수도 모두 지정이 가능합니다. `Target Host` 에는 
테스트 대상 서버를 입력해주고 `Save and Start` 를 입력해주면 본격적으로 테스트를 진행합니다. 

<br>

![image](https://user-images.githubusercontent.com/58355531/116384191-94abbe00-a852-11eb-9bcb-e850cdb31ec2.png){: .align-center}

저는 200명의 사용자가 이용했을 경우 이러한 결과를 얻을 수 있었습니다. 의미있는 결과를 위해 테스트는 약 6분 동안
진행하였습니다. 사용자 수가 늘어날 수록 Errors의 수가 점점 늘어나는 현상을 볼 수 있었습니다. 

최고 TPS는 164를 찍었지만 평균적으로는 TPS 64~80(제가 예상한 것보단 낮은 수치였습니다.) 가량을 보여주었고 심하게 요동치는 그래프로 보아 현재 성능면에선 불안정한 것으로 보였습니다. 

<br>

![image](https://user-images.githubusercontent.com/58355531/116382749-33cfb600-a851-11eb-81e5-7455f0c44c72.png){: .align-center}

`htop` 을 우분투에 설치하여 이 명령어를 이용하면 cpu, memory 사용량을 모니터링할 수 있는데요, 200명 유저일 때 cpu, memory가 98~100%를 찍고 있었습니다. 
아무래도 200명 보다 낮거나 서버의 사양이 높아야지 더 많은 유저들이 이용가능할 것 같다는 결론을 내릴 수 있었습니다. 

생각보다 많은 유저들이 사용할 수 있는 상태가 아님을 알 수 있었고 JVM 튜닝, 기타 성능의 저하를
불러오는 부분(ex.SQL 쿼리튜닝)을 찾아 많은 개선이 필요하다는 것을 알 수 있었습니다! 

<br>
<br>


## Project Github URL 

[![오구리이미지](https://user-images.githubusercontent.com/58355531/99896015-085c0480-2cd0-11eb-998d-8b8faeb43e17.gif)](https://github.com/f-lab-edu/event-recommender-festa "이미지를 클릭하면 프로젝트 깃허브를 볼 수 있습니다 :)")

[FESTA 프로젝트 Github 보러가기 Click! (또는 위의 이미지 Click!)](https://github.com/f-lab-edu/event-recommender-festa)

<br>
<br>
<br>

## Referenced by

- nGrinder 공식 Github 주소: <https://github.com/naver/ngrinder>
  
<br>
<br>
<br>
<br>
<br>
<br>

