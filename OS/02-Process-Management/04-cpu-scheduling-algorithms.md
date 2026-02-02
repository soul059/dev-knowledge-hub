# Chapter 2.4: CPU Scheduling Algorithms

## Overview

CPU scheduling algorithms determine which process in the ready queue gets the CPU next. Different algorithms optimize for different goals and are suited for different types of systems.

---

## Classification of Scheduling Algorithms

```
                    SCHEDULING ALGORITHMS
                            │
            ┌───────────────┴───────────────┐
            │                               │
      Non-Preemptive                   Preemptive
            │                               │
    ┌───────┴───────┐              ┌────────┴────────┐
    │               │              │        │        │
   FCFS            SJF           SRTF   Priority   Round
   (SJN)       (Non-Preemptive)         (Preemptive) Robin
```

---

## 1. First-Come, First-Served (FCFS)

### Concept
Processes are executed in the order they arrive. The first process to arrive gets the CPU first.

### Characteristics
- **Type**: Non-preemptive
- **Data Structure**: Simple FIFO queue
- **Selection**: Process at head of queue
- **Implementation**: Easiest to implement

### Algorithm

```
FCFS Algorithm:
1. Processes are placed in ready queue as they arrive
2. CPU is allocated to process at front of queue
3. Process runs until completion (or I/O wait)
4. Next process from queue gets CPU
```

### Example

| Process | Arrival Time | Burst Time |
|---------|--------------|------------|
| P1 | 0 | 24 |
| P2 | 1 | 3 |
| P3 | 2 | 3 |

**Gantt Chart:**
```
┌────────────────────────────────────────┬─────────┬─────────┐
│                  P1                     │   P2    │   P3    │
└────────────────────────────────────────┴─────────┴─────────┘
0                                        24       27        30
```

**Calculations:**
```
Process | Completion | Turnaround | Waiting
        | Time (CT)  | Time (TAT) | Time (WT)
--------|------------|------------|----------
P1      | 24         | 24-0 = 24  | 24-24 = 0
P2      | 27         | 27-1 = 26  | 26-3 = 23
P3      | 30         | 30-2 = 28  | 28-3 = 25

Average Waiting Time = (0 + 23 + 25) / 3 = 16 ms
Average Turnaround Time = (24 + 26 + 28) / 3 = 26 ms
```

### Convoy Effect

The **Convoy Effect** is a major problem with FCFS:

```
Scenario: One CPU-bound process followed by many I/O-bound processes

P1 (CPU-bound, 100ms burst):  ████████████████████████████████████████
P2 (I/O-bound, 2ms burst):                                            ██
P3 (I/O-bound, 2ms burst):                                              ██
P4 (I/O-bound, 2ms burst):                                                ██

All short processes wait for one long process - like a convoy!
```

### Advantages and Disadvantages

| Advantages | Disadvantages |
|------------|---------------|
| Simple to implement | Convoy effect |
| Fair (FIFO order) | Poor average waiting time |
| No starvation | Not suitable for time-sharing |

---

## 2. Shortest Job First (SJF) / Shortest Job Next (SJN)

### Concept
Select the process with the smallest CPU burst time. Can be preemptive or non-preemptive.

### Non-Preemptive SJF

Once a process starts, it runs to completion.

### Example (Non-Preemptive SJF)

| Process | Arrival Time | Burst Time |
|---------|--------------|------------|
| P1 | 0 | 7 |
| P2 | 2 | 4 |
| P3 | 4 | 1 |
| P4 | 5 | 4 |

**Scheduling Steps:**
```
At t=0: Only P1 available → P1 runs (7ms)
At t=7: P2, P3, P4 available → P3 has shortest burst (1ms)
At t=8: P2, P4 available → P2 has shorter burst (4ms)
At t=12: Only P4 left → P4 runs (4ms)
```

**Gantt Chart:**
```
┌─────────────────┬───┬───────────────┬───────────────┐
│       P1        │P3 │      P2       │      P4       │
└─────────────────┴───┴───────────────┴───────────────┘
0                 7   8              12              16
```

**Calculations:**
```
Process | CT  | TAT = CT-AT | WT = TAT-BT
--------|-----|-------------|------------
P1      | 7   | 7-0 = 7     | 7-7 = 0
P2      | 12  | 12-2 = 10   | 10-4 = 6
P3      | 8   | 8-4 = 4     | 4-1 = 3
P4      | 16  | 16-5 = 11   | 11-4 = 7

Average Waiting Time = (0 + 6 + 3 + 7) / 4 = 4 ms
Average Turnaround Time = (7 + 10 + 4 + 11) / 4 = 8 ms
```

### SJF is Optimal

SJF gives the **minimum average waiting time** for a given set of processes.

```
Proof Intuition:
- Short jobs complete quickly, reducing wait for others
- Long jobs at end only affect their own wait time

Example comparison:
FCFS order [8, 4, 2]:  Waits = 0, 8, 12  → Avg = 6.67
SJF order [2, 4, 8]:   Waits = 0, 2, 6   → Avg = 2.67  ← Better!
```

### Problem: Predicting Burst Time

We cannot know CPU burst time in advance. Solutions:
1. **User estimation**: Ask user to specify
2. **Historical data**: Use previous burst times
3. **Exponential averaging**: Predict based on past

### Exponential Averaging Formula

```
τ(n+1) = α × t(n) + (1-α) × τ(n)

Where:
τ(n+1) = Predicted next CPU burst
t(n)   = Actual length of nth burst
τ(n)   = Predicted value of nth burst
α      = Weight (0 ≤ α ≤ 1), typically 0.5

Example:
If α = 0.5, τ(0) = 10, and actual bursts are [6, 4, 6, 4, ...]

τ(1) = 0.5 × 6 + 0.5 × 10 = 8
τ(2) = 0.5 × 4 + 0.5 × 8 = 6
τ(3) = 0.5 × 6 + 0.5 × 6 = 6
...
```

---

## 3. Shortest Remaining Time First (SRTF)

### Concept
Preemptive version of SJF. When a new process arrives, if its burst time is shorter than the remaining time of current process, preempt.

### Example

| Process | Arrival Time | Burst Time |
|---------|--------------|------------|
| P1 | 0 | 8 |
| P2 | 1 | 4 |
| P3 | 2 | 9 |
| P4 | 3 | 5 |

**Scheduling Steps:**
```
t=0: P1 arrives, starts (remaining: 8)
t=1: P2 arrives (burst 4 < P1's remaining 7) → P2 preempts P1
t=2: P3 arrives (burst 9 > P2's remaining 3) → P2 continues
t=3: P4 arrives (burst 5 > P2's remaining 2) → P2 continues
t=5: P2 completes. P1(7), P3(9), P4(5) waiting → P4 shortest
t=10: P4 completes. P1(7), P3(9) waiting → P1 shorter
t=17: P1 completes. Only P3 left
t=26: P3 completes
```

**Gantt Chart:**
```
┌───┬─────────────┬───────────────────┬─────────────────────┬─────────────────────────────┐
│P1 │     P2      │        P4         │         P1          │             P3              │
└───┴─────────────┴───────────────────┴─────────────────────┴─────────────────────────────┘
0   1             5                   10                    17                            26
```

**Calculations:**
```
Process | CT  | TAT = CT-AT | WT = TAT-BT
--------|-----|-------------|------------
P1      | 17  | 17-0 = 17   | 17-8 = 9
P2      | 5   | 5-1 = 4     | 4-4 = 0
P3      | 26  | 26-2 = 24   | 24-9 = 15
P4      | 10  | 10-3 = 7    | 7-5 = 2

Average Waiting Time = (9 + 0 + 15 + 2) / 4 = 6.5 ms
Average Turnaround Time = (17 + 4 + 24 + 7) / 4 = 13 ms
```

### SRTF Advantages and Disadvantages

| Advantages | Disadvantages |
|------------|---------------|
| Optimal average waiting time | Frequent context switches |
| Better response for short jobs | Starvation of long processes |
| Good for interactive systems | Requires burst time prediction |

---

## 4. Priority Scheduling

### Concept
Each process is assigned a priority. The CPU is allocated to the process with the highest priority.

### Priority Types

```
Priority Numbers:
┌─────────────────────────────────────────────────────┐
│  Convention 1: Lower number = Higher priority       │
│  (0 is highest, used in UNIX)                       │
├─────────────────────────────────────────────────────┤
│  Convention 2: Higher number = Higher priority      │
│  (Larger is more important)                         │
└─────────────────────────────────────────────────────┘
```

### Example (Non-Preemptive, Lower = Higher Priority)

| Process | Arrival Time | Burst Time | Priority |
|---------|--------------|------------|----------|
| P1 | 0 | 10 | 3 |
| P2 | 0 | 1 | 1 |
| P3 | 0 | 2 | 4 |
| P4 | 0 | 1 | 5 |
| P5 | 0 | 5 | 2 |

**Order by Priority: P2 (1) → P5 (2) → P1 (3) → P3 (4) → P4 (5)**

**Gantt Chart:**
```
┌───┬───────────────┬─────────────────────────────┬───────┬───┐
│P2 │      P5       │             P1              │  P3   │P4 │
└───┴───────────────┴─────────────────────────────┴───────┴───┘
0   1               6                             16      18  19
```

**Calculations:**
```
Process | CT  | WT = CT - BT (all arrived at 0)
--------|-----|--------------------------------
P1      | 16  | 16 - 10 = 6
P2      | 1   | 1 - 1 = 0
P3      | 18  | 18 - 2 = 16
P4      | 19  | 19 - 1 = 18
P5      | 6   | 6 - 5 = 1

Average Waiting Time = (6 + 0 + 16 + 18 + 1) / 5 = 8.2 ms
```

### Priority Assignment

| Type | Description | Example |
|------|-------------|---------|
| **Static** | Fixed at creation | System processes: high priority |
| **Dynamic** | Changes during execution | Aging to prevent starvation |
| **Internal** | Based on measurable criteria | Memory requirements, I/O ratio |
| **External** | Set by user/admin | Importance, fees paid |

### Starvation Problem

Low-priority processes may **never execute** if high-priority processes keep arriving.

```
Problem:
High Priority Processes: ████████████████████████████████████→
                         Keep arriving continuously
                         
Low Priority Process:    Waiting... Waiting... Waiting... ∞
                         Never gets CPU!
```

### Solution: Aging

**Aging** gradually increases the priority of waiting processes.

```
Aging Algorithm:
Every N time units, increase priority of all waiting processes by 1

Example:
Process X, Initial Priority: 50 (low)

Time 0:   Priority = 50 (waiting)
Time 10:  Priority = 49 (waiting, aged)
Time 20:  Priority = 48 (waiting, aged)
...
Time 200: Priority = 30 (now competitive!)
Time 300: Priority = 20 (will run soon)
```

---

## 5. Round Robin (RR)

### Concept
Each process gets a small unit of CPU time (time quantum). After quantum expires, process is preempted and added to the end of the ready queue.

### Characteristics
- **Type**: Preemptive
- **Key Parameter**: Time quantum (q)
- **Data Structure**: Circular queue
- **Fairness**: All processes treated equally

### Algorithm

```
Round Robin Algorithm:
1. Set time quantum = q
2. Pick first process from ready queue
3. Run for min(remaining burst, q)
4. If process not complete, add to end of queue
5. Pick next process, repeat from step 3
```

### Example

| Process | Arrival Time | Burst Time |
|---------|--------------|------------|
| P1 | 0 | 10 |
| P2 | 0 | 4 |
| P3 | 0 | 5 |

**Time Quantum = 3**

**Scheduling Steps:**
```
Queue: [P1, P2, P3]
t=0-3:   P1 runs (remaining: 7), Queue: [P2, P3, P1]
t=3-6:   P2 runs (remaining: 1), Queue: [P3, P1, P2]
t=6-9:   P3 runs (remaining: 2), Queue: [P1, P2, P3]
t=9-12:  P1 runs (remaining: 4), Queue: [P2, P3, P1]
t=12-13: P2 runs (remaining: 0), completes! Queue: [P3, P1]
t=13-15: P3 runs (remaining: 0), completes! Queue: [P1]
t=15-18: P1 runs (remaining: 1), Queue: [P1]
t=18-19: P1 runs (remaining: 0), completes!
```

**Gantt Chart:**
```
┌─────────┬─────────┬─────────┬─────────┬───┬─────┬─────────┬───┐
│   P1    │   P2    │   P3    │   P1    │P2 │ P3  │   P1    │P1 │
└─────────┴─────────┴─────────┴─────────┴───┴─────┴─────────┴───┘
0         3         6         9        12  13    15        18  19
```

**Calculations:**
```
Process | CT  | TAT = CT-AT | WT = TAT-BT
--------|-----|-------------|------------
P1      | 19  | 19-0 = 19   | 19-10 = 9
P2      | 13  | 13-0 = 13   | 13-4 = 9
P3      | 15  | 15-0 = 15   | 15-5 = 10

Average Waiting Time = (9 + 9 + 10) / 3 = 9.33 ms
Average Turnaround Time = (19 + 13 + 15) / 3 = 15.67 ms
```

### Effect of Time Quantum

```
Time Quantum Selection:
┌──────────────────────────────────────────────────────────────────┐
│                                                                  │
│  q → ∞:  Round Robin becomes FCFS                               │
│          (Each process completes before switch)                  │
│                                                                  │
│  q → 0:  Called "Processor Sharing"                             │
│          (Too many context switches - overhead dominates)        │
│                                                                  │
│  Ideal q: Should be larger than 80% of CPU bursts               │
│           Typically 10-100 milliseconds                          │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘

                 Context Switches vs Time Quantum
                 
Context     │
Switches    │\
            │ \
            │  \
            │   \____
            │        \_______
            └──────────────────────→ Time Quantum
                    Optimal range
```

### Rule of Thumb

```
80% Rule:
Choose time quantum such that ~80% of CPU bursts complete within one quantum

Typical values:
- Interactive systems: 10-100 ms
- Batch systems: 100-500 ms
```

---

## 6. Multilevel Queue Scheduling

### Concept
Ready queue is partitioned into separate queues based on process characteristics. Each queue can have its own scheduling algorithm.

### Structure

```
┌──────────────────────────────────────────────────────────────┐
│               MULTILEVEL QUEUE SCHEDULING                     │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  Highest Priority                                            │
│  ┌─────────────────────────────────────────────────────────┐ │
│  │ System Processes                        [Priority]       │ │
│  └──────────────────────────────────┬──────────────────────┘ │
│                                     │                        │
│  ┌──────────────────────────────────▼──────────────────────┐ │
│  │ Interactive Processes                   [RR]             │ │
│  └──────────────────────────────────┬──────────────────────┘ │
│                                     │                        │
│  ┌──────────────────────────────────▼──────────────────────┐ │
│  │ Interactive Editing Processes           [RR]             │ │
│  └──────────────────────────────────┬──────────────────────┘ │
│                                     │                        │
│  ┌──────────────────────────────────▼──────────────────────┐ │
│  │ Batch Processes                         [FCFS]           │ │
│  └──────────────────────────────────┬──────────────────────┘ │
│                                     │                        │
│  ┌──────────────────────────────────▼──────────────────────┐ │
│  │ Student Processes                       [FCFS]           │ │
│  └─────────────────────────────────────────────────────────┘ │
│  Lowest Priority                                             │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

### Scheduling Between Queues

| Method | Description |
|--------|-------------|
| **Fixed Priority** | Higher queue always first (may cause starvation) |
| **Time Slice** | Each queue gets % of CPU time (e.g., 80% interactive, 20% batch) |

---

## 7. Multilevel Feedback Queue Scheduling

### Concept
Allows processes to move between queues based on their behavior. Process that uses too much CPU time is moved to lower-priority queue.

### Structure

```
┌──────────────────────────────────────────────────────────────────────┐
│             MULTILEVEL FEEDBACK QUEUE SCHEDULING                      │
├──────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  Queue 0: RR, q=8ms    [Highest Priority]                           │
│  ┌─────────────────────────────────────────────────────────────────┐│
│  │ New processes enter here                                        ││
│  │ If not complete in 8ms → demote to Queue 1                      ││
│  └──────────────────────────────────────────────────┬──────────────┘│
│                                                     │ Demote        │
│  Queue 1: RR, q=16ms                               ↓                │
│  ┌─────────────────────────────────────────────────────────────────┐│
│  │ Processes that need more time                                   ││
│  │ If not complete in 16ms → demote to Queue 2                     ││
│  └──────────────────────────────────────────────────┬──────────────┘│
│                                                     │ Demote        │
│  Queue 2: FCFS         [Lowest Priority]           ↓                │
│  ┌─────────────────────────────────────────────────────────────────┐│
│  │ Long-running (CPU-bound) processes                              ││
│  │ Runs when higher queues are empty                               ││
│  └─────────────────────────────────────────────────────────────────┘│
│                                                                      │
│  Aging: Long-waiting processes in lower queues can be               │
│         promoted back to higher queues                               │
│                                                                      │
└──────────────────────────────────────────────────────────────────────┘
```

### Parameters to Define

1. Number of queues
2. Scheduling algorithm for each queue
3. Method to determine when to demote a process
4. Method to determine when to promote a process
5. Method to determine which queue a process enters initially

### Example

```
Process P enters system:
1. P enters Queue 0 (RR, q=8)
2. Gets 8ms, needs more → P demoted to Queue 1
3. Gets 16ms, needs more → P demoted to Queue 2
4. Runs in FCFS when queues 0 and 1 are empty

If P becomes I/O-bound later, it may be promoted back up

This separates:
- I/O-bound (short bursts) → stay in high priority
- CPU-bound (long bursts) → sink to low priority
```

---

## Algorithm Comparison

### Comparison Table

| Algorithm | Preemptive? | Starvation? | Overhead | Best For |
|-----------|-------------|-------------|----------|----------|
| FCFS | No | No | Low | Batch systems |
| SJF | No | Yes | Low | When burst known |
| SRTF | Yes | Yes | High | Short jobs |
| Priority | Both | Yes | Medium | Important tasks |
| RR | Yes | No | High | Time-sharing |
| MLQ | Both | Yes | Medium | Multiple types |
| MLFQ | Yes | No (aging) | High | General purpose |

### Average Waiting Time Comparison

```
For the same process set, typical average waiting times:

                Longer                           Shorter
                   │                                │
       FCFS ───────┼────────────────────────────────┤
                   │                                │
   Priority ───────┼──────────────────────┤         │
                   │                      │         │
         RR ───────┼───────────────────┤  │         │
                   │                   │  │         │
        SJF ───────┼───────────┤       │  │         │
                   │           │       │  │         │
       SRTF ───────┼─────┤     │       │  │         │
                   │     │     │       │  │         │
                   └─────┴─────┴───────┴──┴─────────┘
                         SRTF is optimal
```

---

## Numerical Practice Problem

### Problem

| Process | Arrival Time | Burst Time |
|---------|--------------|------------|
| P1 | 0 | 5 |
| P2 | 1 | 3 |
| P3 | 2 | 8 |
| P4 | 3 | 6 |

Calculate average waiting time and turnaround time for:
1. FCFS
2. SJF (Non-Preemptive)
3. Round Robin (q=2)

### Solution

**1. FCFS:**
```
Gantt: |--P1--|--P2--|----P3----|----P4----|
       0     5      8         16         22

WT: P1=0, P2=5-1=4, P3=8-2=6, P4=16-3=13
Avg WT = (0+4+6+13)/4 = 5.75

TAT: P1=5, P2=8-1=7, P3=16-2=14, P4=22-3=19
Avg TAT = (5+7+14+19)/4 = 11.25
```

**2. SJF (Non-Preemptive):**
```
t=0: Only P1 → runs
t=5: P2(3), P3(8), P4(6) → P2 shortest
t=8: P3(8), P4(6) → P4 shorter
t=14: Only P3 left

Gantt: |--P1--|--P2--|----P4----|----P3----|
       0     5      8         14         22

WT: P1=0, P2=5-1=4, P3=14-2=12, P4=8-3=5
Avg WT = (0+4+12+5)/4 = 5.25

TAT: P1=5, P2=8-1=7, P3=22-2=20, P4=14-3=11
Avg TAT = (5+7+20+11)/4 = 10.75
```

**3. Round Robin (q=2):**
```
Queue at each step:
t=0: [P1]
t=0-2: P1 runs (rem:3), Queue: [P1] → [P2, P1]
t=2-4: P2 runs (rem:1), Queue: [P1, P3, P2]
t=4-6: P1 runs (rem:1), Queue: [P3, P2, P4, P1]
t=6-8: P3 runs (rem:6), Queue: [P2, P4, P1, P3]
t=8-9: P2 runs (rem:0), done! Queue: [P4, P1, P3]
t=9-10: P4 runs (rem:4), Queue: [P1, P3, P4]
t=10-11: P1 runs (rem:0), done! Queue: [P3, P4]
... continue

Gantt: |P1|P2|P1|P3|P2|P4|P1|P3|P4|P3|P4|P3|P3|
       0  2  4  6  8  9 11 12 14 16 18 20 22 24

CT: P1=11, P2=9, P3=22, P4=20
WT: P1=11-5=6, P2=9-1-3=5, P3=22-2-8=12, P4=20-3-6=11
Avg WT = (6+5+12+11)/4 = 8.5
```

---

## Summary

| Algorithm | Key Feature | When to Use |
|-----------|-------------|-------------|
| FCFS | First arrived, first served | Simple batch jobs |
| SJF | Shortest job runs first | Known burst times |
| SRTF | Preemptive SJF | Interactive + efficiency |
| Priority | Based on importance | System processes |
| Round Robin | Time-sliced equally | Time-sharing systems |
| MLFQ | Adaptive to behavior | General-purpose OS |

---

## Quick Revision Questions

1. What is the convoy effect in FCFS?
2. Why is SJF considered optimal?
3. What is the main disadvantage of priority scheduling?
4. How does the time quantum affect Round Robin performance?
5. How does MLFQ differ from MLQ?

---

[← Previous: Process Scheduling](./03-process-scheduling.md) | [Next: Threads →](./05-threads.md)
