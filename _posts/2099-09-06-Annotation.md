---

title: Annotation 이란?

date: 2022-09-06
categories: [Java, Spring, Annotation]
tags: [Java, Spring, Annotation]
layout: post
toc: true
math: true
mermaid: true

---

# Annotation(어노테이션) 은 무엇인가??

구글 번역기를 돌려보면 **"주석"** 이라고 나온다.


> Annotations, a form of metadata, provide data about a program that is not part of the program itself.
>
> Annotations have no direct effect on the operation of the code they annotate.

어노테이션은, 프로그램에 부가적인 데이터를 전달할 수 있는 메타데이터이다.

우리가 코드를 설명하기 위해 달아놓는 일반적인 주석과 다르게,

실제로 코드에 영향을 끼치는 특수 주석이라고 보면 된다.

<br>

### 주석 - javadoc 등장배경

추가적으로, 예전에는 코드를 작성하고 그 코드에 대한 설명을 명세하는 문서를 함께 관리를 하였는데

이 방식이 제대로 적용되지 않는 상황이 빈번하게 발생하여, 코드 + 문서의 개념을 도입하게 된 것이 javadoc 이다.

<br>

### 주석 - Annotation 등장배경

마찬가지로, 문서와 코드와의 버전관리로 인한 문제로 등장하게 된 것이다.

예전에는 XML 파일에 프로젝트 코드에 대한 설정 정보를 셋팅해놓았는데, 같은 프로젝트를 하는 사람이 1000명이 있다고하면... 똑같은 내용이 들어있는 XML 파일을 모두 가지고 있어야했다.

이 역시 관리가 힘들어 지면서 소스코드 계층에서 설정과 관련된 기능을 명시할 수 있도록 하기 위한 것이 Annotation 이다.

---

# Annotation(어노테이션)의 용도

### Information for the compiler
컴파일러에게 코드 문법 에러를 체크하도록 정보를 제공

### Compile-time and deployment-time processing
어노테이션 정보를 처리하여 코드, XML 파일 등을 생성 가능

### Runtime processing
실행시(Runtime) 특정 기능을 실행하도록 정보를 제공

### Build.gradle

    compileOnly 'org.projectlombok:lombok'
    annotationProcessor 'org.projectlombok:lombok'

Spring 으로 프로젝트를 진행하다 보면 보게된 의존성 추가 부분이다.

다양한 어노테이션을 제공하는 Lombok 라이브러리를

**compileOnly** 옵션으로 지정하여 컴파일 시에만 빌드하고 최종 빌드 결과물에는 포함되지 않는다.

**annotationProcessor** 옵션은 빌드 함에 있어서 기본적으로 포함되어있는 어노테이션이 아닌경우 명시해주는 옵션이다.

---

# Annotation(어노테이션) 타입

### 마커 어노테이션 (Maker Annotation)

    @NewAnnotation
    public class DigerClass() {
    }

위 코드 처럼 메서드 혹은 클래스 위에 부가정보 없이 선언만 하는 타입이다.

### 단일 값 어노테이션 (Single Value Annotation)

    @NewAnnotation(id = 1009)
    public class DigerClass() {
    }

하나의 값만 입력받을 수 있는 타입이다.

### 다중 값 어노테이션 (Multi Value Annotation)

    @NewAnnotation(id = 1009, name = “diger”, roles = {“admin”, “user"})
    public class DigerClass() {
    }

여러 개의 값을 입력받을 수 있는 타입이다.

---

# Java 표준 Annotation

### @Override

컴파일러에 오버라이딩 하는 메서드임을 알린다.

### @Deprecated

컴파일러에 앞으로 사용하지 않을 것을 알린다.

### @SuppressWarnings

컴파일러에 특정 경고 메세지를 표시하지 말라는 것을 알린다.

### @SafeVarargs (Java 7)

제네릭 타입의 가변인자에 사용한다.

### @FunctionalInterface (Java 8)

함수형 인터페이스임을 알린다.

---

# Meta Annotation

다른 어노테이션에 적용되는 어노테이션으로 쉽게 말하면, 다른 어노테이션에게 부가적인 정보를 전달해주는 어노테이션인 것이다.

### @Retention

어노테이션의 생명주기를 설정한다.

RetentionPolicy.SOURCE : 컴파일 전까지만 유효

RetentionPolicy.CLASS : 컴파일러가 클래스를 참조할 때까지 유효

RetentionPolicy.RUNTIME : 컴파일 이후 런타임 시기에도 JVM에 의해 참조가 가능

<br>

### @Target

어노테이션을 어디에 붙여 쓸 건지 지정한다.

> @Target 이 지정된 각 타입에는 어노테이션을 쓸 수 있게된다.

ElementType.PACKAGE : 패키지에 붙이기

ElementType.TYPE : 타입(클래스도 가능)에 붙이기

ElementType.ANNOTATION_TYPE : 어노테이션 타입에 붙이기

ElementType.CONSTRUCTOR : 생성자에 붙이기

ElementType.FIELD : 멤버 변수에 붙이기

ElementType.LOCAL_VARIABLE : 지역 변수에 붙이기

ElementType.METHOD : 메서드에 붙이기

ElementType.PARAMETER : 전달인자에 붙이기

ElementType.TYPE_PARAMETER : 전달인자 타입에 붙이기

ElementType.TYPE_USE : 타입 선언에 붙이기

<br>

### @Documented

해당 어노테이션을 Javadoc에 포함시킴

<br>

### @Inherited

어노테이션의 상속을 가능하게 함


### @Repeatable

Java8 부터 지원하며, 연속적으로 어노테이션을 선언할 수 있게 함

---

# Spring 3대 Annotation 들춰보기 (Controller, Service, Repository)

### Annotation - Controller

    @Target(ElementType.TYPE)
    @Retention(RetentionPolicy.RUNTIME)
    @Documented
    @Component
    public @interface Controller {

        /**
         * The value may indicate a suggestion for a logical component name,
         * to be turned into a Spring bean in case of an autodetected component.
         * @return the suggested component name, if any (or empty String otherwise)
         */
        @AliasFor(annotation = Component.class)
        String value() default "";
    }

| Target           | Retention               |
|------------------|-------------------------|
| ElementType.Type | RetentionPolicy.RUNTIME |

#### @Target

ElementType.TYPE으로 정의되어, 우리가 흔히 Controller 클래스 파일을 만들었을 때, 해당 클래스에 어노테이션을 붙일 수 있게 해두었다.

#### @Retention

런타임 시에도 유지되어야 하므로 RetentionPolicy.RUNTIME으로 작성 해두었다.

<br>

### Annotation - Service

    @Target(ElementType.TYPE)
    @Retention(RetentionPolicy.RUNTIME)
    @Documented
    @Component
    public @interface Service {

        /**
         * The value may indicate a suggestion for a logical component name,
         * to be turned into a Spring bean in case of an autodetected component.
         * @return the suggested component name, if any (or empty String otherwise)
         */
        @AliasFor(annotation = Component.class)
        String value() default "";

    }

| Target           | Retention               |
|------------------|-------------------------|
| ElementType.Type | RetentionPolicy.RUNTIME |

#### @Target

ElementType.TYPE으로 정의되어, 우리가 흔히 Controller 클래스 파일을 만들었을 때, 해당 클래스에 어노테이션을 붙일 수 있게 해두었다.

#### @Retention

런타임 시에도 유지되어야 하므로 RetentionPolicy.RUNTIME으로 작성 해두었다.

<br>

### Annotation - Repository

    @Target(ElementType.TYPE)
    @Retention(RetentionPolicy.RUNTIME)
    @Documented
    @Component
    public @interface Repository {

        /**
         * The value may indicate a suggestion for a logical component name,
         * to be turned into a Spring bean in case of an autodetected component.
         * @return the suggested component name, if any (or empty String otherwise)
         */
        @AliasFor(annotation = Component.class)
        String value() default "";

    }

| Target           | Retention               |
|------------------|-------------------------|
| ElementType.Type | RetentionPolicy.RUNTIME |

#### @Target

ElementType.TYPE으로 정의되어, 우리가 흔히 Controller 클래스 파일을 만들었을 때, 해당 클래스에 어노테이션을 붙일 수 있게 해두었다.

#### @Retention

런타임 시에도 유지되어야 하므로 RetentionPolicy.RUNTIME으로 작성 해두었다.

---


> 나는 처음에 @Controller 라는 어노테이션이 정말 자동적으로 엄청난 일을 해주는 것으로 생각했었다.
>
> 그런데, Spring 이 @Controller 어노테이션을 어떻게 인식하는지를 알아보고, 생각하다가 깨달은 내용은
>
> 그저 우리가 @Controller 가 붙은 어노테이션에 대한 클래스에 그 역할에 맞는 코드를 작성했던 것이 정답이였다.
>
> 그러니까, @Controller, @Service, @Repository 이 3가지 주요 어노테이션은 개발자들에게 이건 컨트롤러야. 이건 서비스야. 이건 레포야. 를
>
> 알려주기 위한 어노테이션이었던 것 뿐, 정말 말 그대로 주석에 해당하는 것이었다....

---

# 그러면 3대 어노테이션에도 달려있는, 진짜진짜 큰일을 해주는 것 같은 @Component 는 ???

    @Target(ElementType.TYPE)
    @Retention(RetentionPolicy.RUNTIME)
    @Documented
    @Indexed
    public @interface Component {

        /**
         * The value may indicate a suggestion for a logical component name,
         * to be turned into a Spring bean in case of an autodetected component.
         * @return the suggested component name, if any (or empty String otherwise)
         */
        String value() default "";

    }

| Target           | Retention               |
|------------------|-------------------------|
| ElementType.Type | RetentionPolicy.RUNTIME |


#### @Target

ElementType.TYPE으로 정의되어, 우리가 흔히 Controller 클래스 파일을 만들었을 때, 해당 클래스에 어노테이션을 붙일 수 있게 해두었다.

#### @Retention

런타임 시에도 유지되어야 하므로 RetentionPolicy.RUNTIME으로 작성 해두었다.


### Indexed

@Component 어노테이션에는 위의 어노테이션들과는 다르게 추가적인 메타 어노테이션이 존재한다.

> Consider the default Component annotation that is meta-annotated with this annotation.
>
> If a component is annotated with Component, an entry for that component will be added to the index using the org.springframework.stereotype.Component stereotype.

Spring 이 Bean 으로 등록할 수 있도록 컴포넌트를 등록하고 싶을 때, @Indexed 어노테이션을 달아놓으라는 얘기이다.

---

# (아니 근데) 컴포넌트스캔의 동작과정이 궁금하다!!

막상 @Component 어노테이션을 까보니까 얘도 별게 없었다..

도대체 그러면 어떻게 @Component 어노테이션을 달면 컴포넌스 스캔의 대상이 되는것인가..?

    @Target(ElementType.TYPE)
    @Retention(RetentionPolicy.RUNTIME)
    @Documented
    @Component
    public @interface Configuration {

        /**
         * Explicitly specify the name of the Spring bean definition associated with the
         * {@code @Configuration} class. If left unspecified (the common case), a bean
         * name will be automatically generated.
         * <p>The custom name applies only if the {@code @Configuration} class is picked
         * up via component scanning or supplied directly to an
         * {@link AnnotationConfigApplicationContext}. If the {@code @Configuration} class
         * is registered as a traditional XML bean definition, the name/id of the bean
         * element will take precedence.
         * @return the explicit component name, if any (or empty String otherwise)
         * @see AnnotationBeanNameGenerator
         */
        @AliasFor(annotation = Component.class)
        String value() default "";

        /**
         * Specify whether {@code @Bean} methods should get proxied in order to enforce
         * bean lifecycle behavior, e.g. to return shared singleton bean instances even
         * in case of direct {@code @Bean} method calls in user code. This feature
         * requires method interception, implemented through a runtime-generated CGLIB
         * subclass which comes with limitations such as the configuration class and
         * its methods not being allowed to declare {@code final}.
         * <p>The default is {@code true}, allowing for 'inter-bean references' via direct
         * method calls within the configuration class as well as for external calls to
         * this configuration's {@code @Bean} methods, e.g. from another configuration class.
         * If this is not needed since each of this particular configuration's {@code @Bean}
         * methods is self-contained and designed as a plain factory method for container use,
         * switch this flag to {@code false} in order to avoid CGLIB subclass processing.
         * <p>Turning off bean method interception effectively processes {@code @Bean}
         * methods individually like when declared on non-{@code @Configuration} classes,
         * a.k.a. "@Bean Lite Mode" (see {@link Bean @Bean's javadoc}). It is therefore
         * behaviorally equivalent to removing the {@code @Configuration} stereotype.
         * @since 5.2
         */
        boolean proxyBeanMethods() default true;

    }

<br>

1. ConfigurationClassParser 가 Configuration 클래스를 파싱한다. @Configuration 어노테이션 클래스를 파싱하는 것이다.

2. ComponentScan 설정을 파싱한다. base-package 에 설정한 패키지를 기준으로 ComponentScanAnnotationParser가 스캔하기 위한 설정을 파싱한다.

3. base-package 설정을 바탕으로 모든 클래스를 로딩한다.

4. ClassLoader가 로딩한 클래스들을 BeanDefinition으로 정의한다. 생성할 빈의 대한 정의를 하는 것이다.

5. 생성할 빈에 대한 정의를 토대로 빈을 생성한다.
