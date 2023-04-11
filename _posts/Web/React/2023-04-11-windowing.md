---
layout: posts
title: "[React] windowing 기법을 통한 성능 최적화 feat.react-virtualized"
categories:
  - React
last_modified_at: 2023-04-11
author_profile: true
tags:
  - 성능 최적화
  - windowing
  - react
toc: true
toc_sticky: true
sidebar:
  title: Posts
  nav: "sidebar-contents"
---


DOM에 그릴 엘리먼트가 매우매우 많을 때 왜 브라우저의 성능은 안좋아질까?


### 문제 상황

검색어를 입력할 수 있는 input과 검색 결과가 dropdown 내부에 보여지는 컴포넌트에서,
검색 결과가 많을 경우(1000명 이상) 브라우저가 눈에 띄게 느려지며 사용자의 인터렉션에 의한 이벤트 처리가 매우 느려지는 현상이 발생했다.


Input에 포커스가 잡히는 속도, 검색어 입력 후 반영되는 속도, 입력이 완료된 이후 검색 결과를 반영하는 속도(이건 너무 느려서 아예 결과도 보여지지 않았음)가 매우 느렸는데, 처음 해당 문제를 마주했을 떄는 DOM에 붙은 엘리먼트 1000개는 컴퓨터 입장에서 그렇게 많은 것도 아닐텐데 도대체 왜 이벤트 처리(인터렉션)이 이렇게까지 느려졌지? 라는 생각이 들었다.


## 측정할 수 없으면...
우선 어떤 상황인지 정확하게 확인하기 위해 크롬 개발자도구를 열었다. 

```
측정 도구: chorme dev tools - lighthouse, performance
데이터 개수(dropdown item): 약 1,300개
시나리오: 
첫 렌더링 이후 input 클릭 > 드롭다운이 열리면 검색어 입력 > 검색 결과 기다린 후 종료
```

크롬 개발자 도구 - Lighthouse를 통해 측정한 보고서
[lighthouse-before.html.zip](https://github.com/goorm-dev/edu-core/files/11201696/lighthouse-before.html.zip)

뭔진 잘 몰라도 일단 화면이 느리구나~ 9초 정도 지연이 있었겠구나~ 를 알 수 있다.

![veryveryslow](https://user-images.githubusercontent.com/89910087/231197982-afa38fc5-83d6-47a9-b137-5c5f7d4e953a.png)

### 🧀 tl;dr 

검색 결과를 보여주는 드롭 다운에 1000개 이상의 목록을 그리게 되며 DOM에 붙은 엘리먼트들이 매우 많아지고, 이에 따라 브라우저가 실행해야 하는 스크립트의 양이 어마무시하게 증가함.

물론 그냥 텍스트만 들어있는 div 태그라면 1,000개든 2,000개든 그렇게 무거운 작업이 아니지만,

- 드롭다운으로 열리는 메뉴 내부의 아이템이므로 다른 경우에 비해 연산할 것이 더 많음
- Dropdown Item에서 Text만 존재하는 것이 아니라 컴포넌트를 그리고 있음
    - `<div>{item.name}<div>`만 그릴 경우 인터랙션 지연이 크게 없음
        ![justText](https://user-images.githubusercontent.com/89910087/231198606-91944b5e-87c7-4057-b547-05cb3aecdf27.png)

        여기서 CSS를 추가하거나, 조금만 더 무겁게 구성해도 바로 지연 시간이 늘어나는 것도 확인했다.
        

**각각의 성능 측정 지표를 조금 더 살펴보자!**

### [**Total Blocking Time(TBT)**](https://developer.chrome.com/ko/docs/lighthouse/performance/lighthouse-total-blocking-time/)

> *TBT는 마우스 클릭, 화면 탭 또는 키보드 누름과 같은 사용자 입력으로부터 페이지가 응답하지 못하도록 차단된 총 시간을 측정합니다.*
> 

<img width="768" alt="tbt" src="https://user-images.githubusercontent.com/89910087/231199048-441c01cb-8c98-4734-9ce1-bab22117da82.png">

scripting 작업으로 분류되는 Script Evaluation 작업으로 인해 Input의 인터렉션이 지연되고 있는 상황

### **[Interaction to Next Paint (INP)](https://web.dev/inp/?utm_source=lighthouse&utm_medium=devtools)**

> *INP는 사용자가 페이지에서 수행한 모든 상호 작용의 대기 시간을 관찰하고 모든(또는 거의 모든) 상호 작용이 아래에 있는 단일 값을 보고합니다.*
> 
- 좋은 INP는?
    
    ![inpStandard](https://user-images.githubusercontent.com/89910087/231199303-fa9cb995-b83a-4d8c-9368-fab4114a258e.png)
    

<img width="767" alt="poorInp" src="https://user-images.githubusercontent.com/89910087/231199291-f0e38796-b9b6-42ed-8fe6-a556afde004f.png">

위 글에 따르면 500ms 만 넘어도 POOR Responsiveness라는데 여기는 무려 6,400ms..

보고서에 따르면 가장 많은 병목이 일어나고 있는 부분이 Scripting 작업, 정확하게는 `Script Evaluation` 작업이다. 이로 인해 Input의 이벤트가 지연되고 있다고 볼 수 있겠다. 

![poor-perfomance-crop](https://user-images.githubusercontent.com/89910087/231199576-ced19cb1-23b4-48c1-9ba0-43bf5e6ba710.png)


그렇다면 `Script Evaluation` 작업을 줄인다면 인터렉션 지연도 자연스럽게 해결 될 것 같다!

- [Script evaluation이 뭘 하는 작업인데?](https://stackoverflow.com/questions/58882699/lighthouse-audit-script-evaluation-vs-script-parsing)
    
    GPT에게 물어봤습니다.
    
    스크립트 크기가 매우 클 경우 `script evaluation` 작업에 매우 많은 시간이 소요되므로 페이지가 느려질 수 있다고 합니다.
    
   <img width="815" alt="gpt-script-evaluation" src="https://user-images.githubusercontent.com/89910087/231199882-41df8187-0ddd-4e9e-aed3-e859fabcb2d3.png">
    
- 그래서 최적화가 가능해?
    - 네, [여기에서](https://web.dev/fast/#optimize-your-third-party-resources) 매우 많고 다양한 방법을 소개하고 있습니다!
    - 추가로, 이친구도 답변 아주 잘 해줍니다.
    제일 처음으로 소개하는 방법이 `Limiting the number of DOM elements` 네요.
        
    <img width="718" alt="gpt-improve-method" src="https://user-images.githubusercontent.com/89910087/231199853-10812fb2-c2c7-42ff-ae4c-5a75978f0eae.png">
        

### 목표는
강의자 검색 클릭 시 드롭다운이 지체 없이 그려지고 이후 검색어를 입력할 때도 사용자 입장에서 별도의 지연을 느끼지 않도록 INP시간을 개선하는 것이다.

- 1차 목표: 500ms 이하의 interection latency
- 2차 목표: 200ms 이하의 interection latency


## 본격적으로 문제를 해결해보자

위 문제를 해결하고 프로젝트 목표를 달성하기 위해 필요한 것은 무엇인가요?

---

위에서 살펴봤던 TBT를 개선하는 방법은 무엇이 있을까? 공식 문서에 따르면 아래와 같은 방안을 제시한다.

![improveTBT](https://user-images.githubusercontent.com/89910087/231200811-40193b4c-c5de-4d35-a5a9-0ba520b39d3c.png)

불필요한 JS 로딩 및 실행?? >>> windowing!

[공식 문서 - 성능 최적화 파트](https://ko.reactjs.org/docs/optimizing-performance.html#virtualize-long-lists)에서도 `Windowing` 기법(`List Virtualization`이라고도 함)을 소개하고 있다.

### [windowing 이란?](https://web.dev/i18n/ko/virtualize-long-lists-react-window/)

> concept of only rendering or write the visible portion in the current **“window”**
> to the DOM. The number of items that rendered at first time are smaller than the original one.


그려야 할 엘리먼트가 아무리 많아도 실제로 DOM에 붙는 요소는 한정되어 있기 때문에 목록 개수가 더 많아지더라도 성능에 차이가 없다.

기존 프로젝트 내에 [react-virtualized](https://web.dev/i18n/ko/virtualize-long-lists-react-window/)가 이미 설치 되어 있어 해당 라이브러리를 사용했다. (react-window가 더 작고 가볍지만 지원하는 기능이 적음. react-virtualized가 사용량이 더 많음)


### 생각해봤던 다른 해결책

- API 응답 응답 페이지네이션
    - 페이지네이션도 하나의 방법이겠지만, 별도 장치 없이 페이지네이션만 적용할 경우 목록 가장 하단까지 정보를 불러온 경우 똑같은 문제가 발생하게 됨. 따라서 다른 방안을 찾게 되었고, windowing 적용으로 최종 결정


## 결과

windowing 적용한 페이지에 대한 보고서

[lighthouse-after.html.zip](https://github.com/goorm-dev/edu-core/files/11201810/lighthouse-after.html.zip)

![after](https://user-images.githubusercontent.com/89910087/231201393-1d1163a9-511f-43e7-857e-1ccbe09da2d9.png)

목표했던 INP 200ms 이하로 개선되었고, 실제로도 여러 이벤트들에 대한 지연이 전혀 느껴지지 않았다!


**`Before`**

![performance-before](https://user-images.githubusercontent.com/89910087/231201834-fe3e4c88-4629-4bad-9e59-ebdbd84a6563.png)

**`After`**

![performace-after](https://user-images.githubusercontent.com/89910087/231201825-5b2541b7-70cb-4544-8d25-03646875a0cc.png)

performance 탭에서도 차이를 쉽게 확인할 수 있다.

- 눈에 띄는 긴 task들이 모두 사라짐
- 각 task를 처리하는 속도가 매우 빨라짐
- 상단 타임라인(box 영역)을 가득 채우던 노란색 영역들(아래 파이차트에서 보이는 scripting 작업)이 매우 줄어듬

→ 수치상으로 약 90% 이상 빨라짐