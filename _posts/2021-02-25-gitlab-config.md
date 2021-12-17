---
layout: post
title: GitLab 백업 및 기타 설정 그리고 트러블슈팅
feature-img: assets/img/titles/gitlab-horizontal-color.png
thumbnail: assets/img/contents/gc-1.png
author: csupreme19
categories: DevOps CICD
tags: [Gitlab, Docker, SSH, Git, Backup, Restore, SSL, TLS, SMTP, Nginx]

---

# GitLab 백업 및 기타 설정 그리고 트러블슈팅

![gitlab-horizontal-color.png]({{ "/assets/img/titles/gitlab-horizontal-color.png"}})

[Configuring Omnibus GitLab](https://docs.gitlab.com/omnibus/settings/README.html)

GitLab 백업, SSH, 메일 서버 등 설정

---
## Backup 설정
### Backup 플랜

- 백업 주기: 3일
- 백업 범위(기본값)
  - `db` (database)
  - `uploads` (attachments)
  - `builds` (CI job output logs)
  - `artifacts` (CI job artifacts)
  - `lfs` (LFS objects)
  - `registry` (Container Registry images)
  - `pages` (Pages content)
  - `repositories` (Git repositories data)
- 백업 위치: GitLab 로컬, nas의 nfs
- 백업 실행: 3일마다 새벽 2시
- 보관 주기: 30일

> `gitlab.rb`, `gitlab-secrets.json`과 같은 설정파일들은 기본적으로 백업되지 않으며 필요시 별도로 백업 실행

### 수동 Backup

#### 1. 백업 실행

```sh
$ docker exec -it gitlab-ce bash # -t: 실행 쉘 확인 용도
$ gitlab-backup create

# 12.1 이하 버전인 경우
$ gitlab-rake gitlab:backup:create
```



#### 2. 백업 파일 확인

```sh
# cd {docker_volume}/gitlab/data/backups
$ cd /data/docker_volumes/gitlab/data/backups
```

[TIMESTAMP]_gitlab_backup.tar 생성 확인



#### 3. 설정 파일 백업시

```sh
/data/docker_volumes/gitlab/config/gitlab-secrets.json
/data/docker_volumes/gitlab/config/gitlab.rb
```

위 두 파일 별도로 백업할 것



### 수동 Restore

#### 1. 복구 준비

1. ##### 복구된 버전과 복구하려는 gitlab 버전이 같아야함
2. ##### gitlab reconfigure 실행 필요
3. ##### gitlab이 실행되고 있어야함
4. ##### 복구할 파일(.tar) 위치는 /var/opt/gitlab/backups 에 있다고 가정


DB 접근하는 프로세스 종료

```sh
# gitlab shell 진입
$ docker exec -it gitlab-ce bash

# process 정지 처리
$ gitlab-ctl stop unicorn
$ gitlab-ctl stop puma
$ gitlab-ctl stop sidekiq

# 종료 확인
$ gitlab-ctl status 
```

#### 2. 복구 실행

```sh
$ chmod 0644 1612940314_2021_092_10_13.8.1_gitlab_backup.tar	# 모든 사용자 읽기 권한 추가
$ docker exec -t gitlab-ce gitlab-backup restore BACKUP={TIMESTAMP} force=yes

# 12.1 이하 버전인 경우
$ docker exec -t gitlab-ce gitlab-rake gitlab:backup:restore BACKUP={TIMESTAMP} force=yes
```

예를 들어 파일명이 1612940314_2021_092_10_13.8.1_gitlab_backup.tar이라면

[TIMESTAMP]_gitlab_backup.tar 이므로 TIMESTAMP는 1612940314_2021_092_10_13.8.1

복구 시점에 psql must be owner of 관련 오류는 정상임

#### 3. 서비스 재구동 및 점검

```sh
$ docker exec -it gitlab-ce bash
$ gitlab-ctl reconfigure
$ gitlab-ctl restart

$ gitlab-rake gitlab:check SANITIZE=true
```



### 자동 Backup 설정

#### 1. NFS 설정

mount 및 권한 설정은 완료되었다는 가정하에 작성하였습니다.

GitLab 백업 폴더 생성

```sh
# mkdir -p {nfs 볼륨 경로}/gitlab/data/backups
$ mkdir -p /nfs_nas/volumes/gitlab/data/backups
```



#### 2. docker compose 설정

```sh
# docker_compose yaml 설정 파일
$ vim ~/gitlab-ce/docker-compose.yml
```

```yaml
...
    environment:
      GITLAB_OMNIBUS_CONFIG: |
        gitlab_rails['backup_keep_time'] = 2592000
...
```

gitlab_rails 설정값 변경(추가)

- backup_path: 백업 경로(local)
- backup_keep_time: 백업 파일 보관 주기 (2592000초 = 30일)

gitlab이 설치된 volume의  gitlab.rb 설정파일 수정해도 됨



#### 3. 백업 스크립트 작성

```sh
$ vim ~/gitlab-ce/gitlab_backup.sh
```

```sh
 #!/bin/sh
 BACKUP_DIR='/data/docker_volumes/gitlab/data/backups'
 BACKUP_REMOTE_DIR='/nfs_nas1/volumes/gitlab/data/backups'
 BACKUP_REMOTE_DIR2='/nfs_nas2/backups/gitlab'
 BACKUP_LIFETIME=30
 docker exec gitlab-ce gitlab-backup create CRON=1
 BACKUP_FILE=`ls -t $BACKUP_DIR | head -n 1`
 echo "gitlab-backup create::: $BACKUP_DIR/$BACKUP_FILE "

 BACKUP_REMOVE_FILES=`find $BACKUP_DIR -name "*_gitlab_backup.tar" -mtime +$BACKUP_LIFETIME -type f`
 if [ -n "$BACKUP_REMOVE_FILES" ]; then
 echo "gitlab-backup remove old backups::: $BACKUP_REMOVE_FILES"
 rm -rf $BACKUP_REMOVE_FILES
 fi

 BACKUP_REMOTE_REMOVE_FILES=`find $BACKUP_REMOTE_DIR -name "*_gitlab_backup.tar" -mtime +$BACKUP_LIFETIME -type f`
 if [ -n "$BACKUP_REMOTE_REMOVE_FILES" ]; then
 echo "gitlab-backup remove old backups::: $BACKUP_REMOTE_REMOVE_FILES"
 rm -rf $BACKUP_REMOTE_REMOVE_FILES
 fi

 BACKUP_REMOTE_REMOVE_FILES2=`find $BACKUP_REMOTE_DIR2 -name "*_gitlab_backup.tar" -mtime +$BACKUP_LIFETIME -type f`
 if [ -n "$BACKUP_REMOTE_REMOVE_FILES2" ]; then
 echo "gitlab-backup remove old backups::: $BACKUP_RETMOE_REMOVE_FILES2"
 rm -rf $BACKUP_REMOTE_REMOVE_FILES2
 fi

 cp $BACKUP_DIR/$BACKUP_FILE $BACKUP_REMOTE_DIR
 echo "gitlab-backup copy to storage::: $BACKUP_REMOTE_DIR/$BACKUP_FILE $BACKUP_REMOTE_DIR/$BACKUP_FILE"
 cp $BACKUP_DIR/$BACKUP_FILE $BACKUP_REMOTE_DIR2
 echo "gitlab-backup copy to storage::: $BACKUP_REMOTE_DIR2/$BACKUP_FILE $BACKUP_REMOTE_DIR2/$BACKUP_FILE"
 echo "gitlab-backup done."
```

> 위 스크립트는 백업 명령 실행 후 백업 파일을 mv, rm 하는 간단한 스크립트이며 상황별로 다를 수 있음



#### 4. 백업 cron 등록

```sh
$ crontab -e
0 2 */3 * * /root/gitlab-ce/gitlab_backup.sh > /root/gitlab-ce/gitlab_backup.sh.log
```

gitlab 컨테이너 내부에서 crontab 설정도 가능

- CRON=1: 에러가 발생한 경우에만 출력



#### 5. cron 테스트

```sh
$ run-parts /var/spool/cron -v
```


---
## Email 설정(SMTP)

> docker container 환경에서 gitlab의 email 설정

### 사전 준비 사항
  - mail server 정보 확인: gmail 또는 별도 무료 사용 가능한 mail server 확보
  - 본 문서에서는 mailgun 서버를 적용

  - [SMTP Settings#Mailgun](https://docs.gitlab.com/omnibus/settings/smtp.html#mailgun)

### SMTP 설정

#### 1. gitlab의 설정파일 수정
docker-compose.yml의 gitlab environment에 추가 또는 gitlab 설치된 volume에서 gitlab.rb 파일 찾아 수정 

해당 파일 내용 중 mailgun 메일서버 정보를 추가

```shell
# mailgun smtp 설정
gitlab_rails['smtp_enable'] = true
gitlab_rails['smtp_address'] = "smtp.mailgun.org"
gitlab_rails['smtp_port'] = 587
gitlab_rails['smtp_authentication'] = "plain"
gitlab_rails['smtp_enable_starttls_auto'] = true
gitlab_rails['smtp_user_name'] = "mail서버 발급받은 id"
gitlab_rails['smtp_password'] = "암호"
gitlab_rails['smtp_domain'] = "mg.gitlab.com"
```

#### 2. docker container 재기동

#### 3. mail 보내기 테스트

gitlab의 shell로 진입

```shell
$ docker exec -it <container id> bash
```
gitlab-rails console 실행

```shell
$ gitlab-rails console
```
gitlab-rails console 상에서 mail 보내기 테스트

```shell
$ Notify.test_email('<받는 email address>', '<email 제목>', '<email 내용>').deliver_now
```
email이 왔는지 확인

---
## SSH Key 설정

### SSH Key 등록

#### 1. 접속하고자 하는 사용자에서 SSH Key 발급

```sh
$ ssh-keygen -t rsa
```

RSA 키 발급, passphrase는 일반적으로 비워둠(Enter)



#### 2. 발급된 Public key 확인

```sh
chmod 755 ~/.ssh/id_rsa.pub
vi ~/.ssh/id_rsa.pub
```

id_rsa.pub안의 퍼블릭키 복사해두기



#### 3. GitLab 계정에 public key 등록

![gc-1.png]({{ "/assets/img/contents/gc-1.png"}})

User Settings - SSH Keys

id_rsa.pub에 있는 전체 키 내용 복사하여 붙여넣기

Title은 자신이 구분할  수 있는 제목 아무거나

만료일은 해당 키의 만료일로 설정

#### 4. git 계정 설정

```sh
git config --global user.name "{USERNAME}"
git config --global user.email {USER_EMAIL}
```

USERNAME: gitlab 계정명
USER_EMAIL: gitlab 계정의 이메일 주소

#### 5. SSH 접속 확인

```sh
# ssh -T git@{host} -p {port} -v
$ ssh -T git@gitlab.yourdomain.com -p 20722 -v
Welcome to GitLab, @csupreme!
```

-p: 포트 설정

-v: 디버그 로그 확인

처음 접속시 호스트 키 저장 여부를 물으면 yes

#### 6. Git clone 확인

```sh
# git clone ssh://git@{host}:{port}/{repository}.git
git clone ssh://git@gitlab.yourdomain.com:20722/csupreme/commit-test.git
```

별도의 사용자, 암호 입력 없이 SSH Key를 이용하여 clone 확인

---
## Nginx SSL 설정
### 선행사항

- Docker Container 환경(GitLab Omnibus)
- Let's Encrypt Key 발급 완료

### Nginx 설정

#### 1. docker_compose.yml 수정

```yaml
    environment:
      GITLAB_OMNIBUS_CONFIG: |
        letsencrypt['enable'] = false
        external_url 'https://gitlab.yourdomain.com:20443'
        nginx['redirect_http_to_https'] = true
    ports:
        - '20780:20443'
        - '20443:20443'
        - '20722:22'
```

letsencrypt['enable'] = false로 설정하는 이유

gitlab-ctl reconfigure시 자동으로 인증서를 갱신하여 덮어쓰기 때문

수동으로 발급한 인증서이므로 letsencrypt의 자동 갱신 기능을 비활성화 하기 위해

#### 2. SSL 인증서 복사

```sh
# 참고) 컨테이너 /etc/gitlab에 마운트된 경로
$ sudo mkdir -p /data/docker_volumes/gitlab/config/ssl
$ sudo chmod 755 /data/docker_volumes/gitlab/config/ssl
$ cp /etc/letsencrypt/live/yourdomain.com/privkey.pem /data/docker_volumes/gitlab_2/config/ssl
$ cp /etc/letsencrypt/live/yourdomain.com/fullchain.pem /data/docker_volumes/gitlab_2/config/ssl
```

#### 3. PEM 변환

```sh
$ openssl rsa -in privkey.pem -text > gitlab.yourdomain.com.key
$ openssl x509 -inform PEM -in fullchain.pem -out gitlab.yourdomain.com.crt
```

#### 4. GitLab 재설정

```sh
$ docker exec -t gitlab-ce gitlab-ctl reconfigure

# 도커 포트 매핑이 변경된 경우 컨테이너 재실행 필요
$ docker-compose down
$ docker-compose up -d
```

#### 5. Https 접속 확인 및 인증서 확인

```sh
$ cat /data/docker_volumes/gitlab/logs/nginx/gitlab_access.log
```
![gc-2.png]({{ "/assets/img/contents/gc-2.png"}})


---

## Troubleshooting

### 1. SSH Connection은 성공이지만 서버에서 Connection을 닫을 때

![gc-3.png]({{ "/assets/img/contents/gc-3.png"}})

로그 확인

```sh
$ cat /data/docker_volumes/gitlab/logs/sshd/current
```

```sh
2021-02-10_01:55:53.62004 @@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
2021-02-10_01:55:53.62005 @         WARNING: UNPROTECTED PRIVATE KEY FILE!          @
2021-02-10_01:55:53.62005 @@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
2021-02-10_01:55:53.62005 Permissions 0755 for '/etc/gitlab/ssh_host_rsa_key' are too open.
2021-02-10_01:55:53.62006 It is required that your private key files are NOT accessible by others.
2021-02-10_01:55:53.62006 This private key will be ignored.
2021-02-10_01:55:53.62006 key_load_private: bad permissions
2021-02-10_01:55:53.62006 Could not load host key: /etc/gitlab/ssh_host_rsa_key
2021-02-10_01:55:53.62007 @@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
2021-02-10_01:55:53.62007 @         WARNING: UNPROTECTED PRIVATE KEY FILE!          @
2021-02-10_01:55:53.62007 @@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
2021-02-10_01:55:53.62007 Permissions 0755 for '/etc/gitlab/ssh_host_ecdsa_key' are too open.
2021-02-10_01:55:53.62007 It is required that your private key files are NOT accessible by others.
2021-02-10_01:55:53.62007 This private key will be ignored.
2021-02-10_01:55:53.62007 key_load_private: bad permissions
2021-02-10_01:55:53.62014 Could not load host key: /etc/gitlab/ssh_host_ecdsa_key
2021-02-10_01:55:53.62014 @@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
2021-02-10_01:55:53.62015 @         WARNING: UNPROTECTED PRIVATE KEY FILE!          @
2021-02-10_01:55:53.62015 @@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
2021-02-10_01:55:53.62019 Permissions 0755 for '/etc/gitlab/ssh_host_ed25519_key' are too open.
2021-02-10_01:55:53.62019 It is required that your private key files are NOT accessible by others.
2021-02-10_01:55:53.62019 This private key will be ignored.
2021-02-10_01:55:53.62020 key_load_private: bad permissions
2021-02-10_01:55:53.62020 Could not load host key: /etc/gitlab/ssh_host_ed25519_key

```

Private Key 파일 권한이 755로 되어 있어 SSHD에서 자체적으로 Bad Permission을 리턴



#### 권한 수정

```sh
$ chmod 600 ssh_host_ecdsa_key
$ chmod 600 ssh_host_ed25519_key
$ chmod 600 ssh_host_rsa_key
$ chmod 644 ssh_host_ecdsa_key.pub
$ chmod 644 ssh_host_ed25519_key.pub
$ chmod 644 ssh_host_rsa_key.pub
```



#### 접속 및 커밋 확인

```sh
$ ssh -T git@gitlab.yourdomain.com -p 20722
Welcome to GitLab, @csupreme!
```

```sh
$ git clone ssh://git@gitlab.yourdomain.com:20722/csupreme/commit-test.git
```


### 2. ssh 접속시 아래와 같이 Host 정보가 변경되었다고 나올 때

![gc-4.png]({{ "/assets/img/contents/gc-4.png"}})

기존 서버의 IP는 동일하지만 서버의 SHA256 fingerprint가 바뀐 상황

ssh의 known_host에 있는 해당 서버의 RSA Host key키 부분을 삭제한 뒤 접속하면 된다.

```sh
$ vim /Users/{USERNAME}/.ssh/known_hosts
```

[gitlab.yourdomain.com]:20722 부분 삭제 후 재접속

![gc-5.png]({{ "/assets/img/contents/gc-5.png"}})

접속시 바뀐 호스트 키 등록 yes후 진행

---

## Reference

1. [Configuring Omnibus GitLab](https://docs.gitlab.com/omnibus/settings/README.html)
2. [SMTP Settings#Mailgun](https://docs.gitlab.com/omnibus/settings/smtp.html#mailgun)