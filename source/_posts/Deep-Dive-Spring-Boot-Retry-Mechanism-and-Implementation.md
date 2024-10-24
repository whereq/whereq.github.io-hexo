---
title: Deep Dive Spring Boot Retry Mechanism and Implementation
date: 2024-10-23 20:52:55
categories:
- Spring Boot
- WebFlux
tags:
- Spring Boot
- WebFlux
---


# Spring Boot Retry Mechanism - Detailed Guide with Sample Code

## Index
- [Spring Boot Retry Mechanism - Detailed Guide with Sample Code](#spring-boot-retry-mechanism---detailed-guide-with-sample-code)
  - [Index](#index)
  - [1. Introduction to Spring Retry](#1-introduction-to-spring-retry)
  - [2. How Retry Mechanism Works](#2-how-retry-mechanism-works)
  - [3. Plain Retry Implementation using `RetryTemplate`](#3-plain-retry-implementation-using-retrytemplate)
    - [Step-by-Step Guide:](#step-by-step-guide)
  - [4. Annotation-Based Retry Implementation using `@Retryable`](#4-annotation-based-retry-implementation-using-retryable)
    - [Step-by-Step Guide:](#step-by-step-guide-1)
  - [5. Recovering from Failure using `@Recover`](#5-recovering-from-failure-using-recover)
    - [Key Points:](#key-points)
  - [6. Flowchart: Retry Mechanism](#6-flowchart-retry-mechanism)
  - [7. Multiple Recover Methods](#7-multiple-recover-methods)
    - [Example: Multiple `@Recover` Methods](#example-multiple-recover-methods)
    - [Key Points:](#key-points-1)
    - [How It Works:](#how-it-works)
    - [Summary:](#summary)
  - [8. Conclusion](#8-conclusion)


## 1. Introduction to Spring Retry

Spring Boot provides a retry mechanism through the **Spring Retry** module. It allows developers to automatically retry operations that fail due to certain exceptions. This mechanism is highly useful for handling transient faults in distributed systems, like network issues, service unavailability, etc.

Spring Retry can be implemented in two ways:
- Using **RetryTemplate** (programmatically)
- Using **@Retryable** annotation (declaratively)

Spring Retry also supports recovery methods using **@Recover**, which can be invoked when the retries are exhausted.

---

## 2. How Retry Mechanism Works

The retry mechanism in Spring works as follows:
- When a method fails with an exception, Spring can automatically retry that method a configurable number of times.
- The retry can be based on specific exceptions or a more generic condition.
- Spring can also add a backoff policy (delay between retries) and an exponential backoff (increasing delays).
- After the maximum retry attempts, a **@Recover** method can be invoked to handle failure gracefully.

---

## 3. Plain Retry Implementation using `RetryTemplate`

The **RetryTemplate** is used to implement the retry mechanism programmatically. Here’s how it can be done:

### Step-by-Step Guide:
1. **Add Dependencies** in `pom.xml` (if using Maven):
```xml
<dependency>
    <groupId>org.springframework.retry</groupId>
    <artifactId>spring-retry</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-aop</artifactId>
</dependency>
```

2. **Define RetryTemplate Bean**:
```java
@Configuration
public class RetryConfig {
    @Bean
    public RetryTemplate retryTemplate() {
        RetryTemplate retryTemplate = new RetryTemplate();

        // Define retry policy (e.g., 3 attempts for any exception)
        SimpleRetryPolicy retryPolicy = new SimpleRetryPolicy();
        retryPolicy.setMaxAttempts(3);

        // Define backoff policy (e.g., 2 seconds between retries)
        FixedBackOffPolicy backOffPolicy = new FixedBackOffPolicy();
        backOffPolicy.setBackOffPeriod(2000); // 2000 ms = 2 seconds

        retryTemplate.setRetryPolicy(retryPolicy);
        retryTemplate.setBackOffPolicy(backOffPolicy);

        return retryTemplate;
    }
}
```

3. **Use RetryTemplate in Business Logic**:
```java
@Service
public class MyService {

    @Autowired
    private RetryTemplate retryTemplate;

    public void performTask() {
        retryTemplate.execute(context -> {
            // Logic that might fail
            System.out.println("Attempting task...");
            if (new Random().nextInt(10) < 8) {
                throw new RuntimeException("Failed task");
            }
            System.out.println("Task succeeded");
            return null;
        });
    }
}
```

4. **Main Application**:
```java
@SpringBootApplication
public class RetryApplication {
    public static void main(String[] args) {
        ApplicationContext context = SpringApplication.run(RetryApplication.class, args);
        MyService service = context.getBean(MyService.class);
        service.performTask();
    }
}
```

This implementation retries the `performTask()` method up to 3 times with a 2-second delay between attempts.

---

## 4. Annotation-Based Retry Implementation using `@Retryable`

The annotation-based approach uses `@Retryable` to automatically retry methods without having to explicitly define a `RetryTemplate`.

### Step-by-Step Guide:

1. **Add Dependencies** (same as in RetryTemplate approach).

2. **Enable Retry in Spring Boot Application**:
```java
@SpringBootApplication
@EnableRetry
public class RetryApplication {
    public static void main(String[] args) {
        SpringApplication.run(RetryApplication.class, args);
    }
}
```

3. **Use @Retryable Annotation**:
```java
@Service
public class MyService {

    @Retryable(
        value = { RuntimeException.class }, // Retry on specific exceptions
        maxAttempts = 3,                     // Retry up to 3 times
        backoff = @Backoff(delay = 2000)     // 2 seconds delay between retries
    )
    public void performTask() {
        System.out.println("Attempting task...");
        if (new Random().nextInt(10) < 8) {
            throw new RuntimeException("Task failed");
        }
        System.out.println("Task succeeded");
    }
}
```

4. **Recover from Failure using @Recover**:
You can define a recovery method that will be called when all retry attempts are exhausted.
```java
@Service
public class MyService {

    @Retryable(
        value = { RuntimeException.class },
        maxAttempts = 3,
        backoff = @Backoff(delay = 2000)
    )
    public void performTask() {
        System.out.println("Attempting task...");
        if (new Random().nextInt(10) < 8) {
            throw new RuntimeException("Task failed");
        }
        System.out.println("Task succeeded");
    }

    @Recover
    public void recover(RuntimeException e) {
        System.out.println("Recovering from failure: " + e.getMessage());
    }
}
```

---

## 5. Recovering from Failure using `@Recover`

The `@Recover` annotation is used to define a fallback method that will be called when all retry attempts fail. 

### Key Points:
- The method annotated with `@Recover` must have the same return type as the retryable method.
- It must accept the exception type being handled as a parameter.

Example:
```java
@Recover
public String recoverFromFailure(RuntimeException e) {
    System.out.println("Recovery logic for exception: " + e.getMessage());
    return "Fallback result";
}
```

In the above example, when all retries fail, the `recoverFromFailure()` method will be called, and the fallback result will be returned.

---

## 6. Flowchart: Retry Mechanism

Below is a textual representation of the Spring Boot retry mechanism flowchart:

```
+------------------------------------------+
|      Start Application                   |
+------------------------------------------+
               |
               V
+------------------------------------------+
|  Method Invoked (e.g., performTask)      |
+------------------------------------------+
               |
               V
+------------------------------------------+
|  Is Retryable? (Check @Retryable or RT)  |
+------------------------------------------+
   Yes |                           No |
       V                              V
+-------------------------------------+  +-----------------------------+
|  Try Method Execution               |  |  No retry, continue normal  |
|                                     |  |  method execution           |
+-------------------------------------+  +-----------------------------+
               |
               V
+------------------------------------------+
|  Method Execution Fails?                 |
+------------------------------------------+
   Yes |                           No |
       V                              V
+------------------------------------------+ +-----------------------------+
|  Retry Policy Check (max attempts?)      | |  Method succeeds, continue  |
+------------------------------------------+ +-----------------------------+
   Yes |                           No |
       V                              V
+------------------------------------------+ +-----------------------------+
|  Retry Backoff (delay before retry)      | |  Call @Recover method       |
+------------------------------------------+ +-----------------------------+
               |
               V
+------------------------------------------+
|  Retry Method Execution                  |
+------------------------------------------+
               |
               V
+------------------------------------------+
|  Retry Limit Reached or Success?         |
+------------------------------------------+
  Retry Limit |                 Success |
              V                         V
+------------------------------------------+ +-----------------------------+
|  Call @Recover method                    | |  Continue normal execution  |
+------------------------------------------+ +-----------------------------+
```

---

## 7. Multiple Recover Methods

In Spring Retry, while it might seem like there is no explicit way to specify a "recover" method in the `@Retryable` annotation itself, you can still have multiple recover methods in the same class. Spring Retry allows you to define **different recovery methods** by relying on **method signatures**.

The framework will match the appropriate recovery method based on the **parameters** of the method, so each `@Recover` method corresponds to a specific exception type (or a list of exception types) that it's designed to handle.

Here's how it works:

- The recovery method must have the **same return type** as the retryable method.
- The **first parameter** of the recovery method must be the exception that triggered the retry attempts.
- Any **additional parameters** must match the parameters of the retryable method, in the same order.

### Example: Multiple `@Recover` Methods

```java
@Service
public class ApiService {

    private final RestTemplate restTemplate;

    @Autowired
    public ApiService(RestTemplateBuilder restTemplateBuilder) {
        this.restTemplate = restTemplateBuilder.build();
    }

    // Retryable method for HTTP client errors
    @Retryable(
        value = { HttpClientErrorException.class },
        maxAttempts = 4,
        backoff = @Backoff(delay = 1000)
    )
    public String invokeClientApi(String url) {
        return restTemplate.getForObject(url, String.class);
    }

    // Retryable method for HTTP server errors
    @Retryable(
        value = { HttpServerErrorException.class },
        maxAttempts = 4,
        backoff = @Backoff(delay = 1000)
    )
    public String invokeServerApi(String url) {
        return restTemplate.getForObject(url, String.class);
    }

    // Recovery method for HTTP client errors
    @Recover
    public String recoverClientError(HttpClientErrorException e, String url) {
        return "Client error fallback for URL: " + url;
    }

    // Recovery method for HTTP server errors
    @Recover
    public String recoverServerError(HttpServerErrorException e, String url) {
        return "Server error fallback for URL: " + url;
    }
}
```

### Key Points:
- **Different `@Recover` methods** are defined based on the exception type (e.g., `HttpClientErrorException` vs. `HttpServerErrorException`).
- The recovery method's **first parameter** is the exception type that triggered the retry failure. This is how Spring determines which recover method to invoke.
- **Additional parameters** (like `url` in the example) must match the parameters of the corresponding retryable method.

### How It Works:
1. If the retryable method (`invokeClientApi` or `invokeServerApi`) fails after exhausting the retry attempts, Spring will look for a recovery method based on the exception type.
2. The recovery method (`recoverClientError` or `recoverServerError`) will be called depending on whether a `HttpClientErrorException` or a `HttpServerErrorException` occurred.

### Summary:
A class can have **multiple `@Recover` methods**. Spring matches the correct recovery method based on the exception type and method signature, so you can create recover methods for different types of exceptions in the same class.

## 8. Conclusion

Spring Boot’s retry mechanism provides a simple and powerful way to retry failed operations, with both **programmatic** (RetryTemplate) and **declarative** (annotations) support. Additionally, recovery methods allow handling failure gracefully after retry attempts are exhausted.

Whether you choose to use `RetryTemplate` or `@Retryable`, both approaches are robust solutions for retrying failed methods in Spring applications.