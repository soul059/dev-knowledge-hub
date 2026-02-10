# Chapter 3.1: DDL - Data Definition Language

[← Back to Table of Contents](../README.md)

---

## 📚 Chapter Overview

**Data Definition Language (DDL)** is a subset of SQL used to define and modify database structures. DDL commands create, alter, and drop database objects like tables, indexes, views, and schemas.

```
┌─────────────────────────────────────────────────────────────────┐
│                     CHAPTER LEARNING GOALS                       │
├─────────────────────────────────────────────────────────────────┤
│  • Understand DDL and its role in database management           │
│  • Master CREATE, ALTER, DROP, TRUNCATE commands                │
│  • Learn about data types and constraints                       │
│  • Practice defining schemas and tables                         │
│  • Understand the impact of DDL operations                      │
└─────────────────────────────────────────────────────────────────┘
```

---

## 1. What is DDL?

```
┌─────────────────────────────────────────────────────────────────────┐
│                    DDL - DATA DEFINITION LANGUAGE                    │
└─────────────────────────────────────────────────────────────────────┘

    DDL deals with the STRUCTURE (schema) of the database, not the data.
    
    
    DDL COMMANDS:
    ──────────────
    
    ┌───────────┬────────────────────────────────────────────────────┐
    │  Command  │                    Purpose                         │
    ├───────────┼────────────────────────────────────────────────────┤
    │  CREATE   │  Creates new database objects (tables, views, etc)│
    │  ALTER    │  Modifies existing database objects               │
    │  DROP     │  Removes database objects permanently             │
    │  TRUNCATE │  Removes all data from a table (keeps structure)  │
    │  RENAME   │  Renames a database object                        │
    └───────────┴────────────────────────────────────────────────────┘
    
    
    DDL vs DML:
    ────────────
    
    ┌──────────────────────────────────────────────────────────────────┐
    │                                                                   │
    │   DDL                         DML                                │
    │   ───                         ───                                │
    │   Defines structure           Manipulates data                   │
    │   CREATE TABLE                INSERT INTO                        │
    │   ALTER TABLE                 UPDATE                             │
    │   DROP TABLE                  DELETE FROM                        │
    │   Auto-commit                 Can be rolled back                 │
    │                                                                   │
    └──────────────────────────────────────────────────────────────────┘
    
    
    REAL-WORLD ANALOGY:
    ────────────────────
    
    DDL = Architect's Blueprint (defines building structure)
    DML = Moving furniture/people (changes contents, not structure)
```

---

## 2. CREATE Statement

### CREATE DATABASE

```
┌─────────────────────────────────────────────────────────────────────┐
│                       CREATE DATABASE                                │
└─────────────────────────────────────────────────────────────────────┘

    SYNTAX:
    ────────
    
    CREATE DATABASE database_name;
    
    
    EXAMPLE:
    ─────────
    
    CREATE DATABASE UniversityDB;
    
    
    VISUALIZATION:
    ───────────────
    
    Before:                          After:
    ┌─────────────────────┐          ┌─────────────────────┐
    │    Database Server  │          │    Database Server  │
    │  ┌───────────────┐  │          │  ┌───────────────┐  │
    │  │ (empty)       │  │   →→→    │  │ UniversityDB  │  │
    │  └───────────────┘  │          │  │               │  │
    │                     │          │  │  (empty db)   │  │
    │                     │          │  └───────────────┘  │
    └─────────────────────┘          └─────────────────────┘
    
    
    USE DATABASE:
    ──────────────
    
    USE UniversityDB;   -- Switch to this database
```

### CREATE TABLE

```
┌─────────────────────────────────────────────────────────────────────┐
│                         CREATE TABLE                                 │
└─────────────────────────────────────────────────────────────────────┘

    SYNTAX:
    ────────
    
    CREATE TABLE table_name (
        column1 datatype [constraints],
        column2 datatype [constraints],
        ...
        [table_constraints]
    );
    
    
    EXAMPLE - SIMPLE:
    ──────────────────
    
    CREATE TABLE Student (
        Stud_ID     CHAR(5),
        Name        VARCHAR(50),
        Age         INT,
        Dept_ID     CHAR(3)
    );
    
    
    EXAMPLE - WITH CONSTRAINTS:
    ────────────────────────────
    
    CREATE TABLE Student (
        Stud_ID     CHAR(5)      PRIMARY KEY,
        Name        VARCHAR(50)  NOT NULL,
        Age         INT          CHECK (Age >= 16 AND Age <= 100),
        Dept_ID     CHAR(3)      REFERENCES Department(Dept_ID),
        Email       VARCHAR(100) UNIQUE
    );
    
    
    TABLE CREATION VISUALIZED:
    ───────────────────────────
    
    CREATE TABLE Student (...);
    
           ↓ Creates:
    
    Student
    ┌─────────┬──────────┬─────┬─────────┬───────────────┐
    │ Stud_ID │   Name   │ Age │ Dept_ID │     Email     │
    │ PK      │ NOT NULL │ CK  │ FK      │    UNIQUE     │
    ├─────────┼──────────┼─────┼─────────┼───────────────┤
    │         │          │     │         │               │
    │   (empty - no data yet)                            │
    │         │          │     │         │               │
    └─────────┴──────────┴─────┴─────────┴───────────────┘
```

### CREATE INDEX

```
┌─────────────────────────────────────────────────────────────────────┐
│                         CREATE INDEX                                 │
└─────────────────────────────────────────────────────────────────────┘

    SYNTAX:
    ────────
    
    CREATE INDEX index_name ON table_name (column1, column2, ...);
    
    CREATE UNIQUE INDEX index_name ON table_name (column);
    
    
    EXAMPLES:
    ──────────
    
    -- Simple index
    CREATE INDEX idx_student_name ON Student(Name);
    
    -- Unique index
    CREATE UNIQUE INDEX idx_student_email ON Student(Email);
    
    -- Composite index
    CREATE INDEX idx_enrollment ON Enrollment(Stud_ID, Course_ID);
    
    
    INDEX VISUALIZATION:
    ─────────────────────
    
    Student Table                 idx_student_name
    ┌─────────┬─────────┐         ┌─────────┬──────────┐
    │ Stud_ID │  Name   │         │  Name   │  RowPtr  │
    ├─────────┼─────────┤         ├─────────┼──────────┤
    │  S101   │  Alice  │    ←────│  Alice  │  → S101  │
    │  S102   │   Bob   │    ←────│   Bob   │  → S102  │
    │  S103   │ Charlie │    ←────│ Charlie │  → S103  │
    └─────────┴─────────┘         └─────────┴──────────┘
                                  (sorted by Name)
    
    Index speeds up: SELECT * FROM Student WHERE Name = 'Alice';
```

### CREATE VIEW

```
┌─────────────────────────────────────────────────────────────────────┐
│                          CREATE VIEW                                 │
└─────────────────────────────────────────────────────────────────────┘

    SYNTAX:
    ────────
    
    CREATE VIEW view_name AS
    SELECT column1, column2, ...
    FROM table_name
    WHERE condition;
    
    
    EXAMPLE:
    ─────────
    
    CREATE VIEW CS_Students AS
    SELECT Stud_ID, Name, Age
    FROM Student
    WHERE Dept_ID = 'D01';
    
    
    VIEW VS TABLE:
    ────────────────
    
    ┌────────────────────────────────────────────────────────────────┐
    │                                                                 │
    │   Table = Physical storage of data                             │
    │   View  = Virtual table (stored query, not stored data)        │
    │                                                                 │
    └────────────────────────────────────────────────────────────────┘
    
    
    VISUALIZATION:
    ───────────────
    
    Student (Base Table)           CS_Students (View)
    ┌─────────┬─────────┬─────────┐    ┌─────────┬─────────┬─────┐
    │ Stud_ID │  Name   │ Dept_ID │    │ Stud_ID │  Name   │ Age │
    ├─────────┼─────────┼─────────┤    ├─────────┼─────────┼─────┤
    │  S101   │  Alice  │   D01   │───→│  S101   │  Alice  │ 20  │
    │  S102   │   Bob   │   D02   │    │  S103   │ Charlie │ 21  │
    │  S103   │ Charlie │   D01   │───→└─────────┴─────────┴─────┘
    │  S104   │  Diana  │   D03   │    (Only D01 students shown)
    └─────────┴─────────┴─────────┘
```

---

## 3. SQL Data Types

```
┌─────────────────────────────────────────────────────────────────────┐
│                       COMMON SQL DATA TYPES                          │
└─────────────────────────────────────────────────────────────────────┘


    NUMERIC TYPES:
    ───────────────
    
    ┌──────────────┬─────────────────────────────────────────────────┐
    │    Type      │              Description                        │
    ├──────────────┼─────────────────────────────────────────────────┤
    │ INT/INTEGER  │ Whole numbers (-2 billion to +2 billion)       │
    │ SMALLINT     │ Smaller integers (-32,768 to 32,767)           │
    │ BIGINT       │ Large integers                                  │
    │ DECIMAL(p,s) │ Exact decimal (p=precision, s=scale)           │
    │ NUMERIC(p,s) │ Same as DECIMAL                                 │
    │ FLOAT        │ Floating-point (approximate)                    │
    │ REAL         │ Single-precision floating-point                 │
    └──────────────┴─────────────────────────────────────────────────┘
    
    Example: DECIMAL(10,2) → 12345678.90 (10 digits, 2 after decimal)
    

    CHARACTER TYPES:
    ─────────────────
    
    ┌──────────────┬─────────────────────────────────────────────────┐
    │    Type      │              Description                        │
    ├──────────────┼─────────────────────────────────────────────────┤
    │ CHAR(n)      │ Fixed-length string (padded with spaces)       │
    │ VARCHAR(n)   │ Variable-length string (up to n characters)    │
    │ TEXT         │ Variable-length text (very large)              │
    └──────────────┴─────────────────────────────────────────────────┘
    
    CHAR vs VARCHAR:
    ┌─────────────────────────────────────────────────────────────────┐
    │ CHAR(10) 'Hello' → 'Hello     ' (5 chars + 5 spaces = 10)     │
    │ VARCHAR(10) 'Hello' → 'Hello' (only 5 characters stored)       │
    └─────────────────────────────────────────────────────────────────┘
    

    DATE/TIME TYPES:
    ─────────────────
    
    ┌──────────────┬─────────────────────────────────────────────────┐
    │    Type      │              Description                        │
    ├──────────────┼─────────────────────────────────────────────────┤
    │ DATE         │ Date (YYYY-MM-DD)                               │
    │ TIME         │ Time (HH:MM:SS)                                 │
    │ DATETIME     │ Date and time combined                          │
    │ TIMESTAMP    │ Date and time with timezone                     │
    │ YEAR         │ Year value                                      │
    └──────────────┴─────────────────────────────────────────────────┘


    BOOLEAN AND OTHER:
    ───────────────────
    
    ┌──────────────┬─────────────────────────────────────────────────┐
    │    Type      │              Description                        │
    ├──────────────┼─────────────────────────────────────────────────┤
    │ BOOLEAN      │ TRUE or FALSE                                   │
    │ BLOB         │ Binary Large Object (images, files)            │
    │ CLOB         │ Character Large Object                          │
    │ ENUM         │ Enumerated list of values                       │
    └──────────────┴─────────────────────────────────────────────────┘
```

---

## 4. Constraints

```
┌─────────────────────────────────────────────────────────────────────┐
│                       SQL CONSTRAINTS                                │
└─────────────────────────────────────────────────────────────────────┘

    Constraints enforce rules on data to maintain integrity.
    

    PRIMARY KEY:
    ─────────────
    
    • Uniquely identifies each row
    • Cannot be NULL
    • Only one primary key per table
    
    CREATE TABLE Student (
        Stud_ID CHAR(5) PRIMARY KEY,
        Name VARCHAR(50)
    );
    
    -- Or table-level:
    CREATE TABLE Enrollment (
        Stud_ID CHAR(5),
        Course_ID CHAR(6),
        PRIMARY KEY (Stud_ID, Course_ID)  -- Composite key
    );
    

    FOREIGN KEY:
    ─────────────
    
    • References primary key of another table
    • Maintains referential integrity
    
    CREATE TABLE Student (
        Stud_ID CHAR(5) PRIMARY KEY,
        Dept_ID CHAR(3) REFERENCES Department(Dept_ID)
    );
    
    -- Or table-level with options:
    CREATE TABLE Student (
        Stud_ID CHAR(5) PRIMARY KEY,
        Dept_ID CHAR(3),
        FOREIGN KEY (Dept_ID) REFERENCES Department(Dept_ID)
            ON DELETE CASCADE
            ON UPDATE SET NULL
    );
    
    
    FOREIGN KEY ACTIONS:
    ┌─────────────────┬────────────────────────────────────────────┐
    │     Action      │            Description                     │
    ├─────────────────┼────────────────────────────────────────────┤
    │ CASCADE         │ Delete/update child rows automatically     │
    │ SET NULL        │ Set foreign key to NULL                    │
    │ SET DEFAULT     │ Set foreign key to default value           │
    │ RESTRICT        │ Prevent delete/update if children exist    │
    │ NO ACTION       │ Same as RESTRICT (default)                 │
    └─────────────────┴────────────────────────────────────────────┘


    NOT NULL:
    ──────────
    
    • Column cannot contain NULL values
    
    CREATE TABLE Student (
        Stud_ID CHAR(5) NOT NULL,
        Name VARCHAR(50) NOT NULL
    );


    UNIQUE:
    ────────
    
    • All values in column must be different
    • Can have multiple UNIQUE columns
    • NULL is allowed (unless combined with NOT NULL)
    
    CREATE TABLE Student (
        Stud_ID CHAR(5) PRIMARY KEY,
        Email VARCHAR(100) UNIQUE,
        Phone VARCHAR(15) UNIQUE
    );


    CHECK:
    ───────
    
    • Validates data based on condition
    
    CREATE TABLE Student (
        Stud_ID CHAR(5) PRIMARY KEY,
        Age INT CHECK (Age >= 16 AND Age <= 100),
        Gender CHAR(1) CHECK (Gender IN ('M', 'F', 'O'))
    );


    DEFAULT:
    ─────────
    
    • Provides default value if none specified
    
    CREATE TABLE Student (
        Stud_ID CHAR(5) PRIMARY KEY,
        Status VARCHAR(20) DEFAULT 'Active',
        Created_Date DATE DEFAULT CURRENT_DATE
    );
```

### Constraints Visualization

```
┌─────────────────────────────────────────────────────────────────────┐
│                    CONSTRAINT VISUALIZATION                          │
└─────────────────────────────────────────────────────────────────────┘

    Department                          Student
    ┌─────────┬──────────────┐          ┌─────────┬───────┬─────┬─────────┐
    │ Dept_ID │  Dept_Name   │          │ Stud_ID │ Name  │ Age │ Dept_ID │
    │   PK    │  NOT NULL    │          │   PK    │NN, UQ │ CK  │   FK    │
    ├─────────┼──────────────┤          ├─────────┼───────┼─────┼─────────┤
    │   D01   │     CS       │←─────────│  S101   │ Alice │ 20  │   D01   │
    │   D02   │    Math      │←─────────│  S102   │  Bob  │ 22  │   D02   │
    │   D03   │   Physics    │          │  S103   │Charlie│ 21  │   D01   │
    └─────────┴──────────────┘          └─────────┴───────┴─────┴─────────┘
                    ↑                                              │
                    └──────────────── REFERENCES ──────────────────┘
    
    
    Legend:
    PK = PRIMARY KEY    (Unique + NOT NULL)
    FK = FOREIGN KEY    (References another table)
    NN = NOT NULL       (Cannot be NULL)
    UQ = UNIQUE         (All values different)
    CK = CHECK          (Validates condition)
```

---

## 5. ALTER Statement

```
┌─────────────────────────────────────────────────────────────────────┐
│                        ALTER TABLE                                   │
└─────────────────────────────────────────────────────────────────────┘


    ADD COLUMN:
    ────────────
    
    ALTER TABLE Student ADD Phone VARCHAR(15);
    
    Before:                          After:
    ┌─────────┬───────┬─────┐        ┌─────────┬───────┬─────┬─────────┐
    │ Stud_ID │ Name  │ Age │   →    │ Stud_ID │ Name  │ Age │  Phone  │
    ├─────────┼───────┼─────┤        ├─────────┼───────┼─────┼─────────┤
    │  S101   │ Alice │ 20  │        │  S101   │ Alice │ 20  │  NULL   │
    └─────────┴───────┴─────┘        └─────────┴───────┴─────┴─────────┘


    DROP COLUMN:
    ─────────────
    
    ALTER TABLE Student DROP COLUMN Phone;


    MODIFY/ALTER COLUMN:
    ─────────────────────
    
    -- Change data type
    ALTER TABLE Student MODIFY Name VARCHAR(100);
    
    -- Add constraint
    ALTER TABLE Student ALTER COLUMN Age SET NOT NULL;
    
    -- Change default
    ALTER TABLE Student ALTER COLUMN Status SET DEFAULT 'Inactive';


    ADD CONSTRAINT:
    ─────────────────
    
    ALTER TABLE Student 
    ADD CONSTRAINT chk_age CHECK (Age >= 16);
    
    ALTER TABLE Student
    ADD CONSTRAINT fk_dept 
    FOREIGN KEY (Dept_ID) REFERENCES Department(Dept_ID);


    DROP CONSTRAINT:
    ─────────────────
    
    ALTER TABLE Student DROP CONSTRAINT chk_age;
    
    ALTER TABLE Student DROP PRIMARY KEY;  -- Drops PK constraint


    RENAME:
    ────────
    
    -- Rename table
    ALTER TABLE Student RENAME TO Students;
    
    -- Rename column
    ALTER TABLE Student RENAME COLUMN Name TO Full_Name;
```

---

## 6. DROP and TRUNCATE

```
┌─────────────────────────────────────────────────────────────────────┐
│                      DROP vs TRUNCATE                                │
└─────────────────────────────────────────────────────────────────────┘


    DROP TABLE:
    ────────────
    
    DROP TABLE Student;
    
    • Removes table structure AND data
    • Table no longer exists
    • Cannot be rolled back (usually)
    
    Before:                          After:
    ┌─────────┬───────┬─────┐
    │ Stud_ID │ Name  │ Age │        (Table doesn't exist)
    ├─────────┼───────┼─────┤   →
    │  S101   │ Alice │ 20  │        ERROR: Table 'Student' not found
    │  S102   │  Bob  │ 22  │
    └─────────┴───────┴─────┘


    DROP WITH CASCADE:
    ───────────────────
    
    DROP TABLE Department CASCADE;
    
    • Also drops dependent objects (views, foreign key references)


    TRUNCATE TABLE:
    ─────────────────
    
    TRUNCATE TABLE Student;
    
    • Removes ALL data from table
    • Keeps table structure (schema)
    • Faster than DELETE (doesn't log individual rows)
    • Resets auto-increment counters
    
    Before:                          After:
    ┌─────────┬───────┬─────┐        ┌─────────┬───────┬─────┐
    │ Stud_ID │ Name  │ Age │        │ Stud_ID │ Name  │ Age │
    ├─────────┼───────┼─────┤   →    ├─────────┼───────┼─────┤
    │  S101   │ Alice │ 20  │        │         │       │     │
    │  S102   │  Bob  │ 22  │        │   (empty - no rows)   │
    └─────────┴───────┴─────┘        └─────────┴───────┴─────┘


    COMPARISON:
    ────────────
    
    ┌────────────────┬───────────────────┬──────────────────────┐
    │    Feature     │       DROP        │      TRUNCATE        │
    ├────────────────┼───────────────────┼──────────────────────┤
    │ Removes data   │        Yes        │         Yes          │
    │ Removes schema │        Yes        │         No           │
    │ Can use WHERE  │        N/A        │         No           │
    │ Fires triggers │        No         │         No           │
    │ Resets counter │       Yes         │         Yes          │
    │ Speed          │       Fast        │       Faster         │
    │ Rollback       │ Usually No        │    Usually No        │
    └────────────────┴───────────────────┴──────────────────────┘
```

---

## 7. Complete DDL Example

```
┌─────────────────────────────────────────────────────────────────────┐
│               COMPLETE UNIVERSITY DATABASE DDL                       │
└─────────────────────────────────────────────────────────────────────┘


-- Create database
CREATE DATABASE UniversityDB;
USE UniversityDB;

-- Create Department table (parent)
CREATE TABLE Department (
    Dept_ID     CHAR(3)       PRIMARY KEY,
    Dept_Name   VARCHAR(50)   NOT NULL UNIQUE,
    Location    VARCHAR(100)
);

-- Create Student table (references Department)
CREATE TABLE Student (
    Stud_ID     CHAR(5)       PRIMARY KEY,
    Name        VARCHAR(50)   NOT NULL,
    Email       VARCHAR(100)  UNIQUE,
    Age         INT           CHECK (Age >= 16 AND Age <= 100),
    Dept_ID     CHAR(3),
    Enroll_Date DATE          DEFAULT CURRENT_DATE,
    
    FOREIGN KEY (Dept_ID) 
        REFERENCES Department(Dept_ID)
        ON DELETE SET NULL
        ON UPDATE CASCADE
);

-- Create Course table
CREATE TABLE Course (
    Course_ID   CHAR(6)       PRIMARY KEY,
    Title       VARCHAR(100)  NOT NULL,
    Credits     INT           CHECK (Credits >= 1 AND Credits <= 6),
    Dept_ID     CHAR(3)       REFERENCES Department(Dept_ID)
);

-- Create Enrollment table (junction table)
CREATE TABLE Enrollment (
    Stud_ID     CHAR(5),
    Course_ID   CHAR(6),
    Semester    VARCHAR(10)   NOT NULL,
    Grade       CHAR(2)       CHECK (Grade IN ('A','B','C','D','F','I')),
    
    PRIMARY KEY (Stud_ID, Course_ID, Semester),
    FOREIGN KEY (Stud_ID) REFERENCES Student(Stud_ID) ON DELETE CASCADE,
    FOREIGN KEY (Course_ID) REFERENCES Course(Course_ID)
);

-- Create indexes for performance
CREATE INDEX idx_student_name ON Student(Name);
CREATE INDEX idx_student_dept ON Student(Dept_ID);
CREATE INDEX idx_course_title ON Course(Title);

-- Create view
CREATE VIEW CS_Students AS
SELECT s.Stud_ID, s.Name, s.Age
FROM Student s
JOIN Department d ON s.Dept_ID = d.Dept_ID
WHERE d.Dept_Name = 'Computer Science';


    RESULTING SCHEMA:
    ──────────────────
    
    ┌─────────────────┐     ┌─────────────────┐
    │   Department    │     │     Course      │
    │─────────────────│     │─────────────────│
    │ Dept_ID (PK)    │←────│ Dept_ID (FK)    │
    │ Dept_Name       │     │ Course_ID (PK)  │
    │ Location        │     │ Title           │
    └────────┬────────┘     │ Credits         │
             │              └────────┬────────┘
             │                       │
             ↓                       ↓
    ┌─────────────────┐     ┌─────────────────┐
    │    Student      │     │   Enrollment    │
    │─────────────────│     │─────────────────│
    │ Stud_ID (PK)    │←────│ Stud_ID (FK,PK) │
    │ Name            │     │ Course_ID(FK,PK)│────→ Course
    │ Email           │     │ Semester (PK)   │
    │ Age             │     │ Grade           │
    │ Dept_ID (FK)    │     └─────────────────┘
    │ Enroll_Date     │
    └─────────────────┘
```

---

## 📊 Summary Table

| Command | Purpose | Syntax |
|---------|---------|--------|
| **CREATE DATABASE** | Create new database | `CREATE DATABASE db_name;` |
| **CREATE TABLE** | Create new table | `CREATE TABLE t (col1 type, ...);` |
| **CREATE INDEX** | Create index for faster search | `CREATE INDEX idx ON t(col);` |
| **CREATE VIEW** | Create virtual table | `CREATE VIEW v AS SELECT...;` |
| **ALTER TABLE ADD** | Add column/constraint | `ALTER TABLE t ADD col type;` |
| **ALTER TABLE DROP** | Remove column/constraint | `ALTER TABLE t DROP col;` |
| **ALTER TABLE MODIFY** | Change column definition | `ALTER TABLE t MODIFY col type;` |
| **DROP TABLE** | Remove table completely | `DROP TABLE t;` |
| **TRUNCATE TABLE** | Remove all data | `TRUNCATE TABLE t;` |

| Constraint | Purpose | Can be NULL? |
|------------|---------|--------------|
| **PRIMARY KEY** | Unique identifier | No |
| **FOREIGN KEY** | Reference to another table | Yes (unless NOT NULL) |
| **UNIQUE** | No duplicate values | Yes (one NULL) |
| **NOT NULL** | Disallow NULL | No |
| **CHECK** | Validate condition | Yes |
| **DEFAULT** | Provide default value | Yes |

---

## ❓ Quick Revision Questions

1. **What is the difference between DDL and DML?**
   <details>
   <summary>Click for Answer</summary>
   DDL (Data Definition Language) defines database structure - CREATE, ALTER, DROP. DML (Data Manipulation Language) manipulates data - INSERT, UPDATE, DELETE. DDL changes schema; DML changes data within schema.
   </details>

2. **What is the difference between DROP and TRUNCATE?**
   <details>
   <summary>Click for Answer</summary>
   DROP removes the entire table (structure + data). TRUNCATE removes all data but keeps the table structure intact. TRUNCATE is faster as it doesn't log individual row deletions.
   </details>

3. **What is the purpose of FOREIGN KEY with ON DELETE CASCADE?**
   <details>
   <summary>Click for Answer</summary>
   ON DELETE CASCADE automatically deletes child records when the parent record is deleted. For example, if a Department is deleted, all Students in that department would also be deleted automatically.
   </details>

4. **What is the difference between CHAR(10) and VARCHAR(10)?**
   <details>
   <summary>Click for Answer</summary>
   CHAR(10) is fixed-length - always stores exactly 10 characters (pads with spaces). VARCHAR(10) is variable-length - stores up to 10 characters, using only the space needed. Use CHAR for fixed-length data (like country codes), VARCHAR for variable data (like names).
   </details>

5. **Can a table have multiple PRIMARY KEYs? Multiple UNIQUE constraints?**
   <details>
   <summary>Click for Answer</summary>
   A table can have only ONE PRIMARY KEY (but it can be composite - multiple columns). A table CAN have multiple UNIQUE constraints on different columns. PRIMARY KEY = UNIQUE + NOT NULL.
   </details>

6. **How do you add a new column with a NOT NULL constraint to an existing table that has data?**
   <details>
   <summary>Click for Answer</summary>
   You must either: (1) Add the column with a DEFAULT value: ALTER TABLE t ADD col INT NOT NULL DEFAULT 0; or (2) First add as nullable, update all rows, then add NOT NULL constraint.
   </details>

---

## 🔗 Navigation

| Previous | Up | Next |
|----------|----|----|
| [← Domain Relational Calculus](../02-Relational-Model/04-domain-relational-calculus.md) | [📚 Table of Contents](../README.md) | [DML - Data Manipulation Language →](02-dml-data-manipulation-language.md) |

---

*Last Updated: January 2026*
