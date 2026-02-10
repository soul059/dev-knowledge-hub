# Chapter 5.2: Concurrency Control

[← Back to Table of Contents](../README.md)

---

## 📚 Chapter Overview

**Concurrency control** ensures that multiple transactions can execute simultaneously without violating database consistency. It prevents problems that arise when transactions interleave their operations.

```
┌─────────────────────────────────────────────────────────────────┐
│                     CHAPTER LEARNING GOALS                       │
├─────────────────────────────────────────────────────────────────┤
│  • Understand why concurrency control is needed                 │
│  • Learn about concurrency problems                             │
│  • Master schedule types and serializability                    │
│  • Understand locking protocols                                 │
│  • Learn timestamp-based protocols                              │
└─────────────────────────────────────────────────────────────────┘
```

---

## 1. Why Concurrency Control?

```
┌─────────────────────────────────────────────────────────────────────┐
│                   WHY CONCURRENT EXECUTION?                          │
└─────────────────────────────────────────────────────────────────────┘

    Modern databases serve many users simultaneously.
    Concurrent execution provides:
    
    
    1. IMPROVED THROUGHPUT
    ───────────────────────
    
    Sequential:                  Concurrent:
    ┌────┐┌────┐┌────┐          ┌────┐
    │ T1 ││ T2 ││ T3 │          │ T1 │─── CPU
    └────┘└────┘└────┘          ├────┤
    ───────────────────→        │ T2 │─── CPU
         Time                   ├────┤
                                │ T3 │─── CPU
    Total: 3 time units         └────┘
                                Total: 1 time unit (if parallel)
    
    
    2. BETTER RESOURCE UTILIZATION
    ────────────────────────────────
    
    While T1 waits for disk I/O, T2 can use CPU:
    
    Time:  1   2   3   4   5   6
           ─────────────────────
    T1:    CPU │ IO │ CPU
    T2:        │ CPU│    │ CPU
    
    Resources stay busy!
    
    
    3. REDUCED WAITING TIME
    ─────────────────────────
    
    Users don't wait for others to finish.
    Short transactions can run between long ones.


┌─────────────────────────────────────────────────────────────────────┐
│                  THE PROBLEM WITH CONCURRENCY                        │
└─────────────────────────────────────────────────────────────────────┘

    Without control, concurrent transactions can interfere:
    
    ┌─────────────────────────────────────────────────────────────────┐
    │                                                                  │
    │   T1: Read(A); A=A-100; Write(A); Read(B); B=B+100; Write(B);   │
    │                                                                  │
    │   T2: Read(A); A=A*1.1; Write(A); Read(B); B=B*1.1; Write(B);   │
    │                                                                  │
    │   If operations interleave incorrectly → DATA CORRUPTION!       │
    │                                                                  │
    └─────────────────────────────────────────────────────────────────┘
    
    Concurrency control = Manage interleaving to prevent problems
```

---

## 2. Concurrency Problems

```
┌─────────────────────────────────────────────────────────────────────┐
│                    CONCURRENCY PROBLEMS                              │
└─────────────────────────────────────────────────────────────────────┘


    1. LOST UPDATE PROBLEM
    ───────────────────────
    
    Two transactions update same data; one update is lost.
    
    Initial: A = 100
    
    Time     T1                    T2
    ────     ────────────          ────────────
     t1      Read(A) → 100
     t2                            Read(A) → 100
     t3      A = A - 10
             A = 90
     t4                            A = A + 20
                                   A = 120
     t5      Write(A) = 90
     t6                            Write(A) = 120
    
    Final: A = 120 (T1's update to 90 is LOST!)
    Should be: 100 - 10 + 20 = 110


    2. DIRTY READ (Uncommitted Dependency)
    ────────────────────────────────────────
    
    Reading data written by an uncommitted transaction.
    
    Initial: A = 100
    
    Time     T1                    T2
    ────     ────────────          ────────────
     t1      Read(A) → 100
     t2      A = A - 50
     t3      Write(A) = 50
     t4                            Read(A) → 50  ← Dirty read!
     t5      ROLLBACK              
             (A back to 100)
     t6                            Use A = 50    ← Wrong value!
    
    T2 used a value that was never committed!


    3. UNREPEATABLE READ (Inconsistent Read)
    ─────────────────────────────────────────
    
    Same query returns different results within same transaction.
    
    Time     T1                    T2
    ────     ────────────          ────────────
     t1      Read(A) → 100
     t2                            Read(A) → 100
     t3                            A = A + 50
     t4                            Write(A) = 150
     t5                            COMMIT
     t6      Read(A) → 150         
    
    T1 reads A twice, gets different values (100 and 150)!
    

    4. PHANTOM READ
    ─────────────────
    
    New rows appear in repeated query (like ghosts).
    
    Time     T1                            T2
    ────     ────────────────────          ────────────
     t1      SELECT COUNT(*) 
             WHERE age > 20
             → Returns 5 rows
     t2                                    INSERT INTO table
                                          VALUES (...age=25...)
     t3                                    COMMIT
     t4      SELECT COUNT(*)
             WHERE age > 20
             → Returns 6 rows   ← Phantom!
    
    Same query, different result due to new "phantom" row.


    SUMMARY:
    ─────────
    
    ┌──────────────────┬────────────────────────────────────────────┐
    │     Problem      │             What Goes Wrong                │
    ├──────────────────┼────────────────────────────────────────────┤
    │ Lost Update      │ One transaction's changes overwritten      │
    │ Dirty Read       │ Reading uncommitted data                   │
    │ Unrepeatable Read│ Same read returns different values         │
    │ Phantom Read     │ New rows appear in query                   │
    └──────────────────┴────────────────────────────────────────────┘
```

---

## 3. Schedules and Serializability

```
┌─────────────────────────────────────────────────────────────────────┐
│                         SCHEDULES                                    │
└─────────────────────────────────────────────────────────────────────┘

    SCHEDULE: A sequence of operations from multiple transactions
    that shows their order of execution.
    

    EXAMPLE:
    ─────────
    
    T1: Read(A), Write(A), Read(B), Write(B)
    T2: Read(A), Write(A), Read(B), Write(B)
    
    
    SERIAL SCHEDULE (one after another):
    
    Schedule S1: T1 then T2
    ┌────────────────────────────────────────────────────────────────┐
    │ R1(A) W1(A) R1(B) W1(B) │ R2(A) W2(A) R2(B) W2(B)             │
    └─────────────────────────┴──────────────────────────────────────┘
           T1 operations              T2 operations
    
    
    NON-SERIAL SCHEDULE (interleaved):
    
    Schedule S2: Interleaved execution
    ┌────────────────────────────────────────────────────────────────┐
    │ R1(A) R2(A) W1(A) W2(A) R1(B) R2(B) W1(B) W2(B)               │
    └────────────────────────────────────────────────────────────────┘
         Operations from T1 and T2 mixed


┌─────────────────────────────────────────────────────────────────────┐
│                      SERIALIZABILITY                                 │
└─────────────────────────────────────────────────────────────────────┘

    A schedule is SERIALIZABLE if its effect is equivalent to
    some serial schedule.
    
    
    WHY SERIAL SCHEDULES ARE CORRECT:
    ──────────────────────────────────
    
    Serial execution = No interference = Correct result
    
    If T1 → T2 (serial):
    • T1 completes fully before T2 starts
    • No interleaving, no problems
    
    
    WHY WE DON'T USE SERIAL SCHEDULES:
    ────────────────────────────────────
    
    Too slow! No concurrency, no parallelism.
    
    
    SERIALIZABILITY = Best of both worlds:
    ───────────────────────────────────────
    
    ┌─────────────────────────────────────────────────────────────────┐
    │                                                                  │
    │   Serial Schedule:          Serializable Schedule:              │
    │                                                                  │
    │   Correct but SLOW          Correct AND FAST                    │
    │                                                                  │
    │   T1 ─────────────          T1 ─────┬─────                      │
    │         T2 ─────────              T2 ──┴───────                 │
    │                                                                  │
    │   No parallelism            Has parallelism                     │
    │                             Same final result!                   │
    │                                                                  │
    └─────────────────────────────────────────────────────────────────┘


    CONFLICT SERIALIZABILITY:
    ──────────────────────────
    
    Two operations CONFLICT if:
    1. They belong to different transactions
    2. They access the same data item
    3. At least one is a WRITE
    
    
    Conflict types:
    
    ┌──────────────────┬────────────────────────────────────────────┐
    │   Operations     │              Conflict?                     │
    ├──────────────────┼────────────────────────────────────────────┤
    │ Read(A), Read(A) │ NO (both reads)                           │
    │ Read(A), Write(A)│ YES (Read-Write conflict)                 │
    │ Write(A), Read(A)│ YES (Write-Read conflict)                 │
    │ Write(A),Write(A)│ YES (Write-Write conflict)                │
    └──────────────────┴────────────────────────────────────────────┘
    
    
    A schedule is conflict serializable if we can swap
    non-conflicting operations to get a serial schedule.


    PRECEDENCE GRAPH (Testing Serializability):
    ─────────────────────────────────────────────
    
    1. Create node for each transaction
    2. Add edge Ti → Tj if Ti has operation before conflicting Tj operation
    3. If graph has NO CYCLE → Conflict serializable
    
    
    Example Schedule: R1(A) R2(A) W1(A) W2(A) R1(B) R2(B) W1(B) W2(B)
    
    Conflicts:
    • R1(A) before W2(A) → T1 → T2
    • R2(A) before W1(A) → T2 → T1
    • W1(A) before W2(A) → T1 → T2
    • etc.
    
    Graph:
          T1
         ↗  ↘
        ↙    ↖
          T2
    
    CYCLE exists! → NOT conflict serializable
```

---

## 4. Lock-Based Protocols

```
┌─────────────────────────────────────────────────────────────────────┐
│                    LOCK-BASED CONCURRENCY CONTROL                    │
└─────────────────────────────────────────────────────────────────────┘

    LOCK: Permission to access a data item
    
    Before accessing data, transaction must acquire lock.
    After done, transaction releases lock.
    

    TYPES OF LOCKS:
    ─────────────────
    
    ┌─────────────────┬────────────────────────────────────────────┐
    │   Lock Type     │            Description                     │
    ├─────────────────┼────────────────────────────────────────────┤
    │ Shared (S)      │ For reading                                │
    │ "Read Lock"     │ Multiple transactions can hold S-lock      │
    │                 │ on same item simultaneously                │
    ├─────────────────┼────────────────────────────────────────────┤
    │ Exclusive (X)   │ For writing                                │
    │ "Write Lock"    │ Only ONE transaction can hold X-lock       │
    │                 │ No other locks allowed on item             │
    └─────────────────┴────────────────────────────────────────────┘
    
    
    LOCK COMPATIBILITY MATRIX:
    ───────────────────────────
    
                      Lock held by other transaction
                    ┌─────────────────────────────────┐
                    │    S-lock    │    X-lock       │
         ┌──────────┼──────────────┼─────────────────┤
    Lock │  S-lock  │      ✓       │       ✗        │
    requested       │  Compatible  │  NOT Compatible│
         ├──────────┼──────────────┼─────────────────┤
         │  X-lock  │      ✗       │       ✗        │
         │          │NOT Compatible│  NOT Compatible│
         └──────────┴──────────────┴─────────────────┘
    
    Multiple S-locks OK!
    X-lock requires exclusive access.


    LOCK OPERATIONS:
    ─────────────────
    
    lock-S(A)    - Request shared lock on A
    lock-X(A)    - Request exclusive lock on A
    unlock(A)    - Release lock on A
    
    
    EXAMPLE:
    ─────────
    
    T1: Transfer $100 from A to B
    
    lock-X(A)
    Read(A)
    A = A - 100
    Write(A)
    unlock(A)
    lock-X(B)
    Read(B)
    B = B + 100
    Write(B)
    unlock(B)
    
    
    PROBLEM - This protocol allows violations!
    ───────────────────────────────────────────
    
    Time     T1                    T2
    ────     ────────────          ────────────
     t1      lock-X(A)
     t2      Read(A)=100
     t3      A = A - 100
     t4      Write(A)=0
     t5      unlock(A)
     t6                            lock-X(A)
     t7                            Read(A)=0
     t8      lock-X(B)  ← WAIT!    A = A + 100
                        (T2 hasn't│ Write(A)=100
                         unlocked)│ unlock(A)
     t9                            lock-X(B)
     t10                           Read(B)=0
             ...
    
    T2 reads A=0 after T1's deduction, sees INTERMEDIATE state!
```

---

## 5. Two-Phase Locking (2PL)

```
┌─────────────────────────────────────────────────────────────────────┐
│                    TWO-PHASE LOCKING (2PL)                           │
└─────────────────────────────────────────────────────────────────────┘

    2PL RULE: Transaction has two phases:
    
    1. GROWING PHASE: Acquire locks, never release
    2. SHRINKING PHASE: Release locks, never acquire
    
    
    ┌─────────────────────────────────────────────────────────────────┐
    │                                                                  │
    │   Number of                                                      │
    │   Locks held                                                     │
    │       │                                                          │
    │       │         /\                                              │
    │       │        /  \                                             │
    │       │       /    \                                            │
    │       │      /      \                                           │
    │       │     /        \                                          │
    │       │    /          \                                         │
    │       │   /            \                                        │
    │       │  /              \                                       │
    │       │ /                \                                      │
    │       └───────────────────────→ Time                            │
    │         │  Growing  │ Shrinking │                               │
    │         │   Phase   │   Phase   │                               │
    │         └───────────┴───────────┘                               │
    │                     ↑                                            │
    │              Lock Point (all locks acquired)                    │
    │                                                                  │
    └─────────────────────────────────────────────────────────────────┘
    
    
    2PL GUARANTEES SERIALIZABILITY!
    
    
    EXAMPLE:
    ─────────
    
    WITHOUT 2PL (violation):
    
    lock(A) → unlock(A) → lock(B) → unlock(B)
                 ↑             ↑
         Released early   Acquired after release
         (Not 2PL!)
    
    
    WITH 2PL:
    
    lock(A) → lock(B) → unlock(A) → unlock(B)
    │────── Growing ──────│───── Shrinking ─────│
              ↑
    All locks acquired before any release


    2PL VARIATIONS:
    ─────────────────
    
    1. BASIC 2PL
    ──────────────
    As described above. May have cascading rollback problem.
    
    
    2. STRICT 2PL
    ──────────────
    Hold all EXCLUSIVE locks until commit/abort.
    Prevents cascading rollback.
    
    ┌─────────────────────────────────────────────────────────────────┐
    │                                                                  │
    │   Locks │      /───────────────────\                            │
    │         │     /                     │                           │
    │         │    /                      │                           │
    │         │   /         X-locks       │ Release all               │
    │         │  /          held          │ at commit                 │
    │         └───────────────────────────┴───────→ Time              │
    │            Growing        │         │                           │
    │                          Commit     │                           │
    │                                                                  │
    └─────────────────────────────────────────────────────────────────┘
    
    
    3. RIGOROUS 2PL
    ─────────────────
    Hold ALL locks (S and X) until commit/abort.
    Strongest form.


    CASCADING ROLLBACK PROBLEM:
    ────────────────────────────
    
    T1 writes, T2 reads (dirty read), T1 aborts → T2 must rollback!
    
    Time     T1                    T2
    ────     ────────────          ────────────
     t1      lock-X(A)
     t2      Write(A)
     t3      unlock(A)
     t4                            lock-S(A)
     t5                            Read(A)  ← Dirty read
     t6      ROLLBACK
     t7                            MUST ROLLBACK! (cascade)
    
    Strict 2PL prevents this by holding X-locks until commit.
```

---

## 6. Deadlock

```
┌─────────────────────────────────────────────────────────────────────┐
│                          DEADLOCK                                    │
└─────────────────────────────────────────────────────────────────────┘

    DEADLOCK: Circular waiting where transactions wait for each
    other indefinitely.
    

    EXAMPLE:
    ─────────
    
    Time     T1                    T2
    ────     ────────────          ────────────
     t1      lock-X(A) ✓
     t2                            lock-X(B) ✓
     t3      lock-X(B)             lock-X(A)
             WAIT for T2!          WAIT for T1!
             ↓                     ↓
             ∞                     ∞
    
    
    WAIT-FOR GRAPH:
    ─────────────────
    
         T1 ──────────→ T2
          ↑              │
          └──────────────┘
    
    Cycle = DEADLOCK!


    DEADLOCK HANDLING:
    ───────────────────
    
    
    1. DEADLOCK PREVENTION
    ───────────────────────
    
    Don't let deadlock happen in the first place.
    
    Methods:
    
    a) Wait-Die Scheme:
       - Older transaction waits for younger
       - Younger transaction dies (rollback)
       
       T1 (older) wants lock held by T2 (younger) → T1 WAITS
       T2 (younger) wants lock held by T1 (older) → T2 DIES
    
    
    b) Wound-Wait Scheme:
       - Older transaction "wounds" (forces rollback) younger
       - Younger transaction waits for older
       
       T1 (older) wants lock held by T2 (younger) → T2 WOUNDED (rollback)
       T2 (younger) wants lock held by T1 (older) → T2 WAITS
    
    
    ┌────────────────┬─────────────────┬─────────────────┐
    │   Scenario     │    Wait-Die     │   Wound-Wait    │
    ├────────────────┼─────────────────┼─────────────────┤
    │ Older → Younger│     WAITS       │   WOUNDS (RB)   │
    │ Younger → Older│    DIES (RB)    │     WAITS       │
    └────────────────┴─────────────────┴─────────────────┘
    
    
    2. DEADLOCK DETECTION
    ──────────────────────
    
    Let deadlock happen, then detect and resolve.
    
    Steps:
    a) Periodically build wait-for graph
    b) Check for cycles
    c) If cycle found → VICTIM SELECTION (one transaction must die)
    d) Rollback victim
    
    
    VICTIM SELECTION CRITERIA:
    ───────────────────────────
    
    Choose transaction that minimizes cost:
    • How long it has been running
    • How many updates it has made
    • How much longer it needs
    • How many other transactions depend on it
    
    
    3. TIMEOUT
    ───────────────
    
    If transaction waits too long → assume deadlock → rollback
    
    Simple but may cause unnecessary rollbacks.
```

---

## 7. Timestamp-Based Protocols

```
┌─────────────────────────────────────────────────────────────────────┐
│                   TIMESTAMP-BASED PROTOCOLS                          │
└─────────────────────────────────────────────────────────────────────┘

    Alternative to locking: Use TIMESTAMPS to order transactions.
    
    
    TIMESTAMP:
    ───────────
    
    Each transaction T gets unique timestamp TS(T) when it starts.
    
    Can be:
    • System clock value
    • Logical counter (increments each transaction)
    
    Older transaction = smaller timestamp
    

    TIMESTAMP ORDERING PROTOCOL:
    ─────────────────────────────
    
    Each data item X has:
    • W-timestamp(X): Timestamp of last transaction that wrote X
    • R-timestamp(X): Timestamp of last transaction that read X
    
    
    RULES:
    ───────
    
    Transaction T wants to READ(X):
    ─────────────────────────────────
    
    If TS(T) < W-timestamp(X):
        X was written by a later transaction
        T is reading a stale value
        → ROLLBACK T
    Else:
        Execute read
        R-timestamp(X) = max(R-timestamp(X), TS(T))
    
    
    Transaction T wants to WRITE(X):
    ─────────────────────────────────
    
    If TS(T) < R-timestamp(X):
        A later transaction already read old value
        T's write would violate history
        → ROLLBACK T
        
    Else if TS(T) < W-timestamp(X):
        A later transaction already wrote
        T's write is obsolete
        → ROLLBACK T (or use Thomas Write Rule: skip write)
        
    Else:
        Execute write
        W-timestamp(X) = TS(T)


    EXAMPLE:
    ─────────
    
    T1: TS = 10
    T2: TS = 15
    
    Initial: W-timestamp(A) = 5, R-timestamp(A) = 5
    
    Time     T2                    T1
    ────     ────────────          ────────────
     t1      Read(A)
             TS(T2)=15 > W-ts(A)=5 ✓
             R-timestamp(A) = 15
             
     t2                            Write(A)
                                   TS(T1)=10 < R-ts(A)=15 ✗
                                   ROLLBACK T1!
    
    T1's write would affect T2's read (which already happened).
    Can't allow this → T1 must rollback.


    THOMAS WRITE RULE:
    ───────────────────
    
    If TS(T) < W-timestamp(X):
        Instead of rollback, just IGNORE the write
        (A newer write already exists)
    
    This is called the "Thomas Write Rule" or "Ignore obsolete writes"


    COMPARISON: Locking vs Timestamps
    ───────────────────────────────────
    
    ┌─────────────────┬───────────────────┬───────────────────┐
    │    Aspect       │      Locking      │    Timestamps     │
    ├─────────────────┼───────────────────┼───────────────────┤
    │ Waiting         │ Transaction waits │ No waiting        │
    │                 │ for lock          │ (rollback instead)│
    ├─────────────────┼───────────────────┼───────────────────┤
    │ Deadlock        │ Possible          │ Not possible      │
    ├─────────────────┼───────────────────┼───────────────────┤
    │ Rollbacks       │ Less frequent     │ More frequent     │
    ├─────────────────┼───────────────────┼───────────────────┤
    │ Starvation      │ Possible          │ Possible          │
    └─────────────────┴───────────────────┴───────────────────┘
```

---

## 8. Isolation Levels

```
┌─────────────────────────────────────────────────────────────────────┐
│                      SQL ISOLATION LEVELS                            │
└─────────────────────────────────────────────────────────────────────┘

    SQL defines 4 isolation levels - trade-offs between
    consistency and concurrency.
    
    
    ┌───────────────────┬─────────┬─────────────┬─────────┬─────────┐
    │ Isolation Level   │ Dirty   │ Unrepeatable│ Phantom │ Perf.   │
    │                   │ Read    │    Read     │  Read   │         │
    ├───────────────────┼─────────┼─────────────┼─────────┼─────────┤
    │ READ UNCOMMITTED  │  YES    │     YES     │   YES   │ Fastest │
    │ READ COMMITTED    │   NO    │     YES     │   YES   │  Fast   │
    │ REPEATABLE READ   │   NO    │      NO     │   YES   │  Slow   │
    │ SERIALIZABLE      │   NO    │      NO     │    NO   │ Slowest │
    └───────────────────┴─────────┴─────────────┴─────────┴─────────┘
    
    YES = Problem can occur
    NO = Problem prevented
    

    LEVEL DESCRIPTIONS:
    ────────────────────
    
    1. READ UNCOMMITTED (Lowest)
       • No read locks
       • Can read uncommitted data
       • Rarely used (data may be corrupted)
    
    2. READ COMMITTED (Default in many databases)
       • Only reads committed data
       • But same query may return different results
       • Each query sees data as of query start
    
    3. REPEATABLE READ
       • All reads within transaction see same snapshot
       • But new rows (phantoms) may appear
    
    4. SERIALIZABLE (Highest)
       • Complete isolation
       • As if transactions ran one at a time
       • Slowest but safest


    SQL SYNTAX:
    ────────────
    
    SET TRANSACTION ISOLATION LEVEL READ COMMITTED;
    
    SET TRANSACTION ISOLATION LEVEL SERIALIZABLE;


    CHOOSING ISOLATION LEVEL:
    ──────────────────────────
    
    ┌─────────────────────────────────────────────────────────────────┐
    │                                                                  │
    │   Need accuracy?               Need performance?                │
    │        │                            │                           │
    │        ↓                            ↓                           │
    │   SERIALIZABLE                READ UNCOMMITTED                  │
    │                                     or                          │
    │                               READ COMMITTED                    │
    │                                                                  │
    │   Examples:                   Examples:                         │
    │   • Banking                   • Reporting                       │
    │   • Inventory                 • Analytics                       │
    │   • Reservations              • Logging                         │
    │                                                                  │
    └─────────────────────────────────────────────────────────────────┘
```

---

## 📊 Summary Table

| Concurrency Problem | Description | Prevention |
|---------------------|-------------|------------|
| **Lost Update** | One write overwrites another | Exclusive locks |
| **Dirty Read** | Reading uncommitted data | Hold locks until commit |
| **Unrepeatable Read** | Same read, different values | Keep read locks |
| **Phantom Read** | New rows appear | Table/range locks |

| Lock Type | Purpose | Compatibility |
|-----------|---------|---------------|
| **Shared (S)** | Reading | Multiple S-locks OK |
| **Exclusive (X)** | Writing | Only one X-lock |

| Protocol | Mechanism | Deadlock |
|----------|-----------|----------|
| **2PL** | Growing & Shrinking phases | Possible |
| **Strict 2PL** | Hold X-locks until commit | Possible |
| **Timestamp** | Order by timestamps | Not possible |

| Isolation Level | Dirty Read | Unrepeatable | Phantom |
|-----------------|------------|--------------|---------|
| **READ UNCOMMITTED** | ✓ | ✓ | ✓ |
| **READ COMMITTED** | ✗ | ✓ | ✓ |
| **REPEATABLE READ** | ✗ | ✗ | ✓ |
| **SERIALIZABLE** | ✗ | ✗ | ✗ |

---

## ❓ Quick Revision Questions

1. **What is the lost update problem?**
   <details>
   <summary>Click for Answer</summary>
   When two transactions read the same value, both modify it, and write back. The second write overwrites the first, "losing" that update. Example: Both read balance=100, T1 subtracts 10, T2 adds 20, final is 120 instead of correct 110.
   </details>

2. **What is Two-Phase Locking (2PL)?**
   <details>
   <summary>Click for Answer</summary>
   A protocol with two phases: Growing (acquire locks, never release) and Shrinking (release locks, never acquire). Once a transaction releases any lock, it cannot acquire new locks. 2PL guarantees serializability.
   </details>

3. **How does a precedence graph test for serializability?**
   <details>
   <summary>Click for Answer</summary>
   Create a node for each transaction. Add edge Ti→Tj if Ti has an operation before a conflicting operation in Tj. If the graph has no cycles, the schedule is conflict serializable.
   </details>

4. **What is deadlock and how can it be prevented?**
   <details>
   <summary>Click for Answer</summary>
   Deadlock is when transactions wait for each other in a cycle. Prevention methods: Wait-Die (older waits, younger dies), Wound-Wait (older wounds younger, younger waits), or timeout-based rollback.
   </details>

5. **How does timestamp ordering work?**
   <details>
   <summary>Click for Answer</summary>
   Each transaction gets a timestamp at start. Each data item tracks R-timestamp and W-timestamp. Operations are allowed only if they don't violate timestamp order. If violation detected, transaction is rolled back.
   </details>

6. **What is the difference between READ COMMITTED and REPEATABLE READ?**
   <details>
   <summary>Click for Answer</summary>
   READ COMMITTED: Each query sees latest committed data; same query may return different results. REPEATABLE READ: All reads in transaction see same snapshot; same query returns same results. Both prevent dirty reads.
   </details>

---

## 🔗 Navigation

| Previous | Up | Next |
|----------|----|----|
| [← Transaction Concepts](01-transaction-concepts.md) | [📚 Table of Contents](../README.md) | [Recovery Techniques →](03-recovery-techniques.md) |

---

*Last Updated: January 2026*
