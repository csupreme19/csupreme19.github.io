---
layout: post
title: Hibernate 정리
feature-img: assets/img/titles/hibernate-logo.png
thumbnail: assets/img/contents/hibernate-1.png
author: csupreme19
categories: Development Spring
tags: [ORM, JPA, JDBC, Hibernate, DB, RDBMS, Java, Spring]
---

# Hibernate 정리

![hibernate-logo.png]({{ "/assets/img/titles/hibernate-logo.png"}})

본 문서에서는 대표적인 JPA 프레임워크인 하이버네이트에 대해서 정리해봤어요.

---

## Hibernate

![hibernate-1.png](/assets/img/contents/hibernate-1.png)

자바 객체를 데이터베이스에 저장하는 프레임워크예요.

<br>

### 특징

- 로우 레벨 SQL을 내부적으로 처리해줘요.

- JDBC(Java Database Connectivity) 코드 작성을 줄여줘요.

- ORM(Object to Relational Mapping)을 지원해줘요.

<br>

### ORM?

![hibernate-2.png](/assets/img/contents/hibernate-2.png)

Object To Relational Mapping; 객체 관계 매핑

쉽게 얘기하면 자바 클래스와 데이터베이스 테이블을 매핑시키는 것으로

하이버네이트는 JDBC 인터페이스에서의 내부 로직을 처리해주기 때문에 쿼리 작성, 객체 설정 등 추가 작업 없이 객체 형태의 CRUD가 가능해요.

```java
Person person = new Person("John Doe", "johndoe@email.com");

// 저장
int id = (Integer) session.save(person);

// 조회
Person thePerson = session.get(Person.class, id);

// 쿼리 예제(HQL)
Query query = session.createQuery("from Person");
List<Person> persons = query.getResultList();
```

<br>

### 하이버네이트는 JDBC를 사용한다

![hibernate-3.png](/assets/img/contents/hibernate-3.png)

기본적으로 하이버네이트 API를 이용하면 하이버네이트는 내부적으로 JDBC API를 사용하여 로우 레벨 로직들을 처리해줘요.

따라서 JDBC에 기반한 ORM 프레임워크로 하이버네이트를 설정한다는 것은 JDBC를 설정한다는 의미예요.

<br>

### Entiy 클래스

데이터베이스 테이블과 매핑되는 자바 클래스로

일반적으로 getter, setter, 생성자 등을 포함하는 POJO 형태로 정의되어요.

@Entity 애너테이션을 사용하여 설정할 수 있어요.

```java
@Entity
@Table(name="person")		// 기본 name은 클래스명
public class Person {
  
  @Id	// 해당 테이블의 PK인 필드
  @Column(name="id")
  private int id;
  
  @Column()
  private String name;
  
  private String email;	// 컬럼명과 필드명이 일치하면 @Column 선언 필요 없음
  
  @Column(name="phone_number")		// DB와 네이밍 컨벤션이 다른 경우 이와 같이 명시
  private String phoneNumber;
}
```

@Column 애너테이션을 사용하여 매핑되는 필드를 설정할 수 있어요.

<br>

### Session

```java
SessionFactory sessionFactory = new Configuration()
  .configure()
  .addAnnotatedClass(Person.class)
  .buildSessionFactory();
Session session = sessionFactory.getCurrentSession();
```

SessionFactory: 하이버네이트 설정 파일을 읽어 DB에 접속하는 세션을 생성한다. 앱 컨텍스트 내에서 한번만 생성돼요.

Session: SessionFactory에서 생성된 세션으로 이 객체를 사용하여 데이터를 저장하고 가져와요.

<br>

### Transaction

```java
try {
  Person person = new Person("John Doe", "johndoe@email.com");

  // 트랜잭션 시작
  session.beginTransaction();
  
  // 저장
  int id = (Integer) session.save(person);
  
  // 트랜잭션 커밋
  session.getTransaction().commit();
  
} catch(Exception ex) {
  ex.printStackTrace();
} finally {
  session.close();
  sessionFactory.close();
}
```

DB 트랜잭션과 같이 session도 트랜잭션을 가지고 있으며

해당 트랜잭션의 commit()을 수행하기 전까지 엔티티 데이터는 메모리 안에서만 존재하는 구조예요.

<br>

### Primary Key

```java
@Entity
@Table(name="person")
public class Person {
  
  @Id	// 해당 테이블의 PK인 필드
  @GeneratedValue(strategy=GenerationType.IDENTITY)
  @Column(name="id")
  private int id;
  ...
}
```

엔티티 생성시 프라이머리 키값을 설정하는 전략은 아래와 같이 4가지가 존재해요.

| GenerationType          | 설명                                                         |
| ----------------------- | ------------------------------------------------------------ |
| GenerationType.AUTO     | Hibernate가 데이터베이스에 알맞는 전략을 알아서 선택         |
| GenerationType.IDENTITY | 데이터베이스의 PK 생성 전략에 따라 설정 (AUTO_INCREMENT)<br>일반적으로 많이 사용해요. |
| GenerationType.SEQUENCE | 데이터베이스의 PK 시퀀스로 설정하여 키값을 설정해요.         |
| GenerationType.TABLE    | PK 유일성을 보장하는 데이터베이스 테이블을 설정하여 키값을 설정해요. |

<br>

### HQL(Hibernate Query Language)

하이버네이트 쿼리시 사용하는 쿼리 언어로 SQL과 비슷한 문법 구조로 되어있어요.

테이블명 대신 자바 클래스명, 컬럼명 대신 필드명을 사용해요.

```java
List<Person> persons = session
  .createQuery("from Person p where p.name LIKE '%John%'"
              + " OR p.email='johndoe@email.com'")
  .getResultList();
```

<br>

### Update

```java
try {
  // 트랜잭션 시작
  session.beginTransaction();
  
  int id = 1;
  Person person = session.get(Person.class, id);
  
  // 변경
  person.setEmail("johndoe2@email.com");
  
  // 변경(쿼리)
  session
    .createQuery("update Person set name='John Doe 2' where name='John Doe'")
    .executeUpdate();
  
  // 트랜잭션 커밋
  session.getTransaction().commit();
  
} catch(Exception ex) {
  ex.printStackTrace();
} finally {
  session.close();
  sessionFactory.close();
}
```

업데이트의 경우 별도로 update, save 등의 API를 호출하는 것이 아니에요.

하이버네이트에서 가져온 엔티티 객체는 영속성 객체로 해당 트랜잭션을 커밋하면 해당 객체의 변경사항이 모두 업데이트 돼요.

<br>

### CRUD 예제

```java
try {
  Person person = new Person("John Doe", "johndoe@email.com");
  
  // 트랜잭션 시작
  session.beginTransaction();
  
  // Create
  int id = (Integer) session.save(person);
  
  // Read
  Person john = session.get(Person.class, id);
  // Read(HQL)
  john = (Person) session
    .createQuery("from Person p where id= :id")
    .setParameter("id", id)
    .uniqueResult();
  
  // Update
  john.setEmail("johndoe2@email.com");
  // Update(HQL)
  session
    .createQuery("update Person set name='John Doe 2' where name='John Doe'")
    .executeUpdate();
  
  // Delete
  session.delete(john);
 	// Delete(HQL)
  session.
    createQuery("delete from Person where id= :id")
    .setParameter("id", id)
    .executeUpdate();
  
  // 트랜잭션 커밋
  session.getTransaction().commit();
  
} catch(Exception ex) {
  ex.printStackTrace();
} finally {
  session.close();
  sessionFactory.close();
}
```

<br>

### Entity Lifecycle

![hibernate-4.png](/assets/img/contents/hibernate-4.png)

세션 메서드를 사용하여 엔티티를 관리할 때 위와 같은 엔티티 라이프사이클을 가져요.

- New/Transient: new 키워드로 생성되거나 객체가 메모리 등에 존재하는 transient 상태
- Persistent/Managed: 객체가 저장된 상태
- Detached: 커밋되거나 롤백, 세션 클로즈 등으로 하이버네이트 세션에서 떨어진 상태
- Removed: 객체가 삭제된 상태

<br>

### One-To-One Mapping

<div class="mermaid">
erDiagram
	person ||--|| person_detail : One-To-One
	person {
		int id
		string name
		string email
		int person_detail_id
	}
	person_detail {
		int id
		string hobby
		string language
  }
</div>

FK로 join 되는 1:1 관계의 연관관계를 가지면 One-To-One Mapping이에요.

```java
@Entity
@Table(name="person_detail")
public class PersonDetail {
  
  @Id
  @GeneratedValue(strategy=GenerationType.IDENTITY)
  @Column(name="id")
  private int id;
  
  ...
}
```

```java
@Entity
@Table(name="person")
public class Person {
  
  @Id
  @GeneratedValue(strategy=GenerationType.IDENTITY)
  @Column(name="id")
  private int id;
  
  ...
  
  @OneToOne
  @JoinColumn(name="person_detail_id")
  private PersonDetail personDetail;
}
```

위와 같이 `@OneToOne`과 `@JoinColumn`를 사용해요.

테이블 엔티티를 필드로 선언하고 위 두 애너테이션을 명시해주면 

하이버네이트가 백그라운드에서 해당 엔티티의 컬럼을 FK로 참조하여 해당 객체를 가져

<br>

### 양방향 Mapping

<div class="mermaid">
  flowchart LR
    A[Person]
    B[Person Detail]
    A-->B
</div>

위 예시에서는 Person --> PersonDetail의 단방향 매핑만 가능해요.

PersonDetail 엔티티에도 아래와 같이 Person에 대한 참조 필드를 선언하면 양방향으로 매핑이 가능해요.

```java
@Entity
@Table(name="person_detail")
public class PersonDetail {
  
  @OneToOne(mappedBy="personDetail", cascade=CascadeType.ALL)
  private Person person;
  
  ...
}
```

```java
@Entity
@Table(name="person")
public class Person {

	...
   
  @OneToOne
  @JoinColumn(name="person_detail_id")
  private PersonDetail personDetail;
}
```

<div class="mermaid">
  flowchart LR
    A[Person]
    B[Person Detail]
    A-->B
    B-->A
</div>

mappedBy 프로퍼티를 통하여 Person 클래스 안의 personDetail이라는 필드를 참조해요.

양방향 매핑이 설정된 경우 두 엔티티 중 어떤 연산에 하든지 연산은 다른 엔티티로 전파(cascade)돼요.

<br>

### One-To-Many Mapping

<div class="mermaid">
erDiagram
	author ||--|{ post : One-To-Many
	author {
		int id
		string name
		string email
		int post_id
	}
	post {
		int id
		string title
		string content
  }
</div>
1:1로 연관관계가 설정된 One-To-One 매핑과 달리 1:N의 관계를 가지는 매핑이에요.

```java
@Entity
@Table(name="author")
public class Author {
  ...
  @OneToMany(mappedBy=author,
            cascade={CascadeType.PERSIST
                    , CascadeType.DETACH
                    , CascadeType.MERGE
                    , CascadeType.REFRESH})
  private List<Post> posts;
}
```

```java
@Entity
@Table(name="post")
public class Post {
  
  @Id
  @GeneratedValue(strategy=GenerationType.IDENTITY)
  @Column(name="id")
  private int id;
  
  ...
    
  @ManyToOne(cascade={CascadeType.PERSIST
                    , CascadeType.DETACH
                    , CascadeType.MERGE
                    , CascadeType.REFRESH})
  @JoinColumn(name="post_id")
  private Author author;
  
}
```

cascade의 경우 일반적으로 일대다 매핑에서 DELETE는 Cascade하지 않기 때문에 Delete를 제외해야해요.

위 예시를 들자면 게시물이 삭제된다고 작성자가 삭제되면 안되겠죠?

<br>

### Many-To-Many Mapping

<div class="mermaid">
erDiagram
	course ||--|{ course_student_mapping : One-To-Many
  student ||--|{ course_student_mapping: One-To-Many
  course_student_mapping {
  	int course_id
  	int student_id
  }
	student {
		int id
		string name
		string email
	}
	course {
		int id
		string title
		string content
  }
</div>

N:N 다대다 연관관계를 가지는 매핑이에요.

다대다 매핑의 경우 일반적으로 두 엔티티간의 매핑정보를 가지고 있는 조인 테이블을 별도로 가지고 있어요.

```java
@Entity
@Table(name="course")
public class Course {
  ...
  @ManyToMany(cascade={CascadeType.PERSIST
                    , CascadeType.DETACH
                    , CascadeType.MERGE
                    , CascadeType.REFRESH})
  @JoinTable(
    name="course_student_mapping",
    joinColumns=@JoinColumn(name="course_id")
    inverseJoinColumns=@JoinColumn(name="student_id")
  )
  private List<Student> students;
}
```

```java
@Entity
@Table(name="student")
public class Student {
  ...
  @ManyToMany(cascade={CascadeType.PERSIST
                    , CascadeType.DETACH
                    , CascadeType.MERGE
                    , CascadeType.REFRESH})
  @JoinTable(
    name="course_student_mapping",
    joinColumns=@JoinColumn(name="student_id")
    inverseJoinColumns=@JoinColumn(name="course_id")
  )
  private List<Course> courses;
}
```

`@ManyToMany` 애너테이션에 `@JoinTable` 애너테이션으로 매핑 테이블 명시가 필요해요.

다대다 매핑이므로 현재 엔티티에서 조인할 joinColumns과 연관 테이블에서 조인될 inversJoinColumns를 설정해주어야 해요.

<br>

### Cascade Type

| Cascade Type | 설명                                                         |
| ------------ | ------------------------------------------------------------ |
| PERSIST      | 엔티티가 저장되면 연관된 엔티티도 저장                       |
| REMOVE       | 엔티티가 삭제되면 연관된 엔티티도 삭제                       |
| REFRESH      | 엔티티가 리프레시되면 연관된 엔티티도 리프레시               |
| DETACH       | 엔티티가 디태치되면 연관된 엔티티도 디태치(세션에서 관리하지 않음) |
| MERGE        | 엔티티가 머지되면 연관된 엔티티도 머지                       |
| ALL          | 위 모든 타입을 포함                                          |

```java
@Entity
@Table(name="person")
public class Person {
  ...
  
  @OneToOne(cascade=CascadeType.ALL)		// Cascade 설정
  @JoinColumn(name="person_detail_id")
  private PersonDetail personDetail;
}
```

기본값으로 Cascade 연산은 비활성화 되어 있으며 CascadeType을 명시하여 어떤 연산이 Cascade 될지 설정할 수 있어요.

```java
@OneToOne(cascade={CascadeType.REFRESH
                  , CascadeType.PERSIST
  								, CascadeType.REMOVE})		// Multiple cascade types
@JoinColumn(name="person_detail_id")
private PersonDetail personDetail;
```

위와 같이 여러 CascadeType을 지정할 수 있어요.

<br>

### Fetch Type

#### Eager Loading

```java
@Entity
@Table(name="author")
public class Author {
  ...
  @OneToMany(fetch=FetchType.EAGER)
  private List<Post> posts;
}
```

해당 객체를 가져올 때 연관된 객체를 모두 가져오는 로딩이에요.

만약 해당 테이블이 수많은 연관 테이블을 조인하고 있다면 성능 이슈가 발생할 수 있어서 조심해야해요.

#### Lazy Loading

```java
@Entity
@Table(name="author")
public class Author {
  ...
  @OneToMany(fetch=FetchType.LAZY)
  private List<Post> posts;
}
```

해당 객체를 가져올 때가 아닌 연관 테이블에 접근할 때 해당 테이블을 가져오는 로딩이에요.

성능적으로 이득이 있으므로 기본적으로 Lazy Loading을 쓰는 것이 Best Practice라고 하네요.

레이지 로딩을 사용하기 위해서는 연관 엔티티를 가져오는 시점에 세션이 열려있어야 해요.

<br>

#### Default Fetch Types

| Mapping     | 기본값          |
| ----------- | --------------- |
| @OneToOne   | FetchType.EAGER |
| @OneToMany  | FetchType.LAZY  |
| @ManyToOne  | FetchType.EAGER |
| @ManyToMany | FetchType.LAZY  |

페치 타입을 별도로 명시하지 않는 경우 기본값으로 위와 같은 페치타입을 가지고 있어요.

---

## Reference

1. [Udemy Spring & Hibernate for Beginners](https://www.udemy.com/course/spring-hibernate-tutorial/)

