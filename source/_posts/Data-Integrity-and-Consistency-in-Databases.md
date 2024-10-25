---
title: Data Integrity and Consistency in Databases
date: 2024-10-24 20:25:06
categories:
- RDBMS
tags:
- RDBMS
---


- [Data Integrity and Consistency in Databases](#data-integrity-and-consistency-in-databases)
  - [Introduction](#introduction)
  - [Definitions](#definitions)
  - [Example Scenario: E-commerce System](#example-scenario-e-commerce-system)
    - [Products Table](#products-table)
    - [Orders Table](#orders-table)
    - [Part 1: Data Integrity](#part-1-data-integrity)
    - [Part 2: Data Consistency in a Transaction](#part-2-data-consistency-in-a-transaction)
      - [Transaction Scenario: Placing an Order](#transaction-scenario-placing-an-order)
  - [Example of Inconsistent State Without Transactions](#example-of-inconsistent-state-without-transactions)
  - [Conclusion](#conclusion)

---

# Data Integrity and Consistency in Databases

---

<a name="introduction"></a>
## Introduction

To understand **data integrity** and **consistency**, let’s break them down with an example, illustrating how they work in a transactional database environment.

---

<a name="definitions"></a>
## Definitions

- **Data Integrity**: Refers to the accuracy and reliability of the data. Data integrity ensures that the data is **correct, complete**, and consistent throughout its lifecycle. It is often enforced through rules such as **constraints** (e.g., primary keys, foreign keys, unique constraints).
  
- **Consistency**: In the context of transactions, consistency ensures that the database transitions from one **valid state** to another. This means that after every transaction, the database adheres to all rules and constraints that maintain its correctness.

---

<a name="example-scenario-ecommerce-system"></a>
## Example Scenario: E-commerce System

Imagine an e-commerce system where users place orders for products. We have two important tables:

1. **Products** table: Stores information about products, including `product_id`, `name`, `price`, and `quantity_in_stock`.
2. **Orders** table: Stores orders placed by customers, including `order_id`, `product_id`, `quantity_ordered`, and `status`.

### Products Table

| product_id | name          | price | quantity_in_stock |
|------------|---------------|-------|-------------------|
| 1          | Laptop        | 1000  | 10                |
| 2          | Smartphone    | 800   | 20                |
| 3          | Headphones    | 150   | 30                |

### Orders Table

| order_id | product_id | quantity_ordered | status   |
|----------|------------|------------------|----------|
| 101      | 1          | 2                | PENDING  |
| 102      | 2          | 1                | SHIPPED  |

---

<a name="part-1-data-integrity"></a>
### Part 1: Data Integrity

**Data Integrity** ensures that the data adheres to business rules and constraints. Let's define some **integrity constraints** in this system:

1. **Foreign Key Constraint**: Ensures that every `product_id` in the `Orders` table refers to a valid `product_id` in the `Products` table. This prevents orders from being placed for non-existent products.

   ```sql
   ALTER TABLE Orders
   ADD CONSTRAINT fk_product_id
   FOREIGN KEY (product_id) REFERENCES Products(product_id);
   ```

   **Example Violation**:  
   If someone tries to insert an order with `product_id = 999` (which does not exist in the Products table), the foreign key constraint would **reject** the transaction.

   ```sql
   INSERT INTO Orders (order_id, product_id, quantity_ordered, status)
   VALUES (103, 999, 1, 'PENDING');
   ```

   **Result**: Integrity violation. The order cannot be placed because `product_id = 999` does not exist in the `Products` table.

2. **Check Constraint**: Ensures that no negative quantity can be ordered or stored in the inventory.

   ```sql
   ALTER TABLE Products
   ADD CONSTRAINT chk_positive_quantity CHECK (quantity_in_stock >= 0);
   ```

   **Example Violation**:  
   If an update tries to set `quantity_in_stock` to a negative value, it will be rejected.

   ```sql
   UPDATE Products
   SET quantity_in_stock = -5
   WHERE product_id = 1;
   ```

   **Result**: Integrity violation. Negative stock values are not allowed.

3. **Unique Constraint**: Ensures no duplicate `product_id` exists in the `Products` table.

   ```sql
   ALTER TABLE Products
   ADD CONSTRAINT unique_product_id UNIQUE (product_id);
   ```

   **Example Violation**:  
   If we try to insert another product with the same `product_id`:

   ```sql
   INSERT INTO Products (product_id, name, price, quantity_in_stock)
   VALUES (1, 'Tablet', 500, 15);
   ```

   **Result**: Integrity violation. Duplicate product IDs are not allowed.

These constraints enforce **data integrity** by ensuring that only valid data can be inserted or updated in the database.

---

<a name="part-2-data-consistency-in-a-transaction"></a>
### Part 2: Data Consistency in a Transaction

**Data Consistency** means that after a transaction, all defined integrity rules are maintained, and the system remains in a valid state. Let's explore this with a transactional scenario.

#### Transaction Scenario: Placing an Order

When a user places an order, several things happen as part of a **single transaction**:

1. The system **decreases** the quantity of the product in the `Products` table.
2. A new order is **inserted** into the `Orders` table.
3. If the order cannot be completed for some reason (e.g., insufficient stock), the entire transaction is **rolled back**.

Let’s simulate this with an example where a customer places an order for 3 laptops (`product_id = 1`).

1. **Initial State**:
   - `Products` table shows 10 laptops in stock.
   - No new order is placed yet.

2. **Transaction Start**:
   - Begin the transaction:
   
     ```sql
     BEGIN TRANSACTION;
     ```

3. **Check Stock**:  
   First, the system checks if there are enough laptops in stock.

   ```sql
   SELECT quantity_in_stock FROM Products WHERE product_id = 1;
   -- Result: 10
   ```

4. **Decrease Stock**:  
   If there is enough stock, decrease the stock by the ordered quantity.

   ```sql
   UPDATE Products
   SET quantity_in_stock = quantity_in_stock - 3
   WHERE product_id = 1;
   ```

   After the update, the stock of laptops is reduced to 7.

5. **Insert Order**:  
   Insert the new order into the `Orders` table.

   ```sql
   INSERT INTO Orders (order_id, product_id, quantity_ordered, status)
   VALUES (103, 1, 3, 'PENDING');
   ```

6. **Commit Transaction**:  
   If everything succeeds (no constraint violations, no failures), commit the transaction.

   ```sql
   COMMIT;
   ```

7. **Final State**:
   - The `Products` table now shows 7 laptops in stock.
   - A new order has been added to the `Orders` table.

**Data Consistency** is maintained because the system follows the ACID properties. The **database's state** transitions from a valid state to another valid state (the stock is updated, and the order is recorded correctly).

---

<a name="example-of-inconsistent-state-without-transactions"></a>
## Example of Inconsistent State Without Transactions

Now let’s imagine what would happen if transactions were **not used**, leading to **inconsistent data**.

1. **Decrease Stock**:  
   The system updates the stock to 7 but fails to insert the order due to a system error.

   ```sql
   UPDATE Products
   SET quantity_in_stock = quantity_in_stock - 3
   WHERE product_id = 1;
   -- Success: stock reduced to 7
   ```

2. **Insert Order Fails**:  
   The order insertion fails (due to a network issue or other error).

   ```sql
   INSERT INTO Orders (order_id, product_id, quantity_ordered, status)
   VALUES (103, 1, 3, 'PENDING');
   -- Error: Failure to insert the order
   ```

Since the database **was not in a transaction**, the stock was reduced but the order was never recorded. This results in an **inconsistent state** where the stock is reduced but no corresponding order exists, violating the system’s expected behavior.

---

<a name="conclusion"></a>
## Conclusion

- **Data Integrity** ensures that only valid and accurate data can be entered or modified in the database by using constraints like foreign keys, check constraints, and unique constraints.
- **Data Consistency** ensures that after a transaction, the database remains in a valid state, where all integrity rules are maintained and the system behaves as expected.

By using proper transaction management and enforcing data integrity rules, we can prevent inconsistent states and maintain the overall reliability of an online system.

