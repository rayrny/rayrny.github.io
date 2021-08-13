---
layout: posts
title: "[Express] Unhandled promise rejection 에러"
categories:
  - Express
last_modified_at: 2021-08-10
author_profile: true
tags:
  - 웹 프레임워크
  - express
  - error handling
toc: true
toc_sticky: true
sidebar:
  title: Posts
  nav: "sidebar-contents"
---

### 비동기 함수에서의 Promise 함수 사용

-----

async-await 에서 예외 처리를 안해서 발생하는 오류이다. 아래의 오류 메시지를 잘 읽어보면 catch() 써야한다는 식의 내용이 있다.

결론부터 말하자면 잘 사용중이었고, 전혀 다른 부분(mongoose 스키마 관련)에서 오류가 난 것이었지만 정리해두면 나중에라도 도움이 될 것 같아 포스팅을 따로 적는다.

내가 마주친 에러는 이것이다.

```
 UnhandledPromiseRejectionWarning: Unhandled promise rejection. This error originated either by throwing inside of an async function without a catch block, or by rejecting a promise which was not handled with .catch(). 
 To terminate the node process on unhandled promise rejection, use the CLI flag `--unhandled-rejections=strict` (see https://
nodejs.org/api/cli.html#cli_unhandled_rejections_mode). (rejection id: 1)
(node:2265) [DEP0018] DeprecationWarning: Unhandled promise rejections are deprecated. In the future, promise rejections that are not handled will terminate the Node.js process
with a non-zero exit code.
```

간단하게 정리하자면, 어떤 함수에서 Promise 함수를 호출할 때, 꼭 ```try-catch```문을 사용해야 한다. 비동기 함수는 ```try{}``` 안에서 호출해야 한다.

```javascript
// 아래애서 호출하게 될 비동기 함수
const unPackFile = (file, user) => {
	return new Promise((resolve, reject) => {
		  //...(내용 생략)
    if(...) {
			resolve(true);
		} else {
			reject(e);
		}
	});
}

// 위의 비동기 함수를 호출하는 부분
async (req, res) => {
	try {
    // 비동기 함수 호출은 try 안에서!!
		const unPacked = await unPackFile(file, user);
	  //...(내용 생략)
		}
		
	} catch (e) {
    // 여기서 catch를 사용해서 에러 처리!
		console.log(e);
	}
}

```

#### Express의 error-handling 미들 웨어

-----

```WrapAsync```와 같이 헬퍼 함수를 사용하여 에러를 처리한다. 각각의 라우터에서 에러 핸들링을 하지 않고 한번에 처리하는 방법이라고 한다.

(사실 이해하지 못했다.. 그치만 나중에 조금이라도 성장한 내가 보면 알 수 있지 않을까? 무튼 중요한 방법인 듯 해서 적어둔다.)

아래 블로그에 가면 정확한 내용이 적혀있다. 

> https://yohanpro.com/posts/nodejs/error-handling


늘 그렇듯 체계적으로 공부를 하고 시작을 하기보단 프로젝트에 필요하기 때문에 찾아보며 공부하는 나... 🤦‍♀️ 덕분에 에러 하나 해결하는데 한나절이 걸렸다.