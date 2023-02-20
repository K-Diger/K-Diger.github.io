---

title: JSCODE - 자바 스터디 6회차 예외(Exception), try-catch
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

    public PhoneNumber(String input) {
        this.phoneNumber = input;
    }

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

