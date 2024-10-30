---
title: Deep Dive into Executors and ThreadPoolExecutor
date: 2024-10-29 11:08:29
categories:
- Deep Dive
- Java Core
- Multi-Threading
tags:
- Deep Dive
- Java Core
- Multi-Threading
---

- [1. Overview of Thread Pool Scaling Strategies](#1-overview-of-thread-pool-scaling-strategies)
- [2. FixedThreadPool - Predictable Task Loads](#2-fixedthreadpool---predictable-task-loads)
- [3. CachedThreadPool - Unpredictable, Short-Lived Tasks](#3-cachedthreadpool---unpredictable-short-lived-tasks)
- [4. Dynamic Scaling with Flexible Configuration](#4-dynamic-scaling-with-flexible-configuration)
- [Detailed Steps](#detailed-steps)
- [5. Thread Pool Strategy Decision](#5-thread-pool-strategy-decision)
- [6. Drawbacks of Using `Executors` to Create Thread Pools](#6-drawbacks-of-using-executors-to-create-thread-pools)
- [7. Best Practices Solution: Using `ThreadPoolExecutor` Directly](#7-best-practices-solution-using-threadpoolexecutor-directly)
- [8. `ThreadPoolExecutor` with Best Practice Configurations](#8-threadpoolexecutor-with-best-practice-configurations)
- [9. Summary](#9-summary)

---

Scaling a **thread pool** efficiently is essential to balance resource usage with workload demands, ensuring performance without over-provisioning or exhausting system resources. Below are common scaling strategies, including scenarios for using each and detailed production code examples with comments.

<a name="1-overview-thread-pool-scaling-strategies"></a>
### 1. Overview of Thread Pool Scaling Strategies

In thread pool management, the primary goal is to allocate **just enough threads** to handle tasks effectively without overloading the CPU or running into resource constraints. The **Executor framework** in Java provides tools for **dynamic scaling** via multiple built-in pool types and strategies, such as `FixedThreadPool` for predictable workloads and `CachedThreadPool` for on-demand scaling.

---

<a name="2-fixedthreadpool---predictable-task-loads"></a>
### 2. FixedThreadPool - Predictable Task Loads

For workloads where the number of concurrent tasks is consistent or predictable, using a `FixedThreadPool` is efficient. The pool is created with a predefined number of threads, ensuring no new threads are added or removed during runtime. This strategy is suitable for **batch processing** or tasks with **steady arrival rates**.

**Sample Code**:
```java
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;

public class FixedThreadPoolExample {
    // Define a pool with a fixed number of threads for predictable task load
    private static final int THREAD_POOL_SIZE = 5;
    private static final ExecutorService executorService = Executors.newFixedThreadPool(THREAD_POOL_SIZE);

    public static void main(String[] args) {
        // Simulate task submission to the fixed thread pool
        for (int i = 0; i < 10; i++) {
            final int taskId = i;
            executorService.submit(() -> processTask(taskId));
        }

        // Shutdown the executor service to release resources after processing
        executorService.shutdown();
    }

    private static void processTask(int taskId) {
        System.out.println("Processing Task ID: " + taskId + " with thread " + Thread.currentThread().getName());
        try {
            // Simulating task processing duration
            Thread.sleep(1000);
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        }
    }
}
```

**Output**:
```
Processing Task ID: 0 with thread pool-1-thread-1
Processing Task ID: 1 with thread pool-1-thread-2
Processing Task ID: 2 with thread pool-1-thread-3
Processing Task ID: 3 with thread pool-1-thread-4
Processing Task ID: 4 with thread pool-1-thread-5
Processing Task ID: 5 with thread pool-1-thread-1
Processing Task ID: 6 with thread pool-1-thread-2
Processing Task ID: 7 with thread pool-1-thread-3
Processing Task ID: 8 with thread pool-1-thread-4
Processing Task ID: 9 with thread pool-1-thread-5
```

**Explanation**:
1. **Thread Pool Initialization**:
   - A **fixed number of threads** (5) handle the tasks sequentially.
   - The thread pool is initialized with 5 threads, named `pool-1-thread-1` to `pool-1-thread-5`.

2. **Task Submission**:
   - The `for` loop submits 10 tasks to the thread pool.
   - Each task is assigned a unique `taskId` from 0 to 9.

3. **Task Execution**:
   - The tasks are executed by the available threads in the thread pool.
   - Since there are 5 threads, the first 5 tasks (0 to 4) are executed concurrently by the 5 threads.
   - After the first 5 tasks are completed, the next 5 tasks (5 to 9) are executed by the same threads, but the order of execution may vary.

4. **Thread Reuse**:
   - The executor will **reuse threads**, limiting thread creation overhead.
   - The threads in the thread pool are reused to execute subsequent tasks. For example, `pool-1-thread-1` might execute task 0 and then task 5.

5. The **`shutdown` method** releases resources once all tasks complete.

**Advantages:** Predictable memory usage and better **performance control** for tasks with a steady rate.

---

<a name="cachedthreadpool---unpredictable-short-lived-tasks"></a>
### 3. CachedThreadPool - Unpredictable, Short-Lived Tasks

A `CachedThreadPool` dynamically adjusts the pool size depending on task demand. This pool is ideal for handling **bursty** or **sporadic loads**. Threads are created as needed and, if idle, removed after a timeout period, allowing the system to scale back.

**Sample Code**:
```java
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;

public class CachedThreadPoolExample {
    // Using CachedThreadPool for sporadic and short-lived tasks
    private static final ExecutorService executorService = Executors.newCachedThreadPool();

    public static void main(String[] args) {
        // Simulate submitting bursty tasks
        for (int i = 0; i < 10; i++) {
            final int taskId = i;
            executorService.submit(() -> handleBurstTask(taskId));
        }

        // Shutdown executor once all tasks are completed
        executorService.shutdown();
    }

    private static void handleBurstTask(int taskId) {
        System.out.println("Handling Burst Task ID: " + taskId + " with thread " + Thread.currentThread().getName());
        try {
            // Simulating brief processing
            Thread.sleep(500);
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        }
    }
}
```

**Output**:
```
Handling Burst Task ID: 0 with thread pool-1-thread-1
Handling Burst Task ID: 1 with thread pool-1-thread-2
Handling Burst Task ID: 2 with thread pool-1-thread-3
Handling Burst Task ID: 3 with thread pool-1-thread-4
Handling Burst Task ID: 4 with thread pool-1-thread-5
Handling Burst Task ID: 5 with thread pool-1-thread-6
Handling Burst Task ID: 6 with thread pool-1-thread-7
Handling Burst Task ID: 7 with thread pool-1-thread-8
Handling Burst Task ID: 8 with thread pool-1-thread-9
Handling Burst Task ID: 9 with thread pool-1-thread-10
```

**Explanation:**
1. **Thread Pool Initialization**:
   - The `CachedThreadPool` **adds threads on demand** up to `Integer.MAX_VALUE`, suitable for short-lived tasks.
   - The `CachedThreadPool` dynamically creates new threads as needed and reuses previously constructed threads when they are available.

2. **Task Submission**:
   - The `for` loop submits 10 tasks to the thread pool.
   - Each task is assigned a unique `taskId` from 0 to 9.

3. **Task Execution**:
   - The tasks are executed by the available threads in the thread pool.
   - Since the `CachedThreadPool` creates new threads as needed, it will create up to 10 threads to handle the 10 tasks.

4. **Thread Creation and Reuse**:
   - Threads that are **idle for 60 seconds** are removed, preventing memory buildup.
   - Ideal for systems that handle **irregular spikes** without knowing task demand in advance.
   - The `CachedThreadPool` creates new threads for each task initially because no threads are available.
   - The threads are named `pool-1-thread-1` to `pool-1-thread-10`.

**Advantages:** Scalability and flexibility for **dynamic task arrival rates**.

---

<a name="4-dynamic-scaling-with-flexible-configuration"></a>
### 4. Dynamic Scaling with Flexible Configuration

In a production environment, scaling thread pools dynamically may involve using the `ThreadPoolExecutor` with customized parameters, such as adjustable core pool size and task queue types. This allows for **on-the-fly adjustments** based on real-time performance metrics.

**Sample Code**:
```java
import java.util.concurrent.*;

public class DynamicThreadPoolScaling {
    public static void main(String[] args) {
        // Custom ThreadPoolExecutor with flexible parameters for dynamic scaling
        ThreadPoolExecutor executor = new ThreadPoolExecutor(
            2, // corePoolSize
            10, // maximumPoolSize
            60L, // keepAliveTime
            TimeUnit.SECONDS,
            new LinkedBlockingQueue<>(5), // workQueue
            Executors.defaultThreadFactory(),
            new ThreadPoolExecutor.CallerRunsPolicy() // Rejection policy
        );

        for (int i = 0; i < 20; i++) {
            final int taskId = i;
            executor.submit(() -> performDynamicTask(taskId));
        }

        // Shut down the executor after tasks completion
        executor.shutdown();
    }

    private static void performDynamicTask(int taskId) {
        System.out.println("Performing Dynamic Task ID: " + taskId + " with thread " + Thread.currentThread().getName());
        try {
            Thread.sleep(1000);
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        }
    }
}
```

**Explanation of Parameters:**
- `corePoolSize`: Defines the **minimum** number of threads.
- `maximumPoolSize`: Sets a **scalable limit** for threads.
- `LinkedBlockingQueue`: Limits task queue to prevent memory overflows.
- `CallerRunsPolicy`: Ensures tasks run in the caller's thread if limits are reached, providing **backpressure control**.

**Output**:
```
Performing Dynamic Task ID: 0 with thread pool-1-thread-1
Performing Dynamic Task ID: 1 with thread pool-1-thread-2
Performing Dynamic Task ID: 2 with thread pool-1-thread-1
Performing Dynamic Task ID: 3 with thread pool-1-thread-2
Performing Dynamic Task ID: 4 with thread pool-1-thread-1
Performing Dynamic Task ID: 5 with thread pool-1-thread-2
Performing Dynamic Task ID: 6 with thread pool-1-thread-1
Performing Dynamic Task ID: 7 with thread pool-1-thread-2
Performing Dynamic Task ID: 8 with thread pool-1-thread-1
Performing Dynamic Task ID: 9 with thread pool-1-thread-2
Performing Dynamic Task ID: 10 with thread pool-1-thread-3
Performing Dynamic Task ID: 11 with thread pool-1-thread-4
Performing Dynamic Task ID: 12 with thread pool-1-thread-5
Performing Dynamic Task ID: 13 with thread pool-1-thread-6
Performing Dynamic Task ID: 14 with thread pool-1-thread-7
Performing Dynamic Task ID: 15 with thread pool-1-thread-8
Performing Dynamic Task ID: 16 with thread pool-1-thread-9
Performing Dynamic Task ID: 17 with thread pool-1-thread-10
Performing Dynamic Task ID: 18 with thread main
Performing Dynamic Task ID: 19 with thread main
```

**Explaination of Output**:
1. **ThreadPoolExecutor Initialization**:
   - The `ThreadPoolExecutor` is initialized with:
     - `corePoolSize`: 2
     - `maximumPoolSize`: 10
     - `keepAliveTime`: 60 seconds
     - `workQueue`: `LinkedBlockingQueue` with a capacity of 5
     - `threadFactory`: Default thread factory
     - `rejectionPolicy`: `CallerRunsPolicy`

2. **Task Submission**:
   - The `for` loop submits 20 tasks to the thread pool.
   - Each task is assigned a unique `taskId` from 0 to 19.

3. **Task Execution**:
   - Initially, the core threads (`pool-1-thread-1` and `pool-1-thread-2`) handle the first 2 tasks.
   - The next 5 tasks are queued in the `LinkedBlockingQueue`.
   - As the core threads finish their tasks, they pick up tasks from the queue.
   - When the queue is full (5 tasks), new threads are created up to the `maximumPoolSize` (10 threads) to handle the remaining tasks.
   - When all 10 threads are busy and the queue is full, the `CallerRunsPolicy` kicks in, and the remaining tasks are executed by the main thread (`main`).

### Detailed Steps

1. **Task 0 to 1 Execution**:
   - Task 0 is executed by `pool-1-thread-1`.
   - Task 1 is executed by `pool-1-thread-2`.

2. **Task 2 to 6 Queued**:
   - Tasks 2 to 6 are queued in the `LinkedBlockingQueue`.

3. **Task 2 to 6 Execution**:
   - Tasks 2 to 6 are executed by `pool-1-thread-1` and `pool-1-thread-2` as they finish their initial tasks.

4. **Task 7 to 11 Queued**:
   - Tasks 7 to 11 are queued in the `LinkedBlockingQueue`.

5. **Task 7 to 11 Execution**:
   - Tasks 7 to 11 are executed by `pool-1-thread-1` and `pool-1-thread-2` as they finish their previous tasks.

6. **Task 12 to 16 Execution**:
   - New threads (`pool-1-thread-3` to `pool-1-thread-7`) are created to handle tasks 12 to 16.

7. **Task 17 to 19 Execution**:
   - New threads (`pool-1-thread-8` to `pool-1-thread-10`) are created to handle tasks 17 to 19.

8. **Task 18 and 19 Execution by Main Thread**:
   - Since all threads are busy and the queue is full, the `CallerRunsPolicy` causes the main thread to execute tasks 18 and 19.

---

<a name="5-thread-pool-strategy-decision"></a>
### 5. Thread Pool Strategy Decision

```plaintext
                +-------------------------+
                |  Task Load Predictable? |
                +-----------+-------------+
                            |
                            |
                   Yes      |       No
                 +----------+------------+
                 |                       |
         +-------v----------+     +------v------------+
         |  FixedThreadPool |     |  CachedThreadPool |
         +-------+----------+     +-----+-------------+
                 |                      |
                 |                      |
        Use if tasks are steady    Use if tasks are bursty or
        and concurrent             unpredictable
```

Using `Executors` to create thread pools (e.g., via `Executors.newFixedThreadPool()` or `Executors.newCachedThreadPool()`) is convenient but introduces hidden risks and drawbacks in certain production scenarios. Let's look at the specific drawbacks and explore best practices for mitigating these issues.

---

<a name="6-drawbacks-of-using-executors-to-create-thread-pools"></a>
### 6. Drawbacks of Using `Executors` to Create Thread Pools

1. **Unbounded Queue with `newFixedThreadPool`**:
   - The `newFixedThreadPool(int nThreads)` method uses a **`LinkedBlockingQueue`** without a maximum size limit for its task queue. This **unbounded queue** can lead to **memory exhaustion** if tasks are submitted faster than they can be processed.
   - When a large number of tasks are added, the queue will keep growing, which can lead to **OutOfMemoryError** and system instability.

2. **Unbounded Thread Creation in `newCachedThreadPool`**:
   - The `newCachedThreadPool()` uses a **`SynchronousQueue`** that directly hands off tasks to threads. If no threads are idle to handle new tasks, it **creates new threads** up to `Integer.MAX_VALUE`.
   - With a high load, this pool may create excessive threads, leading to **CPU saturation** and potential **memory exhaustion**, especially in long-running applications.

3. **Lack of Control Over Rejection Policy**:
   - The methods in `Executors` do not allow explicit configuration of **rejection policies** for when tasks exceed pool capacity. Default policies can fail to handle high-load scenarios gracefully, often leading to performance degradation and unhandled task rejections.

4. **Harder to Manage Idle Threads**:
   - For pools like `newCachedThreadPool`, idle threads remain for a default **60 seconds** before being terminated. While this may be appropriate for some workloads, it can lead to inefficiencies in scenarios where **shorter idle times** would conserve resources.

---

<a name="7-best-practices-solution-using-threadpoolexecutor-directly"></a>
### 7. Best Practices Solution: Using `ThreadPoolExecutor` Directly

To avoid these pitfalls, it's best to directly instantiate a `ThreadPoolExecutor` where you can **explicitly configure** the task queue size, rejection policies, and timeout values. Here’s how to address each issue with recommended configurations:

A. **Configure Bounded Queues to Prevent Memory Leaks**:
   - Use a bounded task queue, like `ArrayBlockingQueue` or a sized `LinkedBlockingQueue`, to **control the number of waiting tasks** and prevent out-of-memory errors.
   - This limits the number of tasks waiting in the queue, ensuring system stability by capping memory use.

   ```java
   new ThreadPoolExecutor(
       corePoolSize,
       maxPoolSize,
       keepAliveTime,
       TimeUnit.SECONDS,
       new ArrayBlockingQueue<>(queueSize)
   );
   ```

B. **Limit Thread Growth to Prevent Resource Exhaustion**:
- Use a capped `maximumPoolSize` when tasks can surge unexpectedly. Setting a limit, e.g., based on the number of available CPU cores, helps prevent CPU saturation.

  When tasks can surge unexpectedly, it's crucial to set a capped `maximumPoolSize` to prevent the thread pool from creating an excessive number of threads. An unbounded `maximumPoolSize` can lead to CPU saturation, excessive memory usage, and degraded performance due to excessive context switching.

  **CPU Saturation**
  CPU saturation occurs when the CPU is overwhelmed with too many threads, leading to:
  - **High CPU Utilization**: The CPU spends most of its time executing threads, leaving little time for other system processes.
  - **Context Switching Overhead**: Frequent context switching between threads consumes CPU cycles, reducing overall efficiency.
  - **Memory Pressure**: Each thread consumes memory, and a large number of threads can lead to memory exhaustion.

  To prevent CPU saturation, it's recommended to set a `maximumPoolSize` based on the number of available CPU cores. This ensures that the thread pool can handle a reasonable number of concurrent tasks without overwhelming the CPU.

  A common approach is to set the `maximumPoolSize` to a value slightly higher than the number of CPU cores to account for both CPU-bound and I/O-bound tasks. A simple formula is `maximumPoolSize = N * U`

  Where:
  - **N**: Number of available CPU cores.
  - **U**: Utilization factor (typically between 1.0 and 2.0).
    - **U = 1.0**: For CPU-bound tasks where each thread is expected to fully utilize a CPU core.
    - **U = 1.5 to 2.0**: For mixed CPU-bound and I/O-bound tasks, where some threads may be waiting on I/O operations.


  **Example Calculation**
  Suppose you have a system with 8 CPU cores and you expect a mix of CPU-bound and I/O-bound tasks. You can set the `maximumPoolSize` as follows `maximumPoolSize = 8 * 1.5 = 12`

- For highly concurrent applications, you can use dynamic sizing strategies with **monitoring metrics** (like CPU and memory usage) to inform adjustments to pool parameters.

C. **Explicitly Set Rejection Policies**:
   - When tasks exceed the pool’s capacity, control the pool’s behavior with rejection policies. `ThreadPoolExecutor` provides options like `CallerRunsPolicy`, `DiscardPolicy`, and `DiscardOldestPolicy`:
      - **CallerRunsPolicy**: Executes tasks in the caller thread, which helps slow down task submissions under heavy load.
      - **DiscardOldestPolicy**: Removes the oldest queued task, making room for newer, high-priority tasks.
   - This choice depends on your application’s tolerance for unprocessed tasks.

   ```java
   new ThreadPoolExecutor(
       corePoolSize,
       maxPoolSize,
       keepAliveTime,
       TimeUnit.SECONDS,
       new LinkedBlockingQueue<>(queueSize),
       new ThreadPoolExecutor.CallerRunsPolicy()
   );
   ```

   **Adjust Idle Timeout for Cached Threads**:
   - For pools similar to `CachedThreadPool`, customize the **idle timeout** to conserve resources by releasing threads sooner when they are not needed. `ThreadPoolExecutor` allows control over idle time with `setKeepAliveTime`.
   
   ```java
   ThreadPoolExecutor cachedPool = new ThreadPoolExecutor(
       0, Integer.MAX_VALUE,
       30L, TimeUnit.SECONDS, // Reduced keep-alive time
       new SynchronousQueue<>(),
       Executors.defaultThreadFactory()
   );
   ```

---

<a name="8-threadpoolexecutor-with-best-practice-configurations"></a>
### 8. `ThreadPoolExecutor` with Best Practice Configurations

Below is a best practice setup using `ThreadPoolExecutor` that incorporates bounded queues, a limited thread count, a suitable rejection policy, and a controlled idle timeout:

```java
import java.util.concurrent.*;

public class BestPracticeThreadPool {

    public static ExecutorService createThreadPool() {
        int corePoolSize = 4;
        int maxPoolSize = 10;
        long keepAliveTime = 30L;
        int queueCapacity = 50;

        return new ThreadPoolExecutor(
                corePoolSize,
                maxPoolSize,
                keepAliveTime,
                TimeUnit.SECONDS,
                new ArrayBlockingQueue<>(queueCapacity),
                Executors.defaultThreadFactory(),
                new ThreadPoolExecutor.CallerRunsPolicy() // Rejection policy to handle overloads
        );
    }

    public static void main(String[] args) {
        ExecutorService threadPool = createThreadPool();

        // Submitting example tasks
        for (int i = 0; i < 100; i++) {
            final int taskId = i;
            threadPool.submit(() -> processTask(taskId));
        }

        // Properly shutdown to free resources
        threadPool.shutdown();
    }

    private static void processTask(int taskId) {
        System.out.println("Processing Task ID: " + taskId + " with thread " + Thread.currentThread().getName());
        try {
            Thread.sleep(500); // Simulate work
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        }
    }
}
```

**Explanation of Configurations:**
- `ArrayBlockingQueue` limits queue size to 50, managing memory consumption.
- `CallerRunsPolicy` applies backpressure to slow down submissions.
- `corePoolSize` and `maxPoolSize` control CPU utilization.
- `keepAliveTime` of 30 seconds clears idle threads efficiently.

---

<a name="9-summary"></a>
### 9. Summary

Using `ThreadPoolExecutor` directly provides complete control over pool configurations:
- **Bounded queues** prevent memory overflows.
- **Limited threads** avoid CPU exhaustion.
- **Rejection policies** ensure controlled behavior under load.
- **Idle timeouts** conserve system resources. 

These adjustments create a resilient and scalable thread pool setup suitable for high-performance, resource-intensive applications.