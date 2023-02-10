---
layout: posts
title: "[Vue 3] 'Vuex' is not defined 오류"
categories:
  - Vue
last_modified_at: 2021-05-11
author_profile: true
tags:
  - 웹 프레임워크
  - vue3
  - vuex
toc: true
toc_sticky: true
sidebar:
  title: Posts
  nav: "sidebar-contents"
---

## Vue 3버전에서 vuex 사용하기


vue 3이 나온지 얼마 안된 상태 + vue 처음 배움으로 인해 구글링을 통해 얻은 정보가 vue 2에 대한건지~ vue 3에도 오류 없이 적용 가능한지~ 알 수가 없다. 어렵게, 어렵게 배우는 중이다.. 😂

main.js에서 ```'Vuex' is not defined```이러한 오류가 나오거나, 앱을 실행 후 콘솔창에서 ```Uncaught TypeError: Cannot read property ‘use’ of undefined```과 같은 오류를 발견한다면 자신의 vuex 버전이 어떤지 확인해보자.

일단 vue 3버전은 vuex 4 버전과 호환된다. 이전에 깔린 vuex가 3버전이어서 main.js에 import가 안됐었다. 

먼저, 이전에 깔려있던 vuex가 있다면 uninstall하고 재설치하자.
```
npm uninstall vuex
npm install vuex@next --save
```

#### 이제 main.js에 import하자

----

<a href="https://rayrny.github.io/vue/vue-vuex/">이전 포스팅</a>에서와 동일하게 store폴더를 생성하고 그 안에 index.js를 생성한 후, main.js에 import하자.

import는 라우터와 동일한 방식으로 해주면 된다.

```javascript
// main.js
import { createApp } from 'vue'
import App from './App.vue'
import router from './router'
import { store } from "./store"; // store 등록!

createApp(App).use(router, store).mount('#app')
```

#### store/index.js

----

이제 index.js로 넘어와서 ```createStore```를 통해 store를 생성하자.

```javascript
// store/index.js
import { createStore } from 'vuex'
import axios from 'axios'
// 라우터랑 동일한 방식으로 export
export const store = createStore ({
  state: {
  },
  getters: {
  },
  mutations: {
  },
  actions: {
  },
})
```

이렇게 하면 vue 3에서 vuex를 사용할 수 있다. 🤩 vuex에 대한 자세한 설명은 위에 언급한 <a href="https://rayrny.github.io/vue/vue-vuex/">이전 포스팅</a>에 따로 정리했다.