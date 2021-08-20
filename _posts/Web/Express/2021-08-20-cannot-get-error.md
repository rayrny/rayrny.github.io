---
layout: posts
title: "[Express] Cannot GET / 에러"
categories:
  - Express
last_modified_at: 2021-08-20
author_profile: true
tags:
  - 웹 프레임워크
  - express
  - error handling
toc: true
toc_sticky: true
sidebar:
  title: Posts
  nav: "sidebar-contents"
---


### 💥 React 새로고침(또는 URL 직접 입력) 시에 Cannot GET / 에러 발생

프로젝트를 진행하던 중, 분명 프론트에서는 라우터를 지정해 주었는데 새로고침을 하면 404 NOT FOUND 에러가 발생하며 페이지가 뜨지 않았다. 

정말 열심히 구글링하며 web-pack 설정도 바꿔보고 package.json에 경로 추가도 해봤지만 전혀 해결되지 않았다.

![ERR](/assets/image/cannot_get.PNG)

#### webpack.config.js

----

일반적인 경우라면 ```webpack.config.js```에 

```js
devServer: {
   historyApiFallback: true,
 },
```

를 추가해 주면 해결 될 것으로 보인다.

#### Express의 경우

----

스택오버플로우를 열심히 뒤져본 결과 webpack을 사용하지 않을 경우에는 아래와 같이 ```index.html```을 등록하면 된다고 하는데, 나는 해당되지 않았다.

```js
app.get('/*', function(req, res) {
 res.sendFile(path.resolve(__dirname, 'public', 'index.html'));
})
```

----

정말 골치가 아프던 와중에 ```express cannot GET```을 검색했다가 해결 방법을 찾았다..

![개비스콘](/assets/image/gabiscon.jpg)


프론트 단에는 경로를 등록했지만 서버 단에 등록하지 않아서 생긴 이슈였다. 🤦‍♀️🤦‍♀️ ```routes/index.js```에 경로를 추가해주니 아주 깔끔하게 해결되었다고 한다.