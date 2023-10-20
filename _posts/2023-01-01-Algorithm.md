---

title: Algorithm

date: 2023-01-01
categories: [Algorithm]
tags: [Algorithm]
layout: post
toc: true
math: true
mermaid: true

---

# DataStructure

## 괄호 - boj.kr/9012

```kotlin
import java.io.BufferedReader
import java.io.BufferedWriter
import java.io.InputStreamReader
import java.io.OutputStreamWriter
import java.util.*

private val br = BufferedReader(InputStreamReader(System.`in`))
private val bw = BufferedWriter(OutputStreamWriter(System.out))

fun main() {
    val t = br.readLine().toInt()

    repeat(t) {
        val input = br.readLine()
        val stack: Stack<String> = Stack()

        repeat(input.length) {
            val current = input.substring(it, it + 1)

            if (current == "(") {
                stack.push(current)
            } else if (current == ")") {
                if (stack.isNotEmpty()) {
                    if (stack.peek() == "(") {
                        stack.pop()
                    }
                } else {
                    stack.push(current)
                }
            }
        }

        if (stack.isEmpty()) {
            println("YES")
        } else {
            println("NO")
        }
    }
}
```

## 균형잡힌 세상 - boj.kr/4949

```kotlin
import java.io.BufferedReader
import java.io.BufferedWriter
import java.io.InputStreamReader
import java.io.OutputStreamWriter
import java.util.*

private val br = BufferedReader(InputStreamReader(System.`in`))
private val bw = BufferedWriter(OutputStreamWriter(System.out))

fun main() {

    while (true) {
        val input = br.readLine()
        if (input == ".") break

        val stack = Stack<Char>()
        var isBalanced = true

        for (char in input) {
            when (char) {
                '(', '[' -> stack.push(char)
                ')' -> {
                    if (stack.isEmpty() || stack.pop() != '(') {
                        isBalanced = false
                        break
                    }
                }

                ']' -> {
                    if (stack.isEmpty() || stack.pop() != '[') {
                        isBalanced = false
                        break
                    }
                }
            }
        }

        if (isBalanced && stack.isEmpty()) {
            println("yes")
        } else {
            println("no")
        }
    }
}
```

## 덱 - boj.kr/10866

```kotlin
import java.io.BufferedReader
import java.io.BufferedWriter
import java.io.InputStreamReader
import java.io.OutputStreamWriter

private val br = BufferedReader(InputStreamReader(System.`in`))
private val bw = BufferedWriter(OutputStreamWriter(System.out))

fun main() {
    val n = br.readLine().toInt()
    val deque = ArrayDeque<Int>()

    repeat(n) {
        val input = br.readLine().split(" ")
        when (input[0]) {
            "push_back" -> deque.addLast(input[1].toInt())
            "pop_back" -> bw.write(if (deque.isEmpty()) "-1\n" else "${deque.removeLast()}\n")
            "push_front" -> deque.addFirst(input[1].toInt())
            "pop_front" -> bw.write(if (deque.isEmpty()) "-1\n" else "${deque.removeFirst()}\n")
            "front" -> bw.write(if (deque.isEmpty()) "-1\n" else "${deque.first()}\n")
            "back" -> bw.write(if (deque.isEmpty()) "-1\n" else "${deque.last()}\n")
            "size" -> bw.write("${deque.size}\n")
            "empty" -> bw.write(if (deque.isEmpty()) "1\n" else "0\n")
        }
    }
    br.close()
    bw.close()
}
```

## 수찾기 - boj.kr/1920

```kotlin
import java.io.BufferedReader
import java.io.BufferedWriter
import java.io.InputStreamReader
import java.io.OutputStreamWriter

private val br = BufferedReader(InputStreamReader(System.`in`))
private val bw = BufferedWriter(OutputStreamWriter(System.out))


fun main() {
    val n = br.readLine().toInt()
    val array = br.readLine().split(" ").map { it.toInt() }.toHashSet()

    val m = br.readLine().toInt()
    val target = br.readLine().split(" ").map { it.toInt() }

    for (value in target) {
        if (value in array) {
            println(1)
        } else {
            println(0)
        }
    }
}
```

# String

# BruteForce

# Implementation

# Greedy

# Back-Tracking

# Graph

# Dijkstra

# Divide And Conquer

# DP
