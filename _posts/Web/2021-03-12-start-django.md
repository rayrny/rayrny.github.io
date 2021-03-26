---
layout: posts
title: "[Django] 장고(Django) 시작하기"
categories:
  - Django
last_modified_at: 2021-03-12
author_profile: true
tags:
  - 웹 프레임워크
toc: true
toc_sticky: true
sidebar:
  title: Posts
  nav: "sidebar-contents"
---

### 파이썬 기반으로 작성된 오픈 소스 웹 프레임워크인 장고를 사용해보자.


#### 설치하기
------
먼저 cmd 창을 열어 장고를 설치한다.

```
pip install Django
```
나는 가장 최신 버전을 설치하고자 ```pip install Django==3.1.7```를 입력했다. 이후 조금 기다리면 성공했다는 메시지가 뜬다.

![Django](/assets/image/django1.PNG)

#### 프로젝트 생성
-----

프로젝트를 만들기 위한 명령어는 아래외 같다.

```
django-admin startproject 프로젝트_이름
```

![Django](/assets/image/django2.PNG)

위의 명령어를 입력한 후 ```dir```를 쳐보면 생성한 프로젝트 폴더가 있는걸 확인할 수 있다.

![Django](/assets/image/django2-2.PNG)

#### 서버가 잘 돌아가는지 확인하기
------

이제 ```cd```명령어를 통해 생성한 프로젝트 폴더까지 이동한 후

```
python manage.py runserver
```

를 입력한다.

![Django](/assets/image/django3.PNG)

이후 나오는 결과를 보면 ```http://127.0.0.1:8000```를 확인할 수 있는데, 웹 브라우저 주소창에 ```127.0.0.1:8000 또는 localhost:8000```를 치면 아래의 화면이 나타나고 정상적으로 프로젝트 생성이 되었음을 확인할 수 있다.

![Django](/assets/image/django4.PNG)


#### 앱 만들기

![Django](/assets/image/django_app.PNG)

위의 명령어를 입력하면 프로젝트 내부에 앱을 만들 수 있다.


![Django](/assets/image/django_app2.PNG)
