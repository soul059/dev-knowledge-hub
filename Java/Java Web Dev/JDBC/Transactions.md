# JDBC Transactions

In Java Database Connectivity (JDBC), a transaction is a sequence of one or more SQL statements that are executed as a single logical unit of work. All statements in the transaction must execute successfully, or none of them are executed. This ensures data integrity and consistency.

### Key Concepts:

*   **ACID Properties**: Transactions in JDBC adhere to the ACID properties (Atomicity, Consistency, Isolation, Durability) to ensure data reliability.
    *   **Atomicity**: Ensures that all operations within a transaction are completed successfully. If any part fails, the entire transaction is aborted.
    *   **Consistency**: Guarantees that a transaction brings the database from one valid state to another.
    *   **Isolation**: Ensures that concurrent transactions do not interfere with each other, preventing issues like dirty reads.
    *   **Durability**: Ensures that once a transaction has been committed, it will remain so, even in the event of a system failure.

*   **Auto-Commit Mode**: By default, JDBC connections are in auto-commit mode. This means each SQL statement is treated as a separate transaction and is committed to the database immediately upon completion.

### Managing Transactions:

To manage transactions manually, you must disable auto-commit mode.

```java
Connection con = DriverManager.getConnection(url, user, password);
con.setAutoCommit(false);
```

Once auto-commit is disabled, you have control over when to commit or roll back the transaction using the following `Connection` object methods:

*   `commit()`: Makes all changes made since the previous commit/rollback permanent.
*   `rollback()`: Discards all changes made in the current transaction.

Here is a typical transaction block structure:

```java
Connection con = null;
try {
    con = DriverManager.getConnection(url, user, password);
    con.setAutoCommit(false);

    // ... execute SQL statements ...

    con.commit();
} catch (SQLException e) {
    if (con != null) {
        try {
            con.rollback();
        } catch (SQLException ex) {
            // ... handle rollback failure ...
        }
    }
    // ... handle original exception ...
} finally {
    if (con != null) {
        try {
            con.close();
        } catch (SQLException e) {
            // ... handle close failure ...
        }
    }
}
```

### Savepoints

JDBC 3.0 introduced the `Savepoint` interface, which allows you to set intermediate points within a transaction. You can then roll back to a specific savepoint instead of rolling back the entire transaction.

*   `setSavepoint(String name)`: Creates a new savepoint.
*   `releaseSavepoint(Savepoint savepoint)`: Removes a savepoint.
*   `rollback(Savepoint savepoint)`: Rolls back the transaction to the state of the specified savepoint.

### Transaction Isolation Levels

Transaction isolation levels define the degree to which one transaction must be isolated from the data modifications made by other transactions. Higher isolation levels prevent more concurrency anomalies but can reduce performance by increasing locking.

You can set the isolation level for a connection using the `setTransactionIsolation()` method.

The standard isolation levels are:

*   **`TRANSACTION_READ_UNCOMMITTED`**: Allows "dirty reads," where a transaction can read uncommitted changes made by another transaction.
*   **`TRANSACTION_READ_COMMITTED`**: Prevents dirty reads, but non-repeatable reads and phantom reads can occur.
*   **`TRANSACTION_REPEATABLE_READ`**: Prevents dirty reads and non-repeatable reads, but phantom reads can still happen.
*   **`TRANSACTION_SERIALIZABLE`**: The highest level of isolation. It prevents all concurrency anomalies by executing transactions serially.
