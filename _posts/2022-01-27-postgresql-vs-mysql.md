---
layout: post
title: PostgreSQL vs MySQL 비교
feature-img: assets/img/titles/postgresql-vs-mysql.png
thumbnail: assets/img/contents/pm-1.png
author: csupreme19
categories: DB PostgreSQL MySQL
tags: [Postgres, PostgreSQL, MySQL, RDBMS, DB]
---

# PostgreSQL vs MySQL 비교

![postgresql-vs-mysql.png]({{ "/assets/img/titles/postgresql-vs-mysql.png"}})

---

## 개요

### 왜 PostgreSQL을 쓰나요?

PostgreSQL과 MySQL의 차이점이 무엇이냐 PostgreSQL을 왜 쓰냐라는 질문을 받았다.

둘 다 비슷한 오픈소스 RDBMS라고만 생각하고 있었고

평소에 PPAS와 PostgreSQL을 사용하고 있지만 왜 쓰게 되었는지는 생각해본적이 없어서 이참에 정리해보았다.

---

## 인기순으로 보자

![pm-1.png]({{ "/assets/img/contents/pm-1.png"}})

[2021년 Stackoverflow 데이터베이스 인기 순위](https://insights.stackoverflow.com/survey/2021#most-popular-technologies-database)

![pm-2.png]({{ "/assets/img/contents/pm-2.png"}})

[2021년 전세계 구글 트렌드](https://trends.google.co.kr/trends/explore?date=2021-01-01%202021-12-31&q=postgresql,mysql)

![pm-3.png]({{ "/assets/img/contents/pm-3.png"}})

[2021년 대한민국 구글 트렌드](https://trends.google.co.kr/trends/explore?date=2021-01-01%202021-12-31&geo=KR&q=postgresql,mysql)

MySQL이 조금 더 인기 있는 것으로 보인다.

---

## PostgreSQL과 MySQL의 차이

### PostgreSQL 특징

- ORDBMS(Object-Relational DBMS)
  - 기존 테이블, 컬럼으로만 사용하는 것이 아닌 데이터 타입, 인덱스 타입 정의 가능
  - 부모-자식 관계로 테이블 상속이 가능
- 오픈소스 프로젝트 훌륭한 개발 커뮤니티에 의해 개발됨
- 복잡한 쿼리와 대용량 데이터 처리에 특화
- 특수한 데이터베이스 상황 처리 가능

- 함수 오버로딩을 지원

- C, C++, Java 등의 프로그래밍 언어로 커스텀 함수 작성 가능
- PostGIS로 지리 정보 데이터 지원
- NoSQL 기능 지원(MySQL도 8부터 공식 지원)
- MVCC(Multi-Version Concurrency Control) 지원으로 동시성 제어

<br>

### MySQL 특징

- RDBMS(Relational DBMS)
  - ORDBMS 보다 읽기 오퍼레이션이 더 빠름
- 오픈소스 프로젝트로 오라클에서 관리함
- 1995년에 시작한 역사를 지녀 방대한 커뮤니티가 있음

- 비교적 간단하고 직관적이며 가벼운 데이터베이스

- 빠르고 신뢰성있으며 일반적인 용도로 사용하기 좋음

- PostgreSQL에 비하여 확장성이나 유틸리티가 부족
- 웹 개발 트랜잭션 같은 간단한 CRUD에 사용하면 좋음
- 윈도우, 맥, 리눅스 등 다양한 플랫폼에 설치 가능

<br>

[우아한형제들 기술 블로그](https://techblog.woowahan.com/6550/) 에 내부 구조와 실행 속도 차이에 대한 자세한 설명이 나와 있으니 더 자세한 내용을 알고 싶으면 참고하면 좋을 것 같다

---

## 결론

한마디로 정리하면 아래와 같다.

<br>

**PostgreSQL**

**fully-featured database**: 대용량, 복잡한 데이터 처리에 특화(데이터 분석 등)

<br>

**MySQL**

**lightweight database**: CRUD와 같은 간단한 트랜잭션에 특화(웹 개발 등)

<br>

두 DBMS 모두 서로 업데이트를 하면서 개선해나가 지원하는 기능이 서로 비슷해졌다.

DB 선택 이전에 PostgreSQL에서 지원하는 기능을 살펴보고 필요 없으면 MySQL을 사용하여도 무방할 듯 하다.

---

## Reference

1. [integrate.io](https://www.integrate.io/blog/postgresql-vs-mysql-which-one-is-better-for-your-use-case/)
1. [우아한형제들 기술 블로그](https://techblog.woowahan.com/6550/)
1. [2021년 Stackoverflow 데이터베이스 인기 순위](https://insights.stackoverflow.com/survey/2021#most-popular-technologies-database)

