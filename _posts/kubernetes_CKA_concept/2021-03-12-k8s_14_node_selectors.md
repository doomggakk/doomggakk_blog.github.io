---
layout: single
title: "[CKA Concept] 14. Node Selectors"
categories: [Kubernetes]
author_profile: true
excerpt: CKA 강의 개념 중 쿠버네티스에서의 Node Selectors에 대해 정리한다. 
toc: true
toc_sticky: true
---

## Node Selectors
- Pod를 Node에 생성할 때 어떤 Node에 생성할지 제약을 주는 Node 선택 제약조건
- Pod의 spec에 기입한다.
- 키-값의 매핑으로 지정한다.
- Pod이 Node에서 동작할 수 있으려면, Node는 키-값 쌍으로 표시되는 Label을 각자 가지고 있어야한다.
- 일반적으로 하나의 키-값 쌍이 사용된다.
- **2가지 방법 중** 간단하고 쉬운 **첫번째 방법**

<br>

### Node Selectors 작성방법
-----------------------

```yaml
apiVersion: 
kind: Pod
metadata:
  name: app-pod
  labels:
    app: app
spec:
  containers:
    - name: nginx
      image: nginx
  nodeSelector:   #<-- nodeSelector 추가
    size : Large
```

- Pod을 정의하는 yaml 파일에서 spec에 기입한다.
- nodeSelector에서 정의한 node의 label은 어떻게 처리되는가?

-> **Pod 생성 전 node에 label을 부여해야한다.**

<br>

### Node에 Label 입력

```shell
kubectl label nodes [node 이름] [키-값 쌍의 label]

ex) kubectl label nodes node01 size=Large
```

<br>

----------------------------
<br>

- Node selector은 앞서 말했듯이 간단하고 쉬운 방법이다.
- 따라서 NOT이나 OR과 같은 세부조건을 적용할 수 없다.
- 이런 조건을 적용하기 위해서는 그 다음에 정리할 **Node Affinity**를 이용해야한다.








<br>
<br>

------------------
**◎ 참고자료**
- Udemy - Certified Kubernetes Administrator (CKA) with Practice Tests

- [쿠버네티스 공식문서 - 노드에 파드 할당하기](https://kubernetes.io/ko/docs/concepts/scheduling-eviction/assign-pod-node/)


