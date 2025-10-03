# JDBC Statements

A JDBC `Statement` is an interface used to execute a static SQL query against a database.

There are three main types of statements in JDBC:

1.  **`Statement`**: Used for simple, static SQL queries. It is vulnerable to SQL injection and should generally be avoided if the query involves user input.

2.  **`PreparedStatement`**: A pre-compiled SQL statement. It is the preferred method for executing queries, especially those with parameters, as it prevents SQL injection and can offer better performance.

3.  **`CallableStatement`**: Used for executing database stored procedures.

### `Statement`

The `Statement` interface is used for executing simple SQL queries with no parameters. You would use it for `CREATE TABLE`, `DROP TABLE`, or simple `SELECT` statements.

**Methods:**

*   `executeQuery(String sql)`: Executes a `SELECT` query and returns a `ResultSet` object.
*   `executeUpdate(String sql)`: Executes an `INSERT`, `UPDATE`, or `DELETE` statement and returns the number of affected rows.
*   `execute(String sql)`: Can execute any kind of SQL statement and returns a boolean indicating whether the result is a `ResultSet`.

**Example of `Statement` (for static queries):**

```java
import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.ResultSet;
import java.sql.Statement;
import java.sql.SQLException;

public class StatementExample {
    public static void main(String[] args) {
        String url = "jdbc:mysql://localhost:3306/yourdatabase";
        String user = "yourusername";
        String password = "yourpassword";
        String sql = "SELECT id, name, email FROM users";

        try (Connection conn = DriverManager.getConnection(url, user, password);
             Statement stmt = conn.createStatement();
             ResultSet rs = stmt.executeQuery(sql)) {

            while (rs.next()) {
                System.out.println("ID: " + rs.getInt("id") +
                                   ", Name: " + rs.getString("name") +
                                   ", Email: " + rs.getString("email"));
            }
        } catch (SQLException e) {
            e.printStackTrace();
        }
    }
}
```
