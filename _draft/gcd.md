---
layout: posts
title: "[알고리즘] GCD LCM"
categorise:
  - Algorithm
last_modified_at: 2021-02-05
author_profile: true
tags:
  - C/C++
  - 유클리드 호제법
toc: true
toc_sticky: true
sidebar:
  title: Posts
  nav: "sidebar-contents"
---


## 최대공약수와 최소공배수를 구하는 알고리즘을 구현해보자.

### | 유클리드 호제법을 통한 최대공약수 구하기
이 알고리즘을 사용하면 시간 복잡도를 O(log N)으로 줄일 수 있다.

두 자연수 a, b에 대해 a를 b로 나눈 나머지를 r이라 할때 a와 b의 최대공약수는 b와 r의 최대공약수와 같다는 원리를 이용한다.

이를 구현하는 방법은 크게 반복문과 재귀 두 가지가 있으며, 간단하게 반복문을 사용해서 구현해보자.

```c++
int getGcd(int a, int b) {
	int tmp = 0;
	if (a < b) { // 대소 비교를 통해 항상 큰수가 a가 되도록 함
		tmp = a;
		a = b;
		b = tmp;
	}
  int r = 0;
	while (b != 0) {
		r = a % b;
		a = b;
		b = r;
	}
	return a;
}
```

재귀문을 사용하면 코드가 보다 간단해진다.

```c++
int getGcd_r(int a, int b) {
	if (b == 0) {
		return a;
	}
	return getGcd_r(b, a%b);
}
```


### | 최소공배수 구하기
최대공약수를 구했다면 최소공배수를 구하는 방법은 매우 간단하다.
a와 b를 곱한 값을 최대공약수로 나눠주면 구할 수 있다.

```c++
int getLCM(int a, int b) {
	int gcd = getGcd(a, b);
	return (a*b) / gcd;
}
```
