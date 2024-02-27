---

title: Effective Java
date: 2023-01-09
categories: [Java, Effective-Java]
tags: [Java, Effective-Java]
layout: post
toc: true
math: true
mermaid: true

---

# Item 1. 생성자 대신 정적 팩터리 메서드를 고려하자

## 보편적인 클래스의 인스턴스 획득 방법 - 생성자 호출

    public class Example1 {

        String userName;

        public Example(String userName){
            this.userName = userName;
        }
    }

    public static void main(String[] args) {
        Example1 ex1 = new Example1("diger");
    }

---

## 고려해볼만 한 인스턴스 획득 방법 - 정적 팩터리 메서드

    public class Example1 {

        String userName;

        public static Example1 initUserName(String userName) {
            return new Example1(userName);
        }
    }

    public static void main(String[] args) {
        Example1 ex1 = Example1.initUserName("diger");
    }

---

## 생성자를 대체하는 정적 팩터리 메서드를 사용했을 때 이점

## 1. 이름을 가질 수 있다.

생성자를 사용했을때 인스턴스를 생성하는 라인은 생성과 동시에 파라미터를 넘겨주는 라인이 어떤 의미인지 명확하게 파악하기 힘든 경우가 있다.

그래서 생성자를 대체할 메서드에 **메서드 이름**을 부여하고, 파라미터를 명시하는 것에서 가독성적인 이점을 얻을 수 있다.

    public static void main(String[] args) {
        Example1 ex1 = new Example1("diger");
    }


    public static void main(String[] args) {
        Example1 ex1 = Example1.initUserName("diger");
    }

위 코드가 그 예시 이다. 간단한 상황이기 때문에 와닿지 않을 수 있지만, 어떤 상황에서 정적 팩터리 메서드를 쓰면 좋은지는 알아두자.

---

## 2. 같은 시그니처, 같은 타입이지만 다른 인스턴스 변수를 받는 파라미터를 생성 할 수 있다.

동일한 생성자 시그니처(매개변수 타입 및 갯수 동일)의 생성자는 여러개 만들 수 없지만, 정적 팩터리 메서드를 활용하면 이를 해결할 수 있다.

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
        Example2 user = Example2.priorityIsUserName("diger);
        Example2 cat = Example2.priorityIsCatName("LuLu");
    }

---

## 3. 생성자를 사용하면 매번 신규 인스턴스가 생성되지만, 정적 팩터리 메서드는 그렇지 않다.

생성자를 통해서 인스턴스를 생성하면 그 시점에는 항상 새로운 객체를 만들게 되는 것이다.

하지만 여러 인스턴스를 만들면 안되는 상황이 올 수도 있다. 이 때 정적 팩터리 메서드를 거쳐 인스턴스를 만드는 방법으로 객체 생성에 관한 통제를 할 수 있게 된다.

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

> private static final Constituion constitutionInstance = new Constitution();

여기서 짚고 넘어가야 할 점은 private static final 키워드로 불변하는 상수값을 셋팅한 후,

정적 팩터리 메서드에서 상수값을 리턴을 하게되는 것이다. 이렇게 하면 모든 인스턴스는 해당 상수값에 의한 인스턴스 하나로 고정된다.

### 4. 반환 타입의 하위 타입 객체를 반환할 수 있다.
### 5. 입력 매개변수에 따라 매번 다른 클래스의 객체를 반환할 수 있다.
### 6. 인터페이스에 정적 팩터리 메서드를 작성했으면 구현체가 없어도 된다.

Java 8 이후, 인터페이스에 정적 메서드를 선언할 수 있게 되며 추가된 사항들이다.

즉, 코드로 살펴보면

<br>

#### HelloService.java

    // 6. 인터페이스에 정적 팩터리 메서드를 작성했으면 구현체가 없어도 된다.
    public interface HelloService {
        String hello();

        static HelloService of(String lang) {

            // 5. 입력 매개변수에 따라 매번 다른 클래스의 객체를 반환할 수 있다.
            if (lang.equals("ko") {
                // 4. 반환 타입의 하위 타입 객체를 반환할 수 있다.
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

        // 6. 인터페이스에 정적 팩터리 메서드를 작성했으면 구현체가 없어도 된다.
        ServiceLoader<HelloService> loader = ServiceLoader.load(HelloService.class);

        Optional<HelloService> helloServiceOptional = loader.findFirst();

        helloServiceOptional.ifPresent(e -> {
            System.out.println(e.hello());
        });
    }

---

## 정적 팩터리 메서드의 문제점

### 1. 상속이 불가능하다.

**정적 팩터리 메서드의 이점 2.** 에서 사용한 예제로, 정적 팩터리만을 사용하도록 (인스턴스값 고정) 하는 클래스는 상속이 불가하다.

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

    pulibc class AdvancedConstitution {
        Constitution constitution;
    }

다음과 같이 꼭 상속을 사용하지 않더라도 사용은 할 수 있게 된다.

장점이 될 수도있고 아닐 수도 있다.

### 2. Java Doc 에서 생성자를 의미하는 메서드를 찾기가 어렵다.

생성자는 클래스의 이름과 같기 때문에 문서에서도, 쉽게 파라미터 등 활용 방법을 알아낼 수 있지만 정적 팩터리 메서드는 말 그대로 하나의 메서드 중 하나이기 때문에

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

---

# Item 2. 생성자에 매개변수가 많을때는 빌더를 고려하자

## 객체 생성의 3가지 방법

### 1. 점층적 생성자 패턴

```java
public class NutritionFacts {
    private final int servingSize;
    private final int servings;
    private final int calories;
    private final int fat;
    private final int sodium;
    private final int carbohydrate;

    public NutritionFacts(int servingSize, int servings) {
        this(servingSize, servings, 0);
    }

    public NutritionFacts(int servingSize, int servings, int calories) {
        this(servingSize, servings, calories, 0);
    }

    public NutritionFacts(int servingSize, int servings, int calories, int fat) {
        this(servingSize, servings, calories, fat, 0);
    }

    public NutritionFacts(int servingSize, int servings, int calories, int fat, int sodium) {
        this(servingSize, servings, calories, fat, sodium, 0);
    }

    public NutritionFacts(int servingSize, int servings, int calories, int fat, int sodium, int carbohydrate) {
        this(servingSize, servings, calories, fat, sodium, carbohydrate, 0);
    }
```

위 처럼 생성자의 매개변수를 늘려가는 방식이 점층적 생성자 패턴이다.

### 점층적 생성자 패턴의 장점

솔직하게 말해서 하나도 없는 것 같다. 이미 다른 패턴을 알고 있어서 그런걸까?

### 점층적 생성자 패턴의 단점

1. 매개변수가 많아지면 어떤 생성자를 사용해야하는지 클라이언트 입장에선 굉장히 모호해진다.
2. 매개변수가 많아지면 코드를 읽기가 어렵다.
3. 생성자 매개변수의 순서가 바뀌어도 컴파일 단계에서는 알아챌 수 없다. 런타임 때 알아채면 다행인 상황이 발생한다.

---

### 2. 자바빈즈 패턴 (Setter)

```java
public class NutritionFacts {
    private int servingSize;
    private int servings;
    private int calories;
    private int fat;
    private int sodium;
    private int carbohydrate;

    public NutritionFacts() {}

    public void setServingSize();
    public void setServings();
    public void setCalories();
    public void setFat();
    public void setSodium();
    public void setCarbohydrate();
}
```

### 자바빈즈 패턴의 장점

1. 읽기가 한결 쉬워진다.
2. 인스턴스 생성 자체는 매우 쉽다.

### 자바빈즈 패턴의 단점

1. 객체 하나 만드는데 메서드를 엄청나게 많이 호출해야한다.
2. 객체가 완전히 생성되기 전까지는 일관성이 무너진 상태이다.
    3. 객체를 생성하고, setServingSize() ... 등의 모든 Setter 메서드를 호출 하기 전과 후의 객체는 일관성이 맞지 않아 버리는 것.
4. 클래스를 불변으로 만들 수 없게 된다. --> 객체의 일관성 유지가 되지 않는다는 것이다.

---

### 3. 빌더 패턴

```java
public class NutritionFacts {
    private final int servingSize;
    private final int servings;
    private final int calories;
    private final int fat;
    private final int sodium;
    private final int carbohydrate;

    public static class Builder {
        private final int servingSize;
        private final int servings;

        private int calories = 0;
        private int fat = 0;
        private int sodium = 0;
        private int carbohydrate = 0;

        public Builder(int servingSize, int servings) {
            this.servingSize = servingSize;
            this.servings = servings;
        }

        public Builder calories(int val) {
            calories = val;
            return this;
        }

        public Builder fat(int val) {
            fat = val;
            return this;
        }

        public Builder sodium(int val) {
            sodium = val;
            return this;
        }

        public Builder carbohydrate(int val) {
            carbohydrate = val;
            return this;
        }

        public NutritionFacts build() {
            return new NutritionFacts(this);
        }
    }

    private NutritionFacts(Builder builder) {
        servingSize = builder.servingSize;
        servings = builder.servings;
        calories = builder.calories;
        fat = builder.fat;
        sodium = builder.sodium;
        carbohydrate = builder.carbohydrate;
    }
}
```

위와 같이 Builder 패턴을 설계하면, NutritionFacts 클래스는 불변이며 모든 매개변수의 기본 값들을 한곳에 셋팅할 수 있게된다.

Builder 패턴을 사용하는 클라이언트의 코드를 보면 다음과 같다.

```java
NutritionFacts cocaCola = new NutritionFacts.Builder(240, 8)
    .calories(100)
    .sodium(35)
    .carbohydrate(27)
    .build();
```

### 빌더 패턴의 장점

1. 위와 같이 메서드 체이닝으로 간편하게 많은 매개변수를 다룰 수 있게 되며 가독성이 향상한다.
2. 객체 생성과 동시에 필드를 할당할 수 있다.
3. 계층적으로 설계된 클래스에 결합하기 좋다.

### 빌더 패턴의 단점

1. 패턴 구현이 다른 방법에 비해 복잡하다.

---

## 결론

생성자의 매개변수가 많을 때는 빌더 패턴이 적합해보인다. 추후 확장 가능성을 생각했을 때도

빌더 패턴이 객체를 생성하는 위 3가지의 방법 중 가장 유연한 패턴이다.

하지만 다른 방법이 무조건적으로 나쁘다고는 장담할 수 있는 사람이 많이는 없을 것이다.

그렇기 때문에 객체를 생성하는 각 방법의 장단점을 알아두는 것이 좋다.

또한 Spring Boot 기반에서 객체 생성을 다룰 때에는 Lombok이라는 어노테이션 프로세서 라이브러리가 있으니 이를 활용하면 빌더패턴을 직접 구현하지 않아도

쉽게 사용할 수 있으니 참고하면 좋을 것 같다.

---

# Item 3. private 생성자 또는 enum 타입으로 싱글톤을 보장할 것

## 싱글톤이란?

애플리케이션 통 틀어서, 해당 클래스의 인스턴스가 하나만 사용되는 것이다.

---

## 싱글톤으로 만들기

### User.java

    public class User {
        public static final User instance = new User();

        // 리플렉션으로 인한 private 생성자 호출을 막기 위한 카운트
        static int count;

        private User() {
            // 생성자가 불리울때마다 카운트를 증가 후 예외처리
            count ++;
            if (count != 1) {
                throw new IllegalStateException("싱글톤이 아니게 됩니다.");
            }
        }
    }

### UserSingleTonTest.java

    public class UserSingleTonTest {

        public static void main(String[] args) {

            // 생성자가 private 하기 때문에 컴파일 에러가 발생
            User user1 = new User();

            // 해당 클래스의 인스턴스를 직접 사용하는 방법
            User user1 = User.instance;

            // 리플렉션 -> class, metohd, constructor 를 하나의 타입으로 사용하는 것
            Constructor<User> constructor = User.class.getConstructor();
            constructor.setAccessible(true);
            constructor.newInstance();

        }
    }

> 지금 내가 진행하고있는 프로젝트에서도 싱글톤이 보장이되고있나..?

---

## 싱글톤으로 만들기 - 정적 팩터리

### User2.java

    public class User2 {
        public static final User2 instance = new User2();

        // 생성자 private
        private User2() {}

        // 인스턴스 반환 정적 팩터리 메서드
        public static User2 getInstance() {
            return instance;
        }
    }

### User2SingleTonTest.java

    public class User2SingleTonTest {

        public static void main(String[] args) {

            // 정적 팩터리 메서드로, 인스턴스를 반환하는 클래스 메서드 사용
            User2 user2 = User2.getInstance();

        }
    }

> 장점 : API 변경 없이 (public static method의 변경 없이) 싱글턴 인스턴스를 사용 가능하다.
>
> --> 기존 User 클래스에서는, 싱글톤이 아니도록 사용하게 하려면, 클라이언트 코드를 사용해야했지만 이는 아니다.
>
> --> 쓰레드당 하나의 객체를 사용하도록 할때가 싱글톤이 아니도록 사용할 상황

---

## 직렬화가 뭐야??? 역직렬화는 또 뭐고

> 직렬화는 객체를 persistance 화 하는것,
>
> --> 객체를 전달할 때 최적화(객체를 옮길 수 있는 상태로)를 위해 사용한다. (직렬화)
>
> --> 객체를 전달받았을 때 최적화(객체를 사용할 수 있는 상태로)를 위해 사용한다. (역직렬화)

---

## 싱글톤으로 만들기 - Enum (글쎄...)

    public enum User3 {
        INSTANCE;

        public String getName() {
            return "user-name";
        }
    }

---

> Spring FrameWork 내부에 등록한 Bean을 사용하는 것은 기본적으로 싱글톤이다.
>
> @Autowired UserRepository
>
> vs
>
> @RequiredArgsConstructor private final UserRepository
>
>
> 필드주입이 위험한 이유?
---

## 현실적인 싱글톤 만들기

> 생성자 주입 방식으로 만들어버립시다

---

# Item 4. 인스턴스화를 막으려거든 private 생성자를 사용하라

## 정적 메서드와 정적 필드만을 담은 클래스를 사용하고 싶을 때

위의 유형이 객체지향적인 구성은 아니다. 하지만 분명히 쓰임새가 있기 마련이다.

그 예시로 Arrays 라이브러리를 살펴보면, Arrays.toString() 등 처럼 실제 Java 공식 라이브러리에서도 위와 같은 구성을 취하곤 한다.

그리고 Collections 처럼 특정 인터페이스를 구현하는 객체를 생성하는 정적 메서드 혹은 팩터리를 모아 놓을 수 도 있다.

final 클래스와 관련한 메서드들을 모아놓을 때도 사용한다. final 클래스를 상송하여 하위 클래스에 메서드에 넣는 건 불가능하기 때문이다.

## 정적 멤버만 담은 클래스는 유틸리티 클래스이다.

인스턴스화 하여 사용할 목적으로 설계된 것이 아니므로 인스턴스화를 막아야할 필요가 있다.

하지만 생성자를 따로 명시하지 않으면 매개변수가 없는 public 생성자가 자동으로 만들어져 이를 막아야지 목적에 맞게 사용할 수 있다.

## 추상 클래스로 만드는 것으로는 인스턴스화를 막을 수 없다.

추상클래스를 상속받는 하위클래스를 만들어 인스턴스화 하는 방법이 가능하기 때문에 인스턴스화를 방지할 수 없다.

또한 추상클래스를 만들어 놓으면, 사용자는 이를 상속해서 사용하라는 의미로 받아들일 수 있기 때문에 주의해야한다.

## 인스턴스화를 막는 방법은 private 생성자를 만드는 것

```java
public class UtilClass {

    private UtilClass() {
        throw new AssertionError();
    }
}

```
위와 같이 인스턴스화를 막기 위해서는 private 생성자를 꼭 명시해줘야함을 잊지 말자.

또한 클래스 자체에서 생성자를 호출하는 것을 막기 위해 예외를 던지는 것도 좋은 방법이다.

이렇게 생성자를 private으로 하면 상속을 불가능하게도 한다.

모든 생성자는 명시적/묵시적으로 상위 클래스의 생성자를 호출하는데 이를 private으로 했으니 하위 클래스가 상위 클래스의 생성자에 접근하지 못하기 때문이다.

---

# Item 5. 자원을 직접 명시하지 말고 의존 객체 주입을 사용하라

## 객체지향을 위해서, 각 객체는 객체 간 메세지를 통해 협력해야한다.

"맞춤법 검사기"라는 객체는 "사전"에 의존한다. 이런 클래스를 정적 유틸리티 클래스와 싱글턴으로 구현하면 다음과 같다.

```java
// 정적 유틸리티 클래스
public class SpellChecker {
    private static final Lexicon dictionary = ...;

    private SpellChecker() {}

    public static boolean isValid(String word) {}
    public static List<String> suggestions(String typo) {}
}
```


```java
// 싱글턴
public class SpellChecker {
    private static final Lexicon dictionary = ...;

    private SpellChecker() {}

    public static SpellChecker INSTANCE = new SpellChecker();

    public static boolean isValid(String word) {}
    public static List<String> suggestions(String typo) {}
}
```

위 두 방식은 모두 사전을 단 하나만 사용한다는 가정이 달려있는 코드이다. 실제로는 사전이 언어별로 따로 있기도 하며 다양하게 존재하기 때문에

유연하지 않은 설계이며, 테스트하기 어려운 점이 있다.

사용하는 자원에 따라 동작이 달라지는 클래스에는 정적 유틸리티 클래스 혹은 싱글턴은 적합하지 않다.

## 유연한 객체로 변환하기

### 방법 1.

dictionary 필드에서 final 한정자를 제거하고 다른 사전으로 교체하는 메서드를 추가한느 방법도 있다.

멀티스레드 환경에서는 쓸 수 없는 방법이다. (스레드 간 동기화 문제 발생!)

### 방법 2.

인스턴스를 생성할 때 생성자에 필요한 자원을 넘겨주는 방식이 있다.

즉, 어떤 SpellChecker를 생성하면서 필요한 Dictionary객체를 직접 주입해주는 방식인데, 코드로 표현하면 다음과 같다.

```java
public class SpellChecker {
    private final Lexicon dictionary;

    public SpellChecker(Lexicon dictionary) {
        this.dictionary = Objects.requirNonNull(dictionary);
    }

    public boolean isValid(String word) {}
    public List<String> suggestions(String typo) {}
}
```

위와 같은 방식을 의존 객체 주입 이라고 하는데, 이 방식은 생성자, 정적 팩터리, 빌더 패턴에도 모두 적용할 수 있으며

의존관계가 늘어나도 생성자에 주입을 해주면 되기 때문에 쉽게 유연함을 가질 수 있다.

이 패턴의 활용하는 것으로, 생성자에 자원 팩터리를 넘겨주는 방식이 있다.

> 팩터리란 매 호출 시 특정 타입의 인스턴스를 반복해서 만들어주는 객체를 뜻한다.

Java 8에서 등장한 Supplier<T> 인터페이스가 팩터리를 표현한 그 예시이며 Supplier<T>를 매개변수로 받는 메서드는

제네릭 기능에 의해 자신이 명시한 타입의 하위 타입이라면 무엇이든지 생성할 수 있는 팩터리를 만들 수 있다.

## 의존 객체 주입이 장점만 있나?

그건 아니긴하다. 의존성이 수 천개가 된다면 어지러워진다.

그렇기 때문에 **Spring**, Guice, Dagger 등 **의존 객체 주입 프레임워크**를 활용하면 이 어지러운 상황을 해소할 수 있다.

## 결론

어떤 클래스가 하나 이상의 자원에 의존하고 그 자원이 클래스 동작에 영향을 준다면 싱글턴/정적 유틸리티 클래스는 사용하지 않는 것이 좋다.

이런 상황에서는 의존 객체 주입을 사용해보자.


---

# Item 6. 불필요한 객체 생성을 피하라

## 좋지 않은 예시

```java
String s = new String("diger");
```

위 문장은 실행될 때 마다, String 인스턴스를 만들어내어 굳이 여러개의 객체를 생성하게 된다. (이 컨벤션은 Java 9 부터 Deprecated되었다.)

---

## 좋지 않은 예시 - 개선

```java
String s = "diger";
```

위 방법 뿐만 아니라, Item 1 에서 언급된 **정적 팩터리 메서드**로 불필요한 객체 생성을 피할 수도 있다.

생성비용이 비싼 객체가 있고 이를 반복해서 사용해야한다면, 캐싱하여 재사용 하자.

> 캐싱은 어떻게 사용할 수 있는가?

---

## 생성 비용이 비싼 객체 캐싱 후 재사용

```java
static boolean isRomanNumberal(String s) {
    return s.matches("^(?=.)M*(C[MD] |D?C{0,3})"
    + "(X[CL] |L?X{0,3})(I[XV]|V?I{0,3})$");
}
```

String.matches 는 정규식으로 문자열 형태를 확인하는 가장 간편한 방식이다.

하지만 성능이 중요한 상황에선, 여러번 반복하여 사용하기 적합하지 않다.

왜냐하면, matches 메서드 내부에서 만드는 Pattern 인스턴스가 한 번 쓰고 버려져, 곧바로 GC 대상이 되지만

Pattern은 입력받은 정규식에 해당하는 유한 상태 머신(스위치 처럼, 특정 입력을 계속 기다리는 객체)을 만들기 때문에 성능이 비싸다.

## 생성 비용이 비싼 객체 캐싱 후 재사용

```java
public class RomanNumerals {
    private static final Pattern ROMAN = Pattern.compile(
            "^(?=.)M*(C[MD] |D?C{0,3})" + "(X[CL] |L?X{0,3})(I[XV]|V?I{0,3})$");

    static boolean isRomanNumeral(String s) {
        return ROMAN.matcher(s).matches();
    }
}
```

위 처럼 final 키워드로 불변하는 Pattern 객체를 생성함으로써 캐싱해두고, 실제 정규식을 수행할 메서드에서, 캐싱된 Pattern 인스턴스를 통해 과정을 마친다.

---

## 불필요한 객체를 만들어 내는 상황 - 오토박싱

오토박싱은 프로그래머가 기본타입과 박싱된(Wrapper클래스 타입) 을 섞어서 쓸 때 자동으로 타입 캐스팅을 해주는 기술이다.

    private static long sum() {
        Long sum = 0L;
        for (long i = 0; i <= Integer.MAX_VALUE; i ++)
            sum += i

        return sum;
    }

<br>

위 코드를 보면, Wrapper 클래스인 Long 타입을 사용하여 반복문을 수행한다.

Wrapper 클래스가 아닌, long 타입을 사용하면 위 반복문의 성능은 6.3초 -> 0.59초로 빨라질 수 있다.

---

## 결론

객체를 생성하지 말라는 것이 아니다.

DB 연결생성과 같은 무거운 객체를 다룰 땐 캐싱 or 기본 타입을 사용하여 객체의 불필요한 생성을 줄여보자.


---

# Item 7. 다 쓴 객체 참조를 해제하라

## 메모리 누수가 발생하는 Stack

```java
import java.util.Arrays;
import java.util.EmptyStackException;

public class Stack {

    private Object[] elements;

    private int size = 0;

    private static final int DEFAULT_INITIAL_CAPACITY = 16;

    public Stack() {
        this.elements = new Object[DEFAULT_INITIAL_CAPACITY];
    }

    public void push(Object e) {
        this.ensureCapacity();
        this.elements[size++] = e;
    }

    private void ensureCapacity() {
        if (this.elements.length == size) {
            this.elements = Arrays.copyOf(elements, 2 * size + 1);
        }
    }

    // !!!!! 메모리 누수 !!!!!
    public Object pop() {
        if (size == 0) {
            throw new EmptyStackException();
        }
        return elements[--size];
    }

    private void ensureCapacity() {
        if (this.elements.length == size) {
            this.elements = Arrays.copyOf(elements, 2 * size + 1);
        }
    }
}
```

위 코드가 왜 누수가 발생할까?

다 쓴 참조가 발생한다.

### 다 쓴 참조란?

더 이상 쓰지 않을 참조 라는 뜻이다.

즉, 내가 메모리를 직접 관리하는 클래스가 있다면, 직접 레퍼런스를 null 로 셋팅해주어, GC 에게 사용하지 않는다는 것을 알려야 한다는 것이다.

만약 이 문제를 해결하지 않고 쌓아둔다면 OOM(OutOfMemory)가 발생할 가능성이 생긴다.

위 코드에서는 elements배열의 "**활성 영역**" 밖의 참조들이 다 쓴 참조에 해당하게 된다.

### 활성 영역이란 무엇일까?

![img.png](https://github.com/K-Diger/K-Diger.github.io/blob/main/images/WhatIsActiveAreaInJavaArray.png?raw=true)

그냥 배열의 모든 원소라고 보면 될 것 같다.

즉 사이즈가 100인 배열이 있다면, 0 ~ 99가 배열의 활성 영역에 해당한다.

### 저런건 GC가 알아서 못가져가나?

GC는 메모리 누수를 찾기가 너무 어렵다.

GC의 객체 수집 기준을 정리하면 다음과 같다.

GC는 "Tracing"이라는 과정을 통해 사용하지 않는 객체를 찾아낸다. 그 기본 기준은 더 이상 참조 받고 있지 않은 객체를 지정하고 있다.

GC가 탐색하는 순서는 "Live Thread"와 Static 영역을 살펴보는 것으로 시작한다.

Live Thread란 현재 실행 중 혹은 향후 실행할 스레드를 나타내는 것으로 "**활성된 스레드**"라고 보면 된다.

다시 정리하면 GC는 **Live Thread에서 시작하여 도달 가능한 모든 객체의 참조를 찾아**낸다.

Live Thread에서 **도달 가능한 객체는 GC의 대상에서 제외**된다.

그리고 **특정 객체를 수집하려면, 그 객체가 참조하는 모든 객체도 수집**해야한다!

그렇기 때문에 **GC는 다른 객체의 참조를 찾기 위하여 재귀적으로 탐색**하는 알고리즘을 가지고 있다.

또한 **도달 불가능한 객체를 참조**로 가지고있는 **도달 가능한 객체**는

**도달 불가능한 객체가 수집**되고 **그 이후에 도달 가능한 객체**가 아무것도 참조하지 않기 때문에 **GC 대상으로 선정**된다.

여기서 도달 불가능한 객체를 예시로 보면 다음과 같다.

```java
String str = new String("Hello, World!");
    str = null; // str is no longer being referenced

or

public void someMethod() {
    SomeObject obj = new SomeObject();
    // obj is reachable
    //... some code
    // obj is no longer reachable
}
```

someMethod가 종료되면 SomeObject에 접근할 수 있는 참조가 존재하지 않기 때문에 SomeObject 또한 GC 대상이 되는 것이다.

---

## 메모리 누수를 방지한 코드 - pop() 메서드 수정

```java
import java.util.Arrays;
import java.util.EmptyStackException;

public class Stack {

    private Object[] elements;

    private int size = 0;

    private static final int DEFAULT_INITIAL_CAPACITY = 16;

    public Stack() {
        this.elements = new Object[DEFAULT_INITIAL_CAPACITY];
    }

    public void push(Object e) {
        this.ensureCapacity();
        this.elements[size++] = e;
    }

    private void ensureCapacity() {
        if (this.elements.length == size) {
            this.elements = Arrays.copyOf(elements, 2 * size + 1);
        }
    }

    // !!!!! 메모리 누수 방지 !!!!!
    public Object pop() {
        if (size == 0) {
            throw new EmptyStackException();
        }

        Object result = this.elements[--size];
        elements[size] = null; // 다 쓴 참조를 해제한다!
        return result;
    }
}

```

이렇게 null 참조를 통하여 메모리 공간을 확보해주는 것과 동시에,

null 처리한 참조를 사용하려 할 때 NPE 를 발생 시킬 수 있게 됨으로써 잘못된 행위를 수행할 경우를 방지할 수 있는 장점도 생긴다.

---

## 캐시 또한 메모리 누수의 원인

객체 참조를 캐시에 넣고, 캐시에 넣은 객체를 사용하고 나서 메모리에 올려두는 상황에서 누수가 발생한다.

---

## 리스너, 콜백 또한 메모리 누수의 원인

자바 생태계에서 굳이 이 컨벤션을 쓸 일이 있을까? 싶지만

콜백에 적당한 return 으로 메모리 해지를 하면 메모리 누수를 방지할 수 있다.

또한 콜백을 위한 week reference로 저장하면 GC가 수거해갈 수 있게 된다. (ex : WeakHashMap)

---

# Item 8. finalizer 와 cleaner 사용을 피하라

## Java 의 두 가지 소멸자

### finalizer

실행 시점을 예측할 수 없고, 오동작-낮은성능-이식성 문제의 원인이 된다.

Java 9 부터는 Deprecated 가 되어 cleaner를 대안으로 소개한다.

### cleaner

finalizer 보다는 덜 위험하지만, 여전히 실행 시점을 예측할 수 없고, 느리다.

## finalizer, cleaner 존재 목적

사용하지 않은 객체를 Heap 영역에서 제거시키기 위함. (GC 구현체가 알아서 사용한다.)

---

## Finalizer - Cleaner 왜 쓰지 말아야 할까?

### 이유 1.
finalizer Cleaner 는 사용되는 시점이 모호할 뿐 만 아니라 코드로 적어놓았다 한들, 사용될지 안될지도 보장되지 않는다..

따라서 상태를 영구적으로 수정하는 작업에서 **절대** finalizer, cleaner 를 사용하면 안된다.

---

### 이유 2.

finalize 동작 중 발생한 예외는 무시되고, 처리할 작업이 남아있더라도 애플리케이션이 종료된다.

심각한 RunTime Error 를 발생할 수 있는 가능성을 활짝 열어두게 되는 것이다.

---

### 이유 3.

가비지 컬렉터에 등록되어있는 finalizer 알고리즘이 있는데, 프로그래머가 굳이굳이 본인이 수동으로

finalizer 를 하게되면, 해당 알고리즘에 문제가 생기는지 심각한 성능 차이가 생기게 된다.

> 간단한 객체를 생성하고 Head 영역에 삭제할 때 성능 지표
>
> GC 에게 맡겼을 때 : 12ns
>
> 수동으로 진행했을 때 : 550ns

---

### 이유 4.

finalizer 를 사용한 클래스는 취약점이 생기게 된다.

생성자, 직렬화 과정에서 예외가 발생할 때, 생성이 되다 중지된 객체에서 하위 클래스의 finalizer 가 수행될 수 있게 된다.

이게 무슨 말이냐면, A라는 객체가 50% 정도는 생기게 된 상태라서, A 객체가 생성된 시점부터 A 객체를 상속받는 B라는 객체가 있을 때

B 객체에서 finalizer 가 수행될 수 있게 되는 것이다.

그리고 이 B 의 finalizer 가 static 필드에 자신의 참조를 할당하여, 가비지 컬렉터가 수집하지 못하게 할 수 있다.

이렇게되면.. B 에서 A에게 상속받은 기능을 사용하려하는데, 사용이 안될 상황이 벌어지고 이는 찾을 수 없는 오류로 남게 될 것이다.

---

## 그럼에도 불구하고 finalizer 가 어디에서 쓰이는가...?

### 쓰임새 1.
자원의 소유자가 close 메서드를 소출하지 않는 것을 대비하여

자원을 늦게 회수하더라도, 회수하지 않은 것을 방지하기 위함

### 쓰임새 2.

네티이브 피어와 연결된 객체이다.

네이티브 피어는, 자바 객체가 네이티브 메서드(다른 언어로 제작된 메서드)를 통해 기능을 위임받은 객체를 말한다.

네이티브 피어는 자바 객체가 아니므로, GC 가 회수하지 못한다.

이럴때가 finalizer가 사용되기 좋은 상황이다.

finalizer 로 성능 저하가 우려된다면, close 메서드를 사용해야한다.

---

# Item 10. equals는 일반 규약을 지켜 재정의하라

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

---

# Item 11. equals를 재정의할꺼면, hashCode도 재정의해라.

## equals를 재정의한 클래스 모두에서 hashCode를 재정의해야 한다.

hashCode를 재정의 하지 않으면, hashCode일반 규약을 어기게 되어 해당 클래스의 인스턴스를 HashMap, HashSet과 같은 컬렉션의 원소로 사용할 때 문제가 될 것이다.

Object 내에 명세된 hashCode, equals 규약

> 1.equals 비교에 사용되는 정보가 변경되지 않았다면, hashCode메서드는 몇 번을 호출해도 항상 같은 값을 반환해야한다.
>
> 단, 애플리케이션을 다시 실행했을 때는 이 값이 달라져도 상관없다.
>
> 2.equals가 두 객체를 같다고 판단했다면, 두 객체의 hashCode는 똑같은 값을 반환해야한다.
>
> 3.equals가 두 객체를 다르다고 판단했더라도, 두 객체의 hashCode가 서로 다른 값을 반환할 필요는 없다.
>
> 하지만, 다른 객체에 대해 다른 값을 반환해야 해시테이블의 성능이 좋아진다. (아마도 테이블 충돌방지 기법인 체이닝 때문에 그런것 같다.)

---

## hashCode 재정의를 잘 못했을 때

크게 문제가 되는 부분은 두 번째 조항으로, 논리적으로 같은 객체는 같은 hashCode를 반환해야한다.

equals는 물리적으로 다른 두 객체를 논리적으로 같다고 할 수는 있다. 하지만 Object의 기본 hashCode 메서드는 이 둘이 전혀 다르다고 판단하여

규약과 달리 서로 다른 값을 반환하게 된다. 다음 예시를 보면 이 상황이 이해가 갈 것이다.

    Map<PhoneNumber, String> m = new HashMap<>();
    m.put(new PhoneNumber(707, 123, 456), "Diger");

    // 아래 코드의 결과값은 무엇일까?
    m.get(new PhoneNumber(707, 123, 456));

위 코드는 null을 반환하게 된다. 왜 일까?

2개의 인스턴스가 사용된 것이라는 이유 때문이다. 1. HashMap에 "Diger"를 넣을 때 사용되었고, 2. 이를 꺼낼 때 사용되었다.

그니까 이게 무슨말인고,, 함은 put을 할 때 생성한 PhoneNumber(A라고 하자) 객체는 A'이라는 해시값을 가진 상태로 A에 대한 해시 테이블에 저장되었다.

그리고 get을 할 때 생성한 PhoneNumber(B라고 하자) 객체는 B라는 해시테이블을 이용하게 된다.

여기서 알 수 있듯이 위에서 생성한 두 객체의 서로가 가진 테이블 정보가 다르다. 논리적 동치인 PhoneNumber를 가졌지만 실제로 저장된 구역이 달라

B라는 테이블에서 값을 조회 해버리게되어 null이 반환되는 것이다.

따라서, PhoneNumber Class는 hashCode가 재정의 되어있지 않기 때문에, 두 객체가 서로 다른 해시코드를 반환하여 두 번째 규약을 지키지 못한 것이다.

    public class PhoneNumber {

        private String number;
        public PhoneNumber(String number) {
            this.number = number;
        }
    }

    public class HashCodeTest {

        public static void main(String[] args) {
            Map<PhoneNumber, String> m = new HashMap<>();

            m.put(new PhoneNumber("123456789"), "Diger");
            System.out.println(m.get(new PhoneNumber("123456789")));
        }
    }

    출력결과 : null


코드로 보면 누가 이렇게 put/get을 하나 싶긴한데 일단 그건 넘어가고, 이러한 상황을 벗어날 수 있는 방법을 알아보자.


### 방법 1.

    @Override
    public int hashCode() {
        return 42;
    }

이러면 모든 객체에서 똑같은 해시코드를 반환하니 괜찮아 보인다. 근데... 만약 이러한 객체를 100만개 생성하면 해시테이블 충돌이 감당할 수 없게 된다. (해시 테이블 조회 시 장점이 사라짐)

좋은 해시 함수라면 서로 다른 인스턴스에 다른 해시코드를 반환해야한다. (hashCode의 세 번째 규약)

### 방법 2.

    @Override
    public int hashCode() {
        int result = Short.hashCode(areaCode);
        result = 31 * result + Short.hashCode(prefix);
        result = 31 * result + Short.hashCode(lineNum);
    }

첫 번째 구문부터 해설하자면, 객체의 첫 번째 핵심 필드(equals 비교에 사용되는 필드)를 기본 타입에 대한 hashCode를 계산하는 구문이다.

두 번째 구문 부터는, 해당 객체의 나머지 필드에 대해 각각 작업을 수행하는 것인데 간략하게 요약하자면 hash 함수를 우리가 직접 정의한 내용인 것이다.


    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (o == null || getClass() != o.getClass()) return false;
        PhoneNumber that = (PhoneNumber) o;
        return Objects.equals(number, that.number);
    }

    @Override
    public int hashCode() {
        return Objects.hash(number);
    }

위 코드는 IDE에서 생성해주는 equals 및 hashCode 구문이다.

Object.hash() 메서드를 파고 들어가 보면 Java Library가 생성한 다음과 같은 코드가 작성되어 있다.

    public static int hashCode(Object a[]) {
        if (a == null)
            return 0;

        int result = 1;

        for (Object element : a)
            result = 31 * result + (element == null ? 0 : element.hashCode());

        return result;
    }

방법 2. 시작에서 작성한 코드랑 상당히 비슷하다. 이정도만 써도 대부분의 경우에서 문제가 없는 것이다.

또한 한 가지 유의해야할 점은, 해시 로직을 외부에 노출되도록 하면 안된다. 하지만 지금 보듯이 Java에서 제공하는 라이브러리에서는 이미 그 로직을 알 수 있는데

이는 현재로써는 어쩔 수 없는 사항이라고 넘어가야한다.

---

# Item 14. Comparable을 구현할지 고려하라

## Comparable 인터페이스의 유일한 메서드 - compareTo()

compareTo() 메서드는 Object클래스의 equals와 매우 유사하다. 다른 점은 딱 두가지가 있는데

compareTo는 동치성 비교와 순서를 비교할 수 있고 제네릭으로 사용할 수 있다는 것이다.

또한 이 메서드를 구현했다는 것은 그 클래스의 인스턴스는 순서를 가진다는 것을 의미하는 것으로 아래와 같이 간단하게 정렬을 수행할 수 있다.

Comparable을 구현한 객체들의 배열은 다음과 같이 정렬 가능하다.

```java
Arrays.sort(a);
```

---

## compareTo() 메서드를 구현하는 것을 고려하기 전에 알고 갈 내용

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

### 그렇다면 왜 0을 반환하는가? 그것도 Integer 타입으로??

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

## 이정도 되었으니 Comparable을 구현할지 정해야하는 기준을 알아보자.

일단 우리가 주로 사용하는 값 클래스 및 열거타입은 모두 Comparable을 구현해뒀다.

이 말이 무슨 뜻이냐면, 알파벳 - 숫자 - 년도 등 순서가 명확한 값 클래스를 작성할 때는 반드시 Comparable 인터페이스를 구현해야한다는 것이다.

## Comparable을 구현할 때 주의할 점은 뭘까?

equals 규약과 똑같다. 이를 철저하게 지키긴해야한다.

> x.compareTo(x) == -y.compareTo(x)
> x.compareTo(x) > 0 && y.compareTo(z) > 0 이면, x.compareTo(z) > 0이다.
> x.compareTo(x) == y.compareTo(z) 이면, x.compareTo(z) == 0이다.
> (x.compareTo(x) == 0) == (x.equals(y))이다.

이 규약을 꼭 지켜야하는이유는, 정렬된 컬렉션인 TreeSet, TreeMap 검색과 정렬 알고리즘을 활용하는 Collections, Arrays에 있다.

TreeSet, TreeMap, Collections, Arrays 는 모두 정렬을 위한 과정에서 내부적으로 compareTo() 메서드를 사용하기 때문이다.

그리고 정렬된 컬렉션들은 동치성을 비교할 때 equals()메서드 대신, compareTo()메서드를 사용하기 때문에 이를 꼭 인지해야한다.

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

---

# Item 15, 16. 클래스와 멤버의 접근을 최소화하라, Public 클래스에서는 Public 필드가 아닌 접근자 메서드를 사용하라.

## 잘 설계된 컴포넌트

모든 내부 구현을 완벽히 숨겨, 구현과 API를 통해서만 다른 컴포넌트와 소통하며, 서로의 내부 동작 방식에는 전혀 개의치 않는다.

> 여기서 말하고자 하는 다른 컴포넌트와의 소통은 무엇인가?

---

## 정보 은닉의 장점

보통 객체지향을 처음 배우면 **캡슐화** 라는 용어를 배우게 된다. 캡슐화는 쉽게 말하면 어떤 클래스가 있을 때 그 내부 필드에는 접근못하도록 **접근제어자를** 손보는 것이다.

그렇다면 이 행위의 장점은 무엇인가?

- 시스템 개발 속도 향상 --> 여러 컴포넌트를 병렬로 개발할 수 있게되기 때문이다.
- 시스템 관리비용 절감 --> 각 컴포넌트를 더 빨리 파악하여 디버깅 할 수 있게 된다.
- 정보 은닉 자체가 성능 향상에 직접적인 도움이 되진 않는다. 하지만 완성된 시스템 내에서 특정 정보 은닉된 요소만 변경하면 다른 컴포넌트에 영향 없이 최적화가 가능하다.
- 재사용성
- 제작 난이도 완화

## 정보 은닉의 원칙

모든 클래스와 멤버의 접근성을 가능한 좁혀야한다.

패키지 외부에서 쓸 이유가 없다면, package-private으로 선언해야한다.

여기서 package-private는 무엇인가? --> 사실 내가 잘 모르는 용어여서 찾아봤다.

그리고 의외로 내가 자주 사용하던 방식이였다.

```java
package user;

public class UserClass {

	private String name;

	String getName() {
		return name;
	}
}
```
이처럼 어떤 클래스의 메서드는 public 하지만, 그 필드는 private 한 형식을 package-private이라고 한다.

스프링으로 개발할 때 Entity 혹은, 의존성 주입을 받는 Controller, Service 등에서 자주 사용하던 방식이다.

여기서 주목하고자 하는 구절은

> public 클래스의 인스턴스 필드는 되도록 public이 아니어야 한다. 라는 것이다.

우리가 당연하게도 알고 있던 이유 때문이다. public 필드는 외부에서 수정이 가능하기 때문에 스레드 안전하지 않다.

그러면 final로 외부에서 변경하지 못하게 하면 되는것 아닌가??

이건 바바리맨을 잡고자 바바리코트를 못입게 만드는... 것이다. 이 클래스의 필드가 의미가 없어진다. 수정을 나조차도 할 수 없기 때문이다.

그리고 주의해야할 점은 배열의 사용에 있다.

```java
public static final Thing[] VALUES = {...};
```

위와 같이 static final인 배열은 길이가 0이 아니라면 모두 변경이 가능하다. 따라서 다음과 같은 배열 필드를 선언하거나 반환하지 않도록 해야한다.


정 위와같은 배열을 쓰고 싶다면 다음 2가지의 해결방법이 있다.

방법 1.
```java
private static final Thing[] PRIVATE_VALUES) = {...};
public static final List<Thing> VALUES = Collections.unmodifiableList(Arrays.asList(PRIVATE_VALUES));
```
배열을 private으로 선언하고, 이를 다룰 불변 List를 만들어 사용하는 것이다.

방법 2.
```java
private static final Thing[] PRIVATE_VALUES) = {...};
public static final Thing[] values() {
    return PRIVATE_VALUES.clone();
    }
```

다음과 같이 배열을 private으로 만들고 그 복사본을 반환하는 public 메서드를 만드는 것이다. (방어적 복사)

---

## Item 15. 클래스와 멤버의 접근 권한을 최소화하라 - 결론

프로그램의 접근성은 최소화, 꼭 필요한 내용만 public으로 설정해야한다.

또한 public class는 상수용 public static ifnal 필드 외에는 **어떤** 필드도 public 으로 선언하면 안된다.

public static final 필드가 참조하는 객체가 불변인지도 꼭 체크해야한다.

---

## 캡슐화를 극대화 해보자.

```java
class Point {
    private double x;
    private double y;

    public Point(double x, double y) {
        this.x = x;
        this.y = y;
    }

    public double getX() {return x;}
    public double getY() {return y;}
}
```

하지만 package-private 클래스나 혹은 private 중첩 클래스라면 데이터 필드를 노출해도 상관없다. --> 이게 무슨말일까.... (p.103)

### Java 라이브러리에서도 public 클래스의 필드를 노출하는 경우가 있다!!

java.awt.package 패키지의 Point와 Dimension 클래스이다.

이는 추후에 등장하겠지만 심각한 성능문제가 발생하고 있는 라이브러리이므로 예외 케이스가 아님을 꼭 짚고 넘어가자.


## Item 16. Public 클래스에서는 Public 필드가 아닌 접근자 메서드를 사용하라 - 결론

가변 필드에 접근하도록 하려면, 어지간하면 Getter, Setter를 사용하자!!

불변필드라면 노출해도 괜찮지만 그래도 완전하게 안심할 수는 없다.

그렇지만 package-private 클래스나 private 중첩 클래스에서는 불변,가변 상관없이 노출하는 것이 나을 때도 있다. (이게 도대체 무슨소리람!..)


---

# Item 17. 변경 가능성을 최소화하라

## 변경 가능성 최소화하기 - 불변 클래스

불변 클래스란?

인스턴스의 내부 값을 수정할 수 없는 클래스이다.

불변 클래스에 포함된 정보는 고정되어 그 정보는 절대로 달라지지 않는다.

ex) String, Integer, Long, BigInteger, BigDecimal 등

## 불변 클래스를 사용하는 이유는?

오류가 생길 여지가 적어 안전하다. 외부에서 값을 수정 할 수 없기 때문에 잘 만들어만 놓았다면 이로 인한 문제가 발생할 여지가 없기 때문이다.

## 클래스를 불변으로 만들기 - 5가지 규칙

- 1. Setter와 같은 수정자를 지양

- 2. 클래스를 확장하지 못하도록
    - 클래스를 final로 선언하는 방법이 그 예시이다.

- 3. 모든 필드를 final로 선언

- 4. 모든 필드를 private 으로 선언
    - public final으로 선언해도 충분할 수 있지만, 다음 릴리스에서 내부 표현을 바꾸지 못할 수 있다.

- 5. 자신 외에는 내부 가변 컴포넌트에 접근할 수 없도록 선언
    - 클래스에 가변 객체를 참조하는 필드가 하나라도 있다면 클라이언트에서 그 참조를 얻지 못하도록 해야한다.

### 5. 자신 외에는 내부 가변 컴포넌트에 접근할 수 없도록 선언 - 예시

```java
public class Post {

    // 가변 객체를 참조하는 필드를 외부에서 접근할 수 없음!
    private User user;
}
```

## 불변 복소수 클래스 - 불변 클래스 예시

```java
public final class Complex {
    private final double re;
    private final double im;

    public Complex(double re, double im) {
        this.re = re;
        this.im = im;
    }

    public double realPart() {
        return re;
    }

    public double imaginaryPart() {
        return im;
    }

    public Complex plus(Complex complex) {
        return new Complex(re + complex.re, im + complex.im);
    }

    ...
}
```

- readPart() 메서드는 실수부를 반환하는 접근자 메서드

- imaginaryPart() 메서드는 허수부 값을 반환하는 접근자 메서드이다.

- plus() 메서드는 말 그대로 덧셈을 수행하는 메서드로 인스턴스 자신을 수정하지 않고 새로운 Complex 인스턴스를 만드는 것을 주목해야한다.

plush() 메서드는 피연산자에 함수를 적용하여 그 결과를 반환하지만, 피연산자 자체는 그대로인 패턴으로 **함수형 프로그래밍** 이라고 한다.

절차적 혹은 명령형 프로그래밍에서는 메서드에서 피연산자인 자신을 수정한다.

즉, this.complex.re = re + complex.re;

와 같이 실제 필드를 변경하여 상태가 변경하게 된다는 것이다.


이렇게 실제 필드를 직접 변경하지 않는다는 것을 나타내기 위해 메서드 네이밍에도 디테일을 담을 수 있는데

보통 덧셈 연산이라고 하면 **add**라는 동사로 네이밍 할 수 있겠지만, **plus**라는 전치사를 활용하여 객체의 값을 변경하지 않는다는 점을 강조할 수 있다.

## 불변 클래스의 장/단점

### 장점 1. 불변 객체는 단순하다.

생성된 시점의 상태를 파괴될 때 까지 간직하기 때문에 어떠한 변화에 영향을 받았는지 신경쓰지 않아도 된다.

### 장점 2. 스레드 안전하다.

여러 스레드에서 동시에 접근하더라도 값 자체는 절대 변하지 않기 때문에 동시성 문제를 유발할 여지가 없다.

### 장점 3. 스레드 안전함으로 공유성이 확보된다.

어떤 스레드이던 다른 스레드에 영향을 줄 수 없으니 공유할 수 있다.

### 장점 4. 불변 클래스 인스턴스를 공유하여 메모리를 절약할 수 있다.

```java
public static final Complex ZERO = new Complex(0, 0);
public static final Complex ONE = new Complex(1, 0);
public static final Complex I = new Complex(0, 1);
```

위와 같이 public static final 키워들를 통해 상수를 활용하는 것으로 인스턴스를 재활용 할 수 있다.

또한 이 방법을 조금 더 클린하게 사용하는 방법인 **정적 팩터리**가 있다.

WrapperClass 및 BigInteger 등이 모두 이 정적 팩터리를 사용하고 있다.

정적 팩터리를 사용하면 여러 클라이언트가 인스턴스를 공유하기 때문에 메모리 사용량/GC 비용이 줄어든다.

또한 이 방식을 더 극대화하면 캐시 기능을 덧붙일 수 있기 때문에 더 권장하는 방식이다.

---

### 장점 5. 방어적 복사가 필요없다.

아무리 복사해봐야 원본과 똑같기 때문이다. String 클래스는 clone() 메서드를 재정의 했지만, 이는 이 사실이 잘 알려지지 않은 Java 초창기에 만들어 진 내용이므로

사용하지 않는 것이 권장된다.

### 장점 6. 불변 객체는 그 자체로 실패 원자성을 제공한다.

즉, 어떤 시점에 접근하더라도 같은 값을 반환하기 때문에 원자성을 제공할 수 있다는 것이다.

---

### 단점 1. 값이 다르면 반드시 독립된 객체로 만들어야한다.

값의 가짓수가 많으면 만약의 각 다른 값이 100개라고 치면, 100개의 인스턴스를 만들어줘야한다.

사실 단점이 한가지로 보이지만 이게 엄청나게 큰 단점이다. 그렇기 때문에 이 단점을 조금 더 개선할 수 있는 방법을 알아보자.

## 불변객체를 사용하는 상황의 단점 개선하기

### 개선 방법 1. 흔하게 쓰일 다단계 연산들을 예측하기

쉽게 말하면 BigIngter 라는 불변 클래스에 add, minus, multiply, divide 등과 같은 자주 쓰일 것 같은 메서드를 public으로 열어둔다면 어느정도 해결된다는 것이다.

## 불변 클래스를 만드는 방법

### 1. 상속이 불가능하게 하는 방법 - final 클래스로 만들기

### 2. 상속이 불가능하게 하는 방법 - 모든 생성자를 private, 혹은 package-private으로 만들기

우리가 잘 아는 정적 팩터리를 이용한 방식으로 한다면 상속이 불가능한 불변 객체를 조금 더 유연하게 사용할 수 있따.

```java
public class Complex {
    private final double re;
    private final double im;

    private Complex(double re, double im) {
        this.re = re;
        this.im = im;
    }

    public static Complex valueOf(double re, double im) {
        return new Complex(re, im);
    }
}
```

public 생성자가 없으면 상속이 불가능한 점을 이용하는 것으로 위와 같이 설계한다면 사실상 final class 와 같게 된다.

---

## 정리

- Getter가 있다고 해서 무조건 Setter를 만들지 말자.

- 클래스는 꼭 필요한 경우가 아니라면 불변이어야 한다.

- 단순한 값 객체는 항상 불변으로 만들자.
    - (DTO도 값 객체라고 보고 불변으로 만들면 될까? Entity 객체도 사실상 값 객체가 아닌가..?)

- 불변 클래스와 쌍을 이루는 가변 동반 클래스를 public 클래스를 제공하자.

- 모든 클래스를 불변으로 만들 수 없다. 따라서 변경할 수 있는 부분을 최소한으로 줄여야한다.

- 변경이 필요하지 않은 필드는 final로 선언하는 것을 잊지 말자!

- 다른 합당한 이유가 없다면 모든 필드는 private final로 선언되어야한다.

- 생성자는 불변식 설정이 모두 완료되어 초기화가 완벽히 끝난 상태의 객체를 생성해야한다.


---

# Item 19. 상속을 고려하여 설계하고 문서화 그렇지 않으면 상속을 금지하기

## 상속을 고려한 설계와 문서화

### 1. 메서드를 재정의할 때 어떤 일이 일어나는지 정리한다.

상속이 가능한 클래스는 재정의할 수 있는 메서드들을 내부적으로 어떻게 이용하는지 문서화 해야한다.

어떤 순서로 호출하는지 각각의 호출 결과가 이어지는 처리에 어떤 영향을 주는지도 담아야한다.

- 여기서 재정의 가능이란, public, protected 메서드 중 final이 아닌 모든 메서드를 이야기한다.

재정의 가능 메서드를 호출할 수 있는 모든 상황을 문서로 남겨야 한다. (백그라운드 스레드, 정적 초기화 과정 등에서도 호출이 일어날 수 있다.)

### 2. 클래스의 내부 동작 과정 중간에 끼어드는 Hook을 protected 메서드 형태로 공개해야할 수도 있다.

내부 메커니즘을 문서로 남기는 것으로 상속을 위한 설계의 끝이 아니다.

드물게는 protected 필드로 공개해야 할 수도 있다.

그런데, 어떤 메서드를 protected로 노출할지는 어떻게 결정하는가?

실제로 하위 클래스를 만들어 시험해보는 것이 최선이다. 이는 유일한 방법이며 꼭 필요한 방법이다.

### 3. 상속용 클래스의 생성자는 직접적/간접적으로 재정의 가능 메서드를 호출해서는 안된다.

상위 클래스의 생성자가 하위 클래스의 생성자보다 먼저 실행되므로 하위 클래스에서 재정의한 메서드가 하위 클래스의 생성자보다 먼저 호출된다.

```java
public class Parent {

    public Parent() {
        parentOverrideTargetMethod();
    }

    public void parentOverrideTargetMethod() {
    }
}

public final class Child extends Parent {

    private final Instant instant;

    Child {
        instant = Instant.now();
    }

    @Override
    public void parentOverrideTargetMethod() {
        System.out.println("먀");
    }

    public static void main(String[] args) {
        Child child = new Child();
        child.parentOverrideTargetMethod();
    }
}
```

위와 같은 코드가 ## 3. 상속용 클래스의 생성자는 직접적/간접적으로 재정의 가능 메서드를 호출해서는 안된다. 의 핵심이다.

또한 clone과 readObject 메서드는 생성자와 비슷한 효과를 내므로 이를 재정의 하는 로직에 직접적/간접적으로 재정의 가능 메서드를 호출해서는 안된다.

Serializable 를 구현하는 상속용 클래스도 주의해야할 점이 있다.

readResolve나 writeReplace 메서드를 갖는다면 private이 아닌 protected로 선언해야한다.

private으로 선언한다면 하위 클래스에서 무시되기 때문이다.

---

## 결론

- 상속용 클래스를 설계할 때 클래스 내부에서 스스로를 어떻게 사용하는지 모두 문서화를 해야한다.

- 다른 곳에서 더 좋은 하위 클래스를 만들 수 있도록 일부 메서드를 protected를 제공해야할 때도 있다. 이는 직접 하위클래스를 만들고 실험해보며 어떤 메서드를 protected로 할지 결정해야한다.

- 클래스를 확장할 명확한 이유가 없다면 상속을 금지하는 편이 나을 수도 있다. final class 를 활용하거나, 모든 생성자를 private화 하는 것이 그 방법이다.

---

# Item 20. 추상 클래스보다는 인터페이스를 우선하라

## 추상 클래스 vs 인터페이스

추상클래스가 정의한 타입을 구현하는 클래스는 반드시 추상 클래스의 하위 클래스가 되어야 한다.

추상클래스는 새로운 타입을 정의하는데 큰 제약이 있게 되는 것이다.

인터페이스가 선언한 메서드를 정의하고, 규약을 잘 지킨 클래스라면, 어떤 클래스를 상속했던간에 같은 타입으로 취급된다.

---

## 인터페이스는 믹스인 정의에 아주 잘 맞는다.

믹스인이란 클래스가 구현할 수 있는 타입으로, 믹스인을 구현한 클래스에 원래의 타입 + 행위를 제공하게 된다.

이처럼 대상 타입의 주된 기능에 선택적인 기능을 혼합한다해서 Mixed In 이라고 부르는 것이다.

추상 클래스로는 믹스인을 정의할 수 없다. 기존 클래스에 덧씌울 수 없기 때문이다.

---

# Item 57. 지역변수의 범위를 최소화하라

## 지역변수의 범위를 줄이는 가장 가장 강력한 기법

> 해당 지역변수가 가장 처음 쓰일 때 선언하기


> 모든 지역변수는 선언과 동시에 초기화하기.
>
> + 만약, 초기화에 필요한 정보가 충분하지 않다면 충분해 질 때까지 선언을 미루기


## 컬렉션을 순회할 때 권장사항

### for-each

    for (Element e : c) {
        doSomething();
    }

위와 같이 for-each 의 형태로 순회하는 것이 적합하다.

반복자가 필요한 상황에서는 아래와 같이 수행하면된다.

### 반복자가 필요한 상황

    for (Iterator<Element> i = c.iterator(); i.hasNext();) {
        Element e = i.next();
    }

---

# Item 58. 전통적인 for 문 보다는 for-each 문을 사용하라

## 컬렉션/배열 순회하기

### 전통적인 방법으로 컬렉션 순회하기

    for (Iterator<Element> i = c.iterator(); i.hasNext(); {
        Element e = i.next();
    }

### 전통적인 방법으로 배열 순회하기

    for (int i = 0; i < a.length; i ++) {
        a[i] = 1;
    }

### 개선된 방법으로 컬렉션/배열 순회하기

    for (Element e : elements) {
        doSomething();
    }

---

## for-each 문을 사용할 수 없는 상황 3가지

### 1. 파괴적인 필터링

컬렉션을 순회하면서, 선택된 원소를 **제거** 해야할 상황에서는, 반복자의 remove 메서드를 호출해야한다.

Java 8 부터 Collection 의 removeIf 메서드를 사용하여 컬렉션을 명시적으로 순회하는 일을 피할 수 있다.

#### 예시코드

    List<Integer> numbers = new ArrayList<>(Arrays.asList(1, 2, 3, 4, 5, 6, 7, 8, 9));

    numbers.removeIf(n -> (n % 3 == 0));

3의 배수에 해당하는 원소를 제거하는 메서드로 위와 같이 Lambda 표현식을 활용하여 removeIf 메서드를 사용할 수 있다.

### 2. 변형

컬렉션이나 배열을 순회하면서, 그 원소의 값이나 전체를 교체해야한다면 리스트의 반복자나 배열의 인덱스를 사용해야한다.

#### 반복자 예시

    public void iteratorTest() {
        List<String> list = new ArrayList<>();

        list.add("diger");
        list.add("song");
        list.add("john");

        Iterator<String> iter = list.iterator();

        while(iter.hasNext()) {
            String s = iter.next();
            s = "변경테스트";
            System.out.println(s);
        }
    }

#### 반복자 예시 출력 결과

    변경테스트
    변경테스트
    변경테스트

### 3. 병렬 반복

여러 컬렉션을 병렬로 순회해야 한다면, 각각의 반복자와 인덱스 변수를 사용하여 엄격하게 제어해야한다.

> 위 세가지 상황에서는 일반적인 for 문을 사용하되 다른 상황이라면 for-each 를 권장한다. 성능 차이도 없다.

---

# Item 60. 정확한 답이 필요하다면 float와 double은 피하라

## float, double

넓은 범위의 수를 빠르게 정밀한 **근사치**로 계산하도록 설계되었다.

> 즉, 근사치이기 때문에 정확한 값이아니다. 최대한 가까운 값을 보여주는 것 뿐이다.

그렇기 때문에 정확하게 계산해야하는 상황에서는 float, double을 피해야할 필요가 있다.

## 근사치값을 볼 수 있는 예시

### 1달러 3센트가 있을 때, 42센트 물건을 구매한 상황

    System.out.println(1.03 - 0.42);

위 코드를 실행한 결과는 다음과 같다.

    0.6100000000000001

우리의 수학적인 계산을 수행하면 그냥 0.61이라는게 계산될 문제인데

실제 출력을 까보면 소수점 저 아래에, 1이라는 소수점이 붙어있다.

## 근사치값을 볼 수 있는 예시 2

### 1달러가 있을 때, 10센트 짜리 물건 9개를 구매한 상황

    System.out.println(1.00 - 0.9);

위 코드를 실행한 결과는 다음과 같다.

    0.09999999999999998

---

## 근사치가 그렇게 문제인가?

아직 적절한 예시를 찾지 못했다. 하지만 그냥 상상을 해보자

어떠한 요청을 10억번을 처리한다고 해보자.

그렇게 되면 저 근사치로 인한 잘못된 소수점이 쌓이고 쌓여 결국에는 소수점이 아닌 실제 정수 부분의 값에도 영향이 끼치게 된다.

서비스 운영 중에 발생한 상황이라면 정말 치명적인 문제다.

---

## 그럼 정확하게 다루려면 어떻게 해야하는가?

### 그전에, 기존 부동소수점 표현 자료형은 소수점 표현을 어떻게 했는가?

double 타입은 기본적으로 64비트로 표현된다.

1 bits 는 양/음 수를 표현하기 위해 사용된다.
11 bits 는 지수를 표현하기 위해 사용된다.
52 bits 는 유효숫자를 표현한다. (소수는 2진수로 표현한다.)

> 소수를 2진수로 표현하는 점을 주목하라.
>
> 무한하지 않은 자릿수의 2진수의 표현 방식에는 분명히 한계가 있다. (2진수 2비트로 7을 표현할 수 있는가?)

byte, char, int, long 은 고정 소수점 숫자 이므로 소수점 표현이 불가능 했다.



### 소수점 표현에는 BigDecimal, 정수 표현에는 int, long 자료형을 활용해보자.

BigDecimal 은 속도가 다소 느리고, 기본 타입보다 사용이 불편하지만 무한에 가까운 정확성을 보여준다.

> 주의할 점. BigDecimal("10.09");
>
> 위와 같이 생성자에 문자열로 넣어줘야한다는 점을 주의해야한다.

    BigDecimal bigDecimal1 = new BigDecimal("1.009");
    BigDecimal bigDecimal1 = new BigDecimal("1.99");

- BigDecimal("1.009") 가 있다면 먼저 10의 세제곱을 곱해서, 정수로 만든다.
- BigDecimal("1.99") 가 있다면 먼저 10의 제곱을 곱해서, 정수로 만든다.

그 후, n제곱을 위한 n 을 저장하여 unscale(역수) 로 곱하기 위한 내용을 저장한다.

위와 같은 과정을 수행하게되면 소수점 또한 사실상 10진수로 저장하기 때문에

우리가 다루는 수의 범위에서는 모든 수를 표현할 수 있게 되는 것이다.
