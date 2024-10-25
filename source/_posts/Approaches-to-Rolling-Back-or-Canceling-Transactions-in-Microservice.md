---
title: Approaches to Rolling Back or Canceling Transactions in Microservice
date: 2024-10-24 23:17:16
categories:
- Transaction 
- Microservice
tags:
- Transaction 
- Microservice
---

Handling rollback or cancellation of transactions in a **Microservice Architecture** when multiple writes are involved is a complex process because each microservice typically has its own database, and there is no central point of coordination like in monolithic applications. Traditional database transactions (ACID) are difficult to apply in microservices, especially when services span across multiple databases, message brokers, and external APIs. Therefore, rollback in microservices needs a more sophisticated approach.

### Key Approaches to Rolling Back or Canceling Transactions in Microservice Architecture

There are several strategies to handle rollback or cancellation in microservices when multiple services and writes are involved. These include:

- [1. Saga Pattern](#1-saga-pattern)
  - [a. Choreography-based Saga](#a-choreography-based-saga)
  - [b. Orchestration-based Saga](#b-orchestration-based-saga)
- [2. Compensation Mechanism](#2-compensation-mechanism)
- [3. Two-Phase Commit (2PC)](#3-two-phase-commit-2pc)
  - [How 2PC Works:](#how-2pc-works)
- [4. Eventual Consistency](#4-eventual-consistency)
  - [How to Handle Rollbacks with Eventual Consistency:](#how-to-handle-rollbacks-with-eventual-consistency)
- [5. Idempotency and Retries](#5-idempotency-and-retries)
  - [Idempotency:](#idempotency)
  - [Best Practices for Idempotency:](#best-practices-for-idempotency)
- [6. Conclusion](#6-conclusion)

---

<a name="saga-pattern"></a>
## 1. Saga Pattern

The **Saga Pattern** is the most popular strategy for handling distributed transactions in microservices. It breaks a large transaction into a sequence of smaller, independent transactions, each handled by a microservice. If any part of the sequence fails, compensating actions (i.e., rollbacks) are triggered to undo the work done by previous steps.

There are two common approaches to implementing the Saga Pattern:
- **Choreography-based Saga**
- **Orchestration-based Saga**

<a name="choreography-based-saga"></a>
### a. Choreography-based Saga

In a **Choreography-based Saga**, each service involved in the transaction emits an event upon completion of its task. Other services listen to these events and take action accordingly. If an error occurs, the service responsible for the failed transaction emits a failure event, and other services listening for that failure event perform compensating actions.

**Example**:
Imagine an e-commerce system where you place an order that spans multiple microservices:
1. **Order Service**: Creates the order.
2. **Inventory Service**: Reserves the product.
3. **Payment Service**: Charges the customer.
4. **Shipping Service**: Ships the product.

If the **Payment Service** fails after reserving the inventory, it can emit a failure event, and the **Inventory Service** would listen to this and release the reserved product (compensating action).

**Advantages**:
- Fully decentralized and scalable.
- Easy to add new services without affecting others.

**Challenges**:
- Harder to track and manage because there is no central coordinator.
- More complex error handling logic in each service.

<a name="orchestration-based-saga"></a>

### b. Orchestration-based Saga

In an **Orchestration-based Saga**, a central coordinator (Orchestrator) is responsible for managing the workflow of the transaction. The Orchestrator sends commands to each service to execute a step in the transaction and handles failures by issuing compensating commands if any service fails.

**Example**:
- The **Orchestrator** sends a command to the **Order Service** to create an order.
- After receiving success, the Orchestrator asks the **Inventory Service** to reserve the product.
- If the **Payment Service** fails, the Orchestrator issues a compensating command to the **Inventory Service** to release the product.

**Advantages**:
- Easier to implement because the Orchestrator handles the entire workflow.
- Centralized error handling and compensations.

**Challenges**:
- The Orchestrator can become a single point of failure or bottleneck.
- Needs to manage state and retries, making it potentially complex.

---

<a name="compensation-mechanism"></a>
## 2. Compensation Mechanism

The **Compensation Mechanism** is the core of both the Saga Pattern and other eventual consistency models. A compensation transaction is essentially the "undo" operation that reverses the effects of a previously completed step in case of failure.

**How It Works**:
Each microservice is responsible for defining and implementing compensating actions for its operations. For example:
- If the **Inventory Service** reserves stock but the **Payment Service** fails, the Inventory Service must implement logic to release the stock.
- If the **Payment Service** successfully charges a customer but the **Shipping Service** fails, the Payment Service must issue a refund.

**Key Considerations**:
- **Compensation is not always straightforward**: Rolling back a transaction isn't always as simple as undoing a change. Some operations might be irreversible (e.g., shipping a product), requiring compensating actions that mitigate the issue (e.g., issuing a refund).
- **State-driven compensation**: Services must maintain state information to understand which transactions require compensation.

---

<a name="two-phase-commit-2pc"></a>
## 3. Two-Phase Commit (2PC)

**Two-Phase Commit (2PC)** is a protocol used for coordinating distributed transactions across multiple systems. It's traditionally used in tightly coupled distributed systems but is less popular in microservice architectures due to performance and complexity concerns.

### How 2PC Works:
1. **Prepare Phase**: The transaction coordinator asks each microservice if they are ready to commit the transaction. The services reply with "yes" or "no."
2. **Commit/Rollback Phase**: If all services reply with "yes," the coordinator instructs them to commit. If any service replies with "no," the coordinator sends a rollback command to all participating services.

**Pros**:
- Strong consistency guarantees.
- Ensures atomicity across distributed systems.

**Cons**:
- **Performance overhead**: 2PC introduces a blocking wait time for services to prepare and commit, which can hurt performance in a microservices architecture.
- **Risk of blocking**: If one service crashes, it can leave other services waiting indefinitely, potentially causing system-wide issues.

Given these challenges, 2PC is rarely used in modern, large-scale microservice architectures.

---

<a name="eventual-consistency"></a>
## 4. Eventual Consistency

In distributed microservice architectures, **eventual consistency** is often the goal rather than strong consistency (ACID). Eventual consistency ensures that after some time, all participants in a distributed transaction will reach a consistent state, even if there are temporary inconsistencies.

### How to Handle Rollbacks with Eventual Consistency:
- **Delayed reconciliation**: Some systems may use periodic reconciliation processes to detect inconsistencies and trigger compensating actions to ensure data integrity.
- **State-based updates**: Services track their state, and if a failure is detected, each service is responsible for ensuring consistency through compensation.

**Example**:
If a **Payment Service** charges a customer but the **Shipping Service** fails to ship the item, the system might temporarily be in an inconsistent state (payment made, no shipment). Over time, a reconciliation job or event listener detects the issue and initiates a refund to bring the system back to consistency.

**Pros**:
- High scalability and resilience.
- Tolerates temporary inconsistencies while ensuring long-term data integrity.

**Cons**:
- Complex to reason about, as temporary inconsistencies can confuse users and developers.
- Some actions (e.g., shipping a product) may be difficult to compensate.

---

<a name="idempotency-and-retries"></a>
## 5. Idempotency and Retries

In microservice architectures, **idempotency** and **retries** play a critical role in handling rollbacks and retries, ensuring that repeated operations do not result in unwanted side effects.

### Idempotency:
An operation is **idempotent** if performing it multiple times has the same result as performing it once. For example, if a **Payment Service** is asked to process a payment twice, it should charge the customer only once.

**Why It Matters**:
- When rolling back a transaction, services may need to retry operations (e.g., issuing a refund). These retries must be idempotent to prevent issues like charging a customer multiple times or reserving inventory repeatedly.
- **Retry policies** are often implemented in communication mechanisms (e.g., message brokers) to ensure reliability.

### Best Practices for Idempotency:
- Use unique transaction IDs for requests so that services can detect duplicate requests.
- Design APIs and actions to ensure they are safe for retries.

---

<a name="conclusion"></a>
## 6. Conclusion

Rolling back or canceling transactions in a **Microservice Architecture** requires careful design to handle distributed data, asynchronous operations, and eventual consistency. The **Saga Pattern** (both choreography and orchestration) is the most widely used approach, allowing each service to independently handle parts of the transaction and apply compensating actions in case of failure. Other techniques like **compensation mechanisms**, **eventual consistency**, and **idempotency** ensure that microservices can maintain integrity, even across complex, distributed environments.

When choosing a rollback strategy, it's essential to balance the need for consistency with the performance and scalability requirements of the system.
