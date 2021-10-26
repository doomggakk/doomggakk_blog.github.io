---
layout: single
title: "[CKA Concept] 36.Network Policy"
categories: [Kubernetes]
author_profile: true
excerpt: CKA 강의의 개념 중 Network Policy 대해 정리한다. 
toc: true
toc_sticky: true
---

## Network Policy
- 기본적으로 ```kubectl get netpol```로 조회함

### Ingress & Egress
![](/assets/img/kubernetes/36_network_policy_1.png)

- 포트에 의해서 transaction이 이루어질 때 네트워크에서는 ingress와 egress를 관리하게 된다. 
- 데이터베이스 기준으로 request 요청을 받는것이 ingress이고, 데이터를 실제로 보내는 것이 egress이다.<br>

![](/assets/img/kubernetes/36_network_policy_2.png)
- 위와같은 egress와 ingress를 설정하는 것이 Network Policy에서 하는 역할이다.<br>

![](/assets/img/kubernetes/36_network_policy_3.png)
- 기본적으로 쿠버네티스의 모든 요소들간의 통신은 **all allow** 정책으로 시행된다. 
- 클러스터내의 모든 Node들은 통신이 가능하다. 
- 이것은 보안적으로 위험할 수 있다.
- 주로 프론트엔드에서 DB로의 접근은 허용하지 않는다. 이러한 조치가 필요하다.
- 아래처럼 network policy와 Pod은 **Label**로 매칭시킨다

```yaml
# network policy
podSelector:
  matchLabels:
    role: db

---------------
# Pod
labels:
  role: db
```

<br>



- DB로 들어오는 ingress 들중 api-pod으로 오는 것만 허용하는 network policy 예이다.

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: db-policy
spec:
  podSelector:
    matchLabels:
      role: db    # network policy와 Pod 매칭
  policyTypes:
    - Ingress     # - Egress도 가능하기때문에 array형식
  ingress:
    - from:
      - podSelector:
          matchLabels:
            name: api-pod
        namespaceSelector:
          matchLabels:
            name: prod      # 네임스페이스가 prod인 api-pod의 ingress만 허용됨
      ports:
        - protocol: TCP
          port: 3306

```

- ingress와 egress

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: db-policy
spec:
  podSelector:
    matchLabels:
      role: db    # network policy와 Pod 매칭
  policyTypes:
    - Ingress
    - Egress
  ingress:
    - from:
      - podSelector:
          matchLabels:
            name: api-pod
        namespaceSelector:
          matchLabels:
            name: prod      # 네임스페이스가 prod인 api-pod의 ingress만 허용됨
      ports:
        - protocol: TCP
          port: 3306
  egress:
    - to:
      - ipBlock:
        cidr: 192.168.5.10/32 # ingress에서도 허용가능. 이 대역의 IP를 허용한다.
      ports:
        - protocol: TCP
          port: 80

```


<br>
<br>


------------------
**◎ 참고자료**

- Udemy - Certified Kubernetes Administrator (CKA) with Practice Tests
- [#33 Image Security & Network Policy - 작성자 ijoos](https://blog.naver.com/ijoos/222171813463)