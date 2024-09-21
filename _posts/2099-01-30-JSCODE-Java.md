---

title: JSCODE - 자바 스터디

date: 2023-01-30
categories: [Java, Study]
tags: [Java, Study]
layout: post
toc: true
math: true
mermaid: true

---

# 1회차 미션

- 1. Java 설치하기

- 2. IDE 설치하기 (IntelliJ)

- 3. IntelliJ에서 Hello World 출력하기!

- 4. 블로그 개설하기

- 5. 블로그 글 작성하기

---

## 1. Java 설치하기

마침 자바 17버전도 깔아야했는데 미션으로 자바 설치하기가 있어서 겸사겸사 설치했다.

자바를 설치하는 과정은 매우매우매우 쉽다. 아래 사진을 따라하기만 하면 설치 끝이다!!

![img.png](https://github.com/K-Diger/K-Diger.github.io/blob/main/images/jscode/week1/img.png?raw=true)

![img_1.png](https://github.com/K-Diger/K-Diger.github.io/blob/main/images/jscode/week1/img_1.png?raw=true)

![img_2.png](https://github.com/K-Diger/K-Diger.github.io/blob/main/images/jscode/week1/img_2.png?raw=true)

![img_3.png](https://github.com/K-Diger/K-Diger.github.io/blob/main/images/jscode/week1/img_3.png?raw=true)

## 2. IDE 설치하기 (IntelliJ)

![img_4.png](https://github.com/K-Diger/K-Diger.github.io/blob/main/images/jscode/week1/img_4.png?raw=true)

IDE는 위 URL을 참고하여 보이는 페이지에서 다운받을 수 있다. 학교를 다니는 중이라면 학생 계정으로 Ultimate 버전을 쓸 수 있다!

## 3. IntelliJ에서 Hello World 출력하기!

참을성이 없어서 프로젝트 JDK 셋팅을 로드해오는걸 못기다리고 작업한 탓인지..

코드를 실행했을 때 JDK를 못찾는 오류가 계속 반복했다.

해결에 도움을 구하려고 팀원들에게 화면 공유를 키고 천천히 프로젝트 생성부터 다시 진행하니 귀신같이 해결됐다..

![img_5.png](https://github.com/K-Diger/K-Diger.github.io/blob/main/images/jscode/week1/img_5.png?raw=true)

## 4. 블로그 개설하기

블로그는 이전에 개설해놓은 GitHub 블로그를 사용했다. 다른 블로그와 다르게 직접 테마를 커스텀할 수 있고

무엇보다 점유율이 낮은 방식이라 좀 **힙한 느낌**이 들어서 계속 쓰게 되더라!

## 5. 블로그 글 작성하기

작성 중 !

```java
public class Markdown {
    public static void main(String[] args) {
        System.out.println("이렇게 코드 블럭도 지원하니 코드를 붙일 때 참 편한 것 같다!");
    }
}
```

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

그렇다 보니 평균을 출력할 때 소수점이 부정확하게 나오게되었는데 왜 그런지 파악하지 못했었다.

당연하게도 **실수형을 나눗셈 할 땐 실수**로 나눠야지 보다 더 정확한 소수점 자릿수가 나오게 된다!

실수를 표현하는 float과 double의 차이점에서도, 각 32bit, 64bit를 가진 타입으로 그 정확도도 double이 더 높다고 한다!

## 아쉬운 점

입력 부분을 다듬지 못했던 것 같다.

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

### Class Loader 1. Loading

.class 파일을 읽고 바이너리 데이터를 생성하여 **메서드 영역**에 저장한다.

이 때 로드된 클래스와 그 모든 연관 부모 클래스의 .class 파일이 Class 혹은 Interface, 혹은 Enum과 관련있는 것인지 체크하고 조건에 맞는다면 이들 또한 메서드 영역에 저장한다.

.class 로딩을 마친 후에 JVM은 **Heap 영역**에 이 파일들을 나타내기위해 Class타입의 객체를 생성한다. (getClass()메서드를 사용할 수 있는 이유!)

### Class Loader 2. Linking

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

> [!] 알아봐야 할 내용 [!]
>
> 간접 참조와 직접 참조의 차이점은 무엇인가요?!...

### Class Loader 3. Initialization

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

### Execution Engine - Garbage Collector

참조되지 않는 객체를 정리해준다.

---

## GC에 대해 설명하기

GC는 메모리(Heap영역)상의 상주하는 사용하지 않는 객체를 수집하는 JVM 실행 엔진의 일부이다.

![image](https://www.programmersought.com/images/220/8462f4bfba5f173585f6f8cd2dce12ac.JPEG)

JVM의 메모리 구조의 일부는 위와 같이 생겼다.

### Young Generation (Eden + Survivor)

새로 생성된 객체는 Young Generation 에서 시작한다. 모든 객체가 시작되는 1개의 Eden 공간과 Garbage Collection Cycle 에서 살아남은 후

Eden 공간에서 살아남은 객체들은 Survivor 영역으로 이동한다.

### Old Generation

Young Generation의 Survivor에서도 수집되지 않은 객체들은 Young Generation 에서 Old Generation 으로 이동된다.

### Permanent Generation (Metaspace)

클래스 및 메서드와 같은 메타데이터가 저장되는 장소이다.

추가적으로, Java 8 에 들어오면서, Permanent Generation 이 사라지고 Metaspace 영역이 생겼다.

Permanent Generation 은 JVM 에 의해서 Heap 영역의 메모리 크기가 강제되던 영역이였다.

하지만 Metaspace 가 Native 영역에 배치되면서, 운영체제가 자동으로 그 크기를 조절할 수 있게 되고, Heap 에서 사용할 수 있는 메모리의 크기가 늘어나게 됐다.

## GC 상태

### Minor/Incremental Garbage Collection

Young Generation 에서 객체가 제거 되었다는 것을 말하는 상태이다.

### Major/Full Garbage Collection

Minor Garbage Collection 에서 살아남은 객체를 Old 로 옮긴다.

Old 에서는 GC 대상이 덜 자주 발생하게 된다.

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

---

# 4회차 미션 1. 학생들의 이름을 가나다 순으로 출력하기

## 요구사항

대여할 책의 번호를 입력하세요.
1. 클린코드(Clean Code) - 대여 가능
2. 객체지향의 사실과 오해 - 대여 가능
3. 테스트 주도 개발(TDD) - 대여 가능

1

정상적으로 대여가 완료되었습니다.

대여할 책의 번호를 입력하세요.

1. 클린코드(Clean Code) - 대여 중
2. 객체지향의 사실과 오해 - 대여 가능
3. 테스트 주도 개발(TDD) - 대여 가능

1

대여 중인 책은 대여할 수 없습니다.

대여할 책의 번호를 입력하세요.

1. 클린코드(Clean Code) - 대여 중
2. 객체지향의 사실과 오해 - 대여 가능
3. 테스트 주도 개발(TDD) - 대여 가능

2

정상적으로 대여가 완료되었습니다.

대여할 책의 번호를 입력하세요.

1. 클린코드(Clean Code) - 대여 중
2. 객체지향의 사실과 오해 - 대여 중
3. 테스트 주도 개발(TDD) - 대여 가능

## 주의사항

- 클래스 내에서 static 메서드는 사용하지마라. (public static void main(String[] args)는 제외)

- Main(프로그램을 실행하는 코드가 존재하는 클래스), Book(책), Library(도서관)의 클래스를 만들어서 활용해라.

- 한 파일에 모든 코드를 작성하지 말고, 1개의 클래스마다 클래스 파일을 별도로 생성해서 사용해라.

---

## 깃헙 레포지토리

[4회차 깃헙 레포지토리](https://github.com/K-Diger/JSCODE-Java-Study/tree/main/src/main/java/src/class4)

---

## 구현 - Main.class

```java
package src.class4;

import java.util.Scanner;

public class Main {

    public static void main(String[] args) {
        Main main = new Main();
        main.run(main.initLibrary());
    }

    public void run(Library library) {
        Scanner scanner = new Scanner(System.in);
        int booksCount = library.borrow(Integer.parseInt(scanner.nextLine()));
        while (booksCount != 0) {
            booksCount = library.borrow(Integer.parseInt(scanner.nextLine()));
        }
        System.out.println("모든 책이 대여되었습니다. 도서관 영업을 마칩니다 !");
    }

    public Library initLibrary() {
        Library library = new Library();
        library.printBorrowGuide(library.makeExampleBooks());
        return library;
    }
}


```

## 구현 - Book.class

```java
package src.class4;

public class Book {

    private Integer number;
    private String name;
    private Boolean status;

    public Book(Integer number, String bookName, Boolean status) {
        this.number = number;
        this.name = bookName;
        this.status = status;
    }

    public Integer getNumber() {
        return this.number;
    }

    public void setNumber(Integer number) {
        this.number = number;
    }

    public String getName() {
        return this.name;
    }

    public Boolean getStatus() {
        return this.status;
    }

    public void setName(String name) {
        this.name = name;
    }

    public void setStatus(Boolean status) {
        this.status = status;
    }
}

```

## 구현 - Library.class

```java
package src.class4;

import static java.lang.Boolean.FALSE;

import java.util.ArrayList;
import java.util.List;

public class Library {

    private final List<Book> books = new ArrayList<>();

    private int booksCount = 0;

    public List<Book> makeExampleBooks() {
        Book book1 = new Book(1, "클린코드(Clean Code)", true);
        Book book2 = new Book(2, "객체지향의 사실과 오해", true);
        Book book3 = new Book(3, "테스트 주도 개발(TDD)", true);
        books.add(book1);
        books.add(book2);
        books.add(book3);

        booksCount = books.size();
        return books;
    }

    public void printBorrowGuide(List<Book> books) {
        System.out.println("대여할 책의 번호를 입력하세요.");
        int i = 1;
        for (Book book : books) {
            if (isAvailableBorrow(book)) {
                System.out.println(i + ". " + book.getName() + " - " + "대여 가능");
            } else {
                System.out.println(i + ". " + book.getName() + " - " + "대여 중");
            }
            i++;
        }
    }

    public boolean isAvailableBorrow(Book book) {
        return book.getStatus();
    }

    public int borrow(Integer number) {
        if (books.get(number - 1).getStatus()) {
            System.out.println("정상적으로 대여가 완료되었습니다.");
            books.get(number - 1).setStatus(FALSE);
            booksCount--;
        } else {
            System.out.println("대여 중인 책은 대여할 수 없습니다.");
        }
        return booksCount;
    }

    public int getBooksCount() {
        return booksCount;
    }
}


```

## 아쉬운 점

- Library 클래스에서, Book을 등록할 때 사용자가 책 이름만 입력하더라도, 책의 번호, 책의 상태까지 자동으로 등록될 수 있는 로직을 만들어 보고 싶었다.

- 이번 학습 주제에 "오버로딩"이 있었는데, 내가 작성한 로직에 어떻게 적용할 수 있었을까?

- 과연 클래스를 클래스답게 사용했는가? 멘토님께서 추천해주신 "객체지향의 사실과 오해" 라는 책의 일부를 적용하면 어떻게 더 깔끔하게 바뀔 수 있을 지는 책을 더 읽어봐야 알 것 같다!

- 과연 Library가 수행하고 있는 기능 중 일부라도 Book 이라는 객체에서 해결할 수는 없었을까??

---

## 그래도 뿌듯했던 것!

- 현재 도서관에 대출 가능한 책의 숫자를 계산하여, 모든 책이 빌려질 때 까지 사용자에게 입력을 받는 요구사항을 추가해봤다.

- 조금 더 개선해서 매 입력을 받을 때마다 Guide를 출력할 수 있도록 해봐야겠다!

---

## 심화미션 - static을 지양하는 이유

Static은 사용하기 편하다. 언제 어디서든 호출이 가능하기 때문이다!

그렇지만 도대체 왜 이 편리함을 제공하는 static을 남발하면 안되는 것일까?

- 장점이자 단점으로 다가오는 점이 있다. 어느 곳에서든지 사용할 수 있다는 점이, 어디서 어떻게 사용하는 것으로 오류가 발생했는 지를 추론하기 힘들어진다.

- 객체지향의 가장 기본적인 원칙을 무너뜨린다. 객체지향은 캡슐화를 통해 외부에서 함부로 값을 수정하지 못하도록 하는 것을 추구하지만, static은 어디서든 값을 접근하여 수정할 수 있기 때문에 객체지향과 거리감이 있는 기능이다.

- GC 대상에 포함되지 않는다. GC는 Heap 영역을 대상으로 메모리 정리를 해준다. static 변수는 method area에 위치하게 되는데, 사용하지 않는 시점이 오더라도 그 메모리는 정리가 되지 않기 때문에 오히려 메모리를 낭비 시키는 기술이 될 수 있다.



## 심화미션 - Call By Reference, Call By Value

### Call By Reference란

특정 메서드를 실행할 때 파라미터에 특정 주소를 직접 전달하며 Pass By Reference 라고도 불린다.

참조 값을 직접 넘기기 때문에 파라미터로 넘겨받은 수신 변수와, 파라미터로 넘기는 송신 변수가 동일한 변수로써 동작한다.

이 방식은 주로 C++ 에서 사용되는데 Java는 Call By Value 방식을 채택하고있다.

### Call By Value란

특정 메서드를 실행할 때 파라미터에 특정 값을 직접 전달하며 Pass By Value 라고도 불린다.

Java 에는 Primitive Type, ReferenceType (Wrapper Class) 두가지가 존재하는데 각 타입에 따라서 동작되는 방식도 살짝 다르다.

우선 Primitive Type은 JVM의 Stack 영역에서 생성되는데 그 특징을 살펴보면 아래 코드와 같다.

```java
public class Main {

    void test() {
        int a = 1;
        int b = 2;

        modify(a, b);
    }

    private void modify(int a, int b) {
        // 여기 있는 파라미터 a, b 는 이름만 같을 뿐 test() 에 있는 a, b 와 다른 변수
        a = 5;
        b = 10;
    }
}
```

modify(a, b) 를 호출하는 순간 Stack 영역에 새로운 변수 a, b 가 새로 생성되어 총 4 개의 변수가 생기게 된다.

이렇게 변수명이 같던 이러한 상관없이 Primitive Type은 그 변수가 가진 값 만을 전달하는 방식으로 동작한다.

ReferenceType 에서는 조금 동작이 다르게 되는데 코드로 살펴보면 다음과 같다.

```java
class Member {
    public String name;

    public User(String name) {
        this.name = name;
    }
}

public class Main {

    void test() {
        Member memberA = new Member("Diger");
        Member memberB = new User("John");

        modify(memberA, memberB);
    }

    private void modify(Member memberA, Member memberB) {
        // modify 의 memberA 와 test 의 memberA 는 같은 객체를 바라보고 있다.
        memberA.name = "DDiger";

        // b 에 새로운 객체를 할당하면 가리키는 객체가 달라지고 원본에는 영향이 없다.
        memberB = new Member("JJohn");
        memberB.name = "JJJohn";
    }
}
```

위 코드와 주석이 조금 복잡하게 느껴질 수 있지만 쉽게 요약하자면 다음과 같다.

Wrapper Class는 기본적으로 "객체"이다. Integer든 String 이든, Member든 해당 타입을 사용한다는 것은

해당 타입을 가진 인스턴스를 만든다는 것이고, 그렇기 때문에 Heap 영역에 해당 타입을 참조하고 있는 것이 되는 것이다.

```java
private void modify(Member memberA, Member memberB) 라는 구문에서도
```

결국에는 Java 는 memberA가 가진 **"값"**, memberB가 가진 **"값"** 만을 사용하는 것이기 때문에

test() 메서드 내에 있는 memberA, memberB와는 다른 것이다.

매개변수로 test()의 memberA, memberB의 값을 받았고 그를 바탕으로 다른 인스턴스를 만들어 버린 것이다!!


[참고한 블로그 - 정리가 매우 잘 되어있다.](https://bcp0109.tistory.com/360)

이를 확실하게 이해하기 위해서는 반드시 JVM의 메모리 구조를 알아야 하고, 왜 Java가 Call By Reference가 아닌지는 코드를 따라 쳐보면 이해가 좀 더 잘갔다.

---

# 공부해 볼 내용!

> 디미터 법칙

- Git Flow

- GitHub Flow

- GitLab-Flow

- Trunk-Based

- Rebase

각 전략마다 특징이 뭔지 파악하자. (23/02/16)

# Git Flow

## Git flow 란?

Git 을 사용하면서 얻고자 하는 이점은 "분산 버전 관리"일 것이다.

Git을 활용한다면 한 프로젝트 레포지토리에 대해서 브랜치를 나누고, 각 브랜치 별로 별도의 작업을 수행하여 독립적으로 버전을 관리하는 것이 가능해진다.

이때, "브랜치를 나누고" 라는 문장에서 브랜치를 나누는 것에 대한 전략이 있다. 그리고 이것을 Git Flow 라고 한다.

## Git flow 의 종류?

- feature

기능의 구현을 담당, feature/구현 중인 기능 와 같이 네이밍 하는게 일반적이다.

feature 는 develop 브랜치의 하위 브랜치이며, 기능 제작이 완료되면 feature 브랜치는 삭제된다.

- develop

feature에서 기능이 개발될 때마다 합쳐지는 브랜치이다.

어느정도 완성된 기능이 많아진다면, 배포를 위한 브랜치인 release로 병합된다.

- release

release 브랜치는 develop 브랜치에서 생성되고, 버그 수정 내용을 develop 브랜치에도 반영한다.

어느정도 완성된 기능들이 모인 브랜치이다.

실 배포를 위한 브랜치라기 보단 배포 시 테스트를 위한 브랜치이다. 완전하게 준비되었다면 master(main) 브랜치로 병합한다.

- hotfix

master(main) 브랜치에서 생성되고 develop 브랜치, release 브랜치와 master(main) 브랜치에 수행한 내용을 반영한다.

배포된 소스에서 버그가 발생되면 이를 처리하기 위한 브랜치이다.

hotfix-N 으로 네이밍을 하는 것이 일반적이다.

- master(main)

최종 배포되는 브랜치이다.

## 각 Branch의 책임 요약

| 브랜치          | 역할 |
|--------------|-------------------|
| master(main) | 제품으로 출시될 수 있는 브랜치 |
| develop      | 다음 출시 버전을 개발하는 브랜치 |

> 알쓸신잡 : master는 Black lives matter 운동으로 인해 main으로 변경되었다.

|보조브랜치| 역할                       |
|---|--------------------------|
| feature | 기능을 개발하는 브랜치             |
| release | 이번 출시 버전을 준비하는 브랜치(QA)   |
| hotfix | 출시 버전에서 발생한 버그를 수정하는 브랜치 |

![Git-Flow](https://user-images.githubusercontent.com/60564431/179346591-d0edee5e-1bff-4600-aee0-330590bdffde.jpg)

위 그림은 브랜치 전략을 그려본 내용이다.

보조 브랜치는, 병합 후 삭제되어야 한다.

![Git-Flow](https://user-images.githubusercontent.com/43775108/125800526-2ea36d8e-6262-4ba5-9ef0-af7845131d85.png)

위 그림은 가장 이상적인 Git Flow 흐름을 나타내는 그림이다.

다음과 같은 브랜치 전략을 수행하기 위해서, 브랜치를 다루는 명령어를 알아보자.

---

## Branch Create

### 1. 로컬 저장소 브랜치 생성하기 (Local Folder)

```shell
git branch 브랜치이름
```

### 2. 원격 저장소 브랜치 생성하기 (GitHub)

```shell
git push origin ##1.에서생성한 브랜치이름
```

## Branch Read

### 1. 브랜치 확인하기

```shell
git branch
```

### 2. 브랜치 변경하기 (해당 브랜치이름으로 브랜치 변경)

```shell
git checkout 브랜치이름
```

---

## Branch Delete

### 1. 로컬 저장소 브랜치 제거하기 (Local Folder)

```shell
git branch -d 브랜치이름
```

### 2. 원격 저장소 브랜치 제거하기 (GitHub)
```shell
git push origin :브랜치이름
```

---

# Branch Update

> 브랜치 이름 변경 방법은, 변경 하고싶은 이름의 브랜치를 생성한 후, 기존 브랜치를 제거하는 것이다.


## 1. 로컬 저장소 브랜치 이름 변경 (Local Folder)

```shell
git branch -m 기존브랜치이름 변경브랜치이름
```

## 2. 원격 저장소 브랜치 생성 (GitHub)

```shell
git push origin 변경브랜치이름
```

## 3. 원격 저장소 기존 브랜치 제거 (GitHub)

```shell
git push origin :기존브랜치이름 변경브랜치이름
```

---

# GitHub Flow

[참고하기 좋은 블로그](https://sihyung92.oopy.io/architecture/gitflow-vs-githubflow#c609762c-92c6-4e20-a997-d839056941be)

아마 우리가 보통 Git을 처음 쓰고 프로젝트를 진행하면서 가장 많이 사용해왔던 패턴이 아닐까 하는 초간단 전략이다.

부연 설명할 것이 없이 아래 그림으로 모든 설명이 끝난다...

![GitHub-Flow](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FcSQzqk%2FbtqUTnAMFqj%2FdliROOijRpnhkVncaTEloK%2Fimg.png)

위 그림처럼 main브랜치는 항상 배포가 가능한 상태에 있으며 모든 브랜치는 main에서 시작한다는 특징과

이렇게 뻗어나간 브랜치들은 **반드시** PR후에 Merge가 되는 방식을 제안하는것이 GitHub Flow이다.

정말 단순하고 쉽게 사용할 수 있는 점이 큰 강점으로 생각되지만 규모가 큰 프로젝트에서는 과연 살아남을 수 있는 전략인지 궁금하긴하다!

---

# GitLab Flow

[참고하기 좋은 블로그](https://sihyung92.oopy.io/architecture/gitflow-vs-githubflow#c609762c-92c6-4e20-a997-d839056941be)

GitHub Flow는 너무 간단해서 배포, 환경 구성, 릴리즈, 통합 에 대한 이슈가 많다고 한다.

그래서 이를 보완하고자 GitLab에서 제안한 전략이다.

![GitLab-Flow](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FcKg4cN%2FbtrsC9Iih0f%2FXl7RoK2KfrzWVkWttjfKNk%2Fimg.png)

위 그림이 GitLab Flow를 잘 나타내는 그림이다.

master 에서 기능 개발을 수행하고 (필요 시 feature 브랜치도 만들어서 기능 개발을 한다.)

완성된 기능을 가진 내용을 pre-production 에 반영하고

그 이후에 production 에 반영하는 형태이다.

즉, 앞서 살펴본 GitHub Flow에서 배포를 위한 브랜치를 따로 추가하는 것으로 생각하면 된다.

---

# Trunk Based Development

GitHub Flow와 비슷하게 보이는 방식이다.

trunck 혹은 master(main) 브랜치에서 모든 작업을 수행하는 방식인데 여기서 반드시 지켜야하는 규악이 있다는 점이 가장 큰 차이점이다.

## Trunk Based Development Rule 1. 페어 프로그래밍

페어 프로그래밍은 한 명은 개발, 한 명은 코드 리뷰를 실시간으로 수행하는 방식인데

PR 요청이 추가로 필요하지 않고 즉각적인 피드백을 적용할 수 있다는 장점으로

Trunk Based Development의 꼭 지켜야하는 규악 중 한 가지로 채택되었다.

## Trunk Based Development Rule 2. 빌드의 신뢰성

현재 반영된 코드가 항상 빌드가 가능한가? 를 고려해야한다.

이를 위해선 자동테스트와 효과적인 테스트 전략이 요구된다.

## Trunk Based Development Rule 3. 작업이 완료되지 않은 내용이 있는가?

main 브랜치에는 미처 작업이 완료되지 않은 내용이 있을 수 있다.

그렇기 때문에 feature 브랜치를 사용하여 작업이 완료되지 않은 부분은 숨겨야한다.

이 때 완료되지 않은 내용이 단순한 작업이라면 branch by abstraction

다소 복잡한 내용이라면 feature 브랜치를 사용하면 된다.

## Trunk Based Development Rule 4. 소규모 개발

릴리즈를 더 자주 할 수 있도록 소규모 단위로 쪼개서 개발한다.

소규모 단위의 적절한 양은 며칠이 아닌, 몇 시간안에 완료될 수 있는 양을 말한다.

## Trunk Based Development Rule 5. 빠른 빌드

빌드/테스트 작업은 수 분 이내로 완료되어야한다. 이 때 작업시간이 길어진다면 아키텍처적인 개선을 고려해봐야한다.


## Trunk Based Development 의 장점

- 빠른 피드백

- 코드 컨벤션

- CI

- 병합 충돌 방지 -> 대규모 리팩터링 가능

---

# Rebase

rebase는 같은 뿌리를 가진 2개 이상의 Branch에서

Branch A의 Base를 Branch B의 최신 커밋으로 base를 옮기는 것이다.

이렇게 되면, Branch B의 그동안의 커밋 내용을 모두 깔끔하게 가져가 더 보기좋은 커밋 히스토리를 만들 수 있지만

한 커밋마다 매번 충돌을 해결해줘야하는 상황도 발생하긴한다.

rebase를 사용하는 명령어와 어떨 때 쓰이면 좋을지는 조금 더 알아본 후 작성해보자 (2023/02/18)

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
        Starter starter = new Starter(new PhoneNumberValidator(), new InputAgent(), new PrintAgent());
        starter.execute();
    }
}
```

## Starter.java

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

    public void execute() {
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

---

# 요구 사항

```text
원하시는 번호를 입력해주세요.
1. 회원 등록
2. 회원 조회
1

원하시는 번호를 입력해주세요.
1. 일반 회원
2. VIP 회원
1

이메일을 입력해주세요.
abcd@naver.com

이름을 입력해주세요.
박재성

나이를 입력해주세요.
20

회원 등록이 성공적으로 완료되었습니다.
원하시는 번호를 입력해주세요.
1. 회원 등록
2. 특정 회원 조회
1

원하시는 번호를 입력해주세요.
1. 일반 회원
2. VIP 회원
2

이메일을 입력해주세요.
iu@naver.com

이름을 입력해주세요.
아이유

나이를 입력해주세요.
20

신청한 PT 횟수를 입력해주세요.
5

이미 등록된 이메일이어서 회원 등록에 실패했습니다.
원하시는 번호를 입력해주세요.
1. 회원 등록
2. 회원 조회
2

조회하려는 회원의 이름을 입력해주세요.
박재성

박재성님은 일반 회원이고, 이메일은 abcd@naver.com이고, 나이는 20살입니다.
원하시는 번호를 입력해주세요.
1. 회원 등록
2. 회원 조회
2

조회하려는 회원의 이름을 입력해주세요.
아이유

아이유님은 VIP 회원이고, 이메일은 iu@naver.com이고, 나이는 20살입니다.
원하시는 번호를 입력해주세요.
1. 회원 등록
2. 회원 조회
```

위와 같이 동작하도록 만들어보자.

# 주의 사항

- 회원 정보를 저장할 저장소(MemberRepository)라는 클래스를 만들어서 활용해라.
    - MemberRepository는 회원을 조회, 저장을 할 수 있어야 한다.

- 회원은 일반 회원과 VIP 회원으로 나뉜다.

- 회원은 일반 회원과 VIP 회원으로 나뉜다.

- 일반 회원을 등록할 때는, 이메일, 이름, 나이 정보를 받아야 한다.

- VIP 회원을 등록할 때는, 이메일, 이름, 나이, 신청 PT 횟수 정보를 받아야 한다.

## 힌트

### Main.java

```java
public class Main {

    public static void main(String[] args) {
        FitnessCenterMemberManagingProgram program = new FitnessCenterMemberManagingProgram();
        program.start();
    }
}
```

### FitnessCenterMemberManagingProgram.java

```java
public class FitnessCenterMemberManagingProgram {

    public void start() {
        Scanner scanner = new Scanner(System.in);
        MemberRepository memberRepository = new MemberRepository();
        while (true) {
            System.out.println("원하시는 번호를 입력해주세요.");
            System.out.println("1. 회원 등록");

            // 로직 추가 작성

        }
    }
}
```

### Member.java

```java
public class Member {

    // 로직 추가 작성
}
```

### MemberRepository.java

```java
public class MemberRepository {

    private List<Member> members = new ArrayList<>();

    public void add(Member member) {
        // 로직 추가 작성
    }
}
```

---

# 구현 전

## 패키지 구조

![img.png](https://github.com/K-Diger/K-Diger.github.io/blob/main/images/jscode/img.png?raw=true)

- DependencyFactory는 클래스 간 의존성 주입을 대신 수행해주는 역할이다.

- GymInputAgent는 이 프로그램에서 입력을 담당하는 역할이다.

- GymPrintAgent는 입력 안내문구 및 예외 문구 등 출력을 담당하는 역할이다.

- GymService는 요구사항에 맞는 로직을 수행하는 역할이다.

- GymServiceValidator는 GymService에서 로직을 처리하기 전 입력값이 문제 없는지 검증하는 역할이다.

- MemberRepository는 인메모리 저장소에서 회원을 등록하고 조회하는 역할을 수행한다.

- Member는 회원의 정보를 담고 있으며 스스로의 정보를 출력할 수 있게끔 할 수 있는 기능을 담고 있다.

- GymStarter는 위 모든 클래스를 종합하여 실행하는 역할을 한다.

# 구현

## DependencyFactory.java

```java
package src.class7;

public class DependencyFactory {

    private static final GymPrintAgent GYM_PRINT_AGENT = new GymPrintAgent();
    private static final GymInputAgent GYM_INPUT_AGENT = new GymInputAgent();
    private static final MemberRepository memberRepository = new MemberRepository();
    private static final GymServiceValidator gymServiceValidator = new GymServiceValidator();
    private static final GymService gymService =
        new GymService(
            GYM_PRINT_AGENT, GYM_INPUT_AGENT, memberRepository, gymServiceValidator
        );

    private static final GymStarter gymStarter =
        new GymStarter(gymService, GYM_PRINT_AGENT, GYM_INPUT_AGENT);

    public static GymStarter getGymStarter() {
        return gymStarter;
    }
}
```

## GymInputAgent.java

```java
package src.class7;

import java.util.Scanner;

public class GymInputAgent {

    private final Scanner scanner = new Scanner(System.in);

    public String input() {
        return scanner.nextLine();
    }
}
```

## GymPrintAgent.java

```java
package src.class7;

public class GymPrintAgent {

    public void basicStringPrint(String message) {
        System.out.println(message);
    }

    public void selectManualGuide() {
        System.out.println("원하시는 번호를 입력해주세요.");
        System.out.println("1. 회원 등록");
        System.out.println("2. 회원 조회");
    }

    public void inputEmailGuide() {
        System.out.println("이메일을 입력해 주세요.");
    }

    public void inputNameGuide() {
        System.out.println("이름을 입력해 주세요.");
    }

    public void inputAgeGuide() {
        System.out.println("나이를 입력해 주세요.");
    }

    public void inputPtTimeGuide() {
        System.out.println("신청한 PT 횟수를 입력해주세요.");
    }

    public void registerSuccessGuide() {
        System.out.println("회원 등록이 성공적으로 완료되었습니다.");
    }

    public void inputEndFlag() {
        System.out.println("계속하시려면 c, 종료하시려면 q 를 입력해주세요.");
    }

    public void numberFormatExceptionPrinter(NumberFormatException numberFormatException) {
        System.out.println(numberFormatException.getMessage());
    }

    public void illegalArgumentExceptionPrinter(IllegalArgumentException illegalArgumentException) {
        System.out.println(illegalArgumentException.getMessage());
    }

    public void inputSearchedTargetUserGuide() {
        System.out.println("조회하려는 회원의 이름을 입력해주세요.");
    }

    public void memberInformation(Member member) {
        System.out.println(member.customToString(member));
    }
}
```

## GymService.java

```java
package src.class7;

public class GymService {

    private final GymPrintAgent gymPrintAgent;
    private final GymInputAgent gymInputAgent;
    private final MemberRepository memberRepository;
    private final GymServiceValidator gymServiceValidator;

    public GymService(
        GymPrintAgent gymPrintAgent,
        GymInputAgent gymInputAgent,
        MemberRepository memberRepository,
        GymServiceValidator gymServiceValidator) {
        this.gymPrintAgent = gymPrintAgent;
        this.gymInputAgent = gymInputAgent;
        this.memberRepository = memberRepository;
        this.gymServiceValidator = gymServiceValidator;
    }

    // 이게 퍼사드가 맞나...?
    public void facade(String inputManual) {
        if (inputManual.equals("1")) {
            enroll();
        } else if (inputManual.equals("2")) {
            findMember();
        }
    }

    private void enroll() {
        String inputEmail = "";
        try {
            gymPrintAgent.inputEmailGuide();
            inputEmail = gymInputAgent.input();
            gymServiceValidator.isEmailEnable(inputEmail);
        } catch (IllegalArgumentException illegalArgumentException) {
            gymPrintAgent.illegalArgumentExceptionPrinter(illegalArgumentException);
            return;
        }

        String inputName = "";
        try {
            gymPrintAgent.inputNameGuide();
            inputName = gymInputAgent.input();
            gymServiceValidator.isNameEnable(inputName);
        } catch (IllegalArgumentException illegalArgumentException) {
            gymPrintAgent.illegalArgumentExceptionPrinter(illegalArgumentException);
            return;
        }

        int inputAge = 0;
        int ptTime = 0;
        try {
            gymPrintAgent.inputAgeGuide();
            inputAge = gymServiceValidator.isParameterInteger(gymInputAgent.input());

            gymPrintAgent.inputPtTimeGuide();
            ptTime = gymServiceValidator.isParameterInteger(gymInputAgent.input());

        } catch (NumberFormatException numberFormatException) {
            gymPrintAgent.numberFormatExceptionPrinter(numberFormatException);
            return;
        }

        try {
            memberRepository.save(new Member(inputEmail, inputName, inputAge, ptTime));
        } catch (IllegalArgumentException illegalArgumentException) {
            gymPrintAgent.illegalArgumentExceptionPrinter(illegalArgumentException);
            return;
        }
        gymPrintAgent.registerSuccessGuide();
    }

    private void findMember() {
        gymPrintAgent.inputSearchedTargetUserGuide();
        final Member member = memberRepository.findByName(gymInputAgent.input());
        if (member == null) {
            gymPrintAgent.illegalArgumentExceptionPrinter(
                new IllegalArgumentException("존재하지 않은 회원입니다.")
            );
        } else {
            gymPrintAgent.memberInformation(member);
        }
    }
}
```

## GymServiceValidator.java

```java
package src.class7;

public class GymServiceValidator {

    public void isEmailEnable(String inputValue) {
        if (!inputValue.contains("@")) {
            throw new IllegalArgumentException("이메일 형식이 올바르지 않습니다.");
        }
    }

    public void isNameEnable(String inputValue) {
        final String regex = "[0-9]+";
        if (inputValue.matches(regex)) {
            throw new IllegalArgumentException("이름 형식이 올바르지 않습니다.");
        }
    }

    public int isParameterInteger(String inputValue) {
        final int temp;
        try {
            temp = Integer.parseInt(inputValue);
        } catch (NumberFormatException numberFormatException) {
            throw new NumberFormatException("정수로만 입력 가능합니다.");
        }
        return temp;
    }
}
```

## MemberRepository.java

```java
package src.class7;

import java.util.ArrayList;
import java.util.List;

public class MemberRepository {

    private final List<Member> members = new ArrayList<>();

    // 디미터의 법칙을 잘 적용했는가?
    public Member findByEmail(String email) {
        for (Member member : members) {
            if (member.getEmail().equals(email)) {
                return member;
            }
        }
        return null;
    }

    public Member findByName(String name) {
        for (Member member : members) {
            if (member.getName().equals(name)) {
                return member;
            }
        }
        return null;
    }

    public void save(Member member) {
        if (findByEmail(member.getEmail()) == null) {
            this.members.add(member);
            return;
        }
        throw new IllegalArgumentException("이미 등록된 이메일이어서 회원 등록에 실패했습니다.");
    }
}
```

## Member.java

```java
package src.class7;

public class Member {

    private String email;
    private String name;
    private Integer age;
    private Integer ptTime;
    private String status;

    private Member(String email) {
    }

    private Member(String email, String name) {
    }

    private Member(String email, String name, Integer age) {
    }

    public Member(String email, String name, Integer age, Integer ptTime) {
        this.email = email;
        this.name = name;
        this.age = age;
        if (ptTime > 0) {
            this.status = "VIP";
        } else {
            this.status = "일반";
        }
        this.ptTime = ptTime;

    }

    public String customToString(Member member) {
        return
            member.getName() + "님은 "
                + member.getStatus()
                + "회원이고, 이메일은 "
                + member.getEmail()
                + "이고, 나이는 " +
                member.getAge() +
                "살입니다.";
    }

    public String getEmail() {
        return email;
    }

    public void setEmail(String email) {
        this.email = email;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public Integer getAge() {
        return age;
    }

    public void setAge(Integer age) {
        this.age = age;
    }

    public String getStatus() {
        return status;
    }

    public void setStatus(String status) {
        this.status = status;
    }
}
```

## GymStarter.java

```java
package src.class7;

public class GymStarter {

    private final GymService gymService;
    private final GymPrintAgent gymPrintAgent;
    private final GymInputAgent gymInputAgent;

    public GymStarter(
        GymService gymService,
        GymPrintAgent gymPrintAgent,
        GymInputAgent gymInputAgent) {

        this.gymService = gymService;
        this.gymPrintAgent = gymPrintAgent;
        this.gymInputAgent = gymInputAgent;
    }

    public void execute() {
        while (true) {
            gymPrintAgent.selectManualGuide();
            String inputManual = gymInputAgent.input();
            gymService.facade(inputManual);

            // 이 부분부터
            gymPrintAgent.inputEndFlag();
            String endFlag = gymInputAgent.input();
            if (endFlag.equals("c")) {
                continue;
            } else if (endFlag.equals("q")) {
                break;
            } else {
                gymPrintAgent.basicStringPrint("잘 못 입력하셨습니다.");
                gymPrintAgent.inputEndFlag();
                endFlag = gymInputAgent.input();
            }
            // 여기까지 코드가 썩 맘에 들지 않는다. 더 좋은 방법은 뭐가 있었을까..
            // 지금 이 부분이 Spring 의 Controller 계층이라고 생각하면
            // 비즈니스 로직이 들어간 거 같아서 좀 찝찝하다.
        }
    }
}
```

## Main.java

```java
package src.class7;

public class Main {

    public static void main(String[] args) {
        GymStarter gymStarter = DependencyFactory.getGymStarter();
        gymStarter.execute();
    }
}
```

---

# 배운 내용과 앞으로 학습해가야할 내용

## 객체지향을 알아가려면

우선은 아래 자료를 순서대로 한 번씩 보고 스스로 습득하는 것이 좋은 것이 되겠다.

- 응집도 : 내부 요소들이 연관돼 있는 정도. 모듈 내 요소들이 하나의 목적을 위해 긴밀하게 협력 - 높은 응집도

- 결합도 : 의존성의 정도. 다른 모듈에 대해 얼마나 많은 지식을 갖고있는지

> 오브젝트보다 객체지향의 사실과 오해 먼저 읽는걸 추천!!

1. 객체지향의 사실과 오해

2. 오브젝트

3. https://www.youtube.com/watch?v=dJ5C4qRqAgA

- 디미터 법칙 (중요!)

[이전에 작성한 코드](https://k-diger.github.io/posts/JSCODE-Java6/)

지금 작성한 코드는 객체지향을 온전히 적용하기 좋은 예제는 아니였다.

객체간의 메세지를 통해 협력 관계를 구축하는 것이 객체지향의 핵심이지만 요구사항이 객체지향을 원한 것은 아니였다.

그래도 Starter.java의 코드 중

29번째 라인인

```java
printAgent.executeInputPhoneNumber(phoneNumber.convertToPhoneNumber());
```

구문은 디미터의 법칙을 잘 적용시킨 사례이다.

디미터의 법칙을 적용하지 않는다면 위 코드는 다음처럼 바뀔 수 있게 된다.

```java
printAgent.executeInputPhoneNumber("010-" + phoneNumber.getNumber() + "입니다.");
```

또한 책임과 역할이라는 용어를 너무 자세히 구분하지 않아도 되긴하다.

한 객체에 너무 많은 책임 혹은 역할만 부여하지 않는다는 것이 중요하다.

또한 다른 객체간의 **의존성 결합을 최소화**하되 그 객체 스스로가 처리할 수 있는 로직을 최대한 끌어내어

**높은 응집도**를 가져가는 것이 좋은 객체지향 설계의 기반이 된다.

그리고 객체지향의 원칙 중 SOLID원칙이 있는데 그 중에서도 DIP 원칙은 결합도를 낮추는데 효과적인 원칙으로 여겨지고 있다.

---

# 활동 내용 회고

## 1회차 활동 - Java, IDE 설치하기 / 블로그 개설 및 작성

기존 Java/Spring을 조금 써봤지만 Java를 조금 더 디테일하게 만약 모르는 부분이 있다면 그 내용까지 확실하게 다시 챙겨가고 싶어서 이 스터디에 들어왔다.

첫 시작은 가볍게 스터디를 위한 프로그램 설치와 학습 내용을 정리하기 위한 블로그를 작성했다.

처음 배정된 팀원들은 Java를 사용해보지 않은 분들이었기 때문에

그나마 조금이라도 도움이 되고자 모르는 부분이나 잘 안되는 부분에 도움을 줄 수 있도록 하였다.

나름 소통을 잘 해보려고 했으나 온라인의 한계인지 그리 활발하게 되지는 않았다.

내가 조금 더 분위기를 활기차게 이끌어 갔다면 더 좋은 분위기가 형성되지 않았을까?! 라는 생각이 들었다.

## 2회차 활동 - 입출력, 변수, 연산자, 형변환, 조건문 / JVM + GC

실제 구현이 부여되기 시작한 회차였다. 이때부터 조금 더 좋은 코드를 작성하는 방법을 의식하면서 코드를 작성해갔지만

그 결과는 뭔가 만족스럽지 않았다.

자바 입문 스터디 였기 때문에 객체지향적으로 좋은 코드를 작성하는 것은 사실 고려하지 않아도 되는 것이었지만

개인적인 목표를 잡은 것이었기 때문에 조금씩 신경쓰이기 시작했다.

그리고 심화미션같은 경우는 나처럼 Java를 조금이라도 써본 사람을 대상으로 도전해보는 주제였고

마침 가물가물했던 개념들을 다시 한 번 보게되고 이전에 정리했던 자료를 다시 한 번 검토함으로써 이전보단 더 좋은 정리본이 생기게 되어 만족스러웠다.


## 3회차 활동 - 반복문, 배열, List, 제네릭 / JCF, Generic, String, StringBuilder, StringBuffer

이때부터였나? 소회의실에 들어가기전 전체 인원을 대상으로 멘토님께서 접근제어자도 고려해보라는 말씀을 해주셨던 것 같다.

사실 접근제어자를 조금 대수롭지 않게 여겼던 지금까지의 코드 작성 습관이 이때부터 의식하기 시작했던 것 같다.

그리고 특히 알고리즘을 풀때 사용했던 String, StringBuilder, StringBuffer 이 세가지 간의 차이점을 정리하는 심화미션이 있었는데

미처 생각하지않고 그저 ~~~해서 좋다. 라고만 해서 사용하던 내용들을 알게되었고 이런 경험이 쌓이면서 디테일을 가진 개발자가 되는 것이겠구나 싶은 생각을 하게 되었다.

## 4회차 활동 - 클래스, 다중생성자, 오버로딩 / static, Call By Reference, Call By Value

본격적으로 객체지향스러운 코드를 작성해보고 싶다는 내용을 적용해보기 시작한 것 같다.

"객체지향과 사실과 오해"라는 책을 그냥 버스에서 오며가며 훑어보기만 했던 것을 반성하고

제대로 각잡고 읽어봐야겠다는 생각을 하게 되었다.

그리고 Call By Reference, Call By Value를 학습하면서 분명 몇 년전 학교에서 배웠던 내용인데 지금 다시 정리해보려고 하니

그냥 모르는 내용이 되어버렸다. 다시 한 번 정리하면서 각 차이점을 학습했고 또 Java라는 언어의 특징과 두 방식의 연관성을 정리하게 되었다.

## 5회차 활동 - Git / Git Flow, Rebase

우선 버전관리 전략에도 여러가지 전략이 있다는 것을 알게된 것이 가장 뿌듯했던 회차였다.

Git Flow, GitHub Flow, GitLab-Flow, Trunk-Based 등

더 많은 전략들이 있을 순 있겠지만 우선 이 정도를 정리해보면서 각 전략의 특징을 비교해보는데

곧 시작하게될 프로젝트에서도 어떤 전략을 적용할 지 상상하면서 재미있게 학습했던 회차였다.

사실 이때부터 팀 구성원이 바뀌게 되어 이것저것 떠드느라 스터디 시간에는 많이 못했지만

며칠 정도는 계속 이 내용을 붙잡고 정리해보려고 했었다.

## 6회차 활동 - 예외처리 / Equals And HashCode

이 활동의 요구사항을 구현하면서 멘토님께 피드백 요청을 엄청나게 많이 올렸다.

사실 객체지향 스터디가 아니기 때문에 이런 피드백을 요구해도 되나? 싶었지만 일단... 그래도 당장 궁금한게 많았기 때문에

피드백을 받지 못하더라도 나중에 다시 알아보도록 하기 위해서 피드백 받고 싶은 내용들을 정리는 해뒀다.

그리고 심화미션을 정리하면서 Equals와 Hashcode에 관하여 정리하게 되었고 이 과정에서 **동일성**, 과 **동등성**의 개념을 조금 더 확실하게 정리해갈 수 있었다.

## 7회차 활동 - 종합(헬스장 회원 관리 프로그램)

멘토님 두 분이서 지난 회차에 피드백 요청한 것을 답변해주셨다. 좋은 객체지향이란 높은 응집도와 낮은 결합도를 새기고 가야하며

디미터의 법칙이라는 내용을 학습해보면 좋을 것이라고 하셨다.

또한 "객체지향의 사실과 오해" -> "오브젝트" 및 해당 서적들의 저자이신 "조영호"님의 세미나 또한 추천해주셨다.

객체지향 스터디가 아님에도 정성스럽게 피드백을 해주시려는 것과 잘 모르는 내용을 한 번 더 짚어주신 것에 감사드리면서 이번 회차를 시작했다.

가만 보면 멘토님이 작성한 예시 코드들이 내가 좋게 짜보자고 고민해서 작성한 결과보다 더 직관적이기도 해서 지금 단계에서 굳이 고민해야했을까?

싶은 생각도 들게 되었다. 그래도 이왕 하기로 한거 최대한 아는 개념들을 쏟아낸 회차였다.
