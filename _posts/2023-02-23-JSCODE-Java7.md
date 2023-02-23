---

title: JSCODE - 자바 스터디 7회차 (헬스장 회원 관리 프로그램)
author: 김도현
date: 2023-02-23
categories: [Java]
tags: [Java]
math: true
mermaid: true

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
        Member member = memberRepository.findByName(gymInputAgent.input());
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

