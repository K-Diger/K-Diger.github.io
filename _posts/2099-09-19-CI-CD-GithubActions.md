---

title: CI/CD 깃헙 액션으로만 끝내버리기
date: 2023-09-19
categories: [CI, CD]
tags: [CI, CD]
layout: post
toc: true
math: true
mermaid: true

---

# 개요는 생략한다.

CI/CD는 구글링하면 뻔한 이야기를 많이 해놓았기 때문에 그냥 그 정의를 갖다 쓴다고 한다.

이 글에서 다루는 내용은 어떻게 깃헙 액션으로 CI/CD를 적용할지에 대해서만 알아본다.

# 깃헙 액션으로 CI 돌려보기

```yml
name: Sulasang CI/CD with Gradle, Github Actions, Docker

on:
  push:
    branches: [ "main" ]
  pull_request:
    branches: [ "main" ]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: 체크아웃
        uses: actions/checkout@v3

      - name: Gradle 캐싱
        uses: actions/cache@v3
        with:
          path: |
            ~/.gradle/caches
            ~/.gradle/wrapper
          key: ${{ runner.os }}-gradle-${{ hashFiles('**/*.gradle*', '**/gradle-wrapper.properties') }}
          restore-keys: |
            ${{ runner.os }}-gradle-

      - name: JDK 17 설치
        uses: actions/setup-java@v3
        with:
          java-version: 17
          distribution: adopt

      - name: 빌드 권한 부여
        run: chmod +x ./gradlew
        shell: bash

      - name: 프로젝트 빌드
        run: ./gradlew clean build
        shell: bash
```

메인 브랜치에 푸쉬 혹은 풀리가 발생했을 때 CI를 돌린다.

혼자하는 거기 때문에 별다른 정책을 정하지 않았지만 PR에 대한 CI는 Git 전략을 가져간다고 했을 때

develop, feature 까지도 추가해도 좋을 것 같다.

---

# 깃헙 액션으로 CD 돌려보기

```yml
      - name: API 빌드 파일 복사
        uses: appleboy/scp-action@master
        with:
          host: ${{ secrets.EC2_HOST }}
          username: ${{ secrets.EC2_USERNAME }}
          key: ${{ secrets.EC2_KEY }}
          source: "sulasang-api/build/libs/sulasang-api-0.0.1-SNAPSHOT.jar"
          target: "/home/ubuntu"

      - name: API EC2 인스턴스 접속 및 애플리케이션 실행
        uses: appleboy/ssh-action@master
        with:
          host: ${{ secrets.EC2_HOST }}
          username: ${{ secrets.EC2_USERNAME }}
          key: ${{ secrets.EC2_KEY }}
          script: |
            sudo cd /home/ubuntu
            sudo fuser -k 8080/tcp
            sudo ./start-api.sh

      - name: Crawler 빌드 파일 복사
        uses: appleboy/scp-action@master
        with:
          host: ${{ secrets.EC2_HOST }}
          username: ${{ secrets.EC2_USERNAME }}
          key: ${{ secrets.EC2_KEY }}
          source: "sulasang-crawler/build/libs/sulasang-crawler-0.0.1-SNAPSHOT.jar"
          target: "/home/ubuntu"

      - name: Crawler EC2 인스턴스 접속 및 애플리케이션 실행
        uses: appleboy/ssh-action@master
        with:
          host: ${{ secrets.EC2_HOST }}
          username: ${{ secrets.EC2_USERNAME }}
          key: ${{ secrets.EC2_KEY }}
          script: |
            sudo cd /home/ubuntu
            sudo fuser -k 8081/tcp
            sudo ./start-crawler.sh
```

위 CI 스크립트에 이어 붙이면 된다. 이 구문은 깃 시크릿에 등록된 환경변수를 사용하고 있고

CI에서 만들어진 빌드 파일을 실제 서버에 복사하는 것이다.

빌드 파일을 복사할 때 프로토콜은 scp를 사용한다.

또한 멀티 모듈로 구성했기 때문에, 각 모듈을 따로 따로 종료 및 새로 띄워주는 동작을 수행한다.

사실 멀티모듈이라 모든 모듈을 다시 띄워야할 필요는 없지만 내가 생각한 시나리오는 Domain(Core)모듈이 변경되어

모든 모듈에 영향을 전파해야하는 상황을 가정했기 때문에 이렇게 구성했다.

그리고 EC2인스턴스에서 실행할 스크립트는 대충 아래와 같이 작성했다.

```shell
sudo java -jar -Dspring.profiles.active=prod sulasang-api/build/libs/sulasang-api-0.0.1-SNAPSHOT.jar --jasypt.encryptor.password="abcdefg" > /home/ubuntu/nohup.out 2>&1 &
```

위 쉘 스크립트가 의미하는 것은 다음과 같다.

1. 백그라운드로 Spring Boot 실행
2. 실행 시 프로파일 설정
3. Jasypt Password 설정
4. nohup logback 설정

---

기본 틀은 위 내용이 전부였다.

사실 도커를 기반으로 하다가 도저히 잘 되지가 않아서 가장 직관적인 방법으로 해봤는데 이게 가장 단시간에 적용된 CI/CD 방법이였다.

이렇게 편한 방법이 있음에도 불구하고 다른 방법으로 CD를 하는 이유에는 조금 더 알아보고 자문을 구해보도록 하겠다.
