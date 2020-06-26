---
title: "Linux CentOS 7 Apache/Tomcat 7 mod_jk로 연동하기"
layout: single    
read_time: true    
comments: true   
categories: 
 - Linux  
toc: true    
toc_sticky: true    
toc_label: contents    
description: Linux에서 tomcat7과 Apache web server 연동하고 에러 해결 정리
last_modified_at: 2020-06-26   
---

하이퍼 v에 설치한 가상환경 Linux CentOS 7 환경 위에서 Tomcat과 Apache를 설치하고 
두 프로그램을 연결해 8080포트 없이 http://(내 리눅스 아이피 주소)/index.jsp 로 바로 뜰 수 있도록 
mod_jk를 활용한 연동 방법에 대한 정리이다.   
<br>
<br>

## JDK 설치 확인 
```bash
java -version
```
<br>

![java 설치확인]({{ site.url }}{{ site.baseurl }}/assets/connection/checkjava.PNG){: .align-center}

터미널에 위의 명령을 입력해서 java 버전이 잘 뜨는지 확인해본다.   
만약에 뜨지 않는다면 jdk 1.8버전으로 설치해주도록 하자  
(이미 설치되어있어서 안하긴 했지만 없다면 구글링...)   
<br>
<br>

## tomcat 7 설치
### 설치확인

안정적인 설치를 위해서 7버전으로 설치를 해주었다.  
<br>

```bash
yum list installed | grep tomcat
```
<br>

![tomcat 설치확인]({{ site.url }}{{ site.baseurl }}/assets/connection/tomcatcheck.PNG){: .align-center}
<br> 

설치가 안되어있는 듯 하니 다운로드 받도록 하자 
<br>
<br>
<br>


### yum install tomcat
yum을 이용해 설치하는 경우도 있고 wget+url 하는 경우도 있는데 
어떤 명령어를 쓰느냐에 따라 설치되는 경로가 다르다. 
<br>

혹시나 /usr/share/ 밑에 tomcat을 설치하고 싶다며 **yum**을 사용하도록 하자
<br>

```bash
yum install -y tomcat*
```
<br>

![yum tomcat 설치]({{ site.url }}{{ site.baseurl }}/assets/connection/yuminstalltomcat.PNG){: .align-center}
<br>

```bash
cd /usr/share/tomcat
ls
```
<br>

cd를 해서 해당 디렉토리로 tomcat파일이 있는지 확인하고 
httpd는 나중에...   
<br>

![tomcat dir확인]({{ site.url }}{{ site.baseurl }}/assets/connection/checkdir.PNG){: .align-center}
<br>
<br>
<br>

### tomcat 방화벽 설정
이렇게 되었다면 8080포트의 방화벽을 설정해 열어주도록 한다.   
<br>

```bash
firewall-cmd --permanent --add-port=8080/tcp
firewall-cmd --reload
```
<br>

![8080방화벽설정]({{ site.url }}{{ site.baseurl }}/assets/connection/firewall8080.PNG){: .align-center}
<br>

**success**가 잘 뜨는지 확인 필수!
<br>
<br>
<br>

### tomcat 실행해보기 

```bash
systemctl enable tomcat
```
<br>

위의 코드를 쳐주면 나중에 **service tomcat start** 명령을 사용할 수 있다.(systemctl start tomcat도 가능)      
일일이 디렉토리 이동해서 ./startup.sh를 굳이 안해줘도 된다!
<br>

![systemctl enable tomcat]({{ site.url }}{{ site.baseurl }}/assets/connection/servicestarttomcat.PNG){: .align-center}
<br>

![tomcat실행확인]({{ site.url }}{{ site.baseurl }}/assets/connection/localhost8080.PNG){: .align-center}
<br>

firefox에서 localhost:8080 해서 위의 고양이가 뜬다면 tomcat 설치 완료이다. 
<br>
<br>
<br>

## Apache(httpd) 설치하기  
### 설치확인
CentOS를 설치할 때 소프트웨어를 어떤 걸 설치했냐에 따라 httpd가 설치되어있는 경우도 있는 것 같은데 
_서버 GUI_로 설치했더니 따로 설치되어있는 것 같지는 않다.  
<br>

```bash
yum list installed | grep httpd
```
<br>

명령어를 입력해도 아무것도 안나온다면 설치가 안되어있다는 의미이다. 
<br>
<br>

### httpd 설치하기 

```bash
yum install -y httpd
```
<br>

명령을 입력해 httpd를 설치해주도록 한다.
<br>

![httpd설치확인]({{ site.url }}{{ site.baseurl }}/assets/connection/check&installhttpd.PNG){: .align-center}
<br>
<br>
<br>

### 80포트, http, https 열어주기 
Apache의 경우 80포트를 사용하기 때문에 해당 포트를 열어주어야 한다. 
<br>

![http열어주기]({{ site.url }}{{ site.baseurl }}/assets/connection/firewallhttp.PNG){: .align-center}
<br>

http와 https포트를 열어 reload 하고 list에 제대로 추가되었는지 확인해보도록 한다.
80포트도 마찬가지로 해주어 밑에 사진처럼 나오면 된다.
<br>

![80포트 오픈]({{ site.url }}{{ site.baseurl }}/assets/connection/firewall80.PNG){: .align-center}
<br>
<br>
<br>

### Apache 실행
```bash
systemctl status httpd
```
<br>

지금 상태에서 친다면 아마 꺼져있는 상태일 것이다. 
<br>

```bash
systemctl start httpd
```
<br>

![Apache 실행확인]({{ site.url }}{{ site.baseurl }}/assets/connection/localhost.PNG){: .align-center}
<br>

Apache 설치완료!
<br>
<br>
<br>

## mod_jk 설치하기
### 컴파일러와 필요한 라이브러리 설치

mod_jk를 설치하는 과정에서 필요한 컴파일러들이 있다. 
한꺼번에 설치해주도록 한다. 

<br>

```bash
yum install gcc gcc-c++ httpd-devel
```
<br>

![컴파일러 설치]({{ site.url }}{{ site.baseurl }}/assets/connection/installcompiler.PNG){: .align-center}
<br>
<br>
<br>

### mod_jk 다운로드
구글에 **modjk download**를 검색해서 tar.gz 파일의 링크를 복사해 
wget 명령을 통해 받아준다.   
<br>

```bash
wget -c 링크복사
```
<br>

![connector 다운로드]({{ site.url }}{{ site.baseurl }}/assets/connection/downloadconnector.PNG){: .align-center}
<br>
<br>
<br>

### connector 압축해제

```bash
tar zxvf tomcat-connector*
```
<br>

다운로드 받은 connector의 압축을 해제해준다. 
<br>

![connector 압축해제]({{ site.url }}{{ site.baseurl }}/assets/connection/zipdisabled.PNG){: .align-center}
<br>
<br>
<br>

### 컴파일 하기

압축해제가 정상적으로 되었다면 이제 다운로드 받은 라이브러리를 통해 컴파일이 필요하다. 
ls를 통해 압축해제 되었는지 확인하고 cd 해서 native 디렉토리로 간다. 
<br>

![native 디렉토리]({{ site.url }}{{ site.baseurl }}/assets/connection/connectorcompile.PNG){: .align-center}
<br>

```bash
./configure --with-apxs=/usr/bin/apxs
```
<br>

나중에 할 make 파일을 생성하기 위해서 위의 디렉토리로 configure해주도록 한다. 
완료되었다면 make를 입력해준다 
<br>

![make]({{ site.url }}{{ site.baseurl }}/assets/connection/makeconnector.PNG){: .align-center}
<br>

make가 컴파일을 실행한다. 
컴파일을 완료하면 install을 해주도록 한다. 
<br>

![makeinstall]({{ site.url }}{{ site.baseurl }}/assets/connection/makeinstall.PNG){: .align-center}
<br>

install이 정상적으로 실행된 후 cd하고 해당 디렉토리에 **mod_jk.so**파일이 생성되어있는지 확인한다!
<br>

![modjk설치확인ㅇ]({{ site.url }}{{ site.baseurl }}/assets/connection/checkmodjk.PNG){: .align-center}
<br>

이렇게 보인다면 설치, 컴파일 모두 정상적으로 실행되었다는 의미이다.
<br>
<br>
<br>

### mod_jk.so Selinux 보안설정

```bash
chcon -u system_u -r object_r -t httpd_modules_t /etc/httpd/modules/mod_jk.so
```
<br>

mod_jk연동하는 과정에서 보안관련해 안되는 경우가 많이 발생하는데 애초에 권한을 부여해 이후 에러를 발생시키지 않도록 한다. 
<br>
<br>
<br>

## Apache 설정하기
### httpd.conf에 LoadModule 추가하기

```bash
vi /etc/httpd/conf/httpd.conf
```
<br>

httpd.conf파일로 들어가 편집을 해줘야 한다. 
/LoadModule을 검색해서 해당 부분으로 이동을 해준다. 
<br>

![LoadModule 추가]({{ site.url }}{{ site.baseurl }}/assets/connection/loadModule.PNG){: .align-center}
<br>

노란색 형광펜 부분을 입력해주도록 한다. 오탈자가 나지 않게 조심하자.....
<br>
<br>
<br>

### mod_jk.conf 파일 생성하기
사진 맨 밑에 보면 Include.....*.conf 라고 쓰여 있는 부분이 있을 것이다. 
conf.modules.d경로가 include 된걸 확인할 수 있는데 해당 경로로 진입해 mod_jk에 대한 
새로운 설정파일을 만들어주어야 한다. 
<br>

```bash
vi /etc/httpd/conf.modules.d/mod_jk.conf
```
<br>

![modconf]({{ site.url }}{{ site.baseurl }}/assets/connection/modconfadd.PNG){: .align-center}
<br>

새로 생성된 파일에 똑같이 입력해주면 된다. wq하고 저장을 잊지말자
<br>
<br>
<br>

### workers파일 생성하기 
mod_jk.conf에서 workers.properties를 설정했었으니 해당 파일을 만들어야 한다. 
<br>

```bash
vi /etc/httpd/conf/workers.properties
```
<br>

![workers 파일 설정]({{ site.url }}{{ site.baseurl }}/assets/connection/addworkers.PNG){: .align-center}
<br>

이거 역시 오탈자에 주의하도록 하자 
<br>
<br>
<br>

### Apache와 Tomcat의 document 위치 설정하기 
이제 httpd.conf에서 두 프로그램의 위치를 설정해 최종 연동에 필요한 정보를 입력한다. 
(여기서 오타내서 나중에 연결안됐으니 주의.....)
<br>

![디렉토리권한]({{ site.url }}{{ site.baseurl }}/assets/connection/serdocumentroot.PNG){: .align-center}
<br>

tomcat이 설치된 경로에 ROOT에게 Directory 권한을 추가해주면 된다.
그런데...(처음에 여기를 잘못침)
<br>
<br>
<br>

### selinux 설정해주기
이제 최종결과를 확인해주기 위해서 보안설정을 해줍니다.
<br>

```bash
chcon -R --type=httpd_sys_rw_content_t /usr/share/tomcat/webapps/ROOT
setenforce 0
```
<br>

이렇게 하고 service tomcat stop -> service tomcat start   
그다음 service httpd stop -> service httpd start 하면 되는데...
<br>
<br>
<br>

## 오류 해결
<br>

![에러]({{ site.url }}{{ site.baseurl }}/assets/connection/apacherestarterror.PNG){: .align-center}
<br>

아..뭐가 문제인걸까요 ㅠ
systemctl status httpd를 확인해보도록 하자 
<br>

![에러가뭐냐]({{ site.url }}{{ site.baseurl }}/assets/connection/whatiserror.PNG){: .align-center}
<br>

확인해보니 httpd.conf파일의 132번째 줄 syntax error라는데 뭔가 철자오류인듯 하다..
파일 편집기를 열어봤더니 Directory 권한 줄때 Require 를 Requre로 i를 빼고 썼다.
<br>

에잇ㅋㅋㅋㅋㅋㅋㅋㅋㅋㅋ
<br>

고치고 다시 restart해본다 
<br>

![오류고침]({{ site.url }}{{ site.baseurl }}/assets/connection/fixerror.PNG){: .align-center}
<br>

된다!!!!!
<br>
<br>
<br>

## index.jsp 윈도우에서 확인해보기
Apache, Tomcat 모두 실행이 완료되었다면 우선 리눅스에서 **localhost/index.jsp** 를 입력해준다 
8080없이 잘 뜬다면..? 윈도우 창에서 입력해보자
<br>

```bash
리눅스 아이피주소/index.jsp
```
<br>

![index.jsp 확인]({{ site.url }}{{ site.baseurl }}/assets/connection/localhost_index.PNG){: .align-center}
<br>

고양이가 나온다면 연동이 완료된것! 
엄청난 삽질 끝에 드디어 연동에 성공했다 ㅠㅠ
감동적이다.....
<br>
<br>
<br>
<br>
<br>
















































