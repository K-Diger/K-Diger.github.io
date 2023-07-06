---

title: JPA - 연관관계 매핑.
author: Diger
date: 2022-12-23
categories: [Java, Spring, JPA]
tags: [Java, Spring, JPA]
layout: post
toc: true
math: true
mermaid: true

---

# 단방향 연관관계 매핑 - 객체지향 패러다임과 맞지 않은 상황

```java
@Entity
public class Member {
    @Id @GeneratedValue
    private Long id;

    @Column(name = "USERNAME")
    private String name;

    @Column(name = "TEAM_ID")
    private Long teamId;
}

@Entity
public class Team {
    @Id @GeneratedValue
    private Long id;

    @Column
    private String name;
}
```

위 코드는 연관관계에 대한 내용을 객체로 이어붙인 것이 아닌, 값 객체를 통해 이어 붙여져 있다. 객체지향스럽지 않다고 볼 수 있는 코드이다.

그럼 어떻게 객체지향스럽게 연관관계를 매핑할 수 있는걸까?

---

# 단방향 연관관계 매핑 - 객체지향 패러다임에 부합할 수 있도록

```java
@Entity
public class Member {
    @Id @GeneratedValue
    private Long id;

    @Column(name = "USERNAME")
    private String name;

    @ManyToOne
    @JoinColumn(name = "TEAM_ID")
    private Team teamId;
}

@Entity
public class Team {
    @Id @GeneratedValue
    private Long id;

    @Column
    private String name;
}
```

위와 같이 코드를 작성하면 우리가 흔히 객체를 다루듯이 Team 내부에 존재할 Getter/Setter 등 도메인 메서드를 사용할 수 있게 된다.

한마디로 정리하면, 위와 달리 데이터 중심으로 모델링 한 것이 아니기 때문에 협력 관계를 구성할 수 있게 된 것이다.

단방향 연관관계는 사실 크게 어려운 개념도 아니고 사용하기에도 어렵지 않다.

---

# 양방향 연관관계, 양방향 연관관계의 주인

Member에서 Team을 사용하고,

Team에서 Member를 사용할 수 있도록 하기 위한 것이 양방향 연관관계이다.

DB는 사실상 양방향으로, 외래키를 통해서 양 테이블을 왔다갔다 할 수 있다. (구체적으로는, 딱히 방향이라는 개념은 없음)

```java
@Entity
public class Member {
    @Id @GeneratedValue
    private Long id;

    @Column(name = "USERNAME")
    private String name;

    @ManyToOne
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

### mappedBy가 꼭 필요한 속성인가???

#### ㅇㅇ 필요하다.

객체는 연관관계가 될 수 있는 상황이 2가지가 있다.

Member -> Team

Team -> Member

따라서 사실상 단방향 연관관계가 2개인 것으로 볼 수 있다. (이걸 억지로 양방향이라고 하는 것임)

#### 테이블은 그냥 양방향임(사실 방향 그런거 없고 외래키를 알면 양쪽을 다 알 수가 있음)

아니 그래서 mappedBy가 꼭 필요한 것인가를 이야기 해보자면

Member라는 객체안에 Team을 변경하고 싶은 상황이 있다고 해보자.

근데 Team이라는 객체에 Member를 List로 받고있으니..

Team 객체에 있는 리스트에서 해당 멤버를 제거할지?

Member 객체에 있는 Team을 변경을 할지?

뭐가 맞는지 모르겠다. 둘다 해버릴까?

> 여기서 하나의 규칙이 등장한다. 위와 같은 두 가지 고민이 생기는 점에 대해서
>
> 하나의 객체에서만 외래키를 관리(등록, 수정)하는 "주인" 이라는 역할을 부여하는 것이다.


이 때, 주인이 아닌 연관관계를 mappedBy를 사용하는 것이다.

또한 주인이 아닌 연관관계에서는 외래키를 관리할 수 없으므로, 읽기 작업만 가능하게 된다.

### 음.. 그럼 누구를 주인으로 해야할까? 둘 중 아무나 하면되나??

외래키가 있는 곳을 주인으로 하면된다. (@ManyToOne 어노테이션이 붙은, 테이블로 따지면 외래키가 존재하는 객체 (1:N 에서 N이 속하는 테이블))

---

# 양방향 연관관계 매핑 시 주의할 점

연관관계의 주인이 아닌 컬럼에서 set으로 외래키를 설정하지 말아야한다!

```java
Team team = new Team();
team.setName("TeamA");

Member member = new Member();
member.setName("member1");

// 문제가 발생하는 부분!!
team.getMembers().add(member);
```

이래버리면, 연관관계의 주인이 아닌 곳(앞으로 하인이라고 하겠다.)에서 연관관계에 대한 정보를 넣어주려고 하기 때문에

읽기 전용인 하인 측에서는 뭐 아무것도 할 수가 없다.

그래서 다음과 같이 변경해야한다.

```java
Team team = new Team();
team.setName("TeamA");

Member member = new Member();
member.setName("member1");

// 문제가 발생하는 부분!!
// team.getMembers().add(member);

// 문제가 발생하는 부분!! --> 해결하기
member.setTeam(team);
```

근데 사실 더 정답에 가까운 방법이 있다. 양 쪽에 값을 넣어주는 것이다.

객체지향적으로 설계하기 위해서는 이 방법이 조금 더 낫다는 것인데 왜그런지 살펴보자.

바로 영속성 컨텍스트의 1차 캐시에 들어있는 내용을 조회할 수도 있는 상황 때문이다.

조금 더 자세히 말하자면, 나는 분명히 member.setTeam(team); 라는 문구로 Member의 Team을 지정해줬는데

team.getMembers(); 를 실행시키면 1차 캐시에 들어있던 비어있는 members를 가져오게 될 수 있다는 것 때문이다.

물론 이러한 문제는 엔티티 매니저의 flush(), clear()를 실행시키면 해결할 순 있으나 영속성 컨텍스트를 다 비워버리는 것 자체가 좀 별로긴 하다.

이것보단 차라리 team.getMembers().add(member); 로 직접 동기화를 시켜주는 편이 더 낫다.

뭐 객체지향 적으로도 이게 더 어울리는 코드이긴하다.

### 테스트 코드를 짤 때에도 문제가 생김!!!

마찬가지로 영속성 컨텍스트로 인한 값 동기화 문제가 발생할 수 있으므로 양쪽에 셋팅하자.

## 연관관계 편의 메서드를 만들어 놓으면 양쪽에 값을 넣는 과정에서의 실수를 줄일 수 있음

```java
public void setTeam(Team team) {
    this.team = team;
    team.getMembers().add(team);
}
```

물론 Setter가 그리 좋은거는 아니지만 예시로 참고하자.

Setter를 자제하려면 그냥 다음과 같이 전용 메서드를 하나 만들면 되는 것이다.

```java
public void changeTeam(Team team) {
    this.team = team;
    team.getMembers().add(team);
}
```

# 근데 꼭 양방향을 해야하나? 애초에 팀이라는 테이블에 멤버목록을 굳이 넣지 않아도, 멤버 테이블에 팀 정보를 넣어놓으면 될텐데.. (상의해보자)

이처럼 굳이 양방향을 안해도 충분히 요구사항을 만족시킬 수 있을거같은데 실수나 버그의 여지가 많은 양방향을 하는 이유는?

### 양방향 매핑 시 무한루프에도 조심해야한다.

ToString 메서드를 만들 때에도, 서로가 가진 객체 참조를 호출하게된다.

Member 호출 : Team
Team 호출 : Member

그럼 Member -> Team -> Member -> ... 이런 무한 루프가 발생할 수 있다.


JSON 생성 라이브러리, Lombok 에서도 문제가 발생하는데 일단 중요한 점은

Entity를 Controller에서 절대 반환하지 말자.

왜냐하면, Entity가 바뀌면 API 스펙이 바뀌는 기이한 현상이 발생하게 된다.

따라서 꼭 DTO를 통해 반환하도록 하자. 그리고 이를 통해 JSON 생성 라이브러리의 무한루프 문제도 해결될 수 있다.

> 양방향 매핑은 최~~ 대한 하지말자. 단방향 매핑으로 설계를 끝내버리는게 최선이다.

## 그럼 최대한 자제해야하는 양방향 매핑은 언제 써야하는가?

역방향으로 탐색해야하는 경우가 있을 수 있긴하다. 그럴땐 불가피하게 쓰긴 해야하는데 웬만하면 단방향 연관관계를 잘 셋팅하면 해결된다.

---

