---
layout: posts
title: "[Vue] This relative module was not found: 오류"
categories:
  - Vue
last_modified_at: 2021-05-14
author_profile: true
tags:
  - 웹 프레임워크
  - vue
  - error handling
toc: true
toc_sticky: true
sidebar:
  title: Posts
  nav: "sidebar-contents"
---

## 간단한 오류를 해결해보자.

간혹 컴포넌트를 import할 경우 위의 에러를 발견할 때가 있다. node-module이 해당 컴포넌트 등을 찾지 못해서 발생하는 오류인데, 확인해야 할 것은

-----

1. 경로를 정확하게 적어서 import했는지(대소문자, views를 view로 적지 않았는지 등)
2. ```import Modal from "../components/Modal";```에서 경로를 ```./components/Modal```로 적진 않았는지
3. node-module 폴더를 삭제하고 다시 npm install 해보기

-----

대부분 1, 2에서 해결 가능할 것으로 생각되며, 나는 2번으로 오류를 해결했다. 루트 컴포넌트인 App.vue에서는 ```./components```로 적어도 돌아갔는데 그 외의 컴포넌트에서는 꼭 점을 두개 적어야 하는건가? (vue3은 @/components로 적어야 함)