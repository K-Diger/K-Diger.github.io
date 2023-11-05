---

title: MongoDB와 Spring Boot 함께 사용하기
date: 2023-11-05
categories: [Java]
tags: [Java]
layout: post
toc: true
math: true
mermaid: true

---

# 연동 과정

## DBMS 셋팅

MongoDB에 접속해서 DB와 Collection을 생성해준다. 여기서 Collection은 RDBMS의 Table과 같은 개념이다.

---

## Spring Boot 셋팅

### Gradle 의존성 추가

아래 의존성을 추가한다.

```groovy
implementation 'org.springframework.boot:spring-boot-starter-data-mongodb'
```

### Application.yml DBMS 경로 추가

```yaml
spring:
  data:
    mongodb:
      uri: mongodb+srv://{USER_NAME}:{USER_PASSWORD}@{CLUSTER_NAME}.ho2nb0a.mongodb.net/{COLLECTION_NAME}?retryWrites=true&w=majority
```

위와 같이 UserName, Password, ClusterName, CollectionName을 정의해주면 끝난다.

---

# 너무 오랜만이라 까먹었던 직렬화 사용법 (Getter Method)

HTTP 응답 폼을 만들기 위해 아래와 같이 작성했는데 자꾸 빈 응답 값을 내려주는 현상이 발생했다.

```java
package com.diger.notonlysqlboard.util.responseform;

import com.fasterxml.jackson.annotation.JsonInclude;
import lombok.AllArgsConstructor;
import lombok.NoArgsConstructor;
import org.springframework.http.HttpStatus;

@JsonInclude(JsonInclude.Include.NON_NULL)
@AllArgsConstructor
@NoArgsConstructor
public class ResponseForm<T> {

    private Integer status;
    private T data;

    public static <T> ResponseForm<T> success(
            HttpStatus httpStatus
    ) {
        return new ResponseForm<>(
                httpStatus.value(),
                null
        );
    }

    public static <T> ResponseForm<T> success(
            HttpStatus httpStatus,
            T data
    ) {
        return new ResponseForm<T>(
                httpStatus.value(),
                data
        );
    }
}
```

직렬화 할 때 JVM 내부적으로 Getter메서드를 활용한다는 점을 깜빡하고 Getter를 추가하지 않았기 때문에 직렬화가 이루어지지 않게 되었다.

직렬화 대상을 다룰땐 Getter를 고려해야한다는 점을 잊지 않도록 주의하자!

---

# MongoRepository는 Dirty Checking이 없다.

당연하게도 JPA와 딱히 관련이 없는 MongoDB는 Dirty Checking 기능이 존재하지 않는다.

즉, 아래와 같은 코드가 있을 때

```java
@Service
@Transactional
@RequiredArgsConstructor
public class UserAuthenticator {

    private final UserRepository userRepository;

    public void execute(LoginId loginId, Password password) {
        User user = userRepository.findByLoginIdAndPassword(
                loginId,
                password
        );

        user.updateAuthority(Authority.NORMAL);
    }
}
```

흔히 쓰던 JPA기반의 RDBMS였으면 Dirty Checking이 발동되어 save()메서드를 호출하지 않아도 자동으로 변경사항이 반영된다.

하지만 NoSQL은 영속성 컨텍스트를 사용하는 방식이 아니기 때문에 아래와 같이 직접 save()메서드를 호출해줘야한다.

또한 @Transactioanl 애노테이션도 딱히 필요가 없다. 만약 MongoDB가 지원하는 트랜잭션을 사용하고 싶다면 별도의 설정을 해줘야한다.

```java
@Service
@RequiredArgsConstructor
public class UserAuthenticator {

    private final UserRepository userRepository;

    public void execute(LoginId loginId, Password password) {
        User user = userRepository.findByLoginIdAndPassword(
                loginId,
                password
        );

        user.updateAuthority(Authority.NORMAL);
        userRepository.save(user);
    }
}
```

```java
@Configuration
public class MongoTransactionConfig {

    @Bean
    public MongoTransactionManager specifyTransactionManager(MongoDatabaseFactory mongoDatabaseFactory) {
        return new MongoTransactionManager(mongoDatabaseFactory);
    }
}
```
