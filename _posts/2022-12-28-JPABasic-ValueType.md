---

title: JPA - 값 타입.
author: Diger
date: 2022-12-28
categories: [Java, Spring, JPA]
tags: [Java, Spring, JPA]
math: true
mermaid: true

---

# 엔티티 타입

@Entity 타입으로 정의하는 객체

데이터가 변해도 식별자로 추적이 가능하다.


# 값 타입

int, Integer 처럼 단순히 값으로 사용하는 객체

식별자가 없어 값이 변경해도 추적이 불가능하다.

## 값 타입 분류

### 기본 값 타입 (primitive)

int, double, char etc ...

생명 주기를 엔티티에 의존한다. (당연하게도 어떤 클래스에서 필드로 사용하는 것이기 때문이다.)

### 래퍼 타입 (wrapper)

Integer, Double, String etc ...

래퍼 타입은 값이 공유가 된다.

```java
Integer a = new Integer(10);
Integer b = a;
```

위와 같은 코드가 있으면 a의 값이 20, 30 등으로 변경된다 하면, b의 값 또한 변경된다. (wrapper타입은 해당 객체에 대한 참조값을 가지기 때문이다.)

따라서 참조값을 공유는 가능하지만, 변경은 되지 않는다.

a.setValue() --> 이런 식으로 변경이 안되니까!

## 임베디드 타입 (embedded)

내가 커스텀한 클래스를 값으로써 사용하는 것이다.

새로운 값 타입을 직접 정의해서 사용하는 것으로 JPA 이를 Embedded 타입이라고 한다.

회원 엔티티가 있다고 해보자.

```java
@Entity
public class Member {

    @Id
    @GeneratedValue
    private Long id;

    private String username;

    private LocalDateTime startDate;
    private LocalDateTime endDate;

    private String city;
    private String street;
    private String zipcode;
}
```

여기서, startDate, endDate를 캡슐화 하고 싶을 때 임베디드 타입에 관한 어노테이션이 사용된다.

```java
@Embedable //임베디드 값 타입 이라는 것을 알리는 어노테이션을 꼭 달아야한다.
public class Period {
    private LocalDateTime startDate;
    private LocalDateTime endDate;

    // 이 때 기본 생성자는 필수적이다.
    public Period() {

    }
}

@Embedable //임베디드 값 타입 이라는 것을 알리는 어노테이션을 꼭 달아야한다.
public class Address {
    private String city;
    private String street;
    private String zipcode;

    // 이 때 기본 생성자는 필수적이다.
    public Address() {

    }
}

@Entity
public class Member {

    @Id
    @GeneratedValue
    private Long id;

    private String username;

    @Embedded
    private Period period;

    @Embedded
    private Address address;
}
```

임베디드 타입은 잘 느껴지다싶이 재사용이 가능해진다. 또한 잘 설계되었다고 말하는 프로젝트에서는 엔티티보다 임베디드 타입을 위한 클래스가 더 많은게 대부분이다.

만약 하나의 임베디드 타입을 하나의 엔티티에서 여러개를 사용해야할 땐 어떻게 해야할까?

```java
@Entity
public class Member {

    @Id
    @GeneratedValue
    private Long id;

    private String username;

    @Embedded
    private Period period;

    @Embedded
    private Address homeAddress;

    @Embedded
    @AttributeOverrides(value = AttributeOverride(name = "city", column = @Column(EMP_START)))
    private Address workAddress;
}
```

### 컬렉션

List<Integer> 등


---

# 값 타입 비교

값 타입은 인스턴스가 달라도 그 내부의 값이 같으면 같은 것으로 취급한다.

```java
int a = 10;
int b = 10;

a == b --> true


Address a = new Address("서울");
Address b = new Address("서울");

a == b --> false
```

자바에서는 == 비교가 참조 값을 비교하는 것이다.

따라서 위 예제에서의 a, b 는 Predefined된 10이라는 같은 참조 값을 가리키기 때문에 같다고 나오지만

아래 예제에서의 a, b 는 각 다른 인스턴스 생성하여 그에 대한 참조값을 가리키기 때문에 다르다고 나온다.

## 동일성(identity 비교) vs 동등성(equivalence 비교)

- 동일성은 인스턴스의 참조 값을 비교한다. 이때 비교에 필요한 연산자는 == 을 사용한다.
- 동등성은 인스턴스의 값을 비교한다. equals() 메서드를 사용하며 재정의를 필요로 할 때도 있다.
  - 기본 equals()메서드는 == 비교이다. 따라서 동등성을 비교하고자 하지만 동일성 비교가 되기 때문에 이를 원한다면 오버라이드 해야한다.

