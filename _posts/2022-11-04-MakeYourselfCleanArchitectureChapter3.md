---

title: 만들면서 배우는 클린 아키텍처

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

# CHAPTER 04. 코드 구성하기

---

### 전체 코드

[CHAPTER 4. 전체 코드 확인하기](https://github.com/Be-GGanboo-With-Java/MadeYourself_CleanArchitecture/tree/main/examplecode/src/main/java/io/reflectoring/buckpal)

---

## 풍부한 도메인 모델 vs 빈약한 도메인 모델

풍부한 도메인 모델은 애플리케이션의 코어에 있는 엔티티에서 가능한 많은 도메인 로직이 구현된다.

엔티티들은 상태를 변경하는 메서드를 제공하고, 비즈니스 규치엑 맞는 유효한 변경만을 허용한다. --> DDD 철학을 따른다.

빈약한 도메인 모델은 엔티티 자체가 굉장히 얇고 엔티티 상태를 표현하는 필드와 이 값에 대한 Getter/Setter 밖에 없다. --> 보통 우리가 사용하는 JPA Entity의 모습이다.


### 풍부한 도메인 모델의 유스케이스 구현부

[CHAPTER 4. 풍부한 도메인 모델의 유스케이스 구현부](https://github.com/Be-GGanboo-With-Java/MadeYourself_CleanArchitecture/blob/main/examplecode/src/main/java/io/reflectoring/buckpal/account/domain/Account.java)

풍부한 도메인 모델의 유스케이스는 도메인 모델의 진입점으로 동작한다. (p.48)

이어서 유스케이스는 사용자의 의도만을 표현하면서 이 의도의 실제 작업을 수행하는 도메인 엔티티 메서드 호출로 변환다.

즉, Controller 역할을 하는 Adapter 가 아닐까 하는 추측이 있다. (이건 좀 이야기 해봐야할듯 유스케이스가 뭔지 정확히 이해가 안되어서 뭔말인지 모르겠다.)


### 빈약한 도메인 모델의 유스케이스 구현부

빈약한 도메인 모델의 유스케이스는 유스케이스 클래스 자체에 있다.

도메인 로직이 유스케이스 클래스에 구현돼 있다는 것이다.

엔티티의 상태 변경, 데이터베이스 접근을 담당하는 아웃고잉 포트에 엔티티에 전달할 책임 역시 유스케이스 클래스에 있다.

> 실제로 우리는 보편적으로 Controller -> Service or Repository -> DB 의 흐름으로 위에서 이야기하는 흐름을 수행한다고 본다.


---

# 클린 아키텍처

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fk.kakaocdn.net%2Fdn%2FEeyCW%2Fbtq4oQ8NtZa%2Fl1Y7hCwXYnpmJzCeClAEKk%2Fimg.jpg)

## 0. Entity

> 엔티티는 가장 일반적이고 고수준의 규칙을 캡슐화한다.
>
> 무슨 소린지 와닿지 않으니 코드로 살펴보자

### Character.java
    @Getter
    @AllArgsConstructor
    public class Character {
        private int level;
        private int exp;
        private String job;
        private final String nickname;

        public void increaseLevel(int level) {
            this.level += level;
        }

        public void increaseEXP(int exp) {
            this.exp += 10;
        }

        public void changeJob(String newJob) {
            this.job = newJob;
        }

        public void changeNickname(String newNickname) {
            this.nickname = newNickname;
        }
    }

Spring Boot 환경에서 "엔티티"는 위와 같이 풍부한 도메인을 바탕으로한 Entity 라고 알아두면 된다.

> 실제 JPA 엔티티가 아님을 유의하자. 그 JPA 엔티티의 복사본이라고 생각하면 좋다.

풍부한 도메인을 셋팅해두면 좋은 이유는 아래와 같다. (메이플스토리 게임내에서 엔티티라고 생각해보자)

만약 유저 Diger 가 몬스터를 사냥해서 레벨이 올랐다고 하자. 그런데 방학때 마다 메이플은 레벨업을 할 떄마다 2레벨을 추가로 올려주는 이벤트를 한다.

이런 상황에서 increaseLevel 메서드를 수정해야하는 상황이 발생하겠는가?

정답은 아니다. 그냥 쓰면된다.

이렇게 애플리케이션의 동작(시나리오)의 변경이 해당 계층에 영향을 주면 안 되도록 할 수 있다는 장점이 있다.

---

## 1. UseCase

> 애플리케이션의 비즈니스 규칙을 캡슐화한다. 해당 계층의 변경은 엔티티 계층에 영향을 주어선 안되며 UI, 프레임워크 등의 변경이 해당 계층으로 영향을 주어선 안된다.
>
> 단, 애플리케이션의 동작이 변경되었을 땐 해당 계층이 영향을 받는다. 해당 계층은 Entity의 비즈니스 로직을 애플리케이션의 동작에 따라 실행한다.
>
> 즉, 애플리케이션 요청을 실질적으로 받아 처리하는 계층이고, Entity에 작성되어있는 비즈니스 로직을 요구사항에 맞춰 실행하는 역할을 한다.

Entity 예시에 이어서 예시 코드를 작성해보겠다.

### NotEventedLevelUpInputBoundary.java
    public interface NotEventedLevelUpInputBoundary {
        void normalLevelUp(long userIdx);
        void eventLevelUp(long userIdx);
    }

UseCase 계층의 추상화를 InputBondary 라고 부른다.

<br>

### LevelUpInteractor.java
    @Service
    @RequiredArgsConstructor
    public class NotEventedLevelUpInteractor implements NotEventedLevelUpInputBoundary {
        private final CharacterGateway characterGateway;

        @Override
        public void normalLevelUp(long userIdx) {
            CharacterGatewayResponseModel characterGatewayResponseModel = characterGateway.findById(userIdx);

            characterGatewayResponseModel.increaseLevel(1);

            return characterGatewayResponseModel;
        }

        @Override
        public void eventLevelUp(long userIdx) {
            CharacterGatewayResponseModel characterGatewayResponseModel = characterGateway.findById(userIdx);

            characterGatewayResponseModel.increaseLevel(3);

            return characterGatewayResponseModel;
        }
    }

UseCase 계층의 구현체를 Interactor 라고 부른다.

Entity에 선언되어있는 메서드를 활용하여 요구사항에 맞는 비즈니스 로직을 수행했다.

예시에 대한 부연 설명을 붙이자면, 이벤트를 하지 않을 때의 레벨업은 레벨이 1 올라야하는 요구사항에 대한 내용이다.

---

### 2. Interface Adapter

> 인터페이스 어댑터는 엔티티, 유스 케이스 계층이 다루기 편한 데이터 포맷에서 UI, DB가 다루기 편한 데이터 포맷으로 바꿔주는 역할이다.


### CharacterGateway.java

    public interface CharacterGateway {
        void normalLevelUp(NormalLevelUpGatewayRequestModel normalLevelUpGatewayRequestModel);
        void eventLevelUp(EventLevelUpGatewayRequestModel eventLevelUpGatewayRequestModel);
    }

### CharacterTable.java

    @Entity
    @Data
    @Builder
    @NoArgsConstructor
    public class CharacterTable {
        @Id @GeneratedValue(strategy = GenerationType.IDENTITY)
        private Long id;

        @Column
        private int level;

        @Column
        private int exp;

        @Column
        private String job;

        @Column
        private final String nickname;
    }

## CharacterJPA.java

    @Service
    @RequiredArgsConstructor
    public class CharacterJPA implements CharacterGateway {

        private final JPACharacterRepository jpaCharacterRepository;

        @Override
        public void normalLevelUp(NormalLevelUpGatewayRequestModel normalLevelUpGatewayRequestModel) {
            jpaCharacterRepository.save(new CharacterTable(
                    createPostGatewayRequestModel.getId(),
                    createPostGatewayRequestModel.getLevel() + 1,
                    createPostGatewayRequestModel.getExp(),
                    createPostGatewayRequestModel.getJob,

            ));
        }

        @Override
        public void EventLevelUp(EventLevelUpGatewayRequestModel eventLevelUpGatewayRequestModel) {
            jpaCharacterRepository.save(new CharacterTable(
                    createPostGatewayRequestModel.getId(),
                    createPostGatewayRequestModel.getLevel() + 3,
                    createPostGatewayRequestModel.getExp(),
                    createPostGatewayRequestModel.getJob,
            ));
        }
    }

---

# 실제 코드를 통해 Entity, UseCase, Adapter 살펴보기

[참고 링크](https://zkdlu.tistory.com/4)

![](../images/cleanArchitecture/chapter4/육각형 아키텍처 패키지 구조.png)

![](../images/cleanArchitecture/chapter4/실전 예제 코드 헥사고날 다이어그램.jpg)


## 0. 실전 Entity

### domain

![](../images/cleanArchitecture/chapter4/Character.png)

---

## 1. 실전 UseCase

### application.ports.dto

![](../images/cleanArchitecture/chapter4/RequestDTO.png)

![](../images/cleanArchitecture/chapter4/ResponseDTO.png)

### application.ports.input

![](../images/cleanArchitecture/chapter4/NormalLevelUpUseCase.png)

![](../images/cleanArchitecture/chapter4/BurningLevelUpUseCase.png)

### application.ports.output

![](../images/cleanArchitecture/chapter4/LevelUpOutputPort.png)

### application.service

![](../images/cleanArchitecture/chapter4/BurningLevelUpService.png)

![](../images/cleanArchitecture/chapter4/NormalLevelUpService.png)

---

## 2. 실전 Adapter

### infrastructure.adapters.in

![](../images/cleanArchitecture/chapter4/NormalLevelUpController.png)

![](../images/cleanArchitecture/chapter4/BurningLevelUpController.png)

### infrastructure.adapters.out.persistence

![](../images/cleanArchitecture/chapter4/ChracterRepository.png)

![](../images/cleanArchitecture/chapter4/LevelUpAdapter-1.png)
![](../images/cleanArchitecture/chapter4/LevelUpAdapter-2.png)

---

# CHAPTER 11. 의식적으로 지름길 사용하기

---

# 0. 용어 재정비

포트 : 인터페이스

어댑터 : 구현체

유스케이스 : 비즈니스 로직을 수행하는 클래스

도메인 : 비즈니스 로직 + 엔티티 + 아웃고잉 어댑터(포트) 로 구성되어 특정 요구사항을 만족시킬 수 있는 집합

---

# 1. 지름길은 깨진 창문 같다.

서로 다른 길가에 B와 P라는 자동차를 각각 한 대씩 뒀다.

B라는 차는 24시간이 지나지도 않았을 때 차의 부품이 전부 도난 당했지만

P라는 차는 비교적 잘사는 동네에 둬서 그런가? 도난당한 부품이 없었다.

그래서, P라는 차에 창문을 한 개 깨뜨린 뒤 다시 관찰한 결과, B와 같이 모든 부품이 도난당했다.

> 이것을 "깨진 창문 이론" 이라고 한다.

창문이 깨져있으니 관리되지 않았다고 판단하여 인간의 뇌는 이를 더 망치고 망가뜨려도 된다고 생각하게 된다는 것이다.

---

# 2. 깨진 창문 이론을 코드에 적용시킨다면?

- 품질이 떨어진 코드에서 작업할 때 더 낮은 품질의 코드를 추가하기가 쉽다.
- 코딩 규칙을 많이 어긴 코드에서 작업할 때 또 다른 규칙을 어기기도 쉽다.
- 지름길을 많이 사용한 코드에서 작업할 때 또 다른 지름길을 추가하기도 쉽다.

> 이 모든것을 고려하면 "레거시"라고 불리는 많은 코드의 품질이 시간이 가면서 더 낮아진다는 것을 알 수 있다.

---

# 3. 깨끗한 상태로 시작할 책임

> 우리는 깨진 창문 심리에 무의식적으로 영향을 받는다.

그래서 가능한 **지름길을 거의 쓰지 않고** **기술 부채를 지지 않은 채**로 프로젝트를 시작하는것이 중요하다.

왜냐하면, 소프트웨어는 보통 비용이 크고 장기적인 노력을 필요로 하기 때문에 깨진 창문을 막는 것이 아주 중요한 작업이기 때문이다.

미완성된 프로젝트를 넘겨받았을 때 깨진 창문의 심리를 작용하는 코드가 많다면 그 프로젝트의 결과는 어떻겠는가.

---

# 4. 지름길이 실용적일 때도 있긴하다!

프로젝트 전체적으로 봤을 때 그리 중요하지 않은 부분이나, 프로토타이핑 작업, 경제적인 이유 등에서는 실용적으로 작용할 수 있다.

하지만 의도적으로 지름길을 사용할때는 세심하게 잘 기록해둬야한다.

> 지름길 기록에 대해선 마이클 나이가드가 제안한 아키텍처 결정기록의 형태를 고려해보자.
>
> [Michael Nygard - Architecture-Decision-Record](https://cognitect.com/blog/2011/11/15/documenting-architecture-decisions)


### ADR - DECISION

> **제목** : 짧은 단어 몇가지로 기록한다.
>
> 예를들어, "ADR 1: Deployment on SpringBoot 2.7.2" or "ADR 9: LDAP for Multitenant Integration" 와 같이 남긴다.
>
> **문맥** : This section describes the forces at play, including technological, political, social, and project local. These forces are probably in tension, and should be called out as such. The language in this section is value-neutral. It is simply describing facts.
>
> **결정** : This section describes our response to these forces. It is stated in full sentences, with active voice. "We will …"
>
> **상태** : 결정 진행 중인지, 결정된 사항인지, 사용하지 않을 것인지 등에 대한 기록이다.
>
> **결과** : 결정을 적용한 후 모든 결과를 작성한다. 긍정적인 결과만이 아니라 부정적, 중립적 결과도 모두 적는다.

---

# 5. 육각형 아키텍처에서 고려해볼 수 있는 (지름길 1)

4장에서, 유스케이스마다 다른 입출력 모델을 가져야 한다고 이야기했었다.

즉, 입력 파라미터의 타입과 반환값의 타입이 유스케이스마다 달라야 한다는 것이다.

![](../images/cleanArchitecture/chapter11-1.jpg)

위 사진을 보면 SendMoneyService, RevokeActivityService 라는 두 개의 유스케이스가

SendMoneyCommand 라는 구현체를 공유하는 모습을 보여주는 내용이다.

이렇게 되면 유스케이스들 사이에 결합이 생기게 된다.

> 이게 무슨 문제인가? --> 만약, SendMoneyCommand 클래스가 변경되면, 두 유스케이스 **모두** 영향을 받는다.
>
> 단일 책임 원칙에서 이야기하는 "변경할 이유"를 공유하는 것이다.

또한 출력 모델을 공유하는 경우에도 이 상황과 마찬가지다.

### 유스케이스 간 입출력 모델을 공유하는 상황은?

유스케이스 간 입출력 모델을 공유하는 상황은 유스케이스가 기능적으로 묶여 있을 때만 유효하다.

그러니까, 특정 세부사항을 변경할 경우 실제로도 두 유스케이스에 영향을 주고 싶을 때이다.

### 이외의 상황에서는 유스케이스 간 입출력 모델 공유는 지름길이 된다.

따라서, 비슷한 개념의 유스케이스를 여러개 만들어야 한다면 유스케이스를 독립적으로 진화할 필요가 있는지 주기적으로 질문해야한다.

---

# 6. 도메인 엔티티를 입출력 모델로 사용하기 (지름길 2)

도메인 엔티티 : Account

인커밍 포트 : SendMoneyUseCase

가 있다고 했을 때 관계도는 다음과 같다.

![](../images/cleanArchitecture/chapter11-2.jpg)

인커밍 포트(SendMoneyUseCase) 는 도메인 엔티티(Account) 에 의존성을 지니고 있다.

그 결과로 Account 엔티티는 변경해야할 이유가 생기게 된다.

> 왜 Account 엔티티가 변경사항이 생기는가?

현재 Account 에 존재하지 않는 정보를 유스케이스가 필요로 한다고 했을 때

이 정보는 최종적으로 Account 엔티티에 저장되어야 하는 것이아니라, 다른 도메인 혹은 다른 바운디드 컨텍스트에 저장돼야한다.

즉, 지금 존재하는 Account 엔티티의 정보만을 필요로 할 것이라면 Account 엔티티를 사용할 것이 아니라, 다른 엔티티를 만들어야 한다는 것이다.

그럼에도 불구하고, Account 엔티티의 필드 하나만 추가하면 끝나는 상황이라 이렇게 해결하고자 할 수 있지만

유스케이스가 단순히 DB 필드 몇 개를 업데이트 하는 수준이 아니라 복잡한 도메인 로직을 구현해야한다면

유스케이스 인터페이스에 대한 전용 입출력 모델을 만들어야한다.

> 이렇게 해야 유스케이스의 변경이 도메인 엔티티까지 연쇄되지 않을 것이기 때문이다.

### 만약 Account 엔티티(풍부한 도메인 엔티티 라는 가정)의 필드를 추가하고 로직을 추가하면 어떻게 될까?

시간이 지나면서 매우 복잡한 엔티티로 이루어진 도메인이 탄생하게 되어 유지보수 측면에서만 봐도 매우 좋지 않게 된다.

따라서 도메인 엔티티를 입력 모델로 사용했더라도, 도메인 로직 요구사항에 따라서 이를 적정선에서 교체해야함을 판단할 줄 알아야한다.

---

# 7. 인커밍 포트 건너뛰기 (지름길 3)

아웃고잉 포트(DB 접근 인터페이스 라고 하자)는 애플리케이션 계층과 아웃고잉 어댑터(DB 접근 구현체 라고 하자) 사이의 의존성을 역전시키기 위한 필수 요소이다.

하지만 인커밍 포트(컨트롤러 인터페이스 라고 하자)는 의존성 역전에 필수적인 요소는 아니다.

인커밍 어댑터가 인커밍 포트 없이 애플리케이션 서비스에 직접 접근할 수 있도록 하자. (우리가 작성해온 보편적인 흐름이다.)

![](../images/cleanArchitecture/chapter11-3.jpg)

인커밍 포트를 제거함으로써 인커밍 어댑터와 애플리케이션 계층 사이의 추상화를 줄였다.

하지만 인커밍 포트는 애플리케이션 중심에 접근하는 진입점을 정의한다.

이를 제거하면, 특정 유스케이스를 구현하기 위해 어떤 서비스 메서드를 호출해야할지 알아내기 위해 애플리케이션의 내부 동작에 대해 더 잘 알아야한다.

> 쉽게 생각해보면, **컨트롤러 인터페이스**를 만들어, 도메인 로직을 호출하는 내용을 명세해두면 이에 대한 **구현체 컨트롤러를 작성**하기만 하면된다.
>
> 하지만 **직접 컨트롤러 클래스**를 만들게 되면, **어떤 도메인 로직이 특정 컨트롤러 메서드에 어울리는지** 전부 **고려할 줄 알아야** 한다는 것이다.

### 인커밍 포트를 유지해야하는 또 다른 이유

> 아키텍처를 쉽게 강제할 수 있다.

10장에서 소개한 아키텍처를 강제하는 옵션들을 이용하면,

**인커밍 어댑터(컨트롤러 구현체)가** 애플리케이션 서비스가 아닌 **인커밍 포트(컨트롤러 인터페이스)만 호출**할 수 있도록 한다.

이렇게 되면, 인커밍 어댑터에서 호출할 의도가 없던 서비스 메서드를 실수로 호출하는 일이 발생할 수 없다.

> 아직 잘 와닿지 않는 얘기이다. 구현체가 인터페이스를 호출한다?
>
> 음... 내가 잘 못 비유하면서 생각된 오류일지도 모른다.

애플리케이션의 규모가 작거나 인커밍 어댑터가 하나밖에 없어서 모든 제어 흐름을 인커밍 포트의 존재가 필요 없이 파악할 수 있다면 괜찮다.

하지만, 애플리케이션의 규모가 이후로도 계속 작게 유지될것이라는 보장이 없으면 지금 이야기하는 지름길 3번은 적절하지 않다.

---

# 8. 애플리케이션 서비스 건너뛰기 (지름길 4)

![](../images/cleanArchitecture/chapter11-4.jpg)

아웃고잉 어댑터에 있는 AccountPersistenceAdapter 클래스는 직접 인커밍 포트를 구현해서

일반적으로 인커밍 포트를 구현하는 애플리케이션 서비스를 대체한다.

### 조금 더 쉽게 정리하자면

간단한 CRUD 유스케이스에서는, 보통 애플리케이션 서비스가 도메인 로직 없이

생성, 업데이트, 삭제 요청을 그대로 영속성 어댑터에 전달하는 방식으로 사용해왔을 것이다.

하지만 이 방법은 인커밍 어댑터(컨트롤러), 아웃고잉 어댑터(레포지토리) 사이에 모델을 공유해야한다.

만약 시간이 지남에 따라 CRUD 유스케이스가 점점 복잡해지면

도메인 로직을 그대로 아웃고잉 어댑터에 추가하고자 하는 생각이 들 것이다. (우린 보통 이런 느낌으로 해결을 해왔다.)

이렇게 되면 도메인 로직이 흩어져서 도메인 로직을 찾거나 유지보수하기 힘들어진다.

<br>

--> UserRepository 에 100개의 메서드가 명세되어있고, 그중의 특정 5개를 사용하여 한 도메인 로직을 만들었다고 해보자.

이런 환경에서 도메인 로직이 무엇인지 명확하게 뽑아낼 수 있겠는가? 에 대한 문제점을 이야기 하는 것이다.

결국에 단순히 전달만 하는 보일러플레이트 코드가 가득한 서비스가 되는것을 방지하기 위해 간단한 CRUD 서비스에서는 애플리케이션 서비스를 건너뛰기 할 수도 있다.

하지만 유스케이스가 단순 엔티티 생성, 엔티티 업데이트, 엔티티 삭제 보다 더 많은 일을 하게 되면 적합하지 않다.

---

# 결론. 유지보수 가능한 소프트웨어를 만드는데 어떻게 도움이 되는 것인가?

지름길이 합리적일 때도 있다. 간단한 CRUD 유스케이스에 대해서는 전체 아키텍처를 구현하는 것이 더 지나치게 느껴질 수도 있고,

이 장에서 이야기하는 지름길이 지름길로 안느껴지고 당연하게 느껴질 수 있다.

> 하지만 모든 애플리케이션은 작게 시작한다. 그러므로 유스케이스가 단순한 CRUD 상태에서 벗어나는 시점이 언제인지 파악을 해야한다.
>
> 그 후 지름길을 장기적으로 더 유지보수 할 수 있는 좋은 아키텍처로 대체해야한다.

### 단순 CRUD 상태에서 벗어나지 않는 유스케이스는 그냥 냅둬도 된다. 오히려 경제적이다.

### 하지만 어떤 경우던, 아키텍처와 지름길에 대해서 왜 선택했는가에 대한 기록을 필수적으로 남겨야 협업간의 좋은 결정을 유도할 수 있게된다.
