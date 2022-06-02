---
layout: posts
title: "[JS] mouse-event 우선순위 파해치기"
categories:
  - Javascript
last_modified_at: 2022-05-16
author_profile: true
tags:
  - 웹
  - Javascript
toc: true
toc_sticky: true
sidebar:
  title: Posts
  nav: "sidebar-contents"
---


### Mouse Event

저번에 이어 마우스 이벤트로 한참 애를 먹었다. 어제는 이녀석 때문에 마크까지 동원되었다.

저번같은 경우는 검색바의 포커스 이벤트와 검색 결과 드롭다운의 클릭 이벤트가 충돌하여 발생했었고, 그 때 나왔던 해결책 중 하나가 e.stopPropagation(), 부코 태그로부터의 이벤트 전파를 막는 역할을 하는 친구다. 얘랑 단짝이 바로 e.preventDefault(). 요녀석은 이전에 채팅기능에서 엔터 key press에 대한 동작을 커스텀하기 위해 사용한 적이 있다. 

서론이 길었는데, 이제 본격적으로 마우스 이벤트에 대해 알아보자.

마우스 이벤트 우선순위:

onMouseDown > onBlur > onClick

위의 우선순위로 인해, 많은 스택 오버플로우 글에서 블러 이벤트가 안먹혀요 ㅜㅜ 이러면 다들 onMouseDown으로 바꿔보라고들 한다.

그치만 나는 위의 경우는 아니었고,  나는 버튼 클릭시 이모지 패널이 나오고, 이모지 패널에서 클릭을 하면 당연하게 이모지가 클릭되어야 하고, 패널 바깥을 클릭하면 패널이 포커스를 잃고 다시 접히는 동작을 원했다.

이런 저런 시도에서, 블러 이벤트를 부여하는 순간 클릭 이벤트가 전혀 동작하지 않았고, 약 1시간의 사투 끝에 별 생각없이 이것도 안되겠지 하면서 시도한 방법이 모든 문제를 해결해 주었다.

주인공은 바로 FocusEvent.relatedTarget()

MDN을 보면 말 그대로 `seconadry target, depending on the type of event` 이고, 함께 표로 정리되어 있어 자세한 경우는 문서를 참고하면 될 것 같다.

blur 이벤트의 경우, blur의 target은 `The [EventTarget](https://developer.mozilla.org/en-US/docs/Web/API/EventTarget) losing focus` , relatedTarget은 `The [EventTarget](https://developer.mozilla.org/en-US/docs/Web/API/EventTarget)receiving focus (if any).`

그러니까 relatedTarget은 위의 상황에서 다른 곳을 ‘클릭’한 경우에 해당되겠다. 

코드를 살펴보면,

```jsx
const handleBlur = (e) => {
	if (!e.currentTarget.contains(e.relatedTarget)) {
		setIsOpenEmoji(false)
	}
}
...

<div className={styles.ChatInput} tabIndex='0' role="button" onBlur={handleBlur}>
			<EmojiPicker 
				addEmoji={addEmoji}
				isOpenEmoji={isOpenEmoji}
				onToggleEmoji={toggleEmoji}
			/>
</div>
```

그러니까, 포커스를 잃을 대상에게 onBlur 이벤트를 주고, handleBlur에서 다른곳을 클릭 했을때 eventTarget이 receiving focus가 존재할 경우에만! 이모지 패널을 닫아준다. 

이모지 패널 안쪽을 클릭한 경우에는 e.relatedTarget이 존재하고, 그 바깥을 클릭한 경우에는 null이 찍혀있는 것을 확인했다.