---
layout: post
title: PostgreSQL pg_ctl vs pg_ctlcluster
feature-img: assets/img/titles/postgresql-logo.svg
thumbnail: assets/img/titles/postgresql-logo.svg
author: csupreme19
categories: DB PostgreSQL
tags: [Postgres, PostgreSQL, RDBMS, DB]

---

# PostgreSQL pg_ctl vs pg_ctlcluster

![postgresql-logo.svg]({{ "/assets/img/titles/postgresql-logo.svg"}})

pg_ctlcluster 명령어에 대하여 알아봤어요.

---
## pg_ctl vs pg_ctlcluster

### pg_ctlcluster

[pg_ctl](https://www.postgresql.org/docs/11/app-pg-ctl.html)

Ubuntu, Debian과 같은 일부 linux에서는 `pg_ctl` 대신 `pg_ctlcluster` 명령어를 제공해요.

위 명령어를 사용하려면 `pg_cluster`를 따로 구성 후에 실행하여야 해요.

클러스터가 구성되어 있는지 확인하려면 아래 명령어 실행하여 확인할 수 있어요.
```shell
$ /usr/bin/pg_lsclusters
Ver Cluster Port Status Owner    Data directory              Log file
11  main    5432 online postgres /data                       /var/log/postgresql/postgresql-11-main.log
13  main    5433 down   postgres /var/lib/postgresql/13/main /var/log/postgresql/postgresql-13-main.log
```

> `pg_ctlcluster`를 강제로 사용하도록 하는 이유는 서로 다른 postgres를 동시에 사용하고 있는 경우가 존재하기 때문이에요.

<br>

### 서비스 형태로의 사용

deb와 같은 패키지 매니저를 이용하여 설치한 경우 `pg_ctl`, `pg_ctlcluster` 명령어 대신 kernel의 systemd에 의해 관리되는 service의 형태로 실행돼요.

> systemctl 명령어를 사용하여 실행하여도 내부적으로 `pg_ctlcluster` 호출하여 실행해요.

#### 명령어

```shell
# 클러스터 구성되어 있는 경우에는 버전 명시
# ex) $ systemctl status postgresql@11-main.service
$ systemctl [start|stop|restart|status|cat] [service]
$ systemctl start postgresql@11-main.service	# 구동
$ systemctl stop postgresql@11-main.service		# 중지
$ systemctl restart postgresql@11-main.service	# 재기동
$ systemctl status postgresql@11-main.service	# 서비스 상태
$ systemctl cat postgresql@11-main.service		# 서비스 정의
```

아래의 명령어를 수행하여 현재 구성되어 있는 postgresql 서비스 정보를 확인할 수 있어요.
```
# 클러스터 구성되어 있는 경우에는 버전 명시
$ systemctl cat postgresql.service
$ systemctl cat postgresql@11-main.service
...
[Service]
Type=forking
# -: ignore startup failure (recovery might take arbitrarily long)
# the actual pg_ctl timeout is configured in pg_ctl.conf
ExecStart=-/usr/bin/pg_ctlcluster --skip-systemctl-redirect %i start
...
```

내부적으로 `pg_ctlcluster` 명령어 호출하도록 설정되어 있음을 확인할 수 있어요.


---

## Reference

1. [pg_ctl](https://www.postgresql.org/docs/11/app-pg-ctl.html)