---

title: 테스트 대역과 테스트 피라미드
date: 2023-11-24
categories: [TestDouble, TestDouble]
tags: [TestDouble, TestDouble]
layout: post
toc: true
math: true
mermaid: true

---

# 테스트 대역이 왜 필요한가?

테스트하고자 하는 대상이 있을 때 이 로직이 다른 객체와 의존관계가 있을 때 의존관계의 로직 결함으로 인해 테스트가 실패할 수 있다.

따라서 실제 동작하는 것처럼 보이는 별개의 객체를 만드는 방식을 적용하는데 이 객체를 **테스트 더블** 이라고 한다.

테스트 대상을 `SUT(System Under Test)`, SUT가 의존하는 요소를 `DOC(Depended On Component)`라고 한다.

테스트 대역은 DOC와 동일한 기능을 제공한다.

---

# 테스트 대역 - Dummy

Dummy는 인스턴스화된 객체만 필요하고 기능까지는 필요하지 않을 때, 주로 파라미터를 전달하기 위해 사용된다.

```java
// 실제 객체
public interface Logger {
    void log();
}

// 테스트 대역 - Dummy
public class DummyLogger implements Logger {
    @Override
    void log() {}
}
```

---

# 테스트 대역 - Fake

Fake는 실제 동작하는 구현을 가지고 있지만, 실제 코드에 사용되지 않는 객체이다.

예를들어, 실제 데이터베이스에 접근해서 테이블을 조회한 값을 꺼내오는 동작이 있을 때, 이를 인-메모리 저장소를 활용해서 동작하도록 말 그대로 가짜 기능을 구현한 것이다.

```java
// 실제 DB에 접근하여 테이블을 조회하는 객체
public class UserRepository {

    public void findByUserId(Long userId) {
        String query = String.format(
                "SELECT * FROM USER WHERE USER.id = ?"
        );
        jdbcTemplate.select() ...

        return ...
    }
}

// 실제 DB가 아닌 인 메모리에서 해당 동작을 수행하는 객체
public class UserRepository {

    private final Map<Long, String> fakeUserDatabase = new HashMap<>();

    public void findByUserId(Long userId) {
        fakeUserDatabase.get(userId);

        return ...
    }
}
```

---

# 테스트 대역 - Stub

Stub은 미리 반환할 데이터가 정의되어 있고, 메서드를 호출했을 때 그 결과를 반환하는 역할만 수행한다.

```java
// 실제 랜덤 정수를 반환하는 메서드를 가진 객체
public class RandomRandomNumber {
    private static final int[] randomNumbers = {0, 1, 2, 3, 4, 5, 6, 7, 8, 9};

    public int generateNumber(int min, int max) {
        long seed = System.currentTimeMillis();
        Random random = new Random(seed);

        return ...
    }
}

// 미리 정해진 랜덤 정수를 반환하는 메서드를 가진 객체
public class StubRandomNumberGenerator {

    public int generateNumber(int min, int max) {
        return 123;
    }
}
```

---

# 테스트 대역 - Spy

`Spy`는 실제 객체를 부분적으로 Stubbing하고, 메서드 호출 여부, 메서드 호출 횟수 등의 정보를 기록하는 객체다.

---

# 테스트 대역 - Mock

`Mock`은 테스트 대상이 의존 객체의 어떤 메서드를 호출하는 것에 대한 기대를 명세할 수 있다.

```java
@Test
void createPost() {
    // ...

    // 의존 객체인 UserService 객체를 모킹하는 부분
    given(userService.findById(1L)).willReturn(new User(1L));

    Object actual = postCreator.execute(request);

    Assertions.assertAll(
            () -> assertThat(actual.getId()).isOne(),
            () -> assertThat(actual.getTitle()).isEqualTo(title),
            () -> assertThat(actual.getContent()).isEqualTo(content)
    );
}
```

---

# 각 테스트 대역 한 줄 요약

- `Dummy`는 인스턴스화된 객체만 필요하고 기능까지는 필요하지 않을 때, 주로 파라미터를 전달하기 위해 사용된다.
- `Fake`는 실제 동작하는 구현을 가지고 있지만, 실제 코드에 사용되지 않는 객체이다. 예를들어, 실제 데이터베이스에 접근해서 테이블을 조회한 값을 꺼내오는 동작이 있을 때, 이를 인-메모리 저장소를 활용해서 동작하도록 말 그대로 가짜 기능을 구현한 것이다.
- `Stub`은 미리 반환할 데이터가 정의되어 있고, 메서드를 호출했을 때 그 결과를 반환하는 역할만 수행한다.
- `Spy`는 실제 객체를 부분적으로 Stubbing하고, 메서드 호출 여부, 메서드 호출 횟수 등의 정보를 기록하는 객체다.
- `Mock`은 테스트 대상이 의존 객체의 어떤 메서드를 호출하는 것에 대한 기대를 명세할 수 있다.

---

# 테스트 피라미드가 왜 필요한가?

[마틴 파울러 - 테스트 피라미드에 관하여](https://martinfowler.com/bliki/TestPyramid.html)

![](https://martinfowler.com/bliki/images/testPyramid/test-pyramid.png)

과거부터 자동화 테스트는 대부분 UI를 활용하여 진행되어왔다. 이 방식의 장점은 테스트를 관리하기 쉽고, 만들어내기도 쉬웠다.

하지만 UI를 통한 테스트는 테스트를 진행하는 절대적인 시간이 오래 소요됐고 빌드 시간을 늘리는 문제가 있다.

이 문제점을 보완하기 위해 유닛 테스트의 자동화를 통해 피라미드 형태의 테스트를 갖추는게 이상적이다.

---

## 테스트 피라미드 - 유닛 테스트

어떤 테스트 대상의 비즈니스 로직을 검증하기 위해 작성되는 영역이다. 즉, 다른 게층이나 의존관계와 독립적으로 로직 자체를 검증하는 단계이다.

Java 진영에서는 `Junit5, Mock`을 활용하여 작성한다.

---

## 테스트 피라미드 - 통합 테스트

단위 테스트를 통해 로직이 정상적임을 확인했으면, 실제 비즈니스 로직 코드와의 이질감을 줄이기 위해 외부 라이브러리 등 과의 소통도 검증해야한다. (DB접근 등)

Java 진영에서는 `@SpirngBootTest`을 활용하여 작성한다.

---

## 테스트 피라미드 - 인수 테스트(UI 테스트, E2E 테스트)

실제 사용자의 요구사항에 대응하는 테스트를 작성하는 것을 말한다.

즉, 실제 API를 사용하는 시나리오에 맞추어 해당 시나리오를 검증하는 것을 말한다.

Java 진영에서는 `RestAssured`, `MockMvc`와 같은 도구를 활용한다.
