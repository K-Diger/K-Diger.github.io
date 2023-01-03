---

title: Spring 핵심원리 기본편 - 6. Bean 스코프
author: 김도현
date: 2022-09-23
categories: [Spring, Bean, BeanScope]
tags: [Spring, Bean, BeanScope]
math: true
mermaid: true

---

0. 빈 스코프란?
1. 프로토타입 스코프와 싱글톤 스코프

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
