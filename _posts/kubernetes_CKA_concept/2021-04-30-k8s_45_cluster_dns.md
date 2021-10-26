---
layout: single
title: "[CKA Concept] 45. Cluster DNS & Core DNS"
categories: [Kubernetes]
author_profile: true
excerpt: CKA 강의의 개념 중 Cluster DNS와 Core DNS에 대해 정리한다. 
toc: true
toc_sticky: true
---

## Cluster DNS
- 쿠버네티스는 cluster를 구성할때 default로 DNS 서버를 구성한다.
- DNS Resolution : 도메인 이름을 IP주소로 변경하는 과정
- Service가 생성되면 DNS 서버는 서비스의 이름과 IP를 매핑하여 기록해놓는다.
- 그래서 아래의 경우 test Pod에서 ```curl http://web-service```명령을 날려 접속이 가능하다.

![](/assets/img/kubernetes/46_cluster_dns_1.png)

<br>

- 서비스의 경우 어느곳에서든 서비스이름.네임스페이스.타입.루트로 접속이 가능하다.
- 아래의 예시사진을 보면 이해가 더욱 쉬울 것이다.

![](/assets/img/kubernetes/46_cluster_dns_2.png)

<br>

- Pod의 경우에는 Pod의 이름이 아닌 IP를 Hostname으로 설정하되 .을 -로 변경하여 저장한다.
- Type은 Pod으로 설정된다.

![](/assets/img/kubernetes/46_cluster_dns_3.png)

<br>

## Core DNS

### DNS 서버
- 기본적으로 Pod이나 Service의 DNS는 각 노드의 /etc/hosts에서 설정이 가능하다.
- 하지만 이러한 방식은 규모가 커지게 되면 번거롭고 관리하기가 불편하다.
- 그래서 DNS Server를 두어 DNS를 중앙화 시킨다고 이전 강의에서 정리하였다.
- 새로운 요소가 생성될 때마다 DNS 서버를 등록하고 DNS서버에는 새로생성된 요소의 IP와 이름을 매핑하여 저장해둔다.
- Service는 이름을, Pod은 IP에서 .을 -로 바꾼형식을 저장한다.
- **k8s에서도 DNS Server가 있는데 이것을 CoreDNS**라고 한다.

<br>

### Core DNS
- CoreDNS 서버는 쿠버네티스 클러스터에서 kube-system 네임스페이스의 Pod으로 생성된다.
- ReplicaSet에 의해 복제본까지 2개의 Pod으로 구성된다.
- ReplicaSet과 함께 Deployment형식으로 생성된다.
- 다른 요소들이 DNS 서버를 통신할 수 있도록 Service도 생성된다.
- coredns는 ./Coredns에 위치하는 Deployment이다.
- coredns는 config 파일이 필요하다. ->  /etc/coredns/Corefile로 생성된다.

<br>

![](/assets/img/kubernetes/46_cluster_dns_4.png)
- kubernetes : 쿠버네티스 플러그인이다. **cluster.local**은 기본 도메인 top level 도메인 주소이다. pods insecure은 Pod ip주소의 .을 -로 변경되어 이름을 지정하는 것을 허용하는지 안하는지? 확인하는 것 같다.
- proxy : nameserver을 설정하는 파일이다. 쿠버네티스 기본적으로 설정되어있는 위치이다.
- 위의 config파일은 Pod에게 configmap으로 전달된다. 아래의 명령어로 조회한다

```
kubectl get configmap -n kube-system
```
<br>

#### Pod에서 Core DNS 바라보기
- 각 Node마다 네임서버(CoreDNS)가 설정되는데 ip주소는 어떤 것으로 설정하는가?<br>
-> CoreDNS의 Service IP로 설정한다.
-> CoreDNS의 Service는 Default로 kube-dns라는 이름을 가진다.
-> 위의 과정은 쿠버네티스가 자동으로 Pod이 생성될 때 설정해준다.
-> 그중 에서도 **Kubelet**이 하는 작업이다.
- kubelet의 config파일을 확인해보면 DNS서버의 도메인과 IP주소를 확인 할 수 있다.

<br>

### 실습으로 알게된 것들
- 클러스터의 dns 확인

```bash
$ kubectl get pod -n kube-system
NAME                                   READY   STATUS    RESTARTS   AGE
coredns-f9fd979d6-ckh2j                1/1     Running   0          61m
coredns-f9fd979d6-nwmq8                1/1     Running   0          61m
                        ...
                        
```

- Pod이 생성될 때 Nameserver IP로 설정되는 것 = dns Service의 IP

```bash
$ kubectl get svc -n kube-system
NAME       TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)                  AGE
kube-dns   ClusterIP   10.96.0.10   <none>        53/UDP,53/TCP,9153/TCP   61m
                        
```

- coreDNS의 config파일 위치

```bash
$ kuebectl get deployment coredns -n kube-system
    
    ...
Pod Template:
  Labels:           k8s-app=kube-dns
  Service Account:  coredns
  Containers:
   coredns:
    Image:       k8s.gcr.io/coredns:1.7.0
    Ports:       53/UDP, 53/TCP, 9153/TCP
    Host Ports:  0/UDP, 0/TCP, 0/TCP
    Args:
      -conf
      /etc/coredns/Corefile   # <------ Corefile위치
      ...

```

<br>

- configmap (dns config파일)을 확인해서 기본 top level 도메인 확인

```bash
$ kubectl describe configmap coredns -n kube-system 

Name:         coredns
Namespace:    kube-system
Labels:       <none>
Annotations:  <none>

Data
====
Corefile:
----
.:53 {
    errors
    health {
       lameduck 5s
    }
    ready
    kubernetes cluster.local in-addr.arpa ip6.arpa { # <----- 기본 도메인 확인
       pods insecure
       fallthrough in-addr.arpa ip6.arpa
       ttl 30
    }
    prometheus :9153
    forward . /etc/resolv.conf {
       max_concurrent 1000
    }
    cache 30
    loop
    reload
    loadbalance
}

Events:  <none>

```









<br>
<br>






------------------
**◎ 참고자료**
- Udemy - Certified Kubernetes Administrator (CKA) with Practice Tests
- [쿠버네티스 공식문서 - DNS 서비스 사용자 정의하기](https://kubernetes.io/ko/docs/tasks/administer-cluster/dns-custom-nameservers/)
