---
layout: posts
title: "[Git] Git Bash 명령어와 사용법 1"
categories:
  - Github
last_modified_at: 2021-02-09
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

## commit, push와 pull & fetch
이번에는 변경사항을 ```push```, ```pull```하는 방법을 알아보자.


### | git add

#### staging 영역에 올리기
먼저 working directory 상에서의 변경된 내용을 staging 영역에 추가하기 위해,
<mark style='background-color: #f5f0ff'>git add</mark> 명령어를 사용한다.

```
$ git add . // 현재 디렉토리 안의 모든 파일
$ git add 파일_이름 // 특정 파일 이름을 뒤에 적으면 그 파일만 올라간다
```
```add .```을 사용해 모든 파일을 올렸다고 해도 변경되거나 새로 추가된 파일만이 staging영역에 올라간다.

#### 파일 상태 확인
올린 내역을 확인하고 싶다면 ```git status``` 명령어를 사용한다.

![add](/assets/image/git-add.png)

변경된 파일이 있지만 아직 staging 영역에 추가 되지 않았을 경우 빨간색으로 표시되고, add를 하고 나면 초록색으로 바뀌는 것을 확인할 수 있다.

![add](/assets/image/git-status.png)

#### git add 취소
만약 파일 상태를 Unstage로 변경하고 싶다면 ```git reset HEAD```명령어를 사용한다.
(뒤에 파일 이름을 적으면 해당 파일만 내릴 수 있다.)
```
$ git reset HEAD
```

![add](/assets/image/git-status2.png)

### | git commit
staging영역에 올라간 파일들은 commit 메시지를 적고 나면 push 할 준비가 끝난다.

```
$ git commit -m "커밋 내용"
```

### | git push
commit message를 적은 후에는 push하고자 하는 브랜치에 푸쉬할수 있다.

```
$ git push origin 브랜치_이름
```
여기까지 하면 푸쉬가 끝난다. 이후 리포지토리에 들어가 푸쉬가 잘 됐는지 확인해 보자.


### | git pull

```pull = fetch + merge```이므로 나는 pull을 더 자주 사용한다.
