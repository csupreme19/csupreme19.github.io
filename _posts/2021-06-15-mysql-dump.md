---
layout: post
title: MySQL dump & restore
feature-img: assets/img/titles/mysql-logo.png
thumbnail: assets/img/titles/mysql-logo.png
author: csupreme19
categories: DB MySQL
tags: [MySQL, DB, RDBMS, SQL, Dump, Restore]

---

# MySQL dump & restore

![mysql-logo.png]({{ "/assets/img/titles/mysql-logo.png"}})

MySQL 덤프와 복구에 관한 내용을 정리해봤어요.

---

## Dump & Restore

#### Dump

현재 DB의 스키마와 데이터, 상태 등을 sql 쿼리 형태로 추출하는 것

#### Restore

Dump된 결과물의 쿼리를 수행하는 것

이론적으로는 dump 한 DB의 상태가 같아져요.



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



### 2. MySQL Dump
```sh
# --databses: 복구할 db, --add-drop-database: database drop문 포함
$ mysqldump --databases db1 db2 --add-drop-database -h localhost -u root -p{비밀번호} > {파일명}.sql
$ mysqldump --databases mydb --add-drop-database -h localhost -u root -ppassword1234 > mysql_mydb_dump_20210615.sql
```



### 3. MySQL Restore

```sh
$ mysql -h localhost -u root -p{비밀번호} < {파일명}.sql
# 예시
$ mysql -h localhost -u root -ppassword1234 < mysql_mydb_dump_20210615.sql
```

