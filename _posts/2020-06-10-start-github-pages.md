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

## 1. Create New Repository

**Please Note:** Git이 아직 설치되어있지 않다면 꼭 설치 해주세요!
{: .notice--danger}
## 2. Repository를 내 PC로 가져와 작업하는 환경 만들기 

## 3. jekyll Theme Download
 - 다운로드 -> repository 하위폴더에 압축해제

## 4. 테마 적용하기 위한 Ruby 설치
- ruby 다운로드
- 압축풀어 루비 설치하기 
- git bash에서 jekyll bundler 설치하기
- gem install bundler(git bash -> 디렉토리 위치 repository로 설정 후 할 것)
- bundle exec jekyll theme으로 실행해서 localhost:4000에 server running 되는지 확인

## 6. 테마 적용 확인해보기 
- git add .  
- git commit -m "Theme changed"  
- git push
- https://{ID}.github.io 에서 확인해보기 
