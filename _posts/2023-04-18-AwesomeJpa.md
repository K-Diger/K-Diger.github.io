---

title: JPA 애노테이션 해쳐모여

date: 2023-04-18
categories: [Spring, JPA]
tags: [JPA]
layout: post
toc: true
math: true
mermaid: true

---

0. XXXToOne
1. 상속 관계 매핑(@Inheritance, @DiscriminatorColumn, @DiscriminatorValue)
2. 중복 컬럼 매핑(@MappedSuperClass)
3. 임베디드 타입
4. CASCADE
5. orphanRemoval
6.

---

# 관계 매핑 (@XXXToOne)

- @OneToOne, @ManyToOne 에는 반드시 fetch = FetchType.LAZY 옵션을 추가해주자.

- OneToMany는 기본 값이 LAZY이므로 딱히 신경 안써도 된다.

---

# 상속 관계 매핑

## 주요 애노테이션

- @Inheritance(strategy=InheritanceType.XXX)
  - JOINED : 엔티티마다 별도의 테이블을 만들고 조인으로 관련 테이블을 연결.
  - SINGLE_TABLE (Default) : 하나의 테이블에 모든 엔티티를 저장
  - TABLE_PER_CLASS : 엔티티마다 별도의 테이블을 생성

- @DiscriminatorColumn(name="DTYPE")
- @DiscriminatorValue("XXX")

---

## @Inheritance 애노테이션을 활용한 매핑 전략

### JOIN 옵션

```java
@Entity
@Inheritance(strategy = InheritanceType.JOINED)
public class Vehicle {
    @Id
    private Long id;
    private String manufacturer;
    private String model;
    // ...
}

@Entity
public class Car extends Vehicle {
    private int numDoors;
    // ...
}

@Entity
public class Motorcycle extends Vehicle {
    private int numWheels;
    // ...
}
```

위와 같이 사용하면, 엔티티마다 별도의 테이블을 생성하며 Join을 활용하여 연결한다.

따라서 Vehicle 엔티티를 Select하면 Car, Motorcycle 테이블까지 조인하여 결과를 반환한다.

이때 부모 클래스의 PK를 FK겸 PK 로하는 테이블이 추가로 생성되는데

즉, DB에 존재하는 테이블은 Vehicle, Car, Motorcycle 이다.

### 단일 테이블 전략 (기본 전략)

```java
@Entity
public class Vehicle {
    @Id
    private Long id;
    private String manufacturer;
    private String model;
    // ...
}

@Entity
public class Car extends Vehicle {
    private int numDoors;
    // ...
}

@Entity
public class Motorcycle extends Vehicle {
    private int numWheels;
    // ...
}
```

이렇게 하면 Vehicle이라는 한 테이블만 사용할 뿐만 아니라 Car, Motorcycle에 있는 모든 데이터가 한 테이블로 몰린다.

즉, DB에 존재하는 테이블은 Vehicle 뿐이다.

## Join과 @DiscriminatorColumn 애노테이션을 활용한 조인 매핑 전략

```java
@Entity
@Inheritance(strategy = InheritanceType.JOINED)
@DiscriminatorColumn
public class Vehicle {
    @Id
    private Long id;
    private String manufacturer;
    private String model;
    // ...
}

@Entity
public class Car extends Vehicle {
    private int numDoors;
    // ...
}

@Entity
public class Motorcycle extends Vehicle {
    private int numWheels;
    // ...
}
```

엔티티를 위와 같이 작성한다면 실제 DB에는

Vehicle 테이블에 **DTYPE**이라는 내용으로 하위 클래스들의 내용이 담긴다. (위 예제에서는 Car, Motorcyle)

이 때 만약 DTYPE이라는 컬럼의 내용을 Car, MotorCycle이 아니라 C, M으로 변경하고 싶으면 해당 하위 클래스에

@DiscriminatorValue 애노테이션을 붙여 네이밍을 수정할 수 있다.

---

# 중복 컬럼 매핑 - @MappedSuperClass

```java
@Entity
@MappedSuperClass
public abstract class Vehicle {
    private String typeName;
    private String createdAt;
    private String updatedAt;
}

@Entity
public class Car extends Vehicle {
    private int numDoors;
}

@Entity
public class Motorcycle extends Vehicle {
    private int numWheels;
}
```

위와 같이 상속 관계 매핑으로 보일 수 있겠지만 딱히 상관은 없고,

상위 클래스에 포함된 필드들을 그대로 하위클래스에도 적용하는 것으로 실제 테이블에도 매핑한 내용이 들어가게 된다.

그리고 @MappedSuperClass가 붙은 클래스는 테이블과 매핑되지 않는다. 말 그대로 매핑을 위한 클래스일 뿐 JPA가 다루진 않는다.

---

# 임베디드 타입 - @Embeddable, @Embedded

임베디드 타입을 활용한다면 높은 응집도와 재사용성을 챙기는 객체지향적인 도메인 설계가 가능해진다.

일단 아래와 같은 엔티티가 있을 때 적용해보자

```java
@Entity
public class Member {

    @Id
    @GeneratedValue
    private Long id;

    private String username;

    // Period
    private String startDate;
    private String endDate;

    // Address
    private String city;
    private String street;
    private String zipcode;
}
```

- @Embeddable - 값 타입을 정의하는 곳에 표시한다.

- @Embedded - 값 타입을 사용하는 곳에 표시한다.

@Embeddable 이 붙은 엔티티는 반드시 기본 생성자(NoArgsConstrucotr) 가 있어야한다.

위 내용을 적용하자면 다음과 같다.

```java
@Entity
public class Member {

    @Id
    @GeneratedValue
    private Long id;

    private String username;

    @Embedded
    private Peroid peroid;

    @Embedded
    private Address address;
}
```

```java
@Embeddable
@NoArgsConstructor
public class Period {

    private LocaDateTime startDate;
    private LocaDateTime endDate;
}
```

```java
@Embeddable
@NoArgsConstructor
public class Address {

    private String city;
    private String street;
    private String zipcode;
}
```

---

# @Cascade, orphanRemoval

```java
@Entity
public class Member {

    @Id
    @GeneratedValue
    private Long id;

    private String username;

    @Embedded
    private Peroid peroid;

    @Embedded
    private Address address;

    @OneToMany(fetch = FetchType.LAZY, cascade = CascadeType.All)
    private Set<Post> posts;
}
```

### cascade = CascadeType.All (Persist, Removal, update, merge, detach etc ...)

부모 엔티티를 저장할 때 자식 엔티티도 함께 저장한다.

여기서 꼭 기억해야하는 점은 연관관계와 전~혀 무관하다.

그러면 언제 이 옵션을 써야할까?

예를 들면 이런 상황에 적절하다.

- 게시글이 있다.

- 그 게시글에 해당하는 첨부파일이 있다.

- 오직 그 게시글만 첨부파일 여러개를 관리한다.

이런 상황에서는 괜찮지만 만약에 다른 한 지점에서라도 첨부파일을 관리하면 절대 쓰면 안된다.

```java
public class CascadeTest {

    public static void main(String[] args) {
        Child child1 = new Child();
        Child child2 = new Child();
        Child child3 = new Child();

        Parent parent = new Parent();
        parent.addChild(child1);
        parent.addChild(child2);
        parent.addChild(child3);

        em.persist(parent);
        em.persist(child1);
        em.persist(child2);
        em.persist(child3);
    }
}
```

원래 같으면 이렇게 자식 엔티티까지 직접 persist해줘야하는데, cascade옵션을 활용한다면 이럴 필요가 없다.

```java
@Entity
public class Parent {

    @Id @GeneratedValue
    private Long id;

    @OneToMany(mappedBy = "parent", cascade = Cascade.ALL)
    private List<Child> children = new ArrayList<>();

    public void addChild(Child child) {
        children.add(child);
        child.setParaent(this);
    }
}
```

위와 같이 부모 엔티티에 Cascade옵션을 준다면 일일히 persist 해줄 필요가 없어진다.

### cascade = CascadeType.REMOVE

```java
@Entity
public class Parent {

    @Id @GeneratedValue
    private Long id;

    @OneToMany(mappedBy = "parent", cascade = Cascade.Removal)
    private List<Child> children = new ArrayList<>();

    public void addChild(Child child) {
        children.add(child);
        child.setParaent(this);
    }
}
```

위에서 살펴본 특징은 All 옵션 중, Persist옵션에 대한 증명이었다.

그러면 Removal 옵션을 살펴보면

```java

@Entity
public class Team {

    @Id @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String name;

    @OneToMany(
        mappedBy = "team",
        fetch = FetchType.LAZY,
        cascade = CascadeType.REMOVE
    )
    private List<Member> members = new ArrayList<>();
}


// JpaLearningTest.java
@DisplayName("CascadeType.REMOVE - 부모 엔티티(Team)을 삭제하는 경우")
@Test
void cascadeType_Remove_InCaseOfTeamRemoval() {
    // given
    Member member1 = new Member();
    Member member2 = new Member();

    Team team = new Team();

    team.addMember(member1);
    team.addMember(member2);

    teamRepository.save(team);

    // when
    teamRepository.delete(team);

    // then
    List<Team> teams = teamRepository.findAll();
    List<Member> members = memberRepository.findAll();

    assertThat(teams).hasSize(0);
    assertThat(members).hasSize(0);
}
```

위와 같은 테스트가 있다.

- 멤버는 2명이다.

- 두 멤버는 모두 한 팀에 속한다.

- 팀을 제거한다.

- 멤버와 팀의 갯수를 증명한다.

위 코드를 실행시키면 delete 쿼리가 총 3번 나가는 걸 확인할 수 있다.

```text
Hibernate:
    insert
    into
        team
        (id, name)
    values
        (null, ?)
Hibernate:
    insert
    into
        member
        (id, name, team_id)
    values
        (null, ?, ?)
Hibernate:
    insert
    into
        member
        (id, name, team_id)
    values
        (null, ?, ?)

Hibernate:
    delete
    from
        member
    where
        id=?
Hibernate:
    delete
    from
        member
    where
        id=?
Hibernate:
    delete
    from
        team
    where
        id=?
```

즉, Team(부모)가 삭제될 때 Member(자식)도 영속성 전이 옵션으로 인해 함께 삭제된다.

cascade의 REMOVE 옵션이 해결해준 것이다.

그러면 부모 엔티티에서 자식 엔티티를 제거하는 경우에는 어떻게 될까?

```java
// JpaLearningTest.java
@DisplayName("CascadeType.REMOVE - 부모 엔티티(Team)에서 자식 엔티티(Member)를 제거하는 경우")
@Test
void cascadeType_Remove_InCaseOfMemberRemovalFromTeam() {
    // given
    Member member1 = new Member();
    Member member2 = new Member();

    Team team = new Team();

    team.addMember(member1);
    team.addMember(member2);

    teamRepository.save(team);

    // when
    team.getMembers().remove(0);

    // then
    List<Team> teams = teamRepository.findAll();
    List<Member> members = memberRepository.findAll();

    assertThat(teams).hasSize(1);
    assertThat(members).hasSize(2);
}
```

- 멤버는 2명이다.

- 두 멤버는 모두 한 팀에 속한다.

- 팀에서 첫 번째 멤버를 제거한다.

- 멤버와 팀의 갯수를 증명한다.

위 코드를 실행시키면 다음과 같이 쿼리가 나간다.

```text
Hibernate:
    insert
    into
        team
        (id, name)
    values
        (null, ?)
Hibernate:
    insert
    into
        member
        (id, name, team_id)
    values
        (null, ?, ?)
Hibernate:
    insert
    into
        member
        (id, name, team_id)
    values
        (null, ?, ?)
```

REMOVE는 그 Remove 옵션을 붙인 대상(부모)이 삭제될 때만 자식 엔티티를 삭제할 뿐

그 안에 있는 엔티티를 삭제해봐야 아무런 작업을 수행하지 않는다.

### orphanRemoval

cascadeType = REMOVE와 똑같다. 따라서 주의 사항 또한 똑같다. 참조하는 곳이 하나여야만 사용해야한다.

### CascadeType.ALL + orphanRemoval = true

이렇게 사용하면 부모 엔티티를 통해서 자식의 생명주기를 관리할 수 있다.

즉, DAO가 따로 필요하지 않다는 것이다.

이 방식은 DDD의 Root 개념을 구현할 때 유용하다.
