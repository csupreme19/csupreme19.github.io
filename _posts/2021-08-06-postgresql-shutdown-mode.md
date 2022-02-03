---
layout: post
title: PostgreSQL Shutdown mode
feature-img: assets/img/titles/postgresql-logo.svg
thumbnail: assets/img/titles/postgresql-logo.svg
author: csupreme19
categories: DB PostgreSQL
tags: [Postgres, PostgreSQL, RDBMS, DB, Shutdown]

---

# PostgreSQL Shutdown mode

![postgresql-logo.svg]({{ "/assets/img/titles/postgresql-logo.svg"}})

Postgres 셧다운 모드에 대하여 알아보고 모드를 바꿔본다.

---
## PostgreSQL Shutdown Mode

[Shutting Down the server](https://www.postgresql.org/docs/11/server-shutdown.html)

Postgres를 중지하는 방법은 3가지가 있다.

1. #### Smart Shutdown(SIGTERM)
2. #### Fast Shutdown(SIGINT)
3. #### Immediate Shutdown(SIGQUIT)

### Smart Shutdown(SIGTERM)

`pg_ctl shutdown -m s[mart]`

내부적으로 SIGTERM 명령과 동일

pg_ctl shutdown 

새로운 connection 접속을 막고 현재 connection의 쿼리를 모두 수행후 세션이 종료되면 Server shutdown을 진행한다.

가장 안전한 방법이지만 long query가 존재할 경우 서버의 중지가 느릴 수 있음

### Fast Shutdown(SIGINT)

`pg_ctl shutdown -m f[ast]`

내부적으로 SIGINT 명령과 동일

새로운 connection 접속을 막고 현재 connection의 쿼리를 중지(abort)하고 Server shutdown을 진행한다.

수행중인 query는 rollback 되며 서버를 즉시 중지할 수 있는 장점이 있다.


### Immediate Shutdown(SIGQUIT)

`pg_ctl shutdown -m i[mmediate]`

내부적으로 SIGQUIT(SIGKILL) 명령과 동일

쿼리 수행여부와 관계없이 서버를 즉시 kill한다.

일반적인 Server shutdown의 프로세스를 하지 않으므로 서버 재시작시 복구 모드에 돌입한다.

### Shutdown 기본 모드 확인

```shell
$ systemctl cat postgresql@11-main.service
...
[Service]
ExecStart=-/usr/bin/pg_ctlcluster --skip-systemctl-redirect %i start
ExecStop=/usr/bin/pg_ctlcluster --skip-systemctl-redirect -m fast %i stop
ExecReload=/usr/bin/pg_ctlcluster --skip-systemctl-redirect %i reload
...
```

위 예시에서는 설정값이 fast shutdown 모드로 되어 있음을 확인할 수 있다.

### Shutdown 모드 수정

```shell
$ systemctl edit postgresql@11-main.service
[Service]
ExecStop=
ExecStop=/usr/bin/pg_ctlcluster --skip-systemctl-redirect -m smart %i stop
```

override.conf 생성 및 적용 확인

```shell
$ systemctl cat postgresql@11-main.service
# /etc/systemd/system/postgresql@11-main.service.d/override.conf
[Service]
ExecStop=
ExecStop=/usr/bin/pg_ctlcluster --skip-systemctl-redirect -m smart %i stop
```

이후 다시 기본 설정 사용하고 싶으면 `postgresql@11-main.service.d/override.conf` 제거

---

## 참고사항

### 명령어(systemctl)

```shell
$ systemctl [start|stop|restart|status|cat] [service]
$ systemctl start postgresql@11-main.service	# 구동
$ systemctl stop postgresql@11-main.service		# 중지
$ systemctl restart postgresql@11-main.service	# 재기동
$ systemctl status postgresql@11-main.service	# 서비스 상태
$ systemctl cat postgresql@11-main.service		# 서비스 정의
$ journalctl -u postgresql@11-main.service		# 서비스 상세 로그 확인
```


---

## Reference

1. [Shutting Down the server](https://www.postgresql.org/docs/11/server-shutdown.html)