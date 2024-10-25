---
title: Reactive Programming in Spring WebFlux Series - III
date: 2024-10-24 13:57:04
categories:
- WebFlux
- Spring Boot
- Reactive
tags:
- WebFlux
- Spring Boot
- Reactive
---

- [Error Handling and Debugging](#error-handling-and-debugging)
  - [Error Handling](#error-handling)
    - [1. onErrorReturn](#1-onerrorreturn)
    - [2. onErrorResume](#2-onerrorresume)
    - [3. onErrorMap](#3-onerrormap)
    - [Global Error Handling in WebFlux](#global-error-handling-in-webflux)
    - [1. Defining a Custom Exception Class](#1-defining-a-custom-exception-class)
    - [2. Creating an Exception Handler](#2-creating-an-exception-handler)
    - [Debugging Reactive Applications](#debugging-reactive-applications)
    - [1. Using the log Operator](#1-using-the-log-operator)
    - [2. Breakpoint Debugging](#2-breakpoint-debugging)
    - [ 3. Using Reactor Debugger](#-3-using-reactor-debugger)
  - [Conclusion](#conclusion)

---

# Error Handling and Debugging
---

<a name="error-handling"></a> 
## Error Handling

Error handling in reactive programming is different from traditional synchronous programming due to the asynchronous nature of reactive streams. In Reactor, errors are handled in various ways:

<a name="onerrorreturn"></a> 
### 1. onErrorReturn

This operator allows returning a default value when an error occurs. Example:

```java
Flux<String> flux = Flux.just("A", "B", "C")
    .concatWith(Flux.error(new RuntimeException("Exception occurred")))
    .onErrorReturn("Default");

flux.subscribe(System.out::println);
```

In this example, when an error occurs, the default value `"Default"` is returned.

<a name="onerrorresume"></a>
### 2. onErrorResume

This operator allows switching to another `Publisher` when an error occurs. Example:

```java
Flux<String> flux = Flux.just("A", "B", "C")
    .concatWith(Flux.error(new RuntimeException("Exception occurred")))
    .onErrorResume(e -> {
        System.out.println("Error: " + e.getMessage());
        return Flux.just("D", "E");
    });

flux.subscribe(System.out::println);
```

Here, when an error occurs, it switches to another `Flux` sequence to continue processing new data.

<a name="onerrormap"></a>
### 3. onErrorMap

This operator allows converting an error into a different type of error. Example:

```java
Flux<String> flux = Flux.just("A", "B", "C")
    .concatWith(Flux.error(new RuntimeException("Exception occurred")))
    .onErrorMap(e -> new CustomException("Custom message", e));

flux.subscribe(System.out::println, e -> System.out.println("Error: " + e.getMessage()));
```

In this case, a `RuntimeException` is converted into a custom `CustomException`.

---

<a name="global-error-handling-in-webflux"></a>
### Global Error Handling in WebFlux

In addition to handling errors in streams, Spring WebFlux also allows defining a global error handler for managing errors across the application.

<a name="defining-a-custom-exception-class"></a>
### 1. Defining a Custom Exception Class

```java
public class CustomException extends RuntimeException {
    public CustomException(String message) {
        super(message);
    }
}
```

<a name="creating-an-exception-handler"></a>
### 2. Creating an Exception Handler

```java
import org.springframework.core.annotation.Order;
import org.springframework.http.HttpStatus;
import org.springframework.http.MediaType;
import org.springframework.web.reactive.function.server.ServerRequest;
import org.springframework.web.reactive.function.server.ServerResponse;
import org.springframework.web.reactive.function.server.ServerResponse.BodyBuilder;
import org.springframework.web.server.ServerWebExchange;
import org.springframework.web.server.WebExceptionHandler;
import reactor.core.publisher.Mono;

@Component
@Order(-2)
public class GlobalErrorHandler implements WebExceptionHandler {

    @Override
    public Mono<Void> handle(ServerWebExchange exchange, Throwable ex) {
        BodyBuilder response = ServerResponse.status(HttpStatus.INTERNAL_SERVER_ERROR)
                .contentType(MediaType.APPLICATION_JSON);

        if (ex instanceof CustomException) {
            response = ServerResponse.status(HttpStatus.BAD_REQUEST);
        }

        return response.bodyValue(new ErrorResponse(ex.getMessage()))
                .flatMap(resp -> resp.writeTo(exchange, new Context()));
    }

    private static class ErrorResponse {
        private final String message;

        public ErrorResponse(String message) {
            this.message = message;
        }

        public String getMessage() {
            return message;
        }
    }
}
```

---

<a name="debugging-reactive-applications"></a>
### Debugging Reactive Applications

Debugging reactive applications can be more challenging than debugging traditional synchronous applications. Here are some debugging techniques:

<a name="using-the-log-operator"></a>
### 1. Using the log Operator

Reactor provides the `log` operator to output log information for each step. Example:

```java
Flux<String> flux = Flux.just("A", "B", "C")
    .concatWith(Flux.error(new RuntimeException("Exception occurred")))
    .log();

flux.subscribe(System.out::println);
```

<a name="breakpoint-debugging"></a>
### 2. Breakpoint Debugging

Setting breakpoints in an IDE allows you to step through the execution. Be mindful of thread switching and asynchronous execution details when debugging reactive code.

### <a name="using-reactor-debugger"></a> 3. Using Reactor Debugger
Reactor provides a special debug tool `Hooks.onOperatorDebug` to help understand and debug operations in reactive streams.

```java
Hooks.onOperatorDebug();

Flux<String> flux = Flux.just("A", "B", "C")
    .concatWith(Flux.error(new RuntimeException("Exception occurred")));

flux.subscribe(System.out::println);
```

---

<a name="conclusion"></a>
## Conclusion

In this article, we explored how to handle errors and debug reactive applications in Spring WebFlux, providing techniques for both stream-level and global error handling.
