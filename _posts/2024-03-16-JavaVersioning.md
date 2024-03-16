---

title: Java LTS 버전별 주요 기능 (8, 11, 17, 21)
date: 2024-03-16
categories: [Java]
tags: [Java]
layout: post
toc: true
math: true
mermaid: true

---

# JRE? JDK?

![JDK](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FMk8as%2FbtqKsoI4jA3%2F5VnitlEi9njb43qk9kHuik%2Fimg.webp)

JRE (Java Runtime Environment) 의 약자로 JVM, Java Class Library, Java Command, JDBC 등, Java 프로그램을 실행할 수 있는 기본 패키징을 가리킨다.

JDK (Java Development Kit) 의 약자로, JRE 를 포함하여, 컴파일러인 javac, javadoc, jar 등 실제 개발환경에 필요한 요소들을 패키징 한 것이다.

- JRE
    - 빌드가 완료된 Java 기반의 프로그램을 실행 할 수 있는 환경
- JDK
    - Java 기반의 프로그램을 작성부터 빌드 후 실행까지 할 수 있는 환경

JDK 를 다운 받다 보면 `Java SE` 8 ... 9, `Java EE` 8 ... 9 뭐 이런 용어가 있다 이것들도 알아봐보자.

---

## Java SE (Java Standard Edition)

말 그대로, 가장 보편적으로 쓰이는 Java 버전을 뜻한다.

JDK 8, 11 등 보편적으로 `JDK(Java)` 버전을 가리킬 때 이 `SE 버전`을 뜻하는 것이다.

---

## Java EE (Java Enterprise Edition)

서버측 개발을 위한 플랫폼으로 이 역시 `SE`를 기반으로 하되, 부가적인 기능이 추가된 것이다.

---

# JDK 8

## 1. 인터페이스 기본/정적 메서드 추가

### 명세 측

```java
public interface Vehicles {

    static String producer() {
        return "N&F Vehicles";
    }

    default String getOverview() {
        return "ATV made by " + producer();
    }

}
```

### 사용 측

```java
public class Client {
    Vehicle vehicle = new VehicleImpl();
    String overview = vehicle.getOverview();
}
```

---

## 2. Static Method, Method 참조 방식 개선

```java
public class StaticMethodReference {

    public static void main(String[] args) {
        // 기존 람다 표현식으로 정적 메서드 호출
        boolean oldFeature = list.stream().anyMatch(u -> User.isRealUser(u));

        // 새롭게 추가된 표현식
        boolean newFeature = list.stream().anyMatch(User::isRealUser);
    }
}
```

위와 같이 기존 Stream/Lambda 내에서 객체의 static메서드를 호출하는 부분을 람다를 표현식을 제거하여 사용할 수 있게 됐다.

```java
public class StaticMethodReference {

    public static void main(String[] args) {
        User user = new User();

        // 인스턴스 메서드 호출
        boolean isLegalName = list.stream().anyMatch(user::isLegalName);

        // 생성자 호출
        Stream<User> stream = list.stream().map(User::new);
    }
}
```

또한 위와 같이 객체 자체의 메서드를 호출하는 구문도 `->`와 같은 람다 표현식을 제거하고 사용할 수 있게 됐다.

공식적인 문법 형태는 `ContainingClass::methodName`이다.

---

## 3. Optional<T>

Null을 다룰 수 있는 래퍼 객체인 Optional이 추가되었다.

```java
public static void main(String[] args) {
    Optional<String> optional = Optional.empty();

    String str = "value";
    Optional<String> optional = Optional.of(str);

    Optional<String> optional = Optional.ofNullable(getString());
}
```

### Java 8 이전 Optional 기능을 대체할 구문

```java
public static void main(String[] args) {
    // #1
    List<String> list = getList();
    List<String> listOpt = list != null ? list : new ArrayList<>();
    // #1

    // #2
    User user = getUser();
    if (user != null) {
        Address address = user.getAddress();
        if (address != null) {
            String street = address.getStreet();
            if (street != null) {
                return street;
            }
        }
    }
    return "not specified";
    // #2

    // #3
    String value = null;
    String result = "";
    try {
        result = value.toUpperCase();
    } catch (NullPointerException exception) {
        throw new CustomException();
    }
    // #3
}
```

```java
public static void main(String[] args) {
    // #1
    List<String> listOpt = getList().orElseGet(() -> new ArrayList<>());
    // #1

    // #2
    Optional<User> user = Optional.ofNullable(getUser());
    String result = user
        .map(User::getAddress)
        .map(Address::getStreet)
        .orElse("not specified");
    // #2

    // #3
    String value = null;
    Optional<String> valueOpt = Optional.ofNullable(value);
    String result = valueOpt.orElseThrow(CustomException::new).toUpperCase();
    // #3
}

```

---

## 4. Stream

### 기본 사용법

```java
public static void main(String[] args) {
    String[] arr = new String[]{"a", "b", "c"};
    Stream<String> stream = Arrays.stream(arr);
    stream = Stream.of("a", "b", "c");
}
```

### 멀티 쓰레드 환경에서의 스트림 사용법

일반적인 `stream()`구문이 아닌 `parallelStream()`구문으로 멀티 쓰레드 환경에서 안전하게 스트림을 사용할 수 있다.

```java
public static void main(String[] args) {
    list.parallelStream().forEach(element -> doWork(element));
}
```

### 스트림 주요 연산

스트림은 기본적으로 `선언`, `중간 연산`, `종단 연산` 이 세 가지로 나뉜다.

스트림이 제공하는 `중간 연산`을 잘 활용하면 반복문을 생성하지 않고도 컬렉션을 순회하는 등 코드의 가독성을 확보할 수 있다.

#### 스트림 중간 연산 - Iterating

```java
public static void main(String[] args) {
    // 스트림 사용 전
    for (String string : list) {
        if (string.contains("a")) {
            return true;
        }
    }

    // 스트림 사용 후
    boolean isExist = list.stream().anyMatch(element -> element.contains("a"));
}
```

#### 스트림 중간 연산 - Filtering

```java
public static void main(String[] args) {
    List<String> list = new ArrayList<>();
    list.add("One");
    list.add("OneAndOnly");
    list.add("Derek");
    list.add("Change");
    list.add("factory");
    list.add("justBefore");
    list.add("Italy");
    list.add("Italy");
    list.add("Thursday");
    list.add("");
    list.add("");

    Stream<String> stream = list.stream().filter(element -> element.contains("d"));
}
```

위와 같은 `filter`연산으로 조건절에 부합하는 내용만 뽑아오는 기능을 적용할 수 있다.

#### 스트림 종단 연산 - Mapping

```java
public static void main(String[] args) {
    List<Detail> details = new ArrayList<>();
    details.add(new Detail());
    Stream<String> stream = details.stream()
        .flatMap(detail -> detail.getParts().stream());
}
```

`flatMap`메서드를 사용하면 PARTS 필드의 모든 요소가 추출되어 새로운 결과 스트림에 추가된다.

`flatMap`메서드와 `map`메서드를 사용할 수 있는데 두 차이점은 아래와 같다.

```java
public static void main(String[] args) {
    // map()
    List<String> words = Arrays.asList("Java", "Stream", "Map");
    List<String> upperWords = words.stream()
        .map(String::toUpperCase)
        .collect(Collectors.toList());
    System.out.println(upperWords);
    // 결과 : [JAVA, STREAM, MAP]

    // flatMap()
    List<List<String>> listOfLists = Arrays.asList(
        Arrays.asList("Java", "is"),
        Arrays.asList("super", "fun")
    );
    List<String> flatList = listOfLists.stream()
        .flatMap(List::stream)
        .collect(Collectors.toList());
    System.out.println(flatList);
    // 결과 : [Java, is, super, fun]
}
```

`flatMap`은 중복된 스트림을 1차원으로 평면화 시키는 메서드이다.

`flatMap 내부의 Arrays::stream`은 **배열을 스트림으로 변환해주는 메서드 참조 표현**이다. `flatMap`을 사용하면 각각의 String 리스트를 스트림으로 만드는 것이 아니라, 한 단계 더 깊은 깊이의 모든 요소를 하나의 스트림으로 합친다.


#### 스트림 종단 연산 - Matching

```java
public static void main(String[] args) {
    boolean isValid = list.stream().anyMatch(element -> element.contains("h")); // true
    boolean isValidOne = list.stream().allMatch(element -> element.contains("h")); // false
    boolean isValidTwo = list.stream().noneMatch(element -> element.contains("h")); // false

    Stream.empty().allMatch(Objects::nonNull); // true
    Stream.empty().anyMatch(Objects::nonNull); // false
}
```

---

# JDK 11

## 1. G1 GC가 기본 값으로 등록됨

LTS 기준 **JDK 8**까지는 **Parallel GC**가 **기본값**이었다.

JDK7에 첫 등장한 **G1 GC**가 **기본값**으로 등록된 버전이다.

하지만 막상 OpenJDK 11을 쓴다한들 OS에 종속적으로 GC가 선택된다. 예를들어 AWS EC2프리티어 스펙(1Core, Memory 1GB)에서는 SerialGC가 기본으로 적용된다.

---

## 2. 문자열 메서드 추가

`isBlank`, `lines`, `strip`, `stripLeading`, `stripTrailing` 및 `repeat`와 같은 새로운 메서드들이 추가됨

---

## 3. Collection.toArray()

Collection에 `toArray()`가 추가되어 List -> Array의 변환이 간편해졌다.

```java
public static void main(String[] args) {
    List<String> sampleList = Arrays.asList("Java", "Kotlin");
    String[] sampleArray = sampleList.toArray(String[]::new);

    assertThat(sampleArray).containsExactly("Java", "Kotlin");
}
```

---

## 4. var 키워드 추가

람다 매개변수에서 로컬 변수 구문인 var 키워드 추가됐다.

```java
public static void main(String[] args) {
    List<String> sampleList = Arrays.asList("Java", "Kotlin");

    String resultString = sampleList.stream()
        .map((@Nonnull var x) -> x.toUpperCase())
        .collect(Collectors.joining(", "));

    assertThat(resultString).isEqualTo("JAVA, KOTLIN");
}
```

---

## 5. 새로운 HTTP Client 추가

Java 9에서 도입된 java.net.http 패키지의 새로운 HTTP 클라이언트가 Java 11의 표준 기능이 됐다.

새 HTTP API는 전반적인 성능을 개선하고 HTTP/1.1 및 HTTP/2를 모두 지원한다.

```java
public static void main(String[] args) {
    HttpClient httpClient = HttpClient.newBuilder()
        .version(HttpClient.Version.HTTP_2)
        .connectTimeout(Duration.ofSeconds(20))
        .build();

    HttpRequest httpRequest=HttpRequest.newBuilder()
        .GET()
        .uri(URI.create("http://localhost:" + port))
        .build();

    HttpResponse httpResponse = httpClient.send(httpRequest, HttpResponse.BodyHandlers.ofString());

    assertThat(httpResponse.body()).isEqualTo("Hello from the server!");
}
```

---

## 6. Java 파일 실행의 간소화

javac를 사용하여 Java 소스 파일을 컴파일할 필요없이 실행 가능해졌다.

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

---

## 7. A No-Op Garbage Collector 추가

Epsilon이라는 새로운 가비지 수집기가 Java 11에서 실험판으로 열렸다.

메모리를 할당하지만 실제로 가비지를 수집하지 않기 때문에 No-Op(작업 없음)라고 한다.

따라서 Epsilon은 메모리 부족 오류를 시뮬레이션하는 데 적용할 수 있다.

분명히 Epsilon은 일반적인 프로덕션 Java 애플리케이션에 적합하지 않지만 유용할 수 있는 몇 가지 특정 사용 사례가 있다.

`성능 시험`, `메모리 부하 테스트`, `VM 인터페이스 테스트 및 수명이 매우 짧은 작업`

이 GC를 사용하려면

```shell
-XX:+UnlockExperimentalVMOptions -XX:+UseEpsilonGC
```
플래그를 사용하면 된다.

---

# Java 17

## 1. Sealed class 추가

Sealed classe와 interface는 다른 클래스나 인터페이스가 확장하거나 구현할 수 없도록 범위를 제한한다.

---

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

---
