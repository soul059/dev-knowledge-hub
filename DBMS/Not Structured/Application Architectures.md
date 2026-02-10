---
tags:
  - database
  - architecture
  - application-design
  - dbms-interaction
---

# Application Architectures

The way application programs are structured to interact with a [[DBMS|Database Management System]] is referred to as the application architecture. These architectures determine how processing logic is distributed and how different parts of the system communicate. The most common models are the one-tier, two-tier, and three-tier architectures.

## 1-Tier Architecture

* **Description:** The database and the application logic (user interface and business logic) all reside on the same machine.
* **Structure:** User Interface -%3E Business Logic -> Database
* **Characteristics:**
    * Simplest architecture.
    * Typically used for single-user desktop applications.
    * Minimal separation of concerns.
* **Pros:** Easy to develop and deploy for simple, standalone applications.
* **Cons:** Poor scalability, security issues (data directly accessible on the user's machine), limited data sharing, difficult to maintain and update (requires updating every client).
* **Example:** A personal database application (like an older desktop contact manager) where the database file and the program are on the same computer.

## 2-Tier Architecture (Client-Server)

* **Description:** The application is split into two main parts: a client (which runs on the user's machine and handles the user interface) and a server (which runs the [[DBMS]] and manages the database).
* **Structure:** User Interface / Business Logic (Client) %3C-> DBMS (Server)
* **Characteristics:**
    * The client application contains both the user interface and often a significant portion of the business logic.
    * The client communicates directly with the database server using database drivers or APIs (like ODBC, JDBC).
    * The server primarily handles database operations (query processing, transaction management, storage).
* **Pros:** Better performance than 1-tier for shared data, improved data consistency due to centralized database management.
* **Cons:** Limited scalability compared to 3-tier (server can become a bottleneck), "fat client" problem (business logic duplicated across clients, difficult to update), security can be a concern if business logic isn't fully on the server.
* **Example:** Traditional desktop applications that connect directly to a central database server on a local network.

## 3-Tier Architecture

* **Description:** The application is divided into three logical tiers: the Presentation Tier, the Application Tier, and the Data Tier.
* **Structure:** Presentation Tier <-> Application Tier <-> Data Tier
    * **Presentation Tier (Client):** The user interface (e.g., web browser, mobile app, desktop GUI). It sends user requests to the Application Tier and displays responses. Contains minimal or no business logic.
    * **Application Tier (Middleware / Business Logic):** Contains the core application logic, processes user requests, performs calculations, coordinates workflows, and interacts with the Data Tier. Acts as a intermediary.
    * **Data Tier (Database Server):** The [[DBMS]] and the database. Stores and manages the data, responding to requests from the Application Tier.
* **Characteristics:**
    * Clear separation of concerns.
    * Scalability is improved by distributing load across tiers (especially scaling the Application Tier).
    * Business logic is centralized on the Application Tier.
* **Pros:** High scalability, improved security (clients don't directly access the database), easier maintenance and updates (business logic changes only need deployment on the application server), improved code reusability.
* **Cons:** More complex to develop and manage than 2-tier.
* **Example:** Most modern web applications and large-scale enterprise systems.

Choosing the appropriate architecture depends on the application's requirements regarding scalability, security, complexity, and development effort. The 3-tier architecture is dominant for web and enterprise applications due to its significant advantages.

---
**Related Notes:**
* [[Database Concepts]]
* [[DBMS]]
* [[Database Users]]
* [[SQL]]