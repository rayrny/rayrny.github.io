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

microtask는 비동기로 실행될 작업이, 현재 실행되고 있는 스크립트 바로 다음에 일어나게 되는 경우이다. 에시로는 MutationObserver와 Promise가 있다. (그으래서 setTimeout의 콜백 함수보다 promise.resolve.then()이 먼저 실행되었던 것!)

**예제 코드의 실행 단계**

![IMG_6A1ED8113C0A-1](https://user-images.githubusercontent.com/48341341/155157754-3b6c921b-194b-447e-b58a-7ee616ea69f2.jpeg)
![IMG_A5DA06294C46-1](https://user-images.githubusercontent.com/48341341/155157775-9dc10da5-9538-4d5f-8c95-85d6f3eb46d1.jpeg)
![IMG_4576D193EAD3-1](https://user-images.githubusercontent.com/48341341/155157789-65466cab-1668-4e3f-9dfb-1dc32beb0cca.jpeg)
