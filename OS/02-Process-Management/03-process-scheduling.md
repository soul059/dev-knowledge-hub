# Chapter 2.3: Process Scheduling

## Overview

Process scheduling is the mechanism by which the operating system decides which process runs on the CPU at any given time. It's fundamental to multiprogramming and ensures efficient CPU utilization.

---

## What is Process Scheduling?

### Definition
**Process Scheduling** is the activity of the process manager that handles the removal of the running process from the CPU and the selection of another process based on a particular strategy.

### Why Scheduling?

```
Without Scheduling:                  With Scheduling:
┌─────────────────────────────┐     ┌─────────────────────────────┐
│ Process 1 runs until done   │     │ P1 │ P2 │ P3 │ P1 │ P2 │ P3│
│ Then Process 2              │     │ All processes get CPU time │
│ Then Process 3              │     │ Fair and efficient         │
│ Others wait indefinitely    │     │                             │
└─────────────────────────────┘     └─────────────────────────────┘
```

### Goals of Scheduling

| Goal | Description |
|------|-------------|
| **CPU Utilization** | Keep CPU as busy as possible (40-90%) |
| **Throughput** | Complete as many processes as possible per time unit |
| **Turnaround Time** | Minimize total time from submission to completion |
| **Waiting Time** | Minimize time spent waiting in ready queue |
| **Response Time** | Minimize time from request to first response |
| **Fairness** | Give each process fair share of CPU |

---

## Types of Schedulers

Operating systems employ multiple levels of scheduling:

```
┌─────────────────────────────────────────────────────────────────┐
│                      SCHEDULING LEVELS                           │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│                    ┌─────────────────┐                          │
│    Jobs on Disk    │  LONG-TERM      │     Jobs in Memory       │
│    (Job Pool)  ───→│  SCHEDULER      │───→ (Ready Queue)        │
│                    │  (Job Scheduler)│                          │
│                    └─────────────────┘                          │
│                             │                                   │
│                             ↓                                   │
│                    ┌─────────────────┐                          │
│   Ready Queue      │  SHORT-TERM     │      CPU                 │
│   (In Memory)  ───→│  SCHEDULER      │───→  (Running)           │
│                    │  (CPU Scheduler)│                          │
│                    └─────────────────┘                          │
│                             │                                   │
│                             ↓                                   │
│                    ┌─────────────────┐                          │
│   Memory ←────────→│  MEDIUM-TERM    │←────→ Disk               │
│                    │  SCHEDULER      │  (Swap In/Out)           │
│                    │  (Swapper)      │                          │
│                    └─────────────────┘                          │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### 1. Long-Term Scheduler (Job Scheduler)

| Aspect | Description |
|--------|-------------|
| **Function** | Selects which jobs to bring into memory |
| **Frequency** | Executes infrequently (seconds to minutes) |
| **Controls** | Degree of multiprogramming |
| **Decision** | Which job from job pool enters ready queue |

```
Job Pool (Disk)                     Ready Queue (Memory)
┌─────────────────┐                ┌─────────────────┐
│ Job A           │                │ Process 1       │
│ Job B           │  Long-Term    │ Process 2       │
│ Job C           │ ──Scheduler──→│ Process 3       │
│ Job D           │                │                 │
│ Job E           │                │                 │
└─────────────────┘                └─────────────────┘
        Many jobs                     Limited by memory
```

**Process Mix Consideration:**
- **I/O-bound processes**: Spend more time doing I/O than computation
- **CPU-bound processes**: Spend more time doing computation
- **Good mix**: Balance of both for optimal resource utilization

### 2. Short-Term Scheduler (CPU Scheduler)

| Aspect | Description |
|--------|-------------|
| **Function** | Selects which ready process gets CPU next |
| **Frequency** | Executes very frequently (milliseconds) |
| **Speed** | Must be very fast |
| **Decision** | Which process in ready queue runs next |

```
Ready Queue                              CPU
┌─────────────────┐                 ┌─────────────┐
│ Process A       │                 │             │
│ Process B       │  Short-Term    │  Running    │
│ Process C       │ ──Scheduler──→ │  Process    │
│ Process D       │                 │             │
└─────────────────┘                 └─────────────┘
  Waiting to run                      Executing
```

### 3. Medium-Term Scheduler (Swapper)

| Aspect | Description |
|--------|-------------|
| **Function** | Swaps processes between memory and disk |
| **Frequency** | When memory is scarce |
| **Purpose** | Reduce degree of multiprogramming |
| **Decision** | Which process to swap out/in |

```
                   MEMORY                          DISK
┌────────────────────────────────┐    ┌────────────────────────┐
│  Process A (Active)            │    │                        │
│  Process B (Active)            │    │  Process X (Swapped)   │
│  Process C (Idle for long)     │───→│  Process Y (Swapped)   │
│                                │←───│                        │
│  Free Space                    │    │                        │
└────────────────────────────────┘    └────────────────────────┘
            Swap Out when memory full
            Swap In when process needed
```

---

## Scheduler Comparison

| Scheduler | Frequency | Speed Required | Location |
|-----------|-----------|----------------|----------|
| **Long-Term** | Minutes | Can be slow | Disk → Memory |
| **Short-Term** | Milliseconds | Very fast | Memory → CPU |
| **Medium-Term** | Seconds | Moderate | Memory ↔ Disk |

---

## Scheduling Queues

Processes move between various scheduling queues:

```
┌─────────────────────────────────────────────────────────────────────────┐
│                        QUEUEING DIAGRAM                                  │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│                     ┌────────────┐                                      │
│                     │   CPU      │                                      │
│                     └──────┬─────┘                                      │
│                            │                                            │
│                            ↓                                            │
│                     ┌────────────┐      ┌─────────────────┐            │
│ ┌───────────┐      │   Ready    │      │    I/O          │            │
│ │   New     │─────→│   Queue    │─────→│    Request      │────┐       │
│ │  Process  │      └────────────┘      └─────────────────┘    │       │
│ └───────────┘                                                  │       │
│                                                                ↓       │
│                                                         ┌──────────┐   │
│                                                         │   I/O    │   │
│                                                         │  Queue   │   │
│                                                         └────┬─────┘   │
│                                                              │         │
│                            ┌─────────────────────────────────┘         │
│                            │                                           │
│                            ↓                                           │
│                     ┌────────────────┐                                 │
│                     │  I/O Complete  │──────→ Back to Ready Queue     │
│                     └────────────────┘                                 │
│                                                                         │
│  Other Events:                                                          │
│  • Time Slice Expired ────→ Back to Ready Queue                        │
│  • Fork Child ────→ Child enters Ready Queue                           │
│  • Wait for Interrupt ────→ Enter Wait Queue                           │
│  • Terminate ────→ Exit                                                 │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

### Types of Queues

| Queue | Contents | Purpose |
|-------|----------|---------|
| **Job Queue** | All processes in the system | Track all processes |
| **Ready Queue** | Processes in memory, ready to run | Wait for CPU |
| **Device Queues** | Processes waiting for I/O device | Wait for I/O |

### Queue Implementation

```
Ready Queue (Linked List of PCBs):

Queue Header
┌────────────┐
│ Head ──────┼───→ ┌─────┐   ┌─────┐   ┌─────┐   ┌─────┐
│ Tail ──────┼──┐  │PCB 1│──→│PCB 2│──→│PCB 3│──→│PCB 4│──→ NULL
└────────────┘  │  └─────┘   └─────┘   └─────┘   └──▲──┘
                │                                   │
                └───────────────────────────────────┘
```

---

## Dispatcher

The **Dispatcher** is the module that gives control of the CPU to the process selected by the short-term scheduler.

### Dispatcher Functions

```
┌──────────────────────────────────────────────────────────────┐
│                      DISPATCHER                               │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  1. Context Switch                                           │
│     • Save state of current process                          │
│     • Load state of selected process                         │
│                                                              │
│  2. Mode Switch                                              │
│     • Switch from kernel mode to user mode                   │
│                                                              │
│  3. Jump to Proper Location                                  │
│     • Jump to the location in user program                   │
│     • Resume execution from where it left off                │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

### Dispatch Latency

**Dispatch Latency** is the time taken by the dispatcher to stop one process and start another.

```
Time ─────────────────────────────────────────────────────→

Process A: ████████████│←──────────────→│
                       │ Dispatch       │
Process B:             │ Latency        │████████████████
                       │                │
                       Save A's state   Load B's state
                       Mode switch      Jump to B
```

**Goal**: Minimize dispatch latency for better system responsiveness.

---

## Preemptive vs Non-Preemptive Scheduling

### Non-Preemptive (Cooperative) Scheduling

Once a process is allocated the CPU, it keeps the CPU until it:
- Terminates
- Switches to waiting state (I/O request)
- Voluntarily yields CPU

```
Non-Preemptive:
Process A: ████████████████████████████████│
                                           │ A finishes or waits
Process B:                                 │████████████████████

Process A runs until it gives up CPU voluntarily
```

### Preemptive Scheduling

The OS can forcibly remove a process from the CPU:
- Time quantum expires
- Higher priority process arrives
- Resource becomes available

```
Preemptive:
Process A: ████│    │████│    │████│    │
               │    │    │    │    │    │ Time slice expired
Process B:     │████│    │████│    │████│

CPU shared between processes by time-slicing
```

### Comparison

| Aspect | Non-Preemptive | Preemptive |
|--------|----------------|------------|
| **CPU Surrender** | Voluntary | Forced |
| **Responsiveness** | Poor | Good |
| **Starvation Risk** | High | Low |
| **Overhead** | Low | Higher (context switches) |
| **Complexity** | Simple | Complex |
| **Use Case** | Batch systems | Interactive systems |

---

## Scheduling Criteria

### 1. CPU Utilization
- Keep the CPU as busy as possible
- Target: 40-90% utilization
- Formula: (Time CPU is busy / Total time) × 100%

### 2. Throughput
- Number of processes completed per unit time
- Higher is better
- Example: 10 processes/second

### 3. Turnaround Time

**Definition**: Total time from process submission to completion.

```
Turnaround Time = Completion Time - Arrival Time

Timeline:
Arrival         Start           Completion
   │              │                 │
   ↓              ↓                 ↓
   ├──────────────┼─────────────────┤
   │  Waiting     │    Running      │
   │              │                 │
   └──────────────┴─────────────────┘
   │←──────── Turnaround Time ─────→│
```

### 4. Waiting Time

**Definition**: Time spent waiting in the ready queue.

```
Waiting Time = Turnaround Time - Burst Time

OR

Waiting Time = Time process spends in Ready Queue

Example:
   Arrival    Queue    Run     Queue    Run     Done
      │         │       │        │       │        │
      ├─────────┼───────┼────────┼───────┼────────┤
      0    Wait: 5ms   Run:3ms  Wait:4ms Run:2ms  14ms

Waiting Time = 5 + 4 = 9ms
```

### 5. Response Time

**Definition**: Time from request submission to first response (not completion).

```
Response Time = First Response Time - Arrival Time

Important for interactive systems (user expects quick feedback)

Example:
   Submit         First Output
      │               │
      ↓               ↓
      ├───────────────┼────────────────────┤
      │←─Response───→│                     │
      │    Time       │    Rest of         │
      │               │    Processing      │
      0              50ms                 200ms
      
Response Time = 50ms (user sees something happening)
```

### Criteria Summary

| Criterion | Goal | Measure |
|-----------|------|---------|
| CPU Utilization | Maximize | Percentage |
| Throughput | Maximize | Processes/time |
| Turnaround Time | Minimize | Time units |
| Waiting Time | Minimize | Time units |
| Response Time | Minimize | Time units |

---

## CPU-I/O Burst Cycle

Process execution consists of alternating cycles of CPU execution and I/O wait.

```
┌──────────────────────────────────────────────────────────────────┐
│                    CPU-I/O BURST CYCLE                            │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  CPU Burst    I/O Burst    CPU Burst    I/O Burst    CPU Burst  │
│  ┌──────┐     ┌──────┐     ┌──────┐     ┌──────┐     ┌──────┐   │
│  │██████│     │░░░░░░│     │██████│     │░░░░░░│     │██████│   │
│  │██████│     │░░░░░░│     │████│       │░░░░░░│     │██│       │
│  │██████│     │░░░░░░│     │████│       │░░░░░░│     └──┘       │
│  └──────┘     │░░░░░░│     └────┘       └──────┘                │
│               └──────┘                                           │
│                                                                  │
│  ████ = CPU Execution    ░░░░ = I/O Wait                        │
│                                                                  │
│  Process alternates between computing and waiting for I/O        │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

### CPU Burst Distribution

```
Number of Bursts
       ↑
       │ ██
       │ ██
       │ ██ ██
       │ ██ ██ ██
       │ ██ ██ ██ ██
       │ ██ ██ ██ ██ ██
       │ ██ ██ ██ ██ ██ ██ ██
       │ ██ ██ ██ ██ ██ ██ ██ ██ ██ ██
       └──────────────────────────────────→ Burst Duration (ms)
         2  4  6  8  10 12 14 16 18 20

Most CPU bursts are short (exponential distribution)
Scheduling algorithms should optimize for short bursts
```

### Process Types by Burst Pattern

| Type | CPU Bursts | I/O Bursts | Example |
|------|------------|------------|---------|
| **CPU-bound** | Long | Short | Scientific computing |
| **I/O-bound** | Short | Long | Text editor, database |
| **Balanced** | Medium | Medium | Web server |

---

## Scheduling Points

When does scheduling occur?

```
┌──────────────────────────────────────────────────────────────────┐
│                    SCHEDULING DECISION POINTS                     │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  1. Process switches from RUNNING to WAITING                     │
│     (e.g., I/O request, wait for child)                         │
│     Non-preemptive: Yes    Preemptive: Yes                      │
│                                                                  │
│  2. Process switches from RUNNING to READY                       │
│     (e.g., interrupt, time quantum expires)                     │
│     Non-preemptive: No     Preemptive: Yes                      │
│                                                                  │
│  3. Process switches from WAITING to READY                       │
│     (e.g., I/O completion)                                      │
│     Non-preemptive: No     Preemptive: Yes                      │
│                                                                  │
│  4. Process TERMINATES                                           │
│     Non-preemptive: Yes    Preemptive: Yes                      │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘

Points 1 and 4: Scheduling MUST occur (no choice)
Points 2 and 3: Scheduling MAY occur (preemptive only)
```

---

## Process Behavior Model

```
                    ┌─────────────────────────────────────────┐
                    │          PROCESS LIFECYCLE               │
                    └─────────────────────────────────────────┘

    ┌─────────┐     Admitted       ┌─────────┐
    │   New   │ ─────────────────→ │  Ready  │ ←───────────────┐
    └─────────┘                    └────┬────┘                 │
                                        │                      │
                                        │ Scheduler            │
                                        │ Dispatch             │
                                        ↓                      │
                                   ┌─────────┐                 │
                              ┌────│ Running │────┐            │
                              │    └────┬────┘    │            │
                              │         │         │            │
                    I/O or    │    Exit │    Interrupt/        │
                    Event Wait│         │    Time Quantum      │
                              ↓         ↓    Expired           │
                         ┌─────────┐ ┌──────────┐             │
                         │ Waiting │ │Terminated│             │
                         └────┬────┘ └──────────┘             │
                              │                                │
                              │ I/O or Event Completion        │
                              └────────────────────────────────┘
```

---

## Summary

| Concept | Description |
|---------|-------------|
| **Scheduling** | Selecting which process runs on CPU |
| **Long-term Scheduler** | Controls admission to memory |
| **Short-term Scheduler** | Selects next process to run |
| **Medium-term Scheduler** | Swaps processes in/out of memory |
| **Dispatcher** | Actually switches context |
| **Preemptive** | Can forcibly take CPU from process |
| **Non-preemptive** | Process keeps CPU until it gives up |

### Key Metrics

| Metric | Meaning | Goal |
|--------|---------|------|
| CPU Utilization | How busy is CPU | Maximize |
| Throughput | Processes completed | Maximize |
| Turnaround Time | Total process time | Minimize |
| Waiting Time | Time in ready queue | Minimize |
| Response Time | Time to first response | Minimize |

---

## Quick Revision Questions

1. What are the three types of schedulers?
2. Differentiate between preemptive and non-preemptive scheduling.
3. What is the role of the dispatcher?
4. Define turnaround time, waiting time, and response time.
5. Why is a good mix of I/O-bound and CPU-bound processes important?

---

[← Previous: Process Control Block](./02-process-control-block.md) | [Next: CPU Scheduling Algorithms →](./04-cpu-scheduling-algorithms.md)
