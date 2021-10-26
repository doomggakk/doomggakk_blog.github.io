---
layout: single
title: "[CKA Concept] 12. Labels and Selectors"
categories: [Kubernetes]
author_profile: true
excerpt: CKA 강의 개념 중 쿠버네티스에서의 Labels과 Selectors에 대해 정리한다. 
toc: true
toc_sticky: true
---

## Labels
- Pod과 같은 오브젝트에 첨부된 **키와값의 쌍**
- 오브젝트의 특성을 식별하는데 사용되며 코어 시스템에 직접적인 영향을 주지는 않는다.
- 오브젝트를 수정할 때나 생성 이후에나 언제든지 수정이 가능하다.
- 오브젝트마다 키와 값으로 Label을 정의할 수 있다.
- 오브젝트의 키는 고유한 값이어야 한다.
- 접두사 kubernetes.io/, k8s.io/ 는쿠버네티스의 핵심 컴포넌트로 예약되어있다.

<br>

## Selectors
- 특별한 개념은 아니고 오브젝트의 Label을 기반으로 검색할 수 있는 개념
- Label은 고유하지 않다.
- 클라이언트와 사용자는 레이블 셀렉터를 통해 오브젝트를 식별할 수 있음
- 레이블 셀렉터는 코어그룹의 기본




ex)

```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: webapp
  labels:
    app: App1
    function: Front-end
spec:
  replicas: 3
  selector:
    matchLabels:
      app: App1    #  ───┐
  metadata:        #     │
    metadata:      #     │
      labels:      #     │
        app: App1  # <───┘
        function: Front-end
    spec:
      containers:
        - name: simple-webapp
          image: simple-webapp
```
<br><br>

### Label을 가지는 오브젝트 조회
- Label을 가지는 Pod 조회

```shell
kubectl get pods --selector [label 키]=[label 값]

kubectl get pods -l [label 키]=[label 값]
```

- Label을 가지는 모든 오브젝트 조회

```shell
kubectl get all --selector [label 키]=[label 값]

kubectl get all -l [label 키]=[label 값]
```

- 여러가지 Label을 가지는 오브젝트 조회 (AND)

```shell
kubectl get all -l [label 키]=[label 값],[label 키]=[label 값],[label 키]=[label 값].....

kubectl get all -l '[label 키] in ([label 값]),[label 키] in ([label 값]).....'
ex) kubectl get all -l 'env in (production),tier in (frontend).....'
```

-> Label의 구분을 콤마(,)로하며 AND(&&)역할을 한다. <br>

- 여러가지 Label을 가지는 오브젝트 조회 (OR)

```shell
ex) kubectl get all -l 'env in (production, qa).....'
```

-> Label값에 포함되는 여러가지 값을 넣어주어 OR을 구현한다.

- Label이 불일치하는 오브젝트 조회 (NOT)

```shell
ex) kubectl get all -l 'env,env notin (frontend).....'
```

-> Label에 notin 명령어를 사용한다.


<br><br>




------------------
**◎ 참고자료**
- Udemy - Certified Kubernetes Administrator (CKA) with Practice Tests

- [쿠버네티스 공식문서 - 레이블과 셀렉터](https://kubernetes.io/ko/docs/concepts/overview/working-with-objects/labels/)