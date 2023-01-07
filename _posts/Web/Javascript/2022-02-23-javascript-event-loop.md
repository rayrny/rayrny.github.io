---
layout: posts
title: "[JS] 자바스크립트가 비동기 작업을 하는 방식: Event Loop, 그리고 Queue"
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


> [https://sculove.github.io/post/javascriptflow/](https://sculove.github.io/post/javascriptflow/)


사실 위의 블로그 포스팅을 다시 정리해보며 개념을 이해하는 과정이라 내가 새롭게 추가한 내용은 거의 없다고 보면 된다.. 

동시에 단 하나의 작업만을 할 수 있는 자바스크립트가 비동기 작업을 할 수 있는 비밀은 바로 Event Loop과 Queue이다.

자바스크립트가 작업들을 확인하는 우선 순위는 다음과 같다. (call stack을 가장 먼저 확인하고 스택에 작업이 없으면 microtask queue를 확인하는 식)

`Call stack`  → `Microtask queue` → `Animation frames` → `Task queue` 

### 자바스크립트의 동작 방식

```jsx
console.log("script start");

setTimeout(function() {
  console.log("setTimeout");
}, 0);

Promise.resolve().then(function() {
  console.log("promise1");
}).then(function() {
  console.log("promise2");
});

requestAnimationFrame(function {  
  console.log("requestAnimationFrame");
});

console.log("script end");
```

위 예제를 단계에 따라 나눠 보는 것은 따로 그림을 그려가며 보는 것이 좋을 것 같아 이후에 해보도록 하고, 우선적으로 꼭 알고 가야 하는 것은 다음 세 가지이다.

1. 비동기 작업으로 등록되는 작업은 task, microtask 그리고 animationFrame 작업이다.
2. microtask가 task보다 우선순위가 높다.
3. microtask가 처리된 후 requestAnimationFrom이 호출되고 이후 브라우저 렌더링이 발생한다.

### Task와 Microtask

먼저 task는 비동기 작업의 순서를 보장하는 형태의 작업이다. (이 말이 task다음 곧바로 다음 task가 실행된다는 의미는 아니다. 말 그대로 예약된 순서를 보장한다는 것!) 예시로는 `setTimeout`의 `callback` 함수 등이 되겠다.

microtask는 비동기로 실행될 작업이, 현재 실행되고 있는 스크립트 바로 다음에 일어나게 되는 경우이다. 에시로는 `MutationObserver`와 `Promise`가 있다. (그으래서 setTimeout의 콜백 함수보다 `promise.resolve.then()`이 먼저 실행되었던 것!)

**예제 코드의 실행 단계**

![IMG_6A1ED8113C0A-1](https://user-images.githubusercontent.com/48341341/155157754-3b6c921b-194b-447e-b58a-7ee616ea69f2.jpeg)
![IMG_A5DA06294C46-1](https://user-images.githubusercontent.com/48341341/155157775-9dc10da5-9538-4d5f-8c95-85d6f3eb46d1.jpeg)
![IMG_4576D193EAD3-1](https://user-images.githubusercontent.com/48341341/155157789-65466cab-1668-4e3f-9dfb-1dc32beb0cca.jpeg)


### 사소한 궁금증에서 시작된...

위의 예제를 살펴본 뒤에 생긴 궁금증, `async await`은 어떤 큐로 가서 동작을 하게 될까?

처음에는 아래와 같은 코드를 넣고 결과를 확인해 보았는데 어디에 있든 위치와 상관없이 계속 `setTimeout`이후에 찍혀서 `Task Queue`로 들어가는줄 알았는데.. 바보였다. 


```jsx
const intervalFunction = () => {
	return new Promise((resolve, reject) => {
		setTimeout(() => {
			resolve(5);
		}, 3000);
	});
}

const asyncAwait = async() => {
    const res = await intervalFunction();
    console.log('async await');
}

asyncAwait();
```

당연히 `setTimeout`을 사용하니까... ㅋㅋㅋㅋ

무튼 `setTimeout`을 부르지 않고 바로 `resolve` 하는 함수로 변경한 뒤에, 스크립트 실행 후 맨 처음으로 호출하도록 했을 때와 `setTimeout함수` 이후에 부른 경우에는 아래와 같은 결과가 나왔다.

> "script start"
> "script end"
> "async await"
> "promise1"
> "promise2"
> "requestAnimationFrame"
> "setTimeout"

그리고 이제 `Promise 함수` 이후에 호출하도록 하니 좀 요상한 결과가 나왔다.

```jsx
console.log("script start");

setTimeout(function() {
  console.log("setTimeout");
}, 0);

Promise.resolve().then(function() {
  console.log("promise1");
}).then(function() {
  console.log("promise2");
});


const intervalFunction = () => {
	return new Promise((resolve, reject) => {
		//setTimeout(() => {
		//	resolve(5);
		//}, 3000);
      resolve(5);
      console.log('interval func');
	});
}

const asyncAwait = async() => {
    const res = await intervalFunction();
    console.log('async await');
}

asyncAwait();

requestAnimationFrame(function() {  
  console.log("requestAnimationFrame");
});

console.log("script end");
```

> "script start"
> "interval func"
> "script end"
> "promise1"
> "async await"
> "promise2"
> "requestAnimationFrame"
> "setTimeout"

`promise1`과 `promise2` 중간에 `async await`이 찍혔다..!

setTimeout보다 먼저 찍혔다는 소리는 await 이 `microtaskQueue`에 들어간다는 의미일까? 그래서 찬찬히 생각해보니 맨 처음 자바스크립트가 아래로 내려가면서 Promise 함수를 콜스택에서 부른 후 `microtaskQueue`에 then1이 등록되고 이후에 asyncAwait 함수가 콜스택에 등록되어 then1 뒤에 async await이 등록되기 때문인 것 같다.

그런데 이제 interval func은 왜 스크립트 시작과 끝 사이에 찍히게 된걸까..?

는 당연히.. 별다른 비동기로 제어하는 작업이 아니니까.. 바보야..
