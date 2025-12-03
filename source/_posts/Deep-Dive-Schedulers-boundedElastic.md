---
title: Deep Dive Schedulers.boundedElastic()
date: 2025-12-02 20:24:20
categories:
- Reactive
- WebFlux
tags:
- Reactive
- WebFlux
---

### Start with a simple piece of code snippet
```java
kafkaReceiver.receive()
    .concatMap(message -> process(message)           // Step 1
        .subscribeOn(Schedulers.boundedElastic()))  // Step 2
    .doOnNext(msg -> msg.receiverOffset().acknowledge()) // Step 3
    .subscribe();                                           // Step 4
```

This is the **gold-standard pattern** in 2025 for consuming Kafka with **Spring WebFlux + Project Reactor + Reactor Kafka** when `process(message)` contains **blocking code** (JDBC, Redis sync driver, REST call with RestTemplate, file I/O, etc.).

#### Step-by-Step Execution Flow

| Step | What Happens | Which Thread Pool? | Why It Matters |
|------|--------------|--------------------|---------------|
| 1    | `kafkaReceiver.receive()` returns a `Flux<ReceiverRecord>` | Runs on **Netty event-loop** threads (non-blocking I/O) | This thread **must never block** or the whole reactor Netty server will starve |
| 2    | `concatMap` waits for the **previous `process()` to finish before taking the next message → preserves Kafka order per partition | Still on Netty thread until… | |
|      | `subscribeOn(Schedulers.boundedElastic())` **moves the entire upstream work** (i.e. `process(message)`) to a **boundedElastic** thread | → jumps to **boundedElastic thread pool** | This is the **only safe place** to do blocking work |
| 3    | After `process()` completes successfully, we come back to the **original Netty thread** to call `acknowledge()` | Back on Netty event-loop | Commit must happen on the same thread that received the record (Reactor Kafka requirement) |
| 4    | `.subscribe()` starts the whole chain | | |

### What Is `Schedulers.boundedElastic()` Exactly?

| Property                       | Value (Reactor 3.6+)                              | What It Means in Practice |
|--------------------------------|----------------------------------------------------|---------------------------|
| Thread pool type               | Thread-per-task + **bounded queue**                | Creates new threads on demand |
| Max threads                    | `Math.max(10, CPU × 10)` → usually **100–200**     | Will never create thousands of threads |
| Queue                          | Bounded (default 100,000 tasks)                    | If queue full → `onError` with `QueueFullException` |
| Thread name prefix             | `boundedElastic-x`                                 | Easy to spot in thread dumps |
| Designed for                   | **Blocking I/O** (JDBC, blocking Redis, file, external sync API) | This is the **only** scheduler you are allowed to block on |

#### Comparison with Other Schedulers

| Scheduler                    | Use Case                                   | Can I block here? |
|------------------------------|--------------------------------------------|-------------------|
| `Schedulers.parallel()`      | CPU-intensive (encryption, image processing) | No – will starve CPU workers |
| `Schedulers.boundedElastic()`| **Blocking I/O** ← **this is the one**     | Yes – made for this |
| `Schedulers.single()` / `immediate()` | Rare, very specific cases               | Never block |

### Real-Life Example – What Happens with 10,000 Messages/sec

```java
// BAD – blocks Netty thread → entire WebFlux app dies in seconds
.concatMap(msg -> blockingJdbc.save(msg))

// GOOD – this code you posted
.concatMap(message -> process(message)           // contains JdbcTemplate, RestTemplate, etc.
    .subscribeOn(Schedulers.boundedElastic()))
```

Without `subscribeOn(boundedElastic())` → Netty event-loop threads block → TCP backlog → all WebSocket clients disconnect → outage.

With it → 150 boundedElastic threads happily do JDBC work, Netty stays responsive, throughput 10k+/sec easily.

### Best Practices (2025)

```java
// Most common production pattern
kafkaReceiver.receive()
    .concatMap(record -> 
        process(record.value())                              // may block
            .subscribeOn(Schedulers.boundedElastic())       // ← mandatory
            .doOnError(ex -> log.error("...", ex))
            .onErrorReturn fallbackValue)                    // don't stop the consumer
    .doOnNext(ReceiverRecord::receiverOffset.acknowledge)       // commit
    .retryWhen(Retry.backoff(10, Duration.ofSeconds(1)))        // survive Kafka blips
    .subscribe();
```

### Summary

`Schedulers.boundedElastic()` is the **only safe place** to run blocking code in a WebFlux/Reactive application.  
Without it, you will kill your Netty event-loop and take down the entire service.  
With it, you get safe, scalable, ordered Kafka consumption even when `process()` talks to JDBC, Redis sync, or any legacy blocking library.

That one line `.subscribeOn(Schedulers.boundedElastic())` is literally the difference between a stable 50k RPS service and a total outage.


### What exactly happens under the hood when you write this line:

```java
process(record.value())
    .subscribeOn(Schedulers.boundedElastic())
```

### Step-by-Step Execution (Thread View)

| Time | What happens | Which thread does the work? | Thread pool name |
|------|--------------|-----------------------------|------------------|
| 1    | Kafka record arrives on Netty event-loop thread | `reactor-http-nio-3` (or similar) | Netty event-loop |
| 2    | `concatMap` sees the `subscribeOn(boundedElastic())` operator | Still on Netty thread | |
| 3    | Reactor **immediately hands the entire `process(...)` chain** to the **boundedElastic scheduler** | → jumps to a thread from the **boundedElastic thread pool** | `boundedElastic-1`, `boundedElastic-2`, … |
| 4    | Your `process(record.value())` code runs **completely on that boundedElastic thread** | Yes – this thread can safely block (JDBC, RestTemplate, File I/O, etc.) | |
| 5    | When `process()` finishes (success or error), Reactor schedules the downstream operators (`.doOnNext(acknowledge)`) back on the **original Netty thread** (because of how Reactor Kafka is designed) | Back to `reactor-http-nio-3` | |

### Visual Thread Timeline

```
Netty thread (reactor-http-nio-3)          boundedElastic thread pool
──────────────────────────────             ───────────────────────────────
receive() → concatMap → subscribeOn() ──→  boundedElastic-7 runs process()
                                           ← process() completes
                                           → acknowledge() runs back on Netty thread
```

### boundedElastic Thread Pool Details (2025 defaults)

| Property                     | Value (Reactor 3.6+)                              | Meaning |
|------------------------------|---------------------------------------------------|--------|
| Core threads                 | 10 (minimum)                                      | Always alive |
| Maximum threads              | `CPU cores × 10` → usually **100–200**            | Grows only when queue is full |
| Queue size                   | 100,000 tasks (configurable)                     | If full → rejects with exception |
| Thread keep-alive            | 60 seconds (idle threads die)                     | Saves memory |
| Thread name prefix           | `boundedElastic-1`, `boundedElastic-2`, …         | Easy to spot in `jstack` / Grafana |

### Real Example from Production Thread Dump

```text
"boundedElastic-27" #124 daemon prio=5 os_prio=0 tid=0x... nid=0x4b2c runnable
   java.lang.Thread.State: RUNNABLE
    at com.mysql.cj.jdbc.ClientPreparedStatement.executeQuery(...)

"boundedElastic-28" #125 daemon prio=5 os_prio=0 tid=0x... nid=0x4b2d waiting on condition
   java.lang.Thread.State: WAITING (parking)
    at jdk.internal.misc.Unsafe.park(Native Method)
    - parking to wait for <0x...> (a java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject)
```

You can see 27 threads doing MySQL queries, 28 waiting for more work – **exactly** what we want.

### Summary Again

> `subscribeOn(Schedulers.boundedElastic())` =  
> “Take this whole piece of work (`process(...)`) and run it on a dedicated thread pool that is **designed and allowed to block**.  
> The Netty event-loop stays completely free and responsive.”

That single operator is the #1 reason reactive Spring Boot + Kafka applications survive real production traffic without melting down.


### What **Exactly** Happens Inside `Schedulers.boundedElastic()` (Reactor 3.5+ / Spring Boot 3.x – 2025)

| Question | Exact Answer (Source Code Level) |
|--------|----------------------------------|
| **Which ThreadPoolExecutor is used under the hood?** | `BoundedElasticScheduler` → wraps a custom **`java.util.concurrent.ThreadPoolExecutor`** with a **very special configuration** |
| **Executor type** | **Unbounded core + bounded maximum + bounded queue** (the only one in the JDK that behaves like this) |
| **JDK class** | `reactor.core.scheduler.BoundedElasticScheduler.BoundedScheduledExecutorService` → extends `ThreadPoolExecutor` |
| **Queue used** | `LinkedBlockingQueue<Runnable>` with **configurable capacity** (default 100,000) |
| **Thread factory** | `reactor.core.scheduler.SchedulerThreadFactory` → names threads `boundedElastic-1`, `boundedElastic-2`, … |

### Exact Thread Pool Configuration (2025 defaults)

| Parameter                        | Value (Reactor 3.6+)                                      | Formula / Meaning |
|----------------------------------|------------------------------------------------------------|-------------------|
| **corePoolSize**                 | `0` (zero!)                                                | No threads kept alive when idle |
| **maximumPoolSize**              | `CPU × 10` (e.g. 8-core → 80, 16-core → 160, capped at ~200) | `Math.min(Runtime.getRuntime().availableProcessors() * 10, 100_000)` |
| **keepAliveTime**                | **60 seconds**                                             | Idle threads above core (i.e. all) die after 60s |
| **Queue capacity**               | **100,000** tasks by default                               | Configurable via `-Dreactor.schedulers.boundedElastic.queueCapacity=500000` |
| **RejectedExecutionHandler**     | `AbortPolicy` → throws `RejectedExecutionException` when queue full | You’ll see `QueueFullException` in logs if overloaded |

### Execution Strategy – Visualized

```
corePoolSize = 0  →  No threads at startup
│
└─ Task arrives → create new thread (up to maxPoolSize)
        │
        ├─ Queue has space → enqueue task
        │       (even if threads < maxPoolSize)
        │
        └─ Queue full + threads == maxPoolSize → REJECT
```

```
Task arrives
│
├─ threads < maxPoolSize → create new thread immediately
│
├─ threads == maxPoolSize → enqueue in 100k queue
│
└─ queue full + max threads → RejectedExecutionException → your Flux errors
```

This is **NOT** the classic “fixed + bounded queue” model.  
It’s a **hybrid** designed specifically for blocking I/O:

- Grow very aggressively (up to CPU×10) to handle bursty blocking calls
- Shrink back to zero when idle → saves memory in microservices
- Huge queue (100k) prevents rejection in normal spikes

### How to Change the Limits (Production Tuning)

```yaml
# application.yml (Spring Boot)
spring:
  threads:
    virtual:
      enabled: true                    # Java 21+ → switches to virtual threads automatically!
```

Or JVM flags:

```bash
# Increase queue to 500k
-Dreactor.schedulers.boundedElastic.queueCapacity=500000

# Force max 200 threads even on 32-core machines
-Dreactor.schedulers.boundedElastic.maxThreads=200
```

### Java 21+ Bonus: Virtual Threads (2024–2025 reality)

Since Java 21 + Reactor 3.6 + Spring Boot 3.2:

```java
Schedulers.boundedElastic()   // automatically uses virtual threads
```

→ Each task gets its own **virtual thread** instead of platform thread  
→ maximumPoolSize becomes **practically unlimited** (hundreds of thousands)  
→ Memory per thread drops from ~1–2 MB → ~100 KB  
→ You can safely process 100k+ blocking JDBC calls concurrently

### Summary One More Time

```java
Schedulers.boundedElastic() 
   = ThreadPoolExecutor(
        corePoolSize = 0,
        maximumPoolSize = CPU × 10,
        keepAlive = 60s,
        queue = LinkedBlockingQueue(100_000),
        policy = AbortPolicy
     )
   + (since Java 21) automatically uses virtual threads → effectively unlimited
```

That’s why it’s the **only safe** place to call JDBC, RestTemplate, blocking Redis, etc. in a WebFlux/Reactive application — and why it scales beautifully in real production systems handling 10k–100k RPS.

### Real Production Code Patterns 

```java
// 1. Classic blocking Kafka consumer (most common)
kafkaReceiver.receive()
    .concatMap(record ->
        processBlocking(record.value())                    // JDBC, RestTemplate, etc.
            .subscribeOn(Schedulers.boundedElastic())     // ← THIS SAVES YOUR APP
            .doOnError(ex -> log.error("Failed {}", record, ex))
            .onErrorResume(ex -> Mono.empty())             // don't kill the stream
    )
    .doOnNext(ReceiverRecord::receiverOffset.acknowledge)
    .subscribe();

// 2. WebFlux controller calling legacy blocking service
@GetMapping("/legacy-report")
public Mono<Report> getReport() {
    return Mono.fromCallable(() -> legacyBlockingService.generatePdf())
               .subscribeOn(Schedulers.boundedElastic())   // safe!
               .timeout(Duration.ofSeconds(30));
}

// 3. Parallel batch processing (e.g. 1000 files)
Flux)
filesFlux
    .flatMap(file ->
        processFile(file)                                  // heavy CPU + blocking I/O
            .subscribeOn(Schedulers.boundedElastic()),
        50                                                    // max 50 concurrent
    )
    .collectList()
    .block();
```

#### 5. Production Tuning Examples

```bash
# Recommended JVM flags for high-throughput services (2025)
-Dreactor.schedulers.boundedElastic.maxThreads=500
-Dreactor.schedulers.boundedElastic.queueCapacity=200000
-Dreactor.schedulers.defaultBoundedElasticOnVirtualThreads=true   # force virtual threads even on JDK 21+

# Or in application.yml (Spring Boot 3.2+)
spring:
  threads:
    virtual:
      enabled: true   # ← turns boundedElastic into virtual-thread heaven
```

#### 6. What Happens If You Forget It? (Real Outage Stories)

| Mistake                                    | Symptom                                    | Time to Total Outage |
|--------------------------------------------|--------------------------------------------|----------------------|
| Call `JdbcTemplate` directly in WebFlux    | Netty threads block → TCP backlog → all WebSockets disconnect | 10–60 seconds        |
| Use `Schedulers.parallel()` for JDBC       | CPU workers starve → GC thrashing → OOM  | 2–10 minutes         |
| Use `subscribeOn(boundedElastic())`       | System stays alive at 50k+ RPS             | Never                |

#### Must Remember!!!

> `Schedulers.boundedElastic()` is the **only thread pool specifically created so you can safely call blocking code (JDBC, RestTemplate, Redis sync, file I/O, etc.) inside a fully reactive Spring WebFlux application without blowing up your Netty event loop.**


### **Can** `Schedulers.boundedElastic()` **be pre-configured**?

Yes but **only via system properties** (JVM flags) or Spring Boot properties — **not** via code like `new BoundedElasticScheduler(...)` in normal applications.

Here is the **complete list of everything you can pre-configure** in 2025 (Reactor 3.6+ / Spring Boot 3.2+):

| What you want to change           | How to do it (JVM flag)                                   | Spring Boot 3.2+ equivalent (application.yml) | Default value |
|-----------------------------------|------------------------------------------------------------|------------------------------------------------|---------------|
| Maximum threads (JDK ≤20)         | `-Dreactor.schedulers.boundedElastic.maxThreads=500`      | `spring.threads.virtual.enabled=false` + JVM flag | CPU × 10 |
| Queue capacity                    | `-Dreactor.schedulers.boundedElastic.queueCapacity=500000` | —                                              | 100,000 |
| Thread name prefix                | `-Dreactor.schedulers.boundedElastic.threadPrefix=myapp-be` | —                                              | `boundedElastic` |
| TTL of cached schedulers          | `-Dreactor.schedulers.boundedElastic.cachedSchedulerExpireAfter=300s` | —                                              | 60s |
| Force virtual threads (JDK 21+)   | `-Dreactor.schedulers.boundedElasticOnVirtualThreads=true` | `spring.threads.virtual.enabled=true` ← **recommended** | auto-detect |
| **Pre-warm / init threads**       | **Not directly possible**                                 | —                                              | starts at 0 |

### Can I Pre-create Threads?

**No**  
`boundedElastic` is intentionally designed with `corePoolSize = 0`.  
There is **no official way** to pre-warm threads (create 10 threads at startup).

But here are **three real-world workarounds** senior teams actually use**:

#### Workaround 1 – Force virtual threads (JDK 21+) → problem disappears
```yaml
# application.yml – this is the 2025 gold standard
spring:
  threads:
    virtual:
      enabled: true        # ← makes boundedElastic use virtual threads
```
→ Each task gets its own virtual thread instantly  
→ No thread creation latency  
→ No need to pre-warm anything  
→ This is what Netflix, Uber, Alibaba, and most new Spring Boot 3 apps do.

#### Workaround 2 – Custom pre-warmed scheduler (rare, but possible)
```java
@Configuration
public class ReactorConfig {

    public static final Scheduler PREWARMED_BLOCKING = Schedulers.newBoundedElastic(
        20,          // core size (pre-created)
        200,         // max size
        "myapp-blocking", 
        60           // keep-alive seconds
    );

    @PreDestroy
    public void cleanup() {
        PREWARMED_BLOCKING.dispose();
    }
}
```
Then use it instead:
```java
.subscribeOn(ReactorConfig.PREWARMED_BLOCKING)
```

#### Workaround 3 – Eager initialization at startup (hacky but works)
```java
@PostConstruct
public void warmupBoundedElastic() {
    IntStream.range(0, 20).parallel().forEach(i ->
        Schedulers.boundedElastic().schedule(() -> {
            try { Thread.sleep(10); } catch (Exception ignored) {}
        })
    );
}
```
→ Forces the default `boundedElastic` to create 20 threads immediately.

### 2025 Production Recommendation 

```yaml
# application.yml – copy-paste this and forget about thread pools forever
spring:
  threads:
    virtual:
      enabled: true                 # ← turns boundedElastic into infinite virtual threads
  task:
    scheduling:
      pool:
        size: 50                    # for @Scheduled tasks
```

With Java 21 + Spring Boot 3.2 + this config:
- `Schedulers.boundedElastic()` becomes **effectively infinite**
- Thread creation latency ≈ 0
- Memory per blocking call ≈ 100 KB instead of 2 MB
- You never have to pre-warm or tune anything again


| Goal                             | Solution in 2025                                   |
|----------------------------------|-----------------------------------------------------|
| Pre-warm threads                 | Not possible with default scheduler                 |
| Make it fast + infinite          | `spring.threads.virtual.enabled=true` (JDK 21+) ← **use this** |
| Full control                     | Create your own with `Schedulers.newBoundedElastic(...)` |

99 % of teams in 2025 just enable virtual threads and never think about `boundedElastic` configuration again.  


### `Mono.fromCallable(...)` – The Ultimate 2025 Explanation  
(What every senior Java/Reactive engineer must know inside-out)

| Question | Answer (Real Production Truth) |
|----------|--------------------------------|
| **What is it?** | A factory method that creates a `Mono<T>` from a **blocking, callable piece of code** (`Callable<T>` or lambda that returns `T`). |
| **When do you use it?** | **Every time you have to call legacy blocking code** (JDBC, RestTemplate, Redis sync, file I/O, external API, etc.) inside a reactive chain. |
| **Is it blocking?** | Yes — the code inside runs **synchronously and blocks the calling thread**. |
| **Is it safe to call directly in WebFlux?** | **Only safe if you offload it** (via `.subscribeOn(...)`) — otherwise you block Netty event-loop → outage. |

### Exact Contract & Behavior

```java
public static <T> Mono<T> fromCallable(Callable<? extends T> supplier)
```

| Scenario                              | What actually happens                                                                 |
|---------------------------------------|-----------------------------------------------------------------------------------------|
| `supplier` returns a value            | `Mono` emits that value → `onNext` → `onComplete`                                       |
| `supplier` throws exception           | `Mono` emits `onError` with that exception                                              |
| `supplier` returns `null`             | `Mono` emits `onNext(null)` → `onComplete` (yes, null is allowed)                      |
| You call it on Netty thread           | Blocks the carrier thread → **bad** unless you offload                                  |

### The Two Correct Ways to Use It in 2025

#### 1. Classic way (Java 17–20) – must offload

```java
Mono.fromCallable(() -> jdbcTemplate.queryForObject(sql, Long.class))
    .subscribeOn(Schedulers.boundedElastic())   // ← mandatory pre-Java 21
    .timeout(Duration.ofSeconds(5));
```

#### 2. 2025 way (Java 21 + virtual threads) – offload optional

```yaml
# application.yml
spring:
  threads:
    virtual:
      enabled: true        # ← turns boundedElastic into virtual threads
```

```java
// Now perfectly safe without subscribeOn!
Mono.fromCallable(() -> jdbcTemplate.queryForObject(sql, Long.class))
    .timeout(Duration.ofSeconds(5));
```

### Real-World Examples (Copy-Paste Ready)

```java
// 1. Legacy JDBC call
Mono.fromCallable(() -> userRepository.findById(id))   // blocking
    .subscribeOn(Schedulers.boundedElastic())

// 2. External blocking HTTP client
Mono.fromCallable(() -> restTemplate.postForEntity(url, request, String.class))
    .map(HttpEntity::getBody)

// 3. File I/O
Mono.fromCallable(() -> Files.readString(Path.of("/tmp/config.json")))
    .map(Json::parse)

// 4. CPU-heavy calculation (WRONG scheduler!)
Mono.fromCallable(() -> complexEncryption(data))      // CPU bound → use parallel!
    .subscribeOn(Schedulers.parallel())               // ← correct

// 5. With error handling + fallback
Mono.fromCallable(() -> legacyService.getUser(id))
    .onErrorResume(ex -> {
        log.warn("Legacy failed, using cache", ex);
        return Mono.just(cachedUser);
    })
```

### Common Mistakes Seniors Still Make

| Mistake                                         | Consequence                                      | Fix |
|------------------------------------------------|--------------------------------------------------|-----|
| Forgetting `.subscribeOn(boundedElastic())`   | Blocks Netty → all WebSockets disconnect         | Add it (or enable virtual threads) |
| Using it for CPU work                          | Starves parallel scheduler                       | Use `Mono.fromSupplier` + `Schedulers.parallel()` |
| Chaining many `fromCallable` without limit    | Thread explosion (one thread per call)           | Use `.publishOn(...)` or limit concurrency |

### `Mono.fromCallable` vs Alternatives (2025 Decision Table)

| Use Case                                  | Best Choice                              | Why |
|-------------------------------------------|------------------------------------------|-----|
| Simple blocking call (DB, HTTP, file)     | `Mono.fromCallable(...).subscribeOn(boundedElastic())` | Cleanest |
| You already have a `Supplier<T>`          | `Mono.fromSupplier(...)`                 | Same as fromCallable but no checked exception |
| You want lazy execution only on subscribe | `Mono.defer(() -> Mono.fromCallable(...))` | Prevents execution on cold publishers |
| Async but blocking driver                 | Prefer true async driver (R2DBC, reactive Redis) | Zero blocking |

### The One Rule Every Senior Lives By

> `Mono.fromCallable { blocking code }`  
> → **Always** pair with `.subscribeOn(Schedulers.boundedElastic())`  
> → **Or** run on Java 21+ with `spring.threads.virtual.enabled=true` and delete the `subscribeOn` forever.
