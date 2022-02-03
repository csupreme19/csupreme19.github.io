---
layout: post
title: Spring Web 둘 이상의 fragment들이 발견된 경우
feature-img: assets/img/titles/spring-logo.svg
thumbnail: assets/img/contents/smwf-1.png
author: csupreme19
categories: Development Spring
tags: [Spring, Servlet, Web]

---

# Spring Web 둘 이상의 fragment들이 발견된 경우

![spring-logo.svg]({{ "/assets/img/titles/spring-logo.svg"}})

Spring Web, Spring MVC를 사용할 때 서버 실행시 web_fragment가 둘 이상 발견되는 경우 해결방법을 정리했어요.

---

## 문제점

![smwf-1.png]({{ "/assets/img/contents/smwf-1.png"}})

#### 에러 메시지(한글)

```java
Caused by: java.lang.IllegalArgumentException: 이름이 [spring_web]인, 둘 이상의 fragment들이 발견되었습니다. 이는 상대적 순서배열에서 불허됩니다. 상세 정보는 서블릿 스펙 8.2.2 2c 장을 참조하십시오. 절대적 순서배열을 사용하는 것을 고려해 보십시오.
	at org.apache.tomcat.util.descriptor.web.WebXml.orderWebFragments(WebXml.java:2262)
	at org.apache.tomcat.util.descriptor.web.WebXml.orderWebFragments(WebXml.java:2220)
	at org.apache.catalina.startup.ContextConfig.webConfig(ContextConfig.java:1294)
	at org.apache.catalina.startup.ContextConfig.configureStart(ContextConfig.java:986)
	at org.apache.catalina.startup.ContextConfig.lifecycleEvent(ContextConfig.java:303)
	at org.apache.catalina.util.LifecycleBase.fireLifecycleEvent(LifecycleBase.java:123)
	at org.apache.catalina.core.StandardContext.startInternal(StandardContext.java:5135)
	at org.apache.catalina.util.LifecycleBase.start(LifecycleBase.java:183)
	... 27 more
```

#### 에러 메시지(영어)

```java
Caused by: java.lang.IllegalArgumentException: More than one fragment with the name [spring_web] was found. This is not legal with relative ordering. See section 8.2.2 2c of the Servlet specification for details. Consider using absolute ordering.
	at org.apache.tomcat.util.descriptor.web.WebXml.orderWebFragments(WebXml.java:2262)
	at org.apache.tomcat.util.descriptor.web.WebXml.orderWebFragments(WebXml.java:2220)
	at org.apache.catalina.startup.ContextConfig.webConfig(ContextConfig.java:1294)
	at org.apache.catalina.startup.ContextConfig.configureStart(ContextConfig.java:986)
	at org.apache.catalina.startup.ContextConfig.lifecycleEvent(ContextConfig.java:303)
	at org.apache.catalina.util.LifecycleBase.fireLifecycleEvent(LifecycleBase.java:123)
	at org.apache.catalina.core.StandardContext.startInternal(StandardContext.java:5135)
	at org.apache.catalina.util.LifecycleBase.start(LifecycleBase.java:183)
	... 27 more
```

Spring Web MVC 프로젝트 구동시 위와같은 오류 메시지가 발생하는 경우 원인을 알아보고 해결 방법을 찾아봤어요.

스프링을 기초부터 다시 정리하는 과정에서 스프링 프레임워크 jar 파일들을 직접 임포트하여 사용하였는데 해당 과정에서 spring_web 프래그먼트 충돌이 발생했어요.

---

## 원인 파악

{% include aligner.html images="/contents/smwf-2.png,/contents/smwf-3.png" column=2 %}

위와 같이 `spring-web-5.3.9.jar`와 `spring-web-5.3.9-sources.jar` 두 jar 파일에 `web-fragment.xml` 가 존재하는 것을 확인할 수 있었어요.

![smwf-4.png]({{"/assets/img/contents/smwf-4.png"}})

실제로 Tomcat 서버 정보를 보면 두개의 jar 파일이 물려있는 것과

각 라이브러리의 `web-frgment.xml`이 충돌되어 발생하는 문제인 것을 확인할 수 있었어요.

<br>

### `web-fragment.xml`이 뭘까?

`web-fragment.xml` 이란 `web.xml`의 논리적인 부분 파일이라고 보면 될 것 같아요.

서블릿 3.0부터 여러개의 xml 설정값을 지원하는데

서블릿 컨텍스트는 `WEB-INF/web.xml` 뿐만 아니라 라이브러리 jar 파일 안의 `META-INF/web-frgament.xml`도 함께 탐색을 한다고 하네요.

즉 라이브러리 그 자체의 `web.xml` 파일이라고 생각하면 될 것 같아요.

`web-fragment.xml`

```xml
<?xml version="1.0" encoding="ISO-8859-1"?>
<web-fragment xmlns="http://java.sun.com/xml/ns/javaee"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://java.sun.com/xml/ns/javaee https://java.sun.com/xml/ns/javaee/web-fragment_3_0.xsd"
	version="3.0" metadata-complete="true">

	<name>spring_web</name>
	<distributable/>

</web-fragment>
```

두 jar 파일에 있는 `web-fragment.xml` 설정값

`web.xml`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xmlns="http://xmlns.jcp.org/xml/ns/javaee"
	xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee http://xmlns.jcp.org/xml/ns/javaee/web-app_3_1.xsd"
	id="WebApp_ID" version="3.1">
...	
</web-app>
```

Spring MVC web.xml과의 차이점을 살펴볼까요

![smwf-5.png]({{"/assets/img/contents/smwf-5.png"}})

![smwf-6.png]({{"/assets/img/contents/smwf-6.png"}})

위 두 부분이 차이나는 것을 확인할 수 있어요.

`web.xml`이 루트 엘리먼트로 web-app을 바라보고 있는 것과 달리 `web-fragment.xml`은 web-fragment 스키마를 루트로 두고 있는 것을 확인할 수 있었어요.

`spring-web-5.3.9.jar`와 `spring-web-5.3.9-sources.jar`에 담겨있는 `web-frgament.xml` 파일이 동일한 spring_web이라는 이름을 가지고 있으므로 서로 충돌이 발생한 것이네요.

---

## 해결 방법

### 1.  중복 jar 삭제

일반적인 경우가 아니라 여러 버전의 스프링 jar 라이브러리를 가지고 있는 경우에 발생하므로 실제 사용할 라이브러리 jar를 제외하고 충돌나는 jar 파일을 지우는 방법이에요.

`-sources.jar`의 경우 컴파일된 코드가 아닌 실제 소스 코드를 가져올 수 있는 라이브러리로 실제 앱 구동시 필요하지 않기 때문에

저의 경우에는 `spring-web-5.3.9-sources.jar`을 제거하여 해결하였어요.

<br>

### 2. 절대적 순서 사용

```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xmlns="http://xmlns.jcp.org/xml/ns/javaee"
	xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee http://xmlns.jcp.org/xml/ns/javaee/web-app_3_1.xsd"
	id="WebApp_ID" version="3.1">
  <!-- 추가 -->
	<absolute-ordering/>
</web-app>
```

web fragment 파일들을 불러오는 순서를 절대적 순서로 변경하여 문제를 해결할 수 있어요.

```
이름이 [spring_web]인, 둘 이상의 fragment들이 발견되었습니다. 이는 상대적 순서배열에서 불허됩니다. 상세 정보는 서블릿 스펙 8.2.2 2c 장을 참조하십시오. 절대적 순서배열을 사용하는 것을 고려해 보십시오.
```

현재 구성된 서블릿 3.1의 스펙 8.2.2 2c를 보면 다음과 같은데

```
c. Duplicate name exception: if, when traversing the web-fragments, multiple
members with the same <name> element are encountered, the application must
log an informative error message including information to help fix the
problem, and must fail to deploy. For example, one way to fix this problem is
for the user to use absolute ordering, in which case relative ordering is ignored.
```

이름이 같은 web-fragment가 존재할 경우 예외를 발생시키도록 규정되어 있어요.

기본값으로 web-fragment 순서들은 상대적 순서로 설정되어 있으며 이를 절대적 순서를 사용하여 문제를 해결하도록 규정하고 있어요.

위의 경우엔 web.xml 파일을 사용하고 있으므로 애너테이션 방식에서는 사용할 수 없다는 단점이 존재하네요.

---

### 참고

![smwf-7.png]({{"/assets/img/contents/smwf-7.png"}})

메이븐 같은 빌드 툴을 사용할 시에도 동일하게 발생하는 것으로 보이네요.

원래 자바 jar 라이브러리들간의 충돌은 일어나서는 안된다고 생각하는데

서블릿 스펙상 충돌이 일어나도록 라이브러리를 배포한 스프링의 이슈라고 보면 될 것 같아요.

> [https://repo.spring.io/ui/native/release/org/springframework/spring/5.3.9](https://repo.spring.io/ui/native/release/org/springframework/spring/5.3.9)
>
> spring repo에서 배포하는 dist 파일에 애초에 저 두 jar 파일이 같이 포함되어 있어요.

---

## Reference

1. [https://www.roseindia.net/servlets/servlet3/webfragmentsOrdering.shtml](https://www.roseindia.net/servlets/servlet3/webfragmentsOrdering.shtml)
2. [https://stackoverflow.com/questions/56281548/how-do-frameworks-like-spring-configure-the-servlet-container-without-web-xml](https://stackoverflow.com/questions/56281548/how-do-frameworks-like-spring-configure-the-servlet-container-without-web-xml)
3. [Udemy Spring & Hibernate for Beginners](https://www.udemy.com/course/spring-hibernate-tutorial/)
4. [Servlet 3.1 Specification](https://javaee.github.io/servlet-spec/downloads/servlet-3.1/Final/servlet-3_1-final.pdf)
