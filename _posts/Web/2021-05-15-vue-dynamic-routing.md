---
layout: posts
title: "[Vue] 동적 라우팅을 통해 url에 아이디를 표시해보자"
categories:
  - Vue
last_modified_at: 2021-05-15
author_profile: true
tags:
  - 웹 프레임워크
  - vue
  - vue-router
toc: true
toc_sticky: true
sidebar:
  title: Posts
  nav: "sidebar-contents"
---


## Vue Router의 동적 라우팅 기능을 통해 url을 자유롭게 구성해보자.

----

라우터의 path를 결정할 때, ```:params```을 통해 패턴을 만들 수 있다. 이를 통해, 마이페이지와 같이 동일한 레이아웃을 가지지만 사용자에 따라 다른 정보를 렌더링할 수 있다.

동적 라우트릉 매칭하기 위해서, 가장 먼저 **router/index.js**의 path를 동적 세그먼트를 사용해 나타내자.

```javascript
// router/index.js

// import 부분 생략
const router = new VueRouter({
  mode: "history",
  routes: [
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
      // 콜론(:)으로 시작하는 부분이 동적 세그먼트
      path: '/mypage/:userId',
      name: 'Mypage',
      component: Mypage,
    }
```

위의 코드를 보면, Home과 Login페이지는 일반적인 라우트 경로를 가지지만 Mypage의 경우 뒤에 사용자의 아이디가 붙는다. 

그렇다면, userId는 어떻게 넘겨주어야 할까? <a href="https://router.vuejs.org/kr/guide/essentials/dynamic-matching.html">공식 문서</a>에는 이 부분에 대해 자세히 나오지 않아 조금 헤맸다. 

필자의 경우, mypage는 네비게이션 가드로 막아두었기 때문에, 로그인을 성공적으로 마친 후에만 마이페이지로 redirect한다. 또한, 로그인 이후에 보이는 상단 메뉴바에서 마이페이지로 이동할 수 있는 링크가 있다. 

-------

때문에, 먼저 로그인 시 redirect 하는 부분에서 userId에 대한 값을 넘겨주기로 했다. (어차피 로그인 할 때 아이디를 입력하기 때문에 따로 axios를 통해 서버로 아이디를 요청하지 않아도 된다.)

다른 부분은 생략하고 페이지를 ```redirect```하는 함수만 살펴보자. 사용자로부터 입력받은 **아이디는 data() 의 user_id**에 저장된다.

아래의 코드는 프로그래밍 방식 네비게이션(```this.$router.push()```)을 사용하였을 때 동적 라우팅을 매칭하는 방법이다. 먼저 라우터의 이름을 적어주고(대소문자 주의!!), 이후 ```params:``` 부분에 동적 세그먼트를 넘겨주자. 물론 변수 대신 바로 상수 값을 넘겨줄 수도 있다. (```params: {userId : 'testID'}```)

```javascript
// views/login.vue
<script>
export default {
  name: 'Login',

  data() {
      return {
      user_id: "", // 아이디가 저장되어 있음
      user_pw: "",
      code: "",
    }
  },

    methods: {
      // 생략


      redirect() {
        // 아래와 같이 코딩하는 것을 프로그래밍 방식 네비게이션이라고 하고,
        // <router-link>를 사용하여 코딩하는 것을 선언적 네비게이션이라고 한다.
        this.$router.push({name: 'Mypage', params: {userId : this.user_id}});
      }
    },
}
</script>
```

-----

<router-link>를 통한 방법은 변수값을 어떻게 넘겨주어야 하는지 모르겠어서 프로그래밍 방식으로 코딩했다.