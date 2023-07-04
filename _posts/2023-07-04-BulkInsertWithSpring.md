---

title: JPA에 익숙한 사용자에게 대량의 데이터를 삽입하는 효율적인 방법
author: 김도현
date: 2023-07-04
categories: [Spring, MySQL]
tags: [Spring, MySQL]
math: true
mermaid: true

---

## 참고 자료

[Baeldung - Save vs SaveAll](https://www.baeldung.com/spring-data-save-saveall)

---

## 문제 상황

프로젝트를 진행하는 중 네이버 뉴스를 크롤링하여 가공하고 데이터베이스에 삽입하는 요구사항이 등장했다.

이 요구사항을 해결하기 위해 크롤링한 데이터를 Data JPA의 save()메서드로 삽입하여 해결하고 있었다.

하지만 여기서 문제가 발생했다. 한 번의 크롤링으로 약 800 ~ 1500개 사이의 데이터를 삽입해야하는데 이걸 일일히 Insert Query를 날리고 있던 것

즉, MySQL의 Connection 관점으로 본다면 크롤링 할 때마다 800 ~ 1500개의 커넥션이 맺고 끊어진다는 것이다.

이로 인해 Connection Pool이 모자라는 `too many connection`이라는 에러가 발생하기도 했고 삽입까지 걸리는 시간이 굉장히 오래 걸렸다. (10분 ~ 15분 사이)

---

## DBMS 관점에서의 해결방안

Replication을 통해 Read/Write DB를 분리한다.

MySQL은 Replication을 지원하여 Master - Slave 관계를 구축하여 Master에 변경 쿼리를 수행하고, Slave에 그 변경점을 복제하여 조회전용으로 사용하는

부하 분산 방법이 존재한다. 이 방법으로 해결 할 수 있을 것이다.

## 애플리케이션 관점에서의 해결방안

일단 코드로 인해 발생하는 쿼리 자체가 문제이다. 매 삽입 마다 Connection을 맺어 하나의 INSERT 구문을 수행하는 것보단

Bulk Insert 방식으로 한 번에 INSERT하는 것이 훨씬 효율적이다.

---

## 애플리케이션 관점에서의 해결방안 - 적용

우선 Bulk Insert를 수행할 수 있는 방법은 세가지 방법이 있다.

- JdbcTemplate로 Bulk Insert 작성하기
- Spring Data JPA에서 제공하는 saveAll() 메서드 사용하기
- MyBatis로 Bulk Insert 작성하기

JPA에 익숙해졌다면 두 번째 방법인 saveAll() 메서드로 해결하고 싶은 욕구가 치솟을 것이다. 하지만 각 방법의 장단점을 비교하고 사용해야 좀 더 효율적으로 해결이 가능할 것이다.

### saveAll() 메서드로 Bulk Insert 처리하기

우선 saveAll()이 어떻게 생겼는지 살펴보자

```kotlin

    // saveAll()
	@Transactional
    @Override
    public <S extends T> List<S> saveAll(Iterable<S> entities) {

        Assert.notNull(entities, "Entities must not be null");

        List<S> result = new ArrayList<>();

        for (S entity : entities) {
            result.add(save(entity));
        }

        return result;
    }

    // saveAll()에서 사용하는 save()메서드
    @Transactional
    @Override
    public <S extends T> S save(S entity) {

        Assert.notNull(entity, "Entity must not be null");

        if (entityInformation.isNew(entity)) {
            em.persist(entity);
            return entity;
        } else {
            return em.merge(entity);
        }
    }
```

save() vs saveAll()로 대량의 데이터를 삽입했을 때 발생하는 문제점은 위 코드에서 살펴볼 수 있다.

두 메서드는 @Transactional 애노테이션을 사용하고 있지만 트랜잭션 전파 옵션을 따로 설정하지 않고 있다.

따라서 기본값 옵션에 따라 이미 진행 중인 트랜잭션이 있다면 그 트랜잭션에 포함되고, 트랜잭션이 걸려있지 않다면 새 트랜잭션을 정의하여 사용한다.

데이터가 1억개 있다고 생각해보자

아래의 save() 메서드는 1억번 Transaction을 건다.
```kotlin
for(int i = 0; i < bookCount; i++) {
    bookRepository.save(new Book("Book " + i, "Author " + i));
}
```

아래의 saveAll() 메서드는 한 번의 Transaction을 재사용한다.
```kotlin
List<Book> bookList = new ArrayList<>();

for (int i = 0; i < bookCount; i++) {
    bookList.add(new Book("Book " + i, "Author " + i));
}

bookRepository.saveAll(bookList);
```

트랜잭션을 1억번 맺고 끊는 것 + DB에 일일히 1억개의 Insert Query를 날리는 것

트랜잭션을 한 번 맺고 끊는 것 + DB에 한번에 1억개의 Insert Query를 날리는 것

당연하게도 후자가 더 성능이 좋을 수 밖에 없다.

실제로 테스트를 때려봐도 성능차이가 꽤 존재했다. 테스트는 약 5000개의 데이터를 테스트 테이블에 삽입하는 상황을 연출했다.

save()

```kotlin
    @Test
    @DisplayName("save 메서드 수행시간")
    fun save() {
        val allNews = newsRepository.findAll()
        val startTime = System.currentTimeMillis()
        for (news in allNews) {
            testDomainRepository.save(
                TestDomain(
                    title = news.title,
                    content = news.content,
                    thumbnailImageUrl = news.thumbnailImageUrl,
                    newsLink = news.newsLink,
                    press = news.press,
                    writtenDateTime = news.writtenDateTime,
                    type = news.type,
                    crawledCount = news.crawledCount,
                    category = news.category,
                )
            )
        }
        val endTime = System.currentTimeMillis()
        println("${endTime - startTime} ms")
    }

출력 결과 : 6134 ms
```

saveAll()

```kotlin
    @Test
    @DisplayName("saveAll 메서드 수행시간")
    fun saveAll() {
        val allNews = newsRepository.findAll()
        val startTime = System.currentTimeMillis()
        val persistenceTarget = mutableListOf<TestDomain>()
        for (news in allNews) {
            persistenceTarget.add(
                TestDomain(
                    title = news.title,
                    content = news.content,
                    thumbnailImageUrl = news.thumbnailImageUrl,
                    newsLink = news.newsLink,
                    press = news.press,
                    writtenDateTime = news.writtenDateTime,
                    type = news.type,
                    crawledCount = news.crawledCount,
                    category = news.category,
                )
            )
        }
        testDomainRepository.saveAll(persistenceTarget)
        val endTime = System.currentTimeMillis()
        println("${endTime - startTime} ms")
    }

출력 결과 : 4736 ms
```

출력 결과로 보아 알 수 있듯이 그리 드라마틱한 성능개선은 이루어지지 않았다. 또한 로그에 찍히는 INSERT 쿼리가 save()메서드와 다를 바 없이 엄청나게 박혔었다.

이게 최선의 방법일까? 다른 방법으로 Bulk Insert를 수행해보자.

### JdbcTemplate로 Bulk Insert 처리하기

Bulk Insert를 처리할 레포지토리는 아래와 같다.

```kotlin
@Repository
class TestDomainBulkInsertRepository(
    private val jdbcTemplate: JdbcTemplate,
) {

    @Transactional
    fun bulkInsert(testDomains: List<TestDomain>, crawledDateTime: LocalDateTime): Long? {
        val sql =
            """INSERT INTO test_domain (title, content, news_link, press, thumbnail_image_url, type, written_date_time, crawled_count, category_id, created_at, modified_at)
                VALUES (?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?)
            """.trimMargin()

        jdbcTemplate.batchUpdate(
            sql,
            object : BatchPreparedStatementSetter {
                override fun setValues(ps: PreparedStatement, i: Int) {
                    val news = testDomains[i]
                    ps.setString(1, news.title)
                    ps.setString(2, news.content)
                    ps.setString(3, news.newsLink)
                    ps.setString(4, news.press)
                    ps.setString(5, news.thumbnailImageUrl)
                    ps.setString(6, news.type)
                    ps.setString(7, news.writtenDateTime)
                    ps.setInt(8, news.crawledCount)
                    ps.setLong(9, news.category.id)
                    ps.setTimestamp(10, Timestamp.valueOf(crawledDateTime))
                    ps.setTimestamp(11, Timestamp.valueOf(LocalDateTime.now()))
                }

                override fun getBatchSize(): Int {
                    return testDomains.size
                }
            }
        )
        return jdbcTemplate.queryForObject("SELECT LAST_INSERT_ID()", Long::class.java)
    }
}

@Test
@DisplayName("Jdbc Template Bulk Insert 메서드 수행시간")
fun `Jdbc Template Bulk Insert`() {
    val allNews = newsRepository.findAll()
    val startTime = System.currentTimeMillis()
    val persistenceTarget = mutableListOf<TestDomain>()
    for (news in allNews) {
        persistenceTarget.add(
            TestDomain(
                title = news.title,
                content = news.content,
                thumbnailImageUrl = news.thumbnailImageUrl,
                newsLink = news.newsLink,
                press = news.press,
                writtenDateTime = news.writtenDateTime,
                type = news.type,
                crawledCount = news.crawledCount,
                category = news.category,
            )
        )
    }
    testDomainBulkInsertRepository.bulkInsert(persistenceTarget, LocalDateTime.now())
    val endTime = System.currentTimeMillis()
    println("${endTime - startTime} ms")
}

출력 결과 : 2952 ms
```

JPA-save (6134) -> JPA-saveAll (4736) -> JdbcTemplate (2952)

꽤나 좋은 성능 개선 사례가 되었다. 기존 코드에 비해 절반 정도 소요시간이 감소되었다.

그럼 성능이 좋으니 무조건 JdbcTemplate을 사용해서 처리하는게 좋을까?

물론 성능이 1순위로 여겨지고 그 이후에 발생하는 부수 효과에 영향이 크게 없다면 바로 적용해도 좋을 것이다.

하지만 나는 이 데이터를 영속화해서 사용하는게 목적이였다.

쉽게 말하자면, 이 testDomain에 해당하는 데이터를 newsCard라는 Wrapper Entity에 사용해야하는데 JdbeTemplate으로 Insert를 수행하면

영속성에 해당되지 않기 때문에 AutoIncrement가 적용된 PK를 인식하지 못하는 현상이 발생했다.

그 이유를 간단하게 알아보면 다음과 같다.

JPA의 Id 생성 전략 중 일반적으로 AutoIncrement와 싱크를 맞추는 옵션인 IDENTITY는 Primary Key 값 지정을 DB에 위임한다는 의미이다.

따라서 객체에 Id를 제외한 값을 넣고 Save Insert를 하면 id는 null 값이 들어간 상태가 된다.

그 후 Save한 데이터를 꺼내오는 것으로 영속화 할 때 그 Id 값을 알 수 있다. 이 근본적인 원인은 IDENTITY 전략은 `지연 쓰기`를 지원하지 않는 것이다.

[더 자세한 내용은 이 링크에 잘 설명되어 있다!](https://stackoverflow.com/questions/27697810/why-does-hibernate-disable-insert-batching-when-using-an-identity-identifier-gen/27732138#27732138)

그러면 다른 옵션 중 TABLE, SEQUENCE도 존재하는데 이 옵션을 쓰면 해결되지 않을까?

일단 MySQL은 SEQUENCE옵션을 지원하지 않는다. 따라서 이 방법은 사용하지 못한다. (다른 DB라면 고려해볼 순 있겠다.)

Table 전략은 PK를 관리하는 테이블을 별도로 관리해야한다는 부가적인 리소스가 소요된다.

자 그러면 어떻게 해야할까? 이 벌크 연산 때문에 PK의 속성을 바꿔야하나? 아니면 PK를 연동할 방법은 없는가?

사실 AutoIncrement를 사용할 방법이 아예없는건 아니다. 일단 소프트웨어적으로 해결한 내 방법은 아래와 같다.

1. 데이터 삽입 전 현재 DB에 있는 가장 마지막 인덱스를 뽑아온다.
2. 데이터를 삽입 후 가장 마지막의 인덱스를 뽑아온다.
3. 1번과 2번의 사이에 해당하는 모든 수를 추출하여 Wrapper Entity에서 보관하고 사용한다.

해결 한 코드 일부

```kotlin
// 크롤러한 뉴스 삽입 전 마지막 News의 Index
var currentLastNewsIndex = 1L

// 현재 DB에 존재하는 가장 마지막 뉴스
val lastNews = newsRepository.findTopByOrderByIdDesc()

// 만약 DB에 뉴스가 존재한다면 해당 뉴스의 id + 1를 다음에 삽입될 인덱스로 지정
if (lastNews != null) {
    currentLastNewsIndex = lastNews.id + 1
}

// 크롤러한 뉴스 삽입 후 마지막 News의 Index
val newNewsLastIndex = newsBulkInsertRepository.bulkInsert(
    newsBundle = persistenceTargetNewsBundle,
    crawledDateTime = crawledDateTime
)

val extractedKeywords = keywordExtractor.extractKeywordV2(
    newsRepository.findById(newNewsLastIndex!!.toLong()).get().content
)

val persistenceNewsCard = NewsCard(
    category = category,
    multipleNews = filterSquareBracket(
        (currentLastNewsIndex..newNewsLastIndex).joinToString(", ")
    ),
    keywords = extractedKeywords,
    createdAt = LocalDateTime.now(),
    modifiedAt = crawledDateTime,
)
persistenceTargetNewsCards.add(persistenceNewsCard)
keywordsCountingPair += countKeyword(keywordsCountingPair, extractedKeywords)
```

### MyBatis로 Bulk Insert 처리하기

음.. 개인적으로 기록하고싶지도 않은 경험이였다. JPA만 써보다가 처음 MyBatis를 써봤는데 설정해줘야할 것도 많고 테스트 로직만 만드는데 한참이 걸렸었다.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE configuration PUBLIC "-//mybatis.org//DTD Config 3.0//EN" "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
    <settings>
        <setting name="mapUnderscoreToCamelCase" value="true"/>
    </settings>
    <environments default="development">
        <environment id="development">
            <transactionManager type="JDBC"/>
            <dataSource type="POOLED">
                <property name="driver" value="com.mysql.cj.jdbc.Driver"/>
                <property name="url" value="jdbc:mysql://localhost:3306/shorts"/>
                <property name="username" value="root"/>
                <property name="password" value="112233"/>
            </dataSource>
        </environment>
    </environments>
</configuration>
```

```kotlin

@Repository
class TestDomainMyBatisRepository {

    fun insert(testDomains: List<TestDomain>) {
        val dataSource = getDataSource()
        val sqlSessionFactory = getSqlSessionFactory(dataSource)
        val sqlSession = sqlSessionFactory.openSession(ExecutorType.BATCH)

        val testDomainMapper = sqlSession.getMapper(TestDomainMyBatisMapper::class.java)

        testDomainMapper.bulkInsert(testDomains)
        sqlSession.flushStatements()
        sqlSession.commit()
        sqlSession.close()
    }

    private fun getDataSource(): DataSource {
        val dataSource = PooledDataSource()
        dataSource.driver = "com.mysql.cj.jdbc.Driver"
        dataSource.url = "jdbc:mysql://localhost:3306/shorts"
        dataSource.username = "root"
        dataSource.password = "112233"
        return dataSource
    }

    private fun getSqlSessionFactory(dataSource: DataSource): SqlSessionFactory {
        val configuration = Configuration()

        // dataSource 설정 추가
        configuration.environment = Environment("development", JdbcTransactionFactory(), dataSource)

        // 매퍼 등의 설정 추가
        configuration.addMapper(TestDomainMyBatisMapper::class.java)

        val sessionFactoryBuilder = SqlSessionFactoryBuilder()
        return sessionFactoryBuilder.build(configuration)
    }
}

interface TestDomainMyBatisMapper {

    @Insert(
        """
        <script>
        INSERT INTO test_domain (title, content, news_link, press, thumbnail_image_url, type, written_date_time, crawled_count, category_id, created_at, modified_at)
        VALUES
        <foreach collection="testDomains" item="test_domain" separator=",">
            (#{test_domain.title},
            #{test_domain.content},
            #{test_domain.newsLink},
            #{test_domain.press},
            #{test_domain.thumbnailImageUrl},
            #{test_domain.type},
            #{test_domain.writtenDateTime},
            #{test_domain.crawledCount},
            #{test_domain.categoryId},
            #{test_domain.createdAt},
            #{test_domain.modifiedAt} )
        </foreach>
        </script>
        """
    )
    fun bulkInsert(@Param("testDomains") testDomains: List<TestDomain>)
}


@Table(name = "news")
@Entity
class TestDomain(

        @Column(columnDefinition = "TEXT", name = "title", nullable = false)
        var title: String,

        @Column(columnDefinition = "TEXT", name = "content", nullable = false)
        var content: String,

        @Column(columnDefinition = "TEXT", name = "thumbnail_image_url", nullable = false)
        var thumbnailImageUrl: String,

        @Column(columnDefinition = "TEXT", name = "news_link", nullable = false)
        var newsLink: String,

        @Column(name = "press", nullable = false, length = 20)
        var press: String, // 언론사

        @Column(name = "written_date_time", nullable = false, length = 30)
        var writtenDateTime: String,

        @Column(name = "type", nullable = false, length = 10)
        var type: String,

        @Column(name = "crawled_count", nullable = false)
        var crawledCount: Int,

        @ManyToOne(fetch = FetchType.LAZY)
        @JoinColumn(name = "category_id", nullable = false)
        var category: Category,
    ) : BaseEntity() {

        // Getter method for 'categoryId'
    fun getCategoryId(): Long {
        return category.id
    }
}


    @Test
    @DisplayName("Mybatis Bulk Insert 메서드 수행시간")
    fun `Mybatis Bulk Insert`() {
        val allNews = newsRepository.findAll()
        val startTime = System.currentTimeMillis()
        val persistenceTarget = mutableListOf<TestDomain>()
        for (news in allNews) {
            persistenceTarget.add(
                TestDomain(
                    title = news.title,
                    content = news.content,
                    thumbnailImageUrl = news.thumbnailImageUrl,
                    newsLink = news.newsLink,
                    press = news.press,
                    writtenDateTime = news.writtenDateTime,
                    type = news.type,
                    crawledCount = news.crawledCount,
                    category = news.category,
                )
            )
        }
        testDomainMyBatisRepository.insert(persistenceTarget)
        val endTime = System.currentTimeMillis()
        println("${endTime - startTime} ms")
    }

출력 결과 : 2912 ms
```

카멜 케이스, 스네이크 케이스에 대한 처리를 따로 해줘야했고 관계 매핑된 컬럼에는 커스텀 getter를 만들어줘야하는 등... 꽤나 많은 에러가 났음에도

성능차이는 jdbcTemplate과 큰 차이는 없었다.

어느정도 고도화 된 프로젝트에서는 고려해볼만 하겠지만 생산성이 중요한 소규모 프로젝트에서는 그리 좋아보이진 않다.

일단 SQL Mapper 구문이 너무 어색하고 불편하다. 런타임 에러가 잦게 발생한다는 단점이 있다.

---

## 결론

이렇게 Bulk INSERT를 처리하는 세 가지 방법 (JPA, JdbcTemplate, MyBatis)를 알아보았다.

확실한건 JPA가 제공하는 기본 메서드인 save()는 대량 INSERT에 적합하지 않다는 것을 느꼈고

MyBatis는 성능이 가장 우수했지만 생산성이 매우 뒤떨어 진다는 것을 느꼈다.

프로젝트 규모와 개발 환경에 따라 다르겠지만 내가 진행하는 프로젝트만큼에서는 성능과 생산성의 타협점인 JdbcTemplate을 사용하는 것을 고려하는 것이 꽤 적절해보인다.
