---
layout: post
title: Clean Code 정리
feature-img: assets/img/titles/clean-code.png
thumbnail: assets/img/titles/clean-code.png
author: csupreme19
categories: Development
tags: [Book, Clean Code]

---

# Clean Code 정리

![clean-code.png]({{ "/assets/img/titles/clean-code.png"}})

[Clean Code(클린 코드)](http://www.kyobobook.co.kr/product/detailViewKor.laf?mallGb=KOR&ejkGb=KOR&barcode=9788966260959)

로버트 C.마틴 저 **클린 코드: 애자일 소프트웨어 장인 정신**을 읽고 정리해봤어요.

---

## 1. 깨끗한 코드

### 코드(Code)

코드는 고객의 **요구사항**을 상세히 **표현**하는 **언어**

<br>

### 깨끗한 코드란?

- 우아하고 **보기에 즐거운** 코드 - [바야네 스트롭스트룹](https://ko.wikipedia.org/wiki/%EB%B9%84%EC%95%BC%EB%84%A4_%EC%8A%A4%ED%8A%B8%EB%A1%AD%EC%8A%A4%ED%8A%B8%EB%A3%B9)
- 단순하고 직접적이며 명쾌한 **잘 읽히는** 코드 - [그래디 부치](https://ko.wikipedia.org/wiki/%EA%B7%B8%EB%9E%98%EB%94%94_%EB%B6%80%EC%B9%98)
- **가독성이 좋으며** 다른 사람이 고치기 쉬운 코드 - ["빅" 데이브 토마스](https://en.wikipedia.org/wiki/David_A._Thomas_(software_developer))
- 세세하며 꼼꼼하며 **단정되게 정리**한 코드 - 마이클 페더스
- **중복이 없는** 간단하게 추상화된 코드 - [론 제프리스](https://en.wikipedia.org/wiki/Ron_Jeffries)

깨끗한 코드라는 것은 그 정의가 모호한 만큼 논쟁의 여지가 있으나 전문가들이 입을 모아 말하는 공통점은 **가독성**이에요.

<br>

### 가독성

**가독성**이 높은 코드는 다른 사람이 고치기 쉬우며 이는 유지보수성이 좋다는 것을 의미해요.

결국 가독성이 높은 코드를 작성하려면 중복 줄이기, 간단한 추상화 고려하기, 철저한 오류 처리, 단위 테스트 등이 동반되어야 하므로 궁극적인 목표라고 볼 수 있겠죠.

![cc-7.jpeg]({{ "/assets/img/contents/cc-7.jpeg"}})

> 코드 작성 시간 < 코드 읽기 시간

실제로 코드를 작성하는 시간보다 코드를 읽고 이해하는 시간이 훨씬 많은 걸 우리는 잘 알고 있어요.

<br>

### 클린 코드에서 말하고자 하는 것

이 책에서 말하는 깨끗한 코드를 작성하는 방법 또한 항상 올바르다는 단정을 할 수는 없지만

이 책에서 말하는 여러 기법들은 오랫동안 심사숙고하며 축적된 교훈과 기법이므로 충분히 깨끗한 코드를 작성할 수 있을 것이라고 생각해요.

![cc-1.png]({{ "/assets/img/contents/cc-1.png"}})

> 코드 품질을 측정하는 유일한 척도 = WTFs/m
>
> [https://www.osnews.com/story/19266/wtfsm/](https://www.osnews.com/story/19266/wtfsm/)

<br>

### 보이스카우트 규칙

![cc-6.png]({{ "/assets/img/contents/cc-6.png"}})

**처음 왔을 때보다 더 깨끗하게 해놓고 떠나라**

시간이 지날수록 코드가 더 좋아지는 프로젝트를 만들 수 있다는 의미예요.

코드의 퇴보를 막고 지속적인 개선을 통하여 코드 품질을 유지할 수 있어요.

---

## 2. 의미 있는 이름

소프트웨어에서의 이름은 어디에서나 쓰여요.

>  함수, 변수, 인수, 클래스, 패키지, 디렉토리, 빌드 파일

이름을 잘 짓는 것 만으로도 가독성이 높은 코드를 작성하는데 큰 도움이 되어요.

<br>

### 의도를 분명히

변수명이 모호하면 코드 맥락을 이해하기 힘들죠.

```java
int d;
int daysSinceCreation;
```

d라는 이름을 보고 어떤 변수인지 유추하기 힘들어요.

daytsSinceCreation은 생성 후 몇일이 지났는지 나타낸다고 알 수 있어요.

```java
public static void copyChars(char a1[], char a2[]) {
  for (int i=0; i<a1.length; i++) {
    a2[i] = a1[i];
  }
}
```

a1[], a2[] 대신 source, destination을 사용한다면 바로 이해할 수 있겠네요.

<br>

### 잘못된 정보 피하기

#### 1. 널리 쓰이는 단어

```java
int hp, aix, sco;
```

위의 단어들은 유닉스 플랫폼을 가리키는 이름이므로 혼란을 야기할 수 있어요.

#### 2. 잘못된 타입 변수명

```java
Map<String, String> userList;
```

이름에 자료형을 넣는다면 자료형을 착각할 수 있어요.

#### 3. 비슷한 이름

```java
public class XYZControllerStringHandler;
public class XYZControllerHandlerString;
```

서로 **다른 역할**을 하는 두 클래스가 비슷한 이름을 가지고 있다면 어떤 것이 핸들러인지 알 수 있을까요?

#### 4. l과 O와 같은 혼동될 수 있는 변수명

```java
int a = 1;
if ( O == 1 )
  a = O1;
else
  l = O1;
```

o, O, 0, l, 1, I와 같은 비슷한 글자는 가독성을 해치기 때문에 되도록 사용하지 마세요.

Consolas와 같은 개발 폰트를 사용하면 구분이 쉽기는 하지만 사용하지 말 것을 권고드려요.

<br>

### 의미 없는 추가적인 이름

```java
public class Product {}

public class ProductInfo {}
public class ProductData {}
public class ProductClass {}
public class ProductObject {}
public class TheProduct {}
public class AProduct {}
```

의미없는 **불용어(Noise Word)**를 추가한 명명법에 불과한 것을 눈치 채셨나요?

모두 Product라는 이름으로 대체할 수 있겠네요.

<br>

### 중복된 이름

```java
int money;
int moneyAmount;

String name;
String nameString;

String tel;
String telVariable;
```

변수명과 중복된 값이 들어가거나 의미없는 불용어가 추가되어 있네요.

<br>

### 발음이 쉬운 이름 사용

```java
class DtaRcrd102 {
  private Date genymdhms;
  private Date modymdhms;
}

class Customer {
  private Date generationTimestamp;
  private Date modificationTimestamp;
}
```

"젠 와이 엠 디 에이치 엠 에스"와 같은 발음은 협업시 소통을 저해하고 이해하기 힘들어 추가적인 설명을 요구하겠죠?

아래와 같이 명명하면 누구나 쉽게 생성일자라는 것을 알 수 있어요.

<br>

### 검색하기 쉬운 이름 사용

```java
const int TASK_DAYS = 230;
const int WORK_DAYS_PER_WEEK = 5;

int taskWeeks = ( TASK_DAYS / WORK_DAYS_PER_WEEK);

int taskWeeks = (230 / 5);
```

만약 `WORK_DAYS_PER_WEEK` 대신 5로 검색한다면 5가 포함된 수많은 수식과 코드들이 나와요.

<br>

### 헝가리식 표기법(Hungarian Notation)

> vUsing adjHungarian nnotation vmakes nreading ncode adjdifficult.
>
> [https://stackoverflow.com/a/112080](https://stackoverflow.com/a/112080)

```java
// 변수명 앞에 타입에 해당하는 문자를 붙임
boolean bCheck;
char chInitial;
int iSize;
// 멤버변수 사용시엔 m_ 접두어
int m_length;
String m_desc;
```

헝가리식 표기법은 예전 C 컴파일러가 타입을 점검하지 않거나 IDE의 기능이 부실했을 때나 사용하던 것이에요.

자료형을 쉽게 알 수 있도록 변수명에 타입을 적어주는 방식인데

요즘에는 컴파일러가 타입 체크를 할 뿐더러 IDE가 코드 작성 단계에서 타입체크를 하므로 사용하지 마세요.

<br>

### 인터페이스 클래스와 구현 클래스

```java
// 예전
public Interface IShapeFactory {}
public Class ShapeFactory {}

// 현재
public Interface ShapeFactory {}
public Class ShapeFactoryImpl {}
```

인터페이스 이름에 접두어 I는 붙이지 않는 편이 좋아요.

호출시에 추상화된 인터페이스를 호출하게 되는데 

굳이 인터페이스임을 알릴 필요도 없으며 I라는 접두어로 추가적인 정보를 줄 필요도 없어요.

위와 같이 명명하면 사용자는 인터페이스를 신경 쓸 필요 없이 ShapeFactory라고만 인식하여 호출하기 때문에 더 좋은 명명법이에요.

<br>

### 문자 하나

```java
int a;
int b;
int c;
```

문자 하나만 사용하는 경우는 절대 피하세요.

>  관용적으로 반복문의 i, j, k...는 이터레이션에 사용하였으므로 괜찮다네요.

<br>

### 클래스명

```java
// 명사
public Class Customer {}

// 동사 금지
public Class Proceed {}

// 넓은 범위의 명사 금지
public Class Data {}
public Class Info {}
```

클래스명은 명사, 명사구로 이름을 짓기를 권장해요.

<br>

### 메서드명

```java
// 동사
public int save() {}

// 명사 금지
public int success() {}
public void user() {}

// 자바의 경우
public void setName() {}
public String getName() {}
```

메서드명은 동사, 동사구로 이름을 짓기를 권장해요.

>  자바의 경우 접근자, 변경자, 조건자는 자바빈 명세에따라 get, set, is 접두어 사용

<br>

### 기발한 이름 피하기

```java
// 기발한 이름
public int detonate() {}

// 평범한 이름
public int delete() {}
```

기발한 이름은 본인이 생각하기엔 멋있을지 몰라도 보는 사람은 짜증날 수 있어요.

<br>

### 한 개념에 한 단어 사용

```java
// 모두 동일
public int fetchName() {}
public int retrieveName() {}
public int getName() {}

// 마찬가지
public Class DeviceController {}
public Class DeviceManager {}
public Class DeviceDriver {}
```

위와 같이 같은 개념에 대하여 여러 단어를 혼동해서 사용한다면 가독성을 저해해요.

<br>

### 한 단어를 두가지 목적으로 사용 금지

```java
public int addNumber(int num1, int num2){}
public int addUser(List<User> users, User user){}

public int addNumber(int num1, int num2){}
public int insertUser(List<User> users, User user){}
```

위의 한 개념에 한 단어 사용과는 반대의 경우로

addNumber는 수를 더하는 것이고 addUser는 리스트에 User를 추가하는 것이잖아요?

따라서 add가 아닌 insert, append와 같은 단어가 적절해요.

같은 행위가 아닌데도 같은 단어를 사용하는 것 또한 맥락을 해쳐요.

<br>

### 불필요한 맥락을 없애라

```java
public Class AAUser {}
public Class AAManagement {}
public Class AAAuthentication {}
```

AA(Admin App)이라는 애플리케이션을 작성한다고 가정했을때 위와 같이 AA를 모두 붙이게 된다면 A를 입력하고 자동완성할 때 모든 클래스가 열거되겠죠?

앞의 AA는 불필요하므로 제거하세요.

---

## 3. 함수

### 작게 만들어라

- 규칙 1: **작게!**

- 규칙 2: **더 작게!**

함수는 100줄을 넘어서는 안돼요.

아니 20줄도 길어요.

중첩 구조가 생길만큼 함수가 길면 안돼요.

다시 말해, if/else, while문 등에 들어가는 블록은 한줄이어야 해요.

![cc-9.png]({{ "/assets/img/contents/cc-9.png"}})

> 우리가 피해야 할 것

<br>

### 한가지만 해라!

- 규칙 1: 함수는 **한 가지**를 해야 한다.
- 규칙 2: 그 **한 가지**를 잘해야 한다.
- 규칙 3: 그  **한 가지**만을 해야 한다.

<br>

### 내려가기 규칙

코드는 위에서 아래로 이야기처럼 내려가면서 읽혀야 좋은 코드예요.

각 함수에서는 한 가지만의 레벨을 구현하며 다음 함수를 호출하도록 작성하세요.

<br>

### switch문 사용 자제

```java
public class UserFactoryImpl implements UserFactory {
  public User makeUser(UserRecord r) throws InvaliedUserType {
    switch (r.type) {
      case ADMIN:
        return new AdminUser(r);
      case MAINTAINER:
        return new MaintainerUser(r);
      case DEVELOPER:
        return new DeveloperUser(r);
      default:
        throw new InvalidUserType(r.type);
    }
  }
}
```

switch문은 피하되 불가피한 경우 다형성 객체로 한번만 사용하세요.

위 코드에서는 추상 팩토리에 switch문을 숨겨서 사용하는 코드예요.

<br>

### 서술적인 이름 사용

```java
public static String testableHtml() {}

private void includeSetupAndTeardownPages() {}
```

짧고 어려운 이름보다 길고 서술적인 이름이 좋아요.

해당 함수를 보고 어떤 역할을 할 수 있는지 알 수 있는 것이 좋아요.

<br>

### 함수 인수는 최대한 적게

```java
public void render(pageData);
public void render();
```

함수 인수는 0개가 가장 이상적이며 필요시 1개, 2개를 사용하세요.

3개 이상은 가능한 피하며 4개는 특별한 사유가 있어야만 사용하세요.

render(pageData)보다 render()가 훨씬 간결하고 이해하기 쉽겠죠?

#### 인수가 1개인 경우

- 질문 함수

```java
boolean isExists(Object obj) {}
```

- 변환 함수

```java
String getUpperCase(String str) {}
```

- 이벤트 함수

```java
void passwordAttemptFailedNtimes(int attemps) {}
```

#### 인수가 2개인 경우

- 좌표계 함수

```java
Point p = new Point(0, 0);
```

#### 인수가 3개인 경우

```java
assertEquals(message, expected, actual);
```

인수가 여러개인 경우 인수의 순서를 헷갈릴 가능성이 많아요.

#### 플래그 인수

```java
render(true)
```

플래그 인수를 사용한다는 것은 해당 함수가 **한 가지 이상의 일**을 한다는 것을 의미해요.

참이면 A 로직, 거짓이면 B로직 = 2개의 로직

따라서 인수로 플래그를 받는다면 **함수를 분리**하세요.

#### 인수 객체

```java
Circle makeCircle(double x, double y, double radius);
Circle makeCircle(Point center, double radius);
```

인수가 여러개가 존재한다면 인수를 객체로 넘기는 것을 고려하세요.

#### 가변 인수

```java
public String format(String format, Object... args) {}
```

인수가 가변적일때 사용하세요.

#### 출력 인수

```java
public void appendFooter(StringBuffer report) {}

// 출력 인수인 경우
appendFooter(report);

// this 사용하는 경우
report.appendFooter();
```

직관적이지 않아 상태 추적이 어려울 수 있어요.

인수는 항상 입력이어야 해요.

위와 같은 출력인수 대신 객체지향에서는 this라는 키워드를 사용하세요.

함수에서 상태를 변경해야할때 인수의 상태를 변경하지 말고 함수가 속한 객체의 상태를 변경하세요.

<br>

### 함수명

```java
// 동사 + 명사 쌍
setName(name)

// 동사 + 명사 + 인수(순서)
assertExpectedEqualsActual(expected, actual)
```

함수명을 작성할때 동사 + 명사 쌍으로 작성하되 인수가 여러개인 경우 함수명에 키워드로 순서를 명시하세요.

<br>

### 명령과 조회를 분리

```java
public boolean set(String key, String value) {}

// 설정하는 함수인가 조회하는 함수인가 혼동됨
if(set("username","johndoe"))
```

위에서 얘기한 함수는 한가지만을 해야한다는 정의에도 부합하는데

위 함수는 key인 키값을 찾아 value로 설정하고 성공하면 true, 실패하면 false를 리턴한다고 해보면

설정과 조회가 한꺼번에 되고 있어 호출시 코드가 지저분해지겠죠?

```java
if(keyExists("username")) {
  set("username","johndoe");
  ...
}
```

위와 같이 설정과 조회를 분리한다면 훨씬 보기좋은 코드를 작성할 수 있어요.

<br>

### 오류 코드보다 예외 사용

```java
// 오류 코드 사용
if(deleteUser(user) == CODE_OK) {
  logger.log("user deleted");
} else {
  logger.log("delete failed");
}

// 예외 처리 사용
try{
  deleteUser(user);
} catch (Exception e){
  logger.log(e.getMessage());
}
```

예외 처리를 사용하면 오류 처리 코드를 원래 코드에서 분리할 수 있어요.

---

## 4. 주석

### 주석은 거짓말을 한다

![cc-8.png]({{ "/assets/img/contents/cc-8.png"}})

> 오직 신만이 알고 있다

코드는 변화하고 진화하지만 그에 맞게 주석을 유지하고 보수하기란 현실적으로 불가능해요.

주석이 코드에서 분리되어 거짓말을 하는 경우가 많아서

항상 주석은 코드를 따라가지 않는다는 점을 명시하세요.

<br>

### 주석은 필요하지 않다

부정확한 주석은 없는 주석보다 잘못된 정보를 제공하므로 훨씬 나빠요.

그래서 애초에 주석이 필요 없는 방향으로 가는 것이 필요해요.

<br>

### 코드로 의도를 표현하라

```java
// 코드의 의도를 모른다
if((user.flags & HOURLY_FLAG) && (employee.age > 55))

// 코드에 의도가 담겨있다
if(employee.isEligibleForFullBenefits())
```

주석을 달기보다 코드 자체에 의도를 담으세요.

<br>

### 좋은 주석 예시

좋은 주석이란 주석을 달지 않는 것이지만

주석이 필요할때는 아래와 같은 예시가 좋은 예시예요.

#### TODO 주석

```java
//TODO 현재 필요하지 않다.
protected VersionInfo makerVersion() throws Exception {
  return null;
}
```

앞으로 할 일을 나타내는 TODO 주석은 필요하지만 현재 구현이 어려운 업무를 기술해요.

대부분의 IDE에서 TODO 주석을 찾아 보여주는 기능을 제공해요.

그렇다고 TODO를 남발하면 안되며 주기적으로 TODO 주석을 점검해 없애야해요.



#### 공개 API의 Javadocs

![cc-3.png]({{ "/assets/img/contents/cc-3.png"}})

표준 Java 라이브러리의 Javadocs가 훌륭한 예시라고 할 수 있겠네요.

잘 작성된 Javadocs는 유용하고 생산성을 높여줘요.

하지만 Javadocs 마찬가지로 주석과 같이 코드를 따라가지 못해 거짓말을 할 가능성이 존재해요.

<br>

### 나쁜 주석 예시

#### 주절거리는 주석

```java
public void loadProperteis() {
  try {
    String path = location+"/"+FILE;
    FileInputStream stream = new FileInputStream(path);
    loadedProperties.load(stream);
  }
  catch(IOException e){
    // 속성 파일이 없으면 기본값을 모두 메모리로 읽어들였다는 의미
  }
}
```

해당 주석만 보고 기본값이 무엇을 의미하는지 알 수 없어요.

주석을 보고 다른 코드를 뒤져봐야한다면 절대로 좋은 주석이 아니에요.



#### 코드와 중복된 주석

```java
// this.closed가 true일때 반환되는 유틸
// 타임아웃에 도달하면 예외 던짐
public void waitForClose(long timeoutMillis) throws Exception {
  if(!closed) {
    wait(timeoutMillis);
  if(!closed) {
    throw new Exception("could not be closed");      
  }
}
```

```java
i++;	// i 증가
```

코드만 보고도 이해할 수 있는 내용을 중복해서 적어놓았네요.

`ContainerBase.java`

```java
public abstract class ContainerBase extends LifecycleMBeanBase implements Container {
    // ----------------------------------------------------- Instance Variables
  
    /**
     * The child Containers belonging to this Container, keyed by name.
     */
    protected final HashMap<String, Container> children = new HashMap<>();

    /**
     * The processor delay for this component.
     */
    protected int backgroundProcessorDelay = -1;

    /**
     * The future allowing control of the background processor.
     */
    protected ScheduledFuture<?> backgroundProcessorFuture;
    protected ScheduledFuture<?> monitorFuture;

    /**
     * The container event listeners for this Container. Implemented as a
     * CopyOnWriteArrayList since listeners may invoke methods to add/remove
     * themselves or other listeners and with a ReadWriteLock that would trigger
     * a deadlock.
     */
    protected final List<ContainerListener> listeners = new CopyOnWriteArrayList<>();
  ...

```

> 톰캣의 `ContainerBase.java`에서 발췌한 코드

Javadocs와 중복되며 기록 외에는 쓸데없는 주석이 많아요.



#### 당연하거나 의무적인 주석

```java
/**
*
* @param name 사용자 이름
* @param email 사용자 이메일
* @param age 사용자 나이
*/
public void addUser(String name, String email, int age) {
  User user = new User();
  user.name = name;
  user.email = email;
  user.age = age;
  userList.add(user);
}
```

주석 없이 코드만 보고 쉽게 이해할 수 있어요.



#### 이력 기록용 주석

```java
/**
* 변경 이력 (21-10-2019부터)
* -----------------------
* 21-10-2019 : ~ 변경
* 27-10-2019 : ~ 추가
* 21-11-2019 : ~ 버그 수정
* 21-12-2019 : ~ Serializable 구현
*/
```

예전에 git과 같은 소스코드 관리 시스템이 없었을 때나 사용하던 것이에요.



#### 저자 표시 주석

```java
/* 20210113 최승훈 추가 */
```

버전관리 시스템에 모든 정보가 나와있으므로 불필요한 주석이에요.



#### 주석처리된 코드

```java
InputStreamResponse resposne = new InputStreamResponse();
// InputStream resultsStream = formatter.getResultStream();
```

주석처리된 코드는 다른 사람이 지우기를 주저하게 만들어요.

원저자가 이유가 있어서 남겨두었겠지 또는 중요하다는 생각에 지우면 안된다고 생각하지만

이런 쓸모없는 코드가 계속 추가되다보면 코드가 더러워져요.

1960년대에는 이런 방식이 유용했으나 현재는 버전 관리 시스템이 코드를 기억하므로 가차없이 지우세요.



`commons.java`

```java
this.bytePos = writeBytes(pngIdBytes, 0);
//hdrPos = bytePos;
writeHeader();
writeResolution();
//dataPos = bytesPos;
if(writeImageData()) {
	writeEnd();
  this.pngBytes = resizeByteARray(this.pngBytes, this.maxPos);
}
else {
  this.pngBytes = null;
}
return this.pngBytes;
```

>  아파치 commons에서 발췌한 코드

누가 어떤 의미로 주석으로 처리하여 남겨두었는지 알 수 없어요.



#### HTML 주석

```java
/**
* <p>
* $lt;execute-tests
*		suitepage=&quot;SuiteAcceptanceTests&quot; /&gt;
* </p>
*/
```

주석은 사람이 읽으라고 만든 것이므로

주석에 HTML 태그를 삽입하는 건 개발자의 몫이 아니라 개발툴의 몫이에요.



#### 너무 많은 정보

```java
/*
RFC 2045 - Multipurpose Internet MAil Extensions (MIME)
6.8.  Base64 Content-Transfer-Encoding
The Base64 Content-Transfer-Encoding is designed to represent
arbitrary sequences of octets in a form that need not be humanly
readable.  The encoding and decoding algorithms are simple, but the
encoded data are consistently only about 33 percent larger than the
unencoded data.
*/
```

필요한 정보만 담으세요.

---

## 5. 형식 맞추기

### 코드 형식

코드 형식은 의사소통의 일환으로 개발자의 일차적 의무예요.

잘 돌아가는 코드도 중요하지만 해당 구현 기능은 다음 버전에서 바뀔 확률이 있지만

오늘 구현한 코드의 가독성은 앞으로 품질에 매우 큰 영향을 미치기 때문이에요.

팀 합의하에 간단한 규칙을 정하고 모두가 따라야해요.

![cc-10.png]({{ "/assets/img/contents/cc-10.png"}})

> 팀 협의하에 규칙을 정하세요.

<br>

### 적절한 코드 길이

#### 자바 프로젝트 평균 줄수

| 프로젝트 | 평균      |
| -------- | --------- |
| junit    | 65줄      |
| testNG   | 70줄      |
| tam      | 70줄      |
| ant      | 200~500줄 |
| Tomcat   | 200~500줄 |

대다수 자바 프로젝트의 평균 파일 크기는 약 65줄로 500줄을 넘지않고 

대부분 200줄정도로도 대규모의 시스템을 구현할 수 있음을 보여주고 있어요.

<br>

### 세로 밀집

```java
public class ReporterConfig {
  private String name;
  
  // 세로 밀집
  private List<Property> properties = new ArrayList<Property>();
  public void add Property(Property property){
    m_properties.add(property);
  }
}
```

연관된 개념 & 기능은 세로로 가까이에 위치하도록 작성하세요.

<br>

### 수직 거리

#### 변수 선언

```java
// 최대한 가까운 시점에 선언
private static void readPreferences() {
  InputStream is = null;
  try {
    is=newFileInputStream(getPreferencesFile());
  } catch (Exception e){
    ...
  }
  ...
}

// 루프내 사용하는 변수는 루프문 내부에 선언
for(int i=0; i<userList.length(); i++){
  User user = userList.get(i);
}

for(User user : userList){
  ...
}
```

변수 선언 시점은 최대한 가까운 사용 시점으로 하세요.

#### 인스턴스 변수 선언

```java
public class User implements Person {
  private String name;
  private String email;
  ...
    
  private String doSomething(){
    ...
  }
  ...
    
  private int age;
  
  private String doSomethingCool(){
    ...
  }
}
```

인스턴스 변수의 경우 보통 클래스의 맨 앞쪽에 선언하세요.

코드를 읽는 사람이 중간에 인스턴스 변수 선언을 발견한다면 매우 혼란스럽겠죠?

#### 종속 함수

```java
public void loadPage(){
  String pageName = getPageName();
  ...
}

public String getPageName(){}
```

어떤 함수가 다른 함수를 호출한다면 해당 함수는 세로로 가까이 배치하세요.

<br>

### 가로 밀집

#### 가로 공백

```java
totalChars += lineSize;	// 연산자 앞 뒤로 공백으로 분리
lines.addLine(lineSize, lineCount);	// 쉼표 뒤 공백으로 인수 분리 명확히
```

#### 가로 정렬

```java
public class User {
  private   String            name;
  private   String            email;
  private   int               age;
  protected UserDetail        detail;
  private   SomeLongClassName someLongName;
  private   Long              requestParsingTimeLimit;
  
  public void User(String name, String email){
    this.name =                 name;
    this.email =                email;
    requestParsingTimeLimit	=   10000;
  }
}
```

위와 같이 선언하게 되면 선언부를 읽다가 변수 유형을 무시하고 변수 이름을 먼저 보게 돼요.

할당문의 경우에는 할당 연산자는 눈에 잘 안들어오고 변수 이름부터 보게돼요.

#### 들여쓰기(Indentation)

```java
public class Comment extends Text {
  public static final String REGEXP ="^#[%\r\n]*(?:(?:\r\n)|\n|\r)?";
  public Comment(Parent parent, STring, text) {super(parent, text);}
  public String render() {return "";}
}
```

IDE에서 자동으로 관리해주기 때문에 중요성을 잃어버리기 쉽지만 가독성에 아주 중요해요.

위와 같이 짧은 코드인 경우에는 한 줄에 쓰고싶은 욕망이 들지만 규칙을 지키기 위하여 들여쓰기를 하세요.

![cc-11.png]({{ "/assets/img/contents/cc-11.png"}})

> 들여쓰기 논쟁
>
> [커밋 통계](http://sideeffect.kr/popularconvention#java)에 따르면 스페이스파가 더 압도적이라고 하네요.

---

## 6. 객체와 자료구조

### 자료 추상화

```java
// 구체적 클래스
public class Point {
  public double x;
  public double y;
}

// 추상화 클래스
public interface Point {
  double getX();
  double getY();
  void setCartesian(double x, double y);
  double getR();
  double getTheta();
  void setPolar(double r, double theta);
}
```

위 구체적 클래스에서는 직교 좌표계 사용이 강제 된다는 것을 알 수 있어요.

추상화 클래스(인터페이스)를 이용한다면 원통 좌표계 같은 다른 좌표계도 사용이 가능하겠죠?

<br>

### 자료/객체 비대칭

![cc-12.png]({{ "/assets/img/contents/cc-12.png"}})

자료 구조와 객체는 서로 비대칭적인 특징을 지녀요.

무슨 말이냐면

**자료 구조**: **자료를 보여주며** 별도의 **함수를 제공하지는 않아요**.

**객체**: 추상화 뒤로 **자료를 숨기며** 그를 다루는 **함수를 제공해요**.

<br>

#### 절차적인 도형

```java
public class Square extends Shape {
  public Point point;
  public double length;
}

public class Circle extends Shape {
  public Point center;
  public double radius;
}

public class Rectangle extends Shape {
  public Point point;
  public double height;
  public double width;
}

public class Geometry {
  public final double PI = 3.14159265358979323846;
  public double area(Object shape) throws NoSuchShapeException {
    if(shape instanceof Square) {
      Square s = (Square)shape;
      return s.length * s.length;
    }
    else if(shape instanceof Circle) {
      Circle c = (Circle)shape;
      return PI * c.radius * c.radius;
    }
    else if(shape instanceof Rectangle){
      Rectangle r = (Rectangle)shape;
      return r.height * r.width;
    }
    
    throw new NoSuchShapeException();
  }
}
```

객체 지향 관점에서 봤을때 아름답지 못한 코드이지만

새로운 함수가 추가되었을 때 각각의 도형 클래스를 변경할 필요가 없는 코드예요.

<br>

#### 예시) 다형적인 도형

```java
public class Square extends Shape {
  public Point point;
  public double length;
  
  public double area() {
    return length * length;
  }
}

public class Circle extends Shape {
	public Point center;
	public double radius;
  public final double PI = 3.14159265358979323846;
  
  public double area() {
    return PI * radius * radius;
  }
}

public class Rectangle extends Shape {
  public Point point;
  public double height;
  public double width;
  
  public double area() {
    return height * width;
  }
}

public class Geometry {
  public final double PI = 3.14159265358979323846;
  public double area(Object shape) throws NoSuchShapeException {
    return shape.area();
    throw new NoSuchShapeException();
  }
}
```

다형성을 활용한 예시인데

이 경우에는 새로운 함수가 추가되었을때 각각의 도형 클래스들에 함수를 추가해주어야 하겠죠?

**때로는 단순한 자료구조와 절차적인 코드가 적합한 경우가 있다는 것을 확인하세요.**

<br>

### 디미터 법칙

디미터 법칙은 메서드가 자신이 조작하는 객체의 내부 구조를 몰라야한다는 법칙이에요.

```java
// 디미터 법칙 준수하지 않음(기차 충돌)
final String outputDir = ctx.getOptions().getScratchDir().getAbsolutePath();

// 디미터 법칙 준수
Options opts = ctx.getOptions();
File scratchDir = opts.getScratchDir();
final STring outputDir = scratchDir.getAbsolutePath();
```

위 예시에서는 getOptions로 불러온 객체의 getScratchDir로 불러온 객체의 getAbsolutePath를 가져오는데

위와 같은 코드를 **기차 충돌(Train wreck)**이라고 불러요.

연속으로 호출하는 객체가 자료구조라면 상관 없지만 객체라면 디미터 법칙에 위배된다고 할 수 있겠죠?

<br>

### 자료 전달 객체(Database Transfer Object)

```java
public class Address {
  private String street;
  private String city;
  private String state;
  private String zip;
  
  public Address(String street, String city, String state, String zip) {
    this.street=street;
    ...
  }
  
  public String getStreet(){
    return this.street;
  }
  ...
    
  public void setStreet(){
    this.street=street;
  }
}
```

일반적으로 자료 구조 객체는 공개 변수만 있고 함수는 없어요.

위와 같은 자료 구조 객체를 자료 전달 객체(Data Transfer Object, DTO)라고 부르며

데이터베이스와 통신하거나 소켓 메시지등에 유용하게 사용돼요.

#### 활성 레코드

```java
public class Address {
  private String street;
  private String city;
  private String state;
  private String zip;
  
  public Address(String street, String city, String state, String zip) {
    this.street=street;
    ...
  }
  
  public String getStreet(){
    return this.street;
  }
  ...
    
  public int save(){
    ...
  }
  
  public String findCity(){
    ...
  }
}
```

위 DTO코드에 save나 find같이 탐색 함수가 추가된 경우는 특수한 경우로 일반적으로 객체가 아닌 자료 구조로 가정하는데

간혹 해당 레코드에 개발자가 비즈니스 로직을 추가하는 경우가 있는데 바람직하지 못한 경우예요.

<br>

### 결론

자료구조는 함수를 추가하기 편하고

객체는 객체 타입을 추가하기 편해요.

요구사항이 새로운 동작을 추가하는 작업이 많은 경우 => 자료구조체 사용

요구사항이 새로운 자료형을 추가하는 경우가 많은 경우 => 객체 사용

---

## 7. 오류 처리

### Try-Catch-Finally문부터 작성하라

![cc-13.png]({{ "/assets/img/contents/cc-13.png"}})

Try catch finally 블록은 트랜잭션과 비슷한 역할을 하기 때문에

예외가 발생하는 상황이면 항상 try-catch-finally 문부터 시작하세요.

<br>

### 예외에 의미를 부여하라

예외에 발생하는 호출 스택만으로는 오류 원인과 위치를 찾기 부족할 때가 있는데

애플리케이션 로깅 기능과 같은 방법을 이용하여 오류 메시지에 충분한 정보를 담아 보내세요.

<br>

### 예외 클래스를 정의하라

```java
LocalPort port = new LocalPort(8080);

try {
  port.open();
} catch (DeviceResponseException e) {
  reportPortError(e);
  logger.log("Device resposne exception", e);
} catch (UnlockedException e) {
  reportPortError(e);
  logger.log("Unlock exception", e);
} catch (GMXError e) {
  reportPortError(e);
  logger.log("Device resposne exception", e);
} finally {
  ...
}
```

위 코드와 같이 발생하는 모든 에러에 대해서 처리하는 것은 불필요한 중복코드를 발생시켜요.

아래와 같이 래퍼(Wrapper) 형태의 예외 클래스를 정의 후 예외 유형을 하나만 사용하는 것이 좋아요.

```java
public class LocalPortWrapper {
  private LocalPort innerPort;
  
  public LocalPortWrapper(int portNumber) {
    innerPort = new LocalPort(portNumber);
  }
  
  public void open(){
  try {
    port.open();
    } catch (DeviceResponseException e) {
      throw new PortDeviceFailureException(e);
    } catch (UnlockedException e) {
      throw new PortDeviceFailureException(e);
    } catch (GMXError e) {
      throw new PortDeviceFailureException(e);
    } finally {
      ...
    }
  }
}
```

```java
try {
  port.open();
} catch (PortDeviceFailureException e) {
  reportPortError(e);
  logger.log(e.getMessage(), e);
} finally {
  ...
}
```

외부 API를 사용하는 경우 래퍼 클래스가 매우 유용해요.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                         <br>

### null을 반환하지 마라

```java
public void register(User user){
  if(user != null) {
    UserRegistry registry = persistentStore.getUserRegistry();
    if(registry != null){
      User exists = registry.getUser(user.getID());
    }
  }
}
```

null값을 반환하는 코드는 개발자에게 null 체크를 강요하기 때문에 좋지 않아요.

누구 하나라도 null 검사를 누락하는 순간 프로그램은 NPE(Null Pointer Exception)이 발생하며 멈출수도 있기 때문에

메서드에서 null을 반환하고 싶다면 대신 예외를 던지거나 특수 사례 객체를 반환하세요.

```java
public class Calculator {
  public double xProject(Point p1, Point p2){
    if(p1==null || p2==null) {
      throw InvalidArgumentException("Invalid Argument for Calculator.xProject");
    }
    return (p2.x - p1.x) * 1.5;
  }
}
```

대부분의 프로그래밍 언어는 호출 인자의 null을 적절히 처리하는 방법이 없기 때문에

애초에 null을 반환하거나 인수로 넘기는 경우를 차단하는 정책이 합리적이에요.

---

## 8. 경계

개발할 때 시스템에 들어가는 외부 패키지나 라이브러리를 사용해요.

**인터페이스 제공자**: 확장성을 최대한 넓혀 더 많은 환경에 제공하려 하고

**인터페이스 사용자**: 자신의 요구사항에 부합하는 구체적인 인터페이스를 원해요.

따라서 이러한 인터페이스 경계에서 문제가 발생할 소지가 있어요.

<br>

### 외부코드 사용하기

```java
Map<String, Device> devices = new HashMap<>();
Device device = devices.get(deviceId);
```

위와 같이 사용할 때 해당 map 인스턴스를 여기저기서 사용한다면 인터페이스가 변경되었을 경우 수정할 코드가 상당히 많아지겠죠?

```java
public class Devices {
	private Map<String, Device> devices = new HashMap<>();
  
  public Device getDeviceById(String id) {
    return (Device) devices.get(id);
  }
}
```

위와 같이 경계 인터페이스인 Map을 Devices 안으로 숨기면 Devices 클래스 안에서 객체 유형을 관리하기 때문에 

Map 인터페이스가 변경되더라도 나머지 코드에는 영향을 끼치지 않아요.

경계 인터페이스를 오픈 API의 인수나 반환값으로 설정하지 않기 때문에 변경에 유리해요.

<br>

### 아직 존재하지 않는 코드 사용하기

개발 도중 아직 정의되지 않은 외부 코드와 인터페이스 해야할 때가 있는데

통제 밖이거나 정의되지 않은 코드를 사용할 때 별도 인터페이스로 분리하여 설계하세요.

자체 인터페이스를 정의 후 정의된 코드만 구현하면 코드는 훨씬 깔끔해지기 때문이에요.

추후에 어댑터를 사용하여 실제 구현된 인터페이스와의 통신이 가능하기도 하고요.

<br>

### 깨끗한 경계

경계에 위치하는 코드(외부 라이브러리, 외부 패키지)는 깔끔히 분리하세요.

외부 패키지를 세세히 알 필요 없이 통제가 불가능한 외부 패키지 의존성을 낮추고 통제가 가능한 우리 코드로 의존하도록 코드를 작성하세요.

1. 외부 패키지 호출 코드를 가능한 줄이기
2. 새로운 클래스로 경계 인터페이스를 감싸기
3. 어댑터 패턴을 사용하여 우리 인터페이스로 변경하기

---

## 9. 단위 테스트

### TDD 이전의 테스트

 단순히 프로그램이 돌아가는 사실을 확인하는 일회성 코드였어요.

### TDD 법칙

지금은 테스트 케이스를 모두 구현하고 통과해야하며 애자일과 TDD를 이용하여 단위테스트를 자동화하는 방법을 사용해요.

1. 실패하는 단위 테스트 작성할 때까지 실제 코드 작성 금지
2. 컴파일을 실패하지 않으면서 실행이 실패하는 정도로만 단위테스트 작성
3. 현재 실패하는 테스트를 통과할 정도로만 실제코드 작성

<br>

### 테스트코드

테스트코드는 실제 코드 못지 않게 중요해요.

테스트케이스는 **예제로 보여주는 문서**이고

잘 만든 테스트 케이스는 그 자체로 훌륭한 문서 역할을 해요.

단위 테스트는 코드를 유연하게 하고 유지보수 쉽게 하고 재사용을 쉽게 만들어요.

#### 테스트코드가 없는 경우

- 모든 변경이 잠정적인 버그가 돼요.
- 한 쪽을 수정하면 다른 쪽에서 안전하다는 보장을 하지 못해요.
- 의도하지 않은 결함이 발생해요.

<br>

### 깨끗한 테스트 코드 만들기

![cc-14.png]({{ "/assets/img/contents/cc-14.png"}})

깨끗한 테스트 코드를 만들려면 세 가지가 필요해요.

1. **가독성**
2. **가독성**
3. **가독성**

테스트코드는 최소의 표현으로 많은 것을 나타내야 하기 때문에

**BUILD-OPERATE-CHECK** 패턴을 사용하는 것이 좋아요.

테스트 자료를 만들고, 조작하고, 확인하기

잡다한 코드를 지우고 위 3가지에 해당하는 코드만 작성하세요.

<br>

### 도메인 특화 테스트 언어

```java
public class UserAssert extends AbstractAssert<UserAssert, User> {
 
    private UserAssert(User actual) {
        super(actual, UserAssert.class);
    }
 
    public static UserAssert assertThatUser(User actual) {
        return new UserAssert(actual);
    }
 
    public UserAssert hasEmail(String email) {
        isNotNull();
 
        Assertions.assertThat(actual.getEmail())
                .overridingErrorMessage( "Expected email to be <%s> but was <%s>",
                        email,
                        actual.getEmail()
                )
                .isEqualTo(email);
 
        return this;
    }
  ...
}
```

```java
@Test
public void registerNewUserAccount() throws Exception {
  ...
    assertThatUser(createdUserAccount)
      .hasEmail(REGISTRATION_EMAIL_ADDRESS)
      .hasFirstName(REGISTRATION_FIRST_NAME)
      .hasLastName(REGISTRATION_LAST_NAME)
      .isRegisteredUser()
      .isRegisteredByUsingSignInProvider(SOCIAL_SIGN_IN_PROVIDER);
}
```

도메인 특화 언어(DSL)로 테스트 코드를 구현하는 기법으로

API 위에다 함수와 유틸리티를 구현한 테스트 API로 쉽게 말해 직접 테스트용 API를 구현하는 방법이에요.

<br>

### 이중 표준

실제 서버 환경과 테스트 환경은 판이하게 달라요.

실제 환경: CPU, 메모리 등의 자원이 제한적(예: 임베디드 시스템)

테스트 환경: 자원이 제한적이지 않은 경우가 대부분

따라서 CPU, 메모리 자원 관리에 효율적인 구현보다 읽기 쉬운 테스트 코드를 작성하세요.

<br>

### 테스트당 assert 하나

```java
public void testGetPageAsXML() throws Exception {
  givenPages("PageOne", "PageTwo");
  whenRequestIsIssued("root", "type:pages");
  assertTrue(...);
}
```

JUnit 테스트 코드 작성시 단일 함수당 단일 assert문을 사용하는 것이 좋아요.

단위 테스트 = 결론이 하나 = 이해가 쉬움

<br>

### 테스트당 개념 하나

```java
public void testAddMonths() {
	SerialDate d1 = SerialDate.createInstance(31, 5, 2004);
	SerialDate d2 = SerialDate.addMonths(1, d1);
	assertEquals(13, d2.getDayOfMonth());
	assertEquals(1, d2.getMonth());
	assertEquals(2022, d2.getYYYY());
}
```

assert문을 여러개 써야 하는 경우가 많은데 이 경우

assert문을 하나만 쓰는 것이 아니라 한 개념에 대해서 하나의 테스트 수행하도록 작성하세요.

<br>

### F.I.R.S.T 규칙

- **Fast**: 빠르다.
  - 테스트는 자주 돌려야 하므로 빨리 수행되어야 해요.
  - 느린 테스트 코드는 돌릴 엄두가 안나기 때문에 코드 품질을 저해하기 때문이에요.



- **Independent**: 독립적이다.
  - 테스트는 서로 의존성이 없어야 해요.
  - 독립적으로 어떤 순서 없이 실행될 수 있어야 해요.



- **Repeatable**: 반복가능하다.
  - 테스트 수행이 불가능한 환경이 없도록 반복 가능해야 해요.
  - 네트워크에 연결되지 않고도 실행할 수 있어야 해요.



- **Self-Validating**: 자가검증한다.
  - 테스트는 bool 값으로 결과를 내야해요. 즉, 성공이냐 실패냐로 결과가 나와야 해요.
  - 통과 여부를 판단하기 위해 추가 작업이 들어가면 안돼요 (ex: 문자열 리턴, 상수값 비교 등)



- **Timely**: 적시에 작성한다.
  - 단위 테스트는 실제 코드 구현 직전에 구현해요.
  - 실제 코드 구현 이후에 테스트 코드 작성시 테스트하기 매우 어렵기 때문이에요.



---

## 10. 클래스

### 클래스 관례

1. 변수 목록

   1. 정적 공개 상수(public static)

   2. 정적 비공개 상수(private static)

   3. 비공개 인스턴스 변수(private)

   4. 공개 변수(public)

      > 사실 공개 변수가 필요한 경우는 거의 없어요.

2. 공개 함수(public)

   1. 비공개 함수(private)는 자신을 호출하는 함수 직후에 작성하세요.

일반적으로 클래스를 작성할때 선언하는 순서는 위와 같은 관례를 따르기 때문에

잘 작성된 클래스는 읽는 것 만으로도 추상화 단계를 순차적으로 내려가며 기사를 읽듯이 읽혀요.

#### protected 접근자

변수나 함수에 접근은 private로 접근을 기본적으로 공개하지 않는 편이 좋아요.

하지만 테스트 코드와 같이 접근을 허용해야하는 경우 protected 접근자 사용하세요.

<br>

### 작게 만들어라

- 규칙 1: **크기가 작아야한다.**

- 규칙 2: **크기가 더 작아야한다.**

함수의 크기가 작다: 행 수가 작다는 것을 의미하고

클래스의 크기가 작다: 클래스의 책임의 수가 작다는 것을 의미해요.

<br>

### 단일 책임 원칙(Single Responsibility Principle, SRP)

모든 클래스는 하나의 책임만을 가져야하는 원칙이에요.

```java
public class SuperDashboard extends JFrame implements MetaDataUser {
  public Component getLastFocusedComponent();
  public void setLastFocused(Component lastFocused);
  public int getMajorVersionNumber();
  public int getMinorVersionNumber();
  public int getBuildNumber();
}
```

위 코드의 경우 함수의 메서드 수는 적절할지 몰라도 클래스의 책임이 너무 많아 보이네요.

1. 버전 정보에 대한 책임
2. 컴포넌트에 대한 책임

위 두가지에 해당하는 책임을 가지고 있으므로 단일 책임 원칙에 위배된다고 할 수 있겠어요.



```java
public class Version {
  public int getMajorVersionNumber(){}
  public int getMinorVersionNumber(){}
  public int getBuildNumber(){}
}
```

위와 같이 분리하는 것이 좋아요.

큰 클래스 몇 개가 아니라 작은 클래스가 여러개인 시스템이 바람직해요.

<br>

### 인터페이스로 변경으로부터 격리

요구사항은 변한다 => 코드가 변한다 => 변경으로부터 격리 필요

```java
public interface StockExchange {
  String currentPrice(String ticker);
}

public Portfolio {
  private StockExchange exchange;
  public Portfolio(StockExchange exchange) {
    this.exchange = exchange;
  }
}

public class SeoulStockExchange implements StockExchange {
  public String currentPrice(String ticker){
    return "1000원";
  }
}
```

5분마다 변경되는 StockExchange 구현체가 있다고 가정해볼까요?

테스트 코드를 작성하기가 쉽지 않네요.

인터페이스/추상 클래스를 사용하여 구현을 변경으로부터 격리하여 테스트 코드 사용시 항상 1000원을 반환하게 한다면 테스트코드 작성이 수월해져요.

이처럼 인터페이스를 사용하여 시스템 결합도를 낮추고 유연성과 재사용성을 높일 수 있어요.

---

## 11. 시스템

### 시스템 생성과 시스템 사용을 분리

```java
public Service getService() {
  if(service==null)
    service = new MyServiceImpl(...);
  return service;
}
```

시스템 사용과 시스템 생성을 분리하지 않고 사용하면 사용과 생성을 같은 로직에서 처리하기 때문에 단일 책임 원칙에 위배돼요.

<br>

### Main 분리

<div class="mermaid">
flowchart LR;
A[main]
B[생성자]
C[App]
D[객체]
A --1. 구축--> B;
B --2. 생성--> D;
A --3. 실행--> C;
C --4. 사용--> D;
</div>
화살표가 main에서 애플리케이션으로 간다 => 애플리케이션은 객체의 생성이나 파괴 같은 라이프사이클에 관여하지 않는다는 뜻이에요.

<br>

### 팩토리

<div class="mermaid">
flowchart LR;
A[main]
B[Factory Implementation]
C[App]
D[Factory Interface\n+ MakeItem]
E[Item]
A --1. 구현--> B;
B --2. 구현--> D;
A --3. 실행--> C;
C --4-1. 객체 요청--> D;
C --4-2. 객체 생성--> E;
D --5. 사용--> E;
</div>
어플리케이션이 객체의 생성 시점만 정의하고 싶은 경우 팩토리 패턴을 사용하세요.

앱이 생성 시점을 결정하지만 객체의 생성은 앱이 몰라요.

<br>

### 의존성 주입(DI)

생성과 사용을 분리하는 제어 역전 기법을 의존성 관리에 적용한 것이에요.

객체지향 관점에서 대부분의 객체는 의존성을 가지고 있어요.

단일 책임 원칙에 따라 의존성을 생성하는 책임은 다른 곳에 전가할 수 있어요.

클래스는 완전히 수동적인 존재로 의존성을 해결하지 않으며 의존성을 주입 받아 사용하고

스프링 프레임워크가 가장 널리 알려진 자바 DI 컨테이너를 핵심 기능으로 제공해요.

#### 초기화 지연(Lazy Initialization)

```java
public Service getService() {
  if(service==null)
    service = new MyServiceImpl(...);
  return service;
}
```

위 코드는 생성과 사용을 분리하지 않은 로직

위 코드에서는 초기화 시점에 모두 생성하는 것이 아닌 사용직전에만 인스턴스를 생성해요.

이를 **초기화 지연(Lazy Initialization)**이라고 한답니다.

DI 개념을 써도 초기화 지연 사용이 가능하며

대부분의 DI 컨테이너에서 팩토리, 프록시, 레이지 로딩등을 지원해요.

<br>

### 확장

작은 마을에 6차선 도로를 깔지는 않겠죠?

작은 마을 -> 도시 -> 큰 도시로 성장하며 확장이 일어나는 방식이에요.

처음부터 올바르게 시스템을 만들 수 없다는 뜻이에요.

#### 시스템 구현

- 당장 주어진 요구사항과 유저 스토리에 맞추어 구현하여야 한다.

시스템은 반복적이고 점진적인 발전이 필요 => 확장성 고려해야 해요.

이를 성취하기 위해 애자일, TDD를 도입하고 리팩터링을 사용하는 것이라고 볼 수 있겠죠.

물리적인 시스템과 달리 소프트웨어 시스템은 관심사를 적절히 분리하여 확장성을 가져갈 수 있어요.

<br>

### 횡단 관심사

로깅, 트랜잭션 등의 관심사를 소스 코드가 아닌 외부에 구성하는 것으로

소스 코드에서 부가 기능(로깅, 트랜잭션, 예외 등)을 횡단 분리(cross-cutting)하는 것이에요.

#### 관점 지향 프로그래밍(Aspect-Oriented Programming)

횡단 관심사를 모듈화로 확보하는 방법론으로 모듈 구성 개념은 관점(Aspect)으로

Aspect를 분리하여 공통 기능을 비즈니스 로직과 코드에 영향을 주지 않게 적용할 수 있어요.

<br>

### 자바 프록시

```java
User user = (User) Proxy.newProxyInstance(
  User.class.getClassLoader(),
  new Class[] { User.class },
  new UserProxyHandler(new UserImpl()));
```

자바 프록시를 사용하기 위해서는 인터페이스와 구현체를 정의하고 InvocationHandler를 구현해야하는 등

코드양이 많으며 복잡하기 때문에 깨끗한 코드를 작성하기 어려워요.

<br>

### 순수 자바 AOP 프레임워크

대부분의 프록시 코드는 비슷한 형태 => 프레임워크 등장(Spring AOP, JBoss AOP)

AOP 프레임워크는 내부적으로 프록시를 사용해요.

#### 스프링의 경우

내부적으로 POJO를 사용해요.

> **POJO(Plain Old Java Object)?**
>
> ![sf-7.png]({{ "/assets/img/contents/sf-7.png"}})
>
> [https://martinfowler.com/bliki/POJO.html](https://martinfowler.com/bliki/POJO.html)
>
> POJO는 종속성이 없는 일반 자바 클래스 객체를 말하는 것이며 그다지 새로운 개념이 아니에요.
>
> 마틴 파울러(Martin Folwer)는 비즈니스 로직을 구현할 때 일반 자바 객체를 사용하는 것이 EJB를 사용하는 것 보다 훨씬 많은 장점을 가지고 있다고 생각하였고 
>
> 사람들이 일반 자바 객체를 사용하는 것을 망설이는 이유가 그럴듯한 이름이 없다고 생각했어요.
>
> 그래서 그럴듯한 이름을 붙여주었더니 아주 잘 나갔다는 후문이 있네요.

POJO는 어떠한 의존성도 가지고 있지 않기 때문에 단순하며 간편해요.

스프링은 빈과 같은 정보를 XML 파일에 명시하여 사용하는데

XML은 읽기 어렵다는 단점에도 프록시 보다는 간단해요.

<br>

### Test Driven 아키텍처 구축

관점으로 관심사(도메인)를 분리(낮은 결합)하는 것

애플리케이션을 POJO로 작성한다면 코드 수준에서 관심사가 분리돼요.

소프트웨어는 물리적인 형체가 없으므로 관점 분리를 통해 효과적인 변경이 가능해요.

=> 테스트 주도 아키텍처 설계가 가능하다는 뜻이에요.

건축설계: BDUF(Big Design Up Front)

> 구현 전에 가능한 모든 것을 설계하는 것

소프트웨어: 단순하면서 확장성있고 가용성이 있는 결과물을 재빨리 출시하여 점진적 확장하는 방식이 맞는 방식이에요.

<br>

### 표준이 항상 정답은 아니다

기존 EJB2는 표준이라는 이유만으로 많이 사용되었는데

가벼운 설계로도 충분한 프로젝트도 EJB2를 채택하는 경우가 존재하여 고객 가치가 뒤로 밀려나는 경우도 있어요.

항상 표준이 정답은 아니다라는 점 확인하세요.

<br>

### 결론

POJO를 사용하여 관점을 분리하고 의도를 명확히 표현하여 기민성을 높여야하고

시스템은 실제로 돌아가는 가장 단순한 방법을 사용해야해요.

---

## 12. 단순 설계 규칙

### 켄트 벡의 단순한 설계 4가지(중요도순)

- 모든 테스트를 실행하세요.
- 중복을 없애세요.
- 프로그래머 의도를 표현하세요.
- 클래서와 메서드 수를 최소로 줄이세요.

<br>

### 단순 설계 규칙

#### 1. 모든 테스트를 실행하세요

테스트가 불가능한 시스템 = 검증이 불가능하다 = 절대 출시하면 안된다.

테스트가 가능한 시스템: 충분한 테스트를 거쳐 **모든 테스트 케이스**를 **항상 통과**하는 시스템

##### 테스트 케이스를 만들고 계속 테스트하세요

위 규칙을 따르면 자동적으로 객체지향 방법론이 지향하는 낮은 결합도와 높은 응집력을 달성할 수 있어요.

테스트 케이스가 많을수록 테스트를 수행하기 쉬워진다 => 테스트 코드 작성이 쉬워진다 => 테스트가 가능한 시스템

결합도가 높으면 테스트 케이스를 작성하기 어려움 => 테스트 코드 작성을 위해 DIP(의존 관계 역전 원칙), SRP(단일 책임 원칙), 인터페이스, 추상화등을 이용해 자연스럽게 결합도를 낮춘다. => 품질이 높아진다.

위 1번의 테스트 케이스를 모두 작성했다면 코드를 정리할 수 있어요.

1. 코드를 추가할때마다 설계가 제대로 되었는지 점검하세요.
2. 점검을 통해 코드를 깔끔하게 정리하세요.
3. 테스트 케이스를 돌려 기존 기능에 영향이 없는지 확인하세요.
4. 위 방법을 점진적으로 반복하세요.

**테스트 케이스가 있으니까** 시스템에 버그가 발생하거나 잘못되는 경우는 없어요.

#### 2. 중복을 없애라

중복은 다음과 같아요.

- 추가 작업
- 추가 위험
- 불필요한 복잡도

```java
int size() {
  return list.length();
}

// 위의 size 함수를 사용하여 중복을 줄인다.
boolean isEmpty() {
  return 0 == size();
}
```

```java
public void render(){
  Image newImage = ImgUtil.getScaledImage(image, scale);
  image.dispose();
  System.gc();
  image = newImage;
}

public void rotate(int degrees) {
  Image newImage = ImgUtil.getScaledImage(image, scale, degress);
  image.dispose();
  System.gc();
  image = newImage;
}
```

위 코드는 메서드간 중복코드가 발생하였네요.

아래와 같이 수정하여 중복을 제거해볼게요.

```java
public void render(){
  replace(ImgUtil.getScaledImage(image, scale));
}

public void rotate(int degrees) {
  replace(ImgUtil.getScaledImage(image, scale, degress));
}

private void replace(Image newImage) {
  image.dispose();
  System.gc();
  image = new Image;
}
```

#### 3. 프로그래머의 의도를 표현하라

코드 작성자는 작성 시점에 자신의 코드에 푹 빠져 있어 이해하기 쉽지만

코드를 유지보수하거나 보는 사람이 깊이 있게 이해하기란 불가능해요.

> 하지만 나중에 코드를 읽을 사람은 바로 자기 자신임을 명심하세요.

개발자의 의도를 분명히 표현하세요.

- 좋은 이름을 짓는다
- 함수와 클래스 크기를 최대한 줄인다
- 표준 명칭을 사용한다
- 단위 테스트 케이스를 꼼꼼히 작성한다

#### 4. 클래스와 메서드 수를 최소한으로 줄이세요

네 규칙 중 우선순위가 가장 낮음

- 중복을 제거한다
- 의도를 표현한다.
- SRP를 준수한다.

가능한 엄격한 정책은 멀리하고 항상 실용적인 방식을 택하세요.

클래스마다 인터페이스를 무조건 생성하라거나 자료 클래스와 동작 클래스를 무조건 분리해야 하는 

정책은 득보다 실이 많은 경우가 있었어요.

---

## 13. 동시성(Concurrency)

### 동시성의 필요성

작업 처리량(Throughput)을 개선하기 위하여, 처리 속도를 개선하기 위하여 필요해요.

사용자 처리에 1초가 걸린다 => 동시 사용자가 1000명이라면? => 동시성 필요

대량의 데이터 분석 필요 => 정보를 나누어 여러 시스템에서 분석한다면 더 빠르다 => 동시성 필요

**싱글 스레드 프로그램**

- 직관적이며 구현이 간편해요.
- 눈에 잘 보여요.
  - 디버깅할때 호출 스택만으로 상태가 드러나요.

**멀티 스레드 프로그램**

- 비직관적이며 구현이 어려워요.
- 눈에 잘 안보여요.
  - 거대한 루프문 하나인 프로그램이 아닌 작은 여러개의 프로그램으로 보여요.

<br>

### 동시성의 오해

1. **동시성은 항상 성능을 높여준다**
   - 동시성은 특수한 상황에서 성능을 높여줘요.
   
   - 여러 스레드가 프로세스를 공유하거나 처리할 독립적인 계산이 많은 경우에만 성능이 높아져요. 
   
   - 일반적인 상황에서 항상 성능이 좋아지는 것은 아니에요.
   
2. **동시성을 구현해도 설계는 변하지 않는다**
   - 싱글 스레드 시스템과 멀티 스레드 시스템은 설계가 완전히 다르답니다.
   
3. **웹 컨테이너를 사용하면 동시성을 이해할 필요가 없다**
- 컨테이너 동작 원리, 동시 수정, 데드락과 같은 문제를 이해해야만 해요.

<br>

### 동시성의 이해

1. **동시성은 부하를 유발한다**

   - 성능 부하가 존재하며 코드도 더 작성해야해요.

2. **동시성은 복잡하다**
   - 간단한 문제라도 동시성을 구현하려면 매우 복잡해져요.
   
3. **동시성 버그는 재현이 어렵다**
   - 동시성 버그는 재현이 어려워 진짜 버그가 아니라 일회성 버그라고 여기기 쉬워요.
   
4. **동시성 구현 하려면 근본적인 설계 재고가 필요하다**

<br>

### 동시성 구현의 어려움

```java
public class X {
  private int number;
  
  public int getNextId() {
    return ++number;
  }
}
```

#### 상황

스레드는 2개이며 단일 인스턴스 X에 동시 접근하는 상황이에요.

#### 결과

1. 첫번째 결과

   - 1번 스레드 number: 43

   - 2번 스레드 number: 44

   - 인스턴스 X의 number: 44

2. 두번째 결과

   - 1번 스레드 number: 44

   - 2번 스레드 number: 43

   - 인스턴스 X의 number: 44

3. 세번째 결과

   - 1번 스레드 number: 43

   - 2번 스레드 number: 43

   - 인스턴스 X의 number: 43

1번과 2번은 예상 가능하나 3번과 같은 잘못된 결과가 발생한다는 것을 확인할 수 있어요.

> JIT 컴파일러의 바이트코드 처리 방식과 JVM의 최소 단위 등을 고려했을 때 두 스레드의 getNextId() 메서드 실행 경로는 최대 12870개라고 한다.

대다수 경로는 올바른 결과를 내놓지만 **잘못된 결과를 내놓는 일부 경로**가 존재해요.

<br>

### 동시성 방어 원칙

### 단일 책임 원칙(Single Responsibility Principle, SRP)

**동시성 코드는 다른 코드와 분리하세요**

동시성 코드는 동시성이라는 단일 책임을 가지고 있으므로 다른 코드와 분리가 필요해요.

ex) 동기 코드와 비동기 코드를 동시에 쓸 수 없어요.

<br>

### 자료 범위를 제한하세요

- 자료를 캡슐화한다.
- 공유 자료를 최대한 줄인다.

위 예시에서 보았듯 여러 스레드가 동일 객체를 공유한다면 스레드 간섭으로 잘못된 결과가 나올 수 있어요.

공유 객체의 코드 내 임계 영역(Critical Section)을 줄이고 synchronized 등의 키워드로 보호가 필요해요.

<br>

### 자료 사본을 사용하세요

- 공유 객체를 읽기 전용으로 사용
- 공유 객체를 복사하여 사용 후 해당 사본에서 결과 가져오기

객체를 복사하는 시간과 부하를 고려하세요.

<br>

### 스레드는 독립적으로 구현하세요

스레드를 독자적인 시스템처럼 사용하세요.

- 다른 스레드와 자료 공유를 하지 않는다
- 가능하면 다른 프로세서에서 사용
- 각 스레드는 클라이언트 요청 하나를 처리
- 모든 정보는 비공유 출처에서 가져오고 로컬 변수에 저장하여 사용

<br>

### 멀티 스레드 용어

#### 한정된 자원(Bound Resource)

한정된 멀티 스레드 환경의 자원이에요.

ex) DB connection, 읽기/쓰기 버퍼

#### 상호 배제(Mutual Exclusion)

한 번에 한 스레드만 공유 자료나 공유 자원을 사용할 수 있는 경우(동시 접근 불가능한 경우)

#### 기아(Starvation)

스레드가 굉장히 오랫동안 또는 영원히 자원을 기다리는 경우

ex) 우선순위 스케줄링: 우선순위가 높은 작업이 계속 요청되면 우선순위가 낮은 작업은 기아 상태

#### 데드락(Deadlock)

![cc-4.png]({{ "/assets/img/contents/cc-4.png"}})

스레드가 서로 끝나기를 기다리는데 각자 필요한 자원을 모두 점유하고 있어 모두 진행이 불가능한 상태

#### 라이브락(Livelock)

두 스레드가 락의 해제와 획득을 무한 반복하는 상태로 진행이 되지만 멈춘 것처럼 보이는 상태

> 데드락과의 차이
>
> 데드락은 해당 자원을 획득하려 대기(Waiting/Pending)하는 상태이고
>
> 라이브락은 프로세스가 살아있는채로 유의미한 진행이 안되는 상태에요.

<br>

### 멀티 스레드 실행 모델

#### **생산자-소비자 모델**

Producer-Consumer 모델 ex) Message Queue

생산자 스레드가 정보를 생성해 버퍼나 큐 등의 대기열에 넣으면 소비자 스레드가 대기열에서 정보를 가져다 사용하는 모델이에요.

대기열: 한정된 자원

#### 식사하는 철학자들 모델

**Dining Philosophers 모델**

![cc-5.png]({{ "/assets/img/contents/cc-5.png"}})

철학자 왼쪽에는 포크가 있고 스파게티를 먹으려면 양손에 포크를 쥐어야하는데

포크가 사용중이라면 포크를 내려놓을 때까지 대기하는 모델이에요.

해당 문제는 실제로 여러 기업 애플리케이션에서 겪는 문제로 설계가 잘못되면 데드락, 라이브락, 처리 효율 저하등의 문제 발생할 수 있어요.

<br>

### 동기화 메서드

**공유 객체 하나에는 메서드 하나만 사용하세요**

동기화하는 메서드 사이에 의존성이 존재하면 동시성으로 찾아내기 힘든 버그 발생할 수 있어요.

<br>

### 동기화 부분 최소화

**임계 영역을 최소화하여 동기화를 최소화하세요**

자바에서는 `synchronized` 키워드로 공유 자원에 대한 락을 설정할 수 있는데

락이 설정된 임계 영역에서는 하나의 스레드만 접근 가능해요.

락 설정은 스레드를 지연시키고 부하를 발생시키므로 개수를 최소화 하는 것이 좋아요.

> 그렇다고 임계 영역의 개수를 줄이는 대신 영역을 거대하게 키우면 스레드 경쟁으로 인해 성능이 떨어질 수 있어요.

<br>

### 올바른 종료 코드 작성

**종료 코드를 초기부터 고민하고 동작하도록 시간을 투자하세요**

스레드를 모두 깔끔하게 종료하기란 매우 어려운 작업이에요.

아래와 같은 상황이 종종 발생하기 때문이에요.

- 부모-자식 스레드 관계 종료시
  - 부모 스레드는 자식 스레드가 모두 종료되어야함 => 자식 스레드가 데드락인 경우 영원히 종료하지 못함

- 생산자-소비자 모델 종료시
  - 생산자 스레드가 종료되었는데 소비자 스레드가 생산자 스레드의 메시지를 대기하는 상태

<br>

### 스레드 코드 테스트

#### 말이 안 되는 실패는 잠정적인 스레드 문제

수천분의 일의 확률로 말이 안되는 실패가 일어난 경우 대개 시스템 오류로 취급하기도 해요.

이런 실패는 아주 가끔씩 발견되어 단순 일회성 버그로 치부하고 넘어가는데 이 경우는 매우 위험한 경우에요.

잠정적인 동시성 스레드 문제로 고려하세요.

#### 스레드 환경과 스레드 환경 밖의 테스트 분리

스레드 환경 밖에서 테스트가 정상인지 확인할 필요가 있어요.

#### 다양한 환경을 지원하도록 구현

단일 스레드, 여러 스레드, 실행 중 스레드 바꾸기 등을 지원하도록 테스트 케이스를 작성하세요.

#### 다른 플랫폼에서 돌려보기

윈도우와 맥에서의 스레드 정책은 다르다는 점을 이해하세요.

<br>

### 결론

- 멀티 스레드 구현은 어렵다는 것을 이해
- 단일 책임원칙 준수
- 멀티 스레드의 잠정적인 동시성 오류 이해
- 공유 코드 임계 영역을 이해
- 아주 때때로 발생하는 일회성 오류를 넘기지 마세요.
- TDD를 적용하여 테스트를 쉽게 하세요.

---

## 17. 냄새(Code Smells)와 휴리스틱

이 장에서는 대표적인 코드 냄새와 휴리스틱에 대해서 나열식으로 기술해봤어요.

> **휴리스틱(Heuristics)**
>
> 시간이나 정보의 부족으로 합리적인 판단이 불가능할때 어림짐작하는 발견법

위의 장에서 설명한 내용과 중복되는 부분이 많으나 정리하는 차원에서 한번 쭉 읽어보세요.

<br>

### 주석

1. **부적절한 정보**
   - 변경 이력과 같은 정보
2. **쓸모 없는 주석**
   - 잘못된 정보 제공하는 오래된 주석
3. **중복된 주석**
   - 코드만으로 충분히 이해가는 내용이 담긴 주석
4. **성의 없는 주석**
   - 주절대지 않고 간결하고 명료하게 작성
5. **주석 처리된 코드**
   - 주석 처리 코드는 즉각 지우세요 => 소스 코드 버전 관리 시스템이 기억
   - 중요한 코드인지 아닌지 얼마나 오래된 코드인지 아닌지는 중요하지 않음

<br>

### 환경

1. **빌드가 여러 단계**
   - 빌드는 하나의 명령으로 끝나야 해요.
   - 부가적인 스크립트나 시스템 의존성을 가져오는 등의 추가 단계가 없어야 해요.
2. **테스트가 여러 단계**
   - 모든 단위 테스트는 단 한번의 클릭(명령)으로 수행 되어야 해요.
   - IDE에서 버튼 하나로 모든 테스트를 하는 것이 이상적

<br>

### 함수

1. **너무 많은 인수**
   - 함수의 인수는 없는 것이 가장 좋아요.
   - 있는 경우 인수를 최소화
   - 인수가 넷 이상인 경우 함수 재설계를 고려하세요.
2. **출력 인수**
   - 사용자 직관에 위배 => 인수는 항상 입력이어야 해요.
   - 함수의 상태를 변경해야 한다면 함수가 속한 객체의 상태를 변경하도록 바꾸세요.
3. **플래그 인수**
   - boolean 인수는 함수가 여러 기능을 수행한다는 뜻
   - 단일 책임 원칙에 위배되므로 플래그 인수는 사용하지 마세요.
4. **죽은 함수**
   - 아무도 호출 하지 않는 함수는 가차없이 지우세요. => 소스 코드 관리 시스템이 기억해요.

<br>

### 일반

1. **한 소스 파일에 여러 언어 사용**
   - 자바 소스 파일에 HTML, YAML, JavaScript 등을 함께 포함하지 않는 것이 좋아요.
2. **당연한 동작 구현하지 않는 경우**
   - 당연한 동작도 구현을 해야 저자를 신뢰하고 함수 기능을 직관적으로 이해할 수 있어요.
3. **경계를 올바르게 처리하지 않는 경우**
   - 모든 경계 조건을 찾아내고 테스트하는 테스트 케이스를 작성하세요.
4. **안전 절차 무시**
   - 컴파일러, IDE의 경고를 꺼버리면 빌드가 쉬워지나 끝없는 디버깅에 시달릴 수도 있어요.
5. **중복**
   - **이 책에 나오는 가장 중요한 규칙**
   - 코드에서 중복을 발견할 때마다 추상화 고려
     - 중복코드를 하위 루틴이나 다른 클래스 분리
     - if/else 문 등으로 같은 조건을 반복 확인하는 경우 다형성으로 대체
   - 어디서든 중복을 발생하면 없애세요.
   - 최근 15년동안 나온 디자인 패턴들은 대다수가 중복을 제거하는 잘 알려진 방법에 불과해요.
6. **잘못된 추상화 수준**
   - 추상화 인터페이스는 세부 구현에 관련된 상수, 변수, 유틸리티를 몰라야 해요.
7. **기초 클래스가 파생 클래스에 의존**
   - 기초 클래스가 파생 클래스에 의존하면 독립적으로 배포가 불가능해요.
8. **과도한 정보**
   - 잘 정의된 인터페이스는 많은 함수를 제공하지 않아 결합도가 낮아요.
9. **죽은 코드**
   - 아무도 호출하지 않는 함수
   - 조건 분기되어 실행되지 않는 함수(throw가 없는 경우의 catch 블록, switch/case의 불가능한 case 조건 등)
10. **수직 분리**
    - 변수와 함수는 사용되는 위치에 가깝게 선언하세요.
    - 비공개 함수는 처음 호출 직후에 정의하세요.
11. **일관성 부족**
    - 특정 개념을 일관성 있게 동일한 방법으로 구현하세요.
12. **잡동사니**
    - 사용되지 않는 변수, 함수, 기본 생성자 등을 지우세요.
13. **인위적 결합**
    - 함수, 상수, 변수 선언시에 시간을 들여 올바른 위치에 넣으려 고민하세요.
14. **기능 욕심**
    - 어떤 클래스가 다른 클래스의 변수와 함수에 관심을 가져서는 안돼요.
15. **선택자 인수**
    - 인수를 넘겨서 해당 인수로 함수의 동작을 분기하거나 제어하는 경우 => 새로운 함수로 분리
16. **모호한 의도**
    - 헝가리식 표기법, 매직 넘버, 행을 바꾸지 않은 수식 등
17. **잘못 지운 책임**
    - 코드를 배치하는 위치는 독자가 자연스럽게 기대하는 위치에 배치하세요.
18. **부적절한 static 함수**
    - 인스턴스 함수를 static 함수로 정의하는 실수 등
19. **서술적 변수**
    - 서술적인 변수 이름을 쓰면 가독성이 늘어나요.
20. **이름과 기능이 일치하는 함수**
    - 이름이 애매모호하면 문서를 찾아보지 않는 이상 기능을 추론하기 어려워요.
21. **알고리즘을 이해하세요**
    - 알고리즘을 충분히 이해하고 코드를 작성하세요.
    - 테스트케이스를 모두 통과하는 것으로는 부족해요.
22. **논리적 의존성은 물리적으로 드러내세요**
    - 예시) 55페이지 크기를 처리할 수 있는 A라는 클래스가 B 클래스를 사용한다면 B 클래스 역시 55페이지 크기를 처리할 수 있어야 해요.
    - B 클래스에 getMaxpPageSize()와 같은 메서드로 페이지 크기를 가져온다면 논리적 의존성이 물리적 의존성으로 변해요.
23. **if/else, switch/case문보다 다형성을 사용하세요**
    - switch문 대신 다형성 객체를 생성하여 사용하세요.
24. **표준 표기법을 따르라**
    - 팀은 업계 표준에 기반하여 팀 표준을 설정하고 그 표준은 팀원 모두가 따라야 해요.
25. **매직 넘버는 명명된 상수로 변경하세요**
    - 숫자는 명명된 상수 뒤로 숨기세요.
    - `SECONDS_PER_DAY=86400`
26. **정확하게 하세요**
    - 통화를 다뤄야한다면 부동 소수점을 쓰지 말고 정수로 선언
    - null을 반환한다면 항상 null체크를 하세요.
    - 동시성 코드를 작성한다면 적절한 락을 거세요.
27. **관례보다 구조를 사용하세요.**
    - switch/case 문보다 추상 메서드 + 기초 클래스
28. **조건을 캡슐화하세요**
    - boolean 논리는 생각보다 어려워요.
    - shouldBeDeleted 등의 변수명으로 확실한 의도를 밝히세요.
29. **부정 조건은 피하세요**
    - if(!shouldBeDeleted(obj)) 보다 if(shouldNotBeDeleted(obj))가 더 직관적으로 이해하기 쉽지 않나요?
30. **함수는 한 가지만 하세요**
    - 중첩 구조가 생길 정도로 길면 안되고 함수는 한 가지만 수행해야 해요.
    - 여러 가지를 수행하는 한 가지 함수보다는 한 가지를 수행하는 여러 함수를 사용하세요.
31. **숨겨진 시간적 결합**
    - 함수 호출 실행 순서를 보장하세요.
32. **일관성 유지**
    - 코드 구조가 일관적이면 남들도 그 일관성을 따른답니다.
33. **경계 조건 캡슐화**
    - 경계 조건은 코드 여기 저기서 처리하지 마세요.
    - 경계 조건을 별도 변수로 캡슐화하여 사용하세요.(ex: int nextLevel = level + 1)
34. **함수는 추상화 수준을 한 단계만 내려가야 함**
    - 함수에서 추상화 수준을 분리하세요.
35. **설정 정보는 최상위 단계에 두세요**
    - 기본 상수나 설정 관련 상수는 함수에 숨겨선 안되고 최상위 단계에 두세요.
36. **추이적 탐색을 피하세요**
    - 디미터 코드 법칙을 준수
    - myClass.doSomething()과 같이 단순해야 해요.

<br>

### 자바

1. **긴 import 목록을 피하고 와일드 카드를 사용하세요**
   - Import package.*;
   - 긴 Import 목록은 읽기에 부담스러워요.
   - 명시적 import는 실제 그 클래스가 반드시 존재해야 하지만 와일드카드 형태는 검색 경로를 추가하는 형식으로 진정한 의존성이 생기지 않는 장점이 있어요.
2. **상수는 상속하지 않는다**
   - 상수를 상속하면 계층에 가려져 가독성이 저해될 수 있어요.
   - 꼭 필요하다면 import static을 활용하세요.
3. **상수 vs Enum**
   - public static final int 보다 enum 열거체를 사용하세요.
   - 훨씬 유연하고 서술적이며 편하기 때문이에요.

<br>

### 이름

1. **서술적인 이름을 사용**
   - 소프트웨어 가독성의 90%는 이름이 결정한답니다.
   - 코드만 보고 어떤 기능을 하는 함수인지 어떤 값을 담고 있는 변수인지 혹은 상수인지 판단 가능하도록 작성하세요.
2. **적절한 추상화 수준에서 이름 지으세요.**
   - 구현을 드러내는 이름은 피하세요.
   - 인터페이스에 추상화 수준이 너무 낮은(너무 구현/구체적인) 이름은 피하세요.
3. **가능하다면 표준 명명법 사용**
   - 데코레이터 패턴의 경우 클래스명에 Decorator 사용 등
4. **명확한 이름**
   - 길지만 서술적인 함수명, 변수명을 사용하면 주석이 필요 없겠죠?
5. **긴 범위는 긴 이름을 사용**
   - 범위(스코프)가 커지면 서술적인 이름을 사용하고 지역변수와 같이 스코프가 작아지면 아주 짧은 이름을 사용하세요.
6. **인코딩을 피하세요**
   - 헝가리안 표기법을 사용하지 마세요.
   - 접두어로 자료형, 범위 같은 정보를 넣어서는 안돼요.
7. **이름으로 부수 효과를 설명**
   - 함수명에 분기 처리되는 부수 효과에 대한 내용도 이름에 작성하는 편이 좋아요.
   - `create() => createOrDefault()`

<br>

### 테스트

1. **불충분한 테스트**
   - 테스트는 잠재적인 버그가 발생할 만한 부분을 모두 커버해야해요.
2. **커버리지 도구를 사용**
   - 커버리지 도구는 테스트가 빠뜨리는 부분을 알려줘요.
   - 대다수 IDE에서는 테스트 커버리지를 시각적으로 표현해줘요.
3. **사소한 테스트를 생략하지 마세요**
   - 사소한 테스트는 오히려 짜기 쉽기 때문에 사소한 테스트의 가치는 실제 구현 코드보다 높아요.
4. **무시한 테스트는 그 자체로 모호함을 뜻한다**
   - 불분명한 요구사항의 경우 @Ignore나 테스트 케이스를 주석처리하게 되는데 컴파일이 가능한 경우 테스트 진행하세요.
5. **경계 조건 테스트**
   - 특이 케이스와 같은 경계 조건을 모두 테스트하세요.
6. **버그 주변은 철저히 테스트**
   - 한 함수에서 버그가 발생하면 그 주변에서 버그가 발생할 확률이 높기 때문이에요.
7. **실패 패턴을 확인**
   - 꼼꼼한 테스트 케이스를 작성했다면 실패하는 테스트 케이스의 패턴만 보고 문제를 진단할 수 있어요.
   - ex) 음수 값이 인수로 들어가는 케이스 모두 실패, 입력값이 5글자를 넘어가면 모두 실패 등
8. **테스트 커버리지 패턴을 확인**
   - 테스트 커버리지 패턴으로 실패 원인 파악 가능
9. **테스트는 빨라야해요**
   - 느린 테스트 케이스는 실행을 꺼리는 경향이 있어 테스트를 누락할 가능성 높아요.

---

## Reference

1. [Clean Code(클린 코드)](http://www.kyobobook.co.kr/product/detailViewKor.laf?mallGb=KOR&ejkGb=KOR&barcode=9788966260959)

