---
layout: single
title: "[CKA Concept] 32. Kube Config"
categories: [Kubernetes]
author_profile: true
excerpt: CKA 강의의 개념 중 Kube Config에 대해 정리한다. 
toc: true
toc_sticky: true
---

## Kube Config
- Kubernetes에서 curl를 사용해서 클러스터에 접근할 때 원래는 key, cert, cacert, cluster 등을 옵션으로 함께 보내줘야한다.

```bash
curl https:/my-kube-playground:6443/api/v1/pods \
--key admin.key
--cert admin.crt
--cacert ca.crt
```

- 또한 kubectl 명령어를 사용할때도 server, cilent-key, client-cert, CA를 옵션에 추가해서 사용해야한다.
- 위의 과정이 너무 번거롭기 때문에 config파일을 생성하여 좀 더 효율적으로 쿠버네티스를 사용한다.
- $HOME/.kube/ 경로에 config라는 파일을 생성한다. 이 경로에 config파일이 있을경우 위치를 적어줄 필요없이 알아서 config파일을 적용한다. 

```
# config (파일이름)

--server my-kube-playground:6443
--client-key admin.key
--client-certificate admin.crt
--certificate-authority ca.crt

-------------------
$ kubectl get pods --kubeconfig config
```

<br>

### Config파일 구조
1.Cluster
- 접근하려는 Cluster들에대한 정보를 입력<br>
ex) Dev, Prod, Google 클러스터 등

2.Contexts
- Cluster와 User을 연결해주는 정보를 입력<br>
ex) Admin@Prod, Dev@Google

3.User
- 클러스터에 접근하려는 여러 사용자들<br>
ex) Admin, Dev User, Prod User 등


### config 파일 YAML

```yaml
apiVersion: v1
kind: Config

current-context: dev-user@google

clusters:

- name: my-kube-playground
  cluster:
    certificate-authority: ca.crt # 절대경로를 넣어도됨
  # certificate-authority-data :    데이터를 인코딩하여 직접 넣어주어도 된다.
    server: https://my-kube-playfround:6443
- name: ....
....
- name: ....
....


contexts:

- name: my-kube-admin@my-kube-playground
  context:
    cluster: my-kube-playground
    user: my-kube-admin
- name: ....
....
- name: ....
....


users:

- name: my-kube-admin 
  user:
    client-certificate: admin.crt
    client-key: admin.key
- name: ....
....
- name: ....
....
```

<br>

- 자신이 작성한 kubectl config파일을 조회할 수 있다.
```bash
kubectl config view --kubeconfig=my-custom-config
```
- current-context를 바꾸어줄 수 있다.

```bash
kubectl config use-context prod-user@production

kubectl get pod --kubeconfig=config
```


### 연습예제
1.$HOME/.kube/config 에 config 파일이 있다.

```bash
echo $HOME
/root

/root/.kube/config에 위치
```

2.직접 만든 config file의 context로 변경하기

```bash
kubectl config use-context --kubeconfig=<직접만든 config 이름> <변경하기 원하는 context 이름>
```

3.직접 만든 config file을 config로 사용하려면 --kubeconfig 옵션을 넣어주어야한다. <br>
이러한 번거로움을 없애려면 $HOME/.kube/config 파일을 교체해주면 된다.



<br>
<br>


------------------
**◎ 참고자료**

- Udemy - Certified Kubernetes Administrator (CKA) with Practice Tests
- [#30 KubeConfig - 작성자 ijoos](https://blog.naver.com/ijoos/222170880953)