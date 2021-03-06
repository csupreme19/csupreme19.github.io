---
layout: post
title: GitLab 구축하기
feature-img: assets/img/titles/gitlab-horizontal-color.png
thumbnail: assets/img/titles/gitlab-horizontal-color.png
author: csupreme19
categories: DevOps CICD
tags: [Gitlab, Docker, SSH, Git]

---

# GitLab 구축하기

![gitlab-horizontal-color.png]({{ "/assets/img/titles/gitlab-horizontal-color.png"}})

[Gitlab Docker Images](https://docs.gitlab.com/omnibus/docker/#install-gitlab-using-docker-compose)

Docker를 활용한 Gitlab 컨테이너 환경의 구축 방법을 정리했어요.

---

## GitLab 구축

### Docker Compose 설치
최신 버전 설치 스크립트는 [docker-compose home](https://docs.docker.com/compose/install/) 에서 확인할 수 있어요.

#### 1. Docker compose stable 설치

```shell
$ curl -L "https://github.com/docker/compose/releases/download/1.28.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
```

![gi-1.png]({{ "/assets/img/contents/gi-1.png"}})

#### 2. 바이너리 실행 권한 설정 및 시스템 바이너리 등록

```shell
$ chmod +x /usr/local/bin/docker-compose
$ ln -s /usr/local/bin/docker-compose /usr/bin/docker-compose
```

#### 3. 설치 및 버전 확인

```sh
$ docker-compose --version
```

![gi-2.png]({{ "/assets/img/contents/gi-2.png"}})


---
### Gitlab 설치

#### 1. GitLab working directory 생성

```sh
$ mkdir ~/gitlab-ce
```



#### 2. docker-compose.yml 작성

```sh
$ vim ~/gitlab-ce/docker-compose.yml
```

```yaml
version: '3'

services:
  gitlab:
    # gitlab 백업 후 복원 시 gitlab의 버전이 일치해야 하기 때문에 버전 고정
    # image: 'gitlab/gitlab-ce:latest'
    image: 'gitlab/gitlab-ce:13.8.4-ce.0'
    restart: always
    container_name: 'gitlab-ce'
    hostname: 'gitlab.yourdomain.com'
    environment:
      TZ: "Asia/Seoul"
      GITLAB_OMNIBUS_CONFIG: |
      external_url 'http://gitlab.yourdomain.com:20780'
      gitlab_rails['time_zone'] = 'Asia/Seoul'
    ports:
      - '20780:20780'	# 외부 포트와 동일하게 매핑
      - '20443:443'
      - '20722:22'
    volumes:
      - '/data/docker_volumes/gitlab/config:/etc/gitlab'
      - '/data/docker_volumes/gitlab/logs:/var/log/gitlab'
      - '/data/docker_volumes/gitlab/data:/var/opt/gitlab'
```

> gitlab wiki의 file attach API 버그로 접속한 포트가 아닌 external_url을 host로 보고 있어 도커 포트 매핑을 동일한 포트로 설정했어요.



#### 3. Docker-compose 실행

**실행**

```sh
$ cd ~/gitlab-ce
$ docker-compose up -d
```

**로그 확인**

```sh
$ docker-compose logs -f
```

**컨테이너 목록 확인**

```sh
$ docker-compose ps -a
```



#### 4. GitLab Web 접속 확인

![gi-3.png]({{ "/assets/img/contents/gi-3.png"}})

초기 접속시 root 계정 비밀번호 설정이 필요해요.


---
### GitLab 설정

#### 1. 사용자 계정 등록 비활성화

![gi-4.png]({{ "/assets/img/contents/gi-4.png"}})

![gi-5.png]({{ "/assets/img/contents/gi-5.png"}})

Admin Area - Settings - General - Sign-up restrictions에서

Sign-up enabled 체크 해제하여 계정 등록을 비활성화하는 것이 좋아요.



#### 2. 저장소 Https Clone 비활성화

![gi-6.png]({{ "/assets/img/contents/gi-6.png"}})

Admin Area - Settings - General - Visibility and access controls

Enabled Git access protocols - Only SSH로 변경 - Save Changes

> Application 레벨에서 막는 것으로 Https 프로토콜 및 포트 접근을 막는 것은 아니에요.


---

### 추가 설정

SSH, Backup, Mail 등 추가 설정은 [GitLab 백업 및 기타 설정 그리고 트러블슈팅]({% post_url 2021-02-25-gitlab-config %}) 문서 참고하세요.

---

## Reference

1. [Gitlab Docker Images](https://docs.gitlab.com/omnibus/docker/#install-gitlab-using-docker-compose)