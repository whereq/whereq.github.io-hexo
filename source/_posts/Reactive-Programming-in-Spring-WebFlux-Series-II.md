---
title: Reactive Programming in Spring WebFlux Series - II
date: 2024-10-24 13:10:13
categories:
- WebFlux
- Spring Boot
- Reactive
tags:
- WebFlux
- Spring Boot
- Reactive
---

## Index
- [In-depth Understanding of Reactor and Asynchronous Data Streams](#in-depth-understanding-of-reactor-and-asynchronous-data-streams)
  - [Introduction to Reactor](#introduction-to-reactor)
  - [Mono and Flux](#mono-and-flux)
    - [Mono](#mono)
    - [Flux](#flux)
  - [Basic Operations](#basic-operations)
  - [Practical Example: Building a Reactive Application with Reactor](#practical-example-building-a-reactive-application-with-reactor)
    - [1. Project Setup](#1-project-setup)
    - [2. Define Data Model](#2-define-data-model)
    - [3. Simulated Data Source](#3-simulated-data-source)
    - [4. Handlers and Router Configuration](#4-handlers-and-router-configuration)
  - [Conclusion](#conclusion)

---
# In-depth Understanding of Reactor and Asynchronous Data Streams

---
<a name="introduction-to-reactor"></a> 
## Introduction to Reactor

Reactor is at the core of Spring WebFlux, providing powerful reactive programming support based on the Reactive Streams standard. Reactor mainly offers two core classes: **Mono** and **Flux**, which are used to represent asynchronous sequences of one or multiple elements, respectively.

---

<a name="mono-and-flux"></a>
## Mono and Flux

<a name="mono"></a>
### Mono
Mono represents an asynchronous sequence of 0 or 1 element. It is typically used when dealing with operations that might return a single result or no result at all.

<a name="flux"></a>
### Flux
Flux represents an asynchronous sequence of 0 to N elements. It is suitable for cases where multiple results need to be handled, such as querying a database that returns multiple records.

---

<a name="basic-operations"></a>
## Basic Operations

Reactor provides a rich set of operators for manipulating data streams, such as transformations, filtering, and aggregations. Some of the commonly used operators include:

- **map**: Transforms each element in the data stream.
- **filter**: Filters elements in the data stream, retaining only those that meet the criteria.
- **flatMap**: Maps each element to a new Publisher and merges these Publishers into a single Flux.
- **concatMap**: Similar to flatMap but maintains the order of the elements.
- **zip**: Combines multiple Publishers by pairing elements based on their indices.

---

<a name="practical-example-building-a-reactive-application-with-reactor"></a>
## Practical Example: Building a Reactive Application with Reactor

<a name="1-project-setup"></a>
### 1. Project Setup

Ensure your project contains the following dependencies:

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-webflux</artifactId>
    </dependency>
    <dependency>
        <groupId>io.projectreactor</groupId>
        <artifactId>reactor-core</artifactId>
    </dependency>
</dependencies>
```

<a name="2-define-data-model"></a>
### 2. Define Data Model

Define a simple user data model:

```java
public class User {
    private String id;
    private String name;

    // Constructors, Getters and Setters
}
```

<a name="3-simulated-data-source"></a>
### 3. Simulated Data Source

Create a simulated data source to generate user data:

```java
import reactor.core.publisher.Flux;
import reactor.core.publisher.Mono;

import java.util.Arrays;
import java.util.List;

public class UserRepository {
    private static final List<User> users = Arrays.asList(
        new User("1", "John Doe"),
        new User("2", "Jane Doe"),
        new User("3", "Sam Smith")
    );

    public Mono<User> findById(String id) {
        return Mono.justOrEmpty(users.stream()
            .filter(user -> user.getId().equals(id))
            .findFirst());
    }

    public Flux<User> findAll() {
        return Flux.fromIterable(users);
    }
}
```

<a name="4-handlers-and-router-configuration"></a>
### 4. Handlers and Router Configuration

Create handler functions and router configurations to handle HTTP requests and return responses:

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
        String userId = request.pathVariable("id");
        return userRepository.findById(userId)
            .flatMap(user -> ServerResponse.ok().body(Mono.just(user), User.class))
            .switchIfEmpty(ServerResponse.notFound().build());
    }

    public Mono<ServerResponse> getAllUsers(ServerRequest request) {
        return ServerResponse.ok().body(userRepository.findAll(), User.class);
    }
}

@Configuration
public class RouterConfig {
    @Bean
    public RouterFunction<ServerResponse> route(UserHandler handler) {
        return RouterFunctions.route()
            .GET("/user/{id}", handler::getUser)
            .GET("/users", handler::getAllUsers)
            .build();
    }
}
```

---

<a name="conclusion"></a> 
## Conclusion

In this article, we took a deep dive into Reactor and its application in Spring WebFlux. By understanding **Mono** and **Flux** and how to handle asynchronous data streams with these tools, we can build more efficient reactive applications.
