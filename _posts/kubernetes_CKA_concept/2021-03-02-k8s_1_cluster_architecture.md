---
layout: single
title: "[CKA Concept] 1. Cluster Architecture"
categories: [Kubernetes]
author_profile: true
excerpt: CKA 강의의 개념 중 Cluster Architecture에 대해 정리한다. 
toc: true
toc_sticky: true
---

## Cluster Architecture

- 크게 Worker Nodes, Master Node 두개의 노드로 나뉜다
- 쿠버네티스는 물리적 vs 가상으로나 온프레미스 vs 클라우드 환경에서 컨테이너 형태의 애플리케이션을 관리하는 노드들로 이루어져 있음


> **※ 온프레미스** : 클라우드와 같은 원격 환경이 아닌 자체적으로 보유한 전산실 서버에 직접 설치해 운영하는 방식


<br>

#### · Worker Nodes  
------------
    - 컨테이너를 쌓을 수 있는 배(Cargo Ship)에 비유
    

<br>

#### · Master Node
------------
    - Control Ship에 비유. 쿠버네티스 클러스터를 관리하는 역할, 다른 노드의 컨테이너 관련된 정보들을 저장하는 역할. 
    - 여러가지 컴포넌트들이 기능하는 노드

<br>

#### · ETCD Cluster 
------------
    - 클러스터에 대한 정보를 보관해두는 곳
    - 이러한 정보들은 key-value 형식으로 데이터베이스에 저장됨
    

<br>

#### · Kube Scheduler
------------
    - 조건이 맞는 필요한 컨테이너를 Worker Node에 적재시키는 크레인과 비교하여 설명
    - 노드에 있는 컨테이너나 애플리케이션을 스케쥴링하는 역할


<br>

#### · Control Manager
------------
    - 다른 노드를 관리하는 컨트롤러들
    - Node Controller : 노드를 관리, 클러스터에 노드를 새로 올리거나 삭제하는 등의 작업을 하는 컨트롤러
    - Replication Controller : 컨테이너의 수를 유지하여 잘 실행되도록 유지하는 컨트롤러

<br>

#### · kube-apiserver
------------
    - 매우 중요한 요소, 클러스터내에서 모든 기능들을 조율하는 역할
    - 내부 및 외부 요청 처리, 최종사용자, 클러스터의 다른부분, 외부컴포넌트 등이 서로 통신할 수 있도록 HTTP API를 제공
    - REST 호출을 사용하여 API 직접 접근 할 수도 있고 커맨드라인 도구를 통해 수행할 수도 있다.

<br>

#### · kubelet
------------
    - Cargo Ship의 선장에 비유
    - 컨테이너나 노드의 상태를 마스터노드에 전달하는 역할도 함
    - kube-api의 지시를 듣고 컨테이너를 생성, 삭제함
    - kube-apiserver는 노드를 관리하기 위해 주기적으로 kubelet으로 부터 상태 보고를 받는다.

<br>

#### · kube-proxy
------------
    - Worker Node에서 컨테이너간의 통신이 필요함<br>
    ex) 웹서버 컨테이너와 DB컨테이너간의 통신 필요
    - kube-proxy를 통해 서비스간의 통신이 가능. 즉 다른 컨테이너간의 통신이 가능함
    
<br>
    
![Cluster Architecture 도식](/assets/img/kubernetes/9_cluster_architecture_1.png)

<br>

------------------
**◎ 참고자료**

- Udemy - Certified Kubernetes Administrator (CKA) with Practice Tests
- [쿠버네티스 공식문서 - 클러스터 아키텍쳐](https://kubernetes.io/ko/docs/concepts/architecture/)
- [쿠버네티스 공식문서 - 컴포넌트](https://kubernetes.io/ko/docs/concepts/overview/components/)