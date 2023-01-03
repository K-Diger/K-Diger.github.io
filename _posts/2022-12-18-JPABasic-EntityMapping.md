---

title: JPA - 엔티티 매핑
author: Diger
date: 2022-12-18
categories: [Java, Spring, JPA]
tags: [Java, Spring, JPA]
math: true
mermaid: true

---

# 0. 들어가기 전, JPA의 DB 스키마 자동 생성

JPA는 DDL을 애플리케이션 실행 시점에 자동 생성해준다.

또한 DBMS마다 조금씩 다른문법을 활용하여 DBMS에 적잘한 DDL을 생성한다.

이렇게 생성된 DDL은 개발단계에서만 사용해야한다.

---

# 1. 객체와 RDB를 어떻게 매핑해야하는가??

우선 객체와 테이블과 매핑해야 하는 요소를 정리하자면 다음과 같다.

| App  | Annotation              | DB |
|------|-------------------------|---|
| 객체   | @Entity, @Table         | 테이블 |
| 필드   | @Column                 | 컬럼 |
| 기본 키 | @Id                     | 기본 키 |
| 연관관계 | @ManyToOne, @JoinColumn | 연관관계 |

# 2. 객체와 테이블을 매핑하려면?

@Entity 어노테이션이 붙은 클래스는 JPA가 관리한다. 이를 엔티티라고 한다.

여기서 주의해야할 점이 있는데

> 기본 생성자가 필수적으로 요구된다. (파라미터가 없는 public, protected --> @NoArgsArgument를 무의식중에 달았던 이유이다.)
> JPA 스펙상 요구하는 것이다. (JPA의 구현체에서 이에 대한 리플렉션, 프록싱 등... 을 사용한다고 한다.)
>
> final 클래스, enum, interface, inner 클래스에는 사용하지 않아야한다.
>
> 저장할 필드에 final 키워드를 붙이면 안된다.

---

# 3. 필드와 컬럼을 매핑하려면?

일반적인 자료형의 필드는 다음과 같이 매핑하면 된다. (여기서 name은 생략해도됨)
> @Column(name = "name)

Enum 타입의 필드는 다음과 같이 매핑하면 된다.
> @Enumerated(EnumType.STRING)

이 때 EnumType.ORDINAL, EnumType.STRING 이 있는데, 이는 각각 Enum 순서와, Enum 이름을 데이터베이스에 저장하는 것이다.

또한 ORDINAL이 기본값이기 때문에 되도록이면 EnumType.STRING으로 매핑된 내용을 인지할 수 있게 하는 것이 좋다.

날짜/시간 같은 필드는 다음과 같이 매핑하면 된다.
> @Temporal(TemporalType.TIMESTAMP)

이미지파일 등과 같이 큰 컨텐츠를 담는 타임은 다음과 같이 매핑한다.
> @Lob

JPA에 매핑하고 싶지 않은 컬럼은 다음과 같은 어노테이션을 붙여준다.
> @Transient

---

## 3.1. @Column 더 자세히 알아보기

@Column 어노테이션은 여러가지 속성 정보를 담을 수 있다.

### 속성 1. name

필드와 매핑할 테이블의 컬럼 이름을 나타낸다. 기본값은 객체의 필드 이름으로 하되 카멜케이스로 작성한 경우, _로 매핑된다.

### 속성 2. insertable, updatable

등록, 변경 가능 여부에 대한 속성이다. 기본값은 두 속성 모두 TRUE이다.

### 속성 3. nullable (DDL)

null 값의 허용 여부를 설정한다. false로 설정하면 DDL 생성 시에 not null 제약조건이 붙는다.

### 속성 4. unique (DDL)

@Table의 uniqueConstraints와 같지만 한 컬럼에 간단히 유니크 제약조건을 걸 때 사용한다.

### 속성 5. columnDefinition

데이터베이스의 컬럼 정보를 직접 줄 수 있다. (varchar(100), int, etc...)

### 속성 6. length (DDL)

문자 길이 제약조건을 건다. String 타입에만 사용 가능하며 기본값은 255이다.

### 속성 7. precision, scale (DDL)

BigDecimal 타입에서 사용한다.(BigInteger도 사용 가능)

precision은 소수점을 포함한 전체 자릿수

scale은 소수의 자리수를 나타낸다.

또한 이 옵션은 double, float 타입에는 적용되지 않는다.

---

# 4. 기본 키 매핑

## 4.1. 매핑 어노테이션
- @Id
- @GeneratedValue

### 기본 키를 직접 할당할 경우

@Id 만 사용한다.

### 기본 키를 자동 할당할 경우

@Id, @GeneratedValue 를 사용한다.

여기서 @GeneratedValue는 여러 옵션을 추가할 수 있다.

### IDENTITY

자동할당에 대한 동작을 데이터베이스에 위임한다.(MySQL auto_increment 일 때 사용하자)

또한 아래와 같은 특징이 있다는 것읏 알아둬보자.

> AUTO_INCREMENT는 데이터베이스에 INSERT Query를 날린 후에야 ID를 알 수 있다.
>
> 그렇기 때문에 이 전략은, em.persist()로 유저 생성 후 즉시 INSERT를 한 후에야 user.getId() 와 같은 문구로 ID를 다룰 수 있게 된다.

### SEQUENCE

데이터베이스 시퀀스 오브젝트를 사용하여 자동할당한다.(ORACLE 환경에서 사용하자)

또한, @SequenceGenerator 가 필요하다.

### TABLE

키 생성용 테이블을 사용한다.(모든 DBMS에서 사용가능하다.) 또한, @TableGenerator가 필요하다.

성능은 좀 아쉬운 방법이다.

### AUTO

각 DBMS 문법에 맞게 자동지정된다.이 옵션은 기본값이다.

