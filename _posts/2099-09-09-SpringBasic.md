---

title: Spring 기본
date: 2022-09-09
categories: [Spring]
tags: [Bean, BeanFactory, ApplicationContext]
layout: post
toc: true
math: true
mermaid: true

---

# 1. 스프링 컨테이너와 Bean

# 1.1. 스프링 컨테이너란?

**스프링 컨테이너**는 **자바 객체의 생명 주기를 관리**하며, **생성된 자바 객체들에게 추가적인 기능을 제공**하는 역할을 수행한다.

여기서 말하는 자바 객체를 스프링에서는 빈(Bean)이라고 칭한다.

> IoC와 DI의 원리가 이 스프링 컨테이너에 적용된다.

### 1.1.1 제어 흐름을 외부에서 관리하는 IOC 역할을 수행한다.

**new 연산자**, **인터페이스 호출**, **팩토리 호출 방식**으로 **객체를 생성**하고 **소멸**시킬 수 있는데 **스프링 컨테이너가 이 역할을 수행**한다.

또한, **객체들 간의 의존 관계**를 스프링 컨테이너가 **런타임 과정에서 알아서 만들**어 준다.

### 1.1.2 의존성 주입, DI

생성자 주입 방식, setter 주입방식, 필드 주입방식 (@Autowired)를 통해 의존성 주입을 수행한다.

---

# 1.2. 스프링 컨테이너가 어떻게 생성이 되는가?

    ApplicationContext applicationContext = new AnnotationConfigApplicationContext(AppConfig.class);

**ApplicationContext** 를 **스프링 컨테이너**라고 한다.

위 코드는 Annotation 기반으로 한 생성방식인데,

AppConfig 라는 클래스 파일에 존재하는 설정 파일을 기본으로 하는 스프링 컨테이너를 생성한 내용이다.

AnnotationConfigApplicationContext.java 라는 클래스에서 스프링 컨테이너 생성을 위한 내용이 구현되어있다.

---

# 1.3. 스프링 컨테이너는 어떻게 생겼나?

### 1.3.1 스프링 컨테이너 內 Spring Bean 저장소

| Bean 이름       | Bean 객체               |
|---------------|-----------------------|
| memberService | MemberServiceImpl@x01 |
| userService   | UserServiceImpl@x02   |
| PostService   | PostServiceImpl@x03   |

위와 같이 스프링 컨테이너 내의, 여러 Bean 의 대한 정보를 담을 수 있는 구조가 있다.

    @Bean(name="CustomBeanName")

으로 Bean 이름을 지정할 수 있지만, 중복이 되면 안되기때문에 굳이 권장하진 않는다.

---

### 의존관계 주입 예시 코드

    @Bean
    public MemberService memberService() {
        return new MemberServiceImple(memberRepository());
    }

스프링은 Bean을 생성하고, 의존관계를 주입하는 단계가 나눠져있다.

위 예시처럼 Java 코드로 Bean을 등록하면, 생성자를 호출하면서 의존관계 주입이 처리가 된다.

> 쉽게 말하자면, memberService() 라는 메서드를 호출할 때 마다 이에 딸려있는 memberRepository 라는 객체를 계속 생성하는 것이다.
>
> memberService() 메서드를 1억번 호출하면, 생성자 호출(인스턴수 갯수) 또한 1억개...

---

# 2. 스프링 컨테이너에 등록된 모든 Bean 조회하기

    AnnotationConfigApplicationContext ac = new AnnotationConfigApplicationContext(AppConfig.class);

    @Test
    @DisplayName("모든 빈 출력하기")
    public void findAllBean() {
        String[] beanDefinitionNames = ac.getBeanDefinitionNames();

        // iter + tab
        for (String beanDefinitionName : beanDefinitionNames) {
            Object bean = ac.getBean(beanDefinitionName);

            // soutv
            System.out.println("name = " + beanDefinitionName + "object = " + bean);
        }
    }


## 코드 상세

- AnnotationConfigApplicationContext ac = new AnnotationConfigApplicationContext(AppConfig.class);

위 구문은, 스프링 컨테이너를 생성하는 코드이다.

- String[] beanDefinitionNames = ac.getBeanDefinitionNames();

위 구문은, 컨테이너에 정의된 Bean 이름을 모두 불러오는 내용이다.

- Object bean = ac.getBean(beanDefinitionName);

위 구문은 Bean이름에 해당하는 실제 Bean 내용(객체)를 불러오는 내용이다.

- System.out.println("name = " + beanDefinitionName + "object = " + bean);

위 구문은, Bean 이름과 Bean 실체를 출력하는 내용이다.

## 출력 결과

    name = org.springframework.context.annotation.internalConfigurationAnnotationProcessor object = org.springframework.context.annotation.ConfigurationClassPostProcessor@46b61c56
    name = org.springframework.context.annotation.internalAutowiredAnnotationProcessor object = org.springframework.beans.factory.annotation.AutowiredAnnotationBeanPostProcessor@2e48362c
    name = org.springframework.context.annotation.internalCommonAnnotationProcessor object = org.springframework.context.annotation.CommonAnnotationBeanPostProcessor@1efe439d
    name = org.springframework.context.event.internalEventListenerProcessor object = org.springframework.context.event.EventListenerMethodProcessor@be68757
    name = org.springframework.context.event.internalEventListenerFactory object = org.springframework.context.event.DefaultEventListenerFactory@7d446ed1
    name = appConfig object = hello.core.AppConfig$$EnhancerBySpringCGLIB$$aa7fa685@12d2ce03
    name = memberService object = hello.core.member.MemberServiceImpl@7e5c856f
    name = memberRepository object = hello.core.member.MemoryMemberRepository@2b175c00
    name = orderService object = hello.core.order.OrderServiceImpl@413f69cc
    name = discountPolicy object = hello.core.discount.RateDiscountPolicy@3eb81efb


위와 같이

**Spring 내부적으로 가지고 있는 Bean**과, **직접 생성한 Bean 이름 + Bean 객체의 실체 정보**를 조회할 수 있다.

---

# 2.1. 스프링 Bean 조회 시 상속관계

**부모 객체 :** Parent

**자식 객체 :** Child 1, Child 2, Child 3, Child 4, Child 5,

**손자 객체 :** Child 6 (Child1 을 상속받음)


다음과 같이 있다고 했을 때,

각 조회에 대한 출력 내용은 다음 표와 같다.

| 조회 대상 Bean | 조회되는 내용                                    |
|------------|--------------------------------------------|
| Parent     | Child 1, Child 2, Child3, Child 4, Child 5 |
| Child 1    | Child 1 Child 6                            |
| Child 6    | Child 6                            |

위 표에서 볼 수 있듯이, **상위 계층에 해당하는 Bean을 조회하면** 그것을 **상속받는 자식 클래스(하위 계층 Bean)은 모두 조회**된다.

그래서 **모든 자바 객체의 최고 계층인 Obejct 타입으로 조회**하면, **모든 Bean 을 조회**할 수 있다.

    @Test
    @DisplayName("부모 타입으로 조회 시, 자식이 둘 이상 있으면, 중복 오류가 발생한다")
    void findBeanByParentTypeDuplicate() {
        DiscountPolicy bean = ac.getBean(DiscountPolicy.class);
        assertThrows(NoUniqueBeanDefinitionException.class, () -> ac.getBean(DiscountPolicy.class));
    }

하지만 위처럼 일반적으로 부모타입으로 조회했을 때 그에 대한 하위 타입이 두 개 이상이라면 오류가 발생하게 된다.

    @Test
    @DisplayName("부모 타입으로 조회 시, 자식이 둘 이상 있으면, 중복 오류가 발생한다")
    void findBeanByParentTypeBeanName() {
        DiscountPolicy bean = ac.getBean("rateDiscountPolicy", DiscountPolicy.class);
    }

그렇기 때문에 특정 Bean 이름을 조회하는 구문으로 변경하면 성공적으로 Bean 조회를 마칠 수 있게 된다.

만약 부모타입에 해당하는 모든 Bean 을 조회하고 싶다면 다음과 같이 작성하면 된다.

    @Test
    @DisplayName("부모 타입으로 모두 조회하기")
    void findAllBeanByParentType() {
        Map<String, DiscountPolicy> beanOfType = ac.getBeansOfType(DiscountPolicy.class);
        assertThat(beanOfType.size()).isEqualTo(2);
    }

또는 앞서 말한 최상위 객체 타입으로, Spring 내부 객체를 포함한 모든 Bean을 조회하도록 하려면 아래와 같이 작성하면 된다.

    @Test
    @DisplayName("최상위 객체로 모두 조회하기")
    void findAllBeanByObjectType() {
        Map<String, Object> beanOfType = ac.getBeansOfType(Object.class);
        for (String key : beanOfType.keySet()) {
            System.out.println("key = " + key);
            System.out.println("beanOfType.get(key) = " + beanOfType.get(key));
        }
    }

---

# 3. BeanFactory(Interface) 와 ApplicationContext(Interface)

BeanFactory <- ApplicationContext <- AnnotationConfigApplicationContext


<- 는 부모라는 것을 의미한다. (BeanFactory를 ApplicationContext 가 부모로 삼는다.)

<br>

### 3.1. BeanFactory

스프링 컨테이너의 최상위 인터페이스이며, Bean 을 관리하고 조회하는 역할을 담당한다.

getBean() 메서드 등 Bean 에 관한 메서드를 지원한다.

### 3.2. ApplicationContext

BeanFactory 기능을 모두 상속받아서 제공한다.

## 3.3. 그러면 BeanFactory와 ApplicationContext를 굳이 분리해서 사용하는 이유는?

ApplicationContext 를 사용하는 이유는, BeanFactory 의 기능과 더불어 추가적인 기능을 사용할 수 있도록 하기 위해서이다.

조금 더 자세히 살펴보자면, ApplicatonContext 라는 인터페이스를 구현하는 MessageSource, EnvironmentCapable, ApplicationEventPublisher, ResourceLoader

라는 각 구현체들이 존재하는데, 이 구현체들을 통해 부가적인 기능을 이용하기 위함인 것이다.

## 3.4. 그래서 스프링 컨테이너는 BeanFactory, ApplicationContext 둘 중 무엇인가?

BeanFactory와 ApplicationContext를 스프링 컨테이너라고 한다.

스프링 컨테이너의 존재의 주요 목적은 Bean 관리이고, BeanFactory 와 ApplicationContext 두 객체 모두 그 역할을 수행할 수 있기 때문이다.

또한, 실제 개발을 하는데에 있어서 주로 ApplicationContext 를 사용하여 부가기능까지 사용하는게 일반적이다.

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

어라라 그러면 @Configuration 을 안붙이면 싱글톤이 깨지는건가?

-> 그렇다!

        if (memoryMemberRepository 가 이미 스프링 컨테이너에 등록되어 있다면?) {
            return 스프링 컨테이너에 있는 memoryMemberRepository 반환;
        }

        else if (memoryMemberRepository 가 스프링 컨테이너에 없다면?) {
            MemoryMemberRepository를 생성하고 스프링 컨테이너에 등록하고 반환한다.;
        }

@Configuration을 사용하지 않으면 위와 같은 조건문을 수행하지않고, 순수 Java 코드로 객체를 생성하는 내용으로 되기 때문이다.

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

---

# 0. Bean 생명주기와 콜백

## 들어가기 전

Spring Container (Application Context) 는 Bean 의 생성, 의존성 주입을 수행하는 역할을 한다.

이때, 각 애플리케이션 역할에 따라 다른 Spring Container 가 생성되고 사용된다.

| Spring Container 구조                |
|------------------------------------|
| ApplicationContext                 |
| ConfigurableApplicationContext     |
| AnnotationConfigApplicationContext |

Spring Container 로써 가장 잘 알고있는 ApplicationContext 는 ApplicationContext 와 LifeCycle 을 상속받은

ConfigurableApplicationContext 가 그 구현체를 가지고 있다.

## Spring Bean 의 생명주기

> 객체 생성 -> 의존관계 주입

생성자 주입 방식의 의존성 주입은 객체 생성 시점에서 다른 의존 객체를 넣어줘야하기 때문에 예외다.

하지만 필드주입 수정자주입 메서드 주입 등 그 외의 방식은 모두 위의 생명주기를 따라갈 뿐만 아니라

Bean, Component 어노테이션을 붙인 객체들 또한 위의 생명 주기를 가지고 있게 된다. (물론 이 설정 Bean 에서도 생성자 주입을 사용하면 그땐 예외)

## Bean 초기화

Bean 의 내용을 변경하는 초기화를 하고 싶으면,

Bean 객체를 생성하고 의존성 주입이 끝난 후에만 초기화 작업을 수행할 수 있다. 이 때 의존성 주입이 완료된 시점을 알 수 있는 방법은 콜백을 활용하는 것이다.

> Spring Container -> Spring Bean Creating -> DI -> Init/CallBack -> Using -> Destroy/CallBack -> Spring Close

싱글톤 환경에서의 전체적인 생명주기는 위와 같이 이루어진다.

## 왜 Bean 초기화를 하는가? 생성자 주입을 통해 Bean 등록을 하면 편한데.

생성자는 파라미터를 받고, 그에 대한 메모리를 할당받아 객체를 생성하는 책임을 가진다.

초기화는 생성된 객체를 가지고 외부와 메세지를 주고받으며 해당 객체의 역할을 수행할 수 있게 하는 작업을 수행한다. (DB Connection, Network Connection 등...)

그래서 초기화는 비용이 비싸다는 특징이 있고, 초기화가 간단한 내부 변경 식만 존재하는 것이 아니면

유지보수 측면에서 생성자/초기화를 분리하는 것이 좋다.

#### + 단일 책임 원칙 - 생성자는 객체 생성에 집중, 초기화는 초기화에만 집중

---

# 1. InitializingBean, DisposableBean

순서대로 각각, afterPropertiesSet(), destroy() 라는 메서드를 가진 각각의 인터페이스이다.

생성자 호출 후 의존관계 주입이 끝나면 afterPropertiesSet() 생성자 콜백메서드가 호출되고

Bean 을 다 사용한 상황이라면 destroy() 소멸자 콜백 메서드가 호출된다.

## 초기화, 소멸 인터페이스의 단점

다른 라이브러리에 포함된 Bean과 결합할 수 없다.

지금은 거의 사용하지 않는 방식이다.

---

# 2. Bean 등록 시 초기화/소멸자 지정

### BeanInitDestroyTest.java

    @Bean(initMethod = "init", destoryMethod = "close")
    public BeanInitDestroyTest() {
        TestObject to = new TestObject();
    }

### TestObject.java

    public class TestObject {

        public void init() {
            System.out.println("초기화 ㄱㄱ");
        }

        public void close() {
            System.out.println("초기화 ㄱㄱ");
        }
    }

객체에 직접 초기화/소멸자 에 관한 메서드를 만들고 @Bean 어노테이션에 옵션을 붙여주기만 하면 된다.

---

# 0. 빈 스코프란?

빈이 존재할 수 있는 범위를 뜻한다. 즉 어느 범위까지 빈으로써의 역할을 수행하는지를 나타내는 것이다.

### 스코프 종류

#### 싱글톤

기본 스코프로, 스프링 컨테이너가 생성되고 소멸될 때까지의 유지된다.

#### 프로토타입

빈의 생성과 의존관계 주입까지만 관여하고 더는 관리하지 않는 매우 짧은 범위의 유형이다.

#### 웹 관련 스코프

request, session, application


### 등록 방법

    @Scope("prototype")
    @Bean
    TestBean HelloBean() { return new HelloBean; }

    @Scope("prototype")
    @Bean
    TestComponent HelloComponent() { return new HelloComponent; }

---

# 1. 프로토타입 스코프와 싱글톤 스코프

### 싱글톤 빈 요청

![](https://user-images.githubusercontent.com/60564431/192430261-871031c5-9abf-4700-ba93-3b81f498e812.png)

싱글톤 스코프는 잘 알고 있듯이 이미 생성되어있는 Spring Container 의 Bean을 재사용 하는 것이다.

### 프로토타입 빈 요청/반환

![](https://user-images.githubusercontent.com/60564431/192430259-8ec503d4-5bf2-403b-95ff-04961507ccc5.png)

![](https://user-images.githubusercontent.com/60564431/192430252-a931283c-d878-489e-89f3-bd0ec732c162.png)

프로토타입은 싱글톤과 달리 Bean 요청 시 새로운 Bean 을 생성하고, 의존관계를 주입한 후 Spring Container에서 소멸된다.

---

> 프로토타입 Scope 는 어떤 상황에서 쓰이는지 더 알아보자.
