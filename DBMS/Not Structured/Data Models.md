---
tags:
  - database
  - data-modeling
  - database-design
  - conceptual-model
  - logical-model
  - physical-model
---

# Data Models

A **Data Model** is a conceptual tool used to describe the structure of a [[Database Concepts|database]], the relationships between data, and the constraints that apply to the data. It provides a way to represent the data requirements of an enterprise at different levels of abstraction.

Data models are essential for [[Database Design|database design]] as they allow designers to visualize and communicate the database structure before implementing it in a [[DBMS]].

## Levels of Data Models (Abstraction)

Data models can be categorized based on the level of abstraction they represent:

1.  **Conceptual Data Models:**
    * **Purpose:** Describe data at a very high level, close to how users perceive the data. Focuses on describing the main data entities, their attributes, and the relationships between them, independent of a specific [[DBMS]] or implementation details.
    * **Representation:** Often graphical, easy for non-technical users to understand.
    * **Example:** [[Entity-Relationship Model|Entity-Relationship (E-R) Model]].

2.  **Logical Data Models:**
    * **Purpose:** Describe data at a level that can be implemented by a specific type of [[DBMS]], but still independent of a particular vendor or physical storage details. Maps the conceptual model into a structure that the DBMS can understand.
    * **Representation:** Uses structures like tables (relations), records, data segments. Defines entities, attributes, and relationships in a structured format.
    * **Examples:** [[Relational Model]], Network Model, Hierarchical Model, Object-Oriented Model.

3.  **Physical Data Models:**
    * **Purpose:** Describe data at the lowest level, detailing *how* data is physically stored on storage media. Includes specifics about file structures, indexing methods, data access paths, and storage allocation.
    * **Representation:** Specific to a particular [[DBMS]] vendor and operating system.
    * **Example:** A model showing tables, columns, data types, Primary/Foreign keys, indexes, partitioning strategy, storage parameters within a specific RDBMS (e.g., Oracle, SQL Server).

## Types of Logical Data Models (Evolution)

Historically, several logical data models have been developed, reflecting the evolution of [[DBMS]] technology:

* **Hierarchical Model:** (Prominent in the 1960s-70s) Data organized in a tree-like structure with parent-child relationships. Each child has only one parent. Efficient for one-to-many relationships but rigid.
* **Network Model:** (Developed from the Hierarchical Model) Data organized in a graph structure, allowing records to have multiple parents and children. More flexible than hierarchical but complex to manage.
* **[[Relational Model]]:** (Proposed by E.F. Codd in 1970) Data organized into two-dimensional tables (relations). Relationships are represented by common attribute values (Foreign Keys). Dominant model due to its simplicity, theoretical foundation, and support for powerful query languages like [[SQL]].
* **Object-Oriented Model:** (Emerged in the 1980s) Data stored as objects, incorporating concepts from object-oriented programming (encapsulation, inheritance, polymorphism).
* **Semi-structured Data Models:** (Popularized by the web) Data has a flexible structure; format may vary even within the same type of entity (e.g., XML, JSON). Leads to [[NoSQL]] databases like Document databases.
* **[[NoSQL]] Models:** (Modern era) A broad category of non-relational models designed for specific needs like massive scalability, handling unstructured data, or high performance for certain workloads. Includes Key-Value stores, Document databases, Column-Family stores, and Graph databases.

Choosing the right data model during the [[Database Design|design process]] is crucial, starting with a conceptual model and then mapping it to a logical model suitable for the chosen [[DBMS]].

---
**Related Notes:**
* [[Database Concepts]]
* [[DBMS]]
* [[Database Design]]
* [[View of Data (Data Abstraction)]]
* [[Entity-Relationship Model]]
* [[Relational Model]]
* [[Relational Algebra]]
* [[NoSQL]] (Create if needed)
* [[Collage/DBMS/Introduction#History and Evolution of Database Systems|Database Evolution]] (Link to or create if this note covers the history of models)