---

title: Java 8 - Stream/Lambda
layout: post
description: Stream/Lambda 를 사용해보자
categories: [Java]
tags: [Stream, Lambda]
redirect_from:
- /2022/08/09/
---

# Stream 이란 무엇인가?

함수형 프로그래밍을 지원하기 위한 문법이다.

### 함수형 프로그래밍 vs 객체지향 프로그래밍

**객체지향 프로그래밍**

매개변수에 객체를 넣고 메서드를 사용할 수 있다.


**함수형 프로그래밍**

매개변수에 함수를 보내고, 변수에 함수를 지정하고, 함수를 리턴 받을 수 있다.

유명한 책인 클린 코드(Clean Code)의 저자 Robert C.Martin은 함수형 프로그래밍을 대입문이 없는 프로그래밍이라고 정의하였다.

출처: https://mangkyu.tistory.com/111


컬렉션, 배열 에 저장된 원소들을 하나씩 참조하며 반복적인 작업을 처리하게 해준다.

즉, for + if 문 없이 배열과 컬렉션을 다룰 수 있게 해주는 문법이다.

주로 Lambda 문법과 결합하여 사용한다.

---

# Stream을 어떻게 사용하는가?

Stream은 **초기연산**, **중간연산**, **종단연산으로 구분**된다.

Stream에서 지원하는 문법은 굉장히 많다. 다 외워서 사용할 것이 아니라

필요할때마다 찾아쓰는 방식으로 익히는게 좋을 것이다.


## 일반적인 리스트의 합을 구하는 방식

    private static int normalSum(List<Integer> numbers) {
        int sum = 0;
        for (int number : numbers) {
            sum += number;
        }
        return sum;
    }

## Stream 을 이용한 리스트의 합을 구하는 방식

    private static int functionalProgrammingSum(List<Integer> numbers) {

        int sum = numbers.stream()
                .reduce(0, (number1, number2) -> number1 + number2);

        return sum;
    }

<br>

> .reduce(초기값, 연산) 의 형태로 사용된다.

초기값부터 시작해서, 각 원소를 차례로 순회하며 연산을 수행하고, 이전 연산의 결과를 다음 초기값으로 넘기면서 결과를 누적한다.

즉, 위의 예시는 0부터 시작하여, number1 + number2 라는 연산을 함수로써 매개변수로 사용하여 순차적으로 더해가는 것이다.

---

# Stream 의 중간 연산 - Sort, Distinct, Filter, Map

## 스트림 중간 연산 (sort)

    private static void middleOperationSorted(List<Integer> numbers) {
        numbers.stream()
        .sorted()
        .forEach(e -> System.out.println(e));
    }

## 스트림 중간 연산 (distinct)

    // 정렬은 하지 않고 중복된 요소를 차례로 제거
    private static void middleOperationDistinct(List<Integer> numbers) {
        numbers.stream()
        .distinct()
        .forEach(e -> System.out.println(e));
    }

## 스트림 중간 연산 (distinct)
    // 중복제거 및 정렬
    private static void middleOperationDistinctSorted(List<Integer> numbers) {
        numbers.stream()
        .distinct().sorted()
        .forEach(e -> System.out.println(e));
    }

## 스트림 중간 연산 (Filter)
    // if문을 대체할 수 있음, 홀수만 출력
    private static void middleOperationFilter(List<Integer> numbers) {
        return numbers.stream()
                .filter(number -> number % 2 == 1)
                .reduce(0, (number1, number2) -> number1 + number2);
    }

## 스트림 중간 연산 (Map)
    // 스트림으로 순회중인 요소를 특정 조건에 맞게 수정한다.
    // 스트림은 값을 직접 수정하지 않는다. 그저 그렇게 보이게만 해주는 것

    // 아래 예시는 현재
    private static void middleOperationMap(List<Integer> numbers) {
        numbers.stream()
                .map(e -> e * e)
                .forEach(e-> System.out.println(e));

        numbers.stream()
                .forEach(e -> System.out.println(e));
    }

<br>

> middleOperationMap 메서드 출력 결과

16
36
64
169
9
225
64
36
441

4
6
8
13
3
15
8
6
21

## 스트림 중간 연산 (Map)

    // 문자열 스트림 모두 소문자로
    private static void middleOperationMapToLowerCase() {
        List.of("Apple", "Ant", "Bat")
                .stream()
                .map(s -> s.toLowerCase())
                .forEach(s -> System.out.println(s));
    }

    // 문자열 스트림 각 원소의 문자열 길이 얻기
    private static void middleOperationMapLenghthOfString() {
        List.of("Apple", "Ant", "Bat")
                .stream()
                .map(s -> s.length())
                .forEach(s -> System.out.println(s));
    }

<br>

> **중간연산**(Sort, Distinct, Filter, Map) 은 **갯수 상관없이 붙여 쓸 수 있다.**
>
> 하지만 **종단연산은 하나임** -> **하나의 값으로 축소하기 때문**

---

## 스트림 종단 연산 (Max)

    // Stream 으로 순회하고 있는 대상에서 가장 큰 값을 리턴
    private static Integer endMax(List<Integer> numbers) {
        return numbers.stream()
                .max((n1, n2) -> Integer.compare(n1, n2));
    }

## 스트림 종단 연산 (Min)

    // Stream 으로 순회하고 있는 대상에서 가장 작은 값을 리턴
    private static Integer endMax(List<Integer> numbers) {
        return numbers.stream()
                .min((n1, n2) -> Integer.compare(n1, n2));
    }


## 스트림 종단 연산 (collect)

    // Stream 으로 순회하고 있는 대상을 필터링 한 후, 해당 결과를 리스트로 반환
    private static List<Integer> endMax(List<Integer> numbers) {
        return numbers.stream()
                .filter(e -> e %2 == 0)
                .collect(Collectors.toList())
    }

---

# 반복문(객체지향 및 구조적 프로그래밍) vs 스트림(함수형 프로그래밍) 성능 비교

### For-loop(반복문)을 이용한 배열 접근 후 최댓값 도출

    int[] a = ints;

    int e = ints.length;

    int m = Integer.MIN_VALUE;

    for(int i=0; i < e; i++)
    if(a[i] > m) m = a[i];

### Stream을 이용한 배열 접근 후 최댓값 도출

    int m = Arrays.stream(ints)
    .reduce(Integer.MIN_VALUE, Math::max);

> 두 코드를 동작시킨 환경에서의 성능차이는 다음과 같다.

int-array, for-loop : 0.36 ms

int-array, seq. stream: 5.35 ms


단편적인 예시이긴 하지만 **일반 배열이 아닌** **ArrayList 에서도 반복문이 더 우세한 결과**를 가지고 있었다.

https://pamyferret.tistory.com/49

또한 위 블로그에 따르면

For-loop이 더 빠른 이유를 설명해놓았는데


## - for문은 단순 인덱스 기반이다.
####  - for문은 단순 인덱스 기반으로 도는 반복문으로 메모리 접근이기 때문에 Stream에 비해 빠르고 오버헤드도 없다.
####  - stream의 경우는 JVM이 이것저것 처리해줘야 하는 것들이 많아 실행 시 느릴 수 밖에 없다.


## - for문은 컴파일러가 최적화를 시킨다.
####  - stream은 java 8부터 지원한 것이고 for문은 그보다 훨씬 오래전부터 계속 사용해왔다.
####  - 그만큼 사용되는 컴파일러는 오래 사용된 for문에 대한 처리가 되어 있어 for문의 경우 미리 최적화를 시킬 수 있지만,
####  - stream의 경우 신생(?)인 만큼 정보가 없어 for문과 같은 정교한 최적화를 수행할 수 없다.


이 두가지 근거가 설득력 있게 다가왔다.

---

하지만 **Stream이 성능이 좋지 않다고해서** 무조건 안좋다는 것은 아니다.

예시의 일부분일 뿐이고, **오히려 성능이 더 좋은 경우가 있을 것**이라고 생각할 뿐더러

**Stream 으로 개발자들에게 주는 가독성 측면**도 **무시할 수 없을** 이점이다.

실제로, **현업자**분들 에게 여쭤봤을 때 **Stream을 은근히 자주 쓴다**는 말씀을 하셨다.

---

# 공식 문서를 읽고 부연 정리하기

공식문서에는 Lamda를 사용할 수 있는 상황을 다른 방법을 여러 예시로 들어 단계별로 접근하고 있다.

---

### 코드로 알아보기 전 참고사항 Main 클래스의 Main 메서드 내용

    public static void main(String[] args) {
        Person person1 = new Person("diger", 24, Person.Gender.MALE);
        Person person2 = new Person("John", 25, Person.Gender.MALE);
        Person person3 = new Person("누구세요?", 26, Person.Gender.MALE);
        Person person4 = new Person("너 몇년도 군번이냐?", 23, Person.Gender.FEMALE);
        Person person5 = new Person("피도 안마른 자식", 22, Person.Gender.FEMALE);

        ArrayList<Person> personList = new ArrayList<>();
        personList.add(person1);
        personList.add(person2);
        personList.add(person3);
        personList.add(person4);
        personList.add(person5);

<br>

다음과 같은 예시 데이터를 셋팅해 주었다.


### 접근 1. 특정한 조건에 해당하는 객체를 찾아서 출력하는 메서드 생성

    // 조건 1. 어떤 객체의 age 가 매개변수로 받아온 age 보다 커야함
    public static void printPersonsOlderThan(List<Person> roster, int age) {
        System.out.println("접근 1 실행");
        for (Person person : roster) {
            if (person.getAge() >= age) {
                System.out.print(age + " 보다 크거나 같은 나이를 가진 사람의 이름은: ");
                person.printPerson(person);
            }
        }
        System.out.println("-------------------------------");
    }

<br>

> **출력 결과**
>
> 접근 1 실행
>
> 25 보다 크거나 같은 나이를 가진 사람의 이름은: John
>
> 25 보다 크거나 같은 나이를 가진 사람의 이름은: 누구세요?

---

### 접근 2. 특정한 조건에 해당하는 객체를 찾아서 출력하는 메서드 생성(접근 1. 보다 더 일반화함)

    public static void printPersonsWithinAgeRange(List<Person> roster, int low, int high) {
        System.out.println("접근 2 실행");
        for (Person person : roster) {
            if (low <= person.getAge() && person.getAge() < high) {
                System.out.print(low + " 보다 크고 " + high + " 보다 작은 나이를 가진 사람의 이름은: ");
                person.printPerson(person);
            }
        }
        System.out.println("-------------------------------");
    }

<br>

> **출력 결과**
>
> 접근 2 실행
>
> 23 보다 크고 26 보다 작은 나이를 가진 사람의 이름은: diger
>
> 23 보다 크고 26 보다 작은 나이를 가진 사람의 이름은: John
>
> 23 보다 크고 26 보다 작은 나이를 가진 사람의 이름은: 너 몇년도 군번이냐?

---

### 접근 3. 특정 조건에 부합하는지 확인하는 클래스와 클래스 메서드를 추가하여 활용하기

#### CheckPerson.java

    public class CheckPerson {

        int age = 24;

        public boolean test(Person person) {
            if (person.getAge() >= age) {
                return true;
            }
            return false;
        }
    }


#### Main.java

    // CheckPerson 클래스 생성 후 test 메서드 작성
    public static void printPersons(List<Person> roster, CheckPerson tester) {
        System.out.println("접근 3 실행");
        for (Person person : roster) {
            if (tester.test(person)) {
                person.printPerson(person);
            }
        }
        System.out.println("-------------------------------");
    }

<br>

> **출력 결과**
>
> 접근 3 실행
>
> diger
>
> John
>
> 누구세요?

---

### 접근 4. 익명 클래스로 특정 조건에 부합하는 클래스를 생성

    public static class Anonymous {

        Person person1 = new Person() {
            public void printPersons(CheckPerson checkPerson) {
                System.out.println("접근 4 실행");
                checkPerson.test(person1);
                System.out.println("Anonymous Class person printing");
            }
        };
    }

---

### 접근 5. 람다 표현식으로 작성한 유저 나이 필터링

    public static void printPersonByLambda(List<Person> personList, int targetAge) {
        System.out.println("접근 5 실행");


        for (Person person : personList) {
            Predicate<Integer> predicate = (age) -> age >= 22;

            if (predicate.test(person.getAge())) {
                System.out.print("Predicate 와 Lambda 를 활용한 유저 나이 필터링, 22살 보다 나이가 많은 유저는? : ");
                System.out.println(person.getName());
            }
        }
        System.out.println("-------------------------------");

    }

<br>

> **출력 결과**
>
> 접근 5 실행
>
> Predicate 와 Lambda 를 활용한 유저 나이 필터링, 22살 보다 나이가 많은 유저는? : diger
>
> Predicate 와 Lambda 를 활용한 유저 나이 필터링, 22살 보다 나이가 많은 유저는? : John
>
> Predicate 와 Lambda 를 활용한 유저 나이 필터링, 22살 보다 나이가 많은 유저는? : 누구세요?
>
> Predicate 와 Lambda 를 활용한 유저 나이 필터링, 22살 보다 나이가 많은 유저는? : 너 몇년도 군번이냐?
>
> Predicate 와 Lambda 를 활용한 유저 나이 필터링, 22살 보다 나이가 많은 유저는? : 피도 안마른 자식

---



### 접근 6. 람다와 스트림 결합

    public static void printPersonByLambdaAdvanced(List<Person> personList) {
        System.out.println("접근 6 실행");

        personList.stream()
                .filter(person -> person.getAge() >= 24)
                .forEach(person -> System.out.println(person.getName()));
    }

> **출력 결과**
>
> 접근 6 실행
>
> diger
>
> John
>
> 누구세요?
