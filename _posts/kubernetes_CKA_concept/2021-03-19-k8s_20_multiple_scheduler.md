---
layout: single
title: "[CKA Concept] 20. Multiple Scheduler"
categories: [Kubernetes]
author_profile: true
excerpt: CKA 강의 개념 중 쿠버네티스에서의 Multiple Scheduler에 대해 정리한다. 
toc: true
toc_sticky: true
---

## Multiple Scheduler
- 기존에 배웠던 내용에서는 기본 kube-scheduler가 worker Node에 Pod을 배정하여 Pod이 생산되었다.
- 하지만 기본 스케줄러가 요구에 맞지 않는다면 직접 커스텀 스케줄러를 만들어 사용할 수 있다.
- 커스텀 스케줄러는 기본 스케줄러와 함께 동작하며 Pod에 스케줄러를 정할 수 있다.

<br>

### Custom Scheduler 생성
---------------------
- /etc/kubernetes/manifests/kube-scheduler.yaml

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: kube-scheduler
  namespace: kube-system
spec:
  containers:
  - command:
    - kube-scheduler
    - --address=127.0.0.1
    - --kubeconfig=/etc/kubernetes/scheduler.conf
    - --leader-elect=true
    image: k8s.gcr.io/kube-scheduler-amd64:v1.11.3
    name: kube-scheduler
```

- my-custom-scheduler.yaml

```yaml
apiVersion: v1
kind: Pod
metadata:
  name:  my-custom-scheduler
  namespace: kube-system
spec:
  containers:
  - command:
    - kube-scheduler
    - --address=127.0.0.1
    - --kubeconfig=/etc/kubernetes/scheduler.conf
    - --leader-elect=true               #<-- 여러개의 scheduler를 사용하려는 경우 false로 수정
    - --scheduler-name=my-custom-scheduler      #<--이름변경

    image: k8s.gcr.io/kube-scheduler-amd64:v1.11.3
    name: kube-scheduler
```

- custom Scheduler를 생성한 후 Pod Spec에 Scheduler를 추가해준다.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
  - name: nginx
    image: nginx
  schedulerName: my-custom-scheduler  #<--Scheduler 추가
```

- scheduler가 정상적으로 운영되어 Pod을 돌게 했는지 확인

```shell
kubectl get events
또는
kubectl logs  my-custom-scheduler -n kube-system
```

<br>

### Test 하면서 알게된 것
-------------------
- kube-scheduler를 여러개 생성해서 사용할 수 있다.
- custom scheduler를 정의할때 -command에 --leader-elect=false로 설정해주어야 여러개의 scheduler 작동이 가능하다
- -command에 --scheduler-name=을 추가하여 이름을 정해준다.
- Pod을 정의할때 원하는 scheduler를 정해주기 위해서는 spec에 schedulerName을 추가해준다.



<br>
<br>

------------------
**◎ 참고자료**
- Udemy - Certified Kubernetes Administrator (CKA) with Practice Tests

- [#20 Multiple Schedulers - Life in Hong Kong](https://blog.naver.com/ijoos/222163273532)


