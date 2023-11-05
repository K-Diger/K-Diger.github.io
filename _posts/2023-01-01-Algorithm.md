---

title: Algorithm
date: 2023-01-01
categories: [Algorithm]
tags: [Algorithm]
layout: post
toc: true
math: true
mermaid: true

---

# Java 지원 자료형

## List

| 연산      | ArrayList                  | LinkedList         |
|---------|----------------------------|--------------------|
| 끝에 삽입   | O(1), O(N)[리스트 크기를 늘려야할 때] | O(1)               |
| 중간에 삽입  | O(N)                       | O(N)[탐색], O(1)[삭제] |
| 끝에서 삭제  | O(1)                       | O(1)               |
| 중간에서 삭제 | O(N)                       | O(N)[탐색], O(1)[삭제] |
| 조회      | O(1)                       | O(N)               |

### ArrayList

동적 배열이다.

### LinkedList

이중 연결 리스트(Head, Tail을 가짐)이다.

---

## Map

모든 구현체의 연산의 시간 복잡도가 O(1)이다.

### HashMap (기본 Map)

Key-Value를 가지는 일반적인 Map이다. 순서를 보장하지 않는다.

### LinkedHashMap (순서 보장 Map)

Key-Value를 가지는 Map이다. 순서를 보장한다.

### TreeMap (특정 조건에 따른 정렬 Map)

값에 따라 순서를 정렬한다. 내부적으로 `Red-Black Tree`로 구현되어있고 정렬 순서를 임의로 지정할 수 있다.

---

## 크고 정밀한 숫자를 위한 자료형

### BigInteger

무한대의 숫자를 표기할 수 있다. 이로 인해 정밀한 수까지도 표현이 가능한 자료형이다.

모든 숫자를 이 자료형을 사용하게되면 성능 저하가 있으니 꼭 필요시 사용해야한다.

`add()`, `subtract()`, `multiply()`, `divide()`, `remainder()`

---
