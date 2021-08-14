---
layout: posts
title: "[Express] res.send() 와 res.json()"
categories:
  - Express
last_modified_at: 2021-08-15
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


### ERR: Unexpected end of JSON input

이전에 진행한 코드를 참고하며 프로젝트를 진행중인데, api의 response를 보내줄 때 ```res.json()```을 사용한 것을 보았고 이후에 다른 코드를 보니 ```res.send()```를 사용하여 response를 보내는 것을 볼 수 있었다. 

응답을 보내는 거니까 send를 써야겠다고 마음 먹었는데, 클라이언트에서 json 객체를 읽지 못하는 상황을 마주쳤다! (내 기억 상, 위의 에러였다.) ```res.json()```으로 바꾸니 해결되었고 이제 그 둘의 차이점이 궁금해졌다.

#### res.send()

------

**send**의 경우, 데이터의 타입을 확인하여 **object** 타입일 경우 ```res.json()```을 호출하게 된다. 이외에 다른 타입(buffer, string, array 등)일 경우 그에 맞게 지정한다.

#### res.json()

----

res.json()은 응답으로 보내줄 데이터를 자동으로 ```json```형식으로 바꿔서 보내게 된다.

따라서, res.send()를 할 경우 데이터의 타입을 확인하여
json일 경우 res.json()을 호출한다고 생각하면 된다.

#### res.end()

------

찾아보니 응답을 보낼 수 있는 방법에 하나가 더 있었다. res.end()의 경우, 보내줄 데이터가 없을 경우, 즉 404를 리턴할 경우에 사용한다. (그러나 res.send(), res.json()을 써도 응답을 종료할 수 있기 때문에 굳이 사용하진 않는다.)


> https://yohanpro.com/posts/nodejs/express-response

