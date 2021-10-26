---
layout: single
title: "[CKA Concept] 10. Imperative VS Declarative"
categories: [Kubernetes]
author_profile: true
excerpt: CKA 강의 개념 중 쿠버네티스에서의 Imperative와 Declarative에 대해 정리한다. 
toc: true
toc_sticky: true
---

## Imperative VS Declarative
- 쿠버네티스에서는 명령어를 직접 입력하거나 구성파일을 만들어 컴포넌트를 생성할 수 있다.
- Imperative : 택시를 예로들면 택시를 타고나서 기사에게 상세하게 길안내를 해주는 것(어느블락에서 왼쪽으로, 어느 코너에서 오른쪽으로 꺾어달라 등)
- Declarative : 우버나 카카오택시 처럼 '목적지는 우리집이다'만 등록하면 택시기사가 알아서 목적지까지 간다.

<br>

### Code에서의 예시
-----------------------------
- Imperative(명령형)
```shell
1. web-server라는 이름의 VM을 생성
2. NGINX 설치
3. config파일에 PORT는 8080으로 수정
...
...
```

- Declarative(선언형)
```yml
VM Name: web-server
Database: nginx
Port: 8080
...
...
```

<br>

### Kubernetes 오브젝트 관리
---------------------------
1. 명령형 커맨드
2. 명령형 오브젝트 구성
3. 선언형 오브젝트 구성

<br>

### 명령형 커맨드 (Imperative commands)
--------------------
ex) Deployment를 생성하여 nginx 컨테이너 인스턴스를 구동시킨다.
```shell
kubectl create deployment nginx --image nginx
```

- **오브젝트 구성에 비한 장단점**
    - **장점**
        - 하나의 동작을 나타내는 단어로 표현
        - 클러스터를 수정하기 위해 단 하나의 단계만을 필요로함
    - **단점**
        - 변경 검토 프로세스와 통합되지 않는다.
        - 변경에 대한 추적을 제공하지 않는다.
        - 활성동작 중인 경우를 제외하고는 레코드의 소스를 제공하지 않는다.
        - 새로운 오브젝트 생성을 위한 템플릿을 제공하지 않는다.

<br>

### 명령형 오브젝트 구성 (Imperative object configuration)
--------------------
ex) 구성파일에 정의된 오브젝트 생성

```shell
kubectl create -f nginx.yaml
```

두 개의 구성 파일에 정의된 오브젝트를 삭제

```shell
kubectl delete -f nginx.yaml -f redis.yaml
```

활성 동작하는 구성을 덮어씀으로써 구성 파일에 정의된 오브젝트를 업데이트
```shell
kubectl replace -f nginx.yaml
```

<br>

- **명령형 커맨드에 비한 장단점**
    - **장점**
        - Git과 같은 소스 컨트롤 시스템에 보관가능
        - 푸시와 감사추적전 변경사항 검토와 같은 프로세스들과 통합할 수 있음.
        - 새로운 오브젝트 생성을 위한 템플릿 제공
    - **단점**
        - 오브젝트 스키마에 대한 기본적 이해필요
        - YAML 파일 기록하는 추가적인 과정 필요

<br>

- **선언형 오브젝트 구성에 비한 장단점**
    - **장점**
        - 선언형 오브젝트 구성보다 간결하고 이해하기 쉬움
        - 쿠버네티스 버전 1.5부터는 더 발전된 명령형 오브젝트 구성을 제공함
    - **단점**
        - 디렉토리가 아닌 파일에 가장 적합
        - 활성 오브젝트에 대한 업데이트는 구성파일에 반영되어야함. 그렇지 않으면 다음 교체중에 손실됨

<br><br>


### 선언형 오브젝트 구성 (Declarative object configuration)
--------------------
ex) configs 디렉터리 내 모든 오브젝트 구성 파일을 처리하고 활성 오브젝트를 생성 또는 패치한다. 먼저 어떠한 변경이 이루어지게 될지 알아보기 위해 diff 하고 나서 적용할 수 있다.
```shell
kubectl diff -f configs/
kubectl apply -f configs/
```

재귀적으로 디렉터리를 처리한다.
```shell
kubectl diff -R -f configs/
kubectl apply -R -f configs/
```
<br>

- **명령형 오브젝트 구성에 비한 장단점**
    - **장점**
        - 활성 오브젝트에 직접 작성된 변경 사항은 구성 파일로 다시 병합되지 않더라도 유지됨
        - 선언형 오브젝트 구성은 디렉터리에서의 작업 및 오브젝트 별 작업 유형(생성, 패치, 삭제)의 자동 감지에 더 나은 지원을 제공
    - **단점**
        - 예상치 못한 결과를 디버깅하고 이해하기가 더 어려움
        - diff를 사용한 부분 업데이트는 복잡한 병합 및 패치 작업을 일으킴

<br><br>


### 시험 Tip
--------------------
- 명령어는 주로 **명령형 커맨드 방식**으로 하는것이 시간절약을 할 수 있다.

ex)<br>

```shell
#### Create
kubectl run --image=nginx nginx

kubectl create deployment --image=nginx nginx

kubectl expose deployment nginx --port 80 # Deployment와 연결되는 Service 생성

#### Update
kubectl edit deployment nginx

kubectl scale deployment nginx --replicas=5

kubectl set image deployment nginx nginx-nginx:1.18
```

<br>

### 참고할만한 추가자료
--------------------
- [명령형 커맨드를 이용한 쿠버네티스 오브젝트 관리하기 >>](https://kubernetes.io/ko/docs/tasks/manage-kubernetes-objects/imperative-command/)
- [오브젝트 구성을 이용한 쿠버네티스 오브젝트 관리하기(명령형) >>](https://kubernetes.io/ko/docs/tasks/manage-kubernetes-objects/imperative-config/)
- [오브젝트 구성을 이용한 쿠버네티스 오브젝트 관리하기(선언형) >>](https://kubernetes.io/ko/docs/tasks/manage-kubernetes-objects/declarative-config/)


<br><br>


------------------
**◎ 참고자료**
- Udemy - Certified Kubernetes Administrator (CKA) with Practice Tests
- [쿠버네티스 공식문서 - 쿠버네티스 오브젝트 관리](https://kubernetes.io/ko/docs/concepts/overview/working-with-objects/object-management/)
