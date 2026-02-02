# Chapter 3.2: Critical Section Problem

## Overview

The Critical Section Problem is a fundamental challenge in concurrent programming. This chapter explores various solutions, from simple but flawed attempts to correct algorithms that satisfy all requirements.

---

## Formal Problem Statement

```
┌──────────────────────────────────────────────────────────────────┐
│                  THE CRITICAL SECTION PROBLEM                     │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Given: n processes {P0, P1, ..., Pn-1}                         │
│                                                                  │
│  Each process has:                                               │
│  • Critical Section (CS): Code accessing shared resources       │
│  • Remainder Section: Other code                                │
│                                                                  │
│  Goal: Design entry and exit protocols such that:               │
│  1. Mutual Exclusion is ensured                                 │
│  2. Progress is maintained                                      │
│  3. Bounded Waiting is guaranteed                               │
│                                                                  │
│  Assumptions:                                                    │
│  • Each process executes at non-zero speed                      │
│  • No assumption about relative speeds                          │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

---

## Incorrect Solutions (Learning from Mistakes)

### Attempt 1: Using a Single Variable

```c
// Shared variable
int turn = 0;  // Whose turn to enter CS

// Process P0                    // Process P1
while (turn != 0)                while (turn != 1)
    ;                                ;
// CRITICAL SECTION              // CRITICAL SECTION
turn = 1;                        turn = 0;
// REMAINDER SECTION             // REMAINDER SECTION
```

**Analysis:**

```
┌──────────────────────────────────────────────────────────────────┐
│                    ATTEMPT 1 ANALYSIS                             │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  ✓ Mutual Exclusion: YES                                        │
│    Only one process can have turn matching its ID               │
│                                                                  │
│  ✗ Progress: NO                                                  │
│    Problem: Strict alternation required!                        │
│                                                                  │
│    Scenario:                                                     │
│    - P0 enters CS, exits, sets turn = 1                         │
│    - P0 wants to enter again (P1 is in remainder section)       │
│    - P0 must wait for P1 to enter and exit first!               │
│    - If P1 never wants CS, P0 waits forever                     │
│                                                                  │
│  Result: FAILS (violates progress)                              │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

### Attempt 2: Using Flags Only

```c
// Shared variables
bool flag[2] = {false, false};  // flag[i] = Pi wants to enter

// Process Pi (i = 0 or 1, j = other process)
flag[i] = true;                 // I want to enter
while (flag[j])                 // Wait if other wants to enter
    ;
// CRITICAL SECTION
flag[i] = false;                // I'm done
// REMAINDER SECTION
```

**Analysis:**

```
┌──────────────────────────────────────────────────────────────────┐
│                    ATTEMPT 2 ANALYSIS                             │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  ✗ Mutual Exclusion: NO (potentially)                           │
│  ✗ Progress: NO (deadlock possible)                             │
│                                                                  │
│  Deadlock Scenario:                                              │
│  Time 1: P0 sets flag[0] = true                                 │
│  Time 2: P1 sets flag[1] = true                                 │
│  Time 3: P0 checks flag[1] → true → waits                       │
│  Time 4: P1 checks flag[0] → true → waits                       │
│                                                                  │
│  Both processes wait forever! DEADLOCK!                         │
│                                                                  │
│        P0                      P1                                │
│    flag[0]=true            flag[1]=true                         │
│        │                       │                                │
│        ▼                       ▼                                │
│    while(flag[1])          while(flag[0])                       │
│        │                       │                                │
│        └───── Both waiting ────┘                                │
│                                                                  │
│  Result: FAILS (deadlock violates progress)                     │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

### Attempt 3: Check Then Set Flag

```c
// Process Pi
while (flag[j])                 // First check if other wants
    ;
flag[i] = true;                 // Then declare I want
// CRITICAL SECTION
flag[i] = false;
// REMAINDER SECTION
```

**Analysis:**

```
┌──────────────────────────────────────────────────────────────────┐
│                    ATTEMPT 3 ANALYSIS                             │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  ✗ Mutual Exclusion: NO                                         │
│                                                                  │
│  Race Condition Scenario:                                        │
│  Time 1: P0 checks flag[1] → false → proceeds                   │
│  Time 2: P1 checks flag[0] → false → proceeds                   │
│  Time 3: P0 sets flag[0] = true                                 │
│  Time 4: P1 sets flag[1] = true                                 │
│  Time 5: Both enter critical section!                           │
│                                                                  │
│  The check and set are not atomic!                              │
│                                                                  │
│  Result: FAILS (violates mutual exclusion)                      │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

---

## Correct Solutions

### Dekker's Algorithm (Historical First Solution)

The first known correct solution for two processes.

```c
// Shared variables
bool flag[2] = {false, false};
int turn = 0;

// Process Pi (i = 0 or 1, j = other)
flag[i] = true;                     // I want to enter
while (flag[j]) {                   // While other wants to enter
    if (turn == j) {                // If it's other's turn
        flag[i] = false;            // Back off
        while (turn == j)           // Wait for my turn
            ;
        flag[i] = true;             // Try again
    }
}
// CRITICAL SECTION
turn = j;                           // Give turn to other
flag[i] = false;                    // I'm done
// REMAINDER SECTION
```

**How Dekker's Works:**

```
┌──────────────────────────────────────────────────────────────────┐
│                    DEKKER'S ALGORITHM                             │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Key Insight: Combines flag[] for intention with turn for       │
│  conflict resolution                                            │
│                                                                  │
│  Scenario Analysis:                                              │
│                                                                  │
│  Case 1: Only Pi wants to enter                                 │
│  • flag[j] = false, so while loop exits immediately            │
│  • Pi enters CS                                                  │
│                                                                  │
│  Case 2: Both want to enter, turn = i                           │
│  • Pi: flag[j] true, but turn ≠ j, stays in outer loop         │
│  • Pj: flag[i] true, turn = i (= j for Pj), backs off          │
│  • Pj sets flag[j] = false, waits for turn                     │
│  • Pi's while(flag[j]) exits, Pi enters CS                      │
│                                                                  │
│  Case 3: Both want, contention                                   │
│  • The one whose turn it is will eventually enter               │
│  • After exiting, it gives turn to the other                    │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

### Peterson's Algorithm (Simpler and Elegant)

```c
// Shared variables
bool flag[2] = {false, false};
int turn;

// Process Pi (i = 0 or 1, j = other)
flag[i] = true;                     // I want to enter
turn = j;                           // But you can go first
while (flag[j] && turn == j)        // Wait if you want AND it's your turn
    ;
// CRITICAL SECTION
flag[i] = false;                    // I'm done
// REMAINDER SECTION
```

**Why Peterson's is Simpler:**

| Aspect | Dekker's | Peterson's |
|--------|----------|------------|
| Back-off mechanism | Yes | No |
| Lines of code | More | Less |
| Easy to understand | Moderate | Easy |
| Symmetric | No | Yes |

---

## Bakery Algorithm (n Processes)

Lamport's Bakery Algorithm extends the solution to n processes.

### Analogy

```
┌──────────────────────────────────────────────────────────────────┐
│                    BAKERY ANALOGY                                 │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Imagine a bakery where customers take numbered tickets:        │
│                                                                  │
│  1. Customer enters, takes a number (higher than all others)    │
│  2. Waits until their number is the smallest                    │
│  3. Gets served (enters critical section)                       │
│  4. Leaves (exits critical section)                             │
│                                                                  │
│  Ticket Numbers:                                                 │
│  ┌─────┬─────┬─────┬─────┬─────┐                                │
│  │  3  │  1  │  5  │  2  │  4  │                                │
│  │ P0  │ P1  │ P2  │ P3  │ P4  │                                │
│  └─────┴─────┴─────┴─────┴─────┘                                │
│         ↑                                                        │
│    P1 served next (smallest non-zero number)                    │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

### Algorithm

```c
// Shared variables
bool choosing[n];      // choosing[i] = true means Pi is getting a number
int number[n];         // number[i] = Pi's ticket number (0 means not interested)

// Initialize
for (int i = 0; i < n; i++) {
    choosing[i] = false;
    number[i] = 0;
}

// Process Pi
do {
    // DOORWAY: Get a ticket number
    choosing[i] = true;
    number[i] = 1 + max(number[0], number[1], ..., number[n-1]);
    choosing[i] = false;
    
    // WAITING: Wait for my turn
    for (int j = 0; j < n; j++) {
        // Wait while Pj is choosing
        while (choosing[j])
            ;
        // Wait while Pj has priority (smaller number, or same number but smaller id)
        while (number[j] != 0 && 
               (number[j] < number[i] || 
                (number[j] == number[i] && j < i)))
            ;
    }
    
    // CRITICAL SECTION
    
    number[i] = 0;      // I'm done, reset my number
    
    // REMAINDER SECTION
    
} while (true);
```

### Handling Ties

```
What if two processes get the same number?

Process ID is used as tiebreaker:
(number[i], i) < (number[j], j) means:
  - number[i] < number[j], OR
  - number[i] == number[j] AND i < j

Example:
P3 has number 5, P7 has number 5
(5, 3) < (5, 7) → P3 has priority
```

### Correctness Proof Sketch

```
┌──────────────────────────────────────────────────────────────────┐
│                BAKERY ALGORITHM CORRECTNESS                       │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  1. MUTUAL EXCLUSION:                                            │
│     If Pi is in CS, then for all Pj ≠ Pi:                       │
│     (number[i], i) < (number[j], j) or number[j] = 0            │
│     So Pj cannot be in CS simultaneously                        │
│     ✓ Guaranteed                                                 │
│                                                                  │
│  2. PROGRESS:                                                    │
│     Process with smallest (number, id) pair will enter          │
│     ✓ Guaranteed                                                 │
│                                                                  │
│  3. BOUNDED WAITING:                                             │
│     After Pi gets number[i], at most n-1 processes can          │
│     enter before Pi (those with smaller numbers or IDs)         │
│     ✓ Guaranteed                                                 │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

---

## Hardware Solutions

### Test-and-Set Lock (TSL)

```c
// Hardware atomic instruction
bool TestAndSet(bool *target) {
    bool old = *target;
    *target = true;
    return old;
}

// Usage
bool lock = false;

// Process Pi
do {
    while (TestAndSet(&lock))   // Spin while lock is held
        ;
    // CRITICAL SECTION
    lock = false;               // Release lock
    // REMAINDER SECTION
} while (true);
```

**Issue: No Bounded Waiting!**

```
Problem: A process might starve if others keep acquiring the lock

Fix: Add a queue or waiting array
```

### Bounded Waiting with Test-and-Set

```c
// Shared variables
bool lock = false;
bool waiting[n];  // waiting[i] = true means Pi is waiting

// Initialize all waiting[i] = false

// Process Pi
do {
    waiting[i] = true;
    bool key = true;
    
    while (waiting[i] && key)
        key = TestAndSet(&lock);
    
    waiting[i] = false;
    
    // CRITICAL SECTION
    
    // Find next waiting process
    int j = (i + 1) % n;
    while (j != i && !waiting[j])
        j = (j + 1) % n;
    
    if (j == i)
        lock = false;           // No one waiting, release lock
    else
        waiting[j] = false;     // Let Pj enter (pass the lock)
    
    // REMAINDER SECTION
    
} while (true);
```

**How This Ensures Bounded Waiting:**

```
┌──────────────────────────────────────────────────────────────────┐
│              BOUNDED WAITING MECHANISM                            │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  When Pi exits CS:                                               │
│  1. Scan from Pi+1 to find next waiting process                 │
│  2. If found (say Pj), set waiting[j] = false                   │
│  3. Pj's while loop exits (waiting[j] is false)                 │
│  4. Pj enters CS WITHOUT acquiring lock!                        │
│                                                                  │
│  Circular scan ensures fairness:                                 │
│  • At most n-1 processes can enter before Pi                    │
│  • FIFO-like ordering in circular manner                        │
│                                                                  │
│      Exit scan order for P0:                                     │
│      P1 → P2 → P3 → ... → Pn-1 → (back to P0)                  │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

---

## Compare-and-Swap Solution

```c
// Hardware atomic instruction
int CompareAndSwap(int *value, int expected, int new_value) {
    int temp = *value;
    if (*value == expected)
        *value = new_value;
    return temp;
}

// Usage
int lock = 0;

// Process Pi
do {
    while (CompareAndSwap(&lock, 0, 1) != 0)
        ;  // Spin until lock acquired
    // CRITICAL SECTION
    lock = 0;
    // REMAINDER SECTION
} while (true);
```

---

## Summary Comparison

| Solution | Processes | Mutual Exclusion | Progress | Bounded Waiting | Busy Wait |
|----------|-----------|------------------|----------|-----------------|-----------|
| **Turn variable only** | 2 | ✓ | ✗ | ✗ | ✓ |
| **Flags only** | 2 | ✗ | ✗ | ✗ | ✓ |
| **Dekker's** | 2 | ✓ | ✓ | ✓ | ✓ |
| **Peterson's** | 2 | ✓ | ✓ | ✓ | ✓ |
| **Bakery** | n | ✓ | ✓ | ✓ | ✓ |
| **Simple TSL** | n | ✓ | ✓ | ✗ | ✓ |
| **TSL + waiting[]** | n | ✓ | ✓ | ✓ | ✓ |

---

## Quick Revision Questions

1. Why does the simple turn-based solution fail?
2. What causes deadlock in the flags-only solution?
3. How does Dekker's algorithm resolve contention?
4. Explain the Bakery Algorithm's ticket-based approach.
5. How is bounded waiting achieved with Test-and-Set?

---

[← Previous: Synchronization Concepts](./01-synchronization-concepts.md) | [Next: Semaphores →](./03-semaphores.md)
