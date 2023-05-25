---

title: Spring과 핵심 디자인 패턴
author: 김도현
date: 2023-05-19
categories: [Spring, Design-Pattern]
tags: [Spring, Design-Pattern]
math: true
mermaid: true

---

# 핵심 패턴 5가지

템플릿 메서드 패턴

전략 패턴

템플릿 콜백 패턴

프록시 패턴

데코레이터 패턴

# 템플릿 메서드 패턴

[Line Engineering Blog](https://engineering.linecorp.com/ko/blog/templete-method-pattern)

`기본개념은 핵심기능과 부가기능을 분리하는 것이다.`

특히 스프링 개발에서는 트랜잭션 코드가 `부가기능`에 해당한다. `핵심 기능은 비즈니스 로직`이기 때문이다.

![](https://github.com/K-Diger/K-Diger.github.io/blob/main/images/design-pattern/template_method1.png)

템플릿 메서드 패턴은 상속을 통해 기능을 확장해서 사용한다.

`변하지 않는 부분`은 `슈퍼 클래스`에 두고 `변하는 부분`은 `추상 메서드`로 정의하여 서브 클래스에서 오버라이드 하여 사용하도록 한다.

![](https://github.com/K-Diger/K-Diger.github.io/blob/main/images/design-pattern/template_method2.png)

---

# 전략 패턴

OCP를 잘 지키면서 조금 더 유연하고 확정성이 뛰어난 패턴은 전략 패턴이다.

![](https://github.com/K-Diger/K-Diger.github.io/blob/main/images/design-pattern/strategy1.png)

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

![](https://github.com/K-Diger/K-Diger.github.io/blob/main/images/design-pattern/strategy2.png)

## StatementStrategy 전략 구현

![](https://github.com/K-Diger/K-Diger.github.io/blob/main/images/design-pattern/startegy3.png)

## 전략을 문맥에 적용 - DI 미적용

![](https://github.com/K-Diger/K-Diger.github.io/blob/main/images/design-pattern/strategy4.png)

위 코드로 구현한 전략패턴을 적용했지만 이상한 점은 직접 구현체를 알고 있어야 한다는 점이다.

특정 구현 클래스를 알고 있다는 것은 OCP원칙을 지켰다고 보기 힘들다.

OCP 원칙은 변경엔 닫혀있으나 확장엔 열려있어야한다.

인터페이스를 의존해야만 새로운 구현을 추가하거나 기존 구현을 변경할 때 영향을 받지 않는 것인데

구현체를 의존하면 새로운 구현을 추가하거나 기존 구현을 변경하면 그 내용을 호출하는 Client의 측에도 변경이 이루어져야하기 때문이다.

DI를 적용하면 이 문제를 어느정도 해결할 수 있다.

## 전략을 문맥에 적용 - DI 적용

![](https://github.com/K-Diger/K-Diger.github.io/blob/main/images/design-pattern/strategy5.png)

클라이언트로부터 매개변수에 전략을 넘겨받은 뒤

해당 전략을 호출한다.

여기서 문맥은 인터페이스가 아닌 구체 클래스이다.

클라이언트가 전략을 선택하는 부분은 다음과 같다.

## 클라이언트에서 전략 선택 및 문맥 호출 - 문맥을 구체 클래스로 직접 선택

![](https://github.com/K-Diger/K-Diger.github.io/blob/main/images/design-pattern/strategy6.png)

전체적인 흐름도를 다시 살펴 보면 다음과 같다.

![](https://github.com/K-Diger/K-Diger.github.io/blob/main/images/design-pattern/strategy7.png)

1. 클라이언트가 사용할 전략을 선택한다. `StatementStrategy st = new DeleteAllStatement()`
2. 문맥에 선택한 전략을 넣어준다. `jdbcContextWithStatementStartegy(st)`

## 클라이언트에서 전략 선택 및 문맥 호출 - 문맥을 구체 클래스로 받지만 외부에서 주입 받도록

![](https://github.com/K-Diger/K-Diger.github.io/blob/main/images/design-pattern/strategy8.png)

# 템플릿 콜백 패턴

# 프록시 패턴

# 데코레이터 패턴
