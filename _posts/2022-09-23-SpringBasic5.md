---

title: Spring 핵심원리 기본편 - 5. Bean 생명주기/콜백

date: 2022-09-23
categories: [Spring, Bean]
tags: [Spring, Bean, LifeCycle, Callback]
layout: post
toc: true
math: true
mermaid: true

---

0. Bean 생명주기와 콜백
1. InitializingBean, DisposableBean
2. Bean 등록 시 초기화/소멸자 지정
3. @PostConstruct, @PreDestroy

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

# 3. @PostConstruct, @PreDestroy

