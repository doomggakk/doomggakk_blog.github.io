---
layout: single
title: "[CKA Concept] 49. Practice Test"
categories: [Kubernetes]
author_profile: true
excerpt: CKA 강의에서 Practice Test를 풀어보며 공부내용을 정리한다.
toc: true
toc_sticky: true
---

### Set image ( Pod의 이미지 변경 )

```bash
kubectl set image <object type> <object name> <container name>=<image name>

ex) kubectl set image pod my-app myapp-container=nginx

```

<br>

### pod생성시 command 추가

```bash
kubectl run nginx --image nginx --command -- sleep 1000

```

<br>

### Pod의 command와 Image의 command
- Docker파일의 ENTRYPOINT의 command, COMMAND의 command는 **pod 옵션 command가 있을 경우 대체된다.**
- Docker File의 ENTRYPOINT : container 생성시 실행되는 명령어 args로 준것과 중복하여 명령어가 실행된다.
- Docker File의 CMD:comtainer 생성시 실행되는 명렁어인 것은 같으나 args로 준 명령어로 대체하여 실행한다.



<br>

-----------------

### node 할당 ( Pod 스케줄링 )

```yaml

apiVersion: v1
kind: Pod
metadata:
  name: my-app
spec:
  nodeName: node01  # node 이름 지정
  containers:
  - name: myapp-container
    image: nginx
...

```

<br>

### label별로 Pod 조회하기

```bash

kubectl get pods --selector <labels>

ex) kubectl get pods --selector env=dev

```

<br>

-----------------

### Node에 Taint 지정 방법

```bash
kubectl taint node <node name> <key>=<value>:<effect>

ex) kubectl taint node node01 spray=mortein:NoSchedule

```

### Node에 Taint 제거 방법

```bash
# '-'를 끝에 붙여준다.
kubectl taint node <node name> <제거할 key>=<제거할 value>:<제거할effect>-

kubectl taint nodes master/controlplane node-role.kubernetes.io/master:NoSchedule-

```

<br>

### Pod에 Toleration 지정 방법

```yaml
---
apiVersion: v1
kind: Pod
metadata:
  name: bee
spec:
  containers:
  - image: nginx
    name: bee
  tolerations:
  - key: spray
    value: mortein
    effect: NoSchedule
    operator: Equal
```

<br>

### label 추가하기

```bash
kubectl label <type> <name> <label>

ex) kubectl label node node01 color=blue
```

<br>

-----------------

### node affinity 추가하기 ( 문서 참고 )
- Node affinity : 해당 label을 가진 node에 pod를 할당한다.
- PodAffinity : 해당 label을 가진 pod가 돌아가고 있는 node에 스케줄링한다.(사용자가 자격이 충족되는 토폴로지 도메인에 원하는 수의 파드를 얼마든지 채울 수 있다.)
**PodAntiAffinity** : 해당 label을 가진 pod가 돌아가고 있지 않는 node에 할당한다.(단일 토폴로지 도메인에 단 하나의 파드만 스케줄 될 수 있다.(각 노드에 하나씩 생기도록 할 때 사용함.))<br>
-> topologykey를 각 노드의 label과 매칭해보고 있는 node에 스케줄링한다. 근데 그 node에 원하는 label을 가진 pod가 이미 스케줄링되어있다면 중복하여 스케줄링되지 않는다.

```yaml
# node affinity
...
spec:
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: kubernetes.io/e2e-az-name
            operator: In
            values:
            - e2e-az1
            - e2e-az2

# pod affinity
...
    spec:
      affinity:
        podAntiAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
          - labelSelector:
              matchExpressions:
              - key: app
                operator: In
                values:
                - web-store
            topologyKey: "kubernetes.io/hostname"
# topologyKey(node label의 key만 비교해봄)를 label로 가진 노드 중에서
# label selector에 해당하는 label을 가진 pod가 스케줄링되어있지 않은 노드에 스케줄링한다.
...

```

<br>

-----------------

### CrashLoopBackOff
- Pod의 메모리가 부족한 경우 출력되는 상태이다.<br>
ex) 프로세스를 돌리는데 15MB가 필요한데 Pod의 메모리 제한이 10MB인 경우

<br>

-----------------

### Static Pod 생성
- /etc/kubernetes/manifests/ 에 yaml 파일생성
- 혹시나 위의 경로에서 Static Pod이 잘 동작 하지 않는경우, /var/lib/kubelet/config.yaml에서 StaticPodPath를 확인해보도록 한다.

```bash

kubectl run pod --image nginx (--restart Never) --dry-run=client -o yaml --command -- sleep 1000 > /etc/kubernetes/manifests/pod.yaml


```

<br>

-----------------

### 다중 스케줄러
- 스케줄러를 여러개 사용할 경우는 /etc/kubernetes/manifests/의 kube-scheduler.yaml파일을 복사하여 사용한다.
- kube-scheduler.yaml파일의 일부를 수정하여야한다.

```yaml
...
metadata:
  creationTimestamp: null
  labels:
    component: my-scheduler           # 변경
    tier: control-plane
  name: my-scheduler                  # name 변경
  namespace: kube-system
spec:
  containers:
  - command:
    - kube-scheduler
    - --authentication-kubeconfig=/etc/kubernetes/scheduler.conf
    - --authorization-kubeconfig=/etc/kubernetes/scheduler.conf
    - --bind-address=127.0.0.1
    - --kubeconfig=/etc/kubernetes/scheduler.conf
    - --leader-elect=false            # true -> false로 변경
    - --scheduler-name=my-scheduler   # 이름 변경
    - --port=10258                    # 포트 변경
    - --secure-port=0
    ...
    ...
        livenessProbe:
      failureThreshold: 8
      httpGet:
        host: 127.0.0.1
        path: /healthz
        port: 10258                   # 포트 위에맞춰 변경
        scheme: HTTPS
      initialDelaySeconds: 10
      periodSeconds: 10
      timeoutSeconds: 15
    name: kube-scheduler
    resources:
      requests:
        cpu: 100m
    startupProbe:
      failureThreshold: 24
      httpGet:
        host: 127.0.0.1
        path: /healthz
        port: 10258                   # 포트 위에맞춰 변경
        scheme: HTTPS
      initialDelaySeconds: 10
    ...


```

<br>

### Pod에 스케줄러 지정
- ```schedulerName```을 사용한다.

```yaml
apiVersion: v1 
kind: Pod 
metadata:
  name: nginx 
spec:
  schedulerName: my-scheduler  # scheduler 지정
  containers:
  - image: nginx
    name: nginx
```

<br>

-----------------

### object들의 cpu, mem 사용량 확인하기

```bash
kubectl top <type>
ex) kubectl top node
```

<br>

-------------------

### Secret 생성하기

```bash
kubectl create secret generic db-secret \
--from-literal=DB_Host=sql01 \
--from-literal=DB_Password=password123 \
--from-literal=DB_User=root

```

### Secret 환경변수에 적용하기

```bash
apiVersion: v1
kind: Pod
metadata:
  name: secret-test-pod
spec:
  containers:
    - name: test-container
      image: k8s.gcr.io/busybox
      command: [ "/bin/sh", "-c", "env" ]
      envFrom:
      - secretRef:
          name: mysecret
  restartPolicy: Never

```

<br>

---------------------

### Node Drain( 노드 드레인 )
- 스케줄링 불가능으로 표시하고 워크로드를 축출하여 유지 보수할 노드를 준비한다.
- **Node가 drain되면 해당 노드에 있던 Pod들은 축출된다.**
```bash
kubectl drain <node> --ignore-daemonsets

ex) kubectl drain node01 --ignore-daemonsets
```

### Node Drain 해체 (cordon 해제)

```bash
kubectl uncordon <node>

ex) kubectl uncordon node01
```

### Node cordon 
- Node를 cordon하면 기존의 Pod에는 영향을 미치지 않고, 새로 스케줄링되는 Pod들은 스케줄링 되지않도록 한다.

```bash
kubectl cordon <node>

ex) kubectl cordon node01
```

<br>

--------

### cluster upgrade - 최신 버전 확인

```bash
kubeadm upgrade plan
```

<br>

### node upgrade
-  한번에 하나의 node를 업그레이드 하는 것이 안전하다.
1. 한개의 node drain 시킨다. (unschedulable)
2. kubeadm 최신버전 설치
3. kubeadm 최신버전 업그레이드
4. kubelet 최신버전 설치
5. kubelet restart

``` bash
#1
kubectl drain controlplane

#2
apt-get update

apt-get install kubeadm=1.20.0-00

#3
kubeadm upgrade apply v1.20.0 # controlplane 노드
kubeadm upgrade node          # worker 노드

#4
apt-get install kubelet=1.20.0-00

#5
systemctl restart kubelet

```

<br>

----------

### ETCD 백업
1. ETCD의 cert, key, ca-file 을 알아낸다.
2. 백업 명령어에 3가지 value를 넣고, 생성 path를 입력한다.

```bash
#1
...
    - --cert-file=/etc/kubernetes/pki/etcd/server.crt
    - --key-file=/etc/kubernetes/pki/etcd/server.key
    - --trusted-ca-file=/etc/kubernetes/pki/etcd/ca.crt
...

#2
ETCDCTL_API=3 etcdctl --endpoints=https://127.0.0.1:2379 \
  --cacert=<trusted-ca-file> --cert=<cert-file> --key=<key-file> \
  snapshot save <backup-file-location>

```




<br>
<br>






------------------
**◎ 참고자료**
- Udemy - Certified Kubernetes Administrator (CKA) with Practice Tests
- [쿠버네티스 공식문서 - DNS 서비스 사용자 정의하기](https://kubernetes.io/ko/docs/tasks/administer-cluster/dns-custom-nameservers/)
