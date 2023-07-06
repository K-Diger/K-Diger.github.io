---

title: Effective-Java Item 3. private 생성자 또는 enum 타입으로 싱글톤을 보장할 것
author: 김도현
date: 2023-01-09
categories: [Effective-Java]
tags: [Private, Constructor, Enum, Singleton]
layout: post
toc: true
math: true
mermaid: true

---

# 싱글톤이란?

애플리케이션 통 틀어서, 해당 클래스의 인스턴스가 하나만 사용되는 것이다.

---

# 싱글톤으로 만들기


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

# 싱글톤으로 만들기 - 정적 팩터리

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

# 직렬화가 뭐야??? 역직렬화는 또 뭐고

> 직렬화는 객체를 persistance 화 하는것,
>
> --> 객체를 전달할 때 최적화(객체를 옮길 수 있는 상태로)를 위해 사용한다. (직렬화)
>
> --> 객체를 전달받았을 때 최적화(객체를 사용할 수 있는 상태로)를 위해 사용한다. (역직렬화)

---

# 싱글톤으로 만들기 - Enum (글쎄...)

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

# 현실적인 싱글톤 만들기


> 생성자 주입 방식으로 만들어버립시다
