---
layout: post
title: PostgreSQL max_connections 설정
feature-img: assets/img/titles/postgresql-logo.svg
thumbnail: assets/img/titles/postgresql-logo.svg
author: csupreme19
categories: DB PostgreSQL
tags: [Postgres, PostgreSQL, RDBMS, DB]

---

# PostgreSQL max_connections 설정

![postgresql-logo.svg]({{ "/assets/img/titles/postgresql-logo.svg"}})

[Connection Settings](https://www.postgresql.org/docs/9.4/runtime-config-connection.html)

PostgreSQL max_connections 설정 방법

---
## max_connections Queries

### max_connections 관련 쿼리

```sql
-- 현재 max_connections 수 확인
select * from pg_settings where name = 'max_connections';

-- 현재 connection 정보 확인
select max_conn,used,res_for_super,max_conn-used-res_for_super res_for_normal 
from 
  (select count(*) used from pg_stat_activity) t1,
  (select setting::int res_for_super from pg_settings where name=$$superuser_reserved_connections$$) t2,
  (select setting::int max_conn from pg_settings where name=$$max_connections$$) t3
  
-- 현재 쿼리 정보
select * from pg_stat_activity;

-- 현재 접속되어 있는 idle, active 쿼리 정보 확인
SELECT 
    pid
    ,datname
    ,usename
    ,application_name
    ,client_hostname
    ,client_port
    ,backend_start
    ,query_start
    ,query
    ,state
FROM pg_stat_activity
WHERE state = 'active' or state = 'idle';
```

> 쿼리 출처
>
> > [https://dba.stackexchange.com/a/161761](https://dba.stackexchange.com/a/161761)<br>[https://stackoverflow.com/a/48576319](https://stackoverflow.com/a/48576319)

### max_connections 조정

`/var/lib/postgresql/data/postgres.conf` 수정

```
max_connections = 100
```
원하는 max_connections로 수정

```shell
# postgres 재시작
$ docker-compose restart postgres	# docker-compose의 경우
$ /etc/init.d/postgresql restart	# 설치형의 경우
```


---

## Reference

1. [Connection Settings](https://www.postgresql.org/docs/9.4/runtime-config-connection.html)
2. [https://dba.stackexchange.com/a/161761](https://dba.stackexchange.com/a/161761)
3. [https://stackoverflow.com/a/48576319](https://stackoverflow.com/a/48576319)