---
layout: single
title: "[CKA Concept] 17. Resource Requirements and Limits"
categories: [Kubernetes]
author_profile: true
excerpt: CKA 강의 개념 중 쿠버네티스에서의 Resource Requirements and Limits에 대해 정리한다. 
toc: true
toc_sticky: true
---

## Resource Requirements and Limits
- Kubernetes에서는 각 노드가 CPU, Memory, Disk를 가지고 있으며 **Scheduler**가 Pod이요구하는 스펙에 따라 Node를 배정한다.
- 만약 Pod이 새로 배정되어야한다면 Node가 가지고있는 자원마다 할당가능한 여유공간에 따라서 배정하거나 배정하지 않는다.
- 어느 Node에도 배정 될 수 없는 상황이라면 Scheduler는 스케줄링을 하지않고 그 Pod을 Pending 상태로 보류시킨다.

<br>

### Pod의 필요한 자원 정의
- Kubernetes에서는 기본적으로 0.5CPU / 256M Memory를 요구한다.
- 기본값보다 더 많은 자원이 요구될경우에는 해당 Pod에 정의해주면 된다.

```yaml
apiVersion:
kind: Pod
metadata:
  name:
  labels:
    name: 
spec:
  containers:
  - name: nginx
    image: nginx
    ports:
      - containerPort: 8080
    resources:              # <─┐
      requests:             #   │
        memory: "1Gi"       #   │
        cpu: 1              # <─┘  
```
- ```cpu: 1```의 의미 : 1 AWS cCPU / 1 GCP Core, / 1 Azure Core을 의미
- 메모리의 단위 사용 : 

|단위|실제크기|
|-|-|
|1G(Gigabyte)|1,000,000,000 bytes|
|1M(Megabyte)|1,000,000 bytes**|**
|1K(Kilobyte)|1,000 bytes|

|단위|실제크기|
|-|-|
|1Gi(Gibibyte)|1,073,741,824 bytes|
|1Mi(Mebibyte)|1,048,576 bytes|
|1Ki(kibibyte)|1,024 bytes|

<br>

### Limit
-------------------
- 자원은 Limit을 가지고있으며 기본값은 1 vCPU, 512 Mi이다.
- 이를 변경하기 위해서는 Pod에 Limit를 새롭게 설정할 수 있다.
- 만약 이값보다 Limit을 벗어나게 되면 CPU는 더이상 사용할 수 없게 되지만, Memory는 이 Limit이 적용되지 않는다.

```yaml
...
spec:
  containers:
  - name: nginx
    image: nginx
    ports:
      - containerPort: 8080
    resources:              
      requests:             
        memory: "1Gi"       
        cpu: 1                
      limits:               # <─┐
        memory: "2Gi"       #   │
        cpu: 2              # <─┘
```
<br>

### Test하면서 모르는 개념
----------------
- OOMKilled는 메모리가 부족해서 생기는 상태오류이다.
- Pod의 설정을 수정할경우

```shell
kubectl get pod [Pod 이름] -o yaml > [파일이름.yaml]
```
명령어를 실행하면 yaml파일이 생성된다.
-> 기존의 Pod을 삭제하고 수정한 yaml파일로 Pod을 다시 생성해준다.


<br>
<br>

------------------
**◎ 참고자료**
- Udemy - Certified Kubernetes Administrator (CKA) with Practice Tests

- [쿠버네티스 공식문서 - 컨테이너 리소스 관리](https://kubernetes.io/ko/docs/concepts/configuration/manage-resources-containers/)

- [#17 Resource Requirement & Limit - Life in Hong Kong](https://blog.naver.com/ijoos/222160395338)





