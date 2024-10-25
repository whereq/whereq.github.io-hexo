---
title: Reactive Programming in Spring WebFlux Series - XII
date: 2024-10-24 14:07:10
categories:
- WebFlux
- Spring Boot
- Reactive
tags:
- WebFlux
- Spring Boot
- Reactive
---


In the previous 11 articles, we explored Spring WebFlux fundamentals, Reactor, error handling, data transformation, reactive database access, performance optimization, security, testing, deployment, operations, and best practices. This article will demonstrate how to build a practical Spring WebFlux application through a comprehensive case study.

- [Case Study](#case-study)
  - [1. Case Background](#1-case-background)
  - [2. Project Setup](#2-project-setup)
  - [3. Data Model](#3-data-model)
  - [4. Repository Layer](#4-repository-layer)
  - [5. Service Layer](#5-service-layer)
  - [6. Handler and Routing Configuration](#6-handler-and-routing-configuration)
  - [7. Testing](#7-testing)
    - [Unit Testing](#unit-testing)
    - [Integration Testing](#integration-testing)
  - [8. Deployment and Monitoring](#8-deployment-and-monitoring)
  - [9. Conclusion](#9-conclusion)

---

# Case Study

---

<a name="case-background"></a>
## 1. Case Background

We will build a simple book management system with the following features:

- Add a book
- Retrieve all books
- Retrieve a book by ID
- Update book details
- Delete a book

<a name="project-setup"></a>
## 2. Project Setup

First, create a Spring Boot project and include the required dependencies:

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
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-security</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-actuator</artifactId>
    </dependency>
</dependencies>
```

<a name="data-model"></a>
## 3. Data Model

Define a `Book` entity:

```java
import org.springframework.data.annotation.Id;
import org.springframework.data.relational.core.mapping.Table;

@Table("books")
public class Book {
    @Id
    private Long id;
    private String title;
    private String author;
    private Double price;

    // Getters and Setters
}
```

<a name="repository-layer"></a>
## 4. Repository Layer

Create a reactive `BookRepository` interface:

```java
import org.springframework.data.repository.reactive.ReactiveCrudRepository;

public interface BookRepository extends ReactiveCrudRepository<Book, Long> {
}
```

<a name="service-layer"></a>
## 5. Service Layer

Create a service class for business logic handling:

```java
import org.springframework.stereotype.Service;
import reactor.core.publisher.Flux;
import reactor.core.publisher.Mono;

@Service
public class BookService {

    private final BookRepository bookRepository;

    public BookService(BookRepository bookRepository) {
        this.bookRepository = bookRepository;
    }

    public Mono<Book> addBook(Book book) {
        return bookRepository.save(book);
    }

    public Flux<Book> getAllBooks() {
        return bookRepository.findAll();
    }

    public Mono<Book> getBookById(Long id) {
        return bookRepository.findById(id);
    }

    public Mono<Book> updateBook(Long id, Book book) {
        return bookRepository.findById(id)
            .flatMap(existingBook -> {
                existingBook.setTitle(book.getTitle());
                existingBook.setAuthor(book.getAuthor());
                existingBook.setPrice(book.getPrice());
                return bookRepository.save(existingBook);
            });
    }

    public Mono<Void> deleteBook(Long id) {
        return bookRepository.deleteById(id);
    }
}
```

<a name="handler-and-routing-configuration"></a>
## 6. Handler and Routing Configuration

Define handler functions and routing configuration:

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
public class BookHandler {

    private final BookService bookService;

    public BookHandler(BookService bookService) {
        this.bookService = bookService;
    }

    public Mono<ServerResponse> addBook(ServerRequest request) {
        return request.bodyToMono(Book.class)
            .flatMap(bookService::addBook)
            .flatMap(book -> ServerResponse.ok().body(Mono.just(book), Book.class));
    }

    public Mono<ServerResponse> getAllBooks(ServerRequest request) {
        return ServerResponse.ok().body(bookService.getAllBooks(), Book.class);
    }

    public Mono<ServerResponse> getBookById(ServerRequest request) {
        Long id = Long.valueOf(request.pathVariable("id"));
        return bookService.getBookById(id)
            .flatMap(book -> ServerResponse.ok().body(Mono.just(book), Book.class))
            .switchIfEmpty(ServerResponse.notFound().build());
    }

    public Mono<ServerResponse> updateBook(ServerRequest request) {
        Long id = Long.valueOf(request.pathVariable("id"));
        return request.bodyToMono(Book.class)
            .flatMap(book -> bookService.updateBook(id, book))
            .flatMap(updatedBook -> ServerResponse.ok().body(Mono.just(updatedBook), Book.class))
            .switchIfEmpty(ServerResponse.notFound().build());
    }

    public Mono<ServerResponse> deleteBook(ServerRequest request) {
        Long id = Long.valueOf(request.pathVariable("id"));
        return bookService.deleteBook(id)
            .then(ServerResponse.ok().build());
    }
}

@Configuration
public class RouterConfig {

    @Bean
    public RouterFunction<ServerResponse> route(BookHandler handler) {
        return RouterFunctions.route()
            .POST("/books", handler::addBook)
            .GET("/books", handler::getAllBooks)
            .GET("/books/{id}", handler::getBookById)
            .PUT("/books/{id}", handler::updateBook)
            .DELETE("/books/{id}", handler::deleteBook)
            .build();
    }
}
```

<a name="testing"></a>
## 7. Testing

<a name="unit-testing"></a>
### Unit Testing

```java
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.autoconfigure.web.reactive.WebFluxTest;
import org.springframework.test.web.reactive.server.WebTestClient;

@WebFluxTest(BookHandler.class)
public class BookHandlerTest {

    @Autowired
    private WebTestClient webTestClient;

    @Test
    void testAddBook() {
        Book book = new Book(null, "Spring WebFlux", "Author", 39.99);
        webTestClient.post().uri("/books")
            .bodyValue(book)
            .exchange()
            .expectStatus().isOk()
            .expectBody()
            .jsonPath("$.title").isEqualTo("Spring WebFlux");
    }

    @Test
    void testGetAllBooks() {
        webTestClient.get().uri("/books")
            .exchange()
            .expectStatus().isOk()
            .expectBodyList(Book.class).hasSize(1);
    }

    @Test
    void testGetBookById() {
        webTestClient.get().uri("/books/1")
            .exchange()
            .expectStatus().isOk()
            .expectBody()
            .jsonPath("$.title").isEqualTo("Spring WebFlux");
    }
}
```

<a name="integration-testing"></a>
### Integration Testing

```java
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.test.web.reactive.server.WebTestClient;

@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
public class ApplicationIntegrationTest {

    @Autowired
    private WebTestClient webTestClient;

    @Test
    void testBookFlow() {
        // Add book
        Book book = new Book(null, "Spring WebFlux", "Author", 39.99);
        webTestClient.post().uri("/books")
            .bodyValue(book)
            .exchange()
            .expectStatus().isOk()
            .expectBody()
            .jsonPath("$.title").isEqualTo("Spring WebFlux");

        // Retrieve all books
        webTestClient.get().uri("/books")
            .exchange()
            .expectStatus().isOk()
            .expectBodyList(Book.class).hasSize(1);

        // Retrieve book by ID
        webTestClient.get().uri("/books/1")
            .exchange()
            .expectStatus().isOk()
            .expectBody()
            .jsonPath("$.title").isEqualTo("Spring WebFlux");

        // Update book
        Book updatedBook = new Book(null, "Spring WebFlux Updated", "Author", 49.99);
        webTestClient.put().uri("/books/1")
            .bodyValue(updatedBook)
            .exchange()
            .expectStatus().isOk()
            .expectBody()
            .jsonPath("$.title").isEqualTo("Spring WebFlux Updated");

        // Delete book
        webTestClient.delete().uri("/books/1")
            .exchange()


            .expectStatus().isOk();

        // Verify deletion
        webTestClient.get().uri("/books/1")
            .exchange()
            .expectStatus().isNotFound();
    }
}
```

<a name="deployment-and-monitoring"></a>
## 8. Deployment and Monitoring

Ensure the application is deployed to production and properly monitored. Use **Spring Boot Actuator** with **Prometheus** and **Grafana** for monitoring:

```properties
management.endpoints.web.exposure.include=health,info,metrics,prometheus
```

<a name="conclusion"></a>
## 9. Conclusion

In this article, we demonstrated how to build a real-world reactive application using Spring WebFlux by integrating the concepts from earlier articles and applying best practices.