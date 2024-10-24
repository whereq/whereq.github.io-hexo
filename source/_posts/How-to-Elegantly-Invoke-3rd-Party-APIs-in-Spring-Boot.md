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

### How to Elegantly Invoke 3rd Party APIs in Spring Boot with Retries and Circuit Breakers

When integrating with third-party APIs, ensuring robust and fault-tolerant API invocations is crucial for a resilient microservices architecture. In this updated guide, we will expand on retry mechanisms and fallback strategies, introducing annotations for retries and fallbacks. We will also explore enhanced usage of Hystrix for circuit breaking, along with retry policies via `RetryTemplate` and Spring's `@Retryable` annotation.

In this article, we'll cover:

### Index
- [How to Elegantly Invoke 3rd Party APIs in Spring Boot with Retries and Circuit Breakers](#how-to-elegantly-invoke-3rd-party-apis-in-spring-boot-with-retries-and-circuit-breakers)
- [Index](#index)
- [1. Basic API Invocation Using `RestTemplate`](#1-basic-api-invocation-using-resttemplate)
- [2. Elegant Error Handling with `RestTemplate`](#2-elegant-error-handling-with-resttemplate)
- [3. Retries Using `RetryTemplate` with Annotations](#3-retries-using-retrytemplate-with-annotations)
  - [Using `@Retryable` Annotation](#using-retryable-annotation)
  - [Explanation:](#explanation)
- [4. Circuit Breaker with Hystrix and Annotations](#4-circuit-breaker-with-hystrix-and-annotations)
  - [Explanation:](#explanation-1)
- [5. Sample Code with Annotations](#5-sample-code-with-annotations)
- [Diagrams](#diagrams)
- [6. Conclusion](#6-conclusion)

### 1. Basic API Invocation Using `RestTemplate`

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

### 2. Elegant Error Handling with `RestTemplate`

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

### 3. Retries Using `RetryTemplate` with Annotations

Spring Retry provides powerful retry mechanisms that can be applied using annotations for cleaner code. The `@Retryable` annotation helps trigger automatic retries when exceptions occur.

#### Using `@Retryable` Annotation

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
- `@Retryable`: Specifies retryable exceptions (e.g., `HttpServerErrorException`), with configurable maximum attempts and backoff strategy.
- `@Recover`: Defines a fallback method that triggers when retries are exhausted.

### 4. Circuit Breaker with Hystrix and Annotations

Circuit breaking prevents repeated failures from overwhelming a service. Using Hystrix annotations, we can implement this pattern efficiently.

```java
@Service
public class ApiService {

    private final RestTemplate restTemplate;

    @Autowired
    public ApiService(RestTemplateBuilder restTemplateBuilder) {
        this.restTemplate = restTemplateBuilder.build();
    }

    @HystrixCommand(fallbackMethod = "fallbackMethod")
    public String invokeApiWithCircuitBreaker(String url) {
        return restTemplate.getForObject(url, String.class);
    }

    public String fallbackMethod(String url) {
        return "Fallback response for: " + url;
    }
}
```

#### Explanation:
- `@HystrixCommand`: Wraps the method invocation with circuit breaker logic, automatically routing to the fallback method when failures occur.

### 5. Sample Code with Annotations

Combining both `@Retryable` and Hystrix’s `@HystrixCommand`, we create a highly resilient service capable of handling third-party API failures.

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
        backoff = @Backoff(delay = 1000)
    )
    @HystrixCommand(fallbackMethod = "fallbackMethod")
    public String invokeApi(String url) {
        return restTemplate.getForObject(url, String.class);
    }

    @Recover
    public String retryRecovery(HttpServerErrorException e, String url) {
        return "Retry failed for: " + url;
    }

    public String fallbackMethod(String url) {
        return "Circuit breaker fallback for: " + url;
    }
}
```

In this sample:
- We combine retries and circuit breaking for improved fault tolerance.
- If retries fail, the `@Recover` method is triggered, but if the circuit breaker trips, the fallback method handles it.

### Diagrams

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

In this flow:
1. The service attempts an API call.
2. If the call fails, it retries up to the configured attempts.
3. If all retries fail, the circuit breaker evaluates whether to invoke the fallback method.

### 6. Conclusion

In Spring Boot, integrating with third-party APIs elegantly involves applying retry mechanisms and circuit breaker patterns. Using `@Retryable` and Hystrix annotations simplifies the code, making it more resilient to failures and easier to maintain. The combination of these two approaches ensures that your system can handle both intermittent and persistent issues with third-party APIs.
