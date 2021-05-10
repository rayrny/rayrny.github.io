---
layout: posts
title: "[Vue] Vue-router에 대해서 알아보자"
categories:
  - Vue
last_modified_at: 2021-05-11
author_profile: true
tags:
  - 웹 프레임워크
  - vue
toc: true
toc_sticky: true
sidebar:
  title: Posts
  nav: "sidebar-contents"
---

## vue router는 컴포넌트들을 매핑하고 어떤 주소로 렌더링 할지를 결정할 수 있다.

설치는 간단하게 ```npm install vue-router```를 통해 할 수 있다.


### 라우터 설치 후..

----

라우터를 설치했다면, 보다 쉬운 관리를 위해 router폴더를 하나 생성한 뒤에 그 안에 index.js파일을 생성하자. 

```
// 폴더구조
src
├── router
│   └── index.js 
├── components
├── views
├── App.vue
└── main.js
```

#### 다음으로,
```main.js```에 라우터를 등록하자. 

```javascript
// main.js
import { createApp } from 'vue'
import App from './App.vue'
// router 등록!
import router from './router'

createApp(App).use(router).mount('#app')
```