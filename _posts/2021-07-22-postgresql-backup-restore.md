---
layout: post
title: PostgreSQL 백업 및 복구 방법
feature-img: assets/img/titles/postgresql-logo.svg
thumbnail: assets/img/titles/postgresql-logo.svg
author: csupreme19
categories: DB PostgreSQL
tags: [PostgreSQL, Postgres, RDBMS, DB, Backup, Restore]

---

# PostgreSQL 백업 및 복구 방법

![postgresql-logo.svg]({{ "/assets/img/titles/postgresql-logo.svg"}})

[Continuous Archiving](https://www.postgresql.org/docs/14/continuous-archiving.html)

---

## 백업

### 백업 방식

Postgres 백업 방식은 [Continuous Archiving](https://www.postgresql.org/docs/14/continuous-archiving.html) 방식으로 두 가지의 파일을 백업하고 있다.

> [PostgreSQL WAL 아카이브 백업](/2021/06/26/postgresql-archive-backup.html)

1. `basebackup`

   postgresql data 파일 자체 백업
   
   이후 WAL 파일을 이용하여 복구시 기준이 되는 postgres 파일이다.

2. `pg_wal`

   postgres WAL(write ahead log) 미리쓰기 파일로 수행된 모든 쿼리를 담고 있다.

### 백업 방법

1. pg_wal 자동 archiving 설정

`postgresql.conf` 설정파일 수정
```shell
wal_level = replica
archive_mode = on
archive_command = 'test ! -f /data/archivedir/%f && cp %p /data/archivedir/%f' # %p: 아카이브 폴더, %f: 아카이브할 파일명
```

`archive_command` 명령어는 아카이빙 시점에 실행되는 커맨드로 0을 리턴해야함

이후 postgres restart가 필요

> config reload 수행하여 설정파일을 reload하는 방식은 모든 셋팅에 적용되는 건 아님<br>위의 `wal_level`, `archive_mode` setting은 restart 필요

postgresql restart

```shell
$ systemctl restart postgresql@11-main.service
```

만약 `archive_command`만 수정되었다면 postgres restart 필요 없이 아래 쿼리로 설정 reload 가능

```sql
postgres=# SELECT pg_reload_conf();
 pg_reload_conf
----------------
 t
(1 row)
```

2. pg_basebackup(수동)

백업의 기준이 되는 basebackup 파일은 `pg_basebackup` 명령어로 수동 생성하여야 한다.

```sh
$ su - postgres
$ /usr/bin/pg_basebackup -D /data/basebackup_$(date +%Y%m%d%H%M%S) -v -Ft -z -P -X stream
# -v: verbose -Ft: format tar -z: gzip -P: Enable progress reporting -X stream: wal method
$ cp -R /data/basebackup_(생성시간) /mnt/postgresql
$ chown -R postgres:postgres /mnt/postgresql/basebackup_(생성시간)
```

백업 output 폴더는 비어있거나 존재하지 않아야하며 db user(`postgres`) permission이 있어야 한다.

LOCAL 백업 -> NAS 백업 cronjob 생성
```shell
$ su postgres
$ cd ~
$ vim archivedir_backup.sh
#!/bin/sh

LOCAL_DIR='/data/archivedir'
NAS_DIR='/mnt/postgresql'

echo "cp LOCAL archivedir to NAS archivedir"
cp --verbose -n -R $LOCAL_DIR $NAS_DIR
echo "done."

$ crontab -e
# m h  dom mon dow   command
0 2 * * * /var/lib/postgresql/archivedir_backup.sh > /var/lib/postgresql/archivedir_backup.sh.log
```

---

## 복구(Postgresql 12 이전)

Postgresql 12 이전 버전에서는`recovery.conf`를 사용하지만 12버전부터는 `postgresql.conf`와 `recovery.signal` 사용

1. 서버 중지
   ```shell
   $ systemctl stop postgresql@11-main.service
   ```

   shutdown 종류에 관한 설명은 [Postgresql shutdown ](/2021/08/06/postgresql-shutdown-mode)참고

2. 용량이 충분하다면 data 디렉토리 아니면 `pg_wal` 디렉토리만이라도 백업
   ```shell
   $ cp /var/lib/postgresql/11/main /var/lib/postgresql/11/main_$(date +%Y%m%d%H%M%S)
   ```
3. data 하위 파일 모두 제거
   ```shell
   $ rm -rf /var/lib/postgresql/11/main/*	# 사용시 주의
   ```
4. 기존에 백업한 data 백업 파일 복구(복사)

   db user(`postgres`)로 permission이 있어야함
   ```shell
   $ cd /mnt/postgresql/basebackup
   $ tar -xvzf base.tar.gz -C /var/lib/postgresql/11/main
   ```
5. `pg_wal` 하위 파일 모두 삭제
   
   ```shell
   $ rm -rf /var/lib/postgresql/11/main/pg_wal/*  # 사용시 주의
   ```
6. data 하위 `recovery.conf` 생성
   ```shell
   $ vim /var/lib/postgresql/11/main/recovery.conf
   
   restore_command = 'cp /mnt/postgresql/archivedir/%f %p'		# %f: archive file, %p: pg_wal path
   recovery_target_time = '2021-07-22 10:53:58'	# Optional
   ```
   `recovery_target_time`은 복구 시점(해당 시점으로 DB가 복구된다)
   
   `recovery_target` 미지정시 마지막 WAL 레코드까지 복구 수행함
   
   자세한 내용은 [Recovery Target Settings](https://www.postgresql.org/docs/11/recovery-target-settings.html) 참고
7. 서버 시작시 `recovery.conf` 파일을 감지하고 복구 모드에 돌입함
   
   복구 완료시 `recovery.conf`를 `recovery.done`으로 파일명 자동 변경
8. 복구 완료되었는지 체크
   1. `recovery.conf` 파일명이 `recovery.done`으로 변경되어 있는지 확인
   
      `recovery.conf` 파일이 존재하면 Postgresql이 복구 모드에 돌입하여 모든 transaction이 read-only 상태이다.
   2. DB 쿼리 조회하여 데이터가 해당 시점으로 복구가 되었는지 확인
   
> 복구시 사용 가능한 WAL segment들을 사용하므로 마지막에는 `file not found` 메세지가 나오는데 이는 정상이다.

---

## 복구(Postgresql 12 이상)

Postgresql 12 이전 버전에서는`recovery.conf`를 사용하지만 12버전부터는 `postgresql.conf`와 `recovery.signal` 사용

> Docker Container 환경

1. 서버 중지

   ```shell
   $ docker-compose stop postgres
   ```
2. 용량이 충분하다면 data 디렉토리 아니면 `pg_wal` 디렉토리만이라도 백업
   ```shell
   $ cp /data/postgres/pgdata /data/postgres/pgdata_$(date +%Y%m%d%H%M%S)
   ```
3. data 하위 파일 모두 제거
   ```shell
   $ rm -rf /data/postgres/pgdata/*	# 사용시 주의
   ```
4. 기존에 백업한 basebackup 파일 복구(복사)

   db user(`postgres`)로 permission이 있어야함
   ```shell
   $ cd /data/postgres/basebackup
   $ tar -xvzf base.tar.gz -C /data/postgres/pgdata
   ```
5. `pg_wal` 하위 파일 모두 삭제
   ```shell
   $ rm -rf /data/postgres/pgdata/pg_wal/*		# 사용시 주의
   ```
6. data 하위 `postgresql.conf` 수정
   ```shell
   $ vim /data/postgres/pgdata/postgresql.conf
   
   # docker container 내부 경로를 사용
   restore_command = 'cp /var/lib/postgresql/data/archivedir/%f %p'		# %f: archive file, %p: pg_wal path
   recovery_target_time = '2021-08-05 21:00:00'	# Optional
   ```
   `recovery_target_time`은 복구를 원하는 시간(해당 시점으로 DB가 복구된다)
   
   `recovery_target` 미지정시 마지막 WAL 레코드까지 복구 수행함
   
   자세한 내용은 [Recovery Target Settings](https://www.postgresql.org/docs/11/recovery-target-settings.html) 참조
7. `recovery.signal` 생성
   ```shell
   $ touch /data/psotgres/pgdata/recovery.signal
   ```
7. 서버 시작시 `recovery.signal` 파일을 감지하고 복구 모드에 돌입함

   복구 완료시 `recovery.signal`을 자동 삭제
8. 복구 완료되었는지 체크
   1. `recovery.signal` 파일이 지워졌는지 확인, 안지워졌으면 지워준다.
   
      `recovery.signal` 파일이 존재하면 Postgresql이 복구 모드에 돌입하여 모든 transaction이 read-only 상태이다.
   2. `postgresql.conf` 복구 관련 설정 주석 처리
      
      ```shell
      # restore_command = 'cp /var/lib/postgresql/data/archivedir/%f %p'
      # recovery_target_time = '2021-08-05 21:00:00'
      ```
      복구 모드가 아니라면 해당 명령어 실행되지 않지만 만약을 대비하여 주석처리
   3. DB 쿼리 조회하여 데이터가 해당 시점으로 복구가 되었는지 확인

> 복구시 사용 가능한 WAL segment들을 사용하므로 마지막에는 `file not found` 메세지가 나오는데 이는 정상이다.

---

## 복구 시점 이해

| Base Backup Label | WAL Archive       | 복원 가능 | 설명                          |
| ----------------- | ----------------- | --------- | ----------------------------- |
|                   | 001               | X         | 베이스 백업 없어서 복구 불가  |
|                   | 002               | X         | 베이스 백업 없어서 복구 불가  |
|                   | 003               | X         | 베이스 백업 없어서 복구 불가  |
| basebackup_004    | 004<br>004.backup | O         | 004.backup 이후 WAL 복원 가능 |
|                   | 005               | O         | 004.backup 이후 WAL 복원 가능 |
|                   | 006               | O         | 004.backup 이후 WAL 복원 가능 |
|                   | 007               | O         | 004.backup 이후 WAL 복원 가능 |
| basebackup_008    | 008<br>008.backup | O         | 008.backup 이후 WAL 복원 가능 |
|                   | 009               | O         | 008.backup 이후 WAL 복원 가능 |

---

## Troubleshooting

만약 복구 수행시 오류가 난다면 꼬일 가능성이 높기 때문에

복구를 처음부터 재수행 하는 것을 권장한다.

### max_connections is a lower setting

```shell
postgres    | 2021-08-05 22:58:17.453 KST [27] FATAL:  hot standby is not possible because max_connections = 100 is a lower setting than on the master server (its value was 200)
```

basebackup 수행 이후로 max_connections 설정이 늘어난 경우로 `postgresql.conf`의 max_connections를 해당 수치로 수정

### archive command fail

```shell
postgres    | 2021-08-05 23:30:28.801 KST [31] LOG:  archive command failed with exit code 1
postgres    | 2021-08-05 23:30:28.801 KST [31] DETAIL:  The failed archive command was: test ! -f /var/lib/postgresql/data/archivedir/0000000100000000000000E8 && cp pg_wal/0000000100000000000000E8 /var/lib/postgresql/data/archivedir/0000000100000000000000E8
```

복구 이후 아카이브 되는 시점에 문제가 발생하는 경우로 복구 이후에 모종의 이유로 꼬였을 때 나는 오류이다.

충돌이 발생하는 파일을 archivedir에서 제거하여 해결(pg_wal에 있는 WAL이 다시 복사되어 문제 없음)

---

## Reference

1. [Continuous Archiving](https://www.postgresql.org/docs/14/continuous-archiving.html)
2. [Recovery Target Settings](https://www.postgresql.org/docs/11/recovery-target-settings.html)