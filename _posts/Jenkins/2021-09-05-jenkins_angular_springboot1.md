---
layout: single
title: "[Jenkins] 1. Jenkins로 Angular + Springboot 프로젝트 배포 (1)"
categories: [Jenkins]
author_profile: true
excerpt: Jenkins로 Angular + Springboot 프로젝트 빌드 배포에 대해서 정리한다.
toc: true
toc_sticky: true
---

## Jenkins로 Angular + Springboot 프로젝트 배포하기 (1)
- 재미로 만들던 Angular + Springboot 프로젝트가 있었다.
- 빌드와 배포가 매우 번거로웠다.(project build -> image build & push -> naver cloud서버에서 update한 이미지로 재배포 -> 소스코드 git에 push) 
- 젠킨스를 이용하여 빌드와 배포를 자동화하고싶었다.
- 이 작업을 목차순서대로 진행해볼 것이다.

1. Docker 설치
2. Jenkins 설치
3. Jenkins Plugin설치 및 빌드환경 설정
4. Jenkins와 github 연동
5. Jenkins로 Spring boot 프로젝트 빌드 설정
6. Jenkins로 Naver Cloud에 배포

<br>
<br>

### Docker 설치
--------

#### Ubuntu 18.04 Docker 설치

- Jenkins를 Docker로 설치할 것이기 때문에 Docker를 먼저 설치해주어야 한다.
- 아래 명령어를 순서대로 입력해준다.

```bash
sudo apt update

sudo apt install apt-transport-https ca-certificates curl software-properties-common

curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -

sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu bionic stable"

sudo apt update

apt-cache policy docker-ce

sudo apt install docker-ce

# docker 상태 확인
systemctl status docker 
```

<br>
<br>

### Jenkins 설치
--------

- Docker 컨테이너로 설치를 진행한다.

```bash
sudo docker run -d -p 8080:8080 -v /home/jenkins:/var/jenkins_home \
-v /var/run/docker.sock:/var/run/docker.sock -v /usr/bin/docker:/usr/bin/docker \
-u root jenkins/jenkins:lts
```
- -d 옵션 : detached 모드. background에서 컨테이너 실행
- -p 옵션 : port publish 옵션. port번호를 명시해준다. (ex) 8080:80 = 호스트의 8080포트와 컨테이너의 80포트를 매핑시킨다.
- -v 옵션 : volume 옵션
- -v /var/run/docker.sock:/var/run/docker.sock : 젠킨스 컨테이너에서도 도커를 사용할 수 있도록 한다.
- -u 옵션 : 사용자 지정.

<br>
<br>


### Jenkins Plugin설치 및 빌드환경 설정
--------

- docker 컨테이너를 실행시키고 난 뒤 localhost:8080에 접속하면 Jenkins 콘솔로 접근할 수 있다.
- 기본 세팅과 설치를 마친 뒤 Home화면으로 오게 되면 여러가지 설정을 진행해주어야 한다.

<br>

#### Jenkins Plugin 설치
- Manage Jenkins - Manage Plugins - available탭에 들어가서 필요한 plugin을 설치해주어야한다. 설치 후에는 Jenkins를 재시작 해주도록 하자.
1. Docker plugin
2. Git plugin
3. Github
4. Maven Integration Plugin
5. NodeJS Plugin
6. Publish Over SSH

![](/assets/img/jenkins/jenkins1.png)

<br>

#### Jenkins Build 설정
- Manage Jenkins - Global ToolConfiguration에서 설정이 필요하다.
1. Git
2. Maven
3. NodeJS -> Global npm packages to install에 angular 패키지를 추가한다.
4. Docker


![](/assets/img/jenkins/jenkins2.png)

<br>

![](/assets/img/jenkins/jenkins3.png)
![](/assets/img/jenkins/jenkins4.png)
![](/assets/img/jenkins/jenkins5.png)
![](/assets/img/jenkins/jenkins6.png)

<br>
<br>

### Jenkins와 github 연동
--------

- Jenkins Project를 생성한 뒤에 github과 연동을 시켜주어야한다.
- New Item - Freestyle Project를 생성해준다.
- 생성을 하고나면 설정 페이지가 보일것이다. 여기서 빌드설정을 해주어야한다.

<br>

#### Source Code Management
- Git 선택
- Repository URL에 git에서 생성한 token과 함께 url을 입력해주어야한다.
url 형식) https://<token 이름>:<token 값>@<git repository 주소>
![](/assets/img/jenkins/jenkins7.png)

<br>

#### Build
- Build할 때 필요한 설정을 해주어야 한다.
- Build 순서대로 명령어를 정리하여 입력해준다.
1. NodeJS installer를 실행시켜 npm 명령어를 사용할 수 있게 해준다.
2. angular 디렉토리에서 angular 프로젝트를 build해준다.
3. spring boot 디렉토리에서 빌드된 angular프로젝트와 함께 스프링부트 프로젝트를 Build 해준다.
4. 생성된 jar파일을 이용하여 docker image를 생성한다.
5. docker hub에 생성한 image를 push 한다.


![](/assets/img/jenkins/jenkins8.png)
![](/assets/img/jenkins/jenkins9.png)

<br>
### 마치며
- 지금까지 Jenkins를 설치하고, Build 설정까지 마무리 지었다.
- 다음 게시물에서 나머지 작업을 이어서 진행하도록 하겠다.
- 주로 배포와 관련된 작업들이 진행될 것이다.

<br>
<br>

------------------
**◎ 참고자료**

[우분투 18.04 도커(Docker) 설치 방법 - 코스모스팜 블로그](https://blog.cosmosfarm.com/archives/248/%EC%9A%B0%EB%B6%84%ED%88%AC-18-04-%EB%8F%84%EC%BB%A4-docker-%EC%84%A4%EC%B9%98-%EB%B0%A9%EB%B2%95/)
