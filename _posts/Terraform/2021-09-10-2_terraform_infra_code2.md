---
layout: single
title: "[Terraform] 2. Terraform 기존 인프라를 코드로 구성하기 (2)"
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

1. Terraforming
2. **etc**

<br>



### Terraform 명령어로 코드 구성하기.
---
- tfstate 파일이 존재한다는 것은 '**terraform으로 resource관리를 할 것 이다**' 라는 것을 의미한다.
- 따라서 인프라를 코드로 구성하여 테라폼으로 관리하려면 **state 파일이 존재하여야 한다.**

<br>

#### state 파일이 존재하지 않는 경우

1.state파일을 형상관리 한 경우 : 

```bash
1. state 복붙 
2. terraform show
3. tf구성파일에 instance 채워넣기
```

2.형상관리한 state 없이 aws에만 resource가 있는 경우 : 

```bash
1. terraform import aws_instance.<resource name> <instance ID>
2. terraform show
3. tf구성파일에 instance 채워넣기 
```

#### state 파일이 존재하는 경우

```bash
1. terraform refresh (state파일 최신화) 
2. terraform show (state 파일기반 instance 구성 코드 출력)
3. 코드 복사하여 main.tf에 입력
4. terraform plan을 입력했을 때 no change가 출력되면 된다.
```

=> 구성코드를 작성할 때 terraform plan 명령어를 입력해가며 오류가 나는 부분은 지워가며 작업해야한다.

=> region이나 provider와 같은 기본적인 내용은 구성파일에 작성해두는 것이 좋다.
 

<br>
<br>

------------------
**◎ 참고자료**

- [Get Started - AWS Terraform - HashiCorp Learn](https://learn.hashicorp.com/collections/terraform/aws-get-started)

- Terraform Workshop - Introduction to Terraform on Azure (Korean)