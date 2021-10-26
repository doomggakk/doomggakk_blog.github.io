---
layout: single
title: "[Terraform] 5. Terraform에서 조건문, 반복문 구현"
categories: [Terraform]
author_profile: true
excerpt: Terraform을 공부하면서 알게된 내용들을 정리한다.
toc: true
toc_sticky: true
---

## Terraform에서 조건문, 반복문 구현
- Terraform은 조건문과 반복문같은 논리적 구성을 구현하기가 매끄럽지 못하다.
- 하지만 구현할 수 있는 방법이 있다. 알아보도록 하자.

<br>

### 테라폼에서는 count라는 메타변수로 조건문, 반복문을 구현할 수 있다.

- resource 내의 count 변수의 값만큼 반복적으로 작업을 수행한다.
- ex) count = 0 → 실행 X

          count = 1 → 1번 실행

          count = 2 → 2번 실행 (사용자 생성으로 test 해보았을 때 중복되어 생성할 수 없다는 오류 발생 )

- 조건문 처리방법
1. flag 변수를 아무거나 하나 선언한다.
2. resource에서 count 변수에 조건문을 넣는다.
3. 적절하게 if - else 문을 작성할 수 있다.

### 예시

- rancher의 user를 생성하는 tf 코드가 있다.
- is_test_odd라는 변수를 선언한다. ( type = bool )
- resource 에서 아래와 같이 count 변수를 사용한다.

```jsx

variable "is_test_odd" {
	default = true
	description = ""
}

resource "rancher2_user" "test3" {
	count = var.is_test_odd ? 1 : 0

	name = "test3"
	username = "test3"
	password = "******"
	enabled = true
}

resource "rancher2_user" "test4" {
	count = var.is_test_odd ? 0 : 1

	name = "test3"
	username = "test3"
	password = "******"
	enabled = true
}
```

## Variable 동적할당 후 조건문을 이용하여 다른 값주기

- variable의 type만 지정해주면 terraform plan명령어나 terraform apply 명령어를 실행할 때 변수의 값을 입력받는다.
- 스크립트에서 변수에 따라 다른값을 할당해준다.

```bash
# variable.tf
variable "environment" {
  type = string
}

resource "kubernetes_namespace" "jenkins" {
    metadata {
        name = var.environment
    }
}

$ terraform plan
var.environment
  Enter a value: <값을 입력해준다>

```

<br>
<br>

------------------
**◎ 참고자료**

- [05. 테라폼 팁과 요령: 반복문, 조건문, 배포 및 주의사항 - 구르미의 개발 이야기](https://gurumee92.tistory.com/252)

- [Terraform 입문 - 반복문, if문, if-else문 - SeungHyeon](https://velog.io/@seunghyeon/Terraform-%EC%9E%85%EB%AC%B8-%EB%B0%98%EB%B3%B5%EB%AC%B8-if%EB%AC%B8-if-else%EB%AC%B8)

- [(Terraform) 메타변수 count로 반복문, 조건문 사용하기](https://honglab.tistory.com/121)

