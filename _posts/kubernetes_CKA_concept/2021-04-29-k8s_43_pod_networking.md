---
layout: single
title: "[CKA Concept] 43. Pod Networking & IPAM"
categories: [Kubernetes]
author_profile: true
excerpt: CKA 강의의 개념 중 Pod Networking와 IPAM 대해 정리한다. 
toc: true
toc_sticky: true
---

## Pod Networking

### 네트워킹 모델
- 쿠버네티스의 Pod 네트워크는 다음과 같은 조건을 만족해야한다.
1. 모든 Pod이 IP주소를 가져야한다.
2. 모든 Pod이 **같은 Node에 있는 Pod과 통신**이 가능해야한다.
3. 모든 Pod은 NAT없이 **다른 Node의 Pod과도 통신**이 가능해야한다.

<br>


#### 1. 모든 Pod이 IP주소를 갖기 & 모든Pod이 같은 노드의 Pod과 통신하기

```bash
1. 노드마다 IP가 할당되고, 컨테이너들을 생성한다.

2. 각 노드마다 브릿지 네트워크를 생성하고 up시키기
$ ip link add v-net-0 type bridge

$ ip link set v-net-0 up

3. 브릿지 네트워크에 IP주소 할당
ip addr add 10.244.1.1/24 dev v-net-0 

4. 스크립트를 만들어 컨테이너마다 브릿지에 연결하도록 한다.
< net.script.sh 파일 >
# Create veth pair
ip link add ...

# Attach veth pair
ip link set ...
ip link set ...

# Assign IP Address
ip -n <namespace> addr add ...
ip -n <namespace> route add ...

# Bring up Interface
ip -n <namespace> link set

```

![](/assets/img/kubernetes/44_pod_networking_1.png)

<br>

#### 2. 모든 Pod이 다른 노드의 Pod과 통신하기
- Node1의 blue Pod에서 Node2의 Purple Pod으로 통신하기

```bash
1. node1의 Route 테이블에 node2의 네트워크 연결 추가
node1$ ip route add 10.244.2.2 via 192.168 1.12 
-> node2, node3이 있다면 이 노드들에도 다른 노드와 통신할 수 있도록 Route 테이블에 추가해준다.

2. 위의 과정이 번거로우므로 Route 테이블을 생성하여 각노드의 브릿지네트워크의 IP들로 GATEWAY를 통해 통신할 수 있도록 설정해준다.
-> 규모가 큰 환경에서는 제약이 있다.
ex) NETWORK             GATEWAY
    10.244.1.0/24       192.168.1.11 # node1
    10.244.2.0/24       192.168.1.12 # node2
    10.244.3.0/24       192.168.1.13 # node3

```

![](/assets/img/kubernetes/44_pod_networking_2.png)

<br>

#### 3. 1번의 과정이 번거롭기 때문에 CNI를 사용
- 아래 스크립트를 실행해 주어야 한다.
- kubelet에서 컨테이너를 생성할 때 /etc/cni/bin 디렉토리에서 스크립트 파일을 찾는다.
- 찾고나서 스크립트파일을 실행한다.

```bash
# net-script.sh
ADD)
    # Create veth pair
    # Attach veth pair
    # Assign IP Address
    # Bring Up Interface
    ip -n <namespace> link set ...

DEL)
    # Delete veth pair
    ip link del ...

```

![](/assets/img/kubernetes/44_pod_networking_3.png)

<br>


### CNI in K8s
- kubelet 옵션에서 CNI정보를 확인 할 수 있다.

```bash
$ ps -aux | grep kubelet
        ...
--cni-bin-dir=/opt/cni/bin --cni-conf-dir=/etc/cni/net.d --network-plugin=cni

# /opt/cni/bin 모든 CNI 플러그인을 볼 수 있음.
$ ls /opt/cni/bin


# /etc/cni/net.d cni conf파일을 볼 수 있음. 예시는 bridge config파일
$ ls /etc/cni/net.d
10-bridge.conf

```

<br>

#### 다른노드와의 통신, CNI 사용 예제1
- 규모가 큰 환경에서 Node간의 통신을 위해 CNI를 사용한다.
- 각 노드마다 agent를 배치하여 agent끼리 통신을한다.
- 통신하려는 패킷을 agent가 중간에서 가로챈뒤 적절한 목적지를 설정해준다.
- 패킷을 받은 다른 노드의 agent는 해당 목적지에 전달해준다.

![](/assets/img/kubernetes/44_pod_networking_4.png)

<br>

#### 다른 노드와의 통신, CNI 사용 예제2
- 각 노드에 CNI Agent가 배치되어있다.(ex) weave)
- 각 agent는 **노드, Pod의 정보, topology**(컴퓨터 네트워크의 요소들(링크, 노드 등)을 물리적으로 연결해 놓은 것)등 노드와 관련된 정보를 가지고 있다.
- agent끼리 통신하면서 다른 노드들의 정보도 다 알고 있다.
1. 패킷이 한 Pod에서 다른 Pod으로 전송되면 패킷을 weave(agent)가 가로채서 어떤 네트워크에 있는지 확인 후에 Node를 목적지로 설정하여 패키징한다.
2. 네트워크를 통해 전송한다.
3. 목적지 노드에 있는 weave(agent)는 패킷을 검색하고 압축을 푼뒤 패킷을 올바른 Pod으로 라우팅한다.

<br>

#### Weave 배포
- 클러스터의 각 노드에 서비스 또는 데몬셋으로 배포한다.
- 또는 k8s에 이미 설정되어있는 경우, 클러스터에 Pod으로 배포하는 것이 더 쉬운 방법이다.
- 기본 k8s 시스템이 노드로 준비되어있고, 노드들의 네트워크가 올바르게 설정되어있다면 kubectl apply 명령어로 클러스터에 쉽게 배포될 수 있다.

```bash
$ kubectl apply -f "https://cloud.weave.works/k8s/net?k8s-version=$(kubectl version | base64 | tr -d '\n')"
```

<br>

## IPAM(IP Address Management)
- 각 Node의 Pod에 IP가 중복되지않도록 할당하는 방법<br> -> Node마다 IP리스트 파일을 만들어 IP를 나열하고 사용중이지 않는 IP를 할당시키는 것이다.

![](/assets/img/kubernetes/44_pod_networking_5.png)

<br>

- 이러한 작업을 **CNI의 host-local** 플러그인을 호출하면 사용할 수 있다.
- /etc/cni/net.d/스크립트.conf 에서 "ipam"의 설정을 "host-local"로 해주어 해당기능이 동작하도록 한다.

![](/assets/img/kubernetes/44_pod_networking_6.png)

<br>

- weave의 ip 범위는 10.32.0.0/12이다
- 이 범위에 할당된 IP의 개수는 약 1,048,574개로 매우많다.


<br>
<br>


------------------
**◎ 참고자료**
- Udemy - Certified Kubernetes Administrator (CKA) with Practice Tests
- [쿠버네티스 공식문서 - 네트워크 플러그인](https://kubernetes.io/ko/docs/concepts/extend-kubernetes/compute-storage-net/network-plugins/)
