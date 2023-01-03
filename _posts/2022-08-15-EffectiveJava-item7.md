---

title: Item 7. 다 쓴 객체 참조를 해제하라
author: 김도현
date: 2022-08-15
categories: [Effective-Java]
tags: [Object, Reference]
math: true
mermaid: true

---

# 메모리 누수가 발생하는 Stack

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

            Object result = this.elements[--size];
            this.elements[size] = null;
            return result;
        }
    }

<br>

> 왜 누수가 발생할까?

<br>

다 쓴참조가 발생한다. (다 쓴 참조란? : 더 이상 쓰지 않을 참조 라는 뜻이다.)

즉, 내가 메모리를 직접 관리하는 클래스가 있다면, 직접 레퍼런스를 null 로 셋팅해주어, GC 에게 사용하지 않는다는 것을 알려야 한다는 것이다.

---

# 메모리 누수를 방지한 코드 - Pop() 메서드 수정

    public Object pop() {
        if (size == 0) {
            throw new EmptyStackException();
        }

        Object result = this.elements[--size];
        this.elements[size] = null;
        return result;
    }


<br>

이렇게 null 참조를 통하여 메모리 공간을 확보해주는 것과 동시에,

null 처리한 참조를 사용하려 할 때 NPE 를 발생 시킬 수 있게 됨으로써 잘못된 행위를 수행할 경우를 방지할 수 있는 장점도 생긴다.

---

# 캐시 또한 메모리 누수의 원인

객체 참조를 캐시에 넣고, 캐시에 넣은 객체를 사용하고 나서 메모리에 올려두는 상황에서 누수가 발생한다.

---

# 리스너, 콜백 또한 메모리 누수의 원인

자바 생태계에서 굳이 이 컨벤션을 쓸 일이 있을까? 싶지만

콜백에 적당한 return 으로 메모리 해지를 하면 메모리 누수를 방지할 수 있다.
