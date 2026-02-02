# Chapter 4.4: Deadlock Detection

## Overview

If a system doesn't use prevention or avoidance, deadlocks may occur. Detection algorithms periodically check for deadlocks, and recovery mechanisms break them when found.

---

## Detection Approaches

```
┌──────────────────────────────────────────────────────────────────┐
│              DEADLOCK DETECTION STRATEGIES                        │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Two cases based on resource instances:                         │
│                                                                  │
│  1. Single Instance of Each Resource Type                       │
│     → Use Wait-For Graph                                        │
│     → Cycle detection = Deadlock                                │
│                                                                  │
│  2. Multiple Instances of Each Resource Type                    │
│     → Use Detection Algorithm (similar to Banker's)             │
│     → More complex analysis required                            │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

---

## Single Instance: Wait-For Graph

### Concept

```
┌──────────────────────────────────────────────────────────────────┐
│                   WAIT-FOR GRAPH                                  │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Derived from Resource Allocation Graph (RAG)                   │
│  • Remove resource nodes                                        │
│  • If Pi → Rq → Pj in RAG, then Pi → Pj in Wait-For Graph      │
│                                                                  │
│  Resource Allocation Graph:          Wait-For Graph:            │
│                                                                  │
│      P1 ──→ R1 ──→ P2                  P1 ──→ P2                │
│       ↑            │                    ↑      │                │
│       │            ↓                    │      ↓                │
│      R2 ←── P3 ←── R3                  P3 ←───┘                │
│                                                                  │
│  Pi → Pj means "Pi is waiting for resource held by Pj"         │
│                                                                  │
│  DEADLOCK exists ⟺ Cycle exists in Wait-For Graph              │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

### Construction Steps

```
Step 1: Start with Resource Allocation Graph

    ○ P1                    ┌────┐
     │                      │ R1 │
     │ request              └────┘
     ↓                         │
   ┌────┐                     │ held by
   │ R2 │                     ↓
   └────┘                   ○ P2
     │                       │
     │ held by               │ request
     ↓                       ↓
    ○ P3 ←── held ─────── ┌────┐
                          │ R3 │
                          └────┘

Step 2: For each request edge Pi → Rj → Pk (where Pk holds Rj)
        Add edge Pi → Pk

Step 3: Remove all resource nodes

Wait-For Graph:
    ○ P1
     │
     │ waits for
     ↓
    ○ P3 ←─────────────○ P2
                waits for

No cycle → No deadlock in this example
```

### Cycle Detection Algorithm

```c
// DFS-based cycle detection
bool visited[n], recStack[n];

bool detectCycle(int v, int waitFor[][]) {
    visited[v] = true;
    recStack[v] = true;
    
    for each process u where v waits for u {
        if (!visited[u] && detectCycle(u, waitFor))
            return true;
        else if (recStack[u])
            return true;  // Cycle found!
    }
    
    recStack[v] = false;
    return false;
}

bool hasDeadlock() {
    initialize visited[] = false, recStack[] = false;
    
    for each process v {
        if (!visited[v] && detectCycle(v, waitFor))
            return true;
    }
    return false;
}
```

### Time Complexity

| Operation | Complexity |
|-----------|------------|
| Graph maintenance | O(n²) space |
| Cycle detection (DFS) | O(n + e) where e = edges |

---

## Multiple Instances: Detection Algorithm

### Data Structures

```
┌──────────────────────────────────────────────────────────────────┐
│                     DATA STRUCTURES                               │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  n = number of processes                                         │
│  m = number of resource types                                    │
│                                                                  │
│  Available[m] : Available instances of each resource type       │
│                                                                  │
│  Allocation[n][m] : Current allocation matrix                   │
│                                                                  │
│  Request[n][m] : Current request matrix                         │
│                  Request[i][j] = current request by Pi for Rj   │
│                                                                  │
│  Note: Unlike Banker's, we use Request (current) not Max        │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

### Detection Algorithm

```
┌──────────────────────────────────────────────────────────────────┐
│                  DETECTION ALGORITHM                              │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  1. Initialize:                                                  │
│     Work[m] = Available[m]                                      │
│     For i = 0 to n-1:                                           │
│         if Allocation[i] != 0:                                  │
│             Finish[i] = false                                   │
│         else:                                                   │
│             Finish[i] = true  // Process holding nothing        │
│                                                                  │
│  2. Find index i such that:                                      │
│     Finish[i] == false AND Request[i] <= Work                   │
│                                                                  │
│     If no such i exists, go to step 4                           │
│                                                                  │
│  3. Execute:                                                     │
│     Work = Work + Allocation[i]                                 │
│     Finish[i] = true                                            │
│     Go to step 2                                                │
│                                                                  │
│  4. If any Finish[i] == false:                                   │
│     Process Pi is DEADLOCKED                                    │
│     All processes with Finish[i] = false are in deadlock        │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

### Example

```
5 processes (P0-P4), 3 resource types (A, B, C)
Available = (0, 0, 0)

          Allocation      Request
Process   A   B   C     A   B   C
  P0      0   1   0     0   0   0
  P1      2   0   0     2   0   2
  P2      3   0   3     0   0   0
  P3      2   1   1     1   0   0
  P4      0   0   2     0   0   2

Step 1: Initialize
  Work = (0, 0, 0)
  Finish = [false, false, false, false, false]
  (All have allocations, so all start as false)

Step 2-3: Find processes that can complete

Iteration 1:
  P0: Request (0,0,0) <= Work (0,0,0)? YES ✓
  Work = (0,0,0) + (0,1,0) = (0, 1, 0)
  Finish = [true, false, false, false, false]

Iteration 2:
  P1: Request (2,0,2) <= Work (0,1,0)? NO
  P2: Request (0,0,0) <= Work (0,1,0)? YES ✓
  Work = (0,1,0) + (3,0,3) = (3, 1, 3)
  Finish = [true, false, true, false, false]

Iteration 3:
  P1: Request (2,0,2) <= Work (3,1,3)? YES ✓
  Work = (3,1,3) + (2,0,0) = (5, 1, 3)
  Finish = [true, true, true, false, false]

Iteration 4:
  P3: Request (1,0,0) <= Work (5,1,3)? YES ✓
  Work = (5,1,3) + (2,1,1) = (7, 2, 4)
  Finish = [true, true, true, true, false]

Iteration 5:
  P4: Request (0,0,2) <= Work (7,2,4)? YES ✓
  Work = (7,2,4) + (0,0,2) = (7, 2, 6)
  Finish = [true, true, true, true, true]

All Finish[i] = true → NO DEADLOCK
```

### Example with Deadlock

```
Change P2's Request to (0, 0, 1):

          Allocation      Request
Process   A   B   C     A   B   C
  P0      0   1   0     0   0   0
  P1      2   0   0     2   0   2
  P2      3   0   3     0   0   1  ← Changed
  P3      2   1   1     1   0   0
  P4      0   0   2     0   0   2

Available = (0, 0, 0)

Iteration 1:
  P0: Request (0,0,0) <= (0,0,0)? YES
  Work = (0, 1, 0), Finish = [T, F, F, F, F]

Iteration 2:
  P1: Request (2,0,2) <= (0,1,0)? NO (2>0, 2>0)
  P2: Request (0,0,1) <= (0,1,0)? NO (1>0)
  P3: Request (1,0,0) <= (0,1,0)? NO (1>0)
  P4: Request (0,0,2) <= (0,1,0)? NO (2>0)

No process can proceed!

Finish = [true, false, false, false, false]

DEADLOCK detected!
Deadlocked processes: P1, P2, P3, P4
```

---

## When to Run Detection?

```
┌──────────────────────────────────────────────────────────────────┐
│              DETECTION FREQUENCY                                  │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Option 1: Every resource request                                │
│  ─────────────────────────────────                              │
│  Pros: Immediate detection                                      │
│  Cons: High overhead O(m × n²) per request                     │
│                                                                  │
│  Option 2: Periodic intervals                                    │
│  ─────────────────────────────                                  │
│  Example: Every 5 minutes, or every 100 requests               │
│  Pros: Lower overhead                                           │
│  Cons: Delayed detection                                        │
│                                                                  │
│  Option 3: When resource utilization drops                       │
│  ──────────────────────────────────────────                     │
│  Trigger: CPU utilization < threshold                          │
│  Rationale: Deadlock causes processes to wait → low CPU use    │
│  Pros: Adaptive, detects when symptoms appear                  │
│                                                                  │
│  Trade-off:                                                      │
│  ┌──────────────────────────────────────────┐                   │
│  │ Frequent checks = More overhead          │                   │
│  │                 = Faster detection       │                   │
│  │                                          │                   │
│  │ Infrequent checks = Less overhead        │                   │
│  │                   = Slower detection     │                   │
│  │                   = Harder to identify   │                   │
│  │                     which process caused │                   │
│  │                     deadlock             │                   │
│  └──────────────────────────────────────────┘                   │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

---

## Recovery from Deadlock

Once deadlock is detected, the system must recover.

### Method 1: Process Termination

```
┌──────────────────────────────────────────────────────────────────┐
│                PROCESS TERMINATION                                │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Option A: Abort ALL deadlocked processes                        │
│  ─────────────────────────────────────────                      │
│  • Breaks deadlock immediately                                  │
│  • Expensive: All work lost                                     │
│                                                                  │
│  Option B: Abort ONE at a time                                   │
│  ────────────────────────────────                               │
│  • Abort one process                                            │
│  • Re-run detection algorithm                                   │
│  • Repeat until deadlock broken                                 │
│  • More selective, but more overhead                            │
│                                                                  │
│  Selection Criteria (minimize cost):                            │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │ 1. Process priority (lower = terminate first)           │    │
│  │ 2. Time executed (less time = terminate)                │    │
│  │ 3. Resources held (fewer = easier to terminate)         │    │
│  │ 4. Resources needed to complete                         │    │
│  │ 5. Interactive vs batch (batch first)                   │    │
│  │ 6. Number of processes to terminate                     │    │
│  └─────────────────────────────────────────────────────────┘    │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

### Method 2: Resource Preemption

```
┌──────────────────────────────────────────────────────────────────┐
│                RESOURCE PREEMPTION                                │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Forcibly take resources from some processes                    │
│                                                                  │
│  Three Issues:                                                   │
│                                                                  │
│  1. SELECTING A VICTIM                                           │
│     ─────────────────────                                       │
│     Which process to preempt from?                              │
│     • Minimize total cost                                       │
│     • Consider: resources held, time executed, priority         │
│                                                                  │
│  2. ROLLBACK                                                     │
│     ────────                                                    │
│     What happens to preempted process?                          │
│     • Total rollback: Restart from beginning                    │
│     • Partial rollback: Return to safe checkpoint               │
│     • Requires checkpoint mechanism                             │
│                                                                  │
│  3. STARVATION                                                   │
│     ──────────                                                  │
│     Same process may be repeatedly chosen as victim             │
│     Solution: Include rollback count in cost factor             │
│     "Process that has been rolled back 3 times gets priority"   │
│                                                                  │
│  Example Rollback:                                               │
│  ┌──────────────────────────────────────────┐                   │
│  │ Process P1: [Start] → [Checkpoint1] →    │                   │
│  │             [Checkpoint2] → [PREEMPTED]  │                   │
│  │                                          │                   │
│  │ After preemption, restart from           │                   │
│  │ Checkpoint2 (partial) or Start (total)   │                   │
│  └──────────────────────────────────────────┘                   │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

---

## Detection vs Avoidance Comparison

| Aspect | Avoidance (Banker's) | Detection |
|--------|---------------------|-----------|
| **When checked** | Before each allocation | Periodically |
| **Information needed** | Max needs in advance | Current requests only |
| **Overhead** | Every request | Periodic only |
| **Resource utilization** | Lower (conservative) | Higher |
| **Deadlock occurrence** | Never | May occur |
| **Recovery needed?** | No | Yes |

---

## Combined Example

```
┌──────────────────────────────────────────────────────────────────┐
│              DETECTION AND RECOVERY EXAMPLE                       │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  System State:                                                   │
│  Processes: P1, P2, P3, P4                                      │
│  Resources: 2 Printers (R1), 1 Scanner (R2)                     │
│                                                                  │
│  Wait-For Graph:                                                 │
│       P1 ──→ P2                                                 │
│       ↑      │                                                  │
│       │      ↓                                                  │
│       P4 ←── P3                                                 │
│                                                                  │
│  Cycle: P1 → P2 → P3 → P4 → P1                                 │
│  DEADLOCK DETECTED!                                             │
│                                                                  │
│  Recovery Decision:                                              │
│  ┌────────────────────────────────────────────────────────┐     │
│  │ Process │ Priority │ Time Run │ Resources │ Decision  │     │
│  ├────────────────────────────────────────────────────────┤     │
│  │   P1    │    3     │  10 min  │     1     │   Keep    │     │
│  │   P2    │    2     │   2 min  │     1     │ TERMINATE │     │
│  │   P3    │    1     │  30 min  │     2     │   Keep    │     │
│  │   P4    │    2     │   5 min  │     1     │   Keep    │     │
│  └────────────────────────────────────────────────────────┘     │
│                                                                  │
│  P2 terminated (low priority, short run time)                   │
│  Resources released → Others can proceed                        │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

---

## Summary

| Concept | Description |
|---------|-------------|
| **Wait-For Graph** | Used for single-instance resources |
| **Detection Algorithm** | Used for multiple instances |
| **Cycle = Deadlock** | Only for single instance |
| **Finish[i] = false** | Process i is deadlocked |
| **Recovery** | Termination or preemption |
| **Victim Selection** | Based on cost minimization |

---

## Quick Revision Questions

1. How is a Wait-For Graph constructed from a RAG?
2. What does it mean if Finish[i] = false after detection?
3. List three factors in selecting which process to terminate.
4. What is the starvation problem in resource preemption?
5. When should detection algorithm be run?

---

[← Previous: Banker's Algorithm](./03-bankers-algorithm.md) | [Next: Memory Management Introduction →](../05-Memory-Management/01-memory-introduction.md)
