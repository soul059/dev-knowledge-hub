---
tags:
  - database
  - data-quality
  - integrity
  - constraints
---

# Data Integrity

**Data Integrity** refers to the overall accuracy, completeness, consistency, and reliability of data stored in a database. Ensuring data integrity is a crucial goal of any [[DBMS|Database Management System]].

## Difficulty in File Systems

In the [[File Oriented Approach]], maintaining data integrity was challenging:

* **Logic Spread Across Applications:** Business rules and validation logic were embedded within each individual application program that accessed the data.
* **Inconsistent Enforcement:** Different programs might implement validation rules differently or might not implement them at all, leading to inconsistent data quality across the system.
* **Maintenance Burden:** Changing or adding new integrity rules required modifying and testing multiple application programs.

## How DBMS Ensures Data Integrity

[[DBMS|Database Management Systems]] provide centralized mechanisms to define and enforce [[SQL Constraints|integrity constraints]], ensuring data validity regardless of how the data is accessed or modified. Integrity constraints are rules that the data must satisfy.

Common types of integrity constraints enforced by a [[DBMS]]:

* **Domain Integrity:** Ensures that attribute values fall within the defined domain (valid set of values and data type) for that attribute. (e.g., an 'Age' column must be a positive integer; a 'Status' column must be 'Active', 'Inactive', or 'Pending'). This relies on [[SQL Data Types]].
* **Entity Integrity:** Ensures that each entity instance (row) in a table can be uniquely identified. This is enforced by the [[SQL Constraints|PRIMARY KEY]] constraint, which requires that the primary key value is unique for every row and cannot be [[SQL NULL Values|NULL]].
* **Referential Integrity:** Ensures that relationships between tables are consistent. This is enforced by [[SQL Constraints|FOREIGN KEY]] constraints. If a foreign key value exists in a table, it must either be null or match an existing primary key value in the referenced table. This prevents orphaned records.
* **Check Constraints:** Allows defining custom rules or conditions that attribute values or combinations of values must satisfy (e.g., `Salary %3E 0`, `EndDate >= StartDate`).

By defining and enforcing these rules centrally within the database schema using [[SQL DDL]], the [[DBMS]] guarantees that the integrity rules are applied consistently for all operations (inserts, updates, deletes), regardless of the application or user performing the operation. This greatly improves the trustworthiness and quality of the data.

Data integrity is closely related to [[Data Consistency]] (ensuring different copies agree) and [[Data Redundancy]] (which can cause inconsistency). A good [[Database Design]] is key to defining and enforcing these integrity rules effectively.

---
**Related Notes:**
* [[Database Concepts]]
* [[DBMS]]
* [[File Oriented Approach]]
* [[SQL Constraints]]
* [[SQL DDL]]
* [[SQL Data Types]]
* [[Data Consistency]]
* [[Data Redundancy]]
* [[Database Design]]
* [[Entity Integrity Constraint]] (Link to or create)
* [[Referential Integrity Constraint]] (Link to or create)
* [[Domain Constraint]] (Link to or create, or link to SQL Constraints/Data Types)
* [[Check Constraint]] (Link to or create, or link to SQL Constraints)