---
layout: posts
title: "[Git] Git Bash 명령어와 사용법"
categories:
  - Github
last_modified_at: 2021-02-08
author_profile: true
tags:
  - jekyll
  - minimal-mistakes
toc: true
toc_sticky: true
sidebar:
  title: Posts
  nav: "sidebar-contents"
---

## git bash에 대해서 알아보자!

깃에 소스코드를 올리거나 수정할 때 직접 upload를 하는 방법도 있지만, 협업을 하거나 관리해야 할 자료들이 많아지면 ```Git Bash```나 ```Source Tree```를 사용하는것이 좋다.

소스트리는 작년에 설치하는 도중 내 노트북의 유저 이름이 한글이라 오류가 난 적이 있어 깃 배쉬를 사용하기 시작했다.

### | git config

처음 깃 배쉬를 설치하면 계정 설정을 해주어야 한다.

```
git config --global user.name "your_name"

// 깃허브에 등록한 이메일을 적어주면 된다.
git config --global user.email "your_email@example.com"
```

### | git init

로컬 저장소를 생성하고 이를 깃 리포지토리에 원격으로 연결하는 작업이다.

프로젝트 폴더를 새로 생성하거나, 이미 에디터를 통해 작업중이던 코드가 있다면 그 코드가 위치한 폴더를 지정하면 된다.

```
$ cd C:/git
```
먼저 cd 를 통해 지정하고자 하는 폴더로 이동한다.

![cd](/assets/image/git-init.PNG)

*주의!*
역슬래쉬(\)는 인식하지 못하므로 ```/```로 변경해 주어야 한다.

git remote add origin 리포지 주소
git remote -v >> 주소 확인

git branch (이름 적으면 바로 생성됨)
마스터에 바로 푸쉬할거면 필요 없음

브랜치 이동하는거
add. status commit push 이거 정리
풀도 정리
풀리퀘 하는것도..
