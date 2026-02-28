# Unit 8: Counting Sort — Limitations & When to Use

[← Previous: Complexity Analysis](04-complexity-analysis.md) | [Next: Variations →](06-variations.md)

---

## Overview

Counting Sort is powerful but comes with strict requirements. Understanding when it works — and when it doesn't — is crucial for choosing the right algorithm.

---

## Limitations

### 1. Only Works on Integers (or Discrete Values)

```
  ✓ Works:  integers, characters, enums, fixed categories
  ✗ Fails:  floating-point numbers, strings (directly), 
            objects without integer keys
  
  Why? Values are used as ARRAY INDICES.
  You can't do count[3.14] or count["hello"]
  
  Workaround: Map values to integers
  - Characters → ASCII values (0-127 or 0-255)
  - Grades A-F → 0-5
  - Float × 100 → integer (loses precision)
```

### 2. Range Must Be Reasonable

```
  ┌────────────────────────────────────────────────────────────┐
  │  Array: [1, 1000000000]   (n=2, k=10⁹)                  │
  │                                                            │
  │  Count array needs 10⁹ slots = ~4 GB memory!             │
  │  For just 2 elements!                                      │
  │                                                            │
  │  Quick Sort: O(2 log 2) = O(1) time, O(1) space          │
  │  Counting:   O(2 + 10⁹) time and space ← TERRIBLE       │
  └────────────────────────────────────────────────────────────┘
  
  Rule: If k >> n, DON'T use Counting Sort.
```

### 3. Not In-Place

```
  Requires O(n + k) extra space:
  - Count array: size k
  - Output array: size n
  
  In memory-constrained environments (embedded systems),
  this is a dealbreaker.
  
  Heap Sort uses O(1) extra space ← better choice there.
```

### 4. Not Adaptive

```
  Already sorted: [1, 2, 3, 4, 5]
  Reversed:       [5, 4, 3, 2, 1]
  Random:         [3, 1, 4, 5, 2]
  
  ALL take the same O(n + k) time.
  
  Insertion Sort on sorted data: O(n) — FASTER!
```

---

## When to Use Counting Sort

```
  ✓ IDEAL SCENARIOS:
  ┌────────────────────────────────────────────────────────────┐
  │  1. Small range of integer values (k ≤ n)                │
  │     Examples: ages, grades, ratings (1-5), dice rolls     │
  │                                                            │
  │  2. As a subroutine in Radix Sort                        │
  │     Sort by each digit (0-9) → k = 10                    │
  │                                                            │
  │  3. Counting character frequencies                        │
  │     k = 26 (lowercase) or 256 (ASCII)                    │
  │                                                            │
  │  4. Sorting data with many duplicates                     │
  │     Same values → same count bucket                       │
  │                                                            │
  │  5. When stability is required AND range is small         │
  │     Merge Sort is stable but O(n log n)                   │
  │     Counting Sort is stable AND O(n) when k is small     │
  └────────────────────────────────────────────────────────────┘
  
  ✗ AVOID WHEN:
  ┌────────────────────────────────────────────────────────────┐
  │  1. Range is huge (k >> n)                               │
  │  2. Data is floating-point                                │
  │  3. Memory is constrained                                 │
  │  4. Data is already partially sorted (use adaptive sort)  │
  │  5. Values are arbitrary objects without clear mapping    │
  └────────────────────────────────────────────────────────────┘
```

---

## Decision Flowchart

```
  Is data integer/discrete?
  │
  ├── NO → Use comparison sort (Quick/Merge/Heap)
  │
  └── YES → Is range k reasonable? (k ≤ n or k constant?)
       │
       ├── NO → Is k too large?
       │    │
       │    ├── Use Radix Sort (breaks digits, each with small k)
       │    └── Use comparison sort
       │
       └── YES → Is stability needed?
            │
            ├── YES → Counting Sort ✓ (with right-to-left)
            │
            └── NO → Is memory constrained?
                 │
                 ├── YES → Simple counting sort (O(k) space)
                 └── NO → Counting Sort ✓
```

---

## Practical Examples

### Example 1: Sort Exam Scores (0-100)

```
  Students: 1000, Scores: 0-100
  n=1000, k=100
  
  Counting Sort: O(1000 + 100) = O(1100) ≈ O(n) ✓
  Quick Sort:    O(1000 × 10) = O(10000)  
  
  Counting Sort wins by ~10x!
```

### Example 2: Sort Employee IDs

```
  Employees: 500, IDs: 1 to 1,000,000
  n=500, k=1,000,000
  
  Counting Sort: O(500 + 1,000,000) = O(1,000,500)
  Quick Sort:    O(500 × 9) = O(4500)
  
  Quick Sort wins by ~200x!
  DON'T use Counting Sort here.
```

### Example 3: Radix Sort Subroutine

```
  Sort 1,000,000 phone numbers (10 digits)
  Each Radix digit sort: k = 10
  
  Per digit: Counting Sort O(n + 10) = O(n)
  Total: 10 passes × O(n) = O(10n) = O(n)
  
  vs Quick Sort: O(n log n) = O(20n)
  
  Radix Sort (with Counting Sort) wins!
```

---

## Summary Table

| Scenario | n | k | Better Algorithm |
|----------|---|---|---|
| Exam scores | 1000 | 100 | Counting Sort |
| RGB pixels | 10⁶ | 256 | Counting Sort |
| Ages | 10⁶ | 150 | Counting Sort |
| Employee IDs | 500 | 10⁶ | Quick Sort |
| 32-bit ints | 10⁶ | 2³² | Radix Sort |
| Floating-point | any | continuous | Quick/Merge Sort |
| Nearly sorted | any | any | Insertion Sort |

---

## Quick Revision Questions

1. **Why can't Counting Sort handle floating-point numbers directly?**
2. **What's the threshold for k that makes Counting Sort impractical?**
3. **Name 3 real-world datasets well-suited for Counting Sort.**
4. **Why is Counting Sort used inside Radix Sort?**
5. **If you have 1000 strings of length 5 (a-z), how would you sort them?**

---

[← Previous: Complexity Analysis](04-complexity-analysis.md) | [Next: Variations →](06-variations.md)
