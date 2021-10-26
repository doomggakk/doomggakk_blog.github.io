---
layout: single
title: "[CKA Concept] 21. Monitor Cluster Components & Application Logs"
categories: [Kubernetes]
author_profile: true
excerpt: CKA 강의 개념 중 쿠버네티스에서의 Monitor Cluster Components와 Application Logs에 대해 정리한다. 
toc: true
toc_sticky: true
---

## Monitor Cluster Components
- Kubernetes에서 모니터링을 해야하는 영역은 다양하며, Application, Pod, Node Level등 서로 다른 영역이 모니터링 된다.
- Kubernetes 자체적으로 전체적인 모니터링기능을 하는 솔루션은 존재하지않아서 Metrics Server, Prometheus, Elastic Stack, DataDog, Dynatrace 등 오픈소스 솔루션을 이용한다.
- 힙스터가 주로 사용되던 모니터링 솔루션이었지만 Deprecated 되어 사용되지 않는다.
- 각 Node는 Metric(지표)를 수집하는데 cAdvisor라는 Metric(지표) 수집도구를 사용한다.
- cAdvisor가 각 Node의 Kubelet에 설치돼서 Node나 Pod의 Metric(지표)를 수집한다.
- 여기서 수집된 Metric(지표)들은 Metrics-Server나 프로메테우스 같은 모니터링 툴에서 다시 수집해 간다.

<br>

### Metrics Server 설치
--------------
- Minikube 사용할 경우

```shell
minikube addons enable metrics-server
```

- 이외의 경우

```shell
git clone https://github.com/kubernetes-incubator/metrics-server.git

kubectl create -f deploy/1.8+/
```

<br>

### Metrics Server View
--------------
- Node Level 정보 확인

```shell
kubectl top node
```
![monitoring](/assets/img/kubernetes/21_monitoring_1.png)

<br>

- Pod Level 정보 확인

```shell
kubectl top pod
```
![monitoring](/assets/img/kubernetes/21_monitoring_2.png)

<br>

## Application Logs
- docker에서는 아래 명령어로 log를 확인할 수 있다.

```shell
docker logs -f ecf
```

- kubernetes에서는 Pod의로그를 확인할 수 있다.

```shell
kubectl logs -f [pod 이름]
```

### Log 확인
---------------------
- 하나의 Container를 가진 Pod log 확인

```shell
kubectl logs [Pod 이름]
또는
kubectl logs -f [Pod 이름] # Log가 실시간으로 찍힘

ex) kubectl logs webapp-1
```

- 두 개 이상의 Container를 가진 Pod log 확인

```shell
kubectl logs [Pod 이름] -c [container 이름]
또는
kubectl logs -f [Pod 이름] [container 이름] # Log가 실시간으로 찍힘

ex) kubectl logs webapp-2 -c simple-webapp
```

<br>
<br>

------------------
**◎ 참고자료**
- Udemy - Certified Kubernetes Administrator (CKA) with Practice Tests

- [쿠버네티스 공식문서 - 리소스 모니터링 도구](https://kubernetes.io/ko/docs/tasks/debug-application-cluster/resource-usage-monitoring/)


- [#21 Monitoring & Managing Application logs - Life in Hong Kong](https://blog.naver.com/ijoos/222163318270)


