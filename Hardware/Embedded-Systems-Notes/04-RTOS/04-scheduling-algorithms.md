# Chapter 4.4: Scheduling Algorithms

## Introduction

The scheduler is the heart of an RTOS, responsible for deciding which task runs at any given moment. The choice of scheduling algorithm directly impacts system responsiveness, determinism, and CPU utilization.

---

## Scheduling Objectives

```
┌─────────────────────────────────────────────────────────────────────┐
│                    SCHEDULING GOALS                                  │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │  1. MEET DEADLINES                                          │    │
│  │     Primary goal for real-time systems                      │    │
│  │     All hard deadlines must be guaranteed                   │    │
│  ├─────────────────────────────────────────────────────────────┤    │
│  │  2. MAXIMIZE CPU UTILIZATION                                │    │
│  │     Keep CPU busy (minimize idle time)                      │    │
│  │     But leave headroom for bursts                           │    │
│  ├─────────────────────────────────────────────────────────────┤    │
│  │  3. MINIMIZE RESPONSE TIME                                  │    │
│  │     High-priority tasks respond quickly                     │    │
│  │     Bounded worst-case latency                              │    │
│  ├─────────────────────────────────────────────────────────────┤    │
│  │  4. FAIRNESS                                                │    │
│  │     Equal-priority tasks get fair share                     │    │
│  │     Prevent starvation                                      │    │
│  ├─────────────────────────────────────────────────────────────┤    │
│  │  5. PREDICTABILITY                                          │    │
│  │     Deterministic behavior                                  │    │
│  │     Analyzable worst-case                                   │    │
│  └─────────────────────────────────────────────────────────────┘    │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## Priority-Based Preemptive Scheduling

The most common RTOS scheduling approach.

```
┌─────────────────────────────────────────────────────────────────────┐
│                    PRIORITY-BASED PREEMPTION                         │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  RULE: Highest-priority READY task always runs                      │
│  PREEMPTION: Higher-priority task immediately takes over            │
│                                                                      │
│  EXAMPLE: 3 tasks with priorities High(H), Medium(M), Low(L)        │
│                                                                      │
│  Task H (Pri 0): ████                    ████                       │
│  Task M (Pri 1):     ████████████████████    ████████               │
│  Task L (Pri 2):                                     ████████       │
│  Idle:          ████                                         ████   │
│                                                                      │
│       t0  t1  t2  t3  t4  t5  t6  t7  t8  t9  t10 t11 t12 t13 t14   │
│       ──────────────────────────────────────────────────────────▶   │
│                                                                      │
│  Timeline:                                                           │
│  t0:  M becomes ready, starts running                               │
│  t1:  H becomes ready, PREEMPTS M                                   │
│  t2:  H completes, M resumes                                        │
│  t6:  M blocks (waiting for event)                                  │
│  t6:  L starts (was waiting)                                        │
│  t7:  H becomes ready, PREEMPTS L                                   │
│  t8:  H completes, M is still blocked, L continues                  │
│  t9:  Event occurs, M becomes ready, PREEMPTS L                     │
│  t11: M completes, L resumes                                        │
│  t13: L completes, Idle runs                                        │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

### Preemption Implementation

```c
void scheduler(void) {
    // Find highest priority ready task
    TCB_t *next = get_highest_priority_ready_task();
    
    if (next != current_task) {
        // Preemption needed
        if (current_task->state == RUNNING) {
            current_task->state = READY;
            add_to_ready_queue(current_task);
        }
        
        // Switch to new task
        current_task = next;
        current_task->state = RUNNING;
        context_switch(next);
    }
}
```

---

## Round-Robin Scheduling

Used for tasks of equal priority.

```
┌─────────────────────────────────────────────────────────────────────┐
│                    ROUND-ROBIN (TIME-SLICING)                        │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  When multiple tasks have SAME priority:                            │
│  • Each gets a fixed TIME SLICE (quantum)                           │
│  • Tasks rotate in circular order                                   │
│  • Provides fairness among equal-priority tasks                     │
│                                                                      │
│  EXAMPLE: Tasks A, B, C all at priority 2, time slice = 4 ticks    │
│                                                                      │
│        ┌───┐   ┌───┐   ┌───┐   ┌───┐   ┌───┐   ┌───┐               │
│  A:    │▓▓▓│   │   │   │   │   │▓▓▓│   │   │   │   │               │
│        └───┘   └───┘   └───┘   └───┘   └───┘   └───┘               │
│        ┌───┐   ┌───┐   ┌───┐   ┌───┐   ┌───┐   ┌───┐               │
│  B:    │   │   │▓▓▓│   │   │   │   │   │▓▓▓│   │   │               │
│        └───┘   └───┘   └───┘   └───┘   └───┘   └───┘               │
│        ┌───┐   ┌───┐   ┌───┐   ┌───┐   ┌───┐   ┌───┐               │
│  C:    │   │   │   │   │▓▓▓│   │   │   │   │   │▓▓▓│               │
│        └───┘   └───┘   └───┘   └───┘   └───┘   └───┘               │
│                                                                      │
│       ──▶ Time                                                       │
│        0   4   8   12  16  20  24 ticks                             │
│                                                                      │
│  TIME SLICE SELECTION:                                               │
│  ┌─────────────┬─────────────────────────────────────────────────┐  │
│  │ Slice Size  │ Trade-off                                       │  │
│  ├─────────────┼─────────────────────────────────────────────────┤  │
│  │ Too small   │ High context switch overhead                    │  │
│  │ (1-2 ticks) │ Responsive but inefficient                      │  │
│  ├─────────────┼─────────────────────────────────────────────────┤  │
│  │ Too large   │ Poor responsiveness                             │  │
│  │ (100+ ticks)│ Tasks wait long for their turn                  │  │
│  ├─────────────┼─────────────────────────────────────────────────┤  │
│  │ Typical     │ 10-20ms (10-20 ticks at 1kHz)                   │  │
│  │ Good balance│ ~1-5% overhead, good responsiveness             │  │
│  └─────────────┴─────────────────────────────────────────────────┘  │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## Rate Monotonic Scheduling (RMS)

Optimal static-priority algorithm for periodic tasks.

```
┌─────────────────────────────────────────────────────────────────────┐
│                    RATE MONOTONIC SCHEDULING                         │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  RULE: Shorter period = Higher priority                             │
│                                                                      │
│  PROPERTIES:                                                         │
│  • Static priority assignment (fixed at design time)                │
│  • Optimal among fixed-priority algorithms                          │
│  • Assumes: periodic tasks, deadlines = periods, independent tasks  │
│                                                                      │
│  EXAMPLE:                                                            │
│  ┌────────┬─────────┬──────────┬──────────────────────────────────┐ │
│  │ Task   │ Period  │ WCET     │ Priority (RMS)                   │ │
│  ├────────┼─────────┼──────────┼──────────────────────────────────┤ │
│  │ τ₁     │ 20ms    │ 5ms      │ Highest (shortest period)        │ │
│  │ τ₂     │ 50ms    │ 10ms     │ Medium                           │ │
│  │ τ₃     │ 100ms   │ 25ms     │ Lowest (longest period)          │ │
│  └────────┴─────────┴──────────┴──────────────────────────────────┘ │
│                                                                      │
│  TIMELINE (0 to 100ms):                                              │
│                                                                      │
│  τ₁(20ms): ██   ██   ██   ██   ██                                   │
│  τ₂(50ms): ████       ████                                          │
│  τ₃(100ms):   ████████████████████████████                          │
│                                                                      │
│       0    20   40   60   80   100 ms                               │
│       ├────┼────┼────┼────┼────┤                                    │
│                                                                      │
│  Detailed execution:                                                 │
│  [0-5]:   τ₁ runs (5ms)                                             │
│  [5-15]:  τ₂ runs (10ms)                                            │
│  [15-20]: τ₃ runs (5ms of 25ms)                                     │
│  [20-25]: τ₁ preempts, runs (5ms)                                   │
│  [25-40]: τ₃ resumes (15ms, completes 20ms)                         │
│  [40-45]: τ₁ runs                                                   │
│  [45-50]: τ₃ runs (5ms more, total 25ms done)                       │
│  [50-55]: τ₂ runs...                                                │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

### RMS Schedulability Test

**Utilization Bound Test** (sufficient but not necessary):

$$U = \sum_{i=1}^{n} \frac{C_i}{T_i} \leq n(2^{1/n} - 1)$$

| n (tasks) | Utilization Bound |
|-----------|-------------------|
| 1         | 1.000 (100%)      |
| 2         | 0.828 (82.8%)     |
| 3         | 0.780 (78.0%)     |
| 4         | 0.757 (75.7%)     |
| 5         | 0.743 (74.3%)     |
| ∞         | 0.693 (69.3%)     |

**Example Calculation**:
- τ₁: C=5ms, T=20ms → U₁ = 5/20 = 0.25
- τ₂: C=10ms, T=50ms → U₂ = 10/50 = 0.20
- τ₃: C=25ms, T=100ms → U₃ = 25/100 = 0.25

Total U = 0.25 + 0.20 + 0.25 = 0.70

Bound for n=3: 0.780

Since 0.70 < 0.780, system is **schedulable** under RMS.

---

## Earliest Deadline First (EDF)

Dynamic priority algorithm - optimal for single processor.

```
┌─────────────────────────────────────────────────────────────────────┐
│                    EARLIEST DEADLINE FIRST (EDF)                     │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  RULE: Task with earliest absolute deadline has highest priority    │
│                                                                      │
│  PROPERTIES:                                                         │
│  • Dynamic priority (changes at runtime)                            │
│  • Optimal: can achieve 100% CPU utilization                        │
│  • More complex to implement than RMS                               │
│  • Fails unpredictably under overload                               │
│                                                                      │
│  EXAMPLE: Same tasks as RMS                                         │
│  ┌────────┬─────────┬──────────┬──────────────────────────────────┐ │
│  │ Task   │ Period  │ WCET     │ Deadline = Period                │ │
│  ├────────┼─────────┼──────────┼──────────────────────────────────┤ │
│  │ τ₁     │ 20ms    │ 5ms      │ D=20ms from release              │ │
│  │ τ₂     │ 50ms    │ 10ms     │ D=50ms from release              │ │
│  │ τ₃     │ 100ms   │ 25ms     │ D=100ms from release             │ │
│  └────────┴─────────┴──────────┴──────────────────────────────────┘ │
│                                                                      │
│  PRIORITY CHANGES OVER TIME:                                         │
│                                                                      │
│  At t=0:                                                             │
│    τ₁ deadline = 20  ◀── Highest priority                          │
│    τ₂ deadline = 50                                                  │
│    τ₃ deadline = 100                                                │
│                                                                      │
│  At t=20 (τ₁ re-released):                                          │
│    τ₁ deadline = 40  ◀── Highest priority again                    │
│    τ₂ deadline = 50                                                  │
│    τ₃ deadline = 100                                                │
│                                                                      │
│  At t=40 (τ₁ re-released):                                          │
│    τ₂ deadline = 50  ◀── Now τ₂ is closest!                        │
│    τ₁ deadline = 60                                                  │
│    τ₃ deadline = 100                                                │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

### EDF Schedulability Test

**Simple test** (necessary and sufficient for EDF):

$$U = \sum_{i=1}^{n} \frac{C_i}{T_i} \leq 1$$

If U ≤ 1, the system is schedulable. EDF can achieve up to 100% utilization!

---

## RMS vs EDF Comparison

```
┌─────────────────────────────────────────────────────────────────────┐
│                    RMS vs EDF COMPARISON                             │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  ┌────────────────────┬──────────────────┬──────────────────────┐   │
│  │ Criterion          │ RMS              │ EDF                  │   │
│  ├────────────────────┼──────────────────┼──────────────────────┤   │
│  │ Priority           │ Static           │ Dynamic              │   │
│  │ Assignment         │ (period-based)   │ (deadline-based)     │   │
│  ├────────────────────┼──────────────────┼──────────────────────┤   │
│  │ Max Utilization    │ ~69% (worst)     │ 100%                 │   │
│  │                    │ ~88% (average)   │                      │   │
│  ├────────────────────┼──────────────────┼──────────────────────┤   │
│  │ Implementation     │ Simple           │ Complex              │   │
│  │ Complexity         │ (sorted queues)  │ (deadline tracking)  │   │
│  ├────────────────────┼──────────────────┼──────────────────────┤   │
│  │ Overload Behavior  │ Predictable      │ Unpredictable        │   │
│  │                    │ (low pri misses) │ (any task may miss)  │   │
│  ├────────────────────┼──────────────────┼──────────────────────┤   │
│  │ Context Switches   │ Fewer            │ More (dynamic)       │   │
│  ├────────────────────┼──────────────────┼──────────────────────┤   │
│  │ Analysis           │ Well-established │ More complex         │   │
│  ├────────────────────┼──────────────────┼──────────────────────┤   │
│  │ RTOS Support       │ All RTOSes       │ Limited              │   │
│  └────────────────────┴──────────────────┴──────────────────────┘   │
│                                                                      │
│  WHEN TO USE:                                                        │
│  • RMS: Safety-critical, needs predictable failure mode            │
│  • EDF: High utilization needed, non-critical applications         │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## Cooperative vs Preemptive Scheduling

```
┌─────────────────────────────────────────────────────────────────────┐
│                    COOPERATIVE vs PREEMPTIVE                         │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  COOPERATIVE SCHEDULING:                                             │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │ • Tasks run until they voluntarily yield                    │    │
│  │ • No forced context switches                                │    │
│  │ • Simpler (no critical sections needed)                     │    │
│  │ • Risk: one task can starve others                          │    │
│  └─────────────────────────────────────────────────────────────┘    │
│                                                                      │
│  Task A:  ████████████────────────────────                          │
│                      │ yield()                                      │
│  Task B:  ────────────████████████████────                          │
│                                       │ yield()                     │
│  Task C:  ────────────────────────────████████████                  │
│                                                                      │
│                                                                      │
│  PREEMPTIVE SCHEDULING:                                              │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │ • Higher-priority task can interrupt lower-priority         │    │
│  │ • Better responsiveness                                     │    │
│  │ • Requires synchronization for shared resources             │    │
│  │ • Used by most RTOSes                                       │    │
│  └─────────────────────────────────────────────────────────────┘    │
│                                                                      │
│  Task A (High):  ████        ████                                   │
│                      ▲           ▲ preempts                         │
│  Task B (Low):   ────────████────────████                           │
│                          │           │                               │
│                     B resumes   B resumes                            │
│                                                                      │
│                                                                      │
│  COMPARISON:                                                         │
│  ┌─────────────────┬───────────────────┬────────────────────────┐   │
│  │ Aspect          │ Cooperative       │ Preemptive             │   │
│  ├─────────────────┼───────────────────┼────────────────────────┤   │
│  │ Responsiveness  │ Poor (depends on  │ Good (bounded)         │   │
│  │                 │ task behavior)    │                        │   │
│  ├─────────────────┼───────────────────┼────────────────────────┤   │
│  │ Complexity      │ Simple            │ Complex (sync needed)  │   │
│  ├─────────────────┼───────────────────┼────────────────────────┤   │
│  │ Stack usage     │ Lower (shared)    │ Higher (per-task)      │   │
│  ├─────────────────┼───────────────────┼────────────────────────┤   │
│  │ Real-time       │ No guarantees     │ Can provide guarantees │   │
│  ├─────────────────┼───────────────────┼────────────────────────┤   │
│  │ Use case        │ Simple systems,   │ Real-time systems      │   │
│  │                 │ super-loop alt    │                        │   │
│  └─────────────────┴───────────────────┴────────────────────────┘   │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## Response Time Analysis

More precise than utilization test for RMS.

### Response Time Calculation

For task $τ_i$, the worst-case response time $R_i$ is:

$$R_i = C_i + \sum_{j \in hp(i)} \left\lceil \frac{R_i}{T_j} \right\rceil C_j$$

Where:
- $C_i$ = WCET of task $i$
- $hp(i)$ = set of tasks with higher priority than $i$
- $T_j$ = period of task $j$

This is solved iteratively:

```
┌─────────────────────────────────────────────────────────────────────┐
│                    RESPONSE TIME ANALYSIS EXAMPLE                    │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  Tasks: τ₁(T=20, C=5), τ₂(T=50, C=15), τ₃(T=100, C=30)             │
│                                                                      │
│  For τ₁ (highest priority, no interference):                        │
│    R₁ = C₁ = 5ms ✓                                                  │
│                                                                      │
│  For τ₂:                                                             │
│    Iteration 1: R₂⁰ = C₂ = 15                                        │
│    Iteration 2: R₂¹ = 15 + ⌈15/20⌉×5 = 15 + 5 = 20                  │
│    Iteration 3: R₂² = 15 + ⌈20/20⌉×5 = 15 + 5 = 20                  │
│    Converged: R₂ = 20ms ✓ (< D₂ = 50ms)                             │
│                                                                      │
│  For τ₃:                                                             │
│    Iteration 1: R₃⁰ = C₃ = 30                                        │
│    Iteration 2: R₃¹ = 30 + ⌈30/20⌉×5 + ⌈30/50⌉×15                   │
│                     = 30 + 2×5 + 1×15 = 55                           │
│    Iteration 3: R₃² = 30 + ⌈55/20⌉×5 + ⌈55/50⌉×15                   │
│                     = 30 + 3×5 + 2×15 = 75                           │
│    Iteration 4: R₃³ = 30 + ⌈75/20⌉×5 + ⌈75/50⌉×15                   │
│                     = 30 + 4×5 + 2×15 = 80                           │
│    Iteration 5: R₃⁴ = 30 + ⌈80/20⌉×5 + ⌈80/50⌉×15                   │
│                     = 30 + 4×5 + 2×15 = 80                           │
│    Converged: R₃ = 80ms ✓ (< D₃ = 100ms)                            │
│                                                                      │
│  All tasks meet deadlines → System is SCHEDULABLE                   │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## Scheduling Configuration in FreeRTOS

```c
/* FreeRTOSConfig.h */

// Enable preemption
#define configUSE_PREEMPTION                1

// Enable time-slicing for equal priorities
#define configUSE_TIME_SLICING              1

// Number of priority levels (0 to configMAX_PRIORITIES-1)
#define configMAX_PRIORITIES                8

// Tick rate (1000 = 1ms tick)
#define configTICK_RATE_HZ                  1000

// Idle task priority (usually 0 = lowest)
#define tskIDLE_PRIORITY                    0

// Time slice duration (in ticks)
// Only applies to tasks of equal priority
// Default: 1 tick (scheduler runs every tick)
```

---

## Summary Table

| Algorithm | Priority | Utilization | Complexity | Use Case |
|-----------|----------|-------------|------------|----------|
| **Fixed Priority Preemptive** | Static | Moderate | Low | Most RTOS |
| **Round-Robin** | Equal | N/A (fairness) | Low | Same-priority tasks |
| **Rate Monotonic (RMS)** | Static (period) | ≤69-88% | Low | Periodic tasks |
| **Earliest Deadline First (EDF)** | Dynamic (deadline) | ≤100% | High | High utilization |
| **Cooperative** | N/A | Variable | Very low | Simple systems |

---

## Quick Revision Questions

1. **What is the difference between preemptive and cooperative scheduling? When would you use each?**

2. **Three periodic tasks: τ₁(T=10ms, C=2ms), τ₂(T=20ms, C=4ms), τ₃(T=50ms, C=10ms). Assign RMS priorities and check if schedulable using the utilization bound test.**

3. **Why does EDF achieve higher utilization than RMS? What is the disadvantage of EDF under overload conditions?**

4. **A system has two equal-priority tasks. How does round-robin scheduling ensure fairness? What happens if the time slice is too small? Too large?**

5. **Using response time analysis, verify that task τ₂(T=30, C=10) meets its deadline when τ₁(T=10, C=3) has higher priority.**

6. **Compare RMS and EDF for a system with U=85%. Which can schedule it? What are the trade-offs?**

---

## Navigation

| Previous | Up | Next |
|----------|-----|------|
| [Chapter 4.3: Task Management](03-task-management.md) | [Unit Index](README.md) | [Chapter 4.5: Synchronization](05-synchronization.md) |

---
