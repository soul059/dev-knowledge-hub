# Chapter 4.2: Deadlock Handling Methods

## Overview

There are four general approaches to handling deadlocks. The choice depends on the system requirements, overhead considerations, and how critical deadlock prevention is.

---

## Deadlock Handling Strategies

```
┌──────────────────────────────────────────────────────────────────┐
│               DEADLOCK HANDLING APPROACHES                        │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  1. PREVENTION     - Ensure at least one necessary condition    │
│                      can never hold                              │
│                                                                  │
│  2. AVOIDANCE      - Dynamically check if allocation is safe    │
│                      before granting resources                   │
│                                                                  │
│  3. DETECTION &    - Allow deadlocks, detect them, then recover │
│     RECOVERY                                                     │
│                                                                  │
│  4. IGNORANCE      - Pretend deadlocks never occur              │
│     (Ostrich)        (Used by most OSes!)                       │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

---

## 1. Deadlock Prevention

Prevent deadlock by ensuring at least one of the four necessary conditions cannot hold.

### Preventing Mutual Exclusion

```
┌──────────────────────────────────────────────────────────────────┐
│              PREVENT MUTUAL EXCLUSION                             │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Idea: Make resources shareable                                  │
│                                                                  │
│  Problem: Not possible for many resources!                      │
│                                                                  │
│  ✗ Printer - Cannot share (output would be garbled)             │
│  ✗ Mutex lock - By definition, must be exclusive                │
│  ✗ Tape drive - Sequential access required                      │
│                                                                  │
│  ✓ Read-only files - Can be shared                              │
│  ✓ Spooling - Print jobs go to queue (virtual resource)         │
│                                                                  │
│  Spooling Example:                                               │
│  ┌──────────┐     ┌────────────┐     ┌─────────┐               │
│  │ Process  │ ──→ │ Spool Queue │ ──→ │ Printer │               │
│  └──────────┘     └────────────┘     └─────────┘               │
│                   (Virtual)           (Physical)                 │
│  Processes don't hold printer, they write to queue              │
│                                                                  │
│  Limitation: Cannot prevent for all resources                   │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

### Preventing Hold and Wait

```
┌──────────────────────────────────────────────────────────────────┐
│                PREVENT HOLD AND WAIT                              │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Two Protocols:                                                  │
│                                                                  │
│  Protocol 1: Request ALL resources at once                      │
│  ─────────────────────────────────────────                      │
│  • Process requests all needed resources before starting        │
│  • Either gets all or waits with none                           │
│                                                                  │
│    Process: "I need R1, R2, R3, R4"                             │
│    System:  "Here, take all" OR "Wait, hold nothing"           │
│                                                                  │
│  Protocol 2: Request only when holding none                     │
│  ───────────────────────────────────────────                    │
│  • Release all held resources before requesting new ones        │
│                                                                  │
│    Phase 1: Request R1, R2 → Use → Release R1, R2              │
│    Phase 2: Request R3, R4 → Use → Release R3, R4              │
│                                                                  │
│  Disadvantages:                                                  │
│  ┌──────────────────────────────────────────────────┐           │
│  │ • Low resource utilization (hold but don't use)  │           │
│  │ • Starvation possible (many resources needed)    │           │
│  │ • Must know all requirements in advance          │           │
│  └──────────────────────────────────────────────────┘           │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

### Preventing No Preemption

```
┌──────────────────────────────────────────────────────────────────┐
│                 PREVENT NO PREEMPTION                             │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Idea: Allow resource preemption                                 │
│                                                                  │
│  Protocol:                                                       │
│  If process Pi requests resource R that is unavailable:         │
│                                                                  │
│  Option 1: Pi releases all its held resources                   │
│            Pi waits for all resources (old + new)               │
│            Restart when all available                            │
│                                                                  │
│  Option 2: If R is held by Pj who is also waiting:             │
│            Preempt R from Pj and give to Pi                     │
│            Pj must wait for R                                   │
│                                                                  │
│  Applicable To:                                                  │
│  ┌────────────────────────────────────────────────┐             │
│  │ ✓ CPU (context switch)                         │             │
│  │ ✓ Memory (page swapping)                       │             │
│  │ ✓ Registers (can save and restore)             │             │
│  └────────────────────────────────────────────────┘             │
│                                                                  │
│  NOT Applicable To:                                              │
│  ┌────────────────────────────────────────────────┐             │
│  │ ✗ Printers (can't preempt mid-print)           │             │
│  │ ✗ Tape drives (sequential access)              │             │
│  │ ✗ Mutex locks (would corrupt data)             │             │
│  └────────────────────────────────────────────────┘             │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

### Preventing Circular Wait

```
┌──────────────────────────────────────────────────────────────────┐
│                PREVENT CIRCULAR WAIT                              │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Idea: Impose a total ordering on resource types                │
│                                                                  │
│  Assign each resource type Ri a unique number F(Ri)             │
│                                                                  │
│  Rule: Process can only request resources in INCREASING order   │
│                                                                  │
│  Example Ordering:                                               │
│  ┌──────────────────────────────────────────────────┐           │
│  │  F(tape_drive) = 1                               │           │
│  │  F(disk_drive) = 5                               │           │
│  │  F(printer)    = 12                              │           │
│  └──────────────────────────────────────────────────┘           │
│                                                                  │
│  Valid:   Request tape(1), then disk(5), then printer(12)      │
│  Invalid: Request printer(12), then disk(5) ✗                  │
│                                                                  │
│  To request Rj while holding Ri:                                │
│  Must have F(Ri) < F(Rj)                                        │
│                                                                  │
│  Why This Works:                                                 │
│  ┌──────────────────────────────────────────────────┐           │
│  │ For circular wait: P1→R1→P2→R2→...→Pn→Rn→P1     │           │
│  │                                                   │           │
│  │ This implies:                                     │           │
│  │ F(R1) < F(R2) < ... < F(Rn) < F(R1)              │           │
│  │                                                   │           │
│  │ But F(R1) < F(R1) is impossible!                 │           │
│  │ Contradiction → No circular wait possible        │           │
│  └──────────────────────────────────────────────────┘           │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

### Prevention Summary

| Condition | Prevention Method | Disadvantage |
|-----------|------------------|--------------|
| **Mutual Exclusion** | Use shareable resources | Not always possible |
| **Hold and Wait** | Request all at once | Low utilization, starvation |
| **No Preemption** | Preempt resources | Not practical for all resources |
| **Circular Wait** | Ordered resource requests | Limits programming flexibility |

---

## 2. Deadlock Avoidance

### Concept

```
┌──────────────────────────────────────────────────────────────────┐
│                  DEADLOCK AVOIDANCE                               │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Idea: Don't grant request if it MIGHT lead to deadlock        │
│                                                                  │
│  Requirement: Know maximum resource needs in advance            │
│                                                                  │
│  Process makes requests → System checks if granting is SAFE     │
│                                                                  │
│         Request                    Check                         │
│  Process ─────→ Resource Manager ────────→ Safe?                │
│                       │                       │                  │
│                       ↓                       ↓                  │
│                    Grant               Yes: Grant                │
│                      ↓                  No: Wait                 │
│                    Use                                           │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

### Safe State

```
┌──────────────────────────────────────────────────────────────────┐
│                      SAFE STATE                                   │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  A state is SAFE if:                                             │
│  There exists a sequence <P1, P2, ..., Pn> such that            │
│  each Pi can get resources needed to complete                   │
│                                                                  │
│  Safe sequence: Each Pi can be satisfied by:                    │
│  • Currently available resources, PLUS                          │
│  • Resources held by all Pj where j < i                         │
│    (because they will complete and release)                     │
│                                                                  │
│  Example:                                                        │
│  Total resources = 12 units                                      │
│                                                                  │
│  Process   Max Need   Current Hold   May Need                   │
│  P1           10          5            5                        │
│  P2            4          2            2                        │
│  P3            9          2            7                        │
│                                                                  │
│  Available = 12 - (5+2+2) = 3                                   │
│                                                                  │
│  Safe sequence: <P2, P1, P3>                                    │
│  1. P2 needs 2, available 3 ✓ → P2 finishes, releases 2        │
│  2. Available = 5, P1 needs 5 ✓ → P1 finishes, releases 10     │
│  3. Available = 10, P3 needs 7 ✓ → P3 finishes                 │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

### Safe vs Unsafe States

```
                    All States
          ┌─────────────────────────────┐
          │                             │
          │    ┌─────────────────┐      │
          │    │   SAFE STATES   │      │
          │    │                 │      │
          │    │   (No deadlock  │      │
          │    │    possible)    │      │
          │    │                 │      │
          │    └─────────────────┘      │
          │                             │
          │    ┌─────────────────┐      │
          │    │  UNSAFE STATES  │      │
          │    │ ┌─────────────┐ │      │
          │    │ │  DEADLOCK   │ │      │
          │    │ │   STATES    │ │      │
          │    │ └─────────────┘ │      │
          │    └─────────────────┘      │
          │                             │
          └─────────────────────────────┘

Key Points:
• Safe state: Definitely can avoid deadlock
• Unsafe state: Deadlock MAY occur (not guaranteed)
• Deadlock: Subset of unsafe states
• Avoidance: Always stay in safe states
```

---

## 3. Detection and Recovery

Allow deadlock to occur, then detect and recover.

### Detection Approaches

```
┌──────────────────────────────────────────────────────────────────┐
│                  DEADLOCK DETECTION                               │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  When to check for deadlock?                                     │
│                                                                  │
│  Option 1: Check every resource request                         │
│  • Immediate detection                                          │
│  • High overhead                                                 │
│                                                                  │
│  Option 2: Check periodically                                    │
│  • Lower overhead                                               │
│  • Delayed detection                                            │
│                                                                  │
│  Option 3: Check when utilization drops                         │
│  • Triggered by symptom                                         │
│  • May miss some deadlocks                                      │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

### Recovery Methods

#### 1. Process Termination

```
┌──────────────────────────────────────────────────────────────────┐
│                PROCESS TERMINATION                                │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Option A: Abort ALL deadlocked processes                        │
│  • Simple to implement                                          │
│  • High cost - lose all work                                    │
│                                                                  │
│  Option B: Abort ONE at a time until deadlock broken            │
│  • Less drastic                                                 │
│  • Must rerun detection after each                              │
│                                                                  │
│  Selection Criteria (who to terminate?):                        │
│  ┌──────────────────────────────────────────────────┐           │
│  │ • Process priority (lower priority first)         │           │
│  │ • How long process has run (less time = abort)   │           │
│  │ • Resources held (fewer = easier to abort)       │           │
│  │ • Resources needed to complete                   │           │
│  │ • How many processes to terminate                │           │
│  │ • Interactive vs batch (batch first)             │           │
│  └──────────────────────────────────────────────────┘           │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

#### 2. Resource Preemption

```
┌──────────────────────────────────────────────────────────────────┐
│                RESOURCE PREEMPTION                                │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Take resources from some processes and give to others          │
│                                                                  │
│  Issues to Address:                                              │
│                                                                  │
│  1. Victim Selection                                             │
│     Which process to preempt from?                              │
│     (Minimize cost/disruption)                                  │
│                                                                  │
│  2. Rollback                                                     │
│     What to do with preempted process?                          │
│     • Total rollback: Restart from beginning                    │
│     • Partial rollback: Roll back to safe checkpoint            │
│                                                                  │
│  3. Starvation                                                   │
│     Same process may be chosen repeatedly!                      │
│     Solution: Include rollback count in selection               │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

---

## 4. Ostrich Algorithm (Ignore)

```
┌──────────────────────────────────────────────────────────────────┐
│                  OSTRICH ALGORITHM                                │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  "Stick head in sand and pretend there is no problem"           │
│                                                                  │
│  Approach: Ignore deadlock entirely                              │
│                                                                  │
│  Rationale:                                                      │
│  • Deadlocks occur rarely                                       │
│  • Prevention/Avoidance overhead may not be worth it            │
│  • Manual intervention (reboot) is acceptable                   │
│                                                                  │
│  Used By:                                                        │
│  • Windows                                                       │
│  • Linux                                                        │
│  • Most general-purpose operating systems                       │
│                                                                  │
│  Trade-off:                                                      │
│  ┌────────────────────────────────────────────────┐             │
│  │ Convenience vs Correctness                     │             │
│  │                                                │             │
│  │ If deadlock happens once per year and takes   │             │
│  │ a 5-minute reboot to fix...                   │             │
│  │                                                │             │
│  │ Is it worth 10% performance overhead to       │             │
│  │ prevent/avoid deadlock?                       │             │
│  │                                                │             │
│  │ For most systems: NO                          │             │
│  │ For safety-critical systems: YES              │             │
│  └────────────────────────────────────────────────┘             │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

---

## Comparison of Methods

| Method | Overhead | Resource Use | User Convenience |
|--------|----------|--------------|------------------|
| **Prevention** | Low-Medium | Low | May restrict programming |
| **Avoidance** | High | Medium | Must declare max needs |
| **Detection** | Medium | High | May lose work |
| **Ignorance** | None | Highest | May need reboot |

---

## Summary

| Approach | When Used | Trade-off |
|----------|-----------|-----------|
| **Prevention** | When overhead acceptable | Restricts resource use |
| **Avoidance** | When max needs known | Computational overhead |
| **Detection** | When occasional deadlock OK | Recovery cost |
| **Ignorance** | General-purpose systems | Manual intervention needed |

---

## Quick Revision Questions

1. List four methods to handle deadlocks.
2. How can circular wait be prevented?
3. What is a safe state?
4. What are the two approaches to process termination for recovery?
5. Why do most operating systems use the ostrich algorithm?

---

[← Previous: Deadlock Introduction](./01-deadlock-introduction.md) | [Next: Banker's Algorithm →](./03-bankers-algorithm.md)
