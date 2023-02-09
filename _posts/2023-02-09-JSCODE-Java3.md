---

title: JSCODE - 자바 스터디 3회차 (반복문, 배열, List(ArrayList), 제네릭) + 심화 미션(JCF, Generic, String vs StringBuilder vs
StringBuffer)
author: 김도현
date: 2023-02-09
categories: [Java, Study]
tags: [Java, Study]
math: true
mermaid: true

---

# 3회차 미션 1. 학생들의 이름을 가나다 순으로 출력하기

## 요구사항

```text
학생의 이름을 입력하고 엔터를 누르세요. (한글로만 입력해야 합니다.)
학생들을 다 입력했다면, print라고 입력해주세요.
박재성
유재석
jason
학생의 이름은 한글로만 입력해야 합니다.
강호동
신동엽
print
[학생 명단(가나다순)]
강호동
박재성
신동엽
유재석
```

위와 같이 동작할 수 있도록 하자!

## 주의사항

- 배열(int[], String[] 등)을 사용하지 말고, ArrayList를 사용해라.

- ArrayList를 사용할 때, 제네릭(Generic)을 사용해라.

- 입력값이 한글 또는 print가 아니라면, 학생의 이름은 한글로만 입력해야 합니다. 라는 문구가 출력된다.

---

## 구현

```java
package src.class3;

import java.util.ArrayList;
import java.util.Collections;
import java.util.List;
import java.util.Scanner;
import java.util.regex.Pattern;

public class PrintStudentMain {
    public static void main(String[] args) {
        execute();
    }

    private static void execute() {
        printGuideMessage();
        printStudents(inputStudents());
    }

    private static void printGuideMessage() {
        System.out.println("학생의 이름을 입력하고 엔터를 누르세요. (한글로만 입력해야합니다.)");
        System.out.println("학생들을 다 입력했다면, print라고 입력해주세요.");
    }

    private static void printStudents(List<String> students) {
        System.out.println("[학생 명단(가나다순)]");
        for (String student : students) {
            System.out.println(student);
        }
    }

    private static void printWarning() {
        System.out.println("학생의 이름은 한글로만 입력해야 합니다.");
    }

    private static List<String> inputStudents() {
        Scanner scanner = new Scanner(System.in);
        List<String> students = new ArrayList<>();
        while (true) {
            String inputValue = scanner.nextLine();
            if (inputValue.equals("print")) {
                return sortStudents(students);
            } else if (Pattern.matches("^[a-zA-Z]*$", inputValue)) {
                printWarning();
            }
            students.add(inputValue);
        }
    }

    private static List<String> sortStudents(List<String> students) {
        Collections.sort(students);
        return students;
    }
}
```

## 배운 내용

- Collections.sort() 메서드는 반환 값이 없다! 해당 메서드를 return 값으로 반환하려 하니 컴파일에러가 발생해서 알게된 내용이었다.

- 처음에는 if (inputValue.equals("print")) 구문을 else if 구문으로 넣었었다. 그러다 보니 print 구문으로 while이 끝나지 않고 정규식으로 필터링 되는 현상이 발생했었는데,
  조건문의 위치 또한 꼼꼼하게 신경 써야한다!

## 아쉬운 점

- while 문 내부가 좀 지저분 한 것 같다. 조건문을 더 메서드로 분리할 수 있었지 않았을까?

---

# 3회차 미션 2. 100m 달리기 선수 중 1등 찾기

## 요구사항 (예시에 살짝 오류가 있는 것 같다?!)

```text
선수의 번호를 입력하세요.
13
이 선수의 100m 달리기 기록이 몇 초인지 입력하세요.
13.56
선수의 번호를 입력하세요.
7
이 선수의 100m 달리기 기록이 몇 초인지 입력하세요.
15.153
선수의 번호를 입력하세요.
7
이 선수의 100m 달리기 기록이 몇 초인지 입력하세요.
12.157
선수의 번호를 입력하세요.
2
이 선수의 100m 달리기 기록이 몇 초인지 입력하세요.
14
선수의 번호를 입력하세요.
print
1등 : 7번 선수 / 12.16초 (참가인원 : 3명)
```

## 주의사항

- 똑같은 선수 번호를 입력할 경우, 새로운 기록으로 갱신한다.

- 100m 달리기 기록을 입력할 때, 소숫점 둘째자리까지 반올림하여 기록한다.

- print 라고 입력하면 1등의 선수를 출력한다.

- 배열(int[], String[] 등)을 사용하지 말고, ArrayList를 사용해라.

- ArrayList를 사용할 때, 제네릭(Generic)을 사용해라.

---

## 구현

```java
package src.class3;

import java.util.ArrayList;
import java.util.List;
import java.util.Scanner;

public class WhoIsNumberOneMain {

    public static void main(String[] args) {
        List<Integer> numbers = new ArrayList<>();
        double bestRecord = 0.0;
        double record = 0.0;
        int bestPlayerIndex = 0;
        execute(numbers, bestRecord, record, bestPlayerIndex);
    }

    private static void printInputNumberGuide() {
        System.out.println("선수의 번호를 입력하세요.");
    }

    private static void printInputRecordGuide() {
        System.out.println("이 선수의 100m 달리기 기록이 몇 초인지 입력하세요.");
    }

    private static void printBestRecordAndPlayer(int number, double record, int numbersLength) {
        System.out.println(
            "1등 : " + number + "번 선수 / "
                + String.format("%.2f", record) + "초 (참가인원 : " + numbersLength + "명)");
    }

    private static String inputNumber() {
        Scanner scanner = new Scanner(System.in);
        printInputNumberGuide();
        return scanner.nextLine();
    }

    private static String inputRecord() {
        Scanner scanner = new Scanner(System.in);
        printInputRecordGuide();
        return scanner.nextLine();
    }

    private static void execute(List<Integer> numbers, double bestRecord, double record, int bestRecordIndex) {
        int index = 0;
        while (true) {
            String inputValue = inputNumber();
            if (inputValue.equals("print")) {
                printBestRecordAndPlayer(numbers.get(bestRecordIndex), bestRecord, numbers.size());
                break;
            }

            if (numbers.contains(Integer.valueOf(inputValue))) {
                numbers.set(numbers.indexOf(Integer.valueOf(inputValue)), Integer.valueOf(inputValue));
            } else {
                numbers.add(Integer.valueOf(inputValue));
            }

            inputValue = inputRecord();
            record = Double.parseDouble(inputValue);
            index++;

            if (record > bestRecord) {
                bestRecord = record;
                bestRecordIndex = index;
            }
        }
    }
}
```

## 배운 내용

- List.set(인덱스, 값) 으로 특정 인덱스의 값을 변경할 수 있는 점이 새로웠다!!

- List.indexOf(인덱스를 찾고자 하는 값) 으로 특정 값에 대한 인덱스를 가져올 수 있다!

## 아쉬운 점

- 무엇인가 많이 아쉽다. main 코드에 execute() 메서드 외의 값이 있어야 한다는 점도 뭔가 마음에 들지 않고

- execute() 메서드의 본문이 너무 길다는 점도 마음에 들지 않는다..

- 최고 기록을 갱신해 줄 때 그 최고기록에 대한 인덱스를 어떻게 더 간결하게 관리할 수 있을까?

- 선수 번호 입력과 선수 기록 입력에 대한 로직을 메서드 단위로 분리하고 싶은데 왜 못했을까?

- static 변수로 execute()메서드에서 사용되는 변수를 관리하면 쉽게 해결할 것 같은데, static은 지양하라는 글을 공유받았기 때문에 더 좋은 방법이 있을 것 같다.

- 이 미션은 반드시 리팩터링 해보자!

---

# 3회차 심화 미션 - JCF, Generic, String vs StringBuilder vs StringBuffer

## JCF 란

데이터를 저장하는 자료구조 및 이를 다루는 알고리즘을 클래스로 구현해 놓은 Framework의 일종이다.

> 나는 Collection 을 이야기할 때 "컬렉션 라이브러리"라고 해왔는데 정확히는 Freamwork 였다!

## JCF을 사용하면 뭐가 좋을까?

### 코드의 재사용성 증가

당연하게도 직접 구현할 필요도 없고 이미 만들어진 클래스들을 재사용하며 사용할 수 있게 된다.

### 나보다 뛰어난 자료구조와 알고리즘의 구현

직접 자료구조를 구현하고 그를 다루는 알고리즘을 구현한다고 한들 나는 수 많은 개발자들의 손을 거쳐간

컬렉션 프레임워크의 내용을 따라잡을 수 없다. 그렇기 때문에 성능상의 이점을 가져갈 수 있다.

## Generic이란?

Java는 기본적으로 Object라는 객체의 하위 객체를 사용하기 때문에

다양한 타입의 객체를 주고 받고 싶을 때 Object라는 타입을 사용하기도 하는데

이렇게 하면 매번 사용하고자 하는 타입으로 캐스팅 해줘야하는 불편함이 있었고 이를 개선하기 위해 등장한게 Generic 이다.

## Genric을 사용하면 뭐가 좋을까?

Generic을 활용해서 사용하고자 하는 타입을 먼저 지정하면 어떤 객체 타입이든 자동으로 캐스팅 되는 편리함을 얻을 수 있다.

- 예시 코드

```java
public class MyCustomList<T> {

    private List<T> list = new ArrayList<>();

    public void addElement(T element) {
        list.add(element);
    }

    public void removeElement(T element) {
        list.remove(element);
    }

    public T get(int index) {  // T를 반환
        return list.get(index);
    }
}

public class GenericMain {
    public static void main(String[] args) {
        MyCustomList<String> stringList = new MyCustomList<>();
        stringList.addElement("string1");
        stringList.addElement("string2");
        stringList.get(0);

        MyCustomList<Integer> intList = new MyCustomList<>();
        intList.addElement(1);
        intList.addElement(2);
        intList.get(0);
    }
}
```

---

## String, StringBuilder, StringBuffer 의 차이점

[참고자료](https://www.digitalocean.com/community/tutorials/string-vs-stringbuffer-vs-stringbuilder)

### String

String은 Java의 기본 타입으로, 불변 객체로 만들어져 있다. 즉, 어디서 어떻게 접근하던지 절대 변하지 않기 때문에 멀티 스레드 환경에서도 사용하기 적합하다는 특징이 있다.

### StringBuffer

StringBuffer는 String이 불변 객체인 이유로 값을 변경할 수 없기 때문에 문자열을 다룰 때 매 번 새로운 문자열을 생성하게 되는 점을 보완하기 위해 등장했다.

또한 StringBuffer는 가변 객체이며 스레드 간 동기화된다는 점으로 스레드 안전하다는 특징이 있다.

즉, 여러개의 메서드가 동시에 한 StringBuffer에 접근해도 내부 값이 동기화 되어있기 때문에 동시성 문제를 예방할 수 있다.

### StringBuilder

StringBuilder도 마찬가지로 문자열 조작을 위해 등장한 것인데, StringBuffer 보다 조금 더 늦게 등장했다.

그 이유로는 StringBuffer는 모든 공용 메서드를 동기화 한다는 점 때문에 성능이 좋지 않다는 내용이 있었기 때문이다.

Java 1.5 버전부터 스레드 안전성을 포기하고 문자열을 다룰 수 있는 StringBuilder로 성능상의 이점을 얻으며 문자열을 다룰 수 있게 되었다.
