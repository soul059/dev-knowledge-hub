# Chapter 3.3: Semaphores

## Overview

Semaphores are synchronization primitives introduced by Edsger Dijkstra in 1965. They provide a more structured and less error-prone way to achieve synchronization compared to hardware-based solutions.

---

## What is a Semaphore?

### Definition

A **Semaphore** is an integer variable that, apart from initialization, is accessed only through two standard atomic operations: **wait()** and **signal()**.

```
┌──────────────────────────────────────────────────────────────────┐
│                       SEMAPHORE                                   │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Components:                                                     │
│  • Integer value (≥ 0 for counting semaphore)                   │
│  • Queue of waiting processes (optional, for blocking)          │
│                                                                  │
│  Operations:                                                     │
│  • wait(S) or P(S) or down(S)  - Decrease, possibly block       │
│  • signal(S) or V(S) or up(S)  - Increase, possibly wake        │
│                                                                  │
│  Note: P and V come from Dutch words:                           │
│        P = Proberen (to test)                                   │
│        V = Verhogen (to increment)                              │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

### Analogy

```
Think of a semaphore as a parking lot counter:

┌─────────────────────────────────────────┐
│           PARKING LOT                    │
│                                         │
│   Available Spaces: 3  ← Semaphore value│
│   ┌────┐ ┌────┐ ┌────┐                  │
│   │    │ │    │ │    │   3 empty spots  │
│   └────┘ └────┘ └────┘                  │
│                                         │
│   wait(S): Car enters, spaces--         │
│   signal(S): Car leaves, spaces++       │
│                                         │
│   If spaces = 0, cars must WAIT         │
│                                         │
└─────────────────────────────────────────┘
```

---

## Types of Semaphores

### 1. Binary Semaphore (Mutex)

```
┌──────────────────────────────────────────────────────────────────┐
│                    BINARY SEMAPHORE                               │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Value: Can only be 0 or 1                                       │
│                                                                  │
│  • 1 = Resource available (unlocked)                            │
│  • 0 = Resource in use (locked)                                 │
│                                                                  │
│  Also called: Mutex (Mutual Exclusion lock)                     │
│                                                                  │
│  Use Case: Protecting critical sections                         │
│            Only ONE process can access at a time                │
│                                                                  │
│  Initialization: mutex = 1                                       │
│                                                                  │
│      Process:                                                    │
│      wait(mutex);        // Lock                                │
│      // Critical Section                                        │
│      signal(mutex);      // Unlock                              │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

### 2. Counting Semaphore

```
┌──────────────────────────────────────────────────────────────────┐
│                   COUNTING SEMAPHORE                              │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Value: Can be any non-negative integer (0, 1, 2, 3, ...)       │
│                                                                  │
│  Use Cases:                                                      │
│  • Control access to resource with multiple instances           │
│  • Count available resources                                    │
│  • Synchronize producer-consumer                                │
│                                                                  │
│  Example: Pool of 5 database connections                        │
│                                                                  │
│  Initialization: db_pool = 5                                     │
│                                                                  │
│  ┌─────────────────────────────────────┐                        │
│  │  Connection Pool: 5                 │                        │
│  │  ┌──┐ ┌──┐ ┌──┐ ┌──┐ ┌──┐          │                        │
│  │  │C1│ │C2│ │C3│ │C4│ │C5│          │                        │
│  │  └──┘ └──┘ └──┘ └──┘ └──┘          │                        │
│  │                                     │                        │
│  │  wait(db_pool) → Get connection    │                        │
│  │  signal(db_pool) → Return connection│                       │
│  └─────────────────────────────────────┘                        │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

---

## Semaphore Operations

### Busy-Wait Implementation (Spinlock Semaphore)

```c
// wait (P) operation
wait(S) {
    while (S <= 0)
        ;          // Busy wait (spin)
    S--;
}

// signal (V) operation
signal(S) {
    S++;
}
```

**Problems with Busy-Wait:**
- Wastes CPU cycles
- Inefficient for long waits

### Blocking Implementation (No Busy-Wait)

```c
typedef struct {
    int value;
    struct process *list;    // Queue of waiting processes
} semaphore;

// wait (P) operation
wait(semaphore *S) {
    S->value--;
    if (S->value < 0) {
        // Add this process to S->list
        block();             // Suspend the process
    }
}

// signal (V) operation
signal(semaphore *S) {
    S->value++;
    if (S->value <= 0) {
        // Remove a process P from S->list
        wakeup(P);           // Resume process P
    }
}
```

**Visualization:**

```
┌──────────────────────────────────────────────────────────────────┐
│              BLOCKING SEMAPHORE OPERATION                         │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Initial: S.value = 1, S.list = empty                           │
│                                                                  │
│  P1: wait(S)                                                     │
│      S.value = 0                                                │
│      0 >= 0, no block                                           │
│      P1 enters CS                                               │
│                                                                  │
│  P2: wait(S)  (while P1 in CS)                                  │
│      S.value = -1                                               │
│      -1 < 0, P2 blocked                                         │
│      S.list = [P2]                                              │
│                                                                  │
│  P3: wait(S)                                                     │
│      S.value = -2                                               │
│      -2 < 0, P3 blocked                                         │
│      S.list = [P2, P3]                                          │
│                                                                  │
│  P1: signal(S)                                                   │
│      S.value = -1                                               │
│      -1 <= 0, wakeup P2                                         │
│      S.list = [P3]                                              │
│      P2 enters CS                                               │
│                                                                  │
│  Note: |S.value| when negative = number of waiting processes    │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

---

## Using Semaphores

### 1. Mutual Exclusion

```c
semaphore mutex = 1;    // Binary semaphore initialized to 1

// Process Pi
do {
    wait(mutex);        // Entry section
    
    // CRITICAL SECTION
    
    signal(mutex);      // Exit section
    
    // REMAINDER SECTION
} while (true);
```

**Execution Trace:**

```
Time    P1              P2              mutex
────    ──              ──              ─────
  0     wait(mutex)                       1
  1     (mutex=0)                         0
  2     IN CS                             0
  3                     wait(mutex)       0
  4                     BLOCKED          -1
  5     signal(mutex)                    -1
  6     (wakeup P2)                       0
  7                     IN CS             0
  8                     signal(mutex)     0
  9                     (mutex=1)         1
```

### 2. Process Synchronization (Ordering)

Ensure S2 in P2 executes only after S1 in P1.

```c
semaphore sync = 0;     // Initialize to 0

// Process P1              // Process P2
S1;                        wait(sync);    // Wait for signal
signal(sync);              S2;            // Execute after S1
```

**How it Works:**

```
┌──────────────────────────────────────────────────────────────────┐
│                  SYNCHRONIZATION EXAMPLE                          │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Scenario 1: P1 runs first                                       │
│  1. P1 executes S1                                               │
│  2. P1 does signal(sync), sync = 1                               │
│  3. P2 does wait(sync), sync = 0, proceeds                       │
│  4. P2 executes S2                                               │
│                                                                  │
│  Scenario 2: P2 runs first                                       │
│  1. P2 does wait(sync), sync = -1, P2 blocks                     │
│  2. P1 executes S1                                               │
│  3. P1 does signal(sync), sync = 0, wakes P2                     │
│  4. P2 executes S2                                               │
│                                                                  │
│  Either way: S1 always happens before S2!                        │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

### 3. Resource Counting

```c
semaphore printers = 3;     // 3 printers available

// Process requesting printer
wait(printers);             // Acquire a printer
// Use printer
signal(printers);           // Release printer
```

---

## Common Patterns

### Signaling Pattern

One process signals another to proceed:

```
semaphore signal = 0;

Process A:              Process B:
   ...                     wait(signal);  // Wait for A
   signal(signal);         // Continue after A signals
   ...
```

### Rendezvous Pattern

Two processes wait for each other at a point:

```
semaphore aArrived = 0;
semaphore bArrived = 0;

Process A:              Process B:
   ...                     ...
   signal(aArrived);       signal(bArrived);
   wait(bArrived);         wait(aArrived);
   // Both here             // Both here
```

### Multiplex Pattern

Allow up to N processes in critical section:

```c
semaphore multiplex = N;    // Allow N concurrent

// Each process
wait(multiplex);
// Critical section (up to N processes)
signal(multiplex);
```

---

## Potential Problems with Semaphores

### 1. Deadlock

```c
semaphore S = 1;
semaphore Q = 1;

// Process P1              // Process P2
wait(S);                   wait(Q);
wait(Q);                   wait(S);    // Deadlock!
...                        ...
signal(S);                 signal(Q);
signal(Q);                 signal(S);
```

```
┌──────────────────────────────────────────────────────────────────┐
│                      DEADLOCK SCENARIO                            │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Time 1: P1 does wait(S) - acquires S                            │
│  Time 2: P2 does wait(Q) - acquires Q                            │
│  Time 3: P1 does wait(Q) - blocks (Q held by P2)                │
│  Time 4: P2 does wait(S) - blocks (S held by P1)                │
│                                                                  │
│          P1 ────── holds S, wants Q                              │
│          │                    │                                  │
│          │    DEADLOCK!       │                                  │
│          │                    │                                  │
│          P2 ────── holds Q, wants S                              │
│                                                                  │
│  Solution: Always acquire semaphores in same order!             │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

### 2. Starvation

A process waiting indefinitely because others keep getting the semaphore.

```
Solution: Use FIFO queue for waiting processes
```

### 3. Priority Inversion

```
┌──────────────────────────────────────────────────────────────────┐
│                   PRIORITY INVERSION                              │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Scenario:                                                       │
│  • Low priority process L holds semaphore S                     │
│  • High priority process H wants S, blocks                      │
│  • Medium priority process M preempts L                         │
│                                                                  │
│  Result: H waits for M (indirectly), even though H > M         │
│                                                                  │
│  Priority: H > M > L                                            │
│                                                                  │
│  Time: ─────────────────────────────────────────────>           │
│                                                                  │
│  L:    [Running, holds S][Preempted by M     ][Finishes]        │
│  M:                       [Running           ]                   │
│  H:          [Blocked waiting for S──────────][Runs]            │
│                                                                  │
│  H waits for M even though H has higher priority!               │
│                                                                  │
│  Solution: Priority Inheritance Protocol                        │
│  L temporarily inherits H's priority while holding S            │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

---

## Semaphore vs Mutex

| Aspect | Semaphore | Mutex |
|--------|-----------|-------|
| **Value Range** | 0 to N | 0 or 1 only |
| **Ownership** | No ownership concept | Owned by locking thread |
| **Release** | Any process can signal | Only owner can unlock |
| **Purpose** | Synchronization + Counting | Mutual exclusion only |
| **Use Case** | Resource pools, signaling | Protecting critical sections |

---

## Implementation Notes

### Atomicity Requirement

Wait and signal must be atomic:
- On uniprocessor: Disable interrupts
- On multiprocessor: Use hardware atomic instructions (test-and-set)

```c
// Atomic implementation of wait using test-and-set
wait(S) {
    while (TestAndSet(&S->guard))
        ;   // Acquire guard lock
    S->value--;
    if (S->value < 0) {
        // Add to queue
        S->guard = 0;   // Release guard
        block();
    } else {
        S->guard = 0;   // Release guard
    }
}
```

---

## Summary

| Concept | Description |
|---------|-------------|
| **Semaphore** | Integer variable with atomic wait/signal |
| **Binary Semaphore** | Value 0 or 1, for mutual exclusion |
| **Counting Semaphore** | Value ≥ 0, for resource counting |
| **wait(S)** | Decrease S, block if S < 0 |
| **signal(S)** | Increase S, wake one blocked process |
| **Deadlock** | Circular wait on semaphores |
| **Priority Inversion** | High priority waits for lower |

---

## Quick Revision Questions

1. What is the difference between binary and counting semaphores?
2. How does the blocking implementation of wait/signal work?
3. Explain how semaphores achieve process synchronization (ordering).
4. What causes deadlock with semaphores? How to prevent it?
5. What is priority inversion? How can it be solved?

---

[← Previous: Critical Section Problem](./02-critical-section-problem.md) | [Next: Classical Synchronization Problems →](./04-classical-problems.md)
