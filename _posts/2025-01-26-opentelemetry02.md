---

title: 관찰 가능성 엔지니어링 - Chapter2 (OpenTelemetry 시그널: 분산 추적, 메트릭, 로그)
date: 2025-01-26
categories: [Opentelementry]
tags: [Opentelementry]
layout: post
toc: true
math: true
mermaid: true

---

# 실습 환경

아래 이미지와 같은 구조를 마련하기 위해 도커 컴포즈를 사용한다.

![](https://github.com/K-Diger/K-Diger.github.io/blob/main/images/otel/Chapter02_1.png?raw=true)

| 서비스                     | 포트    |
|-------------------------|-------|
| Jagger (Trace)          | 16686 |
| Prometheus (Metric)     | 9090  |
| Loki (Log)              | 3100  |
| Grafana (Visualization) | 3000  |
| Opentelemetry Collector | 13133 |
| Web Applicaiton         | 8080  |

실습 환경을 위한 docker compose는 아래와 같다.

```yml
version: "3.7"
services:
    shopper:
        image: codeboten/shopper:chapter2
        container_name: shopper
        environment:
            - OTEL_EXPORTER_OTLP_ENDPOINT=opentelemetry-collector:4317
            - OTEL_EXPORTER_OTLP_INSECURE=true
            - GROCERY_STORE_URL=http://grocery-store:8080/products
        networks:
            - cloud-native-observability
        depends_on:
            - grocery-store
            - opentelemetry-collector
        stop_grace_period: 1s
    grocery-store:
        image: codeboten/grocery-store:chapter2
        container_name: grocery-store
        environment:
            - OTEL_EXPORTER_OTLP_ENDPOINT=http://opentelemetry-collector:4317
            - OTEL_SERVICE_NAME=grocery-store
            - INVENTORY_URL=http://legacy-inventory:5001/inventory
        networks:
            - cloud-native-observability
        depends_on:
            - legacy-inventory
            - opentelemetry-collector
        stop_grace_period: 1s
        ports:
            - 8080:5000
        deploy:
            resources:
                limits:
                    cpus: "0.50"
                    memory: 80M
    legacy-inventory:
        image: codeboten/legacy-inventory:chapter2
        container_name: inventory
        environment:
            - OTEL_EXPORTER_OTLP_ENDPOINT=http://opentelemetry-collector:4317
            - OTEL_SERVICE_NAME=inventory
        networks:
            - cloud-native-observability
        depends_on:
            - opentelemetry-collector
        stop_grace_period: 1s
        ports:
            - 5001:5001
        deploy:
            resources:
                limits:
                    cpus: "0.50"
                    memory: 80M
    jaeger:
        image: jaegertracing/all-in-one:1.29.0
        container_name: jaeger
        ports:
            - 6831:6831/udp
            - 16686:16686
        networks:
            - cloud-native-observability
    prometheus:
        image: prom/prometheus:v2.29.2
        container_name: prometheus
        volumes:
            - ./config/prometheus/config.yml/:/etc/prometheus/prometheus.yml
        command:
            - "--config.file=/etc/prometheus/prometheus.yml"
            - "--enable-feature=exemplar-storage"
        ports:
            - 9090:9090
        networks:
            - cloud-native-observability
    opentelemetry-collector:
        image: otel/opentelemetry-collector-contrib:0.43.0
        container_name: opentelemetry-collector
        volumes:
            - ./config/collector/config.yml/:/etc/opentelemetry-collector.yml
            - /var/run/docker.sock:/var/run/docker.sock
        command:
            - "--config=/etc/opentelemetry-collector.yml"
        networks:
            - cloud-native-observability
        ports:
            - 4317:4317
            - 13133:13133
            - 8889:8889
        stop_grace_period: 1s
    loki:
        image: grafana/loki:2.3.0
        container_name: loki
        ports:
            - 3100:3100
        command: -config.file=/etc/loki/local-config.yaml
        networks:
            - cloud-native-observability
    promtail:
        image: grafana/promtail:2.3.0
        container_name: promtail
        volumes:
            - /var/log:/var/log
        command: -config.file=/etc/promtail/config.yml
        networks:
            - cloud-native-observability
    grafana:
        image: grafana/grafana:8.3.3
        container_name: grafana
        ports:
            - 3000:3000
        volumes:
            - ./config/grafana/provisioning:/etc/grafana/provisioning
        networks:
            - cloud-native-observability
        environment:
            - GF_AUTH_ANONYMOUS_ENABLED=true
            - GF_AUTH_ORG_ROLE=Editor
            - GF_AUTH_ANONYMOUS_ORG_ROLE=Admin
            - GF_AUTH_DISABLE_LOGIN_FORM=true

networks:
    cloud-native-observability:
```

---

# 분산 추적

추적은 전체 시스템에 대한 고유한 요청을 나타내며 `동기`, `비동기`로 구분된다.

또한 각 추적에 기록된 작업은 시스템에서 처리된 작업의 단위를 의미하는 `스팬`으로 표기된다.

## Span

단일 메서드 호출이나 메서드 내에서 호출되는 코드의 일부분을 나타낸다.

추적 내에서의 여러 스팬은 부모-자식 관계로 연결되어있고 각 자식 스팬은 부모 스팬에 대한 정보를 갖고 있다.

추적 내에 있는 첫 번째 스팬을 `루트 스팬`이라고 하며 당연하게도 부모 스팬에 대한 식별자를 가지고 있지 않다.

아래는 추적, 스팬의 관계를 도식화한 그림이다.

![](https://github.com/K-Diger/K-Diger.github.io/blob/main/images/otel/Chapter02_2.png?raw=true)

그리고 Jagger에서 실제 추적 정보를 조회하면 아래와 같이 볼 수 있다.

![](https://github.com/K-Diger/K-Diger.github.io/blob/main/images/otel/Chapter02_3.png?raw=true)

---

## Span Context

W3C의 권고에 따라 정의된 Span의 구성요소는 아래와 같다.

### 추적 ID

고유 식별자로, 전체 시스템에서 요청을 식별할 수 있는 요소

### 스팬 ID

컨텍스트와 상호 작용한 최종 스팬과 연결된다.

하위 스팬이 존재하는 경우 상위 스팬의 스팬 ID는 부모 식별자로 불린다.

### 추적 플래그

추적 수준(Trace Level), 샘플링 여부(Sampling Decision)등 추적에 관한 메타데이터를 담고있다.

### 추적 상태

개별 벤더가 각자의 시스템에서 필요로 하는 정보를 전파, 추적 데이터를 해석할 수 있도록 한다. ex) `vendorA=123456`와 같은 형태로 추적 상태 필드에 값을 넣는다.

---

## 실제 Span의 조회

![](https://github.com/K-Diger/K-Diger.github.io/blob/main/images/otel/Chapter02_4.png?raw=true)

---

# 메트릭

메트릭이 갖는 필드는 아래와 같다.

메트릭은 단순한 리소스 사용량을 측정할 수 있는 것 뿐만 아니라 아래처럼 Request Counter등 다양한 요소를 얻을 수 있다.

이런 다양한 메트릭 지표를 통해 On-Call 알림을 보내도록 설정하여 엔지니어에게 현재 서비스의 대한 경고를 보낸다.

![](https://github.com/K-Diger/K-Diger.github.io/blob/main/images/otel/Chapter02_5.png?raw=true)

`메트릭` 이라는 용어 자체는 서로 다른 측정값들을 캡슐화한 용어이다. 구체적인 데이터는 `데이터 포인트`라는 데이터 타입을 포착한다.

## 데이터 포인트 - 카운터(Counter)

단순히 증가만 하는 누적 값이다.

```
# HTTP 요청 카운터
http_requests_total{method="GET", endpoint="/api/users"} 23421
http_requests_total{method="POST", endpoint="/api/users"} 7832

# 시스템 에러 카운터
system_errors_total{type="connection_timeout"} 382
system_errors_total{type="internal_error"} 89
```

## 데이터 포인트 - 게이지(Gauge)

현재 상태를 나타내는 순간값이다.

```
# CPU 사용률
cpu_usage_percent{core="0"} 78.5
cpu_usage_percent{core="1"} 92.3

# 메모리 사용량
memory_usage_bytes{pod="backend-prod-1"} 1.28e+9
```

## 데이터 포인트 - 히스토그램(Histogram)

값의 분포를 구간(bucket)별로 측정하는 관측값이다. 구간은 보통 le(less equal) 레이블로 표시한다.

```
# HTTP 응답시간 분포
http_request_duration_seconds_bucket{le="0.1"} 12323  # 100ms 이하
http_request_duration_seconds_bucket{le="0.3"} 14236  # 300ms 이하
http_request_duration_seconds_bucket{le="1.0"} 15564  # 1s 이하
http_request_duration_seconds_bucket{le="+Inf"} 15600 # 전체
```

## 데이터 포인트 - 요약(Summary)

히스토그램과 유사하나 서버측에서 백분위를 계산하여 클라이언트 부하는 적으나 정확도는 떨어진다.

```
# API 응답시간 요약
api_response_latency_seconds{quantile="0.5"} 0.042  # 중앙값
api_response_latency_seconds{quantile="0.9"} 0.087  # 90th 백분위
api_response_latency_seconds{quantile="0.99"} 0.184 # 99th 백분위
```

---

# OpenTelemetry 데이터 포인트 구조

OpenTelemetry가 활성 스팬 정보를 메트릭에 포함시키도록하여 더 많은 정보를 얻게 해준다.

OpenTelemetry에서 정의된 데이터 포인트는 아래와 같은 정보들을 갖는다.

## 1. 추적 ID (Trace ID)
- 전체 트랜잭션의 고유 식별자
- 형식: 16바이트 hex 문자열
```yaml
trace_id: "4bf92f3577b34da6a3ce929d0e0e4736"
```

## 2. 스팬 ID (Span ID)
- 개별 작업 단위의 식별자
- 형식: 8바이트 hex 문자열
```yaml
span_id: "00f067aa0ba902b7"
```

## 3. 타임스탬프 (Timestamp)
- 이벤트 발생 시각 (나노초 단위)
```yaml
timestamp: "2024-01-26T09:00:00.123456789Z"
```

## 4. 표준 속성 (Standard Attributes)

### 서비스 식별
```yaml
service:
  name: "payment-service"
  version: "1.2.3"
  environment: "production"
  instance_id: "pod-xyz-123"
```

### HTTP 요청 정보
```yaml
http:
  method: "POST"
  url: "/api/v1/users"
  status_code: 200
  user_agent: "Mozilla/5.0..."
```

### 데이터베이스 작업
```yaml
database:
  system: "postgresql"
  name: "users"
  operation: "SELECT"
  statement: "SELECT * FROM users WHERE id = ?"
```

### 클라우드/인프라 정보
```yaml
cloud:
  provider: "aws"
  region: "us-east-1"
  zone: "us-east-1a"
kubernetes:
  pod_name: "backend-pod-123"
  namespace: "production"
```

### 사용자 정의 속성
```yaml
custom:
  customer_id: "12345"
  transaction_id: "tx_789"
  error_type: "connection_timeout"
```

---

# 로그

로그는 공통 로그 형식이 있지만 이를 모두가 완벽하게 지킬 순 없을것이다. 그럼에도 다음 두 항목은 반드시 구성되어야하는 요소이다.

- 이벤트가 발생한 타임스탬프
- 이벤트를 나타내는 메세지

구조화된 로깅에서는 Key-Value로 표현하기도하고, 구분자와 정의된 순서를 이용한 로그를 만들어내기도한다.

