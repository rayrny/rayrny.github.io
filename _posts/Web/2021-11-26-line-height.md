---
layout: posts
title: "[WEB] line-height: 18px을 쓰면 안 되는 이유"
categories:
  - Web
last_modified_at: 2021-11-26
author_profile: true
tags:
  - 웹
  - MDN
  - Javascript
  - CSS
toc: true
toc_sticky: true
sidebar:
  title: Posts
  nav: "sidebar-contents"
---

### line-height 속성은 뭔데?

MDN 문서에 의하면 line-height 속성은 Line box의 높이를 지정한다. 한 마디로 텍스트 줄 간격을 설정하는 것이다. 또한 뒤이어 이런 설명이 나타나는데, 번역본을 보아도 쉽게 이해가 되지 않는다.

> *On block-level elements, it specifies the minimum height of line boxes within the element. On `non-replaced inline elements`, it specifies the height that is used to calculate line box height.*
> 

Block-level 요소의 경우 인라인 박스의 최소 높이를 지정한다고 한다. 앞의 설명과 일치하니, 다음 문장을 보자. non-replaced inline 요소는 라인 박스의 높이를 계산하기 위해 사용되는 높이를 지정한다고 한다. 도대체 `non-replaced inline elements` 는 무엇일까?

### 교체되지 않은 인라인 요소

lnline 요소라는 것은 알고 있는데 앞에 붙은 non-replaced가 무슨 뜻인지 한참 찾아보았다. 우리의 친구 [stackoverflow](https://stackoverflow.com/questions/12468176/what-is-a-non-replaced-inline-element)의 한 질문에서 답을 찾을 수 있었다. 

![스크린샷 2021-11-22 오전 10 36 11](https://user-images.githubusercontent.com/48341341/143008237-4a5883a8-a6b4-4c90-abfa-52f7f2b1924e.png)


우선 replaced element(대체 요소)의 한 예로는 `<img>` 태그가 있다. <img> 태그와 여타 다른 inline 속성을 가지는 태그들, 이를테면 `<a>, <span>` 등을 생각하며 답변을 마저 살펴보자.

> *So “non-replaced element” is just an element that is rendered as such, instead of causing some external content to appear in its place.*
> 

간단 명료한 설명이다. '외부 컨텐츠가 들어가는 것이 아닌' 경우가 바로 `non-replaced`한 것이다. MDN에 대체 요소(replaced element)에 대해 검색해 보니 이곳에서도 설명이 꽤 잘 나와 있었다. 

<aside>
💡 **대체 요소**(replaced element)란 자신의 표현 결과가 CSS의 범위를 벗어나는 요소로서, CSS 서식 모델과는 분리된 외부 객체인 요소입니다.

간단히 말해서, 대체 요소는 자신의 콘텐츠가 현재 문서 스타일의 영향을 받지 않는 요소라고 할 수 있습니다. CSS는 대체 요소의 위치에 영향을 줄 수 있지만 콘텐츠에는 미치지 못합니다. `<iframe>`등 일부 대체 요소는 자신만의 스타일 시트를 가질 수도 있지만, 부모 문서의 스타일을 상속하지는 않습니다.

</aside>

**replaced element**

- ```<iframe>```
- ```<video>```
- ```<embed>```
- ```<img>```

위의 예시들을 보면 이제 `replaced element`에 대해 감을 잡았을 것이고, 우리가 궁금해 했던 `non*-*replaced element` 란 위의 태그들을 제외한 일반적인 inline 속성을 가진 태그들 임을 알아냈다. 이제 다시 처음 질문으로 돌아가보자.

### 그래서 line-height는 숫자로 지정해야 한다고?

MDN에 따르면 line-height는 `unitless numbers` 를 사용하는 것을 권장하고 있다. 흔히 사용하는 `px, rem, em` 등의 단위로 지정하는 것을 지양하라는 것인데, 이유가 무엇일까?

바로 상속(inheritance)때문이다. 

아래의 예시를 보면, 초록 박스는 ```unitless number```, 분홍 박스는 ```em```, 파랑 박스는 ```%```를 통해 ```line-height```를 지정해 주었다. 

![스크린샷 2021-11-22 오전 11 06 05](https://user-images.githubusercontent.com/48341341/143008273-45724ac8-245b-475b-a6b1-30517e8fe0c2.png)


```html
<div class="box green">
	<h1>오늘은 line-height 속성에 대해서 알아볼 것이다.</h1>
	line-height는 unitless number로 지정하는 것이 좋다. 상속으로 인해 의도하지 않은 결과가 나타날 수 있기 때문이다.
</div>

<div class="box red">
	<h1>오늘은 line-height 속성에 대해서 알아볼 것이다.</h1>
	line-height는 unitless number로 지정하는 것이 좋다. 상속으로 인해 의도하지 않은 결과가 나타날 수 있기 때문이다.
</div>

<div class="box blue">
	<h1>오늘은 line-height 속성에 대해서 알아볼 것이다.</h1>
	line-height는 unitless number로 지정하는 것이 좋다. 상속으로 인해 의도하지 않은 결과가 나타날 수 있기 때문이다.
</div>
```

```css
.green {
  line-height: 1.1;
  border: solid limegreen;
}

.red {
  line-height: 1.1em;
  border: solid pink;
}

.blue {
  line-height: 110%;
  border: solid skyblue;
}

h1 {
  font-size: 30px;
}

.box {
  width: 18em;
  display: inline-block;
  vertical-align: top;
  font-size: 15px;
}
```

	
부모인 ```<div>```태그의 ```font-size```가 ```15px```로 지정되어 있기 때문에 ```15px * 1.1 = 16.5px```로 line-height가 ```<h1>``` 태그에도 동일하게 지정된 것을 볼 수 있다. 때문에 글씨가 깨져버린다. 아래의 설명으로 달린 부분은 폰트가 15px이기 때문에 의도한 대로 설정되었지만 ```<h>``` 태그의 경우 font-size가 30px이지만 부모의 font-size인 15px을 기준으로 line-height가 계산되어 문제가 발생했다.

따라서 이러한 문제를 피하기 위해서는 단위가 없는 숫자를 통해 line-height를 지정해 주도록 하자!
