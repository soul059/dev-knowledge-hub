---
tags:
  - database
  - data-quality
  - inconsistency
  - file-system-issues
---

# Data Inconsistency

**Data Inconsistency** occurs when different copies of the same data hold conflicting values. This is a direct consequence of uncontrolled [[Data Redundancy|data redundancy]] and the lack of centralized management, as seen in the [[File Oriented Approach]].

## How Inconsistency Arises

In file-oriented systems:

1.  Data about the same entity (e.g., a customer's address) is stored redundantly in multiple files owned by different applications.
2.  A user or application updates one copy of the data (e.g., the address in the customer file) but fails to update the other copies (e.g., the address in the sales file or the service file).
3.  The various files now have inconsistent information about the customer's address.

This problem is exacerbated in large organizations with many applications and complex interdependencies between files.

## Problems Caused by Inconsistency

Data inconsistency leads to unreliable and misleading information:

* **Inaccurate Reporting:** Reports generated from different files or different applications may produce conflicting results, making decision-making difficult.
* **Unreliable Data:** Users cannot trust the data they access, as they don't know which version is the correct one.
* **Difficulty in Reconciliation:** Resolving inconsistencies retrospectively can be a complex and time-consuming process.

## How DBMS Prevents Inconsistency

[[DBMS|Database Management Systems]] are designed to prevent data inconsistency through several mechanisms:

* **Reducing Redundancy:** By centralizing data and using [[Database Design|database design]] techniques like [[Database Normalization|normalization]], DBMS minimizes uncontrolled [[Data Redundancy]]. For non-redundant data, inconsistency is inherently avoided as there's only one copy.
* **Centralized Updates:** All updates to the data go through the [[DBMS]]. This allows the system to manage changes to redundant data in a controlled manner (if controlled redundancy exists) or, more typically, ensures the single copy of non-redundant data is updated correctly.
* **[[SQL Constraints|Integrity Constraints]]:** The [[DBMS]] allows defining and enforcing rules ([[Data Integrity]]) that data must follow. While this doesn't directly prevent all forms of inconsistency arising from logic errors in applications, it enforces fundamental rules (like [[Referential Integrity Constraint|referential integrity]]) that help maintain logical consistency between related data.
* **[[Transaction Management|Transaction Management]]:** [[SQL Transactions|Transactions]] ensure that a set of related updates (e.g., updating a redundant data item in multiple places in a controlled redundancy scenario) is treated as an atomic unit, either all succeeding or all failing, preventing inconsistencies from partial updates.

By addressing the root cause (uncontrolled redundancy) and providing centralized control and enforcement mechanisms, a [[DBMS]] ensures a much higher level of data consistency compared to traditional file systems.

---
**Related Notes:**
* [[Database Concepts]]
* [[DBMS]]
* [[File Oriented Approach]]
* [[Data Redundancy]]
* [[Data Integrity]]
* [[Database Normalization]] (Create if needed)
* [[Transaction Management]]
* [[SQL Constraints]]