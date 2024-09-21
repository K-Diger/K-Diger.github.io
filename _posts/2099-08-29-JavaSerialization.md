---

title: 직렬화에 관하여

date: 2022-08-29
categories: [Java, Serialization]
tags: [Java, Serialization]
layout: post
toc: true
math: true
mermaid: true

---

# 직렬화란?

객체의 상태를 영속화 하는 메커니즘이다.

즉, 객체를 다른 환경(File, Memory, DB 등...)에 저장 후 나중에 재구성할 수 있도록 하는 것으로

구체적으로는, 객체를 byte Stream 으로 변환하여 다른 환경을 저장하는 것이다.

> serialization is the conversion of a Java object into a static stream (sequence) of bytes

Baeldung 에서 발췌한 설명글이다. 직렬화는 객체를 static stream 으로 변환하는 것이다.

### 질문 1.

JVM의 static 영역에 객체를 바이트 배열로 변환한 내용을 담는 것인데..

어쨋든 간에 JVM 은 프로그램이 종료되면 그 내용도 사라지므로 직렬화한 내용도 사라지게 될 것이다.

직렬화한 내용을 파일이나 DB에 쓰지 않으면

그러면, 영속화라는 개념에 맞지 않게 되는데 왜 영속화한다고 하는 것일까?

### 질문 2.

    FileOutputStream fos = new FileOutputStream("objectfile.ser");
    ObjectOutputStream out = new ObjectOutputStream(fos);
    out.writeObject(new User("찰리", 31));

위는 직렬화를 수행한 내용을 파일에 저장한 내용이다.

근데 실제 객체의 내용을 직접 파일로 저장하는 것으로 직렬화의 역할을 대체할 수는 없는건가?

### 답변 2.

안된다. 객체의 내용이 아닌 String 이나 Integer 같은 내용을 파일에 쓰고 저장할 때, .getBytes() 를 통해 바이트코드로 변환 후 저장이 가능하기 때문이다.

그래서 String 이나 Integer 만 파일에 쓰는게 아니라 객체의 내용 자체를 파일에 쓰려면 객체의 내용을 Byte 로 변환해주는 직렬화를 사용해야한다.

---

### 질문 3.

Spring 에서 JPA 도 객체의 내용을 DB 에 저장하는 영속성을 가지고 있다.

그럼 JPA 구현체인 Hibernate 에도 직렬화의 코드가 작성되어 있는 것인가?

### 답변 3.

되어있긴 한듯 안한듯.

If an entity instance is to be passed by value as a detached object (e.g., through a remote interface), the entity class must implement the Serializable interface.

--> 굳이 안써도 된다.

JPA 표준 스펙에는 명시가 되어 있긴한데 해당 내용을 사용하는 상황이 그리 흔치 않다.

> [김영한님의 답변](https://www.inflearn.com/questions/17117)
>
> JPA 표준 스펙은 Entity에 Serializable를 구현하도록 되어 있습니다. JPA 구현체에 따라 엔티티를 분산 환경에서 사용할 수 있거나, 직열화해서 다른 곳에 전송할 수 있는 가능성을 열어준거지요.
>
> 하지만 저희가 주로 사용하는 하이버네이트 구현체는 제가 아는 바로 다음 사례를 제외하고는 Entity를 직열화해서 내부에서 사용하는 경우는 못봤습니다.
>
> https://www.inflearn.com/questions/16570
>
> 그래서 저는 실용적인 관점에서 실무에서 엔티티에 Serializable를 거의 사용하지 않습니다.
>
> (하지만 표준 스펙이니 적용해두는게 더 나은 선택입니다.)
>
> 하이버네이트 구현체는 객체를 임시로 직렬화(Serializable)해서 메모리에 올려두는 작업을 하는 것 같습니다.

# 역직렬화란?

직렬화의 반대 메커니즘으로

> 영속화된 객체를 재사용하는 방법이다.

---

# 직렬화 - 코드

    // 직렬화 전 셋팅, ByteOutputStream 과 ObjectOutputStream 의 인스턴스를 만든다.
    ByteArrayOutputStream baos = new ByteArrayOutputSteram();
    ObjectOutputStream oos = new ObjectOutputStream(baos);
    CustomObject customObject = new CustomObject(1);

    // 직렬화 - 1단계 (직렬화 수행)
    oos.writeObject(customObject);

    // 직렬화 - 2단계 (직렬화 된 Byte Stream 획득)
    byte[] serializedObject = byteArrayOutputStream.toByteArray();


---

# 역직렬화 - 코드


    // 역직렬화 전 셋팅, ByteInputStream 과 ObjectOutputStream 의 인스턴스를 만든다.
    ByteArrayInputStream bios = new ByteArrayInputSteram();
    ObjectInputStream ois = new ObjectInputStream(bios);

    // 역직렬화 수행
    CustomObject customObject = (CustomObject) ois.readObject();


---

# 직렬/역직렬화 흐름

> Object -> writeObject -> (File, Memory, DB) -> readObject -> Object


---

# 일단 직렬화를 남발하면 안되는 이유

## 보안 - 보이지 않는 생성자 (read Object)

역직렬화 시 readObject 라는 메서드가 호출된다.

readObject 를 통해 객체를 불러온 후, 그 내부의 Byte 배열의 값을 변경한다면 그 객체의 내용 자체가 변질되게 된다.

ReverseEngineering 을 Java Code 로 하는 꼴이다.

### 예제 코드

    // 역직렬화를 통해 가져온 객체를 담은 바이트 배열
    byte[] serializedObject = getSerializedObject(1);

    // 역직렬화를 통해 가져온 바이트 배열의 값을 임의로 수정
    serializedObject[0] = -1;
    serializedObject[1] = -1;
    serializedObject[2] = -1;
    serializedObject[3] = -1;
    serializedObject[4] = -1;

<br>

## 싱글톤

싱글톤으로 생성된 객체를 직렬화 후 역직렬화를 하게 되면

싱글톤으로 생성했던 같은 객체가 꺼내지는 것이 아니다. (체력이 1남은 드론이 가스를 짓고 취소하면, 가스를 짓던 체력 1의 드론이 아닌 새로운 체력 40의 드론이 나오는 상황과 유사하다.)

