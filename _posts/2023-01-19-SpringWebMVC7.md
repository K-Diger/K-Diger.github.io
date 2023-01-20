---

title: Spring Web MVC - HTTP 응답, Http Message Converter
author: 김도현
date: 2023-01-19
categories: [Spring, MVC]
tags: [Spring, MVC]
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
  - 미디어 타입 : application/json || application/json 관련 ...
  - 요청 예시 : @RequestBody RequestForm requestForm
  - 응답 예시 : @ResponseBody return ResponseForm

...

위와 같이 여러 종류의 Message Converter가 존재하는데 각 컨버터를 배정하기 위해서는

대상 클래스 타입, 미디어 타입을 체크하여 어떤 컨버터를 사용할지 결정한다.

## HTTP Message Converter는 어디서 동작하는건가?

애노테이션 기반의 컨트롤러인 @RequestMapping을 처리하는 HandlerAdapter인 RequsetMappingHandlerAdapter를 주목하자.

### RequsetMappingHandlerAdapter 동작방식

![img.png](images/SpringMVC 구조의 핸들러 어댑터 상세.png)
