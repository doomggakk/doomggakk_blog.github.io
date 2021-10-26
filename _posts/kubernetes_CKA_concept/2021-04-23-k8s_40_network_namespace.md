---
layout: single
title: "[CKA Concept] 40. Network Namespace"
categories: [Kubernetes]
author_profile: true
excerpt: CKA 강의 개념 중 쿠버네티스에서의 Network Namespace에 대해 정리한다. 
toc: true
toc_sticky: true
---

## Network Namespace

### Namespace
- 네트워크의 방이라고 볼 수 있다.
- 방끼리 연결을 시킬수도 있고, 관리자는 외부에서 방안의 내용들을 확인 할 수 도있다.
- 방에 속해있는 사람들은 다른방의 내용을 보지 못한다.
- 하나의 컨테이너가 있을 때 컨테이너 내부와 외부에서 서로 다른 프로세스 아이디로 실행되는 동일한 프로세스라고 생각한다.

<br>

### 라우팅 테이블, ARP 테이블
- **Routing table** : 앞장에서 봤듯이 gateway와 destination을 설정해주는 table
- **ARP** : MAC주소와 IP주소를 매핑시켜놓은 테이블
  - IP주소 : 호스트나 라우터 장비의 인터페이스에 할당된 32bit(Ipv4)의 주소
  - MAC주소 : 데이터 링크 계층에서 사용하는 네트워크 인터페이스(보통 NIC=Network Interface Card)에 할당된 고유 식별주소. *물리 주소나 하드웨어 주소*라고도 불린다.
- **데이터를 받거나 보내려할 때 MAC주소와 IP주소를 둘다 사용하는이유** : <br>IP 주소는 보낸 주소에서부터 도착지 주소까지의 경로를 찾기위해 필요한 주소이며,<br>MAC주소는 한 주소내에서 다양한 기기들이 있을 때, 해당 기기들이 각각 어떤 기기들인지 식별하기 위하여 필요한 주소이다.

<br>

### 네임스페이스 조회 및 arp, route 테이블 조회

```bash
# ip 네임스페이스 조회
ip netns


# ip 네임스페이스 추가
ip netns add <namespace 이름>


# network interface 조회
ip link
1: lo: .....

2: eth0: .....


# namespace내에서 network interface 조회
ip netns exec <namespace 이름> ip link
= ip -n red link
1: lo: .....

=> eth0은 조회되지 않는다.


# arp 테이블 조회
arp


# route 테이블 조회
route


```

<br>

### veth
- 네임스페이스는 바깥 호스트의 eth0 인터페이스를 사용하지 않는다.
- 그래서 네트워크 인터페이스가 존재하지 않는다.
- 따라서 네임스페이스안에 virtual 인터페이스(veth)를 생성해주어야한다.

![](/assets/img/kubernetes/41_network_2.png)

```bash
** 네임스페이스 red와 blue를 연결하는 veth 생성하기

1. 한쌍의 veth 생성
$ ip link add veth-red type veth peer name veth-blue


2. 각 네임스페이스에 veth를 할당해주기
$ ip link set veth-red netns red

$ ip link set veth-blue netns blue


3. 할당된 veth에 IP주소 할당해주기
$ ip -n red addr add 192.168.15.1 dev veth-red

$ ip -n blue addr add 192.168.15.2 dev veth-blue


4. 인터페이스 활성화
$ ip -n red link set veth-red up

$ ip -n blue link set veth-blue up


5. red 네임스페이스에서 blue로 ping 날려보기
$ ip netns exec red ping 192.168.15.2


6. 각 네임스페이스 arp 테이블 조회
$ ip netns exec red arp
Address         HWtype       HWaddress             Flags Mask      Iface
192.168.15.2    ether        ba:b0:6d:68:09:e9     C               veth-red

$ ip netns exec blue arp
Address         HWtype       HWaddress             Flags Mask      Iface
192.168.15.1    ether        7a:9d:9b:c8:3b:7f     C               veth-blue

$ arp
Address         HWtype       HWaddress             Flags Mask      Iface
192.168.1.3     ether        52:54:00:12:35:03     C               eth0
192.168.1.4     ether        52:54:00:12:35:04     C               eth0

```

<br>

### 호스트 내 가상네트워크
- 여러 네임스페이스들이 서로 통신을 하게 하려면 어떻게 해야하는가
- 가상 네트워크와 스위치를 생성해주어야 한다.

![](/assets/img/kubernetes/41_network_1.png)

```bash
# 가상네트워크(브릿지 네트워크) 생성
$ ip link add v-net-0 type bridge


# 인터페이스 조회 시 가상네트워크도 추가되어있음
$ ip link
1: lo: ....

2: eth0: ....

6: v-net-0: .....


# 가상네트워크 스위치 활성화
$ ip link set dev v-net--0 up


# 위에서 생성했던 red <-> blue 연결 제거
$ ip -n red link del veth-red
=> 한쪽을 삭제하면 다른 한쪽도 사라진다.

----------------------------------------------------

** red 네임스페이스와 브릿지 네트워크와 연결하기

# 브릿지 네트워크와 연결될 가상 인터페이스 생성(red 네임스페이스)
$ ip link add veth-red type veth peer name veth-red-br


# veth-red 인터페이스 red네임스페이스에 연결
$ ip link set veth-red netns red


# veth-red-br 인터페이스 v-net-0 브릿지 네트워크에 연결
$ ip link set veth-red-br master v-net-0


# veth-red인터페이스에 ip주소 할당
$ ip -n red addr add 192.168.15.1 dev veth-red


# red 인터페이스에 새로 연결한 인터페이스 활성화
$ ip -n red link set veth-red up

-------------------------------------------------

** 호스트 네트워크와 네임스페이스 연결하기

# 브릿지 네트워크에도 ip주소 할당
$ ip addr add 192.168.15.5/24 dev v-net-0


# 호스트에서 red 네임스페이스로 ping 날릴시 성공적으로 응답함
$ ping 192.168.15.1


```
<br>

### 호스트 밖의 네트워크와 통신하기

``` bash
# 로컬 호스트는 지난 강의에서처럼 LAN과 통신 할 수 있는 Gateway가 설정되어있다.
# bridge로 연결되어있는 네임스페이스와 LAN을 연결시켜주어야한다.

** 네임스페이스와 LAN 통신하기
# v-net-0 인터페이스를 Gateway로하여 호스트와 통신
$ ip netns exec blue ip route add 192.168.1.0/24 via 192.168.15.5

# ping을 날려봤을 경우 응답이 오지 않는다.
-> 그이유는 호스트에서 외부로 나가는 패킷들은 목적지가 정해져있지만 외부에서 들어오는 패킷들은 목적지가 불문명하다. 
-> 따라서 NAT을 설정해주어 내부IP를 공인 IP로 변경시켜주는 기능이 가능하도록 해야한다.

$ iptables -t nat -A POSTROUTING -s 192.168.15.0/24 -j MASQUERADE


** 네임스페이스와 외부 인터넷 통신하기
# 외부 인터넷과 통신할 수 있도록 Default Gateway를 열어준다.
$ ip netns exec blue ip route add default via 192.168.15.5


** 네임스페이스의 blue 네임스페이스 80포트를 사용하는 web어플리케이션 접속가능하도록 하기
# 로컬호스트의 80포트로 들어오는 모든 트래픽을 blue네임스페이스의 80포트로 포워딩해준다.
$ iptables -t nat -A PREROUTING --dport 80 --to-destination 192.168.15.2:80 -j DNAT



```
![](/assets/img/kubernetes/41_network_3.png)
![](/assets/img/kubernetes/41_network_4.png)


<br><br>

------------------
**◎ 참고자료**
- Udemy - Certified Kubernetes Administrator (CKA) with Practice Tests

