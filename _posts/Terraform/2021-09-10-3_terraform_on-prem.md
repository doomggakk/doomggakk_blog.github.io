---
layout: single
title: "[Terraform] 3. On-premis 환경에서 Terraform 사용"
categories: [Terraform]
author_profile: true
excerpt: Terraform을 공부하면서 알게된 내용들을 정리한다.
toc: true
toc_sticky: true
---

## On-premis 환경에서  Terraform 사용
- 회사에서 작업을 하다보면 폐쇄망 환경에서 인프라를 구축하는 경우가 많다.
- 폐쇄망 환경에서는 어떤 방식으로 테라폼을 구성해야할지 알아본다.

<br>

### Terraform 기본 동작 순서 (기본환경)
---

1. **작업 디렉토리 생성**.
2. **Provider 정의** : provider을 선언하여 tf파일을 구성.
3. **Terraform 파일 작성** : HCL 언어로 구성된 tf 파일 작성 
4. **테라폼 프로젝트 초기화** : terraform init 명령어 입력 → 테라폼은 해당 명령어를 입력할 경우 프로바이더 설정을 보고 필요한 플러그인들을 설치한다. 
5. **선언한 리소스들 생성 가능 여부 Plan을 통해 확인** : terraform plan 명령어 입력 → 실제 작업이 수행되기 전에 계획을 보여준다.
6. **Plan에 오류가 없으면 작업 수행** : terraform apply 명령어 입력 → 실제 작업이 이루어진다.

⇒ 작업범위가 디렉토리 기반이다. terraform apply 작업을 수행할 때 한 디렉토리 안에있는 모든 tf파일의 내용을 적용시킨다.

<br>

### 온프레미스에서 테라폼을 사용하기 위해서
---

- 현재 프로젝트에서 사용한 방법을 토대로 작성.
- Terraform Enterprise를 사용한다면 아래의 방법보다 간단하게 작업환경을 구축할 수 있을 것.
1. 테라폼 설치 온프레미스 환경에서 진행
2. **인터넷이 안 될 경우, 동작순서 4번에서 테라폼 프로젝트를 초기화 할 때 프로바이더와 모듈을 받아올 수 없다. → 필요한 프로바이더를 직접 저장소에 담아가서 설치해야한다.**
3. Terraform 환경 설정
- ~/.terraform.d/plugin-cache/registry.terraform.io

    /usr/share/terraform/providers/registry.terraform.io/

    ⇒ 두 개의 디렉토리 구성을 동일하게 해주어야 한다.

    ⇒ 이 디렉토리들 아래에 on-premise 환경에서 사용하려는 provider들을 모두 넣어준다.

- ~/.terraformrc 아래대로 생성

```bash
plugin_cache_dir = "$HOME/.terraform.d/plugin-cache"
disable_checkpoint = true
provider_installation {
	filesystem_mirror {
		path = "/usr/share/terraform/providers"
		include = ["registry.terraform.io/hashicorp/*","registry.terraform.io/rancher/*",.....]
	}
	direct {
		exclude = ["registry.terraform.io/hashicorp/*","registry.terraform.io/rancher/*",.....]
	}
}
```

<br>

### 온프레미스 테스트 결과
---

- EKS module을 인터넷 연결없이 사용하려하니 사용되지 않았다 ( 당연히 EKS에서 정보를 불러오려면 온라인환경이어야 하기 때문 )
- Rancher 프로바이더를 위와 같이 설정한뒤에 tf init으로 초기화를 하니 정상적으로 동작하였다.

<br>
<br>

------------------
**◎ 참고자료**

