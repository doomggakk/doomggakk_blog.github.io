---
layout: single
title: "[CKA Concept] 18. Daemon sets"
categories: [Kubernetes]
author_profile: true
excerpt: CKA 강의 개념 중 쿠버네티스에서의 Daemon sets에 대해 정리한다. 
toc: true
toc_sticky: true
---

## Daemon sets
- Deployment와 유사하게 Pod를 생성하고 관리한다.
- Daemon Sets은 특정 노드 또는 모든 노드에 항상 실행되어야 할 **특정 Pod를 관리**한다.
- 예를들어 모니터링 시스템 구축을 위해 모든 Node에 특정 Pod(로그 수집용)를 관리해야 할 때 사용할 수 있다.
- 특정 Node를 지정하여 사용할 수 도 있다.

![Daemon Set](/assets/img/kubernetes/18_daemon_sets_1.png)


<br>

### Daemon Sets의 Usecase
-------------------
1.Kube-proxy : 새로운 Service를 수시로 검색, Service와 Pod이 통신하는 것을 가능하게 해주는 역할
-> **Node마다 Pod과 Service들이 통신을 할 수 있게 해주는 kube-proxy가 Daemon Sets의 사용예이다.**

<br>
2.Networking

<br>

### Daemon Sets 정의
--------------
- Replica Set의 yaml파일 작성법과 매우 유사하다.

```yaml
apiVersion: apps/v1
kind: DaemonSet     # ReplicaSet과 kind만 다름
metadata:
  name: monitoring-daemon
spec:
  selector:
    matchLabels:
      app: monitoring-agent
  template:
    metadata:
      labels:
        app: monitoring-agent
    spec:
      containers:
      - name: monitoring-agent
        image: monitoring-agent
```

- daemon set 조회

```shell
kubectl get daemonsets
```

<br>

### Test중 헷갈린 개념들
----------------
1. 모든 Namespace의 요소들 조회

```shell
 kubectl get pod --all-namespaces
```

2. namespace는 spec이 아닌 metadata에 선언한다.

3. Daemonsets은 ds로 줄여서 사용가능하다.

``` shell
kubectl get ds
```

<br>
<br>

------------------
**◎ 참고자료**
- Udemy - Certified Kubernetes Administrator (CKA) with Practice Tests

- [쿠버네티스 공식문서 - 데몬셋](https://kubernetes.io/ko/docs/concepts/workloads/controllers/daemonset/)

- [#18 DaemonSets - Life in Hong Kong](https://blog.naver.com/ijoos/222160430404)


