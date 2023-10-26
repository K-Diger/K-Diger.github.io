---

title: Spring Security 구조
date: 2022-08-27
categories: [Spring, Spring-Security]
tags: [Spring, Spring-Security]
layout: post
toc: true
math: true
mermaid: true

---

# 참고 자료

[Spring Docs](https://spring.io/guides/topicals/spring-security-architecture/)

---

# 들어가기 전

## 인증(Authentication) vs 인가(Authorization)

`인증(Authentication) : 요청한 유저가 누구인지 확인하는 과정`

`인가(Authorization) : 인증된 유저가 권한이 있는지 확인하는 과정`

## Principal + Credentials

`Principal`은 요청의 주체를 의미한다. 즉 인증을 수행하고, 인가를 수행해야하는 요청의 주체를 말한다.

주로 ID, Email 등 이 쓰인다.

`Credentials`은 Principal 의 자격을 증명할 수 있는 요소를 가리킨다.

주로 비밀번호가 쓰이며, 이 요소는 변경이 가능하다.

---

# Spring Security 구조

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2F4pcAz%2FbtqyXp7w7Fz%2F2aY0aSm9eJhRVYB1bsMLwK%2Fimg.png)
![](https://www.javainuse.com/series-2-2-min.jpg)

Spring Security 는 기본적으로 Filter 단위에서 동작한다.

즉, Spring Security 는 필터다.

---

# 1. HttpRequest

HTTP 요청이 들어오면, Authentication Filter 가 요청을 가로채어 요청에 대한 인증(Authentication) 을 진행한다.

---

# 2, 3. UsernamePasswordAuthenticationToken

요청으로 들어온 username, password 를 통하여 인증객체를 생성한다.

즉, 요청한 ID 혹은 Email 그리고 Password 를 통해 그 요청 조합에 대한 인증을 수행하는 단계이다.

UsernamePasswordAuthenticationToken 은 Authentication 이라는 인터페이스의 구현체이다.

---

# 4. Authentication Manager (Interface)

![](https://www.javainuse.com/series-2-9-min.JPG)

이전 단계에서 생성된 인증객체(Authentication Object) 를 통해 Authentication Manager 인증 관련 메서드를 호출할 수 있게 된다.

![](https://www.javainuse.com/series-2-13-min.JPG)

`Authentication Manager` 는 인터페이스 이며 이를 구현하는 실체는 `Provider Manager` 가 담당한다.

이 때, `Authentication Manager` 에게 입력되는 객체도 `Authentication Object` 로 들어올 뿐 만 아니라 인증을 수행한 뒤에도 반환
타입은 `Authentication Object` 이다.

![](https://www.javainuse.com/series-3-2-min.JPG)

위 표를 보면, 인증을 수행하기 전 Authentication 객체의 내용을 알 수 있다.

| 필드                 | 인증을 수행하기 전  | 인증을 수행한 후   |
|--------------------|-------------|-------------|
| Principal          | username    | userObject  |
| GrantedAuthorities | Not granted | CUSTOM_ROLE |
| Authenticated      | false       | true        |

---

# 5. Authentication Provider (Interface)

![](https://www.javainuse.com/series-2-10-min.JPG)

여러개의 구현체를 가질 수 있는 인터페이스이다.

Authentication Provider 의 구현체들은 authenticate 메서드를 구현해야한다.

![](https://www.javainuse.com/series-2-15-min.JPG)

이 과정에서, UserDetailsService 가 존재하는데, 실제로 **DB 에 있는 유저 정보를 꺼내서** **요청한 인증 객체**와 **DB 에서 가져온 인증객체**의 Credentials 를 비교한다.

![](https://www.javainuse.com/series-2-11-min.JPG)

---

# 6.1 UserDetailsService (Interface)

loadUserByUsername 를 가지고 있는 인터페이스이다.

username 이 String 형태로 오면, UserDetailsService 의 구현체로부터 DB에 username 에 해당하는 User 객체를 가져오는 역할을 수행하게 한다.

이 인터페이스 역시 여러개의 구현체를 가질 수 있게된다.

# 6.2 UserDetails (Interface)

Credentials 를 담고있는 user 객체를 가지고 있으며, 사용자의 Role (Spring Context 에서 사용하는 Role 임)을 가진다.

---
