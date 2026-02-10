---
tags:
  - database
  - indexing
  - performance-tuning
  - storage-management
  - query-optimization
---

# Database Indexing

**Database Indexing** is a technique used by [[DBMS|Database Management Systems]] to improve the speed of data retrieval operations on database tables. An index is a data structure that allows the DBMS to quickly locate rows in a table without having to scan the entire table.

## Purpose: Faster Data Retrieval

Without an index, to find rows matching a condition in a large table, the [[DBMS]] typically has to perform a **full table scan**, reading every single row and checking if it meets the criteria. This is inefficient for large tables.

Indexing provides a way to access data directly. Think of a book:
* Finding a specific topic by reading the entire book is like a full table scan.
* Finding a specific topic by looking it up in the index at the back and going directly to the page number is like using a database index.

## How Indexes Work (Conceptually)

An index is typically created on one or more columns of a table. It stores the values from the indexed column(s) along with pointers (references) to the physical location of the corresponding rows in the table's data file. The index itself is kept sorted (for ordered indexes) or structured (for hash indexes) to allow for very fast lookups based on the indexed values.

When a query includes a `WHERE` clause condition on an indexed column, the [[Query Optimization|query optimizer]] in the DBMS can choose to use the index to find the required rows much faster than scanning the whole table.

## Trade-offs of Indexing

While indexes significantly speed up read operations ([[SQL DQL|SELECT]] queries), they come with costs:

* **Increased Write Overhead:** Whenever data is inserted, updated, or deleted in the table, the index(es) on that table must also be updated to reflect the changes. This takes time and can slow down write operations ([[SQL DML]]).
* **Additional Storage Space:** Indexes are data structures themselves and require disk space to store.
* **Maintenance Complexity:** The DBMS needs to maintain the index structures, which adds overhead.

Indexing is a trade-off: you gain faster reads at the expense of slower writes and increased storage.

## Types of Indexes

Indexes can be broadly classified based on their structure and how they store data:

1.  **Ordered Indexes (e.g., B+ Tree Indexes):**
    * **How they work:** Values from the indexed column(s) are stored in a sorted order in a tree-like structure (most commonly a B+ Tree). Each leaf node in the tree contains pointers to the actual data rows.
    * **Suitability:** Excellent for equality lookups (`=`) and particularly efficient for **range queries** (`%3E`, `%3C`, `BETWEEN`), as sorted data allows scanning ranges efficiently. Also good for `ORDER BY` clauses.
    * **Common Use:** The default index type in most RDBMS, used for [[SQL Constraints|Primary Keys]] and many secondary indexes.

2.  **Hash Indexes:**
    * **How they work:** Values from the indexed column(s) are stored using a hashing function. The hash function calculates a hash value for each key, which points to the location of the data record.
    * **Suitability:** Extremely fast for **equality lookups** (`=`).
    * **Limitations:** Not suitable for range queries or `ORDER BY` clauses because the hash function scatters data randomly, destroying sorted order.
    * **Common Use:** Less common as the default than B+ Trees, used in specific scenarios where only equality lookups are needed.

## Index Structures and Classifications

Indexes can also be classified based on how they relate to the data storage:

* **Primary Index:** An index created on the column(s) designated as the [[SQL Constraints|PRIMARY KEY]].
    * Often automatically created by the DBMS.
    * Can be a [[#Clustering Index|Clustering Index]].
* **Secondary Index:** An index created on any column(s) other than the primary key. A table can have multiple secondary indexes.
* **Clustering Index:** An index where the physical order of the data rows on disk is stored in the same order as the index key. A table can have only **one** clustering index. It can significantly speed up range queries but slows down inserts/updates as data rows might need to be physically moved. (Primary keys are often clustering indexes by default).
* **Non-Clustering Index:** An index where the physical order of data rows is independent of the index order. Most indexes are non-clustering. The index contains pointers (e.g., row ID) to the actual data location.
* **Dense Index:** An index record exists for *every* search key value in the data file.
* **Sparse Index:** An index record exists only for *some* search key values (typically for blocks of records), reducing index size but requiring a sequential scan of a small portion of the data file. (Often used with clustering indexes).

## When to Use/Create Indexes

Indexing is most beneficial on columns that are:

* Frequently used in the `WHERE` clause of [[SQL DQL|SELECT]] queries.
* Used in `JOIN` conditions ([[SQL Joins]]).
* Used in `ORDER BY` or `GROUP BY` clauses ([[SQL Aggregate Functions and Grouping|Aggregate Functions and Grouping]]).
* [[SQL Constraints|Primary Keys]] (often indexed automatically).
* [[SQL Constraints|Foreign Keys]] (indexing is often beneficial for join performance and referential integrity checks).
* Columns with a high number of distinct values (high cardinality).

## When Not to Overuse Indexes

Indexes are less beneficial or can even hurt performance on:

* Small tables where a full table scan is faster than traversing an index.
* Tables that have very high rates of `INSERT`, `UPDATE`, or `DELETE` operations (write-heavy workloads), as index maintenance cost outweighs read benefits.
* Columns with very few distinct values (low cardinality), as the index might not significantly narrow down the search.
* Columns not frequently used in queries.

Indexes are defined and managed using [[SQL DDL]] commands, such as `CREATE INDEX`.

```sql
-- Example Syntax (varies by RDBMS)
CREATE INDEX index_name
ON table_name (column1, column2, ...);

-- Example: Create an index on the LastName column of the Students table
CREATE INDEX idx_students_lastname
ON Students (LastName);

-- Example: Create a composite index on City and State for a Customers table
CREATE INDEX idx_customers_city_state
ON Customers (City, State);