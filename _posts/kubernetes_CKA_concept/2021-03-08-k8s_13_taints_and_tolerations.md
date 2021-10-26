---
layout: single
title: "[CKA Concept] 13. Taints and Tolerations"
categories: [Kubernetes]
author_profile: true
excerpt: CKA 강의 개념 중 쿠버네티스에서의 Taints과 Tolerations에 대해 정리한다. 
toc: true
toc_sticky: true
---

## 쿠버네티스에서의 Taints and Tolerations
- Taint는 번역하면 오염을 뜻하고, toleration은 용인을 뜻한다.
- Taint : 노드마다 설정이 가능하다. 설정된 taint에 따라서 Pod이 스케줄되지 않는다.
- Toleration: Pod에 설정한다. taint를 무시하고 Pod이 스케줄되도록 한다.
- **즉 Node에 Taint라는 자물쇠를 걸어 Toleration이라는 열쇠를 가진 Pod만 생성되도록 한다.**

<br>

### Taints의 3가지 옵션
-------------------------
- ```NoSchedule``` : toleration이 없으면 pod이 스케줄되지 않음, 기존 실행되던 Pod에는 적용안됨
- ```PreferNoSchedule```: toleration이 없으면 pod을 스케줄링안하려고 하지만 필수는 아님, 클러스터내에 자원이 부족하거나 하면 taint가 걸려있는 노드에서도 Pod이 스케줄링 될 수 있음
- ```NoExecute``` : toleration이 없으면 Pod이 스케줄되지 않으면 기존에 실행되던 Pod도 toleration이 없으면 종료시킴.

<br>

### Taints 명령어
----------------------
- Taint 추가
```shell
kubectl taint nodes node1 key1=value1:NoSchedule
```

- Taint 제거
```shell
kubectl taint nodes node1 key1=value1:NoSchedule-
```

<br>

### Toleration 사용 명령어
----------------------
- Pod **spec**에 Toleration 추가

```yaml
...
tolerations:
  - key: "key1"
    operator: "Equal"
    value: "value1"
    effect: "NoSchedule"

또는

...
tolerations:
  - key: "key1"
    operator: "Exists"
    effect: "NoSchedule"

``` 

-  operator을 지정하지 않으면 operator의 기본값은 Equal이다.
- Toleration은 Key가 동일하고 Effect가 동일한 경우, 테인트와 "일치"한다. 그리고 다음의 경우에도 마찬가지다.

    - operator 가 Exists 인 경우(이 경우 value 를 지정하지 않아야 함)
    - operator 는 Equal 이고 value 는 value 로 같다.

> 참고:
  두가지 특별한 경우
  - operator: Exists 가 있는 비어있는 key 는 모든 키, 값 및 이펙트와 일치하므로 모든 것이 Toleration 된다.
  - 비어있는 effect 는 모든 이펙트를 키 key1 와 일치시킨다.

<br>

### Node Taints 예시
----------------------
1. 아래와 같이 taint를 설정했다고 가정한다.

```shell
kubectl taint nodes node1 key1=value1:NoSchedule

kubectl taint nodes node1 key1=value1:NoExecute

kubectl taint nodes node1 key2=value2:NoSchedule
```
 
2. Pod에는 2가지 Toleration이 있다.

```yaml
tolerations:
  - key: "key1"
    operator: "Equal"
    value: "value1"
    effect: "NoSchedule"
  - key: "key1"
    operator: "Equal"
    value: "value1"
    effect: "NoExecute"
```

3. 위와 같은 경우 taint중 3번째와 일치하는 toleration이 없기 때문에 Pod는 Node에 스케줄 될 수 없다.

<br>

4. 일반적으로 NoExecute Effect가 있는 테인트가 Node에 추가되면, Taint를 용인(Toleration)하지 않은 Pod는 즉시 축출되고 그 반대는 축출되지 않는다. 

<br>

5. 그러나 NoExcute Effect가 있는 Toleration은 Taint가 추가 된 후 Pod가 Node에 바인딩된 시간을 지정하는 선택적 tolerationSeconds 필드를 지정할 수 있다.

```yaml
tolerations:
  - key: "key1"
    operator: "Equal"
    value: "value1"
    effect: "NoExecute"
    tolerationSeconds: 3600
```

-> 이것은 이 Pod가 실행중이고 일치하는 Taint가 노드에 추가되면, 3600초 동안 Node에 바인딩 된 후 , Pod가 축출 된다는 것을 의미한다. 그 전에 Taint를 제거하면 파드가 축출되지 않는다.

<br>
<br>

------------------
**◎ 참고자료**
- Udemy - Certified Kubernetes Administrator (CKA) with Practice Tests

- [쿠버네티스 공식문서 - 테인트(Taints)와 톨러레이션(Tolerations)](https://kubernetes.io/ko/docs/concepts/scheduling-eviction/taint-and-toleration/)

- [Kubernetes taint & toleration - 호롤리한 하루](https://gruuuuu.github.io/cloud/k8s-taint-toleration/#)


