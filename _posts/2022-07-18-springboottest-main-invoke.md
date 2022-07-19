---
layout: post
title: Spring Boot Test는 실제 Application main을 호출하지 않는다
feature-img: assets/img/titles/spring-logo.svg
thumbnail: assets/img/contents/smi-1.png
author: csupreme19
categories: Development Spring
tags: [Spring, Test, Junit, Mockito]

---

# Spring Boot Test는 실제 Application main을 호출하지 않는다

![spring-logo.svg]({{ "/assets/img/titles/spring-logo.svg"}})

---

### 상황

```java
@SpringBootApplication
public class WebApplication {
	private static AppConfig config;
	public static AppConfig getConfig() {
		return config;
	}

	public static void main(String[] args) {
		ConfigurableApplicationContext ctx = SpringApplication.run(WebApplication.class, args);
		config = ctx.getBean(AppConfig.class);
		config.setCtx(ctx);
	}
```

현재 개발중인 서버는 위와 같이 AppConfig라는 Config 클래스에 추가 정보를 설정하는 로직이 작성되어 있다.

```java
@Test
void getConfig_NotNull() {
  log.info("WebApplication.WebConfig={}", WebApplication.getConfig());
  assertThat(WebApplication.getConfig()).isNotNull();
}
```

```java
2022-07-18_13:15:32.787 INFO  [WebApplicationTest] WebApplication.WebConfig=null

java.lang.AssertionError: 
Expecting actual not to be null
```

단위 테스트 작성시 WebApplication의 Config가 null인 상황

`WebApplciation.getConfig()`의 형태로 static 접근하여 사용중인 코드가 많아 해당 부분 의존도가 높아 해당 코드를 당장 개선하기는 어려운 상황



### 원인 파악

`@SpringBootTest` 어노테이션으로 실행되는 Spring Context는 실제 애플리케이션의 main을 호출하지 않는다.

#### @SpringBootTest

```java
2022-07-18_13:15:20.869 INFO  [WebApplicationTest] Starting WebApplicationTest using Java ...
```

#### @SpringBootApplication

```java
2022-07-18_13:18:15.873 INFO [WebApplication] Starting WebApplication using Java ...
2022-07-18_13:18:15.873 INFO [WebApplication] 
 ConfigurableApplicationContext=org.springframework.boot.web.servlet.context.AnnotationConfigServletWebServerApplicationContext@75158918,
```

위의 경우 실제 스프링 부트 애플리케이션이 구동될때 호출되는 main 메서드에 로직이 작성되어 있으므로 빈 등록 여부와 상관 없이 `WebApplication.getConfig()`와 같이 클래스 지정 접근이 불가능한 문제

추가로 Spring boot의 기본 환경은 `@SpringBootTest(webEnvironment = MOCK)`으로 설정되어 있으며 실제 Spring Boot Embedded Tomcat을 실행하지 않음



### 해결

앞서 말했듯 빈을 주입받아 사용하는 것이 아닌 코드 여러 곳에서  `WebApplciation.getConfig()`의 형태로 static 접근하여 사용중으로 당장 코드를 개선하기는 어려운 상황

임시 해결책으로 static 메서드를 mocking할 수 있는 Mockito 라이브러리의 MockedStatic 사용

```java
...
import static org.assertj.core.api.Assertions.assertThat;
import static org.mockito.Mockito.mockStatic;

@Slf4j
@SpringBootTest(classes = {WebConfig.class})
public class WebApplicationTest {

    MockedStatic<WebApplication> application;

    @Autowired
    private WebConfig webConfig;

    @BeforeEach
    void setUp() {
        application = mockStatic(WebApplication.class);
        application.when(WebApplication::getConfig).thenReturn(webConfig);
    }

    @Test
    void getConfig_NotNull() {
        log.info("WebApplication.WebConfig={}", WebApplication.getConfig());
        assertThat(WebApplication.getConfig()).isNotNull();
    }
}
```

`WebConfig`를 스프링 `@Configuration` 어노테이션을 통하여 스프링 빈으로 등록하여 주입받았다.

주입받은 webConfig를 MockedStatic을 이용하여 리턴하도록 설정 후 테스트 진행

![smi-1.png]({{ "/assets/img/contents/smi-1.png"}})

테스트 정상 통과 확인

Spring Context에 등록된 Bean을 Mockito를 이용하여 사용하는 모양이 이상하긴 하지만 

실제 코드를 건드리지 않고 테스트 코드를 작성할 수 있었다.

##### 참고

`@SpringBootTest(classes = {WebConfig.class})` classes에 테스트시 등록할 Spring Bean을 명시해준다.

```java
2022-07-18_13:37:46.993 INFO  [WebApplicationTest] Started WebApplicationTest in 11.488 seconds (JVM running for 12.397)
```

```java
2022-07-18_13:39:17.244 INFO  [WebApplicationTest] Started WebApplicationTest in 0.889 seconds (JVM running for 1.742)
```

필요한 모든 컴포넌트들을 스캔하여 등록하는 것이 아닌 명시해준 클래스만 등록하므로 실행속도가 매우 빠른 것을 볼 수 있다.



### 결론

장기적으로 보았을 때 스프링 빈이 아닌 Application 클래스에 static 수동 등록하여 사용하는 방법은 좋지 않다고 생각한다.

필요한 클래스에 의존성을 주입하여 사용하는 방법으로 순차적 리팩토링이 필요할 것 같다.

혹시 기존 소스를 건드리지 않는 더 좋은 해결방법이 있으면 알려주시면 감사하겠습니다.

