---
layout: single
title: "[Kubernetes] 4. Deployment"
categories: [Kubernetes]
author_profile: true
excerpt: Kubernetes의 구성요소인 Deployment에 대해 정리한다
toc: true
toc_sticky: true
---




## Deployment

### 유스케이스
- ReplicaSet 롤아웃
- 파드의 새로운 상태 선언
- 디플로이먼트의 이전 버전으로 롤백
- 스케일 업
- 일시중지
- 이전 ReplicaSet 정리 등
<br>

## Deployment 생성
```yaml
apiVersion: apps/v1
kind: Deployment # 이부분만 ReplicaSet -> Deployment 수정
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
    selector: 
        matchLabels:
            app: myapp
```
ReplicaSet의 yaml 파일정의에서 kind만 수정해주면된다.

<br><br>

## RollOut & Versioning
- RollBack ? : 현재 버전에 문제가 발생하여 이전 버전으로 되돌리는 것
- RollOut ? : Release 하는 것을 의미
- RollOut 명령어

```shell
kubectl rollout status deployment/myapp-deployment # rollout 상태 출력
```

```shell
kubectl rollout history deployment/myapp-deployment # rollout 히스토리 출력
```

<br><br>

## Deployment Strategy
1. Recreate : 여러개의 replicas를 업데이트 할 때 이전 것들을 모두 내린 뒤 새로운 것을 띄우는 방식
-> 모든 것을 내린 뒤 새로운 것을 올리는 텀에 사용자가 접속하지 못할 수 있는 안좋은 상황이 발생 할 수 있음
-> 따라서 이 방식은 Deployment의 Default Strategy가 아님

2. Rolling Update : Replica를 하나씩 처리. 한개씩 내리고 올리기를 반복하여 사용자가 접근할 수 없는 문제점을 해소함 
-> Deployment의 Default Strategy임

<br><br>

## RollBack과 RollOut 구동화면
- Rollout과 RollBack을 순서대로 구동시켜보도록 하겠다.

<br>

1.만들어놓은 YAML 파일로 Deployment 생성(alias k='kubectl') 
![](/assets/img/kubernetes/4_deployment_rollout_1.png)

<br>

2.Pod 정상적으로 생성, Image = nginx 인 것 확인
![](/assets/img/kubernetes/4_deployment_rollout_2.png)
![](/assets/img/kubernetes/4_deployment_rollout_3.png)

<br>

3.아래 명령어를 통해서 rollout 히스토리 확인

```shell
kubectl rollout history deployment [deployment 이름]
```

- deployment를 생성한 것이 1번에 들어있는 것을 확인 할 수 있다.
![](/assets/img/kubernetes/4_deployment_rollout_4.png)

<br>

4.image 버전 변경

```shell
kubectl set image deployment [deployment 이름] [변경전이미지]=[변경후이미지] --record
```
- --record를 추가하여 rollout history에 기록되도록 설정 
![](/assets/img/kubernetes/4_deployment_rollout_5.png)
- image = nginx:1.18로 변경된 것 확인
![](/assets/img/kubernetes/4_deployment_rollout_7.png)
- rollout history에서 2번째 rollout이 추가된 것 확인
![](/assets/img/kubernetes/4_deployment_rollout_8.png)

<br>

5.Rollback 실행

```shell
kubectl  rollout undo deployment [deployment 이름] --record
```

- Rollback 명령어 입력
![](/assets/img/kubernetes/4_deployment_rollout_9.png)
- Image가 다시 nginx:1.18 -> nginx로 수정된 것 확인
![](/assets/img/kubernetes/4_deployment_rollout_10.png)
- rollout history에서 1번 rollout이 최근 rollout으로 옮겨진것 확인
![](/assets/img/kubernetes/4_deployment_rollout_11.png)

<br>

------------------
**◎ 참고자료**


- Udemy - Kubernetes for the Absolute Beginners - Hands-on
- [쿠버네티스 공식문서 - Deployment](https://kubernetes.io/ko/docs/concepts/workloads/controllers/deployment/)