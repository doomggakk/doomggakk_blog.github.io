---
layout: single
title: "[CKA Concept] 47. Designing k8s cluster"
categories: [Kubernetes]
author_profile: true
excerpt: CKA 여러가지 개념 중 Designing k8s cluster 대해 정리한다. 
toc: true
toc_sticky: true
---

## Designing k8s cluster

### 목적
- 교육용
  - minikube
  - 싱글 노드 클러스터 (kubeadm/GCP/AWS)

- 개발 및 테스트용
  - 멀티 노드 클러스터 (1개의 Master노드와 멀티 Worker노드)
  - kubeadm, GCP, AWS, AKS등의 빠른 Provision 서비스를 이용 

- 운영
  - 여러개의 Master노드를 가진 High Availability 멀티 노드 클러스터
  - kubeadm, GCP, Kops(AWS), 다른 플랫폼등을 이용
  - 노드 5,000개 까지
  - Pod 150,000개 까지
  - 컨테이너 300,000개 까지
  - 노드당 Pod 100개까지

![](/assets/img/kubernetes/48_design_cluster_1.png)
=> 예시로 AWS의 M3.xlarge를 사용할경우 11~100개의 노드를 생성할 수 있다.


<br>

### Cloud? OnPrem?

- 온프레미스 환경에서는 kubeadm을 사용하면 유용하다.
- GKE for GCP
- Kops for AWS
- AKS(Azure K8s Service) for Azure

<br>

### Storage
- 고사양 작업 : SSD Backed Storage
- 여러 connection이 요구되는 작업 : Network based Storage
- Persistent storage volumes : 여러 Pod에서 볼륨에 대해 액세스를 공유하는 경우

<br>

### Node
- virtual / physical 노드로 구성할 수 있다. -> 클라우드에서는 virtual 머신이 선택될 것이다.
- master 노드, worker 노드 갯수를 여러개 선택할 수 있으며 master노드를 worker노드로 사용할 수 있지만 분리 되는 것을 권장한다.


<br>
<br>






------------------
**◎ 참고자료**
- Udemy - Certified Kubernetes Administrator (CKA) with Practice Tests
- [쿠버네티스 공식문서 - DNS 서비스 사용자 정의하기](https://kubernetes.io/ko/docs/tasks/administer-cluster/dns-custom-nameservers/)
