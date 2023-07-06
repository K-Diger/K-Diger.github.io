---

title: ArrayList vs MutableList in Kotlin

date: 2023-06-09
categories: [Kotlin]
tags: [Kotlin]
layout: post
toc: true
math: true
mermaid: true

---

# 참고자료

[Baeldung Docs](https://www.baeldung.com/kotlin/arraylist-vs-mutablelistof)


---

# ArrayList

```kotlin
val arrayList = ArrayList<String>()
arrayList.add("Kotlin")
arrayList.add("Java")
```

---

# MutableList

```kotlin
val mutableList = mutableListOf<String>()
mutableList.add("Kotlin")
mutableList.add("Java")



/**
 * Returns an empty new [MutableList].
 * @sample samples.collections.Collections.Lists.emptyMutableList
 */
@SinceKotlin("1.1")
@kotlin.internal.InlineOnly
public inline fun <T> mutableListOf(): MutableList<T> = ArrayList()
```

---

# ArrayList vs MutableList in kotlin

차이는 없다. 어떤 자료를 살펴봐도 변함 없는것은 MutableList를 만들어도 결국에는 ArrayList를 반환하는 것이다.

## 그러면 왜 MutableList를 사용할까?

`첫 번째 이유`는, 구체 클래스를 명시하는 것 보단 MutableList키워드를 통해 구현체를 알아서 땡겨 쓰도록 하는 것

`두 번째 이유`는, 만약에 현재 존재하는 ArrayList가 취약점이 발견되어서 앞으로 그 구문을 쓰지않아야한다면?

ArrayList라고 직접 명시했다면 그 코드를 전부 고쳐야한다. 그렇지만 MutableList를 사용했다면 MutableList가 가리키는 구현체만 바꿔 끼워주기만하면된다.

구체 클래스에 의존하지 않고 인터페이스에 의존하는 낮은 결합도를 충족시킬 수 있는 장점이 있기 때문에 MutableList를 사용한다.


![Kotlin 공식문서](https://kotlinlang.org/docs/images/collections-diagram.png)

코틀린 공식문서에서는 기존 Java에서 사용하던 Collection들을 Mutable이라는 키워드를 붙여서 제공하는 것으로 나타내고 있는 점도 볼 수 있다.

그만큼 권장 한다는 스펙이라고 보면 될 것 같다.
