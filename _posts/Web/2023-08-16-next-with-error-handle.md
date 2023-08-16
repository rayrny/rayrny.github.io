---
layout: posts
title: "[Next.js] BFF 서버 쿠키 유실 문제 및 서버 에러 전역 핸들링 처리하기"
categories:
  - Next
last_modified_at: 2023-08-16
author_profile: true
tags:
  - next.js
  - BFF
  - 에러 처리
toc: true
toc_sticky: true
sidebar:
  title: Posts
  nav: "sidebar-contents"
---

이 글에서는 아래 두 가지 주제에 대해 다룰 것이다.

1. BFF 서버가 클라이언트의 역할을 할 때, 브라우저와 달리 필요한 쿠키가 없는 상태에 대한 해결 방안
   1. 설명을 더하자면 BFF에서 API 요청을 통해 내부 자원에 접근하는 경우, 즉 `getServerSideProps`를 통해 페이지를 구성할 때 필요한 브라우저 쿠키 값을 가지고 있지 않은 채로 요청을 보내게 되는 경우
   2. Next는 이걸 [불필요한 네트워크 통신을 늘리는 방향](https://nextjs.org/docs/pages/building-your-application/data-fetching/get-server-side-props#getserversideprops-or-api-routes)이라고 했으나, 브라우저든 서버든 API를 사용해 통신하므로 page route에서 validate, 권한 체크 등을 한 번에 관리하기 위해 해당 방식을 선택했다.
2. 전역적인 서버 에러 처리 - [페이지 권한 처리, ErrorBoundary, 각 컴포넌트에서의 처리]

## BFF 서버가 클라이언트의 역할을 할 때, 브라우저와 달리 필요한 쿠키가 없는 상태에 대한 해결 방안

---

### 🫠 문제점

처음에는 무엇이 문제인지 이해하는 것 조차 어려웠다. BFF 서버가 클라이언트의 역할과 서버의 역할을 동시에 하고 있기에 눈에 보이지 않는 부분을 의도적으로 그려내서 이해해야 했기 때문이다.

로그인을 마친 유저가 (로그인이 필요한) 특정 페이지에 접속하려고 하면,

브라우저는 서버에게 페이지 요청을 하고, 이후에 실행되는 `getServerSideProps` 에서 필요한 데이터를 tanstack-query를 통해 prefetch 한다.

- 따라서 서버(BFF)에서 동작하는 코드임에도 다시 API 요청을 보내게 되는데, 이 때 브라우저가 가지고 있는 쿠키를 함께 전송하지 않고 있어서 `UnauthorizedError` 발생

서버에서 prefetch 결과가 에러였으므로 브라우저에서 다시 데이터 요청

- 이 때는 브라우저가 요청을 보내기 때문에 쿠키가 설정되어 있음

→ prefetching의 의미가 전혀 없는 SSR을 하고 있었다.

### 🛠️ 해결 방안

그러면 서버에서 보내는 요청일 경우에도 브라우저의 쿠키를 실어서 요청을 보내주면 문제가 해결되겠구나 싶었다.

먼저 기존의 `getServerSideProps`가 어떻게 짜여있었는지 살펴보면, `withSessionSsr`이라는 HOF를 통해 req의 쿠키를 읽고 검증한 뒤에 유효한 경우에 session에 유저 정보를 담아서 내려준다.

```jsx
// page/.../index.jsx

export const getServerSideProps = withSessionSsr(async (context) => {
    const queryClient = new QueryClient();
    const session = context.req.session; // HOF에서 담아준 유저 정보가 있음
    if (session) {
        /** session 정보 세팅 */
        await queryClient.prefetchQuery(...);
    }

		// 필요한 데이터 prefetch -> 이 때 쿠키를 함께 넘겨줘야 한다.
    await queryClient.prefetchQuery(...);

    return {
        props: {
            dehydratedState: dehydrate(queryClient),
        },
    };
});
```

**방안 1 | 명시적으로 헤더에 쿠키를 매번 넣어주기**

- prefetching시 쿠키를 함께 파라미터로 전달
- 내부에서는 인자로 받은 쿠키를 직접 헤더에 설정하여 axios 요청을 보냄

**방안 2 | axiosInstance.interceptors 사용하기**

방안 1은 여러 곳에서 데이터를 추가적으로 넘겨주고, 헤더를 명시적으로 설정해주어야 하는데, 너무 귀찮고 중간에 변질될 가능성도 크다.

- 추가해주어야 하는 코드가 두 곳인데 이미 너무 복잡한 단계를 거쳐 요청이 가고 있는 상황에 이것까지 여러 곳에서 신경쓰게 되면 DX가 너무 떨어질 것 같았다.

따라서 `axiosInstance.interceptors`를 사용하여 1차적으로 리팩토링 했다.

- 매 요청 시 해당 요청을 가로채서 원하는 쿠키 값을 넣어줌 (공통 헤더를 설정하는 것이 아니고 각 요청에 대한 처리이므로 동시에 요청 보냈을 경우에도 쿠키가 덮어씌워지지 않음)
- `getServerSideProps`에서는 prefetching시 요청을 인터셉트할 수 있도록 처리된 axios 인스턴스를 전달

**방안 3 | 쿠키가 설정된 axiosInstance를 사용하기**

다만 위 방안 사용 시 각 요청간의 충돌을 방지하기 위해선(여러 요청이 동시에 들어와도 독립적인 각자의 쿠키를 가져야 함) 매 번 새롭게 인스턴스를 만들어서 보내야 했고, 그렇다면 굳이 `interceptors`를 사용하지 않아도 될 것 같다는 의견이 있었다.

**_thanks to._ @회사\_동료\_luke**의 생산적인 코드리뷰

> 처음에는 interceptors라는 새로운 것을 사용하는 것에 들떴었는데 덕분에 다시 논리적으로 생각을 할 수 있었고 불필요하게 인터셉터를 추가했다는 것을 깨달았다.

- 서버에서 보내는 요청은 `getServerSideProps` 에서 `createServerAxios` (쿠키가 담긴 axios 인스턴스를 반환하는 함수)를 통해 생성

이렇게 중간에 쿠키를 담아서 요청을 보낼 수 있도록 처리함으로써 문제를 해결할 수 있었다.

### 🌤️ 자잘한 시행 착오

`**queryClient.fetchQuery` 와 `queryClient.prefetchQuery`\*\*

- `prefetchQuery` 는 실패 시에 별도로 에러를 던지지 않고 결과에 에러를 담아서 보내준다.
  - `prefetch` 의 목적을 생각해보면 이해가 되는 동작이긴 하다. 오류 여부와는 상관없이 미리 데이터를 찌르는 것일 뿐이고 이후에 대한 결과는 데이터를 사용하는 곳에서 다시 요청을 보낼 지, 에러에 대해 처리할 지 결정하는게 맞는 것 같다.
- 실패 시 에러를 바로 받아서 처리하고 싶다면 `fetchQuery` 를 사용하자.

**`req.cookies` 와 `req.headers.cookie`**

- axios의 req 와 next 서버의 req 형식이 다르기 때문에 axios req.headers.cookie로 담아준 값을 next 서버쪽 코드인 checkAuthentication에서 읽지 못함 (req.cookies로 접근)
- 이를 해결하기 위해, api 라우트에서 사용하는 `checkAuth() -> withSessionRoute` 에서 `req.headers.cookie`를 `req.cookies`에 넣어줌
  - 실제로 `next-connext` 에서 하고 있는 작업 (근데 왜 서버에서 요청보냈을 때는 저 처리가 안되어있는 건지 정확하게 이해 x, 브라우저가 보냈거나 bff 가 보냈거나 어쨌든 같은 axios 요청이 그대로 api 라우트로 넘어가는게 아닌가?)
    ![https://github.com/goorm-dev/swcamp-site/assets/89910087/ed20efae-74ea-48d5-a8b0-53ed0a39d108](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/e7883c67-430c-4cdb-a14e-8ba50f475c6c/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA_2023-07-27_%E1%84%8B%E1%85%A9%E1%84%92%E1%85%AE_10.04.44.png)
    https://github.com/goorm-dev/swcamp-site/assets/89910087/ed20efae-74ea-48d5-a8b0-53ed0a39d108

## 전역적인 서버 에러 처리

- [페이지 권한 처리, ErrorBoundary, 각 컴포넌트에서의 처리]

---

### 📝 전체적인 컨셉

- 서버 **에러 핸들링 `_error.jsx`에서 하기**
  - 에러 발생 시 무조건 여기로 떨어짐
  - 다만 실제 에러 화면이 띄워지는 것은 production일 때만이고 dev에서는 에러 발생 화면이 보여짐 (for 디버깅)
    > `pages/_error.js` is only used in production. In development you’ll get an error with the call stack to know where the error originated from.
  - 개발 내내 권한 처리가 잘 되는지 확인할 일은 없을거니까 필요한 경우에만 production 빌드해서 확인
- **기존에 사용하고 있던 세션 관련 HOF에서 페이지 권한 처리도 함께 할 수 있도록 확장**
  - 페이지에 대한 권한 체크도 할 수 있도록
  - 지금은 페이지에 대한 권한 체크는 없고 api 요청을 보내는 경우에 그 요청에 대한 권한 확인 + 응답으로 확인이 가능한데, api 요청을 처음부터 보내지 않는 페이지에서 접근을 막아야 하는 경우가 있음

### 🫠 문제점

NEXT를 사용하지 않고 있는 다른 레포에서는 직접 서버를 구성하다 보니, 페이지에 대한 접근 권한 처리를 routes들이 모여 있는 한 곳에서 할 수 있었다.

그러나 NEXT 사용 시에는 흔하게 사용되는 패턴은 아닌지 에러가 발생했을 때 에러 페이지를 보여주는 것은 쉬웠으나, 우리가 구분해둔 에러 statusCode와는 무관하게 모두 500.js로 떨어져버렸다.

아래의 요구사항을 만족하는 구조를 만들어 내기 위해 여러 시도를 해보았다.

- 각 페이지의 `getServerSideProps` 마다 코드를 추가하고 싶지 않음
- API (axios) 에러와 인증과 관련해서 그냥 throw 하는 에러 모두 다룰 수 있어야 함

### 🛠️ 해결 방안

**방안 1 | [getServerSideProps를 try…catch 로 감싸주는 HOC](https://yceffort.kr/2021/10/api-error-handling-nextjs)** → ❌

- 말 그대로 `getServerSideProps`를 try...catch로 감싸서 발생한 에러에 대한 처리가 가능하다.
- 각 페이지마다 추가를 해줘야하므로 요구사항 1 을 만족시키지 못해서 패스했다.

**방안 2 | middleware.js** → ❌

- 미들웨어를 타는 시점에는 아직 해당 페이지 요청에 대한 응답이 완료되지 않아서 판단할 수가 없으므로 패스..

**방안 3 | ErrorBoundary 🤔**

- 이건 페이지 단에서 직접 처리하는 것과 다름이 없어보인다.
- 클라이언트 에러 처리에 적합한 구조
- 페이지 단에서 권한을 처리하기 위해 사용하기엔 목적에 맞지 않은 것 같아서 넘어간다.

**방안 4 | [\_app.js의 PageProps.error](https://suzie.world/posts/2021/error-handling-with-getserversideprops/)** → ❌

- `getInitialProps` 를 사용하는 \_error.js 페이지를 사용하고 던져진 에러를 \_app.js에서 한 번에 설정하는 방법이 있어서 살펴보았다.
- server side에서 발생한 에러는 안잡혀서 패스..

**방안 5 |** [\_error.js](https://nextjs.org/docs/pages/building-your-application/routing/custom-error#more-advanced-error-page-customizing) **✅**

- Next에서 제안하는 커스텀 에러 페이지 고도화 방법이다. (방안 4에서도 이 컨셉을 사용하였다.)
- 여기서 나는 `res.writeHead` 를 사용하여 에러에 따라 미리 만들어둔 에러 페이지(/403, ...)로 리다이렉트 시키는 방법을 차용했다.
  - 미리 처리하고 싶은 에러에 대해서는 각 페이지로 리다이렉트 시키고, 결과적으로 `_error.js`에서는 예상치 못한 에러만 다룰 수 있게된다.
- 그런데, 서버 에러 핸들러까지는 401, 403… 로 잘 오는데, \_error에서 받은건 `res.statusCode`는 일괄 500이다. 😭
  → response는 에러가 발생했으니까 500일 수 밖에..! 에러 객체를 보면 된다.

```jsx
// page/_error.js
CustomErrorPage.getInitialProps = ({ res, err }) => {
  const errorName = err.name || err.response?.data.code;
  let statusCode = 500;
  switch (errorName) {
    case "UnauthorizedError":
      statusCode = 401;
      break;
    default:
      break;
  }

  if (statusCode !== 500) {
    res.writeHead(302, { Location: `/${statusCode}` });
    res.end();
  }

  return { statusCode, myError: statusCode };
};
```

관련해서 여러 레퍼런스를 찾아보다가 나와 동일한 상황에 처한 사람의 이슈를 발견했고, 무려 1년 전 글이지만 누군가에게라도 도움이 되길 바라며 댓글을 달아두었다!

[How to handle unathorized errors thrown by the server · vercel/next.js · Discussion #39530](https://github.com/vercel/next.js/discussions/39530)

### 🌤️ 자잘한 시행 착오

- `_error.jsx`는 없다
  - Next에서 제공하는 예약어 페이지 셋이므로 이름을 정확하게 `_error.js`로 해야한다.
