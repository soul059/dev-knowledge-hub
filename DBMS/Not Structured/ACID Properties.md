---
tags:
  - database
  - acid
  - transactions
  - data-integrity
---

# ACID Properties

The **ACID** properties are a set of four fundamental guarantees that [[DBMS|Database Management Systems]] aim to ensure for [[SQL Transactions|database transactions]]. They are the cornerstone of reliable and consistent transaction processing, especially in multi-user environments and in the face of failures.

ACID is an acronym for:

1.  **Atomicity**
2.  **Consistency**
3.  **Isolation**
4.  **Durability**

## 1. Atomicity

* **Definition:** A transaction is treated as a single, indivisible unit of work. It either executes completely successfully and all its changes are permanently recorded in the database, or if any part of the transaction fails at any point, the entire transaction is aborted, and the database is rolled back to its state *before* the transaction began.
* **Meaning:** There is no concept of a partially completed transaction. It's "all or nothing."
* **Importance:** Prevents inconsistent states resulting from partial updates. For example, in a money transfer, Atomicity ensures that money is either deducted from one account *and* added to the other, or neither happens.

## 2. Consistency

* **Definition:** A transaction takes the database from one **valid state** to another **valid state**. A valid state is one that satisfies all predefined rules and [[SQL Constraints|integrity constraints]] (like Primary Keys, Foreign Keys, Check constraints).
* **Meaning:** The transaction itself does not violate any database rules. If a transaction starts with a consistent database, it must end with a consistent database.
* **Importance:** Ensures that the database remains logically correct and that data adheres to defined business rules after a transaction completes.

## 3. Isolation

* **Definition:** Concurrent transactions execute in isolation from one another. The effect of running multiple transactions simultaneously is the same as if they had executed one after another in some serial order.
* **Meaning:** Transactions do not interfere with each other. Data being modified by one transaction is protected from reads or writes by other transactions until the first transaction is committed.
* **Importance:** Prevents anomalies that can arise from concurrent access, such as dirty reads (reading uncommitted data), non-repeatable reads (reading the same data twice but getting different results), and phantom reads (a query returning a different set of rows upon re-execution within the same transaction due to insertions/deletions by another committed transaction). Different **Isolation Levels** (like Read Uncommitted, Read Committed, Repeatable Read, Serializable) offer varying degrees of isolation with corresponding performance trade-offs.

## 4. Durability

* **Definition:** Once a transaction has been successfully completed and **committed**, its changes are permanent and will survive even in the event of a system failure (like a power outage, crash, or reboot).
* **Meaning:** Committed changes are written to non-volatile storage (like disk) in a way that guarantees they will not be lost.
* **Importance:** Ensures that committed data is persistent and reliable, providing confidence that once a transaction is confirmed, its effects are final. This is typically achieved through mechanisms like logging and checkpointing managed by the [[Transaction Management]]'s recovery component.

The ACID properties are the cornerstone of transaction processing reliability in most traditional [[DBMS|Relational Database Management Systems]], ensuring data integrity and system robustness.

---
**Related Notes:**
* [[Database Concepts]]
* [[DBMS]]
* [[Transaction Management]]
* [[SQL Transactions]]
* [[SQL Constraints]]
* [[Concurrency Control]]
* [[Database Recovery]]