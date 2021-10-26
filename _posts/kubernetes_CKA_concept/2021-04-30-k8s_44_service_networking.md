---
layout: single
title: "[CKA Concept] 44. Service Networking"
categories: [Kubernetes]
author_profile: true
excerpt: CKA 강의의 개념 중 Service Networking에 대해 정리한다. 
toc: true
toc_sticky: true
---

## Service Networking

### Service 종류

1. ClusterIP : 클러스터안의 모든 Pod들의 통신이 가능(Node 상관없이) ex) DB
2. NodePort : 클러스터 밖의 외부로부터의 통신이 가능하도록 하는 서비스 ex) web 서비스
3. LoadBalancer : 외부에서 하나의 접속정보를 통하여 Pod과의 통신을 가능하게 하도록함

### Pod과 Service

- Pod의 생성과정
    - Pod은 각 node의 kubelet이 apiserver을 통해 Pod변경을 체크한 뒤 해당 Node에 생성한다.
    - CNI 플러그인을 통해 Pod의 네트워크를 설정한다.

- Service
    - 각 Node에서 동작하고 있는 kube-proxy가 서비스를 구성한다.
    - kube-apiserver을 통해 클러스터의 변화를 확인하고 서비스를 생성한다.
    - 서비스는 Pod과는 다르게 각 Node에 생성되는 것이 아닌 전체 Node상관없이 클러스터내에 형성되어있다.
    - 서비스의 IP를 Listen하는 실제 서비스나 서버가 존재하지 않는다.
    - 프로세스 없고 인터페이스 없고 네임스페이스 없다. = Virtual Object

<br>

#### Service에는 어떻게 접근하는가?
- 서비스가 생성되면 이미 정해져있는 범위내의 IP가 할당된다.
- 각 노드의 kube-proxy는 서비스가 생성되면 할당된 IP를 가지고 포워딩 룰을 만든다.<br>
-> 서비스 IP로 들어온 모든 트래픽을 해당 Pod의 IP로 포워딩하도록 모든 Node에 적용한다.
-> 포워딩하는 서비스의 IP는 IP와 PORT의 조합으로 이루어진다.

![](/assets/img/kubernetes/45_service_networking_1.png)

<br>

#### 포워딩 룰은 어떻게 만들어지는가?
- 3가지 방식의 proxy방식이 존재한다.
    1. userspace
    2. ipvs
    3. iptables(default)
- iptables방식
    - Pod이 생성되고 service가 생성되면서 둘다 IP를 할당 받는다.
    - 두 요소의 IP는 겹치면 안되기 때문에 범위가 미리 지정되어있다.




<br>
<br>

### 실습으로 알게 된 것들
- Node의 IP주소 범위

```bash
# node의 IP 조회
$ kubectl get node -o wide
NAME           STATUS   ROLES    AGE     VERSION   INTERNAL-IP   ...
controlplane   Ready    master   4h41m   v1.19.0   172.17.0.20   ...
node01         Ready    <none>   4h40m   v1.19.0   172.17.0.23   ...

# ip addr 명령으로 범위 조회
$ ip addr
   ...
2: ens3: ....
   ...
   inet 172.17.0.20/16...
   ...

   
```

<br>

- Pod의 IP주소 범위
    - weave의 log를 조회한 뒤 ipalloc-range를 확인한다.

```bash
# weave의 log를 조회한다.
$ kubectl logs -n kube-system <weave pod 이름> weave | grep range

INFO: 2021/04/30 03:42:34.899336 Command line options: map[conn-limit:200 datapath:datapath db-prefix:/weavedb/weave-net docker-api: expect-npc:true http-addr:127.0.0.1:6784 ipalloc-init:consensus=1 ipalloc-range:10.32.0.0/12 metrics-addr:0.0.0.0:6782 name:06:a9:f6:0f:2d:d6 nickname:controlplane no-dns:true no-masq-local:true port:6783]

```

<br>

- Service의 IP주소 범위
    - kube-apiserver의 정보를 조회하여 --service-cluster-ip-range를 확인한다.

```bash
$ kubectl describe pod -n kube-system kube-apiserver-controlplane
```

<br>

- kube-proxy의 갯수
    - kube-system 네임스페이스의 Pod을 조회하거나 daemonset을 조회한다. 

```bash
$ kubectl get pod -n kube-system

$ kubectl get ds -n kube-system
```

<br>

- kube-proxy가 사용하는 proxy 종류 조회
    - kube-proxy의 log를 조회한다.

```bash
$ kubectl get -n kube-system <kube-proxy Pod 이름>
```

<br>

- kube-proxy는 Daemon Set으로 생성된다.






------------------
**◎ 참고자료**
- Udemy - Certified Kubernetes Administrator (CKA) with Practice Tests
- [쿠버네티스 공식문서 - 네트워크 플러그인](https://kubernetes.io/ko/docs/concepts/extend-kubernetes/compute-storage-net/network-plugins/)
