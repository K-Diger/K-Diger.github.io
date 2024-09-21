---

title: OOP-DesignPattern
date: 2023-08-02
categories: [OOP, DesignPattern]
tags: [OOP, DesignPattern]
layout: post
toc: true
math: true
mermaid: true

---

# 좋은 설계의 특징

## 코드의 재사용성이 확보되어있다.

재사용성의 수준은 다음 세 가지로 볼 수 있다.

- 클래스 (라이브러리, 컨테이너, 반복자 등)
- 디자인 패턴
- 프레임워크

## 확장성을 고려하여 설계되어있다.

프로그래머에게는 모든 것이 변한다는 것이 변하지 않는다.

# 어떻게 좋은 설계를 할 수 있을까?

1. 캡/상/추/다

**캡슐화란 데이터를 감추는게 아니라 변할 수 있는 디자인적 요소를 감추는 것 - 마틴 파울러**

## 1. 변화하는 내용을 캡슐화 한다.

- 변하는 부분과 변하지 않는 부분을 구분하여 이를 분리한다.

이 원칙의 가장 큰 **목적은 변화로 인해 발생하는 비용을 최소화**하는 것이다.

### 1.1 메서드 수준에서의 캡슐화

아마존에서 어떤 상품을 주문한다고 하자.

이 때 세금을 포함한 주문의 합계를 계산하는 `getOrderTotal` 메서드가 있을 때

세금 관련 코드는 변경이 일어날 수 있다는 것이 예측된다. (세금 관련 정책이 바뀐다면 변경이 발생할 수 있기 때문)

따라서 `getOrderTotal`메서드는 자주 변경해야할 가능성이 있는 것인데 이 메서드를 사용하는 입장에서는 세금이 어떻게 계산되는지는 전혀 관심이 없다.

그저 세금을 계산해주는 메서드를 사용할 뿐이다.

---

세금을 계산하는 메서드를 캡슐화 하기 전의 코드는 아래와 같다.

```kotlin
class Order {
    fun getOrderTotal(order: List<Order>) {
        var total = 0
        total += order.map { it.price * it.quantity }

        when (order.country) {
            Country.USA -> return total += total * 0.07
            Country.UK -> return total += total * 1.12
        }

        return total
    }
}

```

이 메서드를 캡슐화 한다면 아래와 같이 나타낼 수 있다.

```kotlin
class Order() {
    fun getOrderTotal(order: List<Order>) {
        var total = 0
        total += order.map { it.price * it.quantity } * getTaxRate(order.country)
        return total
    }

    fun getTaxRate(country: Country) {
        when (order.country) {
            Country.USA -> return total += total * 0.07
            Country.UK -> return total += total * 1.12
        }
    }
}
```

위와 같이 세금을 계산하는 로직을 메서드로 추출(캡슐화)하여 `getTaxRate`를 만들면

`getOrderTotal`로직을 만들고 있는 사용자는 세금 계산법의 변화에 전혀 신경쓰지 않아도 된다.

`getTaxRate()`를 변화에 맞게 수정해준다면 실제 비즈니스 로직을 손을 대지 않아도 되기 때문이다.

또한 이렇게 어떤 로직을 메서드로 추출함으로써 다른 메서드나 클래스에서도 사용할 수 있게 되었기 때문에 재사용성을 확보할 수 있게 되었다.

---

### 1.2 클래스 수준에서의 캡슐화

위에서 살펴본 `order`클래스에 많은 메서드들이 더 추가될 수 있다. 즉, 더 많은 책임이 추가될 수 있다는 건데

추가된 행동들에 의해 클래스의 역할을 모호하게 만들 수 있다.

그렇기 때문에 성질이 다른 역할/책임이 있다면 이를 분리하는 것이 코드가 더 간단해질 수 있다.

```kotlin
class Order {
    fun getOrderTotal(order: List<Order>) {
        var total = 0
        total += order.map { it.price * it.quantity } * TaxCalculator.getTaxRate(order.country)
        return total
    }
}

class TaxCalculator {
    fun getTaxRate(country: Country) {
        when (order.country) {
            Country.USA -> return total += total * 0.07
            Country.UK -> return total += total * 1.12
        }
    }
}
```

위 처럼 클래스를 분리한다면 Order는 주문에 대한 역할만 보유하고 있게되며 TaxCaculator는 그저 세금에 대한 계산만 수행하는 역할을 가지고 있게 된다.

---

## 2. 구현이 아닌 인터페이스에 대해 프로그래밍해라

기존 코드를 망가뜨리지 않고 쉽게 확장이 가능하다면 그 디자인은 충분히 유연하다.

그리고 이 유연함을 확보할 수 있는 방법을 마련하기 위한 단계가 있다.

1. 객체 A가 객체 B로부터 정확히 무엇을 필요로 하는가? (어떤 메서드를 실행하는가?)
2. 해당 내용을 인터페이스로 만들어서 이 메서드들을 정의해라
3. 객체 B를 의존하는 클래스(객체 A로 하여금)가 이 인터페이스를 구현하도록 해라
4. 객체 A를 해당 인터페이스를 의존하도록 해라

![](https://github.com/K-Diger/K-Diger.github.io/tree/main/images/architecture-oop/interface.png)

---

## 3. 상속보다 합성을 사용해라

