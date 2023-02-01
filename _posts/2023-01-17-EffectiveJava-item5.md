---

title: Effective-Java Item 5. 자원을 직접 명시하지 말고 의존 객체 주입을 사용하라
author: 김도현
date: 2023-01-17
categories: [Effective-Java]
tags: [Object]
math: true
mermaid: true

---

# 객체지향을 위해서, 각 객체는 객체 간 메세지를 통해 협력해야한다.

"맞춤법 검사기"라는 객체는 "사전"에 의존한다. 이런 클래스를 정적 유틸리티 클래스와 싱글턴으로 구현하면 다음과 같다.

```java
// 정적 유틸리티 클래스
public class SpellChecker {
    private static final Lexicon dictionary = ...;

    private SpellChecker() {}

    public static boolean isValid(String word) {}
    public static List<String> suggestions(String typo) {}
}
```


```java
// 싱글턴
public class SpellChecker {
    private static final Lexicon dictionary = ...;

    private SpellChecker() {}

    public static SpellChecker INSTANCE = new SpellChecker();

    public static boolean isValid(String word) {}
    public static List<String> suggestions(String typo) {}
}
```

위 두 방식은 모두 사전을 단 하나만 사용한다는 가정이 달려있는 코드이다. 실제로는 사전이 언어별로 따로 있기도 하며 다양하게 존재하기 때문에

유연하지 않은 설계이며, 테스트하기 어려운 점이 있다.

사용하는 자원에 따라 동작이 달라지는 클래스에는 정적 유틸리티 클래스 혹은 싱글턴은 적합하지 않다.

# 유연한 객체로 변환하기

## 방법 1.

dictionary 필드에서 final 한정자를 제거하고 다른 사전으로 교체하는 메서드를 추가한느 방법도 있다.

멀티스레드 환경에서는 쓸 수 없는 방법이다. (스레드 간 동기화 문제 발생!)

## 방법 2.

인스턴스를 생성할 때 생성자에 필요한 자원을 넘겨주는 방식이 있다.

즉, 어떤 SpellChecker를 생성하면서 필요한 Dictionary객체를 직접 주입해주는 방식인데, 코드로 표현하면 다음과 같다.

```java
public class SpellChecker {
    private final Lexicon dictionary;

    public SpellChecker(Lexicon dictionary) {
        this.dictionary = Objects.requirNonNull(dictionary);
    }

    public boolean isValid(String word) {}
    public List<String> suggestions(String typo) {}
}
```

위와 같은 방식을 의존 객체 주입 이라고 하는데, 이 방식은 생성자, 정적 팩터리, 빌더 패턴에도 모두 적용할 수 있으며

의존관계가 늘어나도 생성자에 주입을 해주면 되기 때문에 쉽게 유연함을 가질 수 있다.

이 패턴의 활용하는 것으로, 생성자에 자원 팩터리를 넘겨주는 방식이 있다.

> 팩터리란 매 호출 시 특정 타입의 인스턴스를 반복해서 만들어주는 객체를 뜻한다.

Java 8에서 등장한 Supplier<T> 인터페이스가 팩터리를 표현한 그 예시이며 Supplier<T>를 매개변수로 받는 메서드는

제네릭 기능에 의해 자신이 명시한 타입의 하위 타입이라면 무엇이든지 생성할 수 있는 팩터리를 만들 수 있다.

# 의존 객체 주입이 장점만 있나?

그건 아니긴하다. 의존성이 수 천개가 된다면 어지러워진다.

그렇기 때문에 **Spring**, Guice, Dagger 등 **의존 객체 주입 프레임워크**를 활용하면 이 어지러운 상황을 해소할 수 있다.

# 결론

어떤 클래스가 하나 이상의 자원에 의존하고 그 자원이 클래스 동작에 영향을 준다면 싱글턴/정적 유틸리티 클래스는 사용하지 않는 것이 좋다.

이런 상황에서는 의존 객체 주입을 사용해보자.
