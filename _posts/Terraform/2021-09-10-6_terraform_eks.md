---
layout: single
title: "[Terraform] 6. Terraform으로 EKS 구성"
categories: [Terraform]
author_profile: true
excerpt: Terraform을 공부하면서 알게된 내용들을 정리한다.
toc: true
toc_sticky: true
---

## Terraform으로 EKS 구성
- Terraform의 기본 구성을 알아본다.
- Terraform sample 코드를 기반으로 EKS를 구성해본다.


<br>


## Terraform의 기본 구성

### Provider :

- 테라폼과 외부 서비스를 연결해주는 기능을 하는 모듈
- AWS, Azure, GCP, k8s, Helm, rancher, vsphere 등 다양한 Provider를 제공함.

⇒ [https://registry.terraform.io/browse/providers](https://registry.terraform.io/browse/providers)

```jsx
# Kubernetes config파일을 이용하여 Provider 설정
# EKS로 구성된 클러스터로 설정 가능

1. config 파일로 클러스터 지정
provider "kubernetes" {
  config_path = "~/.kube/config"
}

2. EKS 클러스터로 지정
provider "kubernetes" {
  host                   = data.aws_eks_cluster.cluster.endpoint
  token                  = data.aws_eks_cluster_auth.cluster.token
  cluster_ca_certificate = base64decode(data.aws_eks_cluster.cluster.certificate_authority.0.data)
}
```

<br>

### Resource

- Provider가 제공해주는 조작 가능한 대상의 최소단위.
- AWS의 instance

    k8s의 Pod / Service / Deploy 등 

```jsx
# NameSpace 생성

resource "kubernetes_namespace" "example" {
  metadata {
    annotations = {
      name = "example-annotation"
    }

    labels = {
      mylabel = "label-value"
    }

    name = "terraform-example-namespace"
  }
}
```

<br>

### Data Resource :

- Provider에서 제공하는 리소스 정보를 가져와서 테라폼에서 사용할 수 있는 형태로 매핑 시킬 수 있다.
- 이미 클라우드 콘솔에 존재하는 리소스를 가져오는 것.

```jsx
# 'terraform-example'이라는 Pod를 가져오는 data source

data "kubernetes_pod" "test" {
  metadata {
    name = "terraform-example"
  }
}

-> resource에서 data.kubernetes_pod.test로 사용할 수 있음.
```

<br>


### BackEnd :

#### 1) **테라폼 상태 & backend**

클라우드상엔 테라폼으로 생성한 것들 외에도 다양한 리소스들이 있다

테라폼은 자신이 관리하는 리소스들을 식별해야 하는데, 이 때 사용하는 것이 `.tfstate` 파일이다

이 때 **backend**란 테라폼의 상태(.tfstate)를 저장할 **Remote State Storage**에 대한 설정이다

backend를 사용하면 현재 배포된 최신 상태를 외부에 저장/공유하기 때문에 **다른 사람과의 협업이 가능**해진다

AWS S3를 많이 사용한다고 한다

여기서 테라폼의 명령들을 다시 한 번 살펴보겠다 (backend가 지정되었을 시)

- `terraform init`
    - 지정한 backend에 상태 저장을 위한 .tfstate 파일을 생성 (가장 마지막에 적용한 테라폼 내역 저장)
    - init 작업이 완료되면 .tfstate에 정의된 내용을 담은 .terraform 파일이 생성됨
    - 기존에 다른 개발자가 .tfstate에 인프라를 정의해 놓았다면, init 명령을 통해 local(현재 개발자가 작성하고 있는 코드)에서 sync를 맞출 수 있다
- `terraform plan`
    - 정의한 코드가 어떤 인프라를 만들게 되는지 예측 결과를 보여주는 명령
    - backend가 지정되어있지 않을 때와 동일하다
- `terraform apply`
    - 실제 인프라 배포 생성 & 작업 결과가 **backend의 .tfstate 파일에 저장됨**
    - 해당 결과는 local의 .terraform 파일에도 저장됨

추가적으로, 이미 배포된 인프라 리소스들을 terraform state로 옮겨주는 `terrform import` 명령도 있다

#### **2. Locking & Backup**

Terraform 상태는 기본적으로는 로컬 스토리지에 저장을 하지만 backend를 사용해 S3, consul, etcd 등 다양한 원격 저장소를 사용할 수 있다

원격 저장소를 사용하는 이유는 아래와 같다

- **Locking** : 협업 시 동시에 같은 state에 접근하는 것을 막는다
- **Backup** : state 파일 유실 방지

따라서 다음 글에서 진행할 실습에서는 테라폼의 상태 저장을 위한 S3 뿐 아니라 Lock을 위한 DynamoDB Table도 생성해서 사용한다

<br>

---

### 주의할 점

- tf 파일 작성시 value를 입력할 때 **'를 사용하면 오류가 발생하였다. "로 사용해주어야 한다.**
- AWS access_key, secret_key 위치

    → My security credentials → Users → 내 IAM 계정 클릭 → Security credentials 탭

    → Access Key 에서 Key확인, Secret Key를 모를경우 Key 재발급후 사용

- **access_key**, **secret_key**는 vpc.tf의 provider aws {} 블록에 추가해주었다.
- 샘플파일은 vpc.tf, eks-cluster.tf, kubernetes.tf, outputs.tf, security-groups.tf, versions.tf로 나누어져 있다.
- 샘플파일
    1. **vpc.tf** :aws에서의 기본 설정( region, access_key, secret_key)과 vpc 관련 설정을 하는 tf 파일
    2. **eks-cluster.tf** : eks 클러스터 관련 설정을 하는 tf 파일 (worker node의 그룹과 사양을 정해준다.)
    3. **kubernetes.tf** : 쿠버네티스 Provider를 설정해주는 tf 파일. 

        eks-cluster.tf파일에서 가져온 Data Resource를 사용하여 클러스터를 설정한다.

    4. **outputs.tf :** 테라폼 작업 수행 후 출력시키는 내용 설정
    5. **security-groups.tf** : aws security groupt을 설정하는 tf 파일
    6. **versions.tf** : 필요한 provider들의 버전이 정리된 tf 파일
    7. **run.py** : 테스트 구성이 완료된 후 terraform init,  terraform apply명령어와 쿠버네티스 config 파일을 적용시켜주는 명령어를 실행시키는 파이썬 스크립트 파일이다.
    8. **k8s-jenkins.tf** : helm으로 jenkins를 배포하는데 필요한 설정과 내용이 담겨있는 tf 파일이다.


<br>
<br>

------------------
**◎ 참고자료**

- [테라폼(Terraform) 기초 튜토리얼: AWS로 시작하는 - 44BITS](https://www.44bits.io/ko/post/terraform_introduction_infrastrucute_as_code)
- [(Terraform) 테라폼 backend의 이해 - honglab - 티스토리](https://honglab.tistory.com/117)

