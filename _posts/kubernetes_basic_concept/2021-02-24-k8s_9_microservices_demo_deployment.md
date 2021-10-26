---
layout: single
title: "[Kubernetes] 7-2. MicroService demo(Deployment)"
categories: [Kubernetes]
author_profile: true
excerpt: Kubernetes에서 Deploymet를 이용한 MicroService Demo를 정리한다. 
toc: true
toc_sticky: true
---

## MicroService Demo using Deployment

- 이전 게시물에서는 Voting app을 예시로 들어 Microservice를 알아보고 Pod을 생성하여 실행시켜 보았다.
- 이번 게시물에서는 Deployment를 사용하여 Demo를 Voting app을 실행시켜 볼 것이다.

<br>
<br>

### Demo (Deployment)
---------------------
- 5개의 Deployment 생성 (이전에 만들었던 Pod YAML파일 참고)

1.Voting-app-deployment

```yml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: voting-app-deploy
  labels:
    name: voting-app-deploy
    app: demo-voting-app
spec:
  replicas: 1
  selector:
    matchLabels:          # Pod의 metadata의 label부분 복사하여 붙여넣기
      name: voting-app-pod
      app: demo-voting-app
  template:               # Pod의 metadata부분부터 복사하여 붙여넣기
    metadata:
      name: voting-app-pod
      labels:
        name: voting-app-pod
        app: demo-voting-app
    spec:
      containers:
        - name: voting-app
          image: kodekloud/examplevotingapp_vote:v1
          ports:
            - containerPort: 80
```
<br>

2.Result-app-deployment

```yml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: result-app-deploy
  labels:
    name: result-app-deploy
    app: demo-voting-app
spec:
  replicas: 1
  selector:
    matchLabels:
      name: result-app-pod
      app: demo-voting-app
  template:
    metadata:
      name: result-app-pod
      labels:
        name: result-app-pod
        app: demo-voting-app
    spec:
      containers:
        - name: result-app
          image: kodekloud/examplevotingapp_result:v1
          ports:
            - containerPort: 80
```
<br>

3.Redis-deployment

```yml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: redis-deploy
  labels:
    name: redis-deploy
    app: demo-voting-app
spec:
  replicas: 1
  selector:
    matchLabels:
      name: redis-pod
      app: demo-voting-app
  template:
    metadata:
      name: redis-pod
      labels:
        name: redis-pod
        app: demo-voting-app
    spec:
      containers:
        - name: redis
          image: redis
          ports:
            - containerPort: 6379
```
<br>

4.Postgres-deployment

```yml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: postgres-deploy
  labels:
    name: postgres-deploy
    app: demo-voting-app
spec:
  replicas: 1
  selector:
    matchLabels:
      name: postgres-pod
      app: demo-voting-app
  template:
    metadata:
      name: postgres-pod
      labels:
        name: postgres-pod
        app: demo-voting-app
    spec:
      containers:
        - name: postgres
          image: postgres
          ports:
            - containerPort: 5432
          env:
            - name: POSTGRES_USER
              value: "postgres"
            - name: POSTGRES_PASSWORD
              value: "postgres"
```

<br>

5.Worker-deployment

```yml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: worker-deploy
  labels:
    name: worker-deploy
    app: demo-voting-app
spec:
  replicas: 1
  selector:
    matchLabels:
      name: worker-app-pod
      app: demo-voting-app
  template:
    metadata:
      name: worker-app-pod
      labels:
        name: worker-app-pod
        app: demo-voting-app
    spec:
      containers:
        - name: worker-app
          image: kodekloud/examplevotingapp_worker:v1

```
<br>

6.실행화면

- deployment와 pod이 정상적으로 생성됨

![deployment App](/assets/img/kubernetes/7_microservice_7.png)

<br>

- 밑줄친 부분을 보면 Deployment로 생성된 Pod임을 알 수 있음.

![deployment App](/assets/img/kubernetes/7_microservice_8.png)

<br>

- 투표결과가 제대로 반영됨.

![deployment App](/assets/img/kubernetes/7_microservice_9.png)

<br>


------------------
**◎ 참고자료**

- Udemy - Kubernetes for the Absolute Beginners - Hands-on
- [쿠버네티스 공식문서 - 서비스](https://kubernetes.io/ko/docs/concepts/services-networking/service/)
- [쿠버네티스 공식문서 - 파드](https://kubernetes.io/ko/docs/concepts/workloads/pods/)