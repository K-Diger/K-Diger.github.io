---

title: Spring Web MVC - Logging, Spring MVC 기본 기능, HTTP 요청 받기(RequestParam, ModelAttribute)
author: 김도현
date: 2023-01-18
categories: [Spring, MVC]
tags: [Spring, MVC]
math: true
mermaid: true

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

