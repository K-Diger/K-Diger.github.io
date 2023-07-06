---

title: Item 11. equals를 재정의할꺼면, hashCode도 재정의해라.
author: Diger
date: 2022-12-10
categories: [Effective-Java]
tags: [hashCode]
layout: post
toc: true
math: true
mermaid: true

---

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
