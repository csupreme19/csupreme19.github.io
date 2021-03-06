---
layout: post
title: Spring Core 정리
feature-img: assets/img/titles/spring-logo.svg
thumbnail: assets/img/contents/sf-3.png
author: csupreme19
categories: Development Spring
tags: [Spring, Java, IoC, DI, Singleton]

---

# Spring Core 정리

![spring-logo.svg]({{ "/assets/img/titles/spring-logo.svg"}})

본 문서에서는 스프링 프레임워크의 스프링 코어에 대하여 정리해봤어요.

---

## Spring Core

![sf-3.png]({{ "/assets/img/contents/sf-3.png"}})

Spring의 기본이자 핵심 모듈로 Spring Container, DI Container예요.

스프링 프레임워크에서 지원하는 IoC, DI 등의 개념의 주체인 Bean을 관리하는 핵심 기능을 담당해요.

> **Spring Bean?**
>
> Spring Framework에서 관리하는 자바 객체로 일반 자바 클래스와 같다고 생각하면 돼요.
>
> Spring에서는 기존의 EJB 방식의 복잡한 구현을 피하기 위하여 POJO를 사용하는데 
>
> 해당 POJO가 Spring Container에 의해 생성되고 관리되면 해당 객체는 Spring Bean이라고 불러요.

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

---

### 개요

Spring Framework의 핵심 모듈인 Spring Core에 대해서 알아볼 예정이에요.

일반적으로 스프링 프레임워크의 IoC, DI 등의 핵심 기능을 해당 모듈에서 담당하며

스프링 컨테이너, DI 컨테이너 그 자체라고 보아도 무방하다고 생각해요.

---

### Inversion Of Control(IoC) 원칙

**제어의 역전(역전제어); 객체의 생성을 객체 그 자신에 맡기는 것 **

<br>

#### IoC 개념 도입 전

<div class="mermaid">
  flowchart LR
    A[Application]
    B[Class A]
    A--new Class A-->B
    B-.return Instance A.->A
</div>


```java
// POJO
public class Cat {
	public Cat() {
    System.out.println("Constructed!");
  }
}

Cat myCat = new Cat();

// 출력
Constructed!
```

기존 초기 자바 어플리케이션의 경우 위와 같이 클래스 자체가 객체의 생성과 초기화 등의 라이프사이클을 관리해주었어요.

이는 생성과 사용을 동일 객체에서 하기 때문에 단일 책임 원칙에 위배됩니다.

생성된 객체를 여러 클래스에서 사용 및 관리하기가 힘들고 또한 여러개의 중복된 객체가 생성되는 단점이 있어요.

<br>

#### IoC 개념 도입 후

<div class="mermaid">
  flowchart LR
    A[Application]
    B[Spring Container]
    C[XML Configuration]
    D[Bean]
    C--1. Metadata-->B
    B--2. Generate---D
    A--3. getBean-->B
    B-.4. Bean.->A
</div>
<br>

![sf-4.png]({{ "/assets/img/contents/sf-4.png"}})

이를 해결하기 위하여 스프링에서는 모든 객체의 생성과 관리를 객체 그 자신에게 맡기는데 

**제어가 역전** 되었다고 하여 **Inversion of Control**이라고 부릅니다.

이 특성을 통하여 클래스에서 더 이상 객체의 라이프사이클에 대하여 신경쓰지 않고 사용할 수 있어요.

<br>

#### Bean 설정하기(XML)

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

// 출력
Constructed!
```

빈은 어플리케이션의 실행과 종료까지의 생명주기를 가지는 ApplicationContext에 등록되어 사용되고 해당 어플리케이션 컨텍스트는 XML 파일, 애너테이션, 자바 코드 등으로 설정할 수 있어요.

컨텍스트 생성 시점 Bean들을 생성/주입하여 초기화하며 컨텍스트에서 `getBean()` 호출시 스프링 컨테이너에 객체가 존재하는지 확인하고 객체를 가져오는 구조예요.


이를 통해 개발자는 더 이상 객체의 생성과  관리에 대해서 걱정할 필요가 없이 싱글턴 객체를 사용할 수 있어요.

---

### Dependency Injection(DI)

**의존성 주입; 객체가 생성될 때 자신의 의존성을 주입하는 것**

사실 IoC랑 DI는 따로 가는 개념이 아니라 같은 IoC가 DI고 DI가 IoC에요.

역전 제어를 하기 위하여 필요한 프로세스가 DI이기 때문에 IoC 개념을 위한 프로세스라고 보시면 될 것 같아요.

스프링 컨테이너에 빈을 등록하여 관리하는데 해당 Bean이 POJO 형태가 아니라 여러 Dependency를 가지고 있으면 어떻게 될까요?

해당 객체는 스프링 컨테이너에서 사용 못하고 개발자가 직접 수동으로 생성해야 할까요?

모두들 알고 있듯이 객체지향 관점에서 대부분의 객체는 의존성을 가지고 있어요.

<div class="mermaid">
  flowchart LR
    A[Application]
    B[Spring Container]
    C[XML Configuration]
    D[Cat Bean]
    E[Eye Bean]
    C--1. Metadata-->B
    B--2. Generate---E
    B--2. Generate---D
    E--3. Dependency Inject-->D
    A--4. getBean-->B
    B-.5. Bean.->A
</div>

1. 초기 스프링 컨테이너 구동시 스프링 컨테이너에 의해 모든 Bean이 생성돼요.
2. 이후 DI를 통해 의존성을 주입하여 의존성이 주입된 Bean이 준비돼요.
3. 스프링 컨테이너에서 getBean을 통하여 사용하는 모든 Bean은 DI가 완료된 빈이에요.

<br>

#### Constructor Dependency Injection 예시

```xml
<bean id="eye" class="com.csupreme19.springdemo.Eye" />
<bean id="cat" class="com.csupreme19.springdemo.Cat">
	<!-- 의존성 주입 -->
	<constructor-arg ref="eye"/>
</bean>
```

```java
public class Cat {
	
	private String name;
	private String gender;
	public Eye eye;
	
	public Cat() {
    System.out.println("Constructed!");
  }
	
	public Cat(Eye eye) {
    System.out.println("Constructed!");
		this.eye=eye;
	}
}

Cat myCat = ctx.getBean("cat", Cat.class);
System.out.println(myCat.toString());
System.out.println(myCat.eye.toString());
```

Cat에 Eye라는 의존성을 추가했어요.

스프링의 DI를 사용하려면 해당 클래스 또한 빈으로 등록되어 있어야 이후 DI 시점에 주입이 될 수 있어요.

스프링이 구동되면 빈으로 등록된 객체는 본인의 생성자 또는 팩토리 메서드로 인스턴스를 생성하는데 

이 과정에서 의존성이 있는 다른 객체를 주입하게 됩니다.

---

### Bean Scope

#### Singleton

<div class="mermaid">
  flowchart LR
    A[Class B]
    B[Class C]
    C[Class D]
    D[Spring Container]
    F[Instance A]
    A--getBean-->D
    B--getBean-->D
    C--getBean-->D
    D<-.manage.->F
</div>

스프링 컨테이너는 기본적으로 단일 빈 정의에 대해서 단 하나의 빈 객체(인스턴스)만을 관리해요.

이는 GoF 디자인 패턴의 싱글톤 패턴에 해당하며 컨텍스트에서 단일 인스턴스만을 참조하여 객체의 중복생성을 피하고 관리를 쉽게 해줘요.

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

<br>

#### Prototype

<div class="mermaid">
  flowchart LR
    A[Class B]
    B[Class C]
    C[Class D]
    subgraph Spring Container
    D[Spring Container]
    I[Spring Container]
    J[Spring Container]
    end
    F[Instance A]
    G[Instance A']
    H[Instance A'']
    A--getBean-->D
    B--getBean-->I
    C--getBean-->J
    D<-.manage.->F
    I<-.manage.->G
    J<-.manage.->H
</div>


스프링에서는 또다른 빈 스코프가 있는데 Prototype 스코프로

각 스코프별로 컨테이너에 객체가 참조될 때마다 새로운 인스턴스가 생성되는 구조예요.

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
> Prototype 스코프 빈의 경우 스프링 컨테이너에서 라이프사이클을 보장하지 않아요.

singleton, prototype 이외에도 request, session, global-session 등의 스코프가 존재해요.

---

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

Bean은 다음과 같은 생명주기를 가지게 돼요.

1. 스프링 컨테이너 시작
   - 어플리케이션이 시작되면 스프링 컨테이너도 함께 시작해요.
2. 메타데이터에 의해 설정된 빈들을 생성해요.
   - XML 설정, @ComponentScan 등에 의해 설정된 메타데이터를 기반으로 Bean을 스프링 컨테이너에 생성해요.
3. 해당 빈에 대한 의존성을 주입해요.
   - 생성된 Bean들을 의존성으로 가지고 있는 빈에 의존성을 주입해요.
4. 내부 프로세스 진행
   - 빈 팩토리와 같은 post-processor를 실행해요.
5. init method 실행
   - 해당 Bean에 설정된 init method를 실행해요.
6. 스프링 컨테이너 중지
   - 어플리케이션이 중지되면 스프링 컨테이너도 함께 중지돼요.
7. destroy method 실행
   - 해당 Bean에 설정된 destroy method를 실행해요.

<br>

#### 생명주기 확인

```xml
<bean id="testBean" class="com.csupreme19.springdemo.TestBean" init-method="printInit" destroy-method="printDestroy" />
```

```java
public class TestBean {
	
	public TestBean() {
		System.out.println(this.getClass() + " Constructed!");
	}
	
	public void printInit() {
		System.out.println(this.getClass() + " Init bean!");
	}
	
	public void printDestroy() {
		System.out.println(this.getClass() + " Destroy bean!");
	}
}

ApplicationContext ctx = new ClassPathXmlApplicationContext("applicationContext.xml");
TestBean testBean = ctx.getBean("testBean", TestBean.class);
System.out.println("Closing Application Context!");
((AbstractApplicationContext) ctx).close();
System.out.println("Closed Application Context!");
```

```sh
class com.csupreme19.springdemo.TestBean Constructed!
class com.csupreme19.springdemo.TestBean Init bean!
com.csupreme19.springdemo.TestBean@54e1c68b
Closing Application Context!
class com.csupreme19.springdemo.TestBean Destroy bean!
Closed Application Context!
```

생성자 -> Bean 초기화 메서드 -> 빈 확인 -> 컨테이너 종료시 빈 종료 확인

위 순서로 라이프사이클이 존재하는 것을 확인할 수 있었어요.

---

### Spring Context with Annotation

위의 applciation context xml 파일을 사용하지 않고 Annotation을 이용하여 스프링 컨테이너를 설정하는 방법도 있어요.

들어가기 전에 혼동이 올 수 있는 애너테이션을 확실히 정리하고 가는 것이 좋을 것 같네요.

<br>

#### @Component vs @Bean

개발자가 직접 생성한 클래스를 빈의 형태로 등록하여 사용하고자 하는 경우 `@Component`

```java
@Component
public class UserDAO {
  public User getUser(String id, String name, String age) {
    return new User(id, name, age);
  }
}
```

> @Component 애너테이션은 타겟이 클래스로 정해져있어 애초에 메서드 형태로 선언이 불가능해요.

외부 라이브러리에서 생성되어 스프링 컨테이너에 빈 형태로 등록하여 사용하고자 하는 경우 `@Bean`을 주로 사용해요.

직접 생성한 클래스를 Bean으로 등록하여 사용할 수는 있으나 클래스 레벨에서 설정이 불가능 하기 때문에 일반적으로는 위와 같이 사용해요.

```java
@Configuration
public class UserConfig {
  @Bean
  public ExternalUser externalUser() {
    return new ExternalUser();
  }
}
```

>  `@Bean` 애너테이션은 타겟이 메서드로 지정되어 있어 애초에 클래스에 설정이 불가능해요.

<br>

#### 1. AppConfig 작성

```java
@Configuration
@ComponentScan("com.csupreme19.springdemo")
public class AppConfig {
}
```

ApplicationContext 생성시 사용할 Configuration을 작성하는 부분이에요.

`@Configuration` 

- 해당 클래스가 여러개의  `@Bean`을 가지고 있을때 설정하며 현재 빈이 존재하지 않지만 추후 설정시 사용하기 때문에 `@Configuration`을 사용했어요.

`@ComponentScan(basePackage)` 

- 스프링에 주입되는 빈(컴포넌트)들을 스캔할 베이스 패키지를 설정한다는 뜻이에요. 
- 해당 패키지 안에 있는 `@Component`로 설정된 클래스들은 스프링 컨테이너 생성시 스프링 컨테이너에 의해 생성/주입 돼요.

#### 2. Component 작성

```java
@Component
public class UserDAO {
	public User getUser(String id, String name, String age) {
		return new User(id, name, age);
	}
}
```

스프링 컨테이너에 등록되는 컴포넌트(빈)을 작성하는 부분이에요.

#### 3. 스프링 컨텍스트 생성

```java
public class Application {
	public static void main(String[] args) {
		ApplicationContext ctx = new AnnotationConfigApplicationContext(AppConfig.class);
		UserDAO userDAO = ctx.getBean("userDAO", UserDAO.class);
		((AbstractApplicationContext)ctx).close();
	}
}
```

위에서 설정한 AppConfig를 기반으로  `AnnotationConfigApplicationContext`를 생성하는 부분이에요.

해당 컨텍스트에서 빈을 가져와 사용해요.

---

## Reference

1. [docs.spring.io overview](https://docs.spring.io/spring-framework/docs/current/reference/html/overview.html#overview)
2. [docs.spring.io 5.0.0 overview](https://docs.spring.io/spring-framework/docs/5.0.0.M5/spring-framework-reference/html/overview.html)
3. [docs.spring.io spring-core](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#spring-core)
4. [Udemy Spring & Hibernate for Beginners](https://www.udemy.com/course/spring-hibernate-tutorial/)
5. [스프링 철저 입문](http://www.yes24.com/Product/Goods/59192207)

