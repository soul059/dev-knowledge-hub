# Chapter 5.6: Thrashing

## Overview

Thrashing is a condition where a computer's performance degrades severely due to excessive paging. The system spends more time swapping pages in and out of memory than executing actual work. Understanding thrashing is crucial for maintaining system stability.

---

## What is Thrashing?

```
┌──────────────────────────────────────────────────────────────────┐
│                    WHAT IS THRASHING?                             │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Definition: A process is thrashing if it spends more time      │
│              paging than executing.                              │
│                                                                  │
│  ┌────────────────────────────────────────────────────┐         │
│  │  Normal Execution:                                  │         │
│  │  CPU: ███████████████████ (productive work)        │         │
│  │  I/O: ██                  (occasional page faults)  │         │
│  └────────────────────────────────────────────────────┘         │
│                                                                  │
│  ┌────────────────────────────────────────────────────┐         │
│  │  Thrashing:                                         │         │
│  │  CPU: ██                  (little productive work)  │         │
│  │  I/O: ███████████████████ (constant paging)        │         │
│  └────────────────────────────────────────────────────┘         │
│                                                                  │
│  Symptoms:                                                       │
│  • Very high page fault rate                                    │
│  • Very low CPU utilization                                     │
│  • Disk constantly busy with paging I/O                        │
│  • System appears frozen or extremely slow                      │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

---

## Cause of Thrashing

```
┌──────────────────────────────────────────────────────────────────┐
│               CAUSE: OVER-COMMITTED MEMORY                        │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Scenario: System tries to run too many processes               │
│                                                                  │
│  1. Initially: Low CPU utilization                              │
│     └─ OS thinks: "Need more processes!"                       │
│                                                                  │
│  2. OS increases degree of multiprogramming                     │
│     └─ Starts more processes                                   │
│                                                                  │
│  3. Each process needs frames for its working set               │
│     └─ But total frames demanded > available frames            │
│                                                                  │
│  4. Processes start page faulting constantly                    │
│     └─ Each fault causes another process's page to be evicted  │
│                                                                  │
│  5. Evicted pages are immediately needed again                  │
│     └─ More page faults!                                       │
│                                                                  │
│  6. CPU utilization drops (processes waiting for I/O)           │
│     └─ OS thinks: "Need even more processes!"                  │
│                                                                  │
│  7. VICIOUS CYCLE - Thrashing!                                  │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

### The Thrashing Cycle

```
              ┌─────────────────────────────────────┐
              │   CPU utilization appears low      │
              └──────────────────┬──────────────────┘
                                 │
                                 ▼
              ┌─────────────────────────────────────┐
              │   OS starts more processes          │
              └──────────────────┬──────────────────┘
                                 │
                                 ▼
              ┌─────────────────────────────────────┐
              │   Each process gets fewer frames    │
              └──────────────────┬──────────────────┘
                                 │
                                 ▼
              ┌─────────────────────────────────────┐
              │   Page fault rate increases         │◄────────┐
              └──────────────────┬──────────────────┘         │
                                 │                            │
                                 ▼                            │
              ┌─────────────────────────────────────┐         │
              │   Processes spend time waiting      │         │
              │   for paging I/O                    │         │
              └──────────────────┬──────────────────┘         │
                                 │                            │
                                 ▼                            │
              ┌─────────────────────────────────────┐         │
              │   CPU utilization drops further     │─────────┘
              └─────────────────────────────────────┘
                      THRASHING CYCLE!
```

---

## CPU Utilization vs Multiprogramming

```
┌──────────────────────────────────────────────────────────────────┐
│        CPU UTILIZATION vs DEGREE OF MULTIPROGRAMMING             │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  CPU                                                             │
│  Utilization                                                     │
│  100% ┤                                                          │
│       │            ●●●●●●●●                                     │
│       │         ●●●        ●●                                   │
│       │       ●●             ●                                  │
│       │      ●                ●                                 │
│       │     ●                  ●●                               │
│       │    ●                     ●● Thrashing                   │
│       │   ●                        ●● begins                    │
│       │  ●                           ●●●                        │
│       │ ●                               ●●●●●●●●●●              │
│    0% ┼─────────────────────────────────────────────→           │
│       │                                                          │
│            Degree of Multiprogramming                            │
│       ←──── Optimal ────→                                        │
│                                                                  │
│  Key observations:                                               │
│  • Initially: More processes = Better CPU utilization           │
│  • Peak: Optimal multiprogramming level                         │
│  • After peak: Adding more processes HURTS performance          │
│  • Thrashing: CPU utilization collapses                         │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

---

## Locality of Reference

```
┌──────────────────────────────────────────────────────────────────┐
│                  LOCALITY OF REFERENCE                            │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Programs don't access memory randomly                           │
│  They exhibit LOCALITY                                           │
│                                                                  │
│  TEMPORAL LOCALITY:                                              │
│  • If a location is accessed, it will likely be accessed again  │
│  • Example: Loop variables, frequently called functions         │
│                                                                  │
│  SPATIAL LOCALITY:                                               │
│  • If a location is accessed, nearby locations will be accessed │
│  • Example: Array traversal, sequential code execution          │
│                                                                  │
│  Memory Access Pattern:                                          │
│  ┌───────────────────────────────────────────────────────┐      │
│  │  Time →                                                │      │
│  │  Page 5: ████     ██                                  │      │
│  │  Page 3:     ████████████                             │      │
│  │  Page 7:                  ██████████                  │      │
│  │  Page 2:                            ████████          │      │
│  │                                                        │      │
│  │  Program moves through "localities" over time         │      │
│  └───────────────────────────────────────────────────────┘      │
│                                                                  │
│  Key Insight: At any time, only a subset of pages are needed   │
│               This subset is the WORKING SET                    │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

---

## Working Set Model

### Concept

```
┌──────────────────────────────────────────────────────────────────┐
│                    WORKING SET MODEL                              │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Working Set WS(t, Δ):                                          │
│  • Set of pages referenced in the most recent Δ page references │
│  • Δ = working-set window                                       │
│                                                                  │
│  Example:                                                        │
│  Reference string: ...2,6,1,5,7,7,7,7,5,1,6,2,3,4,1,2,3...     │
│                        ←───── Δ = 10 ─────→                     │
│                                                                  │
│  If Δ = 10 and current position is at last reference:           │
│  WS = {1, 2, 3, 4, 5, 6, 7}                                     │
│  |WS| = 7 (working set size)                                    │
│                                                                  │
│  Working Set Size (WSS) varies over time:                       │
│  ┌────────────────────────────────────────────────────┐         │
│  │ WSS                                                 │         │
│  │   │    ●●●●                                        │         │
│  │   │  ●●    ●●      ●●●●●●                         │         │
│  │   │●●        ●●●●●●      ●●                       │         │
│  │   │                        ●●●●●●●●●              │         │
│  │   └──────────────────────────────────→ Time       │         │
│  │    Phase 1   Phase 2     Phase 3                   │         │
│  └────────────────────────────────────────────────────┘         │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

### Working Set Strategy

```
┌──────────────────────────────────────────────────────────────────┐
│               WORKING SET STRATEGY                                │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Goal: Allocate enough frames for each process's working set    │
│                                                                  │
│  Let D = Σ WSSᵢ (total frames demanded by all processes)        │
│  Let m = total available frames                                  │
│                                                                  │
│  If D > m:                                                       │
│  • Thrashing will occur                                         │
│  • Must suspend one or more processes                           │
│                                                                  │
│  If D ≤ m:                                                       │
│  • System is safe from thrashing                                │
│  • Can possibly add more processes                              │
│                                                                  │
│  ┌────────────────────────────────────────────────────┐         │
│  │  Process    WSS (frames needed)                    │         │
│  │  P1         10                                     │         │
│  │  P2         15                                     │         │
│  │  P3         8                                      │         │
│  │  P4         12                                     │         │
│  │  ─────────────────────────                         │         │
│  │  Total D =  45 frames                              │         │
│  │                                                     │         │
│  │  Available m = 50 frames                           │         │
│  │  D < m → Safe (can even add more processes)       │         │
│  │                                                     │         │
│  │  Available m = 40 frames                           │         │
│  │  D > m → Must suspend a process!                  │         │
│  └────────────────────────────────────────────────────┘         │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

### Choosing Δ (Window Size)

```
┌──────────────────────────────────────────────────────────────────┐
│                CHOOSING Δ (WINDOW SIZE)                           │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Δ too small:                                                    │
│  • Working set won't encompass entire locality                  │
│  • Process may not have all needed pages                        │
│  • More page faults                                             │
│                                                                  │
│  Δ too large:                                                    │
│  • Working set includes multiple localities                     │
│  • Overly large allocation                                      │
│  • Wastes memory                                                │
│                                                                  │
│  Δ = infinity:                                                   │
│  • Working set = all pages ever touched                         │
│  • Maximum memory usage                                         │
│                                                                  │
│  Typical values: 10,000 to 1,000,000 references                 │
│                                                                  │
│  ┌────────────────────────────────────────────────────┐         │
│  │     Δ                                              │         │
│  │     ↑                                              │         │
│  │     │  Too Large: Wastes memory                   │         │
│  │     │  ──────────────────────────                 │         │
│  │     │  OPTIMAL: Covers current locality           │         │
│  │     │  ──────────────────────────                 │         │
│  │     │  Too Small: Page faults                     │         │
│  │     └───────────────────────────→                 │         │
│  └────────────────────────────────────────────────────┘         │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

---

## Page Fault Frequency (PFF)

```
┌──────────────────────────────────────────────────────────────────┐
│               PAGE FAULT FREQUENCY (PFF)                          │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  More direct approach than working set                          │
│  Measure page fault rate and adjust frames accordingly          │
│                                                                  │
│  Page Fault                                                      │
│  Rate                                                            │
│    │                                                             │
│    │ ●                                                          │
│    │  ●                                                         │
│    │   ●●   Upper threshold                                     │
│    ├───────────●─────────────  (too many faults)                │
│    │             ●●                                             │
│    │               ●●●                                          │
│    │                  ●●●●                                      │
│    ├──────────────────────●●───  Lower threshold                │
│    │                        ●●●● (too few faults)               │
│    └───────────────────────────────────→                        │
│              Number of Frames                                    │
│                                                                  │
│  Strategy:                                                       │
│  • If fault rate > upper threshold: Give more frames           │
│  • If fault rate < lower threshold: Take away frames           │
│  • If no frames available: Suspend the process                 │
│                                                                  │
│  Simpler to implement than working set                          │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

---

## Solutions to Thrashing

```
┌──────────────────────────────────────────────────────────────────┐
│                SOLUTIONS TO THRASHING                             │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  1. WORKING SET MODEL                                            │
│     • Track working set of each process                         │
│     • Only run processes if Σ WSS ≤ available frames           │
│     • Suspend processes if necessary                            │
│                                                                  │
│  2. PAGE FAULT FREQUENCY                                         │
│     • Monitor page fault rate per process                       │
│     • Adjust allocation based on thresholds                     │
│     • Simpler than working set                                  │
│                                                                  │
│  3. REDUCE MULTIPROGRAMMING                                      │
│     • If thrashing detected, swap out entire processes         │
│     • Give remaining processes more frames                      │
│     • Run fewer processes but faster                            │
│                                                                  │
│  4. ADD MORE RAM                                                 │
│     • Physical solution                                         │
│     • More frames available for all processes                   │
│                                                                  │
│  5. BETTER PAGE REPLACEMENT                                      │
│     • Use local rather than global replacement                 │
│     • Prevents one thrashing process from affecting others      │
│                                                                  │
│  6. PREPAGING                                                    │
│     • Bring in pages before they're needed                      │
│     • Based on working set model                                │
│     • Reduces initial page faults                               │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

---

## Local vs Global Replacement

```
┌──────────────────────────────────────────────────────────────────┐
│              LOCAL vs GLOBAL REPLACEMENT                          │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  GLOBAL REPLACEMENT:                                             │
│  • Select victim from all frames in system                      │
│  • One process can take frames from another                     │
│  • More flexible allocation                                     │
│  • But: One thrashing process can affect all others            │
│                                                                  │
│  ┌──────────────────────────────────────────────┐               │
│  │  P1: [A][B][C]    P2: [X][Y][Z]              │               │
│  │                                               │               │
│  │  P1 needs page D, replaces Y (from P2!)      │               │
│  │  P1: [A][B][D]    P2: [X][ ][Z]              │               │
│  │                         ↑                     │               │
│  │                     P2 hurts                  │               │
│  └──────────────────────────────────────────────┘               │
│                                                                  │
│  LOCAL REPLACEMENT:                                              │
│  • Select victim only from process's own frames                 │
│  • Process cannot affect others' performance                    │
│  • More predictable behavior                                    │
│  • But: May not use memory optimally                           │
│                                                                  │
│  ┌──────────────────────────────────────────────┐               │
│  │  P1: [A][B][C]    P2: [X][Y][Z]              │               │
│  │                                               │               │
│  │  P1 needs page D, can only replace A, B, or C│               │
│  │  P1: [A][B][D]    P2: [X][Y][Z] (unaffected) │               │
│  └──────────────────────────────────────────────┘               │
│                                                                  │
│  Recommendation: Use local replacement to isolate thrashing     │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

---

## Memory-Mapped Files

```
┌──────────────────────────────────────────────────────────────────┐
│                 MEMORY-MAPPED FILES                               │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Technique: Map file contents directly into virtual memory     │
│                                                                  │
│  Traditional I/O:                                                │
│  ┌──────────┐    read()    ┌──────────┐    ┌──────────┐        │
│  │   File   │ ───────────→ │  Buffer  │ ─→ │ Process  │        │
│  │  (disk)  │   write()    │ (kernel) │    │ (memory) │        │
│  └──────────┘    ←──────── └──────────┘ ←─ └──────────┘        │
│                   Two copies!                                    │
│                                                                  │
│  Memory-Mapped:                                                  │
│  ┌──────────┐                           ┌──────────┐            │
│  │   File   │ ────────────────────────→ │ Process  │            │
│  │  (disk)  │   mmap() maps directly    │ (memory) │            │
│  └──────────┘                           └──────────┘            │
│                 Zero-copy!                                       │
│                                                                  │
│  Benefits:                                                       │
│  • No system call overhead for each access                      │
│  • Uses demand paging automatically                             │
│  • Efficient for large files                                    │
│  • Enables file sharing between processes                       │
│                                                                  │
│  void *addr = mmap(NULL, length, PROT_READ,                     │
│                    MAP_PRIVATE, fd, 0);                         │
│  // Now access file as regular memory                           │
│  char c = addr[1000];  // May cause page fault                 │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

---

## Prepaging

```
┌──────────────────────────────────────────────────────────────────┐
│                      PREPAGING                                    │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Problem: Pure demand paging causes many initial page faults    │
│                                                                  │
│  Solution: Bring in pages before they're actually needed        │
│                                                                  │
│  When to prepage:                                                │
│  1. When process starts: Bring in first few pages              │
│  2. When process resumes: Bring in working set                 │
│  3. When locality detected: Bring in adjacent pages            │
│                                                                  │
│  Trade-off:                                                      │
│  • Prepaged pages may never be used (wasted I/O)               │
│  • But if used, saves page fault overhead                       │
│                                                                  │
│  Let s = pages prepaged                                          │
│  Let α = fraction actually used                                  │
│                                                                  │
│  Prepaging is beneficial if:                                     │
│  Cost(prepage s pages) < Cost(α × s demand faults)             │
│                                                                  │
│  Generally α > 0.5 for prepaging to be worthwhile               │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

---

## Real-World Example

```
┌──────────────────────────────────────────────────────────────────┐
│              REAL-WORLD THRASHING SCENARIO                        │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Scenario: Computer with 4 GB RAM                                │
│                                                                  │
│  Running applications:                                           │
│  • Chrome (2 GB with many tabs)                                 │
│  • IDE (1 GB)                                                   │
│  • Database (1.5 GB)                                            │
│  • Other apps (0.5 GB)                                          │
│  ─────────────────────                                          │
│  Total: 5 GB needed (but only 4 GB available!)                  │
│                                                                  │
│  What happens:                                                   │
│  1. Switch to Chrome: Page fault, swap in Chrome pages         │
│  2. Database query: Page fault, swap out Chrome pages          │
│  3. Chrome notification: Page fault, swap out DB pages         │
│  4. Repeat endlessly...                                         │
│                                                                  │
│  Symptoms you'd observe:                                         │
│  • Hard drive constantly active                                 │
│  • Applications take forever to respond                         │
│  • Task Manager shows low CPU but high disk usage              │
│  • System feels "frozen"                                        │
│                                                                  │
│  Solutions:                                                      │
│  • Close some applications                                      │
│  • Add more RAM                                                 │
│  • Use lighter alternatives (less memory-hungry apps)          │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

---

## Summary

| Concept | Description |
|---------|-------------|
| **Thrashing** | Excessive paging, more time swapping than computing |
| **Cause** | Too many processes, not enough frames per process |
| **Locality** | Programs access memory in clusters over time |
| **Working Set** | Set of pages in recent Δ references |
| **WSS** | Working Set Size - frames needed by process |
| **PFF** | Page Fault Frequency - adjust frames based on fault rate |
| **Global Replacement** | Victim from any process - can spread thrashing |
| **Local Replacement** | Victim from own frames - isolates problems |
| **Prepaging** | Load pages before needed |

---

## Quick Revision Questions

1. What is the relationship between CPU utilization and degree of multiprogramming during thrashing?
2. How does the working set model prevent thrashing?
3. Explain the difference between local and global page replacement.
4. What is Page Fault Frequency and how does it help?
5. How does locality of reference relate to the working set concept?

---

[← Previous: Page Replacement Algorithms](./05-page-replacement.md) | [Next: File Concepts →](../06-File-Systems/01-file-concepts.md)
