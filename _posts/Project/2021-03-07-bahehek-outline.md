---
layout: posts
title: "👾 바흐흑을 구해죠바흐흑 - 게임 진행"
categories:
  - WebGame
last_modified_at: 2021-03-07
author_profile: true
tags:
  - JavaScript
  - HTML
toc : true
toc_sticky : true
header:
  teaser: /assets/image/bahehek/teaser.PNG
sidebar:
  title: Posts
  nav: "sidebar-contents"
---

## Let's Play!

> <a href="https://save-the-bahehek.netlify.app/">https://save-the-bahehek.netlify.app/</a>

### 👽 Plot
-----

![시작화면](/assets/image/bahehek/start.PNG)

뮤직 플래닛 행성의 주민인 ‘바흐흑’은 지구로 소풍을 왔다가 집으로 돌아가는 길에 고속 우주선이 고장이 나는 바람에 직접 우주 공간을 헤엄쳐 집으로 돌아가야 한다.

뮤직 플래닛 행성의 주변에는 무수히 많은 '뮤직-홀'이 존재하는데, 이 뮤직홀을 모두 지나쳐야 행성으로 들어갈 수 있다. 

다양한 스테이지로 구성된 뮤직홀을 모두 통과해 ‘바흐흑’이 집으로 돌아갈 수 있도록 도와주자!


### 💫 How to Play

-----

![게임 설명](/assets/image/bahehek/howto.PNG)

맵을 구성하고 있는 화살표를 빠르게 입력해서 스테이지를 깨야 한다.

곳곳에 숨어있는 함정 아이템들이 발동되면 화면에 나타나는 안내를 잘 보고 지시하는 명령을 따라야 한다. 



### 🎶 Do it Together

-----

![플레잉 1](/assets/image/bahehek/map1.PNG)

이는 첫 번째 스테이지의 맵이다! 각 단계의 숫자에 맞추어 맵의 모양을 구성해 플레이어가 몇 단계인지 쉽게 알아볼 수 있도록 했다.


![플레잉 2](/assets/image/bahehek/ing.PNG)

해당 단계를 끝내면 아래의 화면이 나오고, 한번 클릭을 하면 다음 단계로 넘어간다.

![플레잉 3](/assets/image/bahehek/clear.PNG)

-----

20콤보를 달성하면 <mark style='background-color: #f5f0ff'>버닝 타임</mark>이 시작된다.

![플레잉 4](/assets/image/bahehek/combo.PNG)

![플레잉 5](/assets/image/bahehek/combo2.PNG)

-------

플레이어가 지루함을 느끼지 않게, 맵의 화면은 총 세가지로 구성했다.

![플레잉 6](/assets/image/bahehek/map2.PNG)

![플레잉 7](/assets/image/bahehek/map3-2.PNG)

-----

10개의 스테이지를 모두 완료하면 아래의 화면과 함께 게임이 끝난다.

![플레잉 8](/assets/image/bahehek/end.PNG)


-----

프론트엔드 스택으로만 구성된 정적 애플리케이션을 배포하기 위해 ```Netlify```를 사용했다.

간단한 회원가입 후, 코드 파일을 선택해 업로드만 하면 바로 배포가 완료되며, ```domain setting```을 통해 url을 수정할 수 있다.