---

title: Spring 핵심원리 기본편 - 1. 스프링 컨테이너와 Bean
author: 김도현
date: 2022-09-09
categories: [Spring]
tags: [Bean, BeanFactory, ApplicationContext]
math: true
mermaid: true

---

1. 스프링 컨테이너와 Bean
2. 스프링 컨테이너에 등록된 모든 Bean 조회하기
3. BeanFactory와 ApplicationContext

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
