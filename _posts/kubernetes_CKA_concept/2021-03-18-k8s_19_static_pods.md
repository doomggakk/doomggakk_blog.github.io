---
layout: single
title: "[CKA Concept] 19. Static Pods"
categories: [Kubernetes]
author_profile: true
excerpt: CKA 강의 개념 중 쿠버네티스에서의 Static Pods에 대해 정리한다. 
toc: true
toc_sticky: true
---

## Static Pods
- 이전에 공부했던 내용에서는 기본적으로 kubelet이 Pod을 생성하기 위해서 Master Node 요소들의(kube-api, kube-scheduler 등)작업이 진행된 뒤 kubelet에 전달되어져 Pod을 생성 하였다.
- 하지만 Master Node의 요소가 없다면, 그리고 Worker Node도 단하나밖에 존재하지 않는상황이라면 Pod을 어떻게 생성해야할까?
- 이런 상황에서 단 하나의 Node에 존재하는 Kubelet이 생성하는 Pod을 **Static Pod**라고 한다. 
- Kubelet은 이 Static Pod을 생성한 뿐만 아니라 감시하고 실패할 경우 다시 구동하기도 한다.

> **참고** : 만약 클러스터로 구성된 쿠버네티스를 구동하고 있고, Static Pod을 사용하여 모든 Node에 Pod를 구동하고 있다면 **Static Pod보다는 Daemon Set**을 사용하는 것이 바람직하다.

<br>

### Static Pod의 생성
-----------------------
- Static Pod을 생성하기 위해서는 생성할 Pod을 Yaml 파일로 정의한 뒤 **/etc/kubernetes/manifests** 디렉토리에 넣어주면 된다.
- Yaml 파일을 넣어주면 Kubelet은 이 파일을 통해서 생성 뿐만아니라 Pod의 상태를 체크하여 운영될 수 있도록 관리한다.
- 파일을 변경하면 Kubelet이 다시 Pod을 생성한다. 
- Yaml파일을 지우면 Pod 또한 제거되게 된다.


**cf) 이와 같은 방법으로 Pod은 생성할 수 있지만 Replica Set이나 Deployment, Service등의 기능은 불가능하다.**

<br>

### Static Pod의 Yaml파일을 저장하는 폴더를 만드는 방법
-------------------
1. kubelet.service에 직접정의

```yaml
 # 아래와 같이 추가해준다.
 ExecStart=/usr/local/bin/kubelet \\
   ...
   --pod-manifest-path=/etc/Kubernetes/manifests
   ...
```

2.  config파일생성 후 kubelet.service에 정의

```yaml
 # kubelet.service
 ExecStart=/usr/local/bin/kubelet \\
   ...
   --config=kubeconfig.yaml \\
   ...
```

```yaml
 # kubeconfig.yaml
 staticPodPath: /etc/Kubernetes/manifests
```

<br>

### Test하다가 모르는 것들
------------------
- Static Pod을 생성하는 yaml파일이 저장되어있는 디렉토리 찾기

1.config 파일의 위치를 알아내는 명령어

```
ps -ef | grep kubelet | grep config
```
![](/assets/img/kubernetes/19_staticpod_1.png)


2.config 파일에서 Static Pod을 생성하는 Yaml 파일이 저장되어있는 디렉토리를 찾아냄

```shell
cd [config 파일 위치하는 경로]
grep -i static [config 파일]  # -i 옵션은 대소문자 구문X
```
![](/assets/img/kubernetes/19_staticpod_2.png)

3.image가 주어지고 command가 주어진 Static Pod 생성하는법

```shell
kubectl run static-pod --image=busybox --command sleep 1000 --restart=Never --dry-run=client -o yaml > static-pod.yaml
```
```--dry-run=client``` : 실행되지는 않고 결과만 보여주는 명령어
```-o yaml``` : yaml 파일로 출력해줌
```--command sleep 1000``` : command를 sleep 1000으로 정해줌
```--restart=Never``` : 재시작하지 않도록 한다.

4.다른 node의 Static Pod 제거방법
- 해당Node에 IP로 접속한다.

```shell
ssh 172.17.0.19
```
- 위와 같은 방법으로 Static Pod Yaml파일 저장 디렉토리를 찾아서 Yaml파일을 제거해준다.


<br>
<br>

------------------
**◎ 참고자료**
- Udemy - Certified Kubernetes Administrator (CKA) with Practice Tests

- [쿠버네티스 공식문서 - 스태틱(static) 파드 생성하기](https://kubernetes.io/ko/docs/tasks/configure-pod-container/static-pod/)

- [#19 Static Pods - Life in Hong Kong](https://blog.naver.com/ijoos/222161460850)


