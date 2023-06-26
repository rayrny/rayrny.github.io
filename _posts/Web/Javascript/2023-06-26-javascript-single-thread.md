---
layout: posts
title: "[JS] 자바스크립트 엔진의 동작 방식 (feat. 싱글 스레드, Event loop, Web API)"
categories:
  - Javascript
last_modified_at: 2023-06-26
author_profile: true
tags:
  - 웹
  - Javascript
  - 싱글 스레드
  - 비동기
toc: true
toc_sticky: true
sidebar:
  title: Posts
  nav: "sidebar-contents"
---

싱글 스레드 기반 언어인 Javascript는 어떻게 사용자 경험을 해치지 않고 많은 요청들을 동시에 처리할 수 있을까?

### 자바스크립트 엔진

자바스크립트는 콜스택과 메모리힙이라는 메모리 구조를 통해 데이터 및 코드 실행을 관리한다. 자바스크립트 자체가 비동기 동작을 지원하는 것은 아니다. 자바스크립트 엔진은 단순히 태스크가 요청되면 콜 스택을 통해 요청 작업을 순차적으로 실행할 뿐이다.

👉 **엔진의 주요 구성요소**

1) **Memory Heap** : 메모리 할당이 일어나는 곳

2) **Call Stack** : 코드 실행에 따라 호출 스택이 쌓이는 곳

### 자바스크립트는 어떻게 한번에 여러 요청을 처리할까?

// TODO: 노션에 긁어온 정보들을 토대로 내가 갈무리를 한 번 한 뒤에 작성할 것
