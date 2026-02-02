# Chapter 4.3: Banker's Algorithm

## Overview

The Banker's Algorithm is a deadlock avoidance algorithm developed by Edsger Dijkstra. It models a banker who allocates limited funds (resources) to customers (processes) while ensuring there's always enough to satisfy at least one customer completely.

---

## The Banker's Analogy

```
┌──────────────────────────────────────────────────────────────────┐
│                    THE BANKER'S ANALOGY                           │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Scenario: A bank with limited capital                          │
│                                                                  │
│  ┌─────────────────────────────────────────────────┐            │
│  │              BANK                                │            │
│  │                                                 │            │
│  │  Total Capital: $10,000                         │            │
│  │  Available: $2,000                              │            │
│  │                                                 │            │
│  │  Customers with credit limits:                  │            │
│  │  ┌──────────┬───────────┬───────────┐          │            │
│  │  │ Customer │ Max Limit │ Current   │          │            │
│  │  ├──────────┼───────────┼───────────┤          │            │
│  │  │    A     │  $6,000   │  $1,000   │          │            │
│  │  │    B     │  $5,000   │  $1,000   │          │            │
│  │  │    C     │  $4,000   │  $2,000   │          │            │
│  │  │    D     │  $7,000   │  $4,000   │          │            │
│  │  └──────────┴───────────┴───────────┘          │            │
│  │                                                 │            │
│  │  Allocated: $8,000 | Available: $2,000          │            │
│  └─────────────────────────────────────────────────┘            │
│                                                                  │
│  Rule: Never allocate if it might lead to a state where        │
│        the bank cannot satisfy at least one customer's          │
│        maximum need (and get money back)                        │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

---

## Data Structures

The algorithm uses the following data structures for n processes and m resource types:

```
┌──────────────────────────────────────────────────────────────────┐
│                     DATA STRUCTURES                               │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  n = number of processes                                         │
│  m = number of resource types                                    │
│                                                                  │
│  1. Available[m]                                                 │
│     Available[j] = number of available instances of Rj          │
│                                                                  │
│  2. Max[n][m]                                                    │
│     Max[i][j] = maximum demand of Pi for Rj                     │
│                                                                  │
│  3. Allocation[n][m]                                             │
│     Allocation[i][j] = current allocation of Rj to Pi           │
│                                                                  │
│  4. Need[n][m]                                                   │
│     Need[i][j] = additional need of Pi for Rj                   │
│     Need[i][j] = Max[i][j] - Allocation[i][j]                   │
│                                                                  │
│  Example:                                                        │
│  ┌────────┬───────────────┬─────────────────┬──────────────┐    │
│  │Process │   Max[i]      │  Allocation[i]  │   Need[i]    │    │
│  ├────────┼───────────────┼─────────────────┼──────────────┤    │
│  │   P0   │  (7, 5, 3)    │    (0, 1, 0)    │  (7, 4, 3)   │    │
│  │   P1   │  (3, 2, 2)    │    (2, 0, 0)    │  (1, 2, 2)   │    │
│  │   P2   │  (9, 0, 2)    │    (3, 0, 2)    │  (6, 0, 0)   │    │
│  │   P3   │  (2, 2, 2)    │    (2, 1, 1)    │  (0, 1, 1)   │    │
│  │   P4   │  (4, 3, 3)    │    (0, 0, 2)    │  (4, 3, 1)   │    │
│  └────────┴───────────────┴─────────────────┴──────────────┘    │
│                                                                  │
│  Available = (3, 3, 2)   [3 of R0, 3 of R1, 2 of R2]            │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

---

## Safety Algorithm

Determines if the system is in a safe state.

### Algorithm

```
┌──────────────────────────────────────────────────────────────────┐
│                    SAFETY ALGORITHM                               │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  1. Initialize:                                                  │
│     Work[m] = Available[m]     // Copy of available             │
│     Finish[n] = {false, false, ..., false}                      │
│                                                                  │
│  2. Find an index i such that:                                   │
│     Finish[i] == false  AND  Need[i] <= Work                    │
│                                                                  │
│     If no such i exists, go to step 4                           │
│                                                                  │
│  3. Execute:                                                     │
│     Work = Work + Allocation[i]   // Pi finishes, releases      │
│     Finish[i] = true                                            │
│     Go to step 2                                                │
│                                                                  │
│  4. Check result:                                                │
│     If Finish[i] == true for all i:                             │
│         System is in SAFE state                                 │
│     Else:                                                       │
│         System is in UNSAFE state                               │
│                                                                  │
│  Note: Need[i] <= Work means each element of Need[i]            │
│        is less than or equal to corresponding Work element      │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

### Example Walkthrough

```
Given:
Available = (3, 3, 2)

Process   Allocation    Max        Need
  P0      (0, 1, 0)   (7, 5, 3)  (7, 4, 3)
  P1      (2, 0, 0)   (3, 2, 2)  (1, 2, 2)
  P2      (3, 0, 2)   (9, 0, 2)  (6, 0, 0)
  P3      (2, 1, 1)   (2, 2, 2)  (0, 1, 1)
  P4      (0, 0, 2)   (4, 3, 3)  (4, 3, 1)

Step-by-step:

Work = (3, 3, 2), Finish = [F, F, F, F, F]

Iteration 1: Find process with Need <= Work
  P0: Need (7,4,3) <= (3,3,2)? NO (7>3)
  P1: Need (1,2,2) <= (3,3,2)? YES ✓
  
  Execute P1: 
    Work = (3,3,2) + (2,0,0) = (5, 3, 2)
    Finish = [F, T, F, F, F]

Iteration 2:
  P0: Need (7,4,3) <= (5,3,2)? NO
  P2: Need (6,0,0) <= (5,3,2)? NO (6>5)
  P3: Need (0,1,1) <= (5,3,2)? YES ✓
  
  Execute P3:
    Work = (5,3,2) + (2,1,1) = (7, 4, 3)
    Finish = [F, T, F, T, F]

Iteration 3:
  P0: Need (7,4,3) <= (7,4,3)? YES ✓
  
  Execute P0:
    Work = (7,4,3) + (0,1,0) = (7, 5, 3)
    Finish = [T, T, F, T, F]

Iteration 4:
  P2: Need (6,0,0) <= (7,5,3)? YES ✓
  
  Execute P2:
    Work = (7,5,3) + (3,0,2) = (10, 5, 5)
    Finish = [T, T, T, T, F]

Iteration 5:
  P4: Need (4,3,1) <= (10,5,5)? YES ✓
  
  Execute P4:
    Work = (10,5,5) + (0,0,2) = (10, 5, 7)
    Finish = [T, T, T, T, T]

All Finish[i] = true
Safe Sequence: <P1, P3, P0, P2, P4>
System is SAFE!
```

---

## Resource Request Algorithm

When process Pi requests resources.

### Algorithm

```
┌──────────────────────────────────────────────────────────────────┐
│                RESOURCE REQUEST ALGORITHM                         │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Request[i] = request vector for process Pi                     │
│                                                                  │
│  1. If Request[i] > Need[i]:                                     │
│     ERROR! Process exceeded maximum claim                       │
│     (Raise error and terminate)                                 │
│                                                                  │
│  2. If Request[i] > Available:                                   │
│     Process must WAIT (resources not available)                 │
│                                                                  │
│  3. Pretend to allocate (tentatively):                          │
│     Available = Available - Request[i]                          │
│     Allocation[i] = Allocation[i] + Request[i]                  │
│     Need[i] = Need[i] - Request[i]                              │
│                                                                  │
│  4. Run Safety Algorithm:                                        │
│     If SAFE:                                                    │
│         Grant the request (keep changes)                        │
│     If UNSAFE:                                                  │
│         Deny request (rollback changes)                         │
│         Process must wait                                       │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

### Example: Request Handling

```
Current State (from previous example):
Available = (3, 3, 2)
P1 requests (1, 0, 2)

Step 1: Check Request <= Need[P1]
  Request (1,0,2) <= Need (1,2,2)?
  (1<=1, 0<=2, 2<=2) → YES ✓

Step 2: Check Request <= Available
  Request (1,0,2) <= Available (3,3,2)?
  (1<=3, 0<=3, 2<=2) → YES ✓

Step 3: Tentative allocation
  Available = (3,3,2) - (1,0,2) = (2, 3, 0)
  Allocation[P1] = (2,0,0) + (1,0,2) = (3, 0, 2)
  Need[P1] = (1,2,2) - (1,0,2) = (0, 2, 0)

Step 4: Run safety algorithm with new state

New State:
Process   Allocation    Need
  P0      (0, 1, 0)   (7, 4, 3)
  P1      (3, 0, 2)   (0, 2, 0)  ← Updated
  P2      (3, 0, 2)   (6, 0, 0)
  P3      (2, 1, 1)   (0, 1, 1)
  P4      (0, 0, 2)   (4, 3, 1)

Available = (2, 3, 0)

Check: Can we find a safe sequence?
  P1: Need (0,2,0) <= (2,3,0)? YES
  Work after P1: (2,3,0) + (3,0,2) = (5,3,2)
  
  P3: Need (0,1,1) <= (5,3,2)? YES
  Work after P3: (5,3,2) + (2,1,1) = (7,4,3)
  
  ... (continue for all)

Safe sequence exists: <P1, P3, P0, P2, P4>
REQUEST GRANTED!
```

### Example: Request Denied

```
Same initial state, but P4 requests (3, 3, 0)

Step 1: Check Request <= Need[P4]
  Request (3,3,0) <= Need (4,3,1)? YES ✓

Step 2: Check Request <= Available
  Request (3,3,0) <= Available (3,3,2)? YES ✓

Step 3: Tentative allocation
  Available = (3,3,2) - (3,3,0) = (0, 0, 2)
  Allocation[P4] = (0,0,2) + (3,3,0) = (3, 3, 2)
  Need[P4] = (4,3,1) - (3,3,0) = (1, 0, 1)

Step 4: Safety check with Available = (0, 0, 2)

  P0: Need (7,4,3) <= (0,0,2)? NO
  P1: Need (1,2,2) <= (0,0,2)? NO
  P2: Need (6,0,0) <= (0,0,2)? NO
  P3: Need (0,1,1) <= (0,0,2)? NO
  P4: Need (1,0,1) <= (0,0,2)? NO

No process can proceed!
UNSAFE STATE → REQUEST DENIED
P4 must wait
```

---

## Banker's Algorithm Flowchart

```
┌─────────────────────────────────────────────────────────────────┐
│                  BANKER'S ALGORITHM FLOW                         │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│                 Process Pi requests Request[i]                   │
│                           │                                     │
│                           ▼                                     │
│              ┌─────────────────────────┐                        │
│              │ Request[i] <= Need[i]?  │                        │
│              └─────────────────────────┘                        │
│                    │            │                               │
│                   NO           YES                              │
│                    │            │                               │
│                    ▼            ▼                               │
│              ┌──────────┐  ┌─────────────────────────┐         │
│              │  ERROR!  │  │ Request[i] <= Available? │         │
│              │ Exceeded │  └─────────────────────────┘         │
│              │   Max    │       │            │                  │
│              └──────────┘      NO           YES                 │
│                                 │            │                  │
│                                 ▼            ▼                  │
│                           ┌─────────┐  ┌────────────────┐      │
│                           │  WAIT   │  │   Tentatively  │      │
│                           │(blocked)│  │    Allocate    │      │
│                           └─────────┘  └────────────────┘      │
│                                              │                  │
│                                              ▼                  │
│                                    ┌────────────────┐          │
│                                    │ Safety Check   │          │
│                                    │   (Safe?)      │          │
│                                    └────────────────┘          │
│                                       │          │              │
│                                      NO         YES             │
│                                       │          │              │
│                                       ▼          ▼              │
│                                ┌──────────┐  ┌─────────┐       │
│                                │ Rollback │  │  GRANT  │       │
│                                │  & WAIT  │  │ Request │       │
│                                └──────────┘  └─────────┘       │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## Time Complexity

| Algorithm | Complexity |
|-----------|------------|
| **Safety Algorithm** | O(m × n²) |
| **Resource Request** | O(m × n²) |

Where:
- n = number of processes
- m = number of resource types

---

## Limitations of Banker's Algorithm

```
┌──────────────────────────────────────────────────────────────────┐
│                    LIMITATIONS                                    │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  1. Fixed number of processes                                    │
│     • Must know all processes in advance                        │
│     • Doesn't handle dynamic process creation well              │
│                                                                  │
│  2. Fixed number of resources                                    │
│     • Resources can't be added or removed                       │
│     • No device failures allowed                                │
│                                                                  │
│  3. Maximum needs must be known                                  │
│     • Processes must declare max before starting               │
│     • Often difficult to determine                              │
│                                                                  │
│  4. Processes must return resources                              │
│     • Assumes processes eventually terminate                    │
│     • No infinite loops holding resources                       │
│                                                                  │
│  5. Overhead                                                     │
│     • O(m × n²) for each request                               │
│     • May be too slow for real-time systems                    │
│                                                                  │
│  6. Conservative                                                 │
│     • May deny requests that wouldn't cause deadlock           │
│     • Reduces resource utilization                              │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

---

## Practice Problem

```
Given:
5 processes (P0-P4), 3 resource types (A, B, C)
Total resources: A=10, B=5, C=7

Current State:
          Allocation      Max         Need
Process   A   B   C    A   B   C    A   B   C
  P0      0   1   0    7   5   3    7   4   3
  P1      2   0   0    3   2   2    1   2   2
  P2      3   0   2    9   0   2    6   0   0
  P3      2   1   1    2   2   2    0   1   1
  P4      0   0   2    4   3   3    4   3   1

Available: A=3, B=3, C=2

Questions:
1. Is the system in a safe state?
2. If P1 requests (1, 0, 2), can it be granted?
3. If P4 requests (3, 3, 0), can it be granted?

Answers:
1. Yes - Safe sequence: <P1, P3, P4, P0, P2> or <P1, P3, P0, P2, P4>
2. Yes - New Available (2,3,0) still allows safe sequence
3. No - Would leave Available (0,0,2), no process can finish
```

---

## Summary

| Concept | Description |
|---------|-------------|
| **Purpose** | Avoid deadlock by checking safety before allocation |
| **Safe State** | A sequence exists where all can complete |
| **Safety Algorithm** | Finds if safe sequence exists |
| **Request Algorithm** | Tests if granting request keeps system safe |
| **Key Data** | Available, Max, Allocation, Need |
| **Complexity** | O(m × n²) per request |

---

## Quick Revision Questions

1. What are the four data structures used in Banker's Algorithm?
2. How is Need[i][j] calculated?
3. Explain the Safety Algorithm steps.
4. What happens if Request[i] > Need[i]?
5. Why might Banker's Algorithm deny a safe request?

---

[← Previous: Deadlock Handling](./02-deadlock-handling.md) | [Next: Deadlock Detection →](./04-deadlock-detection.md)
