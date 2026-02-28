# Chapter 4: Bit Manipulation Optimization

## Overview
**Bit manipulation** replaces set operations with bitwise operations, dramatically accelerating backtracking algorithms. Instead of hash sets with O(1) amortized lookups, bitmasks give true O(1) operations with tiny memory footprint and cache-friendly access. For problems with n ≤ 20-30 items, bitmasks can speed up backtracking by 10-100×.

---

## Core Bit Operations

```
┌────────────────────────┬──────────────┬─────────────────────┐
│ Operation              │ Set version  │ Bit version         │
├────────────────────────┼──────────────┼─────────────────────┤
│ Add element i          │ set.add(i)   │ mask |= (1 << i)    │
│ Remove element i       │ set.remove(i)│ mask &= ~(1 << i)   │
│ Check if i present     │ i in set     │ mask & (1 << i)     │
│ Toggle element i       │ —            │ mask ^= (1 << i)    │
│ Union of A, B          │ A | B        │ a | b               │
│ Intersection of A, B   │ A & B        │ a & b               │
│ Count elements         │ len(set)     │ popcount(mask)      │
│ Is empty?              │ len(set)==0  │ mask == 0           │
│ Get any element        │ next(iter)   │ mask & (-mask)      │
│ Remove lowest element  │ —            │ mask &= (mask - 1)  │
│ All n elements         │ {0..n-1}     │ (1 << n) - 1        │
│ Complement             │ U - S        │ full ^ mask         │
└────────────────────────┴──────────────┴─────────────────────┘

All bit operations: O(1) time, single CPU instruction.
```

---

## Application 1: N-Queens with Bitmask

```
Standard N-Queens uses 3 sets:
  cols, diags, antiDiags
  → 3 hash sets, each lookup O(1) amortized

Bitmask N-Queens uses 3 integers:
  cols, ld (left diagonal), rd (right diagonal)
  → 3 integers, each operation O(1) actual

FUNCTION solve(row, cols, ld, rd, n):
    IF row == n: RETURN 1
    count = 0
    
    // Available columns = all columns minus blocked ones
    available = ((1 << n) - 1) & ~(cols | ld | rd)
    
    WHILE available != 0:
        bit = available & (-available)      ← Lowest set bit
        available ^= bit                    ← Remove it
        
        count += solve(row + 1,
                       cols | bit,          ← Mark column
                       (ld | bit) << 1,     ← Shift diag left
                       (rd | bit) >> 1)     ← Shift diag right
    
    RETURN count

Why faster:
  • No hash computation
  • No memory allocation
  • Single CPU instructions
  • Cache-line friendly (just 3 integers)

Benchmark (n=14):
  Set-based:    ~2.5 seconds
  Bitmask:      ~0.15 seconds  (~17× faster)
```

---

## How Diagonal Shifting Works

```
When queen placed at column bit position b in row r:

Left diagonal (ld):
  In row r+1, the diagonal attack shifts LEFT by 1
  (ld | b) << 1
  
  Row 0:  . . Q . .     bit = 00100
  Row 1:  . Q . . .     shifted = 01000

Right diagonal (rd):
  In row r+1, the diagonal attack shifts RIGHT by 1
  (rd | b) >> 1

  Row 0:  . . Q . .     bit = 00100
  Row 1:  . . . Q .     shifted = 00010

Column (cols):
  No shift needed — column blocked for ALL rows below.
  cols | b

Visualization for 5×5 after placing queen at col 2:
  cols:        0 0 1 0 0
  ld (row 1):  0 1 0 0 0    (shifted left)
  rd (row 1):  0 0 0 1 0    (shifted right)
  blocked:     0 1 1 1 0    (OR of all three)
  available:   1 0 0 0 1    (complement & mask)
```

---

## Application 2: Subset Generation

```
All subsets of {0, 1, ..., n-1} via bitmask enumeration:

FOR mask = 0 TO (1 << n) - 1:
    subset = []
    FOR i = 0 TO n-1:
        IF mask & (1 << i):
            subset.add(items[i])
    PRINT subset

Example: n=3, items = [a, b, c]
  mask=000 → {}
  mask=001 → {a}
  mask=010 → {b}
  mask=011 → {a,b}
  mask=100 → {c}
  mask=101 → {a,c}
  mask=110 → {b,c}
  mask=111 → {a,b,c}

Iterating through set bits efficiently:
  temp = mask
  WHILE temp != 0:
      bit = temp & (-temp)          ← Lowest set bit
      i = log2(bit)                 ← Its position
      process(items[i])
      temp ^= bit                   ← Remove it
  
  This visits only SET bits — O(k) where k = |subset|.
```

---

## Application 3: Bitmask DP (TSP)

```
Traveling Salesman Problem with n ≤ 20:

State: (current city, set of visited cities)
  visited set → n-bit integer (2^n possible sets)

dp[city][mask] = min cost to visit remaining from (city, mask)

FUNCTION tsp(city, mask):
    IF mask == (1 << n) - 1:         ← All visited
        RETURN dist[city][start]
    IF dp[city][mask] != -1:
        RETURN dp[city][mask]
    
    result = INFINITY
    unvisited = ((1 << n) - 1) ^ mask    ← Complement
    temp = unvisited
    WHILE temp != 0:
        bit = temp & (-temp)
        next = log2(bit)
        result = min(result, 
            dist[city][next] + tsp(next, mask | bit))
        temp ^= bit
    
    dp[city][mask] = result
    RETURN result

Complexity: O(n² × 2^n) — polynomial per state!
  n=20: 20 × 20 × 2^20 ≈ 420 million (feasible)
  n=20 brute force: 20! ≈ 2.4 × 10^18 (impossible)
```

---

## Application 4: Permutations with Used Mask

```
Instead of boolean used[] array, use a bitmask:

FUNCTION permute(nums, mask, current, result):
    IF length(current) == length(nums):
        result.add(copy(current))
        RETURN
    
    available = ((1 << n) - 1) & ~mask     ← Unused elements
    temp = available
    WHILE temp != 0:
        bit = temp & (-temp)
        i = log2(bit)
        current.add(nums[i])
        permute(nums, mask | bit, current, result)
        current.removeLast()
        temp ^= bit

Benefits:
  • Checking "is i used?" → single AND operation
  • Marking used → single OR operation
  • Finding unused → complement + iterate set bits
  • Passing state → single integer (no array copy)
```

---

## Useful Bit Tricks Reference

```
┌───────────────────────────────────────────────────┐
│ Extract lowest set bit:   x & (-x)               │
│   0110 1100 → 0000 0100                           │
│                                                   │
│ Remove lowest set bit:    x & (x - 1)             │
│   0110 1100 → 0110 1000                           │
│                                                   │
│ Check if power of 2:     x & (x - 1) == 0         │
│   0100 0000 → 0000 0000 (yes)                     │
│   0110 0000 → 0100 0000 (no)                      │
│                                                   │
│ Count set bits (popcount): built-in in most langs  │
│   Java: Integer.bitCount(x)                        │
│   Python: bin(x).count('1')                        │
│   C++: __builtin_popcount(x)                       │
│                                                   │
│ Iterate all submasks of mask m:                    │
│   sub = m                                          │
│   WHILE sub > 0:                                   │
│       process(sub)                                 │
│       sub = (sub - 1) & m                          │
│                                                   │
│ Set bit i:               x | (1 << i)              │
│ Clear bit i:             x & ~(1 << i)             │
│ Toggle bit i:            x ^ (1 << i)              │
│ Check bit i:             (x >> i) & 1              │
└───────────────────────────────────────────────────┘
```

---

## When to Use Bitmask

```
USE bitmask when:
  ✓ n ≤ 20-25 (fits in 32-bit integer)
  ✓ State is a SUBSET of items
  ✓ Need fast set operations
  ✓ Want to use sets as DP keys (integer indexing)

DON'T use bitmask when:
  ✗ n > 30 (exceeds integer size)  
  ✗ State isn't a subset (e.g., arbitrary values)
  ✗ Clarity matters more than speed
  ✗ Set operations are infrequent

Size limits:
  32-bit int: n ≤ 30 (2^30 ≈ 10^9 — borderline)
  64-bit long: n ≤ 62 (2^62 ≈ 4×10^18 — too large for DP)
  Practical DP limit: n ≤ 20-23
```

---

## Complexity Comparison

```
┌──────────────────┬──────────────┬──────────────┐
│ Operation        │ HashSet      │ Bitmask      │
├──────────────────┼──────────────┼──────────────┤
│ Add/remove       │ O(1) amort.  │ O(1) exact   │
│ Contains         │ O(1) amort.  │ O(1) exact   │
│ Copy state       │ O(n)         │ O(1)         │
│ Memory per set   │ O(n) objects │ O(1) integer │
│ Cache behavior   │ Poor (ptrs)  │ Excellent    │
│ Constant factor  │ ~50-100 ns   │ ~1-3 ns      │
└──────────────────┴──────────────┴──────────────┘

The real win: O(1) state copying.
  Passing a set down the recursion = copy the integer.
  No need for explicit undo (just don't modify the int).
```

---

## Summary Table

| Aspect | Detail |
|--------|--------|
| Representation | Integer where bit i = element i |
| Max elements | 30 (32-bit) or 62 (64-bit) |
| Key advantage | True O(1) for all set operations |
| State passing | Single integer (no copy needed) |
| Best for | N-Queens, TSP, subset DP, permutations |
| Speedup | Typically 10-100× over hash sets |
| Catch | Limited to small n |

---

## Quick Revision Questions

1. **How do you represent** a set as a bitmask?
2. **What does `x & (-x)`** compute and why?
3. **How does diagonal shifting** work in N-Queens bitmask?
4. **What's the maximum n** for bitmask DP?
5. **Why is bitmask state** faster to pass in recursion?
6. **How do you iterate** all submasks of a given mask?

---

[← Previous: Memoization in Backtracking](03-memoization-in-backtracking.md) | [Next: Symmetry Breaking →](05-symmetry-breaking.md)

[← Back to README](../README.md)
