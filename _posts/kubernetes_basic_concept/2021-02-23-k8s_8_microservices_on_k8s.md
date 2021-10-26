---
layout: single
title: "[Kubernetes] 7-1. MicroService on k8s"
categories: [Kubernetes]
author_profile: true
excerpt: Kubernetes에서의 MicroService에 대해 정리한다.
toc: true
toc_sticky: true
---

## MicroService on Kubernetes
- Voting Application을 예시로 쿠버네티스에서의 microservice를 설명한다.
- Goal : 
1. 컨테이너 생성
2. Pod간의 연결
3. 외부 접속
<br>

### Voting App의 구조
---------------------

 <br>

![VotingApp의 구조](/assets/img/kubernetes/7_microservice_1.png)

<br>

- 구현순서:

1.  각 Pod을 생성
2. Service(ClusterIP)를 생성
    - redis
    - DB
3. Service(NodePort)를 생성
    - Voting App
    - Result App

-> **Worker App이 Service가 필요하지 않은이유** : 외부에 노출되거나 다른 App들의 접근이 있어야할 경우에만 Service가 필요하기 때문

<br>

### Demo
----------------
- 순서 : 
    1. 5개의 Pod 생성
    2. 4개의Serice 생성 (Worker app은 Service가 필요하지 않음)
    3. 실행확인
    4. 5개의 Deployment 생성
    5. 실행확인

<br>

**1. 5개의 Pod 생성**

1. Voting-app-pod

    ``` yml
    apiVersion: v1
    kind: Pod
    metadata:
      name: voting-app-pod
      labels:
        name: voting-app-pod
        app: demo-voting-app
    spec:
      containers:
        - name: voting-app
          image: kodekloud/examplevotingapp_vote:v1 # voting app sample image
          ports:
            - containerPort: 80
    ```

2. Result-app-pod
    ``` yml
    apiVersion: v1
    kind: Pod
    metadata:
      name: result-app-pod
      labels:
        name: result-app-pod
        app: demo-voting-app
    spec:
      containers:
        - name: result-app
          image: kodekloud/examplevotingapp_result:v1 # result app sample image
          ports:
            - containerPort: 80
    ```

3. Redis-pod

    ``` yml
    apiVersion: v1
    kind: Pod
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
            - containerPort: 6379 # redis 기본 port
    ```

4. Postgres-pod

    ``` yml
    apiVersion: v1
    kind: Pod
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
            - containerPort: 5432 # postgres 기본 port
          env:
            - name: POSTGRES_USER
              value: 'postgres'
            - name: POSTGRES_PASSWORD
              value: 'postgres'
    ```

5. Worker-app-pod

    ``` yml
    apiVersion: v1
    kind: Pod
    metadata:
      name: worker-app-pod
      labels:
        name: worker-app-pod
        app: demo-voting-app
    spec:
      containers:
        - name: worker-app
          image: kodekloud/examplevotingapp_worker:v1 # worker app sample image
    ```

6. Pod 생성 결과

    ![Pod 생성결과](/assets/img/kubernetes/7_microservice_2.png)

    <br>
    <br>

**2. 4개의 Service 생성**

1. Voting-app-service

    ``` yml
    apiVersion: v1
    kind: Service
    metadata:
      name: voting-service
      labels:
        name: voting-service
        app: demo-voting-app
    spec:
      type: NodePort        # voting app은 외부접속이 필요
      selector:
        name: voting-app-pod
        app: demo-voting-app
      ports:
        - port: 80
          targePort: 80
          nodePort: 30004
    ```

2. Result-app-service

    ``` yml
    apiVersion: v1
    kind: Service
    metadata:
      name: result-service
      labels:
        name: result-service
        app: demo-voting-app
    spec:
      type: NodePort        # result app은 외부접속이 필요
      selector:
        name: result-app-pod
        app: demo-voting-app
      ports:
        - port: 80
          targePort: 80
          nodePort: 30005
    ```

3. Redis-service

    ``` yml
    apiVersion: v1
    kind: Service
    metadata:
      name: redis
      labels:
        name: redis-service
        app: demo-voting-app
    spec:
      selector:
        name: redis-pod
        app: demo-voting-app # 지정할 Pod의 metdata > Labels을 그대로 집어넣어줌
      ports:
        - port: 6379
          targePort: 6379
    ```

4. Postgres-service

    ``` yml
    apiVersion: v1
    kind: Service
    metadata:
      name: db
      labels:
        name: postgres-service
        app: demo-voting-app
    spec:
      selector:
        name: postgres-pod
        app: demo-voting-app # 지정할 Pod의 metdata > Labels을 그대로 집어넣어줌
      ports:
        - port: 5432
          targePort: 5432
    ```
5. Service 생성 결과

    ![Service 생성결과](/assets/img/kubernetes/7_microservice_3.png)

<br>
<br>

**3. 실행결과 확인**

- NodePort Type으로 생성했던 voting, result Service의 접속 url을 확인

![service url 확인](/assets/img/kubernetes/7_microservice_4.png)

<br>

- Voting App에 접속

![Voting App](/assets/img/kubernetes/7_microservice_5.png)

<br>

- Result App에 접속

![Result App](/assets/img/kubernetes/7_microservice_6.png)

<br>

------------------

- 여기까지 Pod을 생성하여 Voting App을 구성하여 보았다.
- 다음 게시물에서 이어서 Deployment을 생성하여 Voting App을 구성하여 볼 것 이다.

------------------
**◎ 참고자료**

- Udemy - Kubernetes for the Absolute Beginners - Hands-on
- [쿠버네티스 공식문서 - 서비스](https://kubernetes.io/ko/docs/concepts/services-networking/service/)
- [쿠버네티스 공식문서 - 파드](https://kubernetes.io/ko/docs/concepts/workloads/pods/)