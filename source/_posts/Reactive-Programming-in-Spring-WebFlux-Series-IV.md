---
title: Reactive Programming in Spring WebFlux Series - IV
date: 2024-10-24 14:06:07
categories:
- WebFlux
- Spring Boot
- Reactive
tags:
- WebFlux
- Spring Boot
- Reactive
---


- [Data Stream Transformation and Operations](#data-stream-transformation-and-operations)
  - [Data Stream Transformation](#data-stream-transformation)
    - [1. map Operator](#1-map-operator)
    - [2. flatMap Operator](#2-flatmap-operator)
    - [3. filter Operator](#3-filter-operator)
    - [4. reduce Operator](#4-reduce-operator)
  - [Practical Example: Building a Complex Data Stream Processing Application](#practical-example-building-a-complex-data-stream-processing-application)
    - [1. Project Setup](#1-project-setup)
    - [2. Data Model](#2-data-model)
    - [3. Mock Data Source](#3-mock-data-source)
    - [4. Handler Functions and Route Configuration](#4-handler-functions-and-route-configuration)
  - [Conclusion](#conclusion)

---

# Data Stream Transformation and Operations

---

<a name="data-stream-transformation"></a> 
## Data Stream Transformation

In reactive programming, transforming and operating on data streams is a core aspect. Using different operators, we can handle data streams flexibly, applying transformations, filtering, aggregating, and more. Below are some common data stream operations and examples.

<a name="map-operator"></a>
### 1. map Operator

The `map` operator transforms each element in a data stream. It takes a function as an argument and applies it to each element.

```java
Flux<Integer> numbers = Flux.just(1, 2, 3, 4, 5)
    .map(n -> n * n);

numbers.subscribe(System.out::println); // Output: 1, 4, 9, 16, 25
```

<a name="flatmap-operator"></a>
### 2. flatMap Operator

The `flatMap` operator maps each element to a new `Publisher` and merges them into a single `Flux`. It is often used for asynchronous processing.

```java
Flux<String> ids = Flux.just("1", "2", "3");
Flux<User> users = ids.flatMap(id -> userRepository.findById(id));

users.subscribe(System.out::println);
```

<a name="filter-operator"></a>
### 3. filter Operator

The `filter` operator filters data streams, keeping only elements that meet a certain condition.

```java
Flux<Integer> numbers = Flux.just(1, 2, 3, 4, 5)
    .filter(n -> n % 2 == 0);

numbers.subscribe(System.out::println); // Output: 2, 4
```

<a name="reduce-operator"></a>
### 4. reduce Operator

The `reduce` operator aggregates all elements in a data stream into a single value. It takes an accumulator function, combining the previous result with the current element.

```java
Flux<Integer> numbers = Flux.just(1, 2, 3, 4, 5);
Mono<Integer> sum = numbers.reduce((a, b) -> a + b);

sum.subscribe(System.out::println); // Output: 15
```

---

<a name="practical-example-building-a-complex-data-stream-processing-application"></a>
## Practical Example: Building a Complex Data Stream Processing Application

Below is an example showing how to use these operators to build a complex reactive application in Spring WebFlux.

<a name="project-setup"></a>
### 1. Project Setup

Ensure that the following dependencies are included in your project:

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

<a name="data-model"></a>
### 2. Data Model

Define a simple user data model:

```java
public class User {
    private String id;
    private String name;

    // Constructors, Getters and Setters
}
```

<a name="mock-data-source"></a>
### 3. Mock Data Source

Create a mock data source:

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

<a name="handler-functions-and-route-configuration"></a>
### 4. Handler Functions and Route Configuration

Set up handler functions and route configuration:

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

In this article, we explored how to transform and operate on data streams in Spring WebFlux. Using different operators, we can flexibly handle data streams, building complex reactive applications.
