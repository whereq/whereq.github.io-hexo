---
title: Kafka Producer Tuning Guide
date: 2024-10-23 16:01:38
categories:
- Kafka
tags:
- Kafka
---

- [Kafka Producer Tuning Guide](#kafka-producer-tuning-guide)
    - [1. Introduction](#1-introduction)
    - [2. Optimizing Producer Throughput](#2-optimizing-producer-throughput)
      - [2.1 Batching](#21-batching)
      - [2.2 Compression](#22-compression)
      - [2.3 Asynchronous Send](#23-asynchronous-send)
      - [2.4 Producer Configuration for High Throughput](#24-producer-configuration-for-high-throughput)
    - [3. Ensuring Data Reliability](#3-ensuring-data-reliability)
      - [3.1 Acknowledgment (acks)](#31-acknowledgment-acks)
      - [3.2 Retries and Idempotence](#32-retries-and-idempotence)
      - [3.3 Durability with Replication](#33-durability-with-replication)
    - [4. Guaranteeing Exactly Once Semantics](#4-guaranteeing-exactly-once-semantics)
      - [4.1 Idempotent Producer](#41-idempotent-producer)
      - [4.2 Transactions](#42-transactions)
    - [5. Maintaining Data Order](#5-maintaining-data-order)
      - [5.1 Partitioning](#51-partitioning)
      - [5.2 Producer Configuration for Ordering](#52-producer-configuration-for-ordering)
    - [6. Conclusion](#6-conclusion)
    - [Diagram: Kafka Producer Configuration for Throughput and Reliability](#diagram-kafka-producer-configuration-for-throughput-and-reliability)
    - [Configuration Parameters](#configuration-parameters)
      - [`max.in.flight.requests.per.connection`](#maxinflightrequestsperconnection)
        - [How it Works:](#how-it-works)
        - [Default Value:](#default-value)
        - [Impact on Performance and Ordering:](#impact-on-performance-and-ordering)
          - [1. **Throughput:**](#1-throughput)
          - [2. **Message Ordering:**](#2-message-ordering)
          - [3. **Reliability:**](#3-reliability)
        - [Recommended Settings:](#recommended-settings)
        - [Summary of Key Points:](#summary-of-key-points)
        - [Diagram: Effect of `max.in.flight.requests.per.connection` on Kafka Producer Behavior](#diagram-effect-of-maxinflightrequestsperconnection-on-kafka-producer-behavior)

---

# Kafka Producer Tuning Guide

---

### 1. Introduction
<a name="1-introduction"></a>

Apache Kafka is a distributed streaming platform often used for building real-time data pipelines and streaming applications. Kafka producers are responsible for sending records to Kafka brokers, and tuning their performance is essential to achieving high throughput, reliability, and proper data ordering. In this article, we will cover the various aspects of Kafka producer tuning, including optimizing throughput, ensuring reliability, avoiding data duplication, and maintaining data order.

---

### 2. Optimizing Producer Throughput
<a name="2-optimizing-producer-throughput"></a>

Throughput optimization focuses on increasing the number of messages the producer can send per second. Kafka producers offer multiple configuration options to improve throughput:

#### 2.1 Batching
<a name="21-batching"></a>

Kafka producers can batch multiple records before sending them to the broker, reducing the number of network requests.

- **Batch size (`batch.size`)**: Determines the maximum number of bytes in a batch. Increasing this value allows the producer to group more records in one request, increasing throughput.

```java
props.put("batch.size", 16384); // 16KB per batch
```

#### 2.2 Compression
<a name="22-compression"></a>

Enabling compression can reduce the size of data being sent, decreasing network traffic and increasing throughput.

- **Compression type (`compression.type`)**: Supported types are `gzip`, `snappy`, `lz4`, and `zstd`.

```java
props.put("compression.type", "snappy");
```

#### 2.3 Asynchronous Send
<a name="23-asynchronous-send"></a>

Producers can send data asynchronously to minimize the time spent waiting for broker acknowledgments.

- **Linger time (`linger.ms`)**: The producer will wait for up to this time before sending a batch of messages, allowing more records to be added to the batch.

```java
props.put("linger.ms", 10); // Wait 10 ms before sending a batch
```

#### 2.4 Producer Configuration for High Throughput
<a name="24-producer-configuration-for-high-throughput"></a>

Below is a configuration snippet optimized for throughput:

```java
Properties props = new Properties();
props.put("bootstrap.servers", "localhost:9092");
props.put("acks", "1"); // Leader acknowledgment for faster send
props.put("compression.type", "lz4");
props.put("batch.size", 32768); // 32KB batch size
props.put("linger.ms", 5); // Wait for 5ms before sending
props.put("max.in.flight.requests.per.connection", "5"); // Max concurrent requests
```

---

### 3. Ensuring Data Reliability
<a name="3-ensuring-data-reliability"></a>

Reliability involves ensuring that messages are delivered and persisted correctly by the Kafka broker. To achieve this, several configurations control how Kafka handles acknowledgment and retries.

#### 3.1 Acknowledgment (acks)
<a name="31-acknowledgment-acks"></a>

- **`acks` configuration**: Controls how many replicas must acknowledge the record. The safest setting is `acks=all`, ensuring the data is replicated to all in-sync replicas (ISR).

```java
props.put("acks", "all");
```

#### 3.2 Retries and Idempotence
<a name="32-retries-and-idempotence"></a>

- **`retries`**: Controls the number of retry attempts if a request fails.
- **`enable.idempotence`**: Ensures that retries don't result in duplicated messages.

```java
props.put("retries", 3); // Retry 3 times on failure
props.put("enable.idempotence", "true"); // Avoid duplicate messages
```

#### 3.3 Durability with Replication
<a name="33-durability-with-replication"></a>

Kafka ensures durability with data replication across multiple brokers. Setting the **`acks=all`** guarantees that the data is fully replicated before considering the record as successfully sent.

```java
props.put("acks", "all"); // Wait for acknowledgment from all replicas
props.put("min.insync.replicas", 2); // At least 2 replicas must acknowledge
```

---

### 4. Guaranteeing Exactly Once Semantics
<a name="4-guaranteeing-exactly-once-semantics"></a>

Kafka provides **exactly-once semantics (EOS)**, ensuring that data is neither lost nor duplicated. This is critical for financial systems and applications that cannot tolerate duplicates or lost data.

#### 4.1 Idempotent Producer
<a name="41-idempotent-producer"></a>

An idempotent producer ensures that duplicate messages are not written to the Kafka log, even in case of retries. This is achieved by enabling the idempotence feature.

```java
props.put("enable.idempotence", "true");
```

#### 4.2 Transactions
<a name="42-transactions"></a>

Transactions allow multiple operations (e.g., sending messages to different partitions) to be treated as a single unit, ensuring that either all messages are written successfully or none are written.

```java
// Configuring a transactional producer
props.put("transactional.id", "my-transaction-id");

KafkaProducer<String, String> producer = new KafkaProducer<>(props);
producer.initTransactions();
producer.beginTransaction();

try {
    producer.send(new ProducerRecord<>("topic1", "key", "value"));
    producer.send(new ProducerRecord<>("topic2", "key", "value"));
    producer.commitTransaction();
} catch (Exception e) {
    producer.abortTransaction();
}
```

---

### 5. Maintaining Data Order
<a name="5-maintaining-data-order"></a>

By default, Kafka guarantees that records are written in the order they are sent, but only within a partition. You can control data ordering by correctly configuring partitioning strategies.

#### 5.1 Partitioning
<a name="51-partitioning"></a>

Kafka producers can partition records using a **key**. Messages with the same key will always go to the same partition, ensuring their order is maintained.

```java
producer.send(new ProducerRecord<>("topic", "key", "value"));
```

#### 5.2 Producer Configuration for Ordering
<a name="52-producer-configuration-for-ordering"></a>

To maintain message order, especially during retries, the **`max.in.flight.requests.per.connection`** setting is crucial. Setting it to 1 ensures that only one request is sent to the broker at a time.

```java
props.put("max.in.flight.requests.per.connection", 1); // Guarantees ordering
props.put("enable.idempotence", "true"); // Ensures no duplicates
```

---

### 6. Conclusion
<a name="6-conclusion"></a>

Kafka producer tuning involves balancing between throughput, reliability, and data ordering based on the specific requirements of the application. By configuring batching, compression, retries, and idempotence, Kafka producers can achieve high performance while maintaining the necessary level of data reliability and ordering.

---

### Diagram: Kafka Producer Configuration for Throughput and Reliability
<a name="diagram-kafka-producer-configuration-for-throughput-and-reliability"></a>

```plaintext
+----------------------+        +----------------------+        +---------------------+
| Producer Properties  | -----> | Batch, Compression,  | -----> | High Throughput and |
| Configuration        |        | Asynchronous Send    |        | Reliable Delivery   |
+----------------------+        +----------------------+        +---------------------+
                                  |
                                  v
+----------------------+        +---------------------+        +---------------------+
| Idempotence,         | -----> | Exactly-Once        | -----> | Guaranteed Message  |
| Transactions Enabled |        | Semantics           |        | Ordering and No Dups|
+----------------------+        +---------------------+        +---------------------+
```

### Configuration Parameters
<a name="configuration-parameters"></a>

#### `max.in.flight.requests.per.connection` 
<a name="maxinflightrequestsperconnection"></a>

The `max.in.flight.requests.per.connection` configuration in Kafka producers controls how many requests can be sent concurrently to the Kafka broker over a single connection. This setting plays a crucial role in determining how Kafka handles the trade-off between throughput, message ordering, and data reliability.

##### How it Works:
<a name="how-it-works"></a>

- When a producer sends messages to Kafka, each message is sent in a request to the broker.
- If the producer has multiple requests to send at the same time (due to batching or high message rates), these requests can either be sent sequentially or in parallel.
- The `max.in.flight.requests.per.connection` setting defines how many such requests can be "in-flight" — meaning waiting for a response from the broker — at any given time.

##### Default Value:
<a name="default-value"></a>
- By default, this value is set to `5`. This allows the producer to send up to five requests at once before receiving responses for any of them.

```java
props.put("max.in.flight.requests.per.connection", 5);
```

##### Impact on Performance and Ordering:
<a name="impact-on-performance-and-ordering"></a>

###### 1. **Throughput:**
<a name="1-throughput"></a>
   - **Higher Values**: Setting `max.in.flight.requests.per.connection` to a higher number increases throughput, as the producer can send multiple requests simultaneously without waiting for each one to be acknowledged. This reduces the time spent waiting and maximizes network bandwidth usage.
   
   - **Lower Values**: If you lower the value, for example, to `1`, the producer will wait for a response from the broker for each request before sending the next one. While this ensures ordering, it can limit throughput because the producer spends more time waiting for acknowledgments.

###### 2. **Message Ordering:**
<a name="2-message-ordering"></a>
   - **Higher Values**: When `max.in.flight.requests.per.connection` is greater than 1, messages can potentially be delivered out of order if a failure occurs. For example, if two requests (Request A and Request B) are sent in parallel, and Request A fails while Request B succeeds, Kafka may retry Request A, leading to message reordering.
   
   - **Lower Values**: Setting this value to `1` ensures strict message ordering. This is because the producer waits for each request to be acknowledged before sending the next one, preventing any reordering even in the case of retries.

```java
props.put("max.in.flight.requests.per.connection", 1); // Guarantees strict message order
```

###### 3. **Reliability:**
<a name="3-reliability"></a>
   - **Higher Values**: While more concurrent requests can improve throughput, they can lead to situations where retries (in case of failure) result in duplicate messages or out-of-order delivery.
   
   - **Lower Values**: With a value of `1`, Kafka guarantees that even if a retry occurs (due to network failure, for example), messages will be delivered in the correct order. Coupled with `enable.idempotence=true`, this ensures that no duplicate messages are sent, and they arrive in the correct sequence.

##### Recommended Settings:
<a name="recommended-settings"></a>

- **For High Throughput**: 
  - Use higher values (such as `5` or more). This allows Kafka to send multiple messages simultaneously, maximizing throughput at the potential cost of strict ordering if failures occur.

- **For Exactly Once Semantics (EOS)**:
  - Use `1` in conjunction with `enable.idempotence=true` and `retries` to ensure no duplicates and guarantee strict message ordering.
  
```java
// High throughput with risk of out-of-order messages on failure
props.put("max.in.flight.requests.per.connection", 5);
props.put("enable.idempotence", "true");

// Strict ordering and exactly-once delivery
props.put("max.in.flight.requests.per.connection", 1);
props.put("enable.idempotence", "true");
props.put("acks", "all");
props.put("retries", Integer.MAX_VALUE);
```

---

##### Summary of Key Points:
<a name="summary-of-key-points"></a>

- **Higher `max.in.flight.requests.per.connection`** increases throughput but may lead to message reordering on failure.
- **Lower `max.in.flight.requests.per.connection`** ensures strict ordering but may decrease throughput as the producer waits for each acknowledgment.
- For **exactly-once semantics (EOS)** and strict ordering, set it to `1` in conjunction with `enable.idempotence=true`.
  
---

##### Diagram: Effect of `max.in.flight.requests.per.connection` on Kafka Producer Behavior
<a name="diagram-effect-of-maxinflightrequestsperconnection-on-kafka-producer-behavior"></a>

```plaintext
+------------------------+          +----------------------+          +-----------------------+
| max.in.flight = 5      | ------>  | Higher Throughput    |  ----->  | Possible Reordering   |
| Multiple concurrent    |          | Faster message send  |          | Retries can cause out-|
| requests allowed       |          | with reduced latency |          | of-order delivery     |
+------------------------+          +----------------------+          +-----------------------+

+------------------------+          +----------------------+          +-----------------------+
| max.in.flight = 1      | ------>  | Strict Ordering      |  ----->  | Lower Throughput      |
| One request at a time  |          | Each message waits   |          | Wait time for each    |
| Strict sequence        |          | for ack before next  |          | acknowledgment        |
+------------------------+          +----------------------+          +-----------------------+
```