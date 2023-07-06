---

title: Spring 핵심원리 기본편 - 2. 스프링 컨테이너와 싱글톤 컨테이너
author: 김도현
date: 2022-09-10
categories: [Spring, Singleton]
tags: [Singleton, SingletonContainer]
layout: post
toc: true
math: true
mermaid: true

---

0. 스프링에서 싱글톤 패턴을 사용하는 이유
1. 싱글톤 패턴
2. 싱글톤 컨테이너
3. 싱글톤을 사용할 때 주의할 점
4. @Configuration 의 사건의 지평선
5. @Configuration 의 특이점

---

# 0. 스프링에서 싱글톤 패턴을 사용하는 이유?

스프링은 태생이 기업용 온라인 서비스 기술을 지원하기 위함이다.

그 말인 즉슨, 웹 애플리케이션에 특화되었다는 것인데, 웹 애플리케이션은 여러 클라이언트가 동시에 요청을 하는 경우가 잦다.

그런데 여기서, 여러 클라이언트가 동시에 호출할때마다 요청에 해당하는 객체들을 생성하게 된다면?

### 아래 코드로 참고해보자

    @Test
    @DisplayName("스프링 없는 순수한 DI 컨테이너")
    void pureContainer() {
        AppConfig appConfig = new AppConfig();

        // 1. 조회 : 호출될 때 마다 객체를생성
        MemberService memberService1 = appConfig.memberService();

        MemberService memberService2 = appConfig.memberService();

        System.out.println("memberService1 = " + memberService1);
        System.out.println("memberService2 = " + memberService2);

        assertThat(memberService1).isNotSameAs(memberService2);
    }

### 출력결과
    memberService1 = hello.core.member.MemberServiceImpl@3bbc39f8
    memberService2 = hello.core.member.MemberServiceImpl@4ae3c1cd


만약, 1000만명이 동시에 요청을 하는 상황에 놓였다고 생각했을때 객체를 1000만개 생성하고 소멸시키는 라이프사이클을 거쳐야한다.

위의 방식이 좋아보이는가? 그럴 수도 있을 수 있지만, 보편적인 상황에선 좋아보이지 않는다.

더 좋은 방법이 있을 것이다. -> 싱글톤 패턴(애플리케이션 런타임 동안 딱 하나의 인스턴스를 공유하여 사용하는 것)으로 해결해보자.

---

# 1. 싱글톤 패턴

객체 인스턴스를 2개 이상 생성하지 못하도록 통제한다.

이미 만들어진 객체를 **공유** 하여 사용하기 위한 패턴이다.

--> private 생성자를 사용하여 외부에서 임의로 new 키워드를 사용하지 못하도록 막아야한다.

### 1.1 싱글톤 패턴의 장점
이미 만들어진 객체를 사용하는 것이기 때문에, 효율성에서 이득을 가져갈 수 있다.

### 1.2. 싱글톤 패턴의 단점

#### 1.2.1. 싱글톤 패턴을 구현하는 코드가 길어진다.

    public static SingletonService getInstance() {}
    private SingletonService() {}

위 처럼, getInstance() 역할을 하는 메서드나, private 생성자를 일일히 다 추가해 줘야한다는 것이다.

#### 1.2.2. 의존관계에 있어서, 클라이언트가 구체 클래스에 의존한다.

#### 1.2.3. DIP 위반이다. (구체 클래스에 의존하는 것이 아니라 그 추상화에 의존해야한다. (인터페이스에 의존하라))

#### 1.2.4. OCP 위반이다. (기능추가는 가능하도록, 하지만 이미 있는 기능 변경에는 불가능하도록. (이 역시 인터페이스로 해결))

#### 1.2.5. 테스트가 어렵다.

#### 1.2.6. 유연성이 떨어진다. (DI 적용이 번거로움)

#### 1.2.7. 안티패턴으로도 불린다...

---

### 1.3. 싱글톤 패턴 예시코드

#### Dependency.java
    public class Dependency {
        private Dependency() {}
    }

#### Main.java

    public class Main {

        // 싱글톤 패턴에 의한 private 생성자를 적용해뒀기 때문에 컴파일 에러
        // private static final Dependency dependency = new Dependency();

        // 싱글톤 패턴에 의하여 이미 생성되어있는 객체를 사용하는 의미의 코드
        private static final Dependency dependency;
    }

#### SingletonTest.java

    @Test
    @DisplayName("싱글톤 패턴을 적용한 객체 사용")
    void singletonServiceTest() {
        SingletonService singletonService1 = SingletonService.getInstance();
        SingletonService singletonService2 = SingletonService.getInstance();

        System.out.println("singletonService1 = " + singletonService1);
        System.out.println("singletonService2 = " + singletonService2);

        assertThat(singletonService1).isSameAs(singletonService2);
    }

SingletonTest.java 코드를 보면, getInstance() 라는 메서드를 일일히 해줘야하는 점이 불편하게 느껴질 수 있다.

하지만, 스프링이 제공하는 싱글톤 컨테이너를 활용하면 저런 방식으로 사용하지 않아도 된다.

그에 대한 방식은 #2 섹션에서 예시 코드를 작성했다.

### 1.4. 스프링에서는 다르다! 스프링은 싱글톤 패턴을 위해 지원해주는 것들이 많다!

위에서 싱글톤에 대하여 간략하게 이야기했듯이 단점이 많은 패턴으로 알려져있다. 안티패턴으로도 불리기 까지 하는 패턴이지만

스프링에서는 위의 문제점을 해결하기위하여 **싱글톤 컨테이너를 제공**한다.

---

# 2. 싱글톤 컨테이너

스프링 컨테이너에는 싱글턴 패턴을 적용하지 않아도, 객체 인스턴스를 싱글톤으로 관리한다.

위의 언급한 역할을 수행하는 스프링 컨테이너가 싱글톤 컨테이너의 역할을 하는 것이고, 이렇게 **싱글톤 객체를 생성하고 관리하는 기능**을 **싱글톤 레지스트리** 라고 한다.

> 결론적으로, 싱글톤 컨테이너 기능으로 private 생성자 및 getInstance() 역할의 메서드를 추가하지 않아도 되는 것이다.

<br>

### 스프링 컨테이너의 싱글톤 컨테이너 기능 활용

    @Test
    @DisplayName("스프링 컨테이너와 싱글톤")
    void springContainer() {
        ApplicationContext ac = new AnnotationConfigApplicationContext(AppConfig.class);
        MemberService memberService1 = ac.getBean("memberService", MemberService.class);
        MemberService memberService2 = ac.getBean("memberService", MemberService.class);

        System.out.println("memberService1 = " + memberService1);
        System.out.println("memberService2 = " + memberService2);

        assertThat(memberService1).isSameAs(memberService2);
    }

### 출력 결과

    memberService1 = hello.core.member.MemberServiceImpl@5a3bc7ed
    memberService2 = hello.core.member.MemberServiceImpl@5a3bc7ed

같은 객체를 사용하는 결과를 볼 수 있다.

---

# 3. 싱글톤을 사용할 때 주의할 점

싱글톤은 하나의 객체 인스턴스를 공유하기 때문에 싱글톤 객체는 상태를 유지하게 설계하면 안된다.

이게 무슨말이냐면, 만약 UserService 라는 싱글톤 객체에 userId 라는 필드가 있다고 해보자.

그런데, UseDependencyClass1, UseDependencyClass2 에서 각각 싱글톤으로 사용중인 UserService의 userId 를 1, 100 으로 변경한다고 해보자

어떻게 되겠는가? 만약 다른 클래스가 UserService 객체를 사용할 때는 초기에 UserService에 지정된 내용을 사용하려고 해도

이미 다른곳에서 공유중인 클래스에 의해 변경이 되어있기때문에 사용하고자 하는 목적에 대한 코드를 사용하지 못하게 되는 것이다.

더 쉽게 예시 코드를 작성해 보겠다.

### UserService.java

    public class UserService {
        private int userId = 18018008;

        public void settingMyUserId(int userId) {
            this.userId = userId;
        }

        public void printUserId() {
            System.out.println(this.userId);
        }
    }

### UseDependencyClass1.java

    public class UseDependencyClass1 {
        ApplicationContext ac = new AnnotationConfigApplicationContext(AppConfig.class);

        UserService userService1 = ac.getBean("userService", UserService.class);

        userService1.settingMyUserId(1);
    }

### UseDependencyClass2.java

    public class UseDependencyClass2 {
        ApplicationContext ac = new AnnotationConfigApplicationContext(AppConfig.class);

        UserService userService2 = ac.getBean("userService", UserService.class);

        userService1.settingMyUserId(10);
    }

### UseDependencyClass3.java

    public class UseDependencyClass3 {
        ApplicationContext ac = new AnnotationConfigApplicationContext(AppConfig.class);

        UserService userService3 = ac.getBean("userService", UserService.class);

        userService3.printUserId();
    }

위 코드가 차례로 실행 된 것이라고 가정한 상황에서,

UseDependencyClass3.java 가 실행되었을 때 기댓값은 UserService.java 의 초깃값인 18018008 이다.

하지만, UseDependencyClass1.java 와 UseDependencyClass2.java 에서 필드값을 변경하게 되어 가장 마지막으로 변경한 10 이 출력되게 될 것이다.

## 3.1 그래서 이 문제점을 어떻게 해결해야 하는가?

무상태로 설계하여 특정 클라이언트가 값을 수정하지 못하게 하자.

코드적으로 말하면, 클래스 필드 변수를 놓지 말라는 것이다.

혹은 메서드 단위로 변수의 용도를 해결하도록 하는 방법을 고려하자.

### UserService.java

    public class UserService {
        // 이렇게 변수를 놓지말자!!!
        private int userId = 18018008;

        // 각 다른 객체마다 셋팅한 userId를 반환해보자
        public int settingMyUserId(int userId) {
            return userId;
        }

        public void printUserId() {
            System.out.println(this.userId);
        }
    }

### UseDependencyClass1.java

    public class UseDependencyClass1 {
        ApplicationContext ac = new AnnotationConfigApplicationContext(AppConfig.class);

        UserService userService1 = ac.getBean("userService", UserService.class);

        // user1 이 셋팅한 ID 값을 변수로 받는다.
        int user1Id = userService1.settingMyUserId(1);

        // user1 이 셋팅한 ID 값을 토대로 출력을 한다. (비즈니스 로직을 수행한다.)
        userService1.printUserId(user1Id);
    }

---

# 4. @Configuration 의 사건의 지평선

@Configuration 은 본질적으로 싱글톤을 유지하기 위해 존재한다고 볼 수 있다.

아래 코드를 먼저 살펴보자.

### AppConfig.java

    @Configuration
    public class AppConfig {

        @Bean
        public MemberService memberService() {
            System.out.println("AppConfig.memberService");
            return new MemberServiceImpl(memberRepository());
        }

        @Bean
        public MemberRepository memberRepository() {
            System.out.println("AppConfig.memberRepository");
            return new MemoryMemberRepository();
        }

        @Bean
        public OrderService orderService() {
            System.out.println("AppConfig.orderService");
            return new OrderServiceImpl(memberRepository(), discountPolicy());
        }

        @Bean
        public DiscountPolicy discountPolicy() {
            // return new FixDiscountPolicy();
            return new RateDiscountPolicy();
        }
    }

위 코드를 보면 다음과 같은 객체 관계 생성 코드가 보이게 된다.

> @Bean memberService -> new MemoryMemberRepository();
>
> @Bean orderService -> new MemoryMemberRepository();

memberService 가 MemoryMemberRepository() 라는 객체를 생성하고,

orderService 가 MemoryMemberRepository() 라는 객체를 생성하는데,

지금까지 다뤄왔던 싱글톤이 깨질 것 같다는 생각이 들지 않는가?

그럼 일단 직접 확인을 해보겠다.

### 싱글톤이 깨지는가?

    @Test
    void configurationTest() {
        ApplicationContext ac = new AnnotationConfigApplicationContext(AppConfig.class);

        MemberServiceImpl memberService = ac.getBean("memberService", MemberServiceImpl.class);
        OrderServiceImpl orderService = ac.getBean("orderService", OrderServiceImpl.class);

        MemberRepository memberRepository = ac.getBean("memberRepository", MemberRepository.class);

        MemberRepository memberRepository1 = memberService.getMemberRepository();
        MemberRepository memberRepository2 = orderService.getMemberRepository();

        System.out.println("memberService -> memberRepository = " + memberRepository1);
        System.out.println("orderService -> memberRepository = " + memberRepository2);
        System.out.println("memberRepository = " + memberRepository);

        Assertions.assertThat(memberRepository1).isSameAs(memberRepository2).isSameAs(memberRepository);
    }

### 출력 결과

    memberService -> memberRepository = hello.core.member.MemoryMemberRepository@7cbd9d24
    orderService -> memberRepository = hello.core.member.MemoryMemberRepository@7cbd9d24
    memberRepository = hello.core.member.MemoryMemberRepository@7cbd9d24

싱글톤이 안깨진다!! 어떻게 된 일일까? 결론적으로는 앞서 언급했듯이 @Configuration 덕분인 것이다.

그런데, 혹시 new MemoryMemberRepository() 구문 자체가 호출이 안된 것이라고 생각할 수 있으니 AppConfig 를 살짝 수정해서 돌려보자

### AppConfig.java

    @Configuration
    public class AppConfig {
        @Bean
        public MemberService memberService() {
            System.out.println("AppConfig.memberService 호출되었음");
            return new MemberServiceImpl(memberRepository());
        }

        @Bean
        public MemberRepository memberRepository() {
            System.out.println("AppConfig.memberRepository 호출되었음");
            return new MemoryMemberRepository();
        }

        @Bean
        public OrderService orderService() {
            System.out.println("AppConfig.orderService 호출되었음");
            return new OrderServiceImpl(memberRepository(), discountPolicy());
        }
    }

### 출력 결과

    AppConfig.memberService 호출되었음
    AppConfig.memberRepository 호출되었음
    AppConfig.orderService 호출되었음

    memberRepository = hello.core.member.MemoryMemberRepository@7cbd9d24
    discountPolicy = hello.core.discount.RateDiscountPolicy@1672fe87
    memberService -> memberRepository = hello.core.member.MemoryMemberRepository@7cbd9d24
    orderService -> memberRepository = hello.core.member.MemoryMemberRepository@7cbd9d24
    memberRepository = hello.core.member.MemoryMemberRepository@7cbd9d24

객체 생성 시 호출된 횟수를 보면 각 한 번 호출되는게 전부이다.

---

# 5. @Configuration 의 특이점

스프링 컨테이너는 싱글톤 레지스트리이다. 그렇기 때문에 Bean 이 싱글톤이 되도록 보장을 해주도록 되어있다.

### AppConfig 조회하기

    @Test
    void configurationDeep() {
        ApplicationContext ac = new AnnotationConfigApplicationContext(AppConfig.class);
        AppConfig bean = ac.getBean(AppConfig.class);

        System.out.println("bean = " + bean);
    }

### 출력 결과

    bean = hello.core.AppConfig$$EnhancerBySpringCGLIB$$837a649@291f18

보통 다른 Bean 을 조회했을때와 다르게 EnhancerBySpringCGLIB ... 등 부가정보가 들어있다.

즉, 내가 만든 것이 아니라, 스프링이 CGLIB 라는 Byte 코드를 조작하는 라이브러리를 활용하여

AppConfig 클래스를 상속받은 임의의 클래스를 만들고 그 클래스를 Bean으로 등록한 것이다.

> 더 쉽게 말하자면, AppConfig 내용을 담고있는 것을 복제한 후 그 바이트 코드를 조작하여 Bean으로 등록한 것이다.

위 과정이 싱글톤이 되도록 보장해주는 역할을 수행한다.

### AppConfig@CGLIB 예상 코드

    @Bean
    public MemberRepository memberRepository() {
        if (memoryMemberRepository 가 이미 스프링 컨테이너에 등록되어 있다면?) {
            return 스프링 컨테이너에 있는 memoryMemberRepository 반환;
        }

        else if (memoryMemberRepository 가 스프링 컨테이너에 없다면?) {
            MemoryMemberRepository를 생성하고 스프링 컨테이너에 등록하고 반환한다.;
        }
    }

조건문에 명세해놓았듯이, 객체를 호출할 때 해당 객체가 스프링 컨테이너에서 Bean 으로 등록되어있는지 부터 확인하는게 우선이다.

등록되어있다면 이미 등록된 Bean 을 반환을 하는 것이고

등록되어있지 않다면 객체 생성 후 Bean 등록을 하는 흐름인 것이다.

<br>

어라라 그러면 @Configuration 을 안붙이면 싱글톤이 깨지는건가?

-> 그렇다!

        if (memoryMemberRepository 가 이미 스프링 컨테이너에 등록되어 있다면?) {
            return 스프링 컨테이너에 있는 memoryMemberRepository 반환;
        }

        else if (memoryMemberRepository 가 스프링 컨테이너에 없다면?) {
            MemoryMemberRepository를 생성하고 스프링 컨테이너에 등록하고 반환한다.;
        }

@Configuration을 사용하지 않으면 위와 같은 조건문을 수행하지않고, 순수 Java 코드로 객체를 생성하는 내용으로 되기 때문이다.
