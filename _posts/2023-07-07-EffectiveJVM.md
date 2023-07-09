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

현재 나의 서버에서 구동되는 JVM 정보는 위 그림과 같다. (위 사진은 최근 스펙업을 한 2CORE, 4GB Memory로 대체하였다.)

기본값으로 구동되는 옵션의 기준은 아래 링크에 설명되어있다.

[Oracle Docs](https://www.oracle.com/java/technologies/ergonomics5.html)


- 초기 힙 크기 : 물리적 메모리의 1/64 --> 8.388608MB

- 최대 힙 크기 : 물리적 메모리의 1/4 --> 1GB

여튼 기본값을 지정된 초기 힙 크기를 가지고 크롤러를 돌릴 때 문제가 발생한다고 하니까 이를 해결해보자.


# 서버, JVM 튜닝

우선 JVM의 절대적인 Heap 공간이 모자라다는 것이 문제다. 모니터링을 통해 내린 결론은 크롤러가 요구하는 Memory Size는 약 2GB 언저리이다.

따라서 애플리케이션 코드를 더 최적화 하지 않는 이상 최대 메모리를 2GB로 돌리는 것은 불가능할 것이다.

## 서버의 수직적 확장

서버의 스펙을 올린다. 기본 2Core 2GB Memory에서 2Core 4GB Memory로 스펙을 올렸다.

## JVM 튜닝

서버 스펙을 올려 메모리를 넉넉하게 발급했음에도 불구하고 아래와 같은 에러가 뜨고있었다.

```text
Exception: java.lang.OutOfMemoryError thrown from the UncaughtExceptionHandler in thread "http-nio-8081-Poller"
Exception in thread "HikariPool-1 housekeeper" java.lang.OutOfMemoryError: Java heap space
Exception in thread "Catalina-utility-2" java.lang.OutOfMemoryError: Java heap space
```

여전히 Heap 공간이 모자라다고 투정을 부리고 있는 것인데, 이 역시 JVM의 기본 값으로 Heap 사이즈를 설정했기 때문에 그 사이즈로도 감당이 안된다는 것인데

스펙업을 통해 남는 메모리를 조금 더 할당하도록 튜닝해야한다.

### 튜닝 명령어 - Heap Size 설정, GC 지정

```shell
java -Xms256m -Xmx2048m -XX:+UseParallelGC
```

위 명령어는 JVM 옵션을 커스텀하여 실행하도록 하는 것이다.

- **-Xms256m**은 최소 힙 사이즈를 나타내는 것으로 256M를 지정했다.

- **-Xmx2048m**은 최대 힙 사이즈를 나타내는 것으로 2048M(2GB)를 지정했다.

- **-XX:+UseParallelGC**은 GC를 지정해 주는 것인데, ParallelGC를 지정했다.

## GC옵션 더 알아보기

위에서 튜닝한 옵션으로 ParallelGC를 사용했다고 했다. 그러면 GC의 종류는 어떤 것들이 있고 왜 ParallelGC를 사용했는지 알아보자.

[JAVA 9 DOCS](https://docs.oracle.com/javase/9/gctuning/available-collectors.htm#JSGCT-GUID-C7B19628-27BA-4945-9004-EC0F08C76003)

위 문서에 따르면 GC의 종류는 다음으로 나뉜다.

- Serial
- Parallel
- CMS
- G1

### Serial GC (-XX:+UseSerialGC)

Serial GC는 **단일 스레드**를 사용하여 모든 가비지 수집 작업을 수행한다.

따라서 **스레드 간에 통신으로 인한 오버헤드가 없기** 때문에 효율적이다.

작은 데이터 세트(최대 약 100MB)가 있는 응용 프로그램의 다중 프로세서에서 유용할 수 있지만 멀티 코어의 하드웨어를 활용할 수 없기 때문에 **단일 코어 시스템에 가장 적합**하다.


### Parallel GC (-XX:+UseParallelGC)

Parallel GC는 Serial GC와 유사하지만, Serial GC와 Parallel GC의 차이점은 Parallel GC에는 GC 속도를 높이는 데 사용되는 **멀티 스레드**가 있다.

Parallel GC는 멀티 코어 또는 멀티 쓰레드 하드웨어에서 실행되는 중대형 데이터 세트가 포함된 응용 프로그램에서 사용하기 적합하다.

또한 Parallel GC가 가지고 있는 **Parallel Compaction** 이라는 기능은 기본 값으로 제공되는데 GC가 동작을 병렬로 수행할 수 있도록 하는 기능이다.

**Parallel Compaction**이 없으면 단일 쓰레드를 사용하여 GC가 수행되므로 확장성이 크게 제한될 수 있다.

### CMS<Concurrent Mark Sweep> GC, G1<Garbage-First> (-XX:+UseG1GC or -XX:+UseConcMarkSweepGC)

- CMS GC는 Java 9 이후로 사용되지 않기 때문에 부연 설명은 제외하겠다.

G1 GC는 대용량 메모리가 있는 멀티 코어 시스템용이다.

높은 처리량과 가장 짧은 Stop-The-World를 가지고 있다.

G1은 특정 하드웨어 및 운영 체제 구성에서 기본값으로 선택된다. 혹은 다음 옵션으로 사용 가능하다. (-XX:+UseG1GC)

---

## GC 선택에 대한 결론

GC 종류를 알아봤으니 그 다음으로 어떤 GC가 어떨 때 적합한지 정리된 글을 보자면

- 애플리케이션에 다소 엄격한 Stop-The-Wolrd에 대한 요구 사항이 있는 경우가 아니면 먼저 애플리케이션을 실행하고 VM이 수집기를 선택하도록 허용해라.

- 필요한 경우 힙 크기를 조정하여 성능을 향상시켜라.
  - 성능이 여전히 목표를 충족하지 못하는 경우 GC를 직접 선택해라.

- **소규모 애플리케이션**일 경우에는(최대 약 100MB) **Serial GC**를 선택해라. (-XX:+UseSerialGC)
  - 애플리케이션이 **단일 프로세서**에서 실행되고 **Stop-The-World에 대한 제약이 없는 경우** **Serial GC**를 선택해라.

- 애플리케이션 **성능이 첫 번째 우선 순위**이고 **Stop-The-World에 대한 요구 사항이 없거나 1초 이상의 일시 중지가 허용**되는 경우 **기본 값**으로 지정된 GC를 사용하거나 **Parallel GC**를 선택해라. (-XX:+UseParallelGC)

- **응답 시간이 전체 처리량보다 중요**하고 **Stop-The-World 시간이 약 1초 미만**으로 유지해야 하는 경우는 **G1 GC or CMS GC**를 사용해라. (-XX:+UseG1GC or -XX:+UseConcMarkSweepGC)

위 결론에 따라서 나는 Parallel GC를 선택했다. 시간당 한 번씩 로직을 수행하기 때문에 Stop-The-World에 대한 제약사항은 우선수위가 가장 낮으나

애플리케이션 자체의 성능을 극대화 하는게 더 우선이기 때문이다.

메모리 부족이 발생한 상황을 다시 돌아보면 초기 서버에는 괜찮았지만 시간이 지날 수록 쓰레기값들이 누적되면서 메모리가 터져버린 것으로 예상되었다.

따라서 아래와 같은 명령어를 서버 시스템 내의 crontab으로 설정하여 Cache Memory를 정리해 주는 방법을 추가했다.

```shell
0 * * * * sync && echo 3 > /proc/sys/vm/drop_caches
```
