---

title: URI 기깔나게 설계하기 (그런 REST API로 괜찮은가)
author: 김도현
date: 2023-05-23
categories: [URI, URL]
tags: [URI, URL]
math: true
mermaid: true

---

# 참고자료

[Microsoft Doc](https://learn.microsoft.com/en-us/azure/architecture/best-practices/api-design#organize-the-api-design-around-resources)

[그런 REST API로 괜찮은가](https://www.youtube.com/watch?v=RP_f5dMoHFc&t=700)

# REST?

REST란 아키텍처 접근 방식 중 하나이다. 분산 하이퍼미디어 시스템(WEB)을 위한 아키텍처 스타일을 가리킨다.

아키텍처 스타일은 제약조건의 집합을 의미한다.

---

# REST 제약조건

- uniform interface
  - Identification of Resources(리소스가 URI로 식별되면 된다.)
  - Manipulation of Resources Through Representation(HTTP 메서드로 행위를 명시하면 된다.)
  - Self-Descriptive Message
  - Hypermedia as The Engine of Application State(HAETOAS)
- client-server
- stateless
- cache
- layered-system
- code-on-demand (JS)

uniform interface를 제외한 제약조건은 HTTP를 사용한다면 대부분 만족하고 있는 제약조건이다.

그럼 가장 문제가 되는 uniform interface에 대해 자세히 알아보자.

---

# REST 제약조건 - Uniform Interface

- uniform interface
    - Identification of Resources(리소스가 URI로 식별되면 된다.)
    - Manipulation of Resources Through Representation(HTTP 메서드로 행위를 명시하면 된다.)
    - Self-Descriptive Message
    - Hypermedia as The Engine of Application State(HAETOS)

리소스가 URI로 식별되는 것, HTTP 메서드로 행위를 구분하는 것은 REST API란 무엇인가? 등 구글에 검색해보면 똑같은 소리를 다하고 있다.

하지만 진짜 REST를 위해서는 Self-Descriptive Message와 Hypermedia as The Engine of Application State(HAETOS)까지도 만족을 해야하는데

이게 좀 어려운 것이다.

## Self-Descriptive Message

메세지가 스스로를 설명할 수 있어야한다.

### 요청 시 메세지의 예시

```text
GET / HTTP/1.1
```

위 HTTP 요청 메세지는 Self-Descriptive 하지 못한다. 위 메세지를 보고 뭘 하는지 모르기 때문이다.

```text
GET / HTTP/1.1 Host: www.example.org`
```

위 HTTP 요청 메세지는 이제 Self-Descriptive하다.

www.example.org 라는 목적지에 GET요청을 한다는 것을 설명하고 있기 때문이다.

### 응답 시 메세지의 예시

```text
Http/1.1 200 OK

[ {"op": "remove", "path": "/a/b/c} ]
```

위 응답 메세지는 Self-Descriptive를 만족하지 못한다. 어떤 문법으로 응답이 온건지 설명이 안되기 때문이다.

```text
Http/1.1 200 OK
Content-Type: application/json

[ {"op": "remove", "path": "/a/b/c} ]
```

이렇게 수정하면 Self-Descriptive를 만족한다고 볼 수 있다.

하지만 완전하게 지켰다고는 할 수 없는게 `"op"`, `"path"` 등 온전히 이해할 수 없는 속성을 가지기 때문에

이 데이터를 내려주는 명세를 참고해야하기 때문이다.

## HATEOAS

애플리케이션의 상태는 Hyperlink를 이용해 전이되어야한다.

아래 그림이 잘 설명해주는 예시이다.

![애플리케이션 상태의 전이](https://github.com/K-Diger/K-Diger.github.io/blob/main/images/awesome-uri/img_2.png?raw=true)

위 표현을 JSON으로 나타내면 다음과 같다.

```text
HTTP/1.1 200 OK
Content-Type: application/json
Link: </articles/1>; rel="previous",
      </articles/3>; rel="next";
{
    "title": "두 번째 기사",
    "content": "두 번째 기사의 내용"
}
```

헤더에 이전 게시물, 다음 게시물의 링크를 가리킬 수 있는 내용을 추가하면 되는 것이다.

링크헤더는 실제로 표준으로 명세되어있기 때문에 HATEOAS를 만족할 수 있다고 볼 수 있다.

---

# 왜 Uniform-Interface를 지켜야하지?

클라이언트-서버가 각각 독립적으로 진화할 수 있다.

서버의 기능이 변경되어도 클라이언트가 업데이트 할 필요가 없다.

왜 독립적으로 진화가 되는지 더 자세히 알아보면 아래 제약 조건의 특징을 꼽을 수 있다.

## Self-Descriptive

서버나 클라이언트가 변경되더라도 오고가는 메세지는 언제나 Self-Descriptive 하기 때문에 해석이 가능하다.

서버가 어떻게 되든간에 해석이 가능하다는 것

## HATEOAS

애플리케이션 상태 전이의 Late Binding이 가능하다.

링크를 마음대로 바꿀 수 있다. 서버가 내려주는 링크를 바꿔도 클라이언트는 그걸 따라가기만해도 된다. (링크가 동적으로 변경될 수 있다.)

---

# REST API?

REST API는 REST 아키텍처 스타일을 따라야하지만 그렇지 않은 경우가 너무 많다.

특히 Uniform-Interface의 제약조건인 Self-Descriptvie, HAETOAS를 지키기가 꽤나 번거롭고 어렵다.

꼭 위에서 언급한 아키텍처 스타일을 다 지켜야하는가 하면 꼭 지켜야 진정한 REST라고 부른다고 한다. (by REST 개념의 창시자)

하지만 REST로 꼭 API를 만들어야하는건 아니다.

시스템 전체를 통제할 수 있거나 진화에 관심이 없다면 굳이 적용하지 않아도 된다.

시스템 전체를 통제한다는 것은 클라이언트 부분을 의도대로 변경할 수 있냐는 것이다.

---

# 시적 허용

1. REST API를 구현하고 REST API라고 부른다.
2. REST API구현을 포기하고 HTTP API라고 부른다.
3. REST API가 아니지만, REST API라고 부른다. (보편적인 상태)

---

# 시적 허용 금지

## Self-Descriptive를 지키기 위한 방법 1

진정한 REST를 만들고 싶다면 미디어 타입을 정의하고, 그 문서를 작성한 후 특정 필드가 어떤 의미를 지니는지 정의하면된다.

그 후 IANA에 미디어 타입을 등록하면된다.

`번거롭다.`

## Self-Descriptive를 지키기 위한 방법 2

아래와 같이 헤더에 Link를 추가한다.

```text
GET /todos HTTP/1.1
Host: example.org

HTTP/1.1 200 OK
Content-Type: application/json
Link: <https://example.org/docs/todos>; rel="profile"

[
    {"id": 1, "title": "Meow"},
    {"id": 2, "title": "Boom"}
]
```

각 필드가 어떤 내용을 의미하는지 작성한 Profile을 Link 헤더에 삽입하여 응답하면 Self-Descriptie하다.

`Content Negotiation을 할 수 없다.`

## HATEOAS를 지키기 위한 방법 1

```text
GET /todos HTTP/1.1
Content-Type: application/json

HTTP/1.1 200 OK
Link: <https://example.org/docs/todos>; rel="profile"

[
    {"link": "https://example.org/todos/1", "id": 1, "title": "Meow"},
    {"link": "https://example.org/todos/2", "id": 2, "title": "Boom"}
]
```

## HATEOAS를 지키기 위한 방법 2

```text
GET /todos HTTP/1.1
Content-Type: application/json

[
    {"id": 1, "title": "Meow"},
    {"id": 2, "title": "Boom"}
]

HTTP/1.1 204 NO Content
Location: /todos/1
Link: </todos>; rel="collcetion"
```

---

# 그런 REST API로 괜찮은가 - 결론

일단 REST 아키텍처 스타일인 Uniform Interface 중

- Identification of Resources(리소스가 URI로 식별되면 된다.)
- Manipulation of Resources Through Representation(HTTP 메서드로 행위를 명시하면 된다.)

위 두가지 제약조건이라도 잘 지키자.

- Self-Descriptive Message
- Hypermedia as The Engine of Application State(HAETOAS)

이건 너무 어렵긴하다. HTTP API라고 부르는게 맞긴하지만 이미 Well Known으로 퍼져있는 개념이 되어버려서 REST API라고 불러도 상관없다.

---

# URI 기깔나게 설계하기

