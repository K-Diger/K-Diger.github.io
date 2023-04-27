---

title: 자바에서 코틀린으로 - 1 ()
author: 김도현
date: 2023-03-22
categories: [Kotlin]
tags: [Kotlin]
math: true
mermaid: true

---

[Kotlin 공식문서 - PlayGround](https://play.kotlinlang.org/byExample/01_introduction/01_Hello%20world)

---

# Hello World!

```kotlin
package org.kotlinlang.play

fun main() {
    println("Hello, World!")
}
```

```kotlin
fun printMessage(message: String): Unit {
    println(message)
}

fun printMessageWithPrefix(message: String, prefix: String = "Info") {
    println("[$prefix] $message")
}

fun sum(x: Int, y: Int): Int {
    return x + y
}

fun multiply(x: Int, y: Int) = x * y

fun main() {
    printMessage("Hello")
    printMessageWithPrefix("Hello", "Log")
    printMessageWithPrefix("Hello")
    printMessageWithPrefix(prefix = "Log", message = "Hello")
    println(sum(1, 2))
    println(multiply(2, 4))
}
```

### 출력 결과

```text
Hello
[Log] Hello
[Info] Hello
[Log] Hello
3
8
```

# Infix Function

### Infix Function이란?

```kotlin
infix fun dispatcher.함수명(receiver): 리턴타입 {}

val monday: Pair<String, String> = "Monday" to "월요일"

infix fun String.add(other: String): String {
    return this + other
}
```

위와 같은 형식으로 만들 수 있다.

- monday Pair Type에서는 "Monday" 가 dispatcher 이고 "월요일" 이 receiver 인 것이다.

- add 메서드에서 this는 dispatcher를 가리킨다.

```kotlin
fun main() {

    infix fun Int.times(str: String) = str.repeat(this)
    println(2 times "Bye ")

    val pair = "Ferrari" to "Katrina"
    println(pair)

    infix fun String.onto(other: String) = Pair(this, other)
    val myPair = "McLaren" onto "Lucas"
    println(myPair)

    val sophia = Person("Sophia")
    val claudia = Person("Claudia")
}

class Person(val name: String) {
    val likedPeople = mutableListOf<Person>()
    infix fun likes(other: Person) {
        likedPeople.add(other)
    }
}
```

# Operator Function

다음과 같은 예약어를 통해 특정 클래스 내의 메서드나 로컬 메서드를 통해 커스텀한 연산자를 사용할 수 있다.

즉, "**연산자 오버로딩**"을 하여 사용할 수 있는 것이다.

| 표현      | 네이밍              |
|---------|------------------|
| a + b   | a.plus(b)        |
| a - b   | a.minus(b)       |
| a * b   | a.times(b)       |
| a / b   | a.div(b)         |
| a % b   | a.rem(b)         |
| a += b  | a.plusAssign(b)  |
| a -= b  | a.minusAssign(b) |
| a *= b  | a.timesAssign(b) |
| a /= b  | a.divAssign(b)   |
| a %= b  | a.remAssign(b)   |

```kotlin
operator fun Int.times(str: String) = str.repeat(this)
println(2 * "Bye ")

operator fun String.get(range: IntRange) = substring(range)
val str = "Always forgive your enemies; nothing annoys them so much."
println(str[0..14])
```

### 출력 결과

```text
Bye Bye
Always forgive
```

```kotlin
data class Price(val value: Int) {
    operator fun plus(b: Price): Price {
        return Price(value + b.value)
    }
}

val a: Price = Price(10)
val b: Price = Price(50)
val result: Price = a + b

print(result)
```

## 출력 결과

```text
60
```

# vararg Function

',' 로 구분하여 여러개의 파라미터를 전달하여 메서드를 사용할 수 있다.

```kotlin
fun printAll(vararg messages: String) {
    for (m in messages) println(m)
}

printAll("Hello", "Hallo", "Salut", "Hola", "你好")

println("---------------------------------------------------------------")

fun printAllWithPrefix(vararg messages: String, prefix: String) {
    for (m in messages) println(prefix + m)
}

printAllWithPrefix(
    "Hello", "Hallo", "Salut", "Hola", "你好",
    prefix = "Greeting: "
)

println("---------------------------------------------------------------")

fun log(vararg entries: String) {
    printAll(*entries)
}

log("Hello", "Hallo", "Salut", "Hola", "你好")
```

### 출력 결과

```text
Hello
Hallo
Salut
Hola
你好
---------------------------------------------------------------
Greeting: Hello
Greeting: Hallo
Greeting: Salut
Greeting: Hola
Greeting: 你好
---------------------------------------------------------------
Hello
Hallo
Salut
Hola
你好
```

# 변수

```kotlin
var a: String = "initial" // 변경 가능한 타입 선언과 초기화 구문
println(a)
val b: Int = 1 // 변경 불가능한 타입과 초기화 구문
val c = 3 // 변경 불가능한 구문
```

# Null Safety

```kotlin
var neverNull: String = "기본값은 널이 들어올 수 없어용"

neverNull = null

var nullable: String? = "? 를 넣으면 널이 들어와도 됩니다"

nullable = null

var inferredNonNull = "이런식으로 한다면 기본 값이 들어가요 (널 금지)"

inferredNonNull = null

fun strLength(notNull: String): Int {
    return notNull.length
}

strLength(neverNull)
strLength(nullable)
```

# Class

코틀린은 파라미터가 없는 기본 생성자를 제공한다!

만약 속성을 정의하면 그 속성을 모두 포함한 생성자를 기본으로 제공한다.

```kotlin
class Customer

class Contact(val id: Int, var email: String)

fun main() {

    val customer = Customer()

    val contact = Contact(1, "mary@gmail.com")

    println(contact.id)
    contact.email = "jane@gmail.com"
}
```

# Generics

```kotlin
class MutableStack<E>(vararg items: E) {

  private val elements = items.toMutableList()

  fun push(element: E) = elements.add(element)

  fun peek(): E = elements.last()

  fun pop(): E = elements.removeAt(elements.size - 1)

  fun isEmpty() = elements.isEmpty()

  fun size() = elements.size

  override fun toString() = "MutableStack(${elements.joinToString()})"
}
```

### 실행 결과

```text
MutableStack(0.62, 3.14, 2.7, 9.87)
peek(): 9.87
MutableStack(0.62, 3.14, 2.7, 9.87)
pop(): 9.87
MutableStack(0.62, 3.14, 2.7)
pop(): 2.7
MutableStack(0.62, 3.14)
pop(): 3.14
MutableStack(0.62)
pop(): 0.62
MutableStack()
```

# 상속

```kotlin
open class Dog {
    open fun sayHello() {
        println("wow wow!")
    }
}

class Yorkshire : Dog() {
    override fun sayHello() {
        println("wif wif!")
    }
}

fun main() {
    val dog: Dog = Yorkshire()
    dog.sayHello()
}
```

### 실행 결과

```text
wif wif!
```

확장에 매개변수를 활용할 수도 있다.

```kotlin
open class Tiger(val origin: String) {
    fun sayHello() {
        println("A tiger from $origin says: grrhhh!")
    }
}

class SiberianTiger : Tiger("Siberia")

fun main() {
    val tiger: Tiger = SiberianTiger()
    tiger.sayHello()
}
```

### 실행 결과

```text
A tiger from Siberia says: grrhhh!
```

서브 클래스의 매개변수와, 슈퍼 클래스의 매개변수를 모두 활용하는 방법도 존재한다.

```kotlin

    fun sayHello() {
        println("$name, the lion from $origin says: graoh!")
    }
}

class Asiatic(name: String) : Lion(name = name, origin = "India")

fun main() {
    val lion: Lion = Asiatic("Rufo")
    lion.sayHello()
}
```

### 실행 결과

```text
Rufo, the lion from India says: graoh!
```

---

# When

```kotlin
fun main() {
    cases("Hello")
    cases(1)
    cases(0L)
    cases(MyClass())
    cases("hello")
}

fun cases(obj: Any) {
    when (obj) {
        1 -> println("One")
        "Hello" -> println("Greeting")
        is Long -> println("Long")
        !is String -> println("Not a string")
        else -> println("Unknown")
    }
}

class MyClass
```

# 반복문

```kotlin
val cakes = listOf("carrot", "cheese", "chocolate")

for (cake in cakes) {
    println("Yummy, it's a $cake cake!")
}
```

#
