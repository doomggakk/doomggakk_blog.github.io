---
layout: single
title: "[CKA Concept] 42. CNI & Cluster Networking"
categories: [Kubernetes]
author_profile: true
excerpt: CKA 강의의 개념 중 CNI 대해 정리한다. 
toc: true
toc_sticky: true
---

## CNI(Container Networking Interface)

### 네트워크 네임스페이스
1. 네트워크 네임스페이스 생성
2. 브릿지 인터페이스 & 네트워크 생성
3. 가상 케이블 VETH 생성(가상인터페이스, 파이프)
4. vEth 네임스페이스에 붙이기
5. 반대쪽 vEth 브릿지에 붙이기
6. IP 주소 할당
7. 인터페이스 up 시키기
8. NAT - IP Masquerade 사용하여 외부 통신 수행

<br>

-> rkt, mesos 등 컨테이너 기반 솔루션의 네트워크 구성도 매우 비슷하다.
-> CNI를 도입하게됨. CNI는 **컨테이너 런타임 환경에서 네트워킹 문제를 해결하기 위해 프로그램이 개발되어야하는 방법을 정의하는 표준세트**이다.

<br>

### CNI

- CNI는 여러가지 플러그인을 지원한다.
- 하지만 도커는 CNI가아닌 CNM을 사용한다.
- 도커 컨테이너로 CNI를 사용못하는것은 아니다.
- 아래와 같이 none 네트워크로 생성한 뒤 따로 붙여주면 된다.
- 쿠버네티스가 이러한 방식으로 CNI를 사용한다.
- CNI 플러그인의 책임
  - ADD, DEL, CHECK Argument를 지원해야한다.
  - 컨테이너ID, network 네임스페이스등 파라미터를 지원해야한다.
  - Pod에 할당된 IP주소를 관리해야한다.
  - 특정한 형식으로 결과를 리턴해야한다.

![](/assets/img/kubernetes/43_cni_1.png)

<br>

### Cluster Networking
- 쿠버네티스는 마스터와 Worker 노드로 구성된다.
- 각 노드는 네트워크와 연결된 인터페이스가 하나 이상 있어야 한다.
- 각 인터페이스에는 주소가 있어야하며, 호스트에는 호스트이름, MAC주소도 있어야 한다.
- 쿠버네티스의 각 요소들(ETCD, Kube-apiserver, Kubelet, Kube-Controller-Manager 등)은 기본적으로 아래의 Port번호가 할당 된다.

![](/assets/img/kubernetes/43_cni_2.png)

<br>

### 실습으로 알게된 것
- 마스터노드의 interface, MAC IP 알아보기
    - 마스터노드의 IP와 맞는 인터페이스를 찾는다.

```bash
$ kubectl get node -o wide
NAME           STATUS   ROLES    AGE     VERSION   INTERNAL-IP   EXTERNAL-IP ...
controlplane   Ready    master   5m35s   v1.18.0   172.17.0.24   <none>      ...
node01         Ready    <none>   5m6s    v1.18.0   172.17.0.40   <none>      ...

$ ifconfig

        ...

docker0: flags=4099<UP,BROADCAST,MULTICAST>  mtu 1500
        inet 172.18.0.1  netmask 255.255.255.0  broadcast 172.18.0.255
        ether 02:42:74:91:1b:66  txqueuelen 0  (Ethernet)
        RX packets 0  bytes 0 (0.0 B)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 0  bytes 0 (0.0 B)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

ens3: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 172.17.0.24  netmask 255.255.0.0  broadcast 172.17.255.255
        inet6 fe80::42:acff:fe11:18  prefixlen 64  scopeid 0x20<link>
        ether 02:42:ac:11:00:18  txqueuelen 1000  (Ethernet)
        RX packets 35433  bytes 43385991 (43.3 MB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 10480  bytes 6533653 (6.5 MB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

        ...

```

<br>

- Worker 노드의 정보알아보기

```bash
$ kubectl get node -o wide
NAME           STATUS   ROLES    AGE     VERSION   INTERNAL-IP   EXTERNAL-IP ...
controlplane   Ready    master   5m35s   v1.18.0   172.17.0.24   <none>      ...
node01         Ready    <none>   5m6s    v1.18.0   172.17.0.40   <none>      ...


$ ssh node01 ifconfig

        ...

docker0: flags=4099<UP,BROADCAST,MULTICAST>  mtu 1500
        inet 172.18.0.1  netmask 255.255.255.0  broadcast 172.18.0.255
        ether 02:42:17:62:c3:29  txqueuelen 0  (Ethernet)
        RX packets 0  bytes 0 (0.0 B)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 0  bytes 0 (0.0 B)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

ens3: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 172.17.0.40  netmask 255.255.0.0  broadcast 172.17.255.255
        inet6 fe80::42:acff:fe11:28  prefixlen 64  scopeid 0x20<link>
        ether 02:42:ac:11:00:28  txqueuelen 1000  (Ethernet)
        RX packets 93161  bytes 126396113 (126.3 MB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 20806  bytes 2203467 (2.2 MB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

        ...
```

<br>
- route 확인 하기 ( 어떤 게이트웨이를 사용중인가 등 )

```bash
$ route
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
default         _gateway        0.0.0.0         UG    0      0        0 ens3
10.244.0.0      0.0.0.0         255.255.255.0   U     0      0        0 cni0
10.244.1.0      node01          255.255.255.0   UG    0      0        0 ens3
                            ...

$ ip route
default via 172.17.0.1 dev ens3  # 구글같은 사이트의 route. 172.17.0.1 게이트웨이를 통해 이루어짐
10.244.0.0/24 dev cni0 proto kernel scope link src 10.244.0.1 
10.244.1.0/24 via 172.17.0.40 dev ens3 
172.17.0.0/16 dev ens3 proto kernel scope link src 172.17.0.24 
172.18.0.0/24 dev docker0 proto kernel scope link src 172.18.0.1 linkdown 

```

<br>

- 쿠버네티스 요소들 PORT 확인

```bash
$ netstat -natulp | grep <검색할 요소>
```

<br>
<br>


------------------
**◎ 참고자료**
- Udemy - Certified Kubernetes Administrator (CKA) with Practice Tests
