---
layout: single
title: "[CKA Concept] 35.Image Security"
categories: [Kubernetes]
author_profile: true
excerpt: CKA 강의의 개념 중 Image Security 대해 정리한다. 
toc: true
toc_sticky: true
---

## Image Security
- Pod을 생성하는 yaml파일에 컨테이너 조건으로 image를 입력한다.


```yaml
apiVersion: v1
kind: Pod
metadata:
  name: test
spec:
  containers:
    - name: nginx
      image: nginx 
```

<br>

```image: nginx``` : docker.io/nginx/nginx
- 디폴트로 docker hub를 레지스트리 위치로 정함
- Google registry나 private registry를 사용하려면 주소를 적어주면 된다.

![](/assets/img/kubernetes/35_image_security_1.png)


<br>

### Private Repository
- private registry를 로그인 한뒤 image에 입력해주면된다.

```bash
docker login private-registry.io

docker run private-registry.io/apps/internal-app # private registry의 앱 실행

---------
# yaml파일
...
containers:
  - name: nginx
    image: private-registry.io/apps/internal-app


```

- credential 과정을 위하여 secret를 생성해야한다.

```bash
kubectl create secret docker-registry regcred \
--docker-server= private-registry.io
--docker-username= registry-user
--docker-password= registry-password
--docker-email= registry-user@org.com

------

apiVersion: v1
kind: Pod
metadata:
  name: test
spec:
  containers:
    - name: nginx
      image: nginx 
  imagePullSecrets:
    - name: regcred  # 위에서 만든 secret을 추가

```

### Security Context
- security context는 privilege와 access control setting을 해줄때 사용됨

```yml
apiVersion: v1
kind: Pod
metadata:
  name: security-context-demo
spec:
( securityContext:
  runAsUser: 1000   # User ID = 1000
  runAsGroup: 3000  # Group ID = 3000
  fsGroup: 2000 )  # supplement group ID = 2000 
   # 이곳에 추가하여 Pod에대한 access를 control 할 수 도 있다.
  containers:
    - name: nginx
      image: nginx
      securityContext:
        runAsUser: 1000   # User ID = 1000
        runAsGroup: 3000  # Group ID = 3000
        fsGroup: 2000  # supplement group ID = 2000
        capabilities:
          add: ["MAC_ADMIN"] # Capabilities는 컨테이너 level에서만 가능하다. Pod level에서는 불가
```

=> **capabilities** : 커널레벨 역량이라고 한다. root의 권한을 세분화하여 일반 유저도 root의 다양한 권한을 가지도록 만든 보안모델이다.

<br>

### 실습하며 알게된 것

- 아래의 명령어로 쿠버네티스 pod의 sleep 프로세스를 누가 실행했는지 알 수 있다.

```
kubectl exec ubuntu-sleeper -- whoami
```

- Pod에 어떤 명령어를 입력해볼 때(날짜를 입력하는 예시)

```
kubectl exec -it ubuntu-sleeper -- date -s '19 APR 2012 11:14:00'
```

<br>
<br>


------------------
**◎ 참고자료**

- Udemy - Certified Kubernetes Administrator (CKA) with Practice Tests
- [#33 Image Security & Network Policy - 작성자 ijoos](https://blog.naver.com/ijoos/222171813463)