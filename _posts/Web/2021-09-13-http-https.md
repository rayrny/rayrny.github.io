---
layout: posts
title: "[WEB] HTTPvs HTTPS"
categories:
  - Web
last_modified_at: 2021-09-13
author_profile: true
tags:
  - 웹
  - http
  - https
toc: true
toc_sticky: true
sidebar:
  title: Posts
  nav: "sidebar-contents"
---

### HTTPS = HTTP + 데이터 암호화

HTTP는 서버/클라이언트 간 데이터를 주고받기 위한 프로토콜이다. 한마디로, 하이퍼텍스트를 교환하기 위한 통신 규약(프로토콜)으로 80번 포트를 사용하고 있다. 

또한 HTTP는 애플리케이션 레벨의 프로토콜로 TCP/IP 위에서 작동한다. 또한 Stateless하다.

<aside>
💡 **TCP/IP란?** 
컴퓨터들이 서로 정보를 주고받는데 쓰이는 통신 규약(프로토콜)의 모음인 Internet Protocol Suite의 한 종류이다. 인터넷 계층은 IP 프로토콜을 사용해 각 패킷의 전송을 책임지고, 전송 계층에서는 TCP 프로토콜을 사용해 전체 데이터의 전달을 책임진다.

TCP/IP는 패킷 통신 방식의 프로토콜인 IP와 전송 조절 프로토콜인 TCP로 이루어져 있다. IP는 패킷 전달 여부를 보증하지 않고, 패킷을 보낸 순서와 받는 순서가 다를 수 있다.(unreliable datagram service) TCP는 IP 위에서 동작하는 프로토콜로, 데이터의 전달을 보증하고 보낸 순서대로 받게 해준다. [HTTP](https://ko.wikipedia.org/wiki/HTTP), [FTP](https://ko.wikipedia.org/wiki/%ED%8C%8C%EC%9D%BC_%EC%A0%84%EC%86%A1_%ED%94%84%EB%A1%9C%ED%86%A0%EC%BD%9C), [SMTP](https://ko.wikipedia.org/wiki/SMTP) 등 TCP를 기반으로 한 많은 수의 애플리케이션 프로토콜들이 IP 위에서 동작하기 때문에, 묶어서 TCP/IP로 부르기도 한다.
*- 위키백과*

</aside>

### HTTPS

위에서 말했듯이 HTTPS는 HTTP에 데이터 암호화가 추가된 프로토콜이며, 443 포트를 사용한다. 

공개키와 개인키를 통해 데이터를 암호화하는데, 공개키를 통해 암호화하면 개인키로만 복호화할 수 있고 개인키를 통해 암호화하면 공개키를 통해서만 복호화할 수 있다.

> FLOG 에서 애를 먹었던 부분이 바로 이에 대한 부분..! 실제로 HTTPS를 사용하려면 인증서를 발급받아야 하는데 졸업작품에서 사용할 것이어서 따로 돈을 내거나 하지 않았기 때문에..
> 

무튼 브라우저에서는 인증서를 발급한 기관의 공개키를 미리 가지고 있기 때문에 개인키로 암호화된 인증서를 복호화할 수 있다.