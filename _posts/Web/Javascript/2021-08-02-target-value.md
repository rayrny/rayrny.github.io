---
layout: posts
title: "[JS] target과 currentTarget"
categories:
  - Javascript
last_modified_at: 2021-08-02
author_profile: true
tags:
  - 웹 프레임워크
  - JavaScript
  - react
toc: true
toc_sticky: true
sidebar:
  title: Posts
  nav: "sidebar-contents"
---


### 자바스크립트의 event 객체인 target과 currentTarget의 차이점에 대해 알아보자.

-----

아래의 코드를 보자.

```javascript
<div>
    <input name="userName" placeholder="이름을 입력하세요"/>
</div>
```

여기서 console로 각각 event.target과 event.currentTarget을 찍어보면, target은 자식 요소인 ```<input>```태그를 반환하고 currentTarget은 부모 요소인 ```<div>```를 반환한다.

### event.target.value란?

---

위에서 본 것 처럼 event.target을 하면 해당하는 태그를 가리킨다. 그렇다면 그 요소의 값에 접근하려면 어떻게 해야 할까?

```javascript
event.target.value // 태그의 값
event.target.name // 태그의 이름
```

위와 같이 사용하면 된다.

이들의 차이점과 각각의 역할에 대해 다시 정리해 보았는데, 아직 키보드 입력이 버벅거리는? 한 글자 한 글자 바로 입력창에 보이는 것이 아니라 모든 입력이 끝난 후에 뙇! 하고 보여지는 현상? 을 어떻게 하면 없앨 수 있는지는 아직 찾지 못했다. 

currentTarget에서 target으로 변경해 봤지만 변화가 없었고, target.value를 제거하면 input 값이 제대로 전달 되지 않는다. 내 생각에는 입력을 체크하는? 속도가 느린 것 같다. 천천히 입력하면 한 글자씩 나오지만 빠르게 칠 때만 그런 현상이 나타나기 때문이다.. 좀 더 알아봐야겠다.