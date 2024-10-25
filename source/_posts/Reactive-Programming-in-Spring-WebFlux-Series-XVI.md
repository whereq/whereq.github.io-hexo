---
title: Reactive Programming in Spring WebFlux Series - XVI
date: 2024-10-24 14:07:41
categories:
- WebFlux
- Spring Boot
- Reactive
tags:
- WebFlux
- Spring Boot
- Reactive
---


In the previous fifteen articles, we have covered the foundational concepts of Spring WebFlux, Reactor, error handling, data stream transformation, reactive database access, performance optimization, security, testing, deployment and operations, best practices, common pitfalls, integration with microservices architecture, and real-time data push. This article will delve into implementing effective logging and monitoring in Spring WebFlux applications to ensure system observability and stability.

- [Logging and Monitoring](#logging-and-monitoring)
  - [Introduction to Logging](#introduction-to-logging)
    - [Adding Dependencies](#adding-dependencies)
    - [Configuring Logback](#configuring-logback)
    - [Using Logging in Code](#using-logging-in-code)
  - [Introduction to Monitoring](#introduction-to-monitoring)
    - [Spring Boot Actuator](#spring-boot-actuator)
      - [Adding Actuator Dependencies](#adding-actuator-dependencies)
      - [Configuring Actuator](#configuring-actuator)
      - [Using Actuator Endpoints](#using-actuator-endpoints)
    - [Monitoring with Prometheus and Grafana](#monitoring-with-prometheus-and-grafana)
      - [Adding Prometheus Dependencies](#adding-prometheus-dependencies)
      - [Configuring Prometheus](#configuring-prometheus)
      - [Configuring Prometheus Server](#configuring-prometheus-server)
      - [Visualizing with Grafana](#visualizing-with-grafana)
  - [Conclusion](#conclusion)

---

# Logging and Monitoring

---
<a name="introduction-to-logging"></a>
## Introduction to Logging

Logging is a critical component of application development. It helps developers understand the system's operational state, diagnose issues, and perform performance analysis. Spring WebFlux can integrate with various logging frameworks such as SLF4J, Logback, and Log4j2.

<a name="adding-dependencies"></a>
### Adding Dependencies

To begin, add the Logback dependency to your project (Spring Boot uses Logback by default):

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-webflux</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-logging</artifactId>
    </dependency>
</dependencies>
```

<a name="configuring-logback"></a>
### Configuring Logback

Create or modify the `logback-spring.xml` file in the `src/main/resources` directory:

```xml
<configuration>
    <include resource="org/springframework/boot/logging/logback/base.xml"/>
    
    <logger name="org.springframework.web" level="DEBUG"/>
    <logger name="com.example" level="DEBUG"/>
    
    <appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender">
        <encoder>
            <pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] %-5level %logger{36} - %msg%n</pattern>
        </encoder>
    </appender>
    
    <root level="INFO">
        <appender-ref ref="STDOUT"/>
    </root>
</configuration>
```

<a name="using-logging-in-code"></a>
### Using Logging in Code

Use SLF4J to log messages in your code:

```java
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.stereotype.Service;

@Service
public class BookService {
    private static final Logger logger = LoggerFactory.getLogger(BookService.class);
    
    public void addBook(Book book) {
        logger.info("Adding book: {}", book.getTitle());
        // Other logic
    }
}
```

<a name="introduction-to-monitoring"></a>
## Introduction to Monitoring

Monitoring is crucial for ensuring system stability and performance. By monitoring, we can promptly detect and address system issues. Spring Boot Actuator and Prometheus are commonly used monitoring tools.

<a name="spring-boot-actuator"></a>
### Spring Boot Actuator

Spring Boot Actuator provides a wealth of monitoring and management endpoints, making it easier for developers to monitor and manage applications.

<a name="adding-actuator-dependencies"></a>
#### Adding Actuator Dependencies

Add the Actuator dependency to your project:

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-actuator</artifactId>
    </dependency>
</dependencies>
```

<a name="configuring-actuator"></a>
#### Configuring Actuator

Enable Actuator endpoints in `application.properties`:

```properties
management.endpoints.web.exposure.include=health,info,metrics,loggers,httptrace
management.endpoint.health.show-details=always
```

<a name="using-actuator-endpoints"></a>
#### Using Actuator Endpoints

Access `http://localhost:8080/actuator` to view all available Actuator endpoints, such as health checks (`health`), application information (`info`), and loggers (`loggers`).

<a name="monitoring-with-prometheus-and-grafana"></a>
### Monitoring with Prometheus and Grafana

Prometheus and Grafana are popular open-source monitoring and visualization tools suitable for monitoring Spring WebFlux applications.

<a name="adding-prometheus-dependencies"></a>
#### Adding Prometheus Dependencies

Add the Prometheus dependency to your project:

```xml
<dependencies>
    <dependency>
        <groupId>io.micrometer</groupId>
        <artifactId>micrometer-registry-prometheus</artifactId>
    </dependency>
</dependencies>
```

<a name="configuring-prometheus"></a>
#### Configuring Prometheus

Configure Prometheus in `application.properties`:

```properties
management.endpoints.web.exposure.include=prometheus
management.metrics.export.prometheus.enabled=true
```

<a name="configuring-prometheus-server"></a>
#### Configuring Prometheus Server

Add the Spring WebFlux application's monitoring configuration to Prometheus's `prometheus.yml` file:

```yaml
scrape_configs:
  - job_name: 'spring-webflux-app'
    metrics_path: '/actuator/prometheus'
    static_configs:
      - targets: ['localhost:8080']
```

<a name="visualizing-with-grafana"></a>
#### Visualizing with Grafana

Add the Prometheus data source in Grafana and create dashboards to monitor key metrics of the Spring WebFlux application, such as response times, request counts, and error rates.

<a name="conclusion"></a>
## Conclusion

In this article, we explored how to implement effective logging and monitoring in Spring WebFlux applications. By using Logback for logging and Spring Boot Actuator, Prometheus, and Grafana for monitoring, we can ensure system observability and stability. These practices are essential for maintaining the health and performance of modern reactive applications.