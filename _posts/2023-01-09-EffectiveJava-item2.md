---

title: Effective-Java Item 2. 생성자에 매개변수가 많을때는 빌더를 고려하자
author: 김도현
date: 2023-01-09
categories: [Effective-Java]
tags: [Constructor, Builder]
math: true
mermaid: true

---

# 객체 생성의 3가지 방법

## 1. 점층적 생성자 패턴

```java
public class NutritionFacts {
    private final int servingSize;
    private final int servings;
    private final int calories;
    private final int fat;
    private final int sodium;
    private final int carbohydrate;

    public NutritionFacts(int servingSize, int servings) {
        this(servingSize, servings, 0);
    }

    public NutritionFacts(int servingSize, int servings, int calories) {
        this(servingSize, servings, calories, 0);
    }

    public NutritionFacts(int servingSize, int servings, int calories, int fat) {
        this(servingSize, servings, calories, fat, 0);
    }

    public NutritionFacts(int servingSize, int servings, int calories, int fat, int sodium) {
        this(servingSize, servings, calories, fat, sodium, 0);
    }

    public NutritionFacts(int servingSize, int servings, int calories, int fat, int sodium, int carbohydrate) {
        this(servingSize, servings, calories, fat, sodium, carbohydrate, 0);
    }
```

위 처럼 생성자의 매개변수를 늘려가는 방식이 점층적 생성자 패턴이다.

### 점층적 생성자 패턴의 장점

솔직하게 말해서 하나도 없는 것 같다. 이미 다른 패턴을 알고 있어서 그런걸까?

### 점층적 생성자 패턴의 단점

1. 매개변수가 많아지면 어떤 생성자를 사용해야하는지 클라이언트 입장에선 굉장히 모호해진다.
2. 매개변수가 많아지면 코드를 읽기가 어렵다.
3. 생성자 매개변수의 순서가 바뀌어도 컴파일 단계에서는 알아챌 수 없다. 런타임 때 알아채면 다행인 상황이 발생한다.

---

## 2. 자바빈즈 패턴 (Setter)

```java
public class NutritionFacts {
    private int servingSize;
    private int servings;
    private int calories;
    private int fat;
    private int sodium;
    private int carbohydrate;

    public NutritionFacts() {}

    public void setServingSize();
    public void setServings();
    public void setCalories();
    public void setFat();
    public void setSodium();
    public void setCarbohydrate();
}
```

### 자바빈즈 패턴의 장점

1. 읽기가 한결 쉬워진다.
2. 인스턴스 생성 자체는 매우 쉽다.

### 자바빈즈 패턴의 단점

1. 객체 하나 만드는데 메서드를 엄청나게 많이 호출해야한다.
2. 객체가 완전히 생성되기 전까지는 일관성이 무너진 상태이다.
   3. 객체를 생성하고, setServingSize() ... 등의 모든 Setter 메서드를 호출 하기 전과 후의 객체는 일관성이 맞지 않아 버리는 것.
4. 클래스를 불변으로 만들 수 없게 된다. --> 객체의 일관성 유지가 되지 않는다는 것이다.

---

## 3. 빌더 패턴

```java
public class NutritionFacts {
    private final int servingSize;
    private final int servings;
    private final int calories;
    private final int fat;
    private final int sodium;
    private final int carbohydrate;

    public static class Builder {
        private final int servingSize;
        private final int servings;

        private int calories = 0;
        private int fat = 0;
        private int sodium = 0;
        private int carbohydrate = 0;

        public Builder(int servingSize, int servings) {
            this.servingSize = servingSize;
            this.servings = servings;
        }

        public Builder calories(int val) {
            calories = val;
            return this;
        }

        public Builder fat(int val) {
            fat = val;
            return this;
        }

        public Builder sodium(int val) {
            sodium = val;
            return this;
        }

        public Builder carbohydrate(int val) {
            carbohydrate = val;
            return this;
        }

        public NutritionFacts build() {
            return new NutritionFacts(this);
        }
    }

    private NutritionFacts(Builder builder) {
        servingSize = builder.servingSize;
        servings = builder.servings;
        calories = builder.calories;
        fat = builder.fat;
        sodium = builder.sodium;
        carbohydrate = builder.carbohydrate;
    }
}
```

위와 같이 Builder 패턴을 설계하면, NutritionFacts 클래스는 불변이며 모든 매개변수의 기본 값들을 한곳에 셋팅할 수 있게된다.

Builder 패턴을 사용하는 클라이언트의 코드를 보면 다음과 같다.

```java
NutritionFacts cocaCola = new NutritionFacts.Builder(240, 8)
    .calories(100)
    .sodium(35)
    .carbohydrate(27)
    .build();
```

### 빌더 패턴의 장점

1. 위와 같이 메서드 체이닝으로 간편하게 많은 매개변수를 다룰 수 있게 되며 가독성이 향상한다.
2. 객체 생성과 동시에 필드를 할당할 수 있다.
3. 계층적으로 설계된 클래스에 결합하기 좋다.

### 빌더 패턴의 단점

1. 패턴 구현이 다른 방법에 비해 복잡하다.

---

# 결론

생성자의 매개변수가 많을 때는 빌더 패턴이 적합해보인다. 추후 확장 가능성을 생각했을 때도

빌더 패턴이 객체를 생성하는 위 3가지의 방법 중 가장 유연한 패턴이다.

하지만 다른 방법이 무조건적으로 나쁘다고는 장담할 수 있는 사람이 많이는 없을 것이다.

그렇기 때문에 객체를 생성하는 각 방법의 장단점을 알아두는 것이 좋다.

또한 Spring Boot 기반에서 객체 생성을 다룰 때에는 Lombok이라는 어노테이션 프로세서 라이브러리가 있으니 이를 활용하면 빌더패턴을 직접 구현하지 않아도

쉽게 사용할 수 있으니 참고하면 좋을 것 같다.
