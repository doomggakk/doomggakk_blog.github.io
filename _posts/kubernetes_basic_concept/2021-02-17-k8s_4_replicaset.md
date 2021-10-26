---
layout: single
title: "[Kubernetes] 3. ReplicaSet"
categories: [Kubernetes]
author_profile: true
excerpt: Kubernetes의 구성요소인 Replicaset에 대해 정리한다.
toc: true
toc_sticky: true
---



## Replication Controllers and ReplicaSets

- k8s에서의 controller는 Brain.
- **Replication Controller**와 **ReplilcaSets**은 같은 기능을 하지만 **ReplicaSets**이 **Replication Controller**를 대체하는 기술로 자리 잡음
- replica Sets이 필요한 이유
    1. High Availability : 정해놓은 pod 개수대로 반드시 실행되고 있도록 유지해줌.<br>
    -pod의 개수를 1개로 설정해놓았다면 1개의 pod이 실행되다가 오류가 났을경우 자동으로 새로운 pod으로 대체<br>
    2. Load Balancing & Scaling : 사용자가 늘어나면 그에 맞춰 pod을 늘려주거나 첫번째 노드의 리소스가 부족하면 다른 node에 pod을 늘려주는 등의 기능을 함

<br><br>

### ReplicaSet YAML 작성

```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
    name: myapp-replicaset
    labels:
        app: myapp
spec:
    template:
        metadata:
            name: myapp-pod
            labels:
                app: myapp
            spec:
                containers:
                    - name: nginx-container
                      image: nginx
    replicas: 3
    selector: #replica controller에서는 필요없음 
        matchLabels:
            app: myapp
```

- Label과 Selector
    - Replicaset이 자기에게 할당된 Pod를 모니터링하고 관리함
    - 수십수백개의 Pod들 중에 자기가 관리해야할 **Pod의 Label**을 **Replicaset의 selector**에 매칭시켜 구분함
    
    <br><br>

### replicaset 구동화면( alias k='kubectl' 적용 )
 1. yml 파일 적용<br>
![](/assets/img/kubernetes/3_replicaset_1.png)
    <br>
2. 1개의 Pod 삭제 후 상태 확인<br>
![](/assets/img/kubernetes/3_replicaset_2.png)<br>
해당 Pod은 삭제됐지만 새로운 Pod이 생성되어 3개의 Pod을 유지
    <br>
3. 3개의 Pod이 있는 상태에서 같은 Label인 Pod을 하나 더 생성<br>
![](/assets/img/kubernetes/3_replicaset_3.png)<br>
3개의 Pod을 유지하기 위해 새로 생성된 Pod을 자동으로 없앰
### replicas = 3일 때 같은 Label인 Pod 을 추가하는 경우

```yml
apiVersion: v1
kind: Pod
metadata:
    name: nginx-2
    labels:
        app: myapp
spec:
    containers:
        - name: nginx
          image: nginx
```
-> 위와 같이 yml을 작성하여 Pod을 추가

<br>

4. Replicaset의 Scale 수정<br>
![](/assets/img/kubernetes/3_replicaset_4.png)
    
<br><br>

### Scale
- **ReplicaSet의 Scale을 수정하는법**
    <br>
    1. yml 파일 수정 후 적용하는 명령어

    ```shell
    kubectl replace -f filename.yml # yml파일내에서 replicas: 3 -> 6으로 수정 후 아래 명령어 실행
    ```

    2. yml 파일 replicas를 수정하지않고 명령어로 바로 적용

    ```shell
    kubectl scale --replicas=6 -f filename.yml
    ```

    3. --replicas=6 뒤에 Type, Name 순으로 입력하여 특정 이름을 가진 replicaset의 Scale을 수정

    ```shell
    kubectl scale --replicas=6 replicaset myapp-replicaset 
    ```
    
<br><br>




------------------
**◎ 참고자료**


- Udemy - Kubernetes for the Absolute Beginners - Hands-on
- [쿠버네티스 공식문서 - ReplicaSet](https://kubernetes.io/ko/docs/concepts/workloads/controllers/replicaset/)

