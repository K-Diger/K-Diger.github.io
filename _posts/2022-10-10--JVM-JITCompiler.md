---

title: JVM - JIT Compiler
date: 2022-10-10
categories: [JIT-Compiler]
tags: [JIT-Compiler]
layout: post
toc: true
math: true
mermaid: true

---

[IBM Documentation](https://www.ibm.com/docs/en/ztpf/1.1.0.15?topic=reference-jit-compiler)

[Oracle Documentation](https://docs.oracle.com/javase/specs/jvms/se8/html/index.html)

---

# JIT-Compiler

Java는 `JVM`이 `Interpreting`할 수 있는 `ByteCode`를 가진 `.class`로 구성된다. 런타임 시점에, `JVM`은 `.class`파일을 로드하고, `byte code`의 의미를 결정하고 적절한 계산을 수행한다. 즉, `JIT Compiler`는 런타임 때 `byte code`를 `기계 코드로 컴파일`하여 Java 성능을 개선하는데 도움을 주고자 한다.

`JVM`은 `ByteCode`를 실행시킬 수 있는 `인터프리터`가 존재한다. 자주 사용되는 명령어가 있을때 이 `ByteCode`로 이루어진 명령어를 매번 기계어로 해석하는 `interpreting 작업이 반복된다면` 성능은 분명 좋지 않을 것이다. 그래서 JIT Compiler가 등장했다. 자주 사용되는 명령어로 판단 된다면 `JIT Compiler`가 기계어로 변환한다.

`Interpreter` : Java Code -> ByteCode

`JIT Compiler` : Bytes Code -> Machine Code

## JIT Compiler 특징 1

어떤 Method가 Compile 되어있다고 했을 때, 그 Method를 호출하면 ByteCode로 변환하는 Compile 과정을 거치지 않고 Compiled Method(Machine Language)를 바로 호출한다.

Compiled Method 호출하면 프로세서와 메모리의 사용을 요구하지 않아 성능이 좋아진다.

## JIT Compiler 특징 2

JIT Compiler가 첫 컴파일을 수행할 때는 프로세서 + 메모리를 필요로 한다.

JVM 이 처음 시작될 때 수 천개의 메서드가 호출되기 때문에 처음 시작에는 시작 시간이 늦어질 수 있다.

## JIT Compiler 특징 3

모든 메서드가 호출 된다고해서 컴파일 되는게 아니다. 각 메서드에 대해 JVM은 `Count`를 설정한다. 그리고 `메서드가 호출될 때마다` `Count를 감소`시켜 호출 횟수를 관리한다.

`Count가 0`에 도달하면 메서드에 관한 `JIT 컴파일이 시작`된다. 따라서 `자주 사용하는 메서드`는 `JVM의 시작 이후에 바로 컴파일`되고 `덜 사용하는 메서드`는 훨씬 `나중에 컴파일 되거나 컴파일 되지 않는다.`

이 Count(임계값) 은 Application 시작 시간과 장기적인 성능 측면에서의 균형을 유지하기 위해 채택된 방법이다.

---

## JIT 컴파일 - 최적화

JIT Compiler는 각 메서드를 다른 최적화 단계를 부여하여 컴파일한다.

### 최적화 단계

`Cold`, `Warm`, `Hot`, `VeryHot`, `Scorching`

최적화 수준이 뜨거울수록(Scorching 단계와 가까워질수록) 더 높은 성능을 제공하지만 컴파일 비용이 더 많이 들게 된다.

메서드의 `기본 최적화 단계`는 `Warm` 이지만, JIT가 시작 시간을 개선하기 위해 `Cold 단계로 다운그레이드 하기도 한다.`

다른 매커니즘을 통해 메서드를 더 최적화 하여 컴파일 할 수도 있다.

hot, veryHot, scorching 단계의 메서드들이 그 대상이다.

JIT Compiler는 비활성화 할 수 있지만 JIT의 문제 진단을 위한 경우 등을 제외하곤 권장되진 않는다.

---

### JIT Compiler가 코드를 최적화 하는 방법

#### JIT Compiler에게 학습 시킬 것

컴파일을 위한 메서드가 선택되면, `JVM은 JIT`에게 바이트 코드를 전달한다. `JIT`는 바이트코드의 문장을 정확하게 이해할 수 있어야 한다.

그래서 JIT가 메서드를 분석하는데 도움이 되도록 `바이트 코드`는, `기계 코드와 더 유사한 Tree`라는 내부 표현으로 `재구성`된다.

그 후 Trees 가 기계어로 번역이 된다. `BytesCode -> Trees -> MachineCode`

#### JIT Compiler 성능 증대

JIT Compiler는 `여러 개의 컴파일 스레드`를 이용하여 컴파일 작업을 수행할 수 있다.

JIT `컴파일 스레드`는 시스템에서 `사용되지 않고 있는 코어가 있을 경우에만` 성능 향상을 제공한다.

---

## JIT 컴파일 과정

### 1단계 : 인라인

인라인 작업은, A 라는 메서드가 있을 때 이 메서드를 B라는 더 큰(더 자주 사용하는) 메서드나 클래스가 사용한다면

B가 담겨있는 트리에 A 라는 메서드를 병합하는 것이다.

이렇게하면 자주 실행되는 메서드의 호출 속도가 빨라지게 된다.

#### 인라인 정책

1. Trivial inlining
2. Call graph inlining
3. Tail recursion elimination
4. Virtual call guard optimizations

---

### 2단계 : 로컬 최적화

로컬 최적화는 고전적인 Compiler의 기능을 수행하여 최적화를 한다.

#### 최적화 과정

1. 로컬 데이터 흐름 분석과 최적화
2. 레지스터를 사용하여 최적화
3. Java 문법 간소화

---

### 3단계 : 흐름제어 및 최적화

메서드 내부의 흐름을 분석하고, 코드를 재 정렬한다.

#### 흐름 제어 과정

1. 코드 재정렬, 분할 및 제거
2. 루프 감소 및 반전
3. 루프 스트라이딩 및 루프 불변 코드 모션
4. 루프 풀기 및 필링
5. 루프 버전 관리 및 전문화
6. 예외 지향 최적화
7. 스위치 분석

---

### 4단계 : 전역 최적화

1. 전체 메서드를 대상으로 수행된다. 컴파일 시간이 길어 비용이 크지만 성능을 크게 향상시킬 수 있는 과정이다.
2. 전역 최적화
3. 글로벌 데이터 흐름 분석 및 최적화
4. 부분적 중복 제거
5. 탈출 분석
6. GC 및 메모리 할당 최적화
7. 동기화 최적화

### 5단계 : 기계어 생성

기계어를 생성

---
