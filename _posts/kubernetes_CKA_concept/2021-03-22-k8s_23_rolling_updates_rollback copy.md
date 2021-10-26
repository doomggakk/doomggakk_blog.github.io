---
layout: single
title: "[CKA Concept] 23. Rolling Updates and Rollbacks"
categories: [Kubernetes]
author_profile: true
excerpt: CKA 강의 개념 중 쿠버네티스에서의 Rolling Updates와 Rollbacks에 대해 정리한다. 
toc: true
toc_sticky: true
---

## RollOut
- 기본적으로 배포하는 것을 의미한다.
- 두가지 방식이 존재한다.
1. Recreate : 2번째 Version 배포시 이전 버전을 모두 Down시키고나서 새로운 Version을 배포시킴.
    - 모두 Down 시킨 뒤 새로운 Version을 배포하는 텀에서 사용자가 접속하지 못하는 문제점이 발생할 수 있다.  
2. Rolling Update : 한개 Pod씩 순차적으로 배포를 한다.
    - Recreate의 문제점을 해소할 수 있음
    - Deployment의 **Default Strategy**다.

<br>

## Upgrades
1. 하나의 ReplicaSet(1)에서 Pod들이 잘 돌아가고있을 때, Deployment가 또 하나의 ReplicaSet(2)을 만든다.
2. Upgrade가 진행될 때 ReplicaSet(1)과 ReplicaSet(2)에서 동시에 한개의 Pod을 Down(ReplicaSet(1)),UP(ReplicaSet(2) 반복하여 Upgrades를 진행한다.

<br>

## Rollback
- 업그레이드한 Pod들이 문제 생겼을때 이전버전으로 원복시키는 것을 의미한다.
1. Rollback이 필요한 경우 명령어를 입력하여, 이전 버전의 ReplicaSet에 이전 버전의 Pod들이 생성된다.
2. 현재버전의 Pod은 순차적으로 Down된다.

<br>

## Rollout 관련 명령어 정리

- Create : ``` kubectl create -f deployment-definition.yml```
- Get : ``` kubectl get deployment```
- Update : ``` kubectl apply -f deployment-definition.yml```
            ``` kubectl set image deployment/[deployment 이름] nginx=nginx:1.9.1```
- Status : ``` kubectl rollout status deployment/[deployment 이름]```
            ```kubectl rollout history deployment/[deployment 이름]```
- Rollback : ``` kubectl rollout undo deployment/[deployment 이름]```





<br><br>

------------------
**◎ 참고자료**
- Udemy - Certified Kubernetes Administrator (CKA) with Practice Tests
- [#22 Rolling Strategy - Life in Hong Kong](https://blog.naver.com/ijoos/222163357918)
- [[Kubernetes] 4. Deployment - Doomggaakk의 블로그](https://blog.naver.com/ijoos/222163357918)

