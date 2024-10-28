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
- [All-In-One Sample](#all-in-one-sample)
- [ConditionalOnProperty with Scheduled Job in Depth](#conditionalonproperty-with-scheduled-job-in-depth)
  - [Scenario: Running a Scheduled Job in Only One Pod in a Kubernetes Cluster](#scenario-running-a-scheduled-job-in-only-one-pod-in-a-kubernetes-cluster)
    - [Solution Outline](#solution-outline)
    - [Steps to Implement](#steps-to-implement)
    - [Additional Consideration: Use Leader Election](#additional-consideration-use-leader-election)
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

<a name="all-in-one-sample"></a>
## All-In-One Sample

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
@ConditionalOnProperty(name = "scheduler.enabled", havingValue = "true", matchIfMissing = true)
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

<a name="conditionalonproperty-with-scheduled-job-in-depth"></a>
## ConditionalOnProperty with Scheduled Job in Depth
The `@ConditionalOnProperty` annotation in Spring is typically used to conditionally enable or disable beans based on specific configuration properties. In this case:

```java
@ConditionalOnProperty(name = "scheduler.enabled", havingValue = "true")
```

This will activate the annotated bean only if the `scheduler.enabled` property is set to `true` in your configuration (like in `application.properties` or environment variables).

### Scenario: Running a Scheduled Job in Only One Pod in a Kubernetes Cluster

When deploying a Spring application in a Kubernetes cluster across multiple containers (pods), ensuring that a scheduled job only runs in one specific pod can be tricky because Spring's `@Scheduled` jobs will run in every instance where the job bean is active. Here’s how to solve this problem by using the `@ConditionalOnProperty` annotation in combination with Kubernetes configurations.

#### Solution Outline

1. **Set a Leader Pod**: Configure only one of the pods to have `scheduler.enabled=true` using Kubernetes annotations and environment variables, making it the leader pod responsible for running the scheduled job.
2. **Configure Pods with Unique Environment Variables**: Use Kubernetes `configMaps` or `Secrets` to assign the `scheduler.enabled` property as `true` only in the leader pod’s environment. Other pods should not have this property set to `true`, so they won’t run the job.

#### Steps to Implement

1. **Create a ConfigMap or Secret for the Scheduler Property**:
   - Define a ConfigMap that holds the `scheduler.enabled` setting for the lead pod.
   
   ```yaml
   apiVersion: v1
   kind: ConfigMap
   metadata:
     name: scheduler-config
   data:
     scheduler.enabled: "true"
   ```

2. **Use Pod Annotations to Select the Leader Pod**:
   - When deploying multiple replicas, use a label selector or custom script to ensure only one pod is given the `scheduler.enabled=true` setting.

3. **Configure the Deployment to Pass the `scheduler.enabled` Environment Variable**:
   - In your Kubernetes Deployment file, you can use `envFrom` to load the environment variables from the ConfigMap, but with a selector so only one pod uses this ConfigMap.
   
   ```yaml
   apiVersion: apps/v1
   kind: Deployment
   metadata:
     name: scheduler-job
   spec:
     replicas: 3  # Running multiple instances
     selector:
       matchLabels:
         app: scheduler-app
     template:
       metadata:
         labels:
           app: scheduler-app
       spec:
         containers:
           - name: scheduler-container
             image: your-image
             envFrom:
               - configMapRef:
                   name: scheduler-config
             env:
               - name: POD_NAME
                 valueFrom:
                   fieldRef:
                     fieldPath: metadata.name
   ```

4. **Conditional Logic in Spring Boot**:
   - With `@ConditionalOnProperty`, the `@Scheduled` method will only activate if the `scheduler.enabled` property is `true`. So, only the pod with the `scheduler.enabled=true` environment variable will run the job.

   ```java
   @Component
   @ConditionalOnProperty(name = "scheduler.enabled", havingValue = "true")
   public class ScheduledJob {

       @Scheduled(fixedRate = 5000)
       public void performTask() {
           System.out.println("Scheduled job is running on the leader pod!");
       }
   }
   ```

#### Additional Consideration: Use Leader Election

For robust setups, you could implement leader election to dynamically select the leader pod. This can be achieved using tools like [Spring Cloud Kubernetes Leader Election](https://spring.io/projects/spring-cloud-kubernetes) or implementing a custom leader election mechanism based on a shared lock in an external data store, such as Redis or ZooKeeper.

By setting `scheduler.enabled=true` conditionally based on which pod is the elected leader, you can ensure the scheduled job runs in only one container at a time, with the ability to switch seamlessly if the leader pod goes down.

<a name="conclusion"></a>
## Conclusion

Spring Scheduler provides flexibility and control over task scheduling. By using `@Scheduled` with different configurations, we can create complex scheduling workflows that address specific business needs.

