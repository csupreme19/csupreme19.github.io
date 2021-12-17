---
layout: post
title: Elastic Stack Monitoring with Metricbeat
feature-img: assets/img/titles/elastic-logo.png
thumbnail: assets/img/contents/km-1.png
author: csupreme19
categories: DevOps Elastic
tags: [Elasticsearch, Elastic, ELK, Metricbeat, Metric, Monitoring, Observability]

---

# Elastic Stack Monitoring with Metricbeat

![elastic-logo.png]({{ "/assets/img/titles/elastic-logo.png"}})

Elastic Stack(ELK) 노드 자체의 메트릭 정보를 모니터링하기 위한 metricbeat 구축하기

---

## 개요

[Use Metricbeat to send monitoring data](https://www.elastic.co/guide/en/beats/metricbeat/7.x/monitoring-metricbeat-collection.html)

Metricbeat를 사용하여 ELS(Elastic Stack) 모니터링 구축

기존 Legacy 수집 방식과 Metricbeat 수집 방식이 있으나 7.x 버전에서는 Metricbeat를 사용하여 모니터링하도록 가이드하고 있다.

아래 5가지 항목을 모니터링한다.

1. Elasticsearch
2. Logstash
3. Kibana
4. Beats
5. APM

기존 Metricbeat로 수집되는 메트릭 정보들은 `metricbeat-*` 인덱스에 수집이 되지만 `xpack.enabled` 옵션으로 xpack 기능 활성화시 별도의 monitoring cluster로 수집하며 Kibana의 Stack Monitoring 메뉴에서 확인할 수 있다.

---
### Metricbeat 설치

[Elastic Metricbeat 설치 및 설정](2021-07-27-elastic-metricbeat-install) 참고

---
### Metricbeat 설정
`metricbeat.yml` 수정

```yaml
setup.kibana:
  host: "http://10.213.196.6:5601"
  
  # ssl 인증서 생성 시 https 사용
  host: "https://172.25.0.166:5601"

  # ssl 인증서 적용 시 추가
  ssl:
    enabled: true
    certificate_authorities: ["/etc/logstash/certs/ca.crt"]
    certificate: "/etc/logstash/certs/logstash-1.crt"
    key: "/etc/logstash/certs/logstash-1.key"

output.elasticsearch:
  # elasticsearch 노드 IP 넣기
  hosts: ["{엘라스틱서치1}:9200","{엘라스틱서치2}:9200","{엘라스틱서치3}:9200"]

  # ssl 인증서 적용 시 https, 미 적용 시 http 사용
  protocol: "https"

  username: "elastic"
  password: "{암호}"

  # ssl 인증서 적용시
  ssl:
    certificate_authorities: ["/etc/logstash/certs/ca.crt"]
    certificate: "/etc/logstash/certs/logstash-1.crt"
    key: "/etc/logstash/certs/logstash-1.key"
```

elastic stack monitoring 활성화시 시스템 정보를 수집해도 UI에 보이지 않으므로 비활성화한다.
```shell
$ metricbeat modules disable system
```

---
## Elasticsearch Monitoring

### Elasticsearch 설정
#### `elasticsearch.yml` 수정

```yaml
xpack.monitoring.collection.enabled: true	# xpack 모니터링(metricbeat 사용)
monitoring.enabled: false									# legacy 모니터링(metricbeat 미사용)
```
기존 legacy 방식 모니터링은 false, xpack 모니터링은 true로 설정

#### elasticsearch-xpack module 활성화

```shell
$ metricbeat modules disable elasticsearch
$ metricbeat modules enable elasticsearch-xpack
```
xpack과 기본 모듈중 하나만 사용해야 하기 때문에 기존의 모듈은 disable

#### elasticsearch-xpack module 설정

`modules.d/elasticsearch-xpack.yml` 수정
```yaml
- module: elasticsearch
  xpack.enabled: true
  period: 10s
  hosts: ["http://localhost:9200", "http://localhost:9201", "http://localhost:9202"]
  
  # elasticsearch 모니터링용 user(Optional)
  #username: "user"
  #password: "secret"
```

#### elasticsearch 모니터링용 사용자 생성(Optional)
`remote_monitoring_collector` 롤을 가진 사용자를 생성. 생성한 사용자 id는 `elasticsearch_monitoring`

### Metricbeat 구동 및 kibana에서 확인
#### metricbeat 서비스 구동

```shell
$ systemctl start metricbeat.service
```

#### kibana Stack Monitoring 메뉴에서 `Metricbeat monitoring` 상태임을 확인

![esm-1.png]({{ "/assets/img/contents/esm-1.png"}})

---
## Logstash Monitoring

### Logstash 설정

#### 기본 모니터링 설정 끄기
kibana에서 stack monitoring 확인 시 Self monitoring으로 표기될 시 기본 모니터링 옵션을 끈다.
![esm-2.png]({{ "/assets/img/contents/esm-2.png"}})

#### `logstash.yml`에 추가

```sh
monitoring.enabled: false
```

#### logstash-xpack 모듈 활성화
```shell
$ metricbeat modules disable logstash
$ metricbeat modules enable logstash-xpack
```
xpack과 기본 모듈중 하나만 사용해야 하기 때문에 기존의 모듈은 disable

#### logstash-xpack 모듈 설정
[Collect Logstash monitorin data with Metricbeat](https://www.elastic.co/guide/en/logstash/7.12/monitoring-with-metricbeat.html)

#### `modules.d/logstash-xpack.yml` 수정

```yaml
- module: logstash
  metricsets: ["node","node_stats"]
  xpack.enabled: true
  # hosts 값에는 metric 수집 대상을 추가. 
  hosts: ["http://172.25.0.66:9600"]
  period: 10s
  # logstash 모니터링용 사용자(Optional)
  username: "logstash_monitoring"
  password: "Infra1111"
```

#### logstash 모니터링용 user 생성(Optional)
kibana web ui 에서 아래 화면과 같이 모니터링용 user를 생성
![esm-3.png]({{ "/assets/img/contents/esm-3.png"}})

roles에는 `remote_monitoring_collector` 를 선택

### Metricbeat 구동 및 kibana에서 확인

#### metricbeat 서비스 구동

```shell
$ systemctl start metricbeat.service
```

#### kibana Stack Monitoring 메뉴에서 `Metricbeat monitoring` 상태임을 확인

![esm-4.png]({{ "/assets/img/contents/esm-4.png"}})

---
## Kibana Monitoring

### Kibana 설정

#### 기본 모니터링 설정 끄기

kibana에서 stack monitoring 확인 시 `Self monitoring`으로 표기될 시 기본 모니터링 옵션을 끈다.
![esm-5.png]({{ "/assets/img/contents/esm-5.png"}})

#### `kibana.yml`에 추가

```shell
monitoring.kibana.collection.enabled: false
```
기존 legacy 방식으로 모니터링 데이터를 수집하는 `monitoring.kibana.collection.enabled` 설정 값을 false로 변경. 

#### kibana-xpack module 활성화
```sh
$ metricbeat modules disable kibana
$ metricbeat modules enable kibana-xpack
```
xpack과 기본 모듈중 하나만 사용해야 하기 때문에 기존의 모듈은 disable

#### kibana-xpack module 설정
[Kibana module](https://www.elastic.co/guide/en/beats/metricbeat/current/metricbeat-module-kibana.html)

#### `modules.d/kibana-xpack.yml` 수정

```yaml
- module: kibana
  # 두 개의 metricset 설정 가능.
  metricsets: ["stats","status"]
  # metricbeat로 모니터링 시 아래 옵션을 true로 설정해야 함
  xpack.enabled: true
  period: 10s
  # kibana 서버 url
  hosts: ["http://10.213.196.6:5601"]
  scopse: node
  # kibana 모니터링용 user(Optional)
  username: "kibana_monitoring"
  password: "changeme"

  # ssl 인증서 생성하여 적용 시 아래 옵션 적용
  ssl.enabled: true
  ssl.certificate_authorities: ["/etc/kibana/certs/ca.crt"]
  ssl.certificate: "/etc/kibana/certs/kibana.crt"
  ssl.key: "/etc/kibana/certs/kibana.key"
  ssl.verification_mode: "full"
```

#### kibana 모니터링용 사용자 생성(Optional)
`remote_monitoring_collector` 롤을 가진 사용자를 생성. 생성한 사용자 id는 `kibana_monitoring`

### Metricbeat 구동 및 kibana에서 확인
#### metricbeat 서비스 구동

```shell
$ systemctl start metricbeat.service
```

#### kibana Stack Monitoring 메뉴에서 `Metricbeat monitoring` 상태임을 확인

![esm-6.png]({{ "/assets/img/contents/esm-6.png"}})

---

## Reference

1. [Use Metricbeat to send monitoring data](https://www.elastic.co/guide/en/beats/metricbeat/7.x/monitoring-metricbeat-collection.html)

2. [Collect Logstash monitorin data with Metricbeat](https://www.elastic.co/guide/en/logstash/7.12/monitoring-with-metricbeat.html)
3. [Kibana module](https://www.elastic.co/guide/en/beats/metricbeat/current/metricbeat-module-kibana.html)

