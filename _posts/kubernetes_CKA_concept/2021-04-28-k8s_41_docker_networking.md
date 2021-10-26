---
layout: single
title: "[CKA Concept] 41. Docker Networking"
categories: [Kubernetes]
author_profile: true
excerpt: CKA 강의의 개념 중 Docker Networking 대해 정리한다. 
toc: true
toc_sticky: true
---

## Docker Networking

### 도커 네트워크의 종류
1.none
- 컨테이너가 내부, 외부 모두 통신할수 없다.
- 같은 컨테이너를 여러개 만들면 통신이 되지않는 컨테이너들이 생성된다.

```bash
docker run --network none nginx
```

2.host
- 호스트의 네트워크와 컨테이너가 붙는다.
- 호스트와 컨테이너 간의 네트워크 단절이 없다.
- 80포트를 사용하는 web어플리케이션 컨테이너를 접속하게하려면 따로 포트매핑 없이 호스트IP의 80포트로 매핑된다.
- 동시에 같은 포트를 사용하는 컨테이너를 여러개 만들 수 없다.

```bash
docker run --network host nginx
```

3.bridge
- 도커 호스트와 컨테이너가 붙은 내부 Private 네트워크이다.
- Default로 이 네트워크는 172.17.0.0의 주소를 갖는다.

```bash
docker run nginx
```

<br>

- Bridge를 도커에서 조회했을 때는 bridge로 조회되지만 호스트에서 조회할경우 docker0이라는 이름을 가진다.

```bash
$ docker network ls
NETWORK ID     NAME     DRIVER     SCOPE
2b60082193bs   bridge   bridge     local
                ....


$ ip link
1: lo: .....
2: enp0s3: ......
3: docker0: ......

# ip addr 조회시 bridge는 172.17.0.1/24 ip주소를 갖는다.
$ ip addr
3: docker0: ....
      ... 
    inet 172.17.0.1/24
```

<br>

- docker 컨테이너를 생성한 뒤에 컨테이너가 bridge에 연결되는 것은 아래와 같은 원리로 구성된다.
    - docker가 컨테이너생성시 도커 네임스페이스를 생성한다.
    - 이 네임스페이스와 Bridge가 연결된다.

![](/assets/img/kubernetes/42_docker_network_2.png)

<br>

- 도커 호스트 내부의 80포트를 외부에서 접속 할 수 있도록 하기위해 포트매핑을 시켜준다.

```bash
docker run -p 8080:80 nginx
```

![](/assets/img/kubernetes/42_docker_network_1.png)

<br>

- 포트매핑을 시켜주는 원리는 아래와 같다.
- iptable의 nat을 설정하여 구성한다.

![](/assets/img/kubernetes/42_docker_network_3.png)



<br>
<br>


------------------
**◎ 참고자료**
- Udemy - Certified Kubernetes Administrator (CKA) with Practice Tests
