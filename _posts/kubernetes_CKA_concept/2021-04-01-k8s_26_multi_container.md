---
layout: single
title: "[CKA Concept] 26. Multi Container & Init Container"
categories: [Kubernetes]
author_profile: true
excerpt: CKA 강의의 개념 중 Multi Container와 Init Container에 대해 정리한다. 
toc: true
toc_sticky: true
---

## Multi Container
- 쿠버네티스에서 Pod은 멀티 컨테이너를 지원한다.
- Pod 안의 멀티 컨테이너는 같은 LifeCycle, Network, Storage를 사용한다.
- 예를 들면 하나의 Webserver와 Log agent를 같이 두는 경우가 있다.

<br>

### Multi Container YAML

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: simple-webapp
spec:
  containers:
  - name: simple-webapp        #  <─┐
    image: simple-webapp       #    │ 
    ports:                     #    ├─ Array로 처리한다.
      - containerPort: 8080    #    │
  - name: log-agent            #  <─┘ 
    image: log-agent           # 멀티컨테이너를 지원한다는 것을 의미한다.

```

### 실습을 통해 알게된 것
- 컨테이너의 Log를 확인하는 법 :

```shell
kubectl logs <pod 이름> <container 이름>
(컨테이너가 하나인 경우 Pod이름만써도 가능)
```

<br>

## Init Container (초기화 컨테이너)
- Multi container에서 각각의 Pod은 Pod의 lifecycle동안 돌아가도록 설계되어있다. 
- 하지만 Init container는 Pod이 만들어질때 한번만 돌아가야하는 경우에 사용된다.
- Pod의 앱 컨테이너들이 실행되기 이전에 실행되는 특수한 컨테이너이다.
- 각 초기화 컨테이너는 **다음 초기화 컨테이너가 시작되기 전에 성공적으로 완료**되어야 한다.

<br>

### 일반적인 컨테이너와 초기화 컨테이너의 차이점
- 초기화 컨테이너는 lifecycle, livenessProbe, readinessProbe 또는 startupProbe 를 지원하지 않는다. 왜냐하면 초기화 컨테이너는 파드가 준비 상태가 되기 전에 완료를 목표로 실행되어야 하기 때문이다.
- 만약 다수의 초기화 컨테이너가 파드에 지정되어 있다면, kubelet은 해당 초기화 컨테이너들을 한 번에 하나씩 실행한다.
- 각 초기화 컨테이너는 다음 컨테이너를 실행하기 전에 꼭 성공해야 한다. 모든 초기화 컨테이너들이 실행 완료되었을 때, kubelet은 파드의 애플리케이션 컨테이너들을 초기화하고 평소와 같이 실행한다.

<br>

### Init Container 작성방법
- init container가 여러개 있다면 순서에 맞게 실행된다

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
  labels:
    app: myapp
spec:
  containers:
  - name: myapp-container
    image: busybox:1.28
    command: ['sh', '-c', 'echo The app is running! && sleep 3600']
  initContainers:
  - name: init-myservice
    image: busybox:1.28
    command: ['sh', '-c', 'until nslookup myservice; do echo waiting for myservice; sleep 2; done;']
  - name: init-mydb
    image: busybox:1.28
    command: ['sh', '-c', 'until nslookup mydb; do echo waiting for mydb; sleep 2; done;']

```

<br><br>

------------------
**◎ 참고자료**

- Udemy - Certified Kubernetes Administrator (CKA) with Practice Tests
- [쿠버네티스 공식문서 - 초기화 컨테이너](https://kubernetes.io/ko/docs/concepts/workloads/pods/init-containers/)
- [#24 Multi & Init Containers - 작성자 ijoos](https://blog.naver.com/ijoos/222166691131)