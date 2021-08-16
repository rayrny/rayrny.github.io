---
layout: posts
title: "[JS] CSS float속성과 display: flex"
categories:
  - Javascript
last_modified_at: 2021-08-17
author_profile: true
tags:
  - 웹 프레임워크
  - JavaScript
  - CSS
toc: true
toc_sticky: true
sidebar:
  title: Posts
  nav: "sidebar-contents"
---

### 🤦‍♀️ float: right가 안먹힌다?

이전 프로젝트 할 때 아무 문제 없이 왼쪽 오른쪽 정렬을 했던 것 같은데 button의 float 속성이 아무리 해도 안 먹혔다. 처음에는 ```react-bootstrap```의 버튼을 가져와서 css가 안되는 건가 생각했는데, 그건 아니었다.

정말 한 시간을 넘게 헤매다가 원인을 발견했다. 

#### 1. float 속성은 left, right 모두 적용해야 한다.

------

나는 왼쪽에는 텍스트가 있고 저장하기 버튼을 오른쪽에 두고 싶었다. 그래서 너무 당연하게도 버튼에만 ```float: right```를 해주었다. 이전 프로젝트를 진행할 때는 이런 사실을 몰랐지만 용케 left, right 둘 다 적용해서 쓰고 있던 것이었다.

따라서 텍스트에도 ```float: left```속성을 지정해 주었다.

#### 2. display: flex와 float는 함께 쓰지 않는다.

-----

block 속성을 없애기 위해 ```display: flex```를 종종 사용했는데, 이번에도 당연히 한 줄에 두 요소가 들어간다고 생각하여 부모 요소에 flex를 걸었다. 

이제와서 생각해보니 float 속성이라는게 block 안에서 left, right로 띄우는 건데 ```display: flex```를 함께 쓰다니.. 바보였네


다음부터는 같은 실수 안해야지.