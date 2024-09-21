---

title: Spring Cloud Gateway 공식문서 톺아보기와 적용
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

---

# Route

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

# Predicate

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

## Spring Cloud Gateway가 지원하는 11가지 Predicate Factory

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

위 Predicate는 요청 Header를 확인하여 **Header 이름**이 `X-Request-Id`인 내용이 있다면 그 값이 **정규식**에 해당하는 `\d+`에 해당하는지 확인한다.

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

Spring Cloud Gateway는 `XForwardedRemoteAddressResolver`를 가지고 있으며, 이는 `X-Forwarded-For`Header를 바라본다.

`XForwardedRemoteAddressResolver`는 두 개의 Static 생성자를 가지고 있다.

- XForwardedRemoteAddressResolver::trustAll
  - X-Forwarded-For Header에서 발견된 첫 번째 IP 주소를 사용하는 RemoteAddressResolver를 반환한다.
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

# Filter

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

## Spring Cloud Gateway가 지원하는 30가지 Filter Factory

### 1. Header 추가

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

Predicate에 해당하는 요청이라면, `X-Request-Red`라는 Header에 `Blue-{segment}`라는 값을 추가하여 라우팅을 적용할 수 있다.

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

### 3. 응답 Header 추가

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

응답 Header에 `X-Response-Red`라는 이름을 갖고 `Blue`라는 값을 추가할 수 있다.

### 4. 중복 응답 Header 제거

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

응답 Header에 `Access-Control-Allow-Credentials`, `Access-Control-Allow-Origin`의 이름을 가진 중복 응답을 제거한다.

보통 API Gateway 뒷단의 마이크로서비스들이 CORS설정을 추가하는 등의 동일한 Header 조작을 수행할 때 사용된다.

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

FallbackUri로 전달되는 요청의 Header에 추가하는 기능이다.

서킷브레이커에 전달할 수 있는 Header의 속성은 아래와 같다.

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

요청 Header에 `X-Request-Red`라는 값이 있다면 `Blue`라는 Header 이름으로 값을 매핑시켜 라우팅을 적용한다.

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

### 9. PreserveHostHeader (요청 Header를 클라가 보낸것으로? 서버가 지정한 것으로?)

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

요청이 프록시 또는 로드 밸런서를 통해 전달될 때 원래의 호스트 Header를 유지하는 역할이다. 특정 백엔드 서비스가 호스트 Header에 따라 다르게 동작하는 경우 사용할 수 있다.

위 설정은 클라이언트 측에서 보낸 Header를 그대로 사용하겠다는 의미이다.

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

### 12. 요청 Header 제거

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

`X-Request-Foo`에 해당하는 Header를 지운 후 라우팅을 적용한다.

### 13. Response Header 제거

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

`X-Response-Foo`에 해당하는 Header를 지운다.

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

### 16. Response 발원지 경로 재작성

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

예를 들면 [POST]`api.example.com/some/object/name`요청에 대해 Location Response Header 값인
- object-service.prod.example.net/v2/some/object/id가
- api.example.com/some/object/id로 재작성된다.

이 때 `stripVersionMode`라는 파라미터 지정할 수 있다. (NEVER_STRIP, AS_IN_REQUEST (기본값), ALWAYS_STRIP)

- `NEVER_STRIP`: 원래 요청 경로에 **버전이 없더라도 버전은 제거되지 않는다.**
- `AS_IN_REQUEST`: 원래 요청 경로에 **버전이 없는 경우에만 버전이 제거**된다.
- `ALWAYS_STRIP`: 원래 요청 경로에 **버전이 포함되어 있더라도 버전은 항상 제거**된다.

### 17. Response Header 재작성

`Header name`, `regexp`, `replacement`를 매개변수로 받아 Response Header를 재작성한다.

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: rewriteresponseheader_route
        uri: https://example.org
        filters:
        - RewriteResponseHeader=X-Response-Red, , password=[^&]+, password=***
```

위 예시는 `X-Response-Red`라는 Response Header 값을 ` password=[^&]+`로 정규식 검사한뒤 `password=***`로 대체한다.

### 18. 세션 저장 (마이크로서비스 간 세션 공유)

WebSession::save 작업을 강제로 수행하도록 한 후 라우팅을 적용한다.

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: save_session
        uri: https://example.org
        predicates:
        - Path=/foo/**
        filters:
        - SaveSession
```

Spring Security와 Spring Session을 통합하고 다른 마이크로서비스간 인증/인가가 수행되었음을 보장하려는 경우에 이 기능은 필수이다.

### 19. 보안 Header 추가

```yaml
spring:
    cloud:
      gateway:
        filter:
          secure-headers:
            disable=x-frame-options,strict-transport-security

```

요청/응답 간 [다음 보안 Header들](https://blog.appcanary.com/2017/http-security-headers.html)이 추가 된다.

- X-Xss-Protection:1 (mode=block)
- Strict-Transport-Security (max-age=631138519)
- X-Frame-Options (DENY)
- X-Content-Type-Options (nosniff)
- Referrer-Policy (no-referrer)
- Content-Security-Policy (default-src 'self' https:; font-src 'self' https: data:; img-src 'self' https: data:; object-src 'none'; script-src https:; style-src 'self' https: 'unsafe-inline')
- X-Download-Options (noopen)
- X-Permitted-Cross-Domain-Policies (none)

### 20. Request Path 셋팅

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: setpath_route
        uri: https://example.org
        predicates:
        - Path=/red/{segment}
        filters:
        - SetPath=/{segment}
```

- SetPath
  - 이 필터는 경로 템플릿 매개변수를 받아 요청 경로를 조작한다. 템플릿화된 경로 세그먼트를 허용하여 요청 경로를 변경할 수 있다.
- RewritePath
  - 이 필터는 경로의 일부를 다른 값으로 대체할 수 있게 해주는 정규 표현식(regex) 매개변수를 받아 좀 더 복잡한 경로 변형을 할 수 있다.

### 21. Request Header 셋팅

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: setrequestheader_route
        uri: https://example.org
        filters:
        - SetRequestHeader=X-Request-Red, Blue
```

요청 헤더에 `X-Request-Red`로 들어온 값을 Blue로 대체한다.

### 22. Response Header 셋팅

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: setresponseheader_route
        uri: https://example.org
        filters:
        - SetResponseHeader=X-Response-Red, Blue
```

응답 헤더의 `X-Response-Red`값을 Blue로 대체한다.

### 23. Response Status 셋팅

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: setstatusstring_route
        uri: https://example.org
        filters:
        - SetStatus=BAD_REQUEST
      - id: setstatusint_route
        uri: https://example.org
        filters:
        - SetStatus=401
```

응답 상태 코드를 조작할 수 있다. 정수 값 `401`, `404` 등을 넣거나 `NOT_FOUND`와 같은 문자열로 표기할 수 있다.

```yaml
spring:
  cloud:
    gateway:
      set-status:
        original-status-header-name: original-http-status
```

위와 같이 원래의 응답 코드를 헤더에 따로 담을 수도 있다.

### 24. Request Path StripPrefix 적용

이 필터는 parts라는 하나의 매개변수를 받는다. 요청 경로에 제거할 부분의 수를 나타낸다.

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: nameRoot
        uri: https://nameservice
        predicates:
        - Path=/name/**
        filters:
        - StripPrefix=2
```

/name/foo/bar라는 요청 경로가 있다면, `StripPrefix=2`옵션은 `/name/foo 부분을 제거`하고, `/bar`로 요청 경로를 변경한다.

### 25. Retry

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: retry_test
        uri: http://localhost:8080/flakey
        predicates:
        - Host=*.retry.com
        filters:
        - name: Retry
          args:
            retries: 3
            statuses: BAD_GATEWAY
            methods: GET,POST
            backoff:
              firstBackoff: 10ms
              maxBackoff: 50ms
              factor: 2
              basedOnPreviousValue: false
```

최대 재시도 횟수, 응답 Status Code + Method, backoff를 매개변수로 이 조건들에 부합한다면 재시도를 할 수 있도록 한다.

### 26. Request Size Set

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: request_size_route
        uri: http://localhost:8080/upload
        predicates:
        - Path=/upload
        filters:
        - name: RequestSize
          args:
            maxSize: 5000000
```

요청 데이터의 크기를 제한할 수 있다.

### 27. Request Host Set

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: set_request_host_header_route
        uri: http://localhost:8080/headers
        predicates:
        - Path=/headers
        filters:
        - name: SetRequestHost
          args:
            host: example.org
```

요청을 허용할 호스트를 지정할 수 있다.

### 28. Modify Request Body

```java
@Bean
public RouteLocator routes(RouteLocatorBuilder builder) {
    return builder.routes()
        .route("rewrite_request_obj", r -> r.host("*.rewriterequestobj.org")
            .filters(f -> f.prefixPath("/httpbin")
                .modifyRequestBody(String.class, Hello.class, MediaType.APPLICATION_JSON_VALUE,
                    (exchange, s) -> return Mono.just(new Hello(s.toUpperCase())))).uri(uri))
        .build();
}

static class Hello {
    String message;

    public Hello() { }

    public Hello(String message) {
        this.message = message;
    }

    public String getMessage() {
        return message;
    }

    public void setMessage(String message) {
        this.message = message;
    }
}
```

요청 받은 JSON Body를 Class로 역직렬화하여 요청 본문을 조작할 수 있다.

### 29. Modify Response Body

```java
@Bean
public RouteLocator routes(RouteLocatorBuilder builder) {
    return builder.routes()
        .route("rewrite_response_upper", r -> r.host("*.rewriteresponseupper.org")
            .filters(f -> f.prefixPath("/httpbin")
                .modifyResponseBody(String.class, String.class,
                    (exchange, s) -> Mono.just(s.toUpperCase()))).uri(uri)
        .build();
}
```

응답할 Body를 조작할 수 있다.

### 30. Default Filters (모든 route 설정에 필터를 적용하기)

```yaml
spring:
  cloud:
    gateway:
      default-filters:
      - AddResponseHeader=X-Response-Default-Red, Default-Blue
      - PrefixPath=/httpbin
```

---

## Gateway Global Filter간 순서 정하기

```java
@Bean
public GlobalFilter customFilter() {
    return new CustomGlobalFilter();
}

public class CustomGlobalFilter implements GlobalFilter, Ordered {

    @Override
    public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain) {
        log.info("custom global filter");
        return chain.filter(exchange);
    }

    @Override
    public int getOrder() {
        return -1;
    }
}
```

GlobalFilter를 상속받은 CustomFilter를 구현했을 때 이 필터를 적용하기 위한 우선순위를 지정할 수 있다.

숫자가 낮을수록 먼저 수행된다.

## Gateway Metric 생성 (actuator)

`spring-boot-starter-actuator`의존성을 추가하면

```yaml

spring:
  cloud:
    gateway:
      metrics: enabled
```

위 설정이 자동으로 활성화 되어 Gateway의 라우팅 지표를 남길 수 있게된다. (/actuator/metrics/gateway.requests)

지표의 속성은 아래와 같다.

- `routeId`
  - 라우팅 ID
- `routeUri`
  - 라우팅 된 주소
- `outcome`
  - 실제 HTTP 응답 Status
- `status`
  - 클라이언트에 반환된 HTTP Status
- `httpStatusCode`
  - 클라이언트에 반환된 HTTP Status
- `httpMethod`
  - 요청 메서드

---

# Gateway Timeout 설정

아래와 같이 모든 라우팅에 타임아웃을 적용할 수 있다.

```yaml
spring:
  cloud:
    gateway:
      httpclient:
        connect-timeout: 1000
        response-timeout: 5s
```

- `connect-timeout`은 **milliseconds**단위이다.
- `response-timeout`은 **java.time.Duration**으로 변환될 수 있는 단위이다.

또한 각 라우팅마다 다르게 설정하고 싶다면 아래와 같이 작성할 수 있다.

```yaml
spring:
  cloud:
    gateway:
      - id: per_route_timeouts
        uri: https://example.org
        predicates:
          - name: Path
            args:
              pattern: /delay/{timeout}
        metadata:
          response-timeout: 200
          connect-timeout: 200
```

---

# Gateway CORS 허용

아래와 같이 모든 라우팅에 CORS를 허용하는 구문을 만들 수 있다.

```yaml
spring:
  cloud:
    gateway:
      globalcors:
        cors-configurations:
          '[/**]':
            allowedOrigins: "https://docs.spring.io"
            allowedMethods:
            - GET
```

# Gateway Actuator 활성화 하기

```yaml
management:
    endpoint:
      gateway:
        enabled: true # default value
      web:
        exposure:
          include: gateway
```

위 설정으로 액추에이터를 활성화 시킨 후 Endpoint로 접속하여 모니터링을 할 수 있다.

/actuator/gateway를 기본 경로로 가진다.

| ID | HTTP Method | 설명                                 |
|---|---|------------------------------------|
| /actuator/gateway/globalfilters | GET | Route에 적용된 Global Filter 목록 표시       |
| /actuator/gateway/routefilters | GET | 특정 Route에 적용된 GatewayFilter 팩토리 목록 표시 |
| /actuator/gateway/refresh | POST | Route 캐시 제거                        |
| /actuator/gateway/routes | GET | 게이트웨이에 정의된 Route 목록을 표시          |
| /actuator/gateway/routes/{id} | GET | 특정 Route에 대한 정보를 표시              |
| /actuator/gateway/routes/{id} | POST | 게이트웨이에 새로운 Route를 추가             |
| /actuator/gateway/routes/{id} | DELETE | 게이트웨이에서 기존 Route를 제거             |

---

# Custom Predicate 작성하기

```java
public class MyRoutePredicateFactory extends AbstractRoutePredicateFactory<HeaderRoutePredicateFactory.Config> {

    public MyRoutePredicateFactory() {
        super(Config.class);
    }

    @Override
    public Predicate<ServerWebExchange> apply(Config config) {
        return exchange -> {
            ServerHttpRequest request = exchange.getRequest();

            /*
            이 부분에서 Predicate 조건 분기문 둥 커스텀 내용 만들기
             */

            return matches(config, request);
        };
    }

    public static class Config {}

}
```

---

# Custom PreFilter 작성하기

공식문서에 따르면 커스텀 필터를 만들 때 그 네이밍은 `~~~GatewayFilterFactory`로 끝나야한다.

`GatewayFilterFactory` 접미사가 없는 이름의 게이트웨이 필터를 만들고 yml파일에서 참조할 수 있긴하지만 이렇게 참조할 수 있는 방법은 향후 릴리스에서 제거될 수 있다.

```java
public class PreGatewayFilterFactory extends AbstractGatewayFilterFactory<PreGatewayFilterFactory.Config> {

    public PreGatewayFilterFactory() {
        super(Config.class);
    }

    @Override
    public GatewayFilter apply(Config config) {
        return (exchange, chain) -> {
            ServerHttpRequest.Builder builder = exchange.getRequest().mutate();

            /*
            이곳에 요청값 조작 등 처음에 수행될 필터 내용 작성
             */

            return chain.filter(exchange.mutate().request(builder.build()).build());
        };
    }

    public static class Config {}

}
```

---

# Custom PostFilter 작성하기

공식문서에 따르면 커스텀 필터를 만들 때 그 네이밍은 `~~~GatewayFilterFactory`로 끝나야한다.

`GatewayFilterFactory` 접미사가 없는 이름의 게이트웨이 필터를 만들고 yml파일에서 참조할 수 있긴하지만 이렇게 참조할 수 있는 방법은 향후 릴리스에서 제거될 수 있다.

```java
public class PostGatewayFilterFactory extends AbstractGatewayFilterFactory<PostGatewayFilterFactory.Config> {

    public PostGatewayFilterFactory() {
        super(Config.class);
    }

    @Override
    public GatewayFilter apply(Config config) {
        return (exchange, chain) -> {
            return chain.filter(exchange).then(Mono.fromRunnable(() -> {
                ServerHttpResponse response = exchange.getResponse();

                /*
                    이곳에 응답값 조작 등 마지막에 수행될 필터 내용 작성
                 */
            }));
        };
    }

    public static class Config {}

}
```

---

# Custom Global Filter 작성하기

`Custom Global Filter`를 작성하려면 `GlobalFilter 인터페이스를 구현`해야한다. 이 필터는 모든 요청에 적용된다.

```java
@Bean
public GlobalFilter customGlobalFilter() {
    return (exchange, chain) -> exchange.getPrincipal()
        .map(Principal::getName)
        .defaultIfEmpty("Default User")
        .map(userName -> {
            exchange.getRequest().mutate().header("CUSTOM-REQUEST-HEADER", userName).build();
            return exchange;
        })
        .flatMap(chain::filter);
}

@Bean
public GlobalFilter customGlobalPostFilter() {
    return (exchange, chain) -> chain.filter(exchange)
        .then(Mono.just(exchange))
        .map(serverWebExchange -> {
            serverWebExchange.getResponse().getHeaders().set("CUSTOM-RESPONSE-HEADER",
                HttpStatus.OK.equals(serverWebExchange.getResponse().getStatusCode()) ? "It worked": "It did not work");
          return serverWebExchange;
        })
        .then();
}
```

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

## rootApplication

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

## application.yml

```yaml
server:
  port: 8000

spring:
  application:
    name: gateway-service

  main:
    web-application-type: reactive

  cloud:
    gateway:
      globalcors:
        cors-configurations:
          '[/**]':
            allowedOrigins: [ "http://localhost:5173", "http://127.0.0.1:5173" ]
            allow-credentials: true
            allowedHeaders: '*'
            allowedMethods:
              - PUT
              - GET
              - POST
              - DELETE
              - OPTIONS
      routes:
        - id: AUTH-SERVICE
          uri: lb://AUTH-SERVICE
          predicates:
            - Path=/api/auth/**
          filters:
            - RewritePath=/api/auth/?(?<segment>.*), /$\{segment}

        - id: USER-SERVICE
          uri: lb://USER-SERVICE
          predicates:
            - Path=/api/users/,
              /api/users/verify-email,
              /api/users/verify-username,
              /api/users/temporary-join,
              /api/users/join,
              /api/users/web-logout,
              /api/users/profile,
              /api/users/{id},
              /api/users/me
          filters:
            - RewritePath=/api/users/(?<segment>/?.*), /$\{segment}

        - id: TIMELINE-SERVICE
          uri: lb://TIMELINE-SERVICE
          predicates:
            - Path=/api/timeline/**
          filters:
            - RewritePath=/api/timeline/?(?<segment>.*), /$\{segment}

        - id: SOCIAL-SERVICE
          uri: lb://SOCIAL-SERVICE
          predicates:
            - Path=/api/paints/**,
              /api/users/{id}/following,
              /api/users/{id}/follower,
              /api/users/{id}/verified_follower,
              /api/users/{id}/following,
              /api/users/{id}/following,
              /api/users/{id}/paint,
              /api/users/{id}/reply,
              /api/users/{id}/media,
              /api/users/{id}/like,
              /api/users/{id}/like/{paintId},
              /api/users/{id}/repaint,
              /api/users/{id}/repaint/{sourcePaintId}

        - id: TRENDS-SERVICE
          uri: lb://TRENDS-SERVICE
          predicates:
            - Path=/api/trends/**
          filters:
            - RewritePath=/api/trends/?(?<segment>.*), /$\{segment}

        - id: SEARCH-SERVICE
          uri: lb://SEARCH-SERVICE
          predicates:
            - Path=/api/search/**
          filters:
            - RewritePath=/api/search/?(?<segment>.*), /$\{segment}

        - id: DM-SERVICE
          uri: lb://DM-SERVICE
          predicates:
            - Path=/api/dm/**
          filters:
            - RewritePath=/api/dm/?(?<segment>.*), /$\{segment}

        - id: NOTIFICATION-SERVICE
          uri: lb://NOTIFICATION-SERVICE
          predicates:
            - Path=/api/notification/**
          filters:
            - RewritePath=/api/notification/?(?<segment>.*), /$\{segment}

      default-filters:
        - name: AuthorizationGatewayFilterFactory
          args:
            baseMessage: Gateway Authorization Filter
            preLogger: true
            postLogger: true

eureka:
  instance:
    hostname: localhost
  client:
    fetch-registry: true
    register-with-eureka: true
    service-url:
      defaultZone: http://host.docker.internal:8761/eureka

```

위와 같이 커스텀 필터를 모든 라우팅에 적용하고 유레카를 통해 각 마이크로서비스에 로드밸런싱 하는 설정을 적용할 수 있다.
