---

title: Spring Web MVC - Logging, Spring MVC 기본 기능, HTTP 요청
author: 김도현
date: 2023-01-17
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
log.trace("trace log = " + name);
log.debug("debug log = " + name);
log.info("info log = " + name);
log.warn("warn log = " + name);
log.error("error log = " + name);
```
로그를 + 연산자로 붙여서 찍으면 안좋은 이유는 다음과 같다.

자바는 메서드 기능을 실행하기 전, 문자열 + 연산이 있으면 그거부터 실행한다. 결국에는 출력하지 않을 내용도 더하기에 대한 연산이 일어나기 때문에 로그를 찍을때는 그리 좋지 않다.

```java
log.trace("trace log = {}", name);
log.debug("debug log = {}", name);
log.info("info log = {}", name);
log.warn("warn log = {}", name);
log.error("error log = {}", name);
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

- Spring Boot가 제공하는 로그 기능 : http://docs.spring.io/spring-boot/docs/current/reference/html/spring-boot-features.html#bootfeatures-logging

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
