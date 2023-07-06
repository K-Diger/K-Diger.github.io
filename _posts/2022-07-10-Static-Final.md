---

title: Static, Final 에 관하여
author: 김도현
date: 2022-07-10
categories: [Java]
tags: [Static, Final]
layout: post
toc: true
math: true
mermaid: true

---

# static 이란

소속을 정의해주는 것이라고 생각하면 된다.

    class Test {
        public static String Value = "Class"; // 클래스 소속

        public String value = "Instance"; //인스턴스 소속

        public static void classMethod() {
            System.out.println(Value); // OK
            System.out.println(value); // Error!
        }

        public void InstanceMethod() {
            System.out.println(Value); // OK
            System.out.println(value); // OK
        }
    }

    public class StaticApp {
        System.out.println(Test.Value); // OK
        System.out.println(Test.value); // Error!
    }

<br>

> **인스턴스 소속(Static 키워드를 안 붙인경우)**
>
> 인스턴스에 속한, 변수-메서드에 접근할때
>
> 클래스 단위에서 접근하는 것이 금지되어있다. ex) Test.value, Test.InstanceMethod()
>
> 즉, 클래스에 대한 인스턴스를 만들고 그 인스턴스에 직접 변수와 메서드를 호출해야하는 방식이다.

<br>

> **클래스 소속(Static 키워드를 붙인경우)**
>
> Static 키워드를 가진 변수-메서드에는 클래스 단위든, 인스턴스 단위든 상관없이 호출 가능하다.
>
> 객체 내부에 존재하는 것이 아닌, 별도의 공간에 저장한다. (static 변수-메서드 : JVM - 메서드 영역, 인스턴스 변수-메서드 : JVM - 힙 영역)

# static 내부 동작

    // 인스턴스 생성
    Test test = new Test();

인스턴스를 생성했을 때, 해당 클래스가 **static** 키워드를 가진 메서드나 변수를 포함하고 있을 때

인스턴스가 가지고 있는 내용은, **static** 키워드의 직접적인 내용이 아니라, 참조만 할 뿐이다.

그렇지 않은 변수-메서드는 인스턴스 내부에 복제된다.

![image](https://user-images.githubusercontent.com/60564431/178138195-b6a193d8-475c-4096-af73-46249ef0604e.png)

https://www.youtube.com/watch?v=hvTuZshZvIo 이미지 참고

위 그림과, 설명으로도 알수있듯이, 클래스의 static 내용을 변경하게되면, 이를 참조하는 모든 인스턴스의 내용 또한 변한다.

---

# 궁금했던 점 1. 정적 변수 vs 전역 변수

Life Cycle 은 두 타입 모두 프로그램이 종료될 때 까지이다.

Java 에서는 굳이 전역변수를 사용하지 않는다.

정적변수로 그 기능을 대체하는 것을 보인다.

----

# final 이란

추상(abstract)이 상속을 강제하는 것이라면,

final은 상속/변경을 금지하는 규제이다.

### 자바 final 정의

> 프로그램 실행 중 한 번만 할당 가능하다.
>
> 즉, 재할당 하려고 하면, 컴파일 오류가 발생한다.


### final 의 존재 이유

final을 알아보려한 이유는, Spring Boot 로 API를 개발할때,

Service 단에서, Repository를 주입받아 사용할 때

private final ~~~Repository 의 형태를 기계적으로 사용해와서 그 이유를 알고싶었다.

의미를 하나씩 해석을해보면,

> 호출한 클래스 안에서만 사용가능하다. -> private

> 새롭게 가져오고자 하는 ~~~Repository 클래스는, 호출한 클래스에서 변경 불가능하다. -> final
