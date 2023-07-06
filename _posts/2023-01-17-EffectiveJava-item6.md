---

title: Effective-Java Item 6. 불필요한 객체 생성을 피하라
author: 김도현
date: 2023-01-17
categories: [Effective-Java]
tags: [Object]
layout: post
toc: true
math: true
mermaid: true

---

# 좋지 않은 예시

```java
String s = new String("diger");
```

위 문장은 실행될 때 마다, String 인스턴스를 만들어내어 굳이 여러개의 객체를 생성하게 된다. (이 컨벤션은 Java 9 부터 Deprecated되었다.)

---

# 좋지 않은 예시 - 개선

```java
String s = "diger";
```

위 방법 뿐만 아니라, Item 1 에서 언급된 **정적 팩터리 메서드**로 불필요한 객체 생성을 피할 수도 있다.

생성비용이 비싼 객체가 있고 이를 반복해서 사용해야한다면, 캐싱하여 재사용 하자.

> 캐싱은 어떻게 사용할 수 있는가?

---

# 생성 비용이 비싼 객체 캐싱 후 재사용

```java
static boolean isRomanNumberal(String s) {
    return s.matches("^(?=.)M*(C[MD] |D?C{0,3})"
    + "(X[CL] |L?X{0,3})(I[XV]|V?I{0,3})$");
}
```

String.matches 는 정규식으로 문자열 형태를 확인하는 가장 간편한 방식이다.

하지만 성능이 중요한 상황에선, 여러번 반복하여 사용하기 적합하지 않다.

왜냐하면, matches 메서드 내부에서 만드는 Pattern 인스턴스가 한 번 쓰고 버려져, 곧바로 GC 대상이 되지만

Pattern은 입력받은 정규식에 해당하는 유한 상태 머신(스위치 처럼, 특정 입력을 계속 기다리는 객체)을 만들기 때문에 성능이 비싸다.

# 생성 비용이 비싼 객체 캐싱 후 재사용

```java
public class RomanNumerals {
    private static final Pattern ROMAN = Pattern.compile(
            "^(?=.)M*(C[MD] |D?C{0,3})" + "(X[CL] |L?X{0,3})(I[XV]|V?I{0,3})$");

    static boolean isRomanNumeral(String s) {
        return ROMAN.matcher(s).matches();
    }
}
```

위 처럼 final 키워드로 불변하는 Pattern 객체를 생성함으로써 캐싱해두고, 실제 정규식을 수행할 메서드에서, 캐싱된 Pattern 인스턴스를 통해 과정을 마친다.

---

# 불필요한 객체를 만들어 내는 상황 - 오토박싱

오토박싱은 프로그래머가 기본타입과 박싱된(Wrapper클래스 타입) 을 섞어서 쓸 때 자동으로 타입 캐스팅을 해주는 기술이다.

    private static long sum() {
        Long sum = 0L;
        for (long i = 0; i <= Integer.MAX_VALUE; i ++)
            sum += i

        return sum;
    }

<br>

위 코드를 보면, Wrapper 클래스인 Long 타입을 사용하여 반복문을 수행한다.

Wrapper 클래스가 아닌, long 타입을 사용하면 위 반복문의 성능은 6.3초 -> 0.59초로 빨라질 수 있다.

---

# 결론

객체를 생성하지 말라는 것이 아니다.

DB 연결생성과 같은 무거운 객체를 다룰 땐 캐싱 or 기본 타입을 사용하여 객체의 불필요한 생성을 줄여보자.
