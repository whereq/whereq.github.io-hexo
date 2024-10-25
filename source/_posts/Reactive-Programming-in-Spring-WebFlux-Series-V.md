---
title: Reactive Programming in Spring WebFlux Series - V
date: 2024-10-24 14:06:39
categories:
- WebFlux
- Spring Boot
- Reactive
tags:
- WebFlux
- Spring Boot
- Reactive
---

- [Reactive Database Access](#reactive-database-access)
  - [Why Choose Reactive Database Access?](#why-choose-reactive-database-access)
  - [Spring Data R2DBC](#spring-data-r2dbc)
    - [1. Project Setup](#1-project-setup)
    - [2. Database Connection Configuration](#2-database-connection-configuration)
    - [ 3. Data Model](#-3-data-model)
    - [4. Creating a Repository](#4-creating-a-repository)
    - [5. Data Initialization](#5-data-initialization)
    - [6. Handler Functions and Route Configuration](#6-handler-functions-and-route-configuration)
  - [Conclusion](#conclusion)

---

# Reactive Database Access

---

<a name="why-choose-reactive-database-access"></a>
## Why Choose Reactive Database Access?

Traditional database access methods are usually blocking, meaning that when a query is executing, the thread is blocked until the query is completed. This can lead to performance bottlenecks in highly concurrent environments. Reactive database access uses non-blocking I/O operations, allowing applications to handle more concurrent requests, improving system throughput and response times.

---

<a name="spring-data-r2dbc"></a>
## Spring Data R2DBC

Spring Data R2DBC (Reactive Relational Database Connectivity) is a project in the Spring ecosystem that provides reactive programming support for relational databases. R2DBC is a new database connection specification designed to support reactive programming models.

<a name="project-setup"></a>
### 1. Project Setup

First, create a Spring Boot project and include the necessary dependencies:

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-webflux</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-data-r2dbc</artifactId>
    </dependency>
    <dependency>
        <groupId>io.r2dbc</groupId>
        <artifactId>r2dbc-h2</artifactId>
    </dependency>
</dependencies>
```

<a name="database-connection-configuration"></a>
### 2. Database Connection Configuration

Configure the H2 database in `application.properties`:

```properties
spring.r2dbc.url=r2dbc:h2:mem:///testdb
spring.r2dbc.username=sa
spring.r2dbc.password=
spring.r2dbc.generate-unique-name=true

spring.h2.console.enabled=true
spring.h2.console.settings.web-allow-others=true
```

### <a name="data-model"></a> 3. Data Model

Define a simple `User` entity:

```java
import org.springframework.data.annotation.Id;
import org.springframework.data.relational.core.mapping.Table;

@Table("users")
public class User {
    @Id
    private Long id;
    private String name;

    // Getters and Setters
}
```

<a name="creating-a-repository"></a>
### 4. Creating a Repository

Create a reactive `UserRepository` interface:

```java
import org.springframework.data.repository.reactive.ReactiveCrudRepository;

public interface UserRepository extends ReactiveCrudRepository<User, Long> {
}
```

<a name="data-initialization"></a>
### 5. Data Initialization

Use an initializer class to preload some data:

```java
import org.springframework.boot.CommandLineRunner;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

import reactor.core.publisher.Flux;

@Configuration
public class DataInitializer {

    @Bean
    CommandLineRunner init(UserRepository repository) {
        return args -> {
            Flux<User> userFlux = Flux.just(
                    new User(null, "John Doe"),
                    new User(null, "Jane Doe"),
                    new User(null, "Sam Smith"))
                .flatMap(repository::save);

            userFlux
                .thenMany(repository.findAll())
                .subscribe(user -> System.out.println("Saving " + user.toString()));
        };
    }
}
```

<a name="handler-functions-and-route-configuration"></a>
### 6. Handler Functions and Route Configuration

Create handler functions and route configurations to handle HTTP requests and return responses:

```java
import org.springframework.stereotype.Component;
import org.springframework.web.reactive.function.server.RouterFunction;
import org.springframework.web.reactive.function.server.ServerRequest;
import org.springframework.web.reactive.function.server.ServerResponse;
import org.springframework.web.reactive.function.server.RouterFunctions;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

import reactor.core.publisher.Mono;

@Component
public class UserHandler {
    private final UserRepository userRepository;

    public UserHandler(UserRepository userRepository) {
        this.userRepository = userRepository;
    }

    public Mono<ServerResponse> getUser(ServerRequest request) {
        Long userId = Long.valueOf(request.pathVariable("id"));
        return userRepository.findById(userId)
            .flatMap(user -> ServerResponse.ok().body(Mono.just(user), User.class))
            .switchIfEmpty(ServerResponse.notFound().build());
    }

    public Mono<ServerResponse> getAllUsers(ServerRequest request) {
        return ServerResponse.ok().body(userRepository.findAll(), User.class);
    }

    public Mono<ServerResponse> createUser(ServerRequest request) {
        Mono<User> userMono = request.bodyToMono(User.class);
        return userMono.flatMap(userRepository::save)
            .flatMap(user -> ServerResponse.ok().body(Mono.just(user), User.class));
    }
}

@Configuration
public class RouterConfig {
    @Bean
    public RouterFunction<ServerResponse> route(UserHandler handler) {
        return RouterFunctions.route()
            .GET("/user/{id}", handler::getUser)
            .GET("/users", handler::getAllUsers)
            .POST("/user", handler::createUser)
            .build();
    }
}
```

---

<a name="conclusion"></a>
## Conclusion

In this article, we explored how to perform reactive database access in Spring WebFlux. With Spring Data R2DBC, we can easily achieve non-blocking access to relational databases, greatly improving the performance and concurrency handling of applications.
