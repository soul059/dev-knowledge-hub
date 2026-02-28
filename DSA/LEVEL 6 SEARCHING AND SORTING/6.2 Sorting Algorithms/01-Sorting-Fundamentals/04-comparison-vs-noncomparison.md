# Chapter 4: Comparison vs Non-Comparison Sorting

[← Previous: Stable vs Unstable](03-stable-vs-unstable.md) | [Next: Internal vs External →](05-internal-vs-external.md)

---

## Overview

Sorting algorithms are divided into two fundamental categories based on **how they determine order**: comparison-based sorts that use pairwise comparisons, and non-comparison sorts that exploit the structure of the data itself. This distinction has profound implications for **theoretical performance limits**.

---

## Comparison-Based Sorting

A comparison-based algorithm determines the sorted order **only** by comparing pairs of elements using operators like `<`, `>`, `≤`, `≥`, or `==`.

```
  COMPARISON-BASED SORT — Decision Process:
  
  ┌───────────────────────────────────────────┐
  │   Compare element A with element B        │
  │                                           │
  │          A < B ?                           │
  │         /     \                            │
  │       YES      NO                         │
  │       /         \                          │
  │   A goes        B goes                    │
  │   before B      before A                  │
  │                                           │
  │   Each comparison gives 1 bit of info     │
  │   (YES or NO — two outcomes)              │
  └───────────────────────────────────────────┘
```

### Decision Tree Model

Every comparison sort can be modeled as a **binary decision tree**:

```
  Sorting [a, b, c] — Decision Tree:
  
                        a < b?
                       /      \
                     YES       NO
                    /            \
                a < c?          b < c?
               /     \         /     \
             YES     NO      YES     NO
            /         \      /         \
         b < c?      [a,c,b] c < a?   [b,c,a]  ...
        /     \              /     \
   [a,b,c]  [a,c,b]     ...      ...
  
  n! = 6 possible orderings for 3 elements
  Tree must have at least 6 leaves
  Height ≥ log₂(6) ≈ 2.58 → at least 3 comparisons worst case
```

### Key Property
- Can sort **any** type of data (numbers, strings, objects)
- Only needs a **comparison function**
- **Lower bound**: Ω(n log n) comparisons in the worst case

### Examples
| Algorithm | Best | Average | Worst |
|-----------|------|---------|-------|
| Bubble Sort | O(n) | O(n²) | O(n²) |
| Selection Sort | O(n²) | O(n²) | O(n²) |
| Insertion Sort | O(n) | O(n²) | O(n²) |
| Merge Sort | O(n log n) | O(n log n) | O(n log n) |
| Quick Sort | O(n log n) | O(n log n) | O(n²) |
| Heap Sort | O(n log n) | O(n log n) | O(n log n) |

---

## Non-Comparison-Based Sorting

Non-comparison sorts **do not compare elements** directly. Instead, they use properties of the elements (like digit values, counts, or ranges) to determine position.

```
  NON-COMPARISON SORT — Counting Sort Example:
  
  Input: [4, 2, 2, 8, 3, 3, 1]     Range: 1 to 8
  
  Step 1: COUNT occurrences (no comparisons!)
  
  Value:  1  2  3  4  5  6  7  8
  Count: [1, 2, 2, 1, 0, 0, 0, 1]
  
  Step 2: PLACE elements based on counts
  
  Output: [1, 2, 2, 3, 3, 4, 8]
  
  ┌─────────────────────────────────────┐
  │  NO element was compared with       │
  │  another element! Positions are     │
  │  determined by VALUE, not by        │
  │  relative ordering.                 │
  └─────────────────────────────────────┘
```

### Key Property
- Can achieve **O(n)** time — breaking the Ω(n log n) barrier!
- BUT: only works with **specific data types** (integers, characters)
- Requires knowledge of **data range** or **structure**

### Examples
| Algorithm | Time | Space | Requirement |
|-----------|------|-------|-------------|
| Counting Sort | O(n + k) | O(n + k) | Integer keys in range [0, k] |
| Radix Sort | O(d × n) | O(n + k) | Fixed-length keys with d digits |
| Bucket Sort | O(n + k) | O(n + k) | Uniformly distributed data |

---

## Side-by-Side Comparison

```
  ┌──────────────────────────────────────────────────────────────────┐
  │            COMPARISON-BASED                                      │
  │                                                                  │
  │  "Is 7 > 3?"  → YES → put 3 before 7                           │
  │  "Is 5 > 7?"  → NO  → put 5 before 7                           │
  │  "Is 5 > 3?"  → YES → put 3 before 5                           │
  │                                                                  │
  │  Process: Compare → Decide → Rearrange                          │
  │  Lower bound: Ω(n log n)                                        │
  │  Works on: ANY data type with ordering                          │
  ├──────────────────────────────────────────────────────────────────┤
  │         NON-COMPARISON-BASED                                     │
  │                                                                  │
  │  "Value 3 → goes to position 0"                                 │
  │  "Value 5 → goes to position 1"                                 │
  │  "Value 7 → goes to position 2"                                 │
  │                                                                  │
  │  Process: Read value → Calculate position → Place               │
  │  Time: O(n) possible                                            │
  │  Works on: Data with known range/structure                      │
  └──────────────────────────────────────────────────────────────────┘
```

---

## Why Can Non-Comparison Sorts Be Faster?

The key insight is about **information theory**:

```
  Comparison sort:
  - Each comparison yields 1 BIT of information (yes/no)
  - Need to distinguish n! permutations
  - Minimum bits needed: log₂(n!) ≈ n log₂(n)
  - Therefore: Ω(n log n) comparisons minimum
  
  Non-comparison sort:
  - Each element's VALUE directly tells its position
  - A single array access can give log₂(k) bits of info
  - Bypass the comparison information bottleneck
  - Can achieve O(n) when k (range) is manageable
```

### Visual Analogy

```
  COMPARISON SORT ≈ Sorting books by reading titles and comparing
  
    📕 📗 📘 📙
    "Is 📕 before 📗?" → YES
    "Is 📘 before 📙?" → NO
    ... many comparisons needed
  
  NON-COMPARISON SORT ≈ Sorting books into labeled shelves
  
    Shelf A: ___   Shelf B: ___   Shelf C: ___
    
    Book "B" → put on Shelf B    (no comparison with other books!)
    Book "A" → put on Shelf A
    Book "C" → put on Shelf C
    
    Done! Each book goes directly to its correct shelf.
```

---

## Trade-offs

```
  ┌─────────────────────┬─────────────────────────────────────────────┐
  │                     │  Comparison          Non-Comparison         │
  ├─────────────────────┼─────────────────────────────────────────────┤
  │  Generality         │  ✓ Any data type     ✗ Limited data types  │
  │  Speed limit        │  Ω(n log n)          O(n) possible         │
  │  Extra space        │  O(1) possible       Usually O(n + k)      │
  │  Data requirements  │  Only need ≤ / ≥     Need range/structure  │
  │  Stability          │  Depends on algo     Usually stable        │
  │  Large range (k>>n) │  Not affected        Becomes O(k) — slow! │
  │  Floating point     │  ✓ Works             ✗ Needs adaptation    │
  │  Custom objects     │  ✓ With comparator   ✗ Not applicable      │
  └─────────────────────┴─────────────────────────────────────────────┘
```

---

## Decision Guide

```
  What type of data are you sorting?
         │
         ├── General objects / strings / floats
         │         │
         │         └── Use COMPARISON sort
         │             (Merge Sort, Quick Sort, Tim Sort)
         │
         └── Integers in a known, small range?
                   │
                   ├── YES, range ≤ ~10×n
                   │         │
                   │         └── Use NON-COMPARISON sort
                   │             (Counting, Radix, or Bucket Sort)
                   │
                   └── NO, range >> n
                             │
                             └── Use COMPARISON sort
                                 (Non-comparison would waste memory)
```

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| Comparison-based | Uses pairwise comparisons (< > ≤ ≥) |
| Non-comparison | Uses data properties (counts, digits, ranges) |
| Lower bound | Comparison sorts: Ω(n log n); Non-comparison: O(n) possible |
| Generality | Comparison works on any data; Non-comparison needs specific types |
| Space trade-off | Non-comparison often needs O(n + k) extra space |
| When range k >> n | Non-comparison becomes impractical |
| Information theory | Each comparison = 1 bit; value access = log₂(k) bits |

---

## Quick Revision Questions

1. **What is the fundamental difference between comparison and non-comparison sorts?**
2. **Why can't comparison-based sorts do better than O(n log n)?**
3. **Give an example showing how Counting Sort avoids comparisons.**
4. **When would a non-comparison sort be WORSE than a comparison sort?**
5. **Can you sort floating-point numbers with Counting Sort? Why or why not?**
6. **What is the decision tree model, and what does its height represent?**

---

[← Previous: Stable vs Unstable](03-stable-vs-unstable.md) | [Next: Internal vs External →](05-internal-vs-external.md)
