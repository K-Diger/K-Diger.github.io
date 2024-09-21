---

title: Docker, Docker Compose로 Kafka 사용하기
date: 2023-12-29
categories: [Kafka, Docker, DockerCompose]
tags: [Kafka, Docker, DockerCompose]
layout: post
toc: true
math: true
mermaid: true

---

# 참고 자료

[참고 자료](https://devocean.sk.com/blog/techBoardDetail.do?ID=164007)

---

# 1. Docker Compose 설치하기

```shell
$ sudo hdiutil attach Docker.dmg
$ sudo /Volumes/Docker/Docker.app/Contents/MacOS/install
$ sudo hdiutil detach /Volumes/Docker
```

위 명령어를 바탕으로 Docker Compose를 설치한 후 `$ docker-compose version`으로 버전을 확인하여 올바르게 설치되었는지 확인한다.

---

# 2. Docker Compose 설정파일 작성하기

Kafka Broker를 사용하기 위한 프로젝트 폴더 내에 최상단 위치에 아래와 같은 yml파일을 작성한다.

파일 이름은 `docker-compose.yml`으로 지정했다.

### docker-compose-yml
```yml
# 도커 컴포즈 버전을 의미한다.
version: '2'

# 도커 컴포즈는 도커 컨테이너로 수행될 서비스들은 services 하위에 기술한다.
services:

  # 주키퍼에 관한 설정을 명시한다. (주키퍼 서비스 네임이다.)
  zookeeper:

    # confluentinc/cp-zookeeper:latest 주키퍼 이미지를 사용한다.
    # 실전에서 사용하려면 latest 라는 태그를 사용하지 말고, 정확히 원하는 버젼을 기술해서 사용하는 것을 권장한다.
    # latest라고 태그를 지정하면, 매번 컨테이너를 실행할때마다 최신버젼을 다운받아 실행하므로 변경된 버젼으로 인해 원하지 않는 결과를 볼 수 있다.
    image: confluentinc/cp-zookeeper:latest

    # confluentinc 는 몇가지 환경 변수를 설정할 수 있다. environment 하위에 필요한 환경을 작성하자.
    environment:

      # 주키퍼 클러스터에서 주키퍼를 식별할 아이디이다. 동일 클러스터 내에서 이 값은 중복되면 안된다. 단일 브로커이기 때문에 이 값은 의미가 없다
      ZOOKEEPER_SERVER_ID: 1

      # 주키퍼 클라이언트 포트를 기본 주키퍼의 포트인 2181로 지정한다. 즉, 컨테이너 내부에서 주키퍼는 2181로 실행된다.
      ZOOKEEPER_CLIENT_PORT: 2181

      # 주키퍼가 클러스터를 구성할때 동기화를 위한 기본 틱 타임을 지정한다. ms단위로 지정할 수 있으며 여기서는 2000으로 설정했으니 2초가 된다.
      ZOOKEEPER_TICK_TIME: 2000

      # 주키퍼 초기화를 위한 제한 시간을 설정한다. (주키퍼 서버가 리더와 초기 연결을 형성하는 데 허용되는 'tick' 수를 지정)
      # 주키퍼 클러스터는 쿼럼이라는 과정을 통해서 마스터를 선출하게 된다. 이때 주키퍼들이 리더에게 커넥션을 맺을때 지정할 초기 타임아웃 시간이다.
      # 타임아웃 시간은 이전에 지정한 ZOOKEEPER_TICK_TIME 단위로 설정된다.
      # ZOOKEEPER_TICK_TIME을 2000으로 지정했고, ZOOKEEPER_INIT_LIMIT을 5로 잡았으므로 2000 * 5 = 10000ms 즉, 10초가 된다.
      # 즉, 10초내에 초기 연결을 수행하지 못한다면 유효한 노드로 취급하지 않게된다.
      # 이 옵션은 멀티 브로커에서 유효한 속성이다.
      ZOOKEEPER_INIT_LIMIT: 5

      # 이 시간은 주키퍼 리더와 나머지 서버들의 싱크 타임이다. (주키퍼 서버가 리더와 동기화를 유지하는 데 허용되는 'tick' 수를 지정)
      # 이 시간내 싱크응답이 들어오는 경우 클러스터가 정상으로 구성되어 있음을 확인하는 시간이다.
      # 여기서 2로 잡았으므로 2000 * 2 = 4000 으로 4초가 된다. 즉, 4초내에 싱크 응답이 없다면 연결이 끊어진 것으로 간주한다.
      # 이 옵션은 멀티 브로커에서 유효한 속성이다.
      ZOOKEEPER_SYNC_LIMIT: 2

    # 이 항목은 호스트와 컨테이너 간의 네트워크 포트 매핑을 정의한다.
    # "22181:2181"은 호스트의 22181 포트를 컨테이너의 2181 포트에 매핑하라는 것이다.
    ports:
      - "22181:2181"

  # 카프카에 관한 설정을 명시한다. (카프카 서비스 네임이다.)
  kafka:

    # 카프카 브로커는 confluentinc/cp-kafka:latest 를 사용한다.
    # 태그는 latest보다는 지정된 버젼을 사용하는것을 추천한다.
    image: confluentinc/cp-kafka:latest

    # docker-compose 에서는 서비스들의 우선순위를 지정해 주기 위해서 depends_on 을 이용한다.
    # zookeeper 라고 지정하였으므로, 카프카는 주키퍼가 먼저 실행되어 있어야 컨테이너가 올라오게 된다.
    depends_on:
      - zookeeper

    # kafka 브로커의 포트를 의미한다.
    # 외부포트:컨테이너내부포트 형식으로 지정한다.
    ports:
      - "29092:29092"

    # 카프카 브로커를 위한 환경 변수를 지정한다.
    environment:

      # 카프카 브로커 아이디를 지정한다. 유니크해야하며 지금 예제는 단일 브로커기 때문에 없어도 무방하다.
      KAFKA_BROKER_ID: 1

      # 카프카가 주키퍼에 커넥션하기 위한 대상을 지정한다.
      # 여기서는 zookeeper(서비스이름):2181(컨테이너내부포트)로 대상을 지정했다.
      KAFKA_ZOOKEEPER_CONNECT: 'zookeeper:2181'

      # 외부에서 접속하기 위한 리스너 설정을 한다.
      KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://kafka:9092,PLAINTEXT_HOST://localhost:29092

      # 보안을 위한 프로토콜 매핑이디. 이 설정값은 KAFKA_ADVERTISED_LISTENERS 과 함께 key/value로 매핑된다.
      KAFKA_LISTENER_SECURITY_PROTOCOL_MAP: PLAINTEXT:PLAINTEXT,PLAINTEXT_HOST:PLAINTEXT

      # 도커 내부에서 사용할 리스너 이름을 지정한다.
      # 이전에 매핑된 PLAINTEXT가 사용되었다.
      KAFKA_INTER_BROKER_LISTENER_NAME: PLAINTEXT

      # single 브로커인경우에 지정하여 1로 설정했다.
      # 멀티 브로커는 기본값을 사용하므로 이 설정이 필요 없다.
      KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR: 1

      # 카프카 그룹이 초기 리밸런싱할때 컨슈머들이 컨슈머 그룹에 조인할때 대기 시간이다.
      KAFKA_GROUP_INITIAL_REBALANCE_DELAY_MS: 0
```

---

# 3. docker-compose 설정파일을 바탕으로 docker-compose 실행하기

아래 명령어를 입력하여 위에서 설정한 주키퍼, 카프카 컨테이너를 띄운다.

```shell
$ docker-compose -f docker-compose.yml up -d
```

실행 시킨 후 아래 명령어로 정상적으로 컨테이너가 올라왔는지 확인한다.

```shell
$ docker ps
```

---

# 4. docker-compose를 사용하여 토픽 생성하기

아래 명령어를 사용하여 토픽을 생성한다.

```shell
docker-compose exec kafka kafka-topics --create --topic ${토픽 네이밍} --bootstrap-server kafka:9092 --replication-factor 1 --partitions 1
```

이후 아래 명령어로 생성된 토픽을 확인해볼 수 있다.

```shell
$ docker-compose exec kafka kafka-topics --describe --topic ${토픽 네이밍} --bootstrap-server kafka:9092
```

### 토픽 생성 및 조회 시 사용된 도커 컴포즈 명령어의 의미

`docker-compose` : 명령어를 수행한다.

`exec` : 컨테이너 내에서 커맨드를 수행하도록 한다.

`kafka` : 우리가 설정으로 생성한 브로커(서비스) 이름이다.

`kafka-topics` : 카프카 토픽에 대한 명령을 실행한다.

`--describe` : 생성된 토픽에 대한 상세 설명을 보여달라는 옵션이다.

`--topic` : 생성한 토픽 이름을 지정한다.

`--bootstrap-server service:port` : bootstrap-server는 kafak 브로커 서비스를 나타낸다. 이때 서비스:포트 로 지정하여 접근할 수 있다. 결과로 토픽이름, 아이디, 복제계수, 파티션, 리더, 복제정보, isr 등을 확인할 수 있다.

---

# 5. docker-compose로 생성된 토픽에 대한 컨슈머 만들기

컨슈머를 먼저 실행하는 이유는, 일반적으로 컨슈머가 메시지를 컨슘하려고 대기하고 있고, 송신자가 메시지를 생성해서 보내기 때문이다.

1. docker-compose exec kafka bash 를 통해서 컨테이너 내부의 쉘로 접속한다.
2. 이후 kafka-console-consumer 를 이용하여 컨슘한다.
3. 역시 컨슘할 토픽을 지정하고, 브로커를 지정하기 위해서 --bootstrap-server 를 이용했다.

```shell
$ docker-compose exec kafka bash
```

---

# 6. docker-compose로 생성된 토픽에 대한 프로듀서 만들기

```shell
$ docker-compose exec kafka bash
```

---
