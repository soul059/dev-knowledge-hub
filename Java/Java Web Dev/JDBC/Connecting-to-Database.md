# Connecting to a Database

This example demonstrates how to connect to a database using JDBC. This example uses a MySQL database.

First, you need to have the MySQL JDBC driver in your project's classpath.

This example will cover the basic steps for a JDBC connection:

1.  **Load the driver**: This is often done automatically in modern JDBC versions (4.0+).
2.  **Establish a connection**: Use the `DriverManager` class to get a `Connection` object.
3.  **Create a statement**: Use the `Connection` object to create a `Statement` or `PreparedStatement` object.
4.  **Execute a query**: Run your SQL query.
5.  **Process the result set**: If your query returns data, you can iterate through the `ResultSet` object.
6.  **Close the resources**: It is crucial to close the `ResultSet`, `Statement`, and `Connection` to free up resources.

```java
import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.ResultSet;
import java.sql.SQLException;
import java.sql.Statement;

public class JdbcConnectionExample {

    // Replace with your database connection details
    private static final String DB_URL = "jdbc:mysql://localhost:3306/your_database";
    private static final String USER = "your_username";
    private static final String PASS = "your_password";

    public static void main(String[] args) {
        // Using try-with-resources to ensure all resources are closed automatically
        try (
            // 1. & 2. Establish the connection
            Connection conn = DriverManager.getConnection(DB_URL, USER, PASS);
            // 3. Create a statement
            Statement stmt = conn.createStatement()
        ) {
            // 4. Execute a query
            String sql = "SELECT id, first_name, last_name FROM employees";
            try (ResultSet rs = stmt.executeQuery(sql)) {
                // 5. Process the result set
                while (rs.next()) {
                    // Retrieve by column name
                    int id = rs.getInt("id");
                    String firstName = rs.getString("first_name");
                    String lastName = rs.getString("last_name");

                    // Display values
                    System.out.print("ID: " + id);
                    System.out.print(", First Name: " + firstName);
                    System.out.println(", Last Name: " + lastName);
                }
            }
        } catch (SQLException e) {
            e.printStackTrace();
        }
    }
}
```

**Important Notes:**

*   **Dependencies**: Before running this code, you need to add the appropriate JDBC driver for your database to your project's classpath. For MySQL, you would include the MySQL Connector/J JAR file.
*   **Credentials**: Never hardcode your database credentials directly in production code. It is better to use a configuration file or environment variables to store this sensitive information.
*   **`try-with-resources`**: This example uses a `try-with-resources` statement, which automatically handles the closing of `Connection`, `Statement`, and `ResultSet` objects. This is the recommended approach to prevent resource leaks.
*   **`PreparedStatement`**: For queries that you execute multiple times, or for queries that take user input, it is highly recommended to use `PreparedStatement` instead of `Statement` to prevent SQL injection attacks.
