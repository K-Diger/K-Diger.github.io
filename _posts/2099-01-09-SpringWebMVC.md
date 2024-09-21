---

title: Spring Web MVC에 관하여

date: 2023-01-09
categories: [Spring, MVC]
tags: [Spring, MVC]
layout: post
toc: true
math: true
mermaid: true

---

# 웹 서버란?

HTTP 기반으로 동작하고 정적 리소스를 제공하는 것에 부가기능을 처리할 수 있는 서버를 말한다.

예시로는 NGINX, APACHE가 있다.

---

# 웹 애플리케이션 서버란? (WAS)

HTTP 기반으로 동작하고, 웹 서버 기능을 포함하여 정적 리소스를 제공하는 것 뿐만 아니라, 프로그램 코드를 실행하여 애플리케이션 로직을 수행할 수 있다.

동적 HTML, HTTP API(JSON), 서블릿, JSP, 스프링 MVC가 웹 애플리케이션 서버에서 동작한다.

예시로는 톰캣, Jetty, Undertow 등이 있다.

---

# WAS가 웹 애플리케이션에 최적인 것 같은데 그럼 WAS 하나로만 다 해결이 되나?

WAS 하나로는 버겁다. 과부화가 발생할 확률이 굉장히 높다.

WAS가 HTML,CSS,JS 같이 단순 정적 컨텐츠를 내려주는 역할은 금방 할 수 있지만

비즈니스 로직을 처리하고 DB와 소통하는 기능은 꽤나 비용이 큰 작업이다.

그렇기 때문에 WAS 앞단에, WS 를 배치하여 앞서 언급한 정적 컨텐츠들을 내려줄 수 있도록 전담하는 구성하고 WAS는 비즈니스 로직에만 전담하는 것이 좋고

정적 리소스가 많이 요구되면 WS만 증설하고 애플리케이션 리소스가 많이 요구되면 WAS를 증설하는 등 유연한 확장이 가능해지는 모델이 될 수 있다.

---

# 서블릿이란? 그리고 이건 왜 필요한거야??

![img.png](https://github.com/K-Diger/K-Diger.github.io/blob/main/images/%EC%84%9C%EB%B2%84%EA%B0%80%ED%95%B4%EC%95%BC%ED%95%A0%EC%9D%BC.png?raw=true)

위 그림과 같이 실제로 웹 애플리케이션 서버가 해줘야하는 일은 저렇게 산더미만큼 있다.

실제로 개발을 해보면 알겠듯이 우리는 비즈니스 로직을 만드는데에도 몸이 부족하다고 느껴질만큼 비용이 소요된다.

근데 여기서 비즈니스 로직을 작성하는 외의 웹 애플리케이션 서버가 해야하는 모든 기능을 만들어야한다면 어떻게 될까?

이러한 의문에서 나오는 해결책이 **서블릿** 이라는 개념이다.

## 서블릿 맛보기
```java
@WebServlet(name = "helloServlet", urlPatterns = "/hello")
public class HelloServlet extends HttpServlet {
    @Override
    protected void service(HttpServletRequest request, HttpServletResponse response) {
        // 비즈니스로직
    }
}
```
위 코드가 서블릿의 어느 조각이다. HTTP 요청/응답을 코드 단에서 편리하게 다룰 수 있도록 해주는 내용이 가장 와닿는 장점일 것이다.

하지만 HTTP 스펙을 아예몰라선 좀 그렇다 당연하게도 HTTP 배경지식은 꽤나 깔고 들어가야 더 정교한 웹 애플리케이션 서버를 만들 수 있다.

---

# 서블릿 컨테이너란?

앞서 간단하게 작성한 서블릿 코드들을 관리해주는 컨테이너로, 특정 요청이 들어오면 적합한 서블릿에 매핑해주며 생명주기까지 관리해주는 역할을 한다.

물론 우리가 직접 이 컨테이너를 개발할 필요는 없다.

서블릿 컨테이너는 객체 생성 + 초기화 + 호출 + 종료의 생명주기를 관리한다.

또한 서블릿 객체는 싱글톤으로 관리되는데, 요청이 올 때마다 서블릿객체를 생성해준다는 것 자체가 너무나도 비효율적이기 때문이다.

따라서 최초 로딩 시점에 서블릿 객체를 미리 만들어서 재활용한다. 그리고 모든 요청은 동일한 서블릿 객체 인스턴스에 접근한다.

서블릿은 동시 요청에 대한 멀티 쓰레드 처리를 지원한다!

![img.png](https://github.com/K-Diger/K-Diger.github.io/blob/main/images/%EC%84%9C%EB%B8%94%EB%A6%BF%EC%9D%84%ED%98%B8%EC%B6%9C%ED%95%98%EB%8A%94%EA%B2%83%EC%9D%80.png?raw=true)

정리하자면 서블릿의 도움을 받아 손쉽게 웹 애플리케이션과 클라이언트간의 HTTP 통신을 할 수 있게되는 것이다.

근데 서블릿은 도대체 누가 호출할까?

---

# 쓰레드

바로 앞서 말한 궁금증의 답은 쓰레드이다. 쓰레드가 서블릿을 호출한다.

쓰레드란, **애플리케이션 코드를 하나하나 순차적으로 실행**하는 것으로 Java Main Method를 실행하면 main이라는 이름의 쓰레드가 실행된다.

쓰레드가 없으면 Application이 동작할 수 없으며, 쓰레드는 한번에 하나의 코드 라인만 수행하는 특징이 있어 동시 처리가 필요하다면 쓰레드를 추가적으로 생성해야한다.

## 매 요청마다 쓰레드를 생성하는 것이 정말 좋을까?

### 장점

동시 요청을 처리할 수 있게 되고

서버의 리소스가 허용할 때 까지 처리가능하며

하나의 쓰레드가 지연되고 있어도 나머지 쓰레드는 정상적으로 동작한다.

### 단점

쓰레드는 생성 비용 자체는 매우 비싸다.

요청이 올 때마다 쓰레드를 생성하면 응답 속도가 늦어진다.

쓰레드는 컨텍스트 스위칭 비용이 발생한다.

쓰레드 생성에 제한이 없어 서버의 리소스를 초과할 수 있다.

장/단점을 비교해봤으니 결정을 내리면 된다. 과연 쓰레드를 매 요청마다 생성하는게 좋을까? 웬만한 웹 애플리케이션 서버에서는 절대 아니라고 본다.

---

# 쓰레드 풀

앞서 말한대로 쓰레드를 매 요청마다 생성하는거는 WAS의 니즈에 맞지 않는다.

따라서 이 방법외로 멀티 쓰레드를 제공할 수 있는 방법으로 미리 쓰레드를 만들어 놓은 후 필요할 때마다 가져가서 사용하는 방식이 있는데

이 미리 만들어진 쓰레드를 보관하는 곳을 쓰레드 풀이라고 한다.

무한정으로 다중 쓰레드를 수용할 순 없으나 그 쓰레드 양은 일반적인 WAS 규모에서는 충분하다.

만약에 쓰레드 풀에 남은 쓰레드가 없다면 요청을 거절하거나, 특정 시간만큼 대기하도록 조치하는 방식을 사용한다.

---

# 쓰레드 풀 알아야 할 것.

WAS의 주요 튜닝 포인트는 최대 쓰레드 수이다.

최대 쓰레드 수를 낮게 설정하면, 동시 요청에 대해서 서버 리소스(CPU, RAM)는 여유롭지만 요청 처리는 너무나 느려진다.

최대 쓰레드 수를 높게 설정하면, 동시 요청에 대해서 서버 리소스가 버티지 못하고 종료된다.

장애 발생 시 클라우드 기반의 WAS라면 서버부터 늘리고, 이후에 튜닝을 해야한다.

쓰레드 풀의 적정 숫자는 어떻게 알 수 있을까?

애플리케이션 로직의 복잡도, CPU, Memory 리소스에 따라 모두 다르기 때문에

Apache ab, JMiter, nGrinder 와 같은 성능 테스트 툴을 이용하여 측정하는 것이 좋다.

# WAS가 멀티 쓰레드를 알아서 해준다!

다만 개발 중에 공유변수 및 싱글톤 객체를 주의해서 사용해야함을 꼭 잊지말자.

---

# 서블릿 컨테이너의 동작 방식

![img.png](images/서블릿 컨테이너 동작 방식.png)

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

---

# 과거 스프링에서 사용했던 Controller의 모습은?

```java
public interface Controller {
    ModelAndView handleRequest(HttpServletRequest request, HttpServletResponse response) throws Exception;
}
```

코드를 보면 알 수 있듯이 ModelAndView 타입을 반환하는 인터페이스에 서블릿 요청/응답을 매개변수로 받아 컨트롤러로써 사용했다.

그래서 Controller 인터페이스를 상속받아 실제 Controller 간단하게 구현하면 다음과 같다.

```java
public class RealController implements Controller {
    @Override
    public ModelAndView handleRequest(HttpServletRequest request, HttpServletResponse response) throws Exception {
        return null;
    }
}
```

# 요청과 응답 과정 (Handler, HandlerAdapter)

0. 요청은 Dispatcher Servlet이 관제한다. 요청이 들어오면 아래와 같은 순서로 요청에 대한 응답을 처리한다.
1. Dispatcher Servlet이 특정 요청을 처리할 수 있는 Handler를 찾는다. (Handler Mapping)
2. Dispatcher Servlet이 Handler를 찾았으면 이 Handler를 다룰 수 있는 Handler Adapter를 찾는다.
3. Dispatcher Servlet이 Handler Adapter를 찾았으면 본격적으로 해당 Handler Adapter를 호출한다.
4. Handler Adapter는 자기가 다룰 수 있는 Handler를 호출한다.
5. Handler 실행 결과는 Dispatcher Servlet에게 ModelAndView를 전달하는 것으로 한다.
6. Dispatcher Servlet은 반환 받은 ModelAndView를 ViewResolver에게 전달하여 View를 반환받도록 한다.
7. View를 전달받은 Dispatcher Servlet은 Model를 호출하여 HTML응답으로 요청자에게 결과를 반환한다.

# 나는 Handler, Handler Adapter 이런 것들을 등록한 적이 없는데 어떻게 되는걸까?

Spring Boot기반에서는 자동으로 특정 Handler Mapping과 Handler Adapter를 구현해두었기 때문에 우리는 이를 자동으로 사용하고 있는 것이다.

### 주요 Handler Mapping의 종류

0 = RequestMappingHandlerMapping : 애노테이션 기반의 컨트롤러인 @RequestMapping에서 사용되는 Handler Mapping이다.

1 = BeanNameUrlHandlerMapping : 스프링 Bean의 이름으로 핸들러를 찾는다.

우선순위는 좌항에 적은 숫자와 같으며, @RequestMapping 어노테이션이 붙은 핸들러 중 적절한 Handler가 없으면 다음 우선순위를 탐색한다.

### 주요 Handler Adpater의 종류

0 = RequestMappingHandlerAdapter : 애노테이션 기반의 컨트롤러인 @RequestMapping에서 사용되는 Adapter이다.

1 = HttpRequestHandlerAdapter : HttpRequestHandler 처리

2 = SimpleControllerHandlerAdapter : Controller 인터페이스

마찬가지로 우선순위는 좌항에 적은 숫자와 같으며, 적절한 Handler Adpater가 없으면 다음 우선순위를 탐색한다.

# 요청과 응답 과정 흐름 (코드적으로 살펴보기)

### 1. 핸들러 매핑으로 핸들러 조회

- HandlerMapping을 실행하여 핸들러를 찾는다.
- 찾아낸 핸들러인 RealController를 반환한다.

### 2. 핸들러 어댑터 조회

- HandlerAdapter의 supports()를 순서대로 호출한다. (여기서 순서란 위에서 언급한 Handler Adapter의 종류를 순서대로 호출한다는 것)
- SimpleControllerHandlerAdapter가 Controller 인터페이스를 다룰 수 있으므로 SimpleControllerHandlerAdapter를 반환한다.

### 3. 핸들러 어댑터 실행

- Dispatcher Servlet이 SimpleControllerHandlerAdapter를 실행하며, 어댑터에 핸들러(Controller)의 정보 또한 넘겨준다.
- SimpleControllerHandlerAdapter는 실행할 대상인 핸들러(RealController)를 실행하고 결과를 반환한다.

> 우리는 주로 @RequestMapping을 사용한다! 이게 바로 RequestMappingHandlerMapping, RequestMappingHandlerAdapter를 줄인 것이다.

# 뷰 리졸버는 어떻게 사용하는가?

```java
public class RealController implements Controller {
    @Override
    public ModelAndView handleRequest(HttpServletRequest request, HttpServletResponse response) throws Exception {
        return new ModelAndView("new-form");
    }
}
```
위와 같이 반환하고자 하는 View파일 명을 넣어 ModelAndView객체를 생성하여 반환하면 된다.

또한 프로젝트 전역에 ViewResolver를 등록해줘야하는데 application.properties 혹은 application.yml에 다음과 같이 설정정보를 입력해주면 된다.

```properties
spring.mvc.view.prefix=/WEB-INF/views/
spring.mvc.view.suffix=.jsp
```

# 그럼 뷰 리졸버는 어떻게 동작하는가?

Spring Boot는 위와 같은 properties 설정 정보로 InternalResourceViewResolver 라는 뷰 리졸버를 등록한다.

```java
@Bean
InternalResourceViewResolver internalResourceViewResolver() {
    return new InternalResourceViewResolver("/WEB-INF/views/", ".jsp");
}
```
실제로는 이러한 Bean을 등록해줘야하지만 Spring Boot가 해주는 것이다.

### 스프링 부트가 자동으로 등록하는 뷰 리졸버 종류

1 = BeanNameViewResovler : 빈 이름으로 뷰를 찾아서 반환한다.
2 = InternalResourceViewResolver : JPS를 처리할 수 있는 뷰를 반환한다.

결국에는 별거 없다. 뷰 리졸버를 통해서 특정 뷰를 처리할 수 있는 리졸버를 찾고 그에 따른 뷰를 반환해주는 것이다.

---

# 우리가 자주 사용하던 @Controller의 역할은 어떤 것이 있을까?

1. Component Scan의 대상이 될 수 있다.
2. Spring MVC로부터 애노테이션 기반의 컨트롤러라고 인식하게 할 수 있다.

즉, RequestMappingHandlerMapping은 @RequestMapping 혹은 @Controller가 클래스에 달려있는 경우에 처리하는데 이를 인식할 수 있도록 하는 것이다.

또한 이러한 기능 외의 편리한 기능을 제공하는데 이를 코드로 살펴보면 다음과 같다.

```java
@Controller
@RequestMapping("/mvc")
public class SpringMvcControllerV2 {
    private MemberRepository memberRepository = MemberRepository.getInstance();

    @RequestMapping("/new-form")
    public ModelAndView newForm() {
        return new ModelAndView("/new-form");
    }
}
```

```java
@Controller
@RequestMapping("/mvc")
public class SpringMvcControllerV3 {
    private MemberRepository memberRepository = MemberRepository.getInstance();

    @RequestMapping("/new-form")
    public String newForm() {
        return "/new-form";
    }
}
```

위와 같이 직접 ModelAndView를 만들고 반환하지 않고 String으로 View 이름만 적어줘도 똑같이 동작하게 된다!

```java
@Controller
@RequestMapping("/mvc")
public class SpringMvcControllerV2 {
    private MemberRepository memberRepository = MemberRepository.getInstance();

    @RequestMapping("/save")
    public ModelAndView save(HttpServletRequest request, HttpServletResponse response) {
        String username = request.getParameter("username");

        ModelAndView modelAndView = new ModelAndView("save-result");
        modelAndView.addObject("member", member);

        return modelAndView;
    }
}
```

```java
@Controller
@RequestMapping("/mvc")
public class SpringMvcControllerV3 {
    private MemberRepository memberRepository = MemberRepository.getInstance();

    @RequestMapping("/save")
    public String save(@RequestParam("username") String username, Model model) {
        model.addAttribute("member", member);
        return "save-result";
    }
}
```

파라미터를 받는 내용도 애노테이션 기반으로 간단하게 받아올 수 있으며 타입 캐스팅 또한 지원한다.

그리고 ModelAndView를 반환하는 코드 또한 파라미터로 Model을 받아 그 model에 값을 넣고 반환하기만 하면된다.

이 모든 것은 @Controller가 가진 애노테이션 기반으로 MVC를 작성할 수 있게하는 방법의 큰 장점이다.

---


# 로깅 라이브러리

Spring Boot Logging Library는 기본으로 다음과 같은 로깅 라이브러리를 사용한다.

- SLF4J
- Logback

로그 라이브러리는 Logback, Log4J, Log4J2 등 다양하게 존재하지만 그것을 통합하여 인터페이스로 제공하는 것이 SLF4J이다.

따라서 SLF4J는 인터페이스 이며, Logback 같은 라이브러리들이 그 구현체이다.

로그를 찍는 방법은 아래와 같은데 아래는 그리 좋지 못한 로깅 방식이다.

```java
log.trace("trace log = "+name);
    log.debug("debug log = "+name);
    log.info("info log = "+name);
    log.warn("warn log = "+name);
    log.error("error log = "+name);
```

로그를 + 연산자로 붙여서 찍으면 안좋은 이유는 다음과 같다.

자바는 메서드 기능을 실행하기 전, 문자열 + 연산이 있으면 그거부터 실행한다. 결국에는 출력하지 않을 내용도 더하기에 대한 연산이 일어나기 때문에 로그를 찍을때는 그리 좋지 않다.

```java
log.trace("trace log = {}",name);
    log.debug("debug log = {}",name);
    log.info("info log = {}",name);
    log.warn("warn log = {}",name);
    log.error("error log = {}",name);
```

그래서 위와같이 작성하는 편이 더 낫다.

또한 logging의 레벨을 설정할 수 있는데 운영 서버에는 info가 적절하며 개발 서버는 debug 정도가 적절하다.

```properties
logging.level.hello.springmvc=trace
logging.level.hello.springmvc=debug
logging.level.hello.springmvc=info
logging.level.hello.springmvc=warn
logging.level.hello.springmvc=error
```

Spring Boot가 셋팅한 Logging Level 기본 값은 INFO임을 잊지말자.

### 로깅 참고 자료

- SLF4J : http://www.slf4j.org
- Logback : http://logback.qos.sh

- Spring Boot가 제공하는 로그
  기능 : http://docs.spring.io/spring-boot/docs/current/reference/html/spring-boot-features.html#bootfeatures-logging

# @RestController와 @Controller와의 차이점

@Controller는 반환값이 String이면 View로 인식된다. 따라서 View Resolver를 거쳐서 View를 찾고 렌더링 된다.

하지만 @RestController 반환값을 View로 인식하는 것이 아니라, Http Body에 바로 입력한다.

## URL 매핑관련 이야기

### 기본 URL 매핑

```java
import lombok.extern.slf4j.Slf4j;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
@Slf4j
public class MappingController {

    // URL 매핑
    @RequestMapping("{/hello/basic, /hello/go}")
    public String helloBasic() {
        log.info("hello basic");
        return "ok";
    }
}
```

위와 같이 배열로 매핑이 가능하기도 한다.

### 기본 URL 매핑 + PathVariable

또한 URL에 PathVariable을 이용해서 매핑이 가능하기도 한데 이는 다음과 같이 사용할 수 있다.

```java
import lombok.extern.slf4j.Slf4j;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
@Slf4j
public class MappingController {
    @GetMapping("/mapping/{userId}")
    public String mappingPath(@PathVariable("userId") String data) {
        log.info("mappingPath userId = {}", data);
        return "ok";
    }

    @GetMapping("/mapping/{userId}")
    public String mappingPathCompact(@PathVariable("userId") String userId) {
        log.info("mappingPath userId = {}", userId);
        return "ok";
    }
}
```

PathVariable으로 받을 PathVariable과 실제 애플리케이션 내에서 사용할 변수명을 똑같이 맞추면 @PathVariable(여기에 값이 없어도 된다.)

### 기본 URL 매핑 + Parameter

```java
import lombok.extern.slf4j.Slf4j;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
@Slf4j
public class MappingController {
    @GetMapping("/mapping/{userId}")
    public String mappingParam(@RequestParam("userId") String userId) {
        log.info(userId);
        return "ok";
    }

    @GetMapping(value = "/mapping-param", params = "mode=debug")
    public String mappingParam() {
        log.info(mappingParam());
        return "ok";
    }
}
```

### 기본 URL 매핑 + 특정 헤더

```java
import lombok.extern.slf4j.Slf4j;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
@Slf4j
public class MappingController {
    @GetMapping(value = "/mapping-header", params = "mode=debug")
    public String mappingHeader() {
        log.info(mappingHeader());
        return "ok";
    }
}
```

### 기본 URL 매핑 + 컨텐츠 타입

위와 같이 특정 헤더가 있어야먄 요청이 되도록 할 수 있다.

```java
import lombok.extern.slf4j.Slf4j;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
@Slf4j
public class MappingController {
    @PostMapping(value = "/mapping-consume", consumes = "application/json")
    public String mappingConsume() {
        log.info("mappingConsume");
        return "ok";
    }
}
```

위와 같이 요청받는 미디어 타입을 지정할 수도 있다. (Exception Code : 415)

```java
import lombok.extern.slf4j.Slf4j;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
@Slf4j
public class MappingController {
    @PostMapping(value = "/mapping-produce", produces = "text/html")
    public String mappingProduces() {
        log.info("mappingProduces");
        return "ok";
    }
}
```

위와 같이 반환 미디어 타입을 지정할 수도 있다. (Exception Code : 406)

---

# @ResponseBody 애노테이션에 몰랐던 사실과 요청 파라미터 받기

## @ResponseBody라는 애노테이션

```java
import lombok.extern.slf4j.Slf4j;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@Controller
@Slf4j
public class MappingController {

    @ResponseBody
    @RequestMapping(value = "/request-param-v2")
    public String requestParamV2(@RequestParam("username") String memberName, @RequsetParam("userAge") int memberAge) {
        return "ok";
    }
}
```

위와 같이 작성하면 우리는 반환값에 ok라는 뷰를 찾도록 한다는 것을 알고 있다.

그런데, @ResponseBody라는 애노테이션을 해당 메서드에 붙여준다면, 이는 우리가 알고 있는 @RestController와 동일하게 동작한다.

즉, 반환값으로 View를 찾아 내려주는 것이 아닌, 응답 Body에 그대로 넣어주는 것이다!

## 요청 파라미터 받아보기

RequsetParam은 Parameter Name과 변수명을 같게 한다면 더 간단하게 아래와 같이 사용할 수 있다.

```java
import lombok.extern.slf4j.Slf4j;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.Controller;

@Controller
@Slf4j
public class MappingController {

    @ResponseBody
    @RequestMapping(value = "/request-param-v2")
    public String requestParamV2(@RequestParam String memberName, @RequsetParam int memberAge) {
        return "ok";
    }
}
```

그런데 더 간단하게도 사용이 가능하다 ㄷㄷ...

```java
import lombok.extern.slf4j.Slf4j;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.Controller;

@Controller
@Slf4j
public class MappingController {

    @ResponseBody
    @RequestMapping(value = "/request-param-v2")
    public String requestParamV2(String memberName, int memberAge) {
        return "ok";
    }
}
```

위와 같은 방식으로도 Query Parameter를 받을 수 있다!

물론 이 방식은 String, int, Integer 등 단순 타입만 가능하다.

근데 마냥 좋고 편한건 아닌 것 같다. 파라미터로 받는다는 내용이 그리 명확하게 보이진 않는다는 점 때문이다.

## 요청 파라미터의 강제성 옵션 부여

```java
import lombok.extern.slf4j.Slf4j;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.Controller;

@Controller
@Slf4j
public class MappingController {

    @ResponseBody
    @RequestMapping(value = "/request-param-v2")
    public String requestParamV2(
        @RequestParam(required = true) String memberName,
        @RequestParam(required = false) int memberAge) {
        return "ok";
    }
}
```

기본 값은 true 옵션이다. 만약 필수로 지정한 파라미터가 들어오지 않는다면 BAD_REQUEST가 발생한다.

## 요청 파라미터의 기본값 부여

```java
import lombok.extern.slf4j.Slf4j;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.Controller;

@Controller
@Slf4j
public class MappingController {

    @ResponseBody
    @RequestMapping(value = "/request-param-v2")
    public String requestParamV2(
        @RequestParam(required = true, defaultValue = "testValue") String memberName,
        @RequestParam(required = false, defaultValue = "testAge") int memberAge) {
        return "ok";
    }
}
```

위와 같이 요청 파라미터를 받지 못하였을 때 자동으로 기본 값을 세팅해 줄 수도 있다.

## 모든 요청 파라미터를 받고 싶을 땐 - Map

```java
import lombok.extern.slf4j.Slf4j;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.Controller;

@Controller
@Slf4j
public class MappingController {

    @ResponseBody
    @RequestMapping(value = "/request-param-v2")
    public String requestParamV2(@RequestParam Map<String, Object> paramMap) {
        log.info(paramMap.get("userName"), paramMap.get("userAge"));
        return "ok";
    }
}
```

위와 같이 Map 자료형으로 다 받아버린 다음에, 원하는 Key값을 통해 실제 요청 값을 받아올 수 있다.

## 모든 요청 파라미터를 받고 싶을 땐 - MultiValueMap

```java
import lombok.extern.slf4j.Slf4j;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.Controller;

@Controller
@Slf4j
public class MappingController {

    @ResponseBody
    @RequestMapping(value = "/request-param-v2")
    public String requestParamV2(@RequestParam MultiValueMap<String, Object> multiValueMap) {
        int[] keys = multiValueMap.keys();
        int user1Id = keys[0];
        int user2Id = keys[1];
        log.info(paramMap.get("userName"), paramMap.get("userAge"));
        return "ok";
    }
}
```

위와 같이 MultiValueMap을 활용한다면, 같은 파라미터 네임에 각기 다른 요청 값을 받아와 처리할 수도 있다.

## 요청 값 받는 것을 자동화 해보자! - ModelAttribute

```java
import lombok.extern.slf4j.Slf4j;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.Controller;

@Controller
@Slf4j
public class MappingController {

    @Data
    class HelloData {
        private String username;
        private Integer age;
    }

    @ResponseBody
    @RequestMapping(value = "/model-attribute-v1")
    public String modelAttribute(@ModelAttribute HelloData helloData) {
        log.info(helloData.getUsername, helloData.getAge);
        return "ok";
    }
}
```

@ModelAttribute 를 사용하면, Spring MVC가 하는 일은 다음과 같다.

1. 해당 애노테이션이 붙은 모델 (HelloData)를 생성한다.
2. 요청 파라미터의 이름으로 HelloData 객체의 프로퍼티를 찾고, 해당 프로퍼티의 Setter를 사용하여 파라미터의 값을 바인딩 한다.

만약 바인딩 과정에서 Integer로 받아야할 값에 String을 넣게 되면 BindException예외를 터뜨린다.

근데 더 간단하게 아래와 같이도 받을 수도 있다.

```java
import lombok.extern.slf4j.Slf4j;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.Controller;

@Controller
@Slf4j
public class MappingController {

    @Data
    class HelloData {
        private String username;
        private Integer age;
    }

    @ResponseBody
    @RequestMapping(value = "/model-attribute-v1")
    public String modelAttribute2(HelloData helloData) {
        log.info(helloData.getUsername, helloData.getAge);
        return "ok";
    }
}
```

RequsetParam도 생략가능하고, ModelAttirbute도 생략가능한데 그러면 Spring MVC는 도대체 어떻게 판단하고 바인딩을 하는걸까?

Spring에서는 애노테이션을 생략하면, 매개변수로 들어올 타입을 검증한다.

Integer, String 등 단순 타입이면 @RequestParam으로

그 외의 모델 객체를 만든 것이라면 @ModelAttribute로 인식한다!


조금 더 깊게 이야기하면, Argument Resolver로 지정된 타입이 아닐 시 @ModelAttribute로 인식하는 것인데.

Argument Resvoler란 무엇일까?

해당 메서드에 들어올 수 있는 예약된 매개변수 타입이 있을 수 있다.

컨트롤러에서 사용하는 메서드는 HttpServletRequest, HttpServletResponse 가 그 예시가 될 수 있다.

따라서 개발자가 직접 작성한 객체가 아닌 것들 외에는 @ModelAttribute로 인식한다고 알고 있으면 된다.

# HTTP Message Body에 요청 받기

## Row한 서블릿을 바탕으로 Message Body 읽기

```java
@Slf4j
@Controller
public class RequestBodyStringController {
    @PostMapping("/request-body-string-v1")
    public void requestBodyString(HttpServletRequest request, HttpServletResponse response) throws IOException {
        ServletInputStream inputStream = request.getInputStream();
        String messageBody = StramUtils.copyToString(inputStream, StandardCharsets.UTF_8);

        log.info(messageBody);
        response.getWriter.write("ok");
    }
}
```

## Row한 서블릿으로 Message Body 읽기 -> 조금 더 개선하기 (사실 서블릿으로 받을 필요까지는 없다!)

```java
@Slf4j
@Controller
public class RequestBodyStringController {
    @PostMapping("/request-body-string-v2")
    public void requestBodyString(InputStream inputStream, Writer responseWriter) throws IOException {
        String messageBody = StramUtils.copyToString(inputStream, StandardCharsets.UTF_8);

        log.info(messageBody);
        responseWriter.write("ok");
    }
}
```

위와 같이 서블릿을 받지않고 InputStream, Writer를 직접 받을 수 있는 이유는 Spring MVC가 해결해주기 때문이다!

## HttpEntity 활용하기

```java
@Slf4j
@Controller
public class RequestBodyStringController {
    @PostMapping("/request-body-string-v3")
    public HttpEntity<String> requestBodyString(HttpEntity<String> httpEntity) throws IOException {
        String messageBody = httpEntity.body();

        log.info(messageBody);
        return new HttpEntity<>("ok");
    }
}
```

위와 같이 HttpEntity<String> 와 같은 형태로 요청받기/반환하기 를 할 수 있는데, 이는 HttpMessageConverter가 지원해주는 기능이다.


## HttpEntity와 ResponseEntity

```java
@Slf4j
@Controller
public class RequestBodyStringController {
    @PostMapping("/request-body-string-v3")
    public HttpEntity<String> requestBodyString(HttpEntity<String> httpEntity) throws IOException {
        String messageBody = httpEntity.body();

        log.info(messageBody);
        return new ResponseEntity<>("ok", HttpStatus.CREATED);
    }
}
```

위와 같이 ResponseEntity를 사용하여 상태코드를 명시적으로 내려줄 수도 있다.

그리고 ResponseEntity는 HttpEntity를 상속받은 클래스이다.

## @RequestBody로 입력받고 @ResponseBody로 반환하기

```java
@Slf4j
@Controller
public class RequestBodyStringController {

    @ResponseBody
    @PostMapping("/request-body-string-v4")
    public String requestBodyString(@RequestBody String messageBody) throws IOException {
        log.info(messageBody);
        return "ok";
    }
}
```

위와 같이 @ResponseBody가 붙은 메서드의 메서드 반환 타입을 String으로 바꾸고 문자열을 반환하면 JSON 형식으로 Body에 데이터를 응답해준다.

# HTTP Message Body에 JSON으로 요청 받기

## Row한 방법으로 JSON 요청 받기

```java

@Slf4j
@Controller
public class RequestBodyJSONController {

    private ObjectMapper objectMapper = new ObjectMapper();

    @PostMapping("/request-body-json-v1")
    public void requestBodyJsonV1(HttpServletRequest request, HttpServletResponse response) throws IOException {
        ServletInputStream inputStream = request.getInputSteram();
        String messageBody = StreamUtils.copyToString(inputStream, StandardCharsets.UTF_8);

        log.info(messageBody);
        HelloData helloData = objectMapper.readValue(messageBody, HelloData.class);

        log.info("username = {}", helloData.getUsername());
    }
}
```

## @RequestBody로 입력 받기 (@RequsetBody + String messageBody)

```java

@Slf4j
@Controller
public class RequestBodyJSONController {

    private ObjectMapper objectMapper = new ObjectMapper();

    @ResponseBody
    @PostMapping("/request-body-json-v2")
    public String requestBodyJsonV2(@RequestBody String messageBody) throws IOException {
        log.info(messageBody);
        HelloData helloData = objectMapper.readValue(messageBody, HelloData.class);
        log.info("username = {}", helloData.getUsername());

        return "ok";
    }
}
```

## 객체 타입으로 입력 받기 (@RequestBody + DTO)

```java

@Slf4j
@Controller
public class RequestBodyJSONController {

    private ObjectMapper objectMapper = new ObjectMapper();

    @ResponseBody
    @PostMapping("/request-body-json-v3")
    public String requestBodyJsonV3(@RequestBody HelloData helloData) throws IOException {
        log.info("username = {}", helloData.getUsername());
        return "ok";
    }
}
```

이 방법이 가능한 이유는 다음과 같다.

HttpEntity, @RequestBody를 사용하면 HTTP 메시지 컨버터가 HTTP메시지 바디의 내용을 우리가 원하는 문자나 객체로 변환해준다.

HTTP 메시지 컨버터는 문자 뿐만 아니라, JSON도 객체로 변환해주는 것인데 코드로 살펴본다면 V2에서 작성했던

```java
HelloData helloData = objectMapper.readValue(messageBody, HelloData.class);
```
이런 구문을 대신 해준다는 것이다.

# RequestBody는 생략 불가능하다.

매개변수에 애노테이션을 생략하고 입력한다면 @ModelAttribute 라는 애노테이션으로 처리해버린다.

그리고 위에서 다룬 내용을 다시 한 번 살펴보자면

String, int, Integer 같은 단순 타입은 @RequestParam

커스텀한 객체 등 그 외의 타입은 @ModelAttribute 로 인식한다.

---

# HTTP 응답 방식의 3가지

## 정적 리소스로 응답

HTML, CSS, JS

path : /resources/static

## 뷰 템플릿로 응답

SSR (ex : Thymeleaf)

path : /resources/templates

## HTTP API로 응답

HTTP 메세지 바디에 JSON과 같은 타입으로 응답

### 응답 예시코드

```java

@Controller
public class ResponseBodyController {

    @GetMapping("/response-body-string-v1")
    public void responseBodyV1(HttpServletResponse response) throws IOException {
        response.getWriter().write("ok");
    }

    @GetMapping("/response-body-string-v2")
    public ResponseEntity<String> responseBodyV2(HttpServletResponse response) {
        return new ResponseEntity<>("ok", HttpStatus.OK);
    }

    @GetMapping("/response-body-string-v3")
    public String responseBodyV2() {
        return "ok";
    }

    @GetMapping("/response-body-json-v1")
    public HelloData responseBodyJsonV1() {
        HelloData helloData = new HelloData();
        helloData.setUsername("userA");
        return new ResponseEntity<>(helloData, HttpStatus.OK);
    }

    @ResponseStatus(HttpStatus.OK)
    @ResponseBody
    @GetMapping("/response-body-json-v2")
    public HelloData responseBodyJsonV1() {
        HelloData helloData = new HelloData();
        helloData.setUsername("userA");
        return helloData;
    }
}
```

### @RestController = @Controller + @ResponseBody 이다.

# HTTP Message Converter

Spring MVC는 각 경우에 HTTP Message Converter를 적용한다.

- HTTP Request : @RequestBody, HttpEntity(RequestEntity)
- HTTP Response : @ResponseBody, HttpEntity(ResponseEntity)


## HTTP Message Convert는 인터페이스이다.

일단 Http Message Converter는 요청/응답 모두 사용된다.

그 인터페이스 내부에는

canRead(), canWriter() 라는 메서드가 명세되어있는데, Converter가 해당 클래스, 미디어타입을 지원하는지 체크해준다.

read(), writer() 라는 메서드도 명세되어있는데, 이는 Converter를 통해 메시지 읽기/쓰기를 수행하는 기능이다.

## Spring Boot에 담긴 기본 Message Converter

- 0 = ByteArrayHttpMessageConverter
    - 클래스 타입 : byte[]
    - 미디어 타입 : */*
    - 요청 예시 : @RequestBody byte[] data
    - 응답 예시 : @ResponseBody return byte[]

- 1 = StringHttpMessageConverter
    - 클래스 타입 : String
    - 미디어 타입 : */*
    - 요청 예시 : @RequestBody String data
    - 응답 예시 : @ResponseBody return "ok"

- 2 = MappingJackson2HttpMessageConverter
    - 클래스 타입 : Object, HashMap
    - 미디어 타입 : application/json 혹은 application/json 관련 ...
    - 요청 예시 : @RequestBody RequestForm requestForm
    - 응답 예시 : @ResponseBody return ResponseForm

...

위와 같이 여러 종류의 Message Converter가 존재하는데 각 컨버터를 배정하기 위해서는

대상 클래스 타입, 미디어 타입을 체크하여 어떤 컨버터를 사용할지 결정한다.

## HTTP Message Converter는 어디서 동작하는건가?

애노테이션 기반의 컨트롤러인 @RequestMapping을 처리하는 HandlerAdapter인 RequsetMappingHandlerAdapter를 주목하자.

### RequsetMappingHandlerAdapter 동작방식

![img.png](https://github.com/K-Diger/K-Diger.github.io/blob/main/images/SpringMVC%20%EA%B5%AC%EC%A1%B0%EC%9D%98%20%ED%95%B8%EB%93%A4%EB%9F%AC%20%EC%96%B4%EB%8C%91%ED%84%B0%20%EC%83%81%EC%84%B8.png?raw=true)

애노테이션 기반의 컨트롤러는 매우 다양한 파라미터를 사용할 수 있다. (HttpServletRequest, Model, @RequestParam, @ModelAttribute, @RequestBody, HttpEntity)

이렇게 유연한 파라미터를 다룰 수 있는 것은 ArgumentResolver 덕분이다.

애노테이션 기반 컨트롤러를 처리하는 RequestMappingHandlerAdapter는 ArgumentResolver를 호출하여 핸들러가 필요로 하는 여러 파라미터의 객체(값)을 생성한다.

그리고 파라미터 값이 셋팅되면 컨트롤러를 호출한다.

즉, 다시 한 번 정리하자면,

1. Dispatcher Servlet이 요청을 처리할 수 있는 적절한 핸들러를 찾는다. (핸들러 매핑)
2. Disaptcher Servlet이 요청을 처리할 수 있는 핸들러를 다룰 수 있는 핸들러 어댑터를 찾는다.
3. 핸들러 어댑터를 호출한다.
4. 핸들러 어댑터는 요청으로 들어온 값을 자기가 다룰 수 있는 핸들러의 파라미터를 만들기 위해 Argument Resolver를 호출한다.
5. Argument Resolver는 컨트롤러가 다룰 수 있는 타입으로 요청 값을 변환하여 핸들러 어댑터에게 전달한다.
6. 핸들러 어댑터는 반환받은 값을 바탕으로 핸들러를 호출한다.

### ArgumentResolver

```java
public interface HandlerMethodArgumentResolver {
    boolean supportsParameter(MethodParameter parameter);

    @Nullable
    Object resolveArgument(MethodParameter parameter, @Nullable ModelAndViewContainer mavContainer,
                           NativeWebRequest webRequest, @Nullable WebDataBinderFactory binderFactory) throws Exception;
}
```

ArgumentResolver의 동작방식은 다음과 같다.

1. ArgumentResolver의 supportsParameter()를 호출해서 해당 파라미터를 지원하는지 체크한다.
2. 지원한다면, resolveArgument()를 호출해서 실제 객체를 생성한다.
3. 지원하지 않는다면, 다른 ArgumentResolver를 검사한다.

### ReturnValueHandler

```java
public interface HandlerMethodReturnValueHandler {
    boolean supportsReturnType(MethodParameter parameter);

    @Nullable
    void handlerReturnValue(@Nullable Object returnValue, MethodParameter returnType,
                           ModelAndViewContainer mavContainer, NativeWebRequest webRequest) throws Exception;
}
```

ReturnValueHandler는 컨트롤러에서 String으로 View이름만 반환해도 동작하던 이유를 설명한다.

## 다시 본론으로. HttpMessageConverter는 어디에서 동작하는가?

![img.png](https://github.com/K-Diger/K-Diger.github.io/blob/main/images/SpringMVC%20HttpMessageConverter%EC%9D%98%20%EC%9C%84%EC%B9%98.png?raw=true)

위 그림과 같이

ArgumentResolver가 HttpMessageConverter를 호출하고

ReturnValueHandler가 HttpMessageConverter를 호출한다.

결국에는 ArgumentResolver는 MessageConverter를 호출해서 나온 값을 HandlerAdapter에게 전달하는 것이다!

