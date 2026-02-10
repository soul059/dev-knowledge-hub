# Chapter 3.2: DML - Data Manipulation Language

[← Back to Table of Contents](../README.md)

---

## 📚 Chapter Overview

**Data Manipulation Language (DML)** is a subset of SQL used to retrieve, insert, update, and delete data in database tables. While DDL defines the structure, DML works with the actual data.

```
┌─────────────────────────────────────────────────────────────────┐
│                     CHAPTER LEARNING GOALS                       │
├─────────────────────────────────────────────────────────────────┤
│  • Master SELECT for data retrieval                             │
│  • Learn INSERT for adding new records                          │
│  • Understand UPDATE for modifying data                         │
│  • Use DELETE for removing records                              │
│  • Practice combining DML operations                            │
└─────────────────────────────────────────────────────────────────┘
```

---

## 1. DML Overview

```
┌─────────────────────────────────────────────────────────────────────┐
│                    DML - DATA MANIPULATION LANGUAGE                  │
└─────────────────────────────────────────────────────────────────────┘

    DML COMMANDS:
    ──────────────
    
    ┌───────────┬────────────────────────────────────────────────────┐
    │  Command  │                    Purpose                         │
    ├───────────┼────────────────────────────────────────────────────┤
    │  SELECT   │  Retrieves data from tables                       │
    │  INSERT   │  Adds new rows to a table                         │
    │  UPDATE   │  Modifies existing data                           │
    │  DELETE   │  Removes rows from a table                        │
    └───────────┴────────────────────────────────────────────────────┘
    
    
    KEY CHARACTERISTICS:
    ─────────────────────
    
    • DML operations can be ROLLED BACK (within transaction)
    • DML affects data, not structure
    • SELECT is sometimes called DRL (Data Retrieval Language)
    
    
    VISUALIZATION:
    ───────────────
    
                        DML Operations
                              │
        ┌──────────┬──────────┼──────────┬──────────┐
        ↓          ↓          ↓          ↓
    ┌────────┐ ┌────────┐ ┌────────┐ ┌────────┐
    │ SELECT │ │ INSERT │ │ UPDATE │ │ DELETE │
    │  Read  │ │ Create │ │ Modify │ │ Remove │
    └────────┘ └────────┘ └────────┘ └────────┘
        │          │          │          │
        └──────────┴──────────┴──────────┘
                       ↓
              ┌──────────────┐
              │  Table Data  │
              └──────────────┘
```

---

## 2. SELECT Statement

### Basic SELECT

```
┌─────────────────────────────────────────────────────────────────────┐
│                         SELECT BASICS                                │
└─────────────────────────────────────────────────────────────────────┘

    GENERAL SYNTAX:
    ─────────────────
    
    SELECT column1, column2, ...
    FROM table_name
    [WHERE condition]
    [ORDER BY column]
    [LIMIT n];


    SAMPLE TABLE:
    ──────────────
    
    Student
    ┌─────────┬─────────┬─────┬─────────┬─────────────┐
    │ Stud_ID │  Name   │ Age │ Dept_ID │   Email     │
    ├─────────┼─────────┼─────┼─────────┼─────────────┤
    │  S101   │  Alice  │ 20  │   D01   │ a@mail.com  │
    │  S102   │   Bob   │ 22  │   D02   │ b@mail.com  │
    │  S103   │ Charlie │ 21  │   D01   │ c@mail.com  │
    │  S104   │  Diana  │ 23  │   D03   │ d@mail.com  │
    └─────────┴─────────┴─────┴─────────┴─────────────┘


    EXAMPLES:
    ──────────

    -- Select all columns
    SELECT * FROM Student;
    
    -- Select specific columns
    SELECT Name, Age FROM Student;
    
    -- With condition (WHERE)
    SELECT Name, Age FROM Student WHERE Age > 20;
    
    Result:
    ┌─────────┬─────┐
    │  Name   │ Age │
    ├─────────┼─────┤
    │   Bob   │ 22  │
    │ Charlie │ 21  │
    │  Diana  │ 23  │
    └─────────┴─────┘
    
    
    -- With DISTINCT (remove duplicates)
    SELECT DISTINCT Dept_ID FROM Student;
    
    Result: D01, D02, D03
    
    
    -- With ORDER BY
    SELECT Name, Age FROM Student ORDER BY Age DESC;
    
    Result:
    ┌─────────┬─────┐
    │  Name   │ Age │
    ├─────────┼─────┤
    │  Diana  │ 23  │
    │   Bob   │ 22  │
    │ Charlie │ 21  │
    │  Alice  │ 20  │
    └─────────┴─────┘
    
    
    -- With LIMIT
    SELECT * FROM Student LIMIT 2;
    
    Result: First 2 rows only
```

### WHERE Clause Operators

```
┌─────────────────────────────────────────────────────────────────────┐
│                      WHERE CLAUSE OPERATORS                          │
└─────────────────────────────────────────────────────────────────────┘


    COMPARISON OPERATORS:
    ──────────────────────
    
    ┌──────────┬─────────────────────────────────────────┐
    │ Operator │              Description                │
    ├──────────┼─────────────────────────────────────────┤
    │    =     │  Equal to                               │
    │   <> !=  │  Not equal to                           │
    │    <     │  Less than                              │
    │    >     │  Greater than                           │
    │   <=     │  Less than or equal                     │
    │   >=     │  Greater than or equal                  │
    └──────────┴─────────────────────────────────────────┘
    
    Example: SELECT * FROM Student WHERE Age >= 21;


    LOGICAL OPERATORS:
    ───────────────────
    
    SELECT * FROM Student WHERE Age > 20 AND Dept_ID = 'D01';
    SELECT * FROM Student WHERE Age > 22 OR Dept_ID = 'D01';
    SELECT * FROM Student WHERE NOT Dept_ID = 'D03';


    BETWEEN:
    ─────────
    
    SELECT * FROM Student WHERE Age BETWEEN 20 AND 22;
    -- Same as: WHERE Age >= 20 AND Age <= 22
    
    Result:
    ┌─────────┬─────────┬─────┐
    │ Stud_ID │  Name   │ Age │
    ├─────────┼─────────┼─────┤
    │  S101   │  Alice  │ 20  │
    │  S102   │   Bob   │ 22  │
    │  S103   │ Charlie │ 21  │
    └─────────┴─────────┴─────┘


    IN:
    ────
    
    SELECT * FROM Student WHERE Dept_ID IN ('D01', 'D02');
    -- Same as: WHERE Dept_ID = 'D01' OR Dept_ID = 'D02'


    LIKE (Pattern Matching):
    ─────────────────────────
    
    ┌────────────┬────────────────────────────────────────┐
    │  Pattern   │            Matches                     │
    ├────────────┼────────────────────────────────────────┤
    │  'A%'      │  Starts with 'A'                       │
    │  '%e'      │  Ends with 'e'                         │
    │  '%li%'    │  Contains 'li'                         │
    │  'A__'     │  'A' followed by exactly 2 characters  │
    │  '_o%'     │  Second character is 'o'               │
    └────────────┴────────────────────────────────────────┘
    
    SELECT * FROM Student WHERE Name LIKE 'A%';
    Result: Alice
    
    SELECT * FROM Student WHERE Name LIKE '%e';
    Result: Alice, Charlie


    IS NULL / IS NOT NULL:
    ───────────────────────
    
    SELECT * FROM Student WHERE Email IS NOT NULL;
    
    Note: Cannot use = NULL, must use IS NULL
```

### Column Aliases and Expressions

```
┌─────────────────────────────────────────────────────────────────────┐
│                   ALIASES AND EXPRESSIONS                            │
└─────────────────────────────────────────────────────────────────────┘


    COLUMN ALIAS:
    ──────────────
    
    SELECT Name AS Student_Name, Age AS Years FROM Student;
    
    Result:
    ┌──────────────┬───────┐
    │ Student_Name │ Years │
    ├──────────────┼───────┤
    │    Alice     │  20   │
    │     Bob      │  22   │
    └──────────────┴───────┘


    TABLE ALIAS:
    ─────────────
    
    SELECT s.Name, s.Age FROM Student s WHERE s.Age > 20;


    EXPRESSIONS:
    ─────────────
    
    SELECT Name, Age, Age + 10 AS Age_In_10_Years FROM Student;
    
    Result:
    ┌─────────┬─────┬─────────────────┐
    │  Name   │ Age │ Age_In_10_Years │
    ├─────────┼─────┼─────────────────┤
    │  Alice  │ 20  │       30        │
    │   Bob   │ 22  │       32        │
    └─────────┴─────┴─────────────────┘


    STRING CONCATENATION:
    ───────────────────────
    
    -- SQL Server
    SELECT Name + ' (' + Stud_ID + ')' AS Display FROM Student;
    
    -- MySQL
    SELECT CONCAT(Name, ' (', Stud_ID, ')') AS Display FROM Student;
    
    Result:
    ┌────────────────┐
    │    Display     │
    ├────────────────┤
    │ Alice (S101)   │
    │ Bob (S102)     │
    └────────────────┘
```

---

## 3. INSERT Statement

```
┌─────────────────────────────────────────────────────────────────────┐
│                        INSERT STATEMENT                              │
└─────────────────────────────────────────────────────────────────────┘


    SYNTAX 1: Specify all columns
    ──────────────────────────────
    
    INSERT INTO table_name (column1, column2, ...)
    VALUES (value1, value2, ...);
    
    INSERT INTO Student (Stud_ID, Name, Age, Dept_ID)
    VALUES ('S105', 'Eve', 19, 'D02');


    SYNTAX 2: All columns (in order)
    ──────────────────────────────────
    
    INSERT INTO Student
    VALUES ('S105', 'Eve', 19, 'D02', 'e@mail.com');
    
    Warning: Must provide values for ALL columns in correct order!


    MULTIPLE ROWS:
    ────────────────
    
    INSERT INTO Student (Stud_ID, Name, Age, Dept_ID)
    VALUES 
        ('S105', 'Eve', 19, 'D02'),
        ('S106', 'Frank', 20, 'D01'),
        ('S107', 'Grace', 21, 'D03');


    INSERT FROM SELECT:
    ─────────────────────
    
    INSERT INTO Archive_Student (Stud_ID, Name)
    SELECT Stud_ID, Name FROM Student WHERE Age > 22;


    VISUALIZATION:
    ───────────────
    
    Before INSERT:
    ┌─────────┬─────────┬─────┐
    │ Stud_ID │  Name   │ Age │
    ├─────────┼─────────┼─────┤
    │  S101   │  Alice  │ 20  │
    │  S102   │   Bob   │ 22  │
    └─────────┴─────────┴─────┘
    
    INSERT INTO Student VALUES ('S103', 'Charlie', 21);
    
    After INSERT:
    ┌─────────┬─────────┬─────┐
    │ Stud_ID │  Name   │ Age │
    ├─────────┼─────────┼─────┤
    │  S101   │  Alice  │ 20  │
    │  S102   │   Bob   │ 22  │
    │  S103   │ Charlie │ 21  │  ← New row
    └─────────┴─────────┴─────┘
    
    
    CONSTRAINTS CHECK:
    ───────────────────
    
    INSERT INTO Student (Stud_ID, Name, Age, Dept_ID)
    VALUES ('S101', 'Test', 25, 'D01');
    
    ERROR: Duplicate primary key 'S101'!
    
    INSERT INTO Student (Stud_ID, Name, Age, Dept_ID)
    VALUES ('S108', 'Test', 15, 'D01');
    
    ERROR: CHECK constraint violated (Age must be >= 16)
```

---

## 4. UPDATE Statement

```
┌─────────────────────────────────────────────────────────────────────┐
│                        UPDATE STATEMENT                              │
└─────────────────────────────────────────────────────────────────────┘


    SYNTAX:
    ────────
    
    UPDATE table_name
    SET column1 = value1, column2 = value2, ...
    WHERE condition;
    
    ⚠️ WARNING: Without WHERE, ALL rows are updated!


    EXAMPLES:
    ──────────
    
    -- Update single row
    UPDATE Student SET Age = 21 WHERE Stud_ID = 'S101';
    
    
    -- Update multiple columns
    UPDATE Student 
    SET Age = 22, Dept_ID = 'D02' 
    WHERE Stud_ID = 'S101';
    
    
    -- Update multiple rows
    UPDATE Student SET Age = Age + 1 WHERE Dept_ID = 'D01';
    
    
    -- Update with expression
    UPDATE Product SET Price = Price * 1.10;  -- 10% increase
    
    
    -- Update all rows (dangerous!)
    UPDATE Student SET Status = 'Active';  -- Updates EVERYONE


    VISUALIZATION:
    ───────────────
    
    Before UPDATE:
    ┌─────────┬─────────┬─────┬─────────┐
    │ Stud_ID │  Name   │ Age │ Dept_ID │
    ├─────────┼─────────┼─────┼─────────┤
    │  S101   │  Alice  │ 20  │   D01   │
    │  S102   │   Bob   │ 22  │   D02   │
    │  S103   │ Charlie │ 21  │   D01   │
    └─────────┴─────────┴─────┴─────────┘
    
    UPDATE Student SET Age = Age + 1 WHERE Dept_ID = 'D01';
    
    After UPDATE:
    ┌─────────┬─────────┬─────┬─────────┐
    │ Stud_ID │  Name   │ Age │ Dept_ID │
    ├─────────┼─────────┼─────┼─────────┤
    │  S101   │  Alice  │ 21  │   D01   │  ← Updated
    │  S102   │   Bob   │ 22  │   D02   │
    │  S103   │ Charlie │ 22  │   D01   │  ← Updated
    └─────────┴─────────┴─────┴─────────┘


    UPDATE WITH SUBQUERY:
    ──────────────────────
    
    UPDATE Student
    SET Dept_ID = (SELECT Dept_ID FROM Department 
                   WHERE Dept_Name = 'Physics')
    WHERE Stud_ID = 'S101';
```

---

## 5. DELETE Statement

```
┌─────────────────────────────────────────────────────────────────────┐
│                        DELETE STATEMENT                              │
└─────────────────────────────────────────────────────────────────────┘


    SYNTAX:
    ────────
    
    DELETE FROM table_name WHERE condition;
    
    ⚠️ WARNING: Without WHERE, ALL rows are deleted!


    EXAMPLES:
    ──────────
    
    -- Delete single row
    DELETE FROM Student WHERE Stud_ID = 'S104';
    
    
    -- Delete multiple rows
    DELETE FROM Student WHERE Age > 22;
    
    
    -- Delete with complex condition
    DELETE FROM Student 
    WHERE Dept_ID = 'D01' AND Age < 20;
    
    
    -- Delete all rows (like TRUNCATE but slower)
    DELETE FROM Student;


    VISUALIZATION:
    ───────────────
    
    Before DELETE:
    ┌─────────┬─────────┬─────┐
    │ Stud_ID │  Name   │ Age │
    ├─────────┼─────────┼─────┤
    │  S101   │  Alice  │ 20  │
    │  S102   │   Bob   │ 22  │
    │  S103   │ Charlie │ 21  │
    │  S104   │  Diana  │ 23  │
    └─────────┴─────────┴─────┘
    
    DELETE FROM Student WHERE Age > 22;
    
    After DELETE:
    ┌─────────┬─────────┬─────┐
    │ Stud_ID │  Name   │ Age │
    ├─────────┼─────────┼─────┤
    │  S101   │  Alice  │ 20  │
    │  S102   │   Bob   │ 22  │
    │  S103   │ Charlie │ 21  │
    └─────────┴─────────┴─────┘
    
    (Diana removed)


    DELETE vs TRUNCATE:
    ────────────────────
    
    ┌────────────────┬────────────────────┬────────────────────┐
    │    Feature     │       DELETE       │      TRUNCATE      │
    ├────────────────┼────────────────────┼────────────────────┤
    │ WHERE clause   │        Yes         │        No          │
    │ Fires triggers │        Yes         │        No          │
    │ Can rollback   │        Yes         │      Usually No    │
    │ Speed          │       Slower       │       Faster       │
    │ Logs           │    Each row        │    Minimal         │
    │ Reset identity │        No          │       Yes          │
    └────────────────┴────────────────────┴────────────────────┘
    
    
    DELETE WITH SUBQUERY:
    ──────────────────────
    
    DELETE FROM Enrollment
    WHERE Stud_ID IN (SELECT Stud_ID FROM Student 
                      WHERE Dept_ID = 'D03');
```

---

## 6. Aggregate Functions

```
┌─────────────────────────────────────────────────────────────────────┐
│                      AGGREGATE FUNCTIONS                             │
└─────────────────────────────────────────────────────────────────────┘

    Aggregate functions perform calculations on sets of values.
    

    COMMON AGGREGATE FUNCTIONS:
    ────────────────────────────
    
    ┌────────────┬─────────────────────────────────────────┐
    │  Function  │              Description                │
    ├────────────┼─────────────────────────────────────────┤
    │ COUNT()    │ Count of rows/non-null values          │
    │ SUM()      │ Sum of values                          │
    │ AVG()      │ Average of values                      │
    │ MAX()      │ Maximum value                          │
    │ MIN()      │ Minimum value                          │
    └────────────┴─────────────────────────────────────────┘


    Sample Table:
    ──────────────
    
    Student
    ┌─────────┬─────────┬─────┬─────────┐
    │ Stud_ID │  Name   │ Age │ Dept_ID │
    ├─────────┼─────────┼─────┼─────────┤
    │  S101   │  Alice  │ 20  │   D01   │
    │  S102   │   Bob   │ 22  │   D02   │
    │  S103   │ Charlie │ 21  │   D01   │
    │  S104   │  Diana  │ 23  │   D03   │
    └─────────┴─────────┴─────┴─────────┘


    EXAMPLES:
    ──────────
    
    -- Count all rows
    SELECT COUNT(*) FROM Student;
    Result: 4
    
    -- Count non-null values
    SELECT COUNT(Dept_ID) FROM Student;
    Result: 4 (if no nulls)
    
    -- Count distinct values
    SELECT COUNT(DISTINCT Dept_ID) FROM Student;
    Result: 3
    
    -- Sum
    SELECT SUM(Age) FROM Student;
    Result: 86
    
    -- Average
    SELECT AVG(Age) FROM Student;
    Result: 21.5
    
    -- Min and Max
    SELECT MIN(Age), MAX(Age) FROM Student;
    Result: 20, 23


    VISUALIZATION:
    ───────────────
    
    SELECT AVG(Age) FROM Student;
    
    ┌─────┐   ┌─────┐   ┌─────┐   ┌─────┐
    │ 20  │ + │ 22  │ + │ 21  │ + │ 23  │ = 86
    └─────┘   └─────┘   └─────┘   └─────┘
              
              86 ÷ 4 = 21.5
```

---

## 7. GROUP BY and HAVING

```
┌─────────────────────────────────────────────────────────────────────┐
│                      GROUP BY AND HAVING                             │
└─────────────────────────────────────────────────────────────────────┘


    GROUP BY:
    ──────────
    
    Groups rows with same values for aggregate calculations.
    
    SELECT Dept_ID, COUNT(*) AS Student_Count
    FROM Student
    GROUP BY Dept_ID;
    
    
    VISUALIZATION:
    ───────────────
    
    Original:                    Grouped:
    ┌─────────┬─────────┐        ┌─────────┬────────────────┐
    │ Stud_ID │ Dept_ID │        │ Dept_ID │ Student_Count  │
    ├─────────┼─────────┤        ├─────────┼────────────────┤
    │  S101   │   D01   │─┐      │   D01   │       2        │
    │  S102   │   D02   │ │      │   D02   │       1        │
    │  S103   │   D01   │─┘      │   D03   │       1        │
    │  S104   │   D03   │        └─────────┴────────────────┘
    └─────────┴─────────┘


    MORE EXAMPLES:
    ───────────────
    
    -- Average age per department
    SELECT Dept_ID, AVG(Age) AS Avg_Age
    FROM Student
    GROUP BY Dept_ID;
    
    Result:
    ┌─────────┬─────────┐
    │ Dept_ID │ Avg_Age │
    ├─────────┼─────────┤
    │   D01   │  20.5   │
    │   D02   │  22.0   │
    │   D03   │  23.0   │
    └─────────┴─────────┘
    
    
    -- Multiple aggregates
    SELECT Dept_ID, 
           COUNT(*) AS Count,
           MIN(Age) AS Min_Age,
           MAX(Age) AS Max_Age
    FROM Student
    GROUP BY Dept_ID;


    HAVING (Filter groups):
    ────────────────────────
    
    HAVING filters groups AFTER grouping (WHERE filters rows BEFORE).
    
    SELECT Dept_ID, COUNT(*) AS Count
    FROM Student
    GROUP BY Dept_ID
    HAVING COUNT(*) > 1;
    
    Result:
    ┌─────────┬───────┐
    │ Dept_ID │ Count │
    ├─────────┼───────┤
    │   D01   │   2   │  ← Only departments with > 1 student
    └─────────┴───────┘
    

    WHERE vs HAVING:
    ──────────────────
    
    ┌─────────────────────────────────────────────────────────────────┐
    │                                                                  │
    │   WHERE                          HAVING                         │
    │   ─────                          ──────                         │
    │   Filters individual ROWS        Filters GROUPS                 │
    │   Before GROUP BY                After GROUP BY                 │
    │   Cannot use aggregates          Can use aggregates             │
    │                                                                  │
    └─────────────────────────────────────────────────────────────────┘
    
    -- Combined example
    SELECT Dept_ID, AVG(Age) AS Avg_Age
    FROM Student
    WHERE Age >= 20           -- Filter rows first
    GROUP BY Dept_ID
    HAVING AVG(Age) > 21;     -- Then filter groups


    QUERY EXECUTION ORDER:
    ───────────────────────
    
    1. FROM     → Identify tables
    2. WHERE    → Filter rows
    3. GROUP BY → Create groups
    4. HAVING   → Filter groups
    5. SELECT   → Choose columns
    6. ORDER BY → Sort results
    7. LIMIT    → Limit results
```

---

## 8. SQL Query Processing Order

```
┌─────────────────────────────────────────────────────────────────────┐
│                   SQL QUERY EXECUTION ORDER                          │
└─────────────────────────────────────────────────────────────────────┘

    Written Order:              Execution Order:
    ──────────────              ────────────────
    
    SELECT                      1. FROM
    FROM                        2. WHERE
    WHERE                       3. GROUP BY
    GROUP BY                    4. HAVING
    HAVING                      5. SELECT
    ORDER BY                    6. DISTINCT
    LIMIT                       7. ORDER BY
                                8. LIMIT


    VISUALIZATION:
    ───────────────
    
    SELECT Dept_ID, AVG(Age) AS Avg_Age
    FROM Student
    WHERE Age >= 20
    GROUP BY Dept_ID
    HAVING AVG(Age) > 21
    ORDER BY Avg_Age DESC
    LIMIT 2;
    
    
    Step 1: FROM Student
    ┌─────────┬─────────┬─────┬─────────┐
    │ Stud_ID │  Name   │ Age │ Dept_ID │
    ├─────────┼─────────┼─────┼─────────┤
    │  S101   │  Alice  │ 20  │   D01   │
    │  S102   │   Bob   │ 22  │   D02   │
    │  S103   │ Charlie │ 21  │   D01   │
    │  S104   │  Diana  │ 23  │   D03   │
    └─────────┴─────────┴─────┴─────────┘
    
    Step 2: WHERE Age >= 20 (all rows pass)
    
    Step 3: GROUP BY Dept_ID
    ┌─────────┬────────────────────┐
    │ Dept_ID │      Students      │
    ├─────────┼────────────────────┤
    │   D01   │  Alice(20), Charlie(21)  │
    │   D02   │  Bob(22)           │
    │   D03   │  Diana(23)         │
    └─────────┴────────────────────┘
    
    Step 4: HAVING AVG(Age) > 21
    ┌─────────┬─────────┐
    │ Dept_ID │ Avg_Age │
    ├─────────┼─────────┤
    │   D02   │  22.0   │ ✓ passes
    │   D03   │  23.0   │ ✓ passes
    └─────────┴─────────┘
    (D01 with 20.5 is filtered out)
    
    Step 5: SELECT Dept_ID, AVG(Age) AS Avg_Age
    
    Step 6: ORDER BY Avg_Age DESC
    ┌─────────┬─────────┐
    │ Dept_ID │ Avg_Age │
    ├─────────┼─────────┤
    │   D03   │  23.0   │
    │   D02   │  22.0   │
    └─────────┴─────────┘
    
    Step 7: LIMIT 2 (already 2 rows)
```

---

## 📊 Summary Table

| Command | Purpose | Syntax Example |
|---------|---------|----------------|
| **SELECT** | Retrieve data | `SELECT * FROM t WHERE x > 5;` |
| **SELECT DISTINCT** | Unique values | `SELECT DISTINCT col FROM t;` |
| **INSERT** | Add new row | `INSERT INTO t VALUES (1, 'a');` |
| **UPDATE** | Modify data | `UPDATE t SET x = 5 WHERE id = 1;` |
| **DELETE** | Remove rows | `DELETE FROM t WHERE x < 0;` |

| Operator | Description | Example |
|----------|-------------|---------|
| **BETWEEN** | Range check | `WHERE x BETWEEN 1 AND 10` |
| **IN** | Set membership | `WHERE x IN (1, 2, 3)` |
| **LIKE** | Pattern match | `WHERE name LIKE 'A%'` |
| **IS NULL** | NULL check | `WHERE x IS NULL` |

| Aggregate | Purpose | NULL Handling |
|-----------|---------|---------------|
| **COUNT(*)** | Count rows | Counts all |
| **COUNT(col)** | Count non-null | Ignores NULL |
| **SUM** | Total | Ignores NULL |
| **AVG** | Average | Ignores NULL |
| **MIN/MAX** | Extremes | Ignores NULL |

---

## ❓ Quick Revision Questions

1. **What is the difference between WHERE and HAVING?**
   <details>
   <summary>Click for Answer</summary>
   WHERE filters individual rows BEFORE grouping. HAVING filters groups AFTER GROUP BY. WHERE cannot use aggregate functions; HAVING can. Example: WHERE Age > 20 (filter rows), HAVING COUNT(*) > 5 (filter groups).
   </details>

2. **What happens if you run UPDATE without a WHERE clause?**
   <details>
   <summary>Click for Answer</summary>
   ALL rows in the table will be updated with the new value. This is dangerous and usually unintended. Always use WHERE to specify which rows to update.
   </details>

3. **What is the difference between DELETE and TRUNCATE?**
   <details>
   <summary>Click for Answer</summary>
   DELETE can use WHERE clause, fires triggers, can be rolled back, logs each row, and is slower. TRUNCATE cannot use WHERE, doesn't fire triggers, usually cannot be rolled back, does minimal logging, and is faster.
   </details>

4. **How does COUNT(*) differ from COUNT(column)?**
   <details>
   <summary>Click for Answer</summary>
   COUNT(*) counts all rows including those with NULL values. COUNT(column) counts only non-NULL values in that specific column. If a column has 5 NULLs in 10 rows, COUNT(*) = 10 but COUNT(column) = 5.
   </details>

5. **What is the SQL execution order?**
   <details>
   <summary>Click for Answer</summary>
   FROM → WHERE → GROUP BY → HAVING → SELECT → DISTINCT → ORDER BY → LIMIT. This differs from the written order (SELECT first). Understanding this helps debug queries.
   </details>

6. **How do you insert multiple rows in a single INSERT statement?**
   <details>
   <summary>Click for Answer</summary>
   Use multiple value lists: INSERT INTO table (col1, col2) VALUES (val1, val2), (val3, val4), (val5, val6); Each parenthesis represents one row.
   </details>

---

## 🔗 Navigation

| Previous | Up | Next |
|----------|----|----|
| [← DDL - Data Definition Language](01-ddl-data-definition-language.md) | [📚 Table of Contents](../README.md) | [SQL Joins →](03-sql-joins.md) |

---

*Last Updated: January 2026*
