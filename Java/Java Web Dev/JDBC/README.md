# Java Database Connectivity (JDBC)

JDBC is a Java API that defines how a client can access a database. It provides a standard way to connect to and interact with relational databases using SQL.

## JDBC Architecture

The JDBC architecture consists of two main layers:

*   **JDBC API:** Provides the interfaces and classes for Java applications to interact with databases. Key interfaces include `Driver`, `Connection`, `Statement`, `PreparedStatement`, `CallableStatement`, and `ResultSet`.
*   **JDBC Driver Manager:** The `DriverManager` class is the backbone of the JDBC architecture. It is responsible for managing a list of database drivers and matching connection requests from Java applications with the appropriate driver.
*   **JDBC Driver:** A database-specific implementation of the JDBC API. There are four types of JDBC drivers:
    1.  **Type 1 (JDBC-ODBC Bridge):** Translates JDBC calls into ODBC calls. This type of driver is no longer recommended.
    2.  **Type 2 (Native-API Driver):** Uses a native client library to communicate with the database.
    3.  **Type 3 (Network-Protocol Driver):** Uses a middleware server to translate JDBC calls into a database-specific protocol.
    4.  **Type 4 (Thin Driver):** A pure Java driver that communicates directly with the database using its native protocol. This is the most common type of driver.

## Steps to Connect to a Database using JDBC

1.  **Load the JDBC driver:** `Class.forName("com.mysql.cj.jdbc.Driver");`
2.  **Establish a connection:** `Connection con = DriverManager.getConnection(url, user, password);`
3.  **Create a statement:** `Statement stmt = con.createStatement();`
4.  **Execute a query:** `ResultSet rs = stmt.executeQuery("SELECT * FROM users");`
5.  **Process the result set:** `while (rs.next()) { ... }`
6.  **Close the resources:** `rs.close(); stmt.close(); con.close();` (It is crucial to close all resources in a `finally` block or use a try-with-resources statement).

## JDBC Example

This example shows how to connect to a MySQL database and retrieve a list of users.

**1. `pom.xml` (add MySQL driver dependency):**

```xml
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <version>8.0.28</version>
</dependency>
```

**2. `UserDAO.java` (Data Access Object):**

```java
package com.example.dao;

import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.PreparedStatement;
import java.sql.ResultSet;
import java.sql.SQLException;
import java.util.ArrayList;
import java.util.List;

import com.example.model.User;

public class UserDAO {
    private String jdbcURL = "jdbc:mysql://localhost:3306/mydb";
    private String jdbcUsername = "root";
    private String jdbcPassword = "password";

    private static final String SELECT_ALL_USERS = "SELECT * FROM users";

    protected Connection getConnection() {
        Connection connection = null;
        try {
            Class.forName("com.mysql.cj.jdbc.Driver");
            connection = DriverManager.getConnection(jdbcURL, jdbcUsername, jdbcPassword);
        } catch (SQLException | ClassNotFoundException e) {
            e.printStackTrace();
        }
        return connection;
    }

    public List<User> selectAllUsers() {
        List<User> users = new ArrayList<>();
        try (Connection connection = getConnection();
             PreparedStatement preparedStatement = connection.prepareStatement(SELECT_ALL_USERS);) {
            ResultSet rs = preparedStatement.executeQuery();

            while (rs.next()) {
                int id = rs.getInt("id");
                String name = rs.getString("name");
                String email = rs.getString("email");
                users.add(new User(id, name, email));
            }
        } catch (SQLException e) {
            e.printStackTrace();
        }
        return users;
    }
}
```

## `PreparedStatement`

The `PreparedStatement` interface is a subinterface of `Statement`. It is used to execute parameterized queries. Using `PreparedStatement` is highly recommended for several reasons:

*   **Security:** It helps to prevent SQL injection attacks by automatically escaping special characters in the parameters.
*   **Performance:** The database can precompile the SQL query, which can lead to better performance, especially for queries that are executed multiple times.
*   **Readability:** It makes the code more readable by separating the SQL query from the parameters.

**Example with `PreparedStatement`:**

```java
String sql = "INSERT INTO users (name, email) VALUES (?, ?)";
PreparedStatement statement = connection.prepareStatement(sql);
statement.setString(1, "John Doe");
statement.setString(2, "john.doe@example.com");
int rowsInserted = statement.executeUpdate();
```

## Transactions

A transaction is a sequence of operations that are performed as a single logical unit of work. In JDBC, you can manage transactions using the `Connection` object.

*   `setAutoCommit(false)`: Disables auto-commit mode, allowing you to group multiple statements into a single transaction.
*   `commit()`: Commits the transaction, making the changes permanent.
*   `rollback()`: Rolls back the transaction, undoing any changes.

## Batch Processing

Batch processing allows you to execute a group of SQL statements at once. This can significantly improve performance by reducing the number of round trips to the database.

```java
String sql = "INSERT INTO users (name, email) VALUES (?, ?)";
PreparedStatement statement = connection.prepareStatement(sql);

statement.setString(1, "User 1");
statement.setString(2, "user1@example.com");
statement.addBatch();

statement.setString(1, "User 2");
statement.setString(2, "user2@example.com");
statement.addBatch();

int[] updateCounts = statement.executeBatch();
```

## Connection Pooling

Creating a new database connection for each request is expensive. A connection pool is a cache of database connections that can be reused by multiple clients. This can dramatically improve the performance of your application.

Popular connection pooling libraries include:

*   HikariCP
*   c3p0
*   DBCP