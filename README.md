# Where I stopped into Philipp's video

<img src="screenshots/Screenshot 2025-04-16 at 15.11.51.png" alt="Where I stopped into Philipp's video" width="1008" height="655">

# Environment variables inside IntellijIDEA

<img src="screenshots/Screenshot 2025-04-16 at 15.26.01.png" alt="Environment variables inside IntellijIDEA" width="1008" height="655">

# Postman replies

<img src="screenshots/Screenshot 2025-04-16 at 15.26.50.png" alt="Postman replies - Part 1" width="1008" height="655">
<img src="screenshots/Screenshot 2025-04-16 at 15.26.59.png" alt="Postman replies - Part 2" width="1008" height="655">
<img src="screenshots/Screenshot 2025-04-16 at 15.27.40.png" alt="Postman replies - Part 3" width="1008" height="655">

# Gist to the Log

[Here](https://gist.github.com/graffiti75/84be2ed37759c7e6c517eb71016ebfb9) is the Gist showing the Log when I try to run the SpringBootCrashCourse app

# Note.kt

```
package com.cericatto.spring_boot_crash_course.database.model

import org.bson.types.ObjectId
import org.springframework.data.annotation.Id
import org.springframework.data.mongodb.core.mapping.Document
import java.time.Instant

@Document("notes")
data class Note(
    val title: String,
    val content: String,
    val color: Long,
    val createdAt: Instant,
    val ownerId: ObjectId,
    @Id val id: ObjectId = ObjectId.get()
)
```

# NoteRepository.kt
```
package com.cericatto.spring_boot_crash_course.database.repository

import com.cericatto.spring_boot_crash_course.database.model.Note
import org.bson.types.ObjectId
import org.springframework.data.mongodb.repository.MongoRepository

interface NoteRepository: MongoRepository<Note, ObjectId> {
    fun findByOwnerId(ownerId: ObjectId): List<Note>
}
```

# NoteController.kt
```
package com.cericatto.spring_boot_crash_course.controllers

import com.cericatto.spring_boot_crash_course.controllers.NoteController.NoteResponse
import com.cericatto.spring_boot_crash_course.database.model.Note
import com.cericatto.spring_boot_crash_course.database.repository.NoteRepository
import org.bson.types.ObjectId
import org.springframework.web.bind.annotation.GetMapping
import org.springframework.web.bind.annotation.PostMapping
import org.springframework.web.bind.annotation.RequestBody
import org.springframework.web.bind.annotation.RequestMapping
import org.springframework.web.bind.annotation.RequestParam
import org.springframework.web.bind.annotation.RestController
import java.time.Instant

@RestController
@RequestMapping("/notes")
class NoteController(
    private val repository: NoteRepository
) {

    data class NoteRequest(
        val id: String?,
        val title: String,
        val content: String,
        val color: Long
    )

    data class NoteResponse(
        val id: String,
        val title: String,
        val content: String,
        val color: Long,
        val createdAt: Instant
    )

    @PostMapping
    fun save(
        @RequestBody body: NoteRequest
    ): NoteResponse {
        val note = repository.save(
            Note(
                id = body.id?.let { ObjectId(it) } ?: ObjectId.get(),
                title = body.title,
                content = body.content,
                color = body.color,
                createdAt = Instant.now(),
                ownerId = ObjectId()
            )
        )

        return note.toResponse()
    }

    @GetMapping
    fun findByOwnerId(
        @RequestParam(required = true) ownerId: String
    ): List<NoteResponse> {
        return repository.findByOwnerId(ObjectId(ownerId)).map {
            it.toResponse()
        }
    }
}

private fun Note.toResponse(): NoteController.NoteResponse {
    return NoteResponse(
        id = id.toHexString(),
        title = title,
        content = content,
        color = color,
        createdAt = createdAt
    )
}
```

# application.properties
```
spring.application.name=spring_boot_crash_course
server.port=8086
spring.data.mongodb.uri=${MONGODB_CONNECTION_STRING}
spring.security.user.password=test
```

# SecurityConfig.kt
```
package com.cericatto.spring_boot_crash_course.security

import org.springframework.context.annotation.Bean
import org.springframework.context.annotation.Configuration
import org.springframework.security.config.annotation.web.builders.HttpSecurity
import org.springframework.security.config.http.SessionCreationPolicy
import org.springframework.security.web.SecurityFilterChain

@Configuration
class SecurityConfig {

    @Bean
    fun filterChain(httpSecurity: HttpSecurity): SecurityFilterChain {
        return httpSecurity
            .csrf { csrf -> csrf.disable() }
            .sessionManagement {
                it.sessionCreationPolicy(
                    SessionCreationPolicy.STATELESS
                )
            }
            .build()
    }
}
```

# build.gradle.kts

```
plugins {
	kotlin("jvm") version "1.9.25"
	kotlin("plugin.spring") version "1.9.25"
	id("org.springframework.boot") version "3.4.4"
	id("io.spring.dependency-management") version "1.1.7"
}

group = "com.cericatto"
version = "0.0.1-SNAPSHOT"

java {
	toolchain {
		languageVersion = JavaLanguageVersion.of(17)
	}
}

repositories {
	mavenCentral()
}

dependencies {
	implementation("org.springframework.boot:spring-boot-starter-web")
	implementation("org.springframework.boot:spring-boot-starter-data-mongodb")
	implementation("org.springframework.boot:spring-boot-starter-data-mongodb-reactive")
	implementation("org.springframework.boot:spring-boot-starter-security")
	implementation("org.springframework.security:spring-security-crypto")
	implementation("org.springframework.boot:spring-boot-starter-validation")
	implementation("io.projectreactor.kotlin:reactor-kotlin-extensions")
	implementation("org.jetbrains.kotlin:kotlin-reflect")
	implementation("org.jetbrains.kotlinx:kotlinx-coroutines-reactor")
	testImplementation("org.springframework.boot:spring-boot-starter-test")
	testImplementation("io.projectreactor:reactor-test")
	testImplementation("org.jetbrains.kotlin:kotlin-test-junit5")
	testImplementation("org.jetbrains.kotlinx:kotlinx-coroutines-test")
	testImplementation("org.springframework.security:spring-security-test")
	testRuntimeOnly("org.junit.platform:junit-platform-launcher")

	compileOnly("jakarta.servlet:jakarta.servlet-api:6.1.0")
	implementation("io.jsonwebtoken:jjwt-api:0.12.6")
	runtimeOnly("io.jsonwebtoken:jjwt-impl:0.12.6")
	runtimeOnly("io.jsonwebtoken:jjwt-jackson:0.12.6")
}

kotlin {
	compilerOptions {
		freeCompilerArgs.addAll("-Xjsr305=strict")
	}
}

tasks.withType<Test> {
	useJUnitPlatform()
}
```

# SpringBootCrashCourseApplication.kt
```
package com.cericatto.spring_boot_crash_course

import org.springframework.boot.autoconfigure.SpringBootApplication
import org.springframework.boot.runApplication

@SpringBootApplication
class SpringBootCrashCourseApplication

fun main(args: Array<String>) {
	runApplication<SpringBootCrashCourseApplication>(*args)
}
```
