---
layout: post
title: Spring AOP 정리
feature-img: assets/img/titles/spring-logo.svg
thumbnail: assets/img/contents/sf-5.png
author: csupreme19
tags: [Spring, Java, AOP]

---

# Spring AOP 정리

![spring-logo.svg]({{ "/assets/img/titles/spring-logo.svg"}})

본 문서에서는 스프링 AOP에 대하여 정리해보았다.

---

## Spring AOP

![sf-5.png]({{ "/assets/img/contents/sf-5.png"}})

스프링에서 DI, IoC와 함께 주요 개념인 AOP(Aspect Oriented Programming): 관점 지향 프로그래밍 기능을 제공한다. pointcut과 메소드 인터셉터들을 지원한다.

- `spring-aop`
  - AOP의 구현체로 포인트컷, 인터셉터 등을 지원한다.
  - AOP를 이용하여 소스코드에서 AOP 공통 부분을 분리할 수 있다.
- `spring-aspects`
  - AspectJ를 지원하는 모듈이다.
- `spring-instruments`
  - Tomcat과 같은 미들웨어 서버에서의 클래스로더 구현체를 제공한다.

Spring Framework의 또 다른 핵심 기능인 AOP 관점지향 프로그래밍을 담당한다.

---

### AOP가 도대체 뭐야?

AOP는 Aspect Oriented Programming 관점 지향 프로그래밍이다.

![sf-6.png]({{ "/assets/img/contents/sf-6.png"}})



핵심 기능(비즈니스 로직)에서 부가 기능(로깅, 트랜잭션, 예외)을 분리하기 위하여 횡단 분리(cross-cutting)하는 것

Aspect로 분리하여 공통 기능을 비즈니스 로직과 코드에 영향을 주지 않게 적용할 수 있다.

---

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

해당 클래스가 2, 3개라면 큰 문제가 없겠지만 100개 아니 1000개라면 상상도 하기 싫다.

---

### OOP로 해결해보자, OOP의 한계

#### 1. 상속을 쓰면 되잖아

객체지향이 좋은게 뭐겠어? => 상속과 다형성이지

한번 해보자

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

<br>

#### 2. 위임(Delegation)을 써보자

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

---

### AOP를 사용한다면

기존의 OOP를 이용하여 문제를 해결할 수는 있으나 아름다운 코드라고는 할 수 없다.

이러한 문제를 해결하기 위하여 나온 개념이 관점 지향 프로그래밍 AOP이다.



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

---

### AOP 용어 정리

![sf-8.png]({{ "/assets/img/contents/sf-8.png"}})

#### Advice

어떤 액션을 취할지 정의한 공통 코드(실제 구현 코드)

Advice에는 시점에 따른 5가지 타입이 있다.

| 종류            | 시점                           |
| --------------- | ------------------------------ |
| Before          | 메소드 실행 전                 |
| After finally   | 메소드 실행 후(finally)        |
| After returning | 메소드 실행 후(success)        |
| After throwing  | 메소드 실행 후(exception)      |
| Around          | 메소드 실행 전, 메소드 실행 후 |

#### Join Point

Aspect가 해당 코드의 어느 시점에(어디에) 적용되는지 정의한 지점으로 포인트컷의 후보군이 된다.

#### Pointcut

조인포인트중 표현식을 통해 필터링한 특정 조인포인트로 적용 대상을 결정

#### Aspect

횡단 관심사로 어떤 포인트컷에 어떤 어드바이스를 취할지 정의된 코드 모듈(로그 출력, 캐싱, 예외처리 등)

Advice + Pointcut라고 생각할 수 있다.

<br>

#### Weaving

Aspect를 목표 타겟 객체에 연결하는 것

컴파일타임 위빙, 로드타임 위빙, 런타임 위빙이 있다.

> 컴파일타임 위빙은 추가적인 컴파일 단계가 필요하며 런타임 위빙이 가장 느리다.

---

### AOP 프레임워크

- Spring AOP
- AspectJ

Java의 대표적인 두 가지 AOP 프레임워크를 지원한다.

---

### Spring AOP

![spring-icon-logo.png]({{ "/assets/img/titles/spring-icon-logo.png"}})

<div class="mermaid">
  flowchart LR
    A[Application]
    B[AOP Proxy]
  	subgraph Aspect
    C[Logging Aspect]
    D[Security Aspect]
  	end
    E[Target Object]
    A-->B---C---D-->E
</div>




Spring 프레임워크에서 기본적으로 제공하는 AOP이며 일반 AOP 인터페이스를 기반으로한 새로운 인터페이스이자 일부 구현체이다.

스프링 프레임워크 뒷단에서는 보안, 트랜잭션, 캐싱등을 구현할때 Spring AOP를 사용한다고 한다.

AOP Proxy를 통하여 Aspect 위빙된 타겟 어플리케이션과 통신하는 방식을 가지고 있다.

> Spring AOP는 AOP 인터페이스의 일부만을 구현하고 있다. 
>
> 따라서 특정 기능을 사용하려면 결국 AspectJ 라이브러리가 필요하다.

<br>

#### 장점

1. AspectJ 보다 사용이 간단하다.
2. 프록시 패턴을 사용한다.
3. @Aspect 애노테이션을 통해 AspectJ로의 이관이 쉽게 가능하다.

#### 단점

1. 메소드 레벨의 조인포인트만 가능하다.
2. 스프링 컨테이너의 Bean에만 적용이 가능하다.
3. 런타임 위빙으로 약간의 성능 저하가 있을 수 있다.

---

### AspectJ

[https://www.eclipse.org/aspectj](https://www.eclipse.org/aspectj)

AOP 인터페이스의 확실한 구현체 프레임워크로 2001년에 발표된 오리지널 AOP 프레임워크이다.

AOP 기능의 모든 기능을 지원하며 생성자, 필드 joinpoint 및 컴파일 타임, 로드타임 위빙을 지원한다.

<br>

#### 장점

1. 모든 조인포인트를 지원한다.
2. 스프링 빈뿐만이 아니라 POJO도 지원한다.
3. Spring AOP에 비해 더 좋은 성능을 가진다.
4. AOP의 모든 기능을 제공한다.

#### 단점

1. 컴파일타임 위빙은 추가적인 컴파일 단계를 필요로함
2. 포인트컷 문법이 좀 더 복잡하다.

---

### Spring AOP vs AspectJ

| 비교군     | AspectJ                                 | Spring AOP               |
| ---------- | --------------------------------------- | ------------------------ |
| 조인포인트 | 메소드, 생성자, 필드                    | 메소드                   |
| 위빙       | 컴파일타임, 포스트 컴파일타임, 로드타임 | 런타임                   |
| 성능       | 비교적 빠름(컴파일타임 위빙)            | 비교적 느림(런타임 위빙) |
| 타겟       | Spring Bean, POJO                       | Spring Bean              |

AspectJ는 AOP의 완전한 구현체를 목표로 하고 있으므로 AOP 인터페이스의 모든 기능을 지원한다.

하지만 Spring AOP는 AOP의 상위 인터페이스이자 AOP의 일부 구현체로 모든 기능을 지원하지 않는다.

경우에 따라 AspectJ의 구현체를 요구하므로 실질적으로 사용시에는 AspectJ 라이브러리를 포함하게 될 것이다.

---

### Aspect 예제

#### AspectJ Auto Proxy 활성화하기

```java
@Configuration
@EnableAspectJAutoProxy
@ComponentScan("com.csupreme19.aopdemo")
public class AppConfig {
}
```

`@Aspect` 애너테이션을 처리하기 위해서는 AspectJ Auto Proxy를 활성화해야한다.

스프링 어플리케이션 컨텍스트에 `@EnableAspectJAutoProxy` 애너테이션을 추가한다.

#### Aspect

```java
@Aspect
@Component
public class LoggingAspect {
	@Before("execution(public void getUser())")
	public void before() {
		System.out.println("@Before Advice on method");
	}
}
```

#### Target

```java
@Component
public class UserDAO {
	public void getUser() {
		System.out.println(this.getClass() + "UserDAO.getUser()");
	}
}
```

#### 실행결과

```
@Before Advice on method
class com.csupreme19.aopdemo.dao.UserDAOUserDAO.getUser()
```

---

### Pointcut Designator(PCD)

포인트컷 지정자로 포인트컷에 해당하는 문자열은 다음과 같은 패턴을 가진다.

```java
@Pointcut("execution(접근 지정자? 리턴타입 패키지.클래스? 메소드명(파라미터명) throws Exception?)")
@Pointcut("within(패키지.클래스)")
// 이 외에도 @target, @annotation 등의 PCD들이 존재한다.
```

뒤에 ?가 붙은 것은 Optional 패턴으로 비어 있어도 상관 없음

> 접근지정자는 public만 사용 가능하다.

```java
// 예시
@Pointcut("execution(public void com.csupreme19.demo.myClass.methodA() throws RuntimeException)")
@Pointcut("within(com.csupreme19.demo.myClass)")

// 필수 패턴만 적을시
@Pointcut("execution(void methodA())")

// * asterisk로 문자열 대치 가능
@Pointcut("execution(* method*())")

// (): 파라미터 없음
// (*): 단일 파라미터 대치
// (..): 모든 파라미터 대치
@Pointcut("execution(* get*(..))")

// &&(and), ||(or), !(not) 3가지 논리 연산자 가능
@Pointcut("execution(* get*(..)) || execution(* set*(..))")

// Pointcut 선언적으로 사용 가능
@Pointcut("execution(* get*(..))")
private void getter() {}

@Pointcut("execution(* set*(..))")
private void setter() {}

@Before("getter() || setter()")
private void () {}
```

---

### JoinPoint

```java
@Before("execution(* getUser(..))")
public void before(JoinPoint joinPoint) {
  System.out.println("@Before Advice on method");

  MethodSignature methodSignature = (MethodSignature)joinPoint.getSignature();
  System.out.println("Method signature: " + methodSignature);

  Object[] args = joinPoint.getArgs();

  for(Object arg : args) {
    if(arg instanceof String){
      System.out.println((String)arg);
    }
  }
}
```

```java
UserDAO userDAO = ctx.getBean("userDAO", UserDAO.class);
User user = userDAO.getUser("1", "seunghoon.choi", "28");
System.out.println(user.getName());

// 출력
@Before Advice on method
Method signature: User com.csupreme19.aopdemo.dao.UserDAO.getUser(String,String,String)
1
seunghoon.choi
28
class com.csupreme19.aopdemo.dao.UserDAOUserDAO.getUser(id, name, age)
```

Advice가 실행되는 타겟 정보를 JoinPoint라는 인자로 받아서 확인이 가능하다.

또한 실제 실행되는 메서드 정보를 MethodSignature라는 클래스를 이용하여 확인할 수 있다.

---

### Aspect Order

```java
@Aspect
@Component
@Order(1)
public class LoggingAspect {
	@Before("execution(* get*())")
	public void before() {
		System.out.println("@Before Advice on get method");
	}
}
```

```java
@Aspect
@Component
@Order(2)
public class SecurityAspect {
	@Before("execution(* set*(..))")
	public void before() {
		System.out.println("@Before Advice on set method");
	}
}
```

`@Order(Integer)` 애너테이션으로 동일 타겟에 대하여 순서 지정이 가능하다.

낮은 수일 수록 먼저 실행된다.

---

### Advice Type

| 종류            | 시점                    |
| --------------- | ----------------------- |
| @Before         | 타겟 실행 이전          |
| @AfterReturning | 리턴 이후               |
| @AfterThrowing  | 예외 발생 이후          |
| @After          | 타겟 실행 이후(finally) |
| @Around         | 타겟 실행 이전, 이후    |

---

### @Before

<div class="mermaid">
sequenceDiagram
  participant A as App
  participant B as AOP Proxy
  participant C as @Before
  participant D as UserDAO
  A->>B: getUser(..)
  B->>C: getUser(..)
  C->>D: getUser(..)
  D-->>B: return data
  B-->>A: return data
</div>

타겟 메서드 실행 이전 시점에 실행된다.

```java
// getUser 메서드 실행 이전에 실행
@Before("execution(* getUser(..))")
public void before(JoinPoint joinPoint) {
  System.out.println("@Before Advice on method");
}

// 출력
@Before Advice on method
class com.csupreme19.aopdemo.dao.UserDAOUserDAO.getUser(id)
```

---

### @AfterReturning

<div class="mermaid">
sequenceDiagram
  participant A as App
  participant B as AOP Proxy
  participant C as UserDAO
  participant D as @AfterReturning
  A->>B: getUser(..)
  B->>C: getUser(..)
  C-->>D: return data
  D-->>B: return data
  B-->>A: return data
</div>

타겟 메서드가 값을 반환하면 실행된다.

해당 메서드의 타입과 인자로 받는 타입값이 일치해야만 실행된다.

```java
// getUser 메서드가 실행되어 User 클래스 타입이 리턴되었을때 실행
@AfterReturning(pointcut="execution(* getUser(..))", returning="result")
public void afterReturning(JoinPoint joinPoint, User result) {
  System.out.println("@AfterReturning Advice on method:: " + result);
}

// 반환 인자를 Object로 설정하면 모든 타입에 매핑된다.
@AfterReturning(pointcut="execution(* getUser(..))", returning="result")
public void afterReturning(JoinPoint joinPoint, Object result) {
  System.out.println("@AfterReturning Advice on method:: " + result);
}

// 출력
class com.csupreme19.aopdemo.dao.UserDAOUserDAO.getUser(id)
@AfterReturning Advice on method:: com.csupreme19.model.User@6d025197
```

<br>

#### Post processing

<div class="mermaid">
sequenceDiagram
  participant A as App
  participant B as AOP Proxy
  participant C as UserDAO
  participant D as @AfterReturning
  A->>B: getUser(..)
  B->>C: getUser(..)
  C-->>+D: return data
  D->>-D: post process
  D-->>B: return modified data
  B-->>A: return modified data
</div>

```java
UserDAO userDAO = ctx.getBean("userDAO", UserDAO.class);
User user = userDAO.getUser("1");
```

해당 Advice가 실행되는 시점은 getUser() 가 실행되어 user라는 변수에 할당되기 이전 시점이다.

따라서 메모리 할당 이전에 Aspect 레벨에서 데이터 수정 등의 처리가 가능하다.

> AOP는 기존 소스코드(로직)과 분리되어 작성되므로 해당 기능 사용시 데이터 변화에 주의할 것

```java
@AfterReturning(pointcut="execution(* getUser(..))", returning="result")
public void afterReturning(JoinPoint joinPoint, User result) {
  System.out.println("@AfterReturning Advice on method:: " + result.getName());
  result.setName("John Doe");
}
```

```java
User user = userDAO.getUser("1");
System.out.println(user.getName());

// 출력
class com.csupreme19.aopdemo.dao.UserDAOUserDAO.getUser(id)
@AfterReturning Advice on method:: com.csupreme19.model.User@6d025197
John Doe
```

---

### @AfterThrowing

<div class="mermaid">
sequenceDiagram
  participant A as App
  participant B as AOP Proxy
  participant C as UserDAO
  participant D as @AfterThrowing
  A->>B: getUser(..)
  B->>C: getUser(..)
  C-->>D: throw exception
  D-->>B: throw exception
  B-->>A: throw exception
</div>

```java
@AfterThrowing(pointcut="execution(* getUser(..))", throwing="ex")
public void afterThrowing(JoinPoint joinPoint, Exception ex) {
  System.out.println("@AfterThrowing Advice on method");
  System.out.println("Caught on afterThrowing advice: " + ex.getMessage());
}
```

```java
try{
  User user = userDAO.getUser("1134513");
} catch(Exception ex) {
  System.out.println("Caught on afterThrowing main program: " + ex.getMessage());
}

// 출력
class com.csupreme19.aopdemo.dao.UserDAOUserDAO.getUser(id, name, age)
@AfterThrowing Advice on method
Caught on afterThrowing advice: Not found
Caught on afterThrowing main program: Not found
```

예외 발생 시 throw된 예외를 처리할 수 있다.

Aspect에서 예외를 먼저 가져가고 AOP Proxy로 전달하여 메인 프로그램까지 도달하는 모습을 볼 수 있다.

---

### @After

<div class="mermaid">
sequenceDiagram
  participant A as App
  participant B as AOP Proxy
  participant C as UserDAO
  participant D as @After
  A->>B: getUser(..)
  B->>C: getUser(..)
  alt is success
  	C-->>D: return data
  	C-->>B: return data
    B-->>A: return data
  end
  alt is failure
  	C-->>D: throw exception
  	C-->>B: throw exception
    B-->>A: throw exception
  end

</div>

```java
@After("execution(* getUser(..))")
public void after(JoinPoint joinPoint) {
  System.out.println("@After Advice on method");
}
```

```java
User user = userDAO.getUser("1", "seunghoon.choi", "28");
try{
  User user2 = userDAO.getUser("1134513");
} catch(Exception ex) {
  System.out.println("Caught on afterThrowing main program: " + ex.getMessage());
}

// 출력
class com.csupreme19.aopdemo.dao.UserDAOUserDAO.getUser(id, name, age)
@After Advice on method
class com.csupreme19.aopdemo.dao.UserDAOUserDAO.getUser(id, name, age)
@After Advice on method
Caught on afterThrowing main program: Not found
```

해당 메서드가 실행 된 이후에 실행된다.

해당 메서드 성공 여부와 상관없이 예외나 에러가 발생해도 해당 Advice는 실행된다. (finally와 비슷한 원리)

---

### @Around

<div class="mermaid">
sequenceDiagram
  participant A as App
  participant B as AOP Proxy
  participant C as @Around
  participant D as UserDAO
  A->>B: getUser(..)
  B->>C: getUser(..)
  C->>D: getUser(..)
  D-->>C: result
  C-->>B: result
  B-->>A: result
</div>

```java
@Around("execution(* getUser(..))")
public Object before(ProceedingJoinPoint proceedingJoinPoint) throws Throwable {
  System.out.println("@Around Advice on method begin");
  long begin = System.currentTimeMillis();
  
  // 메소드 실행 정보 가져오기
  Signature methodSignature = proceedingJoinPoint.getSignature();
  System.out.println(methodSignature.toShortString());
  
  Object result = null;
  try {
    // 메소드 실행 제어
    result = proceedingJoinPoint.proceed();
  } catch (Exception ex) {
    // 예외 처리
    System.out.println(ex.getMessage());
  }
  
  // 후처리
  if(result instanceof User) {
    ((User) result).setName("John Doe");
  }
  
  long end = System.currentTimeMillis();
  System.out.println("getUser execution time: " + String.valueOf(end-begin) + "ms");
  System.out.println("@Around Advice on method end");
  return result;
}
```

```java
User user = userDAO.getUser("1", "seunghoon.choi", "28");
System.out.println(user.getName());

// 출력(성공시)
@Around Advice on method begin
UserDAO.getUser(..)
class com.csupreme19.aopdemo.dao.UserDAOUserDAO.getUser(id, name, age)
getUser execution time: 10ms
@Around Advice on method end
seunghoon.choi

// 출력(예외처리시)
@Around Advice on method begin
UserDAO.getUser(..)
class com.csupreme19.aopdemo.dao.UserDAOUserDAO.getUser(id, name, age)
Not found
getUser execution time: 1ms
@Around Advice on method end
```

메서드 실행 전 후로 Advice를 실행한다.

위 4가지 타입의 기능을 모두 수행할 수 있다. (로깅, 보안처리, 예외처리, 실행시간 측정 등)

@Around 어드바이스에서는 ProceedingJoinPoint라는 클래스를 인자로 받아서 타겟 메서드의 실행을 제어할 수 있다.

메서드 실행 전후로 Advice가 실행되어 일반적으로 가장 자주 사용되는 Advice이다.

위 예시처럼 로깅, 예외처리, 후처리 등이 모두 가능하기 때문이다.

---

### AOP Best Practices

AOP는 공통 관심사로 공통 로직을 담당하므로 아래와 같이 작성하는 것이 좋다.

- 코드는 최대한 간결하게
- 코드는 최대한 실행속도가 빠르게
- 무겁고 느린 연산 피하기

---

## Reference

1. [docs.spring.io overview](https://docs.spring.io/spring-framework/docs/current/reference/html/overview.html#overview)
2. [docs.spring.io 5.0.0 overview](https://docs.spring.io/spring-framework/docs/5.0.0.M5/spring-framework-reference/html/overview.html)
4. [Udemy Spring & Hibernate for Beginners](https://www.udemy.com/course/spring-hibernate-tutorial/)
5. [스프링 철저 입문](http://www.yes24.com/Product/Goods/59192207)

