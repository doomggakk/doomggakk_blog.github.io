---
layout: single
title: "[Docker] 2. Docker 기본 환경구성"
categories: [Docker]
author_profile: true
excerpt: Doker 강의 개념 중 Docker의 기본환경구성에 대해 정리한다. 
toc: true
toc_sticky: true
---

## Docker의 기본 환경구성

- 기본 세팅

```shell
~$ sudo su - 
~$ ifconfig
~$ apt-get update
~$ apt-get -y install net-tools

~$ apt-get -y install openssh-server
~$ /etc/init.d/ssh start
~$ apt-get install vim

~$ /etc/init.d/ssh restart
~$ ping <ip>
~$ exit

#원격접속 moba
```

- 와이파이로 접속하는 경우 Host-only network가 원활하게 연결 되지 않는 문제가 발생한다.
- 아래처럼 설정을 해주니 정상적으로 접속이 된다.
![](/assets/img/docker/2_docker_1.png)

<br>


- Linux에 설치

```shell
#apt 업데이트
sudo apt-get update

# 패키지 설치
~$ sudo apt-get install -y \
apt-transport-https \
ca-certificates \
curl \
software-properties-common

# GPG키 추가
~$ curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
OK

# GPG키 확인
~$ sudo apt-key fingerpring 0EBFCD88

pub   rsa4096 2017-02-22 [SCEA]
      9DC8 5822 9FC7 DD38 854A  E2D8 8D81 803C 0EBF CD88
uid           [ unknown] Docker Release (CE deb) <docker@docker.com>
sub   rsa4096 2017-02-22 [S]

# 리포지토리 등록
~$ sudo add-apt-repository \
"deb [arch=amd64] https://download.docker.com/linux/ubuntu \
$(lsb_release -cs) \
stable"

~$ sudo apt-get update

# 도커 설치하기
~$ sudo apt-get -y install docker-ce

~$ sudo docker version

# sudo없이 docker 사용하기
~$ sudo usermod -aG docker jeff
~$ sudo systemctl enable docker
~$ sudo systemctl restart docker
~$ sudo reboot

~$ docker version # sudo 없이 사용 가능하다면 확인가능
```

- docker version 입력

```shell
jeff@hostos1:~$ docker version

Client: Docker Engine - Community
 Version:           20.10.5
 API version:       1.41
 Go version:        go1.13.15
 Git commit:        55c4c88
 Built:             Tue Mar  2 20:18:05 2021
 OS/Arch:           linux/amd64
 Context:           default
 Experimental:      true

Server: Docker Engine - Community     # dockerd(daemon) 부분
 Engine:                              # dockerd는 소켓통신함
  Version:          20.10.5
  API version:      1.41 (minimum version 1.12)
  Go version:       go1.13.15
  Git commit:       363e9a8
  Built:            Tue Mar  2 20:16:00 2021
  OS/Arch:          linux/amd64
  Experimental:     false
 containerd:
  Version:          1.4.4
  GitCommit:        05f951a3781f4f2c1911b05e61c160e9c30eaa8e
 runc:
  Version:          1.0.0-rc93
  GitCommit:        12644e614e25b05da6fd08a38ffa0cfe1903fdec
 docker-init:
  Version:          0.19.0
  GitCommit:        de40ad0
```

<br>

- ifconfig 입력 확인

```shell

# docker info
jeff@hostos1:~$ ifconfig

docker0: flags=4099<UP,BROADCAST,MULTICAST>  mtu 1500
        inet 172.17.0.1  netmask 255.255.0.0  broadcast 172.17.255.255
        ether 02:42:69:af:3b:ee  txqueuelen 0  (Ethernet)
        RX packets 0  bytes 0 (0.0 B)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 0  bytes 0 (0.0 B)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
# docker0이 생성되었다.
...
```

<br>

- docker info 명령어 입력

```shell
~$ docker info  # docker 정보 명령

...

Server:
 Containers: 0
  Running: 0
  Paused: 0
  Stopped: 0
 Images: 0
 Server Version: 20.10.5
 Storage Driver: overlay2
  Backing Filesystem: extfs
  Supports d_type: true
  Native Overlay Diff: true
 Logging Driver: json-file
 Cgroup Driver: cgroupfs
 Cgroup Version: 1
 Plugins:
  Volume: local
  Network: bridge host ipvlan macvlan null overlay # host to host = overlay
  Log: awslogs fluentd gcplogs gelf journald json-file local logentries splunk syslog
 Swarm: inactive
 Runtimes: io.containerd.runc.v2 io.containerd.runtime.v1.linux runc
 Default Runtime: runc
 Init Binary: docker-init
 containerd version: 05f951a3781f4f2c1911b05e61c160e9c30eaa8e
 runc version: 12644e614e25b05da6fd08a38ffa0cfe1903fdec
 init version: de40ad0
 Security Options:
  apparmor
  seccomp
   Profile: default
 Kernel Version: 5.0.0-23-generic
 Operating System: Ubuntu 18.04.3 LTS
 OSType: linux
 Architecture: x86_64
 CPUs: 1
 Total Memory: 7.788GiB
 Name: hostos1
 ID: O7PV:B42W:WUEK:6C56:OE6U:XQEY:QXTR:GYGF:MRJ5:3NJM:NPZC:NNDV
 Docker Root Dir: /var/lib/docker      # 이미지가 쌓임
 Debug Mode: false
 Registry: https://index.docker.io/v1/ # hub.docker.com internal 주소
 Labels:                               # docker pull 명령시 이미지 가져오는 주소임
 Experimental: false
 Insecure Registries:
  127.0.0.0/8
 Live Restore Enabled: false
```

<br>

- docker 우분투 설치 (Shell script 방식)

```
# docker에서 제공하는 shell script을 이용한 자동 설치
~$ curl -fsSL https://get.docker.com -o get-docker.sh

# shell script 내용 확인 후 변경 가능
~$ sudo vi get-docker.sh

# 실행 권한 부여
~$ chmod +x get-docker.sh

# 설치
~$ sudo sh get-docker.sh
```

<br>

- C컨테이너 모니터링 도구 CAdvisor

```
~$ docker run \
--volume=/:/rootfs:ro \
--volume=/var/run:/var/run:rw \
--volume=/sys:/sys:ro \
--volume=/var/lib/docker:/var/lib/docker:ro \
--publish=9559:8080 \
--detach=true \
--restart=always \
--name=cadvisor \
google/cadvisor:latest


~$ sudo netstat -nlp | grep 9559 # proxt 잘 연결됐나 확인

192.168.56.101:9559로 크롬 접속
```