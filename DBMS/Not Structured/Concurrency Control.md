---
tags:
  - database
  - concurrency
  - transaction-management
  - isolation-level
---

# Concurrency Control

**Concurrency Control** is a database management activity that coordinates the simultaneous execution of [[SQL Transactions|transactions]] to ensure that the database remains consistent. In a multi-user [[DBMS|Database Management System]], many users may access and modify the same data concurrently. Without proper control, this can lead to errors and data corruption.

## The Problem Without Concurrency Control

In the [[File Oriented Approach]] or simple multi-user systems without concurrency control, allowing interleaved execution of operations from multiple transactions can cause anomalies, such as:

* **Lost Update Problem:** Two transactions read the same data item, both update it, and the second update overwrites the first, causing the first update to be lost.
    * *Example:* Two users simultaneously update a bank balance. User 1 reads balance (100), User 2 reads balance (100). User 1 calculates new balance (100+50=150) and writes 150. User 2 calculates new balance (100-20=80) and writes 80. The final balance is 80, the +50 update is lost.
* **Dirty Read Problem (Reading Uncommitted Data):** A transaction reads data that has been written by another transaction that has not yet committed. If the second transaction then rolls back, the first transaction will have read data that never officially existed in the database.
* **Non-Repeatable Read Problem:** A transaction reads the same data item more than once. Another transaction modifies (updates or deletes) that data item and commits in between the reads. The first transaction gets different values, violating the assumption that its reads are repeatable.
* **Phantom Read Problem:** A transaction executes a query that retrieves a set of rows based on a condition. Another transaction inserts or deletes rows that satisfy the condition and commits. The first transaction re-executes the query and finds a different set of rows ("phantoms") than before.

## How DBMS Provides Concurrency Control

The [[Transaction Management]] component of a [[DBMS]] includes a Concurrency Control Manager specifically designed to handle these issues. Its goal is to manage interleaved transaction execution such that the final result is equivalent to some serial (sequential) execution of those transactions. This property is known as **Serializability**.

Common Concurrency Control techniques include:

* **Locking:** Transactions acquire locks on data items before accessing them. Different types of locks (e.g., shared locks for reading, exclusive locks for writing) prevent conflicting access. Two-Phase Locking (2PL) is a common protocol ensuring serializability.
* **Timestamping:** Assigning a unique timestamp to each transaction. The system then ensures that transaction execution respects the timestamps, often by rolling back transactions that violate timestamp order.
* **Multi-Version Concurrency Control (MVCC):** The system maintains multiple versions of data items. Readers can access older versions while writers are modifying the latest version, reducing contention between reads and writes.

Concurrency Control is essential for achieving the **Isolation** property of [[ACID Properties]], ensuring that concurrent transactions do not negatively impact each other's execution or the consistency of the database.

---
**Related Notes:**
* [[Database Concepts]]
* [[DBMS]]
* [[File Oriented Approach]]
* [[Transaction Management]]
* [[ACID Properties]]
* [[ACID Properties#3. Isolation|Isolation Level]] (Create if needed, or link to Isolation section in ACID Properties)