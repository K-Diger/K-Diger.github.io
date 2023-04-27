---

title: 자바에서 코틀린으로 - 1 ()
author: 김도현
date: 2023-03-22
categories: [Kotlin]
tags: [Kotlin]
math: true
mermaid: true

---

[Kotlin 공식문서 - Basics](https://kotlinlang.org/docs)

---

# 패키지/임포트 선언

```kotlin
package my.demo

import kotlin.text.*



```

# 기본 메서드

```kotlin
fun main() {
    println("Hello world!")
}

fun main(args: Array<String>) {
    println(args.contentToString())
}

fun main(a: Int, b: Int): Int {
    return a + b
}
```

```kotlin
fun printSum(a: Int, b: Int): Unit {
    println("sum of $a and $b is ${a + b}")
}
```

### 출력 결과

sum of -1 and 8 is 7

```kotlin
fun printSum(a: Int, b: Int) {
    println("sum of $a and $b is ${a + b}")
}
```

### 출력 결과

sum of -1 and 8 is 7

Unit 키워드는 Java의 Void와 같다. 따라서 생략 가능한 타입 힌트이다.

# 변수 선언

## val 키워드

val 키워드는 읽기전용 변수를 의미한다. java의 final키워드가 포함된 키워드로 보면 된다.

```kotlin
val a: Int = 1 // 타입과 값을 즉시 초기화
val b = 2 // 값을 초기화한다. 타입은 자동할당된다.
val c: Int // 타입을 할당한다.
c = 3 // 할당된 타입을 가진 변수에 값을 초기화한다.
```

## var 키워드

var 키워드는 재할당이 가능한 변수를 의미한다.

```kotlin
var x = 5
x += 1
```

# Class

```kotlin
class Rectangle(var height: Double, var length: Double) {
    var perimeter = (height + length) * 2
}
```

# String Template

```kotlin
var a = 1
// simple name in template:
val s1 = "a is $a"

a = 2
// arbitrary expression in template:
val s2 = "${s1.replace("is", "was")}, but now is $a"
```

# 조건식

```kotlin
fun maxOf(a: Int, b: Int): Int {
    if (a > b) {
        return a
    } else {
        return b
    }
}
```

# 반복문

```kotlin
val items = listOf("apple", "banana", "kiwifruit")
for (item in items) {
    println(item)
}
```

### 출력 결과

```text
apple
banana
kiwifruit
```

```kotlin
val items = listOf("apple", "banana", "kiwifruit")
for (index in items.indices) {
    println("item at $index is ${items[index]}")
}
```

### 출력 결과

item at 0 is apple
item at 1 is banana
item at 2 is kiwifruit

# 반복문 - While

```kotlin
val items = listOf("apple", "banana", "kiwifruit")
var index = 0
while (index < items.size) {
    println("item at $index is ${items[index]}")
    index++
}
```

### 출력 결과

```text
item at 0 is apple
item at 1 is banana
item at 2 is kiwifruit
```

<br>

```kotlin
fun describe(obj: Any): String {
    when (obj) {
        1 -> "One"
        "Hello" -> "Greeting"
        is Long -> "Long"
        !is String -> "Not a string"
        else -> "Unknown"
    }
}

fun describe(obj: Any): String =
    when (obj) {
        1 -> "One"
        "Hello" -> "Greeting"
        is Long -> "Long"
        !is String -> "Not a string"
        else -> "Unknown"
    }

```

### 출력 결과

```text
One
Greeting
Long
Not a string
Unknown
```

# Range

```kotlin
val x = 10
val y = 9
if (x in 1..y+1) {
    println("fits in range")
}
```

### 출력 결과

```text
fits in range
```

<br>

```text
val list = listOf("a", "b", "c")

if (-1 !in 0..list.lastIndex) {
    println("-1 is out of range")
}
if (list.size !in list.indices) {
    println("list size is out of valid list indices range, too")
}
```

### 출력 결과

```text
for (x in 1..5) {
    print(x)
}
```

```text
12345
```
<br>

```kotlin
for (x in 1..10 step 2) {
    print(x)
}
println()
for (x in 9 downTo 0 step 3) {
    print(x)
}
```

### 출력 결과

```text
13579
9630
```

# Collection

```kotlin
for (item in items) {
    println(item)
}
```

```kotlin
when {
    "orange" in items -> println("juicy")
    "apple" in items -> println("apple is fine too")
}
```

```kotlin
val fruits = listOf("banana", "avocado", "apple", "kiwifruit")
fruits
    .filter { it.startsWith("a") }
    .sortedBy { it }
    .map { it.uppercase() }
    .forEach { println(it) }
```

# Nullable

Null이 가능한 타입에는 해당 타입 뒤에 '?' 키워드로 명시한다.


```kotlin
fun parseInt(str: String): Int? {
    // ...
}
```

```kotlin
fun printProduct(arg1: String, arg2: String) {
    val x = parseInt(arg1)
    val y = parseInt(arg2)

    // Using `x * y` yields error because they may hold nulls.
    if (x != null && y != null) {
        // x and y are automatically cast to non-nullable after null check
        println(x * y)
    }
    else {
        println("'$arg1' or '$arg2' is not a number")
    }
}
```
