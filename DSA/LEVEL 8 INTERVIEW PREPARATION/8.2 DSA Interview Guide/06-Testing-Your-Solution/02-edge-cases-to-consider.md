# Chapter 6.2: Edge Cases to Consider

```
╔══════════════════════════════════════════════════════════╗
║         EDGE CASES TO CONSIDER                           ║
║   The comprehensive edge case encyclopedia               ║
╚══════════════════════════════════════════════════════════╝
```

## Overview

This chapter is a **reference guide** of edge cases organized by problem category. Bookmark this page and review it before mock interviews. Missing a single edge case can cost you the offer.

---

## Numeric Edge Cases

```
┌──────────────────────────────────────────────────────────┐
│  NUMBERS:                                                │
│  ├── Zero: 0 (often a special case)                    │
│  ├── Negative: -1, -100                                │
│  ├── Maximum: 2³¹ - 1 = 2,147,483,647 (INT_MAX)       │
│  ├── Minimum: -2³¹ = -2,147,483,648 (INT_MIN)         │
│  ├── Overflow: INT_MAX + 1 wraps to INT_MIN            │
│  ├── Power of 2: 1, 2, 4, 8, 16, ...                  │
│  └── Single digit: 0-9                                  │
│                                                          │
│  FLOATING POINT:                                         │
│  ├── 0.0, -0.0 (different in some languages)           │
│  ├── Precision: 0.1 + 0.2 ≠ 0.3 exactly               │
│  ├── Infinity: float('inf')                             │
│  └── NaN: float('nan')                                  │
│                                                          │
│  DIVISION:                                               │
│  ├── Division by zero                                   │
│  ├── Integer division rounding: -7 // 2 = ?            │
│  │   Python: -4 (floor)  Java/C++: -3 (truncation)    │
│  └── Modulo with negatives varies by language           │
└──────────────────────────────────────────────────────────┘
```

---

## Array/List Edge Cases

```
┌──────────────────────────────────────────────────────────┐
│  SIZE:                                                   │
│  ├── Empty: []                                          │
│  ├── Single element: [5]                                │
│  ├── Two elements: [1, 2]                               │
│  └── Very large: n = 10⁵ or 10⁶                        │
│                                                          │
│  VALUES:                                                 │
│  ├── All same: [7, 7, 7, 7]                            │
│  ├── All zeros: [0, 0, 0]                               │
│  ├── Contains negatives: [-3, -1, 0, 2, 5]             │
│  ├── Contains INT_MAX/INT_MIN                           │
│  └── Mix of positive and negative                       │
│                                                          │
│  ORDER:                                                  │
│  ├── Already sorted ascending: [1, 2, 3, 4, 5]         │
│  ├── Sorted descending: [5, 4, 3, 2, 1]                │
│  ├── Nearly sorted (one swap away)                      │
│  └── Random order                                        │
│                                                          │
│  POSITION:                                               │
│  ├── Answer at index 0 (first)                          │
│  ├── Answer at index n-1 (last)                         │
│  ├── Answer in the middle                               │
│  └── Multiple valid answers                              │
└──────────────────────────────────────────────────────────┘
```

---

## String Edge Cases

```
┌──────────────────────────────────────────────────────────┐
│  SIZE:                                                   │
│  ├── Empty: ""                                          │
│  ├── Single char: "a"                                   │
│  ├── Two chars: "ab"                                    │
│  └── Very long: length = 10⁵                            │
│                                                          │
│  CONTENT:                                                │
│  ├── All same: "aaaa"                                   │
│  ├── All unique: "abcde"                                │
│  ├── With spaces: "a b c"                               │
│  ├── Leading/trailing spaces: "  hello  "               │
│  ├── Only spaces: "   "                                 │
│  ├── Special chars: "a@b#c$"                            │
│  ├── Numbers in string: "abc123"                        │
│  └── Unicode: "héllo wörld"                             │
│                                                          │
│  PATTERNS:                                               │
│  ├── Palindrome: "aba", "abba"                          │
│  ├── All same char: "aaaa"                              │
│  ├── Single repeating pattern: "abcabc"                 │
│  └── Anagram pair: "listen" / "silent"                  │
│                                                          │
│  CASE:                                                   │
│  ├── All lowercase: "hello"                             │
│  ├── All uppercase: "HELLO"                             │
│  ├── Mixed case: "HeLLo"                                │
│  └── Case-insensitive comparison needed?                │
└──────────────────────────────────────────────────────────┘
```

---

## Tree Edge Cases

```
NULL/EMPTY:          SINGLE:           SKEWED LEFT:
    None                1                  1
                                          /
                                         2
                                        /
                                       3

SKEWED RIGHT:       PERFECT:          NEGATIVE VALUES:
    1                   1                  -1
     \                 / \                /   \
      2               2   3             -5    3
       \             /\ /\             /  \
        3           4 5 6 7           -8   -2

ADDITIONAL:
├── All same values: every node has value 5
├── Very deep tree (depth = 10,000) → stack overflow?
├── BST with duplicates — go left or right?
├── Single child nodes only
└── Leaf nodes only (root with no children = single)
```

---

## Graph Edge Cases

```
EMPTY:         SINGLE NODE:     DISCONNECTED:
(no nodes)         A            A─B   C─D   E

CYCLE:          SELF-LOOP:      COMPLETE:
A → B           A ──┐           A─B
↑   ↓           ↑   │           |╲|
D ← C           └───┘           C─D

ADDITIONAL:
├── Parallel edges: A → B (multiple edges)
├── Bipartite vs non-bipartite
├── Node with no edges (isolated)
├── Negative edge weights (Dijkstra fails!)
├── Very dense: E ≈ V²
├── Very sparse: E ≈ V
└── Tree structure (connected, no cycles)
```

---

## Linked List Edge Cases

```
╔══════════════════════════════════════════════╗
║  Empty:     None                             ║
║  Single:    [1] → None                       ║
║  Two:       [1] → [2] → None                ║
║  Cycle:     [1] → [2] → [3] → [2] (cycle) ║
║  All same:  [5] → [5] → [5] → None         ║
║  Sorted:    [1] → [2] → [3] → None         ║
║  Reversed:  [3] → [2] → [1] → None         ║
║  Head=Tail: Operation on first/last node     ║
╚══════════════════════════════════════════════╝
```

---

## Binary Search Edge Cases

```
┌──────────────────────────────────────────────┐
│  ☐ Target at first index                    │
│  ☐ Target at last index                     │
│  ☐ Target not in array                      │
│  ☐ Array of size 1 (target present)         │
│  ☐ Array of size 1 (target absent)          │
│  ☐ All same elements, target present        │
│  ☐ All same elements, target absent         │
│  ☐ Duplicate targets (find first/last)      │
│  ☐ Even vs odd array length                 │
│  ☐ Very large array (overflow in mid calc)  │
└──────────────────────────────────────────────┘
```

---

## Summary Table

| Data Type | Critical Edge Cases | Why They Matter |
|-----------|-------------------|-----------------|
| Numbers | 0, negative, overflow | Wrong arithmetic, crashes |
| Arrays | Empty, single, all same | Off-by-one, wrong init |
| Strings | Empty, spaces, case | Missing checks, wrong output |
| Trees | Null, skewed, single | Stack overflow, wrong traversal |
| Graphs | Disconnected, cycles | Infinite loops, missed nodes |
| Linked Lists | Empty, single, cycle | Null pointer crash |

---

## Quick Revision Questions

1. **What are the three most universal edge cases for any problem?**
   - Empty input, single element, and "no valid answer exists"

2. **Why is a skewed tree an important edge case?**
   - It turns O(log n) tree operations into O(n), can cause stack overflow with recursion, and tests if your algorithm handles worst-case depth

3. **What numeric edge case catches the most bugs?**
   - Negative numbers — they break sum calculations, absolute value assumptions, and modulo operations

4. **How do you handle the "answer at the boundary" test?**
   - Test with the answer at index 0 and index n-1, ensuring your loops process the first and last elements correctly

5. **What graph edge case is most commonly missed?**
   - Disconnected graphs — algorithms that assume a single connected component will miss nodes in other components

6. **Name three string edge cases that often cause bugs.**
   - Empty string "", strings with leading/trailing spaces, and case-sensitivity mismatches

---

[← Previous: Generating Test Cases](01-generating-test-cases.md) | [Next: Debugging Strategies →](03-debugging-strategies.md)

---
[← Back to README](../README.md)
