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


# 문제 상황

프로젝트를 진행하면서 크롤러를 돌리는 중 아래와 같은 에러가 뜨고 있었다.

![](https://github.com/K-Diger/K-Diger.github.io/assets/60564431/181f8d12-d4b9-4479-9c82-c4f64ee3231a)

크롤링하면서 OOM이 터졌다는 것 같은데 왜 터진지 알아보자..

# 서버 스펙

기존 서버 스펙은 NCP의 Compact옵션인 (CPU: 2CORE, Memory : 2GB)를 사용하고 있었다.

서버 내 Java 버전은 OpenJDK 17버전을 사용하는 것으로 아무런 튜닝 옵션을 주지 않고 애플리케이션을 실행하고 있었기 때문에 다음과 같은 JVM 스펙이 동작하고 있었다.

[JDK 17](https://docs.oracle.com/javase/specs/jvms/se17/html/jvms-2.html)

![image](https://github.com/K-Diger/K-Diger.github.io/assets/60564431/768fd0fa-5052-48fc-acc6-d70a99a8bc5d)

현재 나의 서버에서 구동되는 JVM 정보는 위 그림과 같다. (위 사진은 2CORE, 4GB Memory로 대체하였다.)

기본값으로 구동되는 옵션의 기준은 아래 링크에 설명되어있다.

[Oracle Docs](https://www.oracle.com/java/technologies/ergonomics5.html)

서버급 컴퓨터에서는 기본적으로 다음이 선택됩니다.

- 초기 힙 크기 : 물리적 메모리의 1/64 --> 8.388608MB

- 최대 힙 크기 : 물리적 메모리의 1/4 --> 1GB


    HotSpot VM:
        기본 GC(Garbage Collector): G1(G1GC)
        특징: 멀티 스레드, 동적 컴파일, 메모리 관리, GC 등 다양한 최적화 기능을 제공합니다.

    기본 메모리 설정:
        초기 힙 크기(Initial Heap Size): 1/64th of physical memory up to 1 GB
        최대 힙 크기(Maximum Heap Size): 1/4th of physical memory up to 1 GB
        자세한 메모리 설정은 "-Xms" 및 "-Xmx" 플래그를 사용하여 조정할 수 있습니다.

    기본 GC 설정:
        G1GC(G1 Garbage Collector)가 기본 GC로 사용됩니다.
        G1GC는 다중 스레드로 동작하며, 자동으로 힙 영역을 분할하고 GC를 수행합니다.
        G1GC의 GC 튜닝 옵션은 "-XX:+UseG1GC" 플래그를 사용하여 활성화할 수 있습니다.
