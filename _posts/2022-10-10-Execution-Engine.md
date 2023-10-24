---

title: JVM - Execution Engine
date: 2022-10-10
categories: [JVM-Execution-Engine]
tags: [JVM-Execution-Engine]
layout: post
toc: true
math: true
mermaid: true

---

# JVM Execution Engine

`JIT Compiler` 와 `GC`가 존재하는 영역으로 Native Code (타 언어로 만들어진 메서드 등)을 다루는 영역이다.

실행 엔진은 .class 파일을 실행시키고 ByteCode를 각 행마다 읽어들이며 메모리 공간에 존재하는 데이터와 정보들을 이용하며 명령어를 실행시켜준다.

실행 엔진은 `세 부분으로 분류` 할 수 있다.

## Execution Engine - Interpreter

바이트 코드를 한줄 씩 해석한 다음 실행한다. 인터프리터의 단점으로는, `하나의 메서드를 여러 번` 호출 할 경우, `매 번 해석`을 해줘야 하는 것으로 CPU, Memory를 더 잡아먹게 된다.

## Execution Engine - Just-In-Time Compiler(JIT)

인터프리터의 효율을 증가시키기 위해 사용된다. `전체 바이트 코드를 컴파일`하여 `네이티브 코드(기계어)`로 변경하므로 인터프리터가 반복되는 메서드 호출을 하지 않아도 JIT 에서 해당 부분에 관한 네이티브 코드를 제공하므로 재 해석이 필요하지 않아 효율을 증가 시켜주는 방법이다.

---

## Execution Engine - Garbage Collector

Java 프로그램에서, 자동으로 메모리를 관리해주는 과정이다. C/C++ 와 달리, 프로그래머는 객체의 메모리 할당과 해제를 신경 쓸 필요가 없다. Garbage Collector 이 데몬 쓰레드의 형태로 수행해주기 때문이다.

[Oracle UnderStanding GC](https://blogs.oracle.com/javamagazine/post/understanding-garbage-collectors)

[Oracle GC Tuning Document](https://docs.oracle.com/en/java/javase/17/gctuning/garbage-first-garbage-collector-tuning.html)

