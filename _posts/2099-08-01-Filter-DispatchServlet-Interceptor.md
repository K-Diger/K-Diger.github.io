---

title: Filter Interceptor 그 사이의, Dispatcher-Servlet

date: 2022-08-01
categories: [Spring, Filter, Servlet, Interceptor]
tags: [Filter, Servlet, Interceptor]
layout: post
toc: true
math: true
mermaid: true

---

# 1.1 필터란 무엇인가?

![Filter](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2Fbgl4od%2FbtrAeQhm9zB%2FuSYOombcuhv4jCcUlOnon0%2Fimg.png)

위 그림과 같이, `Filter` 는 실제 Spring Environment 에 도달하기 전에, `사용자의 요청을 먼저 확인하는 위병소`의 역할을 한다.

Spring 에서 Filter를 사용하려면 `javax.servlet` 에 명세되어있는 `Filter`를 구현하여 사용해야한다.

---

# 1.2 필터를 어디에 사용하는가?

- 요청과 응답을 로깅 할 때

- 요청 처리 시각을 로깅 할 때

- Request Body 혹은 Request Header 를 포맷팅 할 때

- Authentication Token 을 검증할 때

---

# 1.3 필터는 어떻게 생겼을까?

```java
package javax.servlet;

import java.io.IOException;

public interface Filter {

    public default void init(FilterConfig filterConfig) throws ServletException {}

    public void doFilter(
            ServletRequest request,
            ServletResponse response,
            FilterChain chain
    ) throws IOException, ServletException;

    public default void destroy() {}
}
```

해당 코드에서 제거된 주석을 자세히 보면, 약 9가지의 쓰임새로 사용된다고 한다.

1. Authentication Filters

2. Logging and Auditing Filters

3. Image conversion Filters

4. Data compression Filters

5. Encryption Filters

6. Tokenizing Filters

7. Filters that trigger resource access events

8. XSL/T filters

9. Mime-type chain Filter

... 너무 많다. 일단 아래와 같은 상황에서 쓰인다고만 알아두자.

```text
1. 요청과 응답을 로깅 할 때

2. 요청 처리 시각을 로깅 할 때

3. Request Body 혹은 Request Header 를 포맷팅 할 때

4. Authentication Token 을 검증할 때
```

---

# 1.4 필터를 어떻게 사용하는가?

Filter Inteface는 3가지 메서드를 명세해 놓았다.

## init()

웹 컨테이너는 init() 메서드를 호출하여 필터 객체를 초기화하고 서비스에 추가한다.

필터 객체를 초기화 한 이후에는, doFilter()를 통해 처리된다.

---

## doFilter()

`매개변수`

1. ServletRequest request – 처리할 요청

2. ServletResponse response – 요청과 관련된 응답

3. chain – 필터의 추가 처리를 위해서, 요청 및 응답을 전달할 체인의 다음 필터에 대한 액세스를 제공


### doFilter() 요약

doFilter()는 특정 `URL-Pattern` 에 부합하는 모든 HTTP 요청이 `디스패처 서블릿으로 전달되기 전`, 웹 컨테이너에 의해 실행되는 메서드이다.

즉, 사용자 요청이 실질적으로 필터링되는 단계이다.

또한 doFilter() 의 파라미터로, FilterChain이 있다. FilterChain의 doFilter를 통해 다음 대상(Next Filter OR Dispatcher Servlet)으로 요청을 전달한다.

---

## destory()

필터 객체를 제거하고, 사용하는 자원을 반환하기 위한 메서드이다.

init() 과 마찬가지로, 웹 컨테이너가 1회 호출하며, 필터 객체를 종료하면, doFilter() 에 의해 처리되지 않는다.

---

# 2.1 디스패처 서블릿이란 무엇인가?

`dispatch : 보내다`

HTTP 프로토콜로 들어오는 모든 요청을 가장 먼저 받아 (Spring Environment 기준) 적합한 컨트롤러에 위임해주는 Front Contoller 이다.

즉, 클라이언트로 부터 요청이 오면 Spring에서 자주 사용되는 Tomcat 같은 서블릿 컨테이너가 요청을 받게 된다.

그리고 서블릿 컨테이너(Tomcat)에서 받은 이 요청들을 Spring의 가장 앞에 위치한 서블릿인 Dispatcher-servlet 으로 보내게 되는 것이다.

# 2.2 왜 쓰는가?

dispatcher-servlet이 해당 애플리케이션으로 들어오는 모든 요청을 핸들링 해주고, 공동 작업을 처리해준다.

## 영화관 티켓 검증하는 직원과 역할과 같다.

해당 요청(영화 티켓) 을 가지고, "저 이거 보러왔서염~" 하면, "넹 인터스텔라는 7관으로가세여", "범죄와의 전쟁은 8관으로 가세염~" 해주듯..

---

# 2.3 문제점은 없었는가?

기존, Dispatcher Servlet이 모든 요청을 처리하다보니, HTML/CSS/JavaScript같은 정적 파일에 관한 요청 또한 모두 가로채어 정적자원을 불러오지 못하는 현상이 발생했다.

이 문제를 해결하기 위한 두 가지 방법을 제안했다.

## 해결책 1. 정적 자원에 대한 요청과 애플리케이션에 대한 요청을 분리한다.

/apps 의 URL로 접근하면, Dispatcher Servlet이 담당한다.

/resources 의 URL 로 접근하면, Dispatcher Servlet이 컨트롤 하지 않는다.

## 해결책 2. 애플리케이션에 관한 요청을 탐색하고, 없으면 정적 자원에 관한 요청으로 처리한다.

Dispatcher Servlet 이 요청을 처리할 컨트롤러를 먼저 찾고

요청에 대한 컨트롤러를 찾을 수 없을 경우, Resources 경로를 탐색하는 것이다.

---

### 해결책 둘 중 어떤 것?

우리는 현재 2번 방법을 사용하고 있다.

1번 방법은 모든 요청에 대하여 해당하는 URL-Pattern을 설계해야하므로 직관적인 설계를 할 수 없을 뿐만 아니라 코드가 상당히 지저분해진다는 단점이 있었기 때문이다.

---

# 2.4 디스패처 서블릿 동작 과정

![Dispatcher-Servlet](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FbImFbg%2FbtrGzZMTuu2%2FCkY4MiKvl5ivUJPoc5I3zk%2Fimg.png)

1. Client Request를 Dispatcher-Servlet 이 받는다.

2. Request를 토대로, 알맞은 Controller를 찾기 위해, Handler-Mapping에 요청 한다.

3. Handler-Mapping이 찾아준 Controller에 요청을 위임하기위해, Handler-Adapter에 맡긴다.

4. Hander-Adapter가 컨트롤러로 요청을 위임한다.

5. 비즈니스 로직을 수행한다.

6. Controller가 Handler-Adapter에 반환 값을 전달한다.

7. Handler-Adapter가 반환값을 처리한다.

8. Server의 응답을 Client에 반환한다.


`Dispatcher-Servlet, Handler-Mapping, Handler-Adapter 는 Spring의 구현체`

`Controller, Service, Repository, ResponseEntity 는 개발자의 몫`

`Database는 외부 시스템이다.`

---

# 3.1 인터셉터란 무엇인가?

![Interceptor](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FbVxpUK%2FbtrAfCDbKqx%2FvKZpnAWkOHKID44Qemz2hK%2Fimg.png)

위 그림과 같이, Request와 Response 사이에서 동작한다.

인터셉터는 필터와 다르게, Spring 단위에서 동작한다.

디스패처 서블릿은 Handler-Mapping을 통해 컨트롤러를 찾도록 요청한다. (Handler-Mapping은 실행 체인을 반환한다.)

실행 체인에 인터셉터가 있다면 인터셉터를 거쳐 컨트롤러가 실행을 하도록 한다.

실행 체인에 인터엡터가 없다면 바로 컨트롤러를 실행한다.

---

# 3.2 인터셉터를 어떻게 사용하는가?

인터셉터를 사용하기 위해, org.springframework.web.servlet의 HandlerInterceptor Interface를 구현해야 한다.

```java
public interface HandlerInterceptor {

    default boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler)
        throws Exception {

        return true;
    }

    default void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler,
                            @Nullable ModelAndView modelAndView) throws Exception {
    }

    default void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler,
                                 @Nullable Exception ex) throws Exception {
    }
}
```

## preHandle

컨트롤러가 호출되기 전에 실행된다.

컨트롤러에 닿기 이전에 처리해야하는 작업이 있으면 이때 요청 정보를 가공하거나 추가할 수 있다.

preHandler의 매개변수인 Object handler 는 Handler-Mapping 이 찾아준 Controller Bean 에 매핑되는 HandlerMethod 타입의 객체이다.

이 HandlerMethod 타입은 @RequestMapping이 붙은 메서드의 정보를 추상화한 객체이다.

마지막으로, preHandle의 반환값은 boolean 타입이다. 반환 값이 true 이면 다음 단계로 진행이 되지만, 반환 값이 false 라면 작업을 중단하고 이후의 작업은 진행되지 않는다.

## postHandle

컨트롤러가 호출된 이후에 실행된다. 따라서 컨트롤러 이후에 처리해야하는 후처리 작업에 사용하는 것이다.

이 메서드에는 Controller가 반환하는 ModelAndView 타입의 정보가 제공되지만,

최근 RestController 를 사용하게 되면서 자주 사용되지 않는다.


## afterCompletion

모든 뷰에서 최종 결과를 생성하는 일을 포함하여, 모든 작업이 완료된 후에 실행된다.

사용한 리소스를 반환하는 역할이다.

---

# 번외. Filter vs Interceptor

## Filter <-> Spring Security

필터는 웹 애플리케이션 전반적으로 사용되는 기능에서 사용된다. (공통적인 보안 및 인증/인가, 모든 요청에 대한 로깅, Spring과 무관하게 작동해야하는 기능 등)

## Interceptor

인터셉터는 클라이언트 요청과 관련되어 전역적으로 처리되어야 하는 때 사용된다. (API 호출에 대한 로깅, 세부적인 보안 및 인증/인가, Controller 로 넘겨주는 데이터 가공)
