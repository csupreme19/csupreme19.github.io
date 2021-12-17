---
layout: post
title: PostgreSQL WAL 아카이브 백업
feature-img: assets/img/titles/postgresql-logo.svg
thumbnail: assets/img/titles/postgresql-logo.svg
author: csupreme19
categories: DB PostgreSQL
tags: [PostgreSQL, Postgres, RDBMS, DB, Backup, Restore]

---

# PostgreSQL WAL 아카이브 백업

![postgresql-logo.svg]({{ "/assets/img/titles/postgresql-logo.svg"}})

[Continuous Archiving](https://www.postgresql.org/docs/14/continuous-archiving.html)

WAL(Write Ahead Log) 파일을 아카이브(백업)하고 이를 이용하여 복구하는 백업 방법

---

## WAL Archiving

PostgreSQL은 발생하는 모든 트랜잭션을 pg_wal/이라는 폴더에 미리쓰기로그 write ahead log(WAL) 파일로 저장한다.

데이터베이스의 모든 트랜잭션들에 대한 로그를 가지고 있기 때문에 postgres 장애 발생시 해당 WAL을 읽어서 복구할 수 있음

해당 로그를 다른 서버로 보내서 다른 서버를 동일하게 복구 가능하며 특정시점까지만 복구를 진행하여 특정시점으로 복구 가능

SQL dump, file system 백업보다 복잡하지만 장점이 있다.

### WAL segment files

**저장위치: `pg_wal/`**

WAL 시퀀스를 나누어 저장한 파일(보통 16MB 단위)

파일명은 시퀀스 위치와 WAL 시퀀스를 나타낸다.

WAL 아카이빙을 사용하지 않으면 몇개의 세그먼트만 재활용하면서 생성됨 
따라서 WAL 데이터를 아카이브하려면 WAL 세그먼트 파일을 다른 곳에 저장해야한다.

파일을 다른곳에 저장하는 방법은 따로 있는 것이 아니라 아무 방법이나 사용 가능
(NFS 저장, 테이프 저장장치 기록, CD 굽기 등등)

---
### WAL Archiving 활성화

`postgresql.conf` 설정파일 수정

- `wal_level` 값을 `replica` 이상으로 설정(`replica` or `logical`)
- `archive_mode` `on`
- `archive_command`에 명령어 지정
	```shell
    # 예시
    archive_command = 'test ! -f /mnt/server/archivedir/%f && cp %p /mnt/server/archivedir/%f' # %p: 아카이브 폴더, %f: 아카이브할 파일명
  ```
  

`archive_command` 명령어는 아카이빙 시점에 실행되는 커맨드

아카이브 커맨드 실행 결과가 0이어야 성공

0이 리턴되면 PostgreSQL은 아카이브가 성공인 것으로 간주하고 해당 파일을 지우거나 재사용함

설정값을 적용하려면 postgres restart 필요

#### 유의사항

- 아카이빙시 overwrite 안하도록 설정 overwrite 수행시 0이 리턴되지 않음
- 아카이브 커맨드 성공 결과 꼭 확인할 것 
  
   > (`pg_wal/` 용량이 가득차게 되면 PostgreSQL이 PANIC shutdown됨)
- `postgresql.conf`, `pg_hba.conf`, `pg_ident.conf` 등과 같은 설정 파일은 백업하지 않음
- 아카이브 단위는 WAL segment

#### 기타 설정

`archive_timeout` 

WAL segment가 완성되지 않아도 시간이 지나면 segment 생성하도록 설정 가능, 하지만 segment당 용량은 똑같으므로 짧은 archive timeout 설정은 권장하지 않는다.

`pg_switch_wal`

강제로 세그먼트를 완성시키고 다음 세그먼트를 시작한다.

설정 후 `postgresql.conf` 적용하려면 서버 재시작

---
## Base Backup 생성
### `pg_basebackup` 툴 사용

pgdata 파일을 일반 파일 형태 혹은 tar 아카이브로 백업 파일을 생성

```shell
$ su - postgres
$ /usr/bin/pg_basebackup -D /var/lib/postgresql/data/basebackup -v -Ft -z -P -X stream
```

basebackup 수행시 output 폴더는 빈 폴더 혹은 없는 폴더여야한다.

basebackup 정상 수행 후 0000000100001234000055CD.007C9330.backup과 같은 파일이 생성된다.
({pg_wal}.{exact position}.backup)

이는 백업 시점에서의 pg_wal을 새로 생성하기 위함이다.

**Base backup 이후 시점부터 WAL을 이용한 데이터 복구가 가능**

.backup 파일 이후의 wal 파일을 이용하여 복구하면됨.

> Base backup 수행시 퍼포먼스에 큰 영향은 없으나 `full_page_writes`가 disabled 되어 있다면 속도가 느릴 수 있다.

#### Base backup history

Base backup 수행시 백업 이력 파일을 생성 

파일명: `백업 시점에 있던 WAL segment의 파일명.위치.backup`

이전 시점의 WAL segment는 복구에 사용되지 않으므로 지워도 무관

Base backup 수행 후 다음 Base backup 수행 시점까지의 WAL 파일들이 저장되므로 용량과 의도하는 백업 주기에 맞춰서 base backup 수행 주기 설정할 것

### Low Level API 사용

`pg_basebackup` 툴을 사용하지 않고 API 사용하여 백업하는 방법

non-exclusive한 방법과 exclusive한 방법이 있다.

> exclusive 방법은 postgresql 9.6 이후로 deprecated 되었으므로 non-exclusive만 작성

#### Non-Exclusive Low-Level Backup

다른 백업이 수행 중에도 백업 동시 수행 가능(pg_basebackup, Low-Level API)

1. WAL 아카이빙이 활성화되고 작동되는지 확인
2. postgres 서버 접속(접속 db는 상관없다). 사용자는 크게 상관 없으나 아래 커맨드를 실행하는 권한이 있어야 한다. (e.g., superuser인 postgres)

```sql
SELECT pg_start_backup('label', false, false);
```

- `'label'`: 백업을 구분할 레이블
- 두번째 `false`: `checkpoint_completion_target` 설정으로 체크포인트 간격을 반으로 줄여 I/O 사용량을 줄인다.
  
   > 만약 속도가 매우 느리다면 true로 설정하여 빠르게 할 수 있으나 DB 퍼포먼스에 영향 줄 가능성도 있음
- 세번째 `false`: exclusive 백업 여부

3. tar와 cpio 같은 파일시스템 백업 툴을 이용하여 data 디렉토리 백업 진행
	아래 Backing Up The Data Directory 참고
4. 위 명령어 수행한 같은 커넥션에서 아래 쿼리 실행
```sql
SELECT * FROM pg_stop_backup(false, true);
```
   백업모드를 중지하는 명령어로 다음 WAL segment로 스위치 진행(standby 서버에서는 수동으로 진행해줘야함)

5. 백업 시점에 존재하던 WAL 세그먼트가 아카이브되었다면 끝.

#### Backing Up The Data Directory

`/usr/local/pgsql/data` 하위에 있는 파일/디렉토리들을 모두 백업할 것

백업 진행시 아래 파일들은 생략을 권장한다.

1. `pg_wal/`: 복구 시 리스크 줄이기 위해
2. `postmaster.pid`, `postmaster.opts`: 현재 실행중인 pid 정보이므로 필요 없음
3. `pg_replslot/`: 레플리케이션에 대한 정보를 가지고 있으므로 리스키
4. `pg_dynshmem/`, `pg_notify/`, `pg_serial/`, `pg_snapshots/`, `pg_stat_tmp/`, `pg_subtrans/`: postmaster 실행시 생성되는 정보이므로 생략 (해당 폴더 자체는 있어야하지만 안에 내용물은 생략)
5. `pgsql_tmp`: postmaster 실행시 삭제되는 임시 정보
6. `pg_internal.init`: 캐시 데이터로 복구됨

---

### 복구 시점 이해

| Base Backup Label | WAL  | 복원 가능 | 설명                         |
| ----------------- | ---- | --------- | ---------------------------- |
|                   | 001  | X         | 베이스 백업 없어서 복구 불가 |
|                   | 002  | X         | 베이스 백업 없어서 복구 불가 |
|                   | 003  | X         | 베이스 백업 없어서 복구 불가 |
| basebackup_004    | 004  | O         | 004 이후 WAL 복원 가능       |
|                   | 005  | O         | 004 이후 WAL 복원 가능       |
|                   | 006  | O         | 004 이후 WAL 복원 가능       |
|                   | 007  | O         | 004 이후 WAL 복원 가능       |
| basebackup_008    | 008  | O         | 008 이후 WAL 복원 가능       |
|                   | 009  | O         | 008 이후 WAL 복원 가능       |

### 타임라인

어느 시점으로 복구해야할지 모를때 WAL에 타임라인을 지정하여 사용할 수 있음.

WAL은 타임라인 ID에 해당하는 파일명을 추가로 가지고 있어서 서로 충돌이 나지 않는다.

타임라인을 지정하면 복구 시점에 `recovery_target_timeline`으로 타임라인을 지정하여 해당 타임라인 시점 복구 가능

---

## Reference

1. [Continuous Archiving](https://www.postgresql.org/docs/14/continuous-archiving.html)
2. [WAL Recovery target](https://www.postgresql.org/docs/current/runtime-config-wal.html#RUNTIME-CONFIG-WAL-RECOVERY-TARGET)