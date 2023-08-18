---

title: 코틀린의 1차 진입 장벽 뚫어버리기 (class, object)

date: 2023-07-07
categories: [JPA, MySQL, Index]
tags: [JPA, MySQL, Index]
layout: post
toc: true
math: true
mermaid: true

---

# 헷갈리는 키워드

[모든 내용은 공식문서를 참고하여 작성하였습니다.](https://kotlinlang.org/docs/classes)

## 클래스 관련 키워드

1. class
2. data class
3. sealed class
4. enum class
5. value class
6. inner class

## 객체 관련 키워드

1. companion object
2. object

---

## class

class는 일반적으로 Java에서 사용하던 클래스와 동일하다. 따라서 간단한 사용법만 훑고 넘어가도 충분하다.

### 생성자

```kotlin
class Person(
    val firstName: String, // Getter
    val lastName: String, // Getter
    var age: Int, // Getter/Setter
) { /*...*/ }
```

기본적으로 위와 같이 **소괄호**로 기본 생성자를 만들 수 있다.

Java에서 이를 구현하려면 롬복의 @AllArgsConstructor를 사용하거나 직접 정의해야했지만 코틀린은 그럴 필요없다.

추가적으로 `val` 키워드로 프로퍼티를 정의한다면 Getter를 자동으로 만들어주고

`var` 키워드로 프로퍼티를 정의한다면 Getter/Setter를 자동으로 만들어준다.

또한 class내에서 사용할 커스텀 Getter/Setter를 만들 수도 있는데 다음과 같다.

```kotlin
class Person(
    val firstName: String,
    val lastName: String,
    private var _age: Int, // Private backing field for age
) {

    val age: Int
        // 커스텀 Getter
        get() {
            return if (_age < 0) 0 else _age
        }

    var ageWithCustomSetter: Int
        get() = _age
        set(value) {
            // 커스텀 Setter: age 값을 음수로 설정하지 않도록 제한
            if (value >= 0) {
                _age = value
            } else {
                println("Invalid age value: $value")
            }
        }
}
```

위 코드를 살펴보면 `_age`, `age`가 별개로 있는 것을 볼 수 있다.

`_age`는 생성자로부터 들어온 값을 실제 `Person`의 저장될 `age`에 커스텀 Setter를 통해 넣기 전 잠시 보관하는 `backing field`이다.

### 부 생성자

```kotlin
class Person(
    val firstName: String,
    val lastName: String,
    private var _age: Int, // Private backing field for age
    val hungry: Boolean,
) {

    constructor (firstName: String, lastName: String, hungry: String): this(firstName, lastName, hungry) {
    }

    val age: Int
        // 커스텀 Getter
        get() {
            return if (_age < 0) 0 else _age
        }

    var ageWithCustomSetter: Int
        get() = _age
        set(value) {
            // 커스텀 Setter: age 값을 음수로 설정하지 않도록 제한
            if (value >= 0) {
                _age = value
            } else {
                println("Invalid age value: $value")
            }
        }
}
```

말 그대로 부 생성자 이다. 자바에서 매개변수를 오버로딩하여 생성자를 여러 개 만드는 것과 같다.

### init block

```kotlin
class Person(
    val firstName: String,
    val lastName: String,
    private var _age: Int, // Private backing field for age
    val hungry: Boolean,
) {

    constructor (firstName: String, lastName: String, hungry: String): this(firstName, lastName, 25, hungry)

    init {
        // firstName과 lastName이 비어있을 경우 "Unknown"으로 초기화
        if (firstName.isEmpty()) {
            firstName = "Unknown"
        }
        if (lastName.isEmpty()) {
            lastName = "Unknown"
        }
        // 음수로 초기화된 _age 값을 0으로 수정
        if (_age < 0) {
            _age = 0
        }
    }

    val age: Int
        // 커스텀 Getter
        get() {
            return if (_age < 0) 0 else _age
        }

    var ageWithCustomSetter: Int
        get() = _age
        set(value) {
            // 커스텀 Setter: age 값을 음수로 설정하지 않도록 제한
            if (value >= 0) {
                _age = value
            } else {
                println("Invalid age value: $value")
            }
        }
}
```

class가 생성되는 시점에 그 값을 검증하는 등의 초기화 시점에서 수행할 로직을 init block 내에 담을 수 있다.

---

## data class

```kotlin
data class User(val name: String, val age: Int)
```

위와 같이 data class를 작성할 수 있다. 역시 `val` 키워드가 붙은 프로퍼티는 **Getter**, `var` 키워드가 붙은 프로퍼티는 **Getter/Setter** 모두 만들어준다.

여기서 일반적인 클래스와의 차이점을 꼽자면, **기본 생성자에 포함된 프로퍼티들에 대해** 다음 메서드들을 자동으로 만들어준다는 것이다.

1. equals()
2. hashCode()
3. toString()
4. copy()

크게 특별한 것은 없고 위의 편리함 덕분에 DTO에 주로 사용된다. 만들어진 목적자체가 DTO를 위해 등장한 것이다. (Java의 Record에서 가져온 것으로 예상됨)

---

## sealed class

sealed class는 abstract class 처럼 클래스 간의 계층구조를 나타낼 수 있고 sealed class의 하위 클래스는 object, class, 등등 모든 클래스가 가능하다.

```kotlin
sealed class Result {
    data class Success(val data: String) : Result()
    data class Error(val message: String) : Result()
}

fun handleResult(result: Result) {
    when (result) {
        is Success -> println("Success: ${result.data}")
        is Error -> println("Error: ${result.message}")
    }
}
```

해당 sealed class의 내부 구조를 살펴보면 abstract class로 선언되어있으며 private 생성자를 통해 어떤 클래스에서도 접근하지 못하도록 되어있다.

따라서 이 sealed class의 내용은 외부에서 상속이 불가능하다.

그리고 보통 sealed class는 when절을 활용한 검증 구문에서도 자주 쓰이는데 아래 코드를 살펴보면 이해하기 쉽다.

sealed class를 사용했을 때

```kotlin
sealed class Result {
    data class Success(val data: String) : Result()
    data class Error(val message: String) : Result()
}

fun handleResult(result: Result) {
    when (result) {
        is Success -> println("Success: ${result.data}")
        is Error -> println("Error: ${result.message}")
    }
}
```

sealed class를 사용하지 않았을 때

```kotlin
class Result {
    data class Success(val data: String)
    data class Error(val message: String)
}

fun handleResult(result: Result) {
    when (result) {
        is Success -> println("Success: ${result.data}")
        is Error -> println("Error: ${result.message}")
        // else 절이 추가되어야 하며, 컴파일러는 모든 상태를 처리하는지 확인할 수 없음
        else -> println("Unknown result")
    }
}
```

위 코드의 차이점처럼 sealed class는 클래스 단위로 상태를 관리하기 때문에 sealed class로 계층화 해두었다면 상태에 대한 값을 명시적으로 검증할 수 있게 된다.

---

## enum class

변하지 않는 값 즉, 상수를 관리하는 클래스이다. 각 상수들을 마치 클래스 객체처럼 사용할 수도 있는 등 유용한 기능을 제공한다.

그런데 상수 값을 필요한 곳에서 `val`로 관리하는 것은 어떨까? 이 방법도 가능하지만 굳이 사용하지 않는 이유는

해당 클래스를 생성할 때 마다 val로 선언된 프로퍼티가 새롭게 상수값이 생성되기 때문이다.

이럴바에 한 곳에서 몰아서 관리하여 여러번 생성하지 않아도 사용할 수 있게 하는 것이 훨씬 나을 것이기 때문에 등장한 것이 enum class 이다.

```kotlin
enum class Planet{
    MERCURY, VENUS, EARTH, MARS, JUPITER, SATURN, URANUS, NEPTUNE
}

class Planet{
    companion object{
        val MERCURY = Planet()
        val VENUS = Planet()
        val EARTH = Planet()
        val MARS = Planet()
        val JUPITER = Planet()
        val SATURN = Planet()
        val URANUS = Planet()
        val NEPTUNE = Planet()
    }
}
```

위 코드에 해당하는 enum class와 아래에 해당하는 class는 동작이 거의 유사하다.

즉 아래와 같이 생성자를 통해서 프로퍼티가 포함된 객체로 사용할 수 있는 것이다.

```kotlin
enum class Planet(mass:Double, radius:Double){
    MERCURY(3.303e+23, 2.4397e6),
    VENUS (4.869e+24, 6.0518e6),
    EARTH (5.976e+24, 6.37814e6),
    MARS (6.421e+23, 3.3972e6),
    JUPITER (1.9e+27, 7.1492e7),
    SATURN (5.688e+26, 6.0268e7),
    URANUS (8.686e+25, 2.5559e7),
    NEPTUNE (1.024e+26, 2.4746e7)
}
```

```kotlin
enum class Planet(val mass:Double, val radius:Double){
    MERCURY(3.303e+23, 2.4397e6),
    VENUS (4.869e+24, 6.0518e6),
    EARTH (5.976e+24, 6.37814e6),
    MARS (6.421e+23, 3.3972e6),
    JUPITER (1.9e+27, 7.1492e7),
    SATURN (5.688e+26, 6.0268e7),
    URANUS (8.686e+25, 2.5559e7),
    NEPTUNE (1.024e+26, 2.4746e7);

    fun diameter(): Double { return radius * 2}
}

fun main(){
    val p1 = Planet.MERCURY
    val p2 = Planet.URANUS

    println("$p1 mass : ${p1.mass}, $p1 diameter : ${p1.diameter()}")
    println("$p2 mass : ${p2.mass}, $p2 diameter : ${p2.diameter()}")
}
```

추가적으로, enum class는 기본 생성자가 private으로 되어있어 상속이 불가능하고 상속받는 것 조차 불가능하다. 하지만 인터페이스를 구현하는 행위는 가능하다.

enum class는 class라는 이름에 걸맞게 프로퍼티, 생성자, 메서드를 가질 수 있다.

또한 values(), valueOf(), ordinal, name, toString(), compareTo() 등 자동으로 생성해주는 편의 메서드가 존재한다.

---

## value class

[참고한 블로그](https://devs0n.tistory.com/83)

value class를 사용하기 전 `엔티티 설계` 코드

```kotlin
class User(
    name: String,
    email: String
) {

    @Id
    @Column(name = "id")
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    val id: Long = 0

    @Column(name = "name")
    val name: String = name

    @Column(name = "email")
    val email: String = email

    init {
        validateUserName()
        validateUserEmail()
    }

    private fun validateUserName() {
        if (this.name.isBlank()) {
            throw IllegalArgumentException("invalid user name: ${this.name}")
        }
    }

    private fun validateUserEmail() {
        val usernameAndDomain = this.email.split('@')

        if (usernameAndDomain.size != 2) {
            throw IllegalArgumentException("invalid user email: ${this.email}")
        }
    }

    override fun toString(): String {
        return "User(id=$id, name='$name', email='$email')"
    }
}
```

위 코드를 조금 더 깊게 살펴보자면 다음과 같은 문제점이 발생할 수 도 있다.

1. 이메일을 다루는 타입이 String으로 너무 범용적이다. 따라서 이메일이라는 형식에 걸맞게 validate를 수행하지 않는다면 올바른 데이터를 사용할 수 없게 될 수 있다.
2. 생성자 혹은 도메인 로직 내 동일한 타입의 인자가 여러 개 있는 경우 인자를 넣는 순서에 주의해야한다.

위 문제점을 어느정도 방지하기 위해 VO를 적용하고자 한다. JPA에서는 `@Embeddable, @Embedded`를 통해 VO를 사용할 수 있도록 제공한다.

코틀린의 `data class`와 `@Embeddable, @Embedded`를 통해 VO를 갖는 엔티티를 설계하는 코드를 살펴보면 아래와 같다.

```kotlin
@Entity
@Table(name = "plans")
class User(
    name: UserName,
    email: UserEmail,
) {

    @Id
    @Column(name = "id")
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    val id: Long = 0

    @Embedded
    val name: UserName = name

    @Embedded
    val email: UserEmail = email

    override fun toString(): String {
        return "User(id=$id, name='$name', email='$email')"
    }
}

@Embeddable
data class UserName(
    @Column(name = "name")
    val value: String
) {

    init {
        if (this.value.isBlank()) {
            throw IllegalArgumentException("invalid user name: ${this.value}")
        }
    }
}

@Embeddable
data class UserEmail(
    @Column(name = "email")
    val value: String
) {

    init {
        val usernameAndDomain = this.value.split('@')

        if (usernameAndDomain.size != 2) {
            throw IllegalArgumentException("invalid user email: ${this.value}")
        }
    }
}
```

위 코드로 적용한다면 아까 고민했던 두 가지의 내용이 어느정도 해결될 것이다. 하지만 여전히 불편한 점은 남아있다.

일일히 해당 VO마다 애노테이션을 추가해가며 생산성을 저하시킨다는 것이다.

그리고 **가장 큰 문제점은 값을 래핑하는데 있어 JPA에 대한 의존성이 너무 크다**는 것이다.

그러면 이 마저도 보완할 방법은 뭐가 있을까? 하는게 value class 이다.

```kotlin
@Entity
@Table(name = "plans")
class User(
    name: UserName,
    email: UserEmail
) {

    @Id
    @Column(name = "id")
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    val id: Long = 0

    @Column(name = "name")
    val name: UserName = name

    @Column(name = "email")
    val email: UserEmail = email

    override fun toString(): String {
        return "User(id=$id, name='$name', email='$email')"
    }
}

@JvmInline
value class UserName(
    val value: String
) {

    init {
        if (this.value.isBlank()) {
            throw IllegalArgumentException("invalid user name: ${this.value}")
        }
    }
}



@JvmInline
value class UserEmail(
    val value: String
) {

    init {
        val usernameAndDomain = this.value.split('@')

        if (usernameAndDomain.size != 2) {
            throw IllegalArgumentException("invalid user email: ${this.value}")
        }
    }
}
```

위 코드가 `value class`를 활용한 코드인데, User 엔티티 필드에 특별한 애노테이션이 없어도 잘 동작하게 되어있다.

위 코드를 컴파일 하고 보면 UserName, UserEmail type이 따로 존재하는 것이 아니라

```java
private final String name;

private final String email;
```

와 같이 컴파일 되어있는 것을 확인할 수 있다. 외부 프레임워크와 독립적이고(JPA) 의도대로 값을 래핑하여 코드의 독립성을 갖추기 위해 VO를 적용하고 싶다면

`value class`를 사용하는게 꽤나 좋은 선택이 될 수 있을 것이라고 본다.

---

## nested || inner class

### Java의 inner class

우선 Java와의 차이점을 보자면 아래 코드는 Java에서 `inner class`로 취급한다.

```java
class Outer {

    private String outer = "Outer";

    class InnerClass {

        public InnerClass() {
            System.out.println(outer);
        }
    }
}
```

따라서 inner class에서 Outer클래스의 `인스턴스 변수에 접근하는 것이 가능`하다.

### Java의 nested class

```java
class Outer {

    private String outer = "Outer";

    static class InnerClass {

        public InnerClass() {
            System.out.println(outer);
        }
    }
}

```

자바는 위와 같이 `static`키워드를 활용하여 nested class를 만든다.

따라서 묵시적으로 참조하고 있던 Outer class 에 대한 메타데이터가 모두 사라지기 때문에 InnerClass의 생성자 로직을 오류를 발생시킨다.

### Kotlin의 nested class

```kotlin
class Outer {

    private val outer = "Outer"

    class InnerClass {

        init {
            print(outer)
        }
    }
}

```

코틀린은 위와 같은 코드를 `nested class`라고 명칭한다.

Java에 존재하는 개념과는 정반대로 동작하며 처음부터 `묵시적으로 Outer를 들고 있지 않도록` 하는 것이다.

따라서 위 코드를 실행시키면 에러가 발생한다.

### Kotlin의 inner class

```kotlin
class Outer {

    private val outer = "Outer"

    inner class InnerClass {

        init {
            print(outer)
        }
    }
}
```

위 코드는 Kotlin의 `inner class`이다. 명시적으로 inner class임을 알려야 Outer에 대한 참조값을 가지게 된다.


---

간단하게 요약하자면 다음과 같다.

- Java - 클래스 내의 클래스 = inner 클래스로, outer 클래스에 대한 참조 값을 묵시적으로 가지고 있다.

- Java - 클래스 내의 static 클래스 = nested 클래스로, outer 클래스에 대한 참조 값을 가지고 있지 않다.

- Kotlin - 클래스 내의 클래스 = nested 클래스로, outer 클래스에 대한 참조 값을 가지고 있지 않다.

- Kotlin - 클래스 내의 inner 클래스 = inner 클래스로, outer 클래스에 대한 참조 값을 가지고 있다.


**공통적으로** inner class는 outer클래스가 생성되어야 그 안에 있는 class가 생성된다.


- nested class는 외부 클래스와 완전히 독립적으로 존재하기 때문에 외부 클래스의 인스턴스와 상관없이 객체를 생성할 수 있다.

- nested class를 Bean으로 등록하려면, 해당 클래스를 @Component등의 Spring Framework 컴포넌트 어노테이션 중 하나로 등록하면 된다.

```kotlin
@Component
class OuterClass {
    @Component
    class NestedClass {
        // ...
    }
}
```

- inner class는 외부 클래스와 종속적이기 때문에 해당 외부 클래스가 생성되어야지 생성될 수 있는 클래스이다.
- 따라서 inner class를 IoC컨테이너에 Bean으로 등록하려면 수동으로 Bean을 등록해주어야한다.

수동 등록 코드
```kotlin
@Component
class OuterClass {
    @Component
    inner class InnerClass {
        // ...
    }
}

// Spring 컨텍스트에 수동으로 등록하는 예시:
val outerInstance = OuterClass()
val innerInstance = outerInstance.InnerClass()
```

---


## object

코틀린에서는 `static`키워드가 존재하지 않는다. `object` 키워드로 대체하기로 했기 때문이다.

`object` 키워드로 싱글턴 패턴을 기본적으로 지원해주는데 아래 코드를 컴파일하고 까보면 무슨 의미인지 이해하기 쉽다.

### object로 만들어진 객체는 싱글턴 패턴으로 구성되어있다.

```kotlin
object CustomClassByObject {
    var customValue: Int = 20

    fun printCustomValue() {
        println("customValue is : $customValue")
    }
}
```

```java
public final class CustomClassByObject {
   private static int customValue;
   @NotNull
   public static final CustomClassByObject INSTANCE;

   public final int getCustomValue() {
      return customValue;
   }

   public final void setCustomValue(int var1) {
      customValue = var1;
   }

   public final void printCustomValue() {
      System.out.println("customValue is : " + customValue);
   }

   private CustomClassByObject() {
   }

   static {
      CustomClassByObject var0 = new CustomClassByObject();
      INSTANCE = var0;
      customValue = 20;
   }
}
```

- 클래스의 생성자를 private화
- 클래스의 인스턴스를 담은 변수를 static final로 선언
- 인스턴스 생성과 변수의 초기화는 static 초기화 블록에서 수행하여 동시성 문제 해결 (static 초기화 블럭은 클래스가 메모리에 로딩되는 시점에 실행되어 마치 `synchronized` 키워드가 있는 것 처럼 동작한다.)

### object로 익명 객체를 만들 수 있다.

```kotlin
interface TestInterface {
    val a: Int
    fun calculate(): Int
}

fun main(args: Array<String>) {
    val someObject = object: TestInterface {
        override val a: Int = 5

        override fun calculate(): Int {
            return a * a + 12
        }
    }

    println(someObject.calculate())
}
```

위 처럼 익명객체를 만들기위해 활용 또한 가능하다.

---

## companion object

위에서 알아본 object는 static으로 동작한다고 했다.

그런데 이 object에 존재하는 변수/메서드에 접근하려면 다소 깔끔하지 않은 코드를 남발해야한다.

```kotlin
class ClassForTest {
    // for static field
    object SomeObjectClass {
        var someValue: Int = 24

        fun printSomeValue() {
            println("SomeValue is $someValue")
        }
    }
}

fun main(args: Array<String>) {
    ClassForTest.SomeObjectClass.printSomeValue() // 지저분...
}
```

이 static 필드에 접근하는 방법을 좀 더 간소화 하고자 등장한 키워드가 `companion object`이다.

```kotlin
class ClassForTest {
    companion object {
        val someValueInCompanionObject = 24

        fun printSomeValue() {
            println(someValueInCompanionObject)
        }
    }
}

fun main(args: Array<String>) {
    ClassForTest.printSomeValue()
}
```

해당 클래스의 인스턴스 없이도 접근할 수 있는 멤버를 정의하는 것으로 Java의 static 멤버와 유사한 역할을 한다. 그리고 클래스 내에 하나의 companion object만 정의할 수 있다.

**매번 object 클래스에 접근하여 해당 메서드를 호출하는 것**이 아니라 **해당 메서드나 변수에 직접 접근**이 가능하게 되었다.

---

### object vs companion object 구성의 차이점

```kotlin
class ClassForTest {
    companion object {
        val someValueInCompanionObject = 24

        fun printSomeValue() {
            println(someValueInCompanionObject)
        }
    }

    object ObjectForTest {
        val someValueInObject = 24

        fun printSomeValue() {
            println(someValueInObject)
        }
    }
}
```

위와 같은 `object`와 `companion object`가 있다고 했을 때 그 차이점은 뭔지 들어가보자

### object 키워드 디컴파일링

```kotlin
 public static final class ObjectForTest {
      private static final int someValueInObject;

      @NotNull
      public static final ClassForTest.ObjectForTest INSTANCE;

      public final int getSomeValueInObject() {
         return someValueInObject;
      }

      public final void printSomeValue() {
         int var1 = someValueInObject;
         System.out.println(var1);
      }

      private ObjectForTest() {
      }

      static {
         ClassForTest.ObjectForTest var0 = new ClassForTest.ObjectForTest();
         INSTANCE = var0;
         someValueInObject = 24;
      }
   }
```

### companion object 디컴파일링

```kotlin
   private static final int someValueInCompanionObject = 24;

   @NotNull
   public static final ClassForTest.Companion Companion
		= new ClassForTest.Companion((DefaultConstructorMarker)null);

   public static final class Companion {
      public final int getSomeValueInCompanionObject() {
         return ClassForTest.someValueInCompanionObject;
      }

      public final void printSomeValue() {
         int var1 = ((ClassForTest.Companion)this).getSomeValueInCompanionObject();
         System.out.println(var1);
      }

      private Companion() {
      }

      // $FF: synthetic method
      public Companion(DefaultConstructorMarker $constructor_marker) {
         this();
      }
   }
```

## object vs companion object 요약

- `companion object`는 해당 클래스 외부(부모 클래스 내부)에 선언되고 초기화된다.

- `object`는 해당 클래스 내부에 static 블록을 통해 초기화된다.


object는 단일 인스턴스를 나타내는데 사용되며, 클래스의 내용과 함께 정의된다.

companion object는 클래스 내에서 정적 멤버와 유사한 역할을 하며, 해당 클래스와 밀접한 관련이 있는 기능을 구현하는 데 사용된다.

만약 특정 클래스와 강하게 연결된 정적 멤버를 정의하려면 companion object를 사용하고, 클래스와 무관한 독립적인 싱글턴 객체를 만들고자 한다면 object를 사용하는 것이 좋다.