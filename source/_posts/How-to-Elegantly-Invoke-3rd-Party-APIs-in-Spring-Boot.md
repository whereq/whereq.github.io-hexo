---
title: How to Elegantly Invoke 3rd Party APIs in Spring Boot
date: 2024-10-23 20:20:44
categories:
- Spring Boot
- API
tags:
- Spring Boot
- API
---

- [How to Elegantly Invoke 3rd Party APIs in Spring Boot with Retries and Circuit Breakers](#how-to-elegantly-invoke-3rd-party-apis-in-spring-boot-with-retries-and-circuit-breakers)
    - [1. Basic API Invocation Using `RestTemplate`](#1-basic-api-invocation-using-resttemplate)
    - [Key Points:](#key-points)
    - [2. Elegant Error Handling with `RestTemplate`](#2-elegant-error-handling-with-resttemplate)
    - [Key Points:](#key-points-1)
    - [3. Retries Using `RetryTemplate` with Annotations](#3-retries-using-retrytemplate-with-annotations)
      - [Using `@Retryable` Annotation](#using-retryable-annotation)
      - [Explanation:](#explanation)
    - [Key Points:](#key-points-2)
    - [4. Circuit Breaker with Resilience4j and Annotations](#4-circuit-breaker-with-resilience4j-and-annotations)
      - [Dependency Setup](#dependency-setup)
      - [Circuit Breaker Configuration](#circuit-breaker-configuration)
      - [Using Resilience4j Annotations](#using-resilience4j-annotations)
      - [Explanation:](#explanation-1)
    - [Key Points:](#key-points-3)
    - [5. Sample Code with Annotations](#5-sample-code-with-annotations)
    - [Key Points:](#key-points-4)
    - [Diagrams](#diagrams)
    - [Key Points:](#key-points-5)
    - [6. Conclusion](#6-conclusion)
    - [Key Takeaways:](#key-takeaways)

---

# How to Elegantly Invoke 3rd Party APIs in Spring Boot with Retries and Circuit Breakers

---

When integrating with third-party APIs, ensuring robust and fault-tolerant API invocations is crucial for a resilient microservices architecture. In this updated guide, we will expand on retry mechanisms and fallback strategies, introducing annotations for retries and fallbacks. We will also explore enhanced usage of Resilience4j for circuit breaking, along with retry policies via `RetryTemplate` and Spring's `@Retryable` annotation.

---

### 1. Basic API Invocation Using `RestTemplate`
<a name="1-basic-api-invocation-using-resttemplate"></a>

We start with a simple HTTP client in Spring Boot, using `RestTemplate` to invoke a third-party API. Although basic, this approach lacks resilience for handling errors or failures in the third-party service.

```java
@Service
public class ApiService {
    private final RestTemplate restTemplate;

    @Autowired
    public ApiService(RestTemplateBuilder restTemplateBuilder) {
        this.restTemplate = restTemplateBuilder.build();
    }

    public String invokeExternalApi(String url) {
        return restTemplate.getForObject(url, String.class);
    }
}
```

### Key Points:
- **RestTemplate**: Basic HTTP client for API invocation.
- **Lack of Resilience**: No built-in error handling or retry mechanisms.

---

### 2. Elegant Error Handling with `RestTemplate`
<a name="2-elegant-error-handling-with-resttemplate"></a>

Error handling can be enhanced by implementing custom error handling for different types of HTTP status codes using `ResponseErrorHandler`.

```java
@Service
public class ApiService {

    private final RestTemplate restTemplate;

    @Autowired
    public ApiService(RestTemplateBuilder restTemplateBuilder) {
        this.restTemplate = restTemplateBuilder
                            .errorHandler(new CustomErrorHandler())  // custom error handler
                            .build();
    }

    public String invokeExternalApi(String url) {
        try {
            return restTemplate.getForObject(url, String.class);
        } catch (HttpClientErrorException ex) {
            return "Client error: " + ex.getStatusCode();
        } catch (HttpServerErrorException ex) {
            return "Server error: " + ex.getStatusCode();
        }
    }
}

public class CustomErrorHandler implements ResponseErrorHandler {

    @Override
    public boolean hasError(ClientHttpResponse response) throws IOException {
        return response.getStatusCode().is4xxClientError() || response.getStatusCode().is5xxServerError();
    }

    @Override
    public void handleError(ClientHttpResponse response) throws IOException {
        // Custom error handling
    }
}
```

### Key Points:
- **Custom Error Handler**: Implements `ResponseErrorHandler` for fine-grained error handling.
- **Exception Handling**: Catches specific HTTP exceptions for client and server errors.

---

### 3. Retries Using `RetryTemplate` with Annotations
<a name="3-retries-using-retrytemplate-with-annotations"></a>

Spring Retry provides powerful retry mechanisms that can be applied using annotations for cleaner code. The `@Retryable` annotation helps trigger automatic retries when exceptions occur.

#### Using `@Retryable` Annotation
<a name="using-retryable-annotation"></a>

```java
@Service
public class ApiService {

    private final RestTemplate restTemplate;

    @Autowired
    public ApiService(RestTemplateBuilder restTemplateBuilder) {
        this.restTemplate = restTemplateBuilder.build();
    }

    @Retryable(
        value = { HttpServerErrorException.class },
        maxAttempts = 5,
        backoff = @Backoff(delay = 2000)
    )
    public String invokeExternalApiWithRetry(String url) {
        return restTemplate.getForObject(url, String.class);
    }

    @Recover
    public String recover(HttpServerErrorException e, String url) {
        return "Fallback response for: " + url;
    }
}
```

#### Explanation:
<a name="explanation"></a>
- **@Retryable**: Specifies retryable exceptions (e.g., `HttpServerErrorException`), with configurable maximum attempts and backoff strategy.
- **@Recover**: Defines a fallback method that triggers when retries are exhausted.

### Key Points:
- **Retry Logic**: Automatically retries failed API calls.
- **Fallback Mechanism**: Provides a fallback response when retries fail.

---

### 4. Circuit Breaker with Resilience4j and Annotations
<a name="4-circuit-breaker-with-resilience4j-and-annotations"></a>

Resilience4j is a lightweight, easy-to-use fault tolerance library designed for Java 8 and functional programming. It provides circuit breaking capabilities similar to Hystrix but with a more modern approach.

#### Dependency Setup

First, add the necessary dependencies to your `pom.xml`:

```xml
<dependency>
    <groupId>io.github.resilience4j</groupId>
    <artifactId>resilience4j-spring-boot3</artifactId>
    <version>2.0.0</version>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-aop</artifactId>
</dependency>
```

#### Circuit Breaker Configuration

Configure the circuit breaker in your Spring Boot application:

```java
@Configuration
public class Resilience4jConfig {

    @Bean
    public CircuitBreakerConfig circuitBreakerConfig() {
        return CircuitBreakerConfig.custom()
            .failureRateThreshold(50)
            .waitDurationInOpenState(Duration.ofMillis(1000))
            .ringBufferSizeInClosedState(2)
            .build();
    }

    @Bean
    public CircuitBreakerRegistry circuitBreakerRegistry(CircuitBreakerConfig circuitBreakerConfig) {
        return CircuitBreakerRegistry.of(circuitBreakerConfig);
    }

    @Bean
    public CircuitBreaker circuitBreaker(CircuitBreakerRegistry circuitBreakerRegistry) {
        return circuitBreakerRegistry.circuitBreaker("externalApi");
    }
}
```

#### Using Resilience4j Annotations

Apply the circuit breaker to your service method:

```java
@Service
public class ApiService {

    private final RestTemplate restTemplate;
    private final CircuitBreaker circuitBreaker;

    @Autowired
    public ApiService(RestTemplateBuilder restTemplateBuilder, CircuitBreaker circuitBreaker) {
        this.restTemplate = restTemplateBuilder.build();
        this.circuitBreaker = circuitBreaker;
    }

    @CircuitBreaker(name = "externalApi", fallbackMethod = "fallbackMethod")
    public String invokeApiWithCircuitBreaker(String url) {
        return restTemplate.getForObject(url, String.class);
    }

    public String fallbackMethod(String url, Throwable t) {
        return "Fallback response for: " + url;
    }
}
```

#### Explanation:
<a name="explanation-1"></a>
- **CircuitBreaker**: Configures the circuit breaker with custom settings.
- **@CircuitBreaker**: Annotates the method to be protected by the circuit breaker.
- **Fallback Method**: Provides a fallback response when the circuit breaker trips.

### Key Points:
- **Resilience4j**: Modern fault tolerance library replacing Hystrix.
- **Circuit Breaker**: Prevents cascading failures by short-circuiting failed API calls.
- **Fallback Method**: Provides a fallback response when the circuit breaker trips.

---

### 5. Sample Code with Annotations
<a name="5-sample-code-with-annotations"></a>

Combining both `@Retryable` and Resilience4j’s `@CircuitBreaker`, we create a highly resilient service capable of handling third-party API failures.

```java
@Service
public class ApiService {

    private final RestTemplate restTemplate;
    private final CircuitBreaker circuitBreaker;

    @Autowired
    public ApiService(RestTemplateBuilder restTemplateBuilder, CircuitBreaker circuitBreaker) {
        this.restTemplate = restTemplateBuilder.build();
        this.circuitBreaker = circuitBreaker;
    }

    @Retryable(
        value = { HttpServerErrorException.class },
        maxAttempts = 5,
        backoff = @Backoff(delay = 1000)
    )
    @CircuitBreaker(name = "externalApi", fallbackMethod = "fallbackMethod")
    public String invokeApi(String url) {
        return restTemplate.getForObject(url, String.class);
    }

    @Recover
    public String retryRecovery(HttpServerErrorException e, String url) {
        return "Retry failed for: " + url;
    }

    public String fallbackMethod(String url, Throwable t) {
        return "Circuit breaker fallback for: " + url;
    }
}
```

### Key Points:
- **Combined Resilience**: Combines retries and circuit breaking for improved fault tolerance.
- **Fallback Handling**: Provides fallback responses for both retry failures and circuit breaker trips.

---

### Diagrams
<a name="diagrams"></a>

To visualize how retries and circuit breakers work together, the following diagram illustrates the flow:

```
+-------------------+      +------------------+
| API Call Attempt  |----> | Retry Logic      |
+-------------------+      +------------------+
                              |
            Retry Success<----+---->Retry Failure
                              |
                          Circuit Breaker
                              |
            Success<-----------+------------>Fallback
```

### Key Points:
- **API Call Attempt**: Initial API invocation.
- **Retry Logic**: Retries the API call if it fails.
- **Circuit Breaker**: Evaluates whether to invoke the fallback method after retries fail.

---

### 6. Conclusion
<a name="6-conclusion"></a>

In Spring Boot, integrating with third-party APIs elegantly involves applying retry mechanisms and circuit breaker patterns. Using `@Retryable` and Resilience4j annotations simplifies the code, making it more resilient to failures and easier to maintain. The combination of these two approaches ensures that your system can handle both intermittent and persistent issues with third-party APIs.

### Key Takeaways:
- **Resilient API Invocations**: Use retries and circuit breakers for fault tolerance.
- **Annotation-Based Configuration**: Simplifies implementation and maintenance.
- **Fallback Strategies**: Provide graceful degradation when API calls fail.

By mastering these techniques, you can build robust and resilient microservices that gracefully handle third-party API failures.