---
layout: posts
title: "[WEB] DB 검색 결과 match순으로 정렬"
categories:
  - Web
last_modified_at: 2022-08-07
author_profile: true
tags:
  - 웹
  - 정렬
toc: true
toc_sticky: true
sidebar:
  title: Posts
  nav: "sidebar-contents"
---


## 검색 결과를 앞에서부터 일치하는 순서대로 정렬하고 싶을 땐?

feat. [stackoverflow 질문](https://stackoverflow.com/questions/47811340/how-to-sort-list-start-with-first-and-then-contains-string-search-in-md-autocomp)


**Situation**

우리 채널 검색에서 사용하는 검색 api에서, $regex: {contiansWith}로 find할 경우 우선순위가 매겨지지 않은(앞에서부터 일치하는 결과가 높은 우선순위를 가지지 않은) 결과를 받아온다.



**Try**

만약 앞에서부터 일치하는 결과를 상위에 배치하고자 할 경우, 간단하게는 startWith로 먼저 find 한 뒤에 다시 containsWith로 find 하는 방법이 있다.



- **Problem**
    
    그러나 위의 방법을 사용할 경우 find를 두 번 하게 되므로 시간이 오래 걸리므로 좋은 방법이 아니다.
    
    → find는 한 번만 하고 해당 결과를 클라이언트 단에서 sort 하는 방법이 있을 것이다.
    


**Better Try ...solution**

검색 결과를 reduce를 사용해, 각 항목의 [0]번째 값을 비교하여 일치하는지에 따라 state를 부여해 다른 배열에 담는다. 이후 state==0을 가지는 항목(앞에서부터 일치하는)이 담긴 배열과 아닌 배열을 차례로 concat 하여 결과값을 리턴해준다.



```js
function querySearch(searchInput, items) {
	console.log(0);
	const lowerdItems = items.map((item) => {
		const { channelIndex, channelName, businessName } = item;
		const lowerdItem = {
			...{ channelIndex: channelIndex.toLowerCase() },
			...{ channelName: channelName.toLowerCase() },
			...(businessName && { businessName: businessName.toLowerCase() }),
		};
		return lowerdItem;
	});

	const sortedItems = lowerdItems.reduce(
		function (acc, currentItem) {
			if (
				currentItem.channelIndex[0] === searchInput[0] ||
				currentItem.channelName[0] === searchInput[0] ||
				(currentItem.businessName && currentItem.businessName[0] === searchInput[0])
			) {
				acc[0].push(currentItem);
			} else {
				acc[1].push(currentItem);
			}

			return acc;
		},
		[[], []]
	);

	return sortedItems[0].concat(sortedItems[1]);
}
```