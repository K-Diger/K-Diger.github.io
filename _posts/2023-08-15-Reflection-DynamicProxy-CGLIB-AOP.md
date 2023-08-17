---

title: 스프링은 왜 프록시를 짝사랑하는가... (리플렉션, 다이나믹 프록시, CGLIB, AOP)
date: 2023-08-15
categories: [Reflection, DynamicProxy, CGLIB, AOP]
tags: [Reflection, DynamicProxy, CGLIB, AOP]
layout: post
toc: true
math: true
mermaid: true

---

# 프록시 패턴에 대한 궁금증

스프링에서 AOP를 공부하려다 보면 그 개념의 근원은 프록시부터 시작한다.

AOP가 적용된 로직은 프록시 패턴을 적용하여 요구사항을 해결한다고 알려져있는데 도대체 프록시 패턴이 무엇이고 왜 쓰이고, 스프링은 왜 프록시 패턴으로 AOP를 구성하게 되었는지 알아보려고한다.

---

# 프록시 패턴?

디자인 패턴 중 `구조 패턴`으로 실제 객체에 접근을 제한하고, 그 실제 객체에 전달되기 전 부가적인 로직을 수행할 수 있게 구조화하는 패턴이다.

즉, OCP원칙과 꽤 연관이 있어 보인다. 기존 핵심 로직은 냅두고 부가 기능을 사용하고 싶을 때 프록시에 해당 로직을 추가하는 이 패턴을 적용하면 수월할 것이다.

아래 코드를 살펴보자

```kotlin
@RestController
class ItemController(
    private val itemService: ItemServiceProxy
) {

    @GetMapping("/items/{itemId}")
    fun getItem(@PathVariable itemId: Long): ItemResponse {
        return itemService.getItem(itemId)
    }
}

@Service
class ItemService(
    private val itemRepository: ItemRepository
) {

    fun getItem(itemId: Long): Item {
        val item = itemRepository.findById(itemId)
        return item ?: throw ItemNotFoundException("Item not found with id: $itemId")
    }
}

interface ItemServiceProxy {
    fun getItem(itemId: Long): ItemResponse
}

@Service
class ItemServiceProxyImpl(
    private val itemService: ItemService
) : ItemServiceProxy {
    override fun getItem(itemId: Long): ItemResponse {
        val logger = LoggerFactory.getLogger(this::class.java)
        logger.info("Proxy: Getting item $itemId")

        val item = itemService.getItem(itemId)
        return ItemResponse(item.id, item.name)
    }
}
```

- Controller는 요구사항을 해결하기 위해 ItemServiceProxy(인터페이스)를 주입받는다.

- ItemServiceProxy의 구현체인 ItemServiceProxyImpl은 아래와 같이 구성되어있다.
  - ItemService (핵심 로직)
    - ItemRepository (DB 접근)
  - 로깅 (부가 로직)

요구사항을 해결하기 위한 클라이언트의 입장인 Controller에서는 별 조치 없이 Proxy를 의존하는 것으로도 기능의 확장을 제공받을 수 있다.

Proxy의 구현체인 ProxyImpl에서 핵심 로직을 제외한 부가로직을 수정하기만 하면되기 때문이다.

이러한 용도로 사용되는 프록시 패턴은 다음 등장할 세 가지 기술들인 `Reflection`, `JDK DynamicProxy`, `CGLIB`를 통해 다뤄지기도 하며

이 세 가지 기술들을 활용하여 스프링은 AOP를 구현해낸다.

---

# Reflection

리플렉션은 클래스 혹은 메서드의 메타정보를 이용하여 동적으로 호출하는 메서드를 변경할 수 있다.

아래 시나리오를 살펴보자

```kotlin
class ItemController(
    private val itemService: ItemService
) {
    
    fun get() {
        itemService.itemFindById(1L)
    }
    
    fun getAll() {
        itemService.itemFindAll()
    }
}

class ItemService(
    private val itemRepository: ItemRepository
) {
    
    fun itemFindById(id: Long) {
        println("Id를 통해 Item 조회 시작")
        itemRepository.findById(id)
        println("Id를 통해 Item 종료")
    }

    fun itemFindAll() {
        println("모든 Item 조회 시작")
        itemRepository.findAll()
        println("모든 Item 조회 종료")
    }
}
```

같은 클래스 내에 중복되는 코드가 등장한다. 이 코드를 조금 더 간결하게 다룰 수 있는 방법 중에 리플렉션이 있다.

아래는 리플렉션을 통해 클래스, 메서드의 메타정보를 획득해서 메서드를 호출하는 구문이다.

```kotlin
class ItemController {

    private val itemService = ItemService()

    fun useReflectionGet() {
        invokeAndLog("itemFindById", 1L)
    }

    fun useReflectionGetAll() {
        invokeAndLog("itemFindAll")
    }
}

class ItemService(
    private val itemRepository : ItemRepository,
) {

    fun itemFindById(id: Long) {
        invokeAndLog("itemFindById", 1L)
    }

    fun itemFindAll() {
        invokeAndLog("itemFindAll")
    }

    private fun invokeAndLog(methodName: String, vararg args: Any?) {
        val targetMethod = itemRepository::class.java.getMethod(methodName, *args.map { it?.javaClass ?: Any::class.java }.toTypedArray())

        println("$methodName 시작")
        targetMethod.invoke(itemRepository, *args)
        println("$methodName 종료")
    }
}
```

위와 같이 리플렉션을 활용해서 메서드의 시작과 종료에 대한 로깅을 추상화 하여 사용할 수 있다.

어느정도 깔끔해졌다. 리플렉션을 활용하면 클래스, 메서드의 메타정보를 바탕으로 중복되는 메서드 호출에 대한 흐름을 추상화 할 수 있다.

## Reflection은 주의해야한다!

리플렉션을 적용한 코드는 컴파일 시점에서 잡을 수 없다.

또한 매 코드 실행마다 로우레벨에 위치한 클래스 로더를 통해 클래스 메타정보를 불러와야하기 때문에 성능도 좋지 않다.

---

# JDK 동적 프록시

리플렉션을 사용하여 객체를 직접 다루는 방법을 살펴보았는데 이와 비슷한 기법 중 자바에서 직접 제공하는 동적 프록시 기능이 있다.

`JDK 동적 프록시` : 주로 인터페이스 기반의 프록시를 생성할 때 사용한다. 특히 AOP에서 많이 사용되며, 프록시를 사용하여 횡단 관심사 (cross-cutting concern)를 처리할 때 유용하다.

`리플렉션` : 클래스의 구체적인 정보에 접근해야 할 때 사용한다. 예를 들어, 리플렉션을 사용하여 클래스의 private 메서드나 필드에 접근하거나, 런타임에 클래스의 동작을 변화시킬 때 활용될 수 있다.

```kotlin
interface ItemService {
    fun itemFindById(id: Long)
    fun itemFindAll()
}

class ItemServiceImpl(
    private val itemRepository: ItemRepository
) : ItemService {

    override fun itemFindById(id: Long) {
        itemRepository.itemFindById(id)
    }

    override fun itemFindAll() {
        itemRepository.itemFindAll()
    }
}

class LoggingHandler : InvocationHandler {
    override fun invoke(proxy: Any, method: Method, args: Array<out Any>?): Any? {
        val methodName = method.name
        println("$methodName 시작")
        val result = method.invoke(proxy, *(args ?: emptyArray()))
        println("$methodName 종료")
        return result
    }
}

fun createProxy(target: Any, handler: InvocationHandler): ItemService {
    return Proxy.newProxyInstance(
        target.javaClass.classLoader,
        arrayOf(ItemService::class.java),
        handler
    ) as ItemService
}

class ItemController(
    private val itemService: ItemService
) {

    fun useReflectionGet() {
        itemService.itemFindById(1L)
    }

    fun useRefelectionGetAll() {
        itemService.itemFindAll()
    }
}

class ItemRepository {
    fun itemFindById(id: Long) {
        println("실제 DB에서 아이템 조회")
    }

    fun itemFindAll() {
        println("실제 DB에서 모든 아이템 조회")
    }
}
```