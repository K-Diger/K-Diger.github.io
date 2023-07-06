---

title: 만들면서 배우는 클린 아키텍처 CHAPTER 3. 코드 구성하기

date: 2022-11-04
categories: [Architecture]
tags: [Book]
layout: post
toc: true
math: true
mermaid: true

---

# CHAPTER 03. 코드 구성하기

---

# 가볍게 배경지식 훑어보기

### 육각형 아키텍쳐

![](https://thebook.io/007035/ch02/01/02/02/)
[그림출처]()

### 육각형 아키텍쳐 용어

> 인바운드 어댑터/포트 : 애플리케이션에 표현 계층 대신 비즈니스 로직을 호출하여 외부에서 들어온 요청을 처리하는 인바운드 어댑터로, **Controller** 라고 생각하면 편하다
>
> 아웃바운드 어댑터/포트 : 영속화 계층 대신 비즈니스 로직에 의해 호출되고 외부 애플리케이션을 호출하는 아웃바운드 어댑터로 **Repository** 라고 생각하면 편하다.


### 전통적인 계층형 아키텍처

사용자와의 상호작용을 담당하는 Presentation Layer

엔티티의 영속성을 처리하는 Persistence Layer 로 구분된다.

Presentation Layer은 하위의 도메인 계층에 의존하고, 도미인 계층은 하위의 Persistence Layer 에 의존한다.

---

# 계층 구성

    buckpal
      -- domain
          -- Account
          -- Activity
          -- AccountRepository
          -- AccountService
      -- persistence
          -- AccountRepositoryImpl
      -- web
          -- AccountController

웹 계층, 도메인 계층, 영속성 계층 각각에 대한 전용 패키지인 web, domain, persistence를 뒀다.

의존성 역전 원칙(DIP)을 적용하여 의존성이 domain 패키지에 있는 도메인 코드만을 향한다.

즉, domain 패키지에 AccountRepository 인터페이스를 생성하고

persistence 패키지에 AccountRepositoryImpl 구현체를 작성하여 의존성을 역전시킨 것이다.

## 현재의 계층 구성이 Best 가 아닌 이유 3가지

### 1. 애플리케이션 기능 조각, 특징을 구분짓는 패키지 경계가 없다.

이 구조에서 User 에 관한 기능을 추가해야한다면

web 패키지에 UserController 를 생성하고, domain 패키지에 UserService, UserRepository, User 를 추가하고

persistence 패키지에 UserRepositoryImpl 을 추가해야할 것이다.

지금처럼 한 개가 요구되는 상황에서는 나쁘지 않을 수 있어보이지만

만약 요구사항이 10개만 늘어나는 것만 상상해봐도 벌써 머리가 뜨거워 질 것이다.

### 2. 애플리케이션이 어떤 유스케이스들을 제공하는지 파악할 수 없다.

AccountService 와 AccountController 가 어떤 유스케이스를 구현했는지 파악할 수 있는 구조일까?

실제로 AccountService 에서 계정의 결제 서비스를 담당하고, 계정에게 메일을 보내는 서비스가 만들어져있다고 할때

AccountService 이 클래스 이름만 봐서는 구체적으로 어떤 역할을 하는지 알 수 없다.

즉, 특정 기능을 찾기 위해서는 어떤 서비스가 이를 구현했는지 추측해야하고 해당 서비스에서 어떤 메서드가 그 책임을 맡는지 찾아야한다.

> 추측? 딱 봐도 번거롭다.


### 3. 전체적인 아키텍처를 파악할 수 없다.

어떤 기능이 웹 어댑터에서 호출되는지, 영속성 어댑터가 도메인 계층에 어떤 기능을 제공하는지 알아볼 수 없다.

주문 요청을 처리하는 web 어댑터에 대한 내용을 보기위해 web 패키지를 조사할 순 있다.

하지만 어떤 기능이 웹 어댑터에서 호출되는지, 영속성 계층과는 어떻게 상호작용을 하는지 파악할 수 없다.

---

# 계층 구성 (계층 구성을 조금 보완해보기)

    buckpal
      -- account
          -- Account
          -- AccountController
          -- AccountRepository
          -- AccountRepositoryImpl
          -- SendMoneyService


가장 본질적인 변경은 계좌와 관련된 모든 코드를 최상위 account 패키지에 넣었다는 것이다.

이렇게 구성함으로써 외부에서 접근하면 안되는 클래스에 대해 package-private 접근 수준을 이용하여 패키지 간 경계를 강화할 수 있다.

또한 기존 AccountService 클래스명을 SendMoneyService 로 변경하여 "송금하기" 에 대한 유스케이스를 쉽게 찾을 수 있게 되었다. (물론 계층 구조에서도 가능하긴하다.)

애플리케이션의 기능을 코드를 통해 볼 수 있게 만드는 것을 "소리치는 아키텍쳐" 라고 하는데, 이 모습을 가리키는 것이다.

## 현재의 구조 또한 최선이 아닌 이유

어댑터를 나타내는 패키지 명이 없고, 인커밍 포트, 아웃고잉 포트를 확인할 수 없다.

또한, 도메인 코드와 영속성 코드 간의 의존성을 역전시켜서 SendMoneyService 가 AccountRepository 인터페이스만 의존하도록 했음에도

도메인 코드가 영속성 코드에 의존하는 것을 막을 수 없다. (AccountRepositoryImpl 을 의존하는 방법을 막을 수 없다.)

---

# 표현력 있는 구조의 시작

    buckpal
      -- account
          -- adapter
              -- in
                  -- web
                      -- AccountController
              -- out
                  -- persistence
                      -- AccountPersistenceAdapter
                      -- SpringDataAccountRepository
          -- domain
              -- Account
              -- Activity
          -- application
              -- SendMoneyService
              -- port
                  -- in
                      -- SendMoneyUseCase
                  -- out
                      -- LoadAccountPort
                      -- UpdateAccountStatePort

이 구조의 요소들은 패키지 하나에 하나씩 직접 매핑된다.

- 최상위 패키지 account는 Account와 관련된 유스케이스를 구현한 모듈임을 알리는 역할을 할 수 있다.
  - account 하위 패키지로는 adater가 있는데, 이 글의 맨 윗부분에서 다룬 Controller, Respository를 구현하고 명세하는 역할을 한다.

- 그 다음 레벨에는 도메인 모델이 속한 doamin 패키지가 있다.

- 이와 같은 레벨에는 Application 패키지가 있다. 이는 도메인 모델의 서비스를 담당한다.
  - SendMoneyService 는 SendMoneyUseCase를 구현하고, LoadAccountPort 와 UpdateAccountStatePort 를 사용한다.

---

# 표현력 있는 구조의 장점

![](../images/DIFlow.jpg)

위와 같은 의존성 주입 흐름을 챙길 수 있다.

---
