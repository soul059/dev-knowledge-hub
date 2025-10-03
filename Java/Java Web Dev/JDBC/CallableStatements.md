# JDBC CallableStatements

A `CallableStatement` is a Java Database Connectivity (JDBC) interface used to execute stored procedures and functions within a database. Stored procedures are precompiled groups of SQL statements and procedural logic stored in the database, which can improve performance by reducing network traffic and allowing for optimized, reusable code.

The `CallableStatement` interface extends `PreparedStatement`, so it inherits the functionality for handling input (IN) parameters. However, it adds specific methods to handle output (OUT) and input/output (INOUT) parameters, as well as to retrieve return values from stored procedures.

### Key Steps for Using `CallableStatement`:

1.  **Create a `CallableStatement` Object**:
    You create a `CallableStatement` by calling the `prepareCall()` method on an active `Connection` object. The method takes a string containing the JDBC escape syntax for a stored procedure call.
    *   **Syntax without a return value**: `{call procedure_name(?, ?, ...)}`
    *   **Syntax with a return value**: `{? = call function_name(?, ?, ...)}`

    ```java
    // For a procedure
    CallableStatement cstmt = conn.prepareCall("{call my_procedure(?, ?)}");

    // For a function with a return value
    CallableStatement cstmtFunc = conn.prepareCall("{? = call my_function(?)}");
    ```

2.  **Set IN and INOUT Parameters**:
    For input (IN) and input/output (INOUT) parameters, you use the `setXXX()` methods inherited from `PreparedStatement` (e.g., `setString()`, `setInt()`) to pass values to the stored procedure. Parameters are referenced by their 1-based index or by name if the driver supports it.

    ```java
    cstmt.setString(1, "some_value"); // Sets the first parameter
    cstmt.setInt(2, 123);          // Sets the second parameter
    ```

3.  **Register OUT and INOUT Parameters**:
    For every OUT and INOUT parameter, you must register its SQL type using the `registerOutParameter()` method before executing the statement. This informs the JDBC driver what data type to expect back from the database.

    ```java
    // Register the second parameter as an OUT parameter of type INTEGER
    cstmt.registerOutParameter(2, java.sql.Types.INTEGER);

    // For a function, register the return value as an OUT parameter
    cstmtFunc.registerOutParameter(1, java.sql.Types.VARCHAR);
    ```

4.  **Execute the Statement**:
    You execute the call using one of the execute methods:
    *   `executeQuery()`: Use if the procedure returns a single `ResultSet`.
    *   `executeUpdate()`: Use if the procedure performs DML operations (INSERT, UPDATE, DELETE) and returns an update count.
    *   `execute()`: A general-purpose method that can handle multiple `ResultSet` objects, update counts, or a combination. It returns `true` if the first result is a `ResultSet` and `false` if it's an update count.

    ```java
    boolean hasResultSet = cstmt.execute();
    ```

5.  **Retrieve Results and OUT Parameters**:
    *   If the procedure returns `ResultSet` objects, you can retrieve them using `getResultSet()` and `getMoreResults()`.
    *   After processing any result sets, you can retrieve the values of the OUT parameters using the `getXXX()` methods (e.g., `getInt()`, `getString()`).

    ```java
    // Process ResultSet if one exists
    if (hasResultSet) {
        try (ResultSet rs = cstmt.getResultSet()) {
            while (rs.next()) {
                // process rows
            }
        }
    }

    // Retrieve the value of the OUT parameter
    int outputValue = cstmt.getInt(2); // Get value from the second parameter
    ```

### Example

Here is a complete example of calling a stored procedure that takes one IN parameter and returns a value through an OUT parameter.

**Stored Procedure (MySQL Syntax):**

```sql
DELIMITER $$
CREATE PROCEDURE get_employee_name (IN p_employee_id INT, OUT p_employee_name VARCHAR(255))
BEGIN
    SELECT first_name INTO p_employee_name
    FROM employees
    WHERE id = p_employee_id;
END$$
DELIMITER ;
```

**Java JDBC Code:**

```java
import java.sql.CallableStatement;
import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.SQLException;
import java.sql.Types;

public class CallableStatementExample {
    public static void main(String[] args) {
        String url = "jdbc:mysql://localhost:3306/your_database";
        String user = "your_username";
        String password = "your_password";

        String sql = "{call get_employee_name(?, ?)}";

        try (Connection conn = DriverManager.getConnection(url, user, password);
             CallableStatement cstmt = conn.prepareCall(sql)) {

            int employeeId = 101;

            // Set the IN parameter
            cstmt.setInt(1, employeeId);

            // Register the OUT parameter
            cstmt.registerOutParameter(2, Types.VARCHAR);

            System.out.println("Executing stored procedure...");
            cstmt.execute();

            // Retrieve the OUT parameter value
            String employeeName = cstmt.getString(2);

            System.out.println("Employee Name for ID " + employeeId + " is: " + employeeName);

        } catch (SQLException e) {
            e.printStackTrace();
        }
    }
}
```
