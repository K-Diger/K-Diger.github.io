---

title: Spring Data JPA 구현체 따라가기
author: Diger
date: 2024-09-19
categories: [Java, Spring, Spring Data JPA]
tags: [Java, Spring, Spring Data JPA]
layout: post
toc: true
math: true
mermaid: true

---

# JpaRepository

```java
public interface MemberRepository extends JpaRepository<Member, Long> {

}
```

위와 같이 Spring Data JPA를 사용하려면 JpaRepository를 상속받아야 한다.

그럼 JpaRepository라는 것은 어떻게 생겼을까?

```java

package org.springframework.data.jpa.repository;

@NoRepositoryBean
public interface JpaRepository<T, ID> extends PagingAndSortingRepository<T, ID>, QueryByExampleExecutor<T> {

    findAll();

    findById();

    ...
}
```

JpaRepository는 data.jpa.repository 패키지 내에 존재하고, findAll, findById 등 엄청나게 많은 메서드들을 명세하고 있다.

data jpa 는 기본적으로, spring data 프로젝트의 일부로,

spring data redis - spring data mongo 등 여러 데이터 접근 기술의 CRUD를 제공하는 프로젝트이다.

또한 PagingAndSortingRepository<T, ID>, QueryByExampleExecutor<T> 를 상속받는데 이 내용은 다음과 같다.

```java
package org.springframework.data.repository;

@NoRepositoryBean
public interface PagingAndSortingRepository<T, ID> extends CrudRepository<T, ID> {

    Iterable<T> findAll(Sort sort);

    Page<T> findAll(Pageable pageable);
}
```

위와 같이 org.springframework.data.repository의 PagingAndSortingRepository는 Iterable타입 Page타입 등

기본적인 정렬과 페이징을 지원하는 내용을 담고 있다.

근데 여기서 CrudRepository를 상속받는데 이 내용도 살펴보면 다음과 같다.

```java
package org.springframework.data.repository;

@NoRepositoryBean
public interface CrudRepository<T, ID> extends Repository<T, ID> {

    save();

    saveAll();

    ...
}
```

위와 같이 말 그대로 CRUD를 지원하는 메서드를 정의하는 인터페이스다.

Crud 레포지토리에서 상속받는 Repository는 Spring이 Bean Scan을 하기 쉽도록 하기 위한 Maker Interface (기능은 없는 인터페이스)이다.

---

# Spring Data JPA 공통 인터페이스 구성

![img.png](https://github.com/K-Diger/K-Diger.github.io/blob/main/images/SpringDataJPA-1.png?raw=true)

---

# Spring Data JPA 분석

```java

@Repository
@Transactional(readOnly = true)
public class SimpleJpaRepository<T, ID> implements JpaRepositoryImplementation<T, ID> {

    private static final String ID_MUST_NOT_BE_NULL = "The given id must not be null!";

    private final JpaEntityInformation<T, ?> entityInformation;
    private final EntityManager em;
    private final PersistenceProvider provider;

    private @Nullable CrudMethodMetadata metadata;
    private EscapeCharacter escapeCharacter = EscapeCharacter.DEFAULT;

    // ------------------- 생략 -------------------//

    @Override
    public Optional<T> findById(ID id) {

        Assert.notNull(id, ID_MUST_NOT_BE_NULL);

        Class<T> domainType = getDomainClass();

        if (metadata == null) {
            return Optional.ofNullable(em.find(domainType, id));
        }

        LockModeType type = metadata.getLockModeType();

        Map<String, Object> hints = new HashMap<>();
        getQueryHints().withFetchGraphs(em).forEach(hints::put);

        return Optional.ofNullable(type == null ? em.find(domainType, id, hints) : em.find(domainType, id, type, hints));
    }
}
```

위와 같이 구현체를 보면 그냥 EntityManager를 사용하는 것 뿐이다.

일단 중요한 점은 @Repository와 @Transactional(readOnly = true) 구문이다.

### @Repository의 기능

1. 컴포넌트 스캔
2. 하부 기술의 예외에 대한 처리

2번기능 같은 경우는 다음과 같은 상황을 이야기 하는 것이다.

JDBC Exception이 발생했을 때, 이를 Spring이 다룰 수 있는 예외로 변환할 수 있는 기능을 제공한다는 것이다.

Data Access 에 관한 하부기술을 JPA에서 JDBC로 변경하여도 @Repository 어노테이션을 붙이면 그 예외를 처리하기 위한

부가적인 작업을 하지 않아도 된다는 말이다. (하부 구현 기술을 바꿔도 기존 비즈니스 로직에 문제가 없게된다!)

### @Transactional

Spring Data JPA 메서드를 사용한다면 @Transactional을 따로 붙여주지 않아도 트랜잭션을 걸고 시작하게 된다.

조회가 많아서 readOnly 옵션을 true로 해주었지만

저장같은 메서드는 readOnly 옵션을 제거한 @Transactional 을 따로 붙여주고 있다.

또한 readOnly를 true로 해주면 flush() 메서드가 호출되지 않기 때문에 필요할 때만 flush를 호출하도록 하면

약간의 성능상 이점을 가져갈 수 있다.

## 여기서 궁금증

@Transactional이랑 영속성 컨텍스트랑 생명주기가 같은데.

save()메서드를 호출하고 다른 작업을 해봤을 대 (저번 주 지원이형이랑 했던 테스트코드) 분명 영속성 컨텍스트에 save한 엔티티가 들어있어서

추가적인 쿼리가 나가지 않았다. 이게 어떻게 된 것일까??

```java

@SpringBootTest
@Transactional
class MemberRepositoryTest {

    @Autowired
    MemberRepository memberRepository;

    @Autowired
    TeamRepository teamRepository;

    @Autowired
    EntityManager manager;

    @BeforeEach
    void init() {
        Team teamJohn = new Team("John");
        Team teamDiger = new Team("Diger");
        Member Diger = new Member("Diger Kim");
        Member John = new Member("John Jeong");

        teamRepository.save(teamJohn);
        teamRepository.save(teamDiger);

        Diger.setTeam(teamDiger);
        John.setTeam(teamJohn);

        memberRepository.save(Diger);
        memberRepository.save(John);
    }

    @Test
    void readMemberWithJoin() {
        System.out.println("2----------------------------");
        List<Member> members = memberRepository.readMemberWithJoin();
        for (Member member : members) {
            System.out.println("member = " + member.getTeam().getName());
        }
        System.out.println("2----------------------------");
    }
}

@Repository
@RequiredArgsConstructor
@Transactional
public class MemberRepository {
    void save(Member member) {
        System.out.println("member Id = " + member.getName());
        em.persist(member);
    }
}
```

---

## Spring Data JPA가 제공하는 save()의 중요한 점

이미 존재하는 엔티티가 아닐 경우에는 persist()

이미 존재하는 엔티티일 경우에는 merge() 메서드를 호출한다.


```java
@Repository
@Transactional(readOnly = true)
public class SimpleJpaRepository<T, ID> implements JpaRepositoryImplementation<T, ID> {
    @Transactional
    @Override
    public <S extends T> S save(S entity) {

        Assert.notNull(entity, "Entity must not be null.");

        if (entityInformation.isNew(entity)) {
            // 새로운 엔티티면 저장
            em.persist(entity);
            return entity;
        } else {
            // 새로운 엔티티가 아니면 병합 (현재 들어온 엔티티를 DB에서 가져온 데이터에 덮어쓴다.)
            // Merge의 단점은 DB의 Select 쿼리가 나가게 된다는 것이다.
            // 따라서 되도록이면 save()메서드(merge())를 통해서 이미 존재하는 엔티티의 값을 수정하고 저장하면 안된다.
            // 그럴땐 Dirty Checking 기능을 활용하도록하자.
            return em.merge(entity);
        }
    }
}
```

# 새로운 엔티티를 구별하는 방법

## 객체 식별 기본 전략

- 식별자가 객체일 때 Null 검증, Null 일땐 새로운 엔티티로 판단한다.

- 식별자가 기본 타입일 때 0이면 새로운 엔티티로 판단한다.


이 전략을 알아둬야하는 이유가 있다.

```java
@Entity
public class Item {

    @Id @GenereatedValue
    private Long id;
}
```

우리는 보통 위와 같이 객체 식별자를 등록하는데, 그렇지 않은 경우에 주의해야할 점이 생기기 때문이다.

만약 다음과 같이 엔티티를 설계했을 때, isNew()메서드에서는 어떻게 인식을 할까?

```java
@Entity
public class Item {

    @Id
    private String uuid;
}

class ItemRepositoryTest {

    @Autowired ItemRepository itemRepository;

    @Test
    public void save() {
        Item item = new Item("A");
        itemRepository.save(item);
    }
}
```

isNew()메서드는 새로운 객체임을 인식하지 못한다. (Null이 아닌 값이 들어있기 때문이다.)

따라서 merge를 하게되는 일이 발생하게 된다.

Merge는 엔티티가 영속성 컨텍스트 생명주기에서 Detached 되었다가 다시 Attached 되는 상황이 아니라면 굳이 쓸 일도 이유도 없는 기능이다.


그럼 위와 같은 상황은 어떻게 해결해야하는가??

## Persistable 구현

```java
@Entity
public class Item implements Persistable<String> {

    @Id
    private String uuid;

    @Override
    public String getUuid() {
        return uuid;
    }

    @Override
    public boolean isNew() {
        // 새로운 객체인지 판별하는 로직을 작성해야한다.
        return false;
    }
}
```

## Persistable 구현 및 JPA Auditing 활용

```java
@Entity
@EntityListeners(AuditingEntityListener.class)
public class Item implements Persistable<String> {

    @Id
    private String uuid;

    @CreatedDate
    private LocalDateTime createdDate;

    @Override
    public String getUuid() {
        return uuid;
    }

    @Override
    public boolean isNew() {
        // 새로운 객체인지 판별하는 로직을 작성
        return createdDate == null;
    }
}
```
