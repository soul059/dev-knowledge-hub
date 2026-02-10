---
tags:
  - database
  - data-abstraction
  - database-architecture
  - schema
---

# View of Data (Data Abstraction)

In [[DBMS|Database Management Systems]], the concept of **View of Data** refers to providing different levels of abstraction to hide the complexity of how data is physically stored and managed. This layered approach is crucial for achieving [[Database Concepts#Data Independence|Data Independence]] and simplifying interaction for various users and applications.

The ANSI-SPARC architecture (a standard model for database systems) defines three levels of abstraction:

1.  **External Level (View Level):**
    * **Description:** The highest level of abstraction. Describes the part of the database that a particular user or group of users is interested in. It hides details of the rest of the database and can customize the data's appearance.
    * **Perspective:** "What the *user* sees."
    * **Mechanism:** Defined using **views** (virtual tables) based on [[SQL DQL|SELECT]] queries on the conceptual/logical schema (See [[SQL Views]]).
    * **Characteristics:**
        * Multiple external schemas can exist for the same conceptual schema.
        * Hides sensitive data or simplifies complex structures.
        * Applications and end users interact with this level.

2.  **Conceptual Level (Logical Level):**
    * **Description:** Describes the entire database structure for the community of users, independent of the physical storage details. It defines the data entities, their attributes, relationships, and [[SQL Constraints|integrity constraints]].
    * **Perspective:** "What the *database contains* and *how it's structured* (logically)."
    * **Mechanism:** Defined using a [[Data Models#Logical Data Models|Logical Data Model]] (e.g., [[Relational Model]]). This is the database's blueprint.
    * **Characteristics:**
        * Represents the global view of the data.
        * Defined by the [[Database Administrator (DBA)]].
        * Provides a stable interface for application programmers.

3.  **Internal Level (Physical Level):**
    * **Description:** The lowest level of abstraction. Describes how the data is physically stored on the storage devices. Details the file structures, indexing methods, data access paths, and physical storage allocation.
    * **Perspective:** "How the *data is physically stored*."
    * **Mechanism:** Managed by the [[Storage Management]] component of the [[DBMS]].
    * **Characteristics:**
        * Closest to the physical hardware.
        * Concerned with performance optimization at the storage level.
        * Typically transparent to most users and application programmers.

## Mapping Between Levels

The [[DBMS]] is responsible for mapping requests and data between these levels:

* **External/Conceptual Mapping:** Translates requests made in terms of an external schema into requests in terms of the conceptual schema, and translates the results from the conceptual schema back to the external view.
* **Conceptual/Internal Mapping:** Translates requests in terms of the conceptual schema into requests in terms of the internal schema, and translates the results from the internal schema back to the conceptual level.

These mappings are crucial for providing [[Database Concepts#Data Independence|Data Independence]]. Changes at a lower level require updating only the mapping to the next higher level, not the schema definition at that higher level.

The multi-level view of data is a fundamental concept in database systems that enables flexibility, security, and data independence, making databases easier to manage and use for diverse purposes.

---
**Related Notes:**
* [[Database Concepts]]
* [[DBMS]]
* [[Data Models]]
* [[Database Concepts#Data Independence|Data Independence]]
* [[SQL Views]]
* [[Database Administrator (DBA)]]
* [[Storage Management]]