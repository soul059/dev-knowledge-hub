---
tags:
  - database
  - security
  - authorization
  - access-control
---

# Data Security

**Data Security** in databases refers to the measures taken to protect the data from unauthorized access, modification, disclosure, or destruction. Ensuring data security is a fundamental responsibility of a [[DBMS|Database Management System]] and the [[Database Administrator (DBA)]].

## Security Issues in File Systems

In the [[File Oriented Approach]], data security was often limited:

* **File-Level Access:** Security was typically managed at the operating system level, granting or denying access to entire files.
* **Lack of Granularity:** It was difficult or impossible to define fine-grained access controls, such as allowing a user to see only specific rows or columns within a file, or to restrict update privileges to certain fields.
* **Application-Specific Logic:** Security checks were often embedded within each application program, leading to potential inconsistencies or vulnerabilities if not implemented correctly in every program.

## How DBMS Provides Data Security

[[DBMS|Database Management Systems]] offer much more sophisticated and centralized data security mechanisms:

* **Authentication:** Verifying the identity of a user attempting to access the database (e.g., username and password).
* **Authorization (Access Control):** Defining and enforcing rules about what authenticated users are allowed to do. This is typically managed using privileges and permissions.
    * **Granting Privileges:** Using commands like [[SQL|`GRANT`]] ([[SQL DCL]]) to give users or roles specific rights (e.g., `SELECT` on a table, `UPDATE` on specific columns, `DELETE` on specific rows, `EXECUTE` on a stored procedure).
    * **Revoking Privileges:** Using commands like [[SQL|`REVOKE`]] ([[SQL DCL]]) to remove previously granted rights.
    * **Granularity:** Permissions can be defined at various levels: database, schema, table, view, column, or even row-level in advanced systems.
* **[[SQL Views|Views]]:** Views can be used to restrict the data that a user can see, effectively hiding sensitive columns or rows by defining the view based on a limited [[SQL DQL|SELECT]] query. Permissions are then granted on the view, not the underlying table.
* **Auditing:** Recording database activities (who accessed what, when, and what operations were performed) to monitor for suspicious behavior and provide accountability.
* **Encryption:** Storing data in an encrypted format (data at rest) or encrypting data during transmission (data in transit) to protect it if the underlying files or network are compromised.
* **Roles:** Grouping privileges into roles, which can then be assigned to users, simplifying permission management.

By centralizing security management and providing fine-grained control, a [[DBMS]] significantly enhances the protection of valuable data compared to the limited capabilities of file systems.

---
**Related Notes:**
* [[Database Concepts]]
* [[DBMS]]
* [[File Oriented Approach]]
* [[Database Administrator (DBA)]]
* [[SQL DCL]]
* [[SQL Views]]
* [[Authentication]] (Create if needed)
* [[Authorization]] (Create if needed)