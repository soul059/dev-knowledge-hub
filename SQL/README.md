# SQL - Complete Reference Guide

A comprehensive SQL reference covering all aspects of relational database querying and management.

## 📚 Table of Contents

- [Overview](#overview)
- [Topics Covered](#topics-covered)
- [Prerequisites](#prerequisites)
- [Learning Path](#learning-path)

## 🎯 Overview

This SQL documentation provides a complete reference for working with relational databases, from basic queries to advanced concepts like transactions, triggers, and stored procedures.

## 📂 Topics Covered

### Core Concepts
**[SQL.md](./SQL.md)** - Introduction and fundamentals

### Data Types & Constraints
- **[SQL Data Types.md](./SQL%20Data%20Types.md)** - Numeric, string, date/time, and other types
- **[SQL Constraints.md](./SQL%20Constraints.md)** - PRIMARY KEY, FOREIGN KEY, UNIQUE, CHECK, NOT NULL
- **[SQL NULL Values.md](./SQL%20NULL%20Values.md)** - Handling NULL in queries

### SQL Sublanguages

#### DDL (Data Definition Language)
**[SQL DDL.md](./SQL%20DDL.md)**
- CREATE, ALTER, DROP, TRUNCATE
- Table and schema management

#### DML (Data Manipulation Language)
**[SQL DML.md](./SQL%20DML.md)**
- INSERT, UPDATE, DELETE
- Data modification operations

#### DQL (Data Query Language)
**[SQL DQL.md](./SQL%20DQL.md)**
- SELECT statements
- Filtering, sorting, and limiting results

#### DCL (Data Control Language)
**[SQL DCL.md](./SQL%20DCL.md)**
- GRANT, REVOKE
- Permission management

### Query Operations

- **[SQL Joins.md](./SQL%20Joins.md)** - INNER, LEFT, RIGHT, FULL, CROSS joins
- **[SQL Subqueries.md](./SQL%20Subqueries.md)** - Nested queries and correlated subqueries
- **[SQL Views.md](./SQL%20Views.md)** - Virtual tables and materialized views
- **[SQL Aggregate Functions and Grouping.md](./SQL%20Aggregate%20Functions%20and%20Grouping.md)** - COUNT, SUM, AVG, GROUP BY, HAVING

### Advanced Topics

- **[SQL Function.md](./SQL%20Function.md)** - Built-in and user-defined functions
- **[SQL Cursor.md](./SQL%20Cursor.md)** - Row-by-row processing
- **[SQL Trigger.md](./SQL%20Trigger.md)** - Automatic actions on data changes
- **[SQL Transactions.md](./SQL%20Transactions.md)** - ACID properties, COMMIT, ROLLBACK

### Beyond SQL
- **[NoSQL.md](./NoSQL.md)** - NoSQL databases and when to use them

## 🔧 Prerequisites

- Basic understanding of data and databases
- Familiarity with tables, rows, and columns concepts

## 🛤️ Recommended Learning Path

### Beginner Track
```
1. SQL Fundamentals (SQL.md)
    ↓
2. Data Types & Constraints
    ↓
3. DDL (Creating tables)
    ↓
4. DML (Inserting/updating data)
    ↓
5. DQL (Basic queries)
```

### Intermediate Track
```
6. Joins (Combining tables)
    ↓
7. Aggregate Functions & Grouping
    ↓
8. Subqueries
    ↓
9. Views
```

### Advanced Track
```
10. Functions & Stored Procedures
    ↓
11. Transactions
    ↓
12. Triggers & Cursors
    ↓
13. DCL (Security)
```

## 💡 How to Use This Guide

| Use Case | Approach |
|----------|----------|
| **Learning SQL** | Follow the beginner track first |
| **Quick Reference** | Navigate directly to specific topic |
| **Interview Prep** | Focus on joins, subqueries, and transactions |
| **Database Design** | Study DDL, constraints, and normalization |

## 🎯 Key SQL Concepts

| Concept | Description |
|---------|-------------|
| **CRUD** | Create, Read, Update, Delete operations |
| **ACID** | Atomicity, Consistency, Isolation, Durability |
| **Normalization** | Reducing data redundancy |
| **Indexing** | Improving query performance |
| **Joins** | Combining data from multiple tables |

## 📝 Quick Reference

```sql
-- Select all columns
SELECT * FROM table_name;

-- Select with conditions
SELECT column1, column2 FROM table_name WHERE condition;

-- Join tables
SELECT * FROM table1 
INNER JOIN table2 ON table1.id = table2.foreign_id;

-- Aggregate with grouping
SELECT category, COUNT(*) FROM products GROUP BY category;

-- Insert data
INSERT INTO table_name (col1, col2) VALUES (val1, val2);

-- Update data
UPDATE table_name SET column = value WHERE condition;

-- Delete data
DELETE FROM table_name WHERE condition;
```

---

*Last Updated: April 2026*
