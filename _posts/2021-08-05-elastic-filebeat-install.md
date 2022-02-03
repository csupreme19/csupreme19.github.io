---
layout: post
title: Elastic Filebeat 설치 및 설정
feature-img: assets/img/titles/elastic-beats-logo.png
thumbnail: assets/img/contents/efi-1.png
author: csupreme19
categories: DevOps Elastic
tags: [Elasticsearch, Elastic, ELK, Filebeat, Beat]

---

# Elastic Filebeat 설치 및 설정

![elastic-beats-logo.png]({{ "/assets/img/titles/elastic-beats-logo.png"}})

[Filebeat Overview](https://www.elastic.co/guide/en/beats/filebeat/7.16/filebeat-overview.html)

로그 정보를 수집하는 Filebeat를 각 VM(Ubuntu)에 설치하여 로그 파일을 Elasticsearch로 전송한다.

---
## Filebeat란?

![efi-1.png]({{ "/assets/img/contents/efi-1.png"}})

Elastic Stack에 포함되는 오픈소스로 파일 데이터와 로그 데이터를 경량화된 방식으로 수집하고 Logstash, Elasticsearch, Kibana 등으로 전달하는 수집기

---
## Filebeat 설치

```shell
# 설치
$ curl -L -O https://artifacts.elastic.co/downloads/beats/filebeat/filebeat-7.12.1-amd64.deb
$ dpkg -i filebeat-7.12.1-amd64.deb

# 설치 후 설치된 버전 upgrade 막기
$ apt-mark hold filebeat

# bin path를 .zshrc 파일에 추가 후 적용
export PATH=/usr/share/filebeat/bin:$PATH
$ source .zshrc

# filebeat 설정
$ cd /etc/filebeat
$ vim filebeat.yml

# system 재부팅 시 자동 실행 설정
$ systemctl enable filebeat
```

### yaml에 elasticsearch, kibana 호스트 정보 설정

```yaml
# =================================== Kibana ===================================
setup.kibana:
  host: "10.213.196.6:5601"
# ---------------------------- Elasticsearch Output ----------------------------
output.elasticsearch:
  hosts: ["10.213.196.68:9200","10.213.196.23:9200","10.213.196.44:9200"]
  protocol: "http"
  username: "elastic"
  password: "elastic"
# ---------------------------- Filebeat inputs ----------------------------
filebeat.inputs:
- type: log
  enabled: false
```

### elasticsearch https 보안 설정 되어 있을 경우

```yaml
# ---------------------------- Elasticsearch Output ----------------------------
output.elasticsearch:
  hosts: ["10.213.196.68:9200","10.213.196.23:9200","10.213.196.44:9200"]
  protocol: "https"
  username: "elastic"
  password: "elastic"
  
  # ssl verification을 하지 않거나 인증서를 수동으로 등록하여야한다. 아래 택 1
  
  # ssl 검증하지 않기
  ssl.verification_mode: "none"
  
  # ssl 인증서
    ssl:
    certificate_authorities: ["/etc/elasticsearch/certs/ca.crt"]
    certificate: "/etc/elasticsearch/certs/data1.crt"
    key: "/etc/elasticsearch/certs/data1.key"
```

---
## Filebeat 구동 및 확인

```shell
$ cd /usr/share/filebeat
$ filebeat setup -e -c /etc/filebeat/filebeat.yml --dashboards
$ systemctl start filebeat
$ systemctl status filebeat
```

### 로그 확인

```shell
$ journalctl -u filebeat
```


---

## Filebeat 모듈 설정

[Elastic Filebeat 모듈 설정]({% post_url 2021-08-06-elastic-filebeat-modules %}) 참고

---

## Reference

1. [Filebeat Overview](https://www.elastic.co/guide/en/beats/filebeat/7.16/filebeat-overview.html)