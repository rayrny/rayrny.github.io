---
layout: posts
title: "[프로그래머스] DP 등굣길 (C++)"
categories:
  - BOJ
last_modified_at: 2021-07-14
author_profile: true
tags:
  - 동적프로그래밍(DP)
  - Algorithm
toc: true
toc_sticky: true
sidebar:
  title: Posts
  nav: "sidebar-contents"
---

# 1463 | 1로 만들기


### 🙋‍♀️ 문제

-----

계속되는 폭우로 일부 지역이 물에 잠겼습니다. 물에 잠기지 않은 지역을 통해 학교를 가려고 합니다. 집에서 학교까지 가는 길은 m x n 크기의 격자모양으로 나타낼 수 있습니다.

아래 그림은 m = 4, n = 3 인 경우입니다.

가장 왼쪽 위, 즉 집이 있는 곳의 좌표는 (1, 1)로 나타내고 가장 오른쪽 아래, 즉 학교가 있는 곳의 좌표는 (m, n)으로 나타냅니다.

격자의 크기 m, n과 물이 잠긴 지역의 좌표를 담은 2차원 배열 puddles이 매개변수로 주어집니다. 오른쪽과 아래쪽으로만 움직여 집에서 학교까지 갈 수 있는 최단경로의 개수를 1,000,000,007로 나눈 나머지를 return 하도록 solution 함수를 작성해주세요.


### 🙋‍♂️ 예제 입출력

-----

|m |n |	puddles |return|
|--|--|---------|------|
|4|3|[[2, 2]|4|


### 🚀 코드

-----

```c++
int solution(int m, int n, vector<vector<int>> puddles) {
	int answer = 0;
	vector<vector<int>> map(n + 1, vector<int>(m + 1, 1));

	for (int i = 0; i < puddles.size(); i++) {
		int x = puddles[i][0];
		int y = puddles[i][1];

		// puddles를 map에 삽입
		map[y][x] = 0;
		// 웅덩이가 좌,상단 가장자리에 있을 경우
		// 하나 있으면 그 뒤에는 갈 필요가 없음 -> 막힌길, 우회도 할 필요 없음
		if (x == 1) {
			for (int j = y; j <= n; j++) {
				map[j][x] = 0;
			}
		}
		if (y == 1) {
			for (int j = x; j <= m; j++) {
				map[y][j] = 0;
			}
		}
	}

	for (int i = 2; i <= n; i++) {
		for (int j = 2; j <= m; j++) {
			if (map[i][j] == 0) {
				continue;
			}
			map[i][j] = (map[i - 1][j] + map[i][j - 1]) % 1000000007;
		}
	}

	answer = map[n][m];
	return answer;
}
```

### 🌠 메모

-----

일단 문제를 처음 봤을 때 DFS로 풀면 좋겠다 생각했는데 돌고 돌아 동적 프로그래밍으로 풀었다. 아이디어를 처음 생각하기가 어려웠는데 고등학교 수학문제에서 **경우의 수 구하듯** 풀면 된다.

따라서 점화식이 ```map[i][j] = map[i - 1][j] + map[i][j - 1]```이다.

1. 먼저 1로 초기화 한 map에 물 웅덩이를 옮긴다. 이때 좌, 상단 가장자리(x==1 or y==1)일 경우 그 뒤에는 모두 0으로 바꿔주자. map을 1로 초기화한 상태이기 때문에 그쪽 길은 경로를 셀 필요가 없으니 0으로 만드는 것이다. -> 이건 문제 풀이 스타일마다 많이 다를 것 같다. map, 즉 DP 테이블을 0으로 초기화 하느냐, 1이냐 -1이냐, 또 웅덩이를 0으로 두느냐 -1로 두느냐에 따라 코드가 달라지는 것 같다.
2. 다음으로는 점화식을 통해 구하기만 하면 된다. 여기서 중요한 점은 저장할 때 모듈러를 적용해야 하는 것이다. 전에 백준에서 큰 수 더하기였는지 무튼 비슷한 경우가 있었는데 점화식을 구할 때 더하기를 하므로 수를 작게 만들어서 저장해야 범위를 벗어나지 않는다.





