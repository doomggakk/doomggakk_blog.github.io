---
layout: single
title: "[Jenkins] 2. Jenkins로 Angular + Springboot 프로젝트 배포 (2)"
categories: [Jenkins]
author_profile: true
excerpt: Jenkins로 Angular + Springboot 프로젝트 빌드 배포에 대해서 정리한다.
toc: true
toc_sticky: true
---

## Jenkins로 Angular + Springboot 프로젝트 배포하기 (2)
- 아래 목차중에서 Naver Cloud에 배포를 하기위한 작업을 진행한다.

1. Docker 설치
2. Jenkins 설치
3. Jenkins Plugin설치 및 빌드환경 설정
4. Jenkins와 github 연동
5. Jenkins로 Spring boot 프로젝트 빌드 설정
6. **Jenkins로 Naver Cloud에 배포**

<br>

### Jenkins로 Naver Cloud에 배포
--------

#### 서버생성 및 SSH 접속 가능하도록 설정
- 네이버 클라우드에서 프로젝트 배포용 서버를 생성한다.
- 서버를 생성한 후에 SSH 접속이 가능하도록  ACG를 생성하여 규칙을 설정해준다.
- 공인 IP를 신청하여 서버에 적용시켜준다.
- ssh 접속이 가능한 것을 확인한 뒤 Jenkins에서 설정을 진행한다.

<br>

#### Docker login을 위한 docker ID, PWD 환경변수 설정 및 SSH 설정
- Dashboard - Manage Jenkins - Configure System에 진입한다.
![](/assets/img/jenkins/jenkins11.png)

<br>

- Docker Login을 위한 ID, PWD를 환경변수로 설정해준다.
![](/assets/img/jenkins/jenkins14.png)

<br>


- SSH 통신을 위해 Private Key를 넣어주고 Host와 접속 User를 설정해준다.
![](/assets/img/jenkins/jenkins12.png)
![](/assets/img/jenkins/jenkins13.png)

<br>

#### Jenkins 프로젝트에서 ssh 통신 설정
- 배포할 cloud 서버에 ssh로 통신하여 프로젝트가 자동으로 배포되도록 해야한다.
- freestyle로 생성했던 Jenkins 프로젝트의 Configure에 진입한다.
- Build Environment에서 Send files or execute commands over SSH after the build runs를 체크해준다.
- 다음의 작업을 진행하도록 docker 명령어를 넣어준다.
1. 기존 실행되던 docker 컨테이너 stop
2. stop한 컨테이너 삭제
3. 기존에 사용했던 docker 이미지 삭제
4. 새로 생성된 이미지로 컨테이너 재 생성
![](/assets/img/jenkins/jenkins10.png)

<br>

### 마치며
--------
- 번거로웠던 프로젝트 빌드, 배포작업이 git push + 젠킨스 빌드버튼 클릭 만으로 수행할 수 있게 되었다.
- 젠킨스를 로컬환경의 VM에 구성하다보니 git webhook 기능을 적용하지는 못하였다.(가능은 함)
- webhook을 적용하면 git에 push만 하면 빌드와 배포까지 자동으로 수행될 것이다.
- 추후에 webhook 작업까지 진행할 예정이다.
- 끝


<br>
<br>

------------------
**◎ 참고자료**

[우분투 18.04 도커(Docker) 설치 방법 - 코스모스팜 블로그](https://blog.cosmosfarm.com/archives/248/%EC%9A%B0%EB%B6%84%ED%88%AC-18-04-%EB%8F%84%EC%BB%A4-docker-%EC%84%A4%EC%B9%98-%EB%B0%A9%EB%B2%95/)
