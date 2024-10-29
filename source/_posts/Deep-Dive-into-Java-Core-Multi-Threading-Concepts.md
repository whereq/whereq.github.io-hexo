---
title: Deep Dive into Java Core Multi-Threading Concepts
date: 2024-10-28 22:04:14
categories:
- Deep Dive
- Java Core
- Multi-Threading
tags:
- Deep Dive
- Java Core
- Multi-Threading
---

- [1. Java Thread Model](#1-java-thread-model)
- [2. Java Memory Model (JMM)](#2-java-memory-model-jmm)
  - [Java Thread Memory Model Diagram](#java-thread-memory-model-diagram)
  - [JMM Key Concepts:](#jmm-key-concepts)
- [3. Thread Lifecycle and State Machine](#3-thread-lifecycle-and-state-machine)
  - [Thread State Machine Diagram](#thread-state-machine-diagram)
- [4. Key Threading Classes in Java](#4-key-threading-classes-in-java)
  - [4.1 Thread Class](#41-thread-class)
  - [4.2 Runnable Interface](#42-runnable-interface)
  - [4.3 Callable and Future](#43-callable-and-future)
  - [4.4 Executors Framework](#44-executors-framework)
- [5. Synchronization and Locks](#5-synchronization-and-locks)
  - [Synchronized](#synchronized)
  - [ReentrantLock](#reentrantlock)
- [6. Real-Life Production Example](#6-real-life-production-example)
  - [Explanation](#explanation)

---

Java multithreading is a fundamental part of the language that allows concurrent execution of multiple parts of a program to achieve greater efficiency and responsiveness. This article explores the latest Java version's core multithreading concepts, the Java Memory Model (JMM), thread lifecycle, key classes, and real-life examples.

<a name="java-thread-model"></a>
## 1. Java Thread Model

Java's thread model is based on the OS thread model, providing abstraction to manage and create threads directly. Java threads allow programs to perform multiple tasks concurrently, improving resource utilization. Thread priorities help in determining the order of thread execution, though scheduling ultimately depends on the underlying OS.

<a name="java-memory-model"></a>
## 2. Java Memory Model (JMM)

<a name="java-memory-model-diagram"></a>
### Java Thread Memory Model Diagram

```
+-------------------+       +-------------------+
|                   |       |                   |
|    Main Memory    |<----->|    Thread 1       |
|                   |       |                   |
+-------------------+       +-------------------+
        |                           |
        |                           |
        v                           v
+-------------------+       +-------------------+
|                   |       |                   |
|    Thread 2       |<----->|    Thread 3       |
|                   |       |                   |
+-------------------+       +-------------------+
```

The Java Memory Model (JMM) defines how threads interact through memory and what values can be read or written by threads. The JMM provides:
1. **Visibility**: Changes made by one thread are visible to other threads.
2. **Atomicity**: Certain actions are indivisible or atomic.
3. **Ordering**: The sequence in which instructions are executed.

### JMM Key Concepts:
- **Happens-Before Relationship**: Ensures that memory writes by one specific action are visible to another specific action.
- **Volatile Keyword**: Variables declared as `volatile` ensure visibility of changes across threads.
- **Synchronized Blocks**: Using `synchronized` enforces an order of access to variables between threads.

<a name="thread-lifecycle-and-state-machine"></a>
## 3. Thread Lifecycle and State Machine

Java threads go through several states:

| State         | Description                                                                             |
|---------------|-----------------------------------------------------------------------------------------|
| New           | The thread is created but not yet started.                                              |
| Runnable      | The thread is eligible to run but may be waiting for a CPU cycle.                       |
| Blocked       | The thread is waiting to acquire a lock to enter a `synchronized` block.                |
| Waiting       | The thread is waiting indefinitely for another thread to perform an action.             |
| Timed Waiting | The thread is waiting for another thread for a specified period.                        |
| Terminated    | The thread has completed execution.                                                     |


<a name="java-state-machine-diagram"></a>
### Thread State Machine Diagram 

```plaintext
NEW -> Runnable -> BLOCKED -> WAITING <-> TIMED_WAITING -> TERMINATED
```

```
+-------------------+       +-------------------+
|                   |       |                   |
|    NEW            |------>|    RUNNABLE       |
|                   |       |                   |
+-------------------+       +-------------------+
        |                           |
        |                           |
        v                           v
+-------------------+       +-------------------+
|                   |       |                   |
|    BLOCKED        |<----->|    WAITING        |
|                   |       |                   |
+-------------------+       +-------------------+
        |                           |
        |                           |
        v                           v
+-------------------+       +-------------------+
|                   |       |                   |
|    TIMED_WAITING  |<----->|    TERMINATED     |
|                   |       |                   |
+-------------------+       +-------------------+
```

<a name="key-threading-classes"></a>
## 4. Key Threading Classes in Java

<a name="thread-class"></a>
### 4.1 Thread Class

The `Thread` class provides methods to create, start, and manage threads. You can define a custom thread by extending the `Thread` class and overriding its `run()` method.

Example:

```java
public class MyThread extends Thread {
    @Override
    public void run() {
        System.out.println("Thread is running");
    }
}

MyThread thread = new MyThread();
thread.start();
```

<a name="runnable-interface"></a>
### 4.2 Runnable Interface

The `Runnable` interface is a functional interface representing a task to be executed on a thread. It separates the task from the thread itself, allowing reusability and flexibility.

Example:

```java
public class MyRunnable implements Runnable {
    @Override
    public void run() {
        System.out.println("Runnable is running");
    }
}

Thread thread = new Thread(new MyRunnable());
thread.start();
```

<a name="callable-and-future"></a>
### 4.3 Callable and Future

`Callable` allows tasks to return results or throw checked exceptions, while `Future` represents the result of an asynchronous computation.

Example:

```java
Callable<Integer> task = () -> {
    Thread.sleep(1000);
    return 123;
};

ExecutorService executor = Executors.newFixedThreadPool(1);
Future<Integer> future = executor.submit(task);

Integer result = future.get();  // blocks until the result is available
executor.shutdown();
```

<a name="executors-framework"></a>
### 4.4 Executors Framework

The Executors framework provides a higher-level API for thread management, thread pooling, and scheduling, abstracting the creation and management of threads.

Example:

```java
ExecutorService executor = Executors.newFixedThreadPool(5);
for (int i = 0; i < 10; i++) {
    executor.submit(() -> {
        System.out.println("Task running");
    });
}
executor.shutdown();
```

<a name="synchronization-and-locks"></a>
## 5. Synchronization and Locks

Java provides several ways to control access to shared resources, including synchronized methods and blocks, the `ReentrantLock` class, and the `ReadWriteLock`.

### Synchronized

The `synchronized` keyword ensures that only one thread can access a method or block at a time.

```java
public synchronized void increment() {
    count++;
}
```

### ReentrantLock

`ReentrantLock` provides explicit lock mechanisms, giving more control over thread coordination.

```java
Lock lock = new ReentrantLock();
lock.lock();
try {
    // critical section
} finally {
    lock.unlock();
}
```

<a name="real-life-production-example"></a>
## 6. Real-Life Production Example

Consider a production scenario where we have multiple tasks needing parallel execution but require control over access to shared resources. Here’s a multithreaded solution using a `ScheduledExecutorService` in a banking transaction processing system.

```java
import java.util.concurrent.*;

public class TransactionProcessor {
    private final ScheduledExecutorService scheduler = Executors.newScheduledThreadPool(3);

    public void startProcessing() {
        Runnable task = () -> {
            // Simulate fetching and processing a transaction
            System.out.println("Processing transaction: " + Thread.currentThread().getName());
            try {
                Thread.sleep(2000);  // Simulating a transaction processing delay
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            }
        };

        // Schedule at fixed rate
        scheduler.scheduleAtFixedRate(task, 0, 5, TimeUnit.SECONDS);
    }

    public static void main(String[] args) {
        TransactionProcessor processor = new TransactionProcessor();
        processor.startProcessing();
    }
}
```

### Explanation
- We use `ScheduledExecutorService` for periodic transaction processing.
- The thread pool’s `scheduleAtFixedRate` method ensures that the task runs every 5 seconds, enabling controlled execution with set intervals.
- This design is beneficial in systems where tasks need continuous or scheduled processing but must prevent overloading resources.

---

This article provides an overview of multithreading concepts, memory models, core classes, and real-world examples. Java’s concurrency mechanisms offer flexibility and control, making it essential to use these constructs carefully to manage resources efficiently and avoid pitfalls such as deadlocks and race conditions.