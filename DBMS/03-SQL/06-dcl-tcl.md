# Chapter 3.6: DCL and TCL

[← Back to Table of Contents](../README.md)

---

## 📚 Chapter Overview

**DCL (Data Control Language)** manages permissions and access rights in the database. **TCL (Transaction Control Language)** manages database transactions. Both are crucial for database security and data integrity.

```
┌─────────────────────────────────────────────────────────────────┐
│                     CHAPTER LEARNING GOALS                       │
├─────────────────────────────────────────────────────────────────┤
│  • Understand database access control with DCL                  │
│  • Master GRANT and REVOKE commands                             │
│  • Learn about roles and privileges                             │
│  • Understand transactions and ACID properties                  │
│  • Use COMMIT, ROLLBACK, and SAVEPOINT                          │
└─────────────────────────────────────────────────────────────────┘
```

---

## 1. SQL Language Categories Overview

```
┌─────────────────────────────────────────────────────────────────────┐
│                  SQL LANGUAGE CATEGORIES                             │
└─────────────────────────────────────────────────────────────────────┘

    ┌─────────────────────────────────────────────────────────────────┐
    │                                                                  │
    │   DDL                DML                DCL                TCL  │
    │   ───                ───                ───                ───  │
    │   CREATE             SELECT             GRANT              COMMIT│
    │   ALTER              INSERT             REVOKE             ROLLBACK│
    │   DROP               UPDATE                                SAVEPOINT│
    │   TRUNCATE           DELETE                                      │
    │                                                                  │
    │   Structure          Data               Permissions      Transactions│
    │                                                                  │
    └─────────────────────────────────────────────────────────────────┘
    
    
    FOCUS OF THIS CHAPTER:
    ───────────────────────
    
    DCL - Data Control Language
    ┌─────────────────────────────────────────────────┐
    │  Controls WHO can do WHAT in the database       │
    │  • GRANT - Give permissions                     │
    │  • REVOKE - Remove permissions                  │
    └─────────────────────────────────────────────────┘
    
    TCL - Transaction Control Language
    ┌─────────────────────────────────────────────────┐
    │  Controls HOW changes are saved/undone          │
    │  • COMMIT - Save changes permanently            │
    │  • ROLLBACK - Undo changes                      │
    │  • SAVEPOINT - Create restoration points        │
    └─────────────────────────────────────────────────┘
```

---

## 2. DCL - Data Control Language

### Privileges Overview

```
┌─────────────────────────────────────────────────────────────────────┐
│                    DATABASE PRIVILEGES                               │
└─────────────────────────────────────────────────────────────────────┘

    OBJECT PRIVILEGES (on specific objects):
    ─────────────────────────────────────────
    
    ┌────────────┬─────────────────────────────────────────────────────┐
    │ Privilege  │                   Description                       │
    ├────────────┼─────────────────────────────────────────────────────┤
    │ SELECT     │ Read data from table/view                           │
    │ INSERT     │ Add new rows to table                               │
    │ UPDATE     │ Modify existing rows                                │
    │ DELETE     │ Remove rows from table                              │
    │ REFERENCES │ Create foreign key referencing table               │
    │ TRIGGER    │ Create triggers on table                            │
    │ EXECUTE    │ Run stored procedures/functions                     │
    │ USAGE      │ Use sequences, schemas, etc.                        │
    │ ALL        │ All available privileges                            │
    └────────────┴─────────────────────────────────────────────────────┘


    SYSTEM PRIVILEGES (database-wide):
    ────────────────────────────────────
    
    ┌─────────────────────┬────────────────────────────────────────────┐
    │     Privilege       │              Description                   │
    ├─────────────────────┼────────────────────────────────────────────┤
    │ CREATE TABLE        │ Create new tables                          │
    │ CREATE VIEW         │ Create new views                           │
    │ CREATE USER         │ Create new database users                  │
    │ CREATE DATABASE     │ Create new databases                       │
    │ DROP ANY TABLE      │ Drop any table in database                 │
    │ GRANT ANY PRIVILEGE │ Grant any privilege to others             │
    └─────────────────────┴────────────────────────────────────────────┘


    PRIVILEGE HIERARCHY:
    ─────────────────────
    
                    ┌─────────────────┐
                    │  DBA (Super)    │
                    └────────┬────────┘
                             │
            ┌────────────────┼────────────────┐
            ↓                ↓                ↓
    ┌───────────────┐ ┌───────────────┐ ┌───────────────┐
    │ CREATE TABLE  │ │ CREATE USER   │ │ GRANT PRIVILEGE│
    └───────────────┘ └───────────────┘ └───────────────┘
            │
            ↓
    ┌──────────────────────────────────────────────────┐
    │  Table-level: SELECT, INSERT, UPDATE, DELETE    │
    └──────────────────────────────────────────────────┘
            │
            ↓
    ┌──────────────────────────────────────────────────┐
    │  Column-level: SELECT(col), UPDATE(col)         │
    └──────────────────────────────────────────────────┘
```

---

## 3. GRANT Command

```
┌─────────────────────────────────────────────────────────────────────┐
│                        GRANT COMMAND                                 │
└─────────────────────────────────────────────────────────────────────┘


    SYNTAX:
    ────────
    
    GRANT privilege_list ON object TO user_list [WITH GRANT OPTION];


    BASIC EXAMPLES:
    ─────────────────
    
    -- Grant SELECT on a table
    GRANT SELECT ON Student TO john;
    
    -- Grant multiple privileges
    GRANT SELECT, INSERT, UPDATE ON Student TO john, mary;
    
    -- Grant all privileges
    GRANT ALL ON Student TO admin_user;
    
    -- Grant to all users
    GRANT SELECT ON Department TO PUBLIC;


    COLUMN-LEVEL PRIVILEGES:
    ──────────────────────────
    
    -- Grant UPDATE only on specific columns
    GRANT UPDATE (Name, Age) ON Student TO hr_user;
    
    -- Grant SELECT only on non-sensitive columns
    GRANT SELECT (Emp_ID, Name, Dept_ID) ON Employee TO intern;


    WITH GRANT OPTION:
    ───────────────────
    
    Allows user to grant the same privilege to others.
    
    GRANT SELECT ON Student TO manager WITH GRANT OPTION;
    
    Now manager can:
    GRANT SELECT ON Student TO team_member;  -- Allowed!


    VISUALIZATION:
    ───────────────
    
    DBA grants to Manager:
    ┌─────────────────────────────────────────────────────────────────┐
    │                                                                  │
    │  DBA ────GRANT SELECT WITH GRANT OPTION────→ Manager           │
    │                                                                  │
    │           Manager can now grant to others:                      │
    │                                                                  │
    │           Manager ────GRANT SELECT────→ User1                   │
    │           Manager ────GRANT SELECT────→ User2                   │
    │                                                                  │
    └─────────────────────────────────────────────────────────────────┘


    GRANT ON VIEW:
    ───────────────
    
    -- Grant access to view (user can't see base table)
    GRANT SELECT ON CS_Students_View TO student_user;
    
    -- Revoke access to underlying table
    REVOKE SELECT ON Student FROM student_user;
```

---

## 4. REVOKE Command

```
┌─────────────────────────────────────────────────────────────────────┐
│                       REVOKE COMMAND                                 │
└─────────────────────────────────────────────────────────────────────┘


    SYNTAX:
    ────────
    
    REVOKE privilege_list ON object FROM user_list [CASCADE | RESTRICT];


    EXAMPLES:
    ──────────
    
    -- Revoke specific privilege
    REVOKE INSERT ON Student FROM john;
    
    -- Revoke multiple privileges
    REVOKE INSERT, UPDATE, DELETE ON Student FROM john;
    
    -- Revoke all privileges
    REVOKE ALL ON Student FROM john;
    
    -- Revoke from PUBLIC
    REVOKE SELECT ON Department FROM PUBLIC;


    CASCADE vs RESTRICT:
    ─────────────────────
    
    SCENARIO:
    DBA gave SELECT to Manager WITH GRANT OPTION
    Manager gave SELECT to User1 and User2
    
    Now DBA wants to revoke from Manager:
    
    REVOKE SELECT ON Student FROM manager CASCADE;
    → Also revokes from User1 and User2 (chain reaction)
    
    REVOKE SELECT ON Student FROM manager RESTRICT;
    → ERROR if manager has granted to others
    → Must revoke from User1, User2 first


    CASCADE VISUALIZATION:
    ───────────────────────
    
    Before REVOKE:
    ┌─────────────────────────────────────────────────────────────────┐
    │                                                                  │
    │   DBA ──→ Manager ──→ User1                                     │
    │               └──────→ User2                                     │
    │                                                                  │
    └─────────────────────────────────────────────────────────────────┘
    
    REVOKE ... FROM Manager CASCADE:
    ┌─────────────────────────────────────────────────────────────────┐
    │                                                                  │
    │   DBA     ╳Manager╳    ╳User1╳                                  │
    │                        ╳User2╳                                   │
    │                                                                  │
    │   All downstream grants also revoked!                           │
    │                                                                  │
    └─────────────────────────────────────────────────────────────────┘


    REVOKE GRANT OPTION:
    ─────────────────────
    
    Remove grant ability but keep the privilege:
    
    REVOKE GRANT OPTION FOR SELECT ON Student FROM manager;
    
    Manager can still SELECT, but cannot GRANT to others.
```

---

## 5. Roles

```
┌─────────────────────────────────────────────────────────────────────┐
│                          ROLES                                       │
└─────────────────────────────────────────────────────────────────────┘

    Roles group privileges together for easier management.
    Instead of granting privileges to each user, grant to role,
    then assign role to users.
    

    CREATING AND USING ROLES:
    ──────────────────────────
    
    -- Create a role
    CREATE ROLE hr_role;
    
    -- Grant privileges to role
    GRANT SELECT, INSERT, UPDATE ON Employee TO hr_role;
    GRANT SELECT ON Department TO hr_role;
    
    -- Assign role to users
    GRANT hr_role TO john, mary, bob;
    
    -- Remove role from user
    REVOKE hr_role FROM bob;
    
    -- Drop role
    DROP ROLE hr_role;


    WITHOUT ROLES vs WITH ROLES:
    ─────────────────────────────
    
    WITHOUT ROLES (tedious):
    ────────────────────────
    GRANT SELECT, INSERT ON Employee TO john;
    GRANT SELECT, INSERT ON Employee TO mary;
    GRANT SELECT, INSERT ON Employee TO bob;
    GRANT SELECT, INSERT ON Employee TO alice;
    ... (repeat for every new HR person)
    
    
    WITH ROLES (efficient):
    ────────────────────────
    GRANT SELECT, INSERT ON Employee TO hr_role;  -- Once
    GRANT hr_role TO john, mary, bob, alice;       -- Assign role
    
    -- New employee?
    GRANT hr_role TO new_person;  -- One command!


    ROLE HIERARCHY:
    ────────────────
    
    Roles can contain other roles:
    
    CREATE ROLE basic_access;
    CREATE ROLE hr_role;
    CREATE ROLE senior_hr;
    
    GRANT SELECT ON Employee TO basic_access;
    GRANT basic_access TO hr_role;  -- hr_role includes basic_access
    GRANT INSERT, UPDATE ON Employee TO hr_role;
    GRANT hr_role TO senior_hr;     -- senior_hr includes hr_role
    GRANT DELETE ON Employee TO senior_hr;
    
    
    ┌────────────────────────────────────────────────────────────────┐
    │                                                                 │
    │   senior_hr                                                    │
    │   ├── DELETE                                                   │
    │   └── hr_role                                                  │
    │       ├── INSERT, UPDATE                                       │
    │       └── basic_access                                         │
    │           └── SELECT                                           │
    │                                                                 │
    │   senior_hr has: SELECT, INSERT, UPDATE, DELETE                │
    │                                                                 │
    └────────────────────────────────────────────────────────────────┘
```

---

## 6. TCL - Transaction Control Language

```
┌─────────────────────────────────────────────────────────────────────┐
│                    WHAT IS A TRANSACTION?                            │
└─────────────────────────────────────────────────────────────────────┘

    A transaction is a sequence of operations performed as a 
    SINGLE LOGICAL UNIT of work.
    
    Either ALL operations complete successfully, or NONE do.
    
    
    EXAMPLE: Bank Transfer
    ───────────────────────
    
    Transfer $100 from Account A to Account B:
    
    Step 1: Deduct $100 from Account A
    Step 2: Add $100 to Account B
    
    Both must happen, or neither should happen!
    
    If only Step 1 completes → $100 is lost! (Bad)
    If only Step 2 completes → $100 appears from nowhere! (Bad)
    
    
    VISUALIZATION:
    ───────────────
    
    ┌─────────────────────────────────────────────────────────────────┐
    │                     TRANSACTION                                  │
    │                                                                  │
    │   BEGIN TRANSACTION                                             │
    │        │                                                        │
    │        ↓                                                        │
    │   ┌──────────────────┐                                         │
    │   │ UPDATE Account A │                                         │
    │   │ SET Balance = Balance - 100                                │
    │   └────────┬─────────┘                                         │
    │            ↓                                                    │
    │   ┌──────────────────┐                                         │
    │   │ UPDATE Account B │                                         │
    │   │ SET Balance = Balance + 100                                │
    │   └────────┬─────────┘                                         │
    │            ↓                                                    │
    │        SUCCESS?                                                 │
    │         ╱    ╲                                                  │
    │        ↓      ↓                                                 │
    │    COMMIT   ROLLBACK                                            │
    │   (Save)    (Undo)                                              │
    │                                                                  │
    └─────────────────────────────────────────────────────────────────┘
```

---

## 7. ACID Properties

```
┌─────────────────────────────────────────────────────────────────────┐
│                      ACID PROPERTIES                                 │
└─────────────────────────────────────────────────────────────────────┘

    Four essential properties that guarantee reliable transactions:
    

    A - ATOMICITY
    ──────────────
    
    "All or Nothing"
    
    Transaction is indivisible unit. Either all operations
    complete successfully, or the entire transaction is rolled back.
    
    ┌────────────────────────────────────────────────────┐
    │  Transaction: Update A, Update B, Update C        │
    │                                                    │
    │  ✓ All succeed → COMMIT all changes               │
    │  ✗ Any fails → ROLLBACK all changes               │
    └────────────────────────────────────────────────────┘


    C - CONSISTENCY
    ─────────────────
    
    "Valid State to Valid State"
    
    Transaction takes database from one consistent state to another.
    All integrity constraints are maintained.
    
    ┌────────────────────────────────────────────────────┐
    │  Before: Total money in bank = $1,000,000         │
    │  Transaction: Transfer $100                        │
    │  After: Total money in bank = $1,000,000          │
    │                                                    │
    │  Consistency: Total money unchanged!               │
    └────────────────────────────────────────────────────┘


    I - ISOLATION
    ──────────────
    
    "Independent Transactions"
    
    Concurrent transactions don't interfere with each other.
    Each transaction sees database as if it's the only one running.
    
    ┌────────────────────────────────────────────────────┐
    │                                                    │
    │  Transaction T1        Transaction T2             │
    │  ─────────────         ─────────────             │
    │  Read Balance: 1000    Read Balance: 1000        │
    │  Withdraw 100          Deposit 50                │
    │  Write: 900            Write: 1050               │
    │                                                    │
    │  Without isolation: Final might be 900 or 1050!  │
    │  With isolation: Final is correctly 950          │
    │                                                    │
    └────────────────────────────────────────────────────┘


    D - DURABILITY
    ───────────────
    
    "Permanent After Commit"
    
    Once committed, changes survive any system failure
    (crashes, power outages, etc.).
    
    ┌────────────────────────────────────────────────────┐
    │  COMMIT executed → Data written to disk           │
    │                  → Transaction log recorded       │
    │                  → Changes are PERMANENT          │
    │                                                    │
    │  Even if power fails right after COMMIT,          │
    │  data is recoverable from log.                    │
    └────────────────────────────────────────────────────┘


    ACID SUMMARY:
    ──────────────
    
    ┌────────────────┬───────────────────────────────────────────────┐
    │   Property     │              Guarantee                        │
    ├────────────────┼───────────────────────────────────────────────┤
    │ Atomicity      │ All operations succeed or all fail           │
    │ Consistency    │ Database stays in valid state                │
    │ Isolation      │ Concurrent transactions don't interfere      │
    │ Durability     │ Committed changes survive failures           │
    └────────────────┴───────────────────────────────────────────────┘
```

---

## 8. COMMIT and ROLLBACK

```
┌─────────────────────────────────────────────────────────────────────┐
│                    COMMIT AND ROLLBACK                               │
└─────────────────────────────────────────────────────────────────────┘


    COMMIT:
    ────────
    
    Saves all changes made in current transaction permanently.
    
    BEGIN TRANSACTION;
    UPDATE Account SET Balance = Balance - 100 WHERE ID = 'A';
    UPDATE Account SET Balance = Balance + 100 WHERE ID = 'B';
    COMMIT;  -- Changes are now permanent
    
    After COMMIT:
    • Changes visible to other transactions
    • Changes survive system crashes
    • Cannot be undone (without another transaction)


    ROLLBACK:
    ──────────
    
    Undoes all changes made in current transaction.
    
    BEGIN TRANSACTION;
    UPDATE Account SET Balance = Balance - 100 WHERE ID = 'A';
    UPDATE Account SET Balance = Balance + 100 WHERE ID = 'B';
    -- Oops, wrong amount!
    ROLLBACK;  -- All changes undone
    
    After ROLLBACK:
    • Database returns to state before transaction started
    • As if the transaction never happened


    VISUALIZATION:
    ───────────────
    
    ┌──────────────────────────────────────────────────────────────────┐
    │                                                                   │
    │   Initial State: Account A = $1000, Account B = $500            │
    │                                                                   │
    │   BEGIN TRANSACTION                                              │
    │   │                                                              │
    │   ├── UPDATE A: $1000 → $900                                    │
    │   │   (changes not yet permanent, only in transaction)          │
    │   │                                                              │
    │   ├── UPDATE B: $500 → $600                                     │
    │   │                                                              │
    │   └── Decision Point:                                            │
    │       │                                                          │
    │       ├── COMMIT                                                 │
    │       │   Final: A = $900, B = $600 (PERMANENT)                 │
    │       │                                                          │
    │       └── ROLLBACK                                               │
    │           Final: A = $1000, B = $500 (UNCHANGED)                │
    │                                                                   │
    └──────────────────────────────────────────────────────────────────┘


    AUTO-COMMIT MODE:
    ──────────────────
    
    Many databases have auto-commit enabled by default.
    Each statement is its own transaction.
    
    -- Auto-commit ON (default)
    UPDATE Student SET Age = 21;  -- Immediately committed
    
    -- Auto-commit OFF
    SET AUTOCOMMIT = 0;
    UPDATE Student SET Age = 21;  -- Not committed yet
    COMMIT;                       -- Now committed
```

---

## 9. SAVEPOINT

```
┌─────────────────────────────────────────────────────────────────────┐
│                        SAVEPOINT                                     │
└─────────────────────────────────────────────────────────────────────┘

    SAVEPOINT creates a named point within a transaction.
    You can rollback to a savepoint instead of entire transaction.
    

    SYNTAX:
    ────────
    
    SAVEPOINT savepoint_name;
    ROLLBACK TO savepoint_name;
    RELEASE SAVEPOINT savepoint_name;


    EXAMPLE:
    ─────────
    
    BEGIN TRANSACTION;
    
    UPDATE Account SET Balance = Balance - 100 WHERE ID = 'A';
    SAVEPOINT after_debit;
    
    UPDATE Account SET Balance = Balance + 100 WHERE ID = 'B';
    -- Oops, B is wrong account! Should be C
    
    ROLLBACK TO after_debit;
    -- Undoes only the B update, keeps A update
    
    UPDATE Account SET Balance = Balance + 100 WHERE ID = 'C';
    -- Correct account now
    
    COMMIT;


    VISUALIZATION:
    ───────────────
    
    ┌──────────────────────────────────────────────────────────────────┐
    │                                                                   │
    │   BEGIN TRANSACTION                                              │
    │        │                                                         │
    │        │ Operation 1: UPDATE A                                   │
    │        ●────────────────────────────── SAVEPOINT sp1             │
    │        │                                                         │
    │        │ Operation 2: UPDATE B                                   │
    │        ●────────────────────────────── SAVEPOINT sp2             │
    │        │                                                         │
    │        │ Operation 3: UPDATE C (ERROR!)                          │
    │        │                                                         │
    │        │ ROLLBACK TO sp2                                         │
    │        │ ←──────────────────────────── Back to sp2               │
    │        │                                                         │
    │        │ Operation 3 (retry): UPDATE C                           │
    │        │                                                         │
    │        │ COMMIT                                                  │
    │        ↓                                                         │
    │   All saved except rolled-back operation                         │
    │                                                                   │
    └──────────────────────────────────────────────────────────────────┘


    MULTIPLE SAVEPOINTS:
    ─────────────────────
    
    BEGIN TRANSACTION;
    
    INSERT INTO Orders VALUES (1, 'Laptop', 1000);
    SAVEPOINT order1;
    
    INSERT INTO Orders VALUES (2, 'Mouse', 50);
    SAVEPOINT order2;
    
    INSERT INTO Orders VALUES (3, 'Keyboard', 100);
    SAVEPOINT order3;
    
    -- Problem with order 3
    ROLLBACK TO order2;  -- Undo order 3 only
    
    -- Problem with order 2 too
    ROLLBACK TO order1;  -- Undo order 2 and 3
    
    COMMIT;  -- Only order 1 is saved


    NESTED TRANSACTIONS (simulated with savepoints):
    ──────────────────────────────────────────────────
    
    BEGIN TRANSACTION;
    
    -- "Outer" transaction work
    INSERT INTO Log VALUES ('Started');
    
    SAVEPOINT nested;
    -- "Inner" transaction work
    UPDATE Inventory SET Qty = Qty - 1;
    -- If inner fails:
    ROLLBACK TO nested;
    
    -- Continue outer transaction
    INSERT INTO Log VALUES ('Completed');
    
    COMMIT;
```

---

## 10. Transaction States

```
┌─────────────────────────────────────────────────────────────────────┐
│                     TRANSACTION STATES                               │
└─────────────────────────────────────────────────────────────────────┘


    STATE DIAGRAM:
    ───────────────
    
                          ┌─────────┐
                          │  BEGIN  │
                          │ (Active)│
                          └────┬────┘
                               │
                    Operations │ executing
                               ↓
                    ┌─────────────────────┐
                    │ PARTIALLY COMMITTED │
                    │ (Operations done,   │
                    │  not yet committed) │
                    └──────────┬──────────┘
                               │
                        ┌──────┴──────┐
                        │ All OK?     │
                        │             │
                    YES ↓             ↓ NO (Error)
               ┌───────────┐    ┌──────────┐
               │ COMMITTED │    │  FAILED  │
               │           │    │          │
               └───────────┘    └────┬─────┘
                                     │
                                     ↓
                              ┌───────────┐
                              │ ABORTED   │
                              │(Rolled back)│
                              └───────────┘


    STATE DESCRIPTIONS:
    ────────────────────
    
    ┌──────────────────────┬─────────────────────────────────────────┐
    │       State          │              Description                │
    ├──────────────────────┼─────────────────────────────────────────┤
    │ Active               │ Transaction is executing                │
    │ Partially Committed  │ Final statement executed, awaiting     │
    │                      │ commit confirmation                     │
    │ Committed            │ Changes made permanent                  │
    │ Failed               │ Error detected, cannot proceed         │
    │ Aborted              │ Transaction rolled back, DB restored   │
    └──────────────────────┴─────────────────────────────────────────┘


    EXAMPLE FLOW:
    ──────────────
    
    BEGIN TRANSACTION;          → State: ACTIVE
    UPDATE Account SET ...;     → State: ACTIVE
    UPDATE Account SET ...;     → State: ACTIVE
    -- All operations done      → State: PARTIALLY COMMITTED
    COMMIT;                     → State: COMMITTED
    
    OR
    
    BEGIN TRANSACTION;          → State: ACTIVE
    UPDATE Account SET ...;     → State: ACTIVE
    -- Error occurs!            → State: FAILED
    ROLLBACK;                   → State: ABORTED
```

---

## 📊 Summary Table

| DCL Command | Purpose | Example |
|-------------|---------|---------|
| **GRANT** | Give permissions | `GRANT SELECT ON t TO u;` |
| **REVOKE** | Remove permissions | `REVOKE SELECT ON t FROM u;` |
| **CREATE ROLE** | Create privilege group | `CREATE ROLE admin;` |

| TCL Command | Purpose | Example |
|-------------|---------|---------|
| **COMMIT** | Save changes permanently | `COMMIT;` |
| **ROLLBACK** | Undo all changes | `ROLLBACK;` |
| **SAVEPOINT** | Create restore point | `SAVEPOINT sp1;` |
| **ROLLBACK TO** | Undo to savepoint | `ROLLBACK TO sp1;` |

| ACID Property | Meaning | Ensures |
|---------------|---------|---------|
| **Atomicity** | All or nothing | Complete or no changes |
| **Consistency** | Valid states | Constraints maintained |
| **Isolation** | Independent | No interference |
| **Durability** | Permanent | Survives failures |

---

## ❓ Quick Revision Questions

1. **What is the difference between GRANT and REVOKE?**
   <details>
   <summary>Click for Answer</summary>
   GRANT gives permissions to users or roles (e.g., GRANT SELECT ON table TO user). REVOKE removes previously granted permissions (e.g., REVOKE SELECT ON table FROM user).
   </details>

2. **What does WITH GRANT OPTION do?**
   <details>
   <summary>Click for Answer</summary>
   WITH GRANT OPTION allows the recipient to grant the same privilege to other users. Example: GRANT SELECT ON table TO user WITH GRANT OPTION; Now 'user' can GRANT SELECT to others.
   </details>

3. **What are the four ACID properties?**
   <details>
   <summary>Click for Answer</summary>
   Atomicity (all or nothing), Consistency (valid state to valid state), Isolation (concurrent transactions don't interfere), Durability (committed changes survive failures).
   </details>

4. **What is the difference between COMMIT and ROLLBACK?**
   <details>
   <summary>Click for Answer</summary>
   COMMIT saves all changes made in the current transaction permanently. ROLLBACK undoes all changes made in the current transaction, restoring the database to its state before the transaction began.
   </details>

5. **What is a SAVEPOINT and when would you use it?**
   <details>
   <summary>Click for Answer</summary>
   SAVEPOINT creates a named point within a transaction. You can ROLLBACK TO savepoint to undo only part of a transaction. Useful when you want to undo recent operations without losing earlier work in the same transaction.
   </details>

6. **Why use roles instead of granting privileges directly to users?**
   <details>
   <summary>Click for Answer</summary>
   Roles simplify privilege management. Grant privileges to a role once, then assign the role to multiple users. When privileges change, update the role instead of each user. Easier to manage, audit, and modify.
   </details>

---

## 🔗 Navigation

| Previous | Up | Next |
|----------|----|----|
| [← Views](05-views.md) | [📚 Table of Contents](../README.md) | [Unit 4: Functional Dependencies →](../04-Database-Design/01-functional-dependencies.md) |

---

*End of Unit 3: SQL*

---

*Last Updated: January 2026*
