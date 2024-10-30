---
title: Best Practices of Java Core ThreadPoolExecutor
date: 2024-10-28 23:01:49
categories:
- Best Practices
- Java Core
- Multi-Threading
tags:
- Best Practices
- Java Core
- Multi-Threading
---

- [1. Why Executors is Discouraged](#1-why-executors-is-discouraged)
  - [Example Code (Not Recommended)](#example-code-not-recommended)
- [2. ThreadPoolExecutor Introduction](#2-threadpoolexecutor-introduction)
- [3. Overview of `ThreadPoolExecutor`](#3-overview-of-threadpoolexecutor)
- [4. Recommended Usage: ThreadPoolExecutor](#4-recommended-usage-threadpoolexecutor)
- [5. Core Configuration Parameters](#5-core-configuration-parameters)
    - [Best Practice: Choose configuration values based on workload type and system capacity.](#best-practice-choose-configuration-values-based-on-workload-type-and-system-capacity)
- [6. Analyzing Executors Source Code](#6-analyzing-executors-source-code)
- [7. ThreadPoolExecutor Constructors and Parameters](#7-threadpoolexecutor-constructors-and-parameters)
  - [Parameter Breakdown](#parameter-breakdown)
- [8. Advanced Queue Management](#8-advanced-queue-management)
  - [Relationships between `corePoolSize` and `maximumPoolSize`](#relationships-between-corepoolsize-and-maximumpoolsize)
  - [Understanding workQueue Types](#understanding-workqueue-types)
- [9. Thread Pool Scaling Strategies](#9-thread-pool-scaling-strategies)
    - [Recommended Scaling:](#recommended-scaling)
- [10. RejectedExecutionHandler: Strategies for Handling Excess Tasks](#10-rejectedexecutionhandler-strategies-for-handling-excess-tasks)
- [11. Enhanced Monitoring Techniques](#11-enhanced-monitoring-techniques)
- [12. Backpressure Handling in Production](#12-backpressure-handling-in-production)
    - [Diagram: Dynamic Task Flow and Backpressure](#diagram-dynamic-task-flow-and-backpressure)
- [13. Example](#13-example)
- [14. Conclusion](#14-conclusion)


<a name="1-why-executors-is-discouraged"></a>
## 1. Why Executors is Discouraged

Java’s **`Executors`** class provides several factory methods to create various types of thread pools:

```java
public static ExecutorService newSingleThreadExecutor()
public static ExecutorService newFixedThreadPool(int nThreads)
public static ExecutorService newCachedThreadPool()
public static ScheduledExecutorService newSingleThreadScheduledExecutor()
public static ScheduledExecutorService newScheduledThreadPool(int corePoolSize)
```

These convenience methods simplify thread pool creation, but they do not allow fine-tuning. **Using `Executors` to create thread pools is generally discouraged** due to potential risks, such as resource exhaustion. The official guidance is to use **`ThreadPoolExecutor`** directly, enabling full control over pool configurations, ensuring stability under varying load conditions.

### Example Code (Not Recommended)

```java
public class ThreadDemo {
    public static void main(String[] args) {
        ExecutorService es = Executors.newSingleThreadExecutor();
    }
}
```

**Why it's discouraged**: Creating a thread pool this way can lead to issues like uncontrolled resource usage. **Best practice**: use `ThreadPoolExecutor` instead.

<a name="2-threadpoolexecutor-introduction"></a>
## 2. ThreadPoolExecutor Introduction
Java's `ThreadPoolExecutor` is a cornerstone of concurrent programming. It provides a high-performance, versatile API for managing threads and handling tasks, but it requires careful configuration to prevent resource wastage, improve system responsiveness, and enhance maintainability.

---

<a name="3-overview-of-threadpoolexecutor"></a>
## 3. Overview of `ThreadPoolExecutor`
The `ThreadPoolExecutor` manages thread allocation, scaling, and task queueing while optimizing thread lifecycle to minimize system overhead. Here is a basic textual model of a `ThreadPoolExecutor` lifecycle and memory model:

```
                     +-----------------------------+
                     |         Task Queue          |
                     |   (BlockingQueue<Runnable>) |
                     +------------+----------------+
                                  |
                +-----------------v-----------------+
                |                                   |
                |       ThreadPoolExecutor          |
                |                                   |
                +----------------+------------------+
                                  |
           +----------------------+--------------------+
           |      Worker Threads                       |
           |   +-----------------------------------+   |
           |   |           Worker Thread 1         |   |
           |   |-----------------------------------|   |
           |   |           Worker Thread 2         |   |
           |   |-----------------------------------|   |
           |   |           Worker Thread N         |   |
           +-------------------------------------------+
```

- **Worker Threads**: Execute tasks within a controlled environment.
- **Task Queue**: Holds tasks awaiting execution, preventing unnecessary thread creation.
- **ThreadPoolExecutor**: Controls the execution strategy, pool scaling, and task rejection policies.

---

<a name="4-recommended-usage-threadpoolexecutor"></a>
## 4. Recommended Usage: ThreadPoolExecutor

To avoid risks, **use `ThreadPoolExecutor`** directly for creating thread pools:

```java
public class ThreadDemo {
    public static void main(String[] args) {
        ExecutorService es = new ThreadPoolExecutor(1, 1, 0L, TimeUnit.MILLISECONDS,
                new LinkedBlockingQueue<Runnable>(10), Executors.defaultThreadFactory(),
                new ThreadPoolExecutor.DiscardPolicy());
    }
}
```

<a name="5-core-configuration-parameters"></a>
## 5. Core Configuration Parameters
The `ThreadPoolExecutor` relies on several key configuration parameters:

- **corePoolSize**: Minimum number of threads in the pool.
- **maximumPoolSize**: Maximum threads allowed in the pool.
- **keepAliveTime**: The time a thread can remain idle before termination if beyond corePoolSize.
- **workQueue**: Queue holding tasks before they are assigned to threads.

#### Best Practice: Choose configuration values based on workload type and system capacity.

---

<a name="6-analyzing-executors-source-code"></a>
## 6. Analyzing Executors Source Code

The methods `newSingleThreadExecutor()`, `newFixedThreadPool(int nThreads)`, and `newCachedThreadPool()` all internally use **`ThreadPoolExecutor`**. Below are the source code implementations from JDK 8:

```java
public static ExecutorService newSingleThreadExecutor() {
    return new FinalizableDelegatedExecutorService(
        new ThreadPoolExecutor(1, 1, 0L, TimeUnit.MILLISECONDS, new LinkedBlockingQueue<Runnable>())
    );
}

public static ExecutorService newFixedThreadPool(int nThreads) {
    return new ThreadPoolExecutor(nThreads, nThreads, 0L, TimeUnit.MILLISECONDS, new LinkedBlockingQueue<Runnable>());
}

public static ExecutorService newCachedThreadPool() {
    return new ThreadPoolExecutor(0, Integer.MAX_VALUE, 60L, TimeUnit.SECONDS, new SynchronousQueue<Runnable>());
}
```

All three methods use **`ThreadPoolExecutor`** but provide different default configurations. These static methods abstract parameters, which can lead to undesired behaviors under high loads.

<a name="7-threadpoolexecutor-constructors-and-parameters"></a>
## 7. ThreadPoolExecutor Constructors and Parameters

When creating thread pools with `ThreadPoolExecutor`, initializing parameters must be specified. One of its constructors takes the following parameters:

```java
public ThreadPoolExecutor(int corePoolSize, int maximumPoolSize, long keepAliveTime, TimeUnit unit,
                          BlockingQueue<Runnable> workQueue, ThreadFactory threadFactory,
                          RejectedExecutionHandler handler)
```

### Parameter Breakdown

- **`corePoolSize`**: Number of threads to keep in the pool.
- **`maximumPoolSize`**: Maximum number of threads allowed.
- **`keepAliveTime`**: Time limit for idle threads over `corePoolSize` before termination.
- **`unit`**: Unit of time for `keepAliveTime`.
- **`workQueue`**: Queue to hold tasks before execution.
- **`threadFactory`**: Factory for creating threads, usually `Executors.defaultThreadFactory()`.
- **`handler`**: Policy for handling tasks when the pool reaches its limit.



<a name="8-advanced-queue-management"></a>
## 8. Advanced Queue Management

**Choosing the Right Queue**:
1. **SynchronousQueue**: No internal capacity, ideal for transient spikes.
2. **LinkedBlockingQueue**: Unbounded, suitable for long-running processes.
3. **ArrayBlockingQueue**: Fixed size, optimal for bounded workloads.

> **Note**: Select a queue type aligned with task load, duration, and resource availability.

### Relationships between `corePoolSize` and `maximumPoolSize`

- If **`thread count < corePoolSize`**, a new thread is created for each task.
- If **`thread count >= corePoolSize`**, tasks are added to the **task queue**.
- If the **queue is full** and **`thread count < maximumPoolSize`**, a new thread is created.
- If **`thread count >= maximumPoolSize`**, the `handler` policy determines the behavior.

> **Note**: These relationships vary with the type of queue (bounded or unbounded). Unbounded queues will prevent maximum thread creation from being reached.

<a name="understanding-workqueue-types"></a>
### Understanding workQueue Types

The **`workQueue`** holds pending tasks that await execution. Different queue types handle tasks differently:

- **`SynchronousQueue`**: A **direct hand-off** mechanism with no capacity. If no thread is immediately available, a new one is created or the task is rejected based on `maximumPoolSize`.
- **`LinkedBlockingQueue`**: An **unbounded queue**. When tasks exceed `corePoolSize`, they are added to the queue instead of creating new threads. **Warning**: Unbounded growth may exhaust memory if tasks are generated faster than processed.
- 
---

<a name="9-thread-pool-scaling-strategies"></a>
## 9. Thread Pool Scaling Strategies
Efficient scaling balances resource use against workload demands. Consider using dynamic scaling mechanisms for responsive systems.

#### Recommended Scaling:
- **FixedThreadPool**: When task load and timing are predictable.
- **CachedThreadPool**: For unpredictable or short-lived task loads.

---

<a name="10-rejectedexecutionhandler-strategies-for-handling-excess-tasks"></a>
## 10. RejectedExecutionHandler: Strategies for Handling Excess Tasks

When the pool and queue are full, the **`handler`** parameter specifies a strategy to handle tasks. Java provides four built-in policies:

- **`DiscardOldestPolicy`**: Removes the oldest task in the queue to accommodate the new task.
- **`CallerRunsPolicy`**: Executes the task in the **caller thread** if the pool is busy, providing a backpressure mechanism.
- **`DiscardPolicy`**: Silently discards the task without further action.
- **`AbortPolicy`**: Throws an **exception** to signal that the pool is at capacity.

> **Note**: Customize `RejectedExecutionHandler` based on workload characteristics. Expecially for applications where resilience is critical, **consider implementing a custom rejection handler** or using `CallerRunsPolicy` to limit load. Proper configuration can prevent issues like out-of-memory errors.

```java
RejectedExecutionHandler customHandler = new RejectedExecutionHandler() {
    public void rejectedExecution(Runnable r, ThreadPoolExecutor executor) {
        System.out.println("Task rejected: " + r.toString());
        // custom logic for rejected task
    }
};
```

---

<a name="11-enhanced-monitoring-techniques"></a>
## 11. Enhanced Monitoring Techniques

Monitoring thread and queue activity is essential for real-time scaling and troubleshooting. Use the following tools and approaches:

- **JMX (Java Management Extensions)**: For real-time metrics on threads, tasks, and system load.
- **Thread Pool Executor's `getPoolSize` and `getQueue().size()` Methods**: To track runtime capacity and pending task counts.

---

<a name="12-backpressure-handling-in-production"></a>
## 12. Backpressure Handling in Production
Backpressure handling in high-demand environments prevents overloads by controlling the flow of tasks:

- **Rejection Policies**: Define fallback strategies for rejected tasks.
- **Queue Sizing**: Set boundaries on task input rate.

#### Diagram: Dynamic Task Flow and Backpressure

```
   Client Requests
         |
    +----v----+     Backpressure Control
    | Task    |--------------------+
    | Queue   |                    |
    +---------+                    |
          |                        |
   +------v-------+                |
   | Thread       |                |
   | Pool         |                |
   +--------------+----------------+
```

---

<a name="13-example"></a>
## 13. Example

```java
import java.util.concurrent.*;

public class OptimizedThreadPoolExecutor {
    public static void main(String[] args) {
        // 1. Configure core parameters with scaling strategy
        ThreadPoolExecutor executor = new ThreadPoolExecutor(
            4,                // core pool size
            10,               // maximum pool size
            30,               // keep-alive time
            TimeUnit.SECONDS, // keep-alive time unit
            new ArrayBlockingQueue<>(100), // task queue type
            new ThreadPoolExecutor.CallerRunsPolicy() // rejection policy
        );

        // 2. Simulate task submission
        for (int i = 0; i < 200; i++) {
            executor.submit(new Task(i));
        }

        executor.shutdown();
    }

    static class Task implements Runnable {
        private final int taskId;

        Task(int taskId) {
            this.taskId = taskId;
        }

        @Override
        public void run() {
            System.out.println("Executing Task " + taskId);
            try {
                Thread.sleep(500); // Simulate work
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            }
        }
    }
}
```

---

<a name="14-conclusion"></a>
## 14. Conclusion
**Summary**: Use `ThreadPoolExecutor` directly with appropriately sized pools, tailored queue types, and effective rejection policies to ensure optimal, safe performance. Mastering `ThreadPoolExecutor` for complex, production-grade applications requires an understanding of core parameters, workload patterns, and monitoring needs. These best practices are intended to improve performance, efficiency, and system resilience in multithreaded environments.