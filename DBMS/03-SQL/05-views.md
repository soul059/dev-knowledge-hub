# Chapter 3.5: Views

[← Back to Table of Contents](../README.md)

---

## 📚 Chapter Overview

A **View** is a virtual table based on the result of a SQL query. Views don't store data themselves but provide a way to simplify complex queries, enhance security, and present data in different formats.

```
┌─────────────────────────────────────────────────────────────────┐
│                     CHAPTER LEARNING GOALS                       │
├─────────────────────────────────────────────────────────────────┤
│  • Understand what views are and why they're used               │
│  • Create, modify, and drop views                               │
│  • Learn about updatable vs non-updatable views                 │
│  • Understand materialized views                                │
│  • Use views for security and abstraction                       │
└─────────────────────────────────────────────────────────────────┘
```

---

## 1. What is a View?

```
┌─────────────────────────────────────────────────────────────────────┐
│                       WHAT IS A VIEW?                                │
└─────────────────────────────────────────────────────────────────────┘

    A view is a VIRTUAL TABLE created by a stored SELECT query.
    It doesn't store data - it retrieves data from base tables each time.
    
    
    VIEW VS TABLE:
    ───────────────
    
    ┌───────────────────────────────────────────────────────────────────┐
    │                                                                    │
    │   TABLE (Base Table)             VIEW (Virtual Table)             │
    │   ──────────────────             ────────────────────             │
    │   Stores actual data             Stores only the query            │
    │   Takes physical space           No data storage                  │
    │   Data exists permanently        Data computed on access          │
    │   Direct data manipulation       Query result each time           │
    │                                                                    │
    └───────────────────────────────────────────────────────────────────┘
    
    
    VISUALIZATION:
    ───────────────
    
    Base Tables:
    ┌──────────────────┐    ┌──────────────────┐
    │     Student      │    │    Department    │
    │──────────────────│    │──────────────────│
    │ S101 │ Alice │D01│    │ D01 │     CS    │
    │ S102 │  Bob  │D02│    │ D02 │    Math   │
    │ S103 │Charlie│D01│    │ D03 │  Physics  │
    └──────────────────┘    └──────────────────┘
           │                        │
           └────────┬───────────────┘
                    ↓
              ┌───────────┐
              │   VIEW    │
              │ CS_Students│
              ├───────────┤
              │  (Query)  │
              └───────────┘
                    │
                    ↓ When accessed:
           ┌───────────────────┐
           │     Results       │
           ├───────────────────┤
           │ Alice │    CS    │
           │Charlie│    CS    │
           └───────────────────┘
    
    
    ANALOGY:
    ─────────
    
    View = A saved filter on a spreadsheet
    
    • The filter doesn't copy data
    • It shows specific rows/columns based on criteria
    • Original data changes → filtered view updates automatically
```

---

## 2. Creating Views

```
┌─────────────────────────────────────────────────────────────────────┐
│                       CREATING VIEWS                                 │
└─────────────────────────────────────────────────────────────────────┘


    BASIC SYNTAX:
    ──────────────
    
    CREATE VIEW view_name AS
    SELECT column1, column2, ...
    FROM table_name
    WHERE condition;


    EXAMPLE 1: Simple view
    ───────────────────────
    
    CREATE VIEW CS_Students AS
    SELECT Stud_ID, Name, Age
    FROM Student
    WHERE Dept_ID = 'D01';
    
    Usage:
    SELECT * FROM CS_Students;
    
    Result:
    ┌─────────┬─────────┬─────┐
    │ Stud_ID │  Name   │ Age │
    ├─────────┼─────────┼─────┤
    │  S101   │  Alice  │ 20  │
    │  S103   │ Charlie │ 21  │
    └─────────┴─────────┴─────┘


    EXAMPLE 2: View with join
    ──────────────────────────
    
    CREATE VIEW Student_Details AS
    SELECT s.Stud_ID, s.Name, s.Age, d.Dept_Name
    FROM Student s
    JOIN Department d ON s.Dept_ID = d.Dept_ID;
    
    Usage:
    SELECT * FROM Student_Details WHERE Age > 20;


    EXAMPLE 3: View with calculated column
    ────────────────────────────────────────
    
    CREATE VIEW Student_Stats AS
    SELECT Dept_ID,
           COUNT(*) AS Student_Count,
           AVG(Age) AS Avg_Age,
           MIN(Age) AS Min_Age,
           MAX(Age) AS Max_Age
    FROM Student
    GROUP BY Dept_ID;
    
    Result:
    ┌─────────┬───────────────┬─────────┬─────────┬─────────┐
    │ Dept_ID │ Student_Count │ Avg_Age │ Min_Age │ Max_Age │
    ├─────────┼───────────────┼─────────┼─────────┼─────────┤
    │   D01   │       2       │  20.5   │   20    │   21    │
    │   D02   │       2       │  20.5   │   19    │   22    │
    │   D03   │       1       │  23.0   │   23    │   23    │
    └─────────┴───────────────┴─────────┴─────────┴─────────┘


    EXAMPLE 4: View with column aliases
    ─────────────────────────────────────
    
    CREATE VIEW Enrollment_Summary (Student, Course, Score) AS
    SELECT s.Name, c.Title, e.Grade
    FROM Student s
    JOIN Enrollment e ON s.Stud_ID = e.Stud_ID
    JOIN Course c ON e.Course_ID = c.Course_ID;
```

---

## 3. Using Views

```
┌─────────────────────────────────────────────────────────────────────┐
│                         USING VIEWS                                  │
└─────────────────────────────────────────────────────────────────────┘


    Views behave like regular tables in SELECT statements:
    

    SELECT FROM VIEW:
    ──────────────────
    
    SELECT * FROM Student_Details;
    
    SELECT Name, Dept_Name FROM Student_Details WHERE Age > 20;
    
    SELECT * FROM Student_Details ORDER BY Age DESC LIMIT 5;


    JOIN WITH VIEWS:
    ─────────────────
    
    SELECT sd.Name, e.Course_ID, e.Grade
    FROM Student_Details sd
    JOIN Enrollment e ON sd.Stud_ID = e.Stud_ID;


    VIEWS IN SUBQUERIES:
    ─────────────────────
    
    SELECT * FROM Student
    WHERE Stud_ID IN (SELECT Stud_ID FROM CS_Students);


    NESTED VIEWS (View on View):
    ─────────────────────────────
    
    CREATE VIEW Young_CS_Students AS
    SELECT * FROM CS_Students WHERE Age < 21;
    
    ⚠️ Nested views can become complex and slow - use sparingly!


    HOW THE DATABASE PROCESSES VIEWS:
    ───────────────────────────────────
    
    Query: SELECT * FROM CS_Students WHERE Age = 20;
    
    Step 1: Expand view definition
    ┌─────────────────────────────────────────────────────────────┐
    │  SELECT * FROM (                                            │
    │      SELECT Stud_ID, Name, Age FROM Student                 │
    │      WHERE Dept_ID = 'D01'                                  │
    │  ) AS CS_Students                                           │
    │  WHERE Age = 20;                                            │
    └─────────────────────────────────────────────────────────────┘
    
    Step 2: Optimize and execute combined query
```

---

## 4. Advantages of Views

```
┌─────────────────────────────────────────────────────────────────────┐
│                    ADVANTAGES OF VIEWS                               │
└─────────────────────────────────────────────────────────────────────┘


    1. SIMPLIFICATION
    ──────────────────
    
    Complex queries become simple:
    
    Instead of:
    SELECT s.Name, d.Dept_Name, 
           (SELECT COUNT(*) FROM Enrollment e WHERE e.Stud_ID = s.Stud_ID) AS Courses
    FROM Student s
    JOIN Department d ON s.Dept_ID = d.Dept_ID
    WHERE d.Dept_Name = 'CS';
    
    With view:
    SELECT * FROM CS_Student_Details;  -- Much simpler!


    2. SECURITY
    ────────────
    
    Hide sensitive columns:
    
    -- Base table has salary information
    Employee(Emp_ID, Name, SSN, Salary, Dept_ID)
    
    -- View hides sensitive data
    CREATE VIEW Employee_Public AS
    SELECT Emp_ID, Name, Dept_ID FROM Employee;
    
    -- Grant access to view only
    GRANT SELECT ON Employee_Public TO public_users;
    
    Users see only: Emp_ID, Name, Dept_ID
    SSN and Salary are hidden!


    3. DATA INDEPENDENCE
    ─────────────────────
    
    If base table structure changes, view can maintain same interface:
    
    Before: Student(ID, Name)
    After:  Student(ID, First_Name, Last_Name)  -- Structure changed!
    
    View maintains compatibility:
    CREATE VIEW Student_Compat AS
    SELECT ID, CONCAT(First_Name, ' ', Last_Name) AS Name
    FROM Student;
    
    Old applications still work with the view!


    4. LOGICAL DATA ORGANIZATION
    ─────────────────────────────
    
    Present data from multiple tables as one logical unit:
    
    ┌──────────┐  ┌──────────┐  ┌──────────┐
    │ Student  │  │Department│  │  Course  │
    └────┬─────┘  └────┬─────┘  └────┬─────┘
         │             │             │
         └─────────────┼─────────────┘
                       ↓
              ┌────────────────────┐
              │  Student_Full_Info │  <- Single logical view
              │  (Combined data)   │
              └────────────────────┘


    SUMMARY:
    ─────────
    
    ┌───────────────────┬─────────────────────────────────────────┐
    │    Advantage      │              Description                │
    ├───────────────────┼─────────────────────────────────────────┤
    │ Simplification    │ Hide query complexity                   │
    │ Security          │ Restrict access to columns/rows        │
    │ Independence      │ Insulate from schema changes           │
    │ Organization      │ Present unified view of data           │
    │ Reusability       │ Define once, use many times            │
    └───────────────────┴─────────────────────────────────────────┘
```

---

## 5. Modifying and Dropping Views

```
┌─────────────────────────────────────────────────────────────────────┐
│                 MODIFYING AND DROPPING VIEWS                         │
└─────────────────────────────────────────────────────────────────────┘


    ALTER VIEW (or CREATE OR REPLACE):
    ────────────────────────────────────
    
    -- Most databases use CREATE OR REPLACE
    CREATE OR REPLACE VIEW CS_Students AS
    SELECT Stud_ID, Name, Age, Email
    FROM Student
    WHERE Dept_ID = 'D01';
    
    -- Some databases support ALTER VIEW
    ALTER VIEW CS_Students AS
    SELECT Stud_ID, Name FROM Student WHERE Dept_ID = 'D01';


    DROP VIEW:
    ───────────
    
    DROP VIEW view_name;
    
    DROP VIEW CS_Students;
    
    -- Drop if exists (no error if missing)
    DROP VIEW IF EXISTS CS_Students;
    
    -- Drop with dependent views
    DROP VIEW view_name CASCADE;


    RENAME VIEW:
    ─────────────
    
    -- Varies by database
    ALTER VIEW old_name RENAME TO new_name;
    
    -- Or
    RENAME VIEW old_name TO new_name;


    SHOW VIEW DEFINITION:
    ──────────────────────
    
    -- MySQL
    SHOW CREATE VIEW view_name;
    
    -- PostgreSQL
    SELECT definition FROM pg_views WHERE viewname = 'view_name';
    
    -- SQL Server
    sp_helptext 'view_name';
```

---

## 6. Updatable Views

```
┌─────────────────────────────────────────────────────────────────────┐
│                      UPDATABLE VIEWS                                 │
└─────────────────────────────────────────────────────────────────────┘

    Some views can be used with INSERT, UPDATE, DELETE.
    These modifications affect the underlying base table.
    

    CONDITIONS FOR UPDATABLE VIEW:
    ───────────────────────────────
    
    ┌───────────────────────────────────────────────────────────────────┐
    │                                                                    │
    │   View is UPDATABLE if:                                           │
    │   ─────────────────────                                           │
    │   ✓ Based on single table                                         │
    │   ✓ Contains PRIMARY KEY of base table                           │
    │   ✓ No aggregate functions (SUM, COUNT, AVG, etc.)               │
    │   ✓ No GROUP BY or HAVING                                        │
    │   ✓ No DISTINCT                                                   │
    │   ✓ No subqueries in SELECT                                      │
    │   ✓ No UNION, INTERSECT, EXCEPT                                  │
    │   ✓ No calculated columns                                         │
    │                                                                    │
    └───────────────────────────────────────────────────────────────────┘


    UPDATABLE VIEW EXAMPLE:
    ────────────────────────
    
    CREATE VIEW CS_Students AS
    SELECT Stud_ID, Name, Age, Dept_ID
    FROM Student
    WHERE Dept_ID = 'D01';
    
    -- UPDATE through view
    UPDATE CS_Students SET Age = 22 WHERE Stud_ID = 'S101';
    ✓ Works! Updates Student table
    
    -- INSERT through view
    INSERT INTO CS_Students (Stud_ID, Name, Age, Dept_ID)
    VALUES ('S106', 'Frank', 20, 'D01');
    ✓ Works! Inserts into Student table
    
    -- DELETE through view
    DELETE FROM CS_Students WHERE Stud_ID = 'S106';
    ✓ Works! Deletes from Student table


    NON-UPDATABLE VIEW EXAMPLES:
    ─────────────────────────────
    
    -- Aggregate function - NOT updatable
    CREATE VIEW Dept_Stats AS
    SELECT Dept_ID, COUNT(*) AS Count FROM Student GROUP BY Dept_ID;
    
    -- Join - NOT updatable
    CREATE VIEW Student_Dept AS
    SELECT s.Name, d.Dept_Name 
    FROM Student s JOIN Department d ON s.Dept_ID = d.Dept_ID;
    
    -- Calculated column - NOT updatable
    CREATE VIEW Student_Age AS
    SELECT Name, Age, Age + 10 AS Future_Age FROM Student;


    VISUALIZATION:
    ───────────────
    
    ┌─────────────────────────────────────────────────────────────────┐
    │                                                                  │
    │   UPDATABLE:                    NON-UPDATABLE:                  │
    │                                                                  │
    │   ┌───────────┐                ┌───────────┐                    │
    │   │   VIEW    │                │   VIEW    │                    │
    │   └─────┬─────┘                └─────┬─────┘                    │
    │         │                            │                          │
    │         ↕                            ↓ (one-way only)           │
    │   ┌───────────┐                ┌───────────┐                    │
    │   │   TABLE   │                │ TABLE(S)  │                    │
    │   └───────────┘                │ + Agg/Join│                    │
    │                                └───────────┘                    │
    │                                                                  │
    └─────────────────────────────────────────────────────────────────┘
```

---

## 7. WITH CHECK OPTION

```
┌─────────────────────────────────────────────────────────────────────┐
│                     WITH CHECK OPTION                                │
└─────────────────────────────────────────────────────────────────────┘

    Ensures that INSERT/UPDATE through view only affects rows
    that remain visible through the view.
    

    WITHOUT CHECK OPTION (Problem):
    ─────────────────────────────────
    
    CREATE VIEW CS_Students AS
    SELECT Stud_ID, Name, Age, Dept_ID
    FROM Student
    WHERE Dept_ID = 'D01';
    
    -- This works but row disappears from view!
    UPDATE CS_Students SET Dept_ID = 'D02' WHERE Stud_ID = 'S101';
    
    Alice is moved to D02 (Math) but CS_Students view shows D01 only!
    Alice "disappears" from view but exists in table.


    WITH CHECK OPTION (Solution):
    ───────────────────────────────
    
    CREATE VIEW CS_Students AS
    SELECT Stud_ID, Name, Age, Dept_ID
    FROM Student
    WHERE Dept_ID = 'D01'
    WITH CHECK OPTION;
    
    -- This is now BLOCKED:
    UPDATE CS_Students SET Dept_ID = 'D02' WHERE Stud_ID = 'S101';
    
    ERROR: CHECK OPTION failed - row would no longer be visible!


    VISUALIZATION:
    ───────────────
    
    Without CHECK OPTION:
    ┌─────────────────────────────────────────────────────────────────┐
    │                                                                  │
    │   CS_Students View (D01 only)    Student Table                  │
    │   ┌───────────────────────┐      ┌───────────────────────────┐  │
    │   │ S101 │ Alice │ D01    │  →   │ S101 │ Alice │ D02  ← Changed│
    │   │ S103 │Charlie│ D01    │      │ S102 │  Bob  │ D02        │  │
    │   └───────────────────────┘      │ S103 │Charlie│ D01        │  │
    │                                  └───────────────────────────┘  │
    │   After update: Alice gone!      Alice still in table!         │
    │                                                                  │
    └─────────────────────────────────────────────────────────────────┘
    
    With CHECK OPTION:
    ┌─────────────────────────────────────────────────────────────────┐
    │                                                                  │
    │   UPDATE to D02?  ──→  CHECK: Would row still be in view?      │
    │                                                                  │
    │                        D02 ≠ D01  ──→  NO! REJECT UPDATE!      │
    │                                                                  │
    └─────────────────────────────────────────────────────────────────┘


    LOCAL vs CASCADED CHECK:
    ──────────────────────────
    
    -- LOCAL: Only checks this view's condition
    CREATE VIEW v1 AS ... WITH LOCAL CHECK OPTION;
    
    -- CASCADED: Also checks parent views' conditions
    CREATE VIEW v1 AS ... WITH CASCADED CHECK OPTION;  -- Default
```

---

## 8. Materialized Views

```
┌─────────────────────────────────────────────────────────────────────┐
│                    MATERIALIZED VIEWS                                │
└─────────────────────────────────────────────────────────────────────┘

    A materialized view STORES the query result physically.
    Unlike regular views, data is pre-computed and cached.
    

    REGULAR VIEW vs MATERIALIZED VIEW:
    ────────────────────────────────────
    
    ┌──────────────────────┬──────────────────────────────────────────┐
    │     Regular View     │         Materialized View                │
    ├──────────────────────┼──────────────────────────────────────────┤
    │ Stores query only    │ Stores query AND result data            │
    │ No physical storage  │ Physical storage required               │
    │ Always up-to-date    │ May be stale (needs refresh)            │
    │ Slower (runs query)  │ Faster (reads cached data)              │
    │ No refresh needed    │ Needs manual/automatic refresh          │
    └──────────────────────┴──────────────────────────────────────────┘


    VISUALIZATION:
    ───────────────
    
    REGULAR VIEW:
    ┌───────────────┐          ┌───────────────┐
    │     Query     │  ──────→ │  Base Tables  │ ──→ Result (each time)
    │   Definition  │          │               │
    └───────────────┘          └───────────────┘
    
    
    MATERIALIZED VIEW:
    ┌───────────────┐          ┌───────────────┐
    │  Stored Data  │          │  Base Tables  │
    │  (Pre-cached) │    ←───  │   (Refresh)   │
    └───────────────┘          └───────────────┘
          │
          └──→ Result (fast read)


    SYNTAX (PostgreSQL):
    ──────────────────────
    
    CREATE MATERIALIZED VIEW mv_student_stats AS
    SELECT Dept_ID, COUNT(*) AS Count, AVG(Age) AS Avg_Age
    FROM Student
    GROUP BY Dept_ID;
    
    -- Refresh the materialized view
    REFRESH MATERIALIZED VIEW mv_student_stats;
    
    -- Refresh concurrently (allows reads during refresh)
    REFRESH MATERIALIZED VIEW CONCURRENTLY mv_student_stats;


    WHEN TO USE:
    ─────────────
    
    ┌───────────────────┬─────────────────────────────────────────────┐
    │  Use Materialized │               When                          │
    ├───────────────────┼─────────────────────────────────────────────┤
    │        YES        │ Complex aggregations, expensive queries    │
    │        YES        │ Data doesn't change frequently             │
    │        YES        │ Query speed is critical                    │
    │        NO         │ Need real-time data                        │
    │        NO         │ Storage is limited                         │
    │        NO         │ Data changes very frequently               │
    └───────────────────┴─────────────────────────────────────────────┘


    REFRESH STRATEGIES:
    ────────────────────
    
    • Complete refresh: Recreate entire view
    • Incremental refresh: Update only changed rows
    • Manual refresh: DBA triggers refresh
    • Automatic refresh: Scheduled (every hour, daily, etc.)
    • On-commit refresh: Refresh after each transaction
```

---

## 9. Views and Security

```
┌─────────────────────────────────────────────────────────────────────┐
│                   VIEWS FOR SECURITY                                 │
└─────────────────────────────────────────────────────────────────────┘


    ROW-LEVEL SECURITY:
    ────────────────────
    
    Only show rows user is allowed to see:
    
    CREATE VIEW My_Department_Students AS
    SELECT * FROM Student
    WHERE Dept_ID = (SELECT Dept_ID FROM Employee 
                     WHERE Emp_ID = CURRENT_USER);
    
    Each user only sees their department's students!


    COLUMN-LEVEL SECURITY:
    ───────────────────────
    
    Hide sensitive columns:
    
    -- Base table
    Employee(ID, Name, SSN, Salary, Phone, Address)
    
    -- HR view (sees salary)
    CREATE VIEW HR_Employee AS
    SELECT ID, Name, Salary, Phone FROM Employee;
    
    -- Public view (no sensitive data)
    CREATE VIEW Public_Employee AS
    SELECT ID, Name, Phone FROM Employee;
    
    
    ACCESS CONTROL:
    ────────────────
    
    -- Revoke direct table access
    REVOKE SELECT ON Employee FROM public_user;
    
    -- Grant view access only
    GRANT SELECT ON Public_Employee TO public_user;


    SECURITY VISUALIZATION:
    ────────────────────────
    
    ┌─────────────────────────────────────────────────────────────────┐
    │                                                                  │
    │   Employee Table (Full Data)                                    │
    │   ┌────┬───────┬──────────┬────────┬─────────┬─────────────┐   │
    │   │ ID │ Name  │   SSN    │ Salary │  Phone  │   Address   │   │
    │   └────┴───────┴──────────┴────────┴─────────┴─────────────┘   │
    │                          │                                      │
    │         ┌────────────────┼────────────────┐                    │
    │         ↓                ↓                ↓                    │
    │   ┌───────────┐    ┌───────────┐    ┌───────────┐             │
    │   │ HR View   │    │ Manager   │    │ Public    │             │
    │   │ ID,Name,  │    │ View      │    │ View      │             │
    │   │ Salary    │    │ ID,Name,  │    │ ID,Name   │             │
    │   │           │    │ Phone     │    │           │             │
    │   └───────────┘    └───────────┘    └───────────┘             │
    │                                                                  │
    └─────────────────────────────────────────────────────────────────┘
```

---

## 📊 Summary Table

| Concept | Description |
|---------|-------------|
| **View** | Virtual table defined by a query |
| **Base Table** | Physical table storing actual data |
| **Updatable View** | View that allows INSERT/UPDATE/DELETE |
| **Materialized View** | View that stores computed results |
| **WITH CHECK OPTION** | Prevents updates that hide rows from view |

| View Type | Stores Data? | Always Current? | Speed |
|-----------|--------------|-----------------|-------|
| **Regular View** | No (query only) | Yes | Slower |
| **Materialized View** | Yes (cached) | No (needs refresh) | Faster |

| Updatable Criteria | Description |
|-------------------|-------------|
| Single table | Must be from one base table |
| Contains PK | Must include primary key |
| No aggregates | No SUM, COUNT, AVG, etc. |
| No GROUP BY | No grouping or HAVING |
| No DISTINCT | No duplicate elimination |
| No calculated columns | No expressions/computations |

---

## ❓ Quick Revision Questions

1. **What is the difference between a view and a table?**
   <details>
   <summary>Click for Answer</summary>
   A table stores actual data physically. A view stores only a query definition and retrieves data from base tables each time it's accessed. Views are virtual; tables are physical.
   </details>

2. **What are three main advantages of using views?**
   <details>
   <summary>Click for Answer</summary>
   (1) Simplification - hide complex queries behind simple view names. (2) Security - restrict access to specific rows/columns. (3) Data independence - insulate applications from schema changes.
   </details>

3. **What conditions make a view updatable?**
   <details>
   <summary>Click for Answer</summary>
   View must be: based on single table, contain primary key, have no aggregate functions, no GROUP BY/HAVING, no DISTINCT, no subqueries in SELECT, no UNION, no calculated columns.
   </details>

4. **What is the purpose of WITH CHECK OPTION?**
   <details>
   <summary>Click for Answer</summary>
   WITH CHECK OPTION prevents INSERT/UPDATE operations that would make the row invisible to the view. It ensures data modified through the view remains visible through the view.
   </details>

5. **What is a materialized view and when would you use it?**
   <details>
   <summary>Click for Answer</summary>
   A materialized view stores the query result physically (pre-computed). Use when: query is complex/expensive, data doesn't change frequently, read performance is critical. Requires periodic refresh to stay current.
   </details>

6. **How can views enhance database security?**
   <details>
   <summary>Click for Answer</summary>
   Views provide: (1) Column-level security - hide sensitive columns like salary or SSN. (2) Row-level security - show only rows user is authorized to see. Users get access to views, not base tables.
   </details>

---

## 🔗 Navigation

| Previous | Up | Next |
|----------|----|----|
| [← Subqueries and Nested Queries](04-subqueries-nested-queries.md) | [📚 Table of Contents](../README.md) | [DCL and TCL →](06-dcl-tcl.md) |

---

*Last Updated: January 2026*
