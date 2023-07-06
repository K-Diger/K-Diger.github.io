---

title: Effective-Java Item 15, 16. 클래스와 멤버의 접근을 최소화하라, Public 클래스에서는 Public 필드가 아닌 접근자 메서드를 사용하라.
author: Diger
date: 2022-12-20
categories: [Effective-Java]
tags: [hashCode]
layout: post
toc: true
math: true
mermaid: true

---

# 잘 설계된 컴포넌트

모든 내부 구현을 완벽히 숨겨, 구현과 API를 통해서만 다른 컴포넌트와 소통하며, 서로의 내부 동작 방식에는 전혀 개의치 않는다.

> 여기서 말하고자 하는 다른 컴포넌트와의 소통은 무엇인가?

---

# 정보 은닉의 장점

보통 객체지향을 처음 배우면 **캡슐화** 라는 용어를 배우게 된다. 캡슐화는 쉽게 말하면 어떤 클래스가 있을 때 그 내부 필드에는 접근못하도록 **접근제어자를** 손보는 것이다.

그렇다면 이 행위의 장점은 무엇인가?

- 시스템 개발 속도 향상 --> 여러 컴포넌트를 병렬로 개발할 수 있게되기 때문이다.
- 시스템 관리비용 절감 --> 각 컴포넌트를 더 빨리 파악하여 디버깅 할 수 있게 된다.
- 정보 은닉 자체가 성능 향상에 직접적인 도움이 되진 않는다. 하지만 완성된 시스템 내에서 특정 정보 은닉된 요소만 변경하면 다른 컴포넌트에 영향 없이 최적화가 가능하다.
- 재사용성
- 제작 난이도 완화

# 정보 은닉의 원칙

모든 클래스와 멤버의 접근성을 가능한 좁혀야한다.

패키지 외부에서 쓸 이유가 없다면, package-private으로 선언해야한다.

여기서 package-private는 무엇인가? --> 사실 내가 잘 모르는 용어여서 찾아봤다.

그리고 의외로 내가 자주 사용하던 방식이였다.

```java
package user;

public class UserClass {

	private String name;

	String getName() {
		return name;
	}
}
```
이처럼 어떤 클래스의 메서드는 public 하지만, 그 필드는 private 한 형식을 package-private이라고 한다.

스프링으로 개발할 때 Entity 혹은, 의존성 주입을 받는 Controller, Service 등에서 자주 사용하던 방식이다.

여기서 주목하고자 하는 구절은

> public 클래스의 인스턴스 필드는 되도록 public이 아니어야 한다. 라는 것이다.

우리가 당연하게도 알고 있던 이유 때문이다. public 필드는 외부에서 수정이 가능하기 때문에 스레드 안전하지 않다.

그러면 final로 외부에서 변경하지 못하게 하면 되는것 아닌가??

이건 바바리맨을 잡고자 바바리코트를 못입게 만드는... 것이다. 이 클래스의 필드가 의미가 없어진다. 수정을 나조차도 할 수 없기 때문이다.

그리고 주의해야할 점은 배열의 사용에 있다.

```java
public static final Thing[] VALUES = {...};
```

위와 같이 static final인 배열은 길이가 0이 아니라면 모두 변경이 가능하다. 따라서 다음과 같은 배열 필드를 선언하거나 반환하지 않도록 해야한다.


정 위와같은 배열을 쓰고 싶다면 다음 2가지의 해결방법이 있다.

방법 1.
```java
private static final Thing[] PRIVATE_VALUES) = {...};
public static final List<Thing> VALUES = Collections.unmodifiableList(Arrays.asList(PRIVATE_VALUES));
```
배열을 private으로 선언하고, 이를 다룰 불변 List를 만들어 사용하는 것이다.

방법 2.
```java
private static final Thing[] PRIVATE_VALUES) = {...};
public static final Thing[] values() {
    return PRIVATE_VALUES.clone();
    }
```

다음과 같이 배열을 private으로 만들고 그 복사본을 반환하는 public 메서드를 만드는 것이다. (방어적 복사)

---

# Item 15. 클래스와 멤버의 접근 권한을 최소화하라 - 결론

프로그램의 접근성은 최소화, 꼭 필요한 내용만 public으로 설정해야한다.

또한 public class는 상수용 public static ifnal 필드 외에는 **어떤** 필드도 public 으로 선언하면 안된다.

public static final 필드가 참조하는 객체가 불변인지도 꼭 체크해야한다.

---

# 캡슐화를 극대화 해보자.

```java
class Point {
    private double x;
    private double y;

    public Point(double x, double y) {
        this.x = x;
        this.y = y;
    }

    public double getX() {return x;}
    public double getY() {return y;}
}
```

하지만 package-private 클래스나 혹은 private 중첩 클래스라면 데이터 필드를 노출해도 상관없다. --> 이게 무슨말일까.... (p.103)

### Java 라이브러리에서도 public 클래스의 필드를 노출하는 경우가 있다!!

java.awt.package 패키지의 Point와 Dimension 클래스이다.

이는 추후에 등장하겠지만 심각한 성능문제가 발생하고 있는 라이브러리이므로 예외 케이스가 아님을 꼭 짚고 넘어가자.


# Item 16. Public 클래스에서는 Public 필드가 아닌 접근자 메서드를 사용하라 - 결론

가변 필드에 접근하도록 하려면, 어지간하면 Getter, Setter를 사용하자!!

불변필드라면 노출해도 괜찮지만 그래도 완전하게 안심할 수는 없다.

그렇지만 package-private 클래스나 private 중첩 클래스에서는 불변,가변 상관없이 노출하는 것이 나을 때도 있다. (이게 도대체 무슨소리람!..)
