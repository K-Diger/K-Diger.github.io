---

title: Spring Web MVC - 핸들러 매핑, 핸들러 어댑터, 뷰 리졸버, Spring MVC
author: 김도현
date: 2023-01-16
categories: [Spring, MVC]
tags: [Spring, MVC]
layout: post
toc: true
math: true
mermaid: true

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

