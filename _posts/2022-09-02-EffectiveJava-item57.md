---

title: Item 57. 지역변수의 범위를 최소화하라

date: 2022-09-02
categories: [Effective-Java]
tags: [LocalVariable]
layout: post
toc: true
math: true
mermaid: true

---

# 지역변수의 범위를 줄이는 가장 가장 강력한 기법

> 해당 지역변수가 가장 처음 쓰일 때 선언하기


> 모든 지역변수는 선언과 동시에 초기화하기.
>
> + 만약, 초기화에 필요한 정보가 충분하지 않다면 충분해 질 때까지 선언을 미루기


# 컬렉션을 순회할 때 권장사항


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
