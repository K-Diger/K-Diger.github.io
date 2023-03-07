---

title: 우아한 아키텍처와 객체지향
author: 김도현
date: 2023-03-06
categories: [OOP]
tags: [OOP]
math: true
mermaid: true

---

모든 내용은 다음 링크를 출처로 두고 있다.

[우아한 객체지향 - 조영호](https://www.youtube.com/watch?v=26S4VFUWlJM)

객체지향적으로 애플리케이션을 짜려면 아키텍처는 어떻게 해야하나?

라는 주제를 다루는 것이다.

---

# 아키텍처에서 가장 중요한 점

아키텍처를 설계함에 있어서 중요한 점은 관심사 분리이다.

왜 중요할까에 대해서는 조금씩 설명한다.

# 관심사 분리 - 계층형 구조

관심사 분리를 위하여 우리는 계층형 구조를 사용한다.

일반적인 계층형 구조는 다음과 같다.

| Presentation(화면조작/사용자 입력 처리) |
|------------------------------|
| Domain(비즈니스 로직/도메인 로직처리)     |
| DataSource(데이터 조작)           |

이렇게 유사한 관심사를 계층으로 분할하여 유연성/재사용성을 확보할 수 있다.

계층형 구조에서 가장 중요한 점은 Domain 계층이다. Domain의 설계로 인해 다른 계층의 설계 구조 또한 변경되기 때문이다.

# 계층형 구조를 설계하는 방법 (패턴)

- 절차지향 (Transaction Script)

- 객체지향 (Domain Model)

이 두 가지가 가장 널리 사용되는 패턴이다.

# 도메인 계층이 가장 중요하다!

도메인 계층을 절차지향으로 작성할 것인지, 객체지향으로 작성할 것인지에 따라서

그 외의 계층들이 그에 맞게 작성하기로 결정된다.

---

# Transaction Script vs Domain Model

## 예제 - 영화 예매 시스템

### 도메인 개념 - 영화

우리가 예매를 하는 것은 "영화"가 아니라 "상영 정보"(날짜, 시간, 극장지점 등)를 예매하는 것이다.

사실상 "영화"라는 개념은 메타데이터이다.

### 도메인 개념 - 상영

상영은 위에서 언급했던 날짜, 시간, 극장지점 등을 가지고 있다.

### 도메인 개념 - 할인 정책

상영정보를 예매할 때 두 가지의 할인 정책이 존재한다.

- 특정 금액을 할인

- 구매 금액의 특정 비율을 할인

### 도메인 개념 - 할인 규칙

할인 규칙은 두 가지의 조건이 존재한다.

- 특정 행위(조조, 10회이상 상영, 예매내역)

- 특정 시간대

### 도메인 개념 - 예매

예매라는 도메인 개념은 최종 결과물에 대한 도메인으로 볼 수 있다.

즉, **"영화 제목"**, **"상영정보"**, "**할인정책 + 할인 규칙이 적용된 가격**" 을 가지고 있다.

## Transaction Script 시나리오 - 예매하기

### 도메인 개념 정리

- 영화 (Movie)

- 상영 (Showing)

- 할인 정책 (Discount)

- 할인 규칙 (Rule)

- 예매 (Reservation)

절차적인 방식은

- 데이터를 어떻게 다룰 지

- 데이터를 다루는 방법을 어떻게 활용할지

를 철저하게 구분해서 사용한다.

예를 들면 DB와 매핑하기 위한 엔티티 클래스와 DAO를 만들어 놓고 시작하는 것이 그 예시이다.

따라서 어떤 비즈니스 로직을 처리하기 위한 서비스를 만든 이후에, 그 로직을 처리하기 위한 DAO들을 주입 받는 방식이 된다.

코드로 예를 들면 이렇게 된다.

```java
public class ReservationService {

    @Transactional
    public Reservation reserveShowing(int customerId, int showingId, int audienceCount) {
        1. DB로부터 Movie, Showing, Rule 조회

        2. if (Rule 이 존재하면){
            Discount를 읽어 할인된 요금 계산
        } else{
            Movie의 정가를 이용해 요금 계산
        }
        3. Reservation 생성 후 DB 저장
    }
}
```

조금 더 구체적인 코드로 바꾸면

```java
public class ReservationService {

    @Transactional
    public Reservation reserveShowing(int customerId, int showingId, int audienceCount) {

        Showing showing = showingDAO.selectShowing(showingId);
        Movie movie = movieDAO.selectMovie(showing.getMovieId());
        List<Rule> rules = ruleDAO.selectRules(movie.getId());

        Rule rule = findRule(showing, rules);
        Money fee = movie.getFee();
        if (rule != null) {
            fee = calculateFee(movie);
        }

        Reservation result = makeReservation(customerId, showingId, audienceCount, fee);
        reservationDAO.insert(result);

        return result;
    }

    private Rule findRule(Showing showing, List<Rule> rules) {
        for (Rule rule : rules) {
            if (rule.isTimeOfDayRule()) {
                if (showing.isDayOfWeek(rule.getDayOfWeek()) &&
                    showing.isDurationBetween(rule.getStartTime(), rule.getEndTime())) {
                    return rule;
                }
            } else {
                if (rule.getSequence() == showing.getSequence()) {
                    return rule;
                }
                return null;
            }
        }
    }

    private Money calculateFee(Movie movie) {
        Discount discount = discountDAO.selectDiscount(movie.getId());

        Money discountFee = Money.ZERO;
        if (discountFee != null) {
            if (discountFee.isAmountType()) {
                discountFee = Money.wons(discount.getFee());
            } else if (discount.isPercentType()) {
                discountFee = movie.getFee().times(discount.getPercent());
            }
        }
        return movie.getFee().minus(discountFee);
    }

    private Reservation makeReservation(
        int customerId, int showingId, int audienceCount, Money payment
    ) {
        Reservation result = new Reservation();
        result.setCustomerId(customerId);
        result.setShowingId(showingId);
        result.setAudienceCount(audienceCount);
        result.setFee(payment);

        return result;
    }
}
```

위와 같이 된다. 위 코드는 메서드를 쪼개긴 했지만 하나의 트랜잭션 안에서 모든 로직을 처리하도록 한다.

그리고 이러한 특징이 Transaction Script 패턴의 특징인 것이다.

그래서 모든 제어가 한 메서드에 몰려있는 Sequence Diagram이 그려지는 형식이 된다.

---

## Transaction Scirpt를 Domain Model로 변환하기 - 객체지향 설계

객체지향으로 작성한다는 것 == Process와 Data를 하나의 덩어리로 보는 것

데이터와 프로세스 볼 것이 아니라, 객체간의 협력 관계를 만들어야한다.

그리고 협력 관계는 객체들 간 메세지를 전송하는 것으로 한다.

이 표현들이 다소 추상적이다.

그래서 CRC Card를 통해 객체지향 설계 도구를 활용하면 조금 더 정리할 수 있다.

![](https://github.com/K-Diger/K-Diger.github.io/blob/main/images/architecture-oop/img.png?raw=true)

- 상단 : 후보(역할/객체 를 의미한다.)

- 좌측 : 책임

- 우측 : 협력관계

### 책임 할당을 잘 하기 위해선?

객체지향에서 객체는 상태를 갖고 있으며 스스로 그 상태를 제어할 수 있어야한다.

또한 그 객체에 적절한 책임 할당을 하기 위해서는, 어떤 행위에 필요한 정보를 가장 많이(잘) 알고 있는 객체에게

책임을 부여하는 것이다. 이것이 책임할당 패턴 중 Creator 패턴으로 불리는 방식이다.

### Domain Model 방식 개발 - CRC 작성

CRC Card를 통해 영화 예매 시나리오의 객체를 설계하면 다음과 같다.

![img.png](https://github.com/K-Diger/K-Diger.github.io/blob/main/images/architecture-oop/img1.png?raw=true)

#### Showing(상영)

- 상영 정보를 알고 있다.

- 예매 정보를 생성한다.

- Movie와 Customer와 협력한다.

#### Movie(영화)

- 영화 정보를 알고 있다.

- 가격을 계산한다.

- Discount Strategy와 협력한다.

#### Discount Strategy(할인 정책)

- 할인 정책을 알고있다.

- 할인된 가격을 계산한다.

- Rule과 협력한다.

#### Rule(할인 규칙)

- 할인 규칙을 알고있다.

- 할인 여부를 판정한다.

## Transaction Scirpt를 Domain Model로 변환하기 - 구현

```java
public class Showing {

    // 예매 정보 생성은 Reservation에게 위임한다.
    public Reservation reserve(Customer customer, int audienceCount) {
        return new Reservation(customer, this, audienceCount);
    }

    // 금액 계산은 Movie에게 위임한다.
    public Money calculateFee() {
        return movie.calculateFee(this);
    }
}
```

```java
public class Reservation {

    // 예매 정보를 생성한다.
    public Reservation(Customer customer, Showing showing, int audienceCount) {
        this.customer = customer;
        this.showing = showing;
        this.fee = showing.cacaluteFee().times(audienceCount);
        this.audienceCount = audienceCount;
    }
}
```

```java
public class Movie {

    // 예매 가격을 계산한다.
    public Money calculateFee(Showing showing) {
        return fee.minus(discountStrategy.cacluateDiscountFee(showing));
    }
}
```

```java
public abstract class DiscountStrategy {

    // 할인 규칙에 해당하는지 확인하는 것을 Rule에게 위임하며
    // 그 내용을 바탕으로 할인될 금액을 계산한다.
    public Money calculateDiscountFee(Showing showing) {
        for (Rule rule : rules) {
            if (rule.isStatisfiedBy(showing)) {
                return getDiscountFee(showing);
            }
        }
        return Money.ZERO;
    }
}
```

```java
public class AmountDiscountStrategy extends DiscountStrategy {

    // 값으로 할인 하는 전략 생성
    protected Money getDiscountFee(Showing showing) {
        return discountMount;
    }
}
```

```java
public class NonDiscountStrategy extends DiscountStrategy {

    // 할인값이 없는 전략 생성
    protected Money getDiscountFee(Showing showing) {
        return Money.ZERO;
    }
}
```

```java
public class PercentDiscountStrategy extends DiscountStrategy {

    // 비율로 할인 하는 전략 생성
    protected Money getDiscountFee(Showing showing) {
        return showing.getFixedFee().times(percent);
    }
}
```

```java
public interface Rule {

    boolean isSatisfiedBy(Showing showing);
}
```

```java
// 10회 째 상영되는 상영인지 확인 - 할인 규칙
public class SequenceRule implements Rule {

    boolean isSatisfiedBy(Showing showing) {
        return showing.isSequence(sequence);
    }
}
```

```java
// 조조, 특정 시간대 등에 포함되는지 확인 - 할인 규칙
public class TimeOfDayRule implements Rule {

    boolean isSatisfiedBy(Showing showing) {
        return showing.isPlayingOn(dayOfWeek) &&
            Interval.closed(startTime, endTime)
                .includes(showing.getPlayingInterval());
    }
}
```

---

# 객체지향적인 설계의 기초

객체지향적으로 작성하는 방법은 책임을 수행할 적절한 객체에게 위임하는 것이 기본 철학이다.

또한 DB연결, 네트워크 혹은 예를들어 AWS와 연결하는 로직 등 애플리케이션 전반적으로, 도메인과 관계없이 사용되는 로직을 애플리케이션 로직이라고 한다.

그리고 우리가 설계하는 도메인 그 자체를 다루는 로직을 도메인 로직이라고 한다.

우리는 이 도메인 로직을 외부로부터 영향받지 않게 철저한 캡슐화를 하여 사용해야한다.

그것이 객체지향의 핵심 중 하나이다.

그래서 등장하는 아키텍처 계층이 있다. 바로 서비스 계층이다.

| Presentation   |
|----------------|
| Service        |
| Domain         |
| Infrastructure |

이 Service 계층이 도메인 모델을 보호하는 역할이다.

즉, 애플리케이션의 경계로써 애플리케이션 로직을 다루고, 도메인 로직의 재사용성을 높여준다.

이 계층에서 트랜잭션이 시작되는 것이 일반적이다.

서비스 계층의 이상적인 예시는 다음과 같다.

```java
public class ReservationService {

    @Transactional
    public Reservation reservationShowing(int reserveId, int showingId, int audiuenceCount) {
        Customer customer = customerRepository.find(reserveId);
        Showing showing = showingRepository.find(showingId);
        Reservation reservation = showing.reserve(customer, audiuenceCount);

        reservationRepository.save(reservation);
        return reservation;
    }
}
```

- 필요한 데이터를 DB에서 읽는다.

- 나머지 처리는 도메인 객체에 위임한다.

- 처리 결과를 DB에 저장한다.

이 흐름을 가진 것이 이상적인 Service Layer가 하는 일인데, 더 간단하게 요약하자면

- 도메인 로직을 처리하기 위한 준비작업

- 도메인 로직을 처리하는데에 발생하는 후 처리(예외, 반환 값 셋팅 등)

## 다시 한 번 비교해보자 Transaction Script vs Domain Model

### Transaction Script Service (Fat Service)

```java
public class ReservationService {

    @Transactional
    public Reservation reserveShowing(int customerId, int showingId, int audienceCount) {

        Showing showing = showingDAO.selectShowing(showingId);
        Movie movie = movieDAO.selectMovie(showing.getMovieId());
        List<Rule> rules = ruleDAO.selectRules(movie.getId());

        Rule rule = findRule(showing, rules);
        Money fee = movie.getFee();
        if (rule != null) {
            fee = calculateFee(movie);
        }

        Reservation result = makeReservation(customerId, showingId, audienceCount, fee);
        reservationDAO.insert(result);

        return result;
    }
}
```

위 패턴으로 인해 reserveShowing이 수행하는 책임은 트랜잭션 관리, 애플리케이션 로직, 도메인 로직이 있다.

위 로직을 처리하면서 어디선가 에러가 발생한다면? 위는 단편적인 간단한 예시이기 때문에 와닿지 않을 수 있겠지만

실제로 로직이 긴 요구사항을 다룰 때에는 어디서 에러가 발생하는지 찾기 어려워진다.

### Domain Model Service (Thin Service)

```java
public class ReservationService {

    @Transactional
    public Reservation reservationShowing(int reserveId, int showingId, int audiuenceCount) {
        Customer customer = customerRepository.find(reserveId);
        Showing showing = showingRepository.find(showingId);
        Reservation reservation = showing.reserve(customer, audiuenceCount);

        reservationRepository.save(reservation);
        return reservation;
    }
}
```

반면에 도메인 모델은 각 객체마다 특정 책임을 맡고 있기 때문에 요구사항 처리 중 문제가 발생한 도메인에서

오류를 쉽게 찾아낼 수 있다. ORM이 등장한 배경도 이와 아주 밀접하게 관련있다. ORM은 도메인 로직 처리를 위해

DB와 패러다임을 매핑할 수 있도록 도와주며 실제 자바 코드로써 도메인 로직을 처리할 수 있도록 도와준다.

## Domain Model과 Transaction Script를 섞어 쓰면 안되나?

안되는건 아니지만 그리 좋진 않다.

Transaction Script 패턴을 도입한 이후부터, Data와 Process는 따로 놀게된다.

위의 Transaction Script 코드를 보면 데이터를 Domain에서 직접 다루지 않고 Service 계층에서 조작하는 것이 그 예시이다.

즉, 데이터 관점에서 보는 것이다. Transaction Script 에서는 Entity 객체를 데이터 덩어리로 밖에 여기지 않는다.

| Domain Model                           | Transaction Script                             |
|----------------------------------------|------------------------------------------------|
| Presentation                           | Presentation                                   |
| Service                                | Domain                                         |
| Domain                                 | Domain                                         |
| Data Source (+ Object Mapper(ex) JPA)) | Datasource (+ Table Data Gateway (ex) xxxDAO)) |

결과적으로 두 패턴의 계층을 비교하면 위와 같다.

# 비교도 간략하게 해봤는데, 도대체 어떤 패턴을 써야하는가?

> "변하지 않는 것은 모든 것이 변한다는 것이다." 라는 말이 있다.

## 영화 예매 예제에서 변화를 도입해보자 - 중복 할인 혜택 추가

이전 예제에서는 할인 혜택이 고정금액/비율금액 둘 중 하나만 적용 되었지만 이번 예제에는 변화를 주기 위해서 여러 개의 할인 혜택을 적용한다고 해보자

### Transaction Script로 변화 적용

```java
public class ReservationService {

    @Transactional
    public Reservation reserveShowing(int customerId, int showingId, int audienceCount) {

        Showing showing = showingDAO.selectShowing(showingId);
        Movie movie = movieDAO.selectMovie(showing.getMovieId());
        List<Rule> rules = ruleDAO.selectRules(movie.getId());

        Rule rule = findRule(showing, rules);
        Money fee = movie.getFee();
        if (rule != null) {
            fee = calculateFee(movie);
        }

        Reservation result = makeReservation(customerId, showingId, audienceCount, fee);
        reservationDAO.insert(result);

        return result;
    }

    // 기존 할인 혜택이 하나만 적용 되었을 때
    private Money calculateFee(Movie movie) {
        Discount discount = discountDAO.selectDiscount(movie.getId());

        Money discountFee = Money.ZERO;
        if (discountFee != null) {
            if (discountFee.isAmountType()) {
                discountFee = Money.wons(discount.getFee());
            } else if (discount.isPercentType()) {
                discountFee = movie.getFee().times(discount.getPercent());
            }
        }
        return movie.getFee().minus(discountFee);
    }

    // 중복 할인 혜택이 적용 될 때
    private Money calculateFee(Movie movie) {
        List<Discount> discounts = discountDAO.selectDiscounts(movie.getId());

        Money discountFee = Money.ZERO;
        for (Discount discount : discounts) {
            if (discountFee != null) {
                if (discountFee.isAmountType()) {
                    discountFee = Money.wons(discount.getFee());
                } else if (discount.isPercentType()) {
                    discountFee = movie.getFee().times(discount.getPercent());
                }
            }
        }
        return movie.getFee().minus(discountFee);
    }
}
```

Transaction Script 패턴을 사용하게 되면 비즈니스 로직에서 호출하는 private 메서드 내의 로직 자체를 건들여버린다.

기존 코드를 손대는 것으로 이게 잘 동작하는지도 알기 힘들다.

### Domain Model로 변화 적용

#### Composite 디자인 패턴을 통해 해결해도 좋다.

Movie의 입장에서는 하나의 DiscountStrategy나 여러개의 DiscountStrategy나 동일하다.

어차피 Movie가 할인 금액을 계산하는 것은 AmountStrategy라는 객체에게 메세지를 통해 책임을 위임하기 때문이다.

```java
public class OverlappedDiscountStrategy extends DiscountStrategy {
    private List<DiscountStrategy> discountStrategies = new ArrayList<>();

    public OverlappedDiscountStrategy(DiscountStrategy... discountStrategies) {
        this.discountStrategies = Arrays.asList(discountStrategies);
    }

    @Override
    protected Money getDiscountFee(Showing showing) {
        Money result = Money.ZERO;

        for (DiscountStrategy strategy : discountStrategies) {
            result = result.plus(strategy.calulateDiscountFee(showing));
        }
        return result;
    }
}
```

기존 코드를 수정하지 않고 DiscountStrategy를 확장한 서브 클래스를 하나 더 만든 것 뿐이다.

다시 말해서, 기존 코드의 변경을 닫은 채 확장에는 열어놓은 OCP 원칙이 적용된 것이다.

이는 의존성의 방향이 추상화로 향하는 DIP의 원칙에 맞게 설계해놓은 덕이다.

추상화에만 의존하고 있다면 사용하는 클라이언트 측은 구현체가 어떻게 바뀌던지 상관없다. 기존 코드를 하나도 손대지 않아도

변경이 적용된 구현체의 기능을 사용할 수가 있게된다.

### 갑자기 궁금한 점

우리는 Spring Boot에서 생성자 기반으로 의존성 주입을 할 때 @RequiredArgsConstructor를 사용하여 간편하게 해결하는 경우가 많다.

근데, 한 인터페이스에 대한 구현체가 여러 개라면 어떻게 의존성 주입을 할 수 있을까?

```java

@Qualifier("implementation1")
public class Implementation1 implements ExampleInterface {

}

@Qualifier("implementation2")
public class Implementation2 implements ExampleInterface {

}

@Qualifier("implementation3")
public class Implementation3 implements ExampleInterface {

}

@Component
@RequiredArgsConstructor
public class ClientClass {
    @Qualifier("implementation2")
    private final ExampleInterface exampleInterface;
}
```

위와 같이 생성자 주입을 기반으로 사용하려면 @Qualifier 애노테이션으로 특정 구현체의 bean 이름을 명시한 후 주입을 해주면 된다.

# 그러면 어떤 것을 추상화 해야하는가?

- 변경하기 어려운 것

- 일찍, 올바르게 결정하고 싶은 것

Domain Model 패턴에서는, Domain의 관련된 추상화를 뽑아내다 보면 그것이 Domain Layer의 Architecture가 된다.

Domain Model은 If ... then ... else ... 와 같은 복잡한 로직을

객체간의 협력 구조로 변경하는 것이다. 여기서 다형성, 연관관계가 적용된다.

# 변경을 무서워하지 말자

위에서 다뤘던 모든 내용이 실제 애플리케이션을 작성하는 코드에 적용한다해서 단번에 해결되진 않는다.

변경과 리팩터링을 지속적으로 해야 이를 이루어낼 수 있고 이 행위가 반복되어야지 비로소 변경하기 쉬운 코드가 된다.

그렇기 때문에 객체지향적으로 리팩터링할 수 있는 설계를 해놓은 후 리팩터링해가는 것을 연습해아한다.
