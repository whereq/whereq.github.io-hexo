---
title: Kafka Transaction
date: 2024-10-23 14:06:39
categories:
- Kafka
tags:
- Kafka
---

# Kafka Transaction Workflow

### Index
- [Kafka Transaction Workflow](#kafka-transaction-workflow)
    - [Index](#index)
    - [1. Introduction](#1-introduction)
    - [2. Idempotence in Kafka](#2-idempotence-in-kafka)
      - [2.1 How Idempotence Works](#21-how-idempotence-works)
    - [3. Kafka Transactions](#3-kafka-transactions)
      - [3.1 How to Enable Kafka Transactions](#31-how-to-enable-kafka-transactions)
        - [Example: Creating a Kafka Producer with Transactions](#example-creating-a-kafka-producer-with-transactions)
      - [3.2 How Kafka Transactions Work](#32-how-kafka-transactions-work)
        - [1) Starting the Producer and Assigning a Transaction Coordinator](#1-starting-the-producer-and-assigning-a-transaction-coordinator)
        - [2) Sending Messages](#2-sending-messages)
        - [3) Committing or Aborting the Transaction](#3-committing-or-aborting-the-transaction)
    - [4. Diagram: Kafka Transaction Flow](#4-diagram-kafka-transaction-flow)
    - [5. Conclusion](#5-conclusion)

---

### 1. Introduction

Let’s first recall the concept of a transaction: either all operations succeed, or all of them fail. Kafka transactions follow the same rule.

Starting from Kafka 0.11.0.0, two major features were introduced: **idempotence** and **transactions**. Understanding transactions requires understanding idempotence because transactions are built on top of idempotence. This article explains how Kafka transactions work in simple terms, without diving too deep into technical details.

The key points we will cover:
- What is idempotence? How to enable it?
- How does idempotence work?
- What are Kafka transactions? How to enable transactions?
- How do Kafka transactions work?

---

### 2. Idempotence in Kafka

Idempotence ensures that no matter how many times the producer sends the same message, the Kafka broker will only persist it once, ensuring no data duplication. Idempotence is enabled by default and can be configured using the `enable.idempotence` setting.

#### 2.1 How Idempotence Works

The idempotence mechanism in Kafka is simple. Each message has a unique key composed of `<PID, Partition, SeqNumber>`:
- **PID (Producer ID):** A unique identifier assigned to each producer when it starts.
- **Partition:** The target partition for the message.
- **SeqNumber:** An incrementing ID assigned to each message by the producer, ensuring that each message has a unique sequence number.

Kafka prevents duplicate message persistence for messages with the same key. However, idempotence is limited to single partition and single session guarantees. If Kafka restarts and a new PID is assigned, duplicates could occur. This is where **Kafka transactions** come into play.

---

### 3. Kafka Transactions

Kafka transactions allow for atomic writes across multiple topics and partitions. All messages in the same transaction are either fully committed or fully aborted. 

Kafka transactions are primarily concerned with **producer transactions**, though **consumer transactions** are also possible. However, consumers rely on the producer’s transaction mechanism, so we will focus on producer transactions here.

#### 3.1 How to Enable Kafka Transactions

Before delving into the internals of Kafka transactions, let’s start with a simple example.

##### Example: Creating a Kafka Producer with Transactions

```java
Properties properties = new Properties();
properties.setProperty(ProducerConfig.BOOTSTRAP_SERVERS_CONFIG, "localhost:9092");
properties.setProperty(ProducerConfig.KEY_SERIALIZER_CLASS_CONFIG, StringSerializer.class.getName());
properties.setProperty(ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG, StringSerializer.class.getName());

// Set the transactional ID, this is required
properties.setProperty(ProducerConfig.TRANSACTIONAL_ID_CONFIG, "transactional_id_1");

KafkaProducer<String, String> producer = new KafkaProducer<>(properties);

// Initialize transaction
producer.initTransactions();

// Start transaction
producer.beginTransaction();

try {
    // Send 10 messages to Kafka, if an error occurs, all messages will be aborted
    for (int i = 0; i < 10; i++) {
        producer.send(new ProducerRecord<>("topic-test", "a message " + i));
    }
    // Commit the transaction
    producer.commitTransaction();
} catch (Exception e) {
    // Abort the transaction if any exception occurs
    producer.abortTransaction();
} finally {
    producer.close();
}
```

In this code:
- The producer is configured with a **transactional ID**, which is essential for Kafka transactions.
- The producer starts a transaction with `beginTransaction()`.
- After sending messages, it commits the transaction using `commitTransaction()`. If any error occurs, the transaction is aborted with `abortTransaction()`.

#### 3.2 How Kafka Transactions Work

##### 1) Starting the Producer and Assigning a Transaction Coordinator
When using transactions, the producer must be assigned a **transactional ID**. Upon startup, Kafka assigns a **Transaction Coordinator** based on this ID. Each Kafka broker has a transaction coordinator responsible for assigning **Producer IDs (PIDs)** and managing transactions.

The allocation of a transaction coordinator involves a special Kafka topic `__transaction_state`, which by default has 50 partitions. Each partition is responsible for a portion of the transactions. Kafka calculates the hash of the transactional ID and assigns the producer to a specific partition, whose leader broker will become the transaction coordinator.

After assigning a coordinator, the coordinator assigns a **PID** to the producer, allowing it to start sending messages.

##### 2) Sending Messages
After receiving its PID, the producer informs the coordinator which partitions the transaction will target. Then, it starts sending messages. These messages are flagged as part of a transaction.

Once the producer has sent all its transactional messages, it sends either a **Commit** or **Abort** request to the transaction coordinator, signaling the end of its task.

##### 3) Committing or Aborting the Transaction
When the producer starts sending messages, the coordinator marks the beginning of the transaction and records it in the `__transaction_state` topic.

Once all messages are sent, or in case of failure, the coordinator will receive a **Commit** or **Abort** request. The coordinator then communicates with all the topics involved in the transaction. If the transaction is successful, the topics confirm receipt of the messages, and the coordinator records the successful transaction.

If the transaction fails, all messages related to the transaction are discarded, and the coordinator records the failure in `__transaction_state`.

---

### 4. Diagram: Kafka Transaction Flow

Below is a diagram representing the Kafka transaction process:

```plaintext
+---------------------------+        +----------------------+        +---------------------+
|   Producer Starts         | -----> | Assign Transaction   | -----> | Send Messages to    |
|   (with transactional ID) |        | Coordinator & Assign |        | Partitions          |
+---------------------------+        | PID                  |        +---------------------+
                                     +----------------------+

                                           |
                                           |
                                           v
+------------------------+        +------------------------+        +----------------------+
|   Transaction Begins   | -----> | Send Commit/Abort      | -----> | Coordinator Confirms |
+------------------------+        | Request to Coordinator |        | Success or Failure   |
                                  +------------------------+        +----------------------+
```

1. **Producer Starts**: The producer is assigned a transactional ID and a transaction coordinator.
2. **Sending Messages**: Messages are sent as part of the transaction.
3. **Commit/Abort**: The producer sends a Commit or Abort request to the coordinator.
4. **Confirmation**: The coordinator confirms the success or failure of the transaction.

---

### 5. Conclusion

Kafka transactions are a powerful feature that enables atomic writes across multiple topics and partitions, ensuring either all messages succeed or all fail. Built on top of Kafka's idempotence mechanism, Kafka transactions provide robust guarantees for message processing, making it a reliable tool for distributed systems.