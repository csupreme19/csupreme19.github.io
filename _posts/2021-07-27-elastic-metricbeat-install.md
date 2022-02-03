---
layout: post
title: Elastic Metricbeat 설치 및 설정
feature-img: assets/img/titles/elastic-beats-logo.png
thumbnail: assets/img/contents/emi-1.png
author: csupreme19
categories: DevOps Elastic
tags: [Elasticsearch, Elastic, ELK, Metricbeat, Beat]

---

# Elastic Metricbeat 설치 및 설정

![elastic-beats-logo.png]({{ "/assets/img/titles/elastic-beats-logo.png"}})

[Metricbeat Overview](https://www.elastic.co/guide/en/beats/metricbeat/current/metricbeat-overview.html)

로그 정보를 수집하는 Filebeat를 각 서버(Ubuntu)에 설치하여 로그 파일을 Elasticsearch로 전송했던 내용을 정리해봤어요.

---
## Metricbeat란?

Elastic Stack에 포함되는 오픈소스로 시스템과 서비스의 규격화된 메트릭 정보를 경량화된 방식으로 수집하고 Logstash, Elasticsearch, Kibana 등으로 전달하는 수집기예요.

---
## Metricbeat 설치
현재 설치된 elasticsearch 버전을 확인 후 **호환되는 버전**의 metricbeat를 설치해야해요.

가능한 elaticsearch 버전과 동일한 버전의 metricbeat을 설치하는 것이 좋아요.

[Elastic components](https://www.elastic.co/kr/downloads/past-releases)

[Metricbeat 7.11.2](https://www.elastic.co/downloads/past-releases/metricbeat-7-11-2)

wget으로 특정 버전 설치해봤어요.
```sh
# Ubuntu
$ wget https://artifacts.elastic.co/downloads/beats/metricbeat/metricbeat-7.11.2-amd64.deb
# 설치
$ dpkg -i metricbeat-7.11.2-amd64.deb

# CentOS
$ wget https://artifacts.elastic.co/downloads/beats/metricbeat/metricbeat-7.12.1-x86_64.rpm
# 설치
$ rpm -ivh ./metricbeat-7.12.1-x86_64.rpm

# 설치 후 설치된 버전 upgrade 막기
$ apt-mark hold metricbeat

# 설치파일 삭제
$ rm metricbeat-7.11.2-amd64.deb
$ rm metricbeat-7.12.1-x86_64.rpm

# bin path를 .zshrc 파일에 추가 후 적용
export PATH=/usr/share/metricbeat/bin:$PATH
```

system booting시 자동 실행을 보장하기 위해 아래와 같이 systemd에 등록이 필요해요.
```sh
$ systemctl enable metricbeat.service
$ systemctl daemon-reload
```

---
## Metricbeat 설정
metricbeat 설치 후 elasticsearch로의 접속 설정 및 보안 설정 진행이 필요해요.

<br>

### Metricbeat 접속 설정 및 보안 설정
미리 구축해둔 elasticsearch에 연결할 예정이므로 `output.elasticsearch` 부분에 연결을 진행할 예정이에요.

설정 진행 시 ssl 부분도 설정하며 인증서 파일은 data node용으로 만들어둔 인증서를 사용해요.

  - `output.elasticsearch` 값을 설정
  - username, password 는 hard-coded 하지 말고 secrets keystore 등으로 대체 필요해요.

**SSL 인증 키 없이** http 접속 시 아래 설정 사용했어요.

`metricbeat.yml`

**SSL 인증서 없는 경우**

```sh
# kibana 접속 정보 설정
setup.kibana:
  host: "http://{키바나 주소}:5601"

output.elasticsearch:
  # master node들 
  hosts: ["{엘라스틱서치 노드1}:9200","{엘라스틱서치 노드2}:9200","{엘라스틱서치 노드3}:9200"]

  # 보안설정 후 https 접속
  protocol: "http"

  username: "elastic"
  password: "elastic"
```

`metricbeat.yml`

**SSL 인증서 있는 경우**

```sh
# kibana 접속 정보 설정
setup.kibana:
  host: "https://{키바나 주소}:5601"

  ssl:
    enabled: true
    certificate_authorities: ["/etc/elasticsearch/certs/ca.crt"]
    certificate: "/etc/elasticsearch/certs/data1.crt"
    key: "/etc/elasticsearch/certs/data1.key"
    
output.elasticsearch:
  # master node들 
  hosts: ["{엘라스틱서치 노드1}:9200", "{엘라스틱서치 노드2}:9200", "{엘라스틱서치 노드3}:9200"]

  # 보안설정 후 https 접속
  protocol: "https"

  #api_key: "id:api_key"
  username: "elastic"
  password: "{암호}"

  # tls 설정 값. logstash-1 서버에 적용중이기 때문에 해당 node의 인증서를 적용
  ssl:
    certificate_authorities: ["/etc/elasticsearch/certs/ca.crt"]
    certificate: "/etc/elasticsearch/certs/data1.crt"
    key: "/etc/elasticsearch/certs/data1.key"
```

```sh
# 테스트 진행 후 Confg OK 출력 확인
$ metricbeat test config -e
```
![emi-1.png]({{ "/assets/img/contents/emi-1.png"}})

---
## System module 설정
```sh
# 필요 시 system 모듈 설정
$ vi /etc/metricbeat/modules.d/system.yml

# 추가 module 없이 활성화를 하면 system metric만 수집
$ metricbeat modules enable
```

### Metricbeat Setup
```sh
# asset 로딩 시 dashboard 생성을 위해 /usr/share/metricbeat 으로 이동하여 실행
$ cd /usr/share/metricbeat

# -e 옵션으로 error 여부를 stdout 으로 출력. log에서 host에 정상 접속 했는지 확인 가능
$ metricbeat setup -e -c /etc/metricbeat/metricbeat.yml --dashboards
```

### Metricbeat 구동 & metric 값 전송
```sh
$ systemctl start metricbeat.service
```

---

## Metricbeat 모듈 설정

[Elastic Metricbeat 모듈 설정]({% post_url 2021-08-02-elastic-metricbeat-modules %}) 참고하세요.

---

## Reference

1. [Metricbeat Overview](https://www.elastic.co/guide/en/beats/metricbeat/current/metricbeat-overview.html)