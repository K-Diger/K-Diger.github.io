---

title: Garbage Collector(GC)
author: 김도현
date: 2022-08-14
categories: [Java, GC]
tags: [Java, GC]
layout: post
toc: true
math: true
mermaid: true

---

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

추가적으로, Java 8 에 들어오면서, Permanent Generation 이 사라지고 Metaspace 영역이 생겼다.

Permanent Generation 은 JVM 에 의해서 Heap 영역의 메모리 크기가 강제되던 영역이였다.

하지만 Metaspace 가 Native 메모리 영역에 배치되면서, 운영체제가 자동으로 그 크기를 조절할 수 있게 되고, Heap 에서 사용할 수 있는 메모리의 크기가 늘어나게 됐다.

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
