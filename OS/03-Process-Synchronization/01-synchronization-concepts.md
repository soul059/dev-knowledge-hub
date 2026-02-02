# Chapter 3.1: Synchronization Concepts

## Overview

When multiple processes or threads access shared data concurrently, problems can arise. Process synchronization provides mechanisms to ensure orderly execution and maintain data consistency.

---

## Why Synchronization?

### The Problem with Concurrent Access

Consider two processes trying to update a shared counter:

```
┌──────────────────────────────────────────────────────────────────┐
│                    THE SYNCHRONIZATION PROBLEM                    │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Shared Variable: counter = 5                                    │
│                                                                  │
│  Process A: counter++          Process B: counter++              │
│                                                                  │
│  At machine level:             At machine level:                 │
│  1. Load counter into R1       1. Load counter into R2          │
│  2. Increment R1               2. Increment R2                  │
│  3. Store R1 to counter        3. Store R2 to counter           │
│                                                                  │
│  Expected Result: counter = 7                                    │
│  Possible Result: counter = 6 (one update lost!)                │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

### Example: Interleaved Execution

```
Time    Process A                    Process B              counter
────    ─────────                    ─────────              ───────
  0     (counter = 5 initially)                                5
  
  1     R1 = counter  (R1 = 5)                                 5
  2                                  R2 = counter (R2 = 5)     5
  3     R1 = R1 + 1   (R1 = 6)                                 5
  4                                  R2 = R2 + 1  (R2 = 6)     5
  5     counter = R1  (counter = 6)                            6
  6                                  counter = R2 (counter = 6) 6
  
Final: counter = 6 (Expected: 7) — ONE INCREMENT LOST!
```

---

## Race Condition

### Definition
A **Race Condition** occurs when multiple processes access and manipulate shared data concurrently, and the outcome depends on the particular order of access (the "race" between processes).

### Analogy
Think of two people trying to withdraw money from the same bank account:

```
Bank Account Balance: $1000

Person A (ATM 1)              Person B (ATM 2)
────────────────              ────────────────
Read balance: $1000           Read balance: $1000
Withdraw $800                 Withdraw $500
Write balance: $200           Write balance: $500

Final Balance: $500 (should be -$300 or blocked!)
The bank lost $800!
```

### When Race Conditions Occur

```
Race conditions occur when:
1. Multiple processes/threads access shared data
2. At least one of them performs a write operation
3. Access is not synchronized

┌──────────────────────────────────────────────────────────────────┐
│  Conditions for Race:                                            │
│                                                                  │
│  ✓ Shared resource (variable, file, device)                     │
│  ✓ Concurrent access by multiple processes                      │
│  ✓ At least one write operation                                 │
│  ✓ No synchronization mechanism                                 │
│                                                                  │
│  If ANY condition is missing → No race condition                │
└──────────────────────────────────────────────────────────────────┘
```

---

## Critical Section

### Definition
A **Critical Section** is a segment of code where shared resources are accessed. When one process is executing in its critical section, no other process should be allowed to execute in its critical section.

### Structure of a Process with Critical Section

```
┌──────────────────────────────────────────────────────────────────┐
│                      PROCESS STRUCTURE                            │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  do {                                                            │
│      ┌─────────────────────────────────┐                        │
│      │     ENTRY SECTION               │ ← Request permission   │
│      │   (acquire lock/permission)     │                        │
│      └─────────────────────────────────┘                        │
│                                                                  │
│      ┌─────────────────────────────────┐                        │
│      │     CRITICAL SECTION            │ ← Access shared data   │
│      │   (access shared resources)     │   Only ONE process     │
│      │                                 │   at a time here       │
│      └─────────────────────────────────┘                        │
│                                                                  │
│      ┌─────────────────────────────────┐                        │
│      │     EXIT SECTION                │ ← Release permission   │
│      │   (release lock/permission)     │                        │
│      └─────────────────────────────────┘                        │
│                                                                  │
│      ┌─────────────────────────────────┐                        │
│      │     REMAINDER SECTION           │ ← Non-critical work    │
│      │   (other processing)            │                        │
│      └─────────────────────────────────┘                        │
│  } while (true);                                                 │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

### Visual Representation

```
Process P1:                          Process P2:
┌────────────────────┐              ┌────────────────────┐
│   Entry Section    │              │   Entry Section    │
│   ┌──────────────┐ │              │   ┌──────────────┐ │
│   │   Wait...    │ │              │   │   Enter!     │ │
│   └──────────────┘ │              │   └──────────────┘ │
│         ↓          │              │         ↓          │
│   Critical Section │              │   Critical Section │
│   ┌──────────────┐ │              │   ┌──────────────┐ │
│   │   BLOCKED    │ │              │   │  EXECUTING   │ │
│   └──────────────┘ │              │   └──────────────┘ │
│         ↓          │              │         ↓          │
│   Exit Section     │              │   Exit Section     │
└────────────────────┘              └────────────────────┘

When P2 is in critical section, P1 must wait!
```

---

## Requirements for Critical Section Solution

Any solution to the critical section problem must satisfy three requirements:

### 1. Mutual Exclusion

```
┌──────────────────────────────────────────────────────────────────┐
│                     MUTUAL EXCLUSION                              │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Definition: If process Pi is executing in its critical section,│
│  then no other process can execute in its critical section.     │
│                                                                  │
│                                                                  │
│        Critical Section                                          │
│        ┌────────────────┐                                        │
│  P1 ──→│   Only ONE     │←── P2 (must wait!)                    │
│        │   process      │                                        │
│        │   at a time    │←── P3 (must wait!)                    │
│        └────────────────┘                                        │
│                                                                  │
│  Guarantees: Exclusive access to shared resources                │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

### 2. Progress

```
┌──────────────────────────────────────────────────────────────────┐
│                         PROGRESS                                  │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Definition: If no process is in its critical section and       │
│  some processes wish to enter, then only those processes NOT    │
│  executing in their remainder section can participate in        │
│  deciding which will enter next, and this selection cannot      │
│  be postponed indefinitely.                                     │
│                                                                  │
│  In simple terms:                                                │
│  • If critical section is free and processes want to enter,    │
│    one of them must be allowed to enter                         │
│  • Decision should not take forever                             │
│                                                                  │
│                    Critical Section                              │
│                    ┌────────────────┐                            │
│    P1 waiting ────→│    EMPTY!      │                           │
│    P2 waiting ────→│  One must      │                           │
│                    │  enter soon    │                           │
│                    └────────────────┘                            │
│                                                                  │
│  Prevents: Deadlock where everyone waits but CS is empty        │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

### 3. Bounded Waiting

```
┌──────────────────────────────────────────────────────────────────┐
│                     BOUNDED WAITING                               │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Definition: There must be a limit on how many times other      │
│  processes can enter their critical section after a process     │
│  has made a request to enter and before that request is         │
│  granted.                                                       │
│                                                                  │
│  In simple terms:                                                │
│  • No process should wait forever (no starvation)               │
│  • After requesting, entry is guaranteed within bounded time    │
│                                                                  │
│  Timeline:                                                       │
│  P1: Request │─────────── Waiting ───────────│ Entry            │
│              ↑                               ↑                   │
│              │    Maximum N other entries    │                   │
│              │   (bounded, not infinite)     │                   │
│                                                                  │
│  Prevents: Starvation (indefinite blocking)                     │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

### Summary of Requirements

| Requirement | What It Prevents | Guarantee |
|-------------|-----------------|-----------|
| **Mutual Exclusion** | Race conditions | Only one in CS |
| **Progress** | Deadlock | Someone enters if CS free |
| **Bounded Waiting** | Starvation | Everyone gets a turn |

---

## Approaches to Synchronization

### 1. Software Solutions

Algorithms implemented purely in software without special hardware support.

```
Examples:
• Peterson's Solution
• Dekker's Algorithm
• Bakery Algorithm (for n processes)
```

### 2. Hardware Solutions

Using special hardware instructions provided by the processor.

```
Examples:
• Test-and-Set instruction
• Compare-and-Swap instruction
• Disable Interrupts
```

### 3. OS-Provided Solutions

Higher-level abstractions provided by the operating system.

```
Examples:
• Semaphores
• Mutexes
• Monitors
• Condition Variables
```

---

## Disabling Interrupts (Hardware Approach)

### Concept
On a uniprocessor system, prevent context switches by disabling interrupts.

```
┌──────────────────────────────────────────────────────────────────┐
│                    DISABLE INTERRUPTS                             │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Process code:                                                   │
│                                                                  │
│  disable_interrupts();    ← No context switch possible          │
│                                                                  │
│  // CRITICAL SECTION                                             │
│  // Access shared data                                           │
│  // No preemption can occur                                      │
│                                                                  │
│  enable_interrupts();     ← Context switches allowed again      │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

### Limitations

| Issue | Description |
|-------|-------------|
| **Uniprocessor only** | Doesn't work on multiprocessors |
| **Dangerous** | User process controls interrupts |
| **System unresponsive** | No interrupts = no I/O handling |
| **Kernel only** | Not suitable for user programs |

---

## Peterson's Solution (Software Approach)

### For Two Processes

A classic software solution for two processes (P0 and P1).

### Shared Variables

```c
int turn;           // Whose turn is it?
bool flag[2];       // flag[i] = true means Pi wants to enter
```

### Algorithm

```c
// Process Pi (where i is 0 or 1, j is the other process)

do {
    flag[i] = true;     // I want to enter
    turn = j;           // But I'll let you go first
    
    while (flag[j] && turn == j)
        ;               // Wait while other wants to enter AND it's their turn
    
    // CRITICAL SECTION
    
    flag[i] = false;    // I'm done, I don't want to enter anymore
    
    // REMAINDER SECTION
    
} while (true);
```

### Visualization

```
┌──────────────────────────────────────────────────────────────────┐
│                    PETERSON'S SOLUTION                            │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Initial: flag[0] = false, flag[1] = false, turn = ?            │
│                                                                  │
│  P0 wants to enter:                                              │
│  1. flag[0] = true    "I want to enter"                         │
│  2. turn = 1          "But you can go first"                    │
│  3. while (flag[1] && turn == 1)                                │
│     └─→ Wait if P1 wants to enter AND it's P1's turn            │
│                                                                  │
│  P1 wants to enter (simultaneously):                             │
│  1. flag[1] = true    "I want to enter"                         │
│  2. turn = 0          "But you can go first"                    │
│  3. while (flag[0] && turn == 0)                                │
│     └─→ Wait if P0 wants to enter AND it's P0's turn            │
│                                                                  │
│  Result: Whoever sets 'turn' LAST will wait                     │
│          (because they deferred to the other)                   │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

### Proof of Correctness

```
1. MUTUAL EXCLUSION:
   For both P0 and P1 to be in CS:
   - flag[0] = true, flag[1] = true
   - P0 passed: (flag[1] == false) OR (turn == 0)
   - P1 passed: (flag[0] == false) OR (turn == 1)
   
   But turn can only be 0 OR 1, not both!
   So if both flags are true, one must be waiting.
   ✓ Mutual exclusion guaranteed

2. PROGRESS:
   If only Pi wants to enter (flag[j] = false),
   Pi's while loop exits immediately.
   ✓ Progress guaranteed

3. BOUNDED WAITING:
   After Pi sets turn = j and enters waiting loop,
   If Pj enters CS, when Pj exits it sets flag[j] = false,
   So Pi will enter. At most one entry by Pj.
   ✓ Bounded waiting guaranteed
```

### Limitations of Peterson's Solution

| Limitation | Description |
|------------|-------------|
| **Only two processes** | Need different algorithm for n processes |
| **Busy waiting** | CPU cycles wasted in while loop |
| **Modern processors** | May not work due to instruction reordering |
| **Not atomic** | Assumes atomic read/write of shared variables |

---

## Hardware Instructions

Modern processors provide atomic instructions for synchronization.

### Test-and-Set (TAS)

```c
// Executed atomically by hardware
bool test_and_set(bool *target) {
    bool rv = *target;    // Save old value
    *target = true;       // Set to true
    return rv;            // Return old value
}
```

**Using Test-and-Set for Mutual Exclusion:**

```c
// Shared variable
bool lock = false;

// Process Pi
do {
    while (test_and_set(&lock))
        ;                    // Wait while lock is true
    
    // CRITICAL SECTION
    
    lock = false;            // Release lock
    
    // REMAINDER SECTION
    
} while (true);
```

### Compare-and-Swap (CAS)

```c
// Executed atomically by hardware
int compare_and_swap(int *value, int expected, int new_value) {
    int temp = *value;
    if (*value == expected)
        *value = new_value;
    return temp;             // Return original value
}
```

**Using Compare-and-Swap:**

```c
// Shared variable
int lock = 0;

// Process Pi
do {
    while (compare_and_swap(&lock, 0, 1) != 0)
        ;                    // Wait until we successfully set lock
    
    // CRITICAL SECTION
    
    lock = 0;                // Release lock
    
    // REMAINDER SECTION
    
} while (true);
```

### Comparison

| Instruction | Operation | Use Case |
|-------------|-----------|----------|
| **Test-and-Set** | Read and set to true atomically | Simple locks |
| **Compare-and-Swap** | Conditional swap based on expected value | Lock-free data structures |

---

## Busy Waiting Problem

### What is Busy Waiting?

When a process continuously checks a condition in a loop while waiting.

```c
// Busy waiting (also called "spinlock")
while (lock == true)
    ;    // Process does nothing but check, wasting CPU!
```

### Problems with Busy Waiting

```
┌──────────────────────────────────────────────────────────────────┐
│                    BUSY WAITING ISSUES                            │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  1. CPU Waste:                                                   │
│     Process uses CPU cycles just to check a condition           │
│                                                                  │
│  2. Priority Inversion:                                          │
│     High-priority process waiting (spinning) while              │
│     low-priority process holds the lock                         │
│     Low-priority process can't run to release lock!             │
│                                                                  │
│  3. Not efficient for long waits                                 │
│                                                                  │
│  When Acceptable:                                                │
│  • Very short critical sections                                 │
│  • Multiprocessor systems                                       │
│  • When sleep/wakeup overhead > spin time                       │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

### Solution: Sleep and Wakeup

Instead of busy waiting, block the process:

```
Busy Waiting:                    Sleep/Wakeup:
while (lock)                     if (lock)
    ;   // Spin, waste CPU          sleep();  // Block, free CPU
                                 // Woken up when lock available
```

---

## Summary

| Concept | Description |
|---------|-------------|
| **Race Condition** | Outcome depends on execution order |
| **Critical Section** | Code that accesses shared resources |
| **Mutual Exclusion** | Only one process in CS at a time |
| **Progress** | Free CS + waiting processes = entry |
| **Bounded Waiting** | No starvation, bounded entry time |
| **Peterson's Solution** | Software solution for 2 processes |
| **Test-and-Set** | Hardware atomic instruction |
| **Busy Waiting** | Continuous checking in loop |

---

## Quick Revision Questions

1. What is a race condition? Give an example.
2. List the three requirements for solving the critical section problem.
3. Explain how Peterson's solution ensures mutual exclusion.
4. What is the test-and-set instruction?
5. Why is busy waiting problematic?

---

[← Previous: IPC](../02-Process-Management/06-inter-process-communication.md) | [Next: Critical Section Problem →](./02-critical-section-problem.md)
