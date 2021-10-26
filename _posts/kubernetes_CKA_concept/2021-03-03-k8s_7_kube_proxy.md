---
layout: single
title: "[CKA Concept] 7. Kube Proxy"
categories: [Kubernetes]
author_profile: true
excerpt: CKA 강의의 개념 중 쿠버네티스에서의 Kube Proxy에 대해 정리한다. 
toc: true
toc_sticky: true
---

## Kube Proxy
- 쿠버네티스에서 모든 Pod은 다른 Pod과 통신할 수 있다.<br>
-> 클러스터에 Pod 네트워킹 솔루션(Pod 내부 가상의 네트워크)이 갖춰져 있기 때문
- 하지만 웹 애플리케이션을 예시로 들었을 때 다음과 같은 문제가 생김<br>
-> 웹 애플리케이션 Pod에서 DB Pod에 연결을 하려하는데 Pod이 변경되어 IP가 일정하지 않은 문제가 발생.<br>
-> 이러한 문제를 해결하기 위해 Service를 생성하여 DB에 연결한다.<br>
-> Service와는 컨테이너 처럼 실제 존재하는 것이 아니기 때문에 interface 같은 것이 없거나, listening 프로세스가 없을 수 있다.<br>
-> 이러한 Service와 Pod 사이의 작업을 위하여 Node마다 **kube-proxy**가 있다.

<br>

### Kube Proxy 역할
------------------
1. 새로운 Service를 수시로 검색
2. Service가 생성 됐을 경우 각 Node가 Service에 도달하기 까지의 적절한 규칙을 만든다.<br>
-> Node마다 iptable을 생성하여 서비스와 실제 Pod의 IP를 기록해둔다.
<br>

### Kube Proxy 설치
------------------

```shell
wget https://storage.googleapis.com/kubernetes-release/release/v1.13.0/bin/linux/amd64/kube-proxy
```


<br>

### Kube Proxy 확인 - kubeadm
------------------

```shell
kubectl get pods -n kube-system
```
-> kube-proxy가 pod으로 되어있는 모습을 확인할 수 있음
![kube-proxy](/assets/img/kubernetes/15_kube_proxy_1.png)

<br>

------------------
**◎ 참고자료**
- Udemy - Certified Kubernetes Administrator (CKA) with Practice Tests
- [쿠버네티스 공식문서 - 쿠버네티스 컴포넌트](https://kubernetes.io/ko/docs/concepts/overview/components/)