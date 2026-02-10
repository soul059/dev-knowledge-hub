---
tags:
  - database
  - data-quality
  - redundancy
  - file-system-issues
---

# Data Redundancy

**Data Redundancy** refers to the unnecessary duplication of data within a database or across different storage locations. In the context of database systems, it's often discussed in contrast to the problems encountered in the [[File Oriented Approach]].

## Redundancy in File Systems

In traditional file processing systems, different applications often maintained their own separate files, even if they stored information about the same entities (like customers or products).

* **Cause:** Each application was developed independently and needed access to certain data, leading to copying that data into its own file structure. Data was not centrally managed.
* **Example:** A bank might have one file for customer account information (including customer name and address) and another file for loan information (also including customer name and address). The customer's name and address would be redundant data, stored in at least two places.

## Problems Caused by Redundancy

Uncontrolled data redundancy is undesirable because it leads to several problems:

1.  **Wasted Storage Space:** Storing the same information multiple times consumes unnecessary disk space.
2.  **Data Inconsistency:** This is the most serious problem. If data is redundant and an update is made to one copy but not all copies, the different copies will hold conflicting values. This leads to [[Data Inconsistency]], where it's impossible to know which value is correct.
3.  **Update Anomalies:** Modifying redundant data requires updating multiple locations. Failure to update all copies consistently results in inconsistency.
4.  **Deletion Anomalies:** Deleting data might inadvertently lead to losing related, non-redundant information if it was stored only with the redundant data.
5.  **Insertion Anomalies:** It might be impossible to add new information without also adding redundant data.

## How DBMS Addresses Redundancy

[[DBMS|Database Management Systems]] significantly reduce or control data redundancy compared to file systems.

* **Centralization:** Data is managed in a central repository rather than scattered in independent files.
* **[[Database Design|Database Design]] and [[Database Normalization|Normalization]]:** Relational database design methodologies, particularly normalization, aim to structure the database schema to minimize or eliminate redundant data by organizing data into well-defined tables and relationships.
* **Single Source of Truth:** For non-redundant data, there is only one place where a piece of information is stored, eliminating inconsistency issues related to that data.
* **Controlled Redundancy:** In some cases (like data warehousing or for performance optimization), some controlled redundancy might be intentionally introduced, but the [[DBMS]] provides mechanisms (like triggers or views) to help manage and keep this controlled redundancy consistent.

By reducing uncontrolled data redundancy, a [[DBMS]] improves storage efficiency and is fundamental to achieving [[Data Inconsistency|data consistency]] and maintaining [[Data Integrity]].

---
**Related Notes:**
* [[Database Concepts]]
* [[DBMS]]
* [[File Oriented Approach]]
* [[Data Inconsistency]]
* [[Database Normalization]] (Create if needed)
* [[Database Design]]