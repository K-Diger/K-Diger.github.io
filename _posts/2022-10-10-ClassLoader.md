---

title: JVM - ClassLoader
date: 2022-10-10
categories: [ClassLoader]
tags: [ClassLoader]
layout: post
toc: true
math: true
mermaid: true

---

# ClassLoader

![Class Loader](https://javatutorial.net/wp-content/uploads/2017/11/jvm-featured-image.png)

Javac에 의해 바이트코드로 변환된 클래스 파일을 JVM 메모리로 로딩하기 위한 하위 시스템이다. Class Loader는 세 가지 주요 기능이 있다.

- `Loading`
- `Linking`,
- `Initialization`

---

## 1. ClassLoader - Loading

클래스 로더는 .class 파일을 읽고, 바이너리 데이터를 생성하여 메서드 영역에 저장한다.

클래스 로더의 종류로는 세 가지가 있으며 각 역할이 부여되어있다.

- `BootStrap` : 시스템 클래스 및 Java 표준 라이브러리에 대한 클래스를 로딩한다. (jre/lib 폴더의 rt.jar에 위치한 클래스들을 로딩한다.)
- `Platform ClassLoader` : jre/lib/ext 폴더 내의 클래스를 로딩한다.
- `Application ClassLoader` : 작성한 코드에 대한 클래스, 환경 변수를 로딩한다.

또한 각 .class 파일 마다 JVM은 아래의 세 가지 정보를 `메서드 영역에 저장`한다.

1. `로드된 클래스`와 그 모든 `연관 부모 클래스`

2. .class 파일이 `Class` or `Interface` or `Enum`어떤 것으로 작성된 것인지?

3. `수정자`, `변수`와 `메서드 정보` 등

우리는 흔히 `getClass()`메서드로 class를 코드 관점에서도 사용이 가능하다. 클래스 로더가 하는 역할이 덕분인데,

.class 를 로딩을 마친 후에, JVM 은 `Heap 영역에 이 파일들을 나타내기 위해` Class 유형의 객체를 생성한다.

또한 코드 단에서 ClassLoader를 확인하고 싶으면 아래와 같이 작성할 수 있다.

```java
public class App {

    public static void main(String[] args) {
        System.out.println("Hello My Name Is main In App.class");
        System.out.println(App.class.getSuperclass());

        ClassLoader classLoader = App.class.getClassLoader();
        System.out.println("classLoader = " + classLoader);
        System.out.println("classLoader.getParent() = " + classLoader.getParent());
        System.out.println("classLoader.getParent().getParent() = " + classLoader.getParent().getParent());
    }

}
/*
출력 결과

Hello My Name Is main In App.class
class java.lang.Object
classLoader = jdk.internal.loader.ClassLoaders$AppClassLoader@251a69d7 // 가장 하위의 클래스 로더
classLoader.getParent() = jdk.internal.loader.ClassLoaders$PlatformClassLoader@7cd84586 // 중간 클래스 로더
classLoader.getParent().getParent() = null // 최상위 클래스 로더 (있긴 있지만 네이티브로 작성되어있어 확인할 수 없다.)
*/
```

---

## 2. ClassLoader - Linking

`Verification`, `Preparation`, `Resolution` 을 수행한다.

### 2.1 ClassLoader - Linking : Verification

.class 파일이 올바른 형식으로 지정되고, 유효한 컴파일러에 의해 생성되었는지 확인한다.

확인에 실패 했을 때 `RuntimeException` 혹은 `java.lang.VerifyError` 가 발생한다.

이 과정이 끝났다면, 클래스 파일을 컴파일 할 준비가 완료 된 것이다.

### 2.2 ClassLoader - Linking : Preparation

클래스 변수에 메모리를 할당하고 메모리를 기본값으로 초기화한다.

### 2.3 ClassLoader - Linking : Resolution

간접 참조 방식을 직접 참조 방식으로 바꾸는 과정이다. 참조된 엔티티를 찾기 위하여 `메서드 영역을 검색`한다.

---

## 3. ClassLoader - Initialization

모든 static 변수가 코드 및 static block 에 정의된 값으로 할당된다. (static 키워드가 들어간 요소들 메모리 할당)

부모에서 자식 순으로 수행된다.

--> static 요소들 메모리 할당 --> 부모 클래스 메모리 할당 --> 자식 클래스 메모리 할당 --> 기타 요소들 할당
