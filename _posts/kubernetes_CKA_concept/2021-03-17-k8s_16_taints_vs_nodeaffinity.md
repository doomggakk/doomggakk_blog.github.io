---
layout: single
title: "[CKA Concept] 16. Taints&Tolerations vs Node Affinity"
categories: [Kubernetes]
author_profile: true
excerpt: CKA 강의 개념 중 쿠버네티스에서의 Taints&Tolerations vs Node Affinity에 대해 정리한다. 
toc: true
toc_sticky: true
---

## Taints & Tolerations vs Node Affinity

- Taints & Tolerations과 Node Affinity를 각각 사용할경우에는 원하는Pod가 원하는 Node에 생성되도록하는데 한계가 있음

<br>

### Taints & Tolerations
----------------------
- Node의 문에 Taint라는 자물쇠를 걸어 Toleration이라는 키를 가진 Pod만 생성되도록 한다.
- **Node가 원하는 Pod만 받을 수 있음**
- **하지만 Taints가 정해지지 않은 Node가 있을 경우 랜덤으로 그 Node에 배정될 수 있음**

![taints_vs_affinity의 한계](/assets/img/kubernetes/18_taints_vs_affinity_1.png)

<br>

### Node Affinity
----------------------
- Node의 문(label)은 각자 다르지만 누구에게나 열려있다.
- Pod은 자기가 선호하는 문을 가진 Node에 할당되도록 조건을 걸 수 있다. = Node Affinity
- **Pod이 선호하는 Node에서 생성될 수 있음**
- **하지만 Node Affinity가 정해지지않은 Pod들이 랜덤으로 어느 Node든지 생성 될 수 있음**

![taints_vs_affinity의 한계](/assets/img/kubernetes/18_taints_vs_affinity_2.png)

<br>

### Taints & Tolerations + Node Affinity
----------------------
- 위의 한계들을 해결하기 위해 **Taints & Toleration 과 Node Affinity를 같이 사용**한다.
- 원하는 Pod만 원하는 Node에 생성 될수 있도록 구현할 수 있다.


![taints_vs_affinity의 한계](/assets/img/kubernetes/18_taints_vs_affinity_3.png)


<br>
<br>

------------------
**◎ 참고자료**
- Udemy - Certified Kubernetes Administrator (CKA) with Practice Tests

- [쿠버네티스 공식문서 - 노드에 파드 할당하기](https://kubernetes.io/ko/docs/concepts/scheduling-eviction/assign-pod-node/)


