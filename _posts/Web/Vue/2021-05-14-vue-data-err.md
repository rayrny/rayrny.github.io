---
layout: posts
title: "[Vue] The 'data' option should be a function 오류"
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

## 컴포넌트 내의 data 접근

-----

Vue에 대해 차근차근 배우지 않은 채로 개발을 하다보니, 정말 엄청나게 많은 오류들을 접할 수 있다. 😂 

컴포넌트의 data에 대해 정의할 때는 아래와 같이,

```javascript
data() {
    return {
        user_name: '',
        user_id: '',
	}; 
}
```

함수와 같이 return 값 내부에 적어 주어야 한다. 

그렇지 않으면 아래의 오류를 만나게 된다 XD

```[Vue warn]: The "data" option should be a function that returns a per-instance value in component definitions```

