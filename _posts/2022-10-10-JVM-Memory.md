---

title: JVM - Memory
date: 2022-10-10
categories: [JVM-Memory]
tags: [JVM-Memory]
layout: post
toc: true
math: true
mermaid: true

---


# 1. Method Area

- `클래스 구조(클래스 이름, 부모 클래스 이름, 메서드 및 변수 정보)` 등 클래스 수준의 모든 정보를 담고 있다.
- `static 변수`를 가지고 있다.
- JVM 당 하나의 메서드 영역을 가지고 있고, `공유 될 수 있는` 영역이다.

---

# 2. Heap

![Java 7 vs Java 8 Heap](https://miro.medium.com/v2/resize:fit:513/0*rKZvTnuUkEc5LoXW.jpg)

- 모든 `객체 및 연관된 인스턴스 변수/배열`이 저장되는 장소이다.
- 이 공간은 `공유될 수` 있으며 추후 `GC 의 대상`이 되는 동적인 영역이다.

Heap 영역은 조금 더 깊게 들어가면 다음 도식화 같이 구성되어있다.

## 2.1 Heap - Young Generation (새로운 객체가 할당되는 공간)

- Young Generation은 새로운 객체들이 할당되는 영역이다.
- Eden 영역과 두 개의 Survivor 영역(S0, S1)으로 구성된다.
- 새로운 객체들은 Eden 영역에 할당된다.

### 2.1.1 Heap Young Generation - Eden

- 새로운 객체가 할당되는 초기 영역이다.
- Eden 영역에 있는 객체들 중 일부는 살아남아서 Survivor 영역으로 이동한다.

### 2.1.2 Heap Young Generation - Survivor (S0, S1)

- Eden 영역에서 일정 기간 동안 살아남은 객체들 중 일부가 여기로 이동한다.
- 이동한 객체들 중 살아남은 객체는 다음 번 GC 사이클에서 다른 Survivor 영역으로 이동한다.
- 여러 번의 이동을 거치면서 살아남은 객체들은 Old Generation으로 이동할 수 있다.

## 2.2 Heap - Old Generation (오랫동안 유지되는 객체가 할당되는 공간)

- Old Generation은 Young Generation에서 오랫동안 살아남은 객체들이 할당되는 공간이다.
- Old Generation 영역에 있는 객체들은 장기간 메모리를 점유하게 되며, 가비지 컬렉션 시에 주요 대상이 된다.

---

# 3. Threads Area(Stack Area)

`모든 쓰레드`가 가지고 있는 쓰레드 고유의 영역이다.

`메서드 호출`이 발생하면 이를 `스택에 저장`하고 `지역 변수`가 이곳에 저장된다.

---

# 4. PC Register

`모든 쓰레드`가 가지고 있는 쓰레드 고유의 영역이다.

`현재 실행중인 JVM 명령의 주소`를 저장한다.

---

# 4. Native Internal Threads (Native Method Stacks)

`모든 쓰레드`가 가지고 있는 쓰레드 고유의 영역이다.

Java 가 아닌 다른 언어(C, C++)로 작성된 Native Code 의 명령을 저장한다.

---

# JVM Memory 요약

## Method 영역 (공유 가능)

1. static 키워드로 선언된 요소들
2. 클래스 이름, 부모클래스, 클래스 메서드, 클래스 변수 등

## Heap (공유 가능)

1. 인스턴스
2. 인스턴스 변수

GC가 참조 되지 않은 메모리를 확인하고 제거하는 영역

## Thread (Stack)

1. 메서드 내에서 사용되는 값들을 저장(매개변수, 메서드에서 선언한 변수, 리턴값 등)
2. 지역변수(메서드에 의한 변수)

---

# JVM 최소 힙 용량에서 최대 힙 용량까지

JVM은 최소 힙 용량과 최대 힙 용량을 가지고 있다. 그렇다면 최소 힙 용량으로 메모리 크기가 커버가 되지 않을 때 어떻게 용량을 늘리는걸까?

한 번에 최대 용량까지 늘리는걸까? 아니면 점진적으로 늘려가는 걸까? 그리고 이 과정에서 생기는 성능 이슈는 없을까?

힙 크기를 변경하는 작업은 JVM 내부에서 수행되며, 이 작업 도중에는 STW가 발생하게 된다.

힙 크기를 늘리면 그만큼 더 넓어진 메모리를 탐색해야하기 때문에 GC의 실행 시간도 증가할 수도 있다.
