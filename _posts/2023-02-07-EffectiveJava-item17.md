---

title: Effective-Java Item 17. 변경 가능성을 최소화하라

date: 2023-02-07
categories: [Effective-Java]
tags: [Comparable]
layout: post
toc: true
math: true
mermaid: true

---

# 변경 가능성 최소화하기 - 불변 클래스

불변 클래스란?

인스턴스의 내부 값을 수정할 수 없는 클래스이다.

불변 클래스에 포함된 정보는 고정되어 그 정보는 절대로 달라지지 않는다.

ex) String, Integer, Long, BigInteger, BigDecimal 등

## 불변 클래스를 사용하는 이유는?

오류가 생길 여지가 적어 안전하다. 외부에서 값을 수정 할 수 없기 때문에 잘 만들어만 놓았다면 이로 인한 문제가 발생할 여지가 없기 때문이다.

## 클래스를 불변으로 만들기 - 5가지 규칙

- 1. Setter와 같은 수정자를 지양

- 2. 클래스를 확장하지 못하도록
  - 클래스를 final로 선언하는 방법이 그 예시이다.

- 3. 모든 필드를 final로 선언

- 4. 모든 필드를 private 으로 선언
  - public final으로 선언해도 충분할 수 있지만, 다음 릴리스에서 내부 표현을 바꾸지 못할 수 있다.

- 5. 자신 외에는 내부 가변 컴포넌트에 접근할 수 없도록 선언
  - 클래스에 가변 객체를 참조하는 필드가 하나라도 있다면 클라이언트에서 그 참조를 얻지 못하도록 해야한다.

### 5. 자신 외에는 내부 가변 컴포넌트에 접근할 수 없도록 선언 - 예시

```java
public class Post {

    // 가변 객체를 참조하는 필드를 외부에서 접근할 수 없음!
    private User user;
}
```

## 불변 복소수 클래스 - 불변 클래스 예시

```java
public final class Complex {
    private final double re;
    private final double im;

    public Complex(double re, double im) {
        this.re = re;
        this.im = im;
    }

    public double realPart() {
        return re;
    }

    public double imaginaryPart() {
        return im;
    }

    public Complex plus(Complex complex) {
        return new Complex(re + complex.re, im + complex.im);
    }

    ...
}
```

- readPart() 메서드는 실수부를 반환하는 접근자 메서드

- imaginaryPart() 메서드는 허수부 값을 반환하는 접근자 메서드이다.

- plus() 메서드는 말 그대로 덧셈을 수행하는 메서드로 인스턴스 자신을 수정하지 않고 새로운 Complex 인스턴스를 만드는 것을 주목해야한다.

plush() 메서드는 피연산자에 함수를 적용하여 그 결과를 반환하지만, 피연산자 자체는 그대로인 패턴으로 **함수형 프로그래밍** 이라고 한다.

절차적 혹은 명령형 프로그래밍에서는 메서드에서 피연산자인 자신을 수정한다.

즉, this.complex.re = re + complex.re;

와 같이 실제 필드를 변경하여 상태가 변경하게 된다는 것이다.


이렇게 실제 필드를 직접 변경하지 않는다는 것을 나타내기 위해 메서드 네이밍에도 디테일을 담을 수 있는데

보통 덧셈 연산이라고 하면 **add**라는 동사로 네이밍 할 수 있겠지만, **plus**라는 전치사를 활용하여 객체의 값을 변경하지 않는다는 점을 강조할 수 있다.

## 불변 클래스의 장/단점

### 장점 1. 불변 객체는 단순하다.

생성된 시점의 상태를 파괴될 때 까지 간직하기 때문에 어떠한 변화에 영향을 받았는지 신경쓰지 않아도 된다.

### 장점 2. 스레드 안전하다.

여러 스레드에서 동시에 접근하더라도 값 자체는 절대 변하지 않기 때문에 동시성 문제를 유발할 여지가 없다.

### 장점 3. 스레드 안전함으로 공유성이 확보된다.

어떤 스레드이던 다른 스레드에 영향을 줄 수 없으니 공유할 수 있다.

### 장점 4. 불변 클래스 인스턴스를 공유하여 메모리를 절약할 수 있다.

```java
public static final Complex ZERO = new Complex(0, 0);
public static final Complex ONE = new Complex(1, 0);
public static final Complex I = new Complex(0, 1);
```

위와 같이 public static final 키워들를 통해 상수를 활용하는 것으로 인스턴스를 재활용 할 수 있다.

또한 이 방법을 조금 더 클린하게 사용하는 방법인 **정적 팩터리**가 있다.

WrapperClass 및 BigInteger 등이 모두 이 정적 팩터리를 사용하고 있다.

정적 팩터리를 사용하면 여러 클라이언트가 인스턴스를 공유하기 때문에 메모리 사용량/GC 비용이 줄어든다.

또한 이 방식을 더 극대화하면 캐시 기능을 덧붙일 수 있기 때문에 더 권장하는 방식이다.

---

### 장점 5. 방어적 복사가 필요없다.

아무리 복사해봐야 원본과 똑같기 때문이다. String 클래스는 clone() 메서드를 재정의 했지만, 이는 이 사실이 잘 알려지지 않은 Java 초창기에 만들어 진 내용이므로

사용하지 않는 것이 권장된다.

### 장점 6. 불변 객체는 그 자체로 실패 원자성을 제공한다.

즉, 어떤 시점에 접근하더라도 같은 값을 반환하기 때문에 원자성을 제공할 수 있다는 것이다.

---

### 단점 1. 값이 다르면 반드시 독립된 객체로 만들어야한다.

값의 가짓수가 많으면 만약의 각 다른 값이 100개라고 치면, 100개의 인스턴스를 만들어줘야한다.

사실 단점이 한가지로 보이지만 이게 엄청나게 큰 단점이다. 그렇기 때문에 이 단점을 조금 더 개선할 수 있는 방법을 알아보자.

## 불변객체를 사용하는 상황의 단점 개선하기

### 개선 방법 1. 흔하게 쓰일 다단계 연산들을 예측하기

쉽게 말하면 BigIngter 라는 불변 클래스에 add, minus, multiply, divide 등과 같은 자주 쓰일 것 같은 메서드를 public으로 열어둔다면 어느정도 해결된다는 것이다.

## 불변 클래스를 만드는 방법

### 1. 상속이 불가능하게 하는 방법 - final 클래스로 만들기

### 2. 상속이 불가능하게 하는 방법 - 모든 생성자를 private, 혹은 package-private으로 만들기

우리가 잘 아는 정적 팩터리를 이용한 방식으로 한다면 상속이 불가능한 불변 객체를 조금 더 유연하게 사용할 수 있따.

```java
public class Complex {
    private final double re;
    private final double im;

    private Complex(double re, double im) {
        this.re = re;
        this.im = im;
    }

    public static Complex valueOf(double re, double im) {
        return new Complex(re, im);
    }
}
```

public 생성자가 없으면 상속이 불가능한 점을 이용하는 것으로 위와 같이 설계한다면 사실상 final class 와 같게 된다.

---

# 정리

- Getter가 있다고 해서 무조건 Setter를 만들지 말자.

- 클래스는 꼭 필요한 경우가 아니라면 불변이어야 한다.

- 단순한 값 객체는 항상 불변으로 만들자.
  - (DTO도 값 객체라고 보고 불변으로 만들면 될까? Entity 객체도 사실상 값 객체가 아닌가..?)

- 불변 클래스와 쌍을 이루는 가변 동반 클래스를 public 클래스를 제공하자.

- 모든 클래스를 불변으로 만들 수 없다. 따라서 변경할 수 있는 부분을 최소한으로 줄여야한다.

- 변경이 필요하지 않은 필드는 final로 선언하는 것을 잊지 말자!

- 다른 합당한 이유가 없다면 모든 필드는 private final로 선언되어야한다.

- 생성자는 불변식 설정이 모두 완료되어 초기화가 완벽히 끝난 상태의 객체를 생성해야한다.
