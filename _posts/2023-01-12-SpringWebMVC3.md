---

title: Spring Web MVC - 서블릿 및 구조

date: 2023-01-12
categories: [Spring, MVC]
tags: [Spring, MVC]
layout: post
toc: true
math: true
mermaid: true

---

# 서블릿 컨테이너의 동작 방식

![img.png](https://github.com/K-Diger/K-Diger.github.io/blob/main/images/%EC%84%9C%EB%B8%94%EB%A6%BF%20%EC%BB%A8%ED%85%8C%EC%9D%B4%EB%84%88%20%EB%8F%99%EC%9E%91%20%EB%B0%A9%EC%8B%9D.png?raw=true)

1. **Spring Boot**가 실행되면 **내장 톰캣 서버**를 띄워준다.
2. 여기서 **톰캣 서버**는 그 내부에서 **서블릿 컨테이너**를 가지고 있다.
3. **서블릿 컨테이너**는 필요에 따른 **서블릿**을 생성해준다.
4. **서블릿**은 **Request**, **Response** **객체를 생성**하여 클라이언트의 요청/응답을 처리한다.

---

# HttpServletRequest

HTTP 요청은 생각보다 간단하게 생기지 않았다. 그래서 서블릿은 이걸 간단하게 쓸 수 있도록 해주는데

그 방법은 HttpServletRequest 객체를 제공하는 것으로 한다.

그외의 부가적인 기능도 다수 제공하는데

- HttpServletRequest는 HTTP 요청 시작부터 종료 전까지 유지되는 임시 저장소 기능 또한 사용하게 해준다.
- 세션 관리 기능을 제공한다.

---

# MVC 패턴이 등장한 이유

서블릿이나 JSP만으로 비즈니스 로직 + 뷰 + 렌더링까지 모두 처리하게 되면 너무 많은 역할을 수행하게되어 유지보수가 어려워진다.

이게 무슨말이냐면, 비즈니스 로직을 호출하는 부분에 변경이 발생해도 해동 코드를 손대야하고, UI를 변경할 일이 있어도 비즈니스 로직이 있는 파일을 수정해야한다.

또한 **변경 주기가 다른 내용은 분리하는 것이 유지보수에**도 좋다.

서블릿은 HTTP 요청과 응답을 처리하는 자바 코드를 실행시키는데 특화 되어있고

JSP는 사용자에게 보여주는 컨텐츠를 다루는데 특화되어 있다.

이 각 특징을 극대화 하기 위해 등장한 것이 MVC 패턴으로

Model : 뷰에 출력할 데이터를 담아두고, 뷰가 필요한 데이터를 모델에 담아서 전달해주는 것으로 비즈니스 로직과 데이터 접근을 몰라도 된다. (화면 렌더링에만 집중하는 계층)

View : 모델에 담겨있는 데이터를 바탕으로 화면을 그린다. (HTML을 생성하는 계층)

Controller : HTTP 요청을 받고 파라미터를 검증하고, 비즈니스 로직을 실행한다. 이를 통해 뷰에 전달할 결과 데이터를 조회하여 모델에 전달한다.

# MVC 패턴의 장점

뷰를 보여주는 JSP는 정말 뷰를 보여주는 것에만 집중할 수 있게 되었고 (모델에서 데이터를 꺼내서 JPS에 넣기만 하면 됨)

서블릿은 HTTP 요청/응답과 비즈니스 로직에 집중할 수 있게 되었다.

# MVC 패턴의 단점

## View로 이동하는 코드가 항상 중복된다.

```java
String viewPath = "/WEB-INF/vies/new-form.jsp";
```

prefix : /WEB-INF/views/

suffix : .jsp

위와 같은 중복된 내용이 항상 따라다녀야한다.

또한 JSP가 아닌, Thymeleaf 같은 다른 뷰로 변경하면 전체 코드를 다 변경해야한다.

## 사용하지 않는 코드 발생

```java
HttpServletRequest request, HttpServletResponse response
```

위와 같은 코드는 사용될 때도 있고 안될 때도 있다. 또한 HttpServletRequest 등과 같은 구현체에 대한 테스트는 매우 어렵다. 추상화가 필요하다.

## 공통 처리가 어렵다.

기능이 복잡해질수록 컨트롤러에서 공통으로 처리해야하는 부분이 증가한다.

공통 기능을 메서드로 묶으면 될 것 같지만 그 공통 기능 자체를 호출하는 코드가 중복이다.

이 문제를 해결하려면 컨트롤러 호출 전에 공통 기능을 처리해야한다.

Front Controller 패턴을 도입하면 이 문제를 해결할 수 있게 된다.

또한 스프링 MVC의 핵심도 Front Controller 을 따른 것이다.

# Front Controller 특징

- 프론트 컨트롤러 (서블릿)으로 클라이언트의 요청을 받는다.
  - 요청이 들어오는 입구를 하나로 하는 것이다.
- 프론트 컨트롤러가 요청에 맞는 컨트롤러를 호출한다. (Handler)
- 프론트 컨트롤러를 제외하면 다른 컨트롤러는 서블릿을 사용하지 않아도 된다.

# Spring Web MVC 와 Front Controller는 무슨 관계인가?

Spring Web MVC 환경에서 DispatcherServlet 이 Front Controller의 역할을 수행한다.

![img_1.png](https://github.com/K-Diger/K-Diger.github.io/blob/main/images/SpringMVC%EA%B5%AC%EC%A1%B0.png?raw=true)

또한 핸들러 어댑터가 있어야 Front Controller가 다양한 Controller 호출할 수 있게 된다.

핸들러 어댑터는 GOF 디자인 패턴 중 어댑터 패턴으로 구성되어 있다.

![img.png](https://github.com/K-Diger/K-Diger.github.io/blob/main/images/SpringMVCAdapter.png?raw=true)

핸들러 어댑터는 위와 같이 구성할 수 있는데

```java
boolean supports(Object handler)
```
이 구문에서 handler는 컨트롤러를 의미하며, 해당 어댑터가 해당 컨트롤러를 처리할 수 있는지 판단하는 메서드이다.

```java
ModelView handler(HttpServletRequest request, HttpServletResponse response, Object handler) {

    }
```
이 구문에서는 적절한 어댑터를 찾아 반환해주는 기능을 수행한다.

![img.png](https://github.com/K-Diger/K-Diger.github.io/blob/main/images/%EB%94%94%EC%8A%A4%ED%8C%A8%EC%B2%98%20%EC%84%9C%EB%B8%94%EB%A6%BF%20%EA%B4%80%EA%B3%84%EA%B5%AC%EC%A1%B0.png?raw=true)

# Spring Boot와 Dispatcher Servlet

스프링 부트 애플리케이션을 실행하면, 스프링 부트가 내장 톰캣을 띄우는 것과 동시에 Dispatcher Servlet을 띄운다.

그리고 이 때, 모든 경로에 대해서 URI 매핑을 수행한다.

하지만 만약 디스패처 서블릿이 아니라 커스텀한 서블릿을 만들고 적용하면 과연 어떤 서블릿이 적용되는 걸까?

그 결론은, 더 구체적인 경로를 지정한 서블릿이 우선순위가 높아진다. 따라서 모든 경로를 대상으로한 서블릿을 만든게 아니라면

커스텀한 서블릿이 먼저 수행된다.

# Dispatecher Servlet 동작 흐름

- WAS로 HTTP 요청이 들어와서 서블릿이 호출되면, service()메서드가 호출된다.
- Spring MVC 는 Dispatcher Servlet의 부모 객체인 FrameworkServlet에서 service()를 Override 해두었다.
- 결국 FrameworkServlet.service()가 실행되면 DispatcherServlet.doDispatch() 메서드가 호출된다.
  - 이 부분이 가장 중요한데, 핸들러를 찾고 컨트롤러를 호출하기 위한 Root 과정이다.
  - doDispatcher() 메서드의 흐름을 살펴보면 다음과 같다.
  - 핸들러 조회(컨트롤러 조회) -> 핸들러 어댑터 조회 -> 핸들러 어댑터 실행 -> 핸들러 어댑터에 의한 핸들러 실행 -> ModelAndView 반환
- 여기서 핸들러 조회 과정에서는 요청 URI 뿐만 아니라, HTTP 스펙에 담겨있는 다양한 헤더 정보 또한 활용된다.
