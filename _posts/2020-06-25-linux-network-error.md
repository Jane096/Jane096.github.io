---
title: "Linux CentOS ver.7 네트워크 연결 오류"
layout: single    
read_time: true    
comments: true   
categories: 
 - Linux  
toc: true    
toc_sticky: true    
toc_label: contents    
description: Linux centOS 7 기준 부팅 후 네트워크 연결 못하는 오류 해결방법
last_modified_at: 2020-06-25   
---   
웹 서버를 구축할 때 대부분 사용하는 Linux를 하이퍼 V 라는 가상환경에서 돌릴 때 
네트워크가 잘 되다가 다시 부팅 했을 때 갑자기 연결하지 못하는 오류를 접했다. 
<br>
<br>

흔히 리눅스 쉘에서 뭔가를 설치하거나 다운 받을 때 **yum**을 쓰는데 이게 동작하지 않는다는 것은 
현재 리눅스가 네트워크 연결을 못한다는 뜻이다. 
![네트워크 오류 이미지]({{ site.url }}{{ site.baseurl }}/assets/linuxnetwork/networkch.PNG){: .align-center}
<br>

```bash
ping 8.8.8.8 
```
<br>

또 다른 방법으로는 위의 코드를 입력하는 것인데, 밑의 사진처럼 출력되지 않는다면 이것 또한 
네트워크 연결 오류로 볼 수 있다. 
<br>
![ping테스트]({{ site.url }}{{ site.baseurl }}/assets/linuxnetwork/pingtest.PNG){: .align-center}
<br>
<br>

오류 해결을 위해 코드블럭 안에 코드를 순서대로 입력하자
<br>

```bash
ip addr
```
![ip addr 콘솔]({{ site.url }}{{ site.baseurl }}/assets/linuxnetwork/ipaddr.PNG){: .align-center}
<br>
**2.eth0**이라는 부분을 잘 확인하자
<br>
<br>

```bash
ifup
```
<br>
![연결성공]({{ site.url }}{{ site.baseurl }}/assets/linuxnetwork/ifup.PNG){: .align-center}
<br>
**연결이 활성화 되었습니다** 메세지가 잘 뜨는지 확인하도록 한다.
<br>
<br>

다시 처음 처럼 **ping 8.8.8.8**을 입력하여 밑의 사진처럼 잘 출력되는지 확인해보자 
<br>
![다시 ping test]({{ site.url }}{{ site.baseurl }}/assets/linuxnetwork/pingtest.PNG){: .align-center}
<br>
<br>
<br>
<br>
<br>




