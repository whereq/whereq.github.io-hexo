---
title: 'WebFlux Deep Dive: Operators, Types, and Real-World Use Cases'
date: 2025-11-21 16:30:04
categories:
- WebFlux
tags:
- WebFlux
---

1. [Core Reactive Types](#core-reactive-types)
2. [Creation Operators](#creation-operators)
3. [Transformation Operators](#transformation-operators)
4. [Combination Operators](#combination-operators)
5. [Error Handling Operators](#error-handling-operators)
6. [Scheduling & Threading](#scheduling--threading)
7. [Advanced Patterns](#advanced-patterns)

## Core Reactive Types

### Mono - Single Result Container
```java
/**
 * Mono represents a stream that emits 0 or 1 element
 * Perfect for: HTTP responses, database saves, single calculations
 */
public class MonoDeepDive {
    
    // Creation examples
    public void monoCreation() {
        // Empty Mono
        Mono<String> empty = Mono.empty();
        
        // From value
        Mono<String> staticValue = Mono.just("Hello World");
        
        // From nullable
        String possiblyNull = getFromCache();
        Mono<String> nullable = Mono.justOrEmpty(possiblyNull);
        
        // From supplier (lazy)
        Mono<String> lazy = Mono.fromSupplier(() -> expensiveOperation());
        
        // From callable
        Mono<String> fromCallable = Mono.fromCallable(() -> {
            return database.getUser(userId); // Potentially blocking
        });
        
        // From future
        CompletableFuture<String> future = asyncHttpCall();
        Mono<String> fromFuture = Mono.fromFuture(future);
    }
    
    // Real-world use case: User profile lookup
    public Mono<UserProfile> getUserProfile(String userId) {
        return Mono.fromCallable(() -> userRepository.findById(userId))
                  .filter(Optional::isPresent)
                  .map(Optional::get)
                  .switchIfEmpty(Mono.error(new UserNotFoundException(userId)));
    }
}
```

### Flux - Multiple Results Stream
```java
/**
 * Flux represents a stream that emits 0 to N elements
 * Perfect for: Collections, real-time streams, paginated results
 */
public class FluxDeepDive {
    
    // Creation examples
    public void fluxCreation() {
        // From values
        Flux<String> fromValues = Flux.just("A", "B", "C");
        
        // From array
        Flux<String> fromArray = Flux.fromArray(new String[]{"A", "B", "C"});
        
        // From iterable
        Flux<String> fromList = Flux.fromIterable(Arrays.asList("A", "B", "C"));
        
        // Range
        Flux<Integer> numbers = Flux.range(1, 10);
        
        // Interval (real-time stream)
        Flux<Long> ticker = Flux.interval(Duration.ofSeconds(1));
        
        // Generate (stateful stream)
        Flux<Integer> fibonacci = Flux.generate(
            () -> new int[]{0, 1}, // initial state
            (state, sink) -> {
                sink.next(state[0]);
                int next = state[0] + state[1];
                state[0] = state[1];
                state[1] = next;
                return state;
            }
        );
    }
    
    // Real-world use case: Real-time stock prices
    public Flux<StockPrice> getStockPrices(String symbol) {
        return Flux.interval(Duration.ofMillis(100))
                  .flatMap(tick -> 
                      Mono.fromCallable(() -> stockService.getCurrentPrice(symbol))
                          .subscribeOn(Schedulers.boundedElastic())
                  )
                  .take(100); // Only take 100 price updates
    }
}
```

## Transformation Operators

### map vs flatMap - The Critical Difference
```java
public class TransformationOperators {
    
    /**
     * map: Synchronous 1:1 transformation
     * Use when: Transformation is fast, non-blocking, CPU-bound
     */
    public void mapOperator() {
        Flux<String> names = Flux.just("john", "jane", "bob");
        
        // Synchronous transformation - stays in same thread
        Flux<String> capitalized = names.map(name -> name.toUpperCase());
        // Output: JOHN, JANE, BOB
        
        // Real-world: Data enrichment (fast)
        Flux<User> users = getUserIds()
            .map(id -> new User(id, "user-" + id));
    }
    
    /**
     * flatMap: Asynchronous 1:N transformation
     * Use when: Need to call async APIs, database, external services
     */
    public void flatMapOperator() {
        Flux<String> userIds = Flux.just("user1", "user2", "user3");
        
        // Asynchronous transformation - can change threads
        Flux<User> users = userIds.flatMap(id -> 
            userRepository.findByIdAsync(id) // Returns Mono<User>
        );
        
        // Real-world: Fetch user details from multiple services
        Flux<Order> orders = getOrderIds()
            .flatMap(orderId -> 
                orderService.getOrderAsync(orderId)
                    .onErrorResume(e -> Mono.empty()) // Skip failed orders
            );
    }
    
    /**
     * flatMapMany: Convert Mono to Flux
     * Use when: You have Mono that emits multiple items
     */
    public void flatMapManyOperator() {
        Mono<String> csvData = readCsvFile();
        
        // Convert Mono<String> to Flux<String[]>
        Flux<String[]> rows = csvData.flatMapMany(content -> 
            Flux.fromArray(content.split("\n"))
                .map(line -> line.split(","))
        );
        
        // Real-world: Get user and their recent activities
        Mono<User> user = getUserProfile(userId);
        Flux<Activity> activities = user.flatMapMany(u -> 
            activityService.getRecentActivities(u.getId())
        );
    }
}
```

### Advanced Transformation Operators
```java
public class AdvancedTransformations {
    
    /**
     * concatMap vs flatMap - Ordering guarantee
     */
    public void orderingGuarantees() {
        Flux<String> ids = Flux.just("1", "2", "3");
        
        // flatMap - no ordering guarantee (faster)
        Flux<String> unordered = ids.flatMap(id -> 
            asyncOperation(id).delayElement(Duration.ofMillis(randomDelay()))
        );
        
        // concatMap - preserves order (slower but ordered)
        Flux<String> ordered = ids.concatMap(id -> 
            asyncOperation(id).delayElement(Duration.ofMillis(randomDelay()))
        );
        
        // Real-world: Processing messages with ordering requirement
        Flux<Message> kafkaMessages = kafkaReceiver.receive();
        kafkaMessages
            .concatMap(msg -> processMessage(msg)) // Maintain message order
            .subscribe();
    }
    
    /**
     * switchMap - Cancel previous emissions
     */
    public void switchMapOperator() {
        // Real-world: Search-as-you-type
        Flux<String> searchTerms = keyPressEvents();
        
        Flux<SearchResults> results = searchTerms
            .debounce(Duration.ofMillis(300)) // Wait for typing to stop
            .switchMap(term -> 
                searchService.search(term) // Cancel previous search if new term arrives
                    .timeout(Duration.ofSeconds(5))
            );
    }
    
    /**
     * groupBy - Group elements by key
     */
    public void groupByOperator() {
        // Real-world: Group orders by customer
        Flux<Order> orders = orderStream();
        
        Flux<GroupedFlux<String, Order>> byCustomer = orders
            .groupBy(Order::getCustomerId);
        
        // Process each customer's orders separately
        byCustomer.flatMap(group -> 
            group
                .window(Duration.ofMinutes(5)) // Window by time
                .flatMap(window -> calculateCustomerStats(window))
        ).subscribe();
    }
}
```

## Combination Operators

```java
public class CombinationOperators {
    
    /**
     * zip - Combine multiple publishers
     */
    public void zipOperator() {
        Mono<User> user = getUser(userId);
        Mono<Profile> profile = getProfile(userId);
        Mono<Preferences> prefs = getPreferences(userId);
        
        // Wait for all to complete, then combine
        Mono<UserDashboard> dashboard = Mono.zip(user, profile, prefs)
            .map(tuple -> {
                User u = tuple.getT1();
                Profile p = tuple.getT2();
                Preferences pref = tuple.getT3();
                return new UserDashboard(u, p, pref);
            });
        
        // Real-world: Aggregate data from multiple microservices
        Mono<OrderDetails> orderDetails = Mono.zip(
            orderService.getOrder(orderId),
            paymentService.getPayment(orderId),
            shippingService.getTracking(orderId)
        ).map(t -> OrderDetails.aggregate(t.getT1(), t.getT2(), t.getT3()));
    }
    
    /**
     * merge vs concat - Parallel vs sequential combination
     */
    public void mergeVsConcat() {
        Flux<String> stream1 = Flux.just("A", "B", "C").delayElements(Duration.ofMillis(100));
        Flux<String> stream2 = Flux.just("1", "2", "3").delayElements(Duration.ofMillis(50));
        
        // merge - interleaves as they arrive (faster)
        Flux<String> merged = Flux.merge(stream1, stream2);
        // Possible output: 1, A, 2, B, 3, C
        
        // concat - waits for first to complete (ordered)
        Flux<String> concatenated = Flux.concat(stream1, stream2);
        // Guaranteed output: A, B, C, 1, 2, 3
    }
    
    /**
     * combineLatest - Combine latest from multiple streams
     */
    public void combineLatest() {
        // Real-world: Real-time dashboard with multiple data sources
        Flux<Double> stockPrice = stockPriceStream();
        Flux<Double> exchangeRate = exchangeRateStream();
        Flux<News> newsFeed = newsStream();
        
        Flux<Dashboard> dashboard = Flux.combineLatest(
            Arrays::asList,
            stockPrice, exchangeRate, newsFeed
        ).map(data -> {
            Double price = (Double) data[0];
            Double rate = (Double) data[1];
            News news = (News) data[2];
            return new Dashboard(price, rate, news);
        });
    }
}
```

## Error Handling Operators

```java
public class ErrorHandlingOperators {
    
    /**
     * Comprehensive error handling strategies
     */
    public void errorHandling() {
        Flux<String> dataStream = getDataStream();
        
        dataStream
            // 1. Timeout handling
            .timeout(Duration.ofSeconds(30))
            
            // 2. Retry with backoff
            .retryWhen(Retry.backoff(3, Duration.ofSeconds(1))
            
            // 3. Fallback values
            .onErrorReturn("fallback-value")
            
            // 4. Fallback method
            .onErrorResume(error -> {
                if (error instanceof TimeoutException) {
                    return getCachedData();
                } else if (error instanceof DatabaseException) {
                    return getBackupData();
                }
                return Flux.error(error);
            })
            
            // 5. Log and continue
            .doOnError(error -> 
                log.error("Stream processing failed", error)
            )
            
            .subscribe();
    }
    
    /**
     * Circuit breaker pattern
     */
    public void circuitBreaker() {
        CircuitBreaker circuitBreaker = CircuitBreaker.ofDefaults("backendService");
        
        Flux<String> resilientStream = Flux.interval(Duration.ofSeconds(1))
            .flatMap(tick -> 
                Mono.fromCallable(() -> externalService.call())
                    .transformDeferred(CircuitBreakerOperator.of(circuitBreaker))
                    .onErrorResume(e -> Mono.just("fallback"))
            );
    }
}
```

## Scheduling & Threading

```java
public class SchedulingDeepDive {
    
    /**
     * Proper threading model for different operations
     */
    public void schedulingOperators() {
        // CPU-intensive work
        Flux<Integer> cpuIntensive = Flux.range(1, 1000)
            .parallel() // Process in parallel
            .runOn(Schedulers.parallel())
            .map(i -> expensiveComputation(i))
            .sequential();
        
        // I/O intensive work
        Flux<String> ioIntensive = Flux.just("id1", "id2", "id3")
            .flatMap(id -> 
                Mono.fromCallable(() -> database.query(id))
                    .subscribeOn(Schedulers.boundedElastic()) // Offload blocking I/O
            );
        
        // Real-time processing with different schedulers
        Flux.interval(Duration.ofMillis(10))
            .publishOn(Schedulers.single()) // Dedicated thread for downstream
            .map(tick -> transformData(tick))
            .publishOn(Schedulers.boundedElastic()) // Switch threads for I/O
            .flatMap(data -> saveToDatabase(data))
            .subscribeOn(Schedulers.parallel()) // Where subscription happens
            .subscribe();
    }
    
    /**
     * Scheduler types and when to use them
     */
    public void schedulerTypes() {
        // Schedulers.immediate() - Current thread
        // Schedulers.single() - Single dedicated thread
        // Schedulers.parallel() - Fixed pool for CPU work
        // Schedulers.boundedElastic() - For blocking I/O (preferred over elastic)
        // Schedulers.elastic() - Legacy, unlimited threads (avoid)
    }
}
```

## Advanced Patterns & Real-World Use Cases

### Pattern 1: API Gateway with Rate Limiting
```java
public class ApiGatewayService {
    
    public Flux<ApiResponse> handleConcurrentRequests(Flux<ApiRequest> requests) {
        return requests
            .window(Duration.ofSeconds(1)) // Window by time
            .flatMap(window -> 
                window
                    .limitRate(100) // Limit to 100 requests per second
                    .flatMap(this::processRequest, 10) // Max 10 concurrent
            )
            .onErrorResume(error -> 
                Flux.just(ApiResponse.error("Service unavailable"))
            );
    }
    
    private Mono<ApiResponse> processRequest(ApiRequest request) {
        return Mono.zip(
            userService.validateToken(request.getToken()),
            rateLimitService.checkLimit(request.getUserId())
        )
        .flatMap(tuple -> routingService.routeRequest(request))
        .timeout(Duration.ofSeconds(5))
        .retryWhen(Retry.fixedDelay(2, Duration.ofMillis(100)));
    }
}
```

### Pattern 2: Real-Time Data Pipeline
```java
public class DataPipeline {
    
    public Flux<ProcessedData> createPipeline() {
        return kafkaSource.stream()
            .groupBy(message -> message.getPartitionKey()) // Maintain ordering per key
            .flatMap(partitionStream -> 
                partitionStream
                    .window(Duration.ofSeconds(5)) // Batch processing
                    .flatMap(this::processBatch)
                    .flatMap(this::enrichWithExternalData)
                    .flatMap(this::validateAndTransform)
                    .buffer(100) // Buffer for bulk insert
                    .flatMap(this::bulkInsertToDatabase)
            , 32) // Process 32 partitions concurrently
            .metrics() // Monitor performance
            .doOnNext(metrics::recordProcessingTime);
    }
    
    private Mono<ProcessedData> processBatch(Flux<Message> batch) {
        return batch
            .collectList()
            .flatMap(list -> 
                Mono.fromCallable(() -> sparkService.processBatch(list))
                    .subscribeOn(Schedulers.boundedElastic())
            );
    }
}
```

### Pattern 3: Resilient Service Composition
```java
public class ResilientServiceComposition {
    
    public Mono<OrderResult> processOrder(Order order) {
        return validateOrder(order)
            .flatMap(validated -> 
                Mono.zip(
                    reserveInventory(validated).onErrorReturn(InventoryReservation.failed()),
                    processPayment(validated).onErrorReturn(PaymentResult.failed()),
                    updateShipping(validated).onErrorReturn(ShippingUpdate.failed())
                )
            )
            .map(tuple -> {
                // Compose final result from partial successes/failures
                return OrderResult.fromComponents(
                    tuple.getT1(), 
                    tuple.getT2(), 
                    tuple.getT3()
                );
            })
            .timeout(Duration.ofSeconds(30))
            .retryWhen(Retry.backoff(3, Duration.ofSeconds(1)));
    }
}
```

## Performance Best Practices

```java
public class PerformanceBestPractices {
    
    // 1. Avoid blocking in reactive chains
    public Flux<String> badExample() {
        return Flux.range(1, 100)
            .map(i -> {
                return blockingDatabaseCall(i); // ❌ BLOCKING!
            });
    }
    
    public Flux<String> goodExample() {
        return Flux.range(1, 100)
            .flatMap(i -> 
                Mono.fromCallable(() -> blockingDatabaseCall(i))
                    .subscribeOn(Schedulers.boundedElastic()) // ✅ Correct
            );
    }
    
    // 2. Use backpressure properly
    public Flux<Data> handleBackpressure() {
        return fastProducer()
            .onBackpressureBuffer(1000) // Buffer up to 1000 elements
            .onBackpressureDrop(dropped -> log.warn("Dropped: {}", dropped))
            .concatMap(this::slowConsumer, 5); // Control concurrency
    }
    
    // 3. Monitor and tune
    public void monitoring() {
        Flux.range(1, 1000)
            .name("processing.pipeline")
            .tag("type", "batch")
            .metrics()
            .doOnNext(item -> 
                Metrics.counter("items.processed").increment()
            )
            .subscribe();
    }
}
```
