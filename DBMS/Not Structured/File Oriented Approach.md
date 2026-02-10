---
tags:
  - database
  - file-system
  - data-management-history
aliases:
  - File System Data Management
  - Traditional File Processing
---

# File Oriented Approach

The **File Oriented Approach**, also known as the **File System** approach or **traditional file processing**, was an early method for managing data using simple operating system file structures. Before the advent of sophisticated [[DBMS|Database Management Systems]], organizations commonly stored and managed their data in flat files, with specific application programs written to interact with these files.

## How it Worked

In a file-oriented system, data was typically organized into records, and related records were grouped into files. Each application program (e.g., payroll system, inventory system, sales reporting) would often have its own set of files, and the programs would directly manage reading from and writing to these files using the operating system's file management capabilities.

## Advantage Over Manual Systems

Compared to purely manual, paper-based record-keeping systems, the file-oriented approach offered some advantages:

* **Increased Speed:** Automated data processing was significantly faster than manual methods.
* **Improved Accuracy (relative):** Reduced human error in calculations and data retrieval once programmed correctly.

## Disadvantages Leading to DBMS

Despite the improvements over manual systems, the file-oriented approach suffered from severe disadvantages that became increasingly problematic as data volumes grew and multiple applications needed to access the same data. These limitations were the primary motivation for the development of [[DBMS|Database Management Systems]].

Key disadvantages include:

1.  **Data Redundancy and Inconsistency:**
    * Data could be duplicated across different files, managed by different applications.
    * This wasted storage space and, more critically, made it difficult to keep all copies in sync. If a customer's address changed, updating it in one file but not others led to **data inconsistency**.

2.  **Difficulty in Accessing Data:**
    * Retrieving information that required combining data from multiple, potentially differently structured files was complex and inefficient.
    * Each new query or reporting requirement often necessitated writing a **new, custom application program** to extract the specific data, which was time-consuming and inflexible.

3.  **Data Isolation:**
    * Data was often scattered across various files, possibly in different formats and structures, created by different programmers over time.
    * This made it difficult to integrate data across applications or departments, creating silos of information.

4.  **Concurrent Access Anomalies:**
    * File systems lacked mechanisms to coordinate multiple users or applications accessing and modifying the same file simultaneously.
    * Concurrent updates could overwrite each other's changes or read data in an inconsistent intermediate state, leading to data corruption ("lost updates").

5.  **Security Problems:**
    * Security was often file-based (granting access to an entire file) rather than data-based (granting access to specific records or fields).
    * It was difficult to implement granular security controls to restrict access to only authorized subsets of data within a file.

6.  **Integrity Problems:**
    * Ensuring data accuracy and adherence to business rules (e.g., age must be positive, valid product codes) was the responsibility of each individual application program.
    * There was no centralized mechanism to define and enforce [[SQL Constraints|integrity constraints]] consistently across all applications accessing the data. Changes to rules required modifying potentially many programs.

7.  **Atomicity Problem:**
    * Operations involving multiple steps (like a funds transfer needing both a debit and a credit) were difficult to make atomic ("all or nothing").
    * If a system failure occurred mid-operation, the database could be left in an inconsistent state without an easy way to roll back the partial changes. (See [[ACID Properties#1. Atomicity|ACID Properties - Atomicity]]).

These fundamental limitations of the file-oriented approach highlighted the need for a more robust, centralized, and abstract system for data management, leading directly to the development and widespread adoption of [[DBMS|Database Management Systems]] and concepts like the [[Relational Model]].

---
**Related Notes:**
* [[Database Concepts]]
* [[DBMS]]
* [[Database Concepts#Purpose of Database Systems|Purpose of Database Systems]]
* [[Data Redundancy]] (Create if needed)
* [[Data Inconsistency]] (Create if needed)
* [[Data Isolation]] (Create if needed)
* [[Concurrency Control]] (Link to or create)
* [[Data Security]] (Link to or create)
* [[Data Integrity]] (Link to or create)
* [[ACID Properties#1. Atomicity|Atomicity]]