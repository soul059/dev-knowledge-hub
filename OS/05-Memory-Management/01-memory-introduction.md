# Chapter 5.1: Memory Management Introduction

## Overview

Memory management is one of the most critical functions of an operating system. It involves managing the computer's primary memory (RAM), allocating space to processes, and ensuring efficient use of this limited resource.

---

## Why Memory Management?

```
┌──────────────────────────────────────────────────────────────────┐
│                 WHY MEMORY MANAGEMENT?                            │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Challenges:                                                     │
│                                                                  │
│  1. Memory is LIMITED                                            │
│     • RAM is expensive compared to disk                         │
│     • Programs often need more memory than available            │
│                                                                  │
│  2. Multiple programs need memory                                │
│     • Multiprogramming requires sharing memory                  │
│     • Each process needs its own protected space                │
│                                                                  │
│  3. Memory must be PROTECTED                                     │
│     • Process A should not access Process B's memory            │
│     • OS memory must be protected from user processes           │
│                                                                  │
│  4. Efficient allocation                                         │
│     • Minimize wasted (fragmented) memory                       │
│     • Fast allocation and deallocation                          │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

---

## Memory Hierarchy

```
┌──────────────────────────────────────────────────────────────────┐
│                    MEMORY HIERARCHY                               │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│                    ┌───────────┐                                │
│                    │ Registers │  Fastest, smallest, most       │
│                    └─────┬─────┘  expensive (CPU internal)      │
│                          │                                       │
│                    ┌─────▼─────┐                                │
│                    │   Cache   │  Very fast, small              │
│                    │  (L1,L2)  │  (KB to MB)                    │
│                    └─────┬─────┘                                │
│                          │                                       │
│              ┌───────────▼───────────┐                          │
│              │     Main Memory       │  Fast, medium size       │
│              │        (RAM)          │  (GB)                    │
│              └───────────┬───────────┘                          │
│                          │                                       │
│        ┌─────────────────▼─────────────────┐                    │
│        │        Secondary Storage          │  Slow, large       │
│        │      (SSD, Hard Disk)             │  (TB)              │
│        └─────────────────┬─────────────────┘                    │
│                          │                                       │
│  ┌───────────────────────▼───────────────────────┐              │
│  │              Tertiary Storage                 │  Very slow   │
│  │          (Tape, Cloud Storage)                │  Huge (PB)   │
│  └───────────────────────────────────────────────┘              │
│                                                                  │
│  Speed ↑                                    Size ↓              │
│  Cost ↑                                     Cost ↓              │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

### Comparison

| Level | Speed | Size | Cost | Managed By |
|-------|-------|------|------|------------|
| **Registers** | ~1 ns | ~KB | $$$$$ | Compiler |
| **Cache** | ~10 ns | ~MB | $$$$ | Hardware |
| **RAM** | ~100 ns | ~GB | $$$ | OS |
| **SSD** | ~100 μs | ~TB | $$ | OS |
| **HDD** | ~10 ms | ~TB | $ | OS |

---

## Basic Concepts

### Address Binding

The process of mapping program addresses to physical memory addresses.

```
┌──────────────────────────────────────────────────────────────────┐
│                    ADDRESS BINDING                                │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Source Code          Object Code           Memory               │
│  ┌────────────┐      ┌────────────┐      ┌────────────┐         │
│  │ int x;     │      │ LOAD 100   │      │ Address    │         │
│  │ x = 5;     │ ───→ │ STORE 104  │ ───→ │ 0x1000     │         │
│  │ y = x + 1; │      │ ADD 108    │      │ 0x1004     │         │
│  └────────────┘      └────────────┘      └────────────┘         │
│                                                                  │
│  Symbolic        Relocatable          Physical/Absolute         │
│  Addresses       Addresses            Addresses                 │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

### Times When Binding Occurs

```
┌──────────────────────────────────────────────────────────────────┐
│                 ADDRESS BINDING TIMES                             │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  1. COMPILE TIME                                                 │
│     ─────────────                                               │
│     • Absolute addresses generated by compiler                  │
│     • Must know memory location in advance                      │
│     • If location changes, must recompile                       │
│     • Used in: MS-DOS .COM programs                             │
│                                                                  │
│  2. LOAD TIME                                                    │
│     ──────────                                                  │
│     • Relocatable code generated by compiler                    │
│     • Loader binds addresses when program loads                 │
│     • If location changes, just reload                          │
│     • Used in: Early systems                                    │
│                                                                  │
│  3. EXECUTION TIME (Run Time)                                    │
│     ────────────────────────────                                │
│     • Binding delayed until run time                            │
│     • Process can be moved in memory during execution           │
│     • Requires hardware support (MMU)                           │
│     • Used in: Modern operating systems                         │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

---

## Logical vs Physical Address

```
┌──────────────────────────────────────────────────────────────────┐
│             LOGICAL VS PHYSICAL ADDRESS                           │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  LOGICAL ADDRESS (Virtual Address)                               │
│  • Generated by CPU during execution                            │
│  • Address as seen by the process                               │
│  • Ranges from 0 to max for each process                        │
│                                                                  │
│  PHYSICAL ADDRESS                                                │
│  • Actual address in main memory                                │
│  • Address as seen by memory unit                               │
│  • Generated by MMU from logical address                        │
│                                                                  │
│                                                                  │
│     CPU                MMU              Memory                   │
│  ┌───────┐         ┌───────┐         ┌───────┐                 │
│  │Logical│  ────→  │       │  ────→  │Physical│                 │
│  │Address│         │Mapping│         │Address │                 │
│  │ 346   │         │       │         │ 14346  │                 │
│  └───────┘         └───────┘         └───────┘                 │
│                                                                  │
│  Logical Address Space: Set of all logical addresses            │
│  Physical Address Space: Set of all physical addresses          │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

---

## Memory Management Unit (MMU)

### Function

```
┌──────────────────────────────────────────────────────────────────┐
│              MEMORY MANAGEMENT UNIT (MMU)                         │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Hardware device that maps logical to physical addresses        │
│                                                                  │
│  Simple MMU: Base Register (Relocation Register)                │
│                                                                  │
│     Logical                                      Physical        │
│     Address                                      Address         │
│        │                                            │           │
│        ▼                                            │           │
│   ┌─────────┐                                       │           │
│   │   346   │                                       │           │
│   └────┬────┘                                       │           │
│        │                                            │           │
│        ▼                                            │           │
│   ┌─────────┐    ┌──────────────┐                   │           │
│   │    +    │ ←─ │ Base Register│ = 14000           │           │
│   └────┬────┘    └──────────────┘                   │           │
│        │                                            │           │
│        ▼                                            ▼           │
│   ┌─────────┐                               ┌─────────┐         │
│   │  14346  │ ─────────────────────────────→│  14346  │         │
│   └─────────┘                               └─────────┘         │
│                                                                  │
│  Physical Address = Logical Address + Base                      │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

### Protection with Limit Register

```
┌──────────────────────────────────────────────────────────────────┐
│                    MEMORY PROTECTION                              │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Two registers provide protection:                               │
│  • Base Register: Starting physical address                     │
│  • Limit Register: Size of the address space                    │
│                                                                  │
│                                                                  │
│  CPU generates                                                   │
│  logical address                                                 │
│        │                                                        │
│        ▼                                                        │
│   ┌─────────┐      ┌─────────────┐                             │
│   │   346   │ ───→ │    < ?      │ ←─ Limit Register (2000)    │
│   └─────────┘      └──────┬──────┘                             │
│                      yes/  \no                                  │
│                        /    \                                   │
│                       ▼      ▼                                  │
│                    ┌────┐  ┌─────────────────────┐              │
│                    │ +  │  │ TRAP: Address Error │              │
│                    └──┬─┘  │ (Segmentation Fault)│              │
│                       │    └─────────────────────┘              │
│                       │                                         │
│          Base ───────→│ (14000)                                │
│                       │                                         │
│                       ▼                                         │
│                   Physical                                      │
│                   Address                                       │
│                   (14346)                                       │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

---

## Memory Allocation Schemes

### Overview

```
┌──────────────────────────────────────────────────────────────────┐
│               MEMORY ALLOCATION SCHEMES                           │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │               MEMORY ALLOCATION                             │ │
│  └───────────────────────┬────────────────────────────────────┘ │
│                          │                                       │
│          ┌───────────────┴───────────────┐                      │
│          │                               │                       │
│          ▼                               ▼                       │
│  ┌───────────────────┐          ┌───────────────────┐          │
│  │   CONTIGUOUS      │          │  NON-CONTIGUOUS   │          │
│  │   ALLOCATION      │          │   ALLOCATION      │          │
│  └─────────┬─────────┘          └─────────┬─────────┘          │
│            │                              │                      │
│    ┌───────┴───────┐              ┌───────┴───────┐             │
│    │               │              │               │              │
│    ▼               ▼              ▼               ▼              │
│ ┌──────┐      ┌──────┐       ┌───────┐     ┌───────────┐       │
│ │Single│      │Multi-│       │Paging │     │Segmentation│       │
│ │Parti-│      │ple   │       │       │     │           │       │
│ │tion  │      │Parti-│       └───────┘     └───────────┘       │
│ └──────┘      │tions │                                          │
│               └──────┘                                          │
│                  │                                               │
│          ┌───────┴───────┐                                      │
│          │               │                                       │
│          ▼               ▼                                       │
│     ┌────────┐     ┌────────┐                                   │
│     │ Fixed  │     │Variable│                                   │
│     │ Size   │     │ Size   │                                   │
│     └────────┘     └────────┘                                   │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

---

## Contiguous Memory Allocation

### Single Partition Allocation

```
┌──────────────────────────────────────────────────────────────────┐
│              SINGLE PARTITION ALLOCATION                          │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Memory divided into two parts:                                  │
│  • One for OS (usually low memory)                              │
│  • One for user process                                         │
│                                                                  │
│  ┌─────────────────────────────┐  0                             │
│  │                             │                                │
│  │     Operating System        │                                │
│  │                             │                                │
│  ├─────────────────────────────┤  Fence                         │
│  │                             │                                │
│  │                             │                                │
│  │     User Process            │                                │
│  │                             │                                │
│  │                             │                                │
│  │                             │                                │
│  └─────────────────────────────┘  Max                           │
│                                                                  │
│  Limitation: Only ONE user process at a time                    │
│              (No multiprogramming)                              │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

### Multiple Partition Allocation

```
┌──────────────────────────────────────────────────────────────────┐
│            MULTIPLE PARTITION ALLOCATION                          │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  FIXED PARTITIONS              VARIABLE PARTITIONS              │
│  (Static)                      (Dynamic)                        │
│                                                                  │
│  ┌───────────────────┐        ┌───────────────────┐             │
│  │        OS         │        │        OS         │             │
│  ├───────────────────┤        ├───────────────────┤             │
│  │   Partition 1     │        │    Process A      │             │
│  │     (2 MB)        │        │     (1.5 MB)      │             │
│  ├───────────────────┤        ├───────────────────┤             │
│  │   Partition 2     │        │    Process B      │             │
│  │     (4 MB)        │        │     (3 MB)        │             │
│  ├───────────────────┤        ├───────────────────┤             │
│  │   Partition 3     │        │      HOLE         │             │
│  │     (6 MB)        │        │     (0.5 MB)      │             │
│  ├───────────────────┤        ├───────────────────┤             │
│  │   Partition 4     │        │    Process C      │             │
│  │     (4 MB)        │        │     (2 MB)        │             │
│  └───────────────────┘        └───────────────────┘             │
│                                                                  │
│  • Partitions fixed           • Partitions vary by              │
│    at boot time                 process size                    │
│  • Internal fragmentation     • External fragmentation          │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

---

## Fragmentation

### Types of Fragmentation

```
┌──────────────────────────────────────────────────────────────────┐
│                    FRAGMENTATION                                  │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  INTERNAL FRAGMENTATION                                          │
│  ─────────────────────                                          │
│  Wasted space INSIDE allocated blocks                           │
│                                                                  │
│  ┌─────────────────────────┐                                    │
│  │   Partition: 4 MB       │                                    │
│  │   ┌─────────────────┐   │                                    │
│  │   │ Process: 3.5 MB │   │                                    │
│  │   ├─────────────────┤   │                                    │
│  │   │ WASTED: 0.5 MB  │ ← Internal fragmentation              │
│  │   └─────────────────┘   │                                    │
│  └─────────────────────────┘                                    │
│                                                                  │
│  EXTERNAL FRAGMENTATION                                          │
│  ──────────────────────                                         │
│  Total free memory exists but is not contiguous                 │
│                                                                  │
│  ┌──────────────┐   Need: 2 MB contiguous                       │
│  │  Process A   │                                               │
│  ├──────────────┤                                               │
│  │  Hole 1 MB   │                                               │
│  ├──────────────┤   Total free: 2.5 MB                          │
│  │  Process B   │   But cannot allocate 2 MB!                   │
│  ├──────────────┤   (holes are 1 + 0.5 + 1 MB)                  │
│  │  Hole 0.5 MB │                                               │
│  ├──────────────┤                                               │
│  │  Process C   │                                               │
│  ├──────────────┤                                               │
│  │  Hole 1 MB   │                                               │
│  └──────────────┘                                               │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

### Solutions to External Fragmentation

| Solution | Description | Disadvantage |
|----------|-------------|--------------|
| **Compaction** | Move processes to create contiguous free space | Time-consuming, needs dynamic relocation |
| **Paging** | Allow non-contiguous allocation | Internal fragmentation (small) |
| **Segmentation** | Logical division of process | External fragmentation possible |

---

## Dynamic Storage Allocation Strategies

When allocating memory from a list of free holes:

### First Fit

```
Allocate the FIRST hole that is big enough

Holes: [10K] → [20K] → [15K] → [30K]
Request: 18K

Search from beginning, find first fit:
[10K] - too small
[20K] - fits! ✓ Allocate here

Result: Fast, may leave small holes at beginning
```

### Best Fit

```
Allocate the SMALLEST hole that is big enough

Holes: [10K] → [20K] → [15K] → [30K]
Request: 18K

Search all holes, find smallest sufficient:
[10K] - too small
[20K] - fits (waste 2K) ✓ BEST
[15K] - too small
[30K] - fits (waste 12K)

Result: Slower search, produces smallest leftover holes
```

### Worst Fit

```
Allocate the LARGEST hole

Holes: [10K] → [20K] → [15K] → [30K]
Request: 18K

Find largest hole:
[30K] - largest ✓ Allocate here

Result: Leaves larger remaining holes (might be useful)
```

### Comparison

| Strategy | Speed | Fragmentation | When to Use |
|----------|-------|---------------|-------------|
| **First Fit** | Fastest | Moderate | General purpose |
| **Best Fit** | Slow | Many tiny holes | When holes are scarce |
| **Worst Fit** | Slow | Large holes | Rarely used |

---

## Summary

| Concept | Description |
|---------|-------------|
| **Logical Address** | Generated by CPU, virtual address |
| **Physical Address** | Actual memory location |
| **MMU** | Translates logical to physical |
| **Base Register** | Starting address of process |
| **Limit Register** | Size of process space |
| **Internal Fragmentation** | Waste inside allocated block |
| **External Fragmentation** | Scattered free holes |

---

## Quick Revision Questions

1. What is the difference between logical and physical addresses?
2. When does address binding occur at execution time?
3. What is the role of the MMU?
4. Explain internal vs external fragmentation.
5. Compare First Fit, Best Fit, and Worst Fit strategies.

---

[← Previous: Deadlock Detection](../04-Deadlocks/04-deadlock-detection.md) | [Next: Paging →](./02-paging.md)
