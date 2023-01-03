---

title: JPA - 프록시.
author: Diger
date: 2022-12-24
categories: [Java, Spring, JPA]
tags: [Java, Spring, JPA]
math: true
mermaid: true

---

# JPA내에서, 진짜 객체 조회 vs 가짜 객체 조회

```java
em.find(); // 진짜 쿼리를 날리고, 그 정보에 대한 객체를 가져옴
em.getReference(); // 실제 쿼리는 안나감, 가짜 객체를 가져옴
```

## 가짜 객체라는게 무엇인가...

em.getReference()를 실행시키면, Hibernate는 데이터베이스의 조회를 미루는 가짜 엔티티를 조회하는데 이 가짜 엔티티의 구성은 다음과 같다.

```java
class Proxy {
    // 실제 객체를 가리키는 참조 값
    Entity target = null;

    // Getter 메서드
    getter();
}
```

이 프록시의 특징은 실제 클래스를 상속 받아서 만들어진다 따라서 위의 코드를 조금 더 정교하게 작성하면 다음과 같다.

```java
class Proxy extends Member {
    // 실제 객체를 가리키는 참조 값
    Entity target = null;

    // Getter 메서드
    getter();
}

class Member {
    long id;
    String name;

    getter();
}
```
따라서 실제 클래스와 겉 모양이 똑같아서 사용하는 입장에서는 진짜 객체인지 프록시 객체인지 구분하지않고 사용해도 되긴한다. (이론상 된다는 것 주의할 점이 있긴하다.)

# 프록시 객체는 실제 객체의 참조를 보관한다.

프록시 객체를 호출하면 프록시 객체는 실제 객체의 메서드를 호출하게 되어있다. (상속 관계 및 위임)

# 프록시가 만들어지고 사용되는 과정

```java
Member member = em.getReference(Member.class, "id1");
member.getName();
```

1. 현재 member는 프록시 객체이기 때문에 getName()메서드를 사용하기 위해선 실제 엔티티를 생성해야한다.
   2. **위/아래의 과정**은 프록시가 고유하게 지닌, **실제 객체를 가리키는 참조 값이 없을 때**의 상황을 말한다.
2. 따라서 **프록시 객체**가 **실제 엔티티의 동작을 사용**할 경우에는 **영속성 컨텍스트에 member를 초기화**하도록 한다.
3. 이렇게 하면, **영속성 컨텍스트**는 실제 DB에 들어가 해당 member를 꺼내온 후 실제 엔티티를 생성해준다.
4. 실제 **엔티티가 생겼으면**, 그 엔티티에 대한 **getName()메서드를 수행**한 후 값을 반환한다.

# 프록시의 특징

1. 프록시 객체는 처음 사용할 때 한 번만 초기화된다.
2. 프록시 객체를 초기화 할 때, 프록시 객체가 실제 엔티티로 바뀌는 것이 아니다. 실제 엔티티가 만들어짐으로써 프록시 객체가 참조할 수 있는 값이 생기는 것이다.
3. 프록시 객체는 원본 엔티티를 상속 받는다. (타입 체크 시 instance of 를 사용해야한다. == 으로 타입비교 절대 금지!!!!!!)
4. 영속성 컨텍스트에 이미 찾고자 하는 엔티티가 있으면 em.getReference()를 호출해도 실제 엔티티를 반환한다.
   5. JPA는 같은 트랜잭션(영속성 컨텍스트)내에서 같은 타입을 조회 시 반드시 True를 반환하도록 명세되어있기 때문이다.

Member findMember = em.find();

Member refMember = em.getReference();

System.out.println(refMember == findMember);

위 결과는 무엇이 나와야 하는가? --> True 이다.

find 메서드로 영속성 컨텍스트에 엔티티가 들어가있기 때문에 getReference()로 프록시 객체를 반환하는 것이 아닌, 영속성 컨텍스트의 객체를 반환하기 때문이다.

// --------

Member refMember = em.getReference();

Member findMember = em.find();

System.out.println(refMember == findMember);

위 결과는 무엇이 나와야 하는가? --> True 이다.

근데, findMember는 실제 엔티티를 가져오는데, 어떻게 프록시 객체를 가져오는 타입 비교가 True가 될 수 있을까?

프록시 객체를 한 번 가져오면, 그 다음 find를 해도 프록시 객체를 가져오도록 만들어져 있기 때문이다.

5. 영속성 컨텍스트의 동작 범위를 벗어난 준영속 상태일 때 프록시 초기화 요청 시 문제가 발생한다.

트랜잭션 주기 == 영속성 컨텍스트의 주기와 거의 비슷하다. 따라서 트랜잭션 문제가 조금 잘 못 된다면

Cloud not Initialize Proxy 라는 에러가 뜰 것인데 이 때 이 문제를 떠올리면 된다.


# 프록시 확인하기

## 프록시 인스턴스의 초기화 여부 확인
PersistenceUnitUtil.isLoaded(Object Entity);

## 프록시 클래스 확인 방법
entity.getClass.getName()

## 프록시 강제 초기화 (JPA 표준에는 강제 초기화가 명세되어있지 않다. (Hibernate의 추가기능이다.))

Hibernate.initialize(entity);

혹은

member.getName() 등으로 초기화한다.

---

# Member를 조회할 때, Team도 함께 조회해야하는가?

흠... Member의 정보만 필요한데 Team까지 끌고오는건 딱히 좋은건 아닌 것 같다.

# 해결책 1. 지연로딩

```java
@Entity
public class Member {
    @Id @GeneratedValue
    private Long id;

    @Column(name = "USERNAME")
    private String name;

    // 지연로딩 구문
    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "TEAM_ID")
    private Team teamId;
}

@Entity
public class Team {
    @Id @GeneratedValue
    private Long id;

    @Column
    private String name;

    @OneToMany(mappedBy = "team")
    private List<Member> members = new ArrayList<>();
}
```

위와 같이 지연로딩에 대한 어노테이션 속성을 넣어주면, JPA는 프록시 객체를 가져오게 된다.

즉, Team에 대한 실제 엔티티를 쿼리를 만들어서 조회하는 것이 아니라, 참조값이 비어있는 껍데기 객체인 프록시 객체를 가져오는 것이라는 말이다.

---

# 프록시와 아주 밀접한 관계가 있다. 지연 로딩 / 즉시 로딩!


## 지연 로딩

```java
@Entity
public class Member {
    @Id @GeneratedValue
    private Long id;

    @Column(name = "USERNAME")
    private String name;

    // 지연로딩 구문
    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "TEAM_ID")
    private Team teamId;
}

Member m = em.find(Member.class, member1.getId());
System.out.println(m.getTeam().getClass());

// 지연로딩 시 (프록시 객체를 가져왔을 때) 실질적으로 쿼리가 나가는 구문
m.getTeam().getName();
```

위와 같이 지연로딩을 설정한 후 Member가 가진 Team의 객체를 조회하면, 프록시 객체가 반환된다.

따라서 위와 같이 getTeam.getName()처럼 Team에 대한 직접적인 행위가 필요할 때 쿼리가 나간다. (프록시 객체가 초기화된다.)

## 즉시 로딩

비즈니스 로직을 수행할 때, Member.getTeam().getName()와 같이 Team과 Member를 함께 쓰는 구문이 많은 경우에는 지연로딩이 그리 좋진않다.

매번 쿼리를 두번씩 쏴야하기 때문이다.

이 상황에선 쿼리를 덜 쓰고 비즈니스 로직을 수행할 수 있도록 도와주는 옵션이 즉시로딩이다.

```java
@Entity
public class Member {
    @Id @GeneratedValue
    private Long id;

    @Column(name = "USERNAME")
    private String name;

    @ManyToOne(fetch = FetchType.EAGER)
    @JoinColumn(name = "TEAM_ID")
    private Team teamId;
}

Member m = em.find(Member.class, member1.getId());
System.out.println(m.getTeam().getClass());

```

즉시 로딩 옵션을 설정하면, em.find() 구문이 시작할 때부터 Member와 Team을 조인하여 가져오기 때문에 프록시 객체가 아닌 실제 객체를 가져온다.

> SUWIKI EvaluatePost <-> Lecture <-> ExamPost 관계에서도 이 내용을 적용해볼만 할듯?


## 즉시 로딩 웬만하면 쓰면 안됨!!

- 전혀 예상하지 못한 SQL이 나가게됨 (Join 이 섞이고 섞여서 엄청나게 복잡한 쿼리가 생김)
- 애초에 Join을 해서 가져오는거라 꽤나 많이 느림
- 즉시 로딩은 N + 1 문제를 발생시킨다.

## N + 1 이 무엇이고 어떻게 발생하는가?

```java
@Entity
public class Member {
    @Id @GeneratedValue
    private Long id;

    @Column(name = "USERNAME")
    private String name;

    @ManyToOne(fetch = FetchType.EAGER)
    @JoinColumn(name = "TEAM_ID")
    private Team teamId;
}

List<Member> members = em.createQuery("select m from Member m", Member.class).getResultList();

```

위와 같은 JPQL 쿼리를 쏘는 상황에서 N + 1 문제가 발생하게 된다.

즉시 로딩은 위에서 언급했듯이, 엔티티 객체에 접근 즉시 연관된 테이블까지 조회하여 객체를 완전 초기화 한다고 했다.

그렇기 때문에 멤버가 100명이고 모두 속한 팀이 다르면.. 그 멤버들이 속한 100개의 팀을 모두 조회하기 위해 100방의 쿼리가 나간다.

쿼리가 나가는 횟수를 줄이려고 즉시로딩을 썼는데 더 나가게 된다.

그리고 이 현상을 N + 1 문제라고 한다.

여기서 N 은 연관관계 때문에 추가로 발생한 쿼리, 1 은 내가 실행하고자 하는 쿼리이다.


## N+1 문제는 지연 로딩으로 어느정도 해결은 된다! (완전히 해결되진 않음)

1. 페치 조인
```java
List<Member> members = em.createQuery("select m from Member m join fetch m.team", Member.class).getResultList();
```
간단하게 페치 조인의 예시를 보면 위와 같다.

페치조인은 SQL에서 지원하는 문법이 아니며 JPQL에서 지원하는 문법임을 알아야한다. 또한 일반 JOIN문이랑 도대체 무슨 차이가 있는가? 함은 다음과 같다.

## 일반적인 조인

select * from member join member.team 을 했을 때는 member의 관한 테이블만 조회한다. 그렇기 때문에 member가 가진 연관관계에 대한 정보를 알려면 그에 대한 쿼리를 또 날려야한다.

이 추가적인 쿼리를 날리면서 N + 1 문제가 발생하게 되는 것이다.

select * from member join fetch member.team 을 했을 때는 member와 각 member 가 가진 team 에 대한 정보도 한번에 가져온다. 이렇게 되면 각 멤버의 Team을 따로 조회하지 않고 영속화 시킬 수 있기 때문에

추가적인 쿼리가 발생하지 않게 된다.

2. 엔티티 그래프 (어노테이션)
3. 배치 사이즈


## @ManyToOne, @OneToOne은 기본 값이 즉시 로딩이다.

위 두 개의 어노테이션은 기본 속성이 즉시 로딩이므로 LAZY 옵션으로 변경하는 것을 꼭 잊지말자.

@OneToMany 혹은, @ManyToMany는 기본이 LAZY 로딩이므로 딱히 안해줘도 되지만... 그냥 습관적으로 해두는게 좋을 것 같다.


---

# 페치조인 더 자세히 보기

```java
import java.util.List;

import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.transaction.annotation.Transactional;

import javax.persistence.EntityManager;

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

        // 이 구문이 없어서 엄청 고생했다.
        // save() 메서드 내에 persist 구문이 있는데 이를 수행하게되면 영속성 컨텍스트에 들어가게 되서
        // FetchJoin이든, Join이든 차이가 없게 된다.
        manager.clear();
    }

    @Test
    void readMemberWithFetchJoin() {
        System.out.println("1----------------------------");
        List<Member> members = memberRepository.readMemberWithFetchJoin();
        for (Member member : members) {
            System.out.println("member = " + member.getTeam().getName());
        }
        System.out.println("1----------------------------");
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
```

### 출력 결과

![img.png](https://github.com/K-Diger/K-Diger.github.io/blob/main/images/jpg?raw=true)

![img.png](https://github.com/K-Diger/K-Diger.github.io/blob/main/images/jpg?raw=true)

---

## 페치 조인과 N + 1

일단 페치 조인은 양방향 연관관계에서만 사용 가능한가에 대한 궁금증이 있었지만, 위의 코드를 양방향으로 매핑하지 않았음에도 페치 조인 쿼리가 나간 것을 보면

그건 아닌 것으로 알 수 있다.

페치조인과 관련된 주제로는 N + 1이라는 문제가 있다.

N + 1이라는 문제는 본질적으로 로딩 방식에 따라 발생하는 문제가 아니다. EAGER 로딩을 사용하여도 N + 1 문제는 발생하기 마련이다.

프록시라는 개념이 본질적인 N + 1의 문제가 된다. A, B, C 라는 객체가 있다고 하자.

지연 로딩은 조회하고자 하는 객체인 A를 제외한, 연관관계로 엮여있는 B, C들은 프록시 객체로 가져온다.

그래서 A라는 엔티티를 조회하고 그 객체에서 B, C에 접근하게 되면 B, C의 실제 객체를 가져와야하기 때문에 쿼리가 2방 더 나가게 된다.

즉시 로딩은 A객체를 조회할 때도 B, C를 프록시가 아닌 진짜 객체를 가져오기 때문에 A 객체에 접근하여 B, C에 접근해도 추가적인 쿼리가 나가지 않는다.

하지만, 즉시 로딩은 A만 사용하고 싶을때에도 B, C를 조인으로 모두 불러온다.

이것이 즉시 로딩의 큰 문제점 중 하나이다.

그렇기 때문에 무조건적으로 모든 객체를 불러오는 것이 아닌 선택적으로 불러올 수 있도록 지연 로딩을 사용하라는 것이다.

지연 로딩은 앞서 언급했듯이 프록시 객체를 불러오는 것에 의하여 추가적인 쿼리(N + 1)가 나가기 때문에 이를 보완할 방법이 필요한데 이 방법 중 하나가 페치 조인인 것이다.

> N + 1 문제는 로딩 방식으로 인한 발생한 본질적인 문제가 아니다. 프록시 조회냐 실제 엔티티 조회냐에 따라 영속성 컨텍스트와 연관되어 있는 문제인 것이다.

## 페치 조인과 일반 조인

