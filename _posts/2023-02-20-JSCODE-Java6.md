---

title: JSCODE - 자바 스터디 6회차 예외(Exception, try-catch), 심화 미션 (Call By Ref, Call By Value, Equals And HashCode)
author: 김도현
date: 2023-02-16
categories: [Java]
tags: [Exception, Try-Catch]
math: true
mermaid: true

---

# 요구 사항

```java
public class Main {
    public static void main(String[] args) {
        Scanner scanner = new Scanner(System.in);
        boolean isContinued = true;
        while (isContinued) {
            System.out.println("숫자를 입력해주세요.");
            String input = scanner.nextLine();
            int number = parseInt(input);
            System.out.println("입력하신 숫자는 " + number + "입니다.");
        }
    }
}
```

위 코드를 활용하여 아래와 같이 동작하도록 하자.

```text
숫자를 입력해주세요.
123
입력하신 숫자는 123입니다.
숫자를 입력해주세요.
abcd
잘못된 값을 입력하셨습니다.
숫자를 입력해주세요.
1234
입력하신 숫자는 1234입니다.
숫자를 입력해주세요.
```

# 주의할 점

- try-catch를 활용하기

- 잘못된 값(숫자가 아닌 값)을 입력할 시 Exception을 발생시키기

---

# 구현

## Main.java

```java
package src.class6;

public class Main {

    public static void main(String[] args) {
        Starter starter = new Starter(
            new PhoneNumberValidator(), new InputAgent(), new PrintAgent());
        starter.start();
    }
}
```

## Stater.java
```java
package src.class6;

public class Starter {

    private final PhoneNumberValidator phoneNumberValidator;
    private final InputAgent inputAgent;
    private final PrintAgent printAgent;

    private boolean isContinued = true;

    public Starter(PhoneNumberValidator phoneNumberValidator, InputAgent inputAgent,
        PrintAgent printAgent) {
        this.phoneNumberValidator = phoneNumberValidator;
        this.inputAgent = inputAgent;
        this.printAgent = printAgent;
    }

    public void start() {
        while (isContinued) {
            try {
                printAgent.executeInputGuide();
                PhoneNumber phoneNumber = new PhoneNumber(inputAgent.execute());

                phoneNumberValidator.executeNumberLength(phoneNumber.getPhoneNumber());
                phoneNumberValidator.executePreSignedNumber(phoneNumber.getPhoneNumber());
                phoneNumberValidator.executeCanConvertInteger(phoneNumber.getPhoneNumber());

                printAgent.executeSuccessGuide();
                printAgent.executeInputPhoneNumber(phoneNumber.convertToPhoneNumber());
                isContinued = false;
            } catch (IllegalArgumentException illegalArgumentException) {
                System.out.println(illegalArgumentException.getMessage());
            }
        }
    }
}
```

## PhoneNumber.java

```java
package src.class6;

public class PhoneNumber {

    private String phoneNumber;
    private String phoneNumberForm;

    private PhoneNumber() {}

    public PhoneNumber(String input) {
        this.phoneNumber = input;
    }

    private PhoneNumber(String input, String input2) {}

    public String getPhoneNumber() {
        return phoneNumber;
    }

    public String convertToPhoneNumber() {
        this.phoneNumberForm = phoneNumber.substring(0, 3) +
            "-" + phoneNumber.substring(3, 7) +
            "-" + phoneNumber.substring(7, 11);
        return phoneNumberForm;
    }
}
```

## PhoneNumberValidator.java

```java
package src.class6;

import static java.lang.Integer.parseInt;

public class PhoneNumberValidator {

    protected void executeCanConvertInteger(String input) {
        try {
            parseInt(input);
        } catch (NumberFormatException numberFormatException) {
            throw new IllegalArgumentException("휴대폰 번호는 숫자여야 합니다.");
        }
    }

    protected void executePreSignedNumber(String input) {
        if (!input.startsWith("010")) {
            throw new IllegalArgumentException("휴대폰 번호는 010으로 시작해야합니다.");
        }
    }

    protected void executeNumberLength(String input) {
        if (input.length() != 11) {
            throw new IllegalArgumentException("휴대폰 번호는 11글자여야 합니다.");
        }
    }
}
```

## InputAgent.java

```java
package src.class6;

import java.util.Scanner;

public class InputAgent {

    private final Scanner scanner = new Scanner(System.in);

    protected String execute() {
        return scanner.nextLine();
    }
}
```

## PrintAgent.java

```java
package src.class6;

public class PrintAgent {

    protected void executeInputGuide() {
        System.out.println("휴대폰 번호를 입력해주세요.");
    }

    protected void executeSuccessGuide() {
        System.out.print("휴대폰 번호를 정상적으로 입력하셨습니다. ");
    }

    protected void executeInputPhoneNumber(String phoneNumberForm) {
        System.out.println("입력하신 휴대폰 번호는 " + phoneNumberForm + "입니다.");
    }
}
```

---

# 구현 시 주목한 점

### 한 클래스는 하나의 책임? 역할? 을 수행한다.

객체지향의 사실과 오해에서 등장하는 용어인 책임, 역할에 대해서 아직 헷갈리긴 하다.

책임은 무엇이고 역할은 무엇일까?

지금 생각하고 있는 것은 책임은 어떤 객체가 수행해야하는 역할을 모아놓은? 느낌으로 알고 있는데 이게 맞는지 잘 모르겠다.

(멘토님 피드백 부탁드립니다 ㅠㅠ)

### 우아한 테크코스(프리코스)에서 받았던 피드백을 최대한 반영해보자

우선 기억나는 피드백만 나열해보자면

- 메서드 길이를 가능한 짧게 (10줄 이내로 짜라는 것이 마지막 주차 요구사항이었던 것 같았다.)

Starter.start() 메서드는 현재 약 17줄 정도 되는데... 흠 과연 더 줄일 순 없었을까?

- 무분별한 getter를 지양해라 그 객체에서 처리할 수 있는 로직을 작성한다면 getter가 굳이 필요 없을 것이다.

- 네이밍을 센스있게! 이름만으로도 알아볼 수 있도록 해보자 또한 축약을 자주 하진 말자.

- 클래스는 상수, 멤버변수, 생성자, 메서드 순으로 작성해야한다.

- 한 메서드가 하나의 일만 하도록 하자.

- 비즈니스 로직과 UI 로직을 분리하자.


그 외의 더 많은 피드백이 있었지만... 프리코스 광탈 후 마음이 아파 피드백을 몇 번이고 되돌아 보지 않은 탓인지

기억이 잘 안난다. 이 글을 포스팅 마치고 꼭 다시 한 번 살펴봐야겠다!


그리고 기억난 피드백이라도 잘 적용이 되었을까? 물론 요구사항 자체가 그리 크진 않아서 쉽게 해냈을 거라고도 생각은 하지만

이런 생각이 근자감이 될 수 있기 때문에 멘토님의 피드백이 꼭 받고 싶다! (이 항목도 피드백 주시면 감사하겠습니다!!)


# 멘토님께

구현을 하면서 위에서 언급한 내용이 궁금했습니다. "객체지향의 사실과 오해"에서 등장하는 용어들인

**역할**, **책임**, **협력**, **메세지**를 이번 회차 요구사항으로 작성한 코드로 대입한다면 어떻게 이해하면 될지 ... 궁금합니다.

용어를 코드로 대입한다는 의미는 다음과 같이 생각하고 있는 것입니다!

- 애플리케이션을 실행하는 **역할** = Starter

- Starter의 **책임** = start()

- Stater의 **협력**관계 = PhoneNumberValidator, InputAgent, PrintAgnet

- Starter의 **메세지** = phoneNumberValidator.executeNumberLength(phoneNumber.getPhonNumber());


이 부분은 설명해주시기 조금 애매한 내용이거나 스스로 생각해봐야 하는 것이면 조금 더 고민해보겠습니다!

---

# Call By Value vs Call By Reference

```java
public class Main {

  public static void main(String[] args) {
    Money money1 = new Money(500);
    Money money2 = new Money(500);
    System.out.println(money1 == money2);
    System.out.println(money1.equals(money2));
  }
}
```

위 코드의 실행 결과가 모두 false 로 나오는 이유는 Call By Value vs Call By Reference에 있다.

Java 는 Call by value로 동작하며 이 방식은 "값을 전달하는 방식" 이다.

그러니까 Money타입을 가지고, 그 내부 **값**을 500으로 가진 인스턴스를 각각 만들어 놓는 코드이기 때문에

위 코드는 어떤 비교연산을 하여도 다르게 나올 수 밖에 없다.

500이라는 값을 가진 참조 값을 토대로 만들어진 인스턴스가 아니라, 500이라는 **값 그자체를** 가진 서로 다른 인스턴스를 만들게 된 것!

- 이 부분에서 틀린 부분이 있다면 피드백 부탁드립니다...!

---

# Equals (동일성, 동등성), HashCode

## Equals - 동일성

equals()는 2개의 객체가 동일한 객체인지 검증한다.

equals는 Object라는 클래스에 구현되어있으며 Java는 기본적으로 모든 객체가 Object라는 부모를 상속 받아 만들어 졌기 때문에

모든 클래스에서 equals()가 있다고 생각하면 된다.

equlas는 두 개의 객체의 **참조**가 동일한지 검증한다. 그리고 이를 동일성 검증 이라고한다.

인텔리제이 덕을 많이 본 덕분에 우리는 다음과 같이 String 간의 == 연산은 사용하지 않는다.
```java
String string1 = new String("Diger");
String string2 = new String("Diger");

System.out.println(string1 == string2);
System.out.println(string1.equals(string2));
```

인텔리제이가 "==" 연산을 권장 사항(equals)으로 변경해주는 이유도 String이 가진 특징인 Wrapper Class 라는 것 때문에 발생하는 동일성 비교 문제 때문이다.

String 타입은 Integer, Long 등과 같이 객체로써 여겨진다. 그리고 이것을 Wrapper Class라고 하는데 이 타입들은 Object를 상속받은 값 타입 객체이기 때문에

```java
String string1 = new String("Diger");
String string2 = new String("Diger");
```
와 같이 같은 값을 가진 객체라 한들, 다른 인스턴스를 생성하는 것이기 때문에 참조값을 비교하는 연산인 == 는 당연하게도 false가 떨어지게 된다.

밑에서 다루겠지만 같은 객체라면, 동일성이 같다면 같은 HashCode값을 가진 다는 것도 중요한 점이다.

## Equals() - 동등성

동등성은 객체가 **가지고 있는 값이 같은지** 검증하는 것이다.

Equals() 연산은 기본적으로 이 동등성을 검증하기 때문에

```java
String string1 = new String("Diger");
String string2 = new String("Diger");

System.out.println(string1.equals(string2));
```
이 출력 결과가 True가 나올 수 있게 되는 것이다.

## HashCode

hashCode() 메서드는 객체의 유일한 intger값을 반환하는데 이는 런타임에 JVM에 할당된 Heap 주소를 반환하는 것이다.

이 메서드는 주로 HashMap, HashTable 등 해시값을 사용하는 자료구조에서 주로 사용되며

객체마다 유일한 Hash값을 가졌다는 특징이 있다.

## Equals와 HashCode

- 동일한 객체는 동일한 메모리 주소를 갖는다. 그러므로 동일한 객체는 동일한 특정 Heap영역을 가리키는 해시코드를 가져야 한다.

그렇기 때문에 동일성을 비교하기 위해 equals() 메서드를 Override 한 객체가 있다면

hashCode() 메서드도 Override 해야 논리적으로 허점이 없게 된다.

equals 메서드를 구현해서 커스텀한 동일성 비교 로직을 만들고 그 동일성 비교의 결과를 바탕으로

hashCode를 활용하려하니 다른 값을 가리키는 결과가 나온다면? 아마 원인을 쉽게 인지하지 못할 수 있다.

따라서 Equals를 Override 했다면, HashCode를 재정의 해야한다. (이펙티브 자바 Item. 11)

---

# Wrapper Class

![Wrapper Class](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2Fbvzp79%2FbtqEbacB01v%2FQQjO7cSc9tTvKJkyzFsK90%2Fimg.png)

위 이미지가 래퍼 클래스를 나타내는 그림이다.

Java가 가진 기본 타입(char, int, float, double, boolean)들을 객체로써 사용하기 위함에 존재한다.

이렇게 객체로써 사용하면 어떤 것이 장점인가? 하면

이 Wrapper Class로 감싸진 기본 타입의 값들은 외부에서 변경할 수 없다는 점이다.

```java
Integer integer = new Integer(2);
integer.setValue(3);
```
위와 같은 코드를 절대로 본 적 없을 것이다.

이렇게 값을 외부에서 함부로 변경하지 못하도록 할 수 있으며 부가적으로 java.util 패키지를 활용할 수 있는 타입이다.

더 나아가서, 멀티쓰레딩 환경에서의 동기화를 지원하기 위해서는 Wrapper Class 변수가 필요하다.
