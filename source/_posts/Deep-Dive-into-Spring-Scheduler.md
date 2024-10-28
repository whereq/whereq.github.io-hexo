---
title: Deep Dive into Spring Scheduler
date: 2024-10-28 16:20:09
categories:
- Deep Dive
- Spring Boot
- Spring Scheduler
tags:
- Deep Dive
- Spring Boot
- Spring Scheduler
---

- [Introduction](#introduction)
- [Understanding Spring Scheduler Basics](#understanding-spring-scheduler-basics)
  - [Enabling Scheduling](#enabling-scheduling)
  - [Simple Scheduling with `@Scheduled`](#simple-scheduling-with-scheduled)
- [Scheduling Scenarios](#scheduling-scenarios)
  - [Sequential Job Execution](#sequential-job-execution)
  - [Parallel Job Execution](#parallel-job-execution)
  - [Job Waiting Until Previous Job Completes](#job-waiting-until-previous-job-completes)
- [Cron Expressions and Fixed Rates](#cron-expressions-and-fixed-rates)
  - [Fixed Rate vs Fixed Delay](#fixed-rate-vs-fixed-delay)
  - [Using Cron Expressions](#using-cron-expressions)
- [Conditional Scheduling with Annotations](#conditional-scheduling-with-annotations)
  - [Conditional Scheduling Example](#conditional-scheduling-example)
- [Real-World Production Sample Code](#real-world-production-sample-code)
- [Conclusion](#conclusion)

---

<a name="introduction"></a>
## Introduction

**Spring Scheduler** provides a simple way to schedule tasks within a Spring application. It offers flexibility in managing **time-based events** and **task execution**, allowing developers to configure tasks in a variety of ways, including sequential, parallel, and conditional execution.

---

<a name="understanding-spring-scheduler-basics"></a>
## Understanding Spring Scheduler Basics

Spring Scheduler uses annotations to simplify the scheduling of methods within beans. The primary annotation used is `@Scheduled`, which can define various timing configurations, such as fixed rate, fixed delay, and cron expressions.

<a name="enabling-scheduling"></a>
### Enabling Scheduling

Before defining tasks, enable scheduling in your Spring application by adding the `@EnableScheduling` annotation.

```java
import org.springframework.context.annotation.Configuration;
import org.springframework.scheduling.annotation.EnableScheduling;

@Configuration
@EnableScheduling
public class SchedulingConfig {
    // Any other scheduling-related configuration if needed
}
```

<a name="simple-scheduling-with-scheduled"></a>
### Simple Scheduling with `@Scheduled`

The `@Scheduled` annotation can be applied to any method to enable scheduling. Below is a simple example:

```java
import org.springframework.scheduling.annotation.Scheduled;
import org.springframework.stereotype.Component;

@Component
public class SimpleScheduler {

    @Scheduled(fixedRate = 5000)
    public void printMessage() {
        System.out.println("This message prints every 5 seconds.");
    }
}
```

In this example, `printMessage()` runs every 5 seconds.

---

<a name="scheduling-scenarios"></a>
## Scheduling Scenarios

Spring Scheduler supports various scenarios for task scheduling, which can be useful for specific use cases in a production environment.

<a name="sequential-job-execution"></a>
### Sequential Job Execution

In sequential execution, jobs run in a specific order, with each job waiting for the previous one to complete before starting.

**Example: Sequential Scheduling Using Fixed Delay**

```java
import org.springframework.scheduling.annotation.Scheduled;
import org.springframework.stereotype.Component;

@Component
public class SequentialScheduler {

    @Scheduled(fixedDelay = 10000)
    public void job1() {
        System.out.println("Job 1 started.");
        // simulate a delay
        try { Thread.sleep(2000); } catch (InterruptedException ignored) {}
        System.out.println("Job 1 completed.");
    }

    @Scheduled(fixedDelay = 10000)
    public void job2() {
        System.out.println("Job 2 started after Job 1.");
        // simulate a delay
        try { Thread.sleep(3000); } catch (InterruptedException ignored) {}
        System.out.println("Job 2 completed.");
    }
}
```

With `fixedDelay`, Job 2 only begins after Job 1 completes.

<a name="parallel-job-execution"></a>
### Parallel Job Execution

To execute jobs in parallel, we configure a task executor that runs multiple tasks concurrently.

**Example: Parallel Scheduling Using a Custom Task Executor**

```java
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.scheduling.annotation.EnableAsync;
import org.springframework.scheduling.concurrent.ThreadPoolTaskExecutor;

@Configuration
@EnableAsync
public class ParallelSchedulingConfig {

    @Bean
    public ThreadPoolTaskExecutor taskExecutor() {
        ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor();
        executor.setCorePoolSize(2); // Number of concurrent tasks allowed
        executor.setMaxPoolSize(5);
        executor.initialize();
        return executor;
    }
}

import org.springframework.scheduling.annotation.Scheduled;
import org.springframework.scheduling.annotation.Async;
import org.springframework.stereotype.Component;

@Component
public class ParallelScheduler {

    @Async
    @Scheduled(fixedRate = 10000)
    public void jobA() {
        System.out.println("Parallel Job A started.");
    }

    @Async
    @Scheduled(fixedRate = 10000)
    public void jobB() {
        System.out.println("Parallel Job B started concurrently with Job A.");
    }
}
```

With `@Async`, `jobA` and `jobB` run concurrently, allowing for parallel execution.

<a name="job-waiting-until-previous-job-completes"></a>
### Job Waiting Until Previous Job Completes

To configure jobs to wait for previous jobs to finish, use `fixedDelay` with asynchronous methods to ensure that each task waits until its predecessor is completed.

**Diagram: Job Waiting until Previous Job Completes**
```
   Job 1   ---> Wait --->   Job 2   ---> Wait --->   Job 3
```

```java
import org.springframework.scheduling.annotation.Scheduled;
import org.springframework.stereotype.Component;

@Component
public class WaitingScheduler {

    @Scheduled(fixedDelay = 12000)
    public void job() {
        System.out.println("Job started and waits until the previous instance completes.");
        // Simulate processing time
        try { Thread.sleep(5000); } catch (InterruptedException ignored) {}
        System.out.println("Job completed.");
    }
}
```

In this scenario, each new execution waits until the previous one finishes.

---

<a name="cron-expressions-and-fixed-rates"></a>
## Cron Expressions and Fixed Rates

Spring Scheduler allows defining precise timing patterns using **fixed rate**, **fixed delay**, and **cron expressions**.

<a name="fixed-rate-vs-fixed-delay"></a>
### Fixed Rate vs Fixed Delay

- **Fixed Rate**: Executes a task at a specified interval, regardless of task completion time.
- **Fixed Delay**: Executes a task with a delay between the end of the previous execution and the start of the next.

```java
@Scheduled(fixedRate = 60000) // Executes every 60 seconds
public void fixedRateTask() {
    System.out.println("Fixed Rate Task");
}

@Scheduled(fixedDelay = 60000) // Executes with a 60-second delay after completion
public void fixedDelayTask() {
    System.out.println("Fixed Delay Task");
}
```

<a name="using-cron-expressions"></a>
### Using Cron Expressions

Cron expressions allow precise scheduling based on time. Format: `second minute hour day month weekday`.

**Example: Scheduling a Task for 5:00 AM Every Day**

```java
@Scheduled(cron = "0 0 5 * * ?")
public void dailyTask() {
    System.out.println("This task runs at 5:00 AM every day.");
}
```

---

<a name="conditional-scheduling-with-annotations"></a>
## Conditional Scheduling with Annotations

Spring allows scheduling tasks based on conditions. Combining `@ConditionalOnProperty` with `@Scheduled`, we can configure tasks to run only when certain conditions are met.

<a name="conditional-scheduling-example"></a>
### Conditional Scheduling Example

This example schedules a job only if a specific property is set to `true`.

```java
import org.springframework.boot.autoconfigure.condition.ConditionalOnProperty;
import org.springframework.scheduling.annotation.Scheduled;
import org.springframework.stereotype.Component;

@Component
@ConditionalOnProperty(name = "scheduler.enabled", havingValue = "true")
public class ConditionalScheduler {

    @Scheduled(fixedRate = 10000)
    public void conditionalTask() {
        System.out.println("This task runs only if 'scheduler.enabled=true'.");
    }
}
```

To enable or disable this task, add `scheduler.enabled=true` or `false` in the configuration.

---

<a name="real-world-production-sample-code"></a>
## Real-World Production Sample Code

Below is a production-level example that schedules multiple jobs with different timings and conditions.

```java
import org.springframework.boot.autoconfigure.condition.ConditionalOnProperty;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.scheduling.annotation.EnableScheduling;
import org.springframework.scheduling.annotation.Scheduled;
import org.springframework.scheduling.concurrent.ThreadPoolTaskExecutor;

@Configuration
@EnableScheduling
public class ProductionSchedulerConfig {

    @Bean
    public ThreadPoolTaskExecutor taskExecutor() {
        ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor();
        executor.setCorePoolSize(3);
        executor.setMaxPoolSize(5);
        executor.initialize();
        return executor;
    }
}

import org.springframework.scheduling.annotation.Async;
import org.springframework.scheduling.annotation.Scheduled;
import org.springframework.stereotype.Component;

@Component
@ConditionalOnProperty(name = "scheduler.run", havingValue = "true", matchIfMissing = true)
public class ProductionScheduler {

    @Async
    @Scheduled(cron = "0 0 12 * * ?") // Executes at noon daily
    public void dailySummaryJob() {
       

 System.out.println("Executing daily summary job...");
    }

    @Async
    @Scheduled(fixedDelay = 15000) // Executes every 15 seconds after the previous execution ends
    public void periodicCleanupJob() {
        System.out.println("Running periodic cleanup job...");
    }
}
```

---

<a name="conclusion"></a>
## Conclusion

Spring Scheduler provides flexibility and control over task scheduling. By using `@Scheduled` with different configurations, we can create complex scheduling workflows that address specific business needs.

