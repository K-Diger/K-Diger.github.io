---

title: Kafka 정확히 한 번 전송/읽기 톺아보기 및 적용
date: 2024-09-26
categories: [Kafka]
tags: [Kafka]
layout: post
toc: true
math: true
mermaid: true

---

# 참고 자료

- [Kafka Producer Internal](https://d2.naver.com/helloworld/6560422)
  - [실전 카프카 개발부터 운영까지 - 프로듀서 내부 동작 원리 ](https://github.com/mash-up-kr/S3A/blob/master/14th_kafka/dohyeon/CH5_%ED%94%84%EB%A1%9C%EB%93%80%EC%84%9C%EC%9D%98%20%EB%82%B4%EB%B6%80%20%EB%8F%99%EC%9E%91%20%EC%9B%90%EB%A6%AC%EC%99%80%20%EA%B5%AC%ED%98%84.md)
  - [토리맘의 한글라이즈 - Producer Config](https://godekdls.github.io/Apache%20Kafka/producer-configuration/)
- [Kafka Consumer Internal](https://d2.naver.com/helloworld/0974525)
  - [실전 카프카 개발부터 운영까지 - 컨슈머 내부 동작 원리 ](https://github.com/mash-up-kr/S3A/blob/master/14th_kafka/dohyeon/CH6_%EC%BB%A8%EC%8A%88%EB%A8%B8%EC%9D%98%20%EB%82%B4%EB%B6%80%20%EB%8F%99%EC%9E%91%20%EC%9B%90%EB%A6%AC%EC%99%80%20%EA%B5%AC%ED%98%84.md)
  - [토리맘의 한글라이즈 - Consumer Config](https://godekdls.github.io/Apache%20Kafka/consumer-configuration/)
- [Kafka Network Internal](https://d2.naver.com/helloworld/0853669)
- [@taesulee93](https://medium.com/@taesulee93/%EC%B9%B4%ED%94%84%EC%B9%B4-%ED%94%84%EB%A1%9C%EB%93%80%EC%84%9C-2495ff5dadef)

---

카프카는 TCP 위에서 동작하는 자체 바이너리 프로토콜을 사용한다. 모든 바이너리 프로토콜은 요청과 응답의 쌍으로 이루어져 있다.

카프카는 이 바이너리 프로토콜을 적절히 구현한 프로듀서와 컨슈머를 클라이언트로 제공하며 KafkaProducer를 사용하여 데이터를 발행하고 KafkaConsumer를 사용하여 데이터를 구독한다.

이러한 특징을 가진 카프카를 실제로 활용할 땐 신뢰성과 성능을 트레이드-오프 해야하는 상황이 발생한다. 이 때 신뢰성을 가장 중요시한 방식을 실제 Spring Boot 프로젝트에서 적용하는 방법은 아래와 같다.

---

# 프로듀서 옵션

## 처리량/지연시간 관련 옵션

### buffer.memory

프로듀서의 버퍼 메모리 옵션을 지정한다. 기본값은 32MB이다.

### batch.size

배치 전송을 위해 메세지(레코드)들을 묶는 단위를 설정한다. 기본값은 16KB이다.

### linger.ms

배치 전송을 위해 버퍼 메모리에서 대기하는 메세지들의 최대 대기시간을 설정한다. 단위는 ms이며, 기본값은 0ms이다. --> 배치 전송을 기다리지 않고 즉시 전송함

### 처리량/지연시간 관련 옵션 사용 시 주의사항

높은 처리량을 위해선 아래 주의사항을 고려해야한다.
- batch.size, linger.ms의 값을 **크게 설정**하는 등 배치 전송의 옵션을 적절히 수정해야한다.
    - 또한 buffer.memory가 커야 오랜시간동안 많은 배치 사이즈를 담아둘 수 있기 때문에 이점을 유의해아한다.

- 즉, 기본 배치 사이즈가 16KB이고 파티션이 3개이면 최소 48KB를 할당해야한다.
    - 그리고 압축 옵션으로 gzip, zstd를 선택하는 것이 도움이 된다.

지연시간이 낮은 방식으로 사용하기 위해선 아래 주의사항을 고려해야한다.
- batch.size, linger.ms의 값을 **작게 설정**하는 등 배치 전송의 옵션을 적절히 수정해야한다.
    - 그리고 압축 옵션으로 lz4, snappy를 사용하면 도움이 된다.

또한 정답은 없기 때문에 옵션값을 조정해가며 판별하는 모니터링이 중요하다.

## 전송 보장 옵션

- 최대 한 번 전송
  - ack를 기다리지 않고 다음 메시지를 전송한다. 메시지 손실을 감수하고 중복전송을 하지 않으며, 높은 처리량을 위해 사용할 수 있다.
  - acks=0으로 설정하고 retries 설정을 off 한다.
- 중복 없는 전송
  - 프로듀서가 카프카에 메시지를 전송하고 ack를 기다린다. ack를 받지 못한다면 ack를 받을 때 까지 재시도한다.
  - 재시도로 인해 메시지를 중복 전송할 가능성이 있다. 이때 메시지 중복을 방지하기 위해 멱등 옵션을 활성화 해야한다.
  - enable.idempotence=true, max.in.flight.requests.per.connection=1~5, acks=all, retries > 0으로 설정한다.
  - ack=1인 경우 리더가 다운되고 팔로워가 리더로 선출된 경우 손실 가능성이 있다.
- 정확히 한 번 전송
  - Offset storage에 저장과 Broker에 전달하는 두 가지 행위를 트랜잭션으로 묶는다.
  - 트랜잭션을 통해 여러 파티션에 메시지 전송 시 정확히 한 번을 보장한다. 프로듀서는 메시지가 잘 도착했고 잘 복제 됐다는 확인을 받으며 재시도가 필요한 경우 위에서 보았던 멱등 옵션으로 다시 보낸다.
  - 컨슈머도 메시지를 읽고 manual commit으로 손실을 방지해야 한다.

전송 보장 동작에 관해서 위 세 가지 방식으로 동작이 가능하다. 이를 달성할 수 있는 옵션은 아래와 같다.

### enable.idempotence

중복 없는 전송을 사용할 것인지에 대한 옵션이다. **기본값은 false이다.**

ture로 설정 시 부가적인 옵션도 설정해줘야한다.

이 옵션을 활성화 하는 것으로 중복 전송을 방지할 수 있는 이유는, 각 프로듀서마다 ID를 할당하고 프로듀서가 카프카에 메세지를 보낼 때 마다

오프셋과 다른 순차적으로 증가하는 시퀀스를 부여한다. 그리고 프로듀서가 메세지를 보냈을 때 시퀀스가 현재 브로커가 알고 있는 시퀀스보다 낮다면 해당 메세지를 폐기한다.

이 옵션은 다른 옵션과 맞물려 동작해야하는데 멱등성을 보장하려면 max.in.flight.requests.per.connection는 5보다 작아야 하고 retries는 0보다 커야 하며 acks는 all이어야 한다.

### max.in.flight.requests.per.connection

ACK를 받지 않은 상태의 하나의 커넥션에서 보낼 수 있는 최대 요청 수 이다. **기본값은 5**이다.

멱등성을 보장하기 위해선 5보다 작아야한다.

### acks

프로듀서 acks와 관련된 옵션으로 **기본값은 1이다.**

-1(all)로 설정해야 중복 없는 전송 옵션을 사용할 수 있다.

### replica.lag.time.max.ms

리더가 팔로워에게 복제를 위한 insync 응답 대기 시간이다. **기본값은 30000ms(30초)이다.**

이 시간이 지나도록 응답이 없으면 ISR에서 추방한다.

### request.timeout.ms

단일 작업의 타임아웃 설정이다. **기본값은 30000ms(30초)이다.**

이 시간이 지나면 재시도가 트리거 된다. 불필요한 재시도를 줄이려면 replica.lag.time.max.ms보다 큰 값이어야 한다.

30000 ~ 60000 사이 설정을 권장한다.

### delivery.timeout.ms

send 메서드 호출 후 ack를 받을 때 까지 대기 시간으로 재시도 작업 전체를 포함한 시간이다. **기본값은 120000ms(2분)이다.**

(request.timeout.ms + linger.ms ) x retries 의 합보다 같거나 커야 한다.

### retries

ack를 받지 못한 경우 재시도를 수행하는 횟수에 대한 옵션이다. **기본값은 2147483647이다.**

반드시 0보다 큰 값으로 설정해야한다.

### retry.backoff.ms

재시도 간의 지연시간을 의미한다. **기본값은 100ms이다.**

---

# 프로듀서 내부 동작 방식

카프카 프로듀서의 구성요소는 3가지이다.

- Producer Client
- RecordAccumulator
- Sender

Producer는 라우팅 계층 없이 브로커와 직접 통신하고 성능 향상을 위해 메모리에 데이터를 모아 일괄 전송한다.

---

## 프로듀서 내부 컴포넌트 - Producer Client

Java/Kotlin 코드로 Producer의 send()메서드로 레코드를 전송한다.

send()호출 시 레코드와 전송 완료 후의 실행할 콜백을 지정할 수 있다. 이 메서드를 호출하면 직렬화, 파티셔닝, 압축작업이 이루어진다.

### Producer Client - Serializer

레코드의 Key, Value는 Byte Array로 변환된다. 이 때 Serializer는 key.serializer, value.serializer로 각각 Key, Value에 대한 직렬화 방식을 지정할 수 있다.

java
configProps.put(ProducerConfig.KEY_SERIALIZER_CLASS_CONFIG, StringSerializer.class);
configProps.put(ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG, StringSerializer.class);

Serializer의 종류는 아래와 같다.

- String
- ByteArray
- ByteBuffer
- Bytes
- Double
- Integer
- Long

---

### Producer Client - 파티셔너

프로듀서가 카프카로 전송한 메세지는 토픽의 각 파티션의 로그 세그먼트에 저장된다.

따라서 프로듀서는 토픽으로 메세지를 보낼 때 해당 토픽의 어느 파티션으로 메세지를 보내야 할지를 결정해야하는데 이 때 파티셔너를 사용한다.

프로듀서가 파티션을 결정하는 알고리즘은 메세지(레코드)의 키를 해시처리하여 파티션을 구하는 방식을 사용한다. 따라서 키값이 동일하면 해당 메세지들은 동일한 파티션에 저장된다.

카프카의 파티션을 늘리는 경우, 파티션 수가 변경되는 것과 동시에 메세지의 키와 매핑된 해시 테이블도 변경된다. 따라서 프로듀서가 파티션이 증가한 이후에 동일한 키로 전송을 해도 다른 파티션에 저장된다.

파티셔너는 레코드의 담긴 Key 통해 파티션을 확인하고 명시한 파티션이 없다면 라운드 로빈으로 파티션을 선택하여 저장한다.

**partitioner.class**를 설정하여 파티셔너를 지정할 수 있고 지정하지 않으면 org.apache.kafka.clients.producer.internals.DefaultPartitioner가 사용된다.

레코드에 지정된 파티션이 없는 경우 DefaultPartitioner는 다음과 같이 동작한다.

- Key 값이 **있는** 경우 : Key 값의 Hash 값을 이용해서 파티션을 할당한다.
- Key 값이 **없는** 경우 : Round-Robin 방식으로 파티션이 할당된다.

---

### 파티셔너 - 라운드 로빈 전략

메세지(레코드)의 키값은 필수값이 아니다. 따라서 개요에서 살펴보았던 해시처리 방식은 모든 상황에 적용할 수 없게 된다. 따라서 키값은 지정하지 않을 때는 라운드 로빈알고리즘을 사용하여 파티션에 메세지가 저장된다.

라운드 로빈으로 파티션에 분배되었다 하더라도 기본적으로 설정해놓은 배치 옵션에 의해 프로듀서 내에서 대기하게 된다.

배치 크기에 도달하기 전까지 기다리지 않는 방법으로는 타이머를 두는 것이 있는데 이는 성능적으로 굉장히 비효율적이다.

---

### 파티셔너 - 스티키 파티셔닝 전략

라운드 로빈 전략은 배치 크기에 도달하는 것을 기다리며 비효율적으로 가용성을 떨어뜨리게 되는 상황이 종종 발생한다.

2.4버전부터는 스티키 파티셔닝 전략으로 그 문제점을 보완한다.

스티키 파티셔닝은 하나의 파티션에 레코드 수를 채워 배치 전송하는 전략이다. 즉, 최소 레코드 수에 도달할때까지 하나의 파티션으로만 레코드를 보내게 된다.

이 전략으로 기본 라운드 로빈보다 약 30%이상 지연시간이 감소했다는 벤치마킹 결과도 있다.

---

### Producer Client - 압축

레코드를 압축하여 네트워크 비용과 디스크 저장 비용을 줄인다. **compression.type**옵션을 통해 압축 시 사용할 코덱을 지정할 수 있고 **기본값은 none(압축하지 않음)**이다.

---

## 프로듀서 내부 컴포넌트 - RecordAccumulator

send()메서드를 호출하면 Broker로 바로 전송되는 것이 아니라 **RecordAccumulator**에 저장된다.

### RecordAccumulator - append()

RecordAccumulator는 batches라는 Map을 가지고 있는데, 이 Key는 TopicPartition이고, Value는 Deque<RecordBatch>이다.

즉, 각 파티션에 대한 Record 묶음을 들고 있는 것이다.

Record의 Serialized Size를 검사한 후 Serialized Size가 **max.request.size** 혹은 **buffer.memory** 설정값보다 크면 RecordTooLargeException이 발생한다. 검증을 마친 후에는 append()메서드를 통해 저장된다.

#### append() 호출 시

![](https://d2.naver.com/content/images/2020/08/62686a80-b576-11ea-8839-3eb52b31945d.png)

1. batches에서 추가될 레코드가 들어갈 파티션의 Deque찾는다.
2. 해당 Deque의 Last에 접근하여 레코드 배치를 확인한 후 추가될 레코드를 저장할 공간이 있는지 확인한다.
3. 여유 공간이 있으면 레코드를 RecordBatch에 추가한다.
4. 여유 공간이 없으면 새로운 레코드 배치를 생성하고 Last에 저장한다.
   5. 이 때 레코드 배치를 생성할 때 버퍼 풀에서 레코드 배치가 사용할 ByteBuffer를 받아온다. 버퍼풀에 여유가 있으면 최대 max.block.ms만큼 블락된다. 이 시간이 지나도 해결이 안되면 TimeoutException이 발생한다.
6. compression.type이 지정되어있으면 레코드가 레코드 배치로 삽입될 때 압축된다.

---

## 프로듀서 내부 컴포넌트 - Sender

RecordAccumulator에 저장된 레코드를 실질적으로 브로커에 전송하는 역할을 수행한다. 이는 비동기적으로 이루어지며 브로커에게 응답을 받은 후 레코드 전송 시 설정한 콜백에 대한 응답을 전달하기도 한다.

### Sender - Sender Thread

- RecordAccumulator에서 레코드를 꺼낸다. 이 때 drain()로 꺼내오게 되는데, 각 브로커별로 전송할 RecordBatch의 List를 얻을 수 있다.

#### Sender Thread - drain()

1. drain()에서는 각 브로커 노드에 속한 TopicPartition 목록을 얻어온다.
2. 그 후 각 노드가 속한 TopicPartition을 보면서 가장 앞에 있는 RecordBatch를 꺼낸다.
3. 꺼낸 RecordBatch를 RecordBatch List에 추가한다.
   4. 이 때 max.request.size가 넘지 않을 때 까지 모은다.

이렇게 하면 각 브로커 노드별의 RecordBatch List가 만들어진다.

#### Sender Thread - ProducerRequest

drain()과정을 통해 만들어진 RecordBatchList가 하나의 ProducerRequest로 만들어져 전송된다.

ProducerRequest는 InFlightRequest라는 각 노드의 Deque에 저장되고 저장된 순서대로 실제 브로커 노드에 전송된다.

브로커 노드로 레코드를 전송할 때는 Multiplexing 방식으로 동작해서 별도의 쓰레드를 사용하지 않고, Sender Thread에서 비동기적으로 이뤄진다.

InFlightRequests Deque의 Size는 max.in.flight.requests.per.connection 설정값에 의해서 정해진다. 이 값은 ProducerClient가 하나의 Broker로 동시에 전송할 수 있는 요청 수를 의미한다.

---

# 실제로 활용해보기 (Java, Spring Boot, Docker Compose)

Docker Compose를 통해 로컬에서 실제 카프카를 띄워보고 위에서 살펴본 정확히 한 번 전송방식을 구현해본다.

## Broker, Controller Docker Compose

```yaml
services:
  kafka_broker:
    user: root
    image: confluentinc/cp-kafka:latest
    container_name: kafka_broker
    ports:
      - "9092:9092"
      - "9101:9101"
      - "29092:29092"
    environment:
        # 각 카프카 브로커의 고유 식별자 설정
        # ${BROKER_ID}: 환경 변수, 실행 시 지정된 값으로 대체
        KAFKA_NODE_ID: ${BROKER_ID}

        # 노드 역할을 'broker'로 지정
        # KRaft 모드: 'broker', 'controller', 또는 둘 다 가능
        KAFKA_PROCESS_ROLES: 'broker'

        # 브로커 간 통신에 사용할 리스너 이름 지정
        KAFKA_INTER_BROKER_LISTENER_NAME: 'PLAINTEXT'

        # 컨트롤러 통신에 사용할 리스너 이름 지정
        KAFKA_CONTROLLER_LISTENER_NAMES: 'CONTROLLER'

        # 각 리스너에 대한 보안 프로토콜 매핑
        # 모든 리스너: 암호화되지 않은 PLAINTEXT 프로토콜 사용
        KAFKA_LISTENER_SECURITY_PROTOCOL_MAP: 'CONTROLLER:PLAINTEXT,PLAINTEXT:PLAINTEXT,PLAINTEXT_HOST:PLAINTEXT'

        # 브로커가 내부적으로 요청을 바인딩하는 주소
        # PLAINTEXT: 29092 포트, PLAINTEXT_HOST: 9092 포트 사용
        KAFKA_LISTENERS: 'PLAINTEXT://:29092,PLAINTEXT_HOST://:9092'

        # 브로커를 외부(컨슈머)에 노출 시킬 주소:포트
        # ${EXTERNAL_IP}: 환경 변수, 실제 IP 주소로 대체
        KAFKA_ADVERTISED_LISTENERS: 'PLAINTEXT://${EXTERNAL_IP}:29092,PLAINTEXT_HOST://${EXTERNAL_IP}:9092'

        # KRaft 모드에서 컨트롤러 쿼럼 구성 지정
        # 3개의 컨트롤러(KAFKA1, KAFKA2, KAFKA3)가 29093 포트 사용
        KAFKA_CONTROLLER_QUORUM_VOTERS: '3001@${KAFKA1}:29093,3002@${KAFKA2}:29093,3003@${KAFKA3}:29093'

        # 오프셋 토픽의 복제 팩터 설정
        KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR: 3

        # 초기 리밸런싱 지연 시간 설정
        KAFKA_GROUP_INITIAL_REBALANCE_DELAY_MS: 0

        # 트랜잭션 상태 로그의 최소 ISR(In-Sync Replicas) 설정
        KAFKA_TRANSACTION_STATE_LOG_MIN_ISR: 3

        # 트랜잭션 상태 로그의 복제 팩터 설정
        KAFKA_TRANSACTION_STATE_LOG_REPLICATION_FACTOR: 3

        # JMX(Java Management Extensions) 포트 설정
        KAFKA_JMX_PORT: 9101

        # JMX 호스트 이름 설정
        KAFKA_JMX_HOSTNAME: localhost

        # 카프카 로그 디렉토리 위치 지정
        KAFKA_LOG_DIRS: '/tmp/kraft-combined-logs'

        # 클러스터의 고유 식별자 설정
        CLUSTER_ID: 'kafka-cluster'
    volumes:
      - ./data:/var/lib/kafka/data
    restart: always
```

## Spring Boot에서 정확히 한 번 전송 사용하기 (Produce/Comsume)

### KafkaConfig.java
```java
@EnableKafka
@Configuration
public class KafkaConfig {

    @Value("${spring.kafka.bootstrap-servers}")
    private String bootstrapServers;

    @Value("${spring.kafka.consumer.group-id}")
    private String groupId;

    @Bean
    public KafkaTemplate<String, KafkaEvent> kafkaTemplate() {
        return new KafkaTemplate<>(producerFactory());
    }

    @Bean
    public ProducerFactory<String, KafkaEvent> producerFactory() {
        final Map<String, Object> configProps = new HashMap<>();

        // Kafka 브로커의 주소 설정
        configProps.put(ProducerConfig.BOOTSTRAP_SERVERS_CONFIG, bootstrapServers);

        // 메시지 키의 직렬화 방식 설정
        configProps.put(ProducerConfig.KEY_SERIALIZER_CLASS_CONFIG, StringSerializer.class);

        // 메시지 값의 직렬화 방식 설정 (JSON 형식)
        configProps.put(ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG, JsonSerializer.class);

        // 메시지 압축 방식 설정 (Zstandard 알고리즘 사용, 높은 압축률)
        configProps.put(
            ProducerConfig.COMPRESSION_TYPE_CONFIG,
            KafkaConstant.Producer.COMPRESSION_TYPE
        );

        // 정확히 한 번 전송을 위한 설정 //

        // 중복 방지 옵션 활성화
        // (정확히 한 번 전송 필요 조건)
        configProps.put(ProducerConfig.ENABLE_IDEMPOTENCE_CONFIG, true);

        // 메시지 전송 보장 수준 설정 ('all': 모든 복제본에서 메세지 복제 완료를 확인)
        // (정확히 한 번 전송 필요 조건)
        configProps.put(ProducerConfig.ACKS_CONFIG, KafkaConstant.Producer.ACKS_LEVEL);

        // 전송 실패 시 재시도 횟수 설정
        // (정확히 한 번 전송 필요 조건)
        configProps.put(ProducerConfig.RETRIES_CONFIG, KafkaConstant.Producer.RETRY_COUNT);

        // ACK를 받지 않은 상태에서 보낼 수 있는 최대 요청 수 설정 (5)
        configProps.put(
            ProducerConfig.MAX_IN_FLIGHT_REQUESTS_PER_CONNECTION,
            KafkaConstant.Producer.MAX_IN_FLIGHT_REQUESTS_PER_CONNECTION_COUNT
        );

        // 트랜잭션 ID 설정 (정확히 한 번 전송을 위한 트랜잭션 기능 활성화)
        // (정확히 한 번 전송 필요 조건)
        configProps.put(
            ProducerConfig.TRANSACTIONAL_ID_CONFIG,
            KafkaConstant.Producer.TRANSACTION_ID
        );

        return new DefaultKafkaProducerFactory<>(configProps);
    }

    @Bean
    public ConsumerFactory<String, KafkaEvent> consumerFactory() {
        final Map<String, Object> configProps = new HashMap<>();

        // 신뢰할 수 있는 패키지 설정 (역직렬화 시 사용)
        configProps.put(JsonDeserializer.TRUSTED_PACKAGES, KafkaConstant.TRUSTED_PACKAGE);

        // Kafka 브로커의 주소 설정
        configProps.put(ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG, bootstrapServers);

        // 컨슈머 그룹 ID 설정
        configProps.put(ConsumerConfig.GROUP_ID_CONFIG, groupId);

        // 메시지 키의 역직렬화 방식 설정
        configProps.put(ConsumerConfig.KEY_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class);

        // 메시지 값의 역직렬화 방식 설정 (JSON 형식)
        configProps.put(ConsumerConfig.VALUE_DESERIALIZER_CLASS_CONFIG, JsonDeserializer.class);

        // 컨슈머가 커밋된 오프셋을 찾을 수 없을 때의 대처 옵션 설정 ('earliest': 오프셋을 찾을 수 없을 때 가장 오래된 메시지부터 읽기 시작)
        configProps.put(ConsumerConfig.AUTO_OFFSET_RESET_CONFIG, AUTO_OFFSET_RESET_STRATEGY);

        // 정확히 한 번 읽기를 위한 설정 //

        // 트랜잭션 격리 수준 설정 (커밋된 메시지만 읽기 - 프로듀서의 트랜잭션 코디네이터와 연계되어 정확히 한 번 처리를 보장)
        // (정확히 한 번 읽기 필요 조건)
        configProps.put(
            ConsumerConfig.ISOLATION_LEVEL_CONFIG,
            KafkaConstant.Consumer.ISOLATION_LEVEL
        );

        // 자동 커밋 비활성화 - (수동으로 커밋 제어)
        // (정확히 한 번 읽기 필요 조건)
        configProps.put(ConsumerConfig.ENABLE_AUTO_COMMIT_CONFIG, false);

        return new DefaultKafkaConsumerFactory<>(configProps);
    }

    @Bean
    public ConcurrentKafkaListenerContainerFactory<String, KafkaEvent>
    kafkaListenerContainerFactory() {
        final ConcurrentKafkaListenerContainerFactory<String, KafkaEvent> factory = new ConcurrentKafkaListenerContainerFactory<>();
        factory.setConsumerFactory(consumerFactory());

        // 수동 커밋 모드 설정
        // 이를 통해 메시지 처리 완료 후 명시적으로 커밋하여 정확히 한 번 처리를 보장
        factory.getContainerProperties().setAckMode(ContainerProperties.AckMode.MANUAL);
        return factory;
    }

    // Error Handling
    @Bean
    public DeadLetterPublishingRecoverer deadLetterPublishingRecoverer(
        final KafkaTemplate<String, KafkaEvent> template) {
        return new DeadLetterPublishingRecoverer(template);
    }

    @Bean
    public DefaultErrorHandler errorHandler(final DeadLetterPublishingRecoverer recoverer) {
        final DefaultErrorHandler errorHandler = new DefaultErrorHandler(recoverer);
        errorHandler.addNotRetryableExceptions(Exception.class);
        return errorHandler;
    }
}
```

### KafkaConstant.java

```java
@NoArgsConstructor(access = AccessLevel.PRIVATE)
public final class KafkaConstant {

    public static final String TRUSTED_PACKAGE = "com.smilegate.ep.be.tlm.core.common.kafka.event";

    @NoArgsConstructor(access = AccessLevel.PRIVATE)
    public static final class Producer {
        // 모든 복제본으로부터 확인을 받음 (높은 신뢰성)
        public static final String ACKS_LEVEL = "all";

        // 전송 실패 시 최대 3번 재시도
        public static final int RETRY_COUNT = 3;

        // ACK 응답을 받지 않은 상태에서 최대 5개까지 요청 가능
        public static final int MAX_IN_FLIGHT_REQUESTS_PER_CONNECTION_COUNT = 5;

        // Zstandard 압축 알고리즘 사용 (높은 압축률)
        public static final String COMPRESSION_TYPE = "zstd";

        // 고유한 트랜잭션 ID 생성
        public static final String TRANSACTION_ID = "tx-" + System.currentTimeMillis();
    }

    @NoArgsConstructor(access = AccessLevel.PRIVATE)
    public static final class Consumer {
        // 트랜잭션이 완료된 메시지만 읽음
        public static final String ISOLATION_LEVEL = "read_committed";

        // 오프셋을 찾을 수 없을 때 가장 오래된 메시지부터 읽기 시작
        public static final String AUTO_OFFSET_RESET_STRATEGY = "earliest";

        public static final String CONSUMER_FACTORY_BEAN = "kafkaListenerContainerFactory";
    }

    @NoArgsConstructor(access = AccessLevel.PRIVATE)
    public static final class Topic {
        public static final String PC_CONTROL = "tlm.v1.pc-control";
        public static final String PC_CONTROL_META_EVENT = "tlm.v1.pc-event";
        public static final String ATTENDANCE_SUMMARY = "tlm.v1.attendance-summary";
        public static final String ATTENDANCE_SET_SUMMARY = "tlm.v1.attendance-set-summary";
    }

    @NoArgsConstructor(access = AccessLevel.PRIVATE)
    public static final class ConsumerGroup {
        public static final String TLM = "tlm-consumer-group";
    }

    @NoArgsConstructor(access = AccessLevel.PRIVATE)
    public static final class ErrorHandler {
        public static final String DLT_HANDLER = "customKafkaListenerErrorHandler";
    }
}
```

### KafkaEvent
```java
public interface KafkaEvent {
    String getTopic();
}

public record TestEvent(
    String recordField1,
    String recordField2,
    String recordField3
) implements KafkaEvent {

    @Override
    public String getTopic() {
        return KafkaConstant.Topic.TEST_TOPIC;
    }
}
```

### KafkaProducerImpl.java
```java
@Component
@RequiredArgsConstructor
public class KafkaProducerImpl implements KafkaProducer {

    private static final Logger log = LoggerFactory.getLogger(KafkaProducerImpl.class);
    private static final String SUCCESSFUL_LOG_MESSAGE = "Sent message to topic: {}, partition: {}, offset: {}";
    private static final String FAIL_LOG_MESSAGE = "Unable to send message to Kafka";

    private final KafkaTemplate<String, KafkaEvent> kafkaTemplate;

    @Override
    @Transactional
    public void sendMessage(final KafkaEvent event) {
        final ProducerRecord<String, KafkaEvent> record = new ProducerRecord<>(event.getTopic(), generateKey(), event);

        kafkaTemplate.executeInTransaction(operations -> {
                final CompletableFuture<SendResult<String, KafkaEvent>> future = operations.send(record);
                future.whenComplete((result, exception) -> {
                        if (exception == null) {
                            logSuccessfulSend(result);
                        } else {
                            logFailedSend(exception);
                        }
                    });
                return null;
            });
    }

    private String generateKey() {
        return UUID.randomUUID().toString();
    }

    private void logSuccessfulSend(final SendResult<String, KafkaEvent> result) {
        if (log.isInfoEnabled()) {
            log.info(
                SUCCESSFUL_LOG_MESSAGE,
                result.getRecordMetadata().topic(),
                result.getRecordMetadata().partition(),
                result.getRecordMetadata().offset()
            );
        }
    }

    private void logFailedSend(final Throwable ex) {
        log.error(FAIL_LOG_MESSAGE, ex);
    }
}
```

### KafkaConsumerImpl.java
```java
@Component
public class KafkaConsumerImpl {

    private static final Logger log = LoggerFactory.getLogger(KafkaConsumerImpl.class);
    private static final String FAILED_LOG_MESSAGE = "Occurred Unknown Consumer Error";

    private final MessageHandler messageHandler;

    @Autowired
    public KafkaConsumerService(final MessageHandler messageHandler) {
        this.messageHandler = messageHandler;
    }

    @KafkaListener(
        topics = PC_CONTROL_META_EVENT,
        groupId = TLM,
        errorHandler = DLT_HANDLER,
        containerFactory = CONSUMER_FACTORY_BEAN
    )
    public void listenWorkTimeUpdates(
        final ConsumerRecord<String, String> kafkaRecord, final Acknowledgment acknowledgment) {
        try {
            messageHandler.handle(kafkaRecord.value(), kafkaRecord.topic());

            // 메시지 처리가 성공적으로 완료된 후에만 수동으로 커밋해야 함
            acknowledgment.acknowledge();
        } catch (final Exception e) {
            // 예외 발생 시 커밋하지 않고 예외를 throw하여 메시지 재처리 가능하게 함
            log.error(FAILED_LOG_MESSAGE, e);
            throw new ListenerExecutionFailedException(FAILED_LOG_MESSAGE, e);
        }
    }
}
```

### KafkaDLTHandler.java
```java
@Component
@RequiredArgsConstructor
public class CustomKafkaListenerErrorHandler implements KafkaListenerErrorHandler {

    private final KafkaTemplate<String, KafkaEvent> kafkaTemplate;

    private static final String DLT_POSTFIX = ".DLT";
    private static final String UNKNOWN_DLT = "unknown_topic.DLT";

    @Override
    public Object handleError(
        final Message<?> message, final ListenerExecutionFailedException exception) {
        final Object payload = message.getPayload();

        if (payload instanceof final ConsumerRecord<?, ?> consumerRecord) {
            final String dltName = consumerRecord.topic() + DLT_POSTFIX;
            kafkaTemplate.send(
                dltName,
                consumerRecord.key().toString(),
                new DeadLetterEvent(
                    dltName,
                    consumerRecord.key().toString(),
                    (KafkaEvent) consumerRecord.value()));
        } else {
            kafkaTemplate.send(
                UNKNOWN_DLT, null, new DeadLetterEvent(UNKNOWN_DLT, null, (KafkaEvent) payload));
        }
        return null;
    }
}
```

이 방식으로 프로듀서는 비동기 방식의 정확히 한 번 전송을 설정했고 컨슈머도 트랜잭션 코디네이터에 의한 메세지를 읽고 수동으로 커밋하도록하여 정확히 한 번 동작을 보장하도록 했다.

물론 최선의 환경적인 요소는 갖추었지만 그럼에도 불구하고 정확히 한 번 동작이 실패할 수도 있다.

만약 프로듀서의 트랜잭션 커밋에 대해서 예기치 못한 네트워크 에러로 전달되지 않으면 메세지를 중복으로 읽게 될 수도 있다.

그래서 이러한 상황을 대비하여 로직으로 중복 처리하지 않도록 풀어나가야하지만 매 컨슘 시 부가적인 로직이 추가되는 만큼 성능에 악영향이 있을 수 있어 카프카를 사용하는 이점이 꽤 상쇄될 수 있을 것으로 보인다.

정말 민감한 데이터가 아니라면 어느정도는 믿음으로 가도 좋을 것 같다.
