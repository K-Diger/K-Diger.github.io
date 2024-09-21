---

title: Opentelemetry
date: 2024-08-30
categories: [Opentelementry]
tags: [Opentelementry]
layout: post
toc: true
math: true
mermaid: true

---

[참고 자료 - NHN FORWARD 22](https://www.youtube.com/watch?v=EZmUxMtx5Fc)

---


## NHN의 오픈텔레메트리 도입 이유

NHN은 사내 클라우드와 외부 클라우드(AWS)를 혼용하여 사용하고 있었다.

사내 클라우드 플랫폼 전체에 장애가 생기면 외부 클라우드에서 서비스를 진행하기 위함이다.

하지만 레거시한 인프라에서는 AWS에서 동작하는 서비스에 대한 모니터링을 수행할 수 없어 이를 해결하기 위한 오픈텔레멘트리를 도입했다.

---

## Observability (관측성)

시스템, 애플리케이션이 값을 출력하는 것을 바탕으로 시스템을 이해할 수 있는 속성이다.

그럼 모니터링이랑 무슨 관계가 있을까?

관측성은 모니터링을 통해서 이루어지는 속성이다.

그럼 구체적으로 어떤 값을 모니터링해야하는지 알아본다.

---

## Telemetry Data

### Log (로그)

시간 기반 텍스트로 우리가 흔히 하는 로그를 의미한다. 이 로그에서 의미있는 데이터를 찾아내야한다. 그러기 위해서 로그 수집 시스템에서 이를 관리해야한다.

이를 조금이나마 완화하기 위해 구조화된 로그(JSON, Key-Value)로 남기도록 한다.

### Metrics (지표)

런타임 환경에서 측정된 값이다. 흔히 성능 지표가 쓰이는데 성능 지표를 바탕으로 오토 스케일링을 적용할 수 있는 여러 정책을 적용할 정보이다.

### Traces ((분산)추적)

어떤 요청이 처리될 때의 경로를 말한다. 시스템이 요청을 처리하는 흐름을 분석할 수 있는 정보이다.

특히 분산 시스템에서 어떤 시스템에서 부하가 발생하는지 등을 관찰하기 좋은 정보이다.

아래와 같이 Span이라는 작업단위를 표현하는 Tree구조로 이를 확인할 수 있다.

![](https://github.com/K-Diger/K-Diger.github.io/blob/main/images/opentelemetry/trace.png?raw=true)

Trace는 특히 비동기 처리하는 것에 엄청나게 의미가 있다. 아래처럼 비동기에 대한 흐름을 한눈에 확인할수도 있다.

![](https://github.com/K-Diger/K-Diger.github.io/blob/main/images/opentelemetry/trace-async.png?raw=true)

---

## Opentelemetry 구조

![](https://github.com/K-Diger/K-Diger.github.io/blob/main/images/opentelemetry/opentelemetry-architecture.png?raw=true)

MicroService는 Otel이 제공하는 SDK, API를 통해 Otel이 정해놓은 형식에 맞춰 각종 지표를 보낸다.
