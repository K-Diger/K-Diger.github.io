---

title: Spring 핵심원리 기본편 - 4. 의존성 주입

date: 2022-09-16
categories: [Spring, DI]
tags: [Spring, DI]
layout: post
toc: true
math: true
mermaid: true

---

0. 의존관계 주입 방법의 종류
1. 생성자 주입
2. 수정자 주입
3. 필드 주입
4. 일반 메서드 주입
5. 권장 의존성 주입 방식은?

---

# 0. 의존관계 주입 방법의 종류

1. 생성자 주입
2. Setter 주입
3. 필드 주입
4. 일반 메서드 주입

---

# 1. 생성자 주입

생성자를 통해서 의존관계를 주입 받는 방법이다.

## 특징

생성자 호출점에 딱 한 번만 호출되는 것이 보장된다.

불변, 필수적인 의존관계에 사용된다.

> 의존성을 주입받은 내용을 변경하지 않아야 할 때, 그리고 해당 의존성이 필수인 클래스에서 사용하면 된다.

## 생성자 주입 방식 - 기본

### OrderServiceImpl.java

    @Component
    public class OrderServiceImpl implements OrderService {
        private MemberRepository memberRepository;
        private DiscountPolicy discountPolicy;

        @Autowired
        public OrderServiceImpl(MemberRepository memberRepository, DiscountPolicy discountPolicy) {
            this.memberRepository = memberRepository;
            this.discountPolicy = discountPolicy;
        }
    }

## 생성자 주입 방식 - 개선

생성자가 한 개만 있다면(생성자 오버로딩이 필요없는 경우라면) @Autowired 어노테이션을 생략해도 된다.

생성자가 한 개라면 그냥 유일한 생성자를 실행하여 의존관계를 주입하면 되기 때문이다.

생성자가 여러 개라면, 어떤 생성자를 써야할지 모르기 때문에 @Autowired 로 처리를 해줘야하는 것이다.

**하지만** Bean 등록이 반드시 되어있어야한다!!

### OrderServiceImpl.java

    @Component
    @RequiredArgsConstructor
    public class OrderServiceImpl implements OrderService {
        private final MemberRepository memberRepository;
        private final DiscountPolicy discountPolicy;
    }

---

# 2. Setter 주입

Spring Container 의 Life Cycle

1. Spring Bean 등록
2. 연관관계 주입 (Autowired 가 걸린 의존관계 명세를 매핑시켜준다.)

Setter 주입에서는 해당 Setter 메서드에 @Autowired 어노테이션이 필수이다.

## 특징

변경 가능성, 선택적인 의존관계에 사용된다.

> 자바빈 프로퍼티
>
> Class 내부의 필드를 직접 접근하지 않고, 메서드(getter, setter) 로 접근하라는 것이 규약이였기 때문에 Setter 주입이 탄생했다.

### OrderServiceImpl.java

    @Component
    public class OrderServiceImpl implements OrderService {
        private MemberRepository memberRepository;
        private DiscountPolicy discountPolicy;

        @Autowired
        public void setMemberRepository(MemberRepository memberRepository() {
            this.memberRepository = memberRepository;
        }

        @Autowired
        public void setDiscountPolicy(DiscountPolicy discountPolicy() {
            this.DiscountPolicy = discountPolicy;
        }
    }

---

# 3. 필드 주입

### OrderServiceImpl.java

    @Component
    public class OrderServiceImpl implements OrderService {

        @Autowired
        private MemberRepository memberRepository;

        @Autowired
        private DiscountPolicy discountPolicy;
    }

코드가 간결하고 좋아보인다!

하지만 IDE 자체에서도 권장하지 않은 방법으로 지정되어있다. 왜일까?

## 필드주입이 안티패턴인 이유

### OrderServiceImpl.java

    @Component
    public class OrderServiceImpl implements OrderService {

        @Autowired
        private MemberRepository memberRepository;

        @Autowired
        private DiscountPolicy discountPolicy;

        @Override
        public Order createdOrder(Long memberId, String itemName, int itemPrice) {
            Member member = memberRepository.findById(memberId);
            int discountPrice = discountPolicy.discount(member, itemPrice);

            return new Order(memberId, itemName, itemPrice, discountPrice);
        }
    }

### fieldInjectionTest.java

    @SpringBootTest
    public class fieldInjectionTest {

        @Autowired
        private MemberRepository memberRepository;

        @Autowired
        private DiscountPolicy discountPolicy;

        @Test
        void fieldInjectionTest() {
            // 필드 주입으로 작성된 OrderService를 사용하기 위해 생성자 호출
            OrderServiceImpl orderService = new OrderServiceImpl();

            // OrderService의 기능을 테스트해보고 싶은데
            // 그 필드에 주입되어있는 의존성들(MemberRepository, DiscountPolicy)이 의존성 매핑이 되어있지 않아서
            // 테스트가 실패하게 된다.
            orderService.createOrder(1L, "itemA", 10000);
        }
    }

그러니까 OrderService 를 테스트를 수행하려해도

OrderService 에서 사용하는 MemberRepository, DiscountPolicy 를 주입해줘야하는데

필드 주입이 되어있다면 불가능하다.

그러므로 아래 두 가지 경우가 아니라면 진지하게 고려해봐야하는 방법이다.

1. 애플리케이션의 실제 코드와 관계 없는 테스트 코드
2. 스프링 설정을 목적으로하는 @Configuration 클래스

---

# 4. 일반 메서드 주입 [!]

### OrderServiceImpl.java

    @Component
    public class OrderServiceImpl implements OrderService {

        private MemberRepository memberRepository;

        private DiscountPolicy discountPolicy;

        @Autowired
        public void init(MemberRepository memberRepository, DiscountPolicy discountPolicy) {
            this.memberRepository = memberRepository;
            this.discountPolicy = discountPolicy;
        }
    }


메서드 단위마다 의존성 주입을 해주는 방식이다.

한번에 여러 필드를 주입받을 수 있는게 장점이지만

잘 사용하지 않는 방법이다. -> 왜 일까?

---

# 5. 권장 의존성 주입 방식은?

## 생성자 주입 방식이 권장되는 DI 방식이다.

애플리케이션 내의 의존성은 종료 시점까지 그 관계를 변경할 일이 거의 없을 뿐만 아니라 변하면 안된다.

이때, 생성자 주입 방식은 불변이라는 특징을 가지고 있다.

무슨 말이냐면, 생성자 주입은 객체를 생성할 때 딱 한 번만 호출되므로 이후에는 호출될 일이 없다.

게다가 객체를 생성한 후 싱글톤을 보장하는 스프링 컨테이너가 있기 때문에 절대 변하지 않는다.

또한 생성자 주입 방식을 활용할 때는 다음과 같은 final 키워드를 활용할 수 있게 된다.

### OrderServiceImpl.java - 정상적인 코드

    @Component
    public class OrderServiceImpl implements OrderService {

        private final MemberRepository memberRepository;

        public OrderServiceImpl(MemberRepository memberRepository) {
            this.memberRepository = memberRepository;
        }
    }

### OrderServiceImpl.java - 컴파일 에러

    @Component
    public class OrderServiceImpl implements OrderService {

        private final MemberRepository memberRepository;

        public OrderServiceImpl(MemberRepository memberRepository) {
        }
    }

이게 왜 장점이냐 하면, final 키워드는 초기화를 해주지 않으면 컴파일 에러가 발생하게 된다.

그러니까 실수로 생성자에 의존성 주입을 해주지 않았다면 컴파일 에러로 의존성 주입이 누락됐다는 것을 알 수 있게 해주는 것이다.

또한 final 키워드의 가장 원초적인 존재 목적답게 절대 수정할 수 없는 값이 되므로 불변성을 추가로 보장한다.


## Setter 주입은 변경 가능성을 열어둔다.

setDependency() 라는 메서드를 public 으로 열어두어야 수정자 주입이 가능하게 된다.

앞서 말했듯이 의존관계는 변경되지 않아야 하는데, 변경하면 안되는 내용을 public 으로 열어두면 문제가 생기게 된다.

---

# 6. 생성자 주입 + 롬복

### OrderServiceImpl.java - 롬복 적용

    @Component
    @RequiredArgsConstructor
    public class OrderServiceImpl implements OrderService {

        private final MemberRepository memberRepository;
    }

롬복이 제공하는 @RequiredArgsConstructor 를 사용하면, 위에서 작성했던 생성자 코드가 없어도 의존성 주입이 완료된다.

@RequiredArgsConstructor 는 **final 키워드**가 붙은 **필수값에** 대하여 생성자를 컴파일 시점에서 자동으로 만들어 주는 어노테이션이다.

---

# 7-1. Bean Type 이 2개 일 때의 의존성 주입

### FixDiscountPolicy.java

    @Component
    public class FixDiscountPolicy implements DiscountPolicy() {}

### RateDiscountPolicy.java

    @Component
    public class RateDiscountPolicy implements DiscountPolicy() {}

### OrderServiceImpl.java

    @Component
    @RequiredArgsConstructor
    public class OrderServiceImpl implements OrderService {

        private final MemberRepository memberRepository;
        private final DiscountPolicy discountPolicy;
    }

위와 같이 DiscountPolicy 라는 상위 타입을 구현한 두 개 이상의 하위 타입이 있다고 했을 때,

위와 같은 의존성 주입 코드는 빌드 에러가 발생하게 된다.

DiscountPolicy 에 해당하는 Bean 을 주입하려하는데, 이를 구현한 Bean 이 2개나 되어서 어떤 것을 주입해줘야할지 판단할 수 없기 때문이다.

그럼 어떻게 해결해야할까?

### 방법 1. 수동 클래스 주입

    @Component
    @RequiredArgsConstructor
    public class OrderServiceImpl implements OrderService {

        private final MemberRepository memberRepository;
        private final FixDiscountPolicy fixDiscountPolicy;
    }

위와 같이 하위 타입으로 의존성을 주입할 수 있겠지만, DIP를 위반하게 된다. (인터페이스가 아닌 구현체에 의존하고 있는 것이기 때문이다.)

### 방법 2. Autowired 필드명

    @Autowired
    private final DiscountPolicy fixDiscountPolicy;

    @Autowired
    private final DiscountPolicy rateDiscountPolicy;

위와 같이, 필드명을 사용하고자 하는 하위 계층으로 선언한다.

@Autowired 는 Type 을 기준으로 매칭을 시도한다. 하지만 위 상황처럼 타입이 여러개 일 때는, 파라미터 이름으로 빈 이름을 추가한다.

### 방법 3. @Qualifier


### 방법 4. @Primary


---

# 7-2. 어노테이션을 직접 만들어서 해결하기

방법 3. 에서 제시한 @Qualifier 는 어노테이션 파라미터에 문자열을 입력하는 방식이기 때문에 컴파일 에러를 발견할 수 없다.

    @Qualifier("mianDiscountPolicy)
    public class RateDiscountPolicy implements DiscountPolicy {}

    @Qualifier("mainDiscountPolicy)
    public class RateDiscountPolicy implements DiscountPolicy {}

위 두 코드를 예시로, 실제 운영중인 애플리케이션 에서는 발견하기 힘들 수도 있다..

사소한 실수를 방지하고자 어노테이션을 따로 만들어 주고 @Qualifier 어노테이션의 기능을 확장하면 된다.

### MainDiscountPolicy.java (어노테이션)

    @Target({ElementType.FIELD, ElementType.METHOD, ElementType.PARAMETER, ElementType.TYPE, ElementType.ANNOTATION_TYPE})
    @Retention(RetentionPolicy.RUNTIME)
    @Inherited
    @Documented
    @Qualifier("mainDiscountPolicy")
    public @interface MainDiscountPolicy {

    }

### RateDiscountPolicy.java

    @MainDiscountPolicy
    public class RateDiscountPolicy implements DiscountPolicy {}

