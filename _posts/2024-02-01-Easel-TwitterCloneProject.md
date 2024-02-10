---

title: Easel(트위터 클론 프로젝트)의 기술 회고
date: 2024-02-01
categories: [Easel]
tags: [Easel, gRPC, SpringCloudGateway, Lock, Mock, MockBean, Cypher, CI/CD, Async, Kafka, Redis, Passport]
layout: post
toc: true
math: true
mermaid: true

---

# 새롭게 접한 기술 스택

- **gRPC 동작원리**
  - gRPC는 왜 빠르게 동작하는가?
  - gRPC Gateway가 필요한 상황
- **Spring Cloud Gateway 동작원리**
  - Spring Cloud Gateway Filter vs Spring Security Filter vs Spring Reactive Security Filter
- **비관적 락 vs 낙관적 락**
  - 낙관적 락을 애플리케이션 단위에서 구현해보기
- **Mock vs MockBean**
  - 모킹 의존성에 관한 두 애노테이션의 차이점
- **Cypher 문법**
- **우리 프로젝트의 CI/CD 배포 플로우**
- **비동기 쓰레드 및 큐가 꽉차면 어떻게되는지?**
- **Kafka**
  - 카프카의 구성요소 및 구조
  - 컨슈머의 내부 동작 원리
  - 프로듀서의 내부 동작 원리
  - 이벤트 전송 간 오류가 발생했을 때 어떻게 처리할 것인지?
    - 프로듀싱 실패
    - 컨슈밍 실패
    - 컨슘 후 컨슈머에서 로직 처리 실패
  - 카프카의 전송 보장 옵션
  - DLT
- **Redis**
  - Redis의 캐싱 전략
  - Caffeine Cache와 비교
- **Passport**
  - Passport란 무엇이고 왜 쓰는 것인가?

- ElasticSearch
  - ES의 구조
  - 다른 DBMS에 비해 검색을 빠르게 지원하는 원리

---

# 트러블 슈팅

---

# gRPC

[gRPC](https://k-diger.github.io/posts/gRPC/)

---

# Kafka

## Kafka Topic

Kafka는 서버 간 통신을 위해 사용하는 메세지를 Topic이라는 곳을 통해서 통신한다.

이 Topic은 여러개의 파티션으로 구성될 수 있다. 그리고 메세지는 이 파티션에 추가되는 방식이다.

파티션별로 메세지의 순서(오프셋)이 존재하여 순서를 보장하도록 구성되어있다.

---

## Kafka Producer

프로듀서는 토픽에 대한 메세지를 보내는 애플리케이션을 가리킨다.

---

## Kafka Consumer

컨슈머는 토픽에 대한 메세지를 소비하는 애플리케이션을 가리킨다.

—

## Kafka Broker

하나의 Kafka 서버를 브로커라고 한다. Spring Boot 환경에서 Kafka를 사용하고자 할 때 아래와 같은 설정정보를 등록하곤 한다.

```yml
spring:
	kafka:
		bootstrap-servers: localhost:9092
```

위 bootstrap-servers 구문으로 Kafka의 Broker 주소를 명시하는 것이다.

Broker는 Producer에게 메세지를 수신하고, 오프셋을 지정한 후 해당 메세지를 디스크에 저장한다.

또한 Consumer의 파티션 읽기 요청에 응답하여 디스크에 저장된 메세지를 전송한다.

이 과정에서 브로커 서버 내 사용하지 않는 메모리를 페이지 캐싱으로 캐싱하여 조금 더 빠르게 I/O작업을 할 수 있도록 구성되어있다.

Producer -> Topic -> Consumer

이렇게 Producer나 Consumer가 바로 토픽에 접근하는 흐름이 아니라

Producer -> Broker -> Topic(Broker's Disk)

와 같은 흐름으로 동작하는 것이였다.

각 파티션은 하나의 브로커가 소유하고 그 브로커를 파티션 리더라고 칭한다.

같은 파티션이 여러 브로커에게 지정될 수 있는데 이런 상황에서는 파티션 리더가 복제된다.

복제된 파티션에는 메세지가 중복으로 저장되지만 장애 발생 시 장애 발생 한 지점의 브로커가 파티션의 소유권을 인계받아 그 파티션을 처리할 수 있는 가용성 측면의 장점이 있다.

---

## Kafka Controller

Kafka 생태계에서는 여러개의 Broker가 있을 때 그 중 하나가 Controller의 역할을 수행한다.

Controller는 각 Broker들에게 담당할 파티션을 할당하고 다른 Broker들이 정상적으로 동작하는지 모니터링하는 관리적인 역할이 부여되고 파티션의 리더를 선출하는 역할을 한다.

모든 Broker는 Controller Node를 생성하도록 시도하는데, 가장 먼저 생성한 Broker가 Controller Broker가 되는 것이다.

---

# Zookeeper

분산 처리 시스템을 총괄하는 코디네이터라고 알려져있다.

이러한 중앙 관리 시스템이 없다면 분산 시스템 환경은 각각의 애플리케이션이 돌아가는 환경이 다르고 네트워크에서 발생하는 문제로 관리적인 측면에서 매우 어려움을 겪을 수 있다.

