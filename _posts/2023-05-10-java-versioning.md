---

title: Java 8 vs Java 11 vs Java 17
author: 김도현
date: 2023-05-10
categories: [Spring, Toby]
tags: [Spring, Toby]
layout: post
toc: true
math: true
mermaid: true

---

# 참고 자료

[Java 8 Documentation by Oracle Page](https://www.oracle.com/java/technologies/javase/8-whats-new.html)

[Java 11 Documentation by Baeldung](https://www.baeldung.com/java-11-new-features)

[Java 17 Documentation by Oracle Blog](https://blogs.oracle.com/java/post/announcing-java17)

---

# Java 8

## 람다

함수형 프로그래밍을 지원하기 위해 람다 표현식이 도입되었습니다.

이를 사용하여 익명 함수를 작성하고 전달할 수 있으며, 코드의 간결성과 가독성을 향상시킬 수 있는 방법이 추가됨

## 스트림 API

데이터 처리를 위한 Stream API 추가.

이를 통해 컬렉션, 배열 등의 데이터를 처리하고 조작할 수 있으며, 병렬 처리를 지원하여 성능을 향상 시킬 수 있음

## 날짜 및 시간 API

Java 8에서는 java.time 패키지를 통해 새로운 날짜와 시간 API가 제공됩니다.

이 API는 날짜, 시간, 기간 등을 표현하고 조작하는 데 유용합니다.

`LocalDateTime`

## 기본 메서드(Default Methods)

인터페이스에서도 메서드의 구현을 제공할 수 있게 되었음

기존에 인터페이스를 구현한 클래스에 새로운 메서드가 추가되더라도 해당 클래스를 수정하지 않고도 기본 구현을 사용할 수 있음

## 병렬 처리

병렬 스트림이라는 개념이 도입되어 멀티코어 시스템에서 병렬로 작업을 처리할 수 있게 됨

이를 통해 빠른 처리와 성능 향상을 얻을 수 있음

## 반복자 개선

반복자를 사용하는 방식이 개선되어 forEach() 메서드를 통해 반복 작업을 수행할 수 있게 되었고 병렬 처리도 가능.

## 타입 어노테이션(Type Annotations)

타입 어노테이션을 사용하여 코드의 가독성을 높일 수 있고, 정적 분석 및 코드 검증을 위한 정보를 추가할 수 있음

타입 어노테이션은 주로 `변수 선언`, `반환 타입`, `매개변수` 등의 `타입에 추가 정보`를 제공하는 데 사용됨

`@NotNull`

## 스크립트 엔진

Java 8에서는 JavaScript, Ruby, Python 등의 스크립트 엔진을 사용하여 Java 코드에 스크립팅 기능을 추가할 수 있게됨

## 그 외

이 외에도 성능 개선, 보안 강화, 개선된 컴파일러 등 여러 가지 기능과 개선 사항이 추가됨

---

# Java 11

## 문자열 메서드 추가

`isBlank`, `lines`, `strip`, `stripLeading`, `stripTrailing` 및 `repeat`와 같은 새로운 메서드들이 추가됨

여러 줄 문자열에서 공백이 아닌 제거된 줄을 추출하기 위해 새로운 메서드를 어떻게 사용할 수 있는지 살펴보겠습니다.

## 파일 메서드 추가

파일 입출력에 관한 편의 메서드 추가

```java
Path filePath = Files.writeString(Files.createTempFile(tempDir, "demo", ".txt"), "Sample text");
String fileContent = Files.readString(filePath);
assertThat(fileContent).isEqualTo("Sample text");
```

## Collection to Array

java.util.Collection 인터페이스에는 IntFunction 인수를 사용하는 toArray 메서드가 추가됨

List -> Array의 변환이 간편해짐

```java
List sampleList = Arrays.asList("Java", "Kotlin");
String[] sampleArray = sampleList.toArray(String[]::new);
assertThat(sampleArray).containsExactly("Java", "Kotlin");
```

## Predicate not 메서드 추가

Predicate Inteface에 정적 not 메소드가 추가됨

```java
List<String> sampleList = Arrays.asList("Java", "\n \n", "Kotlin", " ");
List withoutBlanks = sampleList.stream()
  .filter(Predicate.not(String::isBlank))
  .collect(Collectors.toList());
assertThat(withoutBlanks).containsExactly("Java", "Kotlin");
```

## var 키워드 추가

람다 매개변수에서 로컬 변수 구문인 var 키워드 추가됨

```java
List<String> sampleList = Arrays.asList("Java", "Kotlin");
String resultString = sampleList.stream()
  .map((@Nonnull var x) -> x.toUpperCase())
  .collect(Collectors.joining(", "));
assertThat(resultString).isEqualTo("JAVA, KOTLIN");
```

## 새로운 HTTP Client 추가

Java 9에서 도입된 java.net.http 패키지의 새로운 HTTP 클라이언트가 Java 11의 표준 기능이 됨

새 HTTP API는 전반적인 성능을 개선하고 HTTP/1.1 및 HTTP/2를 모두 지원함

```java
HttpClient httpClient = HttpClient.newBuilder()
  .version(HttpClient.Version.HTTP_2)
  .connectTimeout(Duration.ofSeconds(20))
  .build();
HttpRequest httpRequest = HttpRequest.newBuilder()
  .GET()
  .uri(URI.create("http://localhost:" + port))
  .build();
HttpResponse httpResponse = httpClient.send(httpRequest, HttpResponse.BodyHandlers.ofString());
assertThat(httpResponse.body()).isEqualTo("Hello from the server!");
```

## Java 파일 실행의 간소화

javac를 사용하여 Java 소스 파일을 컴파일할 필요없이 실행 가능

이전 버전
```shell
$ javac HelloWorld.java
$ java HelloWorld
Hello Java 8!
```

신규 버전
```shell
$ java HelloWorld
Hello Java 11!
```

## A No-Op Garbage Collector 추가

Epsilon이라는 새로운 가비지 수집기가 Java 11에서 실험판으로 열렸음

메모리를 할당하지만 실제로 가비지를 수집하지 않기 때문에 No-Op(작업 없음)라고 한다.

따라서 Epsilon은 메모리 부족 오류를 시뮬레이션하는 데 적용할 수 있다.

분명히 Epsilon은 일반적인 프로덕션 Java 애플리케이션에 적합하지 않지만 유용할 수 있는 몇 가지 특정 사용 사례가 있습니다.

`성능 시험`, `메모리 부하 테스트`, `VM 인터페이스 테스트 및 수명이 매우 짧은 작업`

이 GC를 사용하려면

```shell
-XX:+UnlockExperimentalVMOptions -XX:+UseEpsilonGC
```
플래그를 사용하면 된다.

---

# Java 17

## Sealed class 추가

Sealed classe와 interface는 다른 클래스나 인터페이스가 확장하거나 구현할 수 없도록 범위를 제한한다.

### Sealed class vs Final class
`확장 가능성`

- Final 클래스
  - Final 클래스는 어느 곳에서도 상속할 수 없다.

- Sealed 클래스
  - Sealed 클래스는 제한된 범위 내에서만 상속이 가능하다. 특정 클래스나 인터페이스 목록을 permits 절로 지정하여, 해당 클래스나 인터페이스만이 sealed 클래스를 상속하거나 구현할 수 있다.

`유연성`

- Final 클래스
  - Final 클래스는 상속을 금지하므로, 하위 클래스에서 상속 받은 메서드를 오버라이드 할 수 없다.

- Sealed 클래스
  - Sealed 클래스는 permits 절에 선언된 클래스들에 대한 상속을 허용하지만, 해당 클래스들 내에서는 상속을 계속할 수 있다.
  - 즉, Sealed 클래스의 하위 클래스는 다른 메서드를 추가하거나, 상속 계층을 더 확장할 수 있다.

`설계 목적`

- Final 클래스
  - Final 클래스는 보안 상의 이유로 상속을 막거나, 불변성을 보장하고자 할 때 사용된다. 다른 클래스가 해당 클래스를 상속하거나 변경하지 못하도록 하여 의도한 동작을 보장한다.

- Sealed 클래스
  - Sealed 클래스는 클래스 계층 구조의 일부를 통제하고 특정 클래스나 인터페이스를 상속 또는 구현할 수 있는 클래스의 범위를 제한함으로써, 더 명확하고 제한된 클래스 계층을 구성할 수 있게된다.

##
