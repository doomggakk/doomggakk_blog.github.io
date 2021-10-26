---
layout: single
title: "[CKA Concept] 9. Namespace"
categories: [Kubernetes]
author_profile: true
excerpt: CKA 강의 개념 중 쿠버네티스에서의 Namespace에 대해 정리한다. 
toc: true
toc_sticky: true
---

## Namespace
- 쿠버네티스는 동일한 **물리 클러스터**를 기반으로 하는 여러 **가상 클러스터**를 지원한다. 이런 가상 클러스터를 **Namespace**라고 한다.
- 쿠버네티스 클러스터 내의 논리적인 분리 단위
- Namespace는 여러개의 팀이나, 프로젝트에 걸쳐서 많은 사용자가 있는 환경에서 사용하도록 만들어졌다.
- 리소스의 이름은 네임스페이스 내에서 유일해야하지만, 네임스페이스를 통틀어서 유일할 필요는 없다.
- 네임스페이스는 서로 중첩될 수 없으며, 각 쿠버네티스 리소스는 **하나의 Namespace**에만 있을 수 있다.
<br>
> 참고 : 쿠버네티스 시스템 Namespace용으로 예약되어 있으므로, **kube- 접두사**로 네임스페이스를 생성하지 않는다.

<br>

### Namespace 조회
-------------------------

- Namespce 조회
```shell
kubectl get namespace
```
- 원하는 Namespace의 Pod조회
```shell
kubectl get pod --namespace=[namespace 이름]
```

- 쿠버네티스 초기 기본 네임스페이스
1. default : 기본 네임스페이스
2. kube-system : 쿠버네티스 시스템에서 생성한 오브젝트를 위한 네임스페이스 
3. kube-public : 모든 사용자가 읽기 권한으로 접근할 수 있음
4. kube-node-lease : 클러스터가 스케일링될 때 노드 하트비트의 성능을 향상시키는 각 노드와 관련된 리스(lease) 오브젝트에 대한 네임스페이스

<br>

### Namespace 생성
-------------------------

- Namespce 생성

```shell
kubectl create namespace [namespace 이름]
```

```shell
kubectl create -f  namespace.yml
```

```yml
# namespace.yml

apiVersion: v1
kind: Namespace
metadata:
  name: [namespace 이름]
```
<br>

- Pod 생성 후 다른 네임스페이스에 같은 Pod 생성

```shell
kubectl create -f pod-definition.yml --namespace=dev
```

또는

```shell
kubectl create -f pod-definition.yml
```

```yml
# pod-definition.yml

apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
  namespace: dev # namespace 지정
  labels:
  ...
```
<br>-> 다른 namespace에 같은 Pod이 생성됨

<br>


### Pod과 관련된 Namespace 명령어
-------------------------

- Namespace 적용하여 Pod 조회

```shell
kubectl get pod -n [namespace 이름] 
```

- Namespace 적용하여 Pod 생성

```shell
kubectl run [Pod 이름] --image=[image 이름] --namespace=[namespace 이름]

kubectl run [Pod 이름] --image=[image 이름] -n [namespace 이름]
```

- Namespace 상관없이 모든 Pod 조회
```shell
kubectl get pod --all-namespaces
```


<br>

### Namespace 삭제
-------------------------

- Namespce 삭제

```shell
kubectl delete namespace [namespace 이름]
```

<br>

### 기본 Namespace 변경하기 
-------------------------

- 기본 Namespace 변경

```shell
kubectl config set-context --current --namespace=[namespace 이름]

kubectl config view --minify | grep namespace: # 현재 namespace 확인
```

<br>

### Namespace 목적
---------------------

- Namespace별 리소스 할당량 지정
    - Namespace를 통해 CPU/GPU 할당량을 조절할 수 있다.
    - Namespace마다 할당량을 적절히 지정하면서 최적의 개발 및 운영 환경을 구축한다.


![namespace](/assets/img/kubernetes/16_namespace_1.png)


<br>

### DNS
---------------------
- 같은 Namespace에서의 통신은 매우 간단하다.
- 다른 Namespcce에서의 통신은 DNS가 필요하다.



ex) 



1.**default 네임스페이스**의 Pod을 **default 네임스페이스**의 db-service에 연결하는 경우 : 

```shell
mysql.connect("db-service")
```

2.**default 네임스페이스**의 Pod을 **dev 네임스페이스**의 db-service에 연결하는 경우 : 

```shell
mysql.connect("db-service.dev.svc.cluster.local")
```
-> **DNS**를 입력해주어야 한다.

<br>

- DNS의 각 부분에 해당하는 것

```shell
db-service.  dev.    svc.     cluster.local
-----------  ----    -----    -------------
Service      Name    service     domain
  name       space
```

<br>

### Namespace에 속한 리소스 조회
---------------------

- Namespace에 속한 리소스 조회

```shell
kubectl api-resources --namespaced=true
```
- Namespace에 속하지 않는 리소스 조회

```shell
kubectl api-resources --namespaced=false
```

<br>
<br>


------------------
**◎ 참고자료**
- Udemy - Certified Kubernetes Administrator (CKA) with Practice Tests
- [쿠버네티스 공식문서 - 네임스페이스](https://kubernetes.io/ko/docs/concepts/overview/working-with-objects/namespaces/)