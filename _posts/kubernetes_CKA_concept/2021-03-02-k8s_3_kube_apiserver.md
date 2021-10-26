---
layout: single
title: "[CKA Concept] 3. Kube Apiserver"
categories: [Kubernetes]
author_profile: true
excerpt: CKA 강의의 개념 중 쿠버네티스에서의 kube-apiserver에 대해 정리한다. 
toc: true
toc_sticky: true
---

## Kube Apiserver

- kubectl get 명령어 처리 과정
1. 사용자 인증(Authenticate)
2. 요청 유효성 검사
3. 데이터 검색

<br>

- API 직접 호출하여 Pod을 생성하는 처리 과정(node가 할당되지 않았을 경우)
1. 사용자 인증(Authenticate)
2. 요청 유효성 검사
3. 데이터 검색
4. ETCD 업데이트
5. Scheduler가 API-Server를 모니터링하다가 변경사항이 있음을 발견, node가 할당되지 않은 사항도 발견
6. API-Server가 ETCD 클러스터에 정보를 업데이트 한 뒤 적절한 Worker Node의 kubelet 에 정보를 전달함
7. kubelet이 Worker Node에 Pod을 생성하고 Container Runtime Engine에게 이미지 생성하도록 지시
8. 작업이 끝나면 kubelet이 API 서버에 다시 상태를 업데이트 시키고 API 서버가 ETCD에 다시 업데이트

<br>

**※ 다른 컴포넌트들은 kube-apiserver를 통해서 ETCD에 상태 업데이트를 시킴**

<br>

**※ ETCD 클러스터와 통신하는 유일한 컴포넌트는 kube-apiserver 이다.**

<br>

### kube-Apiserver 설치
------------------

```shell
wget https://storage.googleapis.com/kubernetes-release/release/v1.13.0/bin/linux/amd64/kube-apiserver
```


------------------
**◎ 참고자료**
- Udemy - Certified Kubernetes Administrator (CKA) with Practice Tests
