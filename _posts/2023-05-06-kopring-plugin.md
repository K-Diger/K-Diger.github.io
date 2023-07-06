---

title: 멀티모듈 코프링과 플러그인

date: 2023-05-06
categories: [Spring, Toby]
tags: [Spring, Toby]
layout: post
toc: true
math: true
mermaid: true

---

# 루트.build.gradle.kts

```groovy
import org.jetbrains.kotlin.gradle.tasks.KotlinCompile

plugins {
	id("org.springframework.boot") version "3.0.5"
	id("io.spring.dependency-management") version "1.1.0"
	id("org.asciidoctor.jvm.convert") version "3.3.2"
	kotlin("jvm") version "1.7.22"
	kotlin("plugin.spring") version "1.7.22"
	kotlin("plugin.jpa") version "1.7.22"
	kotlin("kapt") version "1.6.21"
}

java.sourceCompatibility = JavaVersion.VERSION_17

allprojects {
	group = "com.mashup"
	version = "0.0.1-SNAPSHOT"

	repositories {
		mavenCentral()
	}
}

subprojects {
	apply(plugin = "org.jetbrains.kotlin.jvm")
	apply(plugin = "org.jetbrains.kotlin.plugin.spring")
	apply(plugin = "org.springframework.boot")
	apply(plugin = "kotlin")
	apply(plugin = "java-library")
	apply(plugin = "kotlin-jpa")
	apply(plugin = "io.spring.dependency-management")
	apply(plugin = "org.asciidoctor.jvm.convert")
	apply(plugin = "kotlin-kapt")

	dependencies {
		implementation("org.springframework.boot:spring-boot-starter-web")
		implementation("com.fasterxml.jackson.module:jackson-module-kotlin")
		implementation("io.github.microutils:kotlin-logging:2.0.8")

		implementation("org.jetbrains.kotlin:kotlin-reflect")
		testImplementation("org.springframework.boot:spring-boot-starter-test")
	}

	tasks.withType<KotlinCompile> {
		kotlinOptions {
			freeCompilerArgs = listOf("-Xjsr305=strict")
			jvmTarget = "17"
		}
	}

	tasks.withType<Test> {
		useJUnitPlatform()
	}
}
```

## plugins 코드블럭

```groovy
plugins {

    // 스프링 부트 프레임워크를 사용할 때 필요한 플러그인
    // 이 플러그인을 적용하면 bootRun, bootJar, bootWar와 같은 스프링 부트 관련 플러그인도 사용할 수 있다.
    id("org.springframework.boot") version "3.0.5"

    // 스프링 부트의 종속성 관리를 쉽게 하기 위한 플러그인
    // 이 플러그인을 적용하면 스프링 부트 버전 관리와 관련된 설정이 자동으로 처리됨
    id("io.spring.dependency-management") version "1.1.0"

    // Asciidoctor와 관련된 플러그인입니다.
    // Asciidoctor는 AsciiDoc 문서를 HTML, PDF, EPUB, 등 다양한 형식으로 변환하는 데 사용됨
    id("org.asciidoctor.jvm.convert") version "3.3.2"

    // 코틀린 언어를 JVM에서 실행할 때 필요한 플러그인입니다.
    kotlin("jvm") version "1.7.22"

    // 스프링 프레임워크와 함께 코틀린 언어를 사용할 때 필요한 플러그인
    kotlin("plugin.spring") version "1.7.22"

    // JPA (Java Persistence API)와 함께 코틀린 언어를 사용할 때 필요한 플러그인
    kotlin("plugin.jpa") version "1.7.22"

    // 코틀린 언어에서 애노테이션 프로세서를 사용할 때 필요한 플러그인
    // 이 플러그인을 사용하면 코틀린으로 작성된 애노테이션 프로세서를 실행할 수 있습니다.
    kotlin("kapt") version "1.6.21"
}
```

## allProjects 코드블럭

```groovy
// 모든 프로젝트의 소스 코드와 컴파일러 버전을 Java 17로 설정하는 구문
// 즉, Java 17 문법을 사용하여 소스 코드를 작성하고, 컴파일러로는 Java 17을 사용한다는 뜻입니다.
java.sourceCompatibility = JavaVersion.VERSION_17

// 모든 프로젝트에 대한 설정을 적용하기 위한 블록입니다.
allprojects {
    // 모든 프로젝트의 그룹 ID를 "com.mashup"으로 설정하는 구문
    // 그룹 ID는 해당 프로젝트를 식별하는 데 사용되며, 일반적으로 회사 또는 조직의 도메인과 유사한 형태로 작성됨
	group = "com.mashup"

    // 모든 프로젝트의 버전을 "0.0.1-SNAPSHOT"으로 설정하는 구문
    // 버전은 소프트웨어의 릴리스 버전을 식별하는 데 사용되며 "-SNAPSHOT"은 개발 버전임을 나타내는 접미사임
	version = "0.0.1-SNAPSHOT"

    // 모든 프로젝트에서 Maven 중앙 저장소를 사용하도록 설정하는 구문
    // Maven 중앙 저장소는 대부분의 Java 라이브러리와 프레임워크가 호스팅되는 공식 저장소이다.
    // 따라서 해당 저장소에서 필요한 라이브러리를 다운로드하여 사용할 수 있다.
    repositories {
		mavenCentral()
	}
}
```

```groovy
// 모든 하위 프로젝트에 대한 설정을 적용하기 위한 블록
subprojects {
    // Kotlin 언어를 사용하여 JVM에서 실행될 수 있는 프로젝트를 작성할 때 필요한 플러그인
    apply(plugin = "org.jetbrains.kotlin.jvm")

    // Spring Framework와 함께 Kotlin을 사용할 때 필요한 플러그인
    apply(plugin = "org.jetbrains.kotlin.plugin.spring")

    // Spring Boot 프레임워크에서 사용하는 플러그인
    // Spring Boot 애플리케이션을 빌드하고 실행하기 위한 여러 기능들을 제공
    apply(plugin = "org.springframework.boot")

    // Kotlin 언어를 사용할 때 필요한 플러그인
	apply(plugin = "kotlin")

    // Java 라이브러리를 작성할 때 필요한 플러그인
	apply(plugin = "java-library")

    // Kotlin과 함께 Java Persistence API(JPA)를 사용할 때 필요한 플러그인
    apply(plugin = "kotlin-jpa")

    // Spring Boot에서 사용하는 라이브러리 의존성 관리를 자동화해주는 플러그인
    apply(plugin = "io.spring.dependency-management")

    // Asciidoctor와 함께 Kotlin을 사용할 때 필요한 플러그인
    apply(plugin = "org.asciidoctor.jvm.convert")

    // Kotlin에서 annotation processing을 사용할 때 필요한 플러그인
    apply(plugin = "kotlin-kapt")

	dependencies {
        // Spring Boot에서 웹 애플리케이션을 개발할 때 필요한 라이브러리
		implementation("org.springframework.boot:spring-boot-starter-web")

        // Kotlin에서 JSON 직렬화 및 역직렬화를 위한 Jackson 라이브러리 모듈
        implementation("com.fasterxml.jackson.module:jackson-module-kotlin")

        // Kotlin에서 로깅을 위한 라이브러리입니다.
		implementation("io.github.microutils:kotlin-logging:2.0.8")

        // Kotlin Reflection API를 사용하기 위한 라이브러리
		implementation("org.jetbrains.kotlin:kotlin-reflect")

        // Spring Boot에서 테스트를 위한 라이브러리
		testImplementation("org.springframework.boot:spring-boot-starter-test")
	}

    // Kotlin 컴파일 태스크에 대한 설정을 정의하는 구문
	tasks.withType<KotlinCompile> {
		kotlinOptions {
            // Kotlin 컴파일러에 전달할 옵션
			freeCompilerArgs = listOf("-Xjsr305=strict")
			jvmTarget = "17"
		}
	}

	tasks.withType<Test> {
		useJUnitPlatform()
	}
}
```

# api module

```groovy
val asciidoctorExtensions: Configuration by configurations.creating

tasks.test {
    outputs.dir("build/generated-snippets")
}

tasks.asciidoctor {
    inputs.dir("build/generated-snippets")
    dependsOn(tasks.test)
    configurations(asciidoctorExtensions.name)
    baseDirFollowsSourceFile()
}

tasks.register("restDocs") {
    dependsOn("asciidoctor")
    doLast {
        copy {
            from(file("$buildDir/asciidoc/html5"))
            into(file("docs"))
        }
    }
}


dependencies {
    // shorts-domain 이라는 모듈을 의존한다. (의존을 통해 해당 모듈에 있는 도메인들을 사용할 수 있게 되는 것)
    implementation(project(":shorts-domain"))

    testImplementation("org.springframework.restdocs:spring-restdocs-mockmvc")
    asciidoctorExtensions("org.springframework.restdocs:spring-restdocs-asciidoctor")
    testImplementation("com.ninja-squad:springmockk:3.1.1")
}
```

