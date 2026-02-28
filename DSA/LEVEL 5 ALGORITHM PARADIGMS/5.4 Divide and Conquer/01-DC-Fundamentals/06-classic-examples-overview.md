# Chapter 6: Classic Examples Overview

## Overview

This chapter provides a rapid tour of the most important D&C algorithms you'll encounter. Each is covered in depth in later units — this is your roadmap to see how D&C manifests across different problem domains.

---

## The Classic D&C Algorithms

```
┌─────────────────────────────────────────────────────────────────┐
│                    CLASSIC D&C ALGORITHMS                       │
│                                                                 │
│  SEARCHING        SORTING          SELECTION                    │
│  ┌────────────┐   ┌────────────┐   ┌────────────┐             │
│  │ Binary     │   │ Merge Sort │   │ Quick      │             │
│  │ Search     │   │ Quick Sort │   │ Select     │             │
│  └────────────┘   └────────────┘   └────────────┘             │
│                                                                 │
│  ARITHMETIC       GEOMETRY         ADVANCED                     │
│  ┌────────────┐   ┌────────────┐   ┌────────────┐             │
│  │ Karatsuba  │   │ Closest    │   │ FFT        │             │
│  │ Strassen   │   │ Pair       │   │ Skyline    │             │
│  │            │   │ Convex Hull│   │ Beautiful  │             │
│  └────────────┘   └────────────┘   │ Array      │             │
│                                    └────────────┘             │
└─────────────────────────────────────────────────────────────────┘
```

---

## 1. Binary Search

**Problem**: Find a target value in a sorted array.

```
Array: [2, 5, 8, 12, 16, 23, 38, 56, 72, 91]
Target: 23

Step 1: mid = arr[4] = 16,  23 > 16 → search right half
        [2, 5, 8, 12, 16, |23, 38, 56, 72, 91]
                            ▲ search here

Step 2: mid = arr[7] = 56,  23 < 56 → search left half
        [23, 38, 56, 72, 91]
         ▲ search here

Step 3: mid = arr[5] = 23,  23 == 23 → FOUND! ✓
```

| Aspect | Detail |
|--------|--------|
| Divide | Compare with middle element |
| Conquer | Recurse on ONE half only |
| Combine | Return the result (trivial) |
| Complexity | **O(log n)** |

---

## 2. Merge Sort

**Problem**: Sort an array in ascending order.

```
[38, 27, 43, 3, 9, 82, 10]

DIVIDE into halves:
[38, 27, 43, 3]        [9, 82, 10]

[38, 27]  [43, 3]      [9, 82]  [10]

[38] [27] [43] [3]     [9] [82] [10]    ← Base cases

MERGE back up:
[27, 38]  [3, 43]      [9, 82]  [10]

[3, 27, 38, 43]        [9, 10, 82]

[3, 9, 10, 27, 38, 43, 82]  ✓ Sorted!
```

| Aspect | Detail |
|--------|--------|
| Divide | Split at midpoint |
| Conquer | Sort both halves recursively |
| Combine | Merge two sorted arrays |
| Complexity | **O(n log n)** always |

---

## 3. Quick Sort

**Problem**: Sort an array in-place.

```
[7, 2, 1, 6, 8, 5, 3, 4]    pivot = 4

PARTITION:
[2, 1, 3] [4] [7, 6, 8, 5]   ← Elements < 4 | 4 | Elements > 4

Recurse on each part:
[1, 2, 3] [4] [5, 6, 7, 8]

[1, 2, 3, 4, 5, 6, 7, 8]  ✓ Sorted!
```

| Aspect | Detail |
|--------|--------|
| Divide | Partition around pivot |
| Conquer | Sort both partitions recursively |
| Combine | Nothing (in-place partition does the work) |
| Complexity | **O(n log n)** average, O(n²) worst |

---

## 4. Quick Select (Kth Smallest Element)

**Problem**: Find the kth smallest element without fully sorting.

```
Array: [7, 2, 1, 6, 8, 5, 3, 4],  k = 3 (3rd smallest)

PARTITION with pivot = 4:
[2, 1, 3] [4] [7, 6, 8, 5]
 index:     3

k = 3 → 3rd smallest is at index 2 (0-based)
Index 2 < 3, so it's in the left partition!

Recurse on left: [2, 1, 3], k = 3
PARTITION with pivot = 3:
[2, 1] [3] → 3rd smallest = 3 ✓
```

| Aspect | Detail |
|--------|--------|
| Divide | Partition around pivot |
| Conquer | Recurse on ONE side only |
| Combine | Return the result |
| Complexity | **O(n)** average |

---

## 5. Closest Pair of Points

**Problem**: Find the two closest points in a set of 2D points.

```
Points plotted:

y
│     •(2,7)
│                 •(7,7)
│  •(1,4)
│        •(4,4)
│                    •(8,3)
│     •(3,1)
│              •(6,1)
└─────────────────────────── x

DIVIDE by x-coordinate:
Left half:  (1,4),(2,7),(3,1)
Right half: (4,4),(6,1),(7,7),(8,3)

CONQUER: Find closest pair in each half
COMBINE: Check if any pair across the dividing line is closer
```

| Aspect | Detail |
|--------|--------|
| Divide | Split points by x-coordinate |
| Conquer | Find closest pair in each half |
| Combine | Check "strip" near dividing line |
| Complexity | **O(n log n)** |

---

## 6. Strassen's Matrix Multiplication

**Problem**: Multiply two n×n matrices faster than O(n³).

```
Standard multiplication: 8 multiplications of (n/2 × n/2) matrices
Strassen's trick: Only 7 multiplications needed!

┌         ┐   ┌         ┐   ┌                 ┐
│ A11 A12 │ × │ B11 B12 │ = │ C11        C12  │
│ A21 A22 │   │ B21 B22 │   │ C21        C22  │
└         ┘   └         ┘   └                 ┘

Standard: 8 multiplications → T(n) = 8T(n/2) + O(n²) → O(n³)
Strassen: 7 multiplications → T(n) = 7T(n/2) + O(n²) → O(n^2.807)
```

| Aspect | Detail |
|--------|--------|
| Divide | Split matrices into 4 quadrants |
| Conquer | 7 recursive multiplications (not 8) |
| Combine | Add/subtract sub-results |
| Complexity | **O(n^2.807)** |

---

## 7. Karatsuba Multiplication

**Problem**: Multiply two n-digit numbers faster than O(n²).

```
Multiply 1234 × 5678:

Split each number:
1234 → a=12, b=34
5678 → c=56, d=78

Standard: 4 multiplications: ac, ad, bc, bd
Karatsuba: 3 multiplications:
  p1 = a×c = 12×56 = 672
  p2 = b×d = 34×78 = 2652
  p3 = (a+b)×(c+d) = 46×134 = 6164

Result = p1×10^4 + (p3-p1-p2)×10^2 + p2
       = 6720000 + 284000 + 2652
       = 7006652 ✓  (1234 × 5678 = 7006652)
```

| Aspect | Detail |
|--------|--------|
| Divide | Split digits in half |
| Conquer | 3 multiplications instead of 4 |
| Combine | Shift and add results |
| Complexity | **O(n^1.585)** |

---

## 8. Maximum Subarray (D&C Approach)

**Problem**: Find the contiguous subarray with the maximum sum.

```
Array: [-2, 1, -3, 4, -1, 2, 1, -5, 4]

DIVIDE at mid:
Left:  [-2, 1, -3, 4]     max subarray sum in left
Right: [-1, 2, 1, -5, 4]  max subarray sum in right
Cross: Subarray crossing the midpoint

LEFT max:  4
RIGHT max: 4
CROSS max: [4, -1, 2, 1] = 6  ← extends from mid to both sides

Answer: max(4, 4, 6) = 6 ✓
```

| Aspect | Detail |
|--------|--------|
| Divide | Split array at midpoint |
| Conquer | Find max subarray in each half |
| Combine | Find max crossing subarray, take overall max |
| Complexity | **O(n log n)** |

---

## 9. Counting Inversions

**Problem**: Count pairs (i,j) where i < j but arr[i] > arr[j].

```
Array: [2, 4, 1, 3, 5]

Inversions: (2,1), (4,1), (4,3) → Total = 3

Using modified Merge Sort:
[2, 4, 1, 3, 5]
→ Split: [2,4] and [1,3,5]
→ Sort & count in [2,4]: 0 inversions
→ Sort & count in [1,3,5]: 0 inversions
→ Merge & count cross-inversions:
  2>1 (and 4>1): 2 inversions from 1
  4>3: 1 inversion from 3
  Total = 3 ✓
```

| Aspect | Detail |
|--------|--------|
| Divide | Split array at midpoint |
| Conquer | Count inversions in each half |
| Combine | Count cross-inversions during merge |
| Complexity | **O(n log n)** |

---

## Algorithm Comparison Chart

| Algorithm | Divide Into | Recurse On | Combine | Time | Space |
|-----------|:-----------:|:----------:|:-------:|:----:|:-----:|
| Binary Search | 2 halves | 1 half | O(1) | O(log n) | O(1) |
| Merge Sort | 2 halves | Both | O(n) merge | O(n log n) | O(n) |
| Quick Sort | 2 partitions | Both | O(1) | O(n log n)* | O(log n) |
| Quick Select | 2 partitions | 1 part | O(1) | O(n)* | O(1) |
| Closest Pair | 2 halves (by x) | Both | O(n) strip | O(n log n) | O(n) |
| Strassen's | 4 quadrants | 7 sub-mults | O(n²) add | O(n^2.807) | O(n²) |
| Karatsuba | 2 halves (digits) | 3 sub-mults | O(n) add | O(n^1.585) | O(n) |
| Max Subarray | 2 halves | Both | O(n) cross | O(n log n) | O(log n) |
| Inversions | 2 halves | Both | O(n) count | O(n log n) | O(n) |

*Average case

---

## The D&C Trick: Reducing Recursive Calls

The biggest breakthroughs in D&C come from reducing the number of subproblems:

```
STANDARD → OPTIMIZED:

Matrix Multiply:  8 sub-multiplications → 7 (Strassen)
  O(n³) → O(n^2.807)

Integer Multiply: 4 sub-multiplications → 3 (Karatsuba)
  O(n²) → O(n^1.585)

Binary Search:    2 possible halves → only explore 1
  O(n) → O(log n)

Quick Select:     2 partitions → only recurse on 1
  O(n log n) → O(n)

KEY INSIGHT: Reducing subproblems from a to (a-1) can
dramatically change the asymptotic complexity!
```

---

## Real-World Applications Summary

| Domain | Algorithm | Application |
|--------|-----------|-------------|
| Databases | Merge Sort | External sorting of large datasets |
| Web Search | Binary Search | Index lookup, autocomplete |
| Cryptography | Karatsuba | RSA key generation (large number multiply) |
| Machine Learning | Strassen | Neural network matrix operations |
| Graphics | Closest Pair | Collision detection |
| Signal Processing | FFT | Audio compression, image processing |
| Games | Quick Select | Median finding for game AI |
| Operating Systems | Quick Sort | File system sorting |

---

## Summary Table

| Classic Example | Key Idea | Where to Learn More |
|-----------------|----------|---------------------|
| Binary Search | Eliminate half the search space | Unit 3 |
| Merge Sort | Sort halves, merge efficiently | Unit 4 |
| Quick Sort | Partition around pivot | Unit 5 |
| Quick Select | Partition, recurse on ONE side | Unit 6 |
| Closest Pair | Split by coordinate, check strip | Unit 7 |
| Strassen's | 7 multiplications instead of 8 | Unit 8 |
| Karatsuba | 3 multiplications instead of 4 | Unit 10 |
| Max Subarray | Check crossing subarray | Unit 9 |

---

## Quick Revision Questions

1. **How does Binary Search differ from other D&C algorithms in terms of the number of subproblems it recurses on?**
2. **What is the key insight that makes Strassen's algorithm faster than standard matrix multiplication?**
3. **Why is Merge Sort O(n log n) in the worst case but Quick Sort is O(n²) in the worst case?**
4. **How is the Counting Inversions problem related to Merge Sort?**
5. **What makes Quick Select O(n) on average while Quick Sort is O(n log n)?**
6. **Name two D&C algorithms that improve performance by reducing the number of recursive calls.**

---

[← Previous: Advantages and Disadvantages](05-advantages-and-disadvantages.md) | [Next Unit: Master Theorem →](../02-Master-Theorem/01-master-theorem-statement.md)
