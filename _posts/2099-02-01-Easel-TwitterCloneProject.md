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
- **Spring Cloud Gateway 동작원리**
  - Spring Cloud Gateway Filter vs Spring Security Filter vs Spring Reactive Security Filter
- **ElasticSearch**
    - ES의 구조
    - 다른 DBMS에 비해 검색을 빠르게 지원하는 원리
- **Redis**
    - Redis의 캐싱 전략
    - Caffeine Cache와 비교
- **Cypher 문법**
- **Passport**
  - Passport란 무엇이고 왜 쓰는 것인가?
- **우리 프로젝트의 CI/CD 배포 플로우**
- **비동기 쓰레드 및 큐가 꽉차면 어떻게되는지?**
- **비관적 락 vs 낙관적 락**
  - 낙관적 락을 애플리케이션 단위에서 구현해보기
- **Mock vs MockBean**
  - 모킹 의존성에 관한 두 애노테이션의 차이점

---

# 학습 및 적용

- [gRPC](https://k-diger.github.io/posts/gRPC/)
- [Spring Cloud Gateway](https://k-diger.github.io/posts/spring-cloud-gateway/)
- [Passport](https://k-diger.github.io/posts/Passport/)
- [ElasticSearch](https://k-diger.github.io/posts/ElasticSearch/)
- [kafka](https://k-diger.github.io/posts/kafka/)
- [Redis](https://k-diger.github.io/posts/redis/)
- [Neo4j (GraphDB)](https://k-diger.github.io/posts/neo4j/)

---

# 트러블 슈팅

