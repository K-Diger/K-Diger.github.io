---

title: Hibernate 란 무엇일까?
author: 김도현
date: 2022-07-25
categories: [Spring, Hibernate]
tags: [Hibernate]
layout: post
toc: true
math: true
mermaid: true

---

[이전 포스트 - JDBC 란?](https://diger-king.github.io/blog/JDBC)

---

# Hibernate 를 알아보기 전에

## JPA란 (Java Persistence API) 무엇인가 부터 알아보자

> JPA : Java ORM 기술에 대한 API 표준 명세를 말한다.

> 즉, JPA는 인터페이스 라고 생각하면된다.
>
> 이떄 JPA를 사용하다 보면 Hibernate를 많이 사용하게 되는데 Hibernate는 JPA의 구현체이다.
>
> Hibernate 이외에도 DataNucleus, EclipseLink 등 다양한 JPA 구현체가 존재한다.

---

# Hibernate 를 알아보자

[Hibernate 공식 유저 가이드](https://docs.jboss.org/hibernate/orm/6.1/userguide/html_single/Hibernate_User_Guide.html)

위 링크를 참고했다.

> @Entity @Id @GeneratedValue

예시로, 위와 같은 객체와 DB를 매핑시키기 위한 어노테이션들을 자주 접했을 것이다.

이것들 모두 JPA라는 인터페이스에 명시가 되어있는 것이고, 이 명시된 사항을 구현한 것이 Hibernate 이다.

---

# Hibernate 와 JPA

![image](https://docs.jboss.org/hibernate/orm/6.1/userguide/html_single/images/architecture/data_access_layers.svg)

위 그림은, **Java 코드로 작성된 객체 기반의 내용**이 **어떻게 DB와 소통을 하는 것 인지**를 나타내준다.

사용자는 **JPA라는 인터페이스**의 내용을 통해 DB에 접근하는 코드를 작성한다.

JPA 는 그 **인터페이스를 구현한 여러 구현체 중 Hibernate** 를 사용한다.

**Hibernate 는 사용자의 부름에 응하여** **미리 구현해둔 JPA 내용을 토대로**, **JDBC 를 이용한 DB와의 소통을 하며** **원하는 결과를 가져다 주는 역할**을 해준다.

---

# JPA 가 요구하는(명시하는, JPA라는 인터페이스가 담고있는) 내용의 다이어그램

![image](https://docs.jboss.org/hibernate/orm/5.4/userguide/html_single/images/architecture/JPA_Hibernate.svg)

### Session Factory
> A Thread-Safe 한 표현방법으로, 애플리케이션 도메인과 DB 모델간의 매핑을 해주는 표현 방법을 제공한다.
>
> Hibernate.Session으로 인한 Acts as a factory for org.hibernate.Session instances. The EntityManagerFactory is the JPA equivalent of a SessionFactory and basically, those two converge into the same SessionFactory implementation.


---

# 결론

우리가 자주 접하는 **JPA를 사용하는 과정**에서,

**JPA 는 우리에게 노예가 어떤 일을 할 수 있는 지를 적어놓은 것**이다.

실제로 **그 일을 수행하는 것은 Hibernate라는 노예가 열심히 만들어놓은 것**을 **이용**하게 되는 것이다.
