# Chapter 5.5: Page Replacement Algorithms

## Overview

When a page fault occurs and no free frames are available, the OS must select a victim page to replace. The choice of which page to evict significantly impacts system performance. This chapter covers the major page replacement algorithms in detail.

---

## Page Replacement Framework

```
┌──────────────────────────────────────────────────────────────────┐
│              PAGE REPLACEMENT PROCESS                             │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  1. Page fault occurs (page not in memory)                      │
│  2. Check if free frame exists                                   │
│     └─ If yes: Use it, skip to step 5                           │
│  3. Select victim page using replacement algorithm               │
│  4. If victim is dirty (modified):                               │
│     └─ Write victim page to disk (swap out)                     │
│  5. Read desired page into frame (swap in)                       │
│  6. Update page tables                                           │
│  7. Restart the instruction                                      │
│                                                                  │
│  ┌────────────────┐      ┌────────────────┐                     │
│  │  Find Victim   │──────│  Victim Dirty? │                     │
│  │   (Algorithm)  │      │    M bit = 1?  │                     │
│  └────────────────┘      └───────┬────────┘                     │
│                           Yes ↓    ↓ No                          │
│                   ┌────────────┐  ┌────────────┐                │
│                   │ Write to   │  │ Skip write │                │
│                   │ disk       │  │            │                │
│                   └────────────┘  └────────────┘                │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

---

## Reference String

```
Reference String: Sequence of page numbers referenced

Memory Reference:     0100, 0432, 0101, 0612, 0102, 0103, 0104...
Page Size: 100 bytes
Page Numbers:         1, 4, 1, 6, 1, 1, 1...

After removing consecutive duplicates:
Reference String:     1, 4, 1, 6, 1...

We'll use this reference string for examples:
7, 0, 1, 2, 0, 3, 0, 4, 2, 3, 0, 3, 2, 1, 2, 0, 1, 7, 0, 1

Length = 20 references
```

---

## 1. FIFO (First-In, First-Out)

### Concept

```
┌──────────────────────────────────────────────────────────────────┐
│                    FIFO ALGORITHM                                 │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Policy: Replace the page that has been in memory longest       │
│                                                                  │
│  Implementation: Queue of pages                                  │
│  • New page enters at rear                                      │
│  • Victim is taken from front                                   │
│                                                                  │
│  ┌────────────────────────────────────────────┐                 │
│  │  Front ← Remove     Queue     ← Add  Rear  │                 │
│  │   [7] ─ [0] ─ [1] ─ [2]                    │                 │
│  │   ↑                                        │                 │
│  │  Oldest (victim)                           │                 │
│  └────────────────────────────────────────────┘                 │
│                                                                  │
│  Pros: Simple to implement                                      │
│  Cons: May replace frequently used pages                        │
│        Suffers from Belady's anomaly                            │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

### Detailed Example

```
Reference String: 7, 0, 1, 2, 0, 3, 0, 4, 2, 3, 0, 3, 2, 1, 2, 0, 1, 7, 0, 1
Number of Frames: 3

Step | Ref | Frame 0 | Frame 1 | Frame 2 | Queue Order | Fault?
-----|-----|---------|---------|---------|-------------|-------
  1  |  7  |    7    |    -    |    -    |     7       |   F
  2  |  0  |    7    |    0    |    -    |    7,0      |   F
  3  |  1  |    7    |    0    |    1    |   7,0,1     |   F
  4  |  2  |    2    |    0    |    1    |   0,1,2     |   F (7 out)
  5  |  0  |    2    |    0    |    1    |   0,1,2     |   Hit
  6  |  3  |    2    |    3    |    1    |   1,2,3     |   F (0 out)
  7  |  0  |    2    |    3    |    0    |   2,3,0     |   F (1 out)
  8  |  4  |    4    |    3    |    0    |   3,0,4     |   F (2 out)
  9  |  2  |    4    |    2    |    0    |   0,4,2     |   F (3 out)
 10  |  3  |    4    |    2    |    3    |   4,2,3     |   F (0 out)
 11  |  0  |    0    |    2    |    3    |   2,3,0     |   F (4 out)
 12  |  3  |    0    |    2    |    3    |   2,3,0     |   Hit
 13  |  2  |    0    |    2    |    3    |   2,3,0     |   Hit
 14  |  1  |    0    |    1    |    3    |   3,0,1     |   F (2 out)
 15  |  2  |    0    |    1    |    2    |   0,1,2     |   F (3 out)
 16  |  0  |    0    |    1    |    2    |   0,1,2     |   Hit
 17  |  1  |    0    |    1    |    2    |   0,1,2     |   Hit
 18  |  7  |    7    |    1    |    2    |   1,2,7     |   F (0 out)
 19  |  0  |    7    |    0    |    2    |   2,7,0     |   F (1 out)
 20  |  1  |    7    |    0    |    1    |   7,0,1     |   F (2 out)

Total Page Faults: 15
Hit Ratio: 5/20 = 25%
```

### Belady's Anomaly

```
┌──────────────────────────────────────────────────────────────────┐
│                   BELADY'S ANOMALY                                │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  With FIFO, MORE frames can cause MORE page faults!             │
│                                                                  │
│  Reference String: 1, 2, 3, 4, 1, 2, 5, 1, 2, 3, 4, 5            │
│                                                                  │
│  With 3 frames:                                                  │
│  1  2  3  4  1  2  5  1  2  3  4  5                             │
│  1  1  1  4  4  4  5  5  5  5  5  5                             │
│  -  2  2  2  1  1  1  1  1  3  3  3                             │
│  -  -  3  3  3  2  2  2  2  2  4  4                             │
│  F  F  F  F  F  F  F  -  -  F  F  -                             │
│                                                                  │
│  Page Faults: 9                                                  │
│                                                                  │
│  With 4 frames:                                                  │
│  1  2  3  4  1  2  5  1  2  3  4  5                             │
│  1  1  1  1  1  1  5  5  5  5  4  4                             │
│  -  2  2  2  2  2  2  1  1  1  1  5                             │
│  -  -  3  3  3  3  3  3  2  2  2  2                             │
│  -  -  -  4  4  4  4  4  4  3  3  3                             │
│  F  F  F  F  -  -  F  F  F  F  F  F                             │
│                                                                  │
│  Page Faults: 10 (MORE faults with MORE frames!)                │
│                                                                  │
│  This counterintuitive result is Belady's Anomaly.              │
│  FIFO and some other algorithms suffer from this.               │
│  LRU and Optimal do NOT have this anomaly.                      │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

---

## 2. Optimal Algorithm (OPT / MIN)

### Concept

```
┌──────────────────────────────────────────────────────────────────┐
│                   OPTIMAL ALGORITHM                               │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Policy: Replace page that will not be used for longest time    │
│                                                                  │
│  Look FORWARD in reference string                                │
│  Choose page with furthest next use (or never used again)       │
│                                                                  │
│  At time T with pages {A, B, C} in memory:                      │
│  Future references: ...B, C, B, A, C, B, A, D, A...             │
│                            ↑           ↑                        │
│                            B next      A next                   │
│                                        C never used → VICTIM    │
│                                                                  │
│  Guarantees MINIMUM possible page faults                        │
│                                                                  │
│  Problem: Requires future knowledge - IMPOSSIBLE to implement!  │
│  Used as theoretical benchmark to compare other algorithms      │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

### Detailed Example

```
Reference String: 7, 0, 1, 2, 0, 3, 0, 4, 2, 3, 0, 3, 2, 1, 2, 0, 1, 7, 0, 1
Number of Frames: 3

For each replacement, look at FUTURE references to decide:

Step | Ref | Frame 0 | Frame 1 | Frame 2 | Future Analysis          | Fault?
-----|-----|---------|---------|---------|--------------------------|-------
  1  |  7  |    7    |    -    |    -    |                          |   F
  2  |  0  |    7    |    0    |    -    |                          |   F
  3  |  1  |    7    |    0    |    1    |                          |   F
  4  |  2  |    2    |    0    |    1    | 7→pos 18, 0→pos 5, 1→pos 14 |   F
     |     |         |         |         | Replace 7 (furthest)      |
  5  |  0  |    2    |    0    |    1    |                          |   Hit
  6  |  3  |    2    |    0    |    3    | 2→pos 9, 0→pos 7, 1→pos 14 |   F
     |     |         |         |         | Replace 1 (furthest)       |
  7  |  0  |    2    |    0    |    3    |                          |   Hit
  8  |  4  |    2    |    4    |    3    | 2→pos 9, 0→pos 11, 3→pos 10 |   F
     |     |         |         |         | Replace 0 (furthest)       |
  9  |  2  |    2    |    4    |    3    |                          |   Hit
 10  |  3  |    2    |    4    |    3    |                          |   Hit
 11  |  0  |    2    |    0    |    3    | 4 never used again       |   F
 12  |  3  |    2    |    0    |    3    |                          |   Hit
 13  |  2  |    2    |    0    |    3    |                          |   Hit
 14  |  1  |    2    |    0    |    1    | 3 never used again       |   F
 15  |  2  |    2    |    0    |    1    |                          |   Hit
 16  |  0  |    2    |    0    |    1    |                          |   Hit
 17  |  1  |    2    |    0    |    1    |                          |   Hit
 18  |  7  |    7    |    0    |    1    | 2 never used again       |   F
 19  |  0  |    7    |    0    |    1    |                          |   Hit
 20  |  1  |    7    |    0    |    1    |                          |   Hit

Total Page Faults: 9 (MINIMUM possible)
Hit Ratio: 11/20 = 55%
```

---

## 3. LRU (Least Recently Used)

### Concept

```
┌──────────────────────────────────────────────────────────────────┐
│                    LRU ALGORITHM                                  │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Policy: Replace page that has not been used for longest time   │
│                                                                  │
│  Look BACKWARD (past) instead of forward                        │
│  Approximation of Optimal using recent history                  │
│                                                                  │
│  Intuition: Pages used recently will likely be used again       │
│             (Locality of Reference)                             │
│                                                                  │
│  Past references: ...A, B, C, A, B, D, C, A, [NOW]              │
│                                      ↑     ↑                    │
│                                     LRU   MRU                   │
│                                    (victim)                     │
│                                                                  │
│  Does NOT suffer from Belady's anomaly                          │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

### Detailed Example

```
Reference String: 7, 0, 1, 2, 0, 3, 0, 4, 2, 3, 0, 3, 2, 1, 2, 0, 1, 7, 0, 1
Number of Frames: 3

Track recency: Most Recent → ... → Least Recent

Step | Ref | Frame 0 | Frame 1 | Frame 2 | Recency (MRU→LRU) | Fault?
-----|-----|---------|---------|---------|-------------------|-------
  1  |  7  |    7    |    -    |    -    |        7          |   F
  2  |  0  |    7    |    0    |    -    |       0,7         |   F
  3  |  1  |    7    |    0    |    1    |      1,0,7        |   F
  4  |  2  |    2    |    0    |    1    |      2,1,0        |   F (7 LRU)
  5  |  0  |    2    |    0    |    1    |      0,2,1        |   Hit
  6  |  3  |    2    |    0    |    3    |      3,0,2        |   F (1 LRU)
  7  |  0  |    2    |    0    |    3    |      0,3,2        |   Hit
  8  |  4  |    4    |    0    |    3    |      4,0,3        |   F (2 LRU)
  9  |  2  |    4    |    0    |    2    |      2,4,0        |   F (3 LRU)
 10  |  3  |    4    |    3    |    2    |      3,2,4        |   F (0 LRU)
 11  |  0  |    0    |    3    |    2    |      0,3,2        |   F (4 LRU)
 12  |  3  |    0    |    3    |    2    |      3,0,2        |   Hit
 13  |  2  |    0    |    3    |    2    |      2,3,0        |   Hit
 14  |  1  |    1    |    3    |    2    |      1,2,3        |   F (0 LRU)
 15  |  2  |    1    |    3    |    2    |      2,1,3        |   Hit
 16  |  0  |    1    |    0    |    2    |      0,2,1        |   F (3 LRU)
 17  |  1  |    1    |    0    |    2    |      1,0,2        |   Hit
 18  |  7  |    1    |    0    |    7    |      7,1,0        |   F (2 LRU)
 19  |  0  |    1    |    0    |    7    |      0,7,1        |   Hit
 20  |  1  |    1    |    0    |    7    |      1,0,7        |   Hit

Total Page Faults: 12
Hit Ratio: 8/20 = 40%
```

### LRU Implementation

```
┌──────────────────────────────────────────────────────────────────┐
│              LRU IMPLEMENTATION METHODS                           │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Method 1: COUNTER-BASED                                        │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │ Each page has a counter (timestamp)                     │    │
│  │ On access: Copy system clock to page's counter          │    │
│  │ On replacement: Find page with smallest counter         │    │
│  │                                                         │    │
│  │ Problem: Must search all pages O(n)                     │    │
│  │          Counter overflow issues                        │    │
│  └─────────────────────────────────────────────────────────┘    │
│                                                                  │
│  Method 2: STACK-BASED                                          │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │ Maintain stack of page numbers                          │    │
│  │ On access: Move page to top of stack                    │    │
│  │ LRU page is always at bottom                           │    │
│  │                                                         │    │
│  │ Problem: Stack operations expensive O(n)                │    │
│  │          Hardware implementation difficult              │    │
│  └─────────────────────────────────────────────────────────┘    │
│                                                                  │
│  Both methods are expensive → Use LRU APPROXIMATIONS           │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

---

## 4. LRU Approximation Algorithms

### Reference Bit Algorithm

```
┌──────────────────────────────────────────────────────────────────┐
│                 REFERENCE BIT ALGORITHM                           │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Each page has a reference bit (R)                              │
│  • Hardware sets R=1 whenever page is accessed                  │
│  • OS periodically clears all reference bits                    │
│                                                                  │
│  Replacement:                                                    │
│  • Look for page with R=0 (not recently used)                  │
│  • If all R=1, clear all bits and try again                    │
│                                                                  │
│  Simple but coarse: Only distinguishes "used" vs "not used"    │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

### Additional Reference Bits Algorithm

```
┌──────────────────────────────────────────────────────────────────┐
│            ADDITIONAL REFERENCE BITS                              │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Keep 8-bit history for each page                               │
│  Periodically (every 100ms):                                     │
│  • Shift bits right                                             │
│  • Insert current R bit at leftmost position                    │
│  • Clear R bit                                                  │
│                                                                  │
│  Example (4-bit history shown):                                 │
│  Time:   T1    T2    T3    T4                                   │
│  Page A: 1000  1100  1110  0111  (used T1,T2,T3)               │
│  Page B: 1000  0100  0010  1001  (used T1,T4)                  │
│  Page C: 0000  1000  0100  0010  (used T2 only)                │
│                                                                  │
│  Lower number = Less recently used = Better victim              │
│                                                                  │
│  Page C (0010) would be replaced first                          │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

### Second-Chance (Clock) Algorithm

```
┌──────────────────────────────────────────────────────────────────┐
│               SECOND-CHANCE (CLOCK) ALGORITHM                     │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  FIFO with second chance based on reference bit                 │
│                                                                  │
│  Algorithm:                                                      │
│  1. Check page at current position (clock hand)                 │
│  2. If R=0: Replace this page                                   │
│  3. If R=1: Clear R, advance hand, repeat step 1               │
│                                                                  │
│  Visualization (circular queue):                                 │
│                                                                  │
│              ┌─────┐                                            │
│         ────→│ A:1 │ R=1, give second chance                   │
│        │     └─────┘                                            │
│        │        ↓                                               │
│     ┌─────┐  ┌─────┐                                           │
│     │ E:0 │←─│ B:0 │←── VICTIM (R=0)                           │
│     └─────┘  └─────┘                                            │
│        ↑        ↓                                               │
│     ┌─────┐  ┌─────┐                                           │
│     │ D:1 │←─│ C:1 │                                           │
│     └─────┘  └─────┘                                            │
│         ↑_______↓                                               │
│          Hand                                                   │
│                                                                  │
│  If all pages have R=1:                                         │
│  • Complete full circle clearing all R bits                     │
│  • Degenerates to FIFO                                          │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

### Enhanced Second-Chance

```
┌──────────────────────────────────────────────────────────────────┐
│             ENHANCED SECOND-CHANCE                                │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Use both Reference (R) and Modify (M) bits                     │
│                                                                  │
│  Four classes:                                                   │
│  ┌──────────┬──────────────────────────────────────────────┐    │
│  │ (R, M)   │ Description                                   │    │
│  ├──────────┼──────────────────────────────────────────────┤    │
│  │ (0, 0)   │ Not used, not modified - BEST VICTIM         │    │
│  │ (0, 1)   │ Not used, modified - need write-back        │    │
│  │ (1, 0)   │ Used, not modified - may be used again      │    │
│  │ (1, 1)   │ Used, modified - WORST VICTIM               │    │
│  └──────────┴──────────────────────────────────────────────┘    │
│                                                                  │
│  Algorithm:                                                      │
│  1. Scan for (0,0) page - replace immediately                  │
│  2. If not found, scan for (0,1) - replace, set R=0 for (1,x) │
│  3. Repeat steps 1 and 2                                       │
│                                                                  │
│  Reduces I/O by preferring clean pages                          │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

---

## 5. Counting-Based Algorithms

### LFU (Least Frequently Used)

```
┌──────────────────────────────────────────────────────────────────┐
│                    LFU ALGORITHM                                  │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Policy: Replace page with smallest reference count             │
│                                                                  │
│  Each page has a counter                                        │
│  Counter incremented on every access                            │
│                                                                  │
│  Page    Count    Status                                        │
│   A        25     Frequently used                               │
│   B         3     Rarely used → VICTIM                         │
│   C        18     Moderate use                                  │
│                                                                  │
│  Problem: Counter never decreases                               │
│  • Page used heavily initially but not anymore stays           │
│  • New pages always have low count                             │
│                                                                  │
│  Solution: Aging - periodically halve all counters             │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

### MFU (Most Frequently Used)

```
┌──────────────────────────────────────────────────────────────────┐
│                    MFU ALGORITHM                                  │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Policy: Replace page with highest reference count              │
│                                                                  │
│  Reasoning: Page with low count was just brought in            │
│             and hasn't had chance to be used yet               │
│                                                                  │
│  Page    Count    Status                                        │
│   A        25     Most used → VICTIM                           │
│   B         3     Just arrived, keep it                        │
│   C        18     Moderate use                                  │
│                                                                  │
│  Rarely used in practice                                        │
│  Neither LFU nor MFU are common                                 │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

---

## Algorithm Comparison

```
┌────────────────────────────────────────────────────────────────────────┐
│                    ALGORITHM COMPARISON                                 │
├────────────────────────────────────────────────────────────────────────┤
│                                                                        │
│  For reference string: 7,0,1,2,0,3,0,4,2,3,0,3,2,1,2,0,1,7,0,1       │
│  With 3 frames:                                                        │
│                                                                        │
│  ┌─────────────┬───────────────┬──────────────────────────────────┐   │
│  │ Algorithm   │ Page Faults   │ Notes                            │   │
│  ├─────────────┼───────────────┼──────────────────────────────────┤   │
│  │ FIFO        │      15       │ Simple, Belady's anomaly        │   │
│  │ Optimal     │       9       │ Minimum possible, not practical │   │
│  │ LRU         │      12       │ Good, expensive to implement    │   │
│  │ Clock       │      ~13      │ LRU approximation, practical    │   │
│  └─────────────┴───────────────┴──────────────────────────────────┘   │
│                                                                        │
│                   Page Faults                                          │
│                   ^                                                    │
│               15 │ ████  FIFO                                         │
│                  │ ████                                               │
│               12 │ ████  ████  LRU                                    │
│                  │ ████  ████                                         │
│                9 │ ████  ████  ████  OPT                              │
│                  └──────────────────────→                             │
│                    FIFO  LRU   OPT                                    │
│                                                                        │
└────────────────────────────────────────────────────────────────────────┘
```

### Summary Table

| Algorithm | Page Faults | Complexity | Belady's Anomaly | Practical? |
|-----------|-------------|------------|------------------|------------|
| FIFO | High | O(1) | Yes | Yes |
| Optimal | Minimum | O(n) | No | No (theoretical) |
| LRU | Low | O(n) | No | Difficult |
| Clock | Moderate | O(n) | Yes | Yes |
| LFU | Variable | O(n) | - | Rarely |

---

## Summary

| Concept | Description |
|---------|-------------|
| **FIFO** | Replace oldest page, simple but poor |
| **Optimal** | Replace page used furthest in future, benchmark |
| **LRU** | Replace least recently used, good approximation |
| **Clock** | FIFO with second chance using R bit |
| **Enhanced Clock** | Uses both R and M bits |
| **Belady's Anomaly** | More frames can cause more faults |
| **Reference Bit** | Hardware-set bit for access tracking |
| **Dirty/Modify Bit** | Indicates page was modified |

---

## Quick Revision Questions

1. Which algorithm guarantees minimum page faults?
2. Explain Belady's anomaly with an example.
3. How does the Clock algorithm give pages a second chance?
4. Why is pure LRU expensive to implement?
5. Compare (R,M) = (0,0) vs (1,1) in enhanced second-chance.

---

[← Previous: Virtual Memory](./04-virtual-memory.md) | [Next: Thrashing →](./06-thrashing.md)
