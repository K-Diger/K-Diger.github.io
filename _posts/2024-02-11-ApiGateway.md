---

title: API Gateway와 조금 더 친해져보기
date: 2024-02-11
categories: [Gateway]
tags: [Gateway]
layout: post
toc: true
math: true
mermaid: true

---

- [Spring Cloud Gateway Reference](https://cloud.spring.io/spring-cloud-gateway/reference/html/)

---

# API Gateway가 왜 필요한가?

MSA환경은 각 도메인 서비스에 여러 대의 인스턴스를 할당하여 스케일 아웃을 통해 확장성/가용성의 이점을 얻을 수 있다.

그렇다면 클라이언트는 UserService라는 도메인 서비스가 스케일 아웃이 된다면 확장된 서비스의 IP주소, 포트번호 등을 매번 갱신해줘야 사용할 수 있게된다.

하지만 이 방법은 언제 어느 갯수만큼 확장될지 모르는 Auto-Scaling환경에 부적합하다. 만약 인스턴스가 1시간동안 10분을 주기로 1개의 인스턴스가 스케일 아웃이 된다면 10분마다 클라이언트 코드를 수정하고 배포해야하기 때문이다.

이러한 문제를 해결하기 위해 API Gateway가 적절한 도구로써 채택되었다.

API Gateway는 각 도메인 서비스에 대한 라우팅과 더불어 요청 데이터를 마이크로서비스에 도달하기 전 사전에 검증할 수 있는 역할도 수행할 수 있다.

API 게이트웨이의 역할은 유동적이지만 일반적으로

- 인증
- 라우팅
- 요청 속도 제한
- 모니터링
- 분석
- 정책 필터링
- 알림
- 보안

등이 있다.

---

# Spring Cloud Gateway 주요 요소

- Route
- Predicate
- Filter

용어를 알아보기 전, API Gateway의 주소는 `http://localhost:8000`이라고 가정한다.

## Route

인스턴스 고유 식별자(ID), 목적지 인스턴스의 실제 주소를 통해 Gateway가 요청을 목적지로 라우팅해준다.

즉, 이 설정을 통해 클라이언트가 인스턴스 고유 식별자에 요청을 보내면 목적지 인스턴스에 라우팅을 해주는 것이다.

일반적인 Spring Cloud Gateway에서 Route설정을 하기 위해선

- 인스턴스 id
- 인스턴스 실제 uri,
- 인스턴스에 도달하기 위한 조건인 predicate
- 요청을 라우팅 하기 전 filter

를 등록한다.

```yaml
spring:
  cloud:
    gateway:
      routes: # 라우팅 설정 등록
        - id: ...
          uri: ...
          predicates:
            - ...
          filters:
            - ...
```

---

## Predicate

Spring Cloud Gateway에서는 Java8에 도입된 Predicate를 사용한다.

Predicate는 Argument를 받아 boolean 값을 반환하는 함수형 인터페이스이다.

요청한 URI의 문자열 패턴을 살펴 본 후 어떤 인스턴스로 라우팅할지 판단하기 위한 요소로 볼 수 있다.

```yaml
spring:
  cloud:
    gateway:
      routes:
        - id: ...
          uri: ...
          predicates: # 클라이언트가 http://localhost:8000/api/auth/~~~ 로 요청했다면 이 라우팅 설정이 적용된다.
            - Path=/api/auth/**
          filters:
            - ...
```

---

## Spring Cloud Gateway가 지원하는 11가지 Predicate

### 1,2,3 : Time After/Before/Between 판별

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: after_route
        uri: https://example.org
        predicates:
        - After=2024-02-10T17:42:47.789-07:00[America/Denver]
```

위 Predicate는 해당 시간 이전의 요청을 라우팅 uri로 보낸다는 의미이다.

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: after_route
        uri: https://example.org
        predicates:
        - Before=2024-02-10T17:42:47.789-07:00[America/Denver]
```

위 Predicate는 해당 시간 이후의 요청을 라우팅 uri로 보낸다는 의미이다.

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: between_route
        uri: https://example.org
        predicates:
        - Between=2017-01-20T17:42:47.789-07:00[America/Denver], 2017-01-21T17:42:47.789-07:00[America/Denver]
```

위 Predicate는 해당 시간 사이대의 요청을 라우팅 uri로 보낸다는 의미이다.

### 4,5 : Cookie, Header 판별

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: cookie_route
        uri: https://example.org
        predicates:
        - Cookie=chocolate, ch.p
```

위 Predicate는 요청 쿠키를 확인하여 **쿠키 이름**이 `chocolate`인 내용이 있다면 그 값이 **정규식**에 해당하는 `ch.p`에 해당하는지 확인한다.

즉, 쿠키 이름과 해당 쿠키의 값이 정규식에 해당하는지 확인하는 내용이다.

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: header_route
        uri: https://example.org
        predicates:
        - Header=X-Request-Id, \d+
```

위 Predicate는 요청 헤더를 확인하여 **헤더 이름**이 `X-Request-Id`인 내용이 있다면 그 값이 **정규식**에 해당하는 `\d+`에 해당하는지 확인한다.

### 6. Method 판별

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: method_route
        uri: https://example.org
        predicates:
        - Method=GET,POST
```

위 Predicate는 요청 Method를 확인하여 Predicate조건에 부합하는지 확인하는 것이다.

### 7,8 : HOST, Path, Query 판별

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: host_route
        uri: https://example.org
        predicates:
        - Host=**.somehost.org, **.anotherhost.org
```

위 Predicate는 요청 HOST를 확인하여 Predicate조건에 부합하는지 확인하는 것이다.

이를 활용하면 **서브 도메인**에 대한 요청을 Predicate로 다룰 수도 있다.

그리고 `ServerWebExchange.getAttributes()`구문으로 요청한 서브도메인이 어떤 것인지에 대한 내용도 확인할 수 있다.

그 서브도메인 값은 `ServerWebExchangeUtils.URI_TEMPLATE_VARIABLES_ATTRIBUTE`이라는 변수에 담겨있다.

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: path_route
        uri: https://example.org
        predicates:
        - Path=/red/{segment}, /blue/{segment}
```

위 Predicate는 요청 Path를 확인하여 Predicate조건에 부합하는지 확인하는 것이다.

그리고 `ServerWebExchange.getAttributes()`구문으로 요청한 Path가 어떤 것인지에 대한 내용도 확인할 수 있다.

그 경로의 값은 `ServerWebExchangeUtils.URI_TEMPLATE_VARIABLES_ATTRIBUTE`이라는 변수에 담겨있다.

### 9. Query

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: query_route
        uri: https://example.org
        predicates:
        - Query=red, gree.
```

위 Predicate는 요청 QueryParameter를 확인하여 Predicate조건에 부합하는지 확인하는 것이다. 조건에는 정규식을 포함할 수 있다.

`red`라는 Query Parameter를 가진 내용이 있거나

`gree`로 시작하는 Query Parameter를 가진 내용이 있으면 Predicate는 참이된다.

### 10. 원격 요청 주소 판별

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: remoteaddr_route
        uri: https://example.org
        predicates:
        - RemoteAddr=192.168.1.1/24
```

위 Predicate는 요청 클라이언트의 주소를 확인하여 Predicate조건에 부합하는지 확인하는 것이다. 요청 주소를 그룹화하기 위해 CIDR를 적용할 수 있다.

그런데 만약 Gateway앞단에 프록시 서버가 있게 된다면 이 `RemoteAddr`은 실제 클라이언트 IP 주소와는 일치하지 않을 수 있다.

이 때 원격 주소가 어떻게 해석되는지를 수정하기 위해 CustomRemoteAddressResolver를 설정할 수 있다.

Spring Cloud Gateway는 `XForwardedRemoteAddressResolver`를 가지고 있으며, 이는 `X-Forwarded-For`헤더를 바라본다.

`XForwardedRemoteAddressResolver`는 두 개의 Static 생성자를 가지고 있다.

- XForwardedRemoteAddressResolver::trustAll
  - X-Forwarded-For 헤더에서 발견된 첫 번째 IP 주소를 사용하는 RemoteAddressResolver를 반환한다.
    - 악의적인 클라이언트가 X-Forwarded-For의 초기 값을 설정하여 스푸핑(야매)을 시도할 수 있다.

- XForwardedRemoteAddressResolver::maxTrustedIndex
  - Spring Cloud Gateway 앞에 실행되는 신뢰할 수 있는 프록시에 대한 인덱스를 사용한다.
  - 예를 들어, Spring Cloud Gateway가 HAProxy를 통해서만 접근 가능하다면, 값으로 1을 사용해야 한다.
    - 두 번의 프록시가 필요하다면, 값으로 2를 사용해야 한다.

만약 X-Forwarded-For: 0.0.0.1, 0.0.0.2, 0.0.0.3 일 때

- maxTrustedIndex: [Integer.MIN_VALUE, 0] -> 초기화 중 IllegalArgumentException 발생 (유효하지 않음)
- maxTrustedIndex: 1 -> 결과: 0.0.0.3
- maxTrustedIndex: 2 -> 결과: 0.0.0.2
- maxTrustedIndex: 3 -> 결과: 0.0.0.1
- maxTrustedIndex: [4, Integer.MAX_VALUE] -> 결과: 0.0.0.1

### 11. 가중치 그룹 판별

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: weight_high
        uri: https://weighthigh.org
        predicates:
        - Weight=group1, 8
      - id: weight_low
        uri: https://weightlow.org
        predicates:
        - Weight=group1, 2
```

위 Predicate는 Weight라는 가중치 그룹을 만들어서 **트래픽의 %로 분산**시킬 수 있는 구문이다.

위 예시에서는 80%의 트래픽이 `weight_high`라는 id에 할당되고, 20%를 `weight_low`라는 id에 할당한다.

---

## Filter

위에서 살펴본 Predicate에 해당하는 요청에 대해 필터를 둘 수 있다.

일반적으로는 들어온 요청에 대한 URI주소를 다시 작성하여 실제 마이크로서비스의 URI로 보낼 수 있다.

```yaml
spring:
  cloud:
    gateway:
      routes:
        - id: ...
          uri: ...
          predicates:
            - ...
          filters:
            - RewritePath=/api/auth/?(?<segment>.*), /$\{segment}
```

라우팅 될 인스턴스가 AuthService이고 주소가 `http://localhost:10001`라고 한다면

위 설정은 클라이언트가 `http://localhost:8000/api/auth`로 요청했다면

실제 요청은 `http://localhost:10001`로 라우팅 해주는 것이고, 요청한 URI 내용 모두 그대로 이어 붙인다는 의미이다.

- `http://localhost:8000/api/auth?testArgument=1` (클라이언트가 요청한 URL)
  - `http://localhost:10001?testArgument=1` (게이트웨이를 통해 라우팅된 URL)

- `http://localhost:8000/api/auth/testPathVariable` (클라이언트가 요청한 URL)
    - `http://localhost:10001/testPathVariable` (게이트웨이를 통해 라우팅된 URL)

---

## Spring Cloud Gateway가 지원하는 30가지 Filter

### 1. 헤더 추가

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: add_request_header_route
        uri: https://example.org
        predicates:
        - Path=/red/{segment}
        filters:
        - AddRequestHeader=X-Request-Red, Blue-{segment}
```

Predicate에 해당하는 요청이라면, `X-Request-Red`라는 헤더에 `Blue-{segment}`라는 값을 추가하여 라우팅을 적용할 수 있다.

### 2. QueryParameter 추가

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: add_request_parameter_route
        uri: https://example.org
        predicates:
        - Host: {segment}.myhost.org
        filters:
        - AddRequestParameter=foo, bar-{segment}
```

Predicate에 해당하는 요청이라면, `foo`라는 QueryParameter에 `bar-{segment}`라는 값을 추가하여 라우팅을 적용할 수 있다.

### 3. 응답헤더 추가

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: add_response_header_route
        uri: https://example.org
        filters:
        - AddResponseHeader=X-Response-Red, Blue
```

응답 헤더에 `X-Response-Red`라는 이름을 갖고 `Blue`라는 값을 추가할 수 있다.

### 4. 중복 응답헤더 제거

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: dedupe_response_header_route
        uri: https://example.org
        filters:
        - DedupeResponseHeader=Access-Control-Allow-Credentials Access-Control-Allow-Origin
```

응답 헤더에 `Access-Control-Allow-Credentials`, `Access-Control-Allow-Origin`의 이름을 가진 중복 응답을 제거한다.

보통 API Gateway 뒷단의 마이크로서비스들이 CORS설정을 추가하는 등의 동일한 헤더 조작을 수행할 때 사용된다.

### 5. 서킷브레이커 적용

API Gateway는 서킷브레이커와 궁합이 좋다. 여기서 서킷 브레이커를 적용하여 Fault Tolerance를 마련할 수 있다.

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: ingredients
        uri: lb://ingredients
        predicates:
        - Path=//ingredients/**
        filters:
        - name: CircuitBreaker
          args:
            name: fetchIngredients
            fallbackUri: forward:/fallback
      - id: ingredients-fallback
        uri: http://localhost:9994
        predicates:
        - Path=/fallback
```

요청 URL 서비스에 서킷 브레이커를 적용하여 서킷 브레이커에 의해 접근이 차단되었을 때 라우팅 될 fallback주소를 명시할 수 있다.

### 6. Fallback Header 적용

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: ingredients
        uri: lb://ingredients
        predicates:
        - Path=//ingredients/**
        filters:
        - name: CircuitBreaker
          args:
            name: fetchIngredients
            fallbackUri: forward:/fallback
      - id: ingredients-fallback
        uri: http://localhost:9994
        predicates:
        - Path=/fallback
        filters:
        - name: FallbackHeaders
          args:
            executionExceptionTypeHeaderName: Test-Header
```

FallbackUri로 전달되는 요청의 헤더에 추가하는 기능이다.

서킷브레이커에 전달할 수 있는 헤더의 속성은 아래와 같다.

- executionExceptionTypeHeaderName ("Execution-Exception-Type")
- executionExceptionMessageHeaderName ("Execution-Exception-Message")
- rootCauseExceptionTypeHeaderName ("Root-Cause-Exception-Type")
- rootCauseExceptionMessageHeaderName ("Root-Cause-Exception-Message")

### 7. Request Header 매핑

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: map_request_header_route
        uri: https://example.org
        filters:
        - MapRequestHeader=Blue, X-Request-Red
```

요청 헤더에 `X-Request-Red`라는 값이 있다면 `Blue`라는 헤더 이름으로 값을 매핑시켜 라우팅을 적용한다.

### 8. Request URI Prefix 적용

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: prefixpath_route
        uri: https://example.org
        filters:
        - PrefixPath=/mypath
```

모든 요청의 경로에 `/mypath`라는 접두사가 붙는다. 즉, `https://example.org/hello`라고 요청한다면 `https://example.org/mypath/hello`로 라우팅이 적용된다.

### 9. PreserveHostHeader (요청 헤더를 클라가 보낸것으로? 서버가 지정한 것으로?)

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: preserve_host_route
        uri: https://example.org
        filters:
        - PreserveHostHeader
```

요청이 프록시 또는 로드 밸런서를 통해 전달될 때 원래의 호스트 헤더를 유지하는 역할이다. 특정 백엔드 서비스가 호스트 헤더에 따라 다르게 동작하는 경우 사용할 수 있다.

위 설정은 클라이언트 측에서 보낸 헤더를 그대로 사용하겠다는 의미이다.

### 10. 요청 제한 (RateLimiter)

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: request_rate_limiter_route
        uri: http://example.org
        filters:
        - name: RequestRateLimiter
          args:
            redis-rate-limiter.replenishRate: 10
            redis-rate-limiter.burstCapacity: 20
        predicates:
        - Path=/api/**
```

RequestRateLimiter는 특정 요청의 비율을 제한하는 데 사용된다.

이 필터는 Redis 또는 Bucket4j와 같은 RateLimiting 구현을 사용하여 사용자가 설정한 요청 빈도를 초과하지 않도록한다.

위 설정에서 `replenishRate`는 토큰이 재충전되는 속도를, `burstCapacity`는 토큰 버킷의 최대 용량을 의미한다.

따라서 위 설정은 `초당 최대 10개`의 요청을 허용하며, 버스트 요청을 처리할 수 있도록 `20개까지의 용량`을 가진다.

### 11. 리다이렉션

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: prefixpath_route
        uri: https://example.org
        filters:
        - RedirectTo=302, https://acme.org
```

`https://example.org`로 요청이 오면 `https://acme.org`로 리다이렉트한다.

### 12. 요청 헤더 제거

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: removerequestheader_route
        uri: https://example.org
        filters:
        - RemoveRequestHeader=X-Request-Foo
```

`X-Request-Foo`에 해당하는 헤더를 지운 후 라우팅을 적용한다.

### 13. 응답 헤더 제거

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: removeresponseheader_route
        uri: https://example.org
        filters:
        - RemoveResponseHeader=X-Response-Foo
```

`X-Response-Foo`에 해당하는 헤더를 지운다.

### 14. 요청 QueryParameter 제거

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: removerequestparameter_route
        uri: https://example.org
        filters:
        - RemoveRequestParameter=red
```

`red`에 해당하는 QueryParameter를 지운 후 라우팅을 적용한다.

### 15. 요청 경로 재작성

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: rewritepath_route
        uri: https://example.org
        predicates:
        - Path=/red/**
        filters:
        - RewritePath=/red(?<segment>/?.*), $\{segment}
```

`https://localhost:8000/red/**`로 들어온 요청을 `https://example.org/**`로 라우팅을 적용한다.

++ YAML 문법에의해 따라 `$`를 표기하려면 `$\`로 대체해야 한다.

### 16. 응답 발원지 경로 재작성

응답이 어떤 서버로부터 왔는지에 대한 정보를 재작성할 수 있다.

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: rewritelocationresponseheader_route
        uri: http://example.org
        filters:
        - RewriteLocationResponseHeader=AS_IN_REQUEST, Location, ,
```

예를 들면 [POST]`api.example.com/some/object/name`요청에 대해 Location 응답 헤더 값인
- object-service.prod.example.net/v2/some/object/id가
- api.example.com/some/object/id로 재작성된다.

이 때 `stripVersionMode`라는 파라미터 지정할 수 있다. (NEVER_STRIP, AS_IN_REQUEST (기본값), ALWAYS_STRIP)

- `NEVER_STRIP`: 원래 요청 경로에 **버전이 없더라도 버전은 제거되지 않는다.**
- `AS_IN_REQUEST`: 원래 요청 경로에 **버전이 없는 경우에만 버전이 제거**된다.
- `ALWAYS_STRIP`: 원래 요청 경로에 **버전이 포함되어 있더라도 버전은 항상 제거**된다.


---

# Spring Cloud Gateway 셋팅

## build.gradle(.kts) 의존성 추가

```groovy
extra["springCloudVersion"] = "2023.0.0"

dependencies {
    implementation("org.springframework.cloud:spring-cloud-starter-netflix-eureka-client")
    implementation("org.springframework.cloud:spring-cloud-starter-gateway")
}

dependencyManagement {
    imports {
        mavenBom("org.springframework.cloud:spring-cloud-dependencies:${property("springCloudVersion")}")
    }
}
```

추가적으로, Service Discovery와 궁합이 좋기 때문에 이를 활용하여 라우팅에 도움을 받는 것이 좋다.

### rootApplication

```java
@SpringBootApplication
@EnableDiscoveryClient
public class GatewayServiceApplication {

    public static void main(String[] args) {
        SpringApplication.run(GatewayServiceApplication.class, args);
    }

}
```

위와 같이 루트에 ServiceDiscovery에 등록하기 위한 `@EnableDiscoveryClient` 애노테이션을 달면 셋팅은 끝난다.

---

## Gateway Filter

