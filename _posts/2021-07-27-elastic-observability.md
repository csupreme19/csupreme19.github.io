---
layout: post
title: Elastic Stack Observability 개요
feature-img: assets/img/titles/elastic-logo.png
thumbnail: assets/img/contents/eo-1.png
author: csupreme19
categories: DevOps Elastic
tags: [Elasticsearch, Elastic, ELK, Kibana, Logstash, Monitoring, Observability]

---

# Elastic Stack Observability 개요

![elastic-logo.png]({{ "/assets/img/titles/elastic-logo.png"}})

[Observability with the elastic stack](https://www.elastic.co/kr/blog/observability-with-the-elastic-stack)

Elastic Stack을 이용하여 MSA 서비스 모니터링시 모니터링 중앙 집중화를 통한 가관측성(Observability)에 대하여 알아본다.

---
## 모니터링의 3요소

{% include aligner.html images="contents/eo-1.png" %}

![eo-2.png]({{ "/assets/img/contents/eo-2.png"}})

운영시 모니터링 요소는 Logs, Metrics, APM 크게 3가지로 분류할 수 있다.

- Logs
- Metrics
- APM(Application Perfromance Monitoring)

추가로 Heartbeat, Packetbeat 등을 이용한 서버의 가용성을 체크하는 Uptime Data가 있음

### Logs Data

![eo-3.png]({{ "/assets/img/contents/eo-3.png"}})

어플리케이션 로그, 서비스 로그 등의 각 모듈의 실제 로그로 Filebeat, FunctionBeat 등을 이용하여 수집한다.

DB 롱쿼리, 호출 내역, 이벤트 데이터 수발신, Kafka 행등 서비스 레벨에서 작성하는 로그를 통한 가시성을 챙길 수 있다.

### Metrics Data

![eo-4.png]({{ "/assets/img/contents/eo-4.png"}})

![eo-5.png]({{ "/assets/img/contents/eo-5.png"}})

호스트, 시스템의 리소스 정보 및 사용량, 부하율, 네트워크 트래픽 등 

주기적인 전체 요약이나 통계적인 정보로 일정 기간에 걸쳐서 샘플링되는 경향을 지닌 데이터이다.

VM의 부하율, 메모리 점유율 등을 파악하여 Scale Out을 하거나 DB의 사용량을 파악하여 Worker node를 늘린다던지 하는 방향으로 사용할 수 있다.

### APM Data

![eo-6.png]({{ "/assets/img/contents/eo-6.png"}})

![eo-7.png]({{ "/assets/img/contents/eo-7.png"}})

어플리케이션 중점의 성능 정보로 실제 어플리케이션 오류 정보나 작업 수행 시간, 서비스간 상호 연동, 트랜잭션 추적, 병목 지점등 많은 정보를 담아낸다.

성능 병목 파악, 에러 검출, Long Query, 트랜잭션 추적 등 어플리케이션 레벨의 가시성을 챙겨준다.

Application에 Agent가 사이드카 형태로 올라가는 방식으로 많이 사용한다.

### Uptime Data

Liveness, Readiness, Healthz 등 Uptime 정보

---

## ELK Stack(Elastic Stack)을 채택한 이유

위의 모니터링을 통하여 가시성을 챙기는 방법은 여러가지가 있을 수 있으나 ELK Stack을 채택한 이유는 크게 다음과 같다.

1. 루씬(Lucene) 기반의 뛰어난 분산처리를 통한 실시간 검색 기능을 제공한다.
2. 메트릭, 로그, APM 등의 모니터링 정보를 중앙 집중화할 수 있다.
3. 오픈소스 기반으로 다양한 서드파티 라이브러리를 제공하여 여러 오픈소스들을 쉽게 연동 가능
4. 다양한 시각화 라이브러리를 제공하여 큰 어려움 없이 데이터 시각화 가능
5. 시계열 데이터를 운영하는데 최적화

#### ELK 적용 이전

![eo-8.png]({{ "/assets/img/contents/eo-8.png"}})

![eo-9.png]({{ "/assets/img/contents/eo-9.png"}})

각기 다른 정보를 모니터링하기 위해 특화된 여러 툴들을 띄워놓고 모니터링한다.

따라서 관리 포인트가 많아지고 데이터의 중복이 발생할 수 있다.

#### ELK 적용 이후

![eo-10.png]({{ "/assets/img/contents/eo-10.png"}})

> 단일 페이지에서 중앙 집중화된 로그를 쉽게 모니터링 할 수 있다.

모든 metric, apm, logs 정보들은 Elasticsearch의 인덱스이므로 데이터를 시각화, 머신러닝, 필터링 하는데 아무런 제약이 없다.

로그 중앙집중화 및 가시성 확보뿐만 아니라 Elasticsearch는 데이터 분석 엔진으로도 탁월한 성능을 자랑한다.


---

## Reference

1. [Observability with the elastic stack](https://www.elastic.co/kr/blog/observability-with-the-elastic-stack)