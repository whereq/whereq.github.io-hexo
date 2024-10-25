---
title: Reactive Programming in Spring WebFlux Series - XVII
date: 2024-10-24 14:07:44
categories:
- WebFlux
- Spring Boot
- Reactive
tags:
- WebFlux
- Spring Boot
- Reactive
---


In the previous sixteen articles, we have covered the foundational concepts of Spring WebFlux, Reactor, data stream transformation, reactive database access, performance optimization, security, testing, deployment and operations, best practices, common pitfalls, integration with microservices architecture, and real-time data push. This article will delve into implementing effective error handling strategies in Spring WebFlux applications to ensure system robustness and user experience.

- [Error Handling Strategies](#error-handling-strategies)
  - [Importance of Error Handling](#importance-of-error-handling)
  - [Basic Concepts of Error Handling](#basic-concepts-of-error-handling)
  - [Global Error Handling](#global-error-handling)
    - [Creating a Global Error Handler](#creating-a-global-error-handler)
    - [Configuration Class Explanation](#configuration-class-explanation)
  - [Controller-Level Error Handling](#controller-level-error-handling)
  - [Error Handling in Reactive Streams](#error-handling-in-reactive-streams)
    - [Using `onErrorResume`](#using-onerrorresume)
    - [Using `onErrorReturn`](#using-onerrorreturn)
    - [Using `onErrorMap`](#using-onerrormap)
  - [Conclusion](#conclusion)

---

# Error Handling Strategies

---
<a name="importance-of-error-handling"></a>
## Importance of Error Handling

In any application, errors are inevitable. How these errors are handled not only affects system stability but also directly impacts user experience. In Spring WebFlux, due to its asynchronous and non-blocking nature, error handling is particularly crucial.

<a name="basic-concepts-of-error-handling"></a>
## Basic Concepts of Error Handling

In Spring WebFlux, error handling typically involves the following levels:

- **Global Error Handling**: Captures unhandled exceptions in the application and provides a unified error response.
- **Controller-Level Error Handling**: Handles errors specific to requests in controllers.
- **Error Handling in Reactive Streams**: Manages and recovers from errors within reactive data streams.

<a name="global-error-handling"></a>
## Global Error Handling

Global error handling can be achieved by implementing the `ErrorWebExceptionHandler` interface.

<a name="creating-a-global-error-handler"></a>
### Creating a Global Error Handler

```java
import org.springframework.boot.web.reactive.error.ErrorWebExceptionHandler;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.core.annotation.Order;
import org.springframework.http.HttpStatus;
import org.springframework.http.MediaType;
import org.springframework.web.reactive.function.server.ServerRequest;
import org.springframework.web.reactive.function.server.ServerResponse;
import org.springframework.web.server.ServerWebExchange;
import reactor.core.publisher.Mono;

@Configuration
public class GlobalErrorHandlerConfig {

    @Bean
    @Order(-2)
    public ErrorWebExceptionHandler globalErrorHandler() {
        return new GlobalErrorHandler();
    }

    private static class GlobalErrorHandler implements ErrorWebExceptionHandler {

        @Override
        public Mono<Void> handle(ServerWebExchange exchange, Throwable ex) {
            ServerRequest request = ServerRequest.create(exchange, HandlerStrategies.withDefaults().messageReaders());
            return renderErrorResponse(request, ex)
                .flatMap(response -> response.writeTo(exchange, HandlerStrategies.withDefaults()));
        }

        private Mono<ServerResponse> renderErrorResponse(ServerRequest request, Throwable ex) {
            return ServerResponse.status(HttpStatus.INTERNAL_SERVER_ERROR)
                .contentType(MediaType.APPLICATION_JSON)
                .bodyValue(new ErrorResponse(ex.getMessage()));
        }
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

<a name="configuration-class-explanation"></a>
### Configuration Class Explanation

In the above configuration, we define a global error handler `GlobalErrorHandler` that captures all unhandled exceptions and returns a unified error response. By implementing the `ErrorWebExceptionHandler` interface, we can customize the error handling logic.

<a name="controller-level-error-handling"></a>
## Controller-Level Error Handling

At the controller level, we can use the `@ExceptionHandler` annotation to handle specific exceptions.

```java
import org.springframework.web.bind.annotation.ExceptionHandler;
import org.springframework.web.bind.annotation.RestControllerAdvice;
import org.springframework.http.ResponseEntity;

@RestControllerAdvice
public class ControllerExceptionHandler {

    @ExceptionHandler(BookNotFoundException.class)
    public ResponseEntity<ErrorResponse> handleBookNotFound(BookNotFoundException ex) {
        ErrorResponse errorResponse = new ErrorResponse(ex.getMessage());
        return ResponseEntity.status(HttpStatus.NOT_FOUND).body(errorResponse);
    }

    // Handle other exceptions
    @ExceptionHandler(Exception.class)
    public ResponseEntity<ErrorResponse> handleException(Exception ex) {
        ErrorResponse errorResponse = new ErrorResponse("Internal Server Error");
        return ResponseEntity.status(HttpStatus.INTERNAL_SERVER_ERROR).body(errorResponse);
    }
}
```

<a name="error-handling-in-reactive-streams"></a>
## Error Handling in Reactive Streams

In reactive streams, we can use various operators to handle errors.

<a name="using-onerrorresume"></a>
### Using `onErrorResume`

The `onErrorResume` operator allows switching to another `Publisher` when an error occurs.

```java
public Mono<Book> findBookById(String id) {
    return bookRepository.findById(id)
        .onErrorResume(ex -> {
            log.error("Error finding book", ex);
            return Mono.empty();
        });
}
```

<a name="using-onerrorreturn"></a>
### Using `onErrorReturn`

The `onErrorReturn` operator allows returning a default value when an error occurs.

```java
public Mono<Book> findBookById(String id) {
    return bookRepository.findById(id)
        .onErrorReturn(new Book("default", "Default Book", "Unknown Author", 0.0));
}
```

<a name="using-onerrormap"></a>
### Using `onErrorMap`

The `onErrorMap` operator allows converting the error to another type of error.

```java
public Mono<Book> findBookById(String id) {
    return bookRepository.findById(id)
        .onErrorMap(ex -> new CustomException("Custom error message", ex));
}
```

<a name="conclusion"></a>
## Conclusion

In this article, we explored how to implement effective error handling strategies in Spring WebFlux applications. By using global error handling, controller-level error handling, and error handling in reactive streams, we can build more robust and user-friendly applications. Proper error handling is essential for maintaining the reliability and performance of modern reactive applications.