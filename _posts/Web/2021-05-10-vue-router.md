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

이제 라우터의 index.js에 가서 url을 매핑해주자!

-----

#### index.js

먼저 라우터를 생성하기 위해서는 vue-router에서 ```createRouter, createWebHistory```라는 함수를 import 해야한다. (vue 3버전부터 방식이 조금 바뀐걸로 알고있다.) 

```javascript
import { createRouter, createWebHistory } from 'vue-router'

// 경로 지정
const routes = [

]
const router = createRouter({
  history: createWebHistory(process.env.BASE_URL),
  routes
})

export default router
```
위와 같이 설정해주면 라우터를 사용할 준비가 끝났다. ```export default router```를 꼭 적어주자! 처음에 빼먹었었는데 계속해서 오류가 나서 한시간을 넘게 헤맸다.. ㅎ

이후 만들어둔 뷰 컴포넌트를 모두 등록해주자.

```javascript
import { createRouter, createWebHistory } from 'vue-router'
import Home from '../views/Home.vue'
import Login from '../views/Login.vue'
import Signup from '../views/Signup.vue'
import Mypage from '../views/Mypage.vue'

// 경로 지정
const routes = [
  {
    path: '/',
    name: 'Home',
    component: Home
  },
  {
    path: '/login',
    name: 'Login',
    component: Login,
  },
  {
    path: '/signup',
    name: 'Signup',
    component: Signup
  },
  {
    path: '/mypage',
    name: 'Mypage',
    component: Mypage,
    // 인증 후에만 접근할 수 있음
    // beforeEnter: [requireAuth]
}
  }

]

const router = createRouter({
  history: createWebHistory(process.env.BASE_URL),
  routes
})

export default router

```

위와 같이 paths에 경로와 이름, 컴포넌트를 매핑해준다.
mypage에 따로 주석처리가 되어있는 부분은 네비게이션 가드에서 자세히 다룰 예정이니 지금은 무시하자.
