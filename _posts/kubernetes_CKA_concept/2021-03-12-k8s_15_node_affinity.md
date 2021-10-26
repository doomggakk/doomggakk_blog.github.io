---
layout: single
title: "[CKA Concept] 15. Node Affinity"
categories: [Kubernetes]
author_profile: true
excerpt: CKA 강의 개념 중 쿠버네티스에서의 Node Affinity에 대해 정리한다. 
toc: true
toc_sticky: true
---

## Node Affinity
- Pod을 Node에 스케줄링할 때 유연한 제약조건에따라 생성되도록 하는 기능 (Affinity=유연)
- Taint & Toleration과는 다르게 Pod에 조건을 주어서 선호하는 Node에 생성되도록 한다.
- 그렇기 때문에 Affinity를 적용하지 않은 Pod도 랜덤으로 어떤 Node든지 할당될 수 있다.
- **Node의 문은 달라도 어떤 Pod에게나 항상 개방되어있고 Pod는 자기가 선호하는 문을가진 Node에만 할당되도록 하는것**
- **그래서 Affinity가 정해지지않은 Pod들은 랜덤으로 어디든 생성 될 수 있음**

<br>

### Node Affinity 작성방법 
---------------------------

```yaml
apiVersion: 
kind: Pod
metadata:
  name: app-pod
  label:
    app: myapp
spec:
  containers:
    - name: nginx
      image: nginx
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: size
            operator: In
            values:
            - Large
            - Medium

```

- 위의 Affinity는 size = Large or Medium의 조건으로 적용된다.
- operator : ```In```, ```NotIn```, ```Exists```, ```DoesNotExist```, ```Gt```, ```Lt```
- Type of Affinity :

1.```requiredDuringSchedulingIgnoredDuringExecution``` : Pod가 Node에 스케줄되도록 **반드시** 규칙을 만족해야 하는 것(Node Selector와 같지만 좀 더 표현적인 구문 사용)
    - ex) 인텔 CPU가 있는 Node에만 파드 실행

2.```preferredDuringSchedulingIgnoredDuringExecution``` : 스케줄러가 시도하려고는 하지만 조건에 맞지 않으면 다른 곳에 스케줄링 한다.
    - ex) 장애 조치영역 XYZ에 Pod를 실행하려하지만 불가능하다면 다른곳에서 일부를 실행하도록 허용

- nodeSelector와 nodeAffinity를 모두 지정한다면 Pod가 후보 Node에 스케줄되기 위해서는 **둘 다 반드시 만족**해야한다.

- nodeSelectorTerms와 연관된 여러 matchExpressions를 지정하면 Pod는 matchExpressions를 **모두 만족**하는 Node에만 스케줄 된다. 

- Pod가 스케줄된 Node의 Label을 지우거나 변경해도 Pod는 제거되지 않는다. Affinity 선택은 **Pod를 스케줄링 하는 시점에만 작동** 한다.



cf) 추가적인 Type
```requiredDuringSchedulingRequiredDuringExecution``` : 스케줄링될 때 반드시 조건에 만족해야하며, 중간에 Label이 변경되면 Pod도 제거된다.

<br>
<br>

------------------
**◎ 참고자료**
- Udemy - Certified Kubernetes Administrator (CKA) with Practice Tests

- [쿠버네티스 공식문서 - 노드에 파드 할당하기](https://kubernetes.io/ko/docs/concepts/scheduling-eviction/assign-pod-node/)


