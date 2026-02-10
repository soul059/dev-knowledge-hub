1.  **Start with the Foundation:**
    * [[Database Concepts]]: This file gives you the essential high-level overview – what a database is, what a DBMS is, why we use them, data independence, and introduces data models and ACID properties.
    * [[File Oriented Approach]]: Read this right after or alongside the "Purpose of Database Systems" in `Database Concepts.md`. It details the problems that databases solved, providing context for why DBMS capabilities are necessary.
    * [[DBMS]]: This delves into the software system itself – its role, key functions, components, and types (like RDBMS vs. NoSQL).

2.  **Understand Data Organization:**
    * [[Data Models]]: This note provides an overview of different approaches to structuring data conceptually, logically, and physically, introducing terms like Relational Model and E-R Model.
    * [[View of Data (Data Abstraction)]]: Explain data abstraction and the three-level architecture (external, conceptual, internal), directly linking to [[Database Concepts#Data Independence|Data Independence]] and how the DBMS hides complexity.

3.  **Focus on the Relational Model:**
    * [[Relational Model]]: This is crucial. Understand the core concepts: relation, tuple, attribute, domain, schema, instance, and the different types of keys (Primary, Foreign, etc.). Most modern databases are based on this.
    * [[Entity-Relationship Model]]: Read this to understand the primary *conceptual* data model used to design Relational databases. It's the standard way to model real-world entities and relationships before translating them to tables.
    * [[Database Design]]: This note outlines the overall process, showing how you move from requirements -%3E conceptual (E-R) -> logical (Relational, Normalization) -> physical design (Indexing). It puts the E-R and Relational Model notes into context.
    * [[Database Normalization]]: Dive into this after understanding the Relational Model and the logical design phase. It explains how to structure your tables to reduce redundancy and improve integrity, going through the Normal Forms.

4.  **Learn the Standard Language: SQL:**
    * [[SQL]]: This is your main entry point into the language. It defines SQL, its purpose, and the different categories of commands (DDL, DML, DQL, DCL, TCL), acting as an index for the specific SQL notes.
    * [[SQL Data Types]]: Understand the basic types of data you can store in SQL columns.
    * [[SQL Constraints]]: Learn how to define rules on your data (like NOT NULL, UNIQUE, Primary Key, Foreign Key, Check) – this is fundamental for [[Data Integrity]] and defined using [[SQL DDL]].

5.  **Master Querying and Manipulation (The Core of SQL):**
    * [[SQL DDL]]: How to create and modify the structure (schema) of your database tables and other objects.
    * [[SQL DML]]: Learn the commands for adding, modifying, and deleting data (`INSERT`, `UPDATE`, `DELETE`).
    * [[SQL DQL]]: This is arguably the most used part – learn the `SELECT` statement and its core clauses (`FROM`, `WHERE`, `GROUP BY`, `HAVING`, `ORDER BY`).
    * [[SQL Joins]]: Essential for combining data from multiple related tables in a relational database.
    * [[SQL Aggregate Functions and Grouping]]: How to perform calculations and summaries on groups of data.
    * [[SQL NULL Values]]: Crucial for understanding how missing data is handled in SQL, as it affects comparisons, joins, and aggregates.
    * [[SQL Subqueries]]: Learn how to use nested queries for more complex filtering and data retrieval.
    * [[SQL Views]]: Understand how to create and use virtual tables based on `SELECT` queries.

6.  **Explore Internal Mechanisms & Advanced Topics:**
    * [[Transaction Management]]: Learn how the DBMS handles units of work to ensure reliability.
    * [[ACID Properties]]: Understand the four core guarantees provided by transaction management (Atomicity, Consistency, Isolation, Durability).
    * [[Database Recovery]]: How the DBMS restores data after failures (related to Durability).
    * [[Concurrency Control]]: How the DBMS manages simultaneous access from multiple users (related to Isolation).
    * [[Storage Management]]: Delve deeper into how the DBMS interacts with physical storage (related to physical design).
    * [[Database Indexing]]: Learn how indexes speed up queries (part of physical design and performance tuning).
    * [[Query Optimization]] (if you create this note): Understand how the DBMS chooses the best way to execute a query, often using indexes.
    * [[Data Integrity]]: A broader concept, now you have the context of constraints, transactions, and design.
    * [[Data Consistency]]: The desired state, contrasted with inconsistency, achieved through various DBMS features.
    * [[Data Inconsistency]]: The problem caused by redundancy, solved by DBMS.
    * [[Data Redundancy]]: The duplication problem that normalization helps solve.
    * [[Data Isolation]]: The scattering problem solved by centralized DBMS.
    * [[Data Security]]: How the DBMS protects data.
    * [[SQL DCL]]: The SQL commands used to implement data security.
    * [[Database Administrator (DBA)]]: The role responsible for many of these management tasks (DDL, DCL, Security, Performance, Recovery).
    * [[Database Users]]: Understand the different types of people who interact with the system.
    * [[Application Architectures]]: How applications connect to the database.

7.  **Formal Theory and Alternatives:**
    * [[Relational Algebra]]: Explore the procedural, theoretical foundation of the Relational Model.
    * [[Relational Calculus]]: Explore the declarative, theoretical foundation of the Relational Model. (These can be read after understanding the Relational Model and SQL).
    * [[NoSQL]]: Read about alternative database approaches that deviate from the Relational Model, understanding *why* they exist based on the limitations you learned about RDBMS.
