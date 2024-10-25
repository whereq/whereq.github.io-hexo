---
title: Considerations on Transactions Across Distributed Systems
date: 2024-10-24 20:43:50
categories:
- Transaction 
- Distributed System
tags:
- Transaction 
- Distributed System
---

### Online Systems: Defining Transactions, Database Transaction Management, and Lock Management

When working with **complex online systems** (e.g., e-commerce platforms, distributed systems, microservices architecture), transactions span beyond a single database. This requires a broader understanding of transaction management, involving **distributed transactions**, **data consistency across systems**, and sophisticated **lock management** techniques. Let's break this down with a focus on the key topics: defining transactions in online systems, transaction management (beyond the database), and lock management in such environments.

---

### 1. Defining Transactions in Online Systems

In an **online system**, a **transaction** is a **sequence of operations** performed as a single logical unit, ensuring **consistency, correctness, and isolation** across different subsystems. These operations could involve:

- **Database operations** (e.g., updating inventory, placing an order)
- **Third-party API calls** (e.g., payment gateways, external services)
- **Message queue handling** (e.g., Kafka, RabbitMQ for event-driven architecture)
- **Microservices interaction** (e.g., coordination between an inventory service and an order service)
  
In simple terms, a transaction in an online system is any series of related actions across multiple components that must either **complete successfully together** or **fail together** to maintain a consistent system state.

#### Key Transaction Properties: ACID vs BASE
In large-scale systems, transactions must be designed for both **strong consistency** (ACID) or **eventual consistency** (BASE), depending on the use case.

1. **ACID (Atomicity, Consistency, Isolation, Durability)**:
   - Guarantees **strong consistency**.
   - Used when all subsystems must maintain a single consistent state.
   - Common in **single database** transactions or within tightly-coupled services.

2. **BASE (Basically Available, Soft-state, Eventually consistent)**:
   - Sacrifices immediate consistency for **availability** and **scalability**.
   - Often used in **distributed systems** where components may not always be in sync (e.g., eventual consistency in NoSQL databases).
  
In online systems, distributed transactions often require a mix of **ACID** and **BASE** properties, depending on the criticality of the operation.

---

### 2. Transaction Management in Complex Online Systems

Transaction management in online systems can no longer rely solely on single-database transaction managers. These systems often require **distributed transaction management**. Some of the most critical techniques include:

#### a. **Distributed Transactions (Two-Phase Commit)**

When a transaction spans across **multiple services** or databases, a **Two-Phase Commit (2PC)** protocol is commonly used to ensure atomicity. 

- **Phase 1 - Prepare**: All participating systems are asked to **prepare** (lock resources and confirm they can commit the transaction).
- **Phase 2 - Commit/Rollback**: If all participants agree, the transaction is **committed**. If any participant fails, the entire transaction is **rolled back**.

Example:
- An order is placed in an online store:
   - The **Order Service** inserts an order into the database.
   - The **Payment Service** processes the payment.
   - The **Inventory Service** updates the stock.
   
All three operations must be successful for the order to be confirmed. The **2PC** ensures that if any step fails, all previous operations are rolled back.

However, **2PC** can be slow and does not scale well in high-throughput environments due to blocking resources during the "prepare" phase.

#### b. **Saga Pattern**

The **Saga Pattern** is an alternative to 2PC, often used in **microservices architectures** where long-lived transactions are undesirable. Instead of blocking resources, the Saga breaks a distributed transaction into a series of smaller transactions.

- Each service involved in the transaction performs its work independently.
- If one step fails, the previous steps are **compensated** by executing defined rollback actions.

Example:
1. **Order Service** places an order.
2. **Payment Service** charges the customer.
3. **Inventory Service** updates the stock.
4. If the **Inventory Service** fails, a compensating transaction **refunds** the payment and **cancels** the order.

**Sagas** offer better **scalability** compared to 2PC and are particularly suited for **event-driven** architectures where each microservice manages its own transaction.

#### c. **Eventual Consistency and Compensation**

In some distributed systems, strict ACID properties are relaxed for **eventual consistency**, especially in **BASE systems** (e.g., NoSQL databases, cloud-native architectures). Instead of locking resources, systems rely on **compensation** logic:

- If a downstream service fails or an external API does not respond, the system might retry the operation or initiate compensating actions to restore consistency.
  
**Eventual Consistency Example**:
- In a **global payment system**, payments made across multiple time zones and currencies may not reflect immediately but will eventually be reconciled to ensure data consistency.

#### d. **Message-Driven Transactions**

In **event-driven systems**, message queues (e.g., **Kafka**, **RabbitMQ**) help decouple services, allowing them to handle transactions asynchronously. Here, services:

- **Emit events** for actions like "Order Placed" or "Payment Processed".
- **Listen** for events from other services to trigger corresponding actions (e.g., update inventory when an order is placed).
  
By designing for **idempotency** (ensuring repeated operations have the same effect as a single operation), these systems can handle eventual consistency and retries gracefully.

---

### 3. Lock Management

In online systems, **lock management** becomes complex due to multiple participants, different types of data stores, and network latencies. Locking is crucial to prevent data corruption or race conditions during concurrent transactions.

#### a. **Optimistic vs Pessimistic Locking**

- **Pessimistic Locking**: A **resource is locked** for the entire duration of the transaction, preventing any other transaction from accessing it.
  - Used when **contention** for the resource is high or when updates must be serialized (e.g., bank account transfers).
  - Example: A row in the `Inventory` table is locked while updating stock.

  ```sql
  SELECT * FROM Products WHERE product_id = 1 FOR UPDATE;
  ```

- **Optimistic Locking**: Assumes conflicts are rare and does not lock the resource. Instead, it checks for conflicts only at the end of the transaction.
  - **Version columns** or **timestamps** are used to detect if another transaction modified the same data. If so, the transaction fails, and a retry is triggered.
  - Example: Update an order only if no other transaction has modified it.
  
  ```sql
  UPDATE Orders
  SET status = 'SHIPPED'
  WHERE order_id = 101 AND version = 1;
  ```

#### b. **Distributed Locks**

In distributed systems, transactions may need to span multiple nodes or databases. To handle this, **distributed locks** ensure that shared resources are not modified concurrently by different parts of the system.

1. **Redis-Based Locks**: A simple solution is to use **Redis** to acquire distributed locks.

   Example:  
   When updating inventory, each microservice tries to acquire a lock for the product:
   ```shell
   SET product_lock:1 my_unique_id NX PX 10000
   ```

   - `NX` ensures the lock is acquired only if it doesn't exist.
   - `PX 10000` ensures the lock expires after 10 seconds if not released.

2. **Zookeeper or Etcd**: More robust systems like **Zookeeper** or **Etcd** can handle distributed coordination, offering **leader election**, **mutexes**, and **barriers**.
  
   Example: Distributed systems can use Zookeeper to ensure that **only one instance** of a service performs a critical operation (e.g., process payments in a sharded environment).

#### c. **Deadlock Detection and Prevention**

In systems with complex transactions and lock dependencies, **deadlocks** can occur, where two transactions are waiting for resources locked by each other.

- **Deadlock Detection**: The system detects circular dependencies and aborts one of the transactions to resolve the deadlock.
  
  Example:
  ```sql
  SET innodb_lock_wait_timeout = 50;
  ```

- **Deadlock Prevention**: In systems like **databases** or **distributed systems**, adopting strategies like **acquiring locks in a consistent order** can prevent deadlocks.

---

### Conclusion

In an **online system**, transaction management and lock management are more complex than in a simple intranet application. The system must account for **distributed transactions** across multiple services and databases, often relying on protocols like **2PC**, **Sagas**, or **message-driven event handling** to ensure consistency and reliability. Locks, whether in-database or distributed, are critical for managing concurrency and ensuring data integrity but must be handled with care to avoid performance bottlenecks or deadlocks.

In essence, designing transactions in a modern online system requires a mix of strategies from **distributed transaction management**, **eventual consistency models**, and **advanced locking mechanisms** to maintain data reliability, availability, and scalability.