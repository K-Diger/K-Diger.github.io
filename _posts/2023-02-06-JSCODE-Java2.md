---

title: JSCODE - 자바 스터디 2회차 (입출력, 변수, 연산자, 형변환, 조건문) + 심화 미션(JVM, GC)
author: 김도현
date: 2023-02-06
categories: [Java, Study]
tags: [Java, Study]
math: true
mermaid: true

---

# 2회차 미션 - 시험 채점기 구현하기

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

---

# 2회차 심화 미션 - JVM, GC

## JVM

JVM은 Class Loader, Memory, Execution Engine으로 구성되어있다.

Java 언어로 작성된 코드를 기계어에 꽤나 가까운 Byte Code로 변환하는 역할을 하며 스스로의 메모리를 가지고 있는 가상머신이다.

## 구성요소 1. Class Loader

클래스 파일을 JVM 으로 로딩하는 역할을 수행하며, 이 때 타 언어로 작성된 네이티브 코드로 작성된 내용 또한 로딩 되는 것이다.

클래스 로더는 다음 과정을 거쳐 그 역할을 수행해낸다.

### Loading

.class 파일을 읽고 바이너리 데이터를 생성하여 **메서드 영역**에 저장한다.

이 때 로드된 클래스와 그 모든 연관 부모 클래스의 .class 파일이 Class 혹은 Interface, 혹은 Enum과 관련있는 것인지 체크하고 조건에 맞는다면 이들 또한 메서드 영역에 저장한다.

.class 로딩을 마친 후에 JVM은 **Heap 영역**에 이 파일들을 나타내기위해 Class타입의 객체를 생성한다. (getClass()메서드를 사용할 수 있는 이유!)

### Linking

Linking 과정은 아래 세 가지 과정을 통해 수행된다.

#### Linking - Verfication

.class 파일이 올바른 형식인지, 유효한 컴파일러에 의해 생성되었는지 확인한다.

확인에 실패한다면, RuntimeException 혹은 VerifyError가 발생한다.

이 과정을 성공적으로 수행한다면 .class 파일을 컴파일할 준비가 되어있는 것이다.

#### Linking - Preparation

클래스 변수에 메모리를 할당하고 메모리를 기본값으로 초기화한다.

#### Linking - Resolution

간접 참조 방식을 직접 참조 방식으로 변경하는 과정으로

참조된 엔티티를 찾기 위해 메서드 영역을 검색한다.

> 간접 참조와 직접 참조의 차이점을 알아보자 (2023.02.06)

### Initialization

모든 static 변수가 코드 및 static block 에 정의된 값으로 할당된다. (static 키워드가 들어간 요소들 메모리 할당)

static 요소들 메모리 할당 –> 부모 클래스 메모리 할당 –> 자식 클래스 메모리 할당 –> 기타 요소들 할당

---

## 구성요소 2. Memory

### Method 영역

- 클래스 구조(클래스 이름, 부모 클래스 이름, 메서드 및 변수 정보) 등 클래스 수준의 모든 정보를 담고 있다.

- static 변수를 가지고 있다.

- JVM 당 하나의 메서드 영역을 가지고 있고, 공유 될 수 있는 영역이다.

### Heap 영역(Eden Space (Young Generation), Suvivor Space, Tenured Space(Old Generation))

- 모든 객체 및 연관된 인스턴스 변수/배열이 저장되는 장소이다.

- 이 영역은 공유될 수 있으며 추후 GC 의 대상이 되는 동적인 영역이다.

### Stack 영역

- 각 스레드마다 배정되며 지역변수 저장과 연산에 주로 사용된다.

- JVM은 모든 쓰레드에, 각 하나의 런타임 스택을 제공한다.

- 각 스택의 모든 블럭은 메서드 호출이 발생하면 이를 스택에 저장한다.

- 지역 변수가 이곳에 저장된다.

- 공유 가능한 영역이다

### PC 레지스터

- 스레드가 현재 실행중인 JVM 명령의 주소를 저장한다. 각 스레드 별로 별도의 PC 레지스터가 있다.

### Native Method 영역

- Java가 아닌 다른 언어로 작성된 Native Code 의 명령을 저장한다.

---

## 구성요소 3. Execution Engine

JIT Compiler 와 GC 가 존재하는 영역으로, Native Code (타 언어로 만들어진 메서드 등) 을 다루는 영역이다.

구체적으로는 Interpreter, JIT 컴파일러, Garbage Collector 로 구성되어있다.

실행 엔진은 .class 파일을 실행시켜주며 byte 코드를 각 행마다 읽어들인다.

이 때 메모리 공간에 존재하는 데이터와 정보들을 이용하며 명령어를 실행시켜준다.

실행 엔진은 세 부분으로 분류 할 수 있다.

### Execution Engine - Interpreter

바이트 코드를 한줄 씩 해석한 다음 실행한다.

이 부분의 단점으로는, 하나의 메서드를 여러번 호출 할 경우, 매번 해석을 해줘야한다.


### Execution Engine - Just-In-Time Compiler(JIT)

Interpreter의 효율을 증가시키기 위해 사용된다.

전체 바이트 코드를 컴파일하여 네이티브 코드(기계어)로 변경하는 역할을 수행한다.

Interpreter는 반복되는 메서드 호출을 볼 때마다 매 번 재 해석 하는 것으로 효율을 떨어뜨리지만

JIT 에서 이미 전체 코드를 컴파일 해두었기 때문에 해당 부분에 관한 네이티브 코드를 제공할 수 있음으로

코드의 재 해석이 필요하지 않아 효율성을 증가시켜주는 방법이다.

전체 코드를 컴파일하는데 시간이 오래걸려서 Interpreter가 먼저 동작하고, 그 과정에서 Compiler 또한 동작하면서

서로의 단점을 보완해주는 관계이다.

### Execution Engine Garbage Collector

참조되지 않는 객체를 정리해준다.

---

## GC에 대해 설명하기

GC는 메모리(Heap영역)상의 상주하는 사용하지 않는 객체를 수집하는 JVM 실행 엔진의 일부이다.

![image](https://www.programmersought.com/images/220/8462f4bfba5f173585f6f8cd2dce12ac.JPEG)

JVM의 메모리 구조의 일부는 위와 같이 생겼다.

### Young Generation

새로 생성된 객체는 Young Generation 에서 시작한다. 모든 객체가 시작되는 1개의 Eden 공간과 Garbage Collection Cycle 에서 살아남은 후

Eden 공간에서 살아남은 객체들이 이동하는 공간인 2개의 Survivor 공간이 있다.

### Old Generation

Young Generation에서 수집되지 않은 객체들은 Young Generation 에서 Old Generation 으로 이동된다.

Old Generation 에서 Garbage Collected 가 되었을 때, 가장 우선순위가 높게 처리되는 공간이다.

### Permanent Generation (Metaspace)

클래스 및 메서드와 같은 메타데이터가 저장되는 장소이다.

추가적으로, Java 8 에 들어오면서, Permanent Generation 이 사라지고 Metaspace 영역이 생겼다.

Permanent Generation 은 JVM 에 의해서 Heap 영역의 메모리 크기가 강제되던 영역이였다.

하지만 Metaspace 가 Native 메모리 영역에 배치되면서, 운영체제가 자동으로 그 크기를 조절할 수 있게 되고, Heap 에서 사용할 수 있는 메모리의 크기가 늘어나게 됐다.

## GC 상태

### Minor/Incremental Garbage Collection

Young Generation 에서 객체가 제거 되었다는 것을 말하는 상태이다.

### Major/Full Garbage Collection

Minor Garbage Collection 에서 살아남은 객체를 Old 로 옮긴다.

Old 에서는 GC 대상이 덜 자주 발생하게 된다.
