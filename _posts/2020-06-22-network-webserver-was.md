---
title: "[네트워크]Web Server와 WAS의 대한 정리"
layout: single    
read_time: true    
comments: true   
categories: 
 - network  
toc: true    
toc_sticky: true    
toc_label: contents    
description: web server와 was의 차이점인데 좀더 디테일하게 정리를 위함
last_modified_at: 2020-06-22   
---   

기술면접을 준비하기 위해서 Web Server와 WAS에 대해 간단하게만 알고 있었는데 오늘 근무를 시작해 
교육을 받으면서 이렇게 얕은 지식으로는 코드 내용 파악이 힘들 것이라고 판단해 내용을 정리해보았다.   
<br>

## Web Server & WAS?
- **웹 서버**는 정적 컨텐츠(html, css, js, pdf도 포함될 수 있다고 한다!)를 제공하기 위한 서버이다. 
- **WAS**는 DB조회가 필요하거나 요청받았을 때 결과가 매번 바뀔 수 있는, 동적 컨텐츠를 제공하는 서버이다.
  .jsp가 대표적인 동적 컨텐츠가 되겠다. 
<br>
<br>

WAS가 정적 컨텐츠를 대부분 제공하고 있기 때문에 web server는 WAS에 포함될 수 있는 개념이다. 
<br>

여기까지가 내가 익히 알고 있던 내용들이었다. 하지만 이걸 왜 사용해야 하는지, 정확하게 둘의 차이점은 뭔지에 대해
제대로 생각해보진 않은 것 같다. 
<br>
<br>
<br>

## Web Server가 사용하는 이유?
1) WAS의 부담을 줄이기 위해서 이다. 
<br>
<br>
![New Git repository생성이미지]({{ site.url }}{{ site.baseurl }}/assets/ready-spring/webserver.PNG){: .align-center}
<br>
WAS앞에 Web Server를 두어 앞단에서 http client로 부터 받은 요청(url)에 대해 정적컨텐츠만을 처리하도록 하고 WAS는 어플리케이션의 로직만 수행하도록 
분배를 시켜 서버의 부담을 줄여주도록 한다. Web Server가 괜히 앞에 있는 것이 아니다. 
<br>
<br>

2) WAS의 환경파일 노출을 막기 위해서다.   
WAS에서는 DB조회나 다양한 로직을 수행하기 때문에 보안에 민감하다. WAS에 들어오는 포트 번호는 web server와 
다르기 때문에 WAS에 들어오는 포트는 방화벽을 추가해 보안을 강화할 수 있다. 
<br>
<br>

아직 내용은 조금 부족하지만 관련 책을 읽거나 인강을 듣고, 내용을 추가해봐야겠다.   
<br>
<br>
<br>
<br>
<br>











