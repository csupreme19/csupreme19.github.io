---
layout: post
title: Elasticsearch ILM 로그 주기 관리
feature-img: assets/img/titles/elastic-logo.png
thumbnail: assets/img/contents/el-1.png
author: csupreme19
categories: DevOps Elastic
tags: [Elasticsearch, Elastic, ELK, Index, ILM]

---

# Elasticsearch ILM 로그 주기 관리 정책

![elastic-logo.png]({{ "/assets/img/titles/elastic-logo.png"}})

Elasticsearch의 인덱스 주기 정책에 대한 정리와 정책 설정 내용을 정리해봤어요.

---

## 개요

[ILM: Manage the index lifecycle](https://www.elastic.co/guide/en/elasticsearch/reference/7.x/index-lifecycle-management.html)

ILM(Index Lifecycle Management) 정책을 이용하여 로그 수집 주기를 관리하는 정책을 의미해요.

#### ex)

- 3달치 로그를 저장하고 나머지는 archive
- 인덱스 개수가 일정치를 넘어가면 archive
- 인덱스 사이즈가 일정치를 넘어가면 archive

위 임계값을 넘으면 아래와 같은 액션 수행이 가능해요.

- **Rollover**: 새로운 쓰기 인덱스를 생성한다.
- **Shrink**: 인덱스의 primary shard 수를 줄인다.
- **Force merge**: 인덱스 shard의 segment 수를 줄인다.
- **Freeze**: 인덱스를 read-only로 만든다.
- **Delete**: data, metadata를 포함한 인덱스를 지운다. 

<br>

## Index Lifecycle

인덱스의 생명주기는 5가지 페이즈가 있어요.
- **Hot**: 활발하게 업데이트되고 쿼리되는 인덱스
- **Warm**: 업데이트는 되지 않지만 쿼리되는 인덱스
- **Cold**: 업데이트 되지 않으며 자주 쿼리되지 않는 인덱스(검색이 되어야하지만 느려도 상관 없는 인덱스)
- **Frozen**: 업데이트 되지 않으며 드물게 쿼리되는 인덱스(검색이 되어야하지만 심하게 느려도 상관 없는 인덱스)
- **Delete**: 더 이상 필요하지 않으며 지워도 상관없는 인덱스

각 phase의 minimum age를 설정할 수 있음, 기본값은 0으로 현재 페이즈의 모든 액션이 끝나면 바로 다음 페이즈로 넘어가요.

<br>

### ILM 기본 정책

[Tutorial: Automate rollover with ILM](https://www.elastic.co/guide/en/elasticsearch/reference/7.x/getting-started-index-lifecycle-management.html) 참고하세요.

<br>

### Kibana UI 이용

Kibana 메뉴 - Stack Management - Data - Index Lifecycle Policies

![el-1.png]({{ "/assets/img/contents/el-1.png"}})

> Beats와 Logstash등에 ILM을 활성화하면 기본 정책이 자동으로 설정돼요.

기본정책을 확인할 수 있어요.

---

## Reference

1. [ILM: Manage the index lifecycle](https://www.elastic.co/guide/en/elasticsearch/reference/7.x/index-lifecycle-management.html)

2. [Tutorial: Automate rollover with ILM](https://www.elastic.co/guide/en/elasticsearch/reference/7.x/getting-started-index-lifecycle-management.html)

