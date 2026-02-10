---
tags:
  - database
  - database-basics
  - dbms
  - acid
  - data-models
aliases:
  - Fundamentals of Databases
  - Database Principles
---

# Database Concepts

This note covers fundamental concepts related to databases and [[DBMS|Database Management Systems]]. It serves as an introduction to the world of organized data storage and management.

## What is a Database?

At its most basic, a database is a structured collection of data. It's designed for efficient storage, retrieval, and management of information. Think of it as an organized container for data.

* **Data:** Refers to raw facts and figures ([[Data, Information, Knowledge#Data|Introduction#Data]]).
* **Information:** Data that has been processed, organized, and given context ([[Data, Information, Knowledge#Information|Introduction#Information]]). Databases primarily store and manage data to be turned into information.

## What is a Database Management System (DBMS)?

A [[DBMS]] is a software system that provides a standard way to create, organize, update, retrieve, and manage data in a database. It acts as an interface between users or applications and the physical database files.

* **Role:** The DBMS handles all access to the database, ensuring data integrity, security, consistency, and controlled access for multiple users.
* **Examples:** MySQL, PostgreSQL, Oracle Database, Microsoft SQL Server, MongoDB, etc.

## Purpose of Database Systems

Database systems were developed to overcome the significant drawbacks of older data management methods, particularly the [[File Oriented Approach|File System]]. Using a DBMS provides numerous advantages:

* **Reduced Data Redundancy:** Minimizing the duplication of data, which saves storage space and prevents inconsistencies.
* **Improved Data Consistency:** Ensuring that multiple copies of the same data (where redundancy is controlled) are in agreement.
* **Enhanced Data Integrity:** Enforcing rules and constraints to ensure the accuracy, validity, and reliability of data.
* **Improved Data Sharing:** Allowing multiple users and applications to access and share the same data concurrently.
* **Increased Data Security:** Providing mechanisms to control who can access or modify specific data.
* **Better Data Access:** Offering powerful query languages (like [[SQL]]) for efficient data retrieval.
* **Support for Concurrent Access:** Managing simultaneous access by multiple users without compromising data consistency.
* **Ensured Atomicity and Durability:** Guaranteeing that operations are atomic ("all or nothing") and permanent once completed, protecting against failures.

## Database Architectures

Database systems can be deployed and accessed using different architectures, defining how users/applications interact with the DBMS and data.

* **One-Tier Architecture:** The database and application reside on the same machine. Suitable for desktop applications (e.g., single-user database).
* **Two-Tier Architecture:** A client application communicates directly with the database server (e.g., traditional client-server applications).
* **Three-Tier Architecture:** Introduces an application server (middleware) between the client and the database server. Common in web applications, improving scalability and separation of concerns. (See [[Application Architectures]]).

## Data Independence

A key advantage of using a DBMS is [[Data Independence]], which refers to the ability to modify the schema at one level of the database system without affecting the schema definition at the next higher level. This is facilitated by the [[View of Data (Data Abstraction)|three-level architecture]] (Physical, Logical, View).

* **Physical Data Independence:** The ability to modify the physical schema (how data is stored on disk) without affecting the logical schema or application programs.
* **Logical Data Independence:** The ability to modify the logical schema (tables, columns, relationships) without requiring changes to application programs or external views. This is harder to achieve than physical independence.

## Data Models

Data models are fundamental concepts used to describe the structure of a database, the relationships between data, and the constraints that apply to the data. They provide a way to represent the data requirements of an enterprise.

* **Conceptual Data Models:** High-level models focusing on entities, attributes, and relationships as perceived by users (e.g., [[Entity-Relationship Model|Entity-Relationship (E-R) Model]]).
* **Logical Data Models:** Represent data in a structure that can be implemented by a DBMS, independent of the specific DBMS software (e.g., [[Relational Model|Relational Model]], Network Model, Hierarchical Model).
* **Physical Data Models:** Describe the low-level details of how data is stored on physical media.

## Database Users and Administrators

Various individuals interact with the database system in different capacities:

* **End Users:** Individuals who use the database for their work (e.g., accessing information, running reports, data entry). Can be [[Database Users#Naïve users|naïve]] (using applications) or [[Database Users#Sophisticated users|sophisticated]] (using query languages).
* **Application Programmers:** Developers who build the applications that interact with the database.
* **Database Administrator (DBA):** The person or team responsible for the overall management, maintenance, performance, security, and integrity of the database system (See [[Database Administrator (DBA)]]).

## ACID Properties

The **ACID** properties are a set of guarantees provided by a [[DBMS]] during [[SQL Transactions|transactions]] to ensure data validity despite errors, power failures, and concurrent access.

* **Atomicity:** A transaction is an indivisible unit. It either completes entirely (all changes are applied), or it doesn't happen at all (none of the changes are applied). This prevents partial updates. (Link to [[SQL Transactions]] and its example).
* **Consistency:** A transaction brings the database from one valid state to another valid state. It ensures that all defined [[SQL Constraints|integrity constraints]] are maintained before and after the transaction executes. A transaction cannot leave the database in a state that violates these rules.
* **Isolation:** Transactions execute independently of each other. The effect of multiple concurrent transactions is the same as if they had executed sequentially. This prevents issues like "dirty reads," "non-repeatable reads," and "phantom reads."
* **Durability:** Once a transaction has been successfully committed, its changes are permanent and will persist even if the system crashes or loses power. The DBMS ensures committed data is written to non-volatile storage.

## Transaction Management

The [[Transaction Management]] component of the [[DBMS]] is responsible for ensuring the ACID properties. It coordinates concurrent access and handles recovery from failures. [[SQL Transactions|SQL provides commands]] (`BEGIN`, `COMMIT`, `ROLLBACK`) to explicitly control transactions.

Understanding these core database concepts provides a solid foundation for learning specific database technologies and models like the Relational Model and SQL.

---
**Related Notes:**
* [[DBMS]]
* [[File Oriented Approach]]
* [[View of Data (Data Abstraction)]]
* [[Data Models]]
* [[Entity-Relationship Model]]
* [[Relational Model]]
* [[Relational Algebra]]
* [[Relational Calculus]]
* [[SQL]]
* [[Database Users]]
* [[Database Administrator (DBA)]]
* [[SQL Transactions]]
* [[Application Architectures]]