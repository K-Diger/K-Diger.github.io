---

title: Fundamental OperatingSystem
date: 2023-10-18
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

- 프로세스는 메모리를 독립적으로 사용하지만 / 쓰레드는 메모리를 공유하여 사용한다.
- 프로세스는 문맥 전환 비용이 크지만 / 쓰레드는 문맥 교환 비용이 비교적 작다.
- 프로세스는 다른 프로세스에서 문제가 생겨도 오류가 전파되지 않기 때문에 안정성이 높지만 / 쓰레드는 공유된 영역을 사용하기 때문에 다른 쓰레드에 문제가 발생하면 전파될 가능성이 커 안정성이 비교적 낮다.

---

# 3. 언제 멀티 프로세스, 멀티 스레드를 사용하면 될까?

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
        add_value(primitiveVariable);

        // 1 : 값 변화가 없음
        System.out.println(var);

        // ------------------------------------ //

        // 배열 자체를 보냄 (call by reference)
        add_reference(referenceVariable);

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

![](https://github.com/K-Diger/K-Diger.github.io/assets/60564431/d78c0e67-e6c4-44c0-9af0-a2c10c3789d4)

![](https://github.com/K-Diger/K-Diger.github.io/assets/60564431/de915f18-4d3e-4976-a43e-a2a78c335d83)

### increaseReferenceValue

![](https://github.com/K-Diger/K-Diger.github.io/assets/60564431/25ab70f0-322c-461a-85f0-a4c0a6248149)

![](https://github.com/K-Diger/K-Diger.github.io/assets/60564431/876d1fc1-7ee0-43b0-ae44-3e74607b2eb1)

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

# 7. Sync-Async, Blocking - Non-Blocking

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

# 8. CPU 스케쥴링 기법

## 선점 (다른 프로세스 스케줄링에 개입)

### Round Robin

각 프로세스마다 동일한 시간을 할당하여 처리하도록 한다.

공평하게 자원을 분배할 수 있는 장점이 있지만

근소한 차이로 작업을 처리하지 못했을 경우 해당 프로세스의 처리가 지연된다는 단점이 존재한다.

### SRT

프로세스의 처리 시간이 가장 짧다고 계산된 순으로 처리하도록한다.

처리량이 높아질 수 있다는 장점이 있지만

우선적으로 처리해야할 프로세스에 대해 지연될 수 있다는 단점이 있다.

### 다단계 큐

![](https://media.vlpt.us/post-images/pa324/fc18cb30-15a2-11ea-934d-7b176b41c2f3/image.png)

작업에 대한 큐를 여러 개 두어 각 큐마다 다른 스케줄링 기법을 적용하여 처리한다.

큐는 수직적으로 배치되며 상위 단계의 작업이 먼저 선점된다.

### 다단계 피드백 큐

![](https://media.vlpt.us/post-images/pa324/07383c80-15a3-11ea-a2b9-311a942f152c/image.png)

새로운 프로세스는 높은 우선순위를 갖지만 프로세스의 실행시간이 늘어날수록 낮은 우선순위 큐로 이동하며 마지막 단계에서 FCFS가 적용된다.



---

## 비선점 (다른 프로세스 스케줄링에 개입)

자원을 반환하지 않고 프로세스를 처리
