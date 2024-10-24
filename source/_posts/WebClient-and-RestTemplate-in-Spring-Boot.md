---
title: WebClient and RestTemplate in Spring Boot
date: 2024-10-23 22:16:39
categories:
- Spring Boot
- WebFlux
tags:
- Spring Boot
- WebFlux
---

# WebClient and RestTemplate in Spring Boot

## Index
- [WebClient and RestTemplate in Spring Boot](#webclient-and-resttemplate-in-spring-boot)
  - [Index](#index)
  - [1. Overview](#1-overview)
  - [2. What is RestTemplate?](#2-what-is-resttemplate)
    - [Characteristics:](#characteristics)
    - [Code Sample:](#code-sample)
  - [3. What is WebClient?](#3-what-is-webclient)
    - [Characteristics:](#characteristics-1)
    - [Code Sample:](#code-sample-1)
  - [4. Key Differences](#4-key-differences)
    - [4.1 Synchronous vs Asynchronous](#41-synchronous-vs-asynchronous)
    - [4.2 Reactive Support](#42-reactive-support)
    - [4.3 Error Handling](#43-error-handling)
    - [4.4 Performance](#44-performance)
    - [4.5 Flexibility \& Features](#45-flexibility--features)
  - [5. Best Practices and Use Cases](#5-best-practices-and-use-cases)
    - [Best Practices:](#best-practices)
  - [6. Real-Life Use Case Example](#6-real-life-use-case-example)
    - [6.1 Using RestTemplate (Synchronous)](#61-using-resttemplate-synchronous)
    - [6.2 Using WebClient (Asynchronous)](#62-using-webclient-asynchronous)
    - [6.3 Adding Retry and Fallback (WebClient)](#63-adding-retry-and-fallback-webclient)
  - [7. Conclusion](#7-conclusion)
  - [8. Diagrams](#8-diagrams)
    - [8.1 Synchronous RestTemplate Workflow](#81-synchronous-resttemplate-workflow)
    - [8.2 Asynchronous WebClient Workflow](#82-asynchronous-webclient-workflow)
  - [9. Deep Dive into Backpressure](#9-deep-dive-into-backpressure)
    - [9.1 What is Backpressure in Reactive Systems?](#91-what-is-backpressure-in-reactive-systems)
    - [9.2 Is Backpressure Good or Bad?](#92-is-backpressure-good-or-bad)
    - [9.3 How WebClient Supports Backpressure](#93-how-webclient-supports-backpressure)
    - [9.4 Code Example of WebClient Supporting Backpressure](#94-code-example-of-webclient-supporting-backpressure)
    - [9.5 Why RestTemplate Doesn't Have Backpressure](#95-why-resttemplate-doesnt-have-backpressure)
    - [9.6 When to Use RestTemplate](#96-when-to-use-resttemplate)
    - [9.7 Conclusion](#97-conclusion)

---

## 1. Overview
Spring Boot offers two primary ways to make HTTP requests: `RestTemplate` and `WebClient`. While both can be used to interact with external APIs, they differ significantly in features, capabilities, and usage. In this article, we'll explore the key differences, when to use each, and best practices through practical examples.

## 2. What is RestTemplate?
`RestTemplate` is a synchronous client designed to perform HTTP requests and handle responses. It has been around for a long time and is simple to use but is gradually being deprecated in favor of `WebClient` for reactive programming.

### Characteristics:
- Synchronous/blocking operations.
- Easy to use for basic CRUD (Create, Read, Update, Delete) operations.
- Simplified API for HTTP interactions.
- Supports a wide range of RESTful operations (GET, POST, PUT, DELETE, etc.).
  
### Code Sample:
```java
RestTemplate restTemplate = new RestTemplate();
String url = "https://catfact.ninja/facts";
ResponseEntity<String> response = restTemplate.getForEntity(url, String.class);

System.out.println("Response: " + response.getBody());
```

## 3. What is WebClient?
`WebClient` is the successor to `RestTemplate`, introduced in Spring 5. It supports both synchronous and asynchronous (non-blocking) calls and is designed for reactive programming, making it more suitable for modern web applications.

### Characteristics:
- Asynchronous and non-blocking.
- Part of the Spring WebFlux module.
- Supports backpressure, which is crucial for reactive streams.
- Allows chaining of multiple operations in a declarative style.

### Code Sample:
```java
WebClient webClient = WebClient.create("https://catfact.ninja");

Mono<String> response = webClient.get()
    .uri("/facts")
    .retrieve()
    .bodyToMono(String.class);

response.subscribe(System.out::println);
```

## 4. Key Differences

| Feature                 | RestTemplate                           | WebClient                                                    |
|-------------------------|----------------------------------------|--------------------------------------------------------------|
| **Type**                | Synchronous                            | Asynchronous and Reactive                                    |
| **Concurrency**         | Blocking (each request waits)          | Non-blocking (multiple requests in parallel)                 |
| **Backpressure**        | No (Processes whole response at once)  | Yes, Supports backpressure in reactive systems, flow control |
| **Error Handling**      | Simple but lacks flexibility           | Advanced error handling with `.onStatus()`                   |
| **Support for Streams** | No                                     | Yes, can handle large streams of data                        |
| **Flexibility**         | Limited customizations                 | Highly flexible and configurable                             |

### 4.1 [Synchronous vs Asynchronous](#synchronous-vs-asynchronous)
- **RestTemplate** operates in a synchronous/blocking fashion, meaning the thread is blocked until the response arrives.
- **WebClient** can operate both synchronously and asynchronously, allowing for non-blocking I/O and better scalability.

### 4.2 [Reactive Support](#reactive-support)
`WebClient` is built to support the reactive paradigm of programming, which allows applications to handle a massive number of requests efficiently by leveraging non-blocking I/O operations. RestTemplate does not have this support.

### 4.3 [Error Handling](#error-handling)
`WebClient` provides more flexible error handling with methods like `.onStatus()` for dealing with specific HTTP status codes.

### 4.4 [Performance](#performance)
Due to its asynchronous nature, `WebClient` typically offers better performance in handling multiple HTTP requests concurrently, making it more suitable for high-performance applications.

### 4.5 [Flexibility & Features](#flexibility--features)
`WebClient` can be configured with timeouts, retries, and fallbacks more easily compared to `RestTemplate`. It supports streaming large amounts of data and provides reactive streams to manage data flow.

## 5. [Best Practices and Use Cases](#best-practices-and-use-cases)

- **When to use RestTemplate:**
  - Small applications with minimal requirements for concurrency.
  - Use cases where synchronous/blocking operations are acceptable.

- **When to use WebClient:**
  - Modern applications requiring scalability and performance.
  - When working with large-scale microservices architectures.
  - For non-blocking and reactive API calls.

### Best Practices:
- Use `WebClient` for new applications that need to scale.
- Avoid using `RestTemplate` for large-scale systems due to its synchronous nature.
- Handle errors properly using `onStatus()` in `WebClient`.

## 6. [Real-Life Use Case Example](#real-life-use-case-example)

Let's assume a situation where you are building a service to fetch data from multiple APIs concurrently. `RestTemplate` would block on each request, but `WebClient` would allow you to make all the calls concurrently and return the responses as they come in.

### 6.1 [Using RestTemplate (Synchronous)](#resttemplate-example)
```java
@RestController
public class ApiController {

    private RestTemplate restTemplate = new RestTemplate();

    @GetMapping("/data")
    public String fetchData() {
        String api1Response = restTemplate.getForObject("https://catfact.ninja/facts", String.class);
        String api2Response = restTemplate.getForObject("https://catfact.ninja/fact", String.class);
        return "Data from API1: " + api1Response + " and API2: " + api2Response;
    }
}
```

### 6.2 [Using WebClient (Asynchronous)](#webclient-example)
```java
@RestController
public class ApiController {

    private WebClient webClient = WebClient.create();

    @GetMapping("/data")
    public Mono<String> fetchData() {
        Mono<String> api1Response = webClient.get().uri("https://catfact.ninja/facts").retrieve().bodyToMono(String.class);
        Mono<String> api2Response = webClient.get().uri("https://catfact.ninja/fact").retrieve().bodyToMono(String.class);

        return Mono.zip(api1Response, api2Response, (api1, api2) -> "Data from API1: " + api1 + " and API2: " + api2);
    }
}
```

### 6.3 [Adding Retry and Fallback (WebClient)](#retry-fallback)
```java
public class ApiService {

    private WebClient webClient = WebClient.builder()
        .baseUrl("https://catfact.ninja")
        .filter(ExchangeFilterFunctions.statusError(HttpStatus::is4xxClientError, clientResponse -> new RuntimeException("API Error")))
        .build();

    @Retryable(value = {RuntimeException.class}, maxAttempts = 3)
    public Mono<String> fetchDataWithRetry() {
        return webClient.get()
            .uri("/data")
            .retrieve()
            .bodyToMono(String.class)
            .onErrorReturn("Fallback Data");
    }
}
```

## 7. [Conclusion](#conclusion)
`RestTemplate` is simple and suitable for synchronous applications, but for modern, scalable applications, `WebClient` is the clear choice due to its non-blocking and reactive features. In Spring Boot applications, it's recommended to move towards `WebClient` for future-proofing your code.

## 8. [Diagrams](#diagrams)

### 8.1 Synchronous RestTemplate Workflow
```plaintext
Request -> Server -> Blocking I/O -> Response -> Client
```

### 8.2 Asynchronous WebClient Workflow
```plaintext
Request -> Server -> Non-Blocking I/O -> Reactive Stream -> Response -> Client
```


## 9. [Deep Dive into Backpressure](#9-deep-dive-into-backpressure)
### 9.1 What is Backpressure in Reactive Systems?

Backpressure refers to a mechanism used in reactive systems to control the flow of data between producers and consumers. It allows the consumer to handle data at a rate it can manage, preventing overload or crashes. If the producer is emitting data faster than the consumer can process, backpressure signals the producer to slow down or pause until the consumer catches up.

### 9.2 Is Backpressure Good or Bad?

**Good**:
- **Efficient Resource Management**: Backpressure prevents resource exhaustion by controlling how much data is in the system.
- **System Stability**: It helps prevent overloading a service, ensuring it operates smoothly without being overwhelmed by data.
- **Improved Scalability**: In distributed systems, especially when handling streams of real-time data, backpressure enables better scalability and responsiveness.

**Challenges**:
- **Increased Complexity**: While beneficial, backpressure introduces more complexity in system design. Developers need to think carefully about flow control.
- **Latency**: If not managed properly, backpressure can introduce latency because the producer has to wait until the consumer is ready to receive more data.

### 9.3 How WebClient Supports Backpressure

In Spring WebFlux, `WebClient` works in a reactive, non-blocking way and is integrated with Project Reactor to support backpressure. When fetching large streams of data, WebClient can handle them without overwhelming the system. Here's how it works:

- **Reactive Streams**: WebClient communicates using `Publisher` and `Subscriber` patterns, which allows it to handle streams of data asynchronously. If the subscriber (consumer) is slower than the publisher (producer), it can request data at a manageable rate, applying backpressure when needed.
  
- **Flow Control**: In a typical scenario, the subscriber can signal how much data it wants to receive at any given moment. This avoids a flood of data that could cause the system to run out of resources. WebClient ensures that consumers can throttle the data based on their processing capacity.

### 9.4 Code Example of WebClient Supporting Backpressure

```java
WebClient webClient = WebClient.create();

Flux<String> response = webClient.get()
    .uri("/large-data-stream")
    .retrieve()
    .bodyToFlux(String.class)
    .limitRate(10); // apply backpressure by limiting the rate of consumption

response.subscribe(data -> {
    System.out.println("Processing: " + data);
});
```

In this example, the `limitRate()` method ensures the consumer processes the data stream at a manageable rate, effectively applying backpressure.

### 9.5 Why RestTemplate Doesn't Have Backpressure

RestTemplate is built on a blocking I/O model, which means it waits for each HTTP request to complete before proceeding to the next task. This approach works well for synchronous operations, but it lacks the flow control mechanisms needed for backpressure. Here's why RestTemplate doesn't support backpressure:

1. **Blocking Nature**: RestTemplate blocks the current thread while waiting for a response, handling each request in sequence, which inherently prevents a need for backpressure.
   
2. **Lack of Reactive Streams**: Unlike WebClient, which is built around reactive streams and the non-blocking `Publisher-Subscriber` pattern, RestTemplate doesn't have the underlying architecture to request data in chunks or manage flow control. It processes the entire response in a blocking call and holds it in memory until the response is complete.

3. **Thread-per-request Model**: RestTemplate uses a traditional thread-per-request model, which consumes a thread for the entire duration of a request. This model isn't designed to handle large streams of data efficiently or manage backpressure when dealing with data that arrives at varying rates.

### 9.6 When to Use RestTemplate

RestTemplate is still useful for simple, synchronous tasks that don’t require fine-grained control over how data is processed, particularly when working with legacy systems or simple API interactions that don't involve heavy real-time data streaming.

However, for reactive programming, high throughput, and complex asynchronous data handling, WebClient is the better choice, as it is designed to handle backpressure effectively.

### 9.7 Conclusion

Backpressure is essential in reactive systems for maintaining stability and scalability. Spring WebClient leverages Project Reactor to support it, making it an excellent choice for modern applications handling large streams of data. While it adds some complexity, the benefits of resource management and smooth performance outweigh the drawbacks.

