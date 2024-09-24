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

### Global Trace

Trace만으로는 MSA환경에서 제대로된 요청을 추적하기 어렵다.

MSA환경에서는 각 마이크로서비스와 통신하는 고유한 Trace Id를 갖게 되지만 여러 마이크로서비스와 통신하는 것이 결국은 어떤 기능을 위한 것인지 한눈에 알아보기가 힘들기 때문이다.

사용자 인증/인가 -> 사용자 잔액 조회 -> 결제

이 세 마이크로서비에서 수행하는 결제 로직이 있다고 했을 때, Trace Id는 X로 동일하지만, 비즈니스 관점에서 이 Trace Id가 어떤 요청에 대한 것이였는지 알아보려면 저 Trace ID를 가진 마이크로서비스를 모두 조회해야한다.

이를 완화할 수 있는 방법이 Global Trace로 이러한 하나의 유스케이스 과정을 하나의 Trace Id로 관리하는 래핑된 Trace Id인 것이다.

![](https://github.com/K-Diger/K-Diger.github.io/blob/main/images/opentelemetry/trace.png?raw=true)

Trace는 특히 비동기 처리하는 것에 엄청나게 의미가 있다. 아래처럼 비동기에 대한 흐름을 한눈에 확인할수도 있다.

![](https://github.com/K-Diger/K-Diger.github.io/blob/main/images/opentelemetry/trace-async.png?raw=true)

---

## Opentelemetry 구조

![](https://github.com/K-Diger/K-Diger.github.io/blob/main/images/opentelemetry/opentelemetry-architecture.png?raw=true)

MicroService는 Otel이 제공하는 SDK, API를 통해 Otel이 정해놓은 형식에 맞춰 각종 지표를 보낸다.

![](https://github.com/K-Diger/K-Diger.github.io/blob/main/images/opentelemetry/opentelemetry-architecture2.png?raw=true)

Receiver는 각종 데이터를 수신하거나 만들어내는 영역이다.

생성된 데이터는 Processor 영역을 거쳐 Exporters를 통해 원하는 백엔드로 전송한다.

Otel은 어떻게 배치했냐에 따라서 아래와 같이 Agent, Gateway로 나뉜다.

![](https://github.com/K-Diger/K-Diger.github.io/blob/main/images/opentelemetry/agentvsgateway?raw=true)

게이트웨이 모드의 장점은 만약, 결과를 전송해야하는 대상이 `Backend 1`이 였을 때 `2`로 바꿔야하는 상황인데

에이전트만으로 데이터를 전송한다면 직접 `Backend 2`로 데이터 전송 방향을 바꾸는 설정을 만져야한다.

하지만 게이트웨이 모드를 사용하면 게이트웨이가 바라보는 서버를 `Backend 2`로 바꾸기만 하면 몇 천대의 에이전트가 있던간에 한번에 `2`로 바꿀 수 있게된다.

아래는 Otel Gateway를 도입한 모니터링 구성도이다.

![](https://github.com/K-Diger/K-Diger.github.io/blob/main/images/opentelemetry/tobe.png?raw=true)

![](https://github.com/K-Diger/K-Diger.github.io/blob/main/images/opentelemetry/tobe2.png?raw=true)

