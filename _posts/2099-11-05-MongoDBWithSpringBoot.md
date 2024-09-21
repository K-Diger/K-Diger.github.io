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

---

# 컬렉션 조회 시 이상현상

아래와 같이 게시글 엔티티를 정의했다.

### Board.java

```java
@Getter
@AllArgsConstructor
@Builder
@Document("board")
public class Board extends BaseDocument {

    private Title title;
    private TextContent textContent;
    private StaticContent staticContent;
    private String writer;
}
```

### StaticContent.java

```java
@Getter
@Document(collection = "staticContent")
public class StaticContent {
    private final List<String> links = new ArrayList<>();

    public StaticContent(List<String> value) {
        links.addAll(value);
    }

    public void validate(String value) {
        if (value.isBlank()) {
            throw new IllegalArgumentException("Content Link Must Not Be Blank");
        }
    }
}
```

위와 같은 List를 가지고 있는 값 타입으로 래핑해서 사용하고 있는 것인데, 이 객체를 가지고 있는 Board를 조회하려고하면 매핑 문제가 발생한다.

```text
Servlet.service() for servlet [dispatcherServlet] in context with path [] threw exception [Request processing failed: org.springframework.data.mapping.MappingException: No property value found on entity class com.diger.notonlysqlboard.core.board.domain.StaticContent to bind constructor parameter to] with root cause```

org.springframework.data.mapping.MappingException: No property value found on entity class com.diger.notonlysqlboard.core.board.domain.StaticContent to bind constructor parameter to

```

이 원인과 해결방법에 대해선 조금 더 알아보기로한다.

---

# MongoDB에서 MultipartFile을 사용하다 생긴 일

아래와 같이 상위 테이블을 하나 선언했다.

```java
@Getter
@AllArgsConstructor
@Document("board")
public class Board extends BaseDocument {

    private final Title title;

    private final TextContent textContent;

    private final StaticContent staticContent;

    private final User writer;

    private final List<Comment> comments;
}
```

그리고 아래와 같이 댓글에 대한 테이블을 선언했따.

```java
@Getter
@AllArgsConstructor
@Document(collection = "comment")
public class Comment extends BaseDocument {
    private final String value;
    private final MultipartFile file;
    private final User writer;
}
```

위 도메인을 선언하고 스프링을 돌리니까 아래와 같은 에러가 발생했다.

```shell
2023-11-08T19:23:17.573+09:00 ERROR 5873 --- [           main] o.s.boot.SpringApplication               : Application run failed

org.springframework.beans.factory.UnsatisfiedDependencyException: Error creating bean with name 'boardCrateApi' defined in file [/Users/Diger/Desktop/Code/NotOnlySQL-Board/out/production/classes/com/diger/notonlysqlboard/core/board/controller/BoardCrateApi.class]: Unsatisfied dependency expressed through constructor parameter 0: Error creating bean with name 'boardCreator' defined in file [/Users/Diger/Desktop/Code/NotOnlySQL-Board/out/production/classes/com/diger/notonlysqlboard/core/board/service/BoardCreator.class]: Unsatisfied dependency expressed through constructor parameter 0: Error creating bean with name 'boardRepository' defined in com.diger.notonlysqlboard.core.board.repository.BoardRepository defined in @EnableMongoRepositories declared on NotOnlySqlBoardApplication: Unable to make field private final java.lang.String java.io.File.path accessible: module java.base does not "opens java.io" to unnamed module @3ce1e309
	at org.springframework.beans.factory.support.ConstructorResolver.createArgumentArray(ConstructorResolver.java:801) ~[spring-beans-6.0.13.jar:6.0.13]
	at org.springframework.beans.factory.support.ConstructorResolver.autowireConstructor(ConstructorResolver.java:240) ~[spring-beans-6.0.13.jar:6.0.13]
	at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.autowireConstructor(AbstractAutowireCapableBeanFactory.java:1352) ~[spring-beans-6.0.13.jar:6.0.13]
	at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.createBeanInstance(AbstractAutowireCapableBeanFactory.java:1189) ~[spring-beans-6.0.13.jar:6.0.13]
	at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.doCreateBean(AbstractAutowireCapableBeanFactory.java:560) ~[spring-beans-6.0.13.jar:6.0.13]
	at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.createBean(AbstractAutowireCapableBeanFactory.java:520) ~[spring-beans-6.0.13.jar:6.0.13]
	at org.springframework.beans.factory.support.AbstractBeanFactory.lambda$doGetBean$0(AbstractBeanFactory.java:325) ~[spring-beans-6.0.13.jar:6.0.13]
	at org.springframework.beans.factory.support.DefaultSingletonBeanRegistry.getSingleton(DefaultSingletonBeanRegistry.java:234) ~[spring-beans-6.0.13.jar:6.0.13]
	at org.springframework.beans.factory.support.AbstractBeanFactory.doGetBean(AbstractBeanFactory.java:323) ~[spring-beans-6.0.13.jar:6.0.13]
	at org.springframework.beans.factory.support.AbstractBeanFactory.getBean(AbstractBeanFactory.java:199) ~[spring-beans-6.0.13.jar:6.0.13]
	at org.springframework.beans.factory.support.DefaultListableBeanFactory.preInstantiateSingletons(DefaultListableBeanFactory.java:973) ~[spring-beans-6.0.13.jar:6.0.13]
	at org.springframework.context.support.AbstractApplicationContext.finishBeanFactoryInitialization(AbstractApplicationContext.java:950) ~[spring-context-6.0.13.jar:6.0.13]
	at org.springframework.context.support.AbstractApplicationContext.refresh(AbstractApplicationContext.java:616) ~[spring-context-6.0.13.jar:6.0.13]
	at org.springframework.boot.web.servlet.context.ServletWebServerApplicationContext.refresh(ServletWebServerApplicationContext.java:146) ~[spring-boot-3.1.5.jar:3.1.5]
	at org.springframework.boot.SpringApplication.refresh(SpringApplication.java:738) ~[spring-boot-3.1.5.jar:3.1.5]
	at org.springframework.boot.SpringApplication.refreshContext(SpringApplication.java:440) ~[spring-boot-3.1.5.jar:3.1.5]
	at org.springframework.boot.SpringApplication.run(SpringApplication.java:316) ~[spring-boot-3.1.5.jar:3.1.5]
	at org.springframework.boot.SpringApplication.run(SpringApplication.java:1306) ~[spring-boot-3.1.5.jar:3.1.5]
	at org.springframework.boot.SpringApplication.run(SpringApplication.java:1295) ~[spring-boot-3.1.5.jar:3.1.5]
	at com.diger.notonlysqlboard.NotOnlySqlBoardApplication.main(NotOnlySqlBoardApplication.java:14) ~[classes/:na]
Caused by: org.springframework.beans.factory.UnsatisfiedDependencyException: Error creating bean with name 'boardCreator' defined in file [/Users/Diger/Desktop/Code/NotOnlySQL-Board/out/production/classes/com/diger/notonlysqlboard/core/board/service/BoardCreator.class]: Unsatisfied dependency expressed through constructor parameter 0: Error creating bean with name 'boardRepository' defined in com.diger.notonlysqlboard.core.board.repository.BoardRepository defined in @EnableMongoRepositories declared on NotOnlySqlBoardApplication: Unable to make field private final java.lang.String java.io.File.path accessible: module java.base does not "opens java.io" to unnamed module @3ce1e309
	at org.springframework.beans.factory.support.ConstructorResolver.createArgumentArray(ConstructorResolver.java:801) ~[spring-beans-6.0.13.jar:6.0.13]
	at org.springframework.beans.factory.support.ConstructorResolver.autowireConstructor(ConstructorResolver.java:240) ~[spring-beans-6.0.13.jar:6.0.13]
	at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.autowireConstructor(AbstractAutowireCapableBeanFactory.java:1352) ~[spring-beans-6.0.13.jar:6.0.13]
	at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.createBeanInstance(AbstractAutowireCapableBeanFactory.java:1189) ~[spring-beans-6.0.13.jar:6.0.13]
	at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.doCreateBean(AbstractAutowireCapableBeanFactory.java:560) ~[spring-beans-6.0.13.jar:6.0.13]
	at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.createBean(AbstractAutowireCapableBeanFactory.java:520) ~[spring-beans-6.0.13.jar:6.0.13]
	at org.springframework.beans.factory.support.AbstractBeanFactory.lambda$doGetBean$0(AbstractBeanFactory.java:325) ~[spring-beans-6.0.13.jar:6.0.13]
	at org.springframework.beans.factory.support.DefaultSingletonBeanRegistry.getSingleton(DefaultSingletonBeanRegistry.java:234) ~[spring-beans-6.0.13.jar:6.0.13]
	at org.springframework.beans.factory.support.AbstractBeanFactory.doGetBean(AbstractBeanFactory.java:323) ~[spring-beans-6.0.13.jar:6.0.13]
	at org.springframework.beans.factory.support.AbstractBeanFactory.getBean(AbstractBeanFactory.java:199) ~[spring-beans-6.0.13.jar:6.0.13]
	at org.springframework.beans.factory.config.DependencyDescriptor.resolveCandidate(DependencyDescriptor.java:254) ~[spring-beans-6.0.13.jar:6.0.13]
	at org.springframework.beans.factory.support.DefaultListableBeanFactory.doResolveDependency(DefaultListableBeanFactory.java:1417) ~[spring-beans-6.0.13.jar:6.0.13]
	at org.springframework.beans.factory.support.DefaultListableBeanFactory.resolveDependency(DefaultListableBeanFactory.java:1337) ~[spring-beans-6.0.13.jar:6.0.13]
	at org.springframework.beans.factory.support.ConstructorResolver.resolveAutowiredArgument(ConstructorResolver.java:910) ~[spring-beans-6.0.13.jar:6.0.13]
	at org.springframework.beans.factory.support.ConstructorResolver.createArgumentArray(ConstructorResolver.java:788) ~[spring-beans-6.0.13.jar:6.0.13]
	... 19 common frames omitted
Caused by: org.springframework.beans.factory.BeanCreationException: Error creating bean with name 'boardRepository' defined in com.diger.notonlysqlboard.core.board.repository.BoardRepository defined in @EnableMongoRepositories declared on NotOnlySqlBoardApplication: Unable to make field private final java.lang.String java.io.File.path accessible: module java.base does not "opens java.io" to unnamed module @3ce1e309
	at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.initializeBean(AbstractAutowireCapableBeanFactory.java:1770) ~[spring-beans-6.0.13.jar:6.0.13]
	at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.doCreateBean(AbstractAutowireCapableBeanFactory.java:598) ~[spring-beans-6.0.13.jar:6.0.13]
	at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.createBean(AbstractAutowireCapableBeanFactory.java:520) ~[spring-beans-6.0.13.jar:6.0.13]
	at org.springframework.beans.factory.support.AbstractBeanFactory.lambda$doGetBean$0(AbstractBeanFactory.java:325) ~[spring-beans-6.0.13.jar:6.0.13]
	at org.springframework.beans.factory.support.DefaultSingletonBeanRegistry.getSingleton(DefaultSingletonBeanRegistry.java:234) ~[spring-beans-6.0.13.jar:6.0.13]
	at org.springframework.beans.factory.support.AbstractBeanFactory.doGetBean(AbstractBeanFactory.java:323) ~[spring-beans-6.0.13.jar:6.0.13]
	at org.springframework.beans.factory.support.AbstractBeanFactory.getBean(AbstractBeanFactory.java:199) ~[spring-beans-6.0.13.jar:6.0.13]
	at org.springframework.beans.factory.config.DependencyDescriptor.resolveCandidate(DependencyDescriptor.java:254) ~[spring-beans-6.0.13.jar:6.0.13]
	at org.springframework.beans.factory.support.DefaultListableBeanFactory.doResolveDependency(DefaultListableBeanFactory.java:1417) ~[spring-beans-6.0.13.jar:6.0.13]
	at org.springframework.beans.factory.support.DefaultListableBeanFactory.resolveDependency(DefaultListableBeanFactory.java:1337) ~[spring-beans-6.0.13.jar:6.0.13]
	at org.springframework.beans.factory.support.ConstructorResolver.resolveAutowiredArgument(ConstructorResolver.java:910) ~[spring-beans-6.0.13.jar:6.0.13]
	at org.springframework.beans.factory.support.ConstructorResolver.createArgumentArray(ConstructorResolver.java:788) ~[spring-beans-6.0.13.jar:6.0.13]
	... 33 common frames omitted
Caused by: java.lang.reflect.InaccessibleObjectException: Unable to make field private final java.lang.String java.io.File.path accessible: module java.base does not "opens java.io" to unnamed module @3ce1e309
	at java.base/java.lang.reflect.AccessibleObject.checkCanSetAccessible(AccessibleObject.java:354) ~[na:na]
	at java.base/java.lang.reflect.AccessibleObject.checkCanSetAccessible(AccessibleObject.java:297) ~[na:na]
	at java.base/java.lang.reflect.Field.checkCanSetAccessible(Field.java:178) ~[na:na]
	at java.base/java.lang.reflect.Field.setAccessible(Field.java:172) ~[na:na]
	at org.springframework.util.ReflectionUtils.makeAccessible(ReflectionUtils.java:779) ~[spring-core-6.0.13.jar:6.0.13]
	at org.springframework.data.mapping.context.AbstractMappingContext$PersistentPropertyCreator.doWith(AbstractMappingContext.java:536) ~[spring-data-commons-3.1.5.jar:3.1.5]
	at org.springframework.util.ReflectionUtils.doWithFields(ReflectionUtils.java:703) ~[spring-core-6.0.13.jar:6.0.13]
	at org.springframework.data.mapping.context.AbstractMappingContext.doAddPersistentEntity(AbstractMappingContext.java:422) ~[spring-data-commons-3.1.5.jar:3.1.5]
	at org.springframework.data.mapping.context.AbstractMappingContext.addPersistentEntity(AbstractMappingContext.java:379) ~[spring-data-commons-3.1.5.jar:3.1.5]
	at org.springframework.data.mapping.context.AbstractMappingContext$PersistentPropertyCreator.lambda$createAndRegisterProperty$3(AbstractMappingContext.java:591) ~[spring-data-commons-3.1.5.jar:3.1.5]
	at java.base/java.lang.Iterable.forEach(Iterable.java:75) ~[na:na]
	at org.springframework.data.mapping.context.AbstractMappingContext$PersistentPropertyCreator.createAndRegisterProperty(AbstractMappingContext.java:588) ~[spring-data-commons-3.1.5.jar:3.1.5]
	at java.base/java.util.stream.ForEachOps$ForEachOp$OfRef.accept(ForEachOps.java:183) ~[na:na]
	at java.base/java.util.stream.ReferencePipeline$2$1.accept(ReferencePipeline.java:179) ~[na:na]
	at java.base/java.util.stream.ReferencePipeline$3$1.accept(ReferencePipeline.java:197) ~[na:na]
	at java.base/java.util.stream.ReferencePipeline$2$1.accept(ReferencePipeline.java:179) ~[na:na]
	at java.base/java.util.HashMap$ValueSpliterator.forEachRemaining(HashMap.java:1779) ~[na:na]
	at java.base/java.util.stream.AbstractPipeline.copyInto(AbstractPipeline.java:509) ~[na:na]
	at java.base/java.util.stream.AbstractPipeline.wrapAndCopyInto(AbstractPipeline.java:499) ~[na:na]
	at java.base/java.util.stream.ForEachOps$ForEachOp.evaluateSequential(ForEachOps.java:150) ~[na:na]
	at java.base/java.util.stream.ForEachOps$ForEachOp$OfRef.evaluateSequential(ForEachOps.java:173) ~[na:na]
	at java.base/java.util.stream.AbstractPipeline.evaluate(AbstractPipeline.java:234) ~[na:na]
	at java.base/java.util.stream.ReferencePipeline.forEach(ReferencePipeline.java:596) ~[na:na]
	at org.springframework.data.mapping.context.AbstractMappingContext$PersistentPropertyCreator.addPropertiesForRemainingDescriptors(AbstractMappingContext.java:559) ~[spring-data-commons-3.1.5.jar:3.1.5]
	at org.springframework.data.mapping.context.AbstractMappingContext.doAddPersistentEntity(AbstractMappingContext.java:423) ~[spring-data-commons-3.1.5.jar:3.1.5]
	at org.springframework.data.mapping.context.AbstractMappingContext.addPersistentEntity(AbstractMappingContext.java:379) ~[spring-data-commons-3.1.5.jar:3.1.5]
	at org.springframework.data.mapping.context.AbstractMappingContext$PersistentPropertyCreator.lambda$createAndRegisterProperty$3(AbstractMappingContext.java:591) ~[spring-data-commons-3.1.5.jar:3.1.5]
	at java.base/java.lang.Iterable.forEach(Iterable.java:75) ~[na:na]
	at org.springframework.data.mapping.context.AbstractMappingContext$PersistentPropertyCreator.createAndRegisterProperty(AbstractMappingContext.java:588) ~[spring-data-commons-3.1.5.jar:3.1.5]
	at java.base/java.util.stream.ForEachOps$ForEachOp$OfRef.accept(ForEachOps.java:183) ~[na:na]
	at java.base/java.util.stream.ReferencePipeline$2$1.accept(ReferencePipeline.java:179) ~[na:na]
	at java.base/java.util.stream.ReferencePipeline$3$1.accept(ReferencePipeline.java:197) ~[na:na]
	at java.base/java.util.stream.ReferencePipeline$2$1.accept(ReferencePipeline.java:179) ~[na:na]
	at java.base/java.util.HashMap$ValueSpliterator.forEachRemaining(HashMap.java:1779) ~[na:na]
	at java.base/java.util.stream.AbstractPipeline.copyInto(AbstractPipeline.java:509) ~[na:na]
	at java.base/java.util.stream.AbstractPipeline.wrapAndCopyInto(AbstractPipeline.java:499) ~[na:na]
	at java.base/java.util.stream.ForEachOps$ForEachOp.evaluateSequential(ForEachOps.java:150) ~[na:na]
	at java.base/java.util.stream.ForEachOps$ForEachOp$OfRef.evaluateSequential(ForEachOps.java:173) ~[na:na]
	at java.base/java.util.stream.AbstractPipeline.evaluate(AbstractPipeline.java:234) ~[na:na]
	at java.base/java.util.stream.ReferencePipeline.forEach(ReferencePipeline.java:596) ~[na:na]
	at org.springframework.data.mapping.context.AbstractMappingContext$PersistentPropertyCreator.addPropertiesForRemainingDescriptors(AbstractMappingContext.java:559) ~[spring-data-commons-3.1.5.jar:3.1.5]
	at org.springframework.data.mapping.context.AbstractMappingContext.doAddPersistentEntity(AbstractMappingContext.java:423) ~[spring-data-commons-3.1.5.jar:3.1.5]
	at org.springframework.data.mapping.context.AbstractMappingContext.addPersistentEntity(AbstractMappingContext.java:379) ~[spring-data-commons-3.1.5.jar:3.1.5]
	at org.springframework.data.mapping.context.AbstractMappingContext$PersistentPropertyCreator.lambda$createAndRegisterProperty$3(AbstractMappingContext.java:591) ~[spring-data-commons-3.1.5.jar:3.1.5]
	at java.base/java.lang.Iterable.forEach(Iterable.java:75) ~[na:na]
	at org.springframework.data.mapping.context.AbstractMappingContext$PersistentPropertyCreator.createAndRegisterProperty(AbstractMappingContext.java:588) ~[spring-data-commons-3.1.5.jar:3.1.5]
	at org.springframework.data.mapping.context.AbstractMappingContext$PersistentPropertyCreator.doWith(AbstractMappingContext.java:542) ~[spring-data-commons-3.1.5.jar:3.1.5]
	at org.springframework.util.ReflectionUtils.doWithFields(ReflectionUtils.java:703) ~[spring-core-6.0.13.jar:6.0.13]
	at org.springframework.data.mapping.context.AbstractMappingContext.doAddPersistentEntity(AbstractMappingContext.java:422) ~[spring-data-commons-3.1.5.jar:3.1.5]
	at org.springframework.data.mapping.context.AbstractMappingContext.addPersistentEntity(AbstractMappingContext.java:379) ~[spring-data-commons-3.1.5.jar:3.1.5]
	at org.springframework.data.mapping.context.AbstractMappingContext$PersistentPropertyCreator.lambda$createAndRegisterProperty$3(AbstractMappingContext.java:591) ~[spring-data-commons-3.1.5.jar:3.1.5]
	at java.base/java.lang.Iterable.forEach(Iterable.java:75) ~[na:na]
	at org.springframework.data.mapping.context.AbstractMappingContext$PersistentPropertyCreator.createAndRegisterProperty(AbstractMappingContext.java:588) ~[spring-data-commons-3.1.5.jar:3.1.5]
	at org.springframework.data.mapping.context.AbstractMappingContext$PersistentPropertyCreator.doWith(AbstractMappingContext.java:542) ~[spring-data-commons-3.1.5.jar:3.1.5]
	at org.springframework.util.ReflectionUtils.doWithFields(ReflectionUtils.java:703) ~[spring-core-6.0.13.jar:6.0.13]
	at org.springframework.data.mapping.context.AbstractMappingContext.doAddPersistentEntity(AbstractMappingContext.java:422) ~[spring-data-commons-3.1.5.jar:3.1.5]
	at org.springframework.data.mapping.context.AbstractMappingContext.addPersistentEntity(AbstractMappingContext.java:379) ~[spring-data-commons-3.1.5.jar:3.1.5]
	at org.springframework.data.mapping.context.AbstractMappingContext.getPersistentEntity(AbstractMappingContext.java:280) ~[spring-data-commons-3.1.5.jar:3.1.5]
	at org.springframework.data.mapping.context.AbstractMappingContext.getPersistentEntity(AbstractMappingContext.java:206) ~[spring-data-commons-3.1.5.jar:3.1.5]
	at org.springframework.data.mapping.context.AbstractMappingContext.getPersistentEntity(AbstractMappingContext.java:92) ~[spring-data-commons-3.1.5.jar:3.1.5]
	at org.springframework.data.mapping.context.MappingContext.getRequiredPersistentEntity(MappingContext.java:74) ~[spring-data-commons-3.1.5.jar:3.1.5]
	at org.springframework.data.mongodb.repository.support.MongoRepositoryFactory.getEntityInformation(MongoRepositoryFactory.java:146) ~[spring-data-mongodb-4.1.5.jar:4.1.5]
	at org.springframework.data.mongodb.repository.support.MongoRepositoryFactory.getTargetRepository(MongoRepositoryFactory.java:128) ~[spring-data-mongodb-4.1.5.jar:4.1.5]
	at org.springframework.data.repository.core.support.RepositoryFactorySupport.getRepository(RepositoryFactorySupport.java:317) ~[spring-data-commons-3.1.5.jar:3.1.5]
	at org.springframework.data.repository.core.support.RepositoryFactoryBeanSupport.lambda$afterPropertiesSet$5(RepositoryFactoryBeanSupport.java:279) ~[spring-data-commons-3.1.5.jar:3.1.5]
	at org.springframework.data.util.Lazy.getNullable(Lazy.java:245) ~[spring-data-commons-3.1.5.jar:3.1.5]
	at org.springframework.data.util.Lazy.get(Lazy.java:114) ~[spring-data-commons-3.1.5.jar:3.1.5]
	at org.springframework.data.repository.core.support.RepositoryFactoryBeanSupport.afterPropertiesSet(RepositoryFactoryBeanSupport.java:285) ~[spring-data-commons-3.1.5.jar:3.1.5]
	at org.springframework.data.mongodb.repository.support.MongoRepositoryFactoryBean.afterPropertiesSet(MongoRepositoryFactoryBean.java:101) ~[spring-data-mongodb-4.1.5.jar:4.1.5]
	at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.invokeInitMethods(AbstractAutowireCapableBeanFactory.java:1817) ~[spring-beans-6.0.13.jar:6.0.13]
	at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.initializeBean(AbstractAutowireCapableBeanFactory.java:1766) ~[spring-beans-6.0.13.jar:6.0.13]
	... 44 common frames omitted
```

빈 주입이 안되어있다는데 도대체가 어지럽다. 아무리 살펴봐도 생성자 주입을 사용하고 있고 빈 주입이 안될 이유가 없었기 때문이다.

## 해결

3~4시간 동안 구글링해서 얻은 해결책들을 모두 대입해봤지만 실패했다.

근데 갑자기 번뜩임과 함께 아래와 같이 MultiparFile Type을 String으로 변경하니 해결됐다.

```java
@Getter
@AllArgsConstructor
@Document(collection = "comment")
public class Comment extends BaseDocument {
    private final String value;
    private final String fileLink;
    private final User writer;
}
```

무슨 상관인진 모르겠지만 MultipartFile 타입을 직접 사용할 땐 주의해야한다는 것을 배웠다.

이 해결방안에 대한 원리는 더 알아보기로 하자
