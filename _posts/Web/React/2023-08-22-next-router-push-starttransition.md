---
layout: posts
title: "🚀 [Next.js] router.push의 동작 원리와 사이드 이펙트 (feat. startTransition)"
categories:
  - Next
last_modified_at: 2023-08-22
author_profile: true
tags:
  - next.js
  - react
toc: true
toc_sticky: true
sidebar:
  title: Posts
  nav: "sidebar-contents"
---


Next router.push을 통해 같은 페이지 내에서의 이동 시, 로딩 fallback은 왜 안보이는 것일까?


이번에 다루고자 하는 주제는 생각보다 여러 상황이 얽혀있었고, 생각보다 더 깊게 라이브러리를 뜯어봤어야 했다. 문제 해결에 있어서 아주 많은 도움을 준 **구**에게 감사 인사를 전한다.

### 문제 상황

```json
"next": "13.4.13", // page router 사용 13.4.6 사용 중 버그가 있는 것을 확인하고 버전 올려서 진행
"@tanstack/react-query": "^4.29.13",
"react": "18.2.0"
```

한 컴포넌트(`AsyncComponent`)가 있고, 이 컴포넌트의 데이터는 react-query의 `useQuery`를 `suspense` 옵션을 활성화 하여 가져온다. 따라서 데이터를 가져오는 동안, 상단에 감싸진 `Suspense`의 fallback인 ‘로딩중’ 텍스트가 보인다.

이제 이 컴포넌트의 버튼을 누르게 되면 쿼리 파라미터 page를 1씩 증가시키고, useQuery에서 사용하는 키에 이 page 쿼리 파라미터가 포함되어 있으므로 키가 변경되어 새로운 데이터를 패치한다.

```jsx
function Test() {
    return (
        <Suspense fallback={<h3>로딩중</h3>}>
            <AsyncComponent />
        </Suspense>
    );
}

function AsyncComponent() {
    const router = useRouter();
    const page = useQueryParam('page', { parser: Number }) ?? 0;
    const { data } = useSuspendedQuery(
				[{ page }, 'key'], 
				() => fetchFunc(page), 
				{ suspense: true }
		);

    return (
        <>
            <div>{page}</div>
            <div>{data}</div>
            <Button
                onClick={() => {
                    router.push(
                        `${router.pathname}${QS.create({ page: page + 1 })}`,
                        undefined,
                        { shallow: true },
                    );
                }}
            >
                버튼
            </Button>
        </>
    );
}
```

## 🫠 기대한 동작과 실제 동작

그렇다면, 위에서 쿼리 파라미터가 변경되어 새로운 데이터를 패칭해오는 동안 suspense의 fallback인 ‘로딩중’이 보여야할 것 같았지만 실제로는 이전 UI를 그대로 유지하다가 갑자기 툭, 변경된 데이터를 보여주었다.

→ react-query의 `keepPreviousData` 옵션과 같은 동작

굉장히 이해할 수 없는 동작이었고, 추정할 수 있는 원인 중 하나는 쿼리 파라미터는 React 의 state로써 관리되고 있지 않다는 점이었다. 그렇기 때문에 쿼리 파라미터가 변경되었을 때 React가 리렌더링을 해야한다고 판단하지 못하는 것은 아닐까? 하는 첫 번째 의심을 하게 되었다. (→ *사실 이게 원인은 아니었다.*)

실제로 아래 코드처럼 page를 state로 관리하면 우리가 생각했던대로 동작한다. (→ *`state`로 관리하고 말고의 문제가 아니라 `router.push`를 쓰지 않았다는 점이 더 중요하게 봐야 하는 부분이다.*)

<details>
  <summary>아래 코드</summary>
  <div markdown="1">

    ```jsx
    function Test() {
        return (
            <Suspense fallback={<h3>로딩중</h3>}>
                <AsyncComponent />
            </Suspense>
        );
    }
    
    function AsyncComponent() {
        const router = useRouter();
        const [page, setPage] = useState(0);    
    		const { data } = useSuspendedQuery(
    				[{ page }, 'key'], 
    				() => fetchFunc(page), 
    				{ suspense: true }
    		);
    
        return (
            <>
                <div>{page}</div>
                <div>{data}</div>
                <Button
                    onClick={() => {
                        setPage(page + 1);
                    }}
                >
                    버튼
                </Button>
            </>
        );
    }
    ```
  </div>
</details>

## **🛠️** 원인과 해결 방안

사실 이런 저런 삽질을 좀 했다. 

<details>
  <summary>펼쳐보기</summary>
  <div markdown="1">

    **state로 관리되지 않는 query parameter?**
    
    - 이런 저런 의심들을 하며 공식 문서를 뒤져보다가, Next.js의 router 관련해서 [같은 페이지 내에서의 이동](https://nextjs.org/docs/pages/api-reference/functions/use-router#resetting-state-after-navigation)이라는 섹션에서 페이지 상태와 관련된 내용을 찾았다.
    
    > the page's state **will not** be reset by default as React does not unmount unless the parent component has changed.
    > 
    
    이 문장에서 `unless the parent component has changed` 에 한 눈이 팔려서 query parameter가 변경될 경우에는 React의 리렌더링이 유발되지 않는가보다, 하고 생각했다. 
    
    - query parameter의 변경 → useQuery의 key 변경 → useQuery 리패치 → Suspense에 Promise가 잡혀있다가 fullfilled되면 useQuery의 data(이건 state로 관리되는 값이니)가 변경되며 그제서야 리렌더링 되어 화면에 반영되는 것이라고 생각했다.
    - 그래서 부모 컴포넌트인 Suspense에 key를 `router.asPath`로 주어 path(query parameter)가 변경될 때 마다 명시적으로 새로운 컴포넌트임을 알려줌으로써 문제를 해결했다.(고 생각했다.)
    
    이 글을 쓰는 시점에서 보면 상당히 비약도 많고 논리적이지도 않은 생각이었으나, 당시에는 이 말도안되는 상황을 이해할 수 있는 정리된 하나의 생각이었기 때문에 이 상태로 이해하고 넘어갔다.
    
    ***사실,*** query parameter도 상태로써 관리되는게 맞다. 그러기 때문에 컴포넌트 함수가 다시 실행되고 변경된 쿼리 키로 데이터도 refetching 할 수 있는 것이다.
    

  </div>
</details>

위의 삽질 후 며칠이 지나 `startTransition` 을 우연히 접하게 되었고, 이번 문제 해결의 key가 되었다.

### 🧀 **tl;dr**

Suspense를 지원하는 라우터의 경우, React에서는 네비게이팅을`startTransition`으로 감싸서 제공할 것을 권장한다. 

실제로 Next의 `router.push` 동작에 있어서 `startTransition`을 사용하고 있기 때문에(page router 기준으로 이동하는 페이지에 해당하는 ***react element를 그릴 때 사용***, app router의 경우 모든 네비게이팅 동작에 사용), 동일한 페이지에서 query parameter가 변경된 경우에는 이미 존재하는 컴포넌트를 굳이 unmount 시키지(Suspense의 fallback을 보여주지) 않는다. 이 때문에 `keepPreviousData` 옵션을 준 것 과 같은 동작을 확인할 수 있다.

이에 대한 해결책은 부모 컴포넌트의 `key`를 변경시키는 것이다. (결국 삽질기에서 시도했던 해결 방법과 동일하긴 하다.)

```jsx
<Suspense fallback={<h3>로딩중</h3>} key={router.asPath()}>
	<AsyncComponent />
</Suspense>
```

### [startTransition](https://react.dev/reference/react/startTransition)이 무엇일까?

<img width="922" alt="starttransition1" src="https://github.com/rayrny/rayrny.github.io/assets/48341341/5469e51c-00b9-406c-b97a-a59ae5eee2ea">

`startTransition`을 통해 특정 작업을 수행하는 동안, react는 ***이미 그려진*** 컴포넌트가 있다면 해당 컴포넌트를 유지시켜 현재 제공하는 UI를 그대로 사용할 수 있도록 한다. (== `Suspense`를 사용중인 경우, 의도적으로 fallback을 보여주지 않는다.)

조금 더 밑으로 내려보면 파라미터로 받는 scope는 loading indicators를 표시하지 않고 트랜지션이 수행된다는 설명을 찾을 수 있다. 더 자세한 사항은 공식 문서에 있으니 꼭 정독하자!

<img width="914" alt="starttransition2" src="https://github.com/rayrny/rayrny.github.io/assets/48341341/5d6558ef-a9af-4cb7-88ad-1056f0bb4f81">

[useTransition](https://react.dev/reference/react/useTransition)에 대한 설명에는 보다 자세하게 예제까지 들어있으므로 이것도 함께 보는 것을 추천한다.

- useTransition에서 ****[Preventing unwanted loading indicators](https://www.notion.so/a2a1002fc210488baff2ede989a96127?pvs=21)** 섹션에서 Suspense를 사용하는 경우에 transition을 사용하면 어떤 결과가 나타나는지 확인할 수 있다.
- `useTransition`과 다른 점은 `startTransition`은 트랜지션으로 인해 pending이 발생하고 있는지 확인할 수 있는 `isPending`을 제공하지 않는다는 것이다.

### Next의 router.push도 이걸 사용하는 건 아닐까?

실제로, react 공식 문서에서는 네비게이팅을 transition으로 표기하는 것을 추천한다. 

> ****[Building a Suspense-enabled router](https://www.notion.so/a2a1002fc210488baff2ede989a96127?pvs=21)****
> 

실제로 확인해보기 위해, 일단 [next github](https://github.com/vercel/next.js)에 들어가 냅다 `startTransition`을 검색해보았다. 

결과가 꽤 나온다! 

심지어 `app router`의 경우 아예 명시적으로 `startTransition`으로 네비게이팅을 한다는 사실도 알 수 있었다.

<img width="775" alt="next-link" src="https://github.com/rayrny/rayrny.github.io/assets/48341341/0c9ee66e-63de-4284-9900-0d5353487474">

Link를 통해 이동하는 경우

<img width="748" alt="next-router" src="https://github.com/rayrny/rayrny.github.io/assets/48341341/5b17e85a-e3a7-41f1-880c-10947e8be811">

router를 사용하여 이동하는 경우

그치만 나는 `page router`를 사용하고 있었으므로 조금 더 내려보다가 react element를 렌더링 할 때 `startTransition`을 사용한다는 것을 알아냈다!

<img width="759" alt="next-starttransition" src="https://github.com/rayrny/rayrny.github.io/assets/48341341/3222352f-8610-40f5-a8fd-dc9b459c2e24">

코드만 이리 저리 탐색해보기에는 너무 거대한 라이브러리라 내가 원하는 답을 바로 찾기 어려웠고, 실제로 저 `startTransition`을 뺐을 때 기대하는대로 동작하는 것을 확인해서 귀납적으로 답을 찾기로 했다.

<br /> <br /> 

**우선 아까 텍스트로만 설명했던 문제 상황을 영상으로 다시 확인해보자.**

👉 gif라 조금 느려요..! 한국인이지만 조금만 여유롭게 시청 바랍니다.

![with-start-transition](https://github.com/rayrny/rayrny.github.io/assets/48341341/dba01af4-e140-4bdb-8d6e-f59c4dfb0cfc)

**부모 컴포넌트(Suspense)에 key를 주었을 때 문제가 해결되는 것을 확인할 수 있다.**

![apply-key-to-suspense](https://github.com/rayrny/rayrny.github.io/assets/48341341/b96b48bf-5fca-414a-a4e1-30047b4cc80f)

✅ **그렇다면 patch-package를 사용해 실제 라이브러리 코드를 수정했을 때는?**

---

patch-package를 사용해 패키지를 직접 수정하는 방법은 여기서 다루진 않겠다. 구글링하면 자세하게 설명하고 있는 다른 글들이 많으니 참고하길 바란다.

![edit-package](https://github.com/rayrny/rayrny.github.io/assets/48341341/68d3681a-138a-474f-af97-26c859af7be3)

아래는 실제로 수정한 코드이다.

```jsx
diff --git a/node_modules/next/dist/client/index.js b/node_modules/next/dist/client/index.js
index 2def4d8..86fd856 100644
--- a/node_modules/next/dist/client/index.js
+++ b/node_modules/next/dist/client/index.js
@@ -385,10 +385,12 @@ function renderReactElement(domEl, fn) {
         // TODO: Remove shouldHydrate variable when React 18 is stable as it can depend on `reactRoot` existing
         shouldHydrate = false;
     } else {
-        const startTransition = _react.default.startTransition;
-        startTransition(()=>{
-            reactRoot.render(reactEl);
-        });
+        // const startTransition = _react.default.startTransition;
+        // startTransition(()=>{
+        //     reactRoot.render(reactEl);
+        // });
+        console.log('start transition 제거');
+        reactRoot.render(reactEl);
     }
```

정말로 `startTransition`을 제거하니까 처음 기대했던 것과 같이 동작하는 것을 확인할 수 있었다!

### 그렇다면 어떻게 해결해야 할까?

사실 react 공식 문서의 Suspense 쪽에도 [startTransition이 등장한다.](https://ko.react.dev/reference/react/Suspense#preventing-already-revealed-content-from-hiding) React는 이미 보여지고 있는 화면을 멈추고 로딩 fallback을 보여주는 것은 사용자 경험을 해칠 수 있다고 말하며, 이를 해소할 수 있는 방안으로 `startTransition`을 소개하고 있다.

또한 Suspense-enabled router(Suspense를 지원하는 라우터)는 내부적으로 `startTransition`을 래핑할 것이라고 생각해버린다. (자기들이 추천하는 방식이므로 따르라는 것이지!)

그리고 정확하게 위 문제를 해결할 수 있는 방안을 함께 소개하고 있으며, 결론적으로는 query parameter 등이 변경되었을 때 React에게 이건 새로운 컴포넌트임을 알려주어야 한다고 한다.

> ****[Resetting Suspense boundaries on navigation](https://ko.react.dev/reference/react/Suspense#resetting-suspense-boundaries-on-navigation)****
> 
- 이는 key props를 통해 달성할 수 있고, 나의 경우에는 부모 컴포넌트인 Suspense에 key를 주어 해결했다.

```jsx
<Suspense fallback={<Txt>로딩중</Txt>} key={router.asPath()}>
	<AsyncComponent />
</Suspense>
```

### ⛅️ 자잘한 시행 착오

우연히 테스트를 하고 있던 Next 버전이 `13.4.6`이었는데, 여기선 같은 경로로 이동했을 때 아예 페이지를 다시 그리지도 않는 굉장히 이상한 동작을 했다.

- `13.3.x` 버전대로 낮추거나 `13.4.13~` 인 최신 버전으로 다시 설치했을 때는 정상적으로 동작했다.
- `13.4.6` 버전에서 HTTP `delete` 메소드 관련해서 [body에 data를 넘겨줘도 항상 비어있는](https://github.com/vercel/next.js/discussions/48072) 이슈도 있다가 이후 버전에서 수정되었는데 아무래도 마의 버전인 것 같다…. ㅂㄷㅂㄷ
- 다시 한 번.. 아무리 대중적으로 많이 사용되는 라이브러리라고 해도 자체적인 버그가 있을 수 있다는 사실을 자각했다.