# Introduction to JDBC

Java Database Connectivity (JDBC) is a standard Java API that allows Java applications to interact with relational databases. It provides a consistent, database-independent way to connect to a data source, send queries and update statements, and process the results. At its core, JDBC acts as a translator between Java applications and various database management systems (DBMS), enabling developers to write code that is not tied to a specific database.

### JDBC Architecture

The JDBC architecture consists of two main layers that facilitate communication between a Java application and a database: the JDBC API and the JDBC Driver.

The architecture typically involves the following components:

*   **Java Application**: The application that requires database access.
*   **JDBC API**: A set of classes and interfaces in the `java.sql` and `javax.sql` packages that provide the methods for database interaction.
*   **JDBC Driver Manager**: A class that manages a list of database drivers and matches a connection request from the application with the appropriate driver.
*   **JDBC Driver**: A software component that enables the Java application to communicate with a specific database.

This architecture can be implemented in two models:

*   **Two-Tier Architecture**: The Java application communicates directly with the database.
*   **Three-Tier Architecture**: The application's commands are sent to a middle tier of services, which then communicates with the database.

### Core JDBC Components

The primary components of the JDBC API include:

*   **DriverManager**: Manages the database drivers.
*   **Connection**: Represents a session with a specific database and is used for executing SQL statements.
*   **Statement**: Used for executing static SQL queries.
*   **PreparedStatement**: An extension of the Statement interface, used for precompiled SQL statements that can be executed multiple times, often with different parameters.
*   **ResultSet**: Represents the set of data returned from a query, allowing the application to iterate through the rows of results.

### Basic Steps for Database Connectivity

The typical workflow for a Java application to connect to a a database using JDBC involves the following steps:

1.  **Load and Register the Driver**: The appropriate JDBC driver is loaded into the application's memory.
2.  **Establish a Connection**: A connection is opened to the database using the `DriverManager.getConnection()` method, which requires a database URL, username, and password.
3.  **Create a Statement**: A `Statement` or `PreparedStatement` object is created from the `Connection` object.
4.  **Execute the Query**: The SQL query is executed using methods like `executeQuery()` for retrieving data or `executeUpdate()` for modifying data.
5.  **Process the Results**: If the query returns data, it is retrieved from the `ResultSet` object.
6.  **Close the Connection**: To release resources, the `ResultSet`, `Statement`, and `Connection` objects are closed.
