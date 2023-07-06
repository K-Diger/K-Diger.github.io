---

title: Item 58. 전통적인 for 문 보다는 for-each 문을 사용하라
author: 김도현
date: 2022-09-02
categories: [Effective-Java]
tags: [For-Loop, For-Each]
layout: post
toc: true
math: true
mermaid: true

---

# 컬렉션/배열 순회하기

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

# 무조건 가능한건 아니다.

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

<br>

> 위 세가지 상황에서는 일반적인 for 문을 사용하되 다른 상황이라면 for-each 를 권장한다. 성능 차이도 없다.
