---

title: Spring - Thread Local
author: 김도현
date: 2023-01-27
categories: [Spring, Thread-Local]
tags: [Spring, Thread-Local]
math: true
mermaid: true

---

# Thread Local이란?

특정 쓰레드만 접근할 수 있는 저장소로, 쓰레드별로 할당되는 저장소이다. (쓰레드 내부에 내부 저장소를 제공하는 것!)

이는 Spring 특유의 "싱글톤"이라는 특성으로 인한 "동시성 문제"를 해결하기 위해 등장한 것이다.

## 싱글톤의 동시성 문제

Diger가 1번 쓰레드를 통해 NameSpace라는 변수에 "diger"라는 값을 넣고 비즈니스 로직을 요청했다.

그 직후 John이 2번 쓰레드를 통해 NameSpace라는 변수에 "john"이라는 값을 넣고 비즈니스 로직을 요청했다.

만약 비즈니스 로직을 처리하는데 5초가 소요되고, John은 Diger가 요청한 지 1초만에 요청한 것이라고 했을 때

다시 보관함에 넣은 값을 돌려받을 때 어떻게 돌려받게 될 것 같은가?

NameSpace라는 변수에는 "Diger" 라는 값이 있었지만, 1초뒤에 "John"이라는 변수로 바뀌어 있다.

그 상태에서 Diger의 비즈니스 로직이 끝나서 NameSpace에 저장한 값을 돌려줄 때는 이미 "John"이라는 값으로 변경된 값을 돌려받는 상황이 되버렸다.

이 문제가 바로 싱글톤/전역변수의 변경에 따른 동시성 문제이다.

