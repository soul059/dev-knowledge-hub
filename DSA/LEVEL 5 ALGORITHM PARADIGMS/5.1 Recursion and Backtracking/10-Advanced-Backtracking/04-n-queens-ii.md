# Chapter 4: N-Queens II

## Overview
**N-Queens II** is the counting variant of the N-Queens problem — instead of listing all solutions, return just the **count**. This subtle change enables powerful optimizations: skip board construction entirely, exploit symmetry to halve the work, and use bit manipulation for ultra-fast constraint checks.

---

## Problem Statement

```
Given integer n, return the NUMBER of distinct solutions
to the N-Queens puzzle.

n=1 → 1    n=5 → 10    n=9  → 352
n=2 → 0    n=6 → 4     n=10 → 724
n=3 → 0    n=7 → 40    n=11 → 2680
n=4 → 2    n=8 → 92    n=12 → 14200
```

---

## Approach 1: Standard Backtracking (Count Only)

```
FUNCTION totalNQueens(n):
    count = 0
    cols = SET()
    diags = SET()         ← row - col
    antiDiags = SET()     ← row + col
    solve(0, n, cols, diags, antiDiags, count)
    RETURN count

FUNCTION solve(row, n, cols, diags, antiDiags, count):
    IF row == n:
        count += 1                      ← Found a solution!
        RETURN
    
    FOR col = 0 TO n-1:
        IF col NOT IN cols
           AND (row-col) NOT IN diags
           AND (row+col) NOT IN antiDiags:
            
            cols.add(col)
            diags.add(row-col)
            antiDiags.add(row+col)
            
            solve(row+1, n, cols, diags, antiDiags, count)
            
            cols.remove(col)            ← Undo
            diags.remove(row-col)
            antiDiags.remove(row+col)

Key difference from N-Queens I:
  - No board needed (don't build solutions)
  - Just increment counter when row == n
  - Saves O(n²) space per solution
```

---

## Approach 2: Bit Manipulation

```
Use BITMASKS instead of sets for O(1) operations:

cols:      bit i set → column i is occupied
diags:     bit j set → diagonal j is occupied  
antiDiags: bit k set → anti-diagonal k is occupied

FUNCTION solve(row, n, cols, diags, antiDiags):
    IF row == n:
        RETURN 1
    
    count = 0
    // available = columns NOT blocked by any constraint
    available = ((1 << n) - 1)        ← all n bits set
              & ~cols                  ← remove taken columns
              & ~(diags >> row)        ← remove taken diagonals
              & ~(antiDiags >> row)    ← remove taken anti-diags
    
    WHILE available != 0:
        pos = available & (-available)  ← Extract lowest set bit
        available -= pos                ← Remove it
        col = log2(pos)
        
        count += solve(row+1, n,
                       cols | pos,
                       diags | (pos << 1),     ← shift diagonal
                       antiDiags | (pos >> 1))  ← shift anti-diag
    
    RETURN count

Simplified bit version (shift diags per row):

FUNCTION solve(row, cols, ld, rd, n):
    IF row == n: RETURN 1
    count = 0
    available = ((1 << n) - 1) & ~(cols | ld | rd)
    WHILE available:
        bit = available & (-available)
        available ^= bit
        count += solve(row+1, cols|bit, (ld|bit)<<1, (rd|bit)>>1, n)
    RETURN count

Where:
  cols = columns taken
  ld   = left diagonals (shifted left each row)
  rd   = right diagonals (shifted right each row)
```

---

## Approach 3: Symmetry Optimization

```
The N-Queens board has LEFT-RIGHT symmetry.
If we mirror any solution horizontally, we get another valid solution.

Strategy:
  In the FIRST ROW, only try columns 0 to n/2 - 1
  Multiply count by 2 at the end.

  Special case for odd n:
    If queen in first row is at the middle column (n/2),
    count those solutions separately (don't double them).

FUNCTION totalNQueens(n):
    count = 0
    
    // Left half of first row
    FOR col = 0 TO (n-1)/2:
        Place queen at (0, col)
        count += 2 × solve(row=1, ...)    ← Double the count
    
    // Middle column (only if n is odd)
    IF n is odd:
        Place queen at (0, n/2)
        count += solve(row=1, ...)         ← Don't double
    
    RETURN count

Speedup: nearly 2× (explores half the first-row branches)
```

---

## Performance Comparison

```
For n=12 (14,200 solutions):

┌──────────────┬──────────────┬────────────────────┐
│ Method       │ Time         │ Notes              │
├──────────────┼──────────────┼────────────────────┤
│ With board   │ ~800ms       │ Builds all configs │
│ Sets only    │ ~400ms       │ No board storage   │
│ Bitmask      │ ~50ms        │ Hardware bitops     │
│ Bitmask+sym  │ ~25ms        │ Half first row     │
└──────────────┴──────────────┴────────────────────┘

Bitmask is ~16× faster than board-based approach!
```

---

## Why Counting Is Easier Than Listing

```
Listing (N-Queens I):
  - Must store/copy board configuration for each solution
  - O(n²) work per solution found
  - Total output size: O(solutions × n²)

Counting (N-Queens II):
  - No board storage needed
  - O(1) work per solution found (just count++)
  - Output: single integer
  - Enables bitmask (just 3 integers for state)
  - Enables symmetry (just multiply)

This principle applies to many problems:
  "Count solutions" is often MUCH faster than "List solutions"
```

---

## Complexity

```
┌──────────────────────────────────────────────┐
│ All approaches have the same worst-case:     │
│                                              │
│ Time:  O(n!)  (tighter: very few valid       │
│        placements per row as rows fill up)   │
│                                              │
│ Space:                                       │
│   Sets approach:   O(n)                      │
│   Bitmask:         O(n) stack only           │
│                    (3 ints for constraints)   │
│   With symmetry:   Same + 2× speedup const   │
│                                              │
│ Practical: n=15 takes seconds with bitmask;  │
│            n=20+ needs hours even optimized   │
└──────────────────────────────────────────────┘
```

---

## Summary Table

| Aspect | Standard | Bitmask | Symmetry |
|--------|----------|---------|----------|
| Constraint storage | 3 Sets | 3 integers | 3 integers |
| Check valid | O(1) set lookup | O(1) bitwise | O(1) bitwise |
| Find candidates | Loop n cols | Extract set bits | Extract set bits |
| First row range | 0..n-1 | 0..n-1 | 0..n/2 |
| Output | Count | Count | Count×2 |
| Relative speed | 1× | ~8-16× | ~16-32× |

---

## Quick Revision Questions

1. **Why doesn't N-Queens II** need a board array?
2. **How does bitmask** represent column conflicts?
3. **What does `bit & (-bit)`** extract and why?
4. **How does symmetry** cut the work nearly in half?
5. **Why is counting** fundamentally faster than listing?
6. **How do diagonal bitmasks** shift between rows?

---

[← Previous: Word Break II](03-word-break-ii.md) | [Next: Generate Parentheses →](05-generate-parentheses.md)

[← Back to README](../README.md)
