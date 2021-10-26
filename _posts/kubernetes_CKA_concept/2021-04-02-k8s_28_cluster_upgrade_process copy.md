---
layout: single
title: "[CKA Concept] 28. Cluster Upgrade Process"
categories: [Kubernetes]
author_profile: true
excerpt: CKA 강의의 개념 중 Cluster Upgrade Process에 대해 정리한다. 
toc: true
toc_sticky: true
---

## Cluster Upgrade Process
- cluster의 구성요소 ( kube-apiserver, controll managet, kube-scheduler 등)들의 버전이 달라도 된다.
- cluster update시 버전관리 시 유의할점 

1. kube-apiserver(X)의 버전을 기준으로 잡는다.
2. controll-manager와 kube-scheduler는 kube-apiserver의 버전보다 1낮거나 같을 수 있다.<br>(X >= version >= X-1 )
3. kubelet과 kube-proxy는 kube-apiserver의 버전보다 2낮거나 같을 수 있다.<br>(X >= version >= X-2 )
4. kubectl은 kube-apiserver의 버전보다 1크거나 1작을 수 있다.<br>(X+1> version > X-1 )

<br>
- version 업을 할 경우 1씩 차례대로 업그레이드 해야한다. <br>
(ex: v1.10 -> v1.12(x) / v1.10 -> v1.11 -> v1.12)<br>

- 업그레이드 방법<br>
1.cloud 서비스 provider를 통해 업그레이드 하는경우 버튼 클릭으로 충분하다.
2.kubeadm과 같은 Tool을 사용하여 업그레이드 하는경우 명령어를 입력하여 진행한다.

```shell
kubectl upgrade plan

kubectl upgrade apply
```

3.직접 버전업데이트를 진행한다. (어려움)

<br>

### 클러스터 업그레이드 진행과정
1. Master Cluster를 먼저 업그레이드 진행한다.
- 업그레이드 진행동안 다운되어있기 때문에 worker node는 master node와 관련된 작업을 수행할 수 없다.
- 하지만 사용자들에게 제공되고있는 서비스는 worker node가 다운되어있지 않기 때문에 정상적으로 제공된다.
<br>

2-1 모든 worker node를 한번에 업그레이드 한다.
- node의 downtime을 줄일 수 있지만 사용자에게 제공되는 서비스가 일시중단 될 수 있다.

2-2 한개의 worker node씩 순차적으로 업그레이드 한다.
- down된 node의 구성요소들은 다른 node들로 옮겨지고 업그레이드가 진행된다.
- 사용자들은 한개의 worker node가 down 되었어도 서비스를 중단없이 제공받을 수 있다.

2-3 새로운 worker node를 클러스터에 생성하여 업그레이드를 진행한다.
- 새로운 노드를 불러오고 여기에 서비스를 하나씩 담게된다. 이렇게 되면 기존의 서비스를 종료하거나 노드를 turn on/off 하는 수고를 없앨 수 있다.

<br>

### 버전 업그레이드 명령어
1. 마스터 업그레이드 명령어

```shell
# kubeadm 업그레이드
controlplane$ apt-mark unhold kubeadm && \
controlplane$ apt-get update && apt-get install -y kubeadm=1.19.6-00 && \
controlplane$ apt-mark hold kubeadm

controlplane$ sudo kubeadm upgrade plan # 버전 체크
controlplane$ sudo kubeadm upgrade apply v1.19.6

# kubelet 업그레이드 (마스터 노드)
controlplane$ kubectl drain controlplane --ignore-daemonsets
apt-mark unhold kubelet kubectl && \
apt-get update && apt-get install -y kubelet=1.19.6-00 && \
controlplane$ apt-mark hold kubelet kubectl

controlplane$ sudo systemctl daemon-reload
controlplane$ sudo systemctl restart kubelet
controlplane$ kubectl uncordon controlplane

# worker 노드 업그레이드
node01$ apt-mark unhold kubeadm && \
node01$ apt-get update && apt-get install -y kubeadm=1.19.6-00 && \
node01$ apt-mark hold kubeadm

controlplane$ kubectl drain controlplane --ignore-daemonsets # 마스터 노드에서
node01$ sudo systemctl daemon-reload
node01$ sudo systemctl restart kubelet
controlplane$ kubectl uncordon node01

```

<br>

### Demo로 알게된 것
- 클러스터에서 workload 될 수 있는 node **=** node중에 taint가 없는 node
- node에 접속하기 : **ssh <node이름>**
- cluster에 있는 **application** = **deployment**
- 노드 업그레이드 순서 코드

```shell
# kubeadm 업그레이드
apt-get update
apt-get install kubeadm=1.19.0-00
kubeadm upgrade apply v1.19.0

# 해당 node에서 kubelet 업그레이드
apt-get install kubeletl=1.19.0-00
```


<br>
<br>


------------------
**◎ 참고자료**

- Udemy - Certified Kubernetes Administrator (CKA) with Practice Tests
- [쿠버네티스 공식문서 - 초기화 컨테이너](https://kubernetes.io/ko/docs/concepts/workloads/pods/init-containers/)
- [#25 Cluster Maintenance (drain, uncordon) - 작성자 ijoos](https://blog.naver.com/ijoos/222167125237)