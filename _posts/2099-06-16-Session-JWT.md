---

title: Server Side Session vs Token
date: 2022-06-16
categories: [Session, Token]
tags: [Session, Token]
layout: post
toc: true
math: true
mermaid: true

---

# 참고자료


[Baeldung Document](https://www.baeldung.com/cs/tokens-vs-sessions)

[Redis Document](https://redis.com/blog/json-web-tokens-jwt-are-dangerous-for-user-sessions/)

---

# 들어가기 전

## 인증(Authentication) vs 인가(Authorization)

`인증(Authentication) : 요청한 유저가 누구인지 확인하는 과정`

`인가(Authorization) : 인증된 유저가 권한이 있는지 확인하는 과정`

---


# 토큰기반 인증 방식 (:JWT)

![](https://www.baeldung.com/wp-content/uploads/sites/4/2021/04/token.png)

위 그림을 천천히 살펴보면 JWT인증 방식의 흐름을 이해할 수 있다.

사용자가 로그인 시도(인증 행위)를 수행하면 Token Server가 이 진위여부를 판단하여 토큰을 발급해 준다.

사용자는 발급받은 토큰을 통해 API를 요청하여 서비스를 이용할 수 있게 되는 것이다.

기본 인증 방식이라고 불리는 BASE64 인코딩 기반의 단순 인증 방식의 단점을 보완한 방식이다.

---

## 토큰 기반 인증 방식의 장점 1

발급 받은 토큰은 해당 인프라에서 어디든지 사용할 수 있다.

A라는 플랫폼이 있고 이 플랫폼에 해당하는 분산 서버가 5대 있다고 하자.

각각 다른 서버임에도 불구하고 인증을 성공적으로 수행하여 발급받은 토큰은 어떤 서버든 관계없이 사용할 수 있다.

---

## 토큰 기반 인증 방식의 장점 2

토큰 기반 인증이라고 한다면 보통 JWT사용하는게 일반적인데 JWT는 구현 방식이 간단하다.

이미 Well Known한 라이브러리가 준비되어있기 때문이다.

---

## 토큰 기반 인증 방식의 장점 3

앞서 말한 JWT는 보통 해시 알고리즘으로 만들어진다. 해시는 단방향이므로 복호화가 불가능하다.

해독이 불가능 한 것은 아니나 레인보우 테이블을 돌린다 하더라도 해독 시간은 어마어마하게 오래 걸린다.

보통 액세스 토큰은 만료기간이 다소 짧기 마련인데 해시 복호화를 때려맞추는 동안 그 토큰은 만료되는게 일반적인 시나리오이다.

---

## 토큰 기반 인증 방식의 단점 1

토큰이 탈취된다면 발급한 서버쪽에서 특별한 조치를 취할 수 없다.

---

## 토큰 기반 인증 방식의 단점 2

토큰을 서명하는 Key가 노출되면 모든 액세스 키를 만들어 사용하는 악용 사례가 발생하게 된다.

---

## 토큰을 어떻게 검증하는건가?

JWT는 `헤더`, `페이로드`, `서명`으로 구성되어있다.

1. 클라이언트에서 AccessToken을 보내면 해당 토큰의 페이로드를 추출한다.

이 때 헤더, 페이로드는 `.` 으로 구분되어 추출하는 것이며 페이로드는 해시되지 않은 claim의 형태를 가지기 때문에 페이로드를 추출 할 수 있는 것이다.

2. 추출한 헤더와 Payload를 바탕으로 서버가 가진 Secret Key를 활용하여 해시한다.

3. 해시한 값이 요청한 AccessToken과 같은지 비교하는 것으로 검증을 마친다.


## 토큰 기반 인증 방식의 보완점

토큰이 탈취되면 손을 쓸 수 없다는 단점을 보완하기위해 AccessToken과 RefreshToken을 함께 사용하는 개념이 등장했다.

AccessToken의 만료기간을 짧게 가져가 혹여나 탈취당하더라도 해당 토큰의 사용가능한 시간을 줄여 피해를 줄이려는 목적과

짧은 만료기간으로 인해 서비스를 주기적으로 로그인해야하는 불편함을 개선하기 위해 RefreshToken을 발급하여

토큰을 갱신하는 API를 요청함으로써 자동 로그인을 유지할 수 있도록 하는 방법이 등장한 것이다.

하지만, 마찬가지로 토큰을 탈취당하거나 Secret Key가 노출되었을 때의 사후 대책은 여전히 해결하지 못한다.

---

# 세션 방식 인증

![](https://www.baeldung.com/wp-content/uploads/sites/4/2021/04/sessions.png)

서버 내부에서 세션 인프라를 사용하는 방식이다. 스프링은 Tomcat과 같은 컨테이너에서 사용할 수 있다.

생성된 세션은 메모리 혹은 DB에 저장하여 관리하기도 한다.

Tomcat 에서 Session을 생성하는 실제 코드를 살펴보면 다음과 같다.

```java
package org.apache.catalina.util;

public class StandardSessionIdGenerator extends SessionIdGeneratorBase {

    @Override
    public String generateSessionId(String route) {

        byte random[] = new byte[16];
        int sessionIdLength = getSessionIdLength();

        // Render the result as a String of hexadecimal digits
        // Start with enough space for sessionIdLength and medium route size
        StringBuilder buffer = new StringBuilder(2 * sessionIdLength + 20);

        int resultLenBytes = 0;

        while (resultLenBytes < sessionIdLength) {
            getRandomBytes(random);
            for (int j = 0;
            j < random.length && resultLenBytes < sessionIdLength;
            j++) {
                byte b1 = (byte) ((random[j] & 0xf0) >> 4);
                byte b2 = (byte) (random[j] & 0x0f);
                if (b1 < 10)
                    buffer.append((char) ('0' + b1));
                else
                    buffer.append((char) ('A' + (b1 - 10)));
                if (b2 < 10)
                    buffer.append((char) ('0' + b2));
                else
                    buffer.append((char) ('A' + (b2 - 10)));
                resultLenBytes++;
            }
        }

        if (route != null && route.length() > 0) {
            buffer.append('.').append(route);
        } else {
            String jvmRoute = getJvmRoute();
            if (jvmRoute != null && jvmRoute.length() > 0) {
                buffer.append('.').append(jvmRoute);
            }
        }

        return buffer.toString();
    }
}
```

1. 16바이트 단위로 랜덤 값을 생성한 후 문자열 형태의 16진수로 변환하는게 핵심 로직이다.

2. 라우트 값이 있는 경우, 랜덤값 뒤에 "." 문자와 같이 라우트 값을 합친다.

3. 마지막으로 jvmRoute 값을 "." 문자와 같이 합친다.

Route 값을 자꾸 합치는 이유는 만에 하나라도 Random값이 중복되는 경우를 방지하기 위함일 것으로 생각된다.

---

## 세션 기반 인증 방식의 장점 1

세션을 서버에서 직접 관리할 수 있다. 특정 세션을 만료시킬 수 있다는 것인데, 만약 세션 탈취가 의심된다면 해당 세션을 만료시켜버리면 공격에 대한 사후 관리가 가능하다.

---

## 세션 기반 인증 방식의 장점 2

서버 내부에서 세션 상태를 관리하므로 특별한 추가 작업 없이도 세션 데이터를 사용할 수 있다.

---

## 세션 기반 인증 방식의 단점 1 - 확장성

보통 회사만 가도 MSA가 기본적으로 깔려 있을 텐데 세션은 서버 측에서 상태를 관리하기 때문에,

서버가 수평 확장되는 경우 세션 데이터의 공유나 중앙 집중화 문제를 해결해야 할 수도 있다.

서버 A, B, C 가 있다고 했을 때 서버 A에서 발급한 세션ID를 B, C에서도 공유해야 API를 사용할 수 있기 때문에 이를 공유할 수 있는 클러스터가 필요한 것이다. 또한 이 클러스터가 장애가 발생한다면 모든 서버에 그 영향이 전파된다는 문제점이 존재한다.

세션 사용은 서버에 따라 스토리지에 대한 액세스에 따라 다르므로 동일한 클라이언트의 모든 호출이 단일 서버에 도달하거나 서버 간에 세션 복제를 구성해야 한다.


---

# 토큰 vs 세션

토큰은 구현이 간편하고 확장성이 뛰어나다. 또한 해시된 페이로드 자체를 해독하는데 상당한 시간을 필요로 하기 때문에 어느정도 보안성도 갖추고 있다.

세션은 보안성이 비교적 높다고 볼 수 있을 것 같다. 서버측에서 인증 정보를 관리하기 때문이다. 하지만 수평적인 확장이 간단하지 않다는 점이 단점으로 꼽힌다.

# 그럼 어떤걸 선택해야할까?

정답이 있는 문제인가 싶지만 내 경험에 빗대면 이렇게 생각한다.

은행, 금융권 같은 곳에는 주로 Session을 사용한다고 한다. 꼭 은행과 금융권이 아닌 도메인들도 보안이 무척 중요하지만 이 도메인들은 특히나 보안성이 강조되는 도메인이다.

토큰과 세션을 비교하면 `성능/확장성 vs 보안성`을 트레이드 오프 해야하는 것을 보이는데 보안성이 더 강조되는 도메인이라면 주저없이 세션을 사용하는게 더 좋을 것으로 생각된다.

하지만 생산성과 확장성도 무시할 항목은 절대 아니며 토큰 방식의 인증 또한 충분히 강력한 보안성을 가지고 있기 때문에 상황에 맞게 선택하는게 어떨까?

# 선택 후 고려해야할 점

세션과 토큰 모두 결국에는 클라이언트도 그 정보를 가지고 있어야한다.

즉, HTTP 환경에서 세션과 JWT가 떠다닌다는 것인데 이를 관리하는 것이 아주 중요하다.

그래서 전 세계적으로 가장 많이 시도되는 공격인 JavaScript기반의 XSS 공격을 잘 방어할 줄 알아야하고 CSRF 공격 또한 대비를 해둬야한다.

이를 위해서 Spring 기반의 애플리케이션에서는 Cookie와 Spring Security를 사용하는게 일반적이다.

쿠키는 HTTP Only 옵션을 통해 JavaScript로 접근하는 XSS공격을 어느정도 방어할 수 있으며, Spring Security의 CSRF disable 옵션을 통해 CSRF 공격 또한 예방할 수 있다.

