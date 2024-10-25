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

- [Spring Boot Retry Mechanism - Detailed Guide with Sample Code](#spring-boot-retry-mechanism---detailed-guide-with-sample-code)
  - [1. Introduction to Spring Retry](#1-introduction-to-spring-retry)
    - [Key Features:](#key-features)
  - [2. How Retry Mechanism Works](#2-how-retry-mechanism-works)
    - [Flow of Retry Mechanism:](#flow-of-retry-mechanism)
  - [3. Plain Retry Implementation using `RetryTemplate`](#3-plain-retry-implementation-using-retrytemplate)
    - [Step-by-Step Guide:](#step-by-step-guide)
    - [Key Points:](#key-points)
  - [4. Annotation-Based Retry Implementation using `@Retryable`](#4-annotation-based-retry-implementation-using-retryable)
    - [Step-by-Step Guide:](#step-by-step-guide-1)
    - [Key Points:](#key-points-1)
  - [5. Recovering from Failure using `@Recover`](#5-recovering-from-failure-using-recover)
    - [Key Points:](#key-points-2)
    - [Advanced Recovery:](#advanced-recovery)
  - [6. Flowchart: Retry Mechanism](#6-flowchart-retry-mechanism)
    - [Key Points:](#key-points-3)
  - [7. Multiple Recover Methods](#7-multiple-recover-methods)
    - [Example: Multiple `@Recover` Methods](#example-multiple-recover-methods)
    - [Key Points:](#key-points-4)
    - [How It Works:](#how-it-works)
    - [Summary:](#summary)
  - [8. Conclusion](#8-conclusion)
    - [Key Takeaways:](#key-takeaways)

---

# Spring Boot Retry Mechanism - Detailed Guide with Sample Code

---

## 1. Introduction to Spring Retry
<a name="1-introduction-to-spring-retry"></a>

Spring Boot provides a robust retry mechanism through the **Spring Retry** module. This module allows developers to automatically retry operations that fail due to certain exceptions, making it highly useful for handling transient faults in distributed systems, such as network issues, service unavailability, and more.

Spring Retry can be implemented in two primary ways:
- **Programmatically** using `RetryTemplate`.
- **Declaratively** using the `@Retryable` annotation.

Additionally, Spring Retry supports recovery methods using the `@Recover` annotation, which can be invoked when all retry attempts are exhausted.

### Key Features:
- **Retry Policies**: Define the number of retry attempts and conditions for retrying.
- **Backoff Policies**: Implement delays between retries, including fixed, exponential, and custom backoff strategies.
- **Exception Handling**: Retry based on specific exceptions or generic conditions.
- **Recovery Mechanism**: Gracefully handle failures using `@Recover` methods.

---

## 2. How Retry Mechanism Works
<a name="2-how-retry-mechanism-works"></a>

The retry mechanism in Spring operates as follows:
- When a method annotated with `@Retryable` or executed via `RetryTemplate` encounters an exception, Spring can automatically retry that method a configurable number of times.
- The retry logic can be customized based on specific exceptions or broader conditions.
- Spring supports various backoff policies, such as fixed delays, exponential backoff, and custom backoff strategies, to manage the interval between retry attempts.
- After exhausting the maximum retry attempts, a `@Recover` method can be invoked to handle the failure gracefully.

### Flow of Retry Mechanism:
1. **Method Invocation**: The retryable method is invoked.
2. **Exception Occurs**: If an exception occurs, Spring checks if the method is retryable.
3. **Retry Policy**: The retry policy determines whether to retry the method.
4. **Backoff Policy**: If retries are allowed, the backoff policy defines the delay before the next retry.
5. **Retry Execution**: The method is retried according to the defined policy.
6. **Recovery**: If all retries fail, the `@Recover` method is invoked.

---

## 3. Plain Retry Implementation using `RetryTemplate`
<a name="3-plain-retry-implementation-using-retrytemplate"></a>

The `RetryTemplate` is used to implement the retry mechanism programmatically. This approach provides fine-grained control over the retry logic and is ideal for complex scenarios.

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

2. **Define `RetryTemplate` Bean**:
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

3. **Use `RetryTemplate` in Business Logic**:
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

### Key Points:
- **RetryTemplate**: Provides a flexible way to define retry policies and backoff strategies.
- **SimpleRetryPolicy**: Defines the maximum number of retry attempts.
- **FixedBackOffPolicy**: Sets a fixed delay between retry attempts.
- **execute() Method**: Executes the retry logic within a lambda expression.

---

## 4. Annotation-Based Retry Implementation using `@Retryable`
<a name="4-annotation-based-retry-implementation-using-retryable"></a>

The annotation-based approach uses the `@Retryable` annotation to automatically retry methods without explicitly defining a `RetryTemplate`. This method is simpler and more concise, making it ideal for straightforward retry scenarios.

### Step-by-Step Guide:

1. **Add Dependencies** (same as in `RetryTemplate` approach).

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

3. **Use `@Retryable` Annotation**:
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

4. **Recover from Failure using `@Recover`**:
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

### Key Points:
- **@Retryable**: Specifies the method to be retried, along with the retry conditions and backoff policy.
- **@Recover**: Defines a fallback method to handle failures after all retries are exhausted.
- **@EnableRetry**: Enables the retry mechanism in the Spring Boot application.

---

## 5. Recovering from Failure using `@Recover`
<a name="5-recovering-from-failure-using-recover"></a>

The `@Recover` annotation is used to define a fallback method that will be called when all retry attempts fail. This method allows you to handle the failure gracefully and provide a fallback response.

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

### Advanced Recovery:
- **Multiple `@Recover` Methods**: You can define multiple recovery methods for different exception types, allowing for more granular error handling.
- **Parameter Matching**: The recovery method's parameters must match the retryable method's parameters, except for the first parameter, which must be the exception type.

---

## 6. Flowchart: Retry Mechanism
<a name="6-flowchart-retry-mechanism"></a>

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
+---------------------------------------------------+
|  Is Retryable? (Check @Retryable or RT)           |
+---------------------------------------------------+
   Yes |                                     No |
       V                                        V
+-------------------------------------+  +-----------------------------+
|  Try Method Execution               |  |  No retry, continue normal  |
|                                     |  |  method execution           |
+-------------------------------------+  +-----------------------------+
               |
               V
+--------------------------------------------------+
|  Method Execution Fails?                         |
+--------------------------------------------------+
   Yes |                                       No |
       V                                          V
+------------------------------------------+ +-----------------------------+
|  Retry Policy Check (max attempts?)      | |  Method succeeds, continue  |
+------------------------------------------+ +-----------------------------+
   Yes |                                          No |
       V                                             V
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
+-------------------------------------------------------+
|  Retry Limit Reached or Success?                      |
+-------------------------------------------------------+
  Retry Limit |                                 Success |
              V                                         V
+------------------------------------------+ +-----------------------------+
|  Call @Recover method                    | |  Continue normal execution  |
+------------------------------------------+ +-----------------------------+
```

### Key Points:
- **Retryable Check**: Determines if the method is eligible for retries.
- **Retry Policy**: Defines the number of retry attempts and conditions.
- **Backoff Policy**: Manages the delay between retry attempts.
- **Recovery Method**: Handles failures after all retries are exhausted.

---

## 7. Multiple Recover Methods
<a name="7-multiple-recover-methods"></a>

In Spring Retry, while it might seem like there is no explicit way to specify a "recover" method in the `@Retryable` annotation itself, you can still have multiple recover methods in the same class. Spring Retry allows you to define **different recovery methods** by relying on **method signatures**.

The framework will match the appropriate recovery method based on the **parameters** of the method, so each `@Recover` method corresponds to a specific exception type (or a list of exception types) that it's designed to handle.

### Example: Multiple `@Recover` Methods
<a name="example-multiple-recover-methods"></a>

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
<a name="key-points-1"></a>
- **Different `@Recover` methods** are defined based on the exception type (e.g., `HttpClientErrorException` vs. `HttpServerErrorException`).
- The recovery method's **first parameter** is the exception type that triggered the retry failure. This is how Spring determines which recover method to invoke.
- **Additional parameters** (like `url` in the example) must match the parameters of the corresponding retryable method.

### How It Works:
<a name="how-it-works"></a>
1. If the retryable method (`invokeClientApi` or `invokeServerApi`) fails after exhausting the retry attempts, Spring will look for a recovery method based on the exception type.
2. The recovery method (`recoverClientError` or `recoverServerError`) will be called depending on whether a `HttpClientErrorException` or a `HttpServerErrorException` occurred.

### Summary:
<a name="summary"></a>
A class can have **multiple `@Recover` methods**. Spring matches the correct recovery method based on the exception type and method signature, so you can create recover methods for different types of exceptions in the same class.

---

## 8. Conclusion
<a name="8-conclusion"></a>

Spring Boot’s retry mechanism provides a simple and powerful way to retry failed operations, with both **programmatic** (`RetryTemplate`) and **declarative** (`@Retryable`) support. Additionally, recovery methods allow handling failure gracefully after retry attempts are exhausted.

Whether you choose to use `RetryTemplate` or `@Retryable`, both approaches are robust solutions for retrying failed methods in Spring applications.

### Key Takeaways:
- **RetryTemplate**: Offers fine-grained control over retry logic.
- **@Retryable**: Provides a simpler, declarative approach to retries.
- **@Recover**: Enables graceful handling of failures after retries are exhausted.
- **Multiple Recover Methods**: Allows for granular error handling based on exception types.

By leveraging Spring Retry, developers can build resilient applications that can handle transient faults effectively, ensuring higher availability and reliability.