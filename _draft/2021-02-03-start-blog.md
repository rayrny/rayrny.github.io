---
layout: posts
title: "[Blog] 깃 블로그 시작하기"
categories:
  - GitBlog
last_modified_at: 2021-02-03
author_profile: true
tags:
  - jekyll
  - minimal-mistakes
toc: true
toc_sticky: true
sidebar:
  title: Posts
  nav: "sidebar-contents"
---

## 이 게시물에서는 블로그를 만들면서 찾아봤던 정보들과 다양한 에러에 대해 정리할 예정이다.

**minimal-mistakes 테마를 사용


### | 블로그 구조 파악

#### 각각의 파일의 의미
    - 새로 만들어야 하는 폴더
    - layout 폴더의 html 파일이 결국엔 블로그의 css를 설정
      - categories.html > category-archive
      - tags.html > tag-archive
      - posts.html > 단일 포스트의 레이아웃
      - category.html > 기본적으로는 어디서 쓰이는지 모르겟으나 필자는 Algorithm 메뉴에서 알고리즘에 해당하는 글들만을 모아서 보여주기 위해 사용
      - arhive-single > 자주 수정하게 되는 파일 중 하나. 정확하진 않지만 게시물들을 보여줄 때의 레이아웃을 결정하는 것 같음 (recent posts에 최근 게시물 쭉 나와있는데 그 각각의 게시물 제목, 날짜 이런거...)



### | 블로그 커스터마이징

#### 상단 메뉴 (Quick-start) 변경
#### 포스트 게시 날짜로 변경 - 참고한 블로그 링크
#### 사이드바 생성
    - post 폴더의 구조
#### 커스텀 스킨 사용
    - 각각의 변수가 어떤 색을 의미하는지
#### 폰트 변경
#### Algorithm 클릭시 해당 테마만 나오게 - 참고한 블로그 링크
    - 태그 보이게 수정한것도
#### 구글 검색 설정
#### 파비콘 설정


#### 게시물 작성할때 마크다운 문법 및 색상 강조
    - 기본적인 문법 정리
    - 형광펜 및 글씨 색상 정하는 태그
    - ``` ``` 를 이용한 하이라이트 색상 변경 방법 (한글은 폰트 깨짐)
    - 팁은 아니지만 코드블럭 작성할때 대문자로 쓰면 인식 안된다..



### | 마주치는 다양한 오류 상황들

#### 페이지 빌드 실패
    - 일단은 깃허브 페이지에 어느 부분이 오류인지 나와있음
    
#### CSS가 적용 안될 때
    - 보통 문법상의 문제 (;없거나, 오타 등의 사소한 이유)


필요시 하나의 게시물이 아닌 여러개로 나눠서 포스팅 할 계획
