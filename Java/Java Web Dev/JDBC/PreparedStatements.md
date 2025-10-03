# JDBC PreparedStatements

A `PreparedStatement` is a feature of the Java Database Connectivity (JDBC) API that allows for the execution of precompiled SQL statements in a secure and efficient manner. It is an object that represents a precompiled SQL statement which can be executed multiple times.

### Key Advantages of `PreparedStatement`

`PreparedStatement` offers several significant advantages over a basic `Statement` object:

*   **Prevents SQL Injection:** It helps prevent SQL injection attacks by automatically escaping special characters in the input parameters. User-supplied data is treated as content and not as part of the SQL command itself.
*   **Improved Performance:** The SQL statement is precompiled by the database management system (DBMS) the first time it is executed. For subsequent executions, the database can just run the statement without having to compile it again, which often results in faster execution, especially for queries that are run multiple times.
*   **Enhanced Readability and Convenience:** It provides a clear separation between the SQL code and the parameters being passed, which improves the overall readability of the code. It also allows for the use of parameter placeholders (`?`) which are then programmatically set using setter methods like `setString()` and `setInt()`.
*   **Support for Various Data Types:** It simplifies the process of inserting non-standard data types, such as `Timestamp` or `InputStream`, into SQL statements.

### How to Use a `PreparedStatement`

Here is a step-by-step guide on how to use a `PreparedStatement`:

1.  **Create a `PreparedStatement` Object:** You create a `PreparedStatement` object by calling the `prepareStatement()` method on a `Connection` object, passing the SQL query as an argument. The query can contain one or more `?` characters as placeholders for parameters.
2.  **Supply Values for Parameters:** Before executing the statement, you must supply values for the placeholders using the appropriate `set<Type>()` methods (e.g., `setString()`, `setInt()`, `setDate()`). The first argument to these methods is the one-based index of the parameter, and the second is the value itself.
3.  **Execute the Statement:** You can execute the prepared statement by calling one of its execute methods:
    *   `executeQuery()`: For `SELECT` statements that return a `ResultSet`.
    *   `executeUpdate()`: For `INSERT`, `UPDATE`, or `DELETE` statements, which returns the number of rows affected.
    *   `execute()`: Can be used for any type of SQL statement.

### Code Example

Here is a basic example of using a `PreparedStatement` to insert data into a table:

```java
import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.PreparedStatement;
import java.sql.SQLException;

public class PreparedStatementExample {

    public static void main(String[] args) {
        String url = "jdbc:mysql://localhost:3306/your_database";
        String username = "your_username";
        String password = "your_password";

        String sql = "INSERT INTO customer (cust_id, cust_name) VALUES (?, ?)";

        try (Connection con = DriverManager.getConnection(url, username, password);
             PreparedStatement pstmt = con.prepareStatement(sql)) {

            // Set the values for the placeholders
            pstmt.setInt(1, 1);
            pstmt.setString(2, "John Doe");

            // Execute the SQL statement
            int rowsAffected = pstmt.executeUpdate();
            System.out.println(rowsAffected + " row(s) affected.");

        } catch (SQLException e) {
            e.printStackTrace();
        }
    }
}
```
