# Chapter 7.3: Disk Scheduling

## Overview

When multiple I/O requests are pending for a disk, the operating system must decide in which order to service them. Disk scheduling algorithms optimize the order to minimize seek time and maximize throughput.

---

## Disk Access Time Components

```
┌──────────────────────────────────────────────────────────────────┐
│              DISK ACCESS TIME COMPONENTS                          │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Total Access Time = Seek Time + Rotational Latency + Transfer  │
│                                                                  │
│  1. SEEK TIME (Biggest factor!)                                  │
│     • Time to move disk head to desired track                   │
│     • Depends on distance traveled                              │
│     • Typically 5-15 ms (average)                               │
│                                                                  │
│  2. ROTATIONAL LATENCY                                           │
│     • Time for desired sector to rotate under head              │
│     • Average = half rotation time                              │
│     • At 7200 RPM: avg = 4.17 ms                               │
│                                                                  │
│  3. TRANSFER TIME                                                │
│     • Time to read/write the data                               │
│     • Depends on data size and disk speed                      │
│     • Usually small compared to seek + rotation                 │
│                                                                  │
│  Example:                                                        │
│  ┌─────────────────────────────────────────────────────────────┐│
│  │ Seek time:       8 ms                                       ││
│  │ Rotational:      4 ms (half rotation at 7200 RPM)          ││
│  │ Transfer:        0.5 ms (8 KB at 16 MB/s)                  ││
│  │ ─────────────────────                                       ││
│  │ Total:           12.5 ms per I/O                           ││
│  └─────────────────────────────────────────────────────────────┘│
│                                                                  │
│  Goal: Minimize SEEK TIME (the largest component)               │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

---

## Disk Scheduling Scenario

```
For all examples, we'll use:

Disk: 200 cylinders (0-199)
Head starts at: cylinder 50
Request queue: 98, 183, 37, 122, 14, 124, 65, 67

┌───────────────────────────────────────────────────────────────────┐
│  0    14   37   50   65 67   98   122 124   183              199 │
│  │     │    │    ▲    │  │    │     │   │     │                │ │
│  │     │    │    │    │  │    │     │   │     │                │ │
│  │     │    │    │    │  │    │     │   │     │                │ │
│  └─────┴────┴────┴────┴──┴────┴─────┴───┴─────┴────────────────┘ │
│           Head                                                    │
│          starts                                                   │
│          here (50)                                                │
└───────────────────────────────────────────────────────────────────┘
```

---

## 1. FCFS (First-Come, First-Served)

```
┌──────────────────────────────────────────────────────────────────┐
│                      FCFS SCHEDULING                              │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Process requests in the order they arrive                       │
│  Simple but not efficient                                        │
│                                                                  │
│  Queue: 98, 183, 37, 122, 14, 124, 65, 67                       │
│  Head: 50                                                        │
│                                                                  │
│  Order: 50 → 98 → 183 → 37 → 122 → 14 → 124 → 65 → 67          │
│                                                                  │
│  Movement visualization:                                         │
│  0        50       100       150       200                       │
│  │         │         │         │         │                       │
│  │    ┌────┘         │         │         │                       │
│  │    │    ┌─────────┴─────────┴────┐    │  50→98 (48)          │
│  │    │    │                        └────┤  98→183 (85)         │
│  │    │    └────────────────────────┐    │  183→37 (146)        │
│  │    │    ┌────────────────────────┘    │  37→122 (85)         │
│  │    └────┘                             │  122→14 (108)        │
│  │    ┌────────────────────────────┐     │  14→124 (110)        │
│  │    │    ┌───────────────────────┘     │  124→65 (59)         │
│  │    │    │                             │  65→67 (2)           │
│  │    └────┘                             │                       │
│                                                                  │
│  Total head movement: 48+85+146+85+108+110+59+2 = 643 cylinders │
│                                                                  │
│  Advantages:                                                     │
│  ✓ Fair (no starvation)                                         │
│  ✓ Simple to implement                                          │
│                                                                  │
│  Disadvantages:                                                  │
│  ✗ High seek time                                               │
│  ✗ Lots of back-and-forth movement                              │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

---

## 2. SSTF (Shortest Seek Time First)

```
┌──────────────────────────────────────────────────────────────────┐
│                      SSTF SCHEDULING                              │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Select request closest to current head position                 │
│  Like SJF for disk scheduling                                    │
│                                                                  │
│  Queue: 98, 183, 37, 122, 14, 124, 65, 67                       │
│  Head: 50                                                        │
│                                                                  │
│  Step by step:                                                   │
│  At 50: closest is 65 (dist=15) or 37 (dist=13) → choose 37    │
│  At 37: closest is 14 (dist=23) → choose 14                    │
│  At 14: closest is 65 (dist=51) → choose 65                    │
│  At 65: closest is 67 (dist=2) → choose 67                     │
│  At 67: closest is 98 (dist=31) → choose 98                    │
│  At 98: closest is 122 (dist=24) → choose 122                  │
│  At 122: closest is 124 (dist=2) → choose 124                  │
│  At 124: only 183 left → choose 183                             │
│                                                                  │
│  Order: 50 → 37 → 14 → 65 → 67 → 98 → 122 → 124 → 183          │
│                                                                  │
│  Movement: 13 + 23 + 51 + 2 + 31 + 24 + 2 + 59 = 205 cylinders │
│                                                                  │
│  Visualization:                                                  │
│  0    14   37   50   65 67   98   122 124   183                 │
│  │     │    │    │    │  │    │     │   │     │                 │
│  │     │    │◄───┘    │  │    │     │   │     │  50→37          │
│  │     │◄───┘         │  │    │     │   │     │  37→14          │
│  │     └──────────────┼──┼────┼─────┼───┼─────┤                  │
│  │                    │◄─┘    │     │   │     │  14→65          │
│  │                    └──►    │     │   │     │  65→67          │
│  │                       └────►     │   │     │  67→98          │
│  │                            └─────►   │     │  98→122         │
│  │                                  └───►     │  122→124        │
│  │                                      └─────►  124→183        │
│                                                                  │
│  Total: 205 cylinders (much better than FCFS's 643!)            │
│                                                                  │
│  Advantages:                                                     │
│  ✓ Much lower seek time than FCFS                               │
│  ✓ Higher throughput                                            │
│                                                                  │
│  Disadvantages:                                                  │
│  ✗ Starvation possible (far requests may wait forever)         │
│  ✗ Not optimal (may not be minimum total)                       │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

---

## 3. SCAN (Elevator Algorithm)

```
┌──────────────────────────────────────────────────────────────────┐
│                    SCAN SCHEDULING                                │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Head moves in one direction, servicing all requests            │
│  At the end, reverses direction (like an elevator)              │
│                                                                  │
│  Queue: 98, 183, 37, 122, 14, 124, 65, 67                       │
│  Head: 50, moving toward 0 (left)                               │
│                                                                  │
│  Order: 50 → 37 → 14 → 0 → 65 → 67 → 98 → 122 → 124 → 183      │
│                                                                  │
│  Visualization:                                                  │
│  0    14   37   50   65 67   98   122 124   183              199 │
│  │     │    │    │    │  │    │     │   │     │                │ │
│  │     │    │◄───┘    │  │    │     │   │     │ ← going left   │ │
│  │     │◄───┘         │  │    │     │   │     │                │ │
│  ◄─────┘              │  │    │     │   │     │ hit 0         │ │
│  └────────────────────►  │    │     │   │     │ → going right │ │
│                       └──►    │     │   │     │                │ │
│                          └────►     │   │     │                │ │
│                               └─────►   │     │                │ │
│                                     └───►     │                │ │
│                                         └─────►                │ │
│                                                                  │
│  Movement: 50 + 183 = 233 cylinders                             │
│                                                                  │
│  (50→0=50, then 0→183=183)                                      │
│                                                                  │
│  Advantages:                                                     │
│  ✓ No starvation                                                │
│  ✓ Uniform wait time                                            │
│  ✓ Simple to implement                                          │
│                                                                  │
│  Disadvantages:                                                  │
│  ✗ Goes to end even if no requests there                       │
│  ✗ Requests at far end just missed must wait for full sweep    │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

---

## 4. C-SCAN (Circular SCAN)

```
┌──────────────────────────────────────────────────────────────────┐
│                   C-SCAN SCHEDULING                               │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Like SCAN, but only services in one direction                  │
│  At the end, returns to beginning without servicing             │
│  More uniform wait time                                          │
│                                                                  │
│  Queue: 98, 183, 37, 122, 14, 124, 65, 67                       │
│  Head: 50, moving toward 199 (right)                            │
│                                                                  │
│  Order: 50 → 65 → 67 → 98 → 122 → 124 → 183 → 199 → 0 → 14 → 37│
│                       ↑                     ↑___|              │
│                    service               jump back              │
│                                         (no service)            │
│                                                                  │
│  Visualization:                                                  │
│  0    14   37   50   65 67   98   122 124   183              199 │
│  │     │    │    │    │  │    │     │   │     │                │ │
│  │     │    │    └────►  │    │     │   │     │ →              │ │
│  │     │    │         └──►    │     │   │     │                │ │
│  │     │    │            └────►     │   │     │                │ │
│  │     │    │                 └─────►   │     │                │ │
│  │     │    │                       └───►     │                │ │
│  │     │    │                           └─────────────────────►│ │
│  │◄────────────────────────────────────────────────────────────┘ │
│  └─────►    │    (jump to 0, no servicing)                      │
│        └────►                                                    │
│                                                                  │
│  Movement: (199-50) + (199-0) + 37 = 149 + 199 + 37 = 385      │
│  (Note: Jump back doesn't count as seek in some implementations)│
│                                                                  │
│  Advantages:                                                     │
│  ✓ More uniform wait time than SCAN                            │
│  ✓ No starvation                                                │
│                                                                  │
│  Disadvantages:                                                  │
│  ✗ More total head movement than SCAN                          │
│  ✗ Returns empty (wasted movement)                              │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

---

## 5. LOOK and C-LOOK

```
┌──────────────────────────────────────────────────────────────────┐
│                  LOOK AND C-LOOK                                  │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  LOOK: Like SCAN but stops at last request (doesn't go to end) │
│  C-LOOK: Like C-SCAN but stops at last request                 │
│                                                                  │
│  LOOK Example:                                                   │
│  Head at 50, going left                                         │
│  Order: 50 → 37 → 14 → (stop, reverse) → 65 → 67 → 98 → ...   │
│                      ↑                                          │
│              Don't go to 0, no requests there                   │
│                                                                  │
│  C-LOOK Example:                                                 │
│  Queue: 98, 183, 37, 122, 14, 124, 65, 67                       │
│  Head: 50, going right                                          │
│                                                                  │
│  Order: 50 → 65 → 67 → 98 → 122 → 124 → 183 → (jump) → 14 → 37 │
│                                          ↑                      │
│              Stop at 183 (last request), not 199                │
│                                                                  │
│  0    14   37   50   65 67   98   122 124   183              199 │
│  │     │    │    │    │  │    │     │   │     │                │ │
│  │     │    │    └────►  │    │     │   │     │                │ │
│  │     │    │         └──►    │     │   │     │                │ │
│  │     │    │            └────►     │   │     │                │ │
│  │     │    │                 └─────►   │     │                │ │
│  │     │    │                       └───►     │                │ │
│  │     │◄───────────────────────────────┘     │                │ │
│  │     └────►  (jump to first request)        │                │ │
│                                                                  │
│  Movement: (183-50) + (183-14) + (37-14) = 133 + 169 + 23 = 325│
│                                                                  │
│  C-LOOK is generally the best practical choice!                  │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

---

## Algorithm Comparison

```
┌──────────────────────────────────────────────────────────────────┐
│                ALGORITHM COMPARISON                               │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  For: Requests 98,183,37,122,14,124,65,67; Head at 50           │
│                                                                  │
│  ┌─────────────┬──────────────────────┬──────────────────────┐  │
│  │ Algorithm   │ Head Movement        │ Characteristics      │  │
│  ├─────────────┼──────────────────────┼──────────────────────┤  │
│  │ FCFS        │ 643 cylinders        │ Fair but slow       │  │
│  │ SSTF        │ 205 cylinders        │ Fast but starvation │  │
│  │ SCAN        │ 233 cylinders        │ Uniform, no starve  │  │
│  │ C-SCAN      │ ~385 cylinders       │ Very uniform wait   │  │
│  │ LOOK        │ ~203 cylinders       │ Like SCAN, smarter  │  │
│  │ C-LOOK      │ ~206 cylinders       │ Best overall       │  │
│  └─────────────┴──────────────────────┴──────────────────────┘  │
│                                                                  │
│  Summary:                                                        │
│  • FCFS: Simple but poor performance                            │
│  • SSTF: Best seek time but can starve                          │
│  • SCAN: Good balance, goes to ends                             │
│  • C-SCAN: Most uniform wait time                               │
│  • LOOK/C-LOOK: Practical improvements to SCAN/C-SCAN           │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

---

## Disk Scheduling for SSDs

```
┌──────────────────────────────────────────────────────────────────┐
│                DISK SCHEDULING FOR SSDs                           │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  SSDs have NO seek time (no moving heads)!                      │
│                                                                  │
│  • All locations accessed equally fast                          │
│  • Disk scheduling algorithms less relevant                     │
│  • Focus shifts to:                                             │
│    - Write amplification                                        │
│    - Wear leveling                                              │
│    - Queue depth (parallel I/O)                                 │
│                                                                  │
│  For SSDs, simple FCFS or NOOP scheduler often works well      │
│                                                                  │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │ Modern approach for SSDs:                               │    │
│  │ • Multi-queue block layer                              │    │
│  │ • Each CPU core has its own I/O queue                  │    │
│  │ • Reduces contention, improves parallelism             │    │
│  └─────────────────────────────────────────────────────────┘    │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

---

## Summary

| Algorithm | Description | Seek Time | Starvation |
|-----------|-------------|-----------|------------|
| **FCFS** | Order of arrival | High | No |
| **SSTF** | Closest first | Low | Yes |
| **SCAN** | Elevator, go to ends | Medium | No |
| **C-SCAN** | One direction only | Medium | No |
| **LOOK** | Like SCAN, stop at last request | Low | No |
| **C-LOOK** | Like C-SCAN, stop at last request | Low | No |

---

## Quick Revision Questions

1. Why is seek time the most important factor to minimize?
2. How does SSTF cause starvation?
3. What is the difference between SCAN and LOOK?
4. Why is C-SCAN considered more fair than SCAN?
5. Why are disk scheduling algorithms less important for SSDs?

---

[← Previous: I/O Methods](./02-io-methods.md) | [Next: Protection →](../08-Security-Protection/01-protection.md)
