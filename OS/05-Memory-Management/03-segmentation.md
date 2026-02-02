# Chapter 5.3: Segmentation

## Overview

Segmentation is a memory management scheme that supports the user's view of memory. A program is viewed as a collection of segments (logical units like functions, arrays, objects) rather than a sequence of bytes.

---

## User's View of Memory

```
┌──────────────────────────────────────────────────────────────────┐
│                 USER'S VIEW OF A PROGRAM                          │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Programmers don't think of memory as a linear array            │
│  They think in terms of logical units:                          │
│                                                                  │
│  ┌─────────────────────────────────────────────────┐            │
│  │                    PROGRAM                       │            │
│  │                                                  │            │
│  │   ┌──────────────┐    ┌──────────────┐         │            │
│  │   │   main()     │    │   Sqrt()     │         │            │
│  │   │   function   │    │   function   │         │            │
│  │   └──────────────┘    └──────────────┘         │            │
│  │                                                  │            │
│  │   ┌──────────────┐    ┌──────────────┐         │            │
│  │   │   Stack      │    │    Symbol    │         │            │
│  │   │              │    │    Table     │         │            │
│  │   └──────────────┘    └──────────────┘         │            │
│  │                                                  │            │
│  │   ┌──────────────────────────────────┐         │            │
│  │   │         Array[1000]              │         │            │
│  │   └──────────────────────────────────┘         │            │
│  │                                                  │            │
│  └─────────────────────────────────────────────────┘            │
│                                                                  │
│  Each logical unit = One SEGMENT                                │
│  Segments can be of DIFFERENT sizes                             │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

---

## Segmentation Concept

### Definition

A program is a collection of segments. Each segment is a logical unit such as:
- Main program
- Procedure/Function
- Object
- Local variables
- Global variables
- Stack
- Symbol table
- Arrays

```
┌──────────────────────────────────────────────────────────────────┐
│                 SEGMENTED MEMORY                                  │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Logical Address Space:          Physical Memory:                │
│                                                                  │
│  ┌─────────────────────┐         ┌─────────────────┐            │
│  │ Segment 0 (Code)    │ ─────→  │                 │            │
│  │ (limit: 1000)       │         │   Segment 0     │            │
│  └─────────────────────┘         │   @ base 1400   │            │
│                                  ├─────────────────┤            │
│  ┌─────────────────────┐         │                 │            │
│  │ Segment 1 (Data)    │         │   Segment 2     │            │
│  │ (limit: 400)        │ ──┐     │   @ base 4300   │            │
│  └─────────────────────┘   │     ├─────────────────┤            │
│                            │     │                 │            │
│  ┌─────────────────────┐   │     │   (Free)        │            │
│  │ Segment 2 (Stack)   │   │     │                 │            │
│  │ (limit: 1100)       │ ──┼─┐   ├─────────────────┤            │
│  └─────────────────────┘   │ │   │   Segment 1     │            │
│                            │ │   │   @ base 6300   │            │
│  ┌─────────────────────┐   │ │   ├─────────────────┤            │
│  │ Segment 3 (Heap)    │   │ └─→ │                 │            │
│  │ (limit: 1000)       │ ──┼───→ │   Segment 3     │            │
│  └─────────────────────┘   │     │   @ base 4700   │            │
│                            │     └─────────────────┘            │
│                            │                                     │
│                            └─────────────────────────────→      │
│                                                                  │
│  Note: Segments can be placed anywhere in physical memory       │
│        and are of varying sizes                                  │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

---

## Logical Address in Segmentation

```
┌──────────────────────────────────────────────────────────────────┐
│               LOGICAL ADDRESS STRUCTURE                           │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Logical Address = <segment-number, offset>                      │
│                                                                  │
│  ┌─────────────────────┬────────────────────────────┐           │
│  │  Segment Number (s) │       Offset (d)           │           │
│  └─────────────────────┴────────────────────────────┘           │
│                                                                  │
│  Examples:                                                       │
│  • Address in main function: <0, 100>                           │
│  • 5th element of array[]: <3, 20>  (if segment 3 is array)    │
│  • Local variable: <1, 8>                                       │
│                                                                  │
│  Note: Unlike paging, segment numbers have meaning to user      │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

---

## Segment Table

### Structure

```
┌──────────────────────────────────────────────────────────────────┐
│                    SEGMENT TABLE                                  │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Each entry contains:                                            │
│  • Base: Starting physical address of segment                   │
│  • Limit: Length of segment                                     │
│                                                                  │
│  ┌─────────────────────────────────────────────────┐            │
│  │  Segment Table                                   │            │
│  ├─────────┬─────────────────┬─────────────────────┤            │
│  │  Index  │      Base       │        Limit        │            │
│  ├─────────┼─────────────────┼─────────────────────┤            │
│  │    0    │      1400       │         1000        │            │
│  │    1    │      6300       │          400        │            │
│  │    2    │      4300       │         1100        │            │
│  │    3    │      4700       │         1000        │            │
│  └─────────┴─────────────────┴─────────────────────┘            │
│                                                                  │
│  Special Registers:                                              │
│  • STBR (Segment Table Base Register): Points to table         │
│  • STLR (Segment Table Length Register): Number of segments    │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

---

## Address Translation

### Hardware Support

```
┌──────────────────────────────────────────────────────────────────┐
│              SEGMENTATION HARDWARE                                │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│      CPU                                                         │
│       │                                                         │
│       ▼                                                         │
│  ┌─────────┬──────────┐                                         │
│  │  s      │    d     │  Logical Address                        │
│  └────┬────┴────┬─────┘                                         │
│       │         │                                                │
│       │         │                                                │
│       ▼         │                                                │
│  ┌────────────────────┐                                         │
│  │ s < STLR ?        │                                         │
│  └─────────┬──────────┘                                         │
│       yes  │  no → TRAP (invalid segment)                       │
│            ▼                                                     │
│  ┌─────────────────────────┐                                    │
│  │  Segment Table          │                                    │
│  │  ┌───────┬───────────┐  │                                    │
│  │  │ base  │   limit   │  │ ← Entry s                         │
│  │  └───┬───┴─────┬─────┘  │                                    │
│  └──────┼─────────┼────────┘                                    │
│         │         │                                              │
│         │         ▼                                              │
│         │    ┌──────────────┐                                   │
│         │    │  d < limit ? │                                   │
│         │    └──────┬───────┘                                   │
│         │      yes  │  no → TRAP (segment overflow)             │
│         │           │                                            │
│         ▼           ▼                                            │
│    ┌─────────────────────┐                                      │
│    │   base + d          │  Physical Address                    │
│    └─────────────────────┘                                      │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

### Example

```
Logical Address: <2, 400>
Segment Table:
  Segment 0: base=1400, limit=1000
  Segment 1: base=6300, limit=400
  Segment 2: base=4300, limit=1100
  Segment 3: base=4700, limit=1000

Step 1: Check segment number valid
  2 < 4 (number of segments)? YES ✓

Step 2: Get segment entry
  Segment 2: base=4300, limit=1100

Step 3: Check offset within limit
  400 < 1100? YES ✓

Step 4: Calculate physical address
  Physical = base + offset = 4300 + 400 = 4700

Result: Logical <2, 400> → Physical 4700
```

---

## Protection and Sharing

### Protection

```
┌──────────────────────────────────────────────────────────────────┐
│              SEGMENT PROTECTION                                   │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Each segment can have protection bits:                         │
│                                                                  │
│  ┌─────────┬────────┬───────┬────────────┬────────────┐         │
│  │ Segment │  Read  │ Write │  Execute   │   Valid    │         │
│  ├─────────┼────────┼───────┼────────────┼────────────┤         │
│  │  Code   │   1    │   0   │     1      │     1      │         │
│  │  Data   │   1    │   1   │     0      │     1      │         │
│  │  Stack  │   1    │   1   │     0      │     1      │         │
│  └─────────┴────────┴───────┴────────────┴────────────┘         │
│                                                                  │
│  Advantages:                                                     │
│  • Natural protection boundaries                                │
│  • Code segment: read + execute (no write = no self-modifying) │
│  • Data segment: read + write (no execute = prevent exploits)  │
│                                                                  │
│  Segment-level protection is more logical than page-level       │
│  because segments correspond to logical program units           │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

### Sharing

```
┌──────────────────────────────────────────────────────────────────┐
│                 SEGMENT SHARING                                   │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Segments can be shared between processes                       │
│                                                                  │
│  Process P1                      Process P2                      │
│  Segment Table                   Segment Table                   │
│  ┌─────┬───────┬───────┐        ┌─────┬───────┬───────┐        │
│  │ Seg │ Base  │ Limit │        │ Seg │ Base  │ Limit │        │
│  ├─────┼───────┼───────┤        ├─────┼───────┼───────┤        │
│  │  0  │ 25000 │ 10000 │──┐     │  0  │ 25000 │ 10000 │──┐     │
│  │  1  │ 4000  │ 2000  │  │ ┌───│  1  │ 10000 │ 1500  │  │     │
│  └─────┴───────┴───────┘  │ │   └─────┴───────┴───────┘  │     │
│                           │ │                             │     │
│                           ▼ │                             ▼     │
│                    ┌─────────┴─────────────────────────────┐   │
│                    │              SHARED                    │   │
│                    │              EDITOR                    │   │
│                    │           (Read-only)                  │   │
│                    │         @ Address 25000                │   │
│                    └────────────────────────────────────────┘   │
│                                                                  │
│  Shared segment MUST have same segment number in all processes  │
│  (because code references segment number directly)              │
│                                                                  │
│  Wait... Actually NO! Segments can have different numbers       │
│  as long as base points to same physical location              │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

---

## Paging vs Segmentation

### Comparison

| Aspect | Paging | Segmentation |
|--------|--------|--------------|
| **Division** | Fixed-size pages | Variable-size segments |
| **Visibility to user** | Invisible | Visible (logical units) |
| **Address** | Single (page, offset) | Two-part (segment, offset) |
| **Fragmentation** | Internal only | External only |
| **Sharing** | Less intuitive | Logical sharing of modules |
| **Protection** | Per page | Per segment (more logical) |
| **Table size** | Can be large | Smaller (fewer segments) |

### Visual Comparison

```
              PAGING                          SEGMENTATION

  Logical Space:                    Logical Space:
  ┌────┬────┬────┬────┐            ┌──────────────────────┐
  │ P0 │ P1 │ P2 │ P3 │            │     Segment 0        │
  └────┴────┴────┴────┘            │     (Code)           │
  All same size                     ├──────────────────────┤
                                    │  Segment 1 (Data)    │
                                    ├──────────────────────┤
                                    │Seg 2│                │
                                    └─────┴────────────────┘
                                    Different sizes

  Physical Memory:                  Physical Memory:
  ┌────┐                           ┌───────────────────┐
  │ P2 │                           │    Segment 2      │
  ├────┤                           ├───────────────────┤
  │ P0 │                           │                   │
  ├────┤                           │    Segment 0      │
  │    │ Free                      │                   │
  ├────┤                           ├───────────────────┤
  │ P3 │                           │                   │ Free
  ├────┤                           │                   │
  │ P1 │                           ├───────────────────┤
  └────┘                           │   Segment 1       │
  No external fragmentation        └───────────────────┘
                                   Has external fragmentation
```

---

## Segmentation with Paging

### Combining Both

```
┌──────────────────────────────────────────────────────────────────┐
│             SEGMENTATION WITH PAGING                              │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Idea: Use segmentation for logical division                    │
│        Use paging to eliminate external fragmentation           │
│                                                                  │
│  Each segment is divided into pages                             │
│                                                                  │
│  Logical Address:                                                │
│  ┌──────────┬──────────────┬─────────────┐                      │
│  │ Segment# │ Page Number  │ Page Offset │                      │
│  └────┬─────┴───────┬──────┴─────────────┘                      │
│       │             │                                            │
│       ▼             │                                            │
│  ┌──────────────┐   │                                           │
│  │Segment Table │   │                                           │
│  │ ┌──────────┐ │   │                                           │
│  │ │PT Base   │─┼───┼──────────────┐                           │
│  │ │PT Length │ │   │              │                           │
│  │ └──────────┘ │   │              │                           │
│  └──────────────┘   │              ▼                           │
│                     │      ┌──────────────┐                    │
│                     └─────→│ Page Table   │                    │
│                            │ for segment  │                    │
│                            ├──────────────┤                    │
│                            │ Frame #      │──→ Physical       │
│                            └──────────────┘    Address         │
│                                                                  │
│  Used in: Intel x86 (historical), MULTICS                      │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

### Intel x86 Example (Protected Mode)

```
Intel uses segmentation with paging:

Logical Address
     │
     ▼
 Segmentation Unit
     │
     ▼
 Linear Address
     │
     ▼
  Paging Unit
     │
     ▼
Physical Address

Modern x86-64: Segmentation is mostly disabled (flat model)
              Only paging is used
```

---

## Summary

| Concept | Description |
|---------|-------------|
| **Segment** | Logical unit of varying size |
| **Segment Table** | Maps segment to base + limit |
| **Address** | <segment-number, offset> |
| **Protection** | Per-segment read/write/execute |
| **Sharing** | Share logical segments |
| **Fragmentation** | External (between segments) |

### When to Use

| Use Segmentation When | Use Paging When |
|----------------------|-----------------|
| Logical structure matters | Simple allocation needed |
| Sharing code/data modules | Avoid external fragmentation |
| Different protection per module | Uniform memory management |

---

## Quick Revision Questions

1. How does segmentation reflect the user's view of memory?
2. What are the two components of a logical address in segmentation?
3. How does the hardware check for segment overflow?
4. Compare protection in paging vs segmentation.
5. How can segmentation and paging be combined?

---

[← Previous: Paging](./02-paging.md) | [Next: Virtual Memory →](./04-virtual-memory.md)
