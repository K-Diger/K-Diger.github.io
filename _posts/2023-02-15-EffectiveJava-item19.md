---

title: Effective-Java Item 19. 상속을 고려하여 설계하고 문서화 그렇지 않으면 상속을 금지하기
author: 김도현
date: 2023-02-13
categories: [Effective-Java]
tags: [Comparable]
layout: post
toc: true
math: true
mermaid: true

---

# 상속을 고려한 설계와 문서화

## 1. 메서드를 재정의할 때 어떤 일이 일어나는지 정리한다.

상속이 가능한 클래스는 재정의할 수 있는 메서드들을 내부적으로 어떻게 이용하는지 문서화 해야한다.

어떤 순서로 호출하는지 각각의 호출 결과가 이어지는 처리에 어떤 영향을 주는지도 담아야한다.

- 여기서 재정의 가능이란, public, protected 메서드 중 final이 아닌 모든 메서드를 이야기한다.

재정의 가능 메서드를 호출할 수 있는 모든 상황을 문서로 남겨야 한다. (백그라운드 스레드, 정적 초기화 과정 등에서도 호출이 일어날 수 있다.)

## 2. 클래스의 내부 동작 과정 중간에 끼어드는 Hook을 protected 메서드 형태로 공개해야할 수도 있다.

내부 메커니즘을 문서로 남기는 것으로 상속을 위한 설계의 끝이 아니다.

드물게는 protected 필드로 공개해야 할 수도 있다.

그런데, 어떤 메서드를 protected로 노출할지는 어떻게 결정하는가?

실제로 하위 클래스를 만들어 시험해보는 것이 최선이다. 이는 유일한 방법이며 꼭 필요한 방법이다.

## 3. 상속용 클래스의 생성자는 직접적/간접적으로 재정의 가능 메서드를 호출해서는 안된다.

상위 클래스의 생성자가 하위 클래스의 생성자보다 먼저 실행되므로 하위 클래스에서 재정의한 메서드가 하위 클래스의 생성자보다 먼저 호출된다.

```java
public class Parent {

    public Parent() {
        parentOverrideTargetMethod();
    }

    public void parentOverrideTargetMethod() {
    }
}

public final class Child extends Parent {

    private final Instant instant;

    Child {
        instant = Instant.now();
    }

    @Override
    public void parentOverrideTargetMethod() {
        System.out.println("먀");
    }

    public static void main(String[] args) {
        Child child = new Child();
        child.parentOverrideTargetMethod();
    }
}
```

위와 같은 코드가 ## 3. 상속용 클래스의 생성자는 직접적/간접적으로 재정의 가능 메서드를 호출해서는 안된다. 의 핵심이다.

또한 clone과 readObject 메서드는 생성자와 비슷한 효과를 내므로 이를 재정의 하는 로직에 직접적/간접적으로 재정의 가능 메서드를 호출해서는 안된다.

Serializable 를 구현하는 상속용 클래스도 주의해야할 점이 있다.

readResolve나 writeReplace 메서드를 갖는다면 private이 아닌 protected로 선언해야한다.

private으로 선언한다면 하위 클래스에서 무시되기 때문이다.

---

# 결론

- 상속용 클래스를 설계할 때 클래스 내부에서 스스로를 어떻게 사용하는지 모두 문서화를 해야한다.

- 다른 곳에서 더 좋은 하위 클래스를 만들 수 있도록 일부 메서드를 protected를 제공해야할 때도 있다. 이는 직접 하위클래스를 만들고 실험해보며 어떤 메서드를 protected로 할지 결정해야한다.

- 클래스를 확장할 명확한 이유가 없다면 상속을 금지하는 편이 나을 수도 있다. final class 를 활용하거나, 모든 생성자를 private화 하는 것이 그 방법이다.
