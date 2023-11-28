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

# 참고 자료

[마틴 파울러 - 테스트 대역](https://martinfowler.com/bliki/TestDouble.html)

[JesusValera](https://jesusvalerareales.com/the-importance-of-tests-in-our-software/)

---

# 테스트 대역이 왜 필요한가?

![](https://jesusvalerareales.com/images/2020-06-11/2.png)

테스트하고자 하는 대상이 있을 때 이 로직이 다른 객체와 의존관계가 있을 때 의존관계의 로직 결함으로 인해 테스트가 실패할 수 있다.

따라서 실제 동작하는 것처럼 보이는 별개의 객체를 만드는 방식을 적용하는데 이 객체를 **테스트 더블** 이라고 한다.

테스트 대상을 `SUT(System Under Test)`, SUT가 의존하는 요소를 `DOC(Depended On Component)`라고 한다.

테스트 대역은 DOC와 동일한 기능을 제공한다.

---

# 테스트 대역 - Dummy

더미는 SUT가 의존하는 객체이지만 테스트 시 사용되지 않는다. 테스트 범위와 관련이 없기 때문에 신경 쓰지 않아야하기 때문이다.

```java
class Service {
    public static final String OUTPUT = 'something';

    public String format(Dependency dependency) {
        return OUTPUT;
    }
}

class ServiceTest extends TestCase {
    public void testFormat() {
        String result = (new Service()).format(null);
        self.assertSame(Service.OUTPUT, result);
    }
}
```

---

# 테스트 대역 - Fake

Fake는 실제 동작하는 구현을 가지고 있지만, 실제 코드에 사용되지 않는 객체이다.

예를들어, 실제 데이터베이스에 접근해서 테이블을 조회한 값을 꺼내오는 동작이 있을 때, 이를 인-메모리 저장소를 활용해서 동작하도록 말 그대로 가짜 기능을 구현한 것이다.

```java
interface UserRepositoryInterface {
    User findByUserId(Long userId);
}

class RealUserRepository implements UserRepositoryInterface {
    @Override
    public User getUserById(Long userId) {
        return jdbcTemplate.query(userId) ... ;
    }
}

class FakeUserRepository implements UserRepositoryInterface {
    @Override
    public User getUserById(Long userId) {
        return new User(uuid, 'Jesus', "['ADMIN_ROLE']");
    }
}
```

---

# 테스트 대역 - Stub

Stub은 가짜 데이터를 반환하는 객체이다. 미리 반환할 데이터가 정의되어 있고, 메서드를 호출했을 때 그 결과를 반환하는 역할만 수행한다.

SUT의 의존대상으로부터 어떠한 리턴값이 필요한 경우 사용된다.

```java
class ServiceTest {
    public void testDoSomething() {
        String uuid = new Service().testTarget(new UserStubService());
        assertThat.isEqualTo("0000-000-000-00001", uuid);
    }
}

class UserRealService {
    public String getUuid(User user) {
        return user.getId();
    }
}

class UserStubService {
    public String getUuid(User user) {
        return '0000-000-000-00001';
    }
}
```

---

# 테스트 대역 - Spy

`Spy`는 실제 객체를 부분적으로 Stubbing하고, 메서드 호출 여부, 메서드 호출 횟수 등의 정보를 기록하는 객체다.

```java
interface LoggerInterface {
    void log(String message);
}

class LoggerSpy implements LoggerInterface {
    public Array<String> messages = new String[9999999];

    public void log(String message) {
        this.messages.add(message);
    }
}

class UserNotifier {

    private final LoggerInterface loggerInterface;

    public UserNotifier(LoggerInterface loggerInterface) {
        this.loggerInterface = loggerInterface;
    }

    public void registerUser(UserModelInterface user) {
        this.logger.log("Notifying the user: {user.name()}");
    }
}
```

---

# 테스트 대역 - Mock

`Mock`은 테스트 대상이 의존 객체의 어떤 메서드를 호출하는 것에 대한 기대를 명세할 수 있다.

```java
class ShoppingService {
    public Float calculateAmount(Lines lines) {
        Float amount = 0;

        /** 이 부분을 테스트 하기 어려워 목으로 대체한다. */
        List<Line> linesTransformed = this.getShoppingCart(lines);
        for (Line line : linesTransformed) {
            amount += line.price();
        }

        return amount;
    }

    protected List<Line> getShoppingCart(Lines lines) {
        return Collections.asList(lines);
    }
}


class LoggerTest extends TestCase {
    public void testMovieBudgetFactory() {
        MockShoppingService service = this.createMock(ShoppingService::class);
        service
            .method('getShoppingCart') // Overriding the method.
            .willReturn([100, 200, 300]);

        Lines stubLines = new Lines(null);
        Float totalAmount = service.calculateAmount(stubLines);

        self.assertEquals(600, totalAmount);
    }
}
```

---

# 각 테스트 대역 요약

- `Fake`는 실제 동작하는 구현을 가지고 있지만, 실제 코드에 사용되지 않는 객체이다. 예를들어, 실제 데이터베이스에 접근해서 테이블을 조회한 값을 꺼내오는 동작이 있을 때, 이를 인-메모리 저장소를 활용해서 동작하도록 말 그대로 가짜 기능을 구현한 것이다.

- `Dummy`는 인스턴스화된 객체만 필요하고 기능까지는 필요하지 않을 때, 주로 파라미터를 전달하기 위해 사용된다.
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

```java
public class ExampleTest {
    @Test
    @DisplayName("단위 테스트")
    void testExample() {

        // given

        // when

        // then
        assertThat(object.getXXX()).isEqualTo(xxx);
    }
}
```

---

## 테스트 피라미드 - 통합 테스트

통합 테스트는 단위테스트 보다 조금 더 큰 범주를 커버한다.

`HTTP 요청`, `데이터베이스 연결`, `캐시 작업` 및 `일부 애플리케이션 로드`(20%-40%)가 필요한 기타 작업과 같은 복잡한 작업에 중점을 둔다.

```java
@SpringBootTest(webEnvironment = WebEnvironment.RANDOM_PORT)
@ExtendWith(SpringExtension.class)
public class ExampleTest {
    @Test
    @DisplayName("통합 테스트")
    void testExample() {

    }
}
```

---

## 테스트 피라미드 - 인수 테스트(UI 테스트, E2E 테스트)

실제 사용자의 요구사항에 대응하는 테스트를 작성하는 것을 말한다.

즉, 실제 API를 사용하는 시나리오에 맞추어 해당 시나리오를 검증하는 것을 말한다.

Java 진영에서는 `RestAssured`, `MockMvc`와 같은 도구를 활용한다.

```java
@SpringBootTest
@ExtendWith(SpringExtension.class)
public class ExampleTest {

    @Test
    @DisplayName("인수 테스트")
    void testExample() {
        // then
        mockMvc.perform(
                        post("/example")
                                .header(HttpHeaders.AUTHORIZATION, jwt)
                                .contentType(MediaType.APPLICATION_JSON)
                                .content(objectMapper.writeValueAsString(exampleRequest)))
                .andExpect(status().isOk())
                .andExpect(jsonPath("$.code").value(200))
                .andDo(print());
    }
}
```
