---
layout: single
title: "[Terraform] 7. Terraform으로 Kubernetes 관리 - K8s, Helm Provider사용"
categories: [Terraform]
author_profile: true
excerpt: Terraform을 공부하면서 알게된 내용들을 정리한다.
toc: true
toc_sticky: true
---

## Terraform으로  Kubernetes 관리 -  k8s, Helm Provider사용
- 이전 게시물에서 구성된 EKS에 테라폼을 사용하여 object들을 관리해본다.
- k8s와 Helm 프로바이더를 사용하여 진행한다.


<br>

### 설치전

> 참고 [https://learn.hashicorp.com/tutorials/terraform/eks](https://learn.hashicorp.com/tutorials/terraform/eks)

- Terraform eks 배포 샘플코드를 기반으로 만들었다.

    ```jsx
    git clone https://github.com/hashicorp/learn-terraform-provision-eks-cluster
    ```

- aws 명령어와 같은 필요한 명령어 사용을 위한 패키지를 설치한다.
    1. aws-iam-authenticator
    2. awscli
    3. kubectl
    4. wget

- aws configure 명령어를 입력하여 자신의 IAM 계정을 등록해준다.

- eks 배포 샘플코드에는 helm provier가 선언되어있지 않기 때문에 **kubernetes.tf**에 아래와 같이 helm provider를 선언해주었다.

```jsx
# kubernetes.tf

provider "kubernetes" {
  host                   = data.aws_eks_cluster.cluster.endpoint
  token                  = data.aws_eks_cluster_auth.cluster.token
  cluster_ca_certificate = base64decode(data.aws_eks_cluster.cluster.certificate_authority.0.data)
}

provider "helm" {
  kubernetes {
    host                   = data.aws_eks_cluster.cluster.endpoint
    token                  = data.aws_eks_cluster_auth.cluster.token
    cluster_ca_certificate = base64decode(data.aws_eks_cluster.cluster.certificate_authority.0.data)
  }

}
```

<br>

### Jenkins Helm으로 설치

- Jenkins를  Helm 차트를 사용하여 설치 할경우 다음과 같은 순서로 진행된다.
1. Namespace 생성
2. PV(Persistent Volume) 생성
3. Service Account, Cluster Role, ClusterRoleBinding 생성
4. jenkins-values.yaml을 이용하여 helm 차트로 jenkins 설치 진행
5. kubectl port-forward로 접속가능

<br>


### Terraform 코드로 작성

- Terraform으로 작업을 진행하기 위해서는 위의 과정을 Terraform 파일에

    코드로 옮기는 작업이 필요하다.




#### 1. Namespace 생성

- k8s provider에서 제공하는 resource를 사용하여 코드를 작성한다.
- yaml파일의 형식과 비슷하기 때문에 형식에 맞게 value를 넣어주면 된다.

```jsx
resource "kubernetes_namespace" "jenkins" {
    metadata {
        name = "jenkins"
    }

}
```

#### 2. PV 생성

```jsx
resource "kubernetes_persistent_volume" "jenkins-pv" {
  metadata {
    name      = "jenkins-pv"
  }
  spec {
    capacity = {
      storage = "20Gi"
    }
    storage_class_name = "jenkins-pv"
    access_modes = ["ReadWriteOnce"]
    persistent_volume_reclaim_policy = "Retain"
    persistent_volume_source {
      host_path {
        path = "/data/jenkins-volume/"
      }
    }
  }
}
```

#### 3. Service Account, Cluster Role, Cluster Role Binding 생성

```jsx
resource "kubernetes_service_account" "jenkins" {
  metadata {
    name = "jenkins"
  }
}

resource "kubernetes_cluster_role" "jenkins" {
  metadata {
    labels = {
        bootstrapping= "rbac-defaults"
    }
    name = "jenkins"
  }

  rule {
    api_groups = ["'*'",""]
    resources  = ["statefulsets","services","replicationcontrollers","replicasets","podtemplates","podsecuritypolicies","pods","pods/log","pods/exec","podpreset","poddisruptionbudget","persistentvolumes","persistentvolumeclaims","jobs","endpoints","deployments","deployments/scale","daemonsets","cronjobs","configmaps","namespaces","events","secrets"]
    verbs      = ["create","get","watch","delete","list","patch","update"]
  }

  rule {
    api_groups = [""]
    resources  = ["nodes"]
    verbs      = ["get","list","watch","update"]
  }

}

resource "kubernetes_cluster_role_binding" "jenkins" {
  metadata {
    labels = {
        bootstrapping= "rbac-defaults"
    }
    name = "jenkins"
  }
  role_ref {
    api_group = "rbac.authorization.k8s.io"
    kind      = "ClusterRole"
    name      = "jenkins"
  }
  subject {
    kind      = "Group"
    name      = "system:serviceaccounts:jenkins"
    api_group = "rbac.authorization.k8s.io"
  }
}
```

#### 4. jenkins-values.yaml을 이용하여 helm 차트로 jenkins 설치 진행

- jenkins에서 제공하는 jenkins-values.yaml파일을 저장한 뒤에 경로를 넣어준다.
- helm provider에서 제공하는 resource를 사용한다.

```jsx
resource "helm_release" "jenkins-ci" {
    name   = "jenkins"
    chart  = "jenkinsci/jenkins"
    values = [
        "${file("./jenkins-values.yml")}"
    ]
}
```

⇒ 위 모든 resource 코드를 하나의 **'k8s-jenkins.tf'**라는 파일에 모아서 저장하였다.

#### 5. kubectl port-forward로 접속가능하도록 명령어 입력

```jsx
$ kubectl port-forward jenkins-0 8080:8080

Forwarding from 127.0.0.1:8080 -> 8080
Forwarding from [::1]:8080 -> 8080
```

- 다음 명령어로 초기 비밀번호를 조회할 수 있다.

```jsx
$ jsonpath="{.data.jenkins-admin-password}"
$ secret=$(kubectl get secret -n jenkins jenkins -o jsonpath=$jsonpath)
$ echo $(echo $secret | base64 --decode)

username : admin
password : 26wJLn8ruNy6dY3dmZh9Wm
```

#### 6. Python script 작성

- 위의 과정을 완료하였으면 파이썬 스크립트를 생성하여 테라폼 코드적용과

    jenkins 포트 포워딩을 자동으로 실행한다.

```jsx

#-*-coding:utf-8-*-*

import sys
import os
import threading
import datetime

rootdir = os.path.dirname(os.path.abspath(__file__))
os.chdir("{0}".format(rootdir))
os.system("terraform init")
os.system("terraform apply -auto-approve")
os.system("aws eks --region $(terraform output -raw region) update-kubeconfig --name $(terraform output -raw cluster_name)")

os.system("kubectl port-forward jenkins-0 8080:8080")
```


<br>



### 결과화면

- EKS 인스턴스 생성 확인 (us-east-2 리전에 생성)

![Screenshot from 2021-08-24 15-44-01.png](/assets/img/terraform/tf7-1.png)

eks-cluster

![Screenshot from 2021-08-24 15-41-10.png](/assets/img/terraform/tf7-2.png)

eks instance

- localhost:8080 접속

![Screenshot from 2021-08-24 15-24-02.png](/assets/img/terraform/tf7-3.png)




<br>
<br>

------------------
**◎ 참고자료**

- [Provision an EKS Cluster (AWS) | Terraform - HashiCorp Learn](https://learn.hashicorp.com/tutorials/terraform/eks)
