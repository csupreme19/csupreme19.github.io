---
layout: post
title: Spring Framework 정리
feature-img: assets/img/titles/spring-logo.svg
thumbnail: assets/img/contents/sf-1.png
author: csupreme19
category: Development Spring
tags: [Spring, Java, J2EE, EJB, POJO, IoC, AOP, DI, Singleton, Web, Servlet]

---

# Spring Framework 정리

![spring-logo.svg]({{ "/assets/img/titles/spring-logo.svg"}})

본 문서에서는 스프링 프레임워크에 대하여 정리해봤어요.

---

## Spring이란?

### Spring이 뭐지?

![sf-1.png]({{ "/assets/img/contents/sf-1.png"}})

스프링이란 Spring Framework를 의미하기도 하지만 더 나아가 Spring Boot, Spring Data, Spring Cloud, Spring Batch 등 다양한 스프링 프로젝트를 포함하는 스프링 생태계(Spring Ecosystem)를 말해요.

초기에는 스프링 프레임워크로 시작을 하였으나 점차 모듈/프로젝트들이 추가되었고 현재의 다양한 스프링 프로젝트가 존재하는 스프링 생태계를 구성하게 됐어요.

> [Spring Projects Github](https://github.com/orgs/spring-projects/repositories)
>
> 스프링 프로젝트들은 오픈소스 생태계를 구성하고 있어 활발하게 확장되고 있다네요.

통상적으로 스프링을 말할때는 Spring Framework 프로젝트를 의미하고 추가적인 프로젝트들은 Spring Data, Spring Cloud등의 이름으로 부르고 있으며 스프링 프레임워크는 모듈로 나누어져 있어 애플리케이션 요구사항에 따라 메시징, 트랜잭션, 영속성, 웹, 리액티브 웹 등의 기능을 제공하는 모듈을 추가하여 사용할 수 있어요.

---

## Spring 등장 배경

결론부터 말하자면 J2EE 명세의 복잡함을 피하기 위한 보완 프로젝트로 등장하게 되었어요.

스프링 탄생 이전에는 자바를 이용한 엔터프라이즈 앱/서비스를 개발할 때 J2EE(Java EE)를 사용해야만 했는데

![sf-7.png]({{ "/assets/img/contents/sf-7.png"}})

EJB의 개념은 매우 뛰어났으나 스프링 등장 이전의 EJB v1, v2는 

EJB를 구현하기 위해서 XML 디스크립터 작성, 홈 인터페이스, 컴포넌트 인터페이스를 생성 등의 과정이 있어 매우 복잡하였고 이는 곧 느린 성능을 의미했다네요.

이런 복잡한 EJB에 대한 대안으로 POJO(Plain Old Java Object) 객체를 사용하는 것이 낫다는 주장이 나왔어요.

> **POJO(Plain Old Java Object)?**
>
> [https://martinfowler.com/bliki/POJO.html](https://martinfowler.com/bliki/POJO.html)
>
> POJO는 종속성이 없는 일반 자바 클래스 객체를 말하는 것이며 그다지 새로운 개념이 아니에요.
>
> 마틴 파울러(Martin Folwer)는 비즈니스 로직을 구현할 때 일반 자바 객체를 사용하는 것이 EJB를 사용하는 것 보다 훨씬 많은 장점을 가지고 있다고 생각하였고 
>
> 사람들이 일반 자바 객체를 사용하는 것을 망설이는 이유가 그럴듯한 이름이 없다고 생각하였다네요.
>
> 그래서 그럴듯한 이름을 붙여주었더니 아주 잘 나갔다는 후문이 있어요.

Spring은 J2EE에 대한 간단하고 가벼운 대안으로 POJO기반의 여러가지의 헬퍼 클래스를 제공하는데서 시작하였고

스프링은 POJO를 사용함으로써 애플리케이션을 스프링과 분리하여 기존 EJB 시스템이 지녔던 강한 결합을 해결했어요.

2004년에 Spring 1.0을 시작으로 현재는 Spring 5까지 나왔으며

이후 Java EE가 역으로 스프링의 영향을 받아서 EJB를 갈아 엎었으며 현재는 Java EE도 스프링과 매우 유사한 기능을 제공하고 있지만

스프링 프레임워크가 매우 큰 인기를 끌게 되어 Java EE를 이용하는 비율은 많이 줄어들었어요.



스프링 프레임워크는 아래와 같은 J2EE 스펙뿐만 아니라

- 서블릿 API, 웹소켓 API, 동시성 유틸, JSON 바인딩, 빈 검증, JPA, JMS

핵심 기능인 DI와 공통 애너테이션을 스펙을 제공하므로 J2EE에 대한 확실한 성공적인 보완 프로젝트가 되었어요.

---

## Spring 팀 철학

1. 모든 수준에서 선택지를 제공
   - 코드 변경 없이 여러 서드파티의 API와 구현체를 사용할 수 있도록 추상화한다.
2. 다양한 관점을 수용
   - 다양한 관점의 어플리케이션 선택을 제공하여 유연성을 제공하고 로직을 강제하지 않는다.
3. 하위 호환성을 제공
   - 스프링은 여러 JDK 버전을 공식 지원하고 스프링의 버전이 바뀔 때 마다 변경점 유지보수를 위한 서드파티 라이브러리를 지원한다.
4. 뛰어난 API 설계
   - 직관적이고 안정적인 API를 제공하는데 많은 힘을 쏟는다.
5. 코드 품질에 엄격한 기준 적용
   - 가능한 최신의 의미있는 정확한 javadoc을 작성한다.

---

## Spring Framework 구성

![sf-2.png]({{ "/assets/img/contents/sf-2.png"}})

일반적으로 말하는 Spring이란 Spring 프로젝트의 출발점이자 핵심인 Spring Framework를 일컫는 말이에요.

스프링 프레임워크의 모듈들은 위와같이 구성돼요.

<br>

### 1. Spring Core

Spring의 기본이자 핵심 모듈로 Spring Container이자 IoC Container이다.

스프링 프레임워크에서 지원하는 IoC, DI 등의 개념의 주체인 Bean을 관리하는 핵심 기능을 담당해요.

> **Spring Bean?**
>
> Spring Framework에서 관리하는 자바 객체로 일반 자바 클래스와 같다고 생각하면 돼요.
>
> Spring에서는 기존의 J2EE의 EJB 방식의 복잡한 구현을 피하기 위하여 POJO를 사용하는데 
>
> 해당 POJO가 Spring Container에 의해 생성되고 관리되면 해당 객체는 Bean이라고 해요.

아래와 같은 4가지 모듈로 구성돼요.

- `spring-core`, `spring-beans`
  - 스프링 프레임워크의 핵심 기능 IoC, DI를 지원해요.
  - BeanFactory를 구현하여 Bean을 싱글턴 객체로 관리하므로 프로그래밍적으로 객체를 관리할 필요가 없어요.
  - 프로그램 로직/비즈니스 로직과 객체의 생성/관리를 분리하는 핵심 기능이에요.
- `spring-context`
  - 위에서 생성한 Bean의 라이프사이클, 스코프인 `ApplicationContext`라는 BeanFactory의 인터페이스를 제공해요.
  - 스프링 컨테이너에서 관리하는 스프링 빈들을 사용하기 위한 인터페이스예요.
- `spring-expression`
  - JSP 명세에 의한 EL(Expression Language)의 확장 기능을 제공해요.
  - 모델, 빈 객체에 쿼리하고 접근하기 위한 EL로 스프링 컨테이너의 오브젝트와 프로퍼티, 변수, 사칙연산 등을 제공해요.

> Spring Core 내용은 아래 문서를 참고하세요.
>
> [Spring Core 정리]({% post_url 2021-12-03-spring-core %})

<br>

### 2. Spring AOP

스프링에서 DI, IoC와 함께 주요 개념인 AOP(Aspect Oriented Programming): 관점 지향 프로그래밍 기능을 제공한다. pointcut과 메소드 인터셉터들을 지원해요.

- `spring-aop`
  - AOP의 구현체로 포인트컷, 인터셉터 등을 지원해요.
  - AOP를 이용하여 소스코드에서 AOP 공통 부분을 분리할 수 있어요.
- `spring-aspects`
  - AspectJ를 지원하는 모듈
- `spring-instruments`
  - Tomcat과 같은 미들웨어 서버에서의 클래스로더 구현체를 제공해요.

> Spring AOP 내용은 아래 문서를 참고하세요.
>
> [Spring AOP 정리]({% post_url 2021-12-06-spring-AOP %})

<br>

### 3. Spring Messaging

- Spring Integration 프로젝트의 추상체를 제공하여 메시지 기반의 어플리케이션을 지원해요.

<br>

### 4. Spring DAO

영속성 데이터 접근/통합,  트랜잭션 처리, ORM 등을 지원해요.

- `spring-jdbc`
  - 스프링에서는 DB에 접근하기 위한 추상화 프레임워크로 JDBC라는 것을 사용해요.
  - 각 DB별 Driver를 구현하여 데이터베이스에 접근, 에러코드 파싱이 가능해요.
- `spring-tx`
  - POJO에 트랜잭션 처리를 위한 애너테이션 및 인터페이스를 제공해요.
- `spring-orm`
  - JPA, Hibernate와 같은 ORM(객체관계 매핑) 인터페이스를 제공해요.
- `spring-oxm`
  - JAXB, Castor와 같은 OXM(객체 XML 매핑) 인터페이스를 제공해요.
- `spring-jms`
  - 메시지 프로듀싱, 컨슈밍을 담당해요.
  - Spring 4.1 이후로 Spring Messaging 모듈과 통합되었다네요.

<br>

### 5. Spring Web

- `spring-web`
  - 서블릿 리스너, 웹 기반 IoC 컨테이너, 웹 Context, HTTP 클라이언트를 제공해요.
- `spring-webmvc`
  - Spring MVC라고도 부르며 Spring MVC와 REST 웹서비스 구현체를 포함해요.
  - 화면단과 도메인 모델, 로직을 구분하기 위하여 Model, View, Controller를 나누어 사용하는 MVC 패턴이 적용돼요.

> Spring Web 내용은 아래 문서를 참고하세요.
>
> [Spring Web 정리]({% post_url 2021-12-06-spring-web %})

<br>

### 6. Spring Test

- JUnit, TestNG 같은 단위 테스트, 통합 테스트 라이브러리를 지원하는 모듈로 Mock 객체와 단위테스트시 Spring Container, 스프링 빈에 접근하기 위한 ApplicationContext를 제공해요.

---

## Reference

1. [docs.spring.io overview](https://docs.spring.io/spring-framework/docs/current/reference/html/overview.html#overview)
2. [docs.spring.io 5.0.0 overview](https://docs.spring.io/spring-framework/docs/5.0.0.M5/spring-framework-reference/html/overview.html)
3. [docs.spring.io spring-core](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#spring-core)
4. [Udemy Spring & Hibernate for Beginners](https://www.udemy.com/course/spring-hibernate-tutorial/)
5. [스프링 철저 입문](http://www.yes24.com/Product/Goods/59192207)

