---
title: Spring WebFlux Guide - 2025
date: 2025-12-02 23:05:55
categories:
- WebFlux
tags:
- WebFlux
---

This is a comprehensive deep dive into the most crucial operators in **Project Reactor**, which is the foundation of **Spring WebFlux**. Understanding these operators is key to mastering non-blocking, reactive programming.

The operators are organized into functional categories, detailing their usage and real-world application.

---

## 1. 🏗️ Composition and Transformation Operators

These operators modify the data emitted by a `Publisher` (Mono or Flux).

| Operator | Signature & Description | Key Difference | Real-World Use Case |
| :--- | :--- | :--- | :--- |
| **`map()`** | `T -> V` (Synchronous transformation) | Applies a synchronous function to each item. **Returns a simple value.** | Converting a database entity (`UserEntity`) into a Data Transfer Object (`UserDTO`) after retrieval. |
| **`flatMap()`** | `T -> Publisher<V>` (Asynchronous transformation) | Applies an asynchronous function that returns a new `Publisher`. Essential for **chaining non-blocking I/O operations.** | Chaining microservice calls: Get `userId` (Mono A) $\rightarrow$ Use `userId` to call an external service (Mono B) $\rightarrow$ Use result B to save to the database (Mono C). |
| **`concatMap()`** | `T -> Publisher<V>` (Sequential `flatMap`) | Similar to `flatMap`, but it processes and emits the results of the inner `Publisher` **sequentially**, preserving the original order of elements. | Processing file uploads: Ensures that multiple files in a list are processed and saved to storage one after the other, respecting the order received, even though the I/O operations are async. |
| **`filter()`** | `T -> Boolean` | Selectively includes items based on a predicate. | Excluding users with an `isActive: false` status before returning a list of clients. |
| **`cast()`** | `T -> V` (Type conversion) | Converts each element to a specified type. | Converting an object retrieved from a generic cache (`Object`) to its expected type (`Product`). |

---

## 2. 🤝 Combination and Merging Operators

These operators combine the output from two or more source `Publishers`.

| Operator | Signature & Description | Key Difference | Real-World Use Case |
| :--- | :--- | :--- | :--- |
| **`merge()`** | `Publisher<T>... -> Flux<T>` (Non-deterministic) | **Interleaves** the elements from multiple sources as they arrive. The output order **is not guaranteed** and depends on timing. | Combining real-time notifications: Combining two async streams of notifications (e.g., email and SMS alerts) into one feed where the order of arrival is acceptable. |
| **`zip()`** | `Publisher<T>, Publisher<V> -> Mono<Tuple2<T, V>>` (Pairwise) | **Combines the *n*-th element** from each source into a `Tuple` (or custom object) once all sources have emitted an element. | Parallel data enrichment: Fetching a user profile (`Mono<User>`) and their recent transaction history (`Mono<History>`) concurrently, then combining both results into a single object for the API response. |
| **`combineLatest()`** | `Publisher<T>, Publisher<V> -> Flux<Tuple2<T, V>>` (Emission-driven) | Emits a new item whenever **any** source emits an item, combining the newly emitted item with the **latest** item from all other sources. | Real-time dashboards: Combining a stream of stock prices with a stream of news headlines. A price update triggers a new emission with the latest price and the latest headline. |
| **`concat()`** | `Publisher<T>... -> Flux<T>` (Sequential) | **Appends** one `Publisher` after another. The second source does not start emitting until the first source completes. | Fallback data loading: First attempt to load data from a fast cache (`Mono A`). If A completes empty, immediately execute a database query (`Mono B`). |

---

## 3. ⏱️ Error Handling and Resilience Operators

These operators manage exceptions and provide fallbacks.

| Operator | Signature & Description | Key Difference | Real-World Use Case |
| :--- | :--- | :--- | :--- |
| **`onErrorReturn()`** | `Class<E>, T` (Simple fallback value) | When an error of a specific type occurs, it immediately **replaces the error signal with a default, static value** and completes the sequence normally. | Handling an external API timeout: If the `weatherService.get()` call fails with a `TimeoutException`, return a cached default value like `"Weather Unknown"` instead of propagating the error. |
| **`onErrorResume()`** | `Class<E>, Function<E, Publisher<V>>` (Fallback publisher) | When an error occurs, it **switches to a fallback `Publisher`**. | Database failure: If the primary database query fails, switch to a fallback query that reads from a reliable but slightly stale read replica. |
| **`retry()`** | `long times` (Retry N times) | **Resubscribes** to the source `Publisher` upon receiving an error signal, up to a specified number of times. | Transient network errors: Automatically retrying an external HTTP call up to three times to overcome temporary network hiccups before giving up. |
| **`doOnError()`** | `Consumer<Throwable>` (Side effect) | Allows execution of a **side effect** (logging, metrics tracking) when an error occurs, without modifying or consuming the error itself. The error is still passed downstream. | Logging exceptions: Recording the full stack trace and metrics to a monitoring system before the error is propagated up the pipeline to be handled by an exception handler. |

---

## 4. 🔗 Backpressure and Hot/Cold Operators

These manage resource flow and execution model.

| Operator | Signature & Description | Key Difference | Real-World Use Case |
| :--- | :--- | :--- | :--- |
| **`subscribeOn()`** | `Scheduler` (Where execution starts) | Determines the **Scheduler (thread)** on which the entire source execution pipeline begins. Crucial for moving blocking operations off the main Event Loop. | **Moving Blocking Code:** Ensuring a blocking JDBC call or a CPU-intensive synchronous task runs on `Schedulers.boundedElastic()`. |
| **`publishOn()`** | `Scheduler` (Where subsequent ops run) | Changes the `Scheduler` used by **all operators downstream** of this call. Does not affect the source. | **Context Switching:** After a heavy calculation, switching back to `Schedulers.parallel()` for non-blocking operations to maximize efficiency. |
| **`share()`** | `Mono<T> -> Mono<T>` (Multi-subscription) | Allows multiple subscribers to share a single subscription to the source `Publisher`, ensuring the source logic is executed only once. | Caching a slow computation: Executing an expensive initialization task (e.g., loading a large configuration file) once, then providing that result to dozens of services without re-running the load logic. |
| **`delayElements()`** | `Duration` (Time-based pacing) | Introduces a delay between the emissions of elements, effectively implementing **backpressure by time**. | Throttling a data stream: Pacing a fast stream of data to match the input rate required by a slower downstream component (like a legacy API with strict rate limits). |

# Deepseek
# **Spring WebFlux Operators: Comprehensive Guide**
## Reactive Programming Operators Deep Dive

---

## **Table of Contents**
1. [Core Concepts & Reactor Basics](#1-core-concepts--reactor-basics)
2. [Creation Operators](#2-creation-operators)
3. [Transformation Operators](#3-transformation-operators)
4. [Filtering Operators](#4-filtering-operators)
5. [Combining Operators](#5-combining-operators)
6. [Error Handling Operators](#6-error-handling-operators)
7. [Utility Operators](#7-utility-operators)
8. [Schedulers & Threading](#8-schedulers--threading)
9. [Backpressure Operators](#9-backpressure-operators)
10. [Real-World Patterns](#10-real-world-patterns)

---

## **1. Core Concepts & Reactor Basics**

### **Reactive Streams Specification**
```java
// Core Interfaces
Publisher<T>    // Emits data
Subscriber<T>   // Consumes data
Subscription    // Controls flow (backpressure)
Processor<T,R>  // Both publisher and subscriber

// Reactor Implementations
Mono<T>         // 0-1 element
Flux<T>         // 0-N elements
```

### **Cold vs Hot Publishers**
```java
// COLD: Each subscriber gets fresh data stream
Flux<Integer> coldFlux = Flux.range(1, 5)
    .doOnSubscribe(s -> System.out.println("New subscription"));

coldFlux.subscribe(); // Prints: New subscription
coldFlux.subscribe(); // Prints: New subscription (again)

// HOT: Shares data among subscribers
ConnectableFlux<Integer> hotFlux = Flux.interval(Duration.ofSeconds(1))
    .publish(); // Convert to hot publisher

hotFlux.connect(); // Start emitting
hotFlux.subscribe(v -> System.out.println("Sub1: " + v));
Thread.sleep(3000);
hotFlux.subscribe(v -> System.out.println("Sub2: " + v)); // Misses first 3 values

// Use case: Real-time stock prices
@RestController
public class StockController {
    private final ConnectableFlux<StockPrice> priceStream;
    
    public StockController() {
        this.priceStream = Flux.interval(Duration.ofMillis(100))
            .map(tick -> fetchStockPrice())
            .publish();
        this.priceStream.connect();
    }
    
    @GetMapping("/prices")
    public Flux<StockPrice> streamPrices() {
        return priceStream;
    }
}
```

---

## **2. Creation Operators**

### **Static Creation Methods**
```java
// 1. just() - Fixed values
Mono.just("Hello")
Flux.just("A", "B", "C")

// 2. empty() - Complete without value
Mono.empty()  // Completes immediately
Flux.empty()  // Completes immediately

// 3. error() - Complete with error
Mono.error(new RuntimeException("Failed"))
Flux.error(new RuntimeException("Failed"))

// 4. never() - Never completes (for testing)
Mono.never()
Flux.never()

// 5. from...() - From existing sources
Flux.fromIterable(List.of(1, 2, 3))
Flux.fromStream(Stream.of(1, 2, 3))
Flux.fromArray(new Integer[]{1, 2, 3})
Mono.fromFuture(CompletableFuture.supplyAsync(() -> "result"))
Mono.fromCallable(() -> database.query())
Mono.fromRunnable(() -> cleanup()) // Returns Mono<Void>

// 6. range() - Number sequence
Flux.range(1, 10) // 1,2,3,...,10

// 7. interval() - Periodic emissions
Flux.interval(Duration.ofSeconds(1)) // 0,1,2,3,... every second
    .take(5) // 0,1,2,3,4

// 8. defer() - Deferred creation (per subscription)
Flux.defer(() -> {
    if (System.currentTimeMillis() % 2 == 0) {
        return Flux.just("Even");
    } else {
        return Flux.just("Odd");
    }
});
// Each subscription gets potentially different result

// REAL-WORLD: Database connection per request
public Mono<User> getUser(String id) {
    return Mono.defer(() -> {
        // Create new connection per subscription
        return Mono.fromCallable(() -> userRepository.findById(id))
            .subscribeOn(Schedulers.boundedElastic());
    });
}
```

### **Programmatic Creation**
```java
// 9. create() - Bridge with callback APIs
Flux.create(sink -> {
    // Bridge to event-based APIs
    EventListener listener = event -> {
        sink.next(event.getData());
        if (event.isComplete()) {
            sink.complete();
        }
    };
    eventSource.register(listener);
    
    // Cleanup on cancel
    sink.onCancel(() -> eventSource.unregister(listener));
    sink.onDispose(() -> eventSource.unregister(listener));
});

// REAL-WORLD: WebSocket to Flux bridge
public Flux<Message> createWebSocketStream(WebSocketSession session) {
    return Flux.create(sink -> {
        session.sendMessage(new TextMessage("CONNECTED"));
        
        session.addHandler(new WebSocketHandler() {
            @Override
            public void handleMessage(WebSocketSession session, WebSocketMessage<?> message) {
                sink.next(convert(message));
            }
            
            @Override
            public void afterConnectionClosed(WebSocketSession session, CloseStatus status) {
                sink.complete();
            }
        });
        
        sink.onDispose(() -> session.close());
    });
}

// 10. generate() - Synchronous, stateful generation
Flux.generate(
    () -> 0, // Initial state
    (state, sink) -> {
        sink.next("Value: " + state);
        if (state == 10) {
            sink.complete();
        }
        return state + 1; // New state
    }
);

// 11. push() - Single-threaded async generation (like create but single-threaded)
Flux.push(sink -> {
    // For single-threaded producers
    singleThreadedEventEmitter.onData(data -> sink.next(data));
    singleThreadedEventEmitter.onComplete(() -> sink.complete());
});

// 12. using() - Resource management
Flux.using(
    () -> database.getConnection(),           // Resource supplier
    connection -> Flux.fromIterable(connection.query()), // Flux factory
    connection -> connection.close()          // Cleanup
);
```

---

## **3. Transformation Operators**

### **Element Transformation**
```java
// 1. map() - Synchronous 1:1 transformation
Flux<Integer> numbers = Flux.range(1, 5);
Flux<String> strings = numbers.map(n -> "Number: " + n);
// 1 → "Number: 1", 2 → "Number: 2", ...

// 2. flatMap() - Asynchronous 1:N transformation
Flux<User> users = Flux.just("user1", "user2", "user3");
Flux<Order> orders = users.flatMap(username -> 
    orderRepository.findByUser(username) // Returns Flux<Order>
);
// Concatenates all inner Flux emissions

// 3. concatMap() - Preserves order (slower)
users.concatMap(username -> 
    orderRepository.findByUser(username)
);
// Waits for each inner Flux to complete before starting next

// 4. flatMapSequential() - Preserves source order but subscribes eagerly
users.flatMapSequential(username -> 
    orderRepository.findByUser(username)
);

// REAL-WORLD: Parallel API calls with ordering
public Flux<Product> getProductsWithReviews(List<String> productIds) {
    return Flux.fromIterable(productIds)
        .flatMap(id -> 
            productService.getProduct(id)  // Mono<Product>
                .flatMap(product ->
                    reviewService.getReviews(id)  // Mono<List<Review>>
                        .map(reviews -> {
                            product.setReviews(reviews);
                            return product;
                        })
                )
        );
}

// 5. switchMap() - Cancels previous inner publisher
searchInputFlux
    .debounce(Duration.ofMillis(300))
    .switchMap(query -> 
        searchService.search(query) // Cancels previous search if new query arrives
    );

// 6. transform() - Operator composition
Function<Flux<String>, Flux<String>> filterAndTransform =
    flux -> flux
        .filter(s -> !s.isEmpty())
        .map(String::toUpperCase);

Flux<String> result = Flux.just("hello", "", "world")
    .transform(filterAndTransform);
```

### **Flattening Operators**
```java
// 7. flatMap vs flatMapMany
Mono<List<Integer>> monoList = Mono.just(List.of(1, 2, 3));
Flux<Integer> flux = monoList.flatMapMany(Flux::fromIterable);

// 8. concatMap vs flatMap comparison
Flux.just(1, 2, 3)
    .flatMap(i -> 
        Mono.just(i * 10)
            .delayElement(Duration.ofMillis(i * 100))
    );
// Output order: 10, 20, 30 (may vary due to delays)

Flux.just(1, 2, 3)
    .concatMap(i -> 
        Mono.just(i * 10)
            .delayElement(Duration.ofMillis(i * 100))
    );
// Output order: 10, 20, 30 (guaranteed order)

// REAL-WORLD: File processing with backpressure
public Flux<String> processLargeFile(String filePath) {
    return Flux.using(
        () -> Files.lines(Paths.get(filePath)),
        Flux::fromStream,
        Stream::close
    )
    .flatMap(line -> 
        Mono.fromCallable(() -> expensiveProcessing(line))
            .subscribeOn(Schedulers.boundedElastic()),
        5 // Maximum concurrency
    );
}

// 9. groupBy() - Group elements by key
Flux<Transaction> transactions = getTransactions();
Flux<GroupedFlux<String, Transaction>> grouped = 
    transactions.groupBy(tx -> tx.getCategory());

grouped.flatMap(group -> 
    group
        .buffer(Duration.ofSeconds(1))
        .map(list -> new CategorySummary(group.key(), list))
);

// 10. window() - Split into windows
Flux.range(1, 100)
    .window(10) // Flux<Flux<Integer>> with windows of size 10
    .flatMap(window -> 
        window.reduce(0, Integer::sum)
            .map(sum -> "Window sum: " + sum)
    );

// Time-based windowing
Flux.interval(Duration.ofMillis(100))
    .window(Duration.ofSeconds(1))
    .flatMap(window -> window.count())
    .subscribe(count -> System.out.println("Events per second: " + count));
```

### **Buffering & Windowing**
```java
// 1. buffer() - Collect into lists
Flux.range(1, 10)
    .buffer(3) // [1,2,3], [4,5,6], [7,8,9], [10]
    .subscribe(list -> System.out.println(list));

// 2. bufferTimeout() - Size OR time based
messageFlux
    .bufferTimeout(100, Duration.ofSeconds(1))
    .subscribe(batch -> batchProcessor.process(batch));

// 3. bufferWhile() / bufferUntil()
Flux<Integer> numbers = Flux.range(1, 10);
numbers.bufferWhile(n -> n % 2 == 0); // Buffer while even
numbers.bufferUntil(n -> n % 3 == 0); // Buffer until divisible by 3

// REAL-WORLD: Batch database inserts
public Mono<Void> saveMessages(Flux<Message> messages) {
    return messages
        .buffer(100) // Batch size
        .delayElements(Duration.ofMillis(100)) // Throttle
        .flatMap(batch -> 
            Mono.fromRunnable(() -> batchInsert(batch))
                .subscribeOn(Schedulers.boundedElastic()),
            3 // Max concurrent batches
        )
        .then();
}

// 4. window() vs buffer()
Flux<Integer> source = Flux.range(1, 10);
source.buffer(3);  // Returns Flux<List<Integer>>
source.window(3);  // Returns Flux<Flux<Integer>>

// 5. windowUntilChanged() - Window on changing values
Flux.just(1, 1, 2, 2, 1, 3, 3)
    .windowUntilChanged()
    .flatMap(window -> window.collectList())
// Windows: [1,1], [2,2], [1], [3,3]
```

---

## **4. Filtering Operators**

### **Basic Filtering**
```java
// 1. filter() - Predicate-based filtering
Flux<Integer> numbers = Flux.range(1, 10);
Flux<Integer> even = numbers.filter(n -> n % 2 == 0);

// 2. ignoreElements() - Ignore all, just completion/error
Mono<Void> completion = Flux.just(1, 2, 3)
    .ignoreElements(); // Only onComplete/onError signals

// 3. distinct() - Remove duplicates
Flux.just(1, 2, 2, 3, 1, 4)
    .distinct() // 1, 2, 3, 4

// 4. distinctUntilChanged() - Remove consecutive duplicates
Flux.just(1, 1, 2, 2, 1, 1)
    .distinctUntilChanged() // 1, 2, 1

// REAL-WORLD: Rate limiting duplicate requests
public Flux<Response> handleRequests(Flux<Request> requests) {
    return requests
        .distinct(Request::getId) // Ignore duplicate IDs
        .filter(req -> !isBlocked(req.getIp()))
        .flatMap(this::processRequest);
}
```

### **Taking & Skipping**
```java
// 1. take(n) - Take first N elements
Flux.range(1, 100).take(5); // 1,2,3,4,5

// 2. takeLast(n) - Take last N elements
Flux.range(1, 100).takeLast(5); // 96,97,98,99,100

// 3. takeWhile(predicate) - Take while true
Flux.range(1, 10)
    .takeWhile(n -> n < 5); // 1,2,3,4

// 4. takeUntil(predicate) - Take until true (inclusive)
Flux.range(1, 10)
    .takeUntil(n -> n == 5); // 1,2,3,4,5

// 5. skip(n) - Skip first N elements
Flux.range(1, 10).skip(5); // 6,7,8,9,10

// 6. skipLast(n) - Skip last N elements
Flux.range(1, 10).skipLast(5); // 1,2,3,4,5

// 7. skipWhile(predicate) - Skip while true
Flux.range(1, 10)
    .skipWhile(n -> n < 5); // 5,6,7,8,9,10

// 8. skipUntil(predicate) - Skip until true
Flux.range(1, 10)
    .skipUntil(n -> n == 5); // 5,6,7,8,9,10

// REAL-WORLD: Pagination with take/skip
public Flux<User> getUsers(int page, int size) {
    return userRepository.findAll()
        .skip(page * size)
        .take(size);
}

// 9. takeDuration() / skipDuration() - Time-based
Flux.interval(Duration.ofMillis(100))
    .take(Duration.ofSeconds(1)) // Take for 1 second
    .subscribe();

// 10. elementAt() - Get element at index
Flux.range(1, 10)
    .elementAt(5); // Mono with value 6
```

### **Sampling & Throttling**
```java
// 1. sample(Duration) - Last element in time window
Flux.interval(Duration.ofMillis(100))
    .sample(Duration.ofSeconds(1)) // Emit last value every second
    .subscribe(System.out::println);

// 2. sampleFirst(Duration) - First element in time window
messageFlux.sampleFirst(Duration.ofMillis(100));

// 3. sampleTimeout() - Sample with dynamic duration
Flux.interval(Duration.ofMillis(100))
    .sampleTimeout(
        v -> Mono.delay(Duration.ofMillis(50 + v * 10))
    );

// 4. throttleFirst / throttleLast (alias for sample)
Flux.interval(Duration.ofMillis(50))
    .throttleFirst(Duration.ofSeconds(1)) // First in window
    .throttleLast(Duration.ofSeconds(1))  // Last in window (same as sample)

// REAL-WORLD: UI event throttling
@Component
public class SearchService {
    public Flux<Result> search(Flux<String> queries) {
        return queries
            .filter(query -> query.length() > 2)
            .debounce(Duration.ofMillis(300)) // Wait for pause
            .distinctUntilChanged() // Ignore same query
            .switchMap(this::executeSearch); // Cancel previous
    }
}

// 5. debounce - Wait for quiet period
buttonClickFlux
    .debounce(Duration.ofMillis(250)) // Prevent double clicks
    .subscribe(this::handleClick);
```

---

## **5. Combining Operators**

### **Sequential Combination**
```java
// 1. concatWith() - Sequential concatenation
Flux<Integer> flux1 = Flux.just(1, 2, 3);
Flux<Integer> flux2 = Flux.just(4, 5, 6);
flux1.concatWith(flux2); // 1,2,3,4,5,6

// 2. concat() - Static version
Flux.concat(flux1, flux2, Flux.just(7, 8));

// 3. concatDelayError() - Delay errors until end
Flux.concatDelayError(
    Flux.just(1, 2).concatWith(Mono.error(new RuntimeException())),
    Flux.just(3, 4)
); // Emits: 1,2,3,4 then error

// REAL-WORLD: Fallback service calls
public Flux<Product> getProducts(String category) {
    return primaryService.getProducts(category)
        .concatWith(secondaryService.getProducts(category))
        .take(10); // Get up to 10 from either source
}
```

### **Parallel Combination**
```java
// 1. mergeWith() - Interleave emissions
Flux<Integer> flux1 = Flux.interval(Duration.ofMillis(100))
    .map(i -> i * 2);
Flux<Integer> flux2 = Flux.interval(Duration.ofMillis(150))
    .map(i -> i * 2 + 1);

flux1.mergeWith(flux2); // Interleaved: 0,1,2,3,4,5,...

// 2. merge() - Static version
Flux.merge(flux1, flux2, flux3);

// 3. mergeSequential() - Subscribe in order, emit as arrive
Flux.mergeSequential(flux1, flux2); // Maintains subscription order

// 4. mergeOrdered() - Merge with ordering
Flux.mergeOrdered(
    Comparator.naturalOrder(),
    flux1, flux2
);

// REAL-WORLD: Aggregating multiple data sources
public Flux<StockPrice> getAllStockPrices(List<String> symbols) {
    return Flux.fromIterable(symbols)
        .flatMap(symbol -> 
            stockService.getPriceStream(symbol),
            5 // Max concurrent subscriptions
        );
}
```

### **Pairing Operators**
```java
// 1. zipWith() - Pair-wise combination
Flux<Integer> numbers = Flux.range(1, 3);
Flux<String> letters = Flux.just("A", "B", "C");

numbers.zipWith(letters, (n, l) -> n + l);
// [1A, 2B, 3C]

// 2. zip() - Static version
Flux.zip(numbers, letters, more)
    .map(tuple -> tuple.getT1() + tuple.getT2());

// 3. zip with combinator
Flux.zip(
    obj -> new Combined((String)obj[0], (Integer)obj[1]),
    flux1, flux2
);

// REAL-WORLD: Parallel API calls with aggregation
public Mono<UserProfile> getUserProfile(String userId) {
    return Mono.zip(
        userService.getUser(userId),
        orderService.getOrders(userId),
        paymentService.getPaymentMethods(userId)
    ).map(tuple -> new UserProfile(
        tuple.getT1(),
        tuple.getT2(),
        tuple.getT3()
    ));
}

// 4. combineLatest - Latest from each
Flux.combineLatest(
    flux1, flux2,
    (v1, v2) -> v1 + " - " + v2
);

// REAL-WORLD: Real-time dashboard
public Flux<DashboardData> getDashboard() {
    return Flux.combineLatest(
        cpuMetrics,
        memoryMetrics,
        networkMetrics,
        (cpu, memory, network) -> new DashboardData(cpu, memory, network)
    );
}
```

### **Race Conditions**
```java
// 1. firstWithSignal / firstWithValue - Race to emit
Mono<String> cache = getFromCache(key);
Mono<String> db = getFromDatabase(key);

Mono.firstWithSignal(cache, db) // First to emit any signal
    .subscribe();

Mono.firstWithValue(cache, db) // First to emit a value
    .subscribe();

// REAL-WORLD: Circuit breaker with fallbacks
public Mono<Response> callWithFallback(Request request) {
    return primaryService.call(request)
        .timeout(Duration.ofSeconds(2))
        .onErrorResume(e -> 
            Mono.firstWithValue(
                secondaryService.call(request),
                Mono.just(Response.fallback())
            )
        );
}

// 2. amb (deprecated, use firstWithSignal)
Flux.firstWithSignal(source1, source2, source3);
```

---

## **6. Error Handling Operators**

### **Error Recovery**
```java
// 1. onErrorReturn - Provide fallback value
Flux.just(1, 2, 3)
    .map(n -> {
        if (n == 2) throw new RuntimeException("Boom!");
        return n;
    })
    .onErrorReturn(0); // 1, 0

// 2. onErrorResume - Switch to fallback publisher
Flux.error(new RuntimeException("Failed"))
    .onErrorResume(e -> 
        Flux.just("fallback1", "fallback2")
    );

// 3. onErrorContinue - Continue with next element
Flux.just(1, 2, 3, 4)
    .map(n -> {
        if (n == 3) throw new RuntimeException("Skip 3");
        return n * 10;
    })
    .onErrorContinue((e, obj) -> 
        log.error("Skipped {} due to {}", obj, e.getMessage())
    ); // 10, 20, 40

// REAL-WORLD: Resilient API calls
public Flux<Product> getProductsWithRetry(List<String> ids) {
    return Flux.fromIterable(ids)
        .flatMap(id -> 
            productService.getProduct(id)
                .onErrorResume(e -> {
                    log.warn("Failed for {}, trying fallback", id);
                    return fallbackService.getProduct(id);
                })
                .onErrorResume(e -> {
                    log.error("Complete failure for {}", id);
                    return Mono.just(Product.unavailable(id));
                })
        );
}
```

### **Retry Strategies**
```java
// 1. retry() - Simple retry
Flux.error(new RuntimeException())
    .retry(3); // Retry 3 times

// 2. retryWhen - Advanced retry with backoff
Flux.error(new RuntimeException())
    .retryWhen(Retry.backoff(3, Duration.ofSeconds(1))
        .maxBackoff(Duration.ofSeconds(10))
        .jitter(0.5) // Add randomness
        .doBeforeRetry(retrySignal -> 
            log.warn("Retrying attempt {}", retrySignal.totalRetries())
        )
        .onRetryExhaustedThrow((spec, signal) -> 
            new ServiceUnavailableException("Max retries exceeded")
        )
    );

// 3. retryWhen with predicate
.retryWhen(Retry.fixedDelay(5, Duration.ofSeconds(1))
    .filter(e -> e instanceof TimeoutException)
    .doBeforeRetry(retrySignal -> {
        if (retrySignal.totalRetries() > 2) {
            refreshConnection();
        }
    })
);

// REAL-WORLD: Exponential backoff for external APIs
private RetrySpec apiRetrySpec() {
    return Retry.backoff(5, Duration.ofSeconds(1))
        .maxBackoff(Duration.ofMinutes(1))
        .transientErrors(true) // Retry on 5xx errors
        .doBeforeRetry(retry -> 
            metrics.recordRetry(retry.failure())
        )
        .onRetryExhaustedThrow((spec, signal) -> 
            new ApiException("Service unavailable after retries")
        );
}

public Mono<Response> callExternalApi(Request request) {
    return webClient.post()
        .bodyValue(request)
        .retrieve()
        .bodyToMono(Response.class)
        .retryWhen(apiRetrySpec());
}
```

### **Timeout & Fallback**
```java
// 1. timeout - Timeout with fallback
Flux.interval(Duration.ofSeconds(2))
    .timeout(Duration.ofSeconds(1))
    .onErrorResume(TimeoutException.class, e -> 
        Flux.just(-1L)
    );

// 2. timeout with fallback publisher
slowFlux
    .timeout(Duration.ofSeconds(1), fallbackFlux)
    .subscribe();

// REAL-WORLD: Service call with timeout cascade
public Mono<Response> callServiceWithTimeouts(Request req) {
    return primaryService.call(req)
        .timeout(Duration.ofMillis(100))
        .onErrorResume(TimeoutException.class, e ->
            secondaryService.call(req)
                .timeout(Duration.ofMillis(200))
        )
        .onErrorResume(TimeoutException.class, e ->
            tertiaryService.call(req)
                .timeout(Duration.ofMillis(500))
        )
        .onErrorReturn(Response.fallback());
}
```

### **Error Classification**
```java
// 1. onErrorMap - Transform error
Flux.error(new IOException("Connection failed"))
    .onErrorMap(IOException.class, e -> 
        new ServiceException("Service unavailable", e)
    );

// 2. onErrorMap with predicate
.onErrorMap(e -> e.getMessage().contains("timeout"), 
    e -> new TimeoutException("Operation timed out")
);

// 3. doOnError - Side effect on error
Flux.error(new RuntimeException())
    .doOnError(e -> 
        metrics.incrementError(e.getClass().getSimpleName())
    )
    .onErrorResume(e -> Mono.empty());

// REAL-WORLD: Circuit breaker pattern
@Component
public class CircuitBreakerService {
    private final CircuitBreaker circuitBreaker;
    
    public Flux<Data> getDataWithCircuitBreaker() {
        return Flux.defer(() -> externalService.getData())
            .transformDeferred(circuitBreaker::run)
            .onErrorResume(CallNotPermittedException.class, e -> 
                Flux.error(new ServiceUnavailableException())
            );
    }
}
```

---

## **7. Utility Operators**

### **Side Effects**
```java
// 1. doOnNext - Peek at values
Flux.range(1, 5)
    .doOnNext(n -> System.out.println("Processing: " + n))
    .map(n -> n * 2)
    .subscribe();

// 2. doOnComplete - On completion
Flux.just(1, 2, 3)
    .doOnComplete(() -> System.out.println("Done!"))
    .subscribe();

// 3. doOnError - On error
Flux.error(new RuntimeException())
    .doOnError(e -> log.error("Failed", e))
    .subscribe();

// 4. doOnSubscribe / doOnCancel / doOnRequest
Flux.range(1, 10)
    .doOnSubscribe(s -> System.out.println("Subscribed"))
    .doOnRequest(n -> System.out.println("Requested: " + n))
    .doOnCancel(() -> System.out.println("Cancelled"))
    .subscribe();

// REAL-WORLD: Request logging
@RestController
public class ApiController {
    @GetMapping("/data")
    public Flux<Data> getData() {
        return dataService.getData()
            .doOnSubscribe(s -> 
                log.info("Request started at {}", Instant.now())
            )
            .doOnComplete(() -> 
                log.info("Request completed successfully")
            )
            .doOnError(e -> 
                log.error("Request failed", e)
            );
    }
}
```

### **Debugging & Metrics**
```java
// 1. log() - Log all signals
Flux.range(1, 3)
    .log("my.flux")
    .subscribe();

// Output:
// [my.flux] onSubscribe(FluxRange.RangeSubscription)
// [my.flux] request(unbounded)
// [my.flux] onNext(1)
// [my.flux] onNext(2)
// [my.flux] onNext(3)
// [my.flux] onComplete()

// 2. checkpoint - Add trace for debugging
Flux.error(new RuntimeException("Oops"))
    .checkpoint("errorSource")
    .subscribe();

// Stack trace includes "errorSource" marker

// 3. metrics - Micrometer integration
Flux.range(1, 100)
    .name("myFlux")
    .metrics()
    .subscribe();

// REAL-WORLD: Production monitoring
public Flux<Event> processEvents(Flux<Event> events) {
    return events
        .name("event.processor")
        .tag("type", "kafka")
        .metrics() // Exposes metrics to Micrometer
        .doOnNext(event -> 
            Metrics.counter("events.processed").increment()
        )
        .doOnError(e -> 
            Metrics.counter("events.failed").increment()
        );
}
```

### **Conditional Operators**
```java
// 1. hasElements / hasElement
Mono<Boolean> hasData = Flux.just(1, 2, 3).hasElements();
Mono<Boolean> hasTwo = Flux.just(1, 2, 3).hasElement(2);

// 2. any / all
Mono<Boolean> anyEven = Flux.just(1, 3, 5).any(n -> n % 2 == 0);
Mono<Boolean> allPositive = Flux.just(1, 2, 3).all(n -> n > 0);

// 3. defaultIfEmpty
Flux.empty()
    .defaultIfEmpty("default"); // Emits "default"

// 4. switchIfEmpty
Flux.empty()
    .switchIfEmpty(Flux.just("fallback", "values"));

// REAL-WORLD: Cache with fallback
public Mono<User> getUser(String id) {
    return cache.get(id)
        .switchIfEmpty(
            database.getUser(id)
                .doOnNext(user -> cache.set(id, user))
        )
        .defaultIfEmpty(User.anonymous());
}
```

### **Collecting & Reducing**
```java
// 1. collectList / collectMap / collectMultimap
Flux.just("a", "b", "c")
    .collectList() // Mono<List<String>>
    .subscribe();

Flux.just(
    new Pair("key1", "value1"),
    new Pair("key2", "value2")
).collectMap(Pair::getKey, Pair::getValue);

// 2. reduce - Accumulate values
Flux.range(1, 5)
    .reduce(0, Integer::sum); // Mono with value 15

// 3. scan - Emit accumulation steps
Flux.range(1, 5)
    .scan(0, Integer::sum); // 0,1,3,6,10,15

// 4. count
Flux.just("a", "b", "c").count(); // Mono with value 3

// REAL-WORLD: Real-time analytics
public Flux<AggregatedStats> streamStats(Flux<Event> events) {
    return events
        .window(Duration.ofSeconds(5))
        .flatMap(window -> 
            window
                .reduce(new StatsAccumulator(), StatsAccumulator::add)
                .map(StatsAccumulator::toStats)
        );
}
```

---

## **8. Schedulers & Threading**

### **Scheduler Types**
```java
// 1. Schedulers.immediate() - Current thread
// 2. Schedulers.single() - Single dedicated thread
// 3. Schedulers.elastic() - Deprecated, use boundedElastic
// 4. Schedulers.boundedElastic() - For blocking I/O
// 5. Schedulers.parallel() - For CPU-intensive work
// 6. Schedulers.fromExecutorService() - Custom executor

// REAL-WORLD: Threading strategy by operation type
public Mono<Result> process(Request request) {
    return validate(request) // Fast, use immediate
        .flatMap(validated -> 
            Mono.fromCallable(() -> blockingIO(validated))
                .subscribeOn(Schedulers.boundedElastic())
        )
        .flatMap(ioResult -> 
            Mono.fromCallable(() -> cpuIntensive(ioResult))
                .subscribeOn(Schedulers.parallel())
        )
        .publishOn(Schedulers.single()) // Switch back to single thread
        .doOnNext(result -> 
            // UI updates or single-threaded operations
            updateUI(result)
        );
}
```

### **Threading Operators**
```java
// 1. subscribeOn - Where subscription happens
Flux.range(1, 10)
    .map(n -> {
        System.out.println("Map on: " + Thread.currentThread().getName());
        return n * 2;
    })
    .subscribeOn(Schedulers.parallel())
    .subscribe();

// 2. publishOn - Switch threads downstream
Flux.range(1, 10)
    .map(n -> n * 2) // On subscription thread
    .publishOn(Schedulers.single())
    .map(n -> n + 1) // On single thread
    .subscribe();

// 3. parallel / runOn - Parallel processing
Flux.range(1, 100)
    .parallel(4) // Split into 4 rails
    .runOn(Schedulers.parallel())
    .map(n -> n * 2)
    .sequential(); // Merge back to single Flux

// REAL-WORLD: Image processing pipeline
public Flux<ProcessedImage> processImages(Flux<Image> images) {
    return images
        .parallel(4) // 4 parallel lanes
        .runOn(Schedulers.parallel()) // Each lane on parallel scheduler
        .map(this::detectEdges) // CPU-intensive
        .sequential() // Back to single flux
        .publishOn(Schedulers.boundedElastic())
        .flatMap(this::saveToStorage) // I/O intensive
        .publishOn(Schedulers.single())
        .doOnNext(this::notifyUI);
}
```

### **Context Propagation**
```java
// 1. contextWrite - Write to Context
Flux.just("Hello")
    .flatMap(word -> 
        Mono.deferContextual(ctx -> 
            Mono.just(word + " " + ctx.get("user"))
        )
    )
    .contextWrite(Context.of("user", "Alice"))
    .subscribe(System.out::println); // Hello Alice

// 2. transformDeferredContextual
Flux.range(1, 5)
    .transformDeferredContextual((flux, ctx) -> 
        flux.map(n -> n + ctx.getOrDefault("offset", 0))
    )
    .contextWrite(Context.of("offset", 100))
    .subscribe(); // 101, 102, 103, 104, 105

// REAL-WORLD: Request tracing
@Component
public class TracingFilter implements WebFilter {
    @Override
    public Mono<Void> filter(ServerWebExchange exchange, WebFilterChain chain) {
        return chain.filter(exchange)
            .contextWrite(Context.of(
                "traceId", exchange.getRequest().getId(),
                "userId", extractUserId(exchange)
            ));
    }
}

@RestController
public class ApiController {
    @GetMapping("/data")
    public Mono<Data> getData() {
        return Mono.deferContextual(ctx -> {
            String traceId = ctx.get("traceId");
            return dataService.getData()
                .doOnSubscribe(s -> 
                    log.info("Trace {}: Started", traceId)
                );
        });
    }
}
```

---

## **9. Backpressure Operators**

### **Backpressure Strategies**
```java
// 1. onBackpressureBuffer - Buffer with limits
Flux.range(1, 1000)
    .onBackpressureBuffer(
        100,                      // Buffer size
        BufferOverflowStrategy.DROP_OLDEST, // Strategy
        buffer -> log.warn("Buffer overflow: {}", buffer.size())
    )
    .subscribe();

// 2. onBackpressureDrop - Drop excess
fastProducer
    .onBackpressureDrop(dropped -> 
        metrics.recordDropped(dropped)
    )
    .subscribe(slowConsumer);

// 3. onBackpressureLatest - Keep only latest
realTimeData
    .onBackpressureLatest()
    .subscribe(uiUpdater);

// 4. onBackpressureError - Error on overflow
Flux.interval(Duration.ofMillis(10))
    .onBackpressureError()
    .subscribe(
        System.out::println,
        e -> log.error("Backpressure overflow", e)
    );

// REAL-WORLD: Adaptive backpressure
public Flux<Data> adaptBackpressure(Flux<Data> source, Consumer<Data> processor) {
    return source
        .onBackpressureBuffer(
            1000,
            BufferOverflowStrategy.DROP_OLDEST,
            buffer -> {
                if (buffer.size() > 800) {
                    // Slow down producer
                    adjustProducerRate(0.5);
                }
            }
        )
        .doOnNext(processor)
        .doOnRequest(n -> {
            // Increase rate when buffer is low
            if (getBufferSize() < 200) {
                adjustProducerRate(1.5);
            }
        });
}
```

### **Request Management**
```java
// 1. limitRequest - Limit total request
Flux.range(1, 100)
    .limitRequest(10) // Only request 10 elements
    .subscribe();

// 2. limitRate - Control request batches
Flux.range(1, 1000)
    .limitRate(100, 50) // Request 100, replenish at 50
    .subscribe();

// 3. subscribe with request
Flux.range(1, 100)
    .subscribe(
        new BaseSubscriber<Integer>() {
            @Override
            protected void hookOnSubscribe(Subscription subscription) {
                request(10); // Request first 10
            }
            
            @Override
            protected void hookOnNext(Integer value) {
                process(value);
                if (value % 10 == 0) {
                    request(10); // Request more in batches
                }
            }
        }
    );

// REAL-WORLD: File streaming with backpressure
public Flux<ByteBuffer> streamFile(String path, int chunkSize) {
    return Flux.using(
        () -> Files.newInputStream(Paths.get(path)),
        in -> {
            byte[] buffer = new byte[chunkSize];
            return Flux.generate(sink -> {
                try {
                    int read = in.read(buffer);
                    if (read == -1) {
                        sink.complete();
                    } else if (read < buffer.length) {
                        sink.next(ByteBuffer.wrap(buffer, 0, read));
                    } else {
                        sink.next(ByteBuffer.wrap(buffer));
                    }
                } catch (IOException e) {
                    sink.error(e);
                }
            });
        },
        in -> {
            try { in.close(); } catch (IOException e) { }
        }
    );
}
```

---

## **10. Real-World Patterns**

### **Pattern 1: API Gateway Aggregation**
```java
@RestController
public class AggregationController {
    
    @GetMapping("/user-dashboard/{userId}")
    public Mono<Dashboard> getUserDashboard(@PathVariable String userId) {
        return Mono.zip(
            userService.getUser(userId).subscribeOn(Schedulers.boundedElastic()),
            orderService.getOrders(userId).collectList().subscribeOn(Schedulers.boundedElastic()),
            paymentService.getPayments(userId).collectList().subscribeOn(Schedulers.boundedElastic()),
            notificationService.getNotifications(userId).collectList().subscribeOn(Schedulers.boundedElastic())
        )
        .map(tuple -> new Dashboard(
            tuple.getT1(),
            tuple.getT2(),
            tuple.getT3(),
            tuple.getT4()
        ))
        .timeout(Duration.ofSeconds(3))
        .onErrorResume(e -> 
            fallbackDashboardService.getDashboard(userId)
        );
    }
    
    @GetMapping("/search")
    public Flux<Product> searchProducts(
        @RequestParam String query,
        @RequestParam(defaultValue = "10") int limit
    ) {
        return Flux.merge(
            productService.search(query).take(limit),
            inventoryService.search(query).take(limit)
        )
        .distinct(Product::getId)
        .sort(Comparator.comparing(Product::getScore).reversed())
        .take(limit);
    }
}
```

### **Pattern 2: Event-Driven Processing**
```java
@Component
public class EventProcessor {
    
    private final Flux<Event> eventStream;
    private final Sinks.Many<Event> sink = Sinks.many().multicast().onBackpressureBuffer(1000);
    
    public EventProcessor() {
        this.eventStream = sink.asFlux()
            .publishOn(Schedulers.parallel())
            .name("event.processor")
            .metrics();
    }
    
    public void processEvent(Event event) {
        sink.tryEmitNext(event);
    }
    
    @PostConstruct
    public void init() {
        eventStream
            .window(Duration.ofSeconds(1), Duration.ofMillis(500))
            .flatMap(window -> 
                window
                    .groupBy(Event::getType)
                    .flatMap(group -> 
                        group
                            .bufferTimeout(100, Duration.ofMillis(500))
                            .flatMap(this::processBatch)
                    )
            )
            .subscribe(
                result -> log.debug("Processed: {}", result),
                error -> log.error("Processing failed", error)
            );
    }
    
    private Mono<ProcessResult> processBatch(List<Event> batch) {
        return Mono.fromCallable(() -> {
                // Batch processing logic
                return new ProcessResult(batch.size());
            })
            .subscribeOn(Schedulers.boundedElastic())
            .timeout(Duration.ofSeconds(5))
            .retryWhen(Retry.backoff(3, Duration.ofSeconds(1)));
    }
}
```

### **Pattern 3: Reactive Caching**
```java
@Component
public class ReactiveCache<K, V> {
    
    private final Cache<K, Mono<V>> cache;
    private final Function<K, Mono<V>> loader;
    
    public ReactiveCache(Function<K, Mono<V>> loader, Duration ttl) {
        this.loader = loader;
        this.cache = Caffeine.newBuilder()
            .expireAfterWrite(ttl)
            .maximumSize(1000)
            .recordStats()
            .build();
    }
    
    public Mono<V> get(K key) {
        return Mono.defer(() -> {
            Mono<V> cached = cache.getIfPresent(key);
            if (cached != null) {
                return cached;
            }
            
            // Use computeIfAbsent to prevent duplicate loads
            return cache.get(key, k -> 
                loader.apply(k)
                    .cache() // Cache the Mono result
                    .doOnError(e -> 
                        cache.invalidate(k)
                    )
            );
        });
    }
    
    // Batch loading
    public Flux<V> getAll(List<K> keys) {
        return Flux.fromIterable(keys)
            .flatMap(this::get, 10) // Concurrent loading
            .collectList()
            .flatMapMany(Flux::fromIterable);
    }
}

// Usage in service
@Service
public class ProductService {
    private final ReactiveCache<String, Product> cache;
    
    public ProductService(ProductRepository repository) {
        this.cache = new ReactiveCache<>(
            repository::findById,
            Duration.ofMinutes(5)
        );
    }
    
    public Mono<Product> getProduct(String id) {
        return cache.get(id)
            .timeout(Duration.ofSeconds(1))
            .onErrorResume(e -> 
                repository.findById(id) // Fallback to direct call
            );
    }
}
```

### **Pattern 4: WebFlux Security Chain**
```java
@Component
public class SecurityChainFilter implements WebFilter {
    
    @Override
    public Mono<Void> filter(ServerWebExchange exchange, WebFilterChain chain) {
        return extractToken(exchange)
            .flatMap(this::validateToken)
            .flatMap(auth -> 
                chain.filter(exchange)
                    .contextWrite(Context.of("auth", auth))
            )
            .switchIfEmpty(
                chain.filter(exchange)
                    .contextWrite(Context.of("auth", Authentication.anonymous()))
            )
            .onErrorResume(AuthenticationException.class, e -> 
                handleUnauthorized(exchange, e)
            );
    }
    
    private Mono<Authentication> validateToken(String token) {
        return Mono.fromCallable(() -> 
                jwtDecoder.decode(token)
            )
            .subscribeOn(Schedulers.boundedElastic())
            .timeout(Duration.ofSeconds(2))
            .retryWhen(Retry.backoff(2, Duration.ofMillis(100)))
            .map(decoded -> new Authentication(decoded.getSubject()))
            .onErrorMap(e -> new AuthenticationException("Invalid token"));
    }
}

@RestController
public class SecureController {
    
    @GetMapping("/secure-data")
    public Mono<SecureData> getSecureData() {
        return Mono.deferContextual(ctx -> {
            Authentication auth = ctx.get("auth");
            if (!auth.hasRole("ADMIN")) {
                return Mono.error(new AccessDeniedException());
            }
            
            return secureService.getData(auth.getUserId())
                .timeout(Duration.ofSeconds(5))
                .retryWhen(Retry.fixedDelay(2, Duration.ofSeconds(1)));
        });
    }
}
```

### **Pattern 5: Database Reactive Transactions**
```java
@Service
@Transactional
public class OrderService {
    
    private final R2dbcEntityTemplate template;
    private final ReactiveTransactionManager txManager;
    
    public Mono<Order> createOrder(OrderRequest request) {
        return Mono.usingWhen(
            TransactionSynchronizationManager.forCurrentTransaction()
                .flatMap(status -> 
                    Mono.from(txManager.getReactiveTransaction(txManager))
                ),
            transaction -> {
                // Execute in transaction
                return template.insert(Order.class)
                    .using(new Order(request))
                    .flatMap(order -> 
                        Flux.fromIterable(request.getItems())
                            .flatMap(item -> 
                                template.insert(OrderItem.class)
                                    .using(new OrderItem(order.getId(), item))
                            )
                            .then(Mono.just(order))
                    )
                    .flatMap(order -> 
                        inventoryService.reserveItems(request.getItems())
                            .thenReturn(order)
                    )
                    .flatMap(order -> 
                        paymentService.processPayment(request.getPayment())
                            .thenReturn(order)
                    );
            },
            transaction -> txManager.commit(transaction),
            (transaction, err) -> txManager.rollback(transaction),
            transaction -> txManager.rollback(transaction)
        );
    }
    
    // Optimistic locking with retry
    public Mono<Order> updateOrder(String orderId, OrderUpdate update) {
        return Mono.defer(() -> 
            template.select(Order.class)
                .from("orders")
                .matching(Query.query(where("id").is(orderId))
                    .with(LockMode.PESSIMISTIC_WRITE))
                .one()
                .flatMap(order -> {
                    order.applyUpdate(update);
                    return template.update(Order.class)
                        .matching(where("id").is(orderId)
                            .and("version").is(order.getVersion()))
                        .apply(Update.update("status", order.getStatus())
                            .set("version", order.getVersion() + 1))
                        .flatMap(updated -> {
                            if (updated == 0) {
                                return Mono.error(new OptimisticLockingFailureException());
                            }
                            return Mono.just(order);
                        });
                })
        )
        .retryWhen(Retry.backoff(3, Duration.ofMillis(100))
            .filter(OptimisticLockingFailureException.class::isInstance));
    }
}
```

---

## **Summary Cheat Sheet**

| **Category** | **Operator** | **Use Case** | **When to Use** |
|-------------|-------------|-------------|-----------------|
| **Creation** | `just`, `from`, `defer` | Initial data | Startup, bridging |
| **Transformation** | `map`, `flatMap`, `concatMap` | Data transformation | 1:1, 1:N, ordered async |
| **Filtering** | `filter`, `take`, `skip` | Data selection | Conditional logic |
| **Combining** | `merge`, `concat`, `zip` | Multiple sources | Aggregation, parallelism |
| **Error Handling** | `onErrorResume`, `retryWhen` | Resilience | Fallbacks, retries |
| **Utility** | `doOnNext`, `log`, `metrics` | Observability | Debugging, monitoring |
| **Threading** | `subscribeOn`, `publishOn` | Concurrency control | I/O vs CPU work |
| **Backpressure** | `onBackpressureBuffer`, `limitRate` | Flow control | Fast producer, slow consumer |

### **Golden Rules:**
1. **Use `flatMap` for async, `concatMap` for ordered async**
2. **Prefer `retryWhen` over simple `retry` for production**
3. **Use `defer` for lazy/conditional publisher creation**
4. **Apply `subscribeOn` early, `publishOn` at switching points**
5. **Always handle errors in reactive chains**
6. **Use backpressure strategies for unbounded streams**
7. **Monitor with metrics and structured logging**
8. **Test with `StepVerifier` for reliable tests**
