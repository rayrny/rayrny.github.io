---
layout: posts
title: "[React] TypeError: Cannot read property '0' of undefined 에러"
categories:
  - React
last_modified_at: 2021-07-21
author_profile: true
tags:
  - 웹 프레임워크
  - react
  - error handling
toc: true
toc_sticky: true
sidebar:
  title: Posts
  nav: "sidebar-contents"
---


### props가 array 임에도 불구하고 에러 발생!

JSON으로 받아온 값을 하위 컴포넌트에 props로 넘겼는데, 하위 컴포넌트에서 ```data[0]```로 접근하려 하니 오류가 뜬다.

정확하게는 상위 컴포넌트에서 map으로 하위 컴포넌트를 뿌리는 상황이었다.

여기 저기 검색해보니 처음 mount 될 때는 undefined인 상태에서 [0]으로 접근하기 때문에 발생하는 문제인 것 같았다. 

따라서 바로 접근하지 않고 검증을 한 뒤에 접근하도록 하자!

```javascript
	useEffect(() => {
		// componentDidMount
		setPhoto(props.photo);

		if(photo.data && photo.data.length > 0) {
			console.log(photo.data[0]);
		}
	})
```

data가 존재하고 그 길이가 0 이상일 때만 접근하도록 하니 에러가 사라졌다!

props로 받아오는 데이터의 경우 이렇게 검증을 먼저 한 후에 사용하도록 하자.