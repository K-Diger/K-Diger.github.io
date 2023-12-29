---

title: 스프링을 잘 쓴다는 것은 애노테이션을 많이 아는 것
date: 2023-11-11
categories: [Spring, SpringSecurity, JPA]
tags: [Spring, SpringSecurity, JPA]
layout: post
toc: true
math: true
mermaid: true

---

# 의존성 주입 관련 애노테이션

- @Autowired
- @Qualifier
- @Inject (deprecated)
- @Named (deprecated)
- @Primary

- @Value
- @Import
- @DependsOn

- @ConstructorProperties
- @Lookup
- @AliasFor

---

## @Autowired/@Inject, @Qualifier/@Named

이 애노테이션들은 의존성을 필드 주입 할 때 사용되는 애노테이션이다. 하지만 필드 주입은 아래와 같은 사유로 권장되지 않는다.

- 순환 참조
- 테스트가 어려움
- 불변, null의 위험성이 있다.

### @Autowired + @Qualifier

```java
public class AutowiredController {

    @Autowired
    @Qualifier("autowiredSampleService2")
    private AutowiredSampleService autowiredSampleService;

    @GetMapping
    public String autowired() {
        return "autowired";
    }
}
```

### @Inject + @Named (Deprecated)

`@Inject`애노테이션도 `@Autowired`애노테이션과 동일한 역할을 수행한다. 또한 @Named와 결합되어 같은 타입의 Bean이 있을 경우 우선적으로 사용할 Bean을 지정할 수 있다.

```java
public class InjectController {

    @Inject
    @Named("injectSampleService2")
    private InjectSampleService injectSampleService;

    @GetMapping
    public String inject() {
        return "inject + named";
    }
}
```

### @Autowired vs @Inject

`@Autowired`는 `required`옵션을 통해 의존성이 선택 사항인지 지정할 수 있도록 할 수 있다.

```java
@Autowired(required = false)
private SomeBean optionalBean;
```

---

## @Primary

@Quauilfier와 유사한 애노테이션이다. 같은 타입을 가진 Bean이 존재할 경우 `@Primary`애노테이션을 활용하여 어떤 Bean을 사용할 것이지 우선순위를 지정할 수 있다.

```java
@Configuration
public class Config {

    @Bean
    public Employee JohnEmployee() {
        return new Employee("John");
    }

    @Bean
    @Primary
    public Employee TonyEmployee() {
        return new Employee("Tony");
    }
}
```

---

## @Value

`@Value`는 외부화된 속성을 주입하는 데 사용된다. `application.yml`파일에 외부 설정 정보를 등록하고 해당 애노테이션으로 그 값을 끌어오는 것이다.

```yaml
catalog:
    name: MovieCatalog
```

```java
@Component
public class MovieRecommender {

    private final String catalog;

    public MovieRecommender(@Value("${catalog.name}") String catalog) {
        this.catalog = catalog;
    }
}
```

또한 아래와 같이 해당 외부 정보를 관리하는 Config파일을 통해 관리할 수도 있다.

```yaml
@Configuration
@PropertySource("classpath:application.yml")
public class AppConfig { }
```

---

## @Import

`@TestConfiguration`과 함께 사용되는 사용 방법이 있다.

```java
@TestConfiguration
public class TestWebConfig {
    @Value("${jwt.secret}")
    private String secret;

    @Bean
    public JwtUtil jwtUtil() {
        return new JwtUtil(secret);
    }

    @Bean
    public UserRepository userRepository() {
        return InMemoryUserRepository.getInstance();
    }

    @Bean
    public AuthenticationService authenticationService() {
        return new AuthenticationService(userRepository(), jwtUtil());
    }
}
```
위와 같은 TestConfigBean을 사용하기 위해 아래와 같이 사용하여 설정 정보를 Import시켜주는 기능을 보유한다.

```java
@WebMvcTest(Controller.class)
@Import(TestWebConfig.class)
class ControllerTest { }
```

**테스트 뿐만 아니라** Application Layer의 설정 Bean을 위와 같이 사용할 수 있게 된다.

---

## @DependsOn

어떤 `Bean`이 필요로 하는 의존성을 표기한다. 이 애노테이션을 활용한다면 `Bean`이 생성되는 순서를 조정할 수도 있다.

```java
@Configuration
@ComponentScan
public class Config {

    @Bean("finalDependencyDestination")
    @DependsOn({"dependency1", "dependency2"})
    public Integer finalDependencyDestination() {
        return 3;
    }

    @Bean("dependency2")
    @DependsOn({"dependency1"})
    public Integer dependency2() {
        return 2;
    }

    @Bean("dependency1")
    public Integer dependency1() {
        return 1;
    }
}
```

위와 같은 코드는 코드의 순서에 관계없이 `dependency1` -> `dependency2` -> `finalDependencyDestination` 순서대로 생성된다.
