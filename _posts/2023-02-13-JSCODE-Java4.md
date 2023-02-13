---

title: JSCODE - 자바 스터디 4회차 (클래스, 다중생성자, 오버로딩) + 심화 미션(static을 지양하는 이유, Call By Reference, Call By Value)
author: 김도현
date: 2023-02-13
categories: [Java, Study]
tags: [Java, Study]
math: true
mermaid: true

---

# 4회차 미션 1. 학생들의 이름을 가나다 순으로 출력하기

## 요구사항

대여할 책의 번호를 입력하세요.
1. 클린코드(Clean Code) - 대여 가능
2. 객체지향의 사실과 오해 - 대여 가능
3. 테스트 주도 개발(TDD) - 대여 가능

1

정상적으로 대여가 완료되었습니다.

대여할 책의 번호를 입력하세요.

1. 클린코드(Clean Code) - 대여 중
2. 객체지향의 사실과 오해 - 대여 가능
3. 테스트 주도 개발(TDD) - 대여 가능

1

대여 중인 책은 대여할 수 없습니다.

대여할 책의 번호를 입력하세요.

1. 클린코드(Clean Code) - 대여 중
2. 객체지향의 사실과 오해 - 대여 가능
3. 테스트 주도 개발(TDD) - 대여 가능

2

정상적으로 대여가 완료되었습니다.

대여할 책의 번호를 입력하세요.

1. 클린코드(Clean Code) - 대여 중
2. 객체지향의 사실과 오해 - 대여 중
3. 테스트 주도 개발(TDD) - 대여 가능

## 주의사항

- 클래스 내에서 static 메서드는 사용하지마라. (public static void main(String[] args)는 제외)

- Main(프로그램을 실행하는 코드가 존재하는 클래스), Book(책), Library(도서관)의 클래스를 만들어서 활용해라.

- 한 파일에 모든 코드를 작성하지 말고, 1개의 클래스마다 클래스 파일을 별도로 생성해서 사용해라.

---

## 구현 - Main.class

```java
package src.class4;

import java.util.Scanner;

public class Main {

    public static void main(String[] args) {
        Main main = new Main();
        main.run(main.initLibrary());
    }

    public void run(Library library) {
        Scanner scanner = new Scanner(System.in);
        int booksCount = library.borrow(Integer.parseInt(scanner.nextLine()));
        while (booksCount != 0) {
            booksCount = library.borrow(Integer.parseInt(scanner.nextLine()));
        }
        System.out.println("모든 책이 대여되었습니다. 도서관 영업을 마칩니다 !");
    }

    public Library initLibrary() {
        Library library = new Library();
        library.printBorrowGuide(library.makeExampleBooks());
        return library;
    }
}


```

## 구현 - Book.class

```java
package src.class4;

public class Book {

    private Integer number;
    private String name;
    private Boolean status;

    public Book(Integer number, String bookName, Boolean status) {
        this.number = number;
        this.name = bookName;
        this.status = status;
    }

    public Integer getNumber() {
        return this.number;
    }

    public void setNumber(Integer number) {
        this.number = number;
    }

    public String getName() {
        return this.name;
    }

    public Boolean getStatus() {
        return this.status;
    }

    public void setName(String name) {
        this.name = name;
    }

    public void setStatus(Boolean status) {
        this.status = status;
    }
}

```

## 구현 - Library.class

```java
package src.class4;

import static java.lang.Boolean.FALSE;

import java.util.ArrayList;
import java.util.List;

public class Library {

    private final List<Book> books = new ArrayList<>();

    private int booksCount = 0;

    public List<Book> makeExampleBooks() {
        Book book1 = new Book(1, "클린코드(Clean Code)", true);
        Book book2 = new Book(2, "객체지향의 사실과 오해", true);
        Book book3 = new Book(3, "테스트 주도 개발(TDD)", true);
        books.add(book1);
        books.add(book2);
        books.add(book3);

        booksCount = books.size();
        return books;
    }

    public void printBorrowGuide(List<Book> books) {
        System.out.println("대여할 책의 번호를 입력하세요.");
        int i = 1;
        for (Book book : books) {
            if (isAvailableBorrow(book)) {
                System.out.println(i + ". " + book.getName() + " - " + "대여 가능");
            } else {
                System.out.println(i + ". " + book.getName() + " - " + "대여 중");
            }
            i++;
        }
    }

    public boolean isAvailableBorrow(Book book) {
        return book.getStatus();
    }

    public int borrow(Integer number) {
        if (books.get(number - 1).getStatus()) {
            System.out.println("정상적으로 대여가 완료되었습니다.");
            books.get(number - 1).setStatus(FALSE);
            booksCount--;
        } else {
            System.out.println("대여 중인 책은 대여할 수 없습니다.");
        }
        return booksCount;
    }

    public int getBooksCount() {
        return booksCount;
    }
}


```

## 아쉬운 점

- Library 클래스에서, Book을 등록할 때 사용자가 책 이름만 입력하더라도, 책의 번호, 책의 상태까지 자동으로 등록될 수 있는 로직을 만들어 보고 싶었다.

- 이번 학습 주제에 "오버로딩"이 있었는데, 내가 작성한 로직에 어떻게 적용할 수 있었을까?

- 과연 클래스를 클래스답게 사용했는가? 멘토님께서 추천해주신 "객체지향의 사실과 오해" 라는 책의 일부를 적용하면 어떻게 더 깔끔하게 바뀔 수 있을 지는 책을 더 읽어봐야 알 것 같다!

- 과연 Library가 수행하고 있는 기능 중 일부라도 Book 이라는 객체에서 해결할 수는 없었을까??

---

## 그래도 뿌듯했던 것!

- 현재 도서관에 대출 가능한 책의 숫자를 계산하여, 모든 책이 빌려질 때 까지 사용자에게 입력을 받는 요구사항을 추가해봤다.

- 조금 더 개선해서 매 입력을 받을 때마다 Guide를 출력할 수 있도록 해봐야겠다!

---

## 심화미션 - static을 지양하는 이유

Static은 사용하기 편하다. 언제 어디서든 호출이 가능하기 때문이다!

그렇지만 도대체 왜 이 편리함을 제공하는 static을 남발하면 안되는 것일까?

- 장점이자 단점으로 다가오는 점이 있다. 어느 곳에서든지 사용할 수 있다는 점이, 어디서 어떻게 사용하는 것으로 오류가 발생했는 지를 추론하기 힘들어진다.

- 객체지향의 가장 기본적인 원칙을 무너뜨린다. 객체지향은 캡슐화를 통해 외부에서 함부로 값을 수정하지 못하도록 하는 것을 추구하지만, static은 어디서든 값을 접근하여 수정할 수 있기 때문에 객체지향과 거리감이 있는 기능이다.

- GC 대상에 포함되지 않는다. GC는 Heap 영역을 대상으로 메모리 정리를 해준다. static 변수는 method area에 위치하게 되는데, 사용하지 않는 시점이 오더라도 그 메모리는 정리가 되지 않기 때문에 오히려 메모리를 낭비 시키는 기술이 될 수 있다.



## 심화미션 - Call By Reference, Call By Value

