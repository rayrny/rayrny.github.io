---
layout: posts
title: "순환 참조, Madge 그리고 eslint"
categories:
  - Javascript
last_modified_at: 2023-03-12
author_profile: true
tags:
  - 웹
  - eslint
  - 트러블 슈팅
toc: true
toc_sticky: true
sidebar:
  title: Posts
  nav: "sidebar-contents"
---

### webpack 버전을 올리다가,

빌드 구조 개선 업무를 진행하며 `webpack4 → webpack5`로 버전 업데이트를 진행하던 중, 이전에는 발생하지 않던 부분에서 순환 참조 오류로 인해 페이지 접속 불가능한 상황이 발생했다.

webpack4에서는 왜 문제가 되지 않았는지는 찾아보는 중...

[Webpack/Issue #13779](https://github.com/webpack/webpack/issues/13779), [Webpack/Issue #13411](https://github.com/webpack/webpack/issues/13411) 외에도 나와 비슷하게 webpack 버전만 4 → 5 로 변경했는데 이러한 에러가 발생했다는 이슈가 있었다. 
그런데 명확한 이유는 없이 죄다 ‘너 그거 순환 참조 에러야!’ 만 적혀있다..

[Webpack/Issue #11958](https://github.com/webpack/webpack/issues/11958)
![처음엔 당연히 undefined겠지만..](https://user-images.githubusercontent.com/48341341/224540245-f594b6b7-9244-49e7-a8cf-435a7b678ce8.png)

![여기서 step 이 undefined인게 문제인데, webpack 4에서는 왜 이걸 에러로 잡지 못했을까?](https://user-images.githubusercontent.com/48341341/224540284-5a49d2ce-fdd3-481a-b405-379357a32d10.png)



### 순환 참조를 확인할 수 있는 유용한 라이브러리

순환 참조를 쉽게 확인할 수 있는 라이브러리를 찾아보다가 [Madge](https://www.npmjs.com/package/madge)를 발견하게 되었고, 별도로 패키지(Graphviz)를 설치해야하긴 하지만 순환 참조에 대한 그래프를 그릴 수 있다는 점이 흥미로워서 바로 설치해봤다.

해당 라이브러리를 통해 에러가 발생한 페이지 폴더 하위를 검사해본 결과
<img width="1405" alt="circular-dependency-before" src="https://user-images.githubusercontent.com/48341341/224540346-48307be7-7273-4bb0-94d0-d2598ade83fa.png">


빨간색으로 표시되는 파일들 위주로 확인해봄으로써, 문제가 되는 부분을 쉽게 찾을 수 있었다. 위 그래프를 조금만 더 단순화 시켜서 순환 참조 고리를 확인해보자!

문제는 SqlQuizScoringSetTree 파일에서 실제로는 scoreingSet만을 사용중인데 constants/sql/index를 참조하게 되며 step을 참조, step 내부에서 SqlQuizExamSetting을 참조하며 순환 참조가 발생하게 된다.

<img width="462" alt="스크린샷 2023-02-23 오후 12 11 59" src="https://user-images.githubusercontent.com/48341341/224540396-b1792d5f-8c0b-4007-ac54-666ece0c00c6.png">


```jsx
// SqlQuizScoringSetTree.jsx

// import SQL from '../../constants/sql';
import sortingSet from '../../constants/sql/scoringSet';
```

따라서 위와 같이 수정한 뒤에 다시 그래프를 추출해보니 순환 참조가 제거된 것을 확인할 수 있었다.
<img width="1292" alt="circular-dependency-after" src="https://user-images.githubusercontent.com/48341341/224540353-57906472-260e-4d03-8dfc-aa0cb047ca21.png">


### import/no-cycle ESLint Rule

위와 같은 과정을 거치며 순환 참조 방지를 위한 eslint 규칙인 `import/no-cycle`을 추가할 것을 조직에 제안하게 되었다.

[**import/no-cycle**](https://github.com/import-js/eslint-plugin-import/blob/v2.27.5/docs/rules/no-cycle.md)

