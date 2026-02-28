# Unit 9: Radix Sort — LSD vs MSD

[← Previous: Algorithm Description](01-algorithm-description.md) | [Next: Implementation →](03-implementation.md)

---

## Overview

There are two flavors of Radix Sort: **LSD (Least Significant Digit)** first and **MSD (Most Significant Digit)** first. They have different characteristics and are suited for different use cases.

---

## LSD Radix Sort (Bottom-Up)

```
  Process digits from RIGHT to LEFT:
  ones → tens → hundreds → ...
  
  Numbers: [170, 045, 075, 090, 002, 024, 802, 066]
  
  Pass 1 (ones):   [170, 090, 002, 802, 024, 045, 075, 066]
  Pass 2 (tens):   [002, 802, 024, 045, 066, 170, 075, 090]
  Pass 3 (hundreds):[002, 024, 045, 066, 075, 090, 170, 802]
  
  ✓ Sorted!
  
  Properties:
  ┌──────────────────────────────────────────────────────┐
  │  • Processes ALL numbers in every pass              │
  │  • Always d passes (d = max digits)                 │
  │  • Non-recursive, simple implementation             │
  │  • Requires STABLE subroutine sort                  │
  │  • Can use Counting Sort as subroutine              │
  └──────────────────────────────────────────────────────┘
```

---

## MSD Radix Sort (Top-Down)

```
  Process digits from LEFT to RIGHT:
  hundreds → tens → ones
  
  Numbers: [170, 045, 075, 090, 002, 024, 802, 066]
  
  Pass 1 (hundreds): Group by first digit
  ┌──────────────────────────────────────────────┐
  │  0xx: [045, 075, 090, 002, 024, 066]        │
  │  1xx: [170]                                  │
  │  8xx: [802]                                  │
  └──────────────────────────────────────────────┘
  
  Pass 2: Recursively sort EACH group by next digit
  
  0xx group → sort by tens:
  ┌──────────────────────────────────────────────┐
  │  00x: [002]                                  │
  │  02x: [024]                                  │
  │  04x: [045]                                  │
  │  06x: [066]                                  │
  │  07x: [075]                                  │
  │  09x: [090]                                  │
  └──────────────────────────────────────────────┘
  
  Each sub-group with 1 element → already sorted
  
  Final: [002, 024, 045, 066, 075, 090, 170, 802]  ✓
```

---

## Comparison: LSD vs MSD

```
  ┌──────────────────┬────────────────┬────────────────────┐
  │  Aspect          │ LSD            │ MSD                │
  ├──────────────────┼────────────────┼────────────────────┤
  │  Direction       │ Right to left  │ Left to right      │
  │  Approach        │ Iterative      │ Recursive          │
  │  Stability req.  │ MUST be stable │ Can be unstable    │
  │  Passes          │ Always d       │ Can short-circuit  │
  │  For equal-len   │ Excellent      │ Good               │
  │  For var-length  │ Needs padding  │ Natural handling   │
  │  Memory          │ O(n + k)       │ O(n + k + d) stack │
  │  Use case        │ Integers       │ Strings, var-len   │
  │  Parallelizable  │ Less so        │ Easily (groups)    │
  └──────────────────┴────────────────┴────────────────────┘
```

---

## Why LSD Works (Proof by Induction)

```
  Claim: After i passes, the array is sorted by the 
         last i digits.
  
  Base case: After pass 1 (ones digit), the array is
  sorted by the last 1 digit. ✓ (just counting sort)
  
  Inductive step: Assume sorted by last i digits.
  Pass i+1 sorts by digit i+1.
  
  Case 1: Two numbers differ in digit i+1
    → Counting sort puts them in correct order by this digit ✓
  
  Case 2: Two numbers have SAME digit i+1
    → Counting sort (stable) preserves their previous order
    → The previous order was correct for last i digits ✓
  
  In both cases, the array is sorted by the last i+1 digits. ✓
  
  After d passes: sorted by all d digits = fully sorted! □
```

---

## MSD Advantages for Variable-Length Data

```
  Strings of different lengths:
  
  ["banana", "apple", "cat", "bat", "app"]
  
  MSD naturally handles this:
  
  Pass 1 (1st char):
    'a': [apple, app]
    'b': [banana, bat]
    'c': [cat]
  
  Recurse on 'a' group (2nd char):
    'p': [apple, app]
  
  Recurse on 'p' group (3rd char):
    'p': [apple, app]
  
  Recurse (4th char):
    'l': [apple]
    '\0': [app]     ← shorter string comes first!
  
  Result: [app, apple, bat, banana, cat]  ✓
  
  LSD would need PADDING all strings to max length:
  "cat___", "bat___", "app___"  ← wasteful!
```

---

## MSD for Strings: Trie-Like Behavior

```
  MSD Radix Sort on strings resembles building a trie:
  
  ["dog", "dot", "do", "day", "dare"]
  
                root
               /    \
             'd'     
            / |  \
          'a' 'o'
         / \   |  \
       'y' 'r' 'g' 't'
           |
          'e'
  
  MSD groups by first character, then recursively
  sorts subgroups — just like trie insertion.
  
  Result: [dare, day, do, dog, dot]
```

---

## Choosing Between LSD and MSD

```
  Use LSD when:
  ┌──────────────────────────────────────────────────────┐
  │  • All items have same number of digits/chars       │
  │  • Sorting integers                                  │
  │  • Simple, non-recursive implementation preferred   │
  │  • Stability across all positions needed            │
  └──────────────────────────────────────────────────────┘
  
  Use MSD when:
  ┌──────────────────────────────────────────────────────┐
  │  • Items have VARIABLE lengths (strings)            │
  │  • Can benefit from early termination               │
  │  • Need lexicographic ordering of strings           │
  │  • Sub-problems can be parallelized                 │
  │  • Many short strings (fewer passes needed)         │
  └──────────────────────────────────────────────────────┘
```

---

## Summary Table

| Feature | LSD | MSD |
|---------|-----|-----|
| Direction | Right → Left | Left → Right |
| Approach | Iterative | Recursive |
| Requires stability | Yes (critical) | Not required |
| Variable-length data | Needs padding | Natural |
| Best for | Fixed-length integers | Strings |
| Short-circuit possible | No | Yes |
| Parallelizable | Limited | Yes (groups) |

---

## Quick Revision Questions

1. **Why does LSD process digits right-to-left?**
2. **Why doesn't MSD require a stable subroutine?**
3. **How does MSD handle strings of different lengths?**
4. **Prove by induction that LSD Radix Sort works correctly.**
5. **When would MSD terminate earlier than LSD?**

---

[← Previous: Algorithm Description](01-algorithm-description.md) | [Next: Implementation →](03-implementation.md)
