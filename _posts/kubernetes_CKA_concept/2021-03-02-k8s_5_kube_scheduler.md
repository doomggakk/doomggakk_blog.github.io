---
layout: single
title: "[CKA Concept] 5.Kube Scheduler"
categories: [Kubernetes]
author_profile: true
excerpt: CKA 강의의 개념 중 쿠버네티스에서의 Kube Scheduler에 대해 정리한다. 
toc: true
toc_sticky: true
---

## Kube Scheduler
- Node에 있는 Pod을 스케줄링하는 역할
- **※ 주의 : kube-scheduler는 Pod을 생성하는 역할이 아닌 어떤 Pod이 어떤 node에서 생성이 되어야할지를 결정해주는 역할을 한다.** <br>
-> Pod을 생성하는 역할은 **kubelet**이 한다.

<br>

- kube-Scheduler은 생성할 컨테이너가 최적의 노드에서 생성되도록 스케줄링 한다.
- 아래와 같은 순서의 예시로 스케줄링한다.
1. 노드 필터링 : 할당해야할 컨테이너의 자원을 충분히 받아들일 수 있는 노드인지를 체크한다.
2. 노드의 우선순위를 매김 : 생성할 컨테이너 자원을 할당한 뒤의 남은 자원이 가장많은 노드가 우선순위가 되도록 정한다.
3. 위의 단계를 거쳐 가장 최적화된 노드에 Pod을 생성 하도록한다.

<br>

### Kube-Scheduler 설치
------------

```shell
wget https://storage.googleapis.com/kubernetes-release/release/v1.13.0/bin/linux/amd64/kube-scheduler
```

### Kube-Scheduler 옵션 확인
------------

```shell
cat /etc/kubernetes/manifests/kube-scheduler.yaml
```

<br>

------------------
**◎ 참고자료**
- Udemy - Certified Kubernetes Administrator (CKA) with Practice Tests
- [쿠버네티스 공식문서 - 쿠버네티스 컴포넌트](https://kubernetes.io/ko/docs/concepts/overview/components/)