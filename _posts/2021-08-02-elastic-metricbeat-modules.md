---
layout: post
title: Elasticsearch Metricbeat 모듈 설정
feature-img: assets/img/titles/elastic-beats-logo.png
thumbnail: assets/img/contents/emm-1.png
author: csupreme19
tags: [Elasticsearch, Elastic, ELK, Metricbeat, Beat]

---

# Elastic Metricbeat 모듈 설정

![elastic-beats-logo.png]({{ "/assets/img/titles/elastic-beats-logo.png"}})

[Metricbeat Modules](https://www.elastic.co/guide/en/beats/metricbeat/current/metricbeat-modules.html)

메트릭 정보를 수집하는 Metricbeat의 모듈별 Output 설정방법을 정리하였다.

---

## Module 설정

---
### Metricbeat 설치

[Metricbeat 설치 및 설정](/2021/07/27/elastic-metricbeat-install.html) 참고


---
### Module 확인
metric을 수집하기 위한 metricbeat의 module들 list 확인
```sh
$ metricbeat modules list
```
---
### System module 설정
```sh
# 필요 시 system 모듈 설정
$ vi /etc/metricbeat/modules.d/system.yml

# 추가 module 없이 활성화를 하면 system metric만 수집함.
$ metricbeat modules enable
```

---
### Postgresql module 설정

Postgres이 설치되어 있는 VM에 진행

#### 모듈 활성화

```sh
$ metricbeat modules enable postgresql
```

#### 모듈 설정

`/etc/metricbeat/modules.d/postgresql.yml` 수정

```yaml
- module: postgresql
  enabled: true
  metricsets:
    - database
    - bgwriter
    - activity
    - statement
  period: 10s
  hosts: ["postgres://10.213.196.207:5432?sslmode=disable"]
  username: postgres
  password: {암호}
```

> ref: [https://www.elastic.co/guide/en/beats/metricbeat/current/metricbeat-module-postgresql.html](https://www.elastic.co/guide/en/beats/metricbeat/current/metricbeat-module-postgresql.html)

#### postgresql dashboards
![emm-1.png]({{ "/assets/img/contents/emm-1.png"}})

---
### Redis module 설정

Redis가 설치되어 있는 VM에 진행

#### 모듈 활성화

```sh
$ metricbeat modules enable redis
```

#### 모듈 설정

`/etc/metricbeat/modules.d/redis.yml` 수정

```yaml
- module: redis
  metricsets:
    - info
    - keyspace
  period: 10s

  hosts: ["127.0.0.1:6379"]
```

> ref: [https://www.elastic.co/guide/en/beats/metricbeat/7.x/metricbeat-module-redis.html](https://www.elastic.co/guide/en/beats/metricbeat/7.x/metricbeat-module-redis.html)

#### redis dashboards

![emm-2.png]({{ "/assets/img/contents/emm-2.png"}})

---
### Mongodb module 설정

Mongodb가 설치되어 있는 VM에 진행

#### 모듈 활성화

```sh
$ metricbeat modules enable mongodb
```

#### 모듈 설정

`/etc/metricbeat/modules.d/mongodb.yml` 수정

```yaml
- module: mongodb
  metricsets:
    - dbstats
    - status
    - collstats
    - metrics
  period: 10s

  hosts: ["localhost:27017"]
```

> ref: [https://www.elastic.co/guide/en/beats/metricbeat/7.x/metricbeat-module-mongodb.html](https://www.elastic.co/guide/en/beats/metricbeat/7.x/metricbeat-module-mongodb.html)

#### mongodb dashboards

![emm-3.png]({{ "/assets/img/contents/emm-3.png"}})

---
### MySQL module 설정

MySQL이 설치되어 있는 VM에 진행

#### 모듈 활성화

```sh
$ metricbeat modules enable mysql
```

#### 모듈 설정

`/etc/metricbeat/modules.d/mysql.yml` 수정

```yaml
- module: mysql
  metricsets:
    - status
    - performance
    
  period: 10s
  
  hosts: ["metricbeat:password1234@tcp(127.0.0.1:13306)/"]
```

> ref: [https://www.elastic.co/guide/en/beats/metricbeat/7.x/metricbeat-module-mysql.html](https://www.elastic.co/guide/en/beats/metricbeat/7.x/metricbeat-module-mysql.html)

#### 접속 에러시

mysql의 root 계정이 localhost에서만 접속하도록 되어 있는지 확인

ex) docker의 경우 컨테이너 접속시 host ip로 접속
```sh
# host ip 확인
$ ifconfig
```

mysql 접속하여 계정 생성 및 권한부여
```sh
$ docer exec -it mysql bash
mysql> select host, user from mysql.user;
+-----------+------------------+
| host      | user             |
+-----------+------------------+
| localhost | mysql.infoschema |
| localhost | mysql.session    |
| localhost | mysql.sys        |
| localhost | root             |
+-----------+------------------+
7 rows in set (0.00 sec)

mysql> create user 'metricbeat'@'172.18.0.1' identified by 'password1234';
Query OK, 0 rows affected (0.02 sec)

mysql > grant all privilieges on *.* to 'metricbeat'@'172.18.0.1' identified by 'password1234';
mysql > flush privileges;

```

> 자세한 내용은 [MySQL 설치 및 사용자 생성](http://localhost:4000/2021/05/31/mysql-install.html) 참고

#### mysql dashboards

![emm-4.png]({{ "/assets/img/contents/emm-4.png"}})

---
### Nginx module 설정

Nginx가 설치되어 있는 VM에 진행

#### 모듈 활성화

```sh
$ metricbeat modules enable nginx
```

#### 모듈 설정

`/etc/metricbeat/modules.d/nginx.yml` 수정

```yaml
- module: nginx
  metricsets: ["stubstatus"]
  enabled: true
  period: 10s

  # Nginx hosts
  hosts: ["http://127.0.0.1:9000"]

  # Path to server status. Default server-status
  server_status_path: "server-status"
```

> ref: [https://www.elastic.co/guide/en/beats/metricbeat/7.x/metricbeat-module-nginx.html](https://www.elastic.co/guide/en/beats/metricbeat/7.x/metricbeat-module-nginx.html)

#### nginx의 `ngx_http_stub_status_module` 모듈 활성화

```
$ cd /etc/nginx/conf.d
$ vim status.conf
    server {
        listen 9000;

		location /server-status {
          stub_status;	# stub_status 활성화
          access_log off;	# 접속 로그 비활성화
          allow 127.0.0.1;	# localhost에서만 접속 허용
          deny all;
		}
    }
$ chmod 750 status.conf
$ chown nginx:nginx status.conf
$ nginx -s reload
```

> metricbeat가 10초마다 localhost:9000/server-status를 호출하여 stub status 정보 가져감

#### nginx dashboards

![emm-5.png]({{ "/assets/img/contents/emm-5.png"}})

![emm-6.png]({{ "/assets/img/contents/emm-6.png"}})

![emm-7.png]({{ "/assets/img/contents/emm-7.png"}})

---
### Kafka module 설정

Kafka가 설치되어 있는 VM에 진행

#### 모듈 활성화

```sh
$ metricbeat modules enable kafka
```

#### 모듈 설정

`/etc/metricbeat/modules.d/kafka.yml` 수정

```yaml
- module: kafka
  metricsets:
    - partition
    - consumergroup
    - broker
    - consumer
    - producer
  period: 10s
  hosts: ["localhost:9092"]
```

> ref: [https://www.elastic.co/guide/en/beats/metricbeat/7.x/metricbeat-module-kafka.html](https://www.elastic.co/guide/en/beats/metricbeat/7.x/metricbeat-module-kafka.html)

#### Kafka with jolokia

broker, consumer, producer metricset을 가져오려면 jolokia를 이용하여 jmx 모니터링하여야한다.

kafka에 jolokia가 javaagent로 붙어서 jvm 위에 실행

[https://dev.to/martinhynar/monitoring-kafka-brokers-using-jolokia-metricbeat-and-elasticsearch-5678](https://dev.to/martinhynar/monitoring-kafka-brokers-using-jolokia-metricbeat-and-elasticsearch-5678) 참고

#### kafka dashboards

![emm-8.png]({{ "/assets/img/contents/emm-8.png"}})

---
### Zookeeper module 설정

Zookeeper가 설치되어 있는 VM에 진행

#### 모듈 활성화

```sh
$ metricbeat modules enable zookeeper
```

#### 모듈 설정

`/etc/metricbeat/modules.d/zookeeper.yml` 수정

```yaml
- module: zookeeper
  metricsets:
    - connection
    - mntr
    - server
  period: 10s
  hosts: ["localhost:2181"]
```

> ref: [https://www.elastic.co/guide/en/beats/metricbeat/7.x/metricbeat-module-zookeeper.html](https://www.elastic.co/guide/en/beats/metricbeat/7.x/metricbeat-module-zookeeper.html)

#### zookeeper dashboards

![emm-9.png]({{ "/assets/img/contents/emm-9.png"}})

---
## Metricbeat setup
```sh
# dashboard 생성을 위해 /usr/share/metricbeat 이동하여 실행
$ cd /usr/share/metricbeat

# -e 옵션으로 error 여부를 stdout 으로 출력. log에서 host에 정상 접속 했는지 확인 가능
$ metricbeat setup -e -c /etc/metricbeat/metricbeat.yml --dashboards
```

setup 완료 후 마지막 문장에 dashboard가 정상 로딩된 것을 확인
![emm-10.png]({{ "/assets/img/contents/emm-10.png"}})

---
## Metricbeat 구동

```sh
$ systemctl start metricbeat.service
```

## Metricbeat log 확인

```sh
$ journalctl -u metricbeat
```

---

## Reference

1. [Metricbeat Modules](https://www.elastic.co/guide/en/beats/metricbeat/current/metricbeat-modules.html)
2. [https://dev.to/martinhynar/monitoring-kafka-brokers-using-jolokia-metricbeat-and-elasticsearch-5678](https://dev.to/martinhynar/monitoring-kafka-brokers-using-jolokia-metricbeat-and-elasticsearch-5678)

