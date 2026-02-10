---
tags:
  - database
  - recovery
  - durability
  - transaction-management
  - fault-tolerance
---

# Database Recovery

**Database Recovery** is the process of restoring the database to a consistent state after a failure. It is a critical function of a [[DBMS|Database Management System]], ensuring that data is durable and reliable. The goal is to recover from failures in a way that preserves the effects of committed [[SQL Transactions|transactions]] while undoing the effects of any transactions that were incomplete at the time of the failure.

Database Recovery is a key part of the [[Transaction Management]] component and is essential for guaranteeing the **Durability** and **Atomicity** properties of [[ACID Properties]].

## Types of Failures

Database systems must be able to recover from various types of failures:

* **Transaction Failure:** A transaction fails to complete successfully due to errors within the transaction itself (e.g., logical error, input error, data constraint violation) or deadlock.
* **System Crash:** A hardware or software error causes the DBMS software or the operating system to stop unexpectedly. Data in volatile storage (main memory) is lost, but data on non-volatile storage (disk) is generally intact.
* **Disk Failure:** A head crash or other hardware problem makes a portion of the disk storage unreadable or unusable. This is a loss of non-volatile storage.
* **Catastrophic Failure:** Non-repairable damage to the data center or site (e.g., fire, flood, earthquake), potentially destroying all data. This requires restoring from off-site backups.

The recovery procedures implemented by the DBMS depend on the type and extent of the failure.

## Core Concepts and Techniques

Database recovery relies on having redundant information stored to reconstruct the state of the database. The primary tools are the transaction log and checkpoints.

### 1. Transaction Log (Journaling)

* **Purpose:** The transaction log (also called a journal or write-ahead log) is a file that records all database modifications. It is written to **non-volatile storage** *before* the actual data changes are written to the data files.
* **Content:** For each database modification, the log typically records:
    * The identifier of the transaction performing the write.
    * The data item affected (e.g., table name, record ID).
    * The *old value* of the data item before the change (for **undo** operations).
    * The *new value* of the data item after the change (for **redo** operations).
    * Special records for transaction start, commit, and abort.
* **Importance:** The log provides the history of all changes, which is necessary to undo or redo operations during recovery. The "write-ahead" principle (log record written before data change) is crucial for [[ACID Properties#Durability|Durability]].

### 2. Checkpoints

* **Purpose:** Periodically, the DBMS performs a checkpoint. A checkpoint is a mechanism to reduce the amount of work needed during recovery after a system crash.
* **Process:** During a checkpoint, the DBMS typically:
    * Writes all modified data blocks currently in main memory to disk.
    * Writes a checkpoint record to the transaction log. This record contains information about the transactions that were active at the time of the checkpoint.
* **Importance:** After a crash, the recovery process can start examining the log from the last checkpoint record instead of from the very beginning, significantly speeding up recovery.

### 3. Rollback (Undo)

* **Purpose:** To undo the effects of a transaction that failed before committing, or to undo the effects of transactions that were active at the time of a system crash. Ensures the **Atomicity** property.
* **Mechanism:** The recovery manager reads the transaction log backward from the point of failure. For each modification record of a transaction that needs to be undone, it uses the *old value* stored in the log to restore the data item to its previous state.

### 4. Rollforward (Redo)

* **Purpose:** To reapply the effects of committed transactions that might not have had all their changes written from main memory to disk before a system crash. Ensures the **Durability** property.
* **Mechanism:** The recovery manager reads the transaction log forward, typically starting from the last checkpoint. For each modification record of a transaction that committed before the crash, it uses the *new value* stored in the log to reapply the change to the database.

## Recovery Process After a System Crash

A common recovery process after a system crash involves both Undo and Redo phases, guided by the transaction log and the last checkpoint:

1.  **Analysis Phase:** The recovery manager reads the log backward from the end to identify all transactions that were active at the time of the crash and the last checkpoint. It determines which transactions need to be undone and which ones need to be redone.
2.  **Redo Phase:** The recovery manager reads the log forward from the last checkpoint record. It reapplies all changes (using the new values) for *all* transactions recorded in the log, regardless of whether they committed or not. This brings the database state forward.
3.  **Undo Phase:** The recovery manager reads the log backward from the end. It undoes the changes (using the old values) only for those transactions that were found to be incomplete in the Analysis phase. This rolls back the uncommitted transactions.

More complex recovery techniques exist to handle specific scenarios or optimize performance, but they build upon the fundamental concepts of logging, checkpointing, Undo, and Redo.

Database recovery is a complex but essential part of the [[DBMS]] that allows businesses to rely on their data surviving unexpected failures.

---
**Related Notes:**
* [[Database Concepts]]
* [[DBMS]]
* [[Transaction Management]]
* [[ACID Properties]]
* [[SQL Transactions]]
* [[Transaction Log]] (Create if needed, or link to section)
* [[Checkpoint]] (Create if needed, or link to section)
* [[Rollback]] (Create if needed, or link to section)
* [[Rollforward]] (Create if needed, or link to section)