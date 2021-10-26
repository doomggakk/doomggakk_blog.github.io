---
layout: single
title: "[Kubernetes] 5. Networking in k8s"
categories: [Kubernetes]
author_profile: true
excerpt: Kubernetes에서의 Networking에 대해 정리한다.
toc: true
toc_sticky: true
---


## Networking in k8s

### Single Node
- node 는 ip를 가짐<br>
**->** 현재 가상환경(hypervisor)에 minikube는 나의 시스템안에 다른 ip를 가진 노드로써 존재함<br>
- **k8s**에서는 **Pod**에 IP주소가 할당됨<br>
**->** Pod간에 링크를 생성할 필요가 없고 컨테이너 포트를 호스트 포트에 매핑할 필요가 거의 없음<br>
**->** **Docker**에서는 **컨테이너**에 IP주소가 할당됨<br>
- 쿠버네티스가 생성되면 **Internal Private Network**가 어떤 IP의 초기값을 가지고 생성됨 **ex) 10.244.0.0**<br>
**->** 내부네트워크에 모든 Pod 들이 연결됨<br>
**->** 다른 POD은 각각 다른 ip 주소를 가짐<br>
ex) 3개의 Pod IP = 10.244.0.2 , 10.244.0.3 , 10.244.0.4<br>
**->** Pod은 서로 이 IP주소를 통해 통신한다.<br>
**->** 그러나 다른 Pod에 이 IP를 가지고 접근하려한다면 Pod이 재생성될때마다 IP가 변하기 때문에 바람직하지 않다.<br>

### Cluster Networking (Multi Nodes)
- 다른 Node의 **Internal Private Network**가 **10.244.0.0**이고,
 각 노드에서 생성한 Pod이 동일한 IP주소를 갖게되는 현상이 발생할 수 있음.<br>
 -> IP주소 충돌로 문제 발생
- 모든 컨테이너나 Pod은 다른 Pod이나 컨테이너와 NAT(Network Address Translation)없이 통신가능함
- 모두 NAT없이 모든 컨테이너와 통신 할 수 있으며 그 반대도 마찬가지임

<br>

**※ NAT(Network Address Translation)**<br>
            - IP 패킷에 있는 출발지 및 목적지의 IP주소와 TCP/UDP포트숫자등을 바꿔 재기록하면 네트워크 트래픽을 주고받게하는 기술<br>
            - 쓰는 이유 : 1. IP 주소 절약, 2. 보안<br>
            
![](/assets/img/kubernetes/5_networking_1.png)<br>출처 : 위키피디아

<br>
<br>

**※ MAC(Media Access Control)주소**<br>
            - 네트워크 카드 하드웨어에 부여되는 고유한 물리적 주소<br>
            - 모든 네트워크 장비나 랜카드에는 고유한 MAC주소를 가지고 있음<br>
            - 랜카드 하나하나마다, 스위치, 라우터 등에도  서로 다른 MAC 주소가 존재<br>
            - 네트워크 상에서 통신을 할때 서로를 구분하여 인식하기 위한 일종의 주소<br>
            - 네트워크상에서 통신 할때 TCP/IP를 이용하여 IP주소를 사용하지만 이 IP를 다시 MAC으로 변환하는 과정이 있음 -> ARP<br>

<br>

------------------
**◎ 참고자료**


- Udemy - Kubernetes for the Absolute Beginners - Hands-on
- [위키백과 - MAC 주소](https://ko.wikipedia.org/wiki/MAC_%EC%A3%BC%EC%86%8C)
- [네이버 블로그- Phantom Network - [네트워크] 맥 어드레스란? ( MAC Address )](https://blog.naver.com/shk50611/221433227927)
- [쿠버네티스 공식문서 - 클러스터 네트워킹](https://kubernetes.io/ko/docs/concepts/cluster-administration/networking/)
- [ [이해하기] NAT (Network Address Translation) - 네트워크 주소 변환](https://www.stevenjlee.net/2020/07/11/%EC%9D%B4%ED%95%B4%ED%95%98%EA%B8%B0-nat-network-address-translation-%EB%84%A4%ED%8A%B8%EC%9B%8C%ED%81%AC-%EC%A3%BC%EC%86%8C-%EB%B3%80%ED%99%98/)