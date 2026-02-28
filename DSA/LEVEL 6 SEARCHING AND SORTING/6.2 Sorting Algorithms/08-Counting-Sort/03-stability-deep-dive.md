# Unit 8: Counting Sort — Stability Deep Dive

[← Previous: Implementation](02-implementation.md) | [Next: Complexity Analysis →](04-complexity-analysis.md)

---

## Overview

Counting Sort's **stability** is its most important property — it's what makes Radix Sort possible. This chapter explores exactly how and why stability works in Counting Sort.

---

## What Is Stability?

```
  A sort is STABLE if elements with equal keys maintain
  their original relative order after sorting.
  
  Input:  [3a, 1, 3b, 2]   (3a appears before 3b)
  
  Stable sort:   [1, 2, 3a, 3b]  ✓ (3a still before 3b)
  Unstable sort: [1, 2, 3b, 3a]  ✗ (order reversed)
```

---

## Why Stability Matters

```
  Scenario: Sort students by grade, then by name.
  
  After sorting by name:
  ┌─────────┬───────┐
  │  Name   │ Grade │
  ├─────────┼───────┤
  │  Alice  │  85   │
  │  Bob    │  72   │
  │  Carol  │  85   │
  │  Dave   │  72   │
  └─────────┴───────┘
  
  Now sort by grade using STABLE sort:
  ┌─────────┬───────┐      
  │  Bob    │  72   │   Within same grade,
  │  Dave   │  72   │   alphabetical order
  │  Alice  │  85   │   is PRESERVED!
  │  Carol  │  85   │      
  └─────────┴───────┘
  
  With UNSTABLE sort:
  ┌─────────┬───────┐      
  │  Dave   │  72   │   Order within same
  │  Bob    │  72   │   grade is RANDOM!
  │  Carol  │  85   │   Alphabetical sort
  │  Alice  │  85   │   was lost!
  └─────────┴───────┘
  
  Stability lets you sort by multiple keys
  (sort by least significant key first → most significant last)
```

---

## How Counting Sort Achieves Stability

```
  The magic is in TWO things:
  1. Prefix sum gives POSITION RANGES
  2. Right-to-left traversal fills from the END of each range
  
  ═══════════════════════════════════════════════════════
  
  Array: [2a, 5, 2b, 3a, 3b]    (a,b mark order)
  
  Count:      [0, 0, 2, 2, 0, 1]
               0  1  2  3  4  5
  
  Prefix sum: [0, 0, 2, 4, 4, 5]
               0  1  2  3  4  5
  
  Meaning:
  - C[2]=2: elements ≤ 2 occupy positions 0-1
  - C[3]=4: elements ≤ 3 occupy positions 0-3
  - C[5]=5: elements ≤ 5 occupy positions 0-4
  
  Position ranges:
  Value 2: positions 0-1  (C[2]=2, so indices 0 and 1)
  Value 3: positions 2-3  (C[3]=4, so indices 2 and 3)
  Value 5: position  4    (C[5]=5, so index 4)
  
  ═══ RIGHT-TO-LEFT placement ═══
  
  Output: [_, _, _, _, _]
  
  i=4: A[4]=3b → C[3]=4 → pos=3 → Output[3]=3b → C[3]=3
  i=3: A[3]=3a → C[3]=3 → pos=2 → Output[2]=3a → C[3]=2
  i=2: A[2]=2b → C[2]=2 → pos=1 → Output[1]=2b → C[2]=1
  i=1: A[1]=5  → C[5]=5 → pos=4 → Output[4]=5  → C[5]=4
  i=0: A[0]=2a → C[2]=1 → pos=0 → Output[0]=2a → C[2]=0
  
  Output: [2a, 2b, 3a, 3b, 5]
  
  2a before 2b ✓ (original order)
  3a before 3b ✓ (original order)
  
  STABLE! ✓
```

---

## What Happens with LEFT-TO-RIGHT?

```
  Same example, but traverse LEFT-TO-RIGHT:
  
  Array: [2a, 5, 2b, 3a, 3b]
  Prefix sum: [0, 0, 2, 4, 4, 5]
  
  i=0: A[0]=2a → C[2]=2 → pos=1 → Output[1]=2a → C[2]=1
  i=1: A[1]=5  → C[5]=5 → pos=4 → Output[4]=5  → C[5]=4
  i=2: A[2]=2b → C[2]=1 → pos=0 → Output[0]=2b → C[2]=0
  i=3: A[3]=3a → C[3]=4 → pos=3 → Output[3]=3a → C[3]=3
  i=4: A[4]=3b → C[3]=3 → pos=2 → Output[2]=3b → C[3]=2
  
  Output: [2b, 2a, 3b, 3a, 5]
  
  2b before 2a ✗ (REVERSED!)
  3b before 3a ✗ (REVERSED!)
  
  UNSTABLE! ✗
  
  ┌──────────────────────────────────────────────────────┐
  │  The prefix sum decrements from the END of range.   │
  │  So right-to-left fills positions from end → start, │
  │  which preserves the relative order.                 │
  │                                                      │
  │  Left-to-right fills in reverse, breaking stability. │
  └──────────────────────────────────────────────────────┘
```

---

## Alternative: Left-to-Right with Different Indexing

```
  You CAN use left-to-right AND maintain stability,
  but you need DIFFERENT prefix sum logic:
  
  Instead of C[i] = last position + 1,
  use C[i] = first position, and INCREMENT after placing.
  
  Exclusive prefix sum: C[i] = number of elements STRICTLY less than i
  
  Count:         [0, 0, 2, 2, 0, 1]
  Exclusive sum: [0, 0, 0, 2, 4, 4]    (start positions)
  
  Left-to-right:
  i=0: A[0]=2a → C[2]=0 → pos=0 → Output[0]=2a → C[2]=1
  i=1: A[1]=5  → C[5]=4 → pos=4 → Output[4]=5  → C[5]=5
  i=2: A[2]=2b → C[2]=1 → pos=1 → Output[1]=2b → C[2]=2
  i=3: A[3]=3a → C[3]=2 → pos=2 → Output[2]=3a → C[3]=3
  i=4: A[4]=3b → C[3]=3 → pos=3 → Output[3]=3b → C[3]=4
  
  Output: [2a, 2b, 3a, 3b, 5]  STABLE! ✓
  
  Both approaches work. The standard (right-to-left with 
  inclusive prefix sum) is from CLRS textbook.
```

---

## Why Stability Enables Radix Sort

```
  Radix Sort sorts multi-digit numbers digit by digit,
  from LEAST significant to MOST significant.
  
  Numbers: [329, 457, 657, 839, 436, 720, 355]
  
  Sort by ones digit (using stable counting sort):
  [720, 355, 436, 457, 657, 329, 839]
  
  Sort by tens digit (using stable counting sort):
  [720, 329, 436, 839, 355, 457, 657]
       ↑                                  
  329 before 839? They have same tens digit (2 vs 3... 
  wait, let me redo)
  
  Actually: sort by tens:
  [720, 329, 436, 839, 355, 457, 657]
  
  Sort by hundreds digit (using stable counting sort):
  [329, 355, 436, 457, 657, 720, 839]  ← SORTED! ✓
  
  ┌──────────────────────────────────────────────────────┐
  │  This ONLY works because each digit-sort is STABLE! │
  │  The previous digit ordering is preserved for equal  │
  │  digits in the current position.                     │
  │                                                      │
  │  If unstable: previous digit ordering would be lost, │
  │  and the final result would be WRONG.                │
  └──────────────────────────────────────────────────────┘
```

---

## Summary Table

| Aspect | Details |
|--------|---------|
| Stability mechanism | Prefix sum + right-to-left placement |
| Why right-to-left | Fills positions from end of range, preserving input order |
| Alternative stable approach | Exclusive prefix sum + left-to-right (increment positions) |
| Why stability matters | Enables multi-key sorting, Radix Sort |
| Without stability | Counting Sort still sorts correctly, just can't compose sorts |

---

## Quick Revision Questions

1. **What makes Counting Sort stable?**
2. **What goes wrong if you traverse left-to-right with standard prefix sum?**
3. **How can you achieve stability with left-to-right traversal?**
4. **Why is stability essential for Radix Sort?**
5. **Give a real-world example where sort stability matters.**

---

[← Previous: Implementation](02-implementation.md) | [Next: Complexity Analysis →](04-complexity-analysis.md)
