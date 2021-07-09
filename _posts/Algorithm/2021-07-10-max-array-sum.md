---
layout: posts
title: "[HakerRank] DP Max Array Sum (C++)"
categories:
  - Algorithm
last_modified_at: 2021-07-10
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

# DP | Max Array Sum


### 🙋‍♀️ 문제

-----

Given an array of integers, find the subset of non-adjacent elements with the maximum sum. Calculate the sum of that subset. It is possible that the maximum sum is 0 the case when all elements are negative.

The following subsets with more than  element exist. These exclude the empty subset and single element subsets which are also valid.

### 🙌 입출력

-----

The first line contains an integer, n. (1 <= n <= 10^5)
The second line contains n space-separated integers arr[i]. (-10^4 <= n <= 10^4)


### 🙋‍♂️ 예제 입출력

-----

```
5
3 5 -7 8 10
```

```
15
```

### 🚀 코드

-----

```c++
#include <iostream>
#include <cstdlib>
#include <vector>
#include <algorithm>
using namespace std;

int maxSubsetSum(vector<int> arr) {
    int len = arr.size();
    vector<int> dp(len); // DP table
    
    dp[0]=arr[0];
    dp[1]=max(arr[0], arr[1]); // dp[1] = arr[1] 하면 안됨!

    for (int i = 2; i < len; i++) {
        dp[i] = max(arr[i], max(arr[i] + dp[i - 2], dp[i - 1]));
    }

    return dp[len-1];
}


int main() {
    int n = 0;
    vector<int> arr;
    int x = 0;
    cin >> n;
	for (int i = 0; i < n; i++) {
        cin >> x;
        arr.push_back(x);
	}
    cout << maxSubsetSum(arr);

	return 0;
}
```

### 🌠 메모

-----

프로그래머스처럼 입출력은 신경 쓰지 않고 ```maxSubsetSum```함수 부분만 작성하면 된다.

오랜만에 다시 동적 프로그래밍 문제를 풀어봤는데, 처음 시도했을 때 만큼 basic한 아이디어를 떠올리는게 오래 걸리진 않았다.(15분이 채 안걸린 것 같다. 물론 오류 고치고, 찾아보고 하면서 총 걸린 시간은 1시간 정도..) 그런데 자꾸 예제 입출력만 성공하고 다른 테스트케이스는 틀렸다.. 😥

일단 처음 문제에서 **non-adjacent elements**에 집중하는 바람에 dp 테이블에 저장할 때도 ```(arr[i] + dp[i - 2], arr[i] + dp[i - 3])```를 비교해서 저장했다. (다행히 금방 오류를 잡았다.)

#### 📌Point

- 아주 기본적인 부분이지만, ```vector<int> dp;```로 선언했을 때는 크기가 지정되지 않았기 때문에 ```dp[i]```라는 위치에 **접근이 불가능**하다.
- 크기 비교는 if문 말고 ```algorithm```헤더의 ```min, max```이용하자. 자주 안쓰다 보니 까먹는 것 같다.
- dp[i] 위치 까지의 최댓값을 구할 때 지금처럼 i번째 원소에서 시작해서 더한 값이 더 클 수 있다.(설명을 잘 못하겠는데, i-1까지의 원소가 모두 음수이고 i부터 양수일 경우는 arr[i] + dp[i - 2], dp[i - 1]를 아무리 해도 arr[i]보다 작다.)```max(arr[i], max(arr[i] + dp[i - 2], dp[i - 1]))```
- 테스트 케이스 일부만 통과할 경우 위처럼 여러 경우를 잘 생각해보자.
- dp 테이블을 뒤에서부터 구성하면 dp[i - 1], arr[i] + dp[i - 2]만 비교해도 된다. 아직 명확하게 깨닫지는 못했지만, 뒤에서 부터 더해오면 이전까지의 원소의 값이 상관 없기? 때문인 것 같다. (i-1까지의 원소가 모두 음수이고 i부터 양수일 경우 -> 를 신경 안써도 되기 때문이지 않을까?)
- 코드에 적용은 안해봤는데, 공간 복잡도를 줄이는 방법을 찾았다. ```no need for a huge array dp(n) ! it's enough to store "last" and "before"``` 댓글의 내용인데, dp 테이블을 모두 가질 필요 없고 dp[i], dp[i-1], dp[i-2]를 변수에 저장하는 식으로 하면 될 것 같다.

<small>참고를 위해 뒤에서 부터 dp 테이블을 만드는 코드도 첨부한다.</small>

```c++
int maxSubsetSum(vector<int> arr) {
    int len = arr.size();
    vector<int> dp(len + 3);

    for (int i = len - 1; i >= 0; i--) {
        dp[i] = max(arr[i] + dp[i + 2], dp[i + 1]);
    }
    return dp[0];
}turn dp[len-1];
}
```