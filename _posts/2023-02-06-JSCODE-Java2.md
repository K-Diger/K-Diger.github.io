---

title: JSCODE - 자바 스터디 2주차
author: 김도현
date: 2023-02-06
categories: [Java, Study]
tags: [Java, Study]
math: true
mermaid: true

---

# 2회차 미션 - 채점기계 구현하기

## 아래와 같이 동작할 수 있도록 하자.

```text
몇 기인지 입력해주세요.
3
HTML 과목 점수를 입력해주세요.
60
CSS 과목 점수를 입력해주세요.
80
Javascript 과목 점수를 입력해주세요.
65
불합격입니다.
전체 과목 중 최고점은 80점입니다.
전체 과목 중 최저점은 60점입니다.
전체 과목의 평균은 68.33333333333333점입니다.
```

```text
몇 기인지 입력해주세요.
2
HTML 과목 점수를 입력해주세요.
63
CSS 과목 점수를 입력해주세요.
82
Javascript 과목 점수를 입력해주세요.
68
합격입니다.
전체 과목 중 최고점은 82점입니다.
전체 과목 중 최저점은 63점입니다.
전체 과목의 평균은 71.0점입니다.
```

```text
몇 기인지 입력해주세요.
3
HTML 과목 점수를 입력해주세요.
100
CSS 과목 점수를 입력해주세요.
100
Javascript 과목 점수를 입력해주세요.
0
합격입니다.
전체 과목 중 최고점은 100점입니다.
전체 과목 중 최저점은 100점입니다.
전체 과목의 평균은 66.66666666666667점입니다.
```

## 주의사항

JSCODE 1~3기가 시험을 봤다. 1, 2기는 평균 점수가 60점 이상이어야 합격이다.

3기는 평균 점수가 70점 이상이어야 합격이다.

다만, 100점 과목이 2개 이상일 경우 평균 점수와 상관없이 합격이다.

- 합격일 경우 `합격입니다.`라는 문구를 출력해야 한다.
- 불합격일 경우 `불합격입니다.`라는 문구를 출력해야 한다.

---

# 구현!

```java
package src.week2;

import java.util.Scanner;

public class Main {

    public static void main(String[] args) {
        execute();
    }

    private static void execute() {
        Scanner scanner = new Scanner(System.in);
        System.out.println("몇 기인지 입력해주세요.");
        int number = scanner.nextInt();
        System.out.println("HTML 과목 점수를 입력해주세요.");
        int htmlScore = scanner.nextInt();
        System.out.println("CSS 과목 점수를 입력해주세요.");
        int cssScore = scanner.nextInt();
        System.out.println("JavaScript 과목 점수를 입력해주세요.");
        int javaScriptScore = scanner.nextInt();

        if (number >= 3) {
            testForThirdAndOver(htmlScore, cssScore, javaScriptScore);
            return;
        }
        testForFirstAndSecond(htmlScore, cssScore, javaScriptScore);
    }

    private static void testForFirstAndSecond(int htmlScore, int cssScore, int javaScriptScore) {
        int sum = htmlScore + cssScore + javaScriptScore;
        double average = sum / 3.0;
        if (average >= 60) {
            System.out.println("합격입니다.");
            printMaxValueAndMinValue(htmlScore, cssScore, javaScriptScore, average);
            return;
        }
        System.out.println("불합격입니다.");
        printMaxValueAndMinValue(htmlScore, cssScore, javaScriptScore, average);
    }

    private static void testForThirdAndOver(int htmlScore, int cssScore, int javaScriptScore) {
        int sum = htmlScore + cssScore + javaScriptScore;
        double average = sum / 3.0;
        if (judgementUnconditionalPass(htmlScore, cssScore, javaScriptScore) || average >= 70) {
            System.out.println("합격입니다.");
            printMaxValueAndMinValue(htmlScore, cssScore, javaScriptScore, average);
            return;
        }
        System.out.println("불합격입니다.");
        printMaxValueAndMinValue(htmlScore, cssScore, javaScriptScore, average);
    }

    private static double calcMaxValue(int htmlScore, int cssScore, int javaScriptScore) {
        double max = Math.max(htmlScore, cssScore);
        return Math.max(max, javaScriptScore);
    }

    private static double calcMinValue(int htmlScore, int cssScore, int javaScriptScore) {
        double min = Math.min(htmlScore, cssScore);
        return Math.min(min, javaScriptScore);
    }

    private static void printMaxValueAndMinValue(int htmlScore, int cssScore, int javaScriptScore, double average) {
        System.out.println("전체 과목 중 최고점은 " + calcMaxValue(htmlScore, cssScore, javaScriptScore) + "점입니다.");
        System.out.println("전체 과목 중 최저점은 " + calcMinValue(htmlScore, cssScore, javaScriptScore) + "점입니다.");
        System.out.println("전체 과목의 평균은 " + average + "점입니다.");
    }

    private static boolean judgementUnconditionalPass(int htmlScore, int cssScore, int javaScriptScore) {
        int count = 0;
        if (htmlScore == 100) {
            count++;
        }
        if (cssScore == 100) {
            count++;
        }
        if (javaScriptScore == 100) {
            count++;
        }
        return count >= 2;
    }
}
```

## 배운 내용

우선 요구사항을 꼼꼼하게 파악하지 못해서 judgementUnconditionalPass()라는 메서드가 수행하는 역할을 인지하지 못했는데, 다행히도 동주님의 피드백으로 알게된 사항이라 급하게 반영했었다.

그리고 처음에는 평균을 구할 때 double 타입의 average를 int 타입인 3으로 나누었다.

그렇다 보니 평균을 출력할 때 반올림되어 나온 소수자리가 나오게되었는데 왜 그런지 파악하지 못했었다.

당연하게도 **실수형을 나눗셈 할 땐 실수**로 나눠야지 보다 더 정확한 소수점 자릿수가 나오게 된다!

실수를 표현하는 float과 double의 차이점에서도, 각 32bit, 64bit를 가진 타입으로 그 정확도도 double이 더 높다고 한다!

## 아쉬운 점

일단 마감 시간이 있어서 그런지, 입력 부분을 다듬지 못했던 것 같다.

**입력을 담당하는 메서드가 있어도 좋을 것 같다.**

그리고 **출력을 담당하는 메서드가 있으면 코드가 더 깔끔해지지 않을까?** 하는 생각이다.

물론 그 역할을 하는 객체를 만드는 것도 좋겠지만 지금 구현해놓은 서브 메서드 단위를 객체라고 생각한다면.. 그래도 전반적으로 좋은 코드는 아닌 것 같다..

일단 구현에 있어서 초점을 맞춘 것은 다음과 같다.

- 메인 메서드를 깔끔하게 표현하기
- 서브 메서드들의 길이가 너무 길지 않게 하기

잘 반영이 된 것인가는 나 혼자서 판단할 것은 아닌 것 같다. 팀원들과 코드 공유를하고 **멘토님의 피드백**이 있으면 좋을 것 같다!
