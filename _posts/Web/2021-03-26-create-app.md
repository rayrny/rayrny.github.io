---
layout: posts
title: "[Django] 장고(Django) 앱 만들기 및 URL 설정"
categories:
  - Django
last_modified_at: 2021-03-26
author_profile: true
tags:
  - 웹 프레임워크
toc: true
toc_sticky: true
sidebar:
  title: Posts
  nav: "sidebar-contents"
---

### 이번에는 프로젝트 내부에 App을 만들고 url을 설정해보자!


#### 🌱 앱 만들기

-----

![Django](/assets/image/django_app.PNG)

```
python manage.py startapp 앱_이름
```
위의 명령어를 입력하면 프로젝트 내부에 앱을 만들 수 있다.

![Django](/assets/image/django_app2.PNG)

*터미널은 vs코드의 터미널을 사용 (cmd창 아님)


#### URL 설정

-----

앱을 생성한 뒤에는 ```프로젝트 폴더```(이하 루트 폴더)에 등록을 해주고 url을 지정해 주어야 한다.
<br>
_참고로_ 나는 ```'mysite'```라는 프로젝트를 만들었고, 추가로 ```'polls', 'accout'```라는 앱을 만든 상태이다.

```django
# mysite/settings.py

INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'polls.apps.PollsConfig',
    'account.apps.AccountConfig', # 새로 만든 app인 polls와 accout를 추가한다.
]
```

위의 이름은 ```생성한 앱/apps.py```에 들어가면 나와있다. (앱이름Config로 동일함)

다음으로 루트폴더의 urls.py에 경로를 등록하자.

```django
# mysite/urls.py

from django.contrib import admin
from django.urls import include, path

urlpatterns = [
    path('admin/', admin.site.urls),
    path('polls/', include('polls.urls')), # polls에 대한 경로
    path('account/', include('account.urls')), # account에 대한 경로
]
```

path의 첫 번째 인자에 오는 것이 ```localhost:8000/```뒤에 붙는 경로이다. 뒤에 include함수를 적는 것은
url을 효율적으로 관리하기 위함인데, ```account/login```과 같이 앱 단위로 묶어주면 코드도 보기 편하고 관리하기도 쉽다.
<br><br>
따라서 각각의 앱 내부에 ```urls.py```파일을 만들고 해당 앱에서 사용할 경로를 작성하자.
```django
# account/urls.py

from django.urls import path
from . import views

urlpatterns = [
    path('', views.home),
    path('signup/', views.signup, name='signup'),
    path('login/', views.login, name='login'),
    path('logout/', views.logout, name='logout'),
]
```
여기에 적어준 url들을 모두 포함시키는 것이 바로 ```include('account.urls')```이다.


<br>
<small>마크다운 코드블럭에서 장고는 지원을 안해준다 😢</small>
