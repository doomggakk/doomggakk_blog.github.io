---
layout: single
title: "[Docker] 3. Springboot Link mySQL"
categories: [Docker]
author_profile: true
excerpt: Doker 강의 개념 중 Springboot Link mySQL에 대해 정리한다. 
toc: true
toc_sticky: true
---

## Springboot Link mySQL
- 1년전에 만들었던 작업물 중에 간단한 웹어플리케이션이 있다.
- Spring boot + Angular 로 이루어진 어플리케이션인데 문제가 많은 작업물이긴 하지만 Cloud서버에 도커를 배포해보고 싶어 시도해보았다.
- Cloud 서버는 Naver Cloud로 정하였다.

<br>

### Spring boot 설정

```properties

spring.datasource.url = jdbc:mysql://myappdb:3306/study_db?&useSSL=false&serverTimezone=Asia/Seoul&characterEncoding=UTF-8&allowPublicKeyRetrieval=true&autoReconnect=true
spring.datasource.username = <username>
spring.datasource.password = <password>
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver

```
- DB설정을 하고 Link를 거는데 작업이 너무 더뎌졌다. 설정에 문제가 있어 Mysql 컨테이너와 연결이 되지 않았다.
- 최종적으로 ```jdbc:mysql://<db 컨테이너 이름>:3306/<사용 db name>```순서로 입력하였더니 정상적으로 연결이 되었다.

<br>

### Docker hub Push & Pull
- Docker file내용은 아래와 같이 구성하였다.

```docker
FROM openjdk:8-jdk-alpine
VOLUME /tmp
ARG JAR_FILE=target/angular-springboot.jar
EXPOSE 8080
COPY ${JAR_FILE} app.jar
ENV JAVA_OPTS=""
ENTRYPOINT ["sh","-c" ,"java $JAVA_OPTS -Djava.security.egd=file:/dev/./urandom -Dspring.profiles.active=dev -jar app.jar"]
```

- 해당 도커파일을 docker hub에 Push

```shell
docker build -t <원하는 이미지 이름> .    # Dockerfile 위치에서 작업해야함
ex) docker build -t myimage

docker tag <이미지 이름> <docker hub 계정 / 저장소이름:태그>
ex) docker tag myimage won3756/myimage:1.0

docker push <태그에서 지정한 docker hub 계정 / 저장소이름:태그>
ex) docker push won3756/myimage:1.0
```

- 네이버 클라우드 서버에서 Pull 받기

```shell
docker pull <Pull받을 docker hub 계정 / 저장소이름:태그>
ex) docker pull won3756/myimage:1.0
````

<br>

### Mysql에서 Springboot에서 접속할 user 등록및 권한 추가

```sql
# mySQL 컨테이너 생성
docker run -d --name <컨테이너 이름> -e MYSQL_ROOT_PASSWORD=<password> -p 3306:3306 -e MYSQL_DATABASE=<사용 db> mysql
ex) docker run -d --name myappdb -e MYSQL_ROOT_PASSWORD=111 -p 3306:3306 -e MYSQL_DATABASE=my_db mysql

mysql> create user '사용자'@'host' identified by '비밀번호';
ex) mysql> create user 'root'@172.17.0.3 identified by '123123';

mysql> grant all privileges on *.* to '사용자'@'localhost'; # 모든 DB 접속권한
mysql> grant all privileges on DB이름.* to '사용자'@'localhost'; # 해당 DB만 접속권한
ex) mysql> grant all privileges on *.* to 'root'@172.17.0.3;

```

<br>
### Spring Boot 컨테이너와 Mysql 컨테이너 링크

```shell
docker run -d --name myapp --link myappdb:mysql -p 80:8080 won3756/myimage:1.0 # 위에서 Pull했던 Docker image

```

- 접속을 해보면 정상적으로 build되어 실행된다.

![](/assets/img/docker/3_docker_1.png)