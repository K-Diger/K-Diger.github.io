---

title: Kong Gateway 사용해보기
date: 2024-03-29
categories: [KongGateway]
tags: [KongGateway]
layout: post
toc: true
math: true
mermaid: true

---

사내 신규 프로젝트를 시작하면서 Spring Cloud Gateway가 아닌 Kong Gateway를 사용하기로 결정됐다.

KongGateway의 장단점을 정리하고 직접 사용해보자.

---

# 1. Docker, DB-Less Kong 시작하기

## 1.1 셋팅

![](https://github.com/K-Diger/K-Diger.github.io/blob/main/images/Konggateway/folders.png?raw=true)

위와 같은 패키지 구조를 가진 프로젝를 생성했다.

엔드포인트에 단순한 문자열을 반환하는 컨트롤러가 정의되어있고

`kong`폴더내에는 `kong.yml`과 `docker-compose`를 정의하기 위한 폴더가 있다.

여기서 `컨트롤러`는 다른 프로젝트 내의 `마이크로서비스`라고 가정한다.

---

## 1.2 마이크로서비스 빌드

```dockerfile
FROM openjdk:17-jdk

COPY build/libs/*.jar app.jar

ENTRYPOINT ["java","-jar","/app.jar"]
```

위와 같이 마이크로서비스를 Dockerfile로 빌드했다.

---

## 1.3 Kong 설정 폴더 구조

![img.png](https://github.com/K-Diger/K-Diger.github.io/blob/main/images/Konggateway/kongfolder.png?raw=true)

지금은 프로젝트 폴더 내에서 위와 같이 폴더링을 했지만 실제로는 Gateway 인스턴스 내부에서 이 구조를 유사하게 가져가면 될 것 같다.

---

## 1.4 Kong yml 셋팅

```yaml
_format_version: "3.0"
_transform: true

services:
  - name: user-service
    url: http://docker.for.mac.localhost:8080
    routes:
      - name: user-route
        paths:
          - /api/v1/users
        methods:
          - GET
          - POST
          - PUT
          - DELETE
        protocols:
          - http
```

`http://localhost:8000/api/v1/users`로 요청하면 `http://localhost:8080/users`로 라우팅하기 위한 전략이다.


`service`블럭에서 서비스를 정의한다. 이 때 `url`속성에 실제 서비스의 `프로토콜`, `주소`, `포트`를 명시하면되는데, 도커를 사용하는 환경에서는 반드시 `http://docker.for.mac.localhost`이렇게 해주어야 제대로 서비스를 찾아내더라..

`routes`블럭에서는 어떤 경로로 들어온 것에 대해서 라우팅을 받아낼지를 의미하는데 `/api/v1/users`경로로 요청하면 실제 마이크로서비스에 도착하는 경로는 ``로 도착하고 `/api/v1/users/users`경로로 요청하면 실제 마이크로서비스에는 `/users`로 도착하게된다.

---

## 1.5 DockerCompose 정의 및 실행

```yaml
version: "3.9"

services:
  application:
    restart: always
    networks:
      - kong-net
    container_name: user-service
    build:
      context: ../
      dockerfile: Dockerfile
    ports:
      - "8080:8080"

  kong:
    restart: always
    networks:
      - kong-net
    image: kong
    volumes:
      - "./config:/usr/local/kong/declarative"
    container_name: kong-gateway
    environment:
      - KONG_DATABASE= off
      - KONG_DECLARATIVE_CONFIG=/usr/local/kong/declarative/kong.yml
      - KONG_PROXY_ACCESS_LOG=/dev/stdout
      - KONG_ADMIN_ACCESS_LOG=/dev/stdout
      - KONG_PROXY_ERROR_LOG=/dev/stderr
      - KONG_ADMIN_ERROR_LOG=/dev/stderr
      - KONG_ADMIN_LISTEN=0.0.0.0:8001, 0.0.0.0:8444 ssl
      - KONG_LOG_LEVEL=debug
      - KONG_PLUGINS=bundled
    ports:
      - "8000:8000"
      - "8001:8001"
      - "8443:8443"
      - "8444:8444"

networks:
  kong-net:
    external: true
```

위와 같이 도커 컴포즈 파일을 정의한다. 마이크로서비스를 도커 이미지로 빌드하고, 그 이미지와 kong gateway를 Pulling하여 컨테이너를 함께 띄운다. 또한 도커 네트워크를 kong-network로 함께 묶는다.

---

