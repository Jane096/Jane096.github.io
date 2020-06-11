---
title: "Github로 블로그 개설하기"    
layout: single    
read_time: true    
comments: true   
categories: 
 - github  
toc: true    
toc_sticky: true    
toc_label: contents    
description: Github로 블로그를 개설하고 글 작성해서 확인하는 방법, 그리고 jekyll theme, ruby 설치하기(+markdowm 연습)  
last_modified_at: 2020-06-11  
---

개발자들 사이에서 개인블로그로 많이 활용되고 있는 Github pages 개설 방법과 jekyll 테마 적용, 환경 설정의 어려움을 어떻게 해결했는지 공유하는 게시물 입니다!
물론, github 아이디가 있다는 가정 하에 올리겠습니다 :)  
<br>

## 1. Create New Repository
![New Git repository생성이미지]({{ site.url }}{{ site.baseurl }}/assets/start_github/repository.PNG){: .align-center}  
내 계정에서 New를 클릭해 새로운 Repository를 생성합니다. 
<br>

![create repository 화면]({{ site.url }}{{ site.baseurl }}/assets/start_github/repositorycreate.PNG){: .align-center}    
신규 Repository를 생성할 때, 꼭 **{Git ID}.github.io** (또는 .com)으로 지정을 해야합니다.  
그리고 initialize this repository with README.md를 클릭하고 create를 클릭합니다.  
<br>

![setting 이미지]({{ site.url }}{{ site.baseurl }}/assets/start_github/setting.PNG){: .align-center}  정상적으로 생성이 되었다면 setting으로 들어가서 밑으로 쭉 내려보면
<br>

![setting 이미지]({{ site.url }}{{ site.baseurl }}/assets/start_github/publishedsetting.PNG){: .align-center}  
이 처럼 https://{내 아이디}.github.io (또는 .com)의 주소로 활성화 되었다는 것을 알려줍니다.  
주소창에 해당 url을 친다면 테마 적용 전에는 README.md의 내용이 그대로 뜨게 됩니다.  
잘 보인다면 성공한겁니다!
<br>

## 2. Repository를 내 PC로 가져와 작업하는 환경 만들기
Github page를 쓰게 되면 어쩔 수 없이 내 로컬 pc에서도 작업해야 하는 경우가 있습니다. Git만 설치되어있다면 간단하게 설정이 가능합니다.    

**Please Note:** Git이 아직 설치되어있지 않다면 꼭 설치 해주세요!
{: .notice--danger}
<br>

![clone repo 이미지]({{ site.url }}{{ site.baseurl }}/assets/start_github/clonerepo.PNG){: .align-center}  
오른쪽 상단에 clone이라는 초록색 버튼을 클릭하면 현재 repository의 web url이 뜨게 됩니다. 이 주소를 복사해주세요.    


저는 c드라이브 밑에 workspaces라는 폴더에 만들 예정이라 workspaces 폴더(원하는 폴더 아무거나 ok)에 오른쪽 키를 누르면 Git Bash here 라는 글자가 뜹니다. 클릭하면 아래와 같은 erminal 창이 나오게 됩니다.


![git terminal이미지]({{ site.url }}{{ site.baseurl }}/assets/start_github/terminal.PNG){: .align-center}    

```java
$ git clone [아까 복사한 web url]
```
위와 같은 코드를 입력해주세요.  
<br>

![git folder 이미지]({{ site.url }}{{ site.baseurl }}/assets/start_github/folder.PNG){: .align-center}  
성공했다면 이렇게 폴더 하위에 {Git ID}.github.io 라는 폴더가 생성되게 됩니다.  
이제 여러분의 pc에서 원격으로 변경사항을 push 하거나 pull을 할 수 있습니다.  
<br>

## 3. jekyll Theme Download
구글에 jekyll theme를 치면 굉장히 많은 템플릿들이 나오는데 그 중에서 맘에 드는 것 하나를 골라주면 됩니다:)  

저는 **minimal-mistakes**라는 테마를 골라 적용해 주었습니다. 우선 디자인이 매우 보기 좋게 깔끔해서 고르게 되었네요ㅎㅎ  
주소는 [https://github.com/mmistakes/minimal-mistakes]({{"https://github.com/mmistakes/minimal-mistakes"}}){:target="_blank"}로 가서 다운로드 하면 됩니다.<br>


![테마 clone 이미지]({{ site.url }}{{ site.baseurl }}/assets/start_github/minimalmistakezip.PNG){: .align-center}  
저는 해당 테마의 github 페이지로 가서 Download ZIP을 해주었습니다. <br>


![테마 압축 후 적용 이미지]({{ site.url }}{{ site.baseurl }}/assets/start_github/clonetheme.PNG){: .align-center}  
해당 파일이 다운로드 된 폴더로 가서 압축해제를 한 후 생성된 github.io폴더 하위에 모두 옮겨주세요.  
위의 pages폴더와 posts폴더는 처음 압축해제 할 때 있는 docs파일 안에 있습니다. docs파일은 sample 게시물들이 들어있기 때문에 미리 다른 곳으로 옮겨 준 후 docs파일 밑에 pages와 posts 두 개의 폴더만 내 github.io폴더에 옮겨주도록 합니다.<br> 


## 4. 블로그 개발 환경을 위한 Ruby 설치
우리가 사용한 jekyll은 ruby라는 스크립트 언어로 작성되어 있기 때문에 로컬 개발 환경을 위해서 루비의 설치가 필요합니다.  
[https://rubyinstaller.org/downloads/]({{" https://rubyinstaller.org/downloads/ "}}){:target="_blank"}에 가서 제일 진한 글씨로 표시되어있는 버전을 클릭해 다운로드 받았습니다. <br> 


![rubyinstall 화면]({{ site.url }}{{ site.baseurl }}/assets/start_github/rubyinstall.PNG){: .align-center} 
다운로드 받은 rubyinstaller 파일을 실행해 그대로 설치하기만 하면 됩니다. <br>


![루비 설치확인 이미지]({{ site.url }}{{ site.baseurl }}/assets/start_github/Inkedrubyinstallok_LI.jpg){: .align-center} <br>
루비 설치가 완료되었다면 cmd창에서 **ruby -v**를 입력해 버전이 뜨는지 확인합니다.  
잘 뜬다면 설치가 완료되었다는 의미입니다.  <br>
<br>

### cmd에서 jekyll bundler 설치하기
설치가 완료되었다면 cmd창을 열어 github.io 폴더로 이동을 합니다. **해당 폴더 안에 Gemfile이 있는지 확인합니다!**<br>


- 그리고 다음 명령어를 이용해 jekyll bundler를 설치합니다.  
```java
cd c:\workspaces\github.io폴더
gem install bundler
```
<br>
- 설치가 완료되었다면 다음 코드를 입력해 localhost에서 github 블로그가 뜨는지 확인합니다.  
```java
bundle exec jekyll serve
```
<br>
- 정상적으로 실행 된다면 cmd창 맨 밑에 아래의 메시지가 뜨게 됩니다. 그러면 조금 후에 주소창에서
localhost:4000을 치면 내 블로그가 올라간 것을 확인할 수 있습니다.  
```java
Server address: http://127.0.0.1:4000
Server running... press ctrl+c to stop.
```
<br>


### 테마 주소 변경하기(config.yml)
config.yml은 블로그의 이름이나 url, posts와 pages의 설정을 담당합니다.
저는 밑에와 같이 변경해주었습니다.
<br>

```java

# remote_theme           : "mmistakes/minimal-mistakes"
minimal_mistakes_skin    : "default" # "air", "aqua", "contrast", "dark", "dirt", "neon", "mint", "plum", "sunrise"

# Site Settings
locale                   : "en-US" #ko-KR로 해도 상관없음(한글로 뜸)
title                    : "Java Developer`s Blog" # 블로그 타이틀
title_separator          : "|"
subtitle                 : "Version 2.0" # 블로그 타이틀 밑에 나오는 조그만 글씨
name                     : "JE"
description              : "Java/Spring/Spring Boot/Server"
url                      : "https://Jane096.github.io/" # github 호스트 주소 입력하기
baseurl                  : # the subpath of your site, e.g. "/blog"
repository               : # GitHub username/repo-name e.g. "mmistakes/minimal-mistakes"
teaser                   : # path of fallback teaser image, e.g. "/assets/images/500x300.png"
logo                     : # path of logo image to display in the masthead, e.g. "/assets/images/88x88.png"
masthead_title           : "Recording life" # 홈페이지 왼쪽 최상단에 들어갈 타이틀 글씨
breadcrumbs              : true # true, false (default)
words_per_minute         : 200
.
.
.
.
# Site Author
author:
  name             : "JE" # footer에 2020.(name) 으로 뜨게 됨
  avatar           : "/assets/image/photo.PNG" # 프로필 사진
  bio              : "Back-End 개발자를 위해 꾸준히 공부 중!" # 프로필 사진 밑에 설명부분
  location         : "Seoul, Korea"
  email            : "" # 내 이메일 입력하면 됨
  links:
    - label: "Email"
      icon: "fas fa-fw fa-envelope-square"
      #url: ""
    - label: "Website"
      icon: "fas fa-fw fa-link"
      # url: "https://your-website.com"
    - label: "Twitter"
      icon: "fab fa-fw fa-twitter-square"
      # url: "https://twitter.com/"
    - label: "Facebook"
      icon: "fab fa-fw fa-facebook-square"
      # url: "https://facebook.com/"
    - label: "GitHub"
      icon: "fab fa-fw fa-github"
      url: "https://github.com/Jane096"
    - label: "Instagram"
      icon: "fab fa-fw fa-instagram"
      # url: "https://instagram.com/"

# Site Footer
footer:
  links:
    - label: "GitHub"
      icon: "fab fa-fw fa-github"
      url: "https://github.com/Jane096"
.
.
.
# Defaults
defaults:
  # _posts
  - scope:
      path: ""
      type: posts
    values:
      layout: single
      author_profile: true
      read_time: true
      comments: true
      share: false
      related: true
      
# _pages
  - scope:
      path: "_pages"
      type: pages
    values:
      layout: single
      author_profile: true
      comments: true
```
이 외 없는 부분은 변경한 것이 전혀 없기 때문에 새로 추가한 부분만 옮겨 보았습니다 ㅎㅎ  
url부분에 github호스트 주소를 정확하게 입력해주세요!<br>


- 맨 하단의 posts는 블로그 포스팅 파일을 모아둔 곳으로 이해하면 됩니다. 
default로 적용할 layout은 layouts폴더 밑의 single.html을 적용하겠다는 의미입니다. 
- pages 는 밑에 설정할 navigation과 관련이 있습니다.<br>
<br>

### navigation 설정하기
기본으로 적용된 테마에는 Categories, Tag, About만이 노출되어 있기에 커스터 마이징을 위해 _config.yml의 _pages와 _data/navigation.yml을 이용하도록 하겠습니다. 


```java
# main links
main:
   - title: "Categories"
     url: /categories/
   - title: "About"
     url: /about/
   - title: "Annual Post"
     url: /posts/
```
navigation.yml에 이렇게 구성을 하였고, 마음대로 더 늘리거나 줄여도 상관없습니다.  
주의할 점은 띄어쓰기와 줄 맞춤입니다. url과 title줄이 맞춰지지 않거나 띄어쓰기를 제대로 안하면 에러가 발생하더군요 ㅠ<br>
<br>

### pages에 필요한 .md파일 생성
docs파일 안에 많은 .md파일들이 기본적으로 들어있어 그냥 사용해도 무방하나 새로 추가를 한다면 title, layout, permalink는 꼭 설정해주어야 합니다.   
(저는 annual post부분만 하나 새로 생성해주었습니다.)<br>

```java
---
title: "Annual Post"  
layout: posts   
permalink: /posts/ 
author_profile: true  
---  
```
<br>

layout은 앞서 설명했듯이 layouts폴더 하위의 있는 html 파일 중 하나를 선택해 사용해야 하고 title, permalink는 navigation.yml에 설정해둔 title, url과 일치해야 합니다. permalink url을 보고 해당 페이지에 layout을 적용해 화면에 보여지게 됩니다. <br><br>



## 6. 테마 적용 확인해보기 
다시 git bash here를 실행해(**반드시 github.io폴더에서 실행할 것!**) 원격으로 한꺼번에 올려주도록 합니다.  

```java
$ git add .
$ git commit -m "first commit"
$ git push
```
<br>
이렇게 순서대로 입력해준다면 아래와 같이 변경사항이 모두 반영되어있는 걸 확인할 수 있습니다.
<br>
![변경사항 이미지]({{ site.url }}{{ site.baseurl }}/assets/start_github/themechange.PNG){: .align-center}  
<br>

### github.io 호스트에서 확인해보기
주소창에 https://{Git ID}.github.io 에서도 커스터마이징이 모두 완료되었는지 확인합니다.
아래와 같이 뜬다면 개설 성공!!!! 만약에 posts파일 밑의 내용을 아무것도 지우지 않았다면 sample 파일들이 잘 떠있을 겁니다! 저는 다 지워버려서 없네요 ㅎㅎ 
<br>
![최종 이미지]({{ site.url }}{{ site.baseurl }}/assets/start_github/final.PNG){: .align-center} <br>
<br>
<br>
