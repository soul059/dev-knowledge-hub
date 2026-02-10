# Chapter 1.3: Database Languages

[← Back to Table of Contents](../README.md)

---

## 📚 Chapter Overview

Database languages are specialized languages used to interact with database management systems. Each language serves a specific purpose in the database ecosystem.

```
┌─────────────────────────────────────────────────────────────────┐
│                     CHAPTER LEARNING GOALS                       │
├─────────────────────────────────────────────────────────────────┤
│  • Understand different types of database languages             │
│  • Learn the purpose of DDL, DML, DCL, and TCL                 │
│  • Explore procedural vs non-procedural languages               │
│  • Understand host language and data sublanguage concepts       │
│  • Learn about 4GL and Query-by-Example                        │
└─────────────────────────────────────────────────────────────────┘
```

---

## 1. Types of Database Languages

```
┌─────────────────────────────────────────────────────────────────────┐
│                    DATABASE LANGUAGE TYPES                           │
└─────────────────────────────────────────────────────────────────────┘

                        DATABASE LANGUAGES
                              │
        ┌─────────────────────┼─────────────────────┐
        │                     │                     │
        ▼                     ▼                     ▼
   ┌─────────┐           ┌─────────┐           ┌─────────┐
   │   DDL   │           │   DML   │           │   DCL   │
   │  Data   │           │  Data   │           │  Data   │
   │Definition│          │Manipul- │           │ Control │
   │ Language │          │ ation   │           │Language │
   └─────────┘           │Language │           └─────────┘
                         └─────────┘
                              │
                    ┌─────────┴─────────┐
                    │                   │
                    ▼                   ▼
              ┌──────────┐        ┌──────────┐
              │Procedural│        │   Non-   │
              │          │        │Procedural│
              └──────────┘        └──────────┘
```

### Overview Table

| Language | Purpose | Examples |
|----------|---------|----------|
| **DDL** | Define database structure | CREATE, ALTER, DROP |
| **DML** | Manipulate data | SELECT, INSERT, UPDATE, DELETE |
| **DCL** | Control access | GRANT, REVOKE |
| **TCL** | Transaction control | COMMIT, ROLLBACK, SAVEPOINT |

---

## 2. Data Definition Language (DDL)

**DDL** is used to define and modify the database schema (structure).

```
┌─────────────────────────────────────────────────────────────────────┐
│                   DATA DEFINITION LANGUAGE (DDL)                     │
└─────────────────────────────────────────────────────────────────────┘

    PURPOSE: Define the structure of the database
    ─────────────────────────────────────────────

    ┌─────────────────────────────────────────────────────────────────┐
    │                      DDL COMMANDS                                │
    ├─────────────────────────────────────────────────────────────────┤
    │                                                                  │
    │   CREATE  ─────►  Create new database objects                   │
    │                   (tables, views, indexes, schemas)             │
    │                                                                  │
    │   ALTER   ─────►  Modify existing database objects              │
    │                   (add/modify/drop columns, constraints)        │
    │                                                                  │
    │   DROP    ─────►  Delete database objects                       │
    │                   (removes structure AND data)                  │
    │                                                                  │
    │   TRUNCATE ────►  Remove all records from table                 │
    │                   (keeps structure, resets identity)            │
    │                                                                  │
    │   RENAME  ─────►  Rename database objects                       │
    │                                                                  │
    └─────────────────────────────────────────────────────────────────┘


    DDL WORKFLOW:
    ─────────────

    ┌──────────────┐     ┌──────────────┐     ┌──────────────┐
    │  DDL Command │────►│ DDL Compiler │────►│    Data      │
    │  (CREATE...) │     │              │     │  Dictionary  │
    └──────────────┘     └──────────────┘     └──────────────┘
                                                     │
                                                     ▼
                                              ┌──────────────┐
                                              │   Metadata   │
                                              │   (Schema    │
                                              │  Information)│
                                              └──────────────┘
```

### DDL Examples

```sql
-- CREATE: Define a new table
CREATE TABLE Student (
    stud_id     INT PRIMARY KEY,
    name        VARCHAR(50) NOT NULL,
    age         INT CHECK (age >= 18),
    dept_id     INT REFERENCES Department(dept_id)
);

-- ALTER: Modify existing table
ALTER TABLE Student ADD email VARCHAR(100);
ALTER TABLE Student MODIFY name VARCHAR(100);
ALTER TABLE Student DROP COLUMN age;

-- DROP: Remove table completely
DROP TABLE Student;

-- TRUNCATE: Remove all data, keep structure
TRUNCATE TABLE Student;
```

### DDL Output: Data Dictionary

```
┌─────────────────────────────────────────────────────────────────────┐
│                        DATA DICTIONARY                               │
│                     (Populated by DDL)                               │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  Table: STUDENT                                                      │
│  ┌────────────┬────────────┬──────────┬─────────────┬────────────┐  │
│  │ Column     │ Data Type  │ Nullable │ Constraint  │ Default    │  │
│  ├────────────┼────────────┼──────────┼─────────────┼────────────┤  │
│  │ stud_id    │ INT        │ NO       │ PRIMARY KEY │ NULL       │  │
│  │ name       │ VARCHAR(50)│ NO       │ NOT NULL    │ NULL       │  │
│  │ age        │ INT        │ YES      │ CHECK       │ NULL       │  │
│  │ dept_id    │ INT        │ YES      │ FOREIGN KEY │ NULL       │  │
│  └────────────┴────────────┴──────────┴─────────────┴────────────┘  │
│                                                                      │
│  Indexes:                                                            │
│  • PK_Student (stud_id) - Clustered                                 │
│                                                                      │
│  Foreign Keys:                                                       │
│  • FK_Dept: dept_id REFERENCES Department(dept_id)                  │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 3. Data Manipulation Language (DML)

**DML** is used to retrieve, insert, update, and delete data in a database.

```
┌─────────────────────────────────────────────────────────────────────┐
│                 DATA MANIPULATION LANGUAGE (DML)                     │
└─────────────────────────────────────────────────────────────────────┘

    PURPOSE: Manipulate the data within database objects
    ─────────────────────────────────────────────────────

    ┌─────────────────────────────────────────────────────────────────┐
    │                      DML COMMANDS                                │
    ├─────────────────────────────────────────────────────────────────┤
    │                                                                  │
    │   SELECT  ─────►  Retrieve data from database                   │
    │                   (Query - Read operation)                       │
    │                                                                  │
    │   INSERT  ─────►  Add new records to table                      │
    │                   (Create operation)                             │
    │                                                                  │
    │   UPDATE  ─────►  Modify existing records                       │
    │                   (Update operation)                             │
    │                                                                  │
    │   DELETE  ─────►  Remove records from table                     │
    │                   (Delete operation)                             │
    │                                                                  │
    └─────────────────────────────────────────────────────────────────┘


    CRUD OPERATIONS MAPPING:
    ─────────────────────────

    ┌───────────────┐     ┌───────────────┐
    │    CRUD       │     │     SQL       │
    ├───────────────┤     ├───────────────┤
    │   Create      │ ──► │   INSERT      │
    │   Read        │ ──► │   SELECT      │
    │   Update      │ ──► │   UPDATE      │
    │   Delete      │ ──► │   DELETE      │
    └───────────────┘     └───────────────┘
```

### DML Examples

```sql
-- SELECT: Retrieve data
SELECT name, age FROM Student WHERE dept_id = 1;

-- INSERT: Add new record
INSERT INTO Student (stud_id, name, age, dept_id)
VALUES (101, 'Alice', 20, 1);

-- UPDATE: Modify existing record
UPDATE Student SET age = 21 WHERE stud_id = 101;

-- DELETE: Remove record
DELETE FROM Student WHERE stud_id = 101;
```

---

## 4. Procedural vs Non-Procedural DML

```
┌─────────────────────────────────────────────────────────────────────┐
│              PROCEDURAL vs NON-PROCEDURAL DML                        │
└─────────────────────────────────────────────────────────────────────┘


    PROCEDURAL DML                    NON-PROCEDURAL DML
    (How to get data)                 (What data to get)
    ─────────────────                 ────────────────────

    ┌───────────────────┐             ┌───────────────────┐
    │  Step 1: Open     │             │                   │
    │          file     │             │  "Get all         │
    │                   │             │   students        │
    │  Step 2: Read     │             │   with age > 20"  │
    │          record   │             │                   │
    │                   │             │                   │
    │  Step 3: Check    │             │   (System figures │
    │          condition│             │    out HOW)       │
    │                   │             │                   │
    │  Step 4: If match,│             └───────────────────┘
    │          output   │
    │                   │
    │  Step 5: Loop     │
    │                   │
    │  Step 6: Close    │
    └───────────────────┘


    USER SPECIFIES:
    ─────────────────────────────────────────────────────────────────
    
    PROCEDURAL:       HOW to retrieve data (step-by-step algorithm)
    NON-PROCEDURAL:   WHAT data is needed (declarative specification)
    
    ─────────────────────────────────────────────────────────────────
```

### Comparison

| Aspect | Procedural | Non-Procedural |
|--------|------------|----------------|
| **User Specifies** | How to get data | What data to get |
| **Complexity** | More complex | Simpler |
| **Navigation** | Explicit | Implicit |
| **Optimization** | User's responsibility | DBMS's responsibility |
| **Example** | PL/SQL loops, cursors | SQL SELECT |
| **Also Called** | Low-level DML | High-level/Declarative DML |

### Example Comparison

```
TASK: Find all students older than 20

PROCEDURAL APPROACH (Pseudocode):
─────────────────────────────────
    cursor = open(Student_table)
    result = []
    while (record = cursor.next()):
        if record.age > 20:
            result.append(record)
    close(cursor)
    return result

NON-PROCEDURAL APPROACH (SQL):
──────────────────────────────
    SELECT * FROM Student WHERE age > 20;
    
    (DBMS handles HOW to execute this)
```

### Real-World Analogy 🏠
- **Procedural**: Giving someone detailed driving directions (turn left, go 2 miles, turn right...)
- **Non-Procedural**: Telling someone the destination (go to the airport) and letting GPS figure out the route

---

## 5. Data Control Language (DCL)

**DCL** manages access rights and permissions in the database.

```
┌─────────────────────────────────────────────────────────────────────┐
│                   DATA CONTROL LANGUAGE (DCL)                        │
└─────────────────────────────────────────────────────────────────────┘

    PURPOSE: Control access to database and its objects
    ────────────────────────────────────────────────────

    ┌─────────────────────────────────────────────────────────────────┐
    │                      DCL COMMANDS                                │
    ├─────────────────────────────────────────────────────────────────┤
    │                                                                  │
    │   GRANT   ─────►  Give privileges to users/roles                │
    │                   (Allow access)                                 │
    │                                                                  │
    │   REVOKE  ─────►  Remove privileges from users/roles            │
    │                   (Deny access)                                  │
    │                                                                  │
    └─────────────────────────────────────────────────────────────────┘


    PRIVILEGE HIERARCHY:
    ─────────────────────

                    ┌──────────────────┐
                    │       DBA        │
                    │  (All Privileges)│
                    └────────┬─────────┘
                             │
             ┌───────────────┼───────────────┐
             │               │               │
             ▼               ▼               ▼
        ┌─────────┐     ┌─────────┐     ┌─────────┐
        │ Admin   │     │ Manager │     │Developer│
        │  Role   │     │  Role   │     │  Role   │
        └────┬────┘     └────┬────┘     └────┬────┘
             │               │               │
             ▼               ▼               ▼
        ALL tables      SELECT,         SELECT only
        ALL operations  INSERT,         (Read access)
                       UPDATE
```

### DCL Examples

```sql
-- GRANT: Give privileges
GRANT SELECT ON Student TO user1;
GRANT SELECT, INSERT, UPDATE ON Student TO manager_role;
GRANT ALL PRIVILEGES ON Student TO admin_user;

-- Grant with ability to pass privilege to others
GRANT SELECT ON Student TO user1 WITH GRANT OPTION;

-- REVOKE: Remove privileges
REVOKE INSERT ON Student FROM user1;
REVOKE ALL PRIVILEGES ON Student FROM temp_user;
```

### Common Privileges

| Privilege | Allows |
|-----------|--------|
| SELECT | Read data |
| INSERT | Add new records |
| UPDATE | Modify existing records |
| DELETE | Remove records |
| ALTER | Modify table structure |
| DROP | Delete tables |
| INDEX | Create indexes |
| ALL | All of the above |

---

## 6. Transaction Control Language (TCL)

**TCL** manages database transactions to ensure data integrity.

```
┌─────────────────────────────────────────────────────────────────────┐
│               TRANSACTION CONTROL LANGUAGE (TCL)                     │
└─────────────────────────────────────────────────────────────────────┘

    PURPOSE: Manage transactions in the database
    ────────────────────────────────────────────

    ┌─────────────────────────────────────────────────────────────────┐
    │                      TCL COMMANDS                                │
    ├─────────────────────────────────────────────────────────────────┤
    │                                                                  │
    │   COMMIT    ─────►  Save all changes permanently                │
    │                     (Make transaction changes persistent)        │
    │                                                                  │
    │   ROLLBACK  ─────►  Undo all changes since last commit          │
    │                     (Cancel transaction)                         │
    │                                                                  │
    │   SAVEPOINT ─────►  Create a point to rollback to               │
    │                     (Partial rollback support)                   │
    │                                                                  │
    │   SET       ─────►  Set transaction characteristics             │
    │   TRANSACTION       (Isolation level, access mode)              │
    │                                                                  │
    └─────────────────────────────────────────────────────────────────┘


    TRANSACTION LIFECYCLE:
    ───────────────────────

    BEGIN TRANSACTION
          │
          ▼
    ┌───────────────┐
    │   Operation 1 │──── INSERT
    └───────┬───────┘
            │
            ▼
    ┌───────────────┐
    │   SAVEPOINT A │──── (Checkpoint)
    └───────┬───────┘
            │
            ▼
    ┌───────────────┐
    │   Operation 2 │──── UPDATE
    └───────┬───────┘
            │
            ▼
    ┌───────────────┐
    │   SAVEPOINT B │──── (Checkpoint)
    └───────┬───────┘
            │
            ▼
    ┌───────────────┐
    │   Operation 3 │──── DELETE
    └───────┬───────┘
            │
     ┌──────┴──────┐
     │             │
     ▼             ▼
  COMMIT      ROLLBACK
  (Save)      (Undo to savepoint or beginning)
```

### TCL Examples

```sql
-- Start transaction (implicit in many DBMS)
BEGIN TRANSACTION;

-- Perform operations
INSERT INTO Account (id, balance) VALUES (1, 1000);
UPDATE Account SET balance = balance - 100 WHERE id = 1;

-- Create savepoint
SAVEPOINT after_transfer;

-- More operations
UPDATE Account SET balance = balance + 100 WHERE id = 2;

-- If something goes wrong, rollback to savepoint
ROLLBACK TO after_transfer;

-- Or rollback entire transaction
ROLLBACK;

-- If everything is OK, commit
COMMIT;
```

---

## 7. Query Language Categories

```
┌─────────────────────────────────────────────────────────────────────┐
│                    QUERY LANGUAGE SPECTRUM                           │
└─────────────────────────────────────────────────────────────────────┘

    Ease of Use ◄───────────────────────────────────────► Power/Control
         │                                                      │
         ▼                                                      ▼
    ┌─────────────────────────────────────────────────────────────────┐
    │                                                                  │
    │   FORM-BASED         QBE          SQL        PROCEDURAL SQL     │
    │   INTERFACE                                  (PL/SQL, T-SQL)    │
    │                                                                  │
    │   ┌─────────┐    ┌─────────┐   ┌─────────┐   ┌─────────────┐   │
    │   │ Fill in │    │ Example │   │SELECT * │   │ BEGIN       │   │
    │   │  forms  │    │  table  │   │FROM t   │   │   FOR i...  │   │
    │   │         │    │ with    │   │WHERE ..│   │   LOOP...   │   │
    │   │ [Name] │    │ sample  │   │         │   │ END;        │   │
    │   │ [Dept] │    │  values │   │         │   │             │   │
    │   └─────────┘    └─────────┘   └─────────┘   └─────────────┘   │
    │                                                                  │
    │   Non-programmers  Power users  Developers   DB Programmers      │
    │                                                                  │
    └─────────────────────────────────────────────────────────────────┘


    FORMAL VS COMMERCIAL:
    ──────────────────────

    FORMAL LANGUAGES              COMMERCIAL LANGUAGES
    (Theoretical)                 (Practical)
    ─────────────────             ────────────────────
    
    Relational Algebra    ◄───►   SQL (Structured Query Language)
    Relational Calculus   ◄───►   QBE (Query By Example)
    Domain Calculus              QUEL (from Ingres)
    Tuple Calculus               OQL (Object Query Language)
```

---

## 8. Query By Example (QBE)

**QBE** is a visual approach to querying databases using example patterns.

```
┌─────────────────────────────────────────────────────────────────────┐
│                      QUERY BY EXAMPLE (QBE)                          │
└─────────────────────────────────────────────────────────────────────┘

    CONCEPT: Show the DBMS what you want by example
    ─────────────────────────────────────────────────

    EXAMPLE QUERY: Find names of students in CS department

    ┌─────────────────────────────────────────────────────────────┐
    │  STUDENT TABLE                                               │
    ├───────────┬─────────────┬───────┬────────────┬──────────────┤
    │  Stud_ID  │    Name     │  Age  │   Major    │   Advisor    │
    ├───────────┼─────────────┼───────┼────────────┼──────────────┤
    │           │    P.       │       │    CS      │              │
    └───────────┴─────────────┴───────┴────────────┴──────────────┘
                      │                     │
                      │                     │
                      ▼                     ▼
                   P. means              Condition:
                   "Print this"          Major = 'CS'


    EQUIVALENT SQL:
    ────────────────
    SELECT Name FROM Student WHERE Major = 'CS';


    QBE OPERATORS:
    ───────────────
    
    P.     →  Print (display this column)
    P.ALL  →  Print including duplicates
    UNQ.   →  Unique (distinct values)
    _x     →  Variable (underscore + name)
    >      →  Greater than
    <      →  Less than
    >=, <= →  Comparison operators
    
    
    COMPLEX QBE EXAMPLE:
    ─────────────────────
    Find students who are older than 20 and in CS department

    ┌───────────┬─────────────┬───────┬────────────┐
    │  Stud_ID  │    Name     │  Age  │   Major    │
    ├───────────┼─────────────┼───────┼────────────┤
    │    P.     │    P.       │  >20  │    CS      │
    └───────────┴─────────────┴───────┴────────────┘
    
    SQL: SELECT Stud_ID, Name FROM Student 
         WHERE Age > 20 AND Major = 'CS';
```

---

## 9. Host Language and Data Sublanguage

```
┌─────────────────────────────────────────────────────────────────────┐
│              HOST LANGUAGE vs DATA SUBLANGUAGE                       │
└─────────────────────────────────────────────────────────────────────┘

    APPLICATION DEVELOPMENT APPROACHES:
    ────────────────────────────────────

    ┌─────────────────────────────────────────────────────────────────┐
    │                    EMBEDDED SQL APPROACH                         │
    │                                                                  │
    │    ┌─────────────────────────────────────────────────────────┐  │
    │    │  HOST LANGUAGE (Java, C, Python, etc.)                  │  │
    │    │                                                          │  │
    │    │    int main() {                                         │  │
    │    │        // Host language code                            │  │
    │    │        int student_id = 101;                            │  │
    │    │                                                          │  │
    │    │        ┌─────────────────────────────────────────────┐  │  │
    │    │        │  DATA SUBLANGUAGE (SQL)                     │  │  │
    │    │        │                                              │  │  │
    │    │        │  EXEC SQL                                    │  │  │
    │    │        │    SELECT name INTO :student_name            │  │  │
    │    │        │    FROM Student                              │  │  │
    │    │        │    WHERE stud_id = :student_id;             │  │  │
    │    │        │  END-EXEC                                    │  │  │
    │    │        └─────────────────────────────────────────────┘  │  │
    │    │                                                          │  │
    │    │        // More host language code                       │  │
    │    │        printf("Name: %s", student_name);                │  │
    │    │    }                                                     │  │
    │    └─────────────────────────────────────────────────────────┘  │
    │                                                                  │
    └─────────────────────────────────────────────────────────────────┘


    TERMINOLOGY:
    ─────────────

    HOST LANGUAGE       →  General-purpose programming language
                           (C, Java, Python, C#, etc.)
                           
    DATA SUBLANGUAGE    →  Database language embedded in host
                           (SQL, DML commands)
                           
    EMBEDDED SQL        →  SQL statements within host language
    
    PRECOMPILER         →  Translates embedded SQL to API calls
```

### Impedance Mismatch Problem

```
┌─────────────────────────────────────────────────────────────────────┐
│                    IMPEDANCE MISMATCH                                │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  Problem: Fundamental differences between programming languages      │
│           and relational databases                                   │
│                                                                      │
│  ┌──────────────────────────┐     ┌──────────────────────────┐      │
│  │   PROGRAMMING LANGUAGE   │     │      RELATIONAL DB       │      │
│  ├──────────────────────────┤     ├──────────────────────────┤      │
│  │ • Variables (single)     │ ≠   │ • Tables (sets of tuples)│      │
│  │ • Objects                │ ≠   │ • Relations              │      │
│  │ • Pointers/References    │ ≠   │ • Keys/Foreign Keys      │      │
│  │ • Iteration              │ ≠   │ • Set-at-a-time          │      │
│  │ • Procedural             │ ≠   │ • Declarative            │      │
│  └──────────────────────────┘     └──────────────────────────┘      │
│                                                                      │
│  SOLUTIONS:                                                          │
│  • Cursors (process one row at a time)                              │
│  • Object-Relational Mapping (ORM)                                  │
│  • Database APIs (JDBC, ODBC)                                       │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 10. Fourth Generation Languages (4GL)

```
┌─────────────────────────────────────────────────────────────────────┐
│               FOURTH GENERATION LANGUAGES (4GL)                      │
└─────────────────────────────────────────────────────────────────────┘

    LANGUAGE GENERATIONS:
    ──────────────────────

    ┌─────────────────────────────────────────────────────────────────┐
    │  1GL: Machine Language (Binary: 0s and 1s)                      │
    │        10110110 01001010 11010011                               │
    └─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
    ┌─────────────────────────────────────────────────────────────────┐
    │  2GL: Assembly Language (Mnemonics)                              │
    │        MOV AX, 5                                                 │
    │        ADD AX, BX                                                │
    └─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
    ┌─────────────────────────────────────────────────────────────────┐
    │  3GL: High-Level Languages (Procedural)                         │
    │        C, Java, Python, COBOL                                   │
    │        int sum = 0;                                             │
    │        for(int i=0; i<10; i++) sum += i;                        │
    └─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
    ┌─────────────────────────────────────────────────────────────────┐
    │  4GL: Very High-Level / Non-Procedural                          │
    │        SQL, Report generators, Form builders                    │
    │        SELECT SUM(amount) FROM sales WHERE year = 2025;        │
    │                                                                  │
    │        Characteristics:                                          │
    │        • Non-procedural (declarative)                           │
    │        • Domain-specific                                         │
    │        • Higher abstraction                                      │
    │        • Faster development                                      │
    │        • Less code needed                                        │
    └─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
    ┌─────────────────────────────────────────────────────────────────┐
    │  5GL: AI-based Languages                                         │
    │        Natural language processing                               │
    │        "Show me all customers who bought more than $1000"       │
    └─────────────────────────────────────────────────────────────────┘


    4GL CATEGORIES:
    ────────────────

    ┌────────────────────┐    ┌────────────────────┐
    │   Query Languages  │    │  Report Generators │
    │   • SQL            │    │  • Crystal Reports │
    │   • QBE            │    │  • JasperReports   │
    └────────────────────┘    └────────────────────┘

    ┌────────────────────┐    ┌────────────────────┐
    │   Form Generators  │    │ Application Gen.   │
    │   • Oracle Forms   │    │  • PowerBuilder    │
    │   • MS Access Forms│    │  • OutSystems      │
    └────────────────────┘    └────────────────────┘
```

---

## 11. Database APIs

```
┌─────────────────────────────────────────────────────────────────────┐
│                       DATABASE APIS                                  │
└─────────────────────────────────────────────────────────────────────┘

    PURPOSE: Standard interface to connect applications to databases
    ────────────────────────────────────────────────────────────────


    ┌─────────────────────────────────────────────────────────────────┐
    │                    COMMON DATABASE APIs                          │
    ├─────────────────────────────────────────────────────────────────┤
    │                                                                  │
    │   ODBC (Open Database Connectivity)                             │
    │   ─────────────────────────────────                             │
    │   • Microsoft standard                                          │
    │   • Language-independent                                        │
    │   • Works with multiple databases                               │
    │                                                                  │
    │   JDBC (Java Database Connectivity)                             │
    │   ────────────────────────────────                              │
    │   • Java standard API                                           │
    │   • Platform-independent                                        │
    │   • Works with any JDBC-compliant database                      │
    │                                                                  │
    │   ADO.NET (ActiveX Data Objects for .NET)                       │
    │   ───────────────────────────────────────                       │
    │   • Microsoft .NET framework                                    │
    │   • Disconnected architecture                                   │
    │   • DataSet, DataAdapter concepts                               │
    │                                                                  │
    │   ORM (Object-Relational Mapping)                               │
    │   ───────────────────────────────                               │
    │   • Hibernate (Java)                                            │
    │   • Entity Framework (.NET)                                     │
    │   • SQLAlchemy (Python)                                         │
    │   • Maps objects to database tables                             │
    │                                                                  │
    └─────────────────────────────────────────────────────────────────┘


    API ARCHITECTURE:
    ─────────────────

    ┌───────────────┐
    │  Application  │
    │  (Java/C#/etc)│
    └───────┬───────┘
            │
            ▼
    ┌───────────────┐
    │   API Layer   │  (JDBC/ODBC/ADO.NET)
    │               │
    └───────┬───────┘
            │
            ▼
    ┌───────────────┐
    │ Driver/Adapter│  (Database-specific)
    │               │
    └───────┬───────┘
            │
            ▼
    ┌───────────────┐
    │    DBMS       │  (MySQL, Oracle, etc.)
    │               │
    └───────────────┘
```

---

## 📊 Summary Table

| Language | Purpose | Commands | Auto-Commit |
|----------|---------|----------|-------------|
| **DDL** | Define schema | CREATE, ALTER, DROP, TRUNCATE | Yes (usually) |
| **DML** | Manipulate data | SELECT, INSERT, UPDATE, DELETE | No |
| **DCL** | Control access | GRANT, REVOKE | Yes |
| **TCL** | Manage transactions | COMMIT, ROLLBACK, SAVEPOINT | N/A |

| Concept | Description |
|---------|-------------|
| **Procedural DML** | User specifies HOW to get data (step-by-step) |
| **Non-Procedural DML** | User specifies WHAT data is needed (declarative) |
| **Host Language** | General-purpose programming language (C, Java) |
| **Data Sublanguage** | Database language embedded in host language |
| **QBE** | Visual query-by-example approach |
| **4GL** | High-level non-procedural languages |
| **Impedance Mismatch** | Difference between programming and database paradigms |

---

## ❓ Quick Revision Questions

1. **What is the difference between DDL and DML?**
   <details>
   <summary>Click for Answer</summary>
   DDL (Data Definition Language) defines the structure/schema of the database (CREATE, ALTER, DROP). DML (Data Manipulation Language) manipulates the data within the database (SELECT, INSERT, UPDATE, DELETE).
   </details>

2. **Explain procedural vs non-procedural DML with an example.**
   <details>
   <summary>Click for Answer</summary>
   Procedural DML requires specifying HOW to retrieve data (using loops, cursors). Non-procedural DML (like SQL) only specifies WHAT data is needed. Example: "SELECT * FROM Students WHERE age > 20" is non-procedural - you don't specify how to search.
   </details>

3. **What is the purpose of TCL commands?**
   <details>
   <summary>Click for Answer</summary>
   TCL (Transaction Control Language) manages database transactions to ensure data integrity. COMMIT saves changes permanently, ROLLBACK undoes changes, and SAVEPOINT creates intermediate restore points within a transaction.
   </details>

4. **What is impedance mismatch in database systems?**
   <details>
   <summary>Click for Answer</summary>
   Impedance mismatch refers to the fundamental differences between programming languages (which work with individual variables) and relational databases (which work with sets of tuples). Solutions include cursors, ORMs, and database APIs.
   </details>

5. **Differentiate between TRUNCATE and DELETE.**
   <details>
   <summary>Click for Answer</summary>
   DELETE is DML - removes specific rows, can have WHERE clause, can be rolled back, fires triggers. TRUNCATE is DDL - removes ALL rows, cannot have WHERE clause, faster, resets auto-increment, typically cannot be rolled back.
   </details>

6. **What is Query By Example (QBE)?**
   <details>
   <summary>Click for Answer</summary>
   QBE is a visual, non-procedural query language where users specify queries by entering example values in a table template. Users mark columns to display with "P." and enter conditions directly in cells instead of writing SQL syntax.
   </details>

---

## 🔗 Navigation

| Previous | Up | Next |
|----------|----|----|
| [← Data Models](02-data-models.md) | [📚 Table of Contents](../README.md) | [Database Users and Administrators →](04-database-users-administrators.md) |

---

*Last Updated: January 2026*
