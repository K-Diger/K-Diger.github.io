---

title: Effective JVM

date: 2023-07-07
categories: [Java, JVM]
tags: [Java, JVM]
layout: post
toc: true
math: true
mermaid: true

---

# 참고자료

[Catsbi's Blog](https://catsbi.oopy.io/3ddf4078-55f0-4fde-9d51-907613a44c0d)

[Gmarket Tech Blog](https://dev.gmarket.com/62)

[Jdk-17 Specification](https://blogs.oracle.com/javamagazine/post/java-jdk-17-generally-available)

[JDK-17 GC Tuning Guide](https://docs.oracle.com/en/java/javase/17/gctuning/ergonomics.html#GUID-DA88B6A6-AF89-4423-95A6-BBCBD9FAE781)

[DZone](https://dzone.com/articles/jvm-architecture-explained)

---

# JVM 이란?

Java Code 나 Application 을 실행시키기 위한 런타임 환경을 제공해주는 엔진이다.

Java 코드를 .class파일(ByteCode)로 변환해준다.

JVM은 Java Runtime Environment(JRE)의 일부이다.

다른 언어의 컴파일러와는 다르게, Java 컴파일러는 JVM 이 인식할 수 있는 코드를 생성해낸다.

---

# JVM 기능

- Java 코드를 ByteCode 로 변환한다. (변환된 ByteCode는 인터프리팅 된다.)
- JVM은 메모리 공간 할당을 담당한다.
- Java Code <-> JVM <-> System

# Java 코드가 실행되는 플로우

Class Loader -> Byte Code Verifier -> Execution Engine

---

# JVM 구조

![JVM: Architecture](https://techvidvan.com/tutorials/wp-content/uploads/sites/2/2020/06/JVM-Model.jpg)

크게 분류하자면 `Class Loader`, `Runtime Data Areas`, `Execution Engine`으로 나눌 수 있다.

각 요소가 어떤 역할을 하는지 알아보자

---

# Class Loader

![Class Loader](https://javatutorial.net/wp-content/uploads/2017/11/jvm-featured-image.png)

클래스 파일을 로딩하기 위한 하위 시스템이다.

Class Loader는 세 가지 주요 기능이 있다.

- `Loading`
- `Linking`,
- `Initialization`

---

## Class Loader - Loading

클래스 로더는 .class 파일을 읽고, 바이너리 데이터를 생성하여 메소드 영역에 저장한다.

클래스 로더의 종류로는 세 가지가 있으며 각 역할이 부여되어있다.

- `BootStrap` : 시스템 클래스 및 Java 표준 라이브러리에 대한 클래스를 로딩한다. (jre/lib 폴더의 rt.jar에 위치한 클래스들을 로딩한다.)
- `Extension Class Loader` : jre/lib/ext 폴더 내의 클래스를 로딩한다.
- `Application Class Loader` : 작성한 코드에 대한 클래스, 환경 변수를 로딩한다.

또한 각 .class 파일 마다 JVM은 아래의 세 가지 정보를 `메서드 영역에 저장`한다.

1. `로드된 클래스`와 그 모든 `연관 부모 클래스`

2. .class 파일이 `Class` or `Interface` or `Enum`어떤 것으로 작성된 것인지?

3. `수정자`, `변수`와 `메서드 정보` 등

우리는 흔히 `getClass()`메서드로 class를 코드 관점에서도 사용이 가능하다. 클래스 로더가 하는 역할이 덕분인데,

.class 를 로딩을 마친 후에, JVM 은 `Heap 영역에 이 파일들을 나타내기 위해` Class 유형의 객체를 생성한다.

---

## Class Loader - Linking

`Verification`, `Preparation`, `Resolution` 을 수행한다.

### Class Loader - Linking : Verification

.class 파일이 올바른 형식으로 지정되고, 유효한 컴파일러에 의해 생성되었는지 확인한다.

확인에 실패 했을 때 `RuntimeException` 혹은 `java.lang.VerifyError` 가 발생한다.

이 과정이 끝났다면, 클래스 파일을 컴파일 할 준비가 완료 된 것이다.

### Class Loader - Linking : Preparation

클래스 변수에 메모리를 할당하고 메모리를 기본값으로 초기화한다.

### Class Loader - Linking : Resolution

간접 참조 방식을 직접 참조 방식으로 바꾸는 과정이다. 참조된 엔티티를 찾기 위하여 `메서드 영역을 검색`한다.

---

## Class Loader - Initialization

모든 static 변수가 코드 및 static block 에 정의된 값으로 할당된다. (static 키워드가 들어간 요소들 메모리 할당)

부모에서 자식 순으로 수행된다.

--> static 요소들 메모리 할당 --> 부모 클래스 메모리 할당 --> 자식 클래스 메모리 할당 --> 기타 요소들 할당

---

# Runtime Data Area

## Method Area

`클래스 구조(클래스 이름, 부모 클래스 이름, 메서드 및 변수 정보)` 등 클래스 수준의 모든 정보를 담고 있다.

`static 변수`를 가지고 있다.

JVM 당 하나의 메서드 영역을 가지고 있고, `공유 될 수 있는` 영역이다.

---

## Heap

모든 `객체 및 연관된 인스턴스 변수/배열`이 저장되는 장소이다.

이 공간은 `공유될 수` 있으며 추후 `GC 의 대상`이 되는 동적인 영역이다.

Heap 영역은 조금 더 깊게 들어가면 다음 도식화 같이 구성되어있다.

![Java 7 vs Java 8 Heap](https://miro.medium.com/v2/resize:fit:513/0*rKZvTnuUkEc5LoXW.jpg)

위 그림으로 대강 틀을 잡고 아래 그림과 설명을 보면 이해가 쉬웠다.

![](https://itzsrv.com/static/55998773d3933af1327d3560a71ff975/083f8/jvm-mem.png)

### Young Generation (새로운 객체가 할당되는 공간)

- Young Generation은 새로운 객체들이 할당되는 영역이다.
- Eden 영역과 두 개의 Survivor 영역(S0, S1)으로 구성된다.
- 새로운 객체들은 Eden 영역에 할당된다.

### Eden 영역

- 새로운 객체가 할당되는 초기 영역이다.
- Eden 영역에 있는 객체들 중 일부는 살아남아서 Survivor 영역으로 이동한다.

### Survivor 영역 (S0, S1)

- Eden 영역에서 일정 기간 동안 살아남은 객체들 중 일부가 여기로 이동한다.
- 이동한 객체들 중 살아남은 객체는 다음 번 GC 사이클에서 다른 Survivor 영역으로 이동한다.
- 여러 번의 이동을 거치면서 살아남은 객체들은 Old Generation으로 이동할 수 있다.

### Old Generation (오랫동안 유지되는 객체가 할당되는 공간)

- Old Generation은 Young Generation에서 오랫동안 살아남은 객체들이 할당되는 공간이다.
- Old Generation 영역에 있는 객체들은 장기간 메모리를 점유하게 되며, 가비지 컬렉션 시에 주요 대상이 된다.

---

## Threads Area(Stack Area)

JVM은 `모든 쓰레드`에, `각 하나의 런타임 스택`을 제공한다.

각 스택의 모든 블럭은 `메서드 호출`이 발생하면 이를 `스택에 저장`하고 `지역 변수`가 이곳에 저장되며 `공유 가능`한 영역이다

---

## PC Register

스레드가 `현재 실행중인 JVM 명령의 주소`를 저장한다. `각 스레드 별로 별도의 PC 레지스터`가 있다.

---

### Native Internal Threads (Native Method Stacks)

Java 가 아닌 다른 언어(C, C++)로 작성된 Native Code 의 명령을 저장한다.

---

## Runtime Area 요약

### Method 영역 (공유 가능)
1. static 키워드로 선언된 요소들
2. 클래스 이름, 부모클래스, 클래스 메서드, 클래스 변수 등

### Heap (공유 가능)
1. 인스턴스
2. 인스턴스 변수

GC가 참조 되지 않은 메모리를 확인하고 제거하는 영역

### Thread (Stack) (공유 가능)
1. 메서드 내에서 사용되는 값들을 저장(매개변수, 메서드에서 선언한 변수, 리턴값 등)
2. 지역변수(메서드에 의한 변수)

---

# Execution Engine

`JIT Compiler` 와 `GC`가 존재하는 영역으로 Native Code (타 언어로 만들어진 메서드 등)을 다루는 영역이다.

실행 엔진은 .class 파일을 실행시켜준다.

또한 byte 코드를 각 행마다 읽어들이며(인터프리터에 의해) 메모리 공간에 존재하는 데이터와 정보들을 이용하며 명령어를 실행시켜준다.

실행 엔진은 `세 부분으로 분류` 할 수 있다.

## Execution Engine - Interpreter

바이트 코드를 한줄 씩 해석한 다음 실행한다.

인터프리터의 단점으로는, `하나의 메서드를 여러 번` 호출 할 경우, `매 번 해석`을 해줘야 하는 것으로 CPU, Memory를 더 잡아먹게 된다.

## Execution Engine - Just-In-Time Compiler(JIT)

인터프리터의 효율을 증가시키기 위해 사용된다.

`전체 바이트 코드를 컴파일`하여 `네이티브 코드(기계어)`로 변경하므로 인터프리터가 반복되는 메서드 호출을 하지 않아도

JIT 에서 해당 부분에 관한 네이티브 코드를 제공하므로

재 해석이 필요하지 않아 효율을 증가 시켜주는 방법이다.

---

# JIT Compiler 란

JAVA의 성능을 증대시키기 위한 런타임 환경의 컴파일러이다.

런타임 중 바이트 코드를 기계어로 컴파일해준다.

[IBM Documentation](https://www.ibm.com/docs/en/ztpf/1.1.0.15?topic=reference-jit-compiler)

[Oracle Documentation](https://docs.oracle.com/javase/specs/jvms/se8/html/index.html)

---

# JIT Compiler 개요

JAVA 프로그램은 다양한 컴퓨터 구조(OS) 에서 JVM 이 인터프리팅 할 수 있는 바이트 코드를 가진 클래스로 구성된다.

런타임 환경에서, JVM 은 클래스 파일(바이트 코드)을 로드하고, 각 개별 바이트 코드의 의미를 결정하고 적절한 계산을 수행한다.

즉, JIT 컴파일러는 런타임 때 바이트코드를 기계 코드로 컴파일하여 JAVA 프로그램의 성능을 개선하는데 도움이 된다.

### 그럼 어떤걸 기계 코드로 바꾸나?

우선 인터프리터와 JIT 컴파일러의 존재와 역할을 알아야한다.

인터프리터 : Java Code -> Bytes Code

JIT 컴파일러 : Bytes Code -> Machine Code

JVM은 바이트 코드를 실행시킬 수 있는 `인터프리터`가 존재한다.

그런데 여기서 자주 사용되는 명령어가 있다고 했을 때 이 바이트 코드로 이루어진 명령어를 매번 기계어로 해석하는 인터프리팅 작업이 반복된다면 성능은 분명 좋지 않을 것이다.

그래서 JIT 컴파일러가 등장했다. 자주 사용되는 명령어로 판단 된다면 `JIT 컴파일러`가 기계어로 변환한다.

---

# JIT Compiler 조금 더 알기

## 조금 더 알기 1

JIT 컴파일러는 기본값으로 무조건 동작하도록 셋팅 되어져있다.

어떤 메서드가 컴파일 되어있다고 했을 때, 그 컴파일 된 메서드를 호출하면

해당 메서드 내용을 ByteCode 로 변환하는 컴파일 과정을 거치지 않고 컴파일된 메서드(기계어)를 바로 호출한다.

컴파일 된 메서드를 호출하면 프로세서와 메모리의 사용을 요구하지 않아 성능이 좋아진다.

## 조금 더 알기 2

JIT 컴파일러가 첫 컴파일을 수행할 때는 프로세서 + 메모리를 필요로 한다.

JVM 이 처음 시작될 때 수 천개의 메서드가 호출되기 때문에 처음 시작에는 시작 시간이 늦어질 수 있다.

## 조금 더 알기 3

하지만, 실제로 모든 메서드가 처음 호출 된다고해서 컴파일 되는게 아니다!

각 메서드에 대해 JVM 은 Count 를 설정한다.

그리고 메서드가 호출될 때마다 Count 를 감소시켜 호출 횟수를 관리한다.

호출 횟수가 0에 도달하면 메서드에 관한 JIT 컴파일이 시작된다.

따라서 자주 사용하는 메서드는 JVM 의 시작 이후에 바로 컴파일되고

덜 사용하는 메서드는 훨씬 나중에 컴파일 되거나 컴파일 되지 않는다.

이 Count(임계값) 은 Application 시작 시간과 장기적인 성능 측면에서의 균형을 유지하기 위해 채택된 방법이다.

---

# JIT 컴파일 - 최적화

JIT 컴파일러는 각 메서드를 다른 최적화 단계를 부여하여 컴파일한다.

## 최적화 단계

`Cold`, `Warm`, `Hot`, `veryHot`, `Scorching`

최적화 수준이 뜨거울수록(Scorching 단계와 가까워질수록) 더 높은 성능을 제공하지만 컴파일 비용이 더 많이 들게 된다.

메서드의 기본 최적화 단계는 Warm 이지만, JIT 가 시작 시간을 개선하기 위해 Cold 단계로 다운그레이드 하기도 한다.

다른 매커니즘을 통해 메서드를 더 최적화 하여 컴파일 할 수도 있다.

hot, veryHot, scorching 단계의 메서드들이 그 대상이다.

JIT 컴파일러는 비활성화 할 수 있지만 JIT의 문제 진단을 위한 경우 등을 제외하곤 권장되진 않는다.

---

# JIT 컴파일러가 코드를 최적화 하는 방법

## JIT 컴파일러에게 학습 시킬 것

컴파일을 위한 메서드가 선택되면, JVM 은 JIT 에게 바이트 코드를 전달한다.

JIT는 바이트코드의 문장을 정확하게 이해할 수 있어야 한다.

그래서 JIT 가 메서드를 분석하는데 도움이 되도록 바이트 코드는, 기계 코드와 더 유사한 Tree 라는 내부 표현으로 재구성된다.

그 후 Trees 가 기계어로 번역이 된다.

즉, BytesCode -> Trees -> MachineCode

## JIT 컴파일러 성능 증대

JIT 컴파일러는 `여러 개의 컴파일 스레드`를 이용하여 컴파일 작업을 수행할 수 있다.

JIT 컴파일 스레드는 시스템에서 사용되지 않고 있는 코어가 있을 경우에만 성능 향상을 제공한다.

---

# JIT 컴파일 과정 (최적화 과정)

## 1단계 : Inlining

인라인 작업은, A 라는 메서드가 있을 때 이 메서드를 B라는 더 큰(더 자주 사용하는) 메서드나 클래스가 사용한다면

B가 담겨있는 트리에 A 라는 메서드를 병합하는 것이다.

이렇게하면 자주 실행되는 메서드의 호출 속도가 빨라지게 된다.

### 인라인 정책

1. Trivial inlining

2. Call graph inlining

3. Tail recursion elimination

4. Virtual call guard optimizations

## 2단계 : Local Optimizations

로컬 최적화는 고전적인 컴파일러의 기능을 수행하여 최적화를 한다.

### 최적화 과정

1. Local data flow analyses and optimizations

2. Register usage optimization

3. Simplifications of Java idioms

## 3단계 : Control Flow Optimizations

메서드 내부의 흐름을 분석하고, 코드를 재 정렬한다.

### 흐름 제어 과정

1. 코드 재정렬, 분할 및 제거

2. 루프 감소 및 반전

3. 루프 스트라이딩 및 루프 불변 코드 모션

4. 루프 풀기 및 필링

5. 루프 버전 관리 및 전문화

6. 예외 지향 최적화

7. 스위치 분석

## 4단계 : Global Optimizations

1. 전체 메서드를 대상으로 수행된다. 컴파일 시간이 길어 비용이 크지만 성능을 크게 향상시킬 수 있는 과정이다.

2. 전역 최적화 과정

3. 글로벌 데이터 흐름 분석 및 최적화

4. 부분적 중복 제거

5. 탈출 분석

6. GC 및 메모리 할당 최적화

7. 동기화 최적화

## 5단계 : Native Code Generation

기계어 생성!

---

## Execution Engine - Garbage Collector

참조되지 않은 객체를 정리해준다.

# Garbage Collection이란?

Java 프로그램에서, 자동으로 메모리를 관리해주는 과정이다.

C/C++ 와 달리, 프로그래머는 객체의 메모리 할당과 해제를 신경 쓸 필요가 없다.

Garbage Collection 이 데몬 쓰레드의 형태로 수행해주기 때문이다.

---

# Garbage Collection 동작 배경

Java 코드는 JVM 에 의해 ByteCode로 컴파일된다.

Java 프로그램이 JVM 에서 실행될 때 객체는 Heap에 저장된다.

이때 일부 객체들은 더 이상 힙에 존재할 필요가 없어질 때가 오게 된다.

이때 Garbage Collection 이 사용되지 않는 객체들을 찾아 제거한다.

---

# Garbage Collection 동작 과정

Garbage Collection 은 Heap 영역을 살펴보고 사용 중인 객체와 사용하지 않는 객체를 식별한다.

이 중 사용하지 않는 객체에 해당되는 요소들은 삭제한다.

사용 중인 객체는 포인터가 유지되고 있지만

사용 중이지 않은 객체는 어떤 부분에서도 포인터가 존재하지 않는다.

Garbage Collection 은 JVM 이 구현한다.

---

# Garbage Collection 의 동작 유형

## JVM 힙 영역의 구분

![](https://www.programmersought.com/images/220/8462f4bfba5f173585f6f8cd2dce12ac.JPEG)

### Young Generation

새로 생성된 객체는 Young Generation 에서 시작한다.

모든 객체가 시작되는 1개의 Eden 공간과

Garbage Collection Cycle 에서 살아남은 후

Eden 공간에서 살아남은 객체들이 이동하는 공간인 2개의 Survivor 공간이 있다.

### Old Generation

수명이 긴 객체는 결국 Young Generation 에서 Old Generation 으로 이동된다.

Old Generation 에서 Garbage Collected 가 되었을 때, 가장 우선순위가 높게 처리되는 공간이다.

### Permanent Generation (Metaspace)

클래스 및 메서드와 같은 메타데이터가 저장되는 장소이다.

추가적으로, `Java 8 에 들어오면서, Permanent Generation 이 사라지고 Metaspace 영역이 생겼다.`

`Permanent Generation` 은 `JVM 에 의해서 Heap 영역의 메모리 크기가 강제되던 영역`이였다.

하지만 `Metaspace 가 Native 메모리 영역`에 배치되면서, `운영체제가 자동으로 그 크기를 조절`할 수 있게 되고, `Heap 에서 사용할 수 있는 메모리의 크기가 늘어나게 됐다.`

---

# GC 상태

## Minor/Incremental Garbage Collection

Young Generation 에서 객체가 제거 되었다는 것을 말하는 유형이다.

## Major/Full Garbage Collection

Minor Garbage Collection 에서 살아남은 객체를 Old 로 옮긴다.

Old 에서는 GC 대상이 덜 자주 발생하게 된다.

---

# Garbage Collection 에 관한 주요 개념

## Unreachable Objects - 도달 할 수 없는 객체

객체에 관한 참조가 포함되어 있지 않으면 객체에 도달 할 수 없다 라고 한다.

```java
public class Main {
    public static void main(String[] args) {
        Integer i = new Integer(4);
        // the new Integer object is reachable  via the reference in 'i'

        i = null;
        // the Integer object is no longer reachable.
    }
}
```

위 코드를 보면, Integer 4를 대입했을 때는, 참조가 가능하지만

null 을 할당한 순간, 메모리를 참조할 수 없기 때문에 Unreachable 하다고 보는 것이다.

## Eligibility for Garbage Collection - GC 자격

위 코드를 보면, Integer 4를 위한 공간을 할당해 줬지만

i = null 이라는 구문 이후 에는 그 공간이 쓸모 없게 되는 것이다.

따라서 이 공간에 관하여 GC에 적합하다고 한다.

## GC 대상으로 만드는 방법

1. 참조 변수를 Null 할당이 가능하도록 한다.

2. 참조 변수를 재할당 한다.

3. 메서드 내에 객체를 생성한다.

4. Isolation Island 를 활용한다.

GC 대상이 될 수 있는 객체를 만들었다 하더라도, 즉시 GC 에 의해 제거되진 않는다. JVM이 GC를 실행시킬 때 객체가 소멸된다.

---

## JVM 에 GC 를 실행시키도록 하는 방법

1. `System.gc()` 메서드를 사용하여 JVM 에 GC 를 실행하도록 하는 Static 메서드를 실행 시킨다.

2. `Runtime.getRuntime().gc()` 메서드를 사용하여 GC 를 요청할 수 있다.

또한 `System.gc()` 는 `Runtime.getRuntime().gc()` 와 사실상 동일한 요청이다.

---

## GC 의 이점

Heap 영역에서 참조되지 않은 객체를 제거하기 때문에 메모리 효율성을 높일 수 있다.

JVM 의 일부분인 GC 가 자동으로 수행되기 때문에 프로그래머의 추가 작업이 필요하지 않다.

---

## Java 코드로 GC 실행시키는 방법

### 일반적인 JAVA CODE (GC 실행 X)

```java
    class Employee {

    private int ID;
    private String name;
    private int age;
    private static int nextId = 1;
    // it is made static because it
    // is keep common among all and
    // shared by all objects

    public Employee(String name, int age) {
        this.name = name;
        this.age = age;
        this.ID = nextId++;
    }

    public void show() {
        System.out.println("Id=" + ID + "\nName=" + name
            + "\nAge=" + age);
    }

    public void showNextId() {
        System.out.println("Next employee id will be="
            + nextId);
    }
}

class UseEmployee {
    public static void main(String[] args) {
        Employee E = new Employee("GFG1", 56);
        Employee F = new Employee("GFG2", 45);
        Employee G = new Employee("GFG3", 25);
        E.show();
        F.show();
        G.show();
        E.showNextId();
        F.showNextId();
        G.showNextId();

        { // It is sub block to keep
            // all those interns.
            Employee X = new Employee("GFG4", 23);
            Employee Y = new Employee("GFG5", 21);
            X.show();
            Y.show();
            X.showNextId();
            Y.showNextId();
        }
        // After countering this brace, X and Y
        // will be removed.Therefore,
        // now it should show nextId as 4.

        // Output of this line
        E.showNextId();
        // should be 4 but it will give 6 as output.
    }
}
```

### GC 실행 JAVA CODE

```java
    class Employee {

    private int ID;
    private String name;
    private int age;
    private static int nextId = 1;

    // it is made static because it
    // is keep common among all and
    // shared by all objects
    public Employee(String name, int age) {
        this.name = name;
        this.age = age;
        this.ID = nextId++;
    }

    public void show() {
        System.out.println("Id=" + ID + "\nName=" + name
            + "\nAge=" + age);
    }

    public void showNextId() {
        System.out.println("Next employee id will be="
            + nextId);
    }

    protected void finalize() {
        --nextId;
        // In this case,
        // gc will call finalize()
        // for 2 times for 2 objects.
    }
}

public class UseEmployee {
    public static void main(String[] args) {
        Employee E = new Employee("GFG1", 56);
        Employee F = new Employee("GFG2", 45);
        Employee G = new Employee("GFG3", 25);
        E.show();
        F.show();
        G.show();
        E.showNextId();
        F.showNextId();
        G.showNextId();

        {
            // It is sub block to keep
            // all those interns.
            Employee X = new Employee("GFG4", 23);
            Employee Y = new Employee("GFG5", 21);
            X.show();
            Y.show();
            X.showNextId();
            Y.showNextId();

            //GC
            X = Y = null;
            System.gc();
            System.runFinalization();
            //GC
        }
        E.showNextId();
    }
}
```

## GC 진행 단계 - Minor Collection

힙 영역 중 Young Generation에 속한 객체를 제거한다.

오라클 공식문서에 따르면 Young Generation에는 죽은 객체들이 많기 때문에 비교적 빠르게 수행된다고 한다.

즉, Stop-The-Wolrd의 지속시간이 짧다는 의미이다.

일반적으로 이 GC 단계에서 살아남은 객체들은 Old Generation으로 이동된다.

## GC 진행 단계 - Major Collection

힙 영역 중 Old Generation에 속한 객체를 제거한다.

Old Generation의 용량이 가득 차면 수행되는 GC단계로, 전체 힙을 대상으로 GC를 수행한다. 따라서 Stop-The-World의 지속시간이 상대적으로 길게 동작한다.

## GC 성능 고려 사항

### 전체 Heap의 용량

GC의 처리량은 사용 가능한 메모리에 반 비례한다.

할당된 메모리가 크면 Heap 영역자체가 꽉 차는 주기가 늘어나기 때문에 GC가 동작하는 주기 자체도 늘어나기 때문이다.

기본적으로 Heap이 할당되는 비율은 다음과 같다.

- -XX:MaxHeapFreeRatio (default value is 70%)
- -XX:MinHeapFreeRatio (default value is 40%)

여기서 사용되는 메모리의 비율은 시스템 전체의 메모리를 나타내는 것이 아닌 JVM에 할당된 메모리를 가리킨다.

### Young Generation의 크기

`-XX:NewRatio=3`와 같은 옵션을 사용하면 Young Generation과 Old Generation의 비율이 1:3이라는 것을 의미한다.

즉, eden 공간과 Survivor 공간을 합친 크기는 전체 힙 크기의 1/4이 된다.

Young Generation의 크기에 따라 GC 성능이 달라질 수 있다.

Young Generation의 크기가 클 수록 Minor Collection이 덜 자주 발생한다. 하지만 정적 크기의 Heap용량을 부여했을 때 Young Generation의 크기가 큰 만큼 Old Generation의 크기가 작아지는 것을 의미하므로

Major Collcetion이 자주 발생할 수 있다. 따라서 최적의 선택을 하기 위해선 객체의 수명을 분석해야한다.

### Survivor Space의 크기

`-XX:SurvivorRatio`옵션으로 Survivor의 크기를 지정할 수 있다.

이 공간의 크기가 너무 작으면 Old Generation으로 오버플로우가 발생할 수 있고

이 공간의 크기가 너무 크다면 쓸데없이 비어있게 되어 낭비될 수 있다.

따라서 JVM은 객체가 오래되기 전에 복사할 수 있는 횟수인 임계값 수를 선택한다.

`-Xlog:gc,age`옵션으로 이 임계값과 Young Generation의 객체들의 수명을 표시할 수 있다.

# 튜닝할 시 고려해아할 점 요약

### Young Generation

- 먼저 JVM에 제공할 수 있는 최대 힙 크기를 결정한다.
  - 그런 다음 최적의 설정을 찾기 위해 Young Generation 규모에 대한 성능 지표를 구성한다.

- 과도한 페이지 오류 및 스래싱을 방지하려면 최대 힙 크기는 항상 시스템에 설치된 메모리 양보다 작아야한다.

- 총 힙 크기가 고정된 경우 Young Generation 크기를 늘리려면 Old Generation 크기를 줄여야 한다.
  - 특정 시간에 애플리케이션에서 사용하는 모든 라이브 데이터와 일정량의 여유 공간(10~20% 이상)을 보유할 수 있을 만큼 이전 세대를 충분히 크게 유지해야한다.

### Old Generation

- 되도록 Yong Generation에 많은 공간을 할당하는 것이 좋다.

---

# GC종류

## GC - Serial Collector

Serial Collector는 단일 스레드를 사용하여 모든 가비지 수집 작업을 수행하여 스레드 간 통신 오버헤드가 없기 때문에 상대적으로 효율적이다.

다중 프로세서 하드웨어를 활용할 수 없기 때문에 단일 프로세서 시스템에 가장 적합하며 특정 하드웨어 및 운영 체제 구성에서 기본적으로 선택되거나 `-XX:+UseSerialGC`옵션을 사용하여 명시적으로 활성화할 수 있다.

---

## GC - Parallel Collector (JDK 6)

Parallel Collector는 Throughput Collector 라고도 하며 Serial Collctor와 유사하다.

Serial Collector와 Parallel Collector의 주요 차이점은 Parallel Collector에는 가비지 수집 속도를 높이는 데 사용되는 여러 스레드가 있다는 것이다.

Parallel Collector는 다중 프로세서 또는 다중 스레드 하드웨어에서 실행되는 중대형 데이터 세트가 있는 애플리케이션을 위한 것으로 `-XX:+UseParallelGC.`옵션을 사용하여 활성화할 수 있다.

병렬 압축은 Parallel Collector가 Major Collection을 병렬로 수행할 수 있도록 하는 기능이다. 병렬 압축이 없으면 Major Collection이 단일 스레드를 사용하여 수행되므로 확장성이 크게 제한될 수 있다.

`-XX:+UseParallelGC`지정된 경우 병렬 압축이 기본적으로 활성화되고 `-XX:-UseParallelOldGC`옵션을 사용하여 병렬 압축을 비활성화할 수 있다.

### Parallel GC의 스레드 수를 지정하기



---

## GC - Garbage-First (G1) Garbage Collector (JDK 11)

G1은 동시 수집기이다. 동시 수집기는 애플리케이션에 대해 비용이 많이 드는 일부 작업을 동시에 수행한다. 이로 인해 높은 처리량을 달성할 수 있다.

G1 GC는 소형 시스템에서 대량의 메모리를 갖춘 대형 멀티프로세서 시스템으로 확장하도록 설계되었으며, Stop-The-World의 지속 시간을 단축시켰다.

G1은 대부분의 하드웨어 및 운영 체제 구성에서 기본적으로 선택되거나 `-XX:+UseG1GC`를 사용하여 명시적으로 활성화할 수 있다.

---

### GC - ZGC (JDK 15)

ZGC(Z Garbage Collector)는 Stop-The-World가 가장 짧은 확장 가능한 Garbage Collector이다.

ZGC는 애플리케이션 스레드 실행을 중단하지 않고 비용이 많이 드는 모든 작업을 동시에 수행한다.

ZGC는 몇 밀리초의 Stop-The-World을 가지지만 이로 인해 일부 처리량이 희생됩니다. 적은 시간의 Stop-The-World 필요한 애플리케이션을 위한 것입니다.

Stop-The-World 지속시간은 사용 중인 힙 크기와 무관하며. ZGC는 8MB에서 16TB까지의 힙 크기를 지원한다. `-XX:+UseZGC`

---

# JVM 튜닝 - 문제 상황

프로젝트를 진행하면서 크롤러를 돌리는 중 아래와 같은 에러가 뜨고 있었다.

![](https://github.com/K-Diger/K-Diger.github.io/assets/60564431/181f8d12-d4b9-4479-9c82-c4f64ee3231a)

크롤링하면서 OOM이 터졌다는 것 같은데 왜 터진지 알아보자..

# 서버 스펙

기존 서버 스펙은 NCP의 Compact옵션인 (CPU: 2CORE, Memory : 2GB)를 사용하고 있었다.

서버 내 Java 버전은 OpenJDK 17버전을 사용하는 것으로 아무런 튜닝 옵션을 주지 않고 애플리케이션을 실행하고 있었기 때문에 다음과 같은 JVM 스펙이 동작하고 있었다.

[JDK 17](https://docs.oracle.com/javase/specs/jvms/se17/html/jvms-2.html)

![image](https://github.com/K-Diger/K-Diger.github.io/assets/60564431/768fd0fa-5052-48fc-acc6-d70a99a8bc5d)

현재 나의 서버에서 구동되는 JVM 정보는 위 그림과 같다. (위 사진은 최근 스펙업을 한 2CORE, 4GB Memory로 대체하였다.)

기본값으로 구동되는 옵션의 기준은 아래 링크에 설명되어있다.

[Oracle Docs](https://www.oracle.com/java/technologies/ergonomics5.html)

- 초기 힙 크기 : 물리적 메모리의 1/64 --> 8.388608MB

- 최대 힙 크기 : 물리적 메모리의 1/4 --> 1GB

여튼 기본값을 지정된 초기 힙 크기를 가지고 크롤러를 돌릴 때 문제가 발생한다고 하니까 이를 해결해보자.

# 서버, JVM 튜닝

우선 JVM의 절대적인 Heap 공간이 모자라다는 것이 문제다. 모니터링을 통해 내린 결론은 크롤러가 요구하는 Memory Size는 약 2GB 언저리이다.

따라서 애플리케이션 코드를 더 최적화 하지 않는 이상 최대 메모리를 2GB로 돌리는 것은 불가능할 것이다.

## 서버의 수직적 확장

서버의 스펙을 올린다. 기본 2Core 2GB Memory에서 2Core 4GB Memory로 스펙을 올렸다.

## JVM 튜닝

서버 스펙을 올려 메모리를 넉넉하게 발급했음에도 불구하고 아래와 같은 에러가 뜨고있었다.

```text
Exception: java.lang.OutOfMemoryError thrown from the UncaughtExceptionHandler in thread "http-nio-8081-Poller"
Exception in thread "HikariPool-1 housekeeper" java.lang.OutOfMemoryError: Java heap space
Exception in thread "Catalina-utility-2" java.lang.OutOfMemoryError: Java heap space
```

여전히 Heap 공간이 모자라다고 투정을 부리고 있는 것인데, 이 역시 JVM의 기본 값으로 Heap 사이즈를 설정했기 때문에 그 사이즈로도 감당이 안된다는 것인데

스펙업을 통해 남는 메모리를 조금 더 할당하도록 튜닝해야한다.

### 튜닝 명령어 - Heap Size 설정, GC 지정

```shell
java -Xms256m -Xmx2048m -XX:+UseParallelGC
```

위 명령어는 JVM 옵션을 커스텀하여 실행하도록 하는 것이다.

- **-Xms256m**은 최소 힙 사이즈를 나타내는 것으로 256M를 지정했다.

- **-Xmx2048m**은 최대 힙 사이즈를 나타내는 것으로 2048M(2GB)를 지정했다.

- **-XX:+UseParallelGC**은 GC를 지정해 주는 것인데, ParallelGC를 지정했다.

## GC 선택에 대한 결론

GC 종류를 알아봤으니 그 다음으로 어떤 GC가 어떨 때 적합한지 정리된 글을 보자면

- 애플리케이션에 다소 엄격한 Stop-The-Wolrd에 대한 요구 사항이 있는 경우가 아니면 먼저 애플리케이션을 실행하고 VM이 수집기를 선택하도록 허용해라.

- 필요한 경우 힙 크기를 조정하여 성능을 향상시켜라.
  - 성능이 여전히 목표를 충족하지 못하는 경우 GC를 직접 선택해라.

- **소규모 애플리케이션**일 경우에는(최대 약 100MB) **Serial GC**를 선택해라. (-XX:+UseSerialGC)
  - 애플리케이션이 **단일 프로세서**에서 실행되고 **Stop-The-World에 대한 제약이 없는 경우** **Serial GC**를 선택해라.

- 애플리케이션 **성능이 첫 번째 우선 순위**이고 **Stop-The-World에 대한 요구 사항이 없거나 1초 이상의 일시 중지가 허용**되는 경우 **기본 값**으로 지정된 GC를 사용하거나 **Parallel GC**를 선택해라. (-XX:+UseParallelGC)

- **응답 시간이 전체 처리량보다 중요**하고 **Stop-The-World 시간이 약 1초 미만**으로 유지해야 하는 경우는 **G1 GC or CMS GC**를 사용해라. (-XX:+UseG1GC or -XX:+UseConcMarkSweepGC)

위 결론에 따라서 나는 Parallel GC를 선택했다. 시간당 한 번씩 로직을 수행하기 때문에 Stop-The-World에 대한 제약사항은 우선순위가 가장 낮으나 애플리케이션 자체의 성능을 극대화 하는게 더 우선이기 때문이다.

메모리 부족이 발생한 상황을 다시 돌아보면 초기 서버에는 괜찮았지만 시간이 지날 수록 쓰레기값들이 누적되면서 메모리가 터져버린 것으로 예상되었다.

따라서 아래와 같은 명령어를 서버 시스템 내의 crontab으로 설정하여 Cache Memory를 정리해 주는 방법을 추가했다.

```shell
0 * * * * sync && echo 3 > /proc/sys/vm/drop_caches
```
