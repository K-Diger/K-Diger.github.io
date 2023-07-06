---

title: Item 10. equals는 일반 규약을 지켜 재정의하라
author: Diger
date: 2022-12-10
categories: [Effective-Java]
tags: [Equals]
layout: post
toc: true
math: true
mermaid: true

---

## 들어가기 전

Object 는 객체를 만들 수 있는 구체 클래스이다. 따라서 상속하여 사용하도록 설계되어있다.

따라서 Object 클래스 내에 final 이 아닌 메서드(equals, hashCode, toString, clone, finalize) 는 모두 Override를 염두에 두고 설계된 것이다.

이때 재정의 시 지켜야하는 일반적인 규약이 명확하게 규정되어있다.

Object 메서드들을 언제 어떻게 재정의 해야하는지 알아보기 위한 내용이 다음에 이어진다.

일단 기본적으로 Object 클래스 내의 equals 메서드는 다음과 같이 구성되어있다.

    public boolean equals(Object obj) {
        return (this == obj);
    }

## equals 재정의 시 함정

다음과 같은 4가지 상황 중 단 한가지라도 포함된다면 재정의 하지 않아야한다.

### 1. 각 인스턴스가 본질적으로 고유하다.

값을 표현하는 것이 아닌, 동작하는 개체를 표현하는 클래스를 가리키는 것으로, Thread 가 그 에씨이다.

Object 클래스의 이미 구현되어있는 equals 메서드는 이러한 클래스에 딱 맞게 구현되어있다.

조금 더 덧붙이자면, Thread 라는 객체를 100개를 할당했을 때 각 객체는 스스로가 고유함을 지니고 있다.

단순하게

    String str1 = "test"
    String str2 = "test"

    System.out.println(str1.equals(str2));

처럼 값을 비교하는 것이 아닌 Thread 처럼 객체 끼리를 비교하는데에 있어서는 굳이 재정의 할 필요가 없다는 말이다.

### 2. 인스턴스의 논리적 동치성(logical equality)를 검사할 필요가 없다.

예를 들어, 정규식에 활용하기 위한 Pattern 인스턴스가 2개 이상 있다고 했을 때 이 인스턴스들이

같은 정규식을 나타내는지 확인하기 위해 equals를 재정의 할 필요가 없는 상황인 것이다.

조금 더 덧붙이자면, 각 인스턴스마다 수행하는 로직(알고리즘)을 equals로 검사할 필요가 없다는 것이다. (만약 이걸 노린거라면 기존의 equals로도 충분하다.)

### 3. 상위 클래스에서 이미 재정의한 equals가 하위 클래스에도 딱 들어맞는다.

Set 구현체는 AbstractSet이 구현한 equals를 상속받아 사용하고, List 구현체는 AbstractList, Map 구현체들은 AbstractMap으로부터 상속받아 그대로 쓴다.

이 부분은 소제목 그대로 상위 클래스에서 정의된 내용이 이미 충분하다면 굳이 재정의 하지 말아야 한다는 것을 말한다.

### 4. 클래스가 private 이거나, package-private 이고, equals 메서드를 호출할 일이 없다.

애초에 호출할 일이 없는데 재정의를 할 비용을 감내해야할 이유도 없다.

---

## equals를 재정의 해야하는 상황

> 객체 식별성(두 객체가 물리적으로 같은가)이 아니라 논리적 동치성(객체 내부에서 나타내고자 하는 값이나 표현이 같은가?)을 확인해야하는데,
>
> 상위 클래스의 equals가 논리적 동치성을 비교하도록 재정의 되지 않았을 때이다.
>
> 주로 값 클래스들이 여기에 해당한다. (값 클래스란? Integer, String 등 과 같이 값을 표현하는 클래스를 말한다.)

두 값 객체를 equals로 비교하고자 하는 행위는, 객체가 일치한지가 아니라, 그 객체가 가지고 있는 값이 일치한것을 알고 싶을 것이다.

equals가 논리적 동치성을 확인하도록 재정의해두면 그 인스턴스는 값을 비교하길 원하는 요구사항에 맞출 수 있다.

> 여기서 우리가 직접 String 클래스를 사용할 때 equals를 재정의할 필요까진 없다. (이미 구현되어있음)

### Object.java

    public boolean equals(Object obj) {
        return (this == obj);
    }

### String.java

    public boolean equals(Object anObject) {
        if (this == anObject) {
            return true;
        }
        if (anObject instanceof String) {
            String aString = (String)anObject;
            if (coder() == aString.coder()) {
                return isLatin1() ? StringLatin1.equals(value, aString.value)
                                  : StringUTF16.equals(value, aString.value);
            }
        }
        return false;
    }

---

## equals 메서드를 재정의할 때는 반드시 일반 규약을 따라야 한다.

### equals 메서드는 동치관계를 구현하며, 다음을 만족한다.

| 속성       | 설명                                                                                           |
|----------|----------------------------------------------------------------------------------------------|
| 반사성      | null이 아닌 모든 참조 값 x에 대해, x.equals(x)는 true이다.                                                 |
| 대칭성      | null이 아닌 모든 참조 값 x, y에 대해, x.equals(y)가 true 면, y.equals(x)도 true이다.                         |
| 추이성      | null이 아닌 모든 참조 값 x, y, z에 대해, x.equals(y)가 true이고, y.equals(z)도 true이면, x.equals(z)도 true이다. |
| 일관성      | null이 아닌 모든 참조 값 x, y에 대해, x.equals(y)를 반복해서 호출하면, 항상 true를 반환하거나 항상 false를 반환한다.            |
| not-null | null이 아닌 모든 참조 값 x에 대해, x.equals(null)은 false이다.                                             |



#### 1. 반사성 : 객체는 자기 자신과 같아야한다.

#### 2. 대칭성 : 두 객체는 서로에 대한 동치 여부에 똑같이 답해야 한다.


    public final class CaseInsensitiveString {
        private final String s;

        public CaseInsensitiveString(String s) {
            this.s = Objects.requireNonNull(s);
        }

        @Override
        public boolean equlas(Object o) {
            if (o instanceof CaseInsensitiveString)
                // 주목해야하는 부분, String.equalsIgnoreCase
                return s.equalsIgnoreCase(((CaseInsensitiveString) o).s);
            if (o instacneof String)
                // 주목해야하는 부분, String.equalsIgnoreCase
                return s.equalsIgnoreCase((String) o);
            return false;
        }
    }

위와 같은 코드가 있을 때 아래 상황에 주목해보자

    CaseInsensitiveString cis = new CaseInsensitiveString("Polish");
    String s = "polish";
    System.out.println(cis.equals(s));
    System.out.println(s.equals(cis));

결과는 각각 true, false를 반환한다.

CaseInsensitiveString에서의 equals 메서드의 구현내부를 보면 String.equalsIgnoreCase 를 사용함으로써

대소문자를 구별하지 않기 때문에 true를 반환하게 되는 것이다.

이는 대칭성 위반이다.

#### 3. 추이성 : 첫 번째 객체와 두 번째 객체가 같고, 두 번째 객체와 세 번째 객체가 같으면, 첫 번째 객체와 세 번째 객체가 같다.

    public class Point {
        private final int x;
        private final int y;

        public Point(int x, int y) {
            this.x = x;
            this.y = y;
        }

        @Override
        public boolean equals(Object o) {
            if (!(o instanceof Point))
                return;
            Point p = (Point) o;
            return p.x == x && p.y == y;
        }
    }


    public class ColorPoint extends Point {
        private final Color color;

        public ColorPoint(int x, int y, Color color) {
            super(x, y);
            this.color = color;
        }
    }

이러한 두 개의 클래스가 있을 때, Point 클래스에서 equals 메서드를 수정하지 않으면, 색깔 정보를 포함한 equals 로직을 수행할 수 없게 된다.

이를 해결하고자 하는 방안으로 위치와 색상이 같을 때만 true를 반환하는 equals로 수정했다고 해보자.

    public class ColorPoint extends Point {
        private final Color color;

        @Override
        public boolean equals(Object o) {
            if (!(o instanceof Point))
                  return;
            return super.equals(o) && ((ColorPoint) o).color == color;
          }
        }
    }

return super.equals(o) && ((ColorPoint) o).color == color;
return super.equals(((ColorPoint) o).color) && ((ColorPoint) o).color == o.color;

Point의 Equals(super.equals)는 색상을 무시하고, ColorPoint의 equals는 입력 매개변수의 클래스 타입이 달라 false를 반환하게 된다.

대칭성을 위배하는 코드가 된 것이다.


이를 올바르게 사용하면 다음과 같다.

    public class ColorPoint extends Point {
        private final Color color;
        private final Point point;

        public ColorPoint(int x, int y, Color color) {
            point = new Point(x, y);
            this.color = Objects.requiredNonNull(Color);
        }

        @Override
        public boolean equals(Object o) {
            if (!(o instanceof Point))
                  return false;

            ColorPoint cp = (ColorPoint) o;
            return cp.point.equals(point) && cp.color.equals(color);
        }
    }

#### 4. 일관성 : 두 객체가 같다면 앞으로도 영원히 같아야한다.

말 그대로다 테스트를 통해 꼼꼼하게 점검해야할 요소이다.


---

## 실질적으로 equals 메서드를 작성하는 팁!

1. == 연산자를 사용해 입력이 자기 자신의 참조인지 확인한다. (자기 자신을 참조한다면 true를 반환한다.)
2. instanceof 연산자로 입력이 올바른 타입인지 확인한다. (그렇지 않다면 false를 반환한다.)
3. 입력을 올바른 타입으로 형변환한다.
4. 입력 객체와 자기 자신의 대응되는 필드들이 모두 일치하는지 검사한다. (모든 필드 중 단 하나라도 일치하지 않으면 false 를 반환한다.)

> 어떤 필드를 먼저 비교하느냐가 equals의 성능을 좌우한다. 가장 좋은 성능을 취하고 싶다면, 다를 가능성이 크거나 비교하는 비용이 싼 필드를 비교한다.
>
> 이때, 동기화용 lock 필드와 같이 객체의 논리적 상태와 관련없는 필드는 비교하면 안된다.
