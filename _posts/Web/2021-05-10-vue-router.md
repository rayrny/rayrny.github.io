---
layout: posts
title: "[Vue] Vue-router에 대해서 알아보자"
categories:
  - Vue
last_modified_at: 2021-05-10
author_profile: true
tags:
  - 웹 프레임워크
  - vue
  - router
toc: true
toc_sticky: true
sidebar:
  title: Posts
  nav: "sidebar-contents"
---

## vue router는 컴포넌트들을 매핑하고 어떤 주소로 렌더링 할지를 결정할 수 있다.

설치는 간단하게 ```npm install vue-router```를 통해 할 수 있다.


### 🐤 라우터 설치 후..

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
import router from './router' // router 등록!

createApp(App).use(router).mount('#app')
```

이제 라우터의 index.js에 가서 url을 매핑해주자!

-----

#### index.js

먼저 라우터를 생성하기 위해서는 vue-router에서 ```createRouter, createWebHistory```라는 함수를 import 해야한다. (vue 3버전부터 방식이 조금 바뀐걸로 알고있다.) 

```javascript
// router/index.js
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
// router/index.js

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
  },
]
```

위와 같이 paths에 경로와 이름, 컴포넌트를 매핑해준다.
mypage에 따로 주석처리가 되어있는 부분은 네비게이션 가드에서 자세히 다룰 예정이니 지금은 무시하자. 

#### 이제, 컴포넌트에서 라우터를 사용해보자!

----

먼저, ```App.vue```에서 라우터를 통해 전환할 부분을 명시해 주자.

```javascript
// App.vue
<template>
  <div id="nav">
    <!-- 라우터로 화면을 전환할 때도 계속해서 바뀌지 않는 부분 -->
    <Header />
    <br>
  <router-view>
  <!-- 화면이 전환되는 부분 -->
  </router-view>
  </div>

</template>
```

위의 코드를 보면, 화면에는 맨 윗 부분에 ```Header```가 있고, url이 바뀔 때마다 ```<router-view>``` 태그가 있는 부분이 위에서 등록한 ```Home.vue, Login.vue``` 등의 컴포넌트로 화면에 표시된다.

----

**페이지 내에서 이동을 할 경우**에는, ```router-link```를 통해 특정 링크로 이동할 수 있다.

위의 코드에 잠시 등장했던 Header.vue를 살펴보자.
```javascript
// Header.vue
<template>
    <div id='header'> 
        <div id='menuWrap'>
        <router-link to="/">Home</router-link> |
        <router-link to="/login">Login</router-link> |
        <router-link to="/mypage">Mypage</router-link>
        </div>
    </div>
</template>
```

![Header](/assets/image/menuWrap.PNG)

헤더에는 ```'menuWrap'```으로 묶인 상단 메뉴바가 있는데, 각각 Home과 Login으로 이동할 수 있다. ```to=""``` 부분에 이동하고자 하는 url을 적어주면 된다.

위의 방식이 **선언적 방식 네비게이션**이라면, **프로그래밍적 방식 네비게이션**도 있다. ```this.$router.push("")```와 같이 라우터의 인스턴스 메소드를 통해 프로그래밍적으로 링크를 이동할 수 있다.

이외에도, push와 같은 역할을 하지만 새로운 history 항목에 추가하지 않고 탐색하는 ```router.replace("")```, history 스택에서 앞 뒤로 이동할 수 있는 ```router.go(n)```이 있다.