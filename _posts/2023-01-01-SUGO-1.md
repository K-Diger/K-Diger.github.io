---

title: 비동기 애노테이션 사용 시 주의점 (@Async, TaskExecutor)
date: 2024-09-21
categories: [SUGO]
tags: [SUGO]
layout: post
toc: true
math: true
mermaid: true
---

# 문제 상황

클라이언트에 푸쉬 알림을 전송하기 위해 FCM 서버에 요청하는 로직을 추가했더니 아래와 같이 658ms가 소요되는 현상이 발생했다.

![](https://github.com/K-Diger/K-Diger.github.io/blob/main/images/async/%EC%84%B1%EB%8A%A5%20%EA%B0%9C%EC%84%A0%20%EC%9D%B4%EC%A0%84%20%EB%91%90%EB%B2%88%EC%A7%B8%20%EC%9A%94%EC%B2%AD.jpg?raw=true)

이 메서드를 추가하기 전에는 100ms도 넘지 않았던 응답시간이 저렇게 급격하게 늘어나니 클라이언트단에서도 응답을 기다리느라 사용자 경험을 해치는 상황이 발생하게 되었다.

---

# 어떻게 속도를 빠르게 할 수 있을까?

외부 서버를 호출하는 로직을 비동기로 처리하면 어떨까? 굳이 외부 서버로 던진 요청이 완료된 것을 확인받은 후에 다른 비즈니스 로직을 처리해야할까? 를 고민하다가 직접 비동기 메서드를 적용해보기로 했다.

---

# ThreadPoolTaskExecutor와 @Async

[Asynchronous calls in spring](https://www.linkedin.com/pulse/asynchronous-calls-spring-boot-using-async-annotation-omar-ismail/)

Spring에서 비동기 작업을 처리하려면 다음 두 가지를 관리해야한다.

- @Asnyc 애노테이션으로 비동기 메서드를 지정한다.
- 비동기 메서드의 처리를 담당하는 스레드 풀을 관리해줘야한다.

스레드 풀을 관리하기 위한 Bean을 등록하기 위해서는 아래의 인터페이스의 구현체를 셋팅해줘야 한다.

```java
public interface AsyncConfigurer {

	@Nullable
	default Executor getAsyncExecutor() {
		return null;
	}

	@Nullable
	default AsyncUncaughtExceptionHandler getAsyncUncaughtExceptionHandler() {
		return null;
	}

}
```

하지만 꼭 위 인터페이스를 구현체로 등록하지 않아도 되긴하다. 그냥 ThreadPoolTaskExecutor를 반환하는 메서드를 Bean으로 등록해주면 되는 것일 뿐이다.

---

# @Async 애노테이션이 어떻게 동작하는건가?

- @Async 애노테이션이 붙은 메서드가 실행될 때, 프록시가 해당 호출을 가로채고 Task Executor에 전달한다.

- Task Executor는 새로운 스레드를 생성하고 해당 스레드에서 호출된 로직을 실행한다. 그리고 이 때 이 메서드의 리턴을 기다리지 않고 다른 작업을 계속 수행한다.

- 이전에 호출시킨 메서드가 리턴되었다면 Task Executor에 결과를 반환한다.

이 때 새로운 스레드를 생성한다는 점이 성능 저하의 문제가 될 수 있다.

```java
public class ThreadPoolTaskExecutor extends ExecutorConfigurationSupport
		implements AsyncListenableTaskExecutor, SchedulingTaskExecutor {

    private final Object poolSizeMonitor = new Object();

    private int corePoolSize = 1;

    private int maxPoolSize = Integer.MAX_VALUE;

    private int keepAliveSeconds = 60;

    private int queueCapacity = Integer.MAX_VALUE;

    private boolean allowCoreThreadTimeOut = false;

    private boolean prestartAllCoreThreads = false;

    ...
}
```

위 코드가 Spring이 관리하는 스레드 풀인 ThreadPoolTaskExecutor이다.

해당 클래스의 프로퍼티를 보면,

- `corePoolSize` : 초기 Thread Size는 1

- `maxPoolSize` : 최대 Thread Pool Size는 약 21억

- `queueCapacity` : Queue 용량은 약 21억

으로 설정되어있다. 위 설정을 그대로 사용해도 좋지만 애플리케이션 환경과 서버의 하드웨어 스펙에 따라 조정해 줄 필요는 있을 것 같다.

---

# 다시 돌아와서, 뭐가 문제였나?

- @Asnyc 애노테이션을 활용하여 비동기 메서드를 지정하지 않았던 점

- ThreadPoolTaskExecutor와 같이 스레드 풀을 관리해주는 설정을 Bean으로 등록하지 않은 점

이 두 가지를 적용하지 않았기 때문에 응답에 지연이 생긴 것으로 볼 수 있다.

---

# 해결 방안 및 성능 개선 결과

```java
@Configuration
@EnableAsync
public class AsyncConfig implements AsyncConfigurer {

    @Override
    public Executor getAsyncExecutor() {
        ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor();
        executor.setCorePoolSize(3);
        executor.setMaxPoolSize(30);
        executor.setQueueCapacity(90);
        executor.setThreadNamePrefix("SUGO-DIGER-ASYNC-");
        executor.initialize();

        return executor;
    }
}
```

위와 같은 설정을 Bean으로 등록해주고 비동기로 처리할 메서드에 `@Async`애노테이션을 붙여주면 끝이다! 이렇게 간단하게 해결할 수 있음에도 성능 차이는 엄청났다.

![](https://github.com/K-Diger/K-Diger.github.io/blob/main/images/async/%EC%84%B1%EB%8A%A5%20%EA%B0%9C%EC%84%A0%20%ED%9B%84%20%EB%91%90%EB%B2%88%EC%A7%B8%20%EC%9A%94%EC%B2%AD.jpg?raw=true)

- 응답 시간 `658ms` -> `122ms` 응답 시간이 81% 빨라졌다.

---

# @Async 애노테이션을 사용할 때 기억해둬야할 것

## 1. 쓰레드와 큐의 사이즈를 지정할 때 신중해야한다.

여기서 다루는 쓰레드는 `논리적인 쓰레드를`말한다.

쓰레드를 생성하는 비용은 꽤나 엄청난 작업이다. 그렇기 때문에 아래 속성을 정의할 때 주의해야한다.

주로 실제 부하 테스트를 통해 측정해보면 정확하게 지정할 수 있겠으나 나는 그 부분까지는 수행하진 못했다.

아래 속성들은 비동기 처리를 위한 추가적으로 생성할 리소스를 의미한다.

`기본값`은 다음과 같이 설정되어있다.

- `corePoolSize` : 최소 비동기 쓰레드 수 : 1

- `maxPoolSize` : 최대 비동기 쓰레드 수 : 약 21억

- `queueCapacity` : queueCapacity는 작업 큐의 사이즈 : 약 21억

- 새로운 쓰레드를 생성하는 것은 리소스가 크기 때문에 일반적으로는 쓰레드 풀을 사용한다.
    - `초기 쓰레드 갯수`를 **10개**로 지정했고, `최대 쓰레드 갯수`를 **100개**로 했을 때 **50개의 요청이 동시에** 들어오면 쓰레드 갯수는 몇 개가 될 것인가?
        - 우선 10개로 처리를 한 후 쓰레드가 잡지 못한 요청은 우선적으로 Queue에 쌓인다.

- `큐 사이즈가 꽉차면` 어떻게 될까?
    - 큐 사이즈가 꽉찬다면 큐에 있는 작업을 처리할 쓰레드를 추가로 늘린다.
        - 이 때 쓰레드의 갯수마저도 이전에 지정한 최대치에 도달한다면 그 이후의 요청은 예외가 발생하게 될 것이다.

- 방금 말한 상황에서 발생하는 `예외는 어떻게 처리`해야하는가?
    - 재시도를 통해 서버에 들어온 요청을 책임지고 마쳐야 한다.

- 재시도를 했음에도 `예외가 계속 발생`하면?
    - 그런 상황에서는 요청 값이 잘못되거나 메서드의 로직 자체가 잘못되어있을 가능성이 있을 것이기 때문에 그때는 `비동기 메서드에 대한 검증`을 다시 해봐야한다.

- 큐의 작업을 처리하기 위해 `쓰레드를 늘렸을 때 그 쓰레드들은 계속 유지`되는가?
    - 아니다. `사용되지 않는 쓰레드들은 일정 시간이 지나면 유휴 상태`에 들어가 반납되는 것으로 공식문서에 명시되어있다.

---

## 2. @Async가 달린 비동기 메서드는 반환 값이 없어야한다.

당연하게도 비동기로 처리할 로직에는 반환 값이 없어야한다.

반환 값이 있다는 것은 그 모든 리소스를 반환할 때 까지 다음 메서드를 수행하지 못하기 때문에 당연히 비동기 작업에는 반환값이 없어야 하는 것이다.

만약 언젠가 처리가 끝나고나서 그 반환값을 사용하고자 한다면 `CompetableFuture`타입으로 반환해야한다.

---

## 3. @Async가 달린 비동기 메서드는 private하면 안된다.

[참고자료 - 왜 @Async 애노테이션이 달린 메서드는 private하면 안되나?](https://dzone.com/articles/effective-advice-on-spring-async-part-1)

@Async 애노테이션은 AOP기반으로 동작한다. 따라서 해당 애노테이션이 붙은 메서드를 감싸는 프록시 객체를 생성하는 것인데

이 프록시 객체는 실제 비동기 작업을 처리할 때 사용된다. 프록시 객체로 동작한다는 것이 그 이유이다.

---

# 비동기로 요청한 내용이 실패했다면 어떻게 되는건가?

지금 로직은 비즈니스 로직을 처리하고 비동기로 외부 서버를 호출한 후 곧바로 리턴하여 클라이언트에게 결과를 전송한다.

그런데 만약 비동기로 외부 서버호출한 내용이 실패한다면 어떻게 되는걸까?

- 재시도를 통해 책임지고 끝낼 수 있도록 한다.
- 예외 처리를 한다.
    - 하지만 이 방법은 아직 잘 모르겠다. 예외처리를 하기 위해선 결국 외부 서버로부터 응답을 받아야하는데, 이 응답을 기다리는 것이 동기로 통신하는 것과 다를 바 없기 때문이다.
