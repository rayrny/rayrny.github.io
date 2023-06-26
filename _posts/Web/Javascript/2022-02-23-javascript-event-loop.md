---
layout: posts
title: "[JS] 자바스크립트가 비동기 작업을 수행하는 방식: Event Loop, 그리고 Queue"
categories:
  - Javascript
last_modified_at: 2022-02-23
author_profile: true
tags:
  - 웹
  - Javascript
  - 비동기
toc: true
toc_sticky: true
sidebar:
  title: Posts
  nav: "sidebar-contents"
---

동시에 단 하나의 작업만을 할 수 있는 자바스크립트가 비동기 작업을 할 수 있는 비밀은 바로 Event Loop과 Queue이다.

자바스크립트가 작업들을 확인하는 우선 순위는 다음과 같다. (Call stack을 가장 먼저 확인하고 스택에 작업이 없으면 Microtask queue를 확인하는 식)

`Call stack` → `Microtask queue` → `Animation frames` → `Task queue`

### 자바스크립트의 동작 순서

```js
console.log("script start");

setTimeout(function () {
  console.log("setTimeout");
}, 0);

Promise.resolve()
  .then(function () {
    console.log("promise1");
  })
  .then(function () {
    console.log("promise2");
  });

requestAnimationFrame(function () {
  console.log("requestAnimationFrame");
});

console.log("script end");
```

<details>
  <summary>정답 (개발자 도구를 열고 직접 확인해도 좋다)</summary>
  <div markdown="1">
1. script start
2. script end
3. promise1
4. promise2
5. setTimeout
6. requestAnimationFrame
  </div>
</details>

위 예제를 단계에 따라 나눠 보는 것은 밑에서 그림을 그려가며 보는 것이 좋을 것 같아 이후에 해보도록 하고, 우선적으로 꼭 알고 가야 하는 것은 다음 세 가지이다.

1. 비동기 작업으로 등록되는 작업은 task, microtask 그리고 animationFrame 작업이다.
2. microtask가 task보다 우선순위가 높다.
3. microtask가 처리된 후 requestAnimationFrom이 호출되고 이후 브라우저 렌더링이 발생한다.

### Task와 Microtask

먼저 task는 비동기 작업의 순서를 보장하는 형태의 작업이다. (이 말이 task다음 곧바로 다음 task가 실행된다는 의미는 아니다. 말 그대로 예약된 순서를 보장한다는 것!)

예시로는 `setTimeout`의 `callback` 함수가 있다.

microtask는 비동기로 실행될 작업이, 현재 실행되고 있는 스크립트 바로 다음에 일어나게 되는 경우이다. 예시로는 `MutationObserver`와 `Promise`가 있다. (그으래서 setTimeout의 콜백 함수보다 `promise.resolve.then()`이 먼저 실행되었던 것!)

**예제 코드의 실행 단계**

![IMG_6A1ED8113C0A-1](https://user-images.githubusercontent.com/48341341/155157754-3b6c921b-194b-447e-b58a-7ee616ea69f2.jpeg)
![IMG_A5DA06294C46-1](https://user-images.githubusercontent.com/48341341/155157775-9dc10da5-9538-4d5f-8c95-85d6f3eb46d1.jpeg)
![IMG_4576D193EAD3-1](https://user-images.githubusercontent.com/48341341/155157789-65466cab-1668-4e3f-9dfb-1dc32beb0cca.jpeg)

### 사소한 궁금증에서 시작된...

위의 예제를 살펴본 뒤에 생긴 궁금증, 그렇다면 `async await`은 어떻게 동작을 하게 될까?

먼저 생각해볼 수 있는 것은 `async await`은 `Promise().then(...)`을 쉽게 사용할 수 있도록 제공하는 문법이라는 것이다.

```jsx
const delay = (time) => {
  return new Promise((res) => setTimeout(res, [time]));
};

const asyncFunc = async () => {
  console.log("await 시작");
  await delay(2000);
  console.log("await 끝");
};

const main = () => {
  console.log("스크립트 시작");
  asyncFunc();
  console.log("스크립트 끝");
};

main();
```

<details>
  <summary>정답 (개발자 도구를 열고 직접 확인해도 좋다)</summary>
  <div markdown="1">
1. 스크립트 시작
2. await 시작
3. 스크립트 끝
4. await 끝
  </div>
</details>

처음에는 `await 시작`이 찍히는 시점이 살짝 이해가 되지 않았는데, 다시보니 asyncAwait()을 만나서 `await 시작`까지는 비동기 작업이 아니니 바로 수행되고, await 이후 부터가 큐에 등록 되는 것이었다.

await 부분을 아래와 같이 변경한 뒤에 실행해도 동일한 결과가 나온다.

```js
const asyncFunc = () => {
  console.log("비동기 시작");
  delay(0).then(() => {
    console.log("비동기 끝");
  });
};
```

눈으로도 직접 확인하고 싶다면, microtasck보다 우선순위가 낮은 requestAnimationFrame를 asyncFunc보다 먼저 위치시킨 뒤에 실행 시켜보자. 실행 결과 `requestAnimationFrame`가 가장 나중에 찍히는 것을 확인할 수 있다.

```js
const delay = (time) => {
  return new Promise((res) => setTimeout(res, [time]));
};

const asyncFunc = async () => {
  console.log("await 시작");
  await delay(2000);
  console.log("await 끝");
};

const main = () => {
  console.log("스크립트 시작");

  requestAnimationFrame(function () {
    console.log("requestAnimationFrame");
  });
  asyncFunc();
  console.log("스크립트 끝");
};

main();
```

위의 테스트를 진행하다가 async await이 Promise.then을 완벽하게 대체하는 것은 아니지 않나? 라는 생각이 들었었는데, (아래 예시에서 `이후 다른 작업`이 찍히는 순서가 다르다.) 이는 스코프에 대한 문제인 것으로 이해했다.

```js
// 1
const asyncFunc = async () => {
  console.log("await 시작");
  await delay(2000);
  console.log("await 끝");

  console.log("이후 다른 작업");
};

// 2
const asyncFunc2 = () => {
  console.log("비동기 시작");
  delay(0).then(() => {
    console.log("비동기 끝");
  });
  console.log("이후 다른 작업");
};
```

await 이후에 오는 일들은 사실 상 then 콜백 안에 모조리 들어간 것과 같다고 봐야한다.
여기서 [].forEach 와 for(... of ...)에서의 비동기 동작 방식의 차이도 발생하게 된다.

---

ref.

> [https://sculove.github.io/post/javascriptflow/](https://sculove.github.io/post/javascriptflow/)
