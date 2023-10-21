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

# 참고 자료

[Jdk-17 Specification](https://blogs.oracle.com/javamagazine/post/java-jdk-17-generally-available)

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

[자세한 내용](https://k-diger.github.io/posts/JITCompiler/)

## Execution Engine - Garbage Collector

참조되지 않은 객체를 정리해준다.

[자세한 내용](https://k-diger.github.io/posts/GarbageCollector/)


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

## GC옵션 더 알아보기

위에서 튜닝한 옵션으로 ParallelGC를 사용했다고 했다. 그러면 GC의 종류는 어떤 것들이 있고 왜 ParallelGC를 사용했는지 알아보자.

[JAVA 9 DOCS](https://docs.oracle.com/javase/9/gctuning/available-collectors.htm#JSGCT-GUID-C7B19628-27BA-4945-9004-EC0F08C76003)

위 문서에 따르면 GC의 종류는 다음으로 나뉜다.

- Serial
- Parallel
- CMS
- G1

### Serial GC (-XX:+UseSerialGC)

Serial GC는 **단일 스레드**를 사용하여 모든 가비지 수집 작업을 수행한다.

따라서 **스레드 간에 통신으로 인한 오버헤드가 없기** 때문에 효율적이다.

작은 데이터 세트(최대 약 100MB)가 있는 응용 프로그램의 다중 프로세서에서 유용할 수 있지만 멀티 코어의 하드웨어를 활용할 수 없기 때문에 **단일 코어 시스템에 가장 적합**하다.


### Parallel GC (-XX:+UseParallelGC)

Parallel GC는 Serial GC와 유사하지만, Serial GC와 Parallel GC의 차이점은 Parallel GC에는 GC 속도를 높이는 데 사용되는 **멀티 스레드**가 있다.

Parallel GC는 멀티 코어 또는 멀티 쓰레드 하드웨어에서 실행되는 중대형 데이터 세트가 포함된 응용 프로그램에서 사용하기 적합하다.

또한 Parallel GC가 가지고 있는 **Parallel Compaction** 이라는 기능은 기본 값으로 제공되는데 GC가 동작을 병렬로 수행할 수 있도록 하는 기능이다.

**Parallel Compaction**이 없으면 단일 쓰레드를 사용하여 GC가 수행되므로 확장성이 크게 제한될 수 있다.

### CMS<Concurrent Mark Sweep> GC, G1<Garbage-First> (-XX:+UseG1GC or -XX:+UseConcMarkSweepGC)

- CMS GC는 Java 9 이후로 사용되지 않기 때문에 부연 설명은 제외하겠다.

G1 GC는 대용량 메모리가 있는 멀티 코어 시스템용이다.

높은 처리량과 가장 짧은 Stop-The-World를 가지고 있다.

G1은 특정 하드웨어 및 운영 체제 구성에서 기본값으로 선택된다. 혹은 다음 옵션으로 사용 가능하다. (-XX:+UseG1GC)

---

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
