---

title: WebSocket In Spring (+STOMP)

date: 2022-10-16
categories: [Java, Spring, WebSocket, STOMP]
tags: [Java, Spring, WebSocket, STOMP]
layout: post
toc: true
math: true
mermaid: true

---

# 참고자료

[참고 자료 - Spring WebSocket 공식문서](https://docs.spring.io/spring-framework/docs/5.2.6.RELEASE/spring-framework-reference/web.html#websocket-stomp-appplication-context-events)

---

# 웹 소켓 개요

단일 TCP 연결을 통해 **전이중 양방향** 통신을 가능하도록 채널을 설정하는 표준화

HTTP 와 다른 프로토콜이지만, 80, 443 포트를 사용하여 기존 방화벽 설정을 재활용할 수 있는 특징을 가진다.

WebSocket 을 이용하려면 양 측의 Handshake 가 필요하다. 이는 HTTP 기반으로 요청이 이루어진다.

![img.png](https://github.com/K-Diger/K-Diger.github.io/blob/master/blog/image/websocket1.png?raw=true)

위 그림을 보듯이 HTTP 요청 헤더에 Upgrade 라는 필드에 websocket 이라는 값을 넣어주는 것이 웹소켓 handshake 의 시작이다.

![img.png](https://github.com/K-Diger/K-Diger.github.io/blob/master/blog/image/websocket2.png?raw=true)

그리고 핸드셰이크가 성공적으로 끝나면 다음과 같은 101 응답코드가 내려온다.

성공적인 핸드셰이크 후 HTTP 업그레이드 요청의 기반이 되는 TCP 소켓은

클라이언트와 서버 모두 계속해서 메시지를 보내고 받을 수 있도록 열려 있게된다.

WebSocket 서버가 웹 서버(예: nginx) 뒤에서 실행 중인 경우

WebSocket 업그레이드 요청을 WebSocket 서버로 전달하도록 구성해야한다.

---

# 웹 소켓 vs HTTP

WebSocket이 HTTP와 호환되도록 설계되고 HTTP 요청으로 시작하더라도 두 프로토콜은 매우 다른 특징과 역할을 가지고 있다.

### HTTP

HTTP 및 REST에서 애플리케이션은 **여러개의 URL 에 대한 특정한 응답**을 수행하는 흐름이다.

따라서 서버는 HTTP URL, 메서드 및 헤더를 기반으로 요청을 적절한 처리기로 라우팅한다.

### WebSocket

WebSocket에는 일반적으로 초기 연결(Handshake)을 위한 **URL이 하나**만 있다.

결과적으로 모든 애플리케이션 메시지는 동일한 TCP Connection 에서 흐른다.

---

# 웹 소켓은 어떨 때 사용하는 것이 좋을까?

WebSocket은 웹 페이지를 동적이고 대화식으로 만들 수 있다. (실시간)

그러나 Ajax와 HTTP 스트리밍 또는 long polling 의 조합이 간단하고 효과적인 솔루션을 제공할 수도 있다.

즉, **"동적"**이라는 요구사항이 있는 기능에 무조건 적으로 WebSocket 이 좋은 솔루션은 아니라는 것이다.

예를 들어 뉴스, 메일 및 소셜 피드는 동적으로 업데이트되어야 하지만, 몇 분마다 업데이트하는 것이 완벽할 수 있다.

반면에 협업, 게임 및 금융 앱은 실시간에 훨씬 더 가까워야 한다.

이럴 때에는 WebSocket 의 역할이 가장 어울리다고 볼 수 있다.

요청-응답의 대기 시간만으로는 결정할 것이 아니다.

메시지 양이 비교적 적은 경우(예: 네트워크 오류 모니터링)HTTP 스트리밍 또는 폴링은 효과적인 솔루션을 제공할 수 있다.

> **WebSocket을 사용하는 가장 좋은 사례는, 낮은 대기 시간, 높은 빈도 및 높은 볼륨의 조합이다..**

---

# STOMP (Simple Text Oriented Messaging Protocol) 개요

WebSocket 위에서 동작하는 문자 기반 메세징 프로토콜이다.

Client - Server 메세지의 유형, 형식, 내용을 정의하는 매커니즘이다.

## STOMP 의 프레임 구조는 다음과 같다.

![img.png](https://github.com/K-Diger/K-Diger.github.io/blob/master/blog/image/stomp1.png?raw=true)


### STOMP 프레임 구조 실제 예시

    SEND

    destination:/chat/message
    content-type:application/json

    {
        "type": "MESSAGE",
        "chattingRoomId": 4,
        "message": "테스트 1",
        "multipartFileList": "",
        "senderId": 1,
        "receiverId": 2
    }^@

클라이언트는 **COMMAND**에 **SEND** 또는 **SUBSCRIBE** 를 사용하여 메시지 내용과 수신 대상을 설명하는 대상 헤더와 함께 메시지를 보내거나 구독할 수 있다.

이를 통해 **브로커를 통해 다른 연결된 클라이언트로 메시지를 보내거**나 **서버에 메시지를 보내 일부 작업을 수행하도록 요청할 수** 있는 간단한 Pub/Sub 메커니즘을 사용할 수 있게된다.

---

# Spring 에서의 STOMP

Spring 에서 STOMP 사용할 때 Spring WebSocket 애플리케이션은 클라이언트에 대한 STOMP **브로커 역할**을 한다.

정확히는 @Controller에서 **메시지 처리 방법** 또는 **구독 추적**, **구독된 사용자에게 메시지를 브로드캐스트**하는 간단한 인메모리 브로커역할을 한다.

또한 **실제 메시지 브로드캐스트를 위해 전용 STOMP 브로커**(예: RabbitMQ, ActiveMQ 및 기타)와 함께 작동하도록 Spring을 구성할 수 있다.

이 경우 **Spring은 브로커에 대한 TCP 연결을 유지**하고 **브로커에게 메시지를 전달하며 연결된 WebSocket 클라이언트로 메시지를 전달**한다.

## STOMP - 구독하기

Spring 에서는 SimpMessagingTemplate 라는 STOMP 전용 메세징 타입이 있다. 이를 통해 구독 요청을 받아낸다.

![img.png](https://github.com/K-Diger/K-Diger.github.io/blob/master/blog/image/stomp2.png?raw=true)

## STOMP - 메세지 보내기

@MessageMapping 어노테이션이 아래와 같은 SEND 요청을 처리한다.

![img.png](https://github.com/K-Diger/K-Diger.github.io/blob/master/blog/image/stomp3.png?raw=true)

이때 메세징 경로 /topic/.. 은 1:N을 의미하고

메세징 경로 /queue/.. 은 1:1 메시지를 의미한다. (표준 사양)


## STOMP - 브로드캐스팅 하기

![img.png](https://github.com/K-Diger/K-Diger.github.io/blob/master/blog/image/stomp4.png?raw=true)

STOMP 서버는 MESSAGE Command 를 수신하면, 그 내용을 브로드캐스팅 한다.

서버는 **요청하지 않은 메시지를 보낼 수 없다.**

서버의 모든 메시지는 특정 클라이언트 구독에 대한 응답이어야 하며

서버 메시지의 subscription-id 헤더는 클라이언트 구독의 id 헤더와 일치해야한다.

## STOMP 환경설정

    import org.springframework.web.socket.config.annotation.EnableWebSocketMessageBroker;
    import org.springframework.web.socket.config.annotation.StompEndpointRegistry;

    @Configuration
    @EnableWebSocketMessageBroker
    public class WebSocketConfig implements WebSocketMessageBrokerConfigurer {

        @Override
        public void registerStompEndpoints(StompEndpointRegistry registry) {
            registry.addEndpoint("/portfolio").withSockJS();
        }

        @Override
        public void configureMessageBroker(MessageBrokerRegistry config) {
            config.setApplicationDestinationPrefixes("/app");
            config.enableSimpleBroker("/topic", "/queue");
        }
    }

### registry.addEndpoint("/portfolio").withSockJS();

핸드셰이크 경로이다. 해당 경로로 핸드셰이크 이후에 Pub/Sub을 수행할 수 있다.

### config.setApplicationDestinationPrefixes("/app");

STOMP messages whose destination header begins with /app are routed to

@MessageMapping methods in @Controller classes.

> 목적지 헤더가 /app 으로 시작하는 경로는 MessageMapping 메서드가 있는 컨트롤러로 라우팅 되어 처리된다.

### config.enableSimpleBroker("/topic", "/queue");

Use the built-in message broker for subscriptions and broadcasting and

route messages whose destination header begins with /topic `or `/queue to the broker.

> 내장된 메세지 브로커를 사용하고, 목적지 주소가 /topic 이거나 /queue 로 시작하는 브로커로 메세지를 전달한다.

---

# STOMP 사용 시 장점

- 메시징 프로토콜 및 메시지 형식을 규정할 필요가 없다.

- Spring Framework의 Java 클라이언트를 포함한 STOMP 클라이언트를 사용할 수 있다.

- 메시지 브로커(예: RabbitMQ, ActiveMQ 및 기타)를 사용하여 구독 및 브로드캐스트 메시지를 관리할 수 있다.

- 애플리케이션 로직은 여러 @Controller 인스턴스로 구성할 수 있으며 STOMP 대상 헤더를 기반으로 메시지를 라우팅할 수 있을 뿐만 아니라

- 주어진 연결에 대해 단일 WebSocketHandler를 사용하여 원시 WebSocket 메시지를 처리할 수 있다.

---

# STOMP 흐름

STOMP 엔드포인트를 등록하면 Spring 애플리케이션은 연결된 클라이언트를 위한 STOMP 브로커가 된다.

![img.png](https://github.com/K-Diger/K-Diger.github.io/blob/master/blog/image/stomp5.png?raw=true)

위와 같은 브로커 및 메세징 코드를 작성했다고 했을 때의 흐름을 살펴보겠다.

1. 클라이언트는 http://localhost:8080/portfolio에 연결하고 WebSocket 연결이 설정되면 STOMP 프레임이 전달된다.

2. 클라이언트는 대상 헤더가 /topic/greeting인 SUBSCRIBE 프레임을 보낸다.
수신 및 디코딩된 메시지는 clientInboundChannel로 전송된 다음 클라이언트 구독을 저장하는 메시지 브로커로 라우팅된다.

3. 클라이언트는 /app/greeting에 aSEND 프레임을 보낸다.
/app 접두사는 MessageMapping 어노테이션이 달린 컨트롤러로 라우팅하는 데 활용된다.
라우팅 후 /app 접두사가 제거되고 대상의 나머지 /greeting 부분은 GreetingController의 @MessageMapping 메서드에 매핑된다.

4. GreetingController에서 반환된 값은 반환 값을 기반으로 하는 페이로드와
/topic/greeting의 기본 대상 헤더(입력 대상에서 파생된 /app이 /topic으로 대체됨)를 포함하는 Spring 메시지로 변환된다.
결과 메시지는 brokerChannel로 전송되고 메시지 브로커가 처리한다.


5. 메시지 브로커는 일치하는 모든 구독자를 찾고 clientOutboundChannel을 통해 각 구독자에게 MESSAGE 프레임을 보낸다.
여기서 메시지는 STOMP 프레임으로 인코딩되어 WebSocket 연결을 통해 전송된다.

---

# STOMP 구성 요소

### @MessageMapping

기본적으로 @MessageMapping 어노테이션이 달린 메서드가 MessageConverter를 활용해 페이로드로 직렬화 되고 직렬화된 내용이 BrokerChannel에 전송된다.

또한 이 메서드에서 구독자에게 브로드캐스트되며, Outbound 메시지의 대상은 Inbound 메시지의 대상과 동일하지만 접두사가 /topic 이다.


### @SendTo, @SendToUser

@SendTo 및 @SendToUser 주석을 사용하여 출력 메시지의 대상을 사용자 지정할 수 있다.

@SendTo는 대상 대상을 사용자 지정하거나 여러 대상을 지정하는 데 사용되고

@SendToUser는 출력 메시지를 입력 메시지와 연결된 사용자에게만 보내는 데 사용된다.

@SendTo와 @SendToUser를 같은 메서드에서 동시에 사용할 수 있으며 둘 다 클래스 수준에서 지원되며 이 경우 클래스의 메서드에 대한 기본값으로 작동한다.

그러나 메서드 수준의 @SendTo 또는 @SendToUser 주석은 클래스 수준에서 이미 선언된 어노테이션이 있다면 클래스에 지정한 어노테이션으로 오버라이딩 되는 것을 주의해야한다.

<br>

### Messaging 특징 (비동기, @SendTo - @SendToUser 의 대체 가능성)

메시지는 비동기식으로 처리될 수 있으며 @MessageMapping 메서드는 ListenableFuture, CompletableFuture 또는 CompletionStage를 반환한다.

@SendTo 및 @SendToUser는 SimpMessagingTemplate을 사용하여 메시지를 보내는 것을 제거하는 편의성 제공에 불과하다.

조금 더 복잡한 처리가 필요한 경우 @MessageMapping 메서드가 SimpMessagingTemplate을 직접 사용하는 것이 좋다.

---

# @SubscribeMapping

@SubscribeMapping은 @MessageMapping과 유사하지만 매핑을 구독 메시지로만 좁힌다.

@MessageMapping과 동일한 메소드 파라미터를 받을 수 있지만

반환 값의 경우 기본적으로 메시지는 클라이언트에 직접 전송되고(구독에 대한 응답으로 clientOutboundChannel을 통해)

브로커(brokerChannel을 통해 일치하는 구독에 대한 브로드캐스트로)가 아니다.

@SendTo 또는 @SendToUser를 추가하면 이 동작이 무시되고 대신 브로커에 전송된다.

## 언제 사용하는가?

브로커는 /topic 및 /queue에 매핑되고, 애플리케이션 컨트롤러는 /app에 매핑된다고 가정했을 때

이 설정에서 브로커는 반복되는 브로드캐스트를 위한 /topic 및 /queue에 대한 모든 구독을 저장하며 애플리케이션이 관여할 필요가 없다.

클라이언트는 또한 일부 /app 대상을 구독할 수 있으며 컨트롤러는 구독을 다시 저장하거나 사용하지 않고(실제로 일회성 요청-응답 교환) 브로커를 포함하지 않고 해당 구독에 대한 응답으로 값을 반환할 수 있다.

> 즉, 요약하자면 @Controller Messaging 과 Subscribe를 구분하는 구문을 작성하지 않아도 되고, 구독에 대한 정보를 관리할 구문도 필요없게 된다.
>
> 하지만 구독요청과 메세징을 독립적으로 처리하지 않을 때는 반드시 동일한 접두사에 넣으면 안된다.
>
> --> /app 의 접두사를 가진 구독요청과 메세징 처리를 같이 작성하면 안되는 것이다.
