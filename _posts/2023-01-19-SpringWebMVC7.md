---

title: Spring Web MVC - HTTP 응답, ArgumentResolver, ReturnValueHandler, Http Message Converter

date: 2023-01-19
categories: [Spring, MVC]
tags: [Spring, MVC]
layout: post
toc: true
math: true
mermaid: true

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

