---
layout: posts
title: "🪐 [WEB] 자네, 프로필에 달 한 번 띄워볼 생각 없나?"
categories:
  - Web
last_modified_at: 2022-06-02
author_profile: true
tags:
  - 웹
  - 오픈소스
  - moon.svg
toc: true
toc_sticky: true
sidebar:
  title: Posts
  nav: "sidebar-contents"
---


## 두 번째 오픈 소스 컨트리뷰트: moon.svg

회사 리더인 마크의 토이 프로젝트인 [moon.svg](https://moon-svg.minung.dev/)! 실시간으로 달의 모양을 보여주는 사이트이고, github 프로필에도 해당 달의 모양을 걸어둘 수 있다. 물론 특정 날짜를 지정해서 원하는 모양의 달도 보여줄 수 있다. 

<img width="859" alt="스크린샷 2022-06-02 오후 10 30 14" src="https://user-images.githubusercontent.com/48341341/171640615-402aa5b1-b636-4973-872f-7af9dada850d.png">

평소 달, 우주를 좋아했던 나는 moon.svg가 개발 완료되자 바로 내 프로필에 걸어서 사용했는데, 달 모양을 좀 더 달처럼 만들어보고 싶어졌다.. ㅋㅅㅋ


### 오픈 소스는 이런 맛에 오픈 소스니까!
퇴근 후 짬짬히 시간을 내서 코드를 파악하고 원하는 모습을 그려나갔다. 사실 단순하게 투명도가 있는 달 음영 이미지를 위에 덮어 씌우고, 실제로 svg가 그려지는 영역만큼 `clip-path` 하려고 했는데 역시 모든 일은 맘처럼 흘러가지 않는법이었다..

**고려해야 할 점 첫 번째**
나는 궁극적으로는 달의 색상도 변경할 수 있도록 하고 싶었다.
때문에 svg를 포기할 수 없었다.

**고려해야 할 점 두 번째**
svg로 clip-path 하면 달의 모양대로 클립이 되지 않는다. 🥲 png로 하면 깔끔했을 텐데..

그래서 나온 모양이 지금과 같은 배경 전체가 달의 음영으로 존재하고, 그 위에 실제 달 영역이 그려지는 형태였다.
생각보다 간단한 코드였지만 퇴근 후 짬을 내서 노트북을 연다는 것이 쉽지많은 않았고, 몇 주가 지난 후에야 첫 번째 pr을 날렸다. 

이후, 코드 리뷰를 통해 `open-closed 원칙`을 적용한 구조에 맞추어 하나의 파일에서는 하나의 테마만을 관리할 수 있도록 수정했다.
<img width="944" alt="스크린샷 2022-06-02 오후 10 36 02" src="https://user-images.githubusercontent.com/48341341/171641736-ad069784-afa1-4a89-a17c-3a18218b91df.png">

또, 처음에는 테마 이름이 shadow였지만 마크의 추천으로 회사에서 사용하고 있는 이름인 ray로 테마 이름을 변경했다. 뿌듯하다!! ✌🏻

이전 포스팅에서도 말했듯이, 오픈 소스 컨트리뷰팅은 정말 모든 곳에 열려있고 마음만 먹으면 쉽게 시도할 수 있다. 내가 맞추어놓은 환경에서 익숙하게 코딩하는 것도 물론 좋지만, 나같은 경우에는 컨트리뷰팅을 하며 다양한 구조를 경험하고, 삽질하는 과정을 통해 조금씩 알아갈 수 있는 점도 좋다.

이 포스팅을 보고 있는 누군가, 혹시 달을 좋아한다면 자네 프로필에 [달](https://moon-svg.minung.dev/) 한 번 달아볼 생각 없나?



### 추가 이야기
