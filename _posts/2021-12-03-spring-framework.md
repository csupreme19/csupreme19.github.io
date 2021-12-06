---
layout: post
title: Spring Framework 정리
feature-img: assets/img/titles/spring-logo.svg
thumbnail: assets/img/contents/sf-1.png
author: csupreme19
tags: [Spring, Java, J2EE, EJB, POJO, IoC, AOP, DI]

---

# Spring Framework 정리

![spring-logo.svg]({{ "/assets/img/titles/spring-logo.svg"}})

---

## Spring이란?

### Spring이 뭐지?

![sf-1.png]({{ "/assets/img/contents/sf-1.png"}})

스프링이란 Spring Framework를 의미하기도 하지만 더 나아가 Spring Boot, Spring Data, Spring Cloud, Spring Batch 등 다양한 프로젝트를 포함하는 스프링 생태계(Spring Ecosystem)를 말한다.

초기에는 스프링 프레임워크로 시작을 하였지만 점차 추가적인 모듈/프로젝트들이 추가되어 현재의 스프링 생태계를 구성하고 있으며 현재도 활발하게 확장되고 있다.

그래도 일반적으로 Spring를 말할때는 Spring Framework를 의미하고 추가적인 프로젝트들은 Spring Data, Spring Cloud등의 이름으로 부르고 있다.

어떠한 프로젝트가 있는지는 spring.io 공식 홈페이지에서 확인할 수 있다.

> [spring.io/projects](https://spring.io/projects)

---

## Spring 탄생 배경

Spring 탄생 이전에는 자바를 이용한 엔터프라이즈 앱/서비스를 개발할 때 J2EE(Java EE)를 사용하여야했다.

EJB등의 개념은 매우 뛰어났으나 스프링 등장 이전의 EJB v1, EJB v2는 

EJB를 구현하기 위해서 XML 디스크립터를 작성하고 홈 인터페이스, 컴포넌트 인터페이스를 생성해야만 하여 매우 복잡하였고 이는 곧 느린 성능을 의미하였다.

이런 EJB에 대한 대안으로 POJO(Plain Old Java Object) 객체를 사용하는 것이 낫다는 주장이 나왔다.

> POJO(Plain Old Java Object)?
>
> [https://martinfowler.com/bliki/POJO.html](https://martinfowler.com/bliki/POJO.html)
>
> POJO는 종속성이 없는 일반 자바 클래스 객체를 말하는 것이며 그다지 새로운 개념이 아니다.
>
> 마틴 파울러는 비즈니스 로직을 구현할 때 일반 자바 객체를 사용하는 것이 EJB를 사용하는 것 보다 훨씬 많은 장점을 가지고 있다고 생각하였고 
>
> 사람들이 일반 자바 객체를 사용하는 것을 망설이는 이유가 그럴듯한 이름이 없다고 생각하였다.
>
> 그래서 그럴듯한 이름을 붙여주었더니 아주 잘 나갔다는 후문

Spring은 J2EE에 대한 간단하고 가벼운 대안으로 POJO기반의 여러가지의 헬퍼 클래스를 제공하는데서 시작하였다.

2004년에 Spring 1.0을 시작으로 현재는 Spring 5까지 나왔으며

추후에는 Java EE가 역으로 Spring의 영향을 받아서 EJB를 갈아 엎었으며 현재는 EJB도 스프링과 매우 유사한 기능을 제공하고 있다.

하지만 예전의 EJB의 악평과 그 사이에 스프링 프레임워크가 매우 큰 인기를 끌어서 Java EE를 이용하는 비율은 많이 줄었다.

---

## Spring Framework?

![sf-2.png]({{ "/assets/img/contents/sf-2.png"}})

일반적으로 말하는 Spring이란 Spring 프로젝트의 출발점이자 핵심인 Spring Framework를 일컫는다.

스프링 프레임워크의 모듈들은 위와같이 구성된다.

### 1. Spring Core

Spring의 기본이자 핵심 모듈로 Spring Container, IoC Container이다.

스프링 프레임워크에서 지원하는 IoC, DI 등의 개념이 탄생한 Bean을 관리하는 핵심 기능을 담당한다.

> **Spring Bean?**
>
> Spring Framework에서 관리하는 자바 객체로 일반 자바 클래스와 같다고 생각하면 된다.
>
> Spring에서는 기존의 J2EE의 EJB 방식의 복잡한 구현을 피하기 위하여 POJO를 사용하는데 해당 POJO가 Spring Container에 의해 생성되고 관리되면 해당 객체는 Bean이라고 한다.

아래와 같은 4가지 모듈로 구성된다.

- `spring-core`, `spring-beans`
  - 스프링 프레임워크의 핵심 기능 IoC, DI를 지원한다.
  - BeanFactory를 구현하여 Bean을 싱글턴 객체로 관리하므로 프로그래밍적으로 객체를 관리할 필요가 없다.
  - 프로그램 로직/비즈니스 로직과 객체의 생성/관리를 분리하는 핵심 기능
- `spring-context`
  - 위에서 생성한 Bean의 라이프사이클, 스코프인 `ApplicationContext`등의 인터페이스를 제공한다.
  - 스프링 컨테이너에서 관리하는 스프링 빈들을 사용하기 위한 인터페이스이다.
- `spring-expression`
  - JSP 명세에 의한 EL(Expression Language)의 확장 기능을 제공한다.
  - 모델 객체에 쿼리하고 접근하기 위한 EL로 스프링 컨테이너의 오브젝트와 프로퍼티, 변수, 사칙연산 등을 제공한다.

### 2. Spring AOP

스프링에서 DI, IoC와 함께 주요 기능인 AOP(Aspect Oriented Programming) 관점 지향 프로그래밍 기능을 제공한다. pointcut과 메소드 인터셉터들을 지원한다.

- `spring-aop`
  - AOP의 구현체로 포인트컷, 인터셉터 등을 지원한다.
  - AOP를 이용하여 소스코드에서 AOP 공통 부분을 분리할 수 있다.
- `spring-aspects`
  - AspectJ를 지원하는 모듈이다.
- `spring-instruments`
  - Tomcat과 같은 미들웨어 서버에서의 클래스로더 구현체를 제공한다.

### 3. Spring Messaging

Spring Integration 프로젝트의 추상체를 제공하여 메시지 기반의 어플리케이션을 지원한다.

### 4. Spring DAO

영속성 데이터 접근/통합,  트랜잭션 처리, ORM 등을 지원한다.

- `spring-jdbc`
  - 스프링에서는 DB에 접근하기 위한 추상화 프레임워크로 JDBC라는 것을 사용한다.
  - 각 DB별 Driver를 구현하여 데이터베이스에 접근, 에러코드 파싱이 가능하다.
- `spring-tx`
  - POJO에 트랜잭션 처리를 위한 어노테이션 및 인터페이스를 제공한다.
- `spring-orm`
  - JPA, Hibernate와 같은 ORM(객체관계 매핑) 인터페이스를 제공한다.
- `spring-oxm`
  - JAXB, Castor와 같은 OXM(객체 XML 매핑) 인터페이스를 제공한다.
- `spring-jms`
  - 메시지 프로듀싱, 컨슈밍을 담당한다.
  - Spring 4.1 이후로 Spring Messaging 모듈과 통합되었다.

### 5. Spring Web

- `spring-web`
  - 서블릿 리스너, 웹 기반 IoC 컨테이너, 웹 Context, HTTP 클라이언트를 제공한다.
- `spring-webmvc`
  - Spring MVC라고도 부르며 Spring MVC와 REST 웹서비스 구현체를 포함한다.
  - 화면단과 도메인 모델, 로직을 구분하기 위하여 Model, View, Controller를 나누어 사용하는 MVC 패턴이 적용된다.

### 6. Spring Test

JUnit, TestNG 같은 단위 테스트, 통합 테스트 라이브러리를 지원하는 모듈로 Mock 객체와 단위테스트시 Spring Container, 스프링 빈에 접근하기 위한 ApplicationContext를 제공한다.

---

## Spring Core

![sf-3.png]({{ "/assets/img/contents/sf-3.png"}})

Spring Framework의 핵심 모듈인 Spring Core에 대해서 알아본다.

일반적으로 스프링 프레임워크의 IoC, DI 등의 핵심 기능을 해당 모듈에서 담당한다.

스프링 컨테이너, IoC 컨테이너 그 자체라고 보아도 무방하다.



### Inversion Of Control(IoC)

**제어의 역전; 객체의 생성과 관리를 외부에 맡기는 것**

기존 초기 자바 어플리케이션의 경우 아래와 같이 개발자가 객체의 생성과 라이프사이클을 관리해주어야 하였다.

```java
// POJO
public class Cat {
  String name;
  String gender;
  
  public Cat() {}
}

Cat myCat = new Cat();
System.out.println(myCat.toString());
```

하지만 위와 같이 사용한다면 생성된 Cat을 여러 클래스에서 사용하기 어려우며 해당 Cat에 대한 속성을 관리하기가 힘들고 또한 여러개의 중복된 Cat 객체가 생성될 가능성이 있다.



![sf-4.png]({{ "/assets/img/contents/sf-4.png"}})

이를 해결하기 위하여 스프링에서는 모든 객체의 생성과 관리를 Bean으로 등록하여 스프링 컨테이너에게 위임한다 이를 제어의 역전(Inversion Of Control)이라고 한다.

제어의 역전이라는 특성을 통하여 개발자는 더 이상 객체의 생성과 관리에 대하여 신경쓰지 않고 비즈니스 로직에만 집중하여 생산성을 높일 수 있다.



#### XML 설정 예시

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" 
    xmlns:context="http://www.springframework.org/schema/context"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
    http://www.springframework.org/schema/beans/spring-beans.xsd
    http://www.springframework.org/schema/context
    http://www.springframework.org/schema/context/spring-context.xsd">
  
  <!-- 역전 제어 -->
	<bean id="cat" class="com.csupreme19.springdemo.Cat" />
    
</beans>
```

```java
ApplicationContext ctx = new ClassPathXmlApplicationContext("applicationContext.xml");
Cat myCat = ctx.getBean("cat", Cat.class);
```

빈은 어플리케이션의 실행과 종료까지의 생명주기를 가지는 ApplicationContext에 등록되어 사용되고 해당 ApplicationContext는 XML 파일, 어노테이션, 자바 코드 등으로 설정할 수 있다.

컨텍스트 생성 시점 Bean들을 주입(생성)하여 초기화하며 컨텍스트에서 `getBean()` 호출시 스프링 컨테이너에 객체가 존재하는지 확인하고 객체를 가져온다.

<div class="mermaid">
  flowchart LR
    A[Application]
    B[Spring Container]
    C[XML Configuration]
    D[Cat Bean]
    C--1. Config-->B
    B--2. Generate---D
    A--3. getBean-->B
    B-.4. Bean.->A
</div>

이를 통해 개발자는 더 이상 객체의 생성과  관리에 대해서 걱정할 필요가 없이 싱글턴 객체를 사용할 수 있다.



### Dependency Injection(DI)

**의존성 주입; 객체의 의존성을 주입하는 것**

스프링 컨테이너에 빈을 등록하여 관리하는데 해당 Bean이 POJO 형태가 아니라 여러 Dependency를 가지고 있으면 어떻게 될까?

알다시피 객체지향 관점에서 대부분의 객체는 의존성을 가지고 있다.

해당 객체는 스프링 컨테이너에서 사용 못하고 개발자가 직접 수동으로 생성해야 할까?

정답은 아니다.

스프링에서는 이러한 의존성을 해결하기 위하여 의존성 주입이라는 개념이 등장한다.



**Constructor Dependency Injection 예시**

```java
public class Cat {
	
	private String name;
	private String gender;
	public Eye eye;
	
	public Cat() {}
	
	public Cat(Eye eye) {
		this.eye=eye;
	}
}

Cat myCat = ctx.getBean("cat", Cat.class);
System.out.println(myCat.toString());
System.out.println(myCat.eye.toString());
```

Cat에 Eye라는 dependency를 추가하였다.

스프링의 DI를 사용하려면 해당 Eye 또한 Bean으로 등록되어 이후 DI 시점에 주입되어야한다.

```xml
<bean id="eye" class="com.csupreme19.springdemo.Eye" />
<bean id="cat" class="com.csupreme19.springdemo.Cat">
	  <!-- 의존성 주입 -->
		<constructor-arg ref="eye"/>
</bean>
```



<div class="mermaid">
  flowchart LR
    A[Application]
    B[Spring Container]
    C[XML Configuration]
    D[Cat Bean]
    E[Eye Bean]
    C--1. Metadata-->B
    B--2-1. Generate---E
    B--2-2. Generate---D
    E--3. Dependency Inject-->D
    A--4. getBean-->B
    B-.5. Bean.->A
</div>



### Bean Scope

#### Singleton

스프링 컨테이너는 기본적으로 단일 빈 정의에 대해서 단 하나의 빈 객체(인스턴스)만을 관리한다.

이는 GoF 디자인 패턴의 싱글톤 패턴에 해당하며 컨텍스트에서 단일 인스턴스만을 참조하여 객체의 중복생성을 피하고 관리를 쉽게 해준다.

```xml
<!-- 두 가지는 서로 동일(기본 스코프는 싱글톤) -->
<bean id="eye" class="com.csupreme19.springdemo.Eye" />
<bean id="eye" class="com.csupreme19.springdemo.Eye" scope="singleton" />
```

```java
ApplicationContext ctx = new ClassPathXmlApplicationContext("applicationContext.xml");
Cat myCat = ctx.getBean("cat", Cat.class);
Cat yourCat = ctx.getBean("cat", Cat.class);
System.out.println(myCat.toString());
System.out.println(yourCat.toString());

// 서로 같은 메모리를 참조하는 동일 인스턴스임을 확인
com.csupreme19.springdemo.Cat@1ed4ae0f
com.csupreme19.springdemo.Cat@1ed4ae0f
```



#### Prototype

스프링에서는 또다른 빈 스코프가 있는데 Prototype 스코프이다.

객체가 참조될 때마다 새로운 인스턴스가 생성된다.

```xml
<bean id="eye" class="com.csupreme19.springdemo.Eye" scope="prototype" />
```

```java
Cat myCat = ctx.getBean("cat", Cat.class);
Cat yourCat = ctx.getBean("cat", Cat.class);
System.out.println(myCat.toString());
System.out.println(yourCat.toString());
System.out.println(myCat.eye.toString());
System.out.println(yourCat.eye.toString());

// 서로 다른 인스턴스임을 확인
com.csupreme19.springdemo.Cat@1ed4ae0f
com.csupreme19.springdemo.Cat@54c5a2ff
// 하지만 주입된 singleton 빈은 같은 인스턴스임을 확인할 수 있다.
com.csupreme19.springdemo.Eye@6d4d66d2
com.csupreme19.springdemo.Eye@6d4d66d2
```

> **주의사항**
>
> Prototype 스코프 빈의 경우 스프링 컨테이너에서 라이프사이클을 보장하지 않는다.



singleton, prototype 이외에도 request, session, global-session 스코프가 존재한다.



### Bean Lifecycle

<div class="mermaid">
  sequenceDiagram
    participant A as Container Start
    participant B as Bean Instantiate
    participant C as Dependencies Inject
    participant D as Internal Spring Process
    participant E as Init Method
    participant F as Destroy Method
    participant G as Bean Destroy
  A->>B: Instantiate beans
B->>C: Inject dependencies
C->>D: Bean factory
D->>E: Custom initialization
loop until container stop
E->F: Beans ready to use
end
F->>G: Custom destruction
</div>



Bean은 다음과 같은 생명주기를 갖는다.

1. 스프링 컨테이너 시작
   - 어플리케이션이 시작되면 스프링 컨테이너도 함께 시작한다.
2. 메타데이터에 의해 설정된 빈들을 생성한다.
   - XML 설정, @ComponentScan 등에 의해 설정된 메타데이터를 기반으로 Bean을 스프링 컨테이너에 생성한다.
3. 해당 빈에 대한 의존성을 주입한다.
   - 생성된 Bean들을 의존성으로 가지고 있는 빈에 의존성을 주입한다.
4. 내부 프로세스 진행
   - 빈 팩토리와 같은 post-processor를 실행한다.
5. init method 실행
   - 해당 Bean에 설정된 init method를 실행한다.
6. 스프링 컨테이너 중지
   - 어플리케이션이 중지되면 스프링 컨테이너도 함께 중지된다.
7. destroy method 실행
   - 해당 Bean에 설정된 destroy method를 실행한다.



```xml
	<bean id="cat" class="com.csupreme19.springdemo.Cat" init-method="printInit" destroy-method="printDestroy">
		<constructor-arg ref="eye"/>
	</bean>
```

```java
public class Cat {
	
	private String name;
	private String gender;
	public Eye eye;
	
	public Cat() {}
	
	public Cat(Eye eye) {
		this.eye=eye;
	}
	
	public void printInit() {
		System.out.println("init-method!");
	}
	
	public void printDestroy() {
		System.out.println("destroy-method!");
	}
}

Cat myCat = ctx.getBean("cat", Cat.class);
```

```sh
init-method!
destroy-method!
```

별도의 메서드를 호출하지 않았으나 init method와 destroy method가 호출된 것을 확인할 수 있다.

---

## Spring AOP

![sf-5.png]({{ "/assets/img/contents/sf-5.png"}})

Spring Framework의 또 다른 핵심 기능인 AOP를 담당한다.



### AOP가 도대체 뭐야?

AOP는 Aspect Oriented Programming 관점 지향 프로그래밍이다.

어떤 로직에서 부가 기능(관심사)을 분리한 관점으로 보는 것

![sf-6.png]({{ "/assets/img/contents/sf-6.png"}})



핵심 기능(비즈니스 로직)에서 부가 기능(로깅, 트랜잭션, 예외)을 분리하기 위하여 횡단 분리하는 것(관점)



##### A는 로그에 관심이 있다.

##### B는 보안 설정에 관심이 있다.

##### C는 예외 처리에 관심이 있다.



따라서 공통 기능을 관점 지향으로 바라보고 해당 기능을 비즈니스 로직에 영향을 주지 않으면서 적용할 수 있다.



### AOP를 왜 써?

아래와 같은 DAO가 있다고 하자.

```java
public void addAccount(Account account) {
	Session session = sessionFactory.getCurrentSession();
	currentSession.save(account);
}
```



로깅과 보안 처리에 대한 요구사항이 들어왔다.

아래와 같이 코드는 늘어날 것이다.

```java
public void addAccount(Account account, String userId) {
  // 로깅 코드 
  // 보안 체크
	Session session = sessionFactory.getCurrentSession();
	currentSession.save(account);
}
```

보안 체크시 사용하기 위한 userId라는 새로운 인자가 추가되었다.

로깅을 DAO뿐만이 아니라 서비스와 컨트롤러에도 적용하고 싶다면 아래와 같이 코드는 늘어날 것이다.



```java
public class AccountService{
// 로깅 코드
// 보안 체크
accountDAO.addAccount(account, userId);
}

public class AccountController{
// 로깅 코드
// 보안 체크
accountService.addAccount(account, userId);
}
```

동일한 기능을 하는 중복코드가 벌써 3개나 생겨버렸다.



만약 이 기능을  addAccount 메소드가 아닌 모든 메서드에 적용하고 싶다면?

해당 기능을 구현했는데 보안 체크 로직을 바꿔달라는 요구사항이 온다면?

위 3 중복 로직을 모두 변경하여야한다.

해당 클래스가 2, 3개라면 큰 문제가 없겠지만 100개 아니 1000개라면 상상도 하기 싫다.



### OOP로 해결해보자, OOP의 한계

#### 1. 상속을 쓰면 되잖아

객체지향이 좋은게 뭐겠어? => 상속과 다형성이지

```java
public class Logging {
  public void logging(){
    // logging
  }
}

public class AccountService extends Logging {
  logging();
	accountDAO.addAccount(account, userId);
}

public class AccountController extends Logging {
  logging();
	accountService.addAccount(account, userId);
}
```

Logging 기능을 담당하는 부모 클래스를 만들고 로깅을 필요로하는 모든 클래스에 대하여 상속을 하였다.

**문제는 Java에서는 다중 상속을 지원하지 않는다.**

별도의 인터페이스를 상속 받는 클래스에는 사용이 불가능하다.

마찬가지로 다중상속이 안되어 보안 처리에 대한 클래스를 별도로 분리할 수 없다.



#### 2. 위임(Delegation)

그럼 상속이 아닌 위임으로는?

```java
public class Logging {
  public void logging(){
    // logging
  }
}

public class Security {
  public void checkSecurity(){
    // security
  }
}

public class AccountServiceImpl implements AccountService {
  Logging logger = new Logging();
  Security security = new Security();
  logger.logging();
  security.checkSecurity();
	accountDAO.addAccount(account, userId);
}

public class AccountController extends Logging {
  Logging logger = new Logging();
  Security security = new Security();
  logger.logging();
  security.checkSecurity();
	accountService.addAccount(account, userId);
}
```

언뜻 보기에는 위 상속 버전에서 발생한 다중상속을 해결하여 문제를 해결한 것 처럼 보이나

결론적으로는 코드의 복잡도만 늘어났고 로깅/보안 체크 수정시에는 모든 클래스를 수정해야하는 문제가 있다.



### AOP를 사용해보자

#### 장점

1. AOP를 사용한다면 Aspect 코드를 단일 클래스에서 관리할 수 있다.
   - 로깅 코드를 수정하고 싶다면 해당 Aspect만 수정하면 되는 것이다.
2. 소스코드의 비즈니스 로직과 부가 기능 로직을 분리할 수 있다.
   - 비즈니스 로직 기능에만 충실하도록 코드를 작성할 수 있어 가독성이 높아진다.
3. 어플리케이션 코드 수정 없이 원하는 부분에만 적용이 가능하다.
   - 예외 발생시, 메서드 호출 전, 후 등



#### 단점

1. 많은 수의 Aspect는 어플리케이션 호출 흐름을 이해하기 어렵게 만든다.
   - 관점 지향 프로그래밍에서는 비즈니스 로직과 Aspect 로직이 분리되므로 많은 부분에 AOP가 적용된다면 흐름을 이해하기 힘들 수 있다.
2. 아주 약간의 성능 저하(런타임 Weaving의 경우에만 한함)
   - 런타임에 위빙을 하는 경우 실행시 미미하지만 약간의 성능 지연이 발생할 수 있다.



### AOP 용어 정리

#### Advice

What?: 어떤 액션을 취할지 정의한 공통 코드

4가지 타입이 있다.

| 종류            | 시점                           |
| --------------- | ------------------------------ |
| Before          | 메소드 실행 전                 |
| After finally   | 메소드 실행 후(finally)        |
| After returning | 메소드 실행 후(success)        |
| After throwing  | 메소드 실행 후(exception)      |
| Around          | 메소드 실행 전, 메소드 실행 후 |

#### Join Point

When?: 해당 코드에 언제 적용되는지 정의한 포인트

#### Pointcut

표현식을 통해 필터링한 특정 조인포인트로 적용 대상을 결정

#### Aspect

어떤 포인트컷에 어떤 어드바이스를 취할지 정의된 코드 모듈



#### Weaving

아스펙트를 목표 타겟 객체에 연결하는 것

컴파일타임 위빙, 로드타임 위빙, 런타임 위빙이 있다.

> 런타임 위빙이 가장 느리다.



### AOP 프레임워크

Java의 대표적인 두 가지 AOP 프레임워크를 지원한다.

### Spring AOP

![spring-icon-logo.png]({{ "/assets/img/titles/spring-icon-logo.png"}})

<div class="mermaid">
  flowchart LR
    A[Application]
    B[AOP Proxy]
    C[Logging Aspect]
    D[Security Aspect]
    E[Target Object]
    A-->B---C---D-->E
</div>



Spring 프레임워크에서 기본적으로 제공하는 AOP이며 일반 AOP 인터페이스를 기반으로한 새로운 인터페이스이다.

스프링 프레임워크 뒷단에서는 보안, 트랜잭션, 캐싱등을 구현할때 Spring AOP를 사용한다고 한다.

AOP Proxy를 통하여 Aspect 위빙된 타겟 어플리케이션과 통신한다.



#### 장점

1. AspectJ 보다 사용이 간단하다.
2. 프록시 패턴을 사용한다.
3. @Aspect 애노테이션을 통해 AspectJ로의 이관이 쉽게 가능하다.

#### 단점

1. 메소드 레벨의 조인포인트만 가능하다.
2. 스프링 컨테이너의 Bean에만 적용이 가능하다.
3. 런타임 위빙으로 약간의 성능 저하가 있을 수 있다.



### AspectJ

[https://www.eclipse.org/aspectj](https://www.eclipse.org/aspectj)

AOP 인터페이스의 확실한 구현체 프레임워크로 2001년에 발표된 오리지널 AOP 프레임워크이다.

AOP 기능의 모든 기능을 지원하며 생성자, 필드 joinpoint 및 컴파일 타임, 로드타임 위빙을 지원한다.



#### 장점

1. 모든 조인포인트를 지원한다.
2. 스프링 빈뿐만이 아니라 POJO도 지원한다.
3. Spring AOP에 비해 더 좋은 성능을 가진다.
4. AOP의 모든 기능을 제공한다.

#### 단점

1. 컴파일타임 위빙은 추가적인 컴파일 단계를 필요로함
2. 포인트컷 문법이 좀 더 복잡하다.



### Spring AOP vs AspectJ

| 비교군     | AspectJ                                 | Spring AOP  |
| ---------- | --------------------------------------- | ----------- |
| 조인포인트 | 메소드, 생성자, 필드                    | 메소드      |
| 위빙       | 컴파일타임, 포스트 컴파일타임, 로드타임 | 런타임      |
| 성능       | 비교적 빠름                             | 비교적 느림 |
| 타겟       | Spring Bean, POJO                       | Spring Bean |



---

## Reference

1. [https://docs.spring.io/spring-framework/docs/current/reference/html/overview.html#overview](https://docs.spring.io/spring-framework/docs/current/reference/html/overview.html#overview)
2. [https://docs.spring.io/spring-framework/docs/5.0.0.M5/spring-framework-reference/html/overview.html](https://docs.spring.io/spring-framework/docs/5.0.0.M5/spring-framework-reference/html/overview.html)
3. [https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#spring-core](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#spring-core)
4. [https://www.udemy.com/course/spring-hibernate-tutorial/](https://www.udemy.com/course/spring-hibernate-tutorial/)

