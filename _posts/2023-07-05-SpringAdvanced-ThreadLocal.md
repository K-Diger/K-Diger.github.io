---

title: Thread Local
date: 2023-07-05
categories: [Spring, ThreadLocal]
tags: [Spring, ThreadLocal]
layout: post
toc: true
math: true
mermaid: true

---

# 참고자료

[Catsbi's Blog](https://catsbi.oopy.io/3ddf4078-55f0-4fde-9d51-907613a44c0d)

[Gmarket Tech Blog](https://dev.gmarket.com/62)


# Thread Local이란?

특정 쓰레드만 접근할 수 있는 저장소로, 쓰레드별로 할당되는 저장소이다. (쓰레드 내부에 내부 저장소를 제공하는 것!)

# 코드로 살펴보기

## ThreadLocal.java 전체 코드

```java
public class ThreadLocal<T> {

    private final int threadLocalHashCode = nextHashCode();

    private static AtomicInteger nextHashCode = new AtomicInteger();

    private static final int HASH_INCREMENT = 0x61c88647;

    private static int nextHashCode() {
        return nextHashCode.getAndAdd(HASH_INCREMENT);
    }

    protected T initialValue() {
        return null;
    }

    public static <S> ThreadLocal<S> withInitial(Supplier<? extends S> supplier) {
        return new SuppliedThreadLocal<>(supplier);
    }

    public ThreadLocal() {
    }

    public T get() {
        Thread t = Thread.currentThread();
        ThreadLocalMap map = getMap(t);
        if (map != null) {
            ThreadLocalMap.Entry e = map.getEntry(this);
            if (e != null) {
                @SuppressWarnings("unchecked")
                T result = (T)e.value;
                return result;
            }
        }
        return setInitialValue();
    }

    boolean isPresent() {
        Thread t = Thread.currentThread();
        ThreadLocalMap map = getMap(t);
        return map != null && map.getEntry(this) != null;
    }

    private T setInitialValue() {
        T value = initialValue();
        Thread t = Thread.currentThread();
        ThreadLocalMap map = getMap(t);
        if (map != null) {
            map.set(this, value);
        } else {
            createMap(t, value);
        }
        if (this instanceof TerminatingThreadLocal) {
            TerminatingThreadLocal.register((TerminatingThreadLocal<?>) this);
        }
        return value;
    }

    public void set(T value) {
        Thread t = Thread.currentThread();
        ThreadLocalMap map = getMap(t);
        if (map != null) {
            map.set(this, value);
        } else {
            createMap(t, value);
        }
    }

     public void remove() {
         ThreadLocalMap m = getMap(Thread.currentThread());
         if (m != null) {
             m.remove(this);
         }
     }

    ThreadLocalMap getMap(Thread t) {
        return t.threadLocals;
    }

    void createMap(Thread t, T firstValue) {
        t.threadLocals = new ThreadLocalMap(this, firstValue);
    }

    static ThreadLocalMap createInheritedMap(ThreadLocalMap parentMap) {
        return new ThreadLocalMap(parentMap);
    }

    T childValue(T parentValue) {
        throw new UnsupportedOperationException();
    }

    static final class SuppliedThreadLocal<T> extends ThreadLocal<T> {

        private final Supplier<? extends T> supplier;

        SuppliedThreadLocal(Supplier<? extends T> supplier) {
            this.supplier = Objects.requireNonNull(supplier);
        }

        @Override
        protected T initialValue() {
            return supplier.get();
        }
    }

    static class ThreadLocalMap {

        static class Entry extends WeakReference<ThreadLocal<?>> {
            Object value;

            Entry(ThreadLocal<?> k, Object v) {
                super(k);
                value = v;
            }
        }

        private static final int INITIAL_CAPACITY = 16;

        private Entry[] table;

        private int size = 0;

        private int threshold; // Default to 0

        private void setThreshold(int len) {
            threshold = len * 2 / 3;
        }

        private static int nextIndex(int i, int len) {
            return ((i + 1 < len) ? i + 1 : 0);
        }

        private static int prevIndex(int i, int len) {
            return ((i - 1 >= 0) ? i - 1 : len - 1);
        }

        ThreadLocalMap(ThreadLocal<?> firstKey, Object firstValue) {
            table = new Entry[INITIAL_CAPACITY];
            int i = firstKey.threadLocalHashCode & (INITIAL_CAPACITY - 1);
            table[i] = new Entry(firstKey, firstValue);
            size = 1;
            setThreshold(INITIAL_CAPACITY);
        }

        private ThreadLocalMap(ThreadLocalMap parentMap) {
            Entry[] parentTable = parentMap.table;
            int len = parentTable.length;
            setThreshold(len);
            table = new Entry[len];

            for (Entry e : parentTable) {
                if (e != null) {
                    @SuppressWarnings("unchecked")
                    ThreadLocal<Object> key = (ThreadLocal<Object>) e.get();
                    if (key != null) {
                        Object value = key.childValue(e.value);
                        Entry c = new Entry(key, value);
                        int h = key.threadLocalHashCode & (len - 1);
                        while (table[h] != null)
                            h = nextIndex(h, len);
                        table[h] = c;
                        size++;
                    }
                }
            }
        }

        private Entry getEntry(ThreadLocal<?> key) {
            int i = key.threadLocalHashCode & (table.length - 1);
            Entry e = table[i];
            if (e != null && e.refersTo(key))
                return e;
            else
                return getEntryAfterMiss(key, i, e);
        }

        private Entry getEntryAfterMiss(ThreadLocal<?> key, int i, Entry e) {
            Entry[] tab = table;
            int len = tab.length;

            while (e != null) {
                if (e.refersTo(key))
                    return e;
                if (e.refersTo(null))
                    expungeStaleEntry(i);
                else
                    i = nextIndex(i, len);
                e = tab[i];
            }
            return null;
        }

        private void set(ThreadLocal<?> key, Object value) {

            Entry[] tab = table;
            int len = tab.length;
            int i = key.threadLocalHashCode & (len-1);

            for (Entry e = tab[i];
                 e != null;
                 e = tab[i = nextIndex(i, len)]) {
                if (e.refersTo(key)) {
                    e.value = value;
                    return;
                }

                if (e.refersTo(null)) {
                    replaceStaleEntry(key, value, i);
                    return;
                }
            }

            tab[i] = new Entry(key, value);
            int sz = ++size;
            if (!cleanSomeSlots(i, sz) && sz >= threshold)
                rehash();
        }

        private void remove(ThreadLocal<?> key) {
            Entry[] tab = table;
            int len = tab.length;
            int i = key.threadLocalHashCode & (len-1);
            for (Entry e = tab[i];
                 e != null;
                 e = tab[i = nextIndex(i, len)]) {
                if (e.refersTo(key)) {
                    e.clear();
                    expungeStaleEntry(i);
                    return;
                }
            }
        }

        private void replaceStaleEntry(ThreadLocal<?> key, Object value,
                                       int staleSlot) {
            Entry[] tab = table;
            int len = tab.length;
            Entry e;

            int slotToExpunge = staleSlot;
            for (int i = prevIndex(staleSlot, len);
                 (e = tab[i]) != null;
                 i = prevIndex(i, len))
                if (e.refersTo(null))
                    slotToExpunge = i;

            for (int i = nextIndex(staleSlot, len);
                 (e = tab[i]) != null;
                 i = nextIndex(i, len)) {

                if (e.refersTo(key)) {
                    e.value = value;

                    tab[i] = tab[staleSlot];
                    tab[staleSlot] = e;

                    if (slotToExpunge == staleSlot)
                        slotToExpunge = i;
                    cleanSomeSlots(expungeStaleEntry(slotToExpunge), len);
                    return;
                }

                if (e.refersTo(null) && slotToExpunge == staleSlot)
                    slotToExpunge = i;
            }

            // If key not found, put new entry in stale slot
            tab[staleSlot].value = null;
            tab[staleSlot] = new Entry(key, value);

            // If there are any other stale entries in run, expunge them
            if (slotToExpunge != staleSlot)
                cleanSomeSlots(expungeStaleEntry(slotToExpunge), len);
        }

        private int expungeStaleEntry(int staleSlot) {
            Entry[] tab = table;
            int len = tab.length;

            // expunge entry at staleSlot
            tab[staleSlot].value = null;
            tab[staleSlot] = null;
            size--;

            // Rehash until we encounter null
            Entry e;
            int i;
            for (i = nextIndex(staleSlot, len);
                 (e = tab[i]) != null;
                 i = nextIndex(i, len)) {
                ThreadLocal<?> k = e.get();
                if (k == null) {
                    e.value = null;
                    tab[i] = null;
                    size--;
                } else {
                    int h = k.threadLocalHashCode & (len - 1);
                    if (h != i) {
                        tab[i] = null;

                        // Unlike Knuth 6.4 Algorithm R, we must scan until
                        // null because multiple entries could have been stale.
                        while (tab[h] != null)
                            h = nextIndex(h, len);
                        tab[h] = e;
                    }
                }
            }
            return i;
        }

        private boolean cleanSomeSlots(int i, int n) {
            boolean removed = false;
            Entry[] tab = table;
            int len = tab.length;
            do {
                i = nextIndex(i, len);
                Entry e = tab[i];
                if (e != null && e.refersTo(null)) {
                    n = len;
                    removed = true;
                    i = expungeStaleEntry(i);
                }
            } while ( (n >>>= 1) != 0);
            return removed;
        }

        private void rehash() {
            expungeStaleEntries();

            // Use lower threshold for doubling to avoid hysteresis
            if (size >= threshold - threshold / 4)
                resize();
        }

        private void resize() {
            Entry[] oldTab = table;
            int oldLen = oldTab.length;
            int newLen = oldLen * 2;
            Entry[] newTab = new Entry[newLen];
            int count = 0;

            for (Entry e : oldTab) {
                if (e != null) {
                    ThreadLocal<?> k = e.get();
                    if (k == null) {
                        e.value = null; // Help the GC
                    } else {
                        int h = k.threadLocalHashCode & (newLen - 1);
                        while (newTab[h] != null)
                            h = nextIndex(h, newLen);
                        newTab[h] = e;
                        count++;
                    }
                }
            }

            setThreshold(newLen);
            size = count;
            table = newTab;
        }

        private void expungeStaleEntries() {
            Entry[] tab = table;
            int len = tab.length;
            for (int j = 0; j < len; j++) {
                Entry e = tab[j];
                if (e != null && e.refersTo(null))
                    expungeStaleEntry(j);
            }
        }
    }
}
```

## ThreadLocal.java 주요 코드

```java
public class ThreadLocal<T> {
    ThreadLocalMap getMap(Thread t) {
        return t.threadLocals;
    }

    void createMap(Thread t, T firstValue) {
        t.threadLocals = new ThreadLocalMap(this, firstValue);
    }

    public void set(T value) {
        Thread t = Thread.currentThread();
        ThreadLocalMap map = getMap(t);
        if (map != null)
            map.set(this, value);
        else
            createMap(t, value);
    }

    public T get() {
        Thread t = Thread.currentThread();
        ThreadLocalMap map = getMap(t);
        if (map != null) {
            ThreadLocalMap.Entry e = map.getEntry(this);
            if (e != null) {
                @SuppressWarnings("unchecked")
                T result = (T)e.value;
                return result;
            }
        }
        return setInitialValue();
    }


    public void remove() {
        ThreadLocalMap m = getMap(Thread.currentThread());
        if (m != null)
            m.remove(this);
    }

    static class ThreadLocalMap {
        static class Entry extends WeakReference<ThreadLocal<?>> {
            Object value;

            Entry(ThreadLocal<?> k, Object v) {
                super(k);
                value = v;
            }
        }
    }

    ThreadLocalMap(ThreadLocal<?> firstKey, Object firstValue) {
        table = new Entry[INITIAL_CAPACITY];
        int i = firstKey.threadLocalHashCode & (INITIAL_CAPACITY - 1);
        table[i] = new Entry(firstKey, firstValue);
        size = 1;
        setThreshold(INITIAL_CAPACITY);
    }

    static class Entry extends WeakReference<ThreadLocal<?>> {
        Object value;
        Entry(ThreadLocal<?> k, Object v) {
            super(k);
            value = v;
        }
    }
}
```

## ThreadLocal.get()

코드를 천천히 살펴보면 구성이 보인다.

```java
    public T get() {
        Thread t = Thread.currentThread();
        ThreadLocalMap map = getMap(t);
        if (map != null) {
            ThreadLocalMap.Entry e = map.getEntry(this);
            if (e != null) {
                @SuppressWarnings("unchecked")
                T result = (T)e.value;
                return result;
            }
        }
        return setInitialValue();
    }
```

get()메서드의 코드를 살펴보면

- ThreadLocal.get() 메서드를 호출하면, `현재 동작 중인 쓰레드`를 가져온 후
- 해당 `현재 동작 중인 쓰레드`를 통해 `ThreadLocalMap`을 가져온다.
- 그 후 `ThreadLocalMap`이 존재한다면 해당 `Map의 Entry`를 가져온다.
- `Entry`가 존재한다면 해당 `Entry의` 존재하는 `value`를 뽑아와 반환한다.

우리가 호출하는 ThreadLocal.get() 메서드를 호출할 때의 데이터가 반환되는 흐름을 아래로 요약할 수 있다.

현재 쓰레드 -> 현재 쓰레드의 ThreadLocalMap -> 현재 쓰레드의 ThreadLocalMap의 Entry -> C현재 쓰레드의 ThreadLocalMap의 Entry의 value

그러면 여기서 의문이 든다.

- get()을 하기 전에 set()은 어떻게 실행되는건가?
- ThreadLocalMap은 어떻게 구성되어있을까?

## ThreadLocalMap.set()

```java
    public void set(T value) {
        Thread t = Thread.currentThread();
        ThreadLocalMap map = getMap(t);
        if (map != null)
            map.set(this, value);
        else
            createMap(t, value);
    }
```

set() 메서드의 코드를 살펴보면

- `현재 동작 중인 쓰레드`를 가져온다.
- 해당 `현재 동작 중인 쓰레드`를 통해 `ThreadLocalMap`을 가져온다.
- 그 후 `ThreadLocalMap`이 존재한다면 해당 `Map`에 값을 삽입한다.
  - 만약 `ThreadLocalMap`이 존재하지 않는다면 map을 생성한 후 삽입한다.

get()메서드와 흐름이 거의 비슷하다.

그럼 아직까지도 풀리지 않는 의문인 Map은 어떻게 생겼는지 알아보자.

## createMap

```java
    void createMap(Thread t, T firstValue) {
        t.threadLocals = new ThreadLocalMap(this, firstValue);
    }
```

createMap() 메서드를 살펴보면

- `Thread`와 `저장할 데이터`를 매개변수로 받는다.
- 그 후 `Thread 객체 내`의 존재하는 `threadLocals 이라는 인스턴스 변수`에 ThreadLocalMap을 삽입한다.

즉, 현재 요청을 처리하고 있는 쓰레드가 인스턴스 변수로 가지고 있는 ThreadLocalMap을 셋팅해주는 것이다.

그러면 ThreadLocalMap()의 생성자가 어떻게 동작하는지만 살펴보면 ThreadLocal이 어떻게 동작하는지 알 수 있을 것이다.

## ThreadLocalMap()

```java
    ThreadLocalMap(ThreadLocal<?> firstKey, Object firstValue) {
        table = new Entry[INITIAL_CAPACITY];
        int i = firstKey.threadLocalHashCode & (INITIAL_CAPACITY - 1);
        table[i] = new Entry(firstKey, firstValue);
        size = 1;
        setThreshold(INITIAL_CAPACITY);
    }
```

### ThreadLocalMap - 매개변수

- 첫 번째 매개변수로 ThreadLocal에 대한 제네릭을 받고 있는데, 특정 쓰레드만 접근할 수 있는 Key를 가리킨다.
- 두 번째 매개변수로 Object 타입의 value를 가지고 있는데, 특정 쓰레드가 쓰레드 로컬에 접근하여 얻을 수 있는 데이터를 가리킨다.

매개변수를 살펴봤으니 첫 번째 라인부터 천천히 살펴보겠다.

### ThreadLocalMap - table

```java
table = new Entry[INITIAL_CAPACITY];
```

table이라는 변수는 ThreadLocal의 인스턴스 변수로, Entry타입을 가지고 있으며 INITIAL_CAPACITY의 값은 16으로 지정되어있다.

Entry는 아래와 같이 구성되어있다.

```java
static class Entry extends WeakReference<ThreadLocal<?>> {
            /** The value associated with this ThreadLocal. */
            Object value;

            Entry(ThreadLocal<?> k, Object v) {
                super(k);
                value = v;
            }
        }
```

Entry는 HashMap처럼 Key, Value를 담기 위한 말 그대로의 Entry이다.

### ThreadLocalMap - 비트 연산

```java
int i = firstKey.threadLocalHashCode & (INITIAL_CAPACITY - 1);
```

Entry타입의 table을 셋팅한 후 ThreadLocal에 접근하기 위한 Key의 HashCode를 초기 용량인 16에 -1한 값과 AND 연산을한다.

초기 용량인 16에 대해서 앞으로 들어올 값들에 대해 해시 충돌을 막기위한 과정이다.

### ThreadLocalMap - table 값 할당

```java
table[i] = new Entry(firstKey, firstValue);
```

바로 직전 라인에서 계산한 결과를 인덱스로 table의 값을 셋팅한다.

마찬가지로 첫 번째 매개변수는 현재 쓰레드가 가지고 있는 Key, 두 번째 매개변수는 저장할 데이터를 가리킨다.

### ThreadLocalMap - Entry size 갱신

```java
size = 1;
```

이전까지의 로직으로 저장한 Entry의 크기에 따라 size를 변경해준다.

### ThreadLocalMap - threshold 갱신

```java
setThreshold(INITIAL_CAPACITY);
```

ThreadLocalMap의 threshold값을 설정한다.

threshold는 table 배열의 크기에 기반하여 충돌 및 재조정을 위한 임계치를 설정하는 데 사용된다.

```java

// The next size value at which to resize.
private int threshold; // Default to 0

/**
 * Set the resize threshold to maintain at worst a 2/3 load factor.
 */
private void setThreshold(int len) {
    threshold = len * 2 / 3;
}
```

크기를 조정하는 로직은 위와 같다.

# ThreaLocal의 구조 요약

ThreadLocal은 스프링만의 특별한 무엇인가가 아니라 Thread와 연계되는 객체였다.

ThradLocal이 구성되고 사용되는 기본 흐름으로는

- `현재 동작 중인 쓰레드`를 가져온 후
- 해당 `현재 동작 중인 쓰레드`의 `ThreadLocalMap`을 가져온다.
  - 여기서 ThraedLocalMap에 값을 할당할 수 도 있다.
- 그 후 `ThreadLocalMap`이 존재한다면 해당 `ThreadLocalMap의 Entry`를 가져온다.
- `ThreadLocalMap`의 `Entry`는 HashMap과 유사한 `Key, Value` 형태를 가지고 있다.
  - ThreadLocalMap의 Entry는 `현재 쓰레드의 Key의 HashCode`와 `초기 Map 용량과 AND 연산`을 수행하여 해시 충돌을 방지한다.
- ThreadLocal에 접근하여 값을 사용하기 위해서는 해당 `Entry의` 존재하는 `value`를 뽑아와 반환한다.

# ThreadLocal의 사용처

## Spring Security

ThreadLocal은 Spring Security에 자주 사용된다.

![](https://img1.daumcdn.net/thumb/R1024x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FbeDENY%2FbtrBs0cquNc%2FPkwRQzgyzhoy1ecQrlQOJk%2Fimg.png)

## @Transcation

```java
package org.springframework.transaction.support;

public abstract class TransactionSynchronizationManager{}
```

[Spring.io - TransactionSynchronizationManager](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/transaction/support/TransactionSynchronizationManager.html)

위 패키지에 해당하는 TransactionSynchronizationManager는 ThreadLocal을 사용하여 트랜잭션간의 동기화 관리자를 저장한다.

스프링에서 제공하는 Transaction Manager는 크게 두 가지 역할을 갖고 있다.

- 트랜잭션 추상화
  - PlatformTransactionManager 인터페이스를 사용 방식을 말한다.
- 리소스 동기화
  - 트랜잭션을 유지하려면 트랜잭션의 시작부터 끝까지 같은 데이터베이스 커넥션을 유지해야한다.
    - 결국 같은 커넥션을 동기화하기 위해서 파라미터로 Connection 을 전달하는 방법이 있다.
    - 하지만 이 방법은 지저분해지고 중복된 코드를 생성할 수 밖에 없다.

여기서 리소스 동기화를 위해 스프링에서 제공하는 것이 TransactionSynchronizationManager이다.

이 때 PlatformTransactionManager가 TransactionSynchronizationManager에서 보관하는 커넥션을 가져와서 사용하는 방식이라고 이해하면 된다.

TransactionSynchronizationManager는 내부적으로 ThreadLocal를 사용하기 때문에 멀티쓰레드 상황에 Connection을 동기화 할 수 있다.

따라서 Connection이 필요하면 TransactionSynchronizationManager를 통해 Connection을 가져오면 되고, 이전처럼 파라미터로 Connection을 전달할 필요가 없어진다.

또한, Thread를 기준으로 리소스 및 트랜잭션 동기화를 관리해줄 수 있게된다.

### 동작 예시

    트랜잭션 시작:
        쓰레드 A가 트랜잭션을 시작
        TransactionSynchronizationManager는 현재 쓰레드에 대한 상태를 유지

    DB 커넥션 확보:
        쓰레드 A가 DB 커넥션을 확보해야 할 때, 일반적으로 DataSource를 통해 커넥션을 얻는다.
        TransactionSynchronizationManager는 현재 쓰레드에 대한 ThreadLocal에 커넥션을 저장합니다.

    다른 쓰레드에서 DB 커넥션 요청:
        동일한 트랜잭션 내에서 다른 쓰레드 B가 DB 커넥션을 요청합니다.
        TransactionSynchronizationManager는 현재 쓰레드 B에 대한 ThreadLocal에서 커넥션을 찾지 못한다.

    커넥션 공유:
        TransactionSynchronizationManager는 현재 쓰레드 A의 ThreadLocal에 저장된 커넥션을 가져와서 쓰레드 B에게 제공한다.
        이를 통해 동일한 트랜잭션 내에서 쓰레드 간에 DB 커넥션을 공유한다.

    트랜잭션 완료:
        트랜잭션이 완료되면 TransactionSynchronizationManager는 현재 쓰레드의 ThreadLocal에 저장된 커넥션을 해제 및 ThreadLocal Clear
        DB 커넥션은 반환 혹은 닫힌다.

## RequestContextHolder

```java
public abstract class RequestContextHolder {

    private static final boolean jsfPresent =
            ClassUtils.isPresent("javax.faces.context.FacesContext", RequestContextHolder.class.getClassLoader());

    private static final ThreadLocal<RequestAttributes> requestAttributesHolder =
            new NamedThreadLocal<>("Request attributes");

    private static final ThreadLocal<RequestAttributes> inheritableRequestAttributesHolder =
            new NamedInheritableThreadLocal<>("Request context");

    ...
}
```

현재 실행 중인 쓰레드에서 어떤 계층에서든 ServletRequest에 접근할 수 있도록 관리하는 유틸 클래스이다.

ThreadLocal기반으로 구성되어있기 때문에 멀티 쓰레드 환경에서 다른 쓰레드에서도 특정 ServletRequest를 사용할 수 있도록 도와준다.

즉, 한 요청을 처리하기 위해 여러 쓰레드가 붙잡고 있을 때 쓰레드 각 A, B, C 간의 ServletRequset를 공유하여 사용할 수 있는 것이다.

# ThreadLocal 사용 시 주의점

ThreadLocal을 사용하고 그 마지막 시점에 도달했을 때는 쓰레드 로컬 내 정보를 지워야한다.

```java
ThreadLocal.remove()
```

정보를 지우는 방법은 위 remove()메서드를 호출하면 된다.

A, B라는 쓰레드가 있을 때 만약 A의 쓰레드 로컬의 정보를 지우지 않는다면 B쓰레드에서 이미 모든 처리가 끝난 A 쓰레드 로컬에 접근하게될 가능성이 생겨 자칫 A의 정보를 획득하거나 잘못된 요청을 수행하는 데이터 위/변조 문제가 발생할 수 있기 때문이다.
