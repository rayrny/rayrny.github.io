---
layout: posts
title: "[C/C++] 변수의 표현 범위"
categories:
  - C
  - C++
last_modified_at: 2021-02-01
author_profile: true
tags:
  - 프로그래밍 언어
  - C/C++
toc: true
toc_sticky: true
---

## C/C++에서 변수 표현 범위

이번 포스팅에서는 문제를 풀때 자주 놓치는 부분인 메모리 제약 사항에 대해서 알아보겠다.

앞으로 문법 태그에서는 코딩을 할때 헷갈리는 문법이나 메모리 제약 사항등을 다룰 예정이다. (ex. 실수형 변수 유효숫자 출력, 문자열 파싱 등)

|정수형 종류|자료형 크기|자료형 범위|
|------|---|---|
|int|4 byte(32 bit)|-2,147,483,648 ~ 2,147,483,647|
|long long|8 byte(64 bit)|-9,223,372,036,854,775,808 ~ 9,223,372,036,854,775,807|
|BigInteger(클래스)|가변적|제한 없음|


** 대체로 <mark style='background-color: #f5f0ff'>long long</mark>에서 다룰 수 있는 수보다 더 큰 정수를 처리하는 문제는 잘 출제되지 않음.
