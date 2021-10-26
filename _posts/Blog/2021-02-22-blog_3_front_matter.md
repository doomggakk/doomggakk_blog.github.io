---
layout: single
title: "[Blog] 2. 머릿말"
categories: [Blog]
author_profile: true
excerpt: 게시물 작성시 필요한 머릿말에 대하여 정리한다.
date: 2021-02-22T16:52:00
toc: true
toc_sticky: true
---

## Front Matter(머릿말)

```yaml
---
layout: post // post, page 중 하나를 넣어 이것이 한 페이지에 속하는 블로그 포스트인지, 홈페이지를 구성하는 하나의 큰 페이지인지를 결정한다.
title: "제목" // 포스트나 페이지의 제목을 넣는다.
date: 2015-02-08T20:39:51-09:00 // 생성일로, 양식은 년-월-일T시:분:초-GMT시간 으로 추측된다.
modified: 2015-02-08T22:21:31-09:00 // 변경일
categories: articles // 어느 페이지에 속하는지를 표시한다.
excerpt: "요약문" // 이 페이지에 대한 요약문을 정하는 것 같은데, 정확히 어떤 동작을 보이는지 아직 모르겠다.
tags: [haroopad] // 태그를 지정할 수 있다. 해시태그인지 어떻게 동작하는지는 역시 아직 모른다.
image: // so-simple-theme 에서 글 맨 위에 노출될 큰 사진을 결정할 때 쓴다. 기본 폴더는 images.
  feature: test.jpg // 사진 파일 이름
  credit: Me // 사진 제작자 및 저작권자
  creditlink: me.org // 저작권자의 홈페이지이거나 이 사진을 직접 구할 수 있는 링크
share: true // so-simple-theme 의 사이드바에 공유 버튼이 추가된다.
comments: true // 블로그 포스트 하나하나마다 개별적으로 disqus 댓글 버튼을 추가해줄 수 있다.
---
```

------------------
**◎ 참고자료**


- [Hangul test : 지킬(Jekyll)로 깃헙 블로그 만들 때- Uno's Blog](https://djkeh.github.io/articles/Hangul-test-jekyll-tips-kor/)