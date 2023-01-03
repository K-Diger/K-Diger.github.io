---

title: Item 1. 생성자 대신 정적 팩터리 메서드
author: 김도현
date: 2022-06-21
categories: [Effective-Java]
tags: [Constructor, FactoryMethod]
math: true
mermaid: true

---

## 보편적인 클래스의 인스턴스 획득 방법 -> public 키워드를 통한 생성자 호출

    public class Example1 {

        String userName;

        public Example(String userName){
            this.userName = userName;
        }
    }

    public static void main(String[] args) {
        Example1 ex1 = new Example1("diger-king");
    }

---

## 고려해볼만 한 인스턴스 획득 방법 -> 정적 팩터리 메서드

    public class Example1 {

        String userName;

        public static Example1 initUserName(String userName) {
            return new Example1(userName);
        }
    }

    public static void main(String[] args) {
        Example1 ex1 = Example1.initUserName("diger-king");
    }

---

## 생성자를 대체하여, 정적 팩터리 메서드를 사용했을 때 이점

<br>

### #1. 이름을 가질 수 있다.

생성자를 사용했을때 인스턴스를 생성하는 라인은
생성과 동시에 파라미터를 넘겨주는 라인이 어떤 의미인지 명확하게 파악하기 힘든 경우가 있다.

그래서, 생성자를 대체할 메서드에 메서드이름을 부여하고, 파라미터를 명시하는 것에서 가독성적인 이점을 얻을 수 있다.

    public static void main(String[] args) {
        Example1 ex1 = new Example1("diger-king");
    }


    public static void main(String[] args) {
        Example1 ex1 = Example1.initUserName("diger-king");
    }

위 코드가 그 예시 이다. 간단한 상황이기 때문에 와닿지 않을 수 있지만, 어떤 상황에서 정적 팩터리 메서드를 쓰면 좋은지는 알아두자.

---

### #1-2. 같은 시그니처, 같은 타입이지만 다른 인스턴스 변수를 받는 파라미터를 생성 할 수 있다.

동일한 생성자 시그니처(매개변수 타입 및 갯수 동일)의 생성자는 여러개 만들 수 없지만,

정적 팩터리 메서드를 활용하면 이를 해결할 수 있다.

코드로 보면 쉽게 이해가 갈 것이다.

<br>

아래 코드는, 유저의 이름과 반려묘의 이름이 각각 있는 클래스에서 인스턴스를 생성할 때의 상황을 예시로 든것이다.

상황 1)

동물병원에서 진료를 접수할 때는 고양이 이름만 있어도 가능하지만

동물용품샵에서 물품을 구매 할 때는 집사의 이름 정보가 필요하다.

<br>

#### 동일 시그니처를 통해 생성자를 표현한 코드 -> 컴파일 에러

    public class Example2 {

        String userName;
        String catName;

        public Example2(String userName, String catName) {
            this.userName = userName;
            this.catName = catName;
        }

        public Example2(String catName, String userName) {
            this.catName = catName;
            this.userName = userName;
        }
    }

<br>

#### 정적 팩터리 메서드로 표현한 코드

    public class Example2 {

        String userName;
        String catName;

        // 기본 생성자도 필요 없음. -> 자동으로 생성해 주기 때문

        public static Example2 priorityIsUserName(String userName) {
            Example2 ex2 = new Example2();
            ex2.userName = userName;

            return ex2;
        }

        public static Example2 priorityIsCatName(String catName) {
            Example ex2 = new Example2();
            ex2.catName = catName;

            return ex2
        }
    }

    public static void main(String[] args) {
        Example2 user = Example2.priorityIsUserName("diger-king);
        Example2 cat = Example2.priorityIsCatName("LuLu");
    }

---

### #2. 생성자를 사용하면 매번 신규 인스턴스가 생성되지만, 정적 팩터리 메서드는 그렇지 않다.

생성자를 통해서 인스턴스를 생성하면 그 시점에는 항상 새로운 객체를 만들게 되는 것이다.

하지만 여러 인스턴스를 만들면 안되는 상황이 올 수도 있다. 이 때 정적 팩터리 메서드를 거쳐 인스턴스를 만드는 방법으로

객체 생성에 관한 통제를 할 수 있게 된다.

<br>

아래 코드는 헌법을 예시로 든 것이다.

헌법은 각 국가마다 하나의 객체로 되어야한다. 헌법이 여러 개가 있으면 법치에 혼동이 오기 때문에 단 하나의 객체만 가져야하는 것이다.



    public class Constitution {

        private String article;

        private String section;

        private Constitution() {}

        private static final Constituion CONSTITUTION_INSTANCE = new Constitution();

        public static Constitution instacne() {
            return CONSTITUTION_INSTANCE;
        }
    }

    public static void main(String[] args) {

        Constitution constitution1 = Constitution.createInstance();
        Constitution constitution2 = Constitution.createInstance();

        System.out.println(constitution1);
        System.out.println(constitution2);

        // 위 두 객체의 값은 같게 출력된다.

    }

<br>

> private static final Constituion constitutionInstance = new Constitution();

여기서 짚고 넘어가야 할 점은 private static final 키워드로 불변하는 상수값을 셋팅한 후,

정적 팩터리 메서드에서 상수값을 리턴을 하게되는 것이다. 이렇게 하면 모든 인스턴스는 해당 상수값에 의한 인스턴스 하나로 고정된다.

---

### #3. 반환 타입의 하위 타입 객체를 반환할 수 있다.
### #4. 입력 매개변수에 따라 매번 다른 클래스의 객체를 반환할 수 있다.
### #5. 인터페이스에 정적 팩터리 메서드를 작성했으면 구현체가 없어도 된다.

Java 8 이후, 인터페이스에 정적 메서드를 선언할 수 있게 되며 추가된 사항들이다.

즉, 코드로 살펴보면

<br>

#### HelloService.java

    // #5. 인터페이스에 정적 팩터리 메서드를 작성했으면 구현체가 없어도 된다.
    public interface HelloService {
        String hello();

        static HelloService of(String lang) {

            // #4. 입력 매개변수에 따라 매번 다른 클래스의 객체를 반환할 수 있다.
            if (lang.equals("ko") {

                // #3. 반환 타입의 하위 타입 객체를 반환할 수 있다.
                return new KoreanHelloService();
            }
            else {
                return new EnglishHelloService();
            }
        }
    }

#### Main.java

    public static void main(String[] args) {

        // ServiceLoader -> Java 에서 제공하는 정적 팩터리 메서드
        // 모든 클래스 경로에서, HelloService 구현체를 전부 가져온다.
        // 반환형은 Iterable
        // 이렇게 하게 되면, 탐색한 클래스들에 대해서 의존성이 필요가 없기 때문에
        // 직접 Import 하여 사용하는 것보다 더 좋은 방법이 될 수도 있다.


        // #5. 인터페이스에 정적 팩터리 메서드를 작성했으면 구현체가 없어도 된다.
        ServiceLoader<HelloService> loader = ServiceLoader.load(HelloService.class);

        Optional<HelloService> helloServiceOptional = loader.findFirst();

        helloServiceOptional.ifPresent(e -> {
            System.out.println(e.hello());
        });
    }

---

## 정적 팩터리 메서드의 문제점

### #1. 상속이 불가능하다.

**정적 팩터리 메서드의 이점 #2.** 에서 사용한 예제로, 정적 팩터리만을 사용하도록 (인스턴스값 고정) 하는 클래스는 상속이 불가하다.

정적 팩터리만을 사용하게하려면, 기본 생성자를 private 로 만들어야한다.

즉, 상속을 허용하지 않게 된다.

    public class Constitution {

        private String article;

        private String section;

        // private 로 묶여있다.
        private Constitution() {}

        private static final Constituion CONSTITUTION_INSTANCE = new Constitution();

        public static Constitution instacne() {
            return CONSTITUTION_INSTANCE;
        }
    }

    public static void main(String[] args) {

        Constitution constitution1 = Constitution.createInstance();
        Constitution constitution2 = Constitution.createInstance();

        System.out.println(constitution1);
        System.out.println(constitution2);

        // 위 두 객체의 값은 같게 출력된다.

    }

<br>

위의 예시에서, 만약 헌법이 개정되어

개정되지않은 헌법 책, 개정된 헌법 책이 있다고하자

이미 개정되지 않은 헌법 책은 있지만, 개정 된 책은 기존 책을 상속을 받지 못하는데 어떻게 해야하나?

    pulibc class AdvancedConstitution {
        Constitution constitution;
    }

다음과 같이 꼭 상속을 사용하지 않더라도 사용은 할 수 있게 된다.

장점이 될 수도있고 아닐 수도 있다.

### #2. Java Doc 에서 생성자를 의미하는 메서드를 찾기가 어렵다.

생성자는 클래스의 이름과 같기 때문에 문서에서도, 쉽게 파라미터 등 활용 방법을 알아낼 수 있지만

정적 팩터리 메서드는 말 그대로 하나의 메서드 중 하나이기 때문에

한 클래스에서 메서드가 엄청나게 많은 경우를 생각하면, 어떤 메서드가 생성자 역할을 하는지 구분하기 어렵게 된다.

따라서 이를 조금이나마 해결하고자 네이밍 컨벤션이 존재한다.

> instance, getInstance : 매개변수로 명시한 인스턴스를 반환하지만, 같은 인스턴스임은 보장하지 않는다.
>
> StackWalker luke = StackWalker.getInstance(options)

> from : 매개변수를 하나만 받고, 해당 타입의 인스턴스를 반환하는 형변환 메서드
>
> Date d = Date.from(instant);

> of : 여러 매개변수를 받아 적합한 타입의 인스턴스를 반환하는 집계 메서드
>
> Set<Rank> faceCards = EnumSet.of(JACK, QUEEN, KING);

> valueOf : from과 of의 더 자세한 버전
>
> BigInteger prime = GitInteger.valueOf(Integer.Max_VALUE);
