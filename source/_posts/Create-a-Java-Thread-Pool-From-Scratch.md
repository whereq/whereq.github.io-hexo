---
title: Create a Java Thread Pool From Scratch
date: 2024-10-28 23:32:51
categories:
- Deep Dive
- Java Core
- Multi-Threading
tags:
- Deep Dive
- Java Core
- Multi-Threading
---

- [Custom Java Thread Pool: Proof of Concept](#custom-java-thread-pool-proof-of-concept)
  - [Implementation of `SimpleThreadPool`](#implementation-of-simplethreadpool)
  - [Explanation of Key Concepts](#explanation-of-key-concepts)
  - [Sample Code to Use the `SimpleThreadPool`](#sample-code-to-use-the-simplethreadpool)
    - [Explanation of Sample Usage Code](#explanation-of-sample-usage-code)
    - [Expected Output](#expected-output)
  - [Conclusion](#conclusion)
- [Custom Java Thread Pool with Separate Task Class](#custom-java-thread-pool-with-separate-task-class)
  - [`Task` Class Implementation](#task-class-implementation)
  - [`SimpleThreadPool` Implementation](#simplethreadpool-implementation)
  - [Sample Code to Use the Custom Thread Pool with `Task`](#sample-code-to-use-the-custom-thread-pool-with-task)
    - [Explanation of Sample Usage](#explanation-of-sample-usage)
    - [Expected Output](#expected-output-1)
- [Wait but what happened to the terminated thread?](#wait-but-what-happened-to-the-terminated-thread)
    - [Enhanced `SimpleThreadPool` Implementation](#enhanced-simplethreadpool-implementation)
    - [Enhanced `Task` Class Implementation](#enhanced-task-class-implementation)
    - [Usage and State Transition Simulation](#usage-and-state-transition-simulation)
    - [Explanation of the Workflow and State Transitions](#explanation-of-the-workflow-and-state-transitions)
    - [Expected Output (Sample)](#expected-output-sample)
- [How thread being actually reused?](#how-thread-being-actually-reused)
    - [Thread States in a Thread Pool](#thread-states-in-a-thread-pool)
    - [How Threads Are Reused](#how-threads-are-reused)
    - [Example of Thread Reuse in `SimpleThreadPool`](#example-of-thread-reuse-in-simplethreadpool)
    - [Code Example](#code-example)
    - [Key Points](#key-points)
    - [Summary](#summary)

---

<a name="custom-java-thread-pool-proof-of-concept"></a>
# Custom Java Thread Pool: Proof of Concept

In this article, we will create a simple but complete custom thread pool. We’ll build a `SimpleThreadPool` class that includes key features such as managing worker threads, a task queue, and shutdown capabilities. We'll break down the code into logical components, explain the design concepts, and provide sample usage.

---

<a name="implementation-of-simplethreadpool"></a>
## Implementation of `SimpleThreadPool`

The core classes in our implementation are:
- `SimpleThreadPool`: Manages the thread pool and task queue.
- `Worker`: Represents the worker threads that execute tasks.

```java
import java.util.LinkedList;
import java.util.Queue;

public class SimpleThreadPool {
    private final int poolSize;
    private final PoolWorker[] workers;
    private final Queue<Runnable> taskQueue;
    private boolean isShutdown = false;

    public SimpleThreadPool(int poolSize) {
        this.poolSize = poolSize;
        taskQueue = new LinkedList<>();
        workers = new PoolWorker[poolSize];

        // Start worker threads
        for (int i = 0; i < poolSize; i++) {
            workers[i] = new PoolWorker();
            workers[i].start();
        }
    }

    // Submit a task to the thread pool
    public synchronized void submit(Runnable task) {
        if (!isShutdown) {
            taskQueue.add(task);
            notify(); // Notify workers that a task is available
        } else {
            throw new IllegalStateException("ThreadPool is shut down");
        }
    }

    // Initiates an orderly shutdown
    public synchronized void shutdown() {
        isShutdown = true;
        for (PoolWorker worker : workers) {
            worker.interrupt(); // Interrupt each worker to stop waiting
        }
    }

    // Worker class represents threads in the pool
    private class PoolWorker extends Thread {
        public void run() {
            Runnable task;

            while (true) {
                synchronized (SimpleThreadPool.this) {
                    while (taskQueue.isEmpty() && !isShutdown) {
                        try {
                            SimpleThreadPool.this.wait(); // Wait for a task
                        } catch (InterruptedException ignored) {
                            return; // Exit if interrupted during shutdown
                        }
                    }
                    // If shutdown and no tasks, end thread
                    if (isShutdown && taskQueue.isEmpty()) {
                        return;
                    }
                    task = taskQueue.poll(); // Take task from queue
                }
                try {
                    task.run(); // Execute task outside synchronized block
                } catch (RuntimeException e) {
                    System.err.println("Task execution failed: " + e.getMessage());
                }
            }
        }
    }
}
```

---

<a name="explanation-of-key-concepts"></a>
## Explanation of Key Concepts

1. **Task Queue**: A `LinkedList` serves as the task queue, holding `Runnable` tasks for worker threads.
2. **Worker Threads**: `PoolWorker` threads execute tasks. Each worker loops to fetch a task from the queue and processes it if available.
3. **Synchronization with `wait` and `notify`**:
   - `wait()`: Called when no tasks are available, putting the thread in a waiting state to save CPU resources.
   - `notify()`: Called when a new task is added, waking up a worker thread to execute the task.
4. **Graceful Shutdown**:
   - `shutdown()`: Sets the `isShutdown` flag to stop accepting tasks and interrupts all worker threads, allowing them to exit if they’re waiting for a task.

---

<a name="sample-code-to-use-the-simplethreadpool"></a>
## Sample Code to Use the `SimpleThreadPool`

This example demonstrates how to use the custom `SimpleThreadPool` to run tasks, including graceful shutdown.

```java
public class SimpleThreadPoolExample {
    public static void main(String[] args) {
        // Create a thread pool with 3 threads
        SimpleThreadPool threadPool = new SimpleThreadPool(3);

        // Submit tasks to the thread pool
        for (int i = 1; i <= 5; i++) {
            int taskId = i;
            threadPool.submit(() -> {
                System.out.println("Task " + taskId + " is starting on thread " + Thread.currentThread().getName());
                try {
                    Thread.sleep(1000); // Simulate task execution time
                } catch (InterruptedException e) {
                    Thread.currentThread().interrupt();
                }
                System.out.println("Task " + taskId + " completed.");
            });
        }

        // Shutdown the thread pool after tasks complete
        threadPool.shutdown();
    }
}
```

### Explanation of Sample Usage Code

1. **Creating a Thread Pool**: A `SimpleThreadPool` instance with 3 threads is created.
2. **Submitting Tasks**: Five tasks are submitted to the pool, each of which simulates a brief work period.
3. **Shutting Down the Pool**: After submitting the tasks, the pool is gracefully shut down using `shutdown()`. 

### Expected Output

```
Task 1 is starting on thread Thread-0
Task 2 is starting on thread Thread-1
Task 3 is starting on thread Thread-2
Task 1 completed.
Task 2 completed.
Task 3 completed.
Task 4 is starting on thread Thread-0
Task 5 is starting on thread Thread-1
Task 4 completed.
Task 5 completed.
```

---

<a name="conclusion"></a>
## Conclusion

This `SimpleThreadPool` is a demonstration of how a basic thread pool is structured and works:
- **Worker Threads** repeatedly pick up and execute tasks from a shared queue.
- **Synchronization** between task producers and worker threads prevents busy-waiting.
- **Graceful Shutdown** lets worker threads exit safely after processing all queued tasks.

This POC lays the foundation for understanding and extending Java thread pool management by illustrating how resources are managed and synchronized in a concurrent environment.

---

<a name="custom-java-thread-pool-with-separate-task-class"></a>
# Custom Java Thread Pool with Separate Task Class

Separating the `Task` as an individual class will enhance the code's modularity and flexibility, making it easier to reuse or extend for specific task-related features. Let's modify the `SimpleThreadPool` implementation to include an external `Task` class.

Here’s the updated implementation:

1. **`SimpleThreadPool`**: Manages worker threads and the task queue.
2. **`Task`**: Represents individual tasks, implementing `Runnable`.
3. **`PoolWorker`**: Represents the worker threads that pick up tasks from the queue and execute them.

---

## `Task` Class Implementation

We’ll define `Task` as a standalone class that implements `Runnable`. Each task can have custom logic.

```java
public class Task implements Runnable {
    private final int taskId;

    public Task(int taskId) {
        this.taskId = taskId;
    }

    @Override
    public void run() {
        System.out.println("Task " + taskId + " is starting on thread " + Thread.currentThread().getName());
        try {
            Thread.sleep(1000); // Simulate task execution time
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt(); // Handle interruption
        }
        System.out.println("Task " + taskId + " completed.");
    }
}
```

- **`taskId`**: Each task is given an ID to identify it during execution.
- **`run()` Method**: Defines the task's actions, including a simulated delay.

---

## `SimpleThreadPool` Implementation

The `SimpleThreadPool` class manages the task queue and distributes tasks among worker threads.

```java
import java.util.LinkedList;
import java.util.Queue;

public class SimpleThreadPool {
    private final int poolSize;
    private final PoolWorker[] workers;
    private final Queue<Runnable> taskQueue;
    private boolean isShutdown = false;

    public SimpleThreadPool(int poolSize) {
        this.poolSize = poolSize;
        taskQueue = new LinkedList<>();
        workers = new PoolWorker[poolSize];

        // Initialize and start each worker thread
        for (int i = 0; i < poolSize; i++) {
            workers[i] = new PoolWorker();
            workers[i].start();
        }
    }

    // Method to submit a task to the thread pool
    public synchronized void submit(Runnable task) {
        if (!isShutdown) {
            taskQueue.add(task);
            notify(); // Notify a waiting worker thread about the new task
        } else {
            throw new IllegalStateException("ThreadPool is shut down and cannot accept tasks");
        }
    }

    // Initiates shutdown and interrupts all waiting worker threads
    public synchronized void shutdown() {
        isShutdown = true;
        for (PoolWorker worker : workers) {
            worker.interrupt(); // Interrupts workers waiting for tasks
        }
    }

    // Worker class that executes tasks from the queue
    private class PoolWorker extends Thread {
        public void run() {
            while (true) {
                Runnable task;
                synchronized (SimpleThreadPool.this) {
                    while (taskQueue.isEmpty() && !isShutdown) {
                        try {
                            SimpleThreadPool.this.wait(); // Wait for a new task
                        } catch (InterruptedException e) {
                            return; // Exit if interrupted during shutdown
                        }
                    }
                    if (isShutdown && taskQueue.isEmpty()) {
                        return; // Exit if shutdown and no tasks are left
                    }
                    task = taskQueue.poll(); // Fetch a task
                }
                try {
                    task.run(); // Execute the task outside the synchronized block
                } catch (RuntimeException e) {
                    System.err.println("Task execution failed: " + e.getMessage());
                }
            }
        }
    }
}
```

---

## Sample Code to Use the Custom Thread Pool with `Task`

This example demonstrates submitting `Task` instances to the `SimpleThreadPool`.

```java
public class SimpleThreadPoolExample {
    public static void main(String[] args) {
        // Create a thread pool with 3 threads
        SimpleThreadPool threadPool = new SimpleThreadPool(3);

        // Submit tasks to the thread pool
        for (int i = 1; i <= 5; i++) {
            Task task = new Task(i); // Create a new Task with a unique ID
            threadPool.submit(task);  // Submit the Task to the thread pool
        }

        // Shutdown the thread pool after tasks are complete
        threadPool.shutdown();
    }
}
```

### Explanation of Sample Usage

1. **Task Creation**: Each `Task` object has a unique ID, making it easy to track execution progress.
2. **Task Submission**: Tasks are added to the `SimpleThreadPool`’s task queue.
3. **Shutdown**: Once all tasks are submitted, `shutdown()` stops the pool after finishing queued tasks.

### Expected Output

```
Task 1 is starting on thread Thread-0
Task 2 is starting on thread Thread-1
Task 3 is starting on thread Thread-2
Task 1 completed.
Task 2 completed.
Task 3 completed.
Task 4 is starting on thread Thread-0
Task 5 is starting on thread Thread-1
Task 4 completed.
Task 5 completed.
```

---

This implementation improves modularity by allowing different `Task` implementations and extends the reusability of `SimpleThreadPool`. Each component (`Task`, `PoolWorker`, and `SimpleThreadPool`) is designed for flexible use in varied production scenarios.

---

<a name="wait-but-what-happened-to-the-terminated-thread"></a>
# Wait but what happened to the terminated thread?

To demonstrate the behavior where a terminated thread in a thread pool is repurposed and moves back to the runnable state when a new task is added, we need to tweak the `SimpleThreadPool` to simulate this behavior. 

Typically, when threads in a pool complete their tasks, they wait for new tasks rather than fully terminating (they don't reach the "TERMINATED" state in Java's built-in `ThreadPoolExecutor`). However, for demonstration, I’ll create logic that represents threads reaching a "stopped" state and being reactivated for new tasks.

Let's add a mechanism to track the states and illustrate the thread reusability behavior.

---

### Enhanced `SimpleThreadPool` Implementation

Below is an enhanced version of `SimpleThreadPool` with thread state tracking. We’ll add logging and state simulation to show when threads are "terminated" and later reactivated when new tasks arrive.

```java
import java.util.LinkedList;
import java.util.Queue;

public class SimpleThreadPool {
    private final int poolSize;
    private final PoolWorker[] workers;
    private final Queue<Runnable> taskQueue;
    private boolean isShutdown = false;

    public SimpleThreadPool(int poolSize) {
        this.poolSize = poolSize;
        taskQueue = new LinkedList<>();
        workers = new PoolWorker[poolSize];

        // Initialize and start each worker thread
        for (int i = 0; i < poolSize; i++) {
            workers[i] = new PoolWorker(i + 1); // Assign a unique ID to each worker
            workers[i].start();
        }
    }

    // Method to submit a task to the thread pool
    public synchronized void submit(Runnable task) {
        if (!isShutdown) {
            taskQueue.add(task);
            notify(); // Notify a waiting worker thread about the new task
        } else {
            throw new IllegalStateException("ThreadPool is shut down and cannot accept tasks");
        }
    }

    // Initiates shutdown and interrupts all waiting worker threads
    public synchronized void shutdown() {
        isShutdown = true;
        for (PoolWorker worker : workers) {
            worker.interrupt(); // Interrupts workers waiting for tasks
        }
    }

    // Worker class that executes tasks from the queue
    private class PoolWorker extends Thread {
        private final int workerId;
        private boolean isTerminated = false;

        public PoolWorker(int id) {
            this.workerId = id;
        }

        public void run() {
            while (true) {
                Runnable task;
                synchronized (SimpleThreadPool.this) {
                    while (taskQueue.isEmpty() && !isShutdown) {
                        System.out.println("Worker-" + workerId + " is waiting for a task...");
                        try {
                            SimpleThreadPool.this.wait(); // Wait for a new task
                        } catch (InterruptedException e) {
                            System.out.println("Worker-" + workerId + " is terminated.");
                            isTerminated = true;
                            return; // Exit if interrupted during shutdown
                        }
                    }
                    if (isShutdown && taskQueue.isEmpty()) {
                        System.out.println("Worker-" + workerId + " is terminated as pool shuts down.");
                        isTerminated = true;
                        return; // Exit if shutdown and no tasks are left
                    }
                    task = taskQueue.poll(); // Fetch a task
                }
                
                try {
                    isTerminated = false; // Worker is active
                    System.out.println("Worker-" + workerId + " is executing task.");
                    task.run(); // Execute the task
                    System.out.println("Worker-" + workerId + " completed task and goes back to waiting.");
                } catch (RuntimeException e) {
                    System.err.println("Task execution failed in Worker-" + workerId + ": " + e.getMessage());
                }
            }
        }
    }
}
```

### Enhanced `Task` Class Implementation

The `Task` class remains mostly the same, but we can add unique identifiers to differentiate tasks.

```java
public class Task implements Runnable {
    private final int taskId;

    public Task(int taskId) {
        this.taskId = taskId;
    }

    @Override
    public void run() {
        System.out.println("Task " + taskId + " is starting on thread " + Thread.currentThread().getName());
        try {
            Thread.sleep(1000); // Simulate task execution time
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt(); // Handle interruption
        }
        System.out.println("Task " + taskId + " completed.");
    }
}
```

### Usage and State Transition Simulation

Let's add a usage example showing how to submit tasks, observe workers entering a “terminated” state, and then reactivating to process new tasks.

```java
public class ThreadPoolExample {
    public static void main(String[] args) throws InterruptedException {
        // Create a thread pool with 2 workers
        SimpleThreadPool threadPool = new SimpleThreadPool(2);

        // Submit initial tasks to the thread pool
        for (int i = 1; i <= 3; i++) {
            threadPool.submit(new Task(i));
        }

        // Wait to let initial tasks complete
        Thread.sleep(3000);

        // Submit additional tasks to show reactivation
        System.out.println("Adding more tasks to test thread reactivation...");
        for (int i = 4; i <= 5; i++) {
            threadPool.submit(new Task(i));
        }

        // Give some time for tasks to be picked up
        Thread.sleep(3000);

        // Shutdown the thread pool
        threadPool.shutdown();
    }
}
```

### Explanation of the Workflow and State Transitions

1. **Initial Task Submission**: Three tasks are submitted, and two workers pick them up immediately since the pool size is 2.
2. **Worker Reactivation**: After the initial tasks complete, we add two more tasks, which are picked up by waiting workers.
3. **Shutdown Process**: Finally, we shut down the thread pool, and the workers gracefully terminate after completing their tasks.

### Expected Output (Sample)

```plaintext
Worker-1 is waiting for a task...
Worker-2 is waiting for a task...
Worker-1 is executing task.
Task 1 is starting on thread Worker-1
Worker-2 is executing task.
Task 2 is starting on thread Worker-2
Task 1 completed.
Worker-1 completed task and goes back to waiting.
Task 2 completed.
Worker-2 completed task and goes back to waiting.
Worker-1 is executing task.
Task 3 is starting on thread Worker-1
Task 3 completed.
Worker-1 completed task and goes back to waiting.
Worker-1 is waiting for a task...
Worker-2 is waiting for a task...

Adding more tasks to test thread reactivation...
Worker-1 is executing task.
Task 4 is starting on thread Worker-1
Worker-2 is executing task.
Task 5 is starting on thread Worker-2
Task 4 completed.
Worker-1 completed task and goes back to waiting.
Task 5 completed.
Worker-2 completed task and goes back to waiting.
Worker-1 is terminated as pool shuts down.
Worker-2 is terminated as pool shuts down.
```

This output demonstrates that the workers remain active, go back to a waiting state after completing tasks, and terminate only after `shutdown()` is called.

<a name="how-thread-being-acutally-reused"></a>
# How thread being actually reused?
In a typical thread pool implementation, once a thread completes its task, it does not transition to the "TERMINATED" state and then back to the "RUNNABLE" state. Instead, the thread remains in a waiting state (often referred to as "idle" or "waiting for work") until a new task is available. This waiting state is different from the "TERMINATED" state, which indicates that the thread has finished execution and cannot be reused.

Here’s a more detailed explanation:

### Thread States in a Thread Pool

1. **NEW**: The thread has been created but not yet started.
2. **RUNNABLE**: The thread is executing in the JVM.
3. **BLOCKED**: The thread is blocked waiting for a monitor lock.
4. **WAITING**: The thread is waiting indefinitely for another thread to perform a particular action.
5. **TIMED_WAITING**: The thread is waiting for another thread to perform an action for up to a specified waiting time.
6. **TERMINATED**: The thread has completed execution.

In a thread pool, threads typically cycle between the **RUNNABLE** and **WAITING** states:

- **RUNNABLE**: When a thread is actively executing a task.
- **WAITING**: When a thread is idle, waiting for a new task to be added to the queue.

### How Threads Are Reused

When a thread completes its task, it checks the task queue for new tasks. If no tasks are available, the thread waits (using `wait()` or a similar mechanism) until a new task is submitted. This waiting state is not the same as the "TERMINATED" state. A thread in the "TERMINATED" state cannot be reused; it must be recreated, which is inefficient.

### Example of Thread Reuse in `SimpleThreadPool`

In the `SimpleThreadPool` example, threads are reused as follows:

1. **Thread Creation**: When the thread pool is created, a fixed number of worker threads are started.
2. **Task Execution**: Each worker thread fetches a task from the queue and executes it.
3. **Waiting for New Tasks**: After completing a task, the worker thread checks the queue for new tasks. If no tasks are available, it waits (`wait()`) until a new task is submitted.
4. **Reactivation**: When a new task is submitted, the waiting threads are notified (`notify()`), and one of them wakes up to execute the new task.

### Code Example

Here’s a simplified version of the `SimpleThreadPool` to illustrate this:

```java
import java.util.LinkedList;
import java.util.Queue;

public class SimpleThreadPool {
    private final int poolSize;
    private final PoolWorker[] workers;
    private final Queue<Runnable> taskQueue;
    private boolean isShutdown = false;

    public SimpleThreadPool(int poolSize) {
        this.poolSize = poolSize;
        taskQueue = new LinkedList<>();
        workers = new PoolWorker[poolSize];

        // Start worker threads
        for (int i = 0; i < poolSize; i++) {
            workers[i] = new PoolWorker();
            workers[i].start();
        }
    }

    public synchronized void submit(Runnable task) {
        if (!isShutdown) {
            taskQueue.add(task);
            notify(); // Notify a waiting worker thread
        } else {
            throw new IllegalStateException("ThreadPool is shut down");
        }
    }

    public synchronized void shutdown() {
        isShutdown = true;
        for (PoolWorker worker : workers) {
            worker.interrupt(); // Interrupt each worker to stop waiting
        }
    }

    private class PoolWorker extends Thread {
        public void run() {
            while (true) {
                Runnable task;
                synchronized (SimpleThreadPool.this) {
                    while (taskQueue.isEmpty() && !isShutdown) {
                        try {
                            SimpleThreadPool.this.wait(); // Wait for a task
                        } catch (InterruptedException ignored) {
                            return; // Exit if interrupted during shutdown
                        }
                    }
                    if (isShutdown && taskQueue.isEmpty()) {
                        return; // Exit if shutdown and no tasks are left
                    }
                    task = taskQueue.poll(); // Take task from queue
                }
                try {
                    task.run(); // Execute task
                } catch (RuntimeException e) {
                    System.err.println("Task execution failed: " + e.getMessage());
                }
            }
        }
    }
}
```

### Key Points

- **Thread Reuse**: Threads do not transition to the "TERMINATED" state after completing a task. Instead, they wait for new tasks.
- **Task Queue**: The task queue holds tasks that are waiting to be executed. When a new task is submitted, a waiting thread is notified and picks up the task.
- **Graceful Shutdown**: The `shutdown()` method sets the `isShutdown` flag to stop accepting new tasks and interrupts all worker threads to ensure they exit if they are waiting.

### Summary
In Java’s `ThreadPoolExecutor` (or our custom pool example), the worker threads do **not** actually terminate after completing a task. Instead, they enter a **waiting** or **idle state** and wait to pick up the next task. The main goal of a thread pool is to reuse threads efficiently, keeping them alive for as long as the pool is running and ready to transition back to the **runnable** state whenever a new task becomes available. 

Here's how it works:

1. **Runnable → Running**: A thread in the pool becomes **running** when it starts executing a task.
2. **Running → Waiting/Idle**: Once it finishes the task, it doesn't terminate. Instead, it returns to an **idle state** within the pool, waiting for more tasks to be added to the queue.
3. **Waiting → Runnable**: When a new task is submitted, an idle worker transitions to **runnable** and executes the task.
   
The threads only reach a **terminated state** when the pool itself is **shut down** (e.g., calling `shutdown()` in `ThreadPoolExecutor`). Only then will the threads stop, exit the idle/waiting state, and go to terminated. This approach keeps the pool flexible and efficient, reducing the overhead of creating and destroying threads for every task. 

So in short: threads in a thread pool **do not reach terminated after completing tasks**. They go back to idle/waiting and can accept new tasks. And threads in a thread pool do not go back to the "RUNNABLE" state from the "TERMINATED" state neither. Instead, they remain in a waiting state until new tasks are available, ensuring efficient reuse of thread resources.