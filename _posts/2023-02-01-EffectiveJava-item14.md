---

title: Effective-Java Item 14. Comparable을 구현할지 고려하라
author: 김도현
date: 2023-02-01
categories: [Effective-Java]
tags: [Comparable]
layout: post
toc: true
math: true
mermaid: true

---

# Comparable 인터페이스의 유일한 메서드 - compareTo()

compareTo() 메서드는 Object클래스의 equals와 매우 유사하다. 다른 점은 딱 두가지가 있는데

compareTo는 동치성 비교와 순서를 비교할 수 있고 제네릭으로 사용할 수 있다는 것이다.

또한 이 메서드를 구현했다는 것은 그 클래스의 인스턴스는 순서를 가진다는 것을 의미하는 것으로 아래와 같이 간단하게 정렬을 수행할 수 있다.

Comparable을 구현한 객체들의 배열은 다음과 같이 정렬 가능하다.

```java
Arrays.sort(a);
```

---

# compareTo() 메서드를 구현하는 것을 고려하기 전에 알고 갈 내용

```java
public class Main {
    public static void main(String[] args) {
        String myStr1 = "Hello";
        String myStr2 = "Hello";
        System.out.println(myStr1.compareTo(myStr2));
    }
}
```

위와 같이 같은 문자열을 대상으로 하여금 compareTo()메서드를 수행하면 어떤 결과가 출력될까?

True?, False?

<br>

정답은, 0이다.

## 그렇다면 왜 0을 반환하는가? 그것도 Integer 타입으로??

compareTo()의 존재 목적은 그저 같은 것인가만을 비교하는 것이 아니다.

글의 초반에도 작성했듯이 어떠한 순서까지 비교하기 때문인데,

그렇다면 Boolean으로도 순서와 값이 같은지 나타낼 수 있지 않은가? 라는 생각이 든다.

정렬을 수행하기 위해선 어떤 요소가 어떤 자리에 있어야 하는지 순서까지 알아야 할 필요가 당연하게도 있다.

그렇기 때문에 그저 순서가 다르다고 표한할 수 있는 Boolean과 순서가 어떻게 다르다고 말해줄 수 있는 Integer형 표현은

사용자에게 제공할 수 있는 정보의 경우의 수가 확실히 다르다.

또한 compareTo() 메서드는 매개변수를 딱 하나만 받는데, 이는 자기 자신과 매개변수를 비교하기 위함이다.

```java
public class Main {
    public static void main(String[] args) {

        System.out.println("------------ String CompareTo() ------------");
        String myStr1 = "Hello";
        String myStr2 = "Hello";
        System.out.println(myStr1.compareTo(myStr2));

        String myStr3 = "Hlelo";
        String myStr4 = "Hello";
        System.out.println(myStr3.compareTo(myStr4));

        String myStr5 = "Helo";
        String myStr6 = "Hello";
        System.out.println(myStr5.compareTo(myStr6));
        System.out.println("------------ String CompareTo() ------------");


        System.out.println("------------ Integer CompareTo() ------------");
        Integer myInteger1 = 1;
        Integer myInteger2 = 2;
        System.out.println(myInteger1.compareTo(myInteger2));

        Integer myInteger3 = 1;
        Integer myInteger4 = 1;
        System.out.println(myInteger3.compareTo(myInteger4));

        Integer myInteger5 = 5;
        Integer myInteger6 = 1;
        System.out.println(myInteger5.compareTo(myInteger6));
        System.out.println("------------ Integer CompareTo() ------------");
    }
}
```

위 코드는 아래와 같이 출력된다.

```java
------------String CompareTo()------------
    0
    7
    3
    ------------String CompareTo()------------
    ------------Integer CompareTo()------------
    -1
    0
    1
    ------------Integer CompareTo()------------

    종료 코드 0(으)로 완료된 프로세스
```

왜 일까 왜 도대체 이런 값이 나오지? 라는 의문이 들었다.

String 타입의 compareTo 메서드는 두 문자열을 사전식으로 비교한다.

그렇기 때문에 char 타입으로 한 글자씩 비교한다는 것이다.

이는 char 타입이 결국엔 ASCII 코드로 활용되는 것이기 때문에

서로 다른 문자의 아스키코드 값을 뺄셈하여 나타낸 값이다.

```java
        String myStr3="Hlelo";
    String myStr4="Hello";
    System.out.println(myStr3.compareTo(myStr4));
```

그래서 위와 같은 코드는, 두 번째 문자부터 그 값이 다르다는 것을 도출해내게 되고

'l' 의 ASCII 값인 108 'e' 의 ASCII 값인 101을 빼서 결과를 출력하게 되는 것이다!

Integer는 compareTo의 결과 값을 뱉는 과정이 그리 복잡하진 않다.

- 메서드를 실행하는 인스턴스가 더 작다면 -1
- 메서드를 실행하는 인스턴스가 같다면 0
- 메서드를 실행하는 인스턴스가 더 크다면 1

을 반환한다.

---

# 이정도 되었으니 Comparable을 구현할지 정해야하는 기준을 알아보자.

일단 우리가 주로 사용하는 값 클래스 및 열거타입은 모두 Comparable을 구현해뒀다.

이 말이 무슨 뜻이냐면, 알파벳 - 숫자 - 년도 등 순서가 명확한 값 클래스를 작성할 때는 반드시 Comparable 인터페이스를 구현해야한다는 것이다.

# Comparable을 구현할 때 주의할 점은 뭘까?

equals 규약과 똑같다. 이를 철저하게 지키긴해야한다.

> x.compareTo(x) == -y.compareTo(x)
> x.compareTo(x) > 0 && y.compareTo(z) > 0 이면, x.compareTo(z) > 0이다.
> x.compareTo(x) == y.compareTo(z) 이면, x.compareTo(z) == 0이다.
> (x.compareTo(x) == 0) == (x.equals(y))이다.

이 규약을 꼭 지켜야하는이유는, 정렬된 컬렉션인 TreeSet, TreeMap 검색과 정렬 알고리즘을 활용하는 Collections, Arrays에 있다.

TreeSet, TreeMap, Collections, Arrays 는 모두 정렬을 위한 과정에서 내부적으로 compareTo() 메서드를 사용하기 때문이다.

그리고 정렬된 컬렉션들은 동치성을 비교할 때 equals()메서드 대신, compareTo()메서드를 사용하기 때문에 이를 꼭 인지해야한다.

<br>

compareTo()메서드는 각 필드가 동치인지(==인지) 비교하는게 아니라 그 순서를 비교한다.

클래스에 핵심 필드가 여러 개라면 어떤 것을 먼저 비교하느냐가 중요해진다.

- 가장 핵심적인 필드를 먼저 비교한다.
- 비교 결과가 0이 아니라면, 즉 순서가 결정되었다면 거기서 결과를 반환해야한다.
- 가장 핵심이 되는 필드가 똑같다면, 똑같지 않을 때 까지 그 다음 필드와 비교해나간다.

```java
public class Main {
    public int compareTo(PhoneNumber phoneNumber) {
        int result = Short.compare(arearCode, phoneNumber.areaCode); // 가장 중요한 필드 (+82)
        if (result == 0) {
            result = Short.compare(prefix, phoneNumber.prefix); // 그 다음 중요한 필드 (010)
            if (result == 0) {
                result = Short.compare(lineNum, phoneNumber, lineNum); (xxx-1234-5678)
            }
        }
        return result;
    }
}
```
