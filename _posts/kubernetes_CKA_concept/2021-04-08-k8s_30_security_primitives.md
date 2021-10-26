---
layout: single
title: "[CKA Concept] 30. Security Primitives"
categories: [Kubernetes]
author_profile: true
excerpt: CKA 강의의 개념 중 Security Primitives에 대해 정리한다. 
toc: true
toc_sticky: true
---

## Security Primitives"
- 접근하는 사람에 따라 -> Authentication 메커니즘(인증)
    - Files : Username & Pwd
    - Files : Username & Token
    - Certificates
    - External Authentication Providers - LDAP(별도 인증 시스템))
    - Service Account
<br>
- 무엇을 하는가에 따라
    - RBAC(Role Based Access Certification)
    - ABAC(Attribute Based Access Certification)
    - Node
    - Webhook 모드

<br>

- k8s에서는 기본적으로 모든 Node들간의 통신이 허용되고 있다.
-> 이를 통제하기 위해서는 Network Policy를 이용해야한다.
- k8s 자체에서 user를 생성하거나 관리하는 것은 불가하다.
-> serviceaccount를 생성하여 관리하는 방법은 가능하다.

![](/assets/img/kubernetes/30_authentication_1.png)
<br>

### Accounts
- Authenticattion(인증) : 사용자가 누구인지를 식별
- Authorization(인가) : 사용자의 권한을 확인
<br>
- 쿠버네티스에서는 모든 사용자의 접근이 **kube-apiserver**에 의해 관리된다.
- kube-apiserver의 인증매커니즘
1. static Password File
2. Static Token File
3. Certificates
4. LDAP(Lightweight Directory Access Protocol)과 같은 별도 인증 시스템

-> 1,2번이 간단하지만 보안에는 취약한 방법이다

<br>

#### user정보를 csv파일에 저장한 후 kube-apiserver에 추가해주는 방법

- ```--basic-auth-file=<user정보 파일>``` 을 추가해준다.

![](/assets/img/kubernetes/30_authentication_3.png)

- Yaml 파일에 추가하는 방법

![](/assets/img/kubernetes/30_authentication_2.png)

- 자신의 Authenticate 정보를 알아내는 명령어

```shell
curl -v -k https://master-node-ip:6443/api/v1/pods -u "<username>:<password>"

ex) curl -v -k https://master-node-ip:6443/api/v1/pods -u "user1:password123"
```

<br>

#### Token을 csv파일에 저장하여 인증처리방법
- Token을 csv파일에 저장한 뒤 --token-auth-file=<token 파일> 을 추가해준다.

![](/assets/img/kubernetes/30_authentication_4.png)
- 다음과 같은 명령어로 authenticate 정보를 조회한다.

```shell
curl -v -k https://master-node-ip:6443/api/v1/pods --header "Authorization: Bearer <Token>"
```

<br>
<br>


------------------
**◎ 참고자료**

- Udemy - Certified Kubernetes Administrator (CKA) with Practice Tests
- [#28 Security Authentication - 작성자 ijoos](https://blog.naver.com/ijoos/222168965561)