---

title: 테스트랑 조금 더 친해져보기 (Mock, Stub, Spy, TestPrincipal)
date: 2023-09-05
categories: [Mock, Stub, Spy, TestPrincipal]
tags: [Mock, Stub, Spy, TestPrincipal]
layout: post
toc: true
math: true
mermaid: true

---

[마틴 파울러 번역](https://sites.google.com/a/jabberstory.net/testing/mocksArentStubs)

---

# 테스트 대역

`Mock`은 테스트 대상(SUT)과 그 협력자 사이의 상호작용을 검사할 수 있는 테스트 대역이다.

테스트 대역의 또다른 유형으로는 `Stub`이 있다.

조금 더 자세하게 구성을 살펴보면 아래와 같다.

### 테스트 대역 구성

- Mock
  - Mock
  - Spy
- Stub
  - Stub
  - Dummy
  - Fake

---


# Mock vs Stub

JPA를 활용해서 user를 데이터베이스에 저장하는 로직을 Stub과 Mock을 활용해서 테스트를 작성해보겠다.

## Mock을 코드로 알아보기

```kotlin
@SpringBootTest
class UserServiceTest {

    @Autowired
    private lateinit var userService: UserService

    @MockBean
    private lateinit var userRepository: UserRepository

    @Test
    fun testSaveUser() {
        // 사용자 정보 생성
        val user = User(username = "johndoe", email = "johndoe@example.com")

        // userRepository.save 메서드를 호출할 때 목(Mock) 객체를 사용
        Mockito.`when`(userRepository.save(user)).thenReturn(user)

        // 사용자 저장
        val savedUser = userService.saveUser(user)

        // 저장된 사용자와 원래 사용자 정보가 같은지 검증
        Assertions.assertEquals(user, savedUser)

        // userRepository.save 메서드가 한 번 호출되었는지 검증
        Mockito.verify(userRepository, Mockito.times(1)).save(user)
    }
}
```

---

## Stub을 코드로 알아보기

```kotlin
@Repository
class UserRepositoryStub {

    private val users: MutableList<User> = mutableListOf()

    fun save(user: User): User {
        user.id = (users.size + 1).toLong() // 간단한 ID 생성 로직
        users.add(user)
        return user
    }

    fun findById(id: Long): User? {
        return users.find { it.id == id }
    }

    fun findAll(): List<User> {
        return users.toList()
    }
}

@SpringBootTest
class UserServiceTest {

    @Autowired
    private lateinit var userService: UserService

    @Autowired
    private lateinit var userRepositoryStub: UserRepositoryStub

    @Test
    fun testSaveUser() {
        // 사용자 정보 생성
        val user = User(username = "johndoe", email = "johndoe@example.com")

        // 사용자 저장
        val savedUser = userService.saveUser(user)

        // 저장된 사용자와 원래 사용자 정보가 같은지 검증
        Assertions.assertEquals(user, savedUser)

        // userRepositoryStub에서 사용자를 찾아와서 확인
        val retrievedUser = userRepositoryStub.findById(savedUser.id!!)
        Assertions.assertEquals(savedUser, retrievedUser)
    }
}
```

---


### Mock, Spy

- `Mock`은 테스트 대상이 상태를 변경하는 행위에 대한 의존성을 호출하는 것에 해당한다.
    - 이메일 발송에 대한 의존성 호출을 Mock으로 대체하여 실제 SMTP서버를 호출하지 않고 테스팅할 수 있도록 한다.

다시 정리하자면, Mock은 어떤 행위를 수행했을 때 기대하는 값을 제공하는 역할이다.

`Spy`는 `Mock`과 같은 역할을 한다. 차이점은 `Spy`는 수동으로 작성하는 반면, `Mock`은 `Mock프레임워크`의 도움을 받아 생성된다.

### Stub, Dummy, Fake

- `Stub`은 입력 데이터를 얻기 위한 의존성을 호출하는 것에 해당한다.
    - 데이터베이스에서 데이터를 검색하여 그 데이터를 바탕으로 테스트를 작성하는 것을 `Stub으로 대체`한다.

다시 정리하자면, Stub은 출력을 생성하도록 입력을 제공하는 역할이다.

---

# 단위 테스트 구성 방법

## AAA 패턴 사용

### 1. 준비 구절

테스트 대상(SUT)과 해당 의존성을 원하는 상태로 만든다.

---

### 2. 실행 구절

테스트 대상(SUT)에서 메서드를 호출하고 준비된 의존성을 전달한다.

---

### 3. 검증 구절

반환 값 혹은 SUT 와 협력하는 관계의 최종 상태 등을 검증한다.

## 주의해야할 점

### 테스트 내 if문 피하기

테스트 구절에 if문이 있다면 너무 많은 것을 검증한다는 지표이다.

따라서 이러한 테스트는 반드시 여러 테스트로 분리해야한다. (통합 테스트의 경우에도 마찬가지이다.)

---

### 준비 구절의 크기

준비 구절은 가장 클 수 밖에 없다. 하지만 너무 크다면 이를 분리하는 게 코드 재사용에 도움이 될 것이다.

이 때 사용되는 두 가지 패턴이 있는데

- 오브젝트 마더
- 테스트 데이터 빌더

위 두 패턴을 활용한다면 재사용성이 있는 테스트 코드를 작성할 수 있다.

---

### 실행 구절이 한 줄 이상인 경우는 지양하기

실행 구절은 한 줄로 구성되어야한다. 이상적인 코드와 그렇지 않은 코드를 살펴보자

이상적인 코드

```java
public void Purchase_succeeds_when_enough_inventory() {

    // Arrange
    var store = new Store();
    store.addInventory(Product.Shampoo, 10);
    var customer = new Customer();

    // Act
    bool success = customer.Purchase(store, Product.Shampoo, 5);

    // Assert
    Assert.True(success);
    Assert.Equal(5, store.getInventory(Product.Shampoo));
}
```

실행 구문이 두 줄로 작성된 코드

```java
public void Purchase_succeeds_when_enough_inventory() {

    // Arrange
    var store = new Store();
    store.addInventory(Product.Shampoo, 10);
    var customer = new Customer();

    // Act
    bool success = customer.Purchase(store, Product.Shampoo, 5);
    store.removeInventory(success, Product.Shampoo, 5);

    // Assert
    Assert.True(success);
    Assert.Equal(5, store.getInventory(Product.Shampoo));
}
```

위 구문을 어떻게 한 줄로 바꿀까? 라고 생각해본다면 테스트 코드 자체를 건드는 것 보다 애초에 객체지향적으로 해결할 수 있도록 해야한다.

즉, `customer.purchase()`구문을 실행하면 객체 협력으로 `store.removeInventory()`를 메세지 보낼 수 있도록 협력관계를 구축하면 두 줄 이상 실행 로직이 필요하지 않게 된다.

---

### 검증 구절의 검증문 크기

검증 구절이 딱 하나만 가져야한다는 지침은 좋지 않다.

단위 테스트의 단위는 동작의 단위일 뿐, 코드의 단위가 아니기 때문에 단일 동작에 대한 단위는 여러 결과를 낼 수 있어 그 모든 결과를 검증해야한다.

그렇다고 너무 커지면 안되기 때문에 어느정도는 선을 지켜야한다.

이를 위해선 적절한 추상화가 적용되어있어야한다.


---

### 테스트 간 테스트 픽스처 재사용

테스트 픽스처란 테스트 실행 대상 객체이다.

이 객체는 정규 의존성인 SUT로 전달되는 인수를 말한다.

DB의 데이터 등이 그 예시이며 이러한 객체는 각 테스트 실행 전에 알려진 고정된 상태를 유지하기 때문에 동일한 결과를 생성한다.

비공개 팩터리 메서드를 통해 공통 초기화 코드를 추출해낸다면 테스트 픽스처를 재사용하기 좋을 것이다.


---

### 단위 테스트 네이밍 규칙

네이밍은 특별한 규칙에 얽매이는 것 보단 주의점만 적용한다면 상관없다.

- 쉬운 영어로 작성하기
- 비개발자들에게 시나리오를 설명하듯이 테스트 이름을 네이밍하기
- 단어를 밑줄로 구분하기
