# Chapter 3.4: Subqueries and Nested Queries

[← Back to Table of Contents](../README.md)

---

## 📚 Chapter Overview

**Subqueries** (also called nested queries or inner queries) are queries placed inside another query. They allow you to use the result of one query as input to another, enabling complex data retrieval in a single statement.

```
┌─────────────────────────────────────────────────────────────────┐
│                     CHAPTER LEARNING GOALS                       │
├─────────────────────────────────────────────────────────────────┤
│  • Understand when and why to use subqueries                    │
│  • Master scalar, row, and table subqueries                     │
│  • Learn subqueries in WHERE, FROM, and SELECT clauses         │
│  • Use operators: IN, EXISTS, ANY, ALL                          │
│  • Distinguish correlated vs non-correlated subqueries          │
└─────────────────────────────────────────────────────────────────┘
```

---

## 1. What is a Subquery?

```
┌─────────────────────────────────────────────────────────────────────┐
│                     SUBQUERY BASICS                                  │
└─────────────────────────────────────────────────────────────────────┘

    A subquery is a SELECT statement nested inside another SQL statement.
    
    
    STRUCTURE:
    ───────────
    
    SELECT columns
    FROM table
    WHERE column = (SELECT column FROM table WHERE condition);
                   └────────────────────────────────────────┘
                           ↑ This is the SUBQUERY
    
    
    TERMINOLOGY:
    ─────────────
    
    ┌────────────────────────────────────────────────────────────────┐
    │                                                                 │
    │   OUTER QUERY (Main Query)        INNER QUERY (Subquery)       │
    │   ─────────────────────           ───────────────────          │
    │   Contains the subquery           The nested SELECT            │
    │   Executes last                   Executes first               │
    │   Uses subquery result            Provides result to outer     │
    │                                                                 │
    └────────────────────────────────────────────────────────────────┘
    
    
    EXAMPLE:
    ─────────
    
    "Find students older than the average age"
    
    SELECT Name FROM Student
    WHERE Age > (SELECT AVG(Age) FROM Student);
    
    Step 1: Inner query calculates AVG(Age) = 21.5
    Step 2: Outer query finds students where Age > 21.5
```

---

## 2. Sample Tables

```
┌─────────────────────────────────────────────────────────────────────┐
│                       SAMPLE TABLES                                  │
└─────────────────────────────────────────────────────────────────────┘

    Student                              Department
    ┌─────────┬─────────┬─────┬─────────┐    ┌─────────┬──────────────┐
    │ Stud_ID │  Name   │ Age │ Dept_ID │    │ Dept_ID │  Dept_Name   │
    ├─────────┼─────────┼─────┼─────────┤    ├─────────┼──────────────┤
    │  S101   │  Alice  │ 20  │   D01   │    │   D01   │      CS      │
    │  S102   │   Bob   │ 22  │   D02   │    │   D02   │     Math     │
    │  S103   │ Charlie │ 21  │   D01   │    │   D03   │   Physics    │
    │  S104   │  Diana  │ 23  │   D03   │    └─────────┴──────────────┘
    │  S105   │   Eve   │ 19  │   D02   │
    └─────────┴─────────┴─────┴─────────┘

    Enrollment                           Course
    ┌─────────┬───────────┬───────┐      ┌───────────┬────────────┬─────────┐
    │ Stud_ID │ Course_ID │ Grade │      │ Course_ID │   Title    │ Credits │
    ├─────────┼───────────┼───────┤      ├───────────┼────────────┼─────────┤
    │  S101   │   CS101   │   A   │      │   CS101   │  Database  │    3    │
    │  S101   │   CS102   │   B   │      │   CS102   │  Networks  │    3    │
    │  S102   │   CS101   │   A   │      │   MA101   │  Calculus  │    4    │
    │  S103   │   MA101   │   B   │      │   PH101   │  Mechanics │    4    │
    │  S104   │   PH101   │   A   │      └───────────┴────────────┴─────────┘
    └─────────┴───────────┴───────┘
```

---

## 3. Types of Subqueries by Return Value

```
┌─────────────────────────────────────────────────────────────────────┐
│                  TYPES OF SUBQUERIES                                 │
└─────────────────────────────────────────────────────────────────────┘


    1. SCALAR SUBQUERY - Returns single value
    ──────────────────────────────────────────
    
    Returns exactly one row and one column.
    
    SELECT Name FROM Student
    WHERE Age = (SELECT MAX(Age) FROM Student);
              ↑
              Returns: 23 (single value)
    
    Result: Diana (age 23)
    
    
    2. ROW SUBQUERY - Returns single row (multiple columns)
    ───────────────────────────────────────────────────────
    
    SELECT * FROM Student
    WHERE (Age, Dept_ID) = (SELECT MAX(Age), Dept_ID 
                            FROM Student 
                            WHERE Dept_ID = 'D01');
                          ↑
                          Returns: (21, 'D01') - one row
    
    
    3. TABLE SUBQUERY - Returns multiple rows and columns
    ──────────────────────────────────────────────────────
    
    SELECT * FROM Student
    WHERE Dept_ID IN (SELECT Dept_ID FROM Department 
                      WHERE Dept_Name LIKE 'C%');
                     ↑
                     Returns: D01 (could be multiple values)
    
    Result: Alice, Charlie (CS department)


    VISUALIZATION:
    ───────────────
    
    ┌──────────────────────────────────────────────────────────────┐
    │                                                               │
    │   SCALAR          ROW              TABLE                     │
    │   ┌───┐           ┌───┬───┐        ┌───┬───┐                │
    │   │ 1 │           │ 1 │ A │        │ 1 │ A │                │
    │   └───┘           └───┴───┘        ├───┼───┤                │
    │                                    │ 2 │ B │                │
    │   One value       One row          ├───┼───┤                │
    │                                    │ 3 │ C │                │
    │                                    └───┴───┘                │
    │                                    Multiple rows             │
    │                                                               │
    └──────────────────────────────────────────────────────────────┘
```

---

## 4. Subqueries in WHERE Clause

```
┌─────────────────────────────────────────────────────────────────────┐
│                   SUBQUERIES IN WHERE CLAUSE                         │
└─────────────────────────────────────────────────────────────────────┘


    WITH COMPARISON OPERATORS (=, >, <, etc.):
    ────────────────────────────────────────────
    
    Requires SCALAR subquery (single value)
    
    -- Students older than average
    SELECT Name, Age FROM Student
    WHERE Age > (SELECT AVG(Age) FROM Student);
    
    Execution:
    1. Inner: AVG(Age) = (20+22+21+23+19)/5 = 21
    2. Outer: WHERE Age > 21
    
    Result:
    ┌─────────┬─────┐
    │  Name   │ Age │
    ├─────────┼─────┤
    │   Bob   │ 22  │
    │  Diana  │ 23  │
    └─────────┴─────┘


    WITH IN OPERATOR:
    ──────────────────
    
    -- Students in CS or Math department
    SELECT Name FROM Student
    WHERE Dept_ID IN (SELECT Dept_ID FROM Department 
                      WHERE Dept_Name IN ('CS', 'Math'));
    
    Execution:
    1. Inner: Returns D01, D02
    2. Outer: WHERE Dept_ID IN ('D01', 'D02')
    
    Result: Alice, Bob, Charlie, Eve


    WITH NOT IN:
    ─────────────
    
    -- Students NOT enrolled in any course
    SELECT Name FROM Student
    WHERE Stud_ID NOT IN (SELECT DISTINCT Stud_ID FROM Enrollment);
    
    Result: Eve (S105 not in Enrollment)
    
    ⚠️ WARNING: NOT IN with NULL values!
    If inner query returns NULL, outer query returns empty set.
    Use NOT EXISTS instead for safety.
```

---

## 5. EXISTS and NOT EXISTS

```
┌─────────────────────────────────────────────────────────────────────┐
│                      EXISTS OPERATOR                                 │
└─────────────────────────────────────────────────────────────────────┘

    EXISTS checks if subquery returns ANY rows (TRUE/FALSE).
    Doesn't care about the actual values returned.
    

    EXISTS:
    ────────
    
    -- Students who are enrolled in at least one course
    SELECT s.Name FROM Student s
    WHERE EXISTS (SELECT 1 FROM Enrollment e 
                  WHERE e.Stud_ID = s.Stud_ID);
    
    Execution:
    For each student, check if ANY enrollment exists
    
    ┌─────────┬─────────────────────────────────────┐
    │ Student │ EXISTS(Enrollment)?                 │
    ├─────────┼─────────────────────────────────────┤
    │  Alice  │ Yes (S101 has enrollments)         │ ✓
    │   Bob   │ Yes (S102 has enrollments)         │ ✓
    │ Charlie │ Yes (S103 has enrollments)         │ ✓
    │  Diana  │ Yes (S104 has enrollments)         │ ✓
    │   Eve   │ No (S105 has no enrollments)       │ ✗
    └─────────┴─────────────────────────────────────┘
    
    Result: Alice, Bob, Charlie, Diana


    NOT EXISTS:
    ────────────
    
    -- Students who are NOT enrolled in any course
    SELECT s.Name FROM Student s
    WHERE NOT EXISTS (SELECT 1 FROM Enrollment e 
                      WHERE e.Stud_ID = s.Stud_ID);
    
    Result: Eve


    EXISTS vs IN:
    ──────────────
    
    ┌─────────────────────────────────────────────────────────────────┐
    │                                                                  │
    │   IN                              EXISTS                        │
    │   ──                              ──────                        │
    │   Compares values                 Checks for row existence      │
    │   Subquery runs once              Subquery runs for each row    │
    │   Problem with NULL               No NULL issue                 │
    │   Better for small subqueries     Better for large outer table  │
    │                                                                  │
    └─────────────────────────────────────────────────────────────────┘
    
    
    SELECT 1 EXPLANATION:
    ──────────────────────
    
    EXISTS doesn't care WHAT you select, only IF rows exist.
    SELECT 1 or SELECT * both work the same.
    SELECT 1 is convention to show we only check existence.
```

---

## 6. ANY and ALL

```
┌─────────────────────────────────────────────────────────────────────┐
│                       ANY AND ALL                                    │
└─────────────────────────────────────────────────────────────────────┘


    ANY (SOME):
    ────────────
    
    Returns TRUE if comparison is TRUE for ANY value returned.
    
    -- Students older than ANY Math student
    SELECT Name, Age FROM Student
    WHERE Age > ANY (SELECT Age FROM Student WHERE Dept_ID = 'D02');
    
    Math students: Bob (22), Eve (19)
    Age > ANY (22, 19) means Age > 19 (the minimum)
    
    Result: Alice(20), Bob(22), Charlie(21), Diana(23)
    (Everyone older than 19)
    
    
    ANY EQUIVALENTS:
    
    = ANY  is same as  IN
    > ANY  means > MIN of list
    < ANY  means < MAX of list


    ALL:
    ─────
    
    Returns TRUE if comparison is TRUE for ALL values returned.
    
    -- Students older than ALL Math students
    SELECT Name, Age FROM Student
    WHERE Age > ALL (SELECT Age FROM Student WHERE Dept_ID = 'D02');
    
    Math students: Bob (22), Eve (19)
    Age > ALL (22, 19) means Age > 22 (the maximum)
    
    Result: Diana(23)
    (Only Diana is older than all Math students)
    
    
    ALL EQUIVALENTS:
    
    <> ALL  is same as  NOT IN
    > ALL   means > MAX of list
    < ALL   means < MIN of list


    VISUALIZATION:
    ───────────────
    
    List = (19, 22)
    
    ┌───────────────────────────────────────────────────────────────┐
    │                                                                │
    │   Age > ANY (19, 22)            Age > ALL (19, 22)            │
    │   ─────────────────             ─────────────────             │
    │   Age > 19                      Age > 22                      │
    │   Passes: 20, 21, 22, 23       Passes: 23                     │
    │                                                                │
    │   0────19────22────30          0────19────22────30            │
    │         │<───────────│              │         │<────────│     │
    │         Greater than ANY           Greater than ALL           │
    │                                                                │
    └───────────────────────────────────────────────────────────────┘
```

---

## 7. Correlated vs Non-Correlated Subqueries

```
┌─────────────────────────────────────────────────────────────────────┐
│              CORRELATED VS NON-CORRELATED SUBQUERIES                 │
└─────────────────────────────────────────────────────────────────────┘


    NON-CORRELATED SUBQUERY:
    ─────────────────────────
    
    • Independent of outer query
    • Executes ONCE
    • Faster
    
    SELECT Name FROM Student
    WHERE Age > (SELECT AVG(Age) FROM Student);
                └────────────────────────────┘
                        ↑
        No reference to outer query's table
        
    Execution:
    1. Run inner query once → 21
    2. Run outer query: WHERE Age > 21


    CORRELATED SUBQUERY:
    ─────────────────────
    
    • Depends on outer query
    • Executes FOR EACH ROW of outer query
    • Slower but more powerful
    
    SELECT s.Name FROM Student s
    WHERE s.Age > (SELECT AVG(Age) FROM Student s2 
                   WHERE s2.Dept_ID = s.Dept_ID);
                                      ↑
                    Reference to outer query (s.Dept_ID)
    
    Execution:
    For each student:
    1. Calculate avg age of THEIR department
    2. Check if their age > that average
    
    ┌─────────┬───────┬─────────┬───────────┬────────────────┐
    │  Name   │ Dept  │   Age   │ Dept Avg  │ Age > Avg?     │
    ├─────────┼───────┼─────────┼───────────┼────────────────┤
    │  Alice  │  D01  │   20    │   20.5    │   No           │
    │   Bob   │  D02  │   22    │   20.5    │   Yes ✓        │
    │ Charlie │  D01  │   21    │   20.5    │   Yes ✓        │
    │  Diana  │  D03  │   23    │   23      │   No           │
    │   Eve   │  D02  │   19    │   20.5    │   No           │
    └─────────┴───────┴─────────┴───────────┴────────────────┘
    
    Result: Bob, Charlie


    VISUALIZATION:
    ───────────────
    
    NON-CORRELATED:
    ┌─────────────────┐
    │  Inner Query    │────────→ Runs ONCE
    │    (Independent) │          Result: 21
    └─────────────────┘
            ↓
    ┌─────────────────┐
    │  Outer Query    │────────→ Uses 21 for all rows
    │    WHERE x > 21 │
    └─────────────────┘
    
    
    CORRELATED:
    ┌─────────────────┐      ┌─────────────────┐
    │  Outer Query    │      │  Inner Query    │
    │    Row 1        │─────→│  (Uses Row 1)   │──→ Result for Row 1
    │    Row 2        │─────→│  (Uses Row 2)   │──→ Result for Row 2
    │    Row 3        │─────→│  (Uses Row 3)   │──→ Result for Row 3
    │    ...          │      │  ...            │
    └─────────────────┘      └─────────────────┘
```

---

## 8. Subqueries in FROM Clause (Derived Tables)

```
┌─────────────────────────────────────────────────────────────────────┐
│               SUBQUERIES IN FROM CLAUSE                              │
└─────────────────────────────────────────────────────────────────────┘

    Subquery in FROM creates a temporary table (derived table).
    MUST give it an alias.
    

    EXAMPLE 1: Simple derived table
    ─────────────────────────────────
    
    SELECT dept_info.Dept_ID, dept_info.Student_Count
    FROM (SELECT Dept_ID, COUNT(*) AS Student_Count
          FROM Student
          GROUP BY Dept_ID) AS dept_info
    WHERE dept_info.Student_Count > 1;
    
    Inner query creates:
    ┌─────────┬───────────────┐
    │ Dept_ID │ Student_Count │
    ├─────────┼───────────────┤
    │   D01   │       2       │
    │   D02   │       2       │
    │   D03   │       1       │
    └─────────┴───────────────┘
    
    Outer query filters:
    ┌─────────┬───────────────┐
    │ Dept_ID │ Student_Count │
    ├─────────┼───────────────┤
    │   D01   │       2       │
    │   D02   │       2       │
    └─────────┴───────────────┘


    EXAMPLE 2: Join with derived table
    ────────────────────────────────────
    
    SELECT s.Name, ds.Student_Count
    FROM Student s
    JOIN (SELECT Dept_ID, COUNT(*) AS Student_Count
          FROM Student
          GROUP BY Dept_ID) AS ds ON s.Dept_ID = ds.Dept_ID;
    
    Result:
    ┌─────────┬───────────────┐
    │  Name   │ Student_Count │
    ├─────────┼───────────────┤
    │  Alice  │       2       │  ← D01 has 2 students
    │   Bob   │       2       │  ← D02 has 2 students
    │ Charlie │       2       │  ← D01 has 2 students
    │  Diana  │       1       │  ← D03 has 1 student
    │   Eve   │       2       │  ← D02 has 2 students
    └─────────┴───────────────┘


    VISUALIZATION:
    ───────────────
    
    ┌───────────────────────────────────────────────────────────────┐
    │                                                                │
    │   SELECT ... FROM (                                           │
    │                     ┌─────────────────────────────────────┐   │
    │                     │      SUBQUERY CREATES               │   │
    │                     │      TEMPORARY TABLE                │   │
    │                     │   ┌─────┬─────┬─────┐               │   │
    │                     │   │ ... │ ... │ ... │               │   │
    │                     │   ├─────┼─────┼─────┤               │   │
    │                     │   │     │     │     │               │   │
    │                     │   └─────┴─────┴─────┘               │   │
    │                     └─────────────────────────────────────┘   │
    │                   ) AS alias                                  │
    │                         ↑                                     │
    │                    MUST HAVE ALIAS                            │
    │                                                                │
    └───────────────────────────────────────────────────────────────┘
```

---

## 9. Subqueries in SELECT Clause

```
┌─────────────────────────────────────────────────────────────────────┐
│              SUBQUERIES IN SELECT CLAUSE                             │
└─────────────────────────────────────────────────────────────────────┘

    Scalar subquery in SELECT adds a computed column.
    MUST return exactly one value per row.
    

    EXAMPLE 1: Add department name to each student
    ─────────────────────────────────────────────────
    
    SELECT s.Name, s.Age,
           (SELECT d.Dept_Name FROM Department d 
            WHERE d.Dept_ID = s.Dept_ID) AS Department
    FROM Student s;
    
    Result:
    ┌─────────┬─────┬────────────┐
    │  Name   │ Age │ Department │
    ├─────────┼─────┼────────────┤
    │  Alice  │ 20  │     CS     │
    │   Bob   │ 22  │    Math    │
    │ Charlie │ 21  │     CS     │
    │  Diana  │ 23  │  Physics   │
    │   Eve   │ 19  │    Math    │
    └─────────┴─────┴────────────┘


    EXAMPLE 2: Count enrollments for each student
    ───────────────────────────────────────────────
    
    SELECT s.Name,
           (SELECT COUNT(*) FROM Enrollment e 
            WHERE e.Stud_ID = s.Stud_ID) AS Course_Count
    FROM Student s;
    
    Result:
    ┌─────────┬──────────────┐
    │  Name   │ Course_Count │
    ├─────────┼──────────────┤
    │  Alice  │      2       │
    │   Bob   │      1       │
    │ Charlie │      1       │
    │  Diana  │      1       │
    │   Eve   │      0       │
    └─────────┴──────────────┘


    EXAMPLE 3: Compare to average
    ───────────────────────────────
    
    SELECT Name, Age,
           Age - (SELECT AVG(Age) FROM Student) AS Diff_From_Avg
    FROM Student;
    
    Result:
    ┌─────────┬─────┬───────────────┐
    │  Name   │ Age │ Diff_From_Avg │
    ├─────────┼─────┼───────────────┤
    │  Alice  │ 20  │     -1.0      │
    │   Bob   │ 22  │     +1.0      │
    │ Charlie │ 21  │      0.0      │
    │  Diana  │ 23  │     +2.0      │
    │   Eve   │ 19  │     -2.0      │
    └─────────┴─────┴───────────────┘
```

---

## 10. Complex Subquery Examples

```
┌─────────────────────────────────────────────────────────────────────┐
│                   COMPLEX SUBQUERY EXAMPLES                          │
└─────────────────────────────────────────────────────────────────────┘


    EXAMPLE 1: Students with highest grade in each course
    ───────────────────────────────────────────────────────
    
    SELECT e1.Course_ID, s.Name, e1.Grade
    FROM Enrollment e1
    JOIN Student s ON e1.Stud_ID = s.Stud_ID
    WHERE e1.Grade = (SELECT MIN(e2.Grade)  -- 'A' is min alphabetically
                      FROM Enrollment e2
                      WHERE e2.Course_ID = e1.Course_ID);
    
    Result: Students with 'A' grade in their respective courses


    EXAMPLE 2: Courses that no student has enrolled in
    ────────────────────────────────────────────────────
    
    SELECT Title FROM Course
    WHERE Course_ID NOT IN (SELECT DISTINCT Course_ID FROM Enrollment);
    
    OR using EXISTS:
    
    SELECT c.Title FROM Course c
    WHERE NOT EXISTS (SELECT 1 FROM Enrollment e 
                      WHERE e.Course_ID = c.Course_ID);


    EXAMPLE 3: Students enrolled in more courses than average
    ───────────────────────────────────────────────────────────
    
    SELECT s.Name, COUNT(e.Course_ID) AS Courses
    FROM Student s
    LEFT JOIN Enrollment e ON s.Stud_ID = e.Stud_ID
    GROUP BY s.Stud_ID, s.Name
    HAVING COUNT(e.Course_ID) > (SELECT AVG(course_count)
                                  FROM (SELECT COUNT(*) AS course_count
                                        FROM Enrollment
                                        GROUP BY Stud_ID) AS avg_calc);


    EXAMPLE 4: Nested subqueries (3 levels)
    ─────────────────────────────────────────
    
    "Find students in departments that offer courses with more than 3 credits"
    
    SELECT Name FROM Student
    WHERE Dept_ID IN (
        SELECT Dept_ID FROM Course
        WHERE Credits > 3
        AND Course_ID IN (
            SELECT Course_ID FROM Enrollment
        )
    );


    EXECUTION ORDER VISUALIZATION:
    ────────────────────────────────
    
    Level 1 (Innermost): Get enrolled course IDs
    Level 2: Get departments offering those courses (Credits > 3)
    Level 3 (Outermost): Get students in those departments
    
    ┌─────────────────────────────────────────────────┐
    │                                                  │
    │   Level 3: SELECT Name FROM Student             │
    │   WHERE Dept_ID IN (                            │
    │       ┌─────────────────────────────────────┐   │
    │       │ Level 2: SELECT Dept_ID FROM Course │   │
    │       │ WHERE Credits > 3 AND Course_ID IN ( │  │
    │       │   ┌─────────────────────────────────┐│  │
    │       │   │ Level 1: SELECT Course_ID      ││  │
    │       │   │ FROM Enrollment                 ││  │
    │       │   └─────────────────────────────────┘│  │
    │       │ )                                    │   │
    │       └─────────────────────────────────────┘   │
    │   )                                             │
    │                                                  │
    └─────────────────────────────────────────────────┘
```

---

## 📊 Summary Table

| Subquery Type | Returns | Example Use |
|---------------|---------|-------------|
| **Scalar** | Single value | `WHERE x = (SELECT MAX(y)...)` |
| **Row** | Single row | `WHERE (a,b) = (SELECT x,y...)` |
| **Table** | Multiple rows/cols | `WHERE x IN (SELECT...)` |
| **Correlated** | Depends on outer | `WHERE x > (SELECT... WHERE s.id = o.id)` |
| **Non-correlated** | Independent | `WHERE x > (SELECT AVG(y)...)` |

| Location | Purpose | Example |
|----------|---------|---------|
| **WHERE** | Filter rows | `WHERE id IN (SELECT...)` |
| **FROM** | Derived table | `FROM (SELECT...) AS t` |
| **SELECT** | Computed column | `SELECT (SELECT COUNT(*)...)` |

| Operator | Description | Returns TRUE when |
|----------|-------------|-------------------|
| **IN** | Set membership | Value in subquery result |
| **NOT IN** | Not in set | Value not in result (⚠️ NULL) |
| **EXISTS** | Existence check | Subquery returns any rows |
| **NOT EXISTS** | Non-existence | Subquery returns no rows |
| **ANY** | Compare to any | Comparison TRUE for at least one |
| **ALL** | Compare to all | Comparison TRUE for all |

---

## ❓ Quick Revision Questions

1. **What is the difference between correlated and non-correlated subqueries?**
   <details>
   <summary>Click for Answer</summary>
   Non-correlated subqueries are independent and execute once. Correlated subqueries reference the outer query and execute once for each row of the outer query. Non-correlated is faster; correlated is more flexible.
   </details>

2. **Why should you prefer NOT EXISTS over NOT IN?**
   <details>
   <summary>Click for Answer</summary>
   NOT IN has problems with NULL values. If the subquery returns any NULL, NOT IN returns empty result (unknown comparison). NOT EXISTS doesn't have this issue and handles NULLs correctly.
   </details>

3. **What is the difference between ANY and ALL?**
   <details>
   <summary>Click for Answer</summary>
   ANY returns TRUE if comparison is TRUE for at least one value (> ANY means > minimum). ALL returns TRUE if comparison is TRUE for all values (> ALL means > maximum).
   </details>

4. **Can you use a subquery in the SELECT clause?**
   <details>
   <summary>Click for Answer</summary>
   Yes, but it must be a scalar subquery (return exactly one value). It's often a correlated subquery that computes a value for each row. Example: SELECT name, (SELECT COUNT(*) FROM orders o WHERE o.customer_id = c.id) FROM customers c.
   </details>

5. **What must you do when using a subquery in the FROM clause?**
   <details>
   <summary>Click for Answer</summary>
   You MUST provide an alias for the derived table. Example: SELECT * FROM (SELECT id, name FROM users) AS user_subset. Without the alias, SQL will throw an error.
   </details>

6. **Write a query to find employees earning more than their department's average salary.**
   <details>
   <summary>Click for Answer</summary>
   SELECT e.name, e.salary FROM Employee e WHERE e.salary > (SELECT AVG(e2.salary) FROM Employee e2 WHERE e2.dept_id = e.dept_id); This is a correlated subquery that computes average for each employee's department.
   </details>

---

## 🔗 Navigation

| Previous | Up | Next |
|----------|----|----|
| [← SQL Joins](03-sql-joins.md) | [📚 Table of Contents](../README.md) | [Views →](05-views.md) |

---

*Last Updated: January 2026*
