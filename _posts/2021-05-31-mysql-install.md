---
layout: post
title: MySQL 설치 및 사용자 생성
feature-img: assets/img/titles/mysql-logo.png
thumbnail: assets/img/titles/mysql-logo.png
author: csupreme19
categories: DB MySQL
tags: [MySQL, DB, RDBMS, Docker, Docker-compose]

---

# MySQL 설치 및 사용자 생성

![mysql-logo.png]({{ "/assets/img/titles/mysql-logo.png"}})

MySQL을 docker-compose로 설치 후 사용자 생성, 권한 설정을 진행한 내용이다.

---

## MySQL 설치

### 1. 호스트 볼륨 생성
```sh
$ mkdir -p /data/mysql
```

### 2. `docker-compose.yaml` 작성
접속 port는 13306(docker 외부) - 3306(docker 내부) 로 설정.

```sh
$ vim docker-compose.yaml
```

```yaml
version: '3.1'
services:
  mysql:
    image: mysql/mysql-server:8.0
    restart: always
    container_name: mysql
    ports:
      - 13306:3306		# 외부 port : 내부 port
    environment:
      TZ: Asia/Seoul
      MYSQL_ROOT_PASSWORD: "root"
    command: 
      - --character-set-server=utf8
      - --collation-server=utf8_unicode_ci
    volumes:
      - /data/mysql:/var/lib/mysql	# 호스트 볼륨 : 컨테이너 볼륨
```

### 3. MySQL 구동

```sh
$ docker-compose up -d mysql

$ docker-compose ps mysql
Name               Command                  State                           Ports
-----------------------------------------------------------------------------------------------------
mysql   /entrypoint.sh --character ...   Up (healthy)   0.0.0.0:13306->3306/tcp, 33060/tcp, 33061/tc

$ docker-compose logs mysql
Attaching to mysql
mysql       | [Entrypoint] MySQL Docker Image 8.0.25-1.2.3-server
mysql       | [Entrypoint] Starting MySQL 8.0.25-1.2.3-server
```
---

### MySQL 설정

### 1. MySQL 컨테이너 접속

- #### Using docker

```sh
# container id 확인
$ docker ps | grep -i mysql
cd1ae64000ec   mysql/mysql-server:8.0   "/entrypoint.sh --ch…"   3 months ago   Up 3 months (healthy)   33060-33061/tcp, 0.0.0.0:13306->3306/tcp   mysql

# 컨테이너 접속
$ docker exec -it cd1ae64000ec /bin/bash

# 접속 확인
bash-4.4#
bash-4.4# whoami
root
```



- #### Using docker-compose

```sh
# 컨테이너 접속
$ docker-compose exec mysql /bin/bash

# 접속 확인
bash-4.4#
bash-4.4# whoami
root
```



### 2. MySQL 사용자 생성

MySQL 계정의 경우 계정명과 호스트가 같이 있다.

위처럼 localhost로 host 생성하는 경우 localhost에서만 접속 가능(외부 접속 불가)하므로 외부 접속을 위한 계정을 생성해보자.



#### 1. MySQL 로그인

```sh
bash-4.4# mysql -u root -p
Enter password:

Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 1600691
Server version: 8.0.25 MySQL Community Server - GPL

Copyright (c) 2000, 2021, Oracle and/or its affiliates.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql>
```



#### 2. 사용자 생성



```sh
# 사용자 확인
mysql > select host, user from mysql.user;
+-----------+------------------+
| host      | user             |
+-----------+------------------+
| localhost | mysql.infoschema |
| localhost | mysql.session    |
| localhost | mysql.sys        |
| localhost | root             |
+-----------+------------------+

# 전체 host에서 접속 가능한 사용자 생성
# create user {사용자명}@{호스트} identified by {비밀번호};
mysql > create user 'csupreme19'@'%' identified by 'password1234';

# 참고) 단일 IP host의 경우
mysql > create user 'csupreme19'@'127.0.0.1' identified by 'password1234';
# 참고) IP 대역 host의 경우
mysql > create user 'csupreme19'@'127.%' identified by 'password1234';

# 사용자 확인
mysql > select host, user from mysql.user;
+-----------+------------------+
| host      | user             |
+-----------+------------------+
| %         | csupreme19       |
| localhost | mysql.infoschema |
| localhost | mysql.session    |
| localhost | mysql.sys        |
| localhost | root             |
+-----------+------------------+

# 접속 확인
$ mysql -h 127.0.0.1 -u csupreme19 -p
```



#### 3. 권한 부여

```sh
# 스키마 권한 추가
# myschema 스키마의 모든 테이블
mysql > grant all privileges on myschema.* to csupreme19@'%';
# 전체 스키마
mysql > grant all privileges on *.* to csupreme19@'%';

# 권한 적용 필수
mysql> flush privileges;
```



