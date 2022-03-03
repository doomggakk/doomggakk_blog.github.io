---
layout: single
title: "[Terraform] 1. Terraform 기존 인프라를 코드로 구성하기 (1)"
categories: [Terraform]
author_profile: true
excerpt: Terraform을 공부하면서 알게된 내용들을 정리한다.
toc: true
toc_sticky: true
---

## Terraform 기존 인프라를 코드로 구성하기
- 작업 하면서 기존의 인프라를 코드화 하는 작업은 매우 필요한 작업이라고 생각한다.
- 다음 블로그의 글 **(기존에 사용 중인 인프라를 Terraform으로 가져오기 - Outsider's Dev Story )** 을 참고하여 공부였고, 문서를 작성하였다.
- 또한 Terraform Workshop을 통해 배운 것으로 정리를 하였다.
- 구성파일이 없다는 것으로 가정하며 내용이 진행된다.

1. **Terraforming**
2. etc

<br>


### Terraforming
---

- 테라폼에서는 **terraform import** 라는 명령어를 제공한다.
- 기존에 구성되어있는 인프라의 상태를 불러오는 명령어인데, 문제는 **상태만** 불러온다는 것이다.
- 테라폼은 **구성파일(tf 파일)과 상태파일(tfstate 파일)**로 하나의 프로젝트를 이루어 구성된다.
- 인프라의 상태파일만 존재한다면 구성파일 또한 무조건 존재하여야 관리가 가능하다는 것이다.

→ gem 으로 설치할 수 있는 Terraforming이라는 라이브러리를 사용하면 문제를 조금 해소할 수 있다.

```bash
get install terraforming 
```

위의 명령어로 쉽게 설치 할 수 있다.

<br>

### Terraforming으로 기존의 인프라 코드화 하기 실습
---
- 가장 기초적인 코드화 작업을 위해 ec2 인스턴스 하나를 대상으로 실습을 진행한다.
- 아래의 순서대로 실습을 진행한다.

>
>1. 코드화 시킬 ec2 먼저 생성.
>2. ruby 설치 및 terraforming 설치
>3. terraforming을 이용해 ec2 인스턴스의 구성파일 생성.
>4. terraform import 명령을 사용하여 ec2 인스턴스의 상태파일 생성.
>5. terraform plan 명령어를 입력해보며 상태파일과 구성파일이 코드화가 잘 되었는지 확인.
>

---

#### 1. 코드화 시킬 ec2 instance 생성

![terraform1.png](/assets/img/terraform/terraform1.png)

#### 2. ruby 설치 및 terraforming 설치

```bash
# ruby 설치
$ sudo apt install ruby

# gem으로 terraforming 설치
$ sudo gem install terraforming

```

#### 3. terraforming을 이용해 ec2 인스턴스의 구성파일 생성.

```bash
# terraforming을 사용하여 ec2 구성내용 조회
# 현재 리전에 존재하는 모든 ec2의 구성이 조회된다.

$ terraforming ec2

...
...
resource "aws_instance" "i-0aaaa9cdb619cf8af" {
    ami                         = "ami-0b9064170e32bde34"
    availability_zone           = "us-east-2b"
    ebs_optimized               = false
    instance_type               = "t2.micro"
    monitoring                  = false
    key_name                    = "test"
    subnet_id                   = "subnet-a50252df"
    vpc_security_group_ids      = ["sg-00c01c6c249f507a9"]
    associate_public_ip_address = true
    private_ip                  = "172.31.18.183"
    source_dest_check           = true

    root_block_device {
        volume_type           = "gp2"
        volume_size           = 8
        delete_on_termination = true
    }

    tags {
    }
}

...
...
```

#### 4. terraform import 명령을 사용하여 ec2 인스턴스의 상태파일 생성.

```bash
# import 형식
terraform import <resource type>.<resource name> <resource id>

# terraform import 명령으로 3번에서 조회한 인스턴스의 상태파일 불러오기
terraform import aws_instance.i-0aaaa9cdb619cf8af i-0aaaa9cdb619cf8af
...
...
Import successful!

```

#### 5. terraform plan 명령어를 입력해보며 상태파일과 구성파일이 코드화가 잘 되었는지 확인.

- 현재 구성한 파일은 아래와 같다.

```bash
.
├── main.tf
├── terraform.tfstate
└── version.tf

------------------------------------------------------------------------

# main.tf

variable "region" {
  default     = "us-east-2"
  description = "AWS region"
}

provider "aws" {
  region = var.region

  access_key   = "aws_access_key 입력"
  secret_key   = "aws_secret_key 입력"
  
}

resource "aws_instance" "i-0aaaa9cdb619cf8af" {
    ami                         = "ami-0b9064170e32bde34"
    availability_zone           = "us-east-2b"
    ebs_optimized               = false
    instance_type               = "t2.micro"
    monitoring                  = false
    key_name                    = "test"
    subnet_id                   = "subnet-a50252df"
    vpc_security_group_ids      = ["sg-00c01c6c249f507a9"]
    associate_public_ip_address = true
    private_ip                  = "172.31.18.183"
    source_dest_check           = true

    root_block_device {
        volume_type           = "gp2"
        volume_size           = 8
        delete_on_termination = true
    }

}

=> terraforming으로 얻어낸 구성파일 내용.

-----------------------------------------------------------------------

# version.tf
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = ">= 3.20.0"
    }

  }

  required_version = "> 0.14"
}
```

- terraform plan 명령어를 입력해보았다.

```bash
$ terraform plan

aws_instance.i-0aaaa9cdb619cf8af: Refreshing state... [id=i-0aaaa9cdb619cf8af]

No changes. Your infrastructure matches the configuration.

Terraform has compared your real infrastructure against your configuration and found no differences, so no changes are needed.
```

→ 위 처럼 No changes가 출력되면 코드화가 정상적으로 이루어졌다고 볼 수 있다.

<br>
<br>

------------------
**◎ 참고자료**

- [기존에 사용 중인 인프라를 Terraform으로 가져오기 - Outsider's Dev Story](https://blog.outsider.ne.kr/1292)
