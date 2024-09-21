---

title: 자문자답 - 운영체제 기본
date: 2024-09-20
categories: [OperatingSystem]
tags: [OperatingSystem]
layout: post
toc: true
math: true
mermaid: true

---

# 1. 프로세스와 쓰레드의 차이점

## 프로세스

프로세스는 디스크에 있는 프로그램이 실제 메모리에 할당되어 실행 중인 작업을 나타낸다.

운영체제는 프로세스가 생성되면 PCB를 통해 해당 프로세스의 정보를 관리하는데 PCB가 담고있는 정보는 아래와 같다.

- PID
- Process의 상태
- PC 레지스터
- 스택 포인터
- 등...

이 정보들이 필요한 이유는 여러 프로세스를 번갈아가면서 처리하는데, 이 때 각 프로세스에 대한 처리를 기록해야 정상적인 처리 운용이 가능하기 때문이다.

프로세스는 문맥교환이 일어날 때 현재 프로세스의 정보를 저장하고 다른 프로세스의 상태를 불러와야하므로 상대적으로 시간이 오래 소요된다.

또한 프로세스의 구조는 아래와 같다.

- Code : 컴파일된 소스 코드 저장
- Data: 전역 변수/초기화된 데이터 저장
- Stack: 임시 데이터(함수 호출, 로컬 변수 등)저장
- Heap: 코드에서 동적으로 생성되는 데이터 저장

## 쓰레드

쓰레드는 프로세스를 처리하기 위한 작업의 단위를 이야기한다. 따라서 프로세스에 여러 개의 쓰레드는 프로세스가 할당받은 자원을 모두 공유한다.

---

# 2. 멀티 프로세스와 멀티 스레드의 공통/차이점

## 공통점

- 하나의 프로그램에서 여러 작업을 동시에 수행할 수 있다.
- 운영체제의 스케줄링을 통해 작업을 관리한다.
- 작업 간의 통신이 필요하다.

## 차이점

- `프로세스`는 `메모리를 독립적`으로 사용하지만 / `쓰레드`는 `메모리를 공유`하여 사용한다.
  - 이로인해 `프로세스`는 `문맥 전환 비용이 크지만` / `쓰레드는 문맥 교환 비용이 작다`.
- `프로세스`는 스레드보다 `많은 메모리 공간`과 `CPU시간`을 차지한다.
- `프로세스`는 다른 프로세스에서 문제가 생겨도 `오류가 전파되지 않기 때문에` 안정성이 높지만 / 쓰레드는 공유된 영역을 사용하기 때문에 다른 쓰레드에 `문제가 발생`하면 `전파`될 가능성이 커 안정성이 비교적 낮다.

---

# 3. 언제 멀티 프로세스, 멀티 스레드를 사용하면 될까? (멀티 프로세스 예시가 적절한게 안떠오름)

## 멀티 프로세스 사례

### 우리가 사용하는 컴퓨팅 환경

인텔리제이를 키고, 데이터그립을 키고, 웹 브라우저를 키고 여러 애플리케이션을 자유롭게 사용한다.

## 멀티 쓰레드 사례

### 웹 브라우저

하나의 브라우저에 `A,B 탭`이 있을 때 `A탭이 사용 불능` 되어도 `B탭은 사용`할 수 있다.

### 게임

하나의 게임 애플리케이션에서 `그래픽 렌더링`, `물리 엔진 컴퓨팅` 등 다양한 작업을 동시에 처리한다.

---

# 4. 프로세스 메모리 영역

- Code : 컴파일된 소스 코드 저장
- Data: 전역 변수/초기화된 데이터 저장
- Stack: 임시 데이터(함수 호출, 로컬 변수 등)저장
- Heap: 코드에서 동적으로 생성되는 데이터 저장

---

# 5. Call By Value(자체 값의 복사), Reference(값의 주소 참조)

[참고 블로그](https://inpa.tistory.com/entry/JAVA-☕-자바는-Call-by-reference-개념이-없다-❓)

## Primitive Type

원시타입의 파라미터는 Value로 사용된다. Primitive Type의 값은 메모리 상에 직접 저장되기 때문에, 파라미터로 전달된 값을 변경하면 원본 값도 변경된다.

### 원시 타입 도식화

```text
       +-----------------------------------+
       |          Stack Memory             |
       |                                   |
       |  +--------------+                 |
       |  |    Variable  |                 |
       |  |  (Primitive Type)              |
       |  +--------------+                 |
       |                                   |
       +-----------------------------------+
```

위 그림과 같이 기본 타입을 직접 가지고 있게 된다.

## Reference Type

참조타입(객체, 배열 등)의 파라미터는 Reference를 가지고 있다. Reference Type의 값은 메모리 상에 객체로 저장되기 때문에, 파라미터로 전달된 Reference를 변경하면 원본 객체의 참조가 변경된다.

파라미터가 `Reference Type`인 경우 `해당 참조`가 `힙 메모리에 있는 객체`를 가리키고 이 `객체는 힙 메모리`에 저장된다.

`객체의 데이터와 메서드는 힙 메모리에 저장되며`, `참조는 스택 메모리에 저장된다`.

따라서 파라미터가 `Reference Type`인 경우 `실제 객체는 힙 메모리`에 저장되고, `스택 메모리의 참조가 해당 객체`를 가리키게 된다.

### 참조 타입 도식화

```text
 +-----------------------------------+
       |          Stack Memory             |
       |                                   |
       |  +--------------+                 |
       |  |   Reference  |                 |
       |  |    Variable  |                 |
       |  +--------------+                 |
       |       (참조 변수)                   |
       |                                   |
       +--------------|-------------------+
                      |
                      |
                      v
       +--------------|-------------------+
       |          Heap Memory              |
       |                                   |
       |  +--------------+                 |
       |  |    Object    |                 |
       |  |   (데이터 및 메서드) |             |
       |  +--------------+                 |
       |        (객체)                      |
       |                                   |
       +-----------------------------------+
```

## 예시 코드

```java
public class main {
    public static void main(String[] args) {

        int primitiveVariable = 1;
        int[] referenceVariable = {1};

        // 변수 자체를 보냄 (call by value)
        increasePrimitiveValue(primitiveVariable);

        // 1 : 값 변화가 없음
        System.out.println(var);

        // ------------------------------------ //

        // 배열 자체를 보냄 (간접적 call by reference)
        increaseReferenceValue(referenceVariable);

        // 101 : 값이 변화함
        System.out.println(arr[0]);
    }

    private static void increasePrimitiveValue(int primitiveVariable) {
        primitiveVariable += 100;
    }

    private static void increaseReferenceValue(int[] referenceVariable) {
        referenceVariable[0] += 100;
    }
}
```

## 예시 코드 메모리 도식화

### increasePrimitiveValue()

함수 실행 전 메모리
![](https://github.com/K-Diger/K-Diger.github.io/assets/60564431/d78c0e67-e6c4-44c0-9af0-a2c10c3789d4)

함수 실행 후 메모리
![](https://github.com/K-Diger/K-Diger.github.io/assets/60564431/de915f18-4d3e-4976-a43e-a2a78c335d83)

### increaseReferenceValue

함수 실행 전 메모리
![](https://github.com/K-Diger/K-Diger.github.io/assets/60564431/dd3296cd-92c0-4d4b-a5d8-b0e18baed1ca)

함수 실행 후 메모리
![](https://github.com/K-Diger/K-Diger.github.io/assets/60564431/d9f0d2e2-ab10-4bba-8c9f-6a818753eb53)

### 결론

각 메서드는 고유한 스택 프레임을 가지기 때문에 참조값을 파라미터로 넘기는 것이 아니면 파라미터 값을 수정해도 다른 메서드에 아무런 영향이 없다.

그리고 이 방식을 Call By Value라고 한다.

## 주소값을 넘기는데 왜 Call by Value라고 하는걸까?

C언어에서는 Call by Reference방식을 사용하는데 그 대표적인 예시가 `*`포인터와, `&`주소 연산이다.

사용자가 직접 주소값에 접근하고 다룰 수 있게 하는 C언어와 달리, 간접적으로 타입에 의존하여 주소값을 넘기는 Java는 Call By Reference라고 하질 않는 것이다.

## 그러면 왜 Java는 Call By Value를 채택했는가?

주소값을 사용자가 직접 다루게 된다면 예기치 못한 코드에서 메모리에 접근하여 값을 변경하고 이를 사용하는 입장에서는 원치 않은 결과를 받을 수 있다.

Call By Value는 `파라미터`로 전달되는 값이 실제 값이 아닌 `복사된 값`이 전달되는 것으로 `오버헤드`가 발생하는 `단점`이 있지만 `코드의 안정성`을 확보할 수 있는 `장점`이 있다.

이러한 이유로 Java는 Call By Value를 사용한다.

---

# 6. Sync-Async, Blocking - Non-Blocking

- 예를들어, IO(입출력)를 처리할 때 Sync와 Async의 차이는 요청한 순서가 지켜지는가 아닌가에 있고

- Blocking과 Non-blocking은 그 요청에 대해 받은 쪽에서 처리가 끝나기 전에 리턴해주는가 아닌가에 있다.

## Blocking

A 함수가 B 함수를 호출 할 때, B 함수가 자신의 작업이 종료되기 전까지 A 함수에게 제어권을 돌려주지 않는 것

## Non-blocking

A 함수가 B 함수를 호출 할 때, B 함수가 제어권을 바로 A 함수에게 넘겨주면서, A 함수가 다른 일을 할 수 있도록 하는 것.

## Sync

A 함수가 B 함수를 호출 할 때, B 함수의 결과를 A 함수가 처리하는 것.

## Async

A 함수가 B 함수를 호출 할 때, B 함수의 결과를 B 함수가 처리하는 것. (callback)

---

## Sync-Blocking

```java
public class SyncNonBlocking {

    public static void main(String[] args) throws InterruptedException {
        // waitOneSecond 함수 호출 후 바로 다음 코드가 실행된다.
        System.out.println("Sync-nonBlocking 시작");
        waitOneSecond();
        System.out.println("Sync-nonBlocking 종료");
    }

    private static void waitOneSecond() throws InterruptedException {
        Thread.sleep(1000);
    }
}
```

## Sync-NonBlocking

```java
public class SyncNonBlocking {

    public static void main(String[] args) throws InterruptedException {
        // waitOneSecond 함수 호출 후 바로 다음 코드가 실행된다.
        System.out.println("Sync-nonBlocking 시작");
        waitOneSecond();
        System.out.println("Sync-nonBlocking 종료");
    }

    private static void waitOneSecond() throws InterruptedException {
        Thread.sleep(1000);
    }
}
```

## Async-Blocking

```java
public class AsyncBlocking {

    public static void main(String[] args) throws InterruptedException {
        // 1초 동안 대기하는 함수
        private static void waitOneSecond() throws InterruptedException {
            Thread.sleep(1000);
        }

        // Async-Blocking 코드
        // 함수 호출 후 다음 코드가 실행되기 전까지 대기한다.
        // 함수의 결과는 나중에 콜백 함수에서 처리한다.
        System.out.println("Async-Blocking 시작");
        ExecutorService executorService = Executors.newSingleThreadExecutor();
        CompletionStage<Void> future = executorService.submit(() -> {
            try {
                waitOneSecond();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            return null;
        });
        future.get();
        System.out.println("Async-Blocking 종료");
        executorService.shutdown();
    }
}

```

## Async-NonBlocking

```java
public class AsyncNonBlocking {

    public static void main(String[] args) throws InterruptedException {
        // 1초 동안 대기하는 함수
        private static void waitOneSecond() throws InterruptedException {
            Thread.sleep(1000);
        }

        // Async-nonBlocking 코드
        // 함수 호출 후 바로 다음 코드가 실행된다.
        // 함수의 결과는 나중에 콜백 함수에서 처리한다.
        System.out.println("Async-nonBlocking 시작");
        ExecutorService executorService = Executors.newSingleThreadExecutor();
        executorService.submit(() -> {
            try {
                waitOneSecond();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        });
        System.out.println("Async-nonBlocking 종료");
        executorService.shutdown();
    }
}
```

---

# 7. CPU 스케쥴링 기법

## 선점형 알고리즘

다른 프로세스에 CPU의 처리 순서를 `양보`한다. 이 과정에서 문맥 교환이 발생하기 때문에 처리 비용이 증가하게 된다.

### Round Robin

각 프로세스마다 동일한 시간을 할당하여 처리하도록 한다. 실제 애플리케이션 로드밸런서에도 자주 사용되는 알고리즘이다. 공평하게 자원을 분배할 수 있는 `장점`이 있지만 근소한 차이로 작업을 처리하지 못했을 경우 해당 프로세스의 처리가 지연된다는 `단점`이 존재한다.

### SRT (Shortest Remaining Time)

프로세스의 처리 시간이 가장 짧다고 계산된 순으로 처리하도록한다. `처리량`이 높아질 수 있다는 `장점`이 있지만 `우선적으로 처리해야할 프로세스에 대해 지연`될 수 있다는 `단점`이 있다.

작업 1, 작업 2, 작업 3이 순서대로 들어오는 환경에서

- 작업 1의 작업 시간 : 10초
- 작업 2의 작업 시간 : 5초
- 작업 3의 작업 시간 : 3초

`작업 1`을 먼저 처리하는 중 `5초 후 작업 2`를 처리하도록 문맥 교환하고, `작업 2 처리를 시작한 3초 후` `작업 3`을 처리하는 흐름이다.

### 다단계 큐

![](https://media.vlpt.us/post-images/pa324/fc18cb30-15a2-11ea-934d-7b176b41c2f3/image.png)

작업에 대한 큐를 여러 개 두어 각 큐마다 다른 스케줄링 기법을 적용하여 처리한다.

큐는 수직적으로 배치되며 상위 단계의 작업이 먼저 선점된다.

### 다단계 피드백 큐

![](https://media.vlpt.us/post-images/pa324/07383c80-15a3-11ea-a2b9-311a942f152c/image.png)

새로운 프로세스는 높은 우선순위를 갖지만 프로세스의 실행시간이 늘어날수록 낮은 우선순위 큐로 이동하며 마지막 단계에서 FCFS가 적용된다.

---

## 비선점형 알고리즘

다른 프로세스에 CPU의 처리 순서를 `양보하지 않는다.`

### FCFS 스케줄링  (First Come First Served Scheduling)

먼저 들어온 작업 순으로 처리된다. 우선순위가 높은 작업을 빠르게 처리하지 못할 수 있다.

### SJF 스케줄링(Shortest Job First Scheduling)

가장 작업 시간이 짧은 순으로 처리된다. 수행시간이 긴 작업은 기아현상에 취약하다.

### HRRN 스케줄링(Highest Response Ratio Next Scheduling)

프로세스 처리의 우선 순위를

- CPU 처리 기간과
- 해당 프로세스의 대기 시간

을 동시에 고려해 선정하는 스케줄링 알고리즘이다. SJF 스케줄링의 문제점인 수행 시간이 긴 프로세스의 무한 대기 현상(기아 현상)을 보완해 개발된 스케줄링이다.
