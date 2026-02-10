---
tags:
  - database
  - dbms-component
  - transactions
  - concurrency-control
  - recovery
---

# Transaction Management

**Transaction Management** is a critical component of a [[DBMS|Database Management System]] responsible for ensuring the reliability and consistency of the database, especially in environments with multiple concurrent users or potential system failures. It ensures that [[SQL Transactions|database transactions]] adhere to the [[ACID Properties]].

## Role and Purpose

The Transaction Management component has two main responsibilities:

1.  **Concurrency Control:** Managing the interaction of multiple [[SQL Transactions|transactions]] that are executing simultaneously to prevent them from interfering with each other in ways that would compromise data consistency.
2.  **Recovery Management:** Ensuring that the database can be restored to a correct state after a system failure (like a crash, power loss, or disk error), preserving the changes of committed transactions and undoing the effects of incomplete transactions.

## Key Concepts

* **Transaction:** A single logical unit of work comprising one or more database operations (e.g., a sequence of [[SQL DML|INSERT]], [[SQL DML|UPDATE]], [[SQL DML|DELETE]] statements). It's the basic unit of reliability and consistency. (See [[SQL Transactions]]).
* **[[ACID Properties]]:** The guarantees that a transaction management system aims to provide for every transaction.
* **Concurrency:** The execution of multiple transactions at the same time. While increasing throughput, it introduces challenges related to data integrity.
* **System Failure:** Events like hardware failure, software errors, or power loss that can interrupt transaction execution.

## Components of Transaction Management

* **Concurrency Control Manager:** This component's goal is to serialize the execution of concurrent transactions, meaning that even though they run at the same time, the final result is equivalent to some sequential execution of those transactions. It prevents anomalies like:
    * **Dirty Reads:** Reading data written by an uncommitted transaction.
    * **Non-Repeatable Reads:** Reading the same data twice in one transaction and getting different values because another committed transaction modified it in between.
    * **Phantom Reads:** A transaction re-running a query and finding new rows inserted by another committed transaction.
    * **Lost Updates:** Two transactions updating the same data, and one update is overwritten by the other.
    * *Common techniques include locking (e.g., two-phase locking) and timestamping.*
* **Recovery Manager:** This component is responsible for restoring the database to a consistent state after a failure. It uses logs (records of database changes) and potentially checkpoints (periodic snapshots of the database state) to:
    * **Rollback (Undo):** Undo the changes of transactions that were incomplete or active at the time of the failure.
    * **Rollforward (Redo):** Reapply the changes of transactions that were committed before the failure but whose changes might not have been fully written to disk.

Effective Transaction Management is fundamental to the robustness and trustworthiness of any [[DBMS]], ensuring that data remains accurate and accessible even under heavy load or unexpected interruptions.

---
**Related Notes:**
* [[Database Concepts]]
* [[DBMS]]
* [[ACID Properties]]
* [[SQL Transactions]]
* [[Concurrency Control]] (Create if needed)
* [[Database Recovery]] (Create if needed)