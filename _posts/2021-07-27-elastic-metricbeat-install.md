---
layout: post
title: Elastic Metricbeat 설치 및 설정
feature-img: assets/img/titles/elastic-beats-logo.png
thumbnail: assets/img/contents/emi-1.png
author: csupreme19
tags: [Elasticsearch, Elastic, ELK, Metricbeat, Beat]

---

# Elastic Metricbeat 설치 및 설정

![elastic-beats-logo.png]({{ "/assets/img/titles/elastic-beats-logo.png"}})

[Metricbeat Overview](https://www.elastic.co/guide/en/beats/metricbeat/current/metricbeat-overview.html)

로그 정보를 수집하는 Filebeat를 각 VM(Ubuntu)에 설치하여 로그 파일을 Elasticsearch로 전송한다.

---
## Metricbeat란

Elastic Stack에 포함되는 오픈소스로 시스템과 서비스의 규격화된 메트릭 정보를 경량화된 방식으로 수집하고 Logstash, Elasticsearch, Kibana 등으로 전달하는 수집기

---
## Metricbeat 설치
현재 설치된 elasticsearch 버전을 확인 후 **호환되는 버전**의 metricbeat를 설치한다.

되도록 elaticsearch 버전과 동일한 버전의 metricbeat을 설치한다.

elastic component들 과거버전 선택 download:
  - https://www.elastic.co/kr/downloads/past-releases

metricbeat 7.11.2 download url:
  - https://www.elastic.co/downloads/past-releases/metricbeat-7-11-2

wget 으로 7.11.2 버전을 download 및 설치:
```sh
# 다운로드
$ wget https://artifacts.elastic.co/downloads/beats/metricbeat/metricbeat-7.11.2-amd64.deb

# CentOS 7 용(rpm)
$ wget https://artifacts.elastic.co/downloads/beats/metricbeat/metricbeat-7.12.1-x86_64.rpm

# 설치
$ dpkg -i metricbeat-7.11.2-amd64.deb
# 7.12.1 버전
$ dpkg -i metricbeat-7.12.1-amd64.deb
#CentOS rpm
$ rpm -ivh ./metricbeat-7.12.1-x86_64.rpm

# 설치 후 설치된 버전 upgrade 막기
$ apt-mark hold metricbeat

# 설치파일 삭제
$ rm metricbeat-7.11.2-amd64.deb

# bin path를 .zshrc 파일에 추가 후 적용
export PATH=/usr/share/metricbeat/bin:$PATH

$ source .zshrc
```

system booting 시 자동 실행 등록.
```sh
$ systemctl enable metricbeat.service
$ systemctl daemon-reload
```



---
## Metricbeat 설정
metricbeat 설치 후 elasticsearch로의 접속 설정 및 보안 설정 진행

config directory:
  - /etc/metricbeat

config file:
  - metricbeat.yml

### Metricbeat 접속 설정 및 보안 설정
미리 구축해둔 elasticsearch에 연결할 예정이므로 `output.elasticsearch` 섹션에 연결 설정을 진행한다.

설정 진행 시 ssl 부분도 설정하며 인증서 파일은 data node용으로 만들어둔 인증서를 사용한다.

elasticsearch 연결 설정:
  - `output.elasticsearch` 값을 설정
  - username, password 는 hard-coded 하지 말고 serets keystore 등으로 대체 필요

metricbeat.yml 설정 내용 - **SSL 인증 키 없이** http 접속 시 아래 설정 사용
```sh
# kibana 접속 정보 설정.
setup.kibana:
  host: "http://10.213.196.6:5601"

output.elasticsearch:
  # master node들 
  hosts: ["10.213.196.68:9200","10.213.196.23:9200","10.213.196.44:9200"]

  # 보안설정 후 https 접속
  protocol: "http"

  username: "elastic"
  password: "elastic"
```


metricbeat.yml 설정 내용 - SSL 인증 키 생성하여 적용 시 아래 설정 사용:
```sh
# kibana 접속 정보 설정.
setup.kibana:
  host: "https://172.25.0.166:5601"

  ssl:
    enabled: true
    certificate_authorities: ["/etc/elasticsearch/certs/ca.crt"]
    certificate: "/etc/elasticsearch/certs/data1.crt"
    key: "/etc/elasticsearch/certs/data1.key"
    
output.elasticsearch:
  # master node들 
  hosts: ["172.25.0.92:9200", "172.25.0.159:9200", "172.25.0.121:9200"]

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

설정파일 테스트. 설정 파일이 위치한 directory 에서 테스트 가능하며 테스트 후 OK가 출력되면 OK:
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

# 추가 module 없이 활성화를 하면 system metric만 수집함.
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

[Elastic Metricbeat 모듈 설정](/2021/08/02/elastic-metricbeat-modules.html) 참고

---

## Reference

1. [Metricbeat Overview](https://www.elastic.co/guide/en/beats/metricbeat/current/metricbeat-overview.html)