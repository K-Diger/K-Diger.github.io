---

title: Spring과 핵심 디자인 패턴 (with 토비의 스프링, 스프링 핵심원리 고급편)

date: 2023-05-19
categories: [Spring, Design-Pattern]
tags: [Spring, Design-Pattern]
layout: post
toc: true
math: true
mermaid: true

---

# 참고 자료

[Line Engineering Blog](https://engineering.linecorp.com/ko/blog/templete-method-pattern)

[토비의 스프링 3장](https://product.kyobobook.co.kr/detail/S000000935358)

[스프링 핵심원리 - 고급편](https://www.inflearn.com/course/%EC%8A%A4%ED%94%84%EB%A7%81-%ED%95%B5%EC%8B%AC-%EC%9B%90%EB%A6%AC-%EA%B3%A0%EA%B8%89%ED%8E%B8)

---

# 핵심 패턴 5가지

1. 템플릿 메서드 패턴

2. 전략 패턴

3. 템플릿 콜백 패턴

4. 프록시 패턴

5. 데코레이터 패턴

---

# 1. 템플릿 메서드 패턴

적용 효과 : `다형성`, `SRP`

기본개념은 `핵심기능과 부가기능`을 분리하는 것이다.

특히 스프링 개발에서는 트랜잭션 코드가 `부가기능`에 해당한다. `핵심 기능은 비즈니스 로직`이기 때문이다.

![](https://github.com/K-Diger/K-Diger.github.io/blob/main/images/design-pattern/template_method1.png?raw=true)

템플릿 메서드 패턴은 상속을 통해 기능을 확장해서 사용한다.

`변하지 않는 부분`은 `슈퍼 클래스`에 두고 `변하는 부분`은 `추상 메서드`로 정의하여 서브 클래스에서 오버라이드 하여 사용하도록 한다.

![](https://github.com/K-Diger/K-Diger.github.io/blob/main/images/design-pattern/template_method2.png?raw=true)

---

# 2. 전략 패턴

OCP를 잘 지키면서 조금 더 유연하고 확정성이 뛰어난 패턴은 전략 패턴이다.

![](https://github.com/K-Diger/K-Diger.github.io/blob/main/images/design-pattern/strategy1.png?raw=true)

전략패턴은 오브젝트를 둘로 분리하고 클래스 레벨에서는 인터페이스를 의존하도록 만드는 패턴이다.

`deleteAll()`메서드에서 변하지 않는 부분은 contextMethod()로 선언한 후 변하지 않는 `맥락`이라는 것을 명시한다.

deleteAll()의 흐름을 보면 아래와 같다.

- DB Connection 가져오기
- PreparedStatement를 만들어줄 외부 기능 호출
- 전달받은 PreparedStatement 실행
- 예외처리
- Statement, Connection 닫기

여기서 PreparedStatement를 만들어주는 외부 기능이 `전략 패턴`의 `전략`에 해당한다.

## StatementStrategy 전략

![](https://github.com/K-Diger/K-Diger.github.io/blob/main/images/design-pattern/strategy2.png?raw=true)

## StatementStrategy 전략 구현

![](https://github.com/K-Diger/K-Diger.github.io/blob/main/images/design-pattern/startegy3.png?raw=true)

## 전략을 문맥에 적용 - DI 미적용

![](https://github.com/K-Diger/K-Diger.github.io/blob/main/images/design-pattern/strategy4.png?raw=true)

위 코드로 구현한 전략패턴을 적용했지만 이상한 점은 직접 구현체를 알고 있어야 한다는 점이다.

특정 구현 클래스를 알고 있다는 것은 OCP원칙을 지켰다고 보기 힘들다.

OCP 원칙은 변경엔 닫혀있으나 확장엔 열려있어야한다.

인터페이스를 의존해야만 새로운 구현을 추가하거나 기존 구현을 변경할 때 영향을 받지 않는 것인데

구현체를 의존하면 새로운 구현을 추가하거나 기존 구현을 변경하면 그 내용을 호출하는 Client의 측에도 변경이 이루어져야하기 때문이다.

DI를 적용하면 이 문제를 어느정도 해결할 수 있다.

## 전략을 문맥에 적용 - DI 적용

![](https://github.com/K-Diger/K-Diger.github.io/blob/main/images/design-pattern/strategy5.png?raw=true)

클라이언트로부터 매개변수에 전략을 넘겨받은 뒤

해당 전략을 호출한다.

여기서 문맥은 인터페이스가 아닌 구체 클래스이다.

클라이언트가 전략을 선택하는 부분은 다음과 같다.

## 클라이언트에서 전략 선택 및 문맥 호출 - 문맥을 구체 클래스로 직접 선택

![](https://github.com/K-Diger/K-Diger.github.io/blob/main/images/design-pattern/strategy6.png?raw=true)

전체적인 흐름도를 다시 살펴 보면 다음과 같다.

![](https://github.com/K-Diger/K-Diger.github.io/blob/main/images/design-pattern/strategy7.png?raw=true)

1. 클라이언트가 사용할 전략을 선택한다. `StatementStrategy st = new DeleteAllStatement()`
2. 문맥에 선택한 전략을 넣어준다. `jdbcContextWithStatementStartegy(st)`

## 클라이언트에서 전략 선택 및 문맥 호출 - 문맥을 구체 클래스로 받지만 외부에서 주입 받도록

![](https://github.com/K-Diger/K-Diger.github.io/blob/main/images/design-pattern/strategy8.png?raw=true)

---

# 3. 템플릿 콜백 패턴

바뀌지 않는 일정한 내용이 존재하고, 그 중 일부분만 바꿔서 사용해야하는 경우에는 전략패턴이 적절하다.

이러한 전략 패턴의 기본 구조에 익명 내부 클래스를 활용한 방식을 `템플릿/콜백 패턴`이라고 부른다.

`전략 패턴`의 `문맥`을 `템플릿`이라고 부르고 익명 내부 클래스로 만들어지는 객체를 `콜백`이라고 부른다.

템플릿 메서드 패턴은 고정된 틀의 로직을 가진 템플릿 메서드를 슈퍼클래스에 두고 바뀌는 부분을 서브클래스의 메서드에 두는 구조로 이루어진다.

콜백은 실행되는 것을 목적으로 다른 오브젝트의 메서드에 전달되는 객체를 말한다.

---

# 4. 프록시 패턴

---

# 5. 데코레이터 패턴

