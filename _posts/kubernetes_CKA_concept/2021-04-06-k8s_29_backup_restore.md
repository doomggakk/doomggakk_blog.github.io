---
layout: single
title: "[CKA Concept] 29. Backup & Restore Methods"
categories: [Kubernetes]
author_profile: true
excerpt: CKA 강의의 개념 중 Backup 과 Restore Methods에 대해 정리한다. 
toc: true
toc_sticky: true
---

## Backup & Restore Methods
- 백업을 해야할 것들
1. Resource Configuration( secret, configmap, yaml 파일 등)
2. ETCD
3. Persistent Volumes

<br>

### Resource Configuration 백업
- github에 코드 저장
- kube-apiserver 에 들어있는 모든 것을 가져와 저장해두는 것

```shell
kubectl get all --all-namespaces -o yaml > all-deploy-services.yaml
```
-> 위와 같은 서비스를 직접 만들필요없이 VELERO(ARK by HeptIO)같은 곳에서 제공하는 것을 사용하면 편하다.
  
<br>

### ETCD 백업
- ETCD는 클러스터의 Status를 저장하고 있는 데이터 저장소이다.
- etcd.service의 --data-dir=<디렉토리>에 ETCD 스토리지가 존재한다.
- Snapshot을 찍어 그 파일을 저장해놓아 백업한다.

```shell
ETCDCTL_API=3 etcdctl snapshot save snapshot.db # snapshot이 현재 디렉토리에 저장됨
ETCDCTL_API=3 etcdctl snapshot status snapshot.db # 상태가 snapshot으로 저장됨

service kube-apiserver stop # kube-apiserver를 잠시 내린다.

ETCDCTL_API=3 etcdctl snapshot restore snapshot.db \
--data-dir /var/lib/etcd-from-backup # ETCD를 복원하고 데이터 저장 디렉토리를 변경한다.

systemctl daemon-reload
service etcd restart
service kube-apiserver start

```

<br>

### etcdctl
- ETCD를 위한 커맨드 라인
- etcdctl을 사용하여 backup이나 restore 작업을 하기 위해서는 ETCDCTL_API=3으로 설정을 해주어야 한다. (ETCDCTL API 버전3을 사용한다는 의미)
```shell
export ETCDCTL_API=3 # Master Node에서 지정해준다.
```
- ```etcdctl snapshot save ```를 사용하려면 ```--cacert```, ```--cert```, ```--endpoints=[127.0.0.1:2379]```, ```--key``` 옵션이 필수로 포함되어야 한다.


<br>

### ETCD BackUP & Restore DEMO
#### ETCD 백업

- cert, cacert, key에 해당하는 정보를 기입해주어야한다.
- ```kubectl describe pod <etcd POD>```을 통해 정보를 얻을 수 있다.

```shell
ETCDCTL_API=3 etcdctl snapshot save --cert=<ETCD의 cert-file> --cacert=<ETCD의 ca-file>  --key=<ETCD의 key-file> <저장위치>

-------------------------------------------------------------------

ETCDCTL_API=3 etcdctl snapshot save --cert=/etc/kubernetes/pki/etcd/server.crt --cacert=/etc/kubernetes/pki/etcd/ca.crt --key=/etc/kubernetes/pki/etcd/server.key /opt/snapshot-pre-boot.db

```
![](/assets/img/kubernetes/29_backup&restore_1.png)

<br>

- 백업파일이 성공적으로 저장되면 아래와같이 출력된다.

![](/assets/img/kubernetes/29_backup&restore_2.png)

<br>

#### ETCD 복원
- kubernetes의 모든 요소들이 사라진 상태이다. (Pod 뿐만아니라 Service, Deployment등)

데이터 손실 전

![](/assets/img/kubernetes/29_backup&restore_3.png)

데이터 손실 후

![](/assets/img/kubernetes/29_backup&restore_4.png)

<br>

- restore 명령어로 복원한다.

```shell
ETCDCTL_API=3 etcdctl snapshot restore <백업해놓은 파일경로> --data-dir=<복원할 경로>

------------------------------------------------------------------------

ETCDCTL_API=3 etcdctl snapshot restore /opt/snapshot-pre-boot.db --data-dir=/var/lib/etcd-backup

```

<br>

- 같은 내용의 데이터가 복원된 것을 확인할 수 있다.

![](/assets/img/kubernetes/29_backup&restore_5.png)

<br>

- 복원된 데이터를 바라보도록 etcd 설정을 수정해야한다.
- ```vi /etc/kubernetes/etcd.yaml```을 입력하여 static pod으로 구성되어있는 etcd YAML 파일을 수정한다.

![](/assets/img/kubernetes/29_backup&restore_6.png)
![](/assets/img/kubernetes/29_backup&restore_7.png)

- 기존의 etcd가 사라지고 새로운 etcd가 생성된다.
- 새롭게 구성요소들이 생성되었다.

![](/assets/img/kubernetes/29_backup&restore_8.png)
![](/assets/img/kubernetes/29_backup&restore_9.png)

<br>

### Demo를 통해 알게된 것
- etcd의 버전 확인 : etcd Pod 상세정보의 image 버전을 확인한다.
- 마스터 노드로부터 ETCD 클러스터에 접근할 수 있는 주소 : **--listen-client-urls** PATH를 확인한다.
-  ETCD CA Certificate file 디렉토리 : **--trusted-ca-file** PATH를 확인한다.

<br>
<br>


------------------
**◎ 참고자료**

- Udemy - Certified Kubernetes Administrator (CKA) with Practice Tests
- [Operating etcd clusters for Kubernetes](https://kubernetes.io/docs/tasks/administer-cluster/configure-upgrade-etcd/)
- [#27 Backup & restore - 작성자 ijoos](https://blog.naver.com/ijoos/222168965561)