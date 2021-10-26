---
layout: single
title: "[Terraform] 4. Terraform 서버환경에 따른 인프라 관리"
categories: [Terraform]
author_profile: true
excerpt: Terraform을 공부하면서 알게된 내용들을 정리한다.
toc: true
toc_sticky: true
---

## Terraform 서버환경에 따른 인프라 관리
- 인프라를 구성할 때 개발환경에 따라 다르게 구성하는 것이 일반적이다.
- 이러한 경우 테라폼을 이용해서 어떻게 효율적으로 관리할 것인지 알아본다.

<br>

### 1. 환경마다 프로젝트를 분리

- 스크립트 내에서 조건문으로 관리하는 것보다 환경별로 디렉토리를 분류하여 tf파일을 작성한다.
- 사용하려는 환경의 프로젝트로 이동해서 테라폼 명령어를 입력한다.
- 모든 리소스를 완벽하게 격리시킬 수 있지만, 중복되는 코드가 많아 효율성이 떨어지고 유지보수가 번거로울 수 있다.

```jsx
terraforming-dev/
└── aws
    ├── api-gateway.tf
    ├── lambda.tf
    └── ...
terraforming-prod/
└── aws
    ├── api-gateway.tf
    ├── lambda.tf
    └── ...
```

<br>

### 2. 같은 프로젝트에서 디렉토리로 분리

- 기본적인 config파일들을 공유한다.
- Terraform에서는 폴더 하나가 **모듈**의 단위가 되기 때문에, 공통으로 사용하는 인프라 구성은 위처럼 모듈화한 뒤 환경별로 변수 값만 바꾸어 재사용하면 개발 환경과 운영 환경에 쉽게 동일한 인프라를 구성할 수 있다.
- 중복되는 variable같은경우에는 한곳에 variable파일을 모아 동일한 variable.tf를 사용하도록 지정 할 수 있다.
- 사용하려는 환경의 디렉토리로 이동해서 테라폼 명령어를 입력한다.
- 하지만 이러한 방법도 중복코드가 많아질 수 밖에 없어 환경에 따른 인프라 관리가 비효율적일 수 있다.

```jsx
terraforming/
└── aws
    ├── env
    │   ├── dev
    │   │   ├── instance.tf
    │   │   └── security_group.tf
    │   ├── prod
    │   │   ├── instance.tf
    │   │   └── security_group.tf
    │   ├── staging
    │   └── testing
    └── modules
        └── module_name
            ├── main.tf
            ├── output.tf
            └── variables.tf
```

<br>

### 3. Terraform workspace 사용

- 위와 같은 문제들에 대한 사용자들의 요구를 반영하여 Terraform workspace라는 기능이 추가되었다.
- 살짝 테스트를 해보니 Git의 branch와 비슷한 느낌을 받을 수 있었다.
- 이 방법은 유지 보수가 쉽고 더 편리하기 때문에 거의 유사한 인프라를 서로 다른 환경으로 배포할 때 유용하게 사용할 수 있다.
- 기본 사용 방법은 아래와 같다.

```bash
# Create a new workspace
$ terraform workspace new prod
$ terraform workspace new test

# Select a workspace (already created)
$ terraform workspace select prod

# list workspaces
$ terraform workspace list
  default
* prod
  test
 
# Plan or Apply
$ terraform plan
$ terraform apply
```

- 환경마다 다른 값을 집어넣어주기 위해서는 아래와 같이 **조건문을 사용**하거나,

    **Map 형태의 변수**를 사용하는 방법이 있다.( 이 방법이 더 깔끔 )

```bash
# 조건문 사용법
resource "aws_route53_zone" "primary" {
  name = "${terraform.workspace == "prod" ? "salesbooster.io" : "dev.salesbooster.io"}"
}

# Map형태의 변수 사용법
variable "root_domain_name" {
  type = "map"
  default = {
    "prod" = "salesbooster.io"
    "dev" = "dev.salesbooster.io"
  }
}
resource "aws_route53_zone" "primary" {
  name = "${var.root_domain_name[terraform.workspace]}"
}
```

- [조건문 문서](https://www.notion.so/Terraform-63326aed512f4f95b1df047e4799e0e0)에서 정리했던 것 처럼 환경에 따라 resource의 사용여부가 다르다면 아래와 같이 조건문과 count변수를 사용하여 처리한다.

```bash
resource "aws_route53_record" "example" {
  count = "${terraform.workspace == "prod" ? 1 : 0}"
	...
}
```

<br>
<br>

------------------
**◎ 참고자료**

- [Terraform의 Workspace를 이용해 배포 환경 분리하기 - Medium](https://medium.com/@blaswan/terraform-workspaces-for-deployment-environments-2deff99356f6)


