---
layout: post
title: Elasticsearch Filebeat 모듈 설정
feature-img: assets/img/titles/elastic-beats-logo.png
thumbnail: assets/img/contents/efm-1.png
author: csupreme19
categories: DevOps Elastic
tags: [Elasticsearch, Elastic, ELK, Filebeat, Beat]

---

# Elastic Filebeat 모듈 설정

![elastic-beats-logo.png]({{ "/assets/img/titles/elastic-beats-logo.png"}})

[Filebeat Modules](https://www.elastic.co/guide/en/beats/filebeat/7.16/filebeat-modules.html)

로그 정보를 수집하는 Filebeat의 모듈별 Output 설정방법을 정리하였다.

---
## Module 설정
---

### Filebeat 설치

[Elastic Filebeat 설치 및 설정](/2021/08/05/elastic-filebeat-install.html) 참고

---
### Module 확인

데이터를 수집하기 위한 filebeat의 module list 확인

```sh
$ filebeat modules list
```
---
### System module 설정
```sh
$ filebeat modules enable system
```

---
### Kafka module 설정

Kafka가 설치되어 있는 VM에 진행

#### 모듈 활성화

```sh
$ filebeat modules enable postgresql
```

#### 모듈 설정

`/etc/filebeat/modules.d/kafka.yml` 수정

```yaml
- module: kafka
  log:
    enabled: true
    
    var.paths:
      - "/data/kafka-logs/controller.log*"
      - "/data/kafka-logs/server.log*"
      - "/data/kafka-logs/state-change.log*"
      - "/data/kafka-logs/kafka-*.log*"
```

> ref: [https://www.elastic.co/guide/en/beats/filebeat/7.x/filebeat-module-kafka.html](https://www.elastic.co/guide/en/beats/filebeat/7.x/filebeat-module-kafka.html)

#### kafka dashboards
![efm-1.png]({{ "/assets/img/contents/efm-1.png"}})

---
### Nginx module 설정

Nginx가 설치되어 있는 VM에 진행

#### 모듈 활성화

```sh
$ filebeat modules enable nginx
```

#### 모듈 설정

`/etc/filebeat/modules.d/nginx.yml` 수정

```yaml
- module: nginx
  # Access logs
  access:
    enabled: true
    var.paths: 
      - "/var/log/nginx/access.log*"

  # Error logs
  error:
    enabled: true
      - "/var/log/nginx/error.log*"
```

> ref: [https://www.elastic.co/guide/en/beats/filebeat/7.x/filebeat-module-nginx.html](https://www.elastic.co/guide/en/beats/filebeat/7.x/filebeat-module-nginx.html)

#### nginx dashboards

![efm-2.png]({{ "/assets/img/contents/efm-2.png"}})

![efm-3.png]({{ "/assets/img/contents/efm-3.png"}})

#### 추가 사항

access.log에 관리자 페이지 관련하여 access_log가 많이 쌓이는 경우

아래와 같이 access_log off; 추가

```shell
$ cd /etc/nginx/conf.d
$ vim management.conf
        location / {
                access_log off;
        }
```

---
### PostgreSQL module 설정

PostgreSQL이 설치되어 있는 VM에 진행

#### 모듈 활성화

```sh
$ filebeat modules enable postgresql
```

#### 모듈 설정

`/etc/filebeat/modules.d/postgresql.yml` 수정

```yaml
- module: postgresql
  log:
    enabled: true
    var.paths:
      - "/data/postgres/pgdata/log/*.log*"
```

> ref: [https://www.elastic.co/guide/en/beats/filebeat/7.x/filebeat-module-postgresql.html](https://www.elastic.co/guide/en/beats/filebeat/7.x/filebeat-module-postgresql.html)

#### postgresql dashboards

![efm-4.png]({{ "/assets/img/contents/efm-4.png"}})

---
### MySQL module 설정

MySQL이 설치되어 있는 VM에 진행

#### 모듈 활성화

```sh
$ filebeat modules enable mysql
```

#### 모듈 설정

`/etc/filebeat/modules.d/mysql.yml` 수정

```yaml
- module: mysql
  error:
    enabled: true
    var.paths:
      - "/data/mysql/logs/error.log*"

  slowlog:
    enabled: true
    var.paths:
      - "/data/mysql/logs/mysql-slow.log*"
```

> ref: [https://www.elastic.co/guide/en/beats/filebeat/7.x/filebeat-module-mysql.html](https://www.elastic.co/guide/en/beats/filebeat/7.x/filebeat-module-mysql.html)

#### mysql dashboards

![efm-5.png]({{ "/assets/img/contents/efm-5.png"}})

---
### MongoDB module 설정

MongoDB가 설치되어 있는 VM에 진행

#### 모듈 활성화

```sh
$ filebeat modules enable mongodb
```

#### 모듈 설정

`/etc/filebeat/modules.d/mongodb.yml` 수정

```yaml
- module: mongodb
  log:
    enabled: true
    var.paths:
      - "/data/mongo/logs/*.log*"
```

> ref: [https://www.elastic.co/guide/en/beats/filebeat/7.x/filebeat-module-mongodb.html](https://www.elastic.co/guide/en/beats/filebeat/7.x/filebeat-module-mongodb.html)

#### mongodb dashboards

![efm-6.png]({{ "/assets/img/contents/efm-6.png"}})

---
### Redis module 설정

Redis가 설치되어 있는 VM에 진행

#### 모듈 활성화

```sh
$ filebeat modules enable redis
```

#### 모듈 설정

`/etc/filebeat/modules.d/redis.yml` 수정

```yaml
- module: redis
  log:
    enabled: true
    var.paths:
      - "/data/redis/redis-server.log*"
```

> ref: [https://www.elastic.co/guide/en/beats/filebeat/7.x/filebeat-module-redis.html](https://www.elastic.co/guide/en/beats/filebeat/7.x/filebeat-module-redis.html)

#### redis dashboards

![efm-7.png]({{ "/assets/img/contents/efm-7.png"}})

---
### Filebeat setup

```shell
# dashboard 생성을 위해 /usr/share/filebeat 이동하여 실행
$ cd /usr/share/filebeat

# -e 옵션으로 error 여부를 stdout 으로 출력. log에서 host에 정상 접속 했는지 확인 가능
$ filebeat setup -e -c /etc/filebeat/filebeat.yml --dashboards
```

setup 완료 후 마지막 문장에 dashboard가 정상 로딩된 것을 확인
![efm-8.png]({{ "/assets/img/contents/efm-8.png"}})

---
## Filebeat 구동

```sh
$ systemctl start filebeat.service
```

## Filebeat log 확인

```sh
$ journalctl -u filebeat
```

---

## Reference

1. [Filebeat Modules](https://www.elastic.co/guide/en/beats/filebeat/7.16/filebeat-modules.html)

