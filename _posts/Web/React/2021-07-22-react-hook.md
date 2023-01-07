---
layout: posts
title: "[React] React Hooks"
categories:
  - React
last_modified_at: 2021-07-22
author_profile: true
tags:
  - 웹 프레임워크
  - react
toc: true
toc_sticky: true
sidebar:
  title: Posts
  nav: "sidebar-contents"
---

### React의 Hook 중에서 useState, useEffect, useCallback에 대해서 정리해보자.

NASA API를 사용한 간단한 프로젝트를 만들던 중 무한 스크롤을 구현하게 되었고, 그 때 처음 사용한 ```useCallback```에 대해서 정리할 겸 글을 적는다.


#### useState

----

함수형 컴포넌트에서도 상태를 관리할 수 있게 한다. 사용 방법은 간단하다.

```javascript
const [val, setVal] = useState();
```

변수와 그 변수의 상태를 관리할 함수를 함께 선언하고 ```useState()```로 초기화 해준다. (문자열일 경우 "", 숫자일 경우 0 등 자유롭게 선언할 수 있다. 배열은 useState([])로 초기화한다.)

상태를 저장해아 하는 변수의 경우 모두 useState를 사용하도록 한다.


#### useEffect

---

화면이 마운트 될 때만 실행되는 useEffect 함수이다. 클래스형 컴포넌트에서 ```componentDidMount, componentDidUpdate```와 같이 사용할 수 있다.

```javascript
useEffect(fucntion, []) // 실행할 함수와 검사하고자 하는 값 또는 배열을 함께 적는다.
```
만약 마운트 될 때, 즉 렌더링 직후 한 번만 실행하고자 하면 빈 배열을 두면 되고, 값이 변경될 때 마다 실행하고 싶다면 뒤에 변수를 넣어주면 된다. 

또한 컴포넌트가 언마운트 되기 전이나 업데이트 전에 수행할 작업이 있다면 다음과 같이 반환할 함수(clean up 함수)를 반환해야 한다.

```javascript
useEffect(() => {

    return () => {
        console.log('clean up');
    }
}, [])
```

또 useEffect 안에서 비동기 작업을 수행하고자 하면 안에서 ```async``` 함수를 선언하고 바로 호출해서 사용하면 된다.

```javascript
useEffect(() => {

    async function func ()  {
        let res = await apiFunc();
    }

    func();
}, [])
```

#### useCallback

---

useCallback의 경우 useMemo랑 비슷하지만 함수 자체를 기억해야 할 때 사용한다. 첫 번째 인자의 함수를 두 번째 인자의 배열이 변경될 때 까지 저장하고 기억한다. 주로 렌더링 성능 최적화를 위해 사용한다.

```javascript
const callback = useCallback(function, []);
```

함수의 경우 언제나 그렇듯 바로 애로우 펑션으로 작성해서 사용 가능하다.

useCallback 함수의 경우 렌더링이 되더라도 감시하고자 하는 값이 바뀌지 않은 한 기존 함수를 계속 반환한다. 무한 스크롤을 구현할 때 이 useCallback 훅을 사용했다. 

어떠한 함수 내부에서 변수의 값이 변경된다고 하자. 컴포넌트가 렌더링 될 때마다 값이 새로운 참조값으로 바뀔 것이다. 그러면 ```useEffect()```가 호출되어 변수의 상태값이 바뀌고 그러면 다시 렌더링이 될 것이고... 

이처럼 무한 반복에 빠질 수 있다. 이럴 경우 ```useCallback``` 훅을 사용하면 컴포넌트가 다시 렌더링 되더라도 함수의 참조값을 유지할 수 있다.

------

이번에 진행한 간단한 사진 검색 어플리케이션을 만들며 이전에 FLOG를 만들며 정신없이 짰던 코드들이 어떤 의미가 있어서 사용되었는지를 생각해볼 수 있었다. 감사한 시간이었다. 😊😊


> https://www.daleseo.com/react-hooks-use-callback/



-----

거의 1년이 지난 후에 내용을 추가하게 되었다. 관련 내용은 예전에 정리해두었지만 블로그에 내용 업데이트를 안 한 듯 하여..

### useRef

컴포넌트에서 특정 DOM 을 선택해야 할 때, `ref` 를 사용해야 한다고 배웠다. 그리고, 함수형 컴포넌트에서 이를 설정 할 때 `useRef` 를 사용한다.
`useRef` 는 DOM 을 선택하는 용도 외에, 컴포넌트 안에서 조회 및 수정 할 수 있는 변수를 관리할 수 있다.
`useRef` 로 관리하는 변수는 값이 바뀐다고 해서 컴포넌트가 리렌더링되지 않는다. 리액트 컴포넌트에서의 상태는 상태를 바꾸는 함수를 호출하고 나서 그 다음 렌더링 이후로 업데이트 된 상태를 조회 할 수 있는 반면, `useRef` 로 관리하고 있는 변수는 설정 후 바로 조회 할 수 있다.

컴포넌트를 의도적으로 리렌더링할 생각으로 사용하는 경우가 아니라면 useRef를 통한 변수 사용은 지양하는 것이 좋다. 클래스형 컴포넌트와 다를 바 없는 변수 관리 방법이 되기 때문이다. 얼마 전 해결했던 강좌 소개 페이지를 보고 그 이유를 깨달았는데, class형 컴포넌트로 작성되어 있었고, 그 마저도 setState를 통해 변수를 관리하지 않고 A함수에서 새롭게 변수를 만들고 B함수에서 그 값을 변경하고... 디버깅하기가 너무 힘들고 실제로 버그를 유발하는 코드였다. 스크롤 위치 계산을 위해서 DOM에 접근 후 특정 엘리먼트의 offsetHeight 등을 사용하는데, 관련된 변수를 모두 그냥 선언해서 사용하다보니, DOM의 업데이트를 전혀 감지하지 못해서 각종 이벤트 등이 동작하지 않았었다. 

스크롤 위치를 전부 계산하는 부분부터 intersection observer로 갈아끼우고 싶었지만, cs로 들어왔던 문의였기에 최대한 기존 코드를 건들지 않고 빠르고 깔끔하게 코드를 수정하고 싶었다. 고민끝에 componentDidUpdate 에서 DOM 요소들을 가져오고 값을 읽어오는 함수를 한 번 더 실행하도록 해서 잘 마무리했다. 



### 컴포넌트 참조 오류가 발생했을 경우

<aside>
⚠️ `Warning: Function components cannot be given refs. Attempts to access this ref will fail. Did you mean to use React.forwardRef()?`

</aside>

위의 에러는 DOM 객체에 접근할 수 없는 경우 발생한다(함수형 컴포넌트 그 자체일 때?). 해결 방법은 안에 div 박스를 하나 더 만들어서 참조할 수 있도록 하면 된다.

### useMemo

`useMemo` 의 첫번째 파라미터에는 어떻게 연산할지 정의하는 함수를 넣어주면 되고 두번째 파라미터에는 deps 배열([] ← 요거)을 넣어주면 되는데, 이 배열 안에 넣은 내용이 바뀌면, 등록한 함수를 호출해서 값을 연산해주고, 만약에 내용이 바뀌지 않았다면 이전에 연산한 값을 재사용한다.

useMemo는 메모이즈된 값을 return하는 **hook**이다.

### useCallback

`useMemo` 는 특정 결과값을 재사용 할 때 사용하는 반면, `useCallback` 은 특정 함수를 새로 만들지 않고 재사용하고 싶을때 사용한다.

`useCallback(fn, deps)`은 `useMemo(() => fn, deps)`와 같다.

함수는 컴포넌트가 리렌더링 될 때마다 새로 만들어지는데, 최적화를 위해서는 함수 재사용이 필수적이다.

주의 할 점은, 함수 안에서 사용하는 상태 혹은 props 가 있다면 꼭, `deps` 배열안에 포함시켜야 한다. 만약 `deps` 배열 안에 함수에서 사용하는 값을 넣지 않게 되면, 함수 내에서 해당 값들을 참조할때 가장 최신 값을 참조 할 것이라고 보장 할 수 없다. props 로 받아온 함수가 있다면, 이 또한 `deps` 에 넣어줘야 한다.

### React.memo

→ **Higher-Order Components(HOC)**이다. (컴포넌트를 인자로 받아 새로운 컴포넌트롤 다시 return해주는 함수) HOC는 리액트의 API가 아니라 리액트가 컴포넌트를 구성하는데 있어서의 일종의 패턴이라고 보면된다.

이를 사용하면 컴포넌트의 props가 바뀌지 않았을 때 리렌더링을 방지한다. 이를 통해 리렌더링이 필요한 상황에서만 리렌더링을 할 수 있다.

다른 말로 하면,

React는 먼저 컴포넌트를 렌더링(rendering) 한 뒤, 이전 렌더된 결과와 비교하여 DOM 업데이트를 결정한다. 만약 렌더 결과가 이전과 다르다면, React는 DOM을 업데이트한다.

컴포넌트가 `React.memo()`로 래핑 될 때, React는 컴포넌트를 렌더링하고 결과를 메모이징(Memoizing)한다. 그리고 다음 렌더링이 일어날 때 `props`가 같다면, React는 메모이징(Memoizing)된 내용을 재사용한다.

React.memo는 오직 props가 변경됐는지 아닌지만 체크한다. 만약 React.memo에 감싸진 함수형 컴포넌트가 함수 내부에서 useState나 useContext같은 훅을 사용하고 있다면, state나 context가 변경될 때마다 리렌더링된다.

같은 props로 렌더링이 자주 일어나는 경우, 예를 들어 영화 정보를 1초마다 업데이트 하는 경우에 일치하지 않는다면 `React.memo()`를 사용할 필요가 없을 가능성이 높다.

경험적으로, 성능적인 이점을 얻지 못한다면 메모이제이션을 사용하지 않는것이 좋다.

> 성능 관련 변경이 잘못 적용 된다면 성능이 오히려 악화될 수 있다. React.memo()를 현명하게 사용하라.
> 

다만, 콜백 함수를 prop으로 사용하는 컴퍼넌트에서 메모이징을 할 때 주의하라. 그리고 같은 렌더링을 할 때 이전과 동일한 콜백 함수 인스턴스를 넘기는지 확실히 하라.

### React.Fragment

`<> </>`로 간단하게 사용할 수 있지만, ~~만약 fragment 태그에 key를 직접 줘야 할 경우 `<React.Fragment key={}> </React.Fragment>`와 같이 명시한다.~~?????? 확인 필요

### React.forwardRef

### 함수형 업데이트

```jsx
setA(prevA → prevA + 1);
```

### useLayoutEffect

useEffect와 동일한 기능을 하지만, 모든 DOM 변경 후에 동기적으로 발생한다. 따라서 DOM의 레이아웃만을 리렌더링 할 경우에 사용한다. `useLayoutEffect` 의 내부에서 갱신은 브라우저가 화면을 그리기 이전에 동기적으로 수행된다. (useEffect로 해결이 안 될 경우에만 사용하는 것을 권장.)

→ 렌더링 최적화를 위한 작업을 할 때, 레이아웃만 건드는 경우에 사용했던 것 같다.
