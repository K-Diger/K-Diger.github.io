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

`기본개념은 핵심기능과 부가기능을 분리하는 것이다.`

특히 스프링 개발에서는 트랜잭션 코드가 `부가기능`에 해당한다. `핵심 기능은 비즈니스 로직`이기 때문이다.

