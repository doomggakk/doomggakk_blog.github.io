---
layout: single
title: "[CKA Concept] 4. Kube Controller Manager"
categories: [Kubernetes]
author_profile: true
excerpt: CKA 강의의 개념 중 쿠버네티스에서의 Kube Controller Manager에 대해 정리한다. 
toc: true
toc_sticky: true
---

## Kube Controller Manager
- 컨트롤러를 구동하는 마스터 상의 컴포넌트.
- 논리적으로는 각 컨트롤러들은 개별 프로세스이다. 하지만 복잡성을 낮추기 위해서 하나의 단일 바이너리로 컴파일 되고 단일 프로세스에서 실행된다.
- 아래의 컨트롤러들을 포함한다.
1. 노드 컨트롤러 : 노드가 다운되었을 때 통지와 대응에 관한 책임을 가진다.
2. 레플리케이션 컨트롤러 : 시스템의 모든 레플리케이션 컨트롤러 오브젝트에 대해 알맞은 수의 파드들을 유지시켜주는 책임을 가진다. 
3. 엔드포인트 컨트롤러 : 엔드포인트 오브젝트를 채운다( 즉 서비스와 파드를 연결시킨다. )
4. 서비스 어카운트 & 토큰 컨트롤러: 새로운 네임스페이스에 대한 기본계정과 API 접근 토큰을 생성한다.

<br>

### 컨트롤러의 역할
------------------
1. 시스템의 여러 컴포넌트들의 상태를 지속적으로 모니터링한다.
2. 자신이 맡은 컴포넌트에 무엇인가 문제가 생기면 조치를 취한다.

<br>

### More about 컨트롤러
------------------
-  컨트롤러 안에는 노드 컨트롤러, 레플리케이션 컨트롤러 이외에도 여러가지 컨트롤러 들이 존재한다. 이 각각의 컨트롤러들은 **kube controller manager라는 단일 프로세스 안에 패키징** 되어있다.
- kube controller manager를 설치하면 그안에 여러가지 컨트롤러들이 설치되어 내장돼있다.

### kube-controller-maneger 설치
------------------

```shell
wget https://storage.googleapis.com/kubernetes-release/release/v1.13.0/bin/linux/amd64/kube-controller-manager
```



<br>

------------------
**◎ 참고자료**
- Udemy - Certified Kubernetes Administrator (CKA) with Practice Tests
- [쿠버네티스 공식문서 - 쿠버네티스 컴포넌트](https://kubernetes.io/ko/docs/concepts/overview/components/)