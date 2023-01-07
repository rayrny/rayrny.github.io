

### scroll 버그 해결 과정

**problem:** 

채팅을 더 불러오면 스크롤이 더 불러온 채팅의 젤 위로 가는 것이 아니라 이전에 보고 있던 위치에 머물러 있었으면 좋겠다.

**reference:** [https://www.neoito.com/blog/how-to-maintain-scroll-position-in-angular-chat-app/](https://www.neoito.com/blog/how-to-maintain-scroll-position-in-angular-chat-app/)


브라우저 스크롤의 원래 동작 방식은 top에서 얼만큼 떨어져 있는가를 기억하고 유지한다. 때문에 아래와 같은 상황에서 원하는 대로 동작하지 않는다.

```
—
a → 스크롤 위치, top에서 1줄 떨어져 있다.
b
c
—
```

여기서 채팅을 더 불러오게 되면,

```
—
D → 스크롤은 이전에 top에서 1줄 떨어져 있었으므로 채팅이 추가된 후에도 이곳으로 이동한다.
E
F
- -(추가된 채팅)
a
b
c
—
```

문제는 `top에서부터의 위치를 기억`하기 때문인데, 그렇다면 `bottom에서부터의 위치를 기억`하면 되지 않을까? 가 위 게시물의 해답이었다.

아래의 setPrevScrollTop을 통해 채팅을 더 불러오기 이전 스크롤이 bottom에서 얼만큼 떨어져 있는지를 구한다. height에서 top을 뺀 값이다!

```jsx
const handleTopObserver = ([entry]) => {
// NOTE: 이전 스크롤 위치가 bottom에서 얼만큼 떨어져 있었는지 기억한 뒤, 채팅을 더 불러온 뒤에 기억한 위치로 이동함으로써 스크롤 위치를 유지
setPrevScrollTop(bodyRef.current.scrollHeight - bodyRef.current.scrollTop);
if (entry.isIntersecting && !isLoadingChatList && !isEndOfChat) {
onGetMoreChats(roomId);
}
};
```

그리고 채팅을 더 불러온 후(로딩이 끝나면) 스크롤을 전체 높이에서, 이전에 계산한 bottom 높이만큼 뺀 값으로 이동시킨다. 그러면 내 스크롤은 다시 밑에서부터 prevScrollTop 값만큼 떨어져 있게 되는 셈이다.  

```jsx
useEffect(() => {
if (!bodyRef.current) {
return;
}
	bodyRef.current.scrollTop =
		!isLoadingChatList && bodyRef.current.scrollHeight - prevScrollTop;
}, [isLoadingChatList]);

```

계산식이 이해되지 않는다면 그림을 직접 그려보는 것을 추천!

성공한 채팅 : [https://user-images.githubusercontent.com/89910087/163533446-0cb16df4-544a-4215-b089-4c46f726f218.mov](https://user-images.githubusercontent.com/89910087/163533446-0cb16df4-544a-4215-b089-4c46f726f218.mov)

실패한 채팅: [https://user-images.githubusercontent.com/89910087/163533475-b6a63e34-d1e6-4f7e-8f9b-4eca97f97310.mov](https://user-images.githubusercontent.com/89910087/163533475-b6a63e34-d1e6-4f7e-8f9b-4eca97f97310.mov)

