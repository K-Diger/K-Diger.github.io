---

title: Effective-Java Item 7. 다 쓴 객체 참조를 해제하라
author: 김도현
date: 2023-01-24
categories: [Effective-Java]
tags: [Object, Reference]
math: true
mermaid: true

---

# 메모리 누수가 발생하는 Stack

```java
import java.util.Arrays;
import java.util.EmptyStackException;

public class Stack {

    private Object[] elements;

    private int size = 0;

    private static final int DEFAULT_INITIAL_CAPACITY = 16;

    public Stack() {
        this.elements = new Object[DEFAULT_INITIAL_CAPACITY];
    }

    public void push(Object e) {
        this.ensureCapacity();
        this.elements[size++] = e;
    }

    private void ensureCapacity() {
        if (this.elements.length == size) {
            this.elements = Arrays.copyOf(elements, 2 * size + 1);
        }
    }

    // !!!!! 메모리 누수 !!!!!
    public Object pop() {
        if (size == 0) {
            throw new EmptyStackException();
        }
        return elements[--size];
    }

    private void ensureCapacity() {
        if (this.elements.length == size) {
            this.elements = Arrays.copyOf(elements, 2 * size + 1);
        }
    }
}
```

위 코드가 왜 누수가 발생할까?

다 쓴 참조가 발생한다.

## 다 쓴 참조란?

더 이상 쓰지 않을 참조 라는 뜻이다.

즉, 내가 메모리를 직접 관리하는 클래스가 있다면, 직접 레퍼런스를 null 로 셋팅해주어, GC 에게 사용하지 않는다는 것을 알려야 한다는 것이다.

만약 이 문제를 해결하지 않고 쌓아둔다면 OOM(OutOfMemory)가 발생할 가능성이 생긴다.

위 코드에서는 elements배열의 "**활성 영역**" 밖의 참조들이 다 쓴 참조에 해당하게 된다.

## 활성 영역이란 무엇일까?

![img.png](https://github.com/K-Diger/K-Diger.github.io/blob/main/images/WhatIsActiveAreaInJavaArray.png?raw=true)

그냥 배열의 모든 원소라고 보면 될 것 같다.

즉 사이즈가 100인 배열이 있다면, 0 ~ 99가 배열의 활성 영역에 해당한다.

## 저런건 GC가 알아서 못가져가나?

GC는 메모리 누수를 찾기가 너무 어렵다.

GC의 객체 수집 기준을 정리하면 다음과 같다.

GC는 "Tracing"이라는 과정을 통해 사용하지 않는 객체를 찾아낸다. 그 기본 기준은 더 이상 참조 받고 있지 않은 객체를 지정하고 있다.

GC가 탐색하는 순서는 "Live Thread"와 Static 영역을 살펴보는 것으로 시작한다.

Live Thread란 현재 실행 중 혹은 향후 실행할 스레드를 나타내는 것으로 "**활성된 스레드**"라고 보면 된다.

다시 정리하면 GC는 **Live Thread에서 시작하여 도달 가능한 모든 객체의 참조를 찾아**낸다.

Live Thread에서 **도달 가능한 객체는 GC의 대상에서 제외**된다.

그리고 **특정 객체를 수집하려면, 그 객체가 참조하는 모든 객체도 수집**해야한다!

그렇기 때문에 **GC는 다른 객체의 참조를 찾기 위하여 재귀적으로 탐색**하는 알고리즘을 가지고 있다.

또한 **도달 불가능한 객체를 참조**로 가지고있는 **도달 가능한 객체**는

**도달 불가능한 객체가 수집**되고 **그 이후에 도달 가능한 객체**가 아무것도 참조하지 않기 때문에 **GC 대상으로 선정**된다.

여기서 도달 불가능한 객체를 예시로 보면 다음과 같다.

```java
String str = new String("Hello, World!");
    str = null; // str is no longer being referenced

or

public void someMethod() {
    SomeObject obj = new SomeObject();
    // obj is reachable
    //... some code
    // obj is no longer reachable
}
```

someMethod가 종료되면 SomeObject에 접근할 수 있는 참조가 존재하지 않기 때문에 SomeObject 또한 GC 대상이 되는 것이다.

---

# 메모리 누수를 방지한 코드 - pop() 메서드 수정

```java
import java.util.Arrays;
import java.util.EmptyStackException;

public class Stack {

    private Object[] elements;

    private int size = 0;

    private static final int DEFAULT_INITIAL_CAPACITY = 16;

    public Stack() {
        this.elements = new Object[DEFAULT_INITIAL_CAPACITY];
    }

    public void push(Object e) {
        this.ensureCapacity();
        this.elements[size++] = e;
    }

    private void ensureCapacity() {
        if (this.elements.length == size) {
            this.elements = Arrays.copyOf(elements, 2 * size + 1);
        }
    }

    // !!!!! 메모리 누수 방지 !!!!!
    public Object pop() {
        if (size == 0) {
            throw new EmptyStackException();
        }

        Object result = this.elements[--size];
        elements[size] = null; // 다 쓴 참조를 해제한다!
        return result;
    }
}

```

이렇게 null 참조를 통하여 메모리 공간을 확보해주는 것과 동시에,

null 처리한 참조를 사용하려 할 때 NPE 를 발생 시킬 수 있게 됨으로써 잘못된 행위를 수행할 경우를 방지할 수 있는 장점도 생긴다.

---

# 캐시 또한 메모리 누수의 원인

객체 참조를 캐시에 넣고, 캐시에 넣은 객체를 사용하고 나서 메모리에 올려두는 상황에서 누수가 발생한다.

---

# 리스너, 콜백 또한 메모리 누수의 원인

자바 생태계에서 굳이 이 컨벤션을 쓸 일이 있을까? 싶지만

콜백에 적당한 return 으로 메모리 해지를 하면 메모리 누수를 방지할 수 있다.

또한 콜백을 위한 week reference로 저장하면 GC가 수거해갈 수 있게 된다. (ex : WeakHashMap)
