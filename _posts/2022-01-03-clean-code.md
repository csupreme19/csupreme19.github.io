---
layout: post
title: Clean Code 정리
feature-img: assets/img/titles/clean-code.png
thumbnail: assets/img/contents/cc-1.png
author: csupreme19
categories: Development
tags: [Book, Clean Code]

---

# Clean Code 정리

![clean-code.png]({{ "/assets/img/titles/clean-code.png"}})

[Clean Code(클린 코드)](http://www.kyobobook.co.kr/product/detailViewKor.laf?mallGb=KOR&ejkGb=KOR&barcode=9788966260959)

로버트 C.마틴 저 **클린 코드: 애자일 소프트웨어 장인 정신**을 읽고 정리해보았다.

---

## 1. 깨끗한 코드

코드(Code): 코드는 고객의 요구사항을 상세히 표현하는 언어

### 깨끗한 코드란?

- 우아하고 **보기에 즐거운** 코드 - [바야네 스트롭스트룹](https://ko.wikipedia.org/wiki/%EB%B9%84%EC%95%BC%EB%84%A4_%EC%8A%A4%ED%8A%B8%EB%A1%AD%EC%8A%A4%ED%8A%B8%EB%A3%B9)
- 단순하고 직접적이며 명쾌한 **잘 읽히는** 코드 - [그래디 부치](https://ko.wikipedia.org/wiki/%EA%B7%B8%EB%9E%98%EB%94%94_%EB%B6%80%EC%B9%98)
- **가독성이 좋으며** 다른 사람이 고치기 쉬운 코드 - ["빅" 데이브 토마스](https://en.wikipedia.org/wiki/David_A._Thomas_(software_developer))
- 세세하며 꼼꼼하며 **단정되게 정리**한 코드 - 마이클 페더스
- **중복이 없는** 간단하게 추상화된 코드 - [론 제프리스](https://en.wikipedia.org/wiki/Ron_Jeffries)

깨끗한 코드라는 것은 그 정의가 모호한 만큼 논쟁의 여지가 있으나 전문가들이 입을 모아 말하는 공통점은 **가독성**이다.

우리는 코딩을 할 때 실제로 코드를 작성하는 시간보다 코드를 읽고 이해하는 시간이 훨씬 많다는 것을 경험으로 알고 있다.

**가독성**이 높은 코드는 다른 사람이 고치기 쉬우며 이는 유지보수성이 좋다는 것을 의미한다.

결국 가독성이 높은 코드를 작성하려면 중복 줄이기, 간단한 추상화 고려하기, 철저한 오류 처리, 단위 테스트 등이 동반되어야 하므로 궁극적인 목표라고 볼 수 있겠다.

이 책에서 말하는 깨끗한 코드를 작성하는 방법 또한 항상 올바르다는 단정을 할 수는 없다.

하지만 이 책에서 말하는 여러 기법들은 오랫동안 심사숙고하며 축적된 교훈과 기법이므로 충분히 깨끗한 코드를 작성할 수 있을 것이다.

![cc-1.png]({{ "/assets/img/contents/cc-1.png"}})

> 코드 품질을 측정하는 유일한 척도 = WTFs/m
>
> [https://www.osnews.com/story/19266/wtfsm/](https://www.osnews.com/story/19266/wtfsm/)

<br>

---

## 2. 의미 있는 이름

소프트웨어에서의 이름은 어디에서나 쓰인다.

>  함수, 변수, 인수, 클래스, 패키지, 디렉토리, 빌드 파일

이름을 잘 짓는 것 만으로도 가독성이 높은 코드를 작성하는데 큰 도움이 된다.

![cc-2.png]({{ "/assets/img/contents/cc-2.png"}})

> 소프트웨어 공학에서 가장 힘든 것이 이름짓기라카더라

<br>

### 의도를 알 수 있도록하라

```java
int d;	// 의도가 분명하지 않음
int daysSinceCreation;	// 의도가 분명한 변수명
```

위 변수 이름을 보면 d라는 이름만을 보고 해당 변수가 어떤 데이터를 담고 있는지 알 수 없다.

```java
public static void copyChars(char a1[], char a2[]) {
  for (int i=0; i<a1.length; i++) {
    a2[i] = a1[i];
  }
}
```

a1[], a2[] 대신 source, destination을 사용한다면 바로 이해할 수 있다.

변수명이 모호하면 코드 맥락을 이해하기 힘들다.

<br>

### 잘못된 정보를 피하라

#### 1. 일반적인 단어

```java
int hp, aix, sco;
```

위의 단어들은 유닉스 플랫폼을 가리키는 이름이므로 혼란을 야기할 수 있다.

#### 2. 잘못된 타입 변수명

```java
Map<String, String> userList;
```

Map 자료형에 List라는 이름이 포함된 이름을 사용한다면 List 자료형으로 착각할 수 있다.

#### 3. 비슷한 이름

```java
public class XYZControllerStringHandler;
public class XYZControllerHandlerString;
```

서로 다른 역할을 하는 두 클래스가 위와 같은 이름을 가지고 있다면 어떤 것이 핸들러인지 알 수 있을까?

#### 4. l과 O와 같은 혼동될 수 있는 변수명

```java
int a = 1;
if ( O == 1 )
  a = O1;
else
  l = O1;
```

일반적으로 Consolas와 같은 개발 폰트를 사용하면 o, O, l, 1, I와 같은 헷갈리는 단어들을 구분해주기는 하지만 가독성을 저해한다.

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

위 클래스는 의미없는 불용어(Noise Word)를 추가한 명명법에 불과하다. 모두 Product라는 이름으로 대체할 수 있다.

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

변수명과 중복된 값이 들어가거나 의미없는 불용어가 추가되어 있다.

<br>

### 발음이 쉬운 이름

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

"젠 와이 엠 디 에이치 엠 에스"와 같은 우스꽝스러운 발음은 이해하기 힘들뿐더러 추가적인 설명을 요구한다.

아래와 같이 명명하면 누구나 쉽게 생성일자라는 것을 알 수 있다.

<br>

### 검색하기 쉬운 이름

```java
const int TASK_DAYS = 230;
const int WORK_DAYS_PER_WEEK = 5;

int taskWeeks = ( TASK_DAYS / WORK_DAYS_PER_WEEK);

int taskWeeks = (230 / 7);
```

만약 `WORK_DAYS_PER_WEEK` 대신 7로 검색한다면 7이 포함된 수많은 수식과 코드들이 나올 것이다.

<br>

### 헝가리식 표기법(Hungarian Notation)

> vUsing adjHungarian nnotation vmakes nreading ncode adjdifficult.
>
> [https://stackoverflow.com/a/112080](https://stackoverflow.com/a/112080)

```java
// 변수명 앞에 타입에 해당하는 문자를 붙여주었다.
boolean bCheck;
char chInitial;
int iSize;
// 멤버변수 사용시엔 m_ 접두어를 적어주기도 하였다.
int m_length;
String m_desc;
```

헝가리식 표기법은 예전 C 컴파일러가 타입을 점검하지 않거나 IDE의 기능이 부실했을 때 자료형을 쉽게 알 수 있도록 변수명에 타입을 적어주는 방법이었다.

요즘에는 컴파일러가 타입 체크를 할 뿐더러 IDE가 코드 작성 단계에서 타입체크를 하므로 사용하지 말 것.

<br>

### 인터페이스 클래스와 구현 클래스

```java
// Old
public Interface IShapeFactory {}
public Class ShapeFactory {}

// Now
public Interface ShapeFactory {}
public Class ShapeFactoryImpl {}
```

옛 코드에서 많이 사용하던 방식인데 인터페이스 이름에 접두어 I는 붙이지 않는 편이 좋다.

호출시에 인터페이스를 호출하게 되는데 굳이 인터페이스임을 알릴 필요도 없으며 (추상화) I라는 접두어로 추가적인 정보는 필요 없다.

위와 같이 명명하면 사용자는 인터페이스를 신경 쓸 필요 없이 ShapeFactory라고만 인식하여 호출하면 된다. 

<br>

### 문자 하나

```java
int a;
int b;
int c;
```

문자 하나만 사용하는 경우 최악이다.(반복문의 i, j, k는 전통적으로 이터레이션에 사용하였으므로 괜찮다.)

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

클래스명은 명사, 명사구로 이름을 짓는다.

<br>

### 메서드명

```java
// 동사
public int save() {}

// 명사 금지
public int success() {}
public void user() {}

// 자바의 경우 접근자, 변경자, 조건자는 자바빈 명세에따라 get, set, is 사용
public void setName() {}
public String getName() {}
```

메서드명은 동사, 동사구로 이름을 짓는다.

자바의 경우 접근자, 변경자, 조건자는 자바빈 명세에따라 get, set, is 접두어 사용

<br>

### 기발한 이름 피하기

```java
// 기발한 이름
public int detonate() {}

// 의도된 동작 이름
public int delete() {}
```

멋있지만 짜증난다.

<br>

### 한 개념에 한 단어 사용

```java
// 모두 동일
public int fetchName() {}
public int retrieveName() {}
public int getName() {}
public Class DeviceController {}
public Class DeviceManager {}
public Class DeviceDriver {}
```

위와 같이 같은 개념에 대하여 여러 단어를 혼동해서 사용한다면 가독성을 저해한다.

<br>

### 한 단어를 두가지 목적으로 사용 금지

```java
public int addNumber(int num1, int num2){}
public int addUser(List<User> users, User user){}

public int addNumber(int num1, int num2){}
public int insertUser(List<User> users, User user){}
```

위의 한 개념에 한 단어 사용과는 반대의 경우이다.

addNumber는 수를 더하는 것이고 addUser는 리스트에 User를 추가하는 것이다.

따라서 add가 아닌 insert, append와 같은 단어가 적절하다.

같은 행위가 아닌데도 같은 단어를 사용하는 것 또한 맥락을 해친다.

<br>

### 불필요한 맥락을 없애라

```java
public Class AAUser {}
public Class AAManagement {}
public Class AAAuthentication {}
```

AA(Admin App)이라는 애플리케이션을 작성한다고 가정했을때 위와 같이 AA를 모두 붙이게 된다면 A를 입력하고 자동완성할 때 모든 클래스가 열거된다.

바람직하지 못하다.



---

## 3. 함수

### 작게 만들어라

- 규칙 1: **작게!**

- 규칙 2: **더 작게!**

함수는 100줄을 넘어서는 안된다. 아니 20줄도 길다.

중첩 구조가 생길만큼 함수가 길면 안된다. 다시 말해, if/else, while문 등에 들어가는 블록은 한줄이어야 한다.

<br>

### 한가지만 해라!

- 규칙 1: 함수는 **한 가지**를 해야 한다.
- 규칙 2: 그 **한 가지**를 잘해야 한다.
- 규칙 3: 그  **한 가지**만을 해야 한다.

<br>

### 내려가기 규칙

코드는 위에서 아래로 이야기처럼 내려가면서 읽혀야 좋다.

각 함수에서는 한 가지만의 레벨을 구현하며 다음 함수를 호출한다.

<br>

### switch문 피해라

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

switch문은 피하되 불가피한 경우 다형성 객체로 한번만 사용하라

위 코드에서는 추상 팩토리에 switch문을 숨겨서 사용한다.

<br>

### 서술적인 이름 사용

```java
public static String testableHtml() {}

private void includeSetupAndTeardownPages() {}
```

짧고 어려운 이름보다 길고 서술적인 이름이 좋다.

해당 함수를 보고 어떤 역할을 할 수 있는지 알 수 있어야 한다.

<br>

### 함수 인수는 최대한 적게

```java
public void render(pageData);
public void render();
```

함수 인수는 0개가 가장 이상적이며 필요시 1개, 2개를 사용한다.

3개 이상은 가능한 피하며 4개는 특별한 사유가 있어야만 사용한다.

render(pageData)보다 render()가 훨씬 간결하고 이해하기 쉽다.

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

인수가 여러개인 경우 인수의 순서를 헷갈릴 가능성이 많다.

#### 플래그 인수

```java
render(true)
```

플래그 인수를 사용한다는 것은 해당 함수가 한 가지 이상의 일을 한다는 것을 의미한다.

참이면 A 로직, 거짓이면 B로직 = 2개의 로직

따라서 절대 사용하지 말 것.

#### 인수 객체

```java
Circle makeCircle(double x, double y, double radius);
Circle makeCircle(Point center, double radius);
```

인수가 여러개가 존재한다면 인수를 객체로 설정하여 인수를 줄일 수 있다.

<br>



---

## Reference

1. [Clean Code(클린 코드)](http://www.kyobobook.co.kr/product/detailViewKor.laf?mallGb=KOR&ejkGb=KOR&barcode=9788966260959)

