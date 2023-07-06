---

title: 만들면서 배우는 클릭 아키텍처 CHAPTER 4. 유스케이스 확인하기

date: 2022-11-04
categories: [Architecture]
tags: [Book]
layout: post
toc: true
math: true
mermaid: true

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
