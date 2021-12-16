---
layout: post
title: Spring Web 정리
feature-img: assets/img/titles/spring-logo.svg
thumbnail: assets/img/contents/sf-9.png
author: csupreme19
tags: [Spring, Java, MVC, Web, Servlet]

---

# Spring Web 정리

![spring-logo.svg]({{ "/assets/img/titles/spring-logo.svg"}})

본 문서에서는 스프링 웹에 대하여 정리해보았다.

---

## Spring Web(Spring MVC)

![sf-9.png]({{ "/assets/img/contents/sf-9.png"}})

- `spring-web`
  - 서블릿 리스너, 웹 기반 IoC 컨테이너, 웹 Context, HTTP 클라이언트를 제공한다.
- `spring-webmvc`
  - Spring MVC라고도 부르며 Spring MVC와 REST 웹서비스 구현체를 포함한다.
  - 화면단과 도메인 모델, 로직을 구분하기 위하여 Model, View, Controller를 나누어 사용하는 MVC 패턴이 적용된다.

---

### 개요

Spring Web은 자바에서 웹 애플리케이션 개발에 필요한 여러 기능들이 포함된 모듈(프레임워크)이다.

MVC란 Model-View-Controller로 이루어지는 웹 개발시 사용하는 디자인 패턴이며

스프링은 MVC 패턴을 활용하여 웹을 구성하기 때문에 Spring MVC라고도 불린다.

---

### Model View Controller(MVC)

![sf-10.png]({{ "/assets/img/contents/sf-10.png"}})

Model(데이터), View(화면), Controller(로직) 크게 세가지 레이어로 분리하여 처리한다.

웹 브라우저에서 프론트 컨트롤러로 요청을 전달하고 프론트 컨트롤러에서는 모델을 생성하여 컨트롤러에게 전달한다.

컨트롤러에서는 비즈니스 로직이 실행되며 해당 로직 실행 후 모델은 다시 프론트 컨트롤러에 전달되어

프론트 컨트롤러는 뷰 템플릿에 모델을 전달한다.

해당 뷰 템플릿에 전달된 모델을 토대로 페이지를 렌더링하여 사용자 브라우저에게 전달하게 된다.

---

#### Front Controller

스프링 MVC에서는 DispatcherServlet이라는 이름으로 개발되어 제공되고 있다.

따라서 직접 구현하는 경우는 드물며 DispatcherServlet과 동일하게 부르기도 한다.

<br>

#### Controller

실제 비즈니스 로직이 구현되는 부분이다.

요청을 받아 데이터를 처리하여 모델에 담아 뷰 템플릿으로 보내는 역할을 수행한다.

<br>

#### Model

DB, 컨트롤러 등에서 처리되어 가져온 실제 데이터가 담기는 컨테이너이다.

<br>

#### View Template

Model을 기반으로 처리하여 보여지는 화면단을 처리하는 템플릿이다.

여러 템플릿 엔진을 지원하며 가장 많이 사용하는 것은 JSP와 JSTL이다.

> JSP: Java Servlet Page
>
> JSTL: JSP Standard Tag Library

---

### Spring MVC를 왜 사용할까?

위에서 말했던 Spring의 핵심 기능을 사용할 수 있다.

- IoC, DI를 이용하여 UI의 재사용성이 높아진다.
- Model(데이터), View(화면), Controller(로직)으로 레이어를 분리하여 결합도를 낮춘다.
- 웹 기반 IoC 컨테이너, 웹 컨텍스트를 사용하여 애플리케이션 상태를 쉽게 관리할 수 있다.
- HTTP 클라이언트, REST 클라이언트 등의 구현체를 제공한다.
- 뷰 레이어의 다양한 확장성을 제공하여 다양한 템플릿 엔진 사용 가능하다.(서블릿, 머스타치, 타임리프, 벨로시티, 프리마커 등)

---

### Spring MVC 설정하기(XML 방식)

#### 1. web.xml 설정하기

```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xmlns="http://xmlns.jcp.org/xml/ns/javaee"
	xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee http://xmlns.jcp.org/xml/ns/javaee/web-app_3_1.xsd"
	id="WebApp_ID" version="3.1">
  
	<!-- 해당 웹앱의 이름을 설정 -->
	<display-name>spring-mvc-demo</display-name>

	<!-- 스프링 MVC 디스패처 서블릿을 설정한다 -->
	<servlet>
		<servlet-name>dispatcher</servlet-name>
		<servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
    	<!-- 해당 XML 파일 정보로 디스패처 서블릿을 생성 -->
		<init-param>
			<param-name>contextConfigLocation</param-name>
			<param-value>/WEB-INF/dispatcherServlet.xml</param-value>
		</init-param>
	</servlet>

	<!-- 해당 디스패처 서블릿이 처리할 URL 패턴을 지정한다 -->
	<servlet-mapping>
		<servlet-name>dispatcher</servlet-name>
		<url-pattern>/</url-pattern>
	</servlet-mapping>
	
</web-app>
```

web.xml은 웹 애플리케이션의 배포 정보를 담고 있는 파일이다.

위에서 말했듯이 Dispatcher Servlet은 스프링 개발팀에 의해 이미 개발구현되어 있으며 우리는 이것을 가져다 사용하면 된다.

1. dispatcher라는 이름의  DispatcherServlet 서블릿을 설정한다.
2. 해당 디스패처 서블릿이 매핑되는 url 패턴을 지정한다.

루트 url로 지정했으므로 모든 요청에 대하여 해당 디스패처 서블릿이 처리하게 된다.

디스패처 서블릿을 여러개 두어 각 URL 요청을 분리하여 처리할 수 있음을 알 수 있다.

#### 2. dispatcherServlet.xml 설정하기

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" 
	xmlns:context="http://www.springframework.org/schema/context"
	xmlns:mvc="http://www.springframework.org/schema/mvc"
	xsi:schemaLocation="
		http://www.springframework.org/schema/beans
    	http://www.springframework.org/schema/beans/spring-beans.xsd
    	http://www.springframework.org/schema/context
    	http://www.springframework.org/schema/context/spring-context.xsd
    	http://www.springframework.org/schema/mvc
        http://www.springframework.org/schema/mvc/spring-mvc.xsd">

	<!-- 스프링 컴포넌트 스캔을 위한 설정 -->
	<context:component-scan base-package="com.csupreme19.springmvc" />

	<!-- 스프링 MVC 컴포넌트를 사용하기 위한 설정 -->
	<mvc:annotation-driven/>

	<!-- 스프링 MVC 뷰 리졸버를 사용하기 위한 설정 -->
	<bean
		class="org.springframework.web.servlet.view.InternalResourceViewResolver">
		<property name="prefix" value="/WEB-INF/view/" />
		<property name="suffix" value=".jsp" />
	</bean>

</beans>
```

![sf-11.png]({{ "/assets/img/contents/sf-11.png"}})

뷰 리졸버에서 해당 뷰 페이지를 찾기 위하여 경로를 설정할 때 prefix와 suffix 경로를 붙여서 사용한다.

#### 3. Controller 작성

```java
@Controller
public class MainController {
	@RequestMapping("/")
	public String mainPage() {
		return "main";
	}
}
```

#### 4. View page 작성

`/WEB-INF/view/main.jsp`

```jsp
<%@ page language="java" contentType="text/html; charset=UTF-8"
    pageEncoding="UTF-8"%>
<!DOCTYPE html>
<html>
<head>
<meta charset="UTF-8">
<title>Insert title here</title>
</head>
<body>
Hi
</body>
</html>
```

---

### Spring MVC 설정하기(Annotation 방식)

#### 1. WebMvcConfig 작성

```java
@Configuration 
@EnableWebMvc
@ComponentScan(basePackages="com.csupreme19.springmvc")
public class WebMvcConfig {
	@Bean
	public ViewResolver viewResolver() {
		InternalResourceViewResolver viewResolver = new InternalResourceViewResolver();
		viewResolver.setPrefix("/WEB-INF/view/");
		viewResolver.setSuffix(".jsp");
		return viewResolver;
	}
}
```

`@Configuration`: `@Bean` 등록을 위한  Configuration 설정

`@EnableWebMvc`: Spring MVC 설정(@Controller, @RequestMapping 등)을 사용하기 위한 설정, <mvc:annotation-driven />과 같은 역할

`@ComponentScan`: 등록된 컴포넌트를 사용하기 위한 설정

 ViewResolver를 빈으로 등록해준다.

#### 2. DispatcherServletInitializer 작성

```java
public class DispatcherServletInitializer extends AbstractAnnotationConfigDispatcherServletInitializer {
	@Override
	protected Class<?>[] getRootConfigClasses() {
		return null;
	}

	@Override
	protected Class<?>[] getServletConfigClasses() {
		return new Class[] {WebMvcConfig.class};
	}
  
	@Override
	protected String[] getServletMappings() {
		return new String[] {"/"};
	}
}
```

`AbstractAnnotationConfigDispatcherServletInitializer`을 상속받아 서블릿 이니셜라이저를 작성한다.

해당 디스패처 서블릿이 사용할 Config 클래스를 1번에서 만든 클래스로 지정해준다.

서블릿 매핑을 설정해준다.

#### 3. Controller 작성

```java
@Controller
public class MainController {
	@RequestMapping("/")
	public String mainPage() {
		return "main";
	}
}
```

#### 4. View page 작성

```jsp
<%@ page language="java" contentType="text/html; charset=UTF-8"
    pageEncoding="UTF-8"%>
<!DOCTYPE html>
<html>
<head>
<meta charset="UTF-8">
<title>Insert title here</title>
</head>
<body>
<h2>Hi!</h2>
</body>
</html>
```

---

### Annotation vs XML

![sf-12.png]({{ "/assets/img/contents/sf-12.png"}})

![sf-13.png]({{ "/assets/img/contents/sf-13.png"}})

XML 방식과 애너테이션 방식은 위와 같이 대응된다.

XML 명세를 사용하는 방식은 옛 방식으로 아직도 사용하긴 하지만

일반적으로 최근에는 애너테이션 기반으로 개발한다.

---

## Reference

1. [docs.spring.io overview](https://docs.spring.io/spring-framework/docs/current/reference/html/overview.html#overview)
2. [docs.spring.io 5.0.0 overview](https://docs.spring.io/spring-framework/docs/5.0.0.M5/spring-framework-reference/html/overview.html)
3. [docs.spring.io spring-core](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#spring-core)
4. [Udemy Spring & Hibernate for Beginners](https://www.udemy.com/course/spring-hibernate-tutorial/)
5. [스프링 철저 입문](http://www.yes24.com/Product/Goods/59192207)

