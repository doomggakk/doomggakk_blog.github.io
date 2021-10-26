---
layout: single
title: "[Blog] 5. 블로그에 Toc 추가"
categories: [Blog]
author_profile: true
excerpt: 블로그에 Toc을 추가하는 방법에 대해 정리한다.
toc: true
toc_sticky: true
---


## 블로그에 Toc(Table of Contents) 추가

### Toc?
----------------------------
- Table of Contents를 뜻하며 jekyll 블로그에서 게시글마다 화면 구석에 따라다니면서 목차를 보여주는 유용한 기능이다.
- 원하는 목차로 바로 이동할 수 있어 매우 편리하다.<br>
ex)
![toc 예시](/assets/img/blog/6_blog_toc_1.png)


### 기능 추가방법
----------------------------
- _includes 디렉토리에 toc.html이 있는지 확인
- Minimal Mistakes 템플릿을 사용하고있다면 toc.html이 이미 생성되어있을 것이다.
- 게시글의 머릿말에 2가지를 추가해준다.<br>

```yaml
toc: true        # Toc 사용여부
toc_sticky: true # Toc이 스크롤을 내려도 화면상단에 고정되도록 설정
```