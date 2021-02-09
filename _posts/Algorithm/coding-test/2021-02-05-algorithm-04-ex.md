---
layout: posts
title: "[구현] Ch04-예제"
categories:
  - Algorithm
  - CodingTest
last_modified_at: 2021-02-05
author_profile: true
tags:
  - 구현
toc: true
toc_sticky: true
sidebar:
  title: Posts
  nav: "sidebar-contents"
---

# 예제 4-1| 상하좌우

<small>
코드만 보고 간단하게 생각했으나, 오랜만에 문자열을 사용하다보니 시행착오가 많았고, 책에 나와있는 솔루션이 독특해서 리뷰하게 됨
</small>

![예제 4-1](/assets/image/04-ex-1.PNG)

### 🌷 초기 코드:
```c++
#include <iostream>
#include <string>
using namespace std;

int main() {
	int n = 0;
	cin >> n;
	char in = 0;
	int x = 1;
	int y = 1;
	string str;
	getchar();
	getline(cin, str);
	for (int i = 0; i < str.size(); i++) {
		in = str[i];
		if (in == 'R') {
			if (x < n) {
				x += 1;
			}
		}
		else if (in == 'L') {
			if (x > 1) {
				x -= 1;
			}
		}
		else if (in == 'D') {
			if (y <= n) {
				y += 1;
			}
		}
		else if (in == 'U') {
			if (y > 1) {
				y -= 1;
			}
		}
	}
	cout << y << ' ' << x;
	return 0;
}
```
-----

### 🌼 코드를 짜며 마주친 에러 상황:
   - 숫자를 받고 엔터를 친 후 문자열을 받을 때는 ```getchar()```를 통해 입력시 치게되는 엔터를 버퍼로 받아놓아야 한다.
   - c++에서 공백을 포함하는 문자열을 받을 때는 ```getline(cin, string_name)```을 사용한다.
   - 변수를 선언할 때 0이 아닌 다른 수로 초기화하는 경우에는 각각 따로 해줘야 한다. > ```int x = 1; int y = 1;``` (한번에 ```int x, y =1;``` 하니까 x가 초기화 되지 않아서 에러발생함)

### 🍀 책에서의 솔루션:
L, R, U, D에 따른 이동 방향을 미리 선언해 두고, 명령어를 배열에 저장한 뒤에 명령어의 개수인 4 만틈 반복문을 돌린다. (위의 코드상에서는 <mark style='background-color: #f5f0ff'>if - else if 문</mark>으로 구현되어 있음)
<br>
dx, dy 배열에는 명령어가 저장된 배열의 순서에 따라 원하는 움직임을 만들 수 있도록 0과 1, -1을 적절히 배치한다.
<br>
이후 (x, y)가 공간을 벗어나는 경우를 무시하며 이동을 수행한다.

코드를 봤을 때 딱 느낀점은 전에 컴퓨터그래픽스 과제를 할때 움직이는 우주선을 구현했을 때 처럼 완성도 있게 짜여져 있다는 것이었다. 보통 명령어가 입력으로 주어지고 이를 수행하는 구현문제는 <mark style='background-color: #f5f0ff'>if문</mark>을 사용해서 각각의 명령어를 처리했는데 이런식으로 접근할 수 있다는 점이 매우 인상적이었다.


### 🌻 수정된 코드:
```c++
#include <iostream>
#include <string>
using namespace std;

int main() {
	int n = 0;
	cin >> n;

	int x = 1;
	int y = 1; // 실제 좌표
	string str;
	getchar();
	getline(cin, str);
	int dx[4] = { 0,0,-1,1 };
	int dy[4] = { -1,1,0,0 }; // 움직임 조절
	int nx, ny = 0; // 결과가 공간의 크기를 넘어가는지 확인
	char moveType[4] = { 'L','R','U','D' };
	for (int i = 0; i < str.size(); i++) {
		for (int j = 0; j < 4; j++) {
			if (str[i] == moveType[j]) {
				nx = x + dx[j];
				ny = y + dy[j];
			}
		}
		if (nx<1 || ny<1 || nx>n || ny>n) {
			// 범위 넘어가면 무시
			continue;
		}
		x = nx;
		y = ny;
	}
	cout << x << ' ' << y;
	return 0;
}
```
------
<br>
# 예제 4-2| 시각

이 문제는 백준 18312번과 동일하므로 링크 따로 첨부함

-<a href="https://jerimo.github.io/boj/boj-18312/">[백준] 18312 시각</a>
