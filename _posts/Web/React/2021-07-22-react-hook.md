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