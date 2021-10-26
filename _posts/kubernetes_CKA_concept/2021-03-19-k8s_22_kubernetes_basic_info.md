---
layout: single
title: "[CKA Concept] 22. Kubernetes Basic Info"
categories: [Kubernetes]
author_profile: true
excerpt: Kubernetes와 밀접하게 사용되는 여러가지 툴이나 솔루션에 대해 정리한다.
toc: true
toc_sticky: true
---

## 여러가지 솔루션
**1. 프로메테우스**
- 주요 모니터링 솔루션으로 사용되던 힙스터는 1.13버전 이후로 deprecated 될 예정이며 그 이후의 모니터링 솔루션으로 가장 많이 언급된다.
- 2016년에 오픈소스 프로젝트로 기부되었다. 지표 수집을 통한 모니터링을 주요 기능으로 하고있다.

**2. Helm**
- Kubernetes의 패키지 매니징 툴이다. node.js의 npm과 비슷한 형태로 쿠버네티스 패키지 배포를 가능하게 하는 Tool이다.
- Chart라고 부르는 Package Format을 사용하는데 Chart는 쿠버네티스 리소스를 describe하는 파일들의 집합이다.


<br>
<br>

------------------
**◎ 참고자료**
- Udemy - Certified Kubernetes Administrator (CKA) with Practice Tests
