---
layout: posts
title: "[React] react-query와 선언적인 에러 처리"
categories:
  - React
last_modified_at: 2023-07-30
author_profile: true
tags:
  - DX/UX
  - react-query
  - react
toc: true
toc_sticky: true
sidebar:
  title: Posts
  nav: "sidebar-contents"
---

회사에서 개발 시에 에러 처리가 고도화되어 있지 않아, 에러 발생 시 에러가 페이지 전역으로 퍼져버려 좋지 않은 경험을 했었다.

또한 다들 많이 신경쓰고 있지 않는 부분이다 보니 처리도 제각각이고 어떤 곳은 에러 처리조차 되어 있지 않은 곳이 많았기에, 이를 해결해보고 싶었다.

## 개요

[React Query Error Handling](https://tkdodo.eu/blog/react-query-error-handling)
위 문서를 참고하여 react-query와 함께 적용할 수 있는 에러 처리에 대한 구조를 잡아보고자 한다.

- 나아가 에러를 한 곳으로 모아 로깅하고 종류를 나누고 알림을 줄 수 있는 구조가 필요

### 목표

- 사용자에게 있어 에러 발생 시 에러의 종류에 따라 세분화 된 안내 제공
  - 예상한 에러(CustomError 등을 사용하여 처리하고 있는 에러)와 예상하지 못 한 에러
- ErrorBoundary를 조금 더 작은 단위로 감싸도록 해서, 사용할 수 있는 UI는 사용할 수 있도록 함

---

## TL;DR

크게 두 가지로 에러 상황을 구분하고 그에 따른 적절한 피드백을 제공하고자 했다.

### 1. 예상한 에러이고, 상황에 대한 피드백과 새로고침 UI를 제공하고 싶어요!

네트워크 에러와 같이 일시적인 장애 상황이거나 사용자가 다른 응답을 제출해 정상적인 응답을 받을 수 있는 경우

`**ErrorResetBoundary**`

- react-query와 함께 사용합니다.
- 내부적으로 react-error-boundary를 wrapping한 자체 ErrorBoundary를 사용중입니다.
- onError에서는 일반적으로 사용자에게 제공할 수 있는 응답을 토스트로 띄웁니다.
- 해당 에러 바운더리에서 처리할 수 없는 에러일 경우(예상치 못한 에러) 상위 에러 바운더리로 에러를 던집니다.

```jsx
import {
  QueryErrorResetBoundary,
  useQueryErrorResetBoundary,
} from "react-query";

function ErrorResetBoundary({
  children,
  fallbackRender,
  onError,
  onReset,
  ...props
}) {
  const { reset } = useQueryErrorResetBoundary();

  const handleError = (error) => {
    if (!isExpectedError(error)) {
      throw error;
    }

    if (!onError) {
      toast(`정보를 불러오는데 실패했습니다. ${error.response.data.err.msg}`, {
        type: toast.TYPE.ERROR,
      });
    } else {
      onError(error);
    }
  };

  const handleReset = () => {
    if (!onReset) {
      reset(); // 캐싱되어 있는 에러를 초기화
    } else {
      onReset(); // 이 경우에도 useQueryErrorResetBoundary().reset을 수행해야 함
    }
  };

  return (
    <QueryErrorResetBoundary>
      <ErrorBoundary
        onError={handleError}
        onReset={handleReset}
        fallbackRender={fallbackRender ?? ErrorRetryFallback}
        {...props}
      >
        {children}
      </ErrorBoundary>
    </QueryErrorResetBoundary>
  );
}
```

**Default Fallback**

```jsx
function ErrorRetryFallback({ resetErrorBoundary }) {
  // react-error-boundary에서 자동으로 넣어주는 resetErrorBoundary을 통해 에러 리셋 가능

  return <ErrorBox onClick={resetErrorBoundary} />;
}
```

### 2. 예상하지 못 한 에러이고, 해당 에러는 사용자가 해결할 수 없어요!

사용자에게 문의할 수 있는 UI를 제공해야 하고, 이 때 발생한 에러는 모니터링 되어야 하는 경우

앱 최상단에 감싸는 에러 바운더리도 이 경우에 해당됨

`**ErrorBoundary**`

- react-error-boundary를 wrapping한 컴포넌트
- onError에서는 기본적으로 예상치 못한 에러를 console.error로 찍어 로그를 남길 수 있다.
- react-query와 함께 사용하는 경우 아래에서 설명하는 `Default Fallback`을 통해 전역적 처리를 추가할 수 있다.

```jsx
function ErrorBoundary({
  children,
  fallbackRender,
  fallback,
  onError,
  onReset,
  ...props
}) {
  return (
    /**
     * fallback UI props의 우선순위
     * 1. fallback={<CustomFallback />}
     * 2. fallbackRender={() => <CustomFallback />}
     * 3. FallbackComponent={CustomFallback} // 구조분해 할당이 되어 있지 않음
     *
     * ex1) <ErrorBoundary />
     * : defaultProps로 지정되어 있는 <ErrorFallback /> 렌더링 (fallbackRender)
     *
     * ex2) <ErrorBoundary fallbackRender={() => <MyFallback />} />
     * : <MyFallback /> 렌더링
     *
     * ex3) <ErrorBoundary fallback={<MyFallback />} />
     * : defaultProps로 fallbackRender가 지정되긴 하지만, fallback의 우선순위가 높으므로 <MyFallback /> 렌더링
     */
    <ReactErrorBoundary
      fallbackRender={fallbackRender}
      fallback={fallback}
      onError={onError}
      onReset={onReset}
      {...props}
    >
      {children}
    </ReactErrorBoundary>
  );
}
```

**Default Fallback**

- 페이지 전체를 덮는 fallback

```jsx
function ErrorFallback({ resetErrorBoundary }) {
  const handleClick = () => {
    resetErrorBoundary();
    history.push(-1);
  };

  return (
    <div className={styles.errorWrapper}>
      <div className={styles.warningBox}>
        <WarningIcon width="28" height="28" />
      </div>
      <div className="text-gray-600">
        예상치 못한 문제가 발생했습니다.
        <br /> 잠시 후 다시 시도해 주세요.
      </div>
      <Button onClick={handleClick} color="basic" outline>
        <BackPageIcon />
        뒤로가기
      </Button>
    </div>
  );
}
```

**react-query를 통해 예상하지 못한 에러에 대해 전역적 처리를 할 경우**

- 앱단에서 queryClient를 선언할 때 글로벌 콜백을 등록할 수 있다.
- 이를 통해 fallback UI만 제공할 뿐 아니라 추가적으로 `서버에서 예상치 못한 에러가 발생했습니다.` 토스트를 일괄적으로 제공할 수 있다.

```jsx
const queryClient = new QueryClient({
	queryCache: new QueryCache({
		onError: (err) => {
			handleUnExpectedError(err);
		},
	}),
	mutationCache: new MutationCache({
		onError: (err) => {
			handleUnExpectedError(err);
		},
	}),
	...
});
```

---

## 기존 방식의 문제점을 분석하고 해결 방안을 제안합니다.

### AS-IS

- 에러 바운더리를 적절한 단위로 감싸 사용하지 않는 경우가 많아, 한 컴포넌트에서 예상치 못한 에러 발생 시 에러가 전역으로 퍼져 화면 전체를 덮는 fallback 렌더링
  ![AS-IS](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/fd661f52-52c0-44cb-9d11-40c6735bbc2a/Untitled.png)
  AS-IS
- 발생한 에러에 대해 처리 방식이 산재되어 있음
  - query, mutation의 onError 콜백을 통한 지역적 처리
  - 특정 컴포넌트에서 isError를 확인해 명령적으로 처리
  - 어떤 에러 처리도 되어있지 않아 에러 발생 시 전역으로 에러가 퍼짐

### TO-BE

- 에러 바운더리를 적절한 단위로 감싸 사용하지 않는 경우가 많아, 한 컴포넌트에서 예상치 못한 에러 발생 시 에러가 전역으로 퍼져 화면 전체를 덮는 fallback 렌더링
  TeachConsoleApp 기준으로 PageLayout 컴포넌트 내부를 적절한 단위 에러 바운더리로 감싸 예상치 못한 에러가 발생했을 경우, 사용할 수 있는 컴포넌트는 사용 가능하도록 변경
  - 에러가 발생했을 경우 에러가 발생한 컴포넌트에 에러를 가둠
  - retry UI를 제공하는 ErrorResetBoundary 사용 시에, 해당 에러 바운더리에서 처리할 수 없는 에러의 경우 에러를 던져 상위 에러바운더리에서 처리할 수 있도록 함

![TO-BE | ErrorBoundary](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/52d50da8-7003-4e3a-8e21-6ca489cbc934/Untitled.png)

TO-BE | ErrorBoundary

![TO-BE | ErrorResetBoundary](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/629be811-20b3-4998-9cfb-a58464f2ada4/Untitled.png)

TO-BE | ErrorResetBoundary

- 발생한 에러에 대해 처리 방식이 산재되어 있음

  ErrorBoundary의 onError를 통해 에러 처리 방식을 일원화하고, 추가적으로 react-query의 queryClient 선언 시 queryCache, mutationCache에 글로벌 콜백을 등록해 전역적 처리가 가능

  - isError를 확인해 컴포넌트에서 명령적으로 에러 처리하는 경우는 모두 제거 가능
  - query의 onError 콜백 사용 시 에러 처리에 대한 응집도가 낮아지는 것을 ErrorBoundary의 onError를 통해 해결

**TkDodo가 추천하는 방식에 대해 현 구조 분석**

> 원하는 대로 혼합하고 일치시킬 수 있습니다. 제가 개인적으로 하고 싶은 일은 백그라운드 재검색(오래된 UI를 그대로 유지하기 위해)에 대한 오류 토스트를 표시하고 다른 모든 작업을 로컬 또는 Error Boundaries로 처리하는 것입니다.

글 초반에 소개한 TkDodo의 게시물을 토대로 현 구조를 살펴보았고 어떤 것이 문제인지, 어떤 방식으로 해소할 수 있는지 고민하며 UX와 DX를 모두 개선할 수 있는 방안을 고안할 수 있었다.

올해 초 ErrorBoundary와 Suspense를 도입한 것에 이어, 에러 처리를 개선할 수 있는 방안을 고민해 볼 수 있어서 의미있는 시간이었다.

_새롭게 알게된 것들_

- isError 플래그
  - 백그라운드(re-fetch) 오류를 잘 해결하지 못함. 백그라운드 리패치가 실패했을 경우 쿼리는 에러와 이전 데이터를 모두 가지고 있음. 에러가 있다고 바로 에러 카드로 보여주는 것 보다 더 다양한 처리를 생각해볼 수 있음
- useQuery의 onError 콜백
  - 하나의 쿼리에 여러 옵저버가 붙어있을 경우 onError 콜백도 여러번 호출됨
    ![onError](https://github.com/rayrny/goo2/assets/48341341/9d5be1ff-3a4a-481c-ac2a-e65ccfffa04c)
- `react-error-boundary` fallback UI props의 우선순위
  - 기존 `ErrorBoundary`를 개선하며 fallback의 우선순위에 대해 알게되었다. 이전에는 이러한 우선순위를 알 지 못해서 특정 props만 받도록 강제했었는데, 함께 일하는 동료가 직접 코드를 까보며 우선순위가 있었음을 알려주었다.

---

**스페셜 땡스 투. @제로 @루키**

- 관련해서 좋은 의견 주셔서 더 나은 구조를 만들어 낼 수 있었습니다. 감사해요 :>

**참고했던 글 들**

- [클라이언트의 사용자 중심 에러 처리](https://jbee.io/react/error-declarative-handling-2/)
- [React Query와 함께 Concurrent UI Pattern을 도입하는 방법](https://tech.kakaopay.com/post/react-query-2/)
