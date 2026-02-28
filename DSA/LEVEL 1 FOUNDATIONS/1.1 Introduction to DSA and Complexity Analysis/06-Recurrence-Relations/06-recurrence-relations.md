# Unit 6: Recurrence Relations

[← Previous: Space Complexity](../05-Space-Complexity/05-space-complexity.md) | [Back to README](../README.md) | [Next: Mathematical Foundations →](../07-Mathematical-Foundations/07-mathematical-foundations.md)

---

## Chapter Overview

When we analyze **recursive algorithms**, we can't just count loop iterations — we need to express the time as a **recurrence relation** and then solve it. This unit teaches three major methods for solving recurrences: substitution, recursion tree, and the Master Theorem.

---

## 6.1 What is a Recurrence Relation?

### Definition

A **recurrence relation** defines a function in terms of **itself on smaller inputs**, plus a base case.

```
General form:
    T(n) = [recursive calls] + [non-recursive work]
    T(base) = constant

Example:
    T(n) = T(n-1) + O(1),      T(1) = 1     ← Factorial
    T(n) = 2T(n/2) + O(n),     T(1) = 1     ← Merge Sort
    T(n) = T(n-1) + T(n-2),    T(0)=0, T(1)=1  ← Fibonacci
```

### How to Write a Recurrence from Code

```
Step 1: Identify the recursive call(s)
Step 2: Identify how the input size changes
Step 3: Identify the non-recursive work done
Step 4: Write the base case

ALGORITHM Example(n)
    IF n ≤ 1 RETURN         ← Base case: T(1) = O(1)
    PRINT "hello"           ← O(1) work
    Example(n/2)            ← One recursive call on n/2

 Recurrence: T(n) = T(n/2) + O(1),  T(1) = O(1)
```

### Common Recurrences in DSA

```
┌──────────────────────────┬─────────────────────┬──────────────┐
│   Algorithm              │   Recurrence        │   Solution   │
├──────────────────────────┼─────────────────────┼──────────────┤
│ Linear traversal         │ T(n) = T(n-1) + 1   │   O(n)       │
│ Binary search            │ T(n) = T(n/2) + 1   │   O(log n)   │
│ Merge sort               │ T(n) = 2T(n/2) + n  │   O(n log n) │
│ Naive Fibonacci          │ T(n) = T(n-1)+T(n-2)│   O(2ⁿ)      │
│ Selection sort (recur.)  │ T(n) = T(n-1) + n   │   O(n²)      │
│ Quick sort (best/avg)    │ T(n) = 2T(n/2) + n  │   O(n log n) │
│ Quick sort (worst)       │ T(n) = T(n-1) + n   │   O(n²)      │
│ Strassen's matrix mult.  │ T(n) = 7T(n/2) + n² │   O(n^2.81)  │
│ Karatsuba multiplication │ T(n) = 3T(n/2) + n  │   O(n^1.585) │
│ Tree traversal           │ T(n) = 2T(n/2) + 1  │   O(n)       │
└──────────────────────────┴─────────────────────┴──────────────┘
```

---

## 6.2 Substitution Method

### The Approach

1. **Guess** the solution (or have an idea from experience)
2. **Prove** it correct using mathematical induction
3. Find the **constants** c and n₀

### Example 1: T(n) = T(n-1) + 1, T(1) = 1

```
Step 1: Expand (unroll) to find a pattern

 T(n) = T(n-1) + 1
      = [T(n-2) + 1] + 1         = T(n-2) + 2
      = [T(n-3) + 1] + 2         = T(n-3) + 3
      = ...
      = T(n-k) + k

 When does it stop? When n-k = 1 → k = n-1

 T(n) = T(1) + (n-1) = 1 + n - 1 = n

 Guess: T(n) = O(n)

Step 2: Prove by induction

 Claim: T(n) ≤ c·n for some constant c ≥ 1

 Base: T(1) = 1 ≤ c·1 = c  ✓ (for c ≥ 1)

 Inductive step: Assume T(k) ≤ c·k for all k < n

   T(n) = T(n-1) + 1
        ≤ c(n-1) + 1           (by induction hypothesis)
        = cn - c + 1
        ≤ cn                   (when c ≥ 1, since -c + 1 ≤ 0)

 Therefore T(n) ≤ cn → T(n) = O(n)  ✓
```

### Example 2: T(n) = T(n/2) + 1, T(1) = 1

```
Step 1: Expand

 T(n) = T(n/2) + 1
      = [T(n/4) + 1] + 1         = T(n/4) + 2
      = [T(n/8) + 1] + 2         = T(n/8) + 3
      = T(n/2^k) + k

 Stops when n/2^k = 1 → k = log₂(n)

 T(n) = T(1) + log(n) = 1 + log(n)

 Guess: T(n) = O(log n)

Step 2: Prove T(n) ≤ c·log(n) for c ≥ 1

 Assume T(k) ≤ c·log(k) for k < n

 T(n) = T(n/2) + 1
      ≤ c·log(n/2) + 1
      = c·(log n - 1) + 1
      = c·log n - c + 1
      ≤ c·log n              (when c ≥ 1)

 Therefore T(n) = O(log n)  ✓
```

### Example 3: T(n) = 2T(n/2) + n, T(1) = 1

```
Step 1: Expand

 T(n) = 2T(n/2) + n
      = 2[2T(n/4) + n/2] + n     = 4T(n/4) + n + n = 4T(n/4) + 2n
      = 4[2T(n/8) + n/4] + 2n    = 8T(n/8) + n + 2n = 8T(n/8) + 3n
      = 2^k · T(n/2^k) + k·n

 Stops when n/2^k = 1 → k = log(n)

 T(n) = 2^(log n) · T(1) + log(n) · n
      = n · 1 + n·log(n)
      = n + n·log(n)
      = O(n log n)

Step 2: Prove T(n) ≤ c·n·log(n) for suitable c

 T(n) = 2T(n/2) + n
      ≤ 2·c·(n/2)·log(n/2) + n
      = c·n·(log n - 1) + n
      = c·n·log n - c·n + n
      = c·n·log n - (c-1)·n
      ≤ c·n·log n              (when c ≥ 1)

 Therefore T(n) = O(n log n)  ✓
```

---

## 6.3 Recursion Tree Method

### The Approach

Draw a tree where:
- Each **node** represents the non-recursive work at that call
- Each **child** represents a recursive subcall
- **Sum up** all nodes to get the total work

### Example 1: T(n) = 2T(n/2) + n (Merge Sort)

```
                        n                    ← Level 0: work = n
                      /   \
                   n/2     n/2               ← Level 1: work = n/2 + n/2 = n
                  / \      / \
               n/4  n/4  n/4  n/4           ← Level 2: work = 4(n/4) = n
              / \  / \  / \  / \
             ...  ...  ...  ... ...         ← ...
            
             1  1  1  1 ... 1  1  1  1      ← Level log(n): n leaves

 Summary:
 ┌───────────┬──────────────┬──────────────────┐
 │   Level   │   # Nodes    │ Work per Level   │
 ├───────────┼──────────────┼──────────────────┤
 │     0     │     1        │    n             │
 │     1     │     2        │    n             │
 │     2     │     4        │    n             │
 │    ...    │    ...       │    n             │
 │  log(n)   │     n        │    n             │
 └───────────┴──────────────┴──────────────────┘

 Total levels: log(n) + 1
 Work per level: n
 Total: n × (log(n) + 1) = O(n log n)  ✓
```

### Example 2: T(n) = T(n/2) + 1 (Binary Search)

```
                    1                ← Level 0: work = 1
                    |
                    1                ← Level 1: work = 1
                    |
                    1                ← Level 2: work = 1
                    |
                   ...
                    |
                    1                ← Level log(n): work = 1

 Single branch — one recursive call each time
 
 Total levels: log(n)
 Work per level: 1
 Total: 1 × log(n) = O(log n)  ✓
```

### Example 3: T(n) = T(n-1) + n (Selection Sort recursive)

```
                    n               ← Level 0: work = n
                    |
                   n-1              ← Level 1: work = n-1
                    |
                   n-2              ← Level 2: work = n-2
                    |
                   ...
                    |
                    2               ← Level n-2: work = 2  
                    |
                    1               ← Level n-1: work = 1

 Total = n + (n-1) + (n-2) + ... + 2 + 1
       = n(n+1)/2
       = O(n²)  ✓
```

### Example 4: T(n) = 3T(n/4) + n² (Unusual recurrence)

```
                        n²                     ← Level 0: n²
                     /  |  \
                (n/4)² (n/4)² (n/4)²            ← Level 1: 3(n/4)² = 3n²/16
               / | \   / | \   / | \
              ...  ... ...  ...  ... ...        ← Level 2: 9(n/16)² = 9n²/256
              
 Work at level k: 3^k × (n/4^k)² = 3^k × n²/16^k = n² × (3/16)^k

 Since 3/16 < 1, this is a DECREASING geometric series!

 Level 0 dominates → total work is dominated by root

 Total ≈ n² × (1 + 3/16 + (3/16)² + ...)
       = n² × 1/(1 - 3/16)
       = n² × 16/13
       = O(n²)

 When the ratio < 1: ROOT DOMINATES → answer is the cost of level 0
```

### Example 5: T(n) = 2T(n-1) + 1 (Exponential)

```
                        1                   ← Level 0: 1
                      /    \
                    1        1              ← Level 1: 2
                  /   \    /   \
                1     1  1     1            ← Level 2: 4
              / \   / \  / \  / \
             1  1  1  1 1  1 1  1           ← Level 3: 8

 Level k: 2^k nodes, each doing O(1) work

 Depth: n (reduces by 1 each time)

 Total = 1 + 2 + 4 + ... + 2^n
       = 2^(n+1) - 1
       = O(2ⁿ)
```

### Recursion Tree — Visual Summary

```
Pattern:
                            root
                          /  |  \
                       (a children, each size n/b)
                       / |  \  / | \  / | \
                      ...
                      base cases

 Key variables:
 a = number of recursive calls (branching factor)
 b = factor by which input shrinks
 f(n) = non-recursive work at each call

 Total work = Σ (work at each level) across all levels
```

---

## 6.4 Master Theorem

### The Most Powerful Tool

The Master Theorem gives a **direct formula** for recurrences of the form:

$$T(n) = aT(n/b) + f(n)$$

where:
- $a \geq 1$ = number of subproblems
- $b > 1$ = factor by which input size shrinks
- $f(n)$ = cost of dividing + combining (non-recursive work)

### The Three Cases

```
 Compare f(n) with n^(log_b(a)):

 Let c_crit = log_b(a)   ← the "critical exponent"


 ┌─────────────────────────────────────────────────────────────────────┐
 │                                                                     │
 │  CASE 1: f(n) = O(n^(c_crit - ε))  for some ε > 0                 │
 │          f(n) grows SLOWER than n^c_crit                            │
 │          "Leaves dominate"                                          │
 │                                                                     │
 │          ═══► T(n) = Θ(n^c_crit) = Θ(n^(log_b(a)))                │
 │                                                                     │
 ├─────────────────────────────────────────────────────────────────────┤
 │                                                                     │
 │  CASE 2: f(n) = Θ(n^c_crit × log^k(n))  for k ≥ 0                │
 │          f(n) grows at the SAME RATE as n^c_crit (times log)       │
 │          "All levels contribute equally"                             │
 │                                                                     │
 │          ═══► T(n) = Θ(n^c_crit × log^(k+1)(n))                   │
 │          Most common sub-case: k=0 → T(n) = Θ(n^c_crit × log n)  │
 │                                                                     │
 ├─────────────────────────────────────────────────────────────────────┤
 │                                                                     │
 │  CASE 3: f(n) = Ω(n^(c_crit + ε))  for some ε > 0                │
 │          f(n) grows FASTER than n^c_crit                            │
 │          "Root dominates"                                           │
 │          AND a·f(n/b) ≤ c·f(n) for c < 1 (regularity condition)   │
 │                                                                     │
 │          ═══► T(n) = Θ(f(n))                                       │
 │                                                                     │
 └─────────────────────────────────────────────────────────────────────┘
```

### Visual Intuition

```
 CASE 1: Leaves dominate              CASE 2: All levels equal
 (more work at bottom)                (work spread evenly)

     ○ small work                         ○ n work
    / \                                  / \
   ○   ○ medium                         ○   ○ n work
  /\ /\                               /\ /\
 ○○○○  LOTS of work                   ○○○○ n work
 ████████████████                     ████████████
 ANSWER = leaves = n^(log_b a)        ANSWER = n^(log_b a) × log n


 CASE 3: Root dominates
 (more work at top)

     ● LOTS of work
    / \
   ○   ○ medium
  /\ /\
 ○○○○ small
 
 ANSWER = f(n) = work at root
```

### Example 1: Merge Sort — T(n) = 2T(n/2) + n

```
a = 2, b = 2, f(n) = n

Step 1: Compute c_crit = log_b(a) = log₂(2) = 1

Step 2: Compare f(n) with n^c_crit
        f(n) = n = n¹
        n^c_crit = n¹

        f(n) = Θ(n¹) → Same rate! → CASE 2 (k=0)

Step 3: Apply formula
        T(n) = Θ(n¹ × log n) = Θ(n log n)  ✓
```

### Example 2: Binary Search — T(n) = T(n/2) + 1

```
a = 1, b = 2, f(n) = 1

Step 1: c_crit = log₂(1) = 0

Step 2: f(n) = 1 = n⁰
        n^c_crit = n⁰ = 1

        f(n) = Θ(n⁰) → Same rate! → CASE 2 (k=0)

Step 3: T(n) = Θ(n⁰ × log n) = Θ(log n)  ✓
```

### Example 3: T(n) = 4T(n/2) + n

```
a = 4, b = 2, f(n) = n

Step 1: c_crit = log₂(4) = 2

Step 2: f(n) = n = n¹
        n^c_crit = n²

        n¹ vs n²  →  f(n) grows SLOWER → CASE 1 (ε = 1)

Step 3: T(n) = Θ(n^c_crit) = Θ(n²)  ✓
```

### Example 4: T(n) = 3T(n/4) + n²

```
a = 3, b = 4, f(n) = n²

Step 1: c_crit = log₄(3) = ln(3)/ln(4) ≈ 0.793

Step 2: f(n) = n²
        n^c_crit = n^0.793

        n² vs n^0.793  →  f(n) grows FASTER → CASE 3 (ε ≈ 1.207)

Step 3: Check regularity: 3·(n/4)² = 3n²/16 ≤ c·n² for c = 3/16 < 1 ✓

        T(n) = Θ(f(n)) = Θ(n²)  ✓
```

### Example 5: Strassen's Matrix — T(n) = 7T(n/2) + n²

```
a = 7, b = 2, f(n) = n²

Step 1: c_crit = log₂(7) ≈ 2.807

Step 2: f(n) = n² 
        n^c_crit = n^2.807

        n² vs n^2.807  →  f(n) grows SLOWER → CASE 1 (ε ≈ 0.807)

Step 3: T(n) = Θ(n^log₂(7)) = Θ(n^2.807)  ✓

 This is why Strassen's is faster than standard O(n³) matrix multiplication!
```

### Master Theorem Quick-Reference

```
┌────────────────────────────────────┬──────────┬──────────────────────────────┐
│ Recurrence                        │   Case   │  Solution                    │
├────────────────────────────────────┼──────────┼──────────────────────────────┤
│ T(n) = T(n/2) + 1                 │ Case 2   │ Θ(log n)                     │
│ T(n) = T(n/2) + n                 │ Case 3   │ Θ(n)                         │
│ T(n) = 2T(n/2) + 1                │ Case 1   │ Θ(n)                         │
│ T(n) = 2T(n/2) + n                │ Case 2   │ Θ(n log n)                   │
│ T(n) = 2T(n/2) + n²               │ Case 3   │ Θ(n²)                        │
│ T(n) = 4T(n/2) + n                │ Case 1   │ Θ(n²)                        │
│ T(n) = 4T(n/2) + n²               │ Case 2   │ Θ(n² log n)                  │
│ T(n) = 4T(n/2) + n³               │ Case 3   │ Θ(n³)                        │
│ T(n) = 7T(n/2) + n²               │ Case 1   │ Θ(n^2.807)                   │
│ T(n) = 3T(n/4) + n·log n          │ Case 3   │ Θ(n log n)                   │
│ T(n) = 2T(n/4) + n^0.5            │ Case 2   │ Θ(√n · log n)                │
└────────────────────────────────────┴──────────┴──────────────────────────────┘
```

---

## 6.5 Examples with Common Algorithms

### Merge Sort — Complete Analysis

```
ALGORITHM MergeSort(A, lo, hi)
    IF lo < hi THEN
        mid ← (lo + hi) / 2
        MergeSort(A, lo, mid)         ← T(n/2)
        MergeSort(A, mid+1, hi)       ← T(n/2)
        Merge(A, lo, mid, hi)         ← O(n)

Recurrence: T(n) = 2T(n/2) + O(n)

Using Master Theorem:
  a=2, b=2, f(n)=n
  c_crit = log₂2 = 1
  f(n) = n = Θ(n¹) = Θ(n^c_crit) → Case 2
  
  T(n) = Θ(n log n)

Verification with recursion tree:
  Level 0: n
  Level 1: n/2 + n/2 = n
  Level 2: n/4 + n/4 + n/4 + n/4 = n
  ...
  Level log n: 1 + 1 + ... + 1 = n

  Total: n × log(n) levels = Θ(n log n)  ✓
```

### Quick Sort — Best/Average vs Worst

```
BEST/AVERAGE: Pivot splits evenly

  T(n) = 2T(n/2) + O(n)  →  Θ(n log n)   (same as merge sort)


WORST: Pivot is always min or max (already sorted input)

  T(n) = T(n-1) + O(n)

  Expansion:
  T(n) = T(n-1) + n
       = T(n-2) + (n-1) + n
       = T(n-3) + (n-2) + (n-1) + n
       = 1 + 2 + 3 + ... + n
       = n(n+1)/2
       = Θ(n²)

  This is why quick sort has O(n²) worst case!
  
  Visual (worst case):
  
  [1, 2, 3, 4, 5, 6, 7, 8]  pivot = 1
  
      1               ← partition: O(n) work
      |
  [2, 3, 4, 5, 6, 7, 8]  pivot = 2
      |
      2               ← partition: O(n-1) work
      |
  [3, 4, 5, 6, 7, 8]
      ...
  
  Unbalanced tree with n levels, each O(n) work → O(n²)
```

### Binary Search — Recursive Analysis

```
T(n) = T(n/2) + O(1)

Master Theorem: a=1, b=2, f(n)=1
  c_crit = log₂(1) = 0
  f(n) = 1 = n⁰ = Θ(n^c_crit) → Case 2
  
  T(n) = Θ(log n)

Substitution verification:
  T(n) = T(n/2) + 1
       = T(n/4) + 1 + 1
       = T(n/8) + 3
       = T(n/2^k) + k
  
  At k = log₂(n): T(1) + log(n) = 1 + log(n) = Θ(log n) ✓
```

---

## 6.6 Solving Complex Recurrences

### When Master Theorem Doesn't Apply

The Master Theorem does NOT apply when:

```
1. The subproblem sizes are unequal:
   T(n) = T(n/3) + T(2n/3) + n   ← different sizes n/3 and 2n/3

2. a < 1:
   T(n) = 0.5·T(n/2) + n   ← a must be ≥ 1

3. The recurrence is not of divide-and-conquer form:
   T(n) = T(n-1) + n   ← subtractive, not divisive
   T(n) = T(√n) + 1    ← reduction by square root

Use recursion tree or substitution for these cases.
```

### Example: Unequal Split — T(n) = T(n/3) + T(2n/3) + n

```
Recursion tree:

                        n                   ← Level 0: n
                      /    \
                   n/3      2n/3            ← Level 1: n/3 + 2n/3 = n
                  /  \      /   \
               n/9  2n/9  2n/9  4n/9       ← Level 2: n/9+2n/9+2n/9+4n/9 = n
              / \   / \    / \    / \
             ...   ...   ...   ...          ← Every level sums to n!

 How deep does it go?
 Shortest path: n → n/3 → n/9 → ... → 1  → depth = log₃(n)
 Longest path:  n → 2n/3 → 4n/9 → ... → 1  → depth = log_{3/2}(n)

 Each level sums to n (can be proven)
 Number of full levels ≈ log_{3/2}(n) = O(log n)

 Total: n × O(log n) = O(n log n)

 This is Quick Sort with 1/3 - 2/3 split — still O(n log n)!
```

### Example: Subtractive — T(n) = T(n-1) + n

```
This is NOT divide-and-conquer, so Master Theorem doesn't apply.

Expansion:
 T(n) = T(n-1) + n
      = T(n-2) + (n-1) + n
      = T(n-3) + (n-2) + (n-1) + n
      = T(1) + 2 + 3 + ... + n
      = 1 + 2 + 3 + ... + n
      = n(n+1)/2
      = O(n²)
```

### Example: Square Root — T(n) = T(√n) + 1

```
Substitution: Let m = log₂(n), so n = 2^m

T(2^m) = T(2^(m/2)) + 1

Let S(m) = T(2^m)

S(m) = S(m/2) + 1

This IS the Master Theorem form! a=1, b=2, f=1
c_crit = log₂(1) = 0 → Case 2
S(m) = Θ(log m)

Back-substitute: m = log n
T(n) = Θ(log(log n))

So T(n) = T(√n) + 1 = O(log log n)  — VERY fast!
```

### Example: Tower of Hanoi — T(n) = 2T(n-1) + 1

```
Expansion:
 T(n) = 2T(n-1) + 1
      = 2[2T(n-2) + 1] + 1       = 4T(n-2) + 2 + 1
      = 4[2T(n-3) + 1] + 3       = 8T(n-3) + 4 + 2 + 1
      = 2^k · T(n-k) + (2^k - 1)

 At k = n-1:  T(1) = 1
 T(n) = 2^(n-1) · 1 + (2^(n-1) - 1)
      = 2^(n-1) + 2^(n-1) - 1
      = 2^n - 1
      = O(2ⁿ)

 Tower of Hanoi: Moving n disks requires 2ⁿ - 1 moves.
 For 64 disks: 2⁶⁴ - 1 = 18,446,744,073,709,551,615 moves!
```

### Solving Recurrences — Decision Flowchart

```
┌──────────────────────────────────────────┐
│ T(n) = aT(n/b) + f(n)?                  │
│ (divide-and-conquer form?)               │
└─────┬────────────────────┬───────────────┘
      │ YES                │ NO
      ▼                    ▼
  ┌────────────┐    ┌─────────────────────┐
  │ Use Master │    │ T(n) = T(n-c)+f(n)? │
  │ Theorem    │    │ (subtract form?)     │
  └────────────┘    └──────┬──────────────┘
                           │
                    ┌──────┴──────┐
                    │ YES         │ NO
                    ▼             ▼
              ┌──────────┐  ┌──────────────┐
              │ Expand &  │  │ Substitution │
              │ find sum  │  │ or Change    │
              │ (usually  │  │ of variable  │
              │ arithmetic│  └──────────────┘
              │ series)   │
              └──────────┘
```

---

## Summary Table

| Method | When to Use | Key Idea |
|--------|-------------|----------|
| Substitution | Any recurrence; need to guess answer | Guess → Prove by induction |
| Recursion Tree | Visual understanding; any recurrence | Draw tree, sum all levels |
| Master Theorem | T(n) = aT(n/b) + f(n) form only | Compare f(n) with n^(log_b a) |
| Case 1 | f(n) polynomially smaller | Leaves dominate → Θ(n^(log_b a)) |
| Case 2 | f(n) same rate (with log factor) | All levels equal → Θ(n^(log_b a) · log^(k+1) n) |
| Case 3 | f(n) polynomially larger | Root dominates → Θ(f(n)) |
| T(n-1) + c | Linear subtraction, constant work | O(n) |
| T(n-1) + n | Linear subtraction, linear work | O(n²) |
| 2T(n-1) + c | Double branching | O(2ⁿ) |
| T(√n) + 1 | Square root reduction | O(log log n) |

---

## Quick Revision Questions

**Q1.** Write the recurrence relation for Binary Search and solve it using the Master Theorem.

<details>
<summary>Answer</summary>
T(n) = T(n/2) + O(1). Master Theorem: a=1, b=2, f(n)=1, c_crit = log₂(1) = 0. f(n) = 1 = n⁰ = Θ(n^c_crit) → Case 2. T(n) = Θ(n⁰ · log n) = Θ(log n).
</details>

**Q2.** Why can't we use the Master Theorem for T(n) = T(n-1) + n?

<details>
<summary>Answer</summary>
The Master Theorem requires the form T(n) = aT(n/b) + f(n) where the input is DIVIDED by b. In T(n) = T(n-1) + n, the input is SUBTRACTED by 1, not divided. This is a subtractive recurrence, not divisive. Solve it by expansion: T(n) = n + (n-1) + ... + 1 = n(n+1)/2 = O(n²).
</details>

**Q3.** For the recurrence T(n) = 4T(n/2) + n³, which Master Theorem case applies and what is T(n)?

<details>
<summary>Answer</summary>
a=4, b=2, f(n)=n³, c_crit = log₂(4) = 2. f(n) = n³ vs n^c_crit = n². Since n³ grows faster than n² (by polynomial factor n¹), Case 3 applies. Check regularity: 4·(n/2)³ = 4n³/8 = n³/2 ≤ c·n³ for c=1/2 < 1 ✓. T(n) = Θ(n³).
</details>

**Q4.** Draw the recursion tree for T(n) = 3T(n/3) + n and determine the solution.

<details>
<summary>Answer</summary>
Level 0: n (root). Level 1: 3 nodes of n/3 each = n total. Level 2: 9 nodes of n/9 each = n total. Every level sums to n. Depth = log₃(n). Total = n × log₃(n) = Θ(n log n). (Same as Master Theorem Case 2: a=3, b=3, c_crit=1, f(n)=n=Θ(n¹).)
</details>

**Q5.** What is the time complexity of the Tower of Hanoi recurrence T(n) = 2T(n-1) + 1? Why?

<details>
<summary>Answer</summary>
T(n) = O(2ⁿ). Each call spawns 2 recursive calls, and the input decreases by 1 (not halved). Expanding: T(n) = 2ⁿ - 1. This is exponential because the branching factor (2) is applied n times (depth n), giving 2ⁿ total operations.
</details>

**Q6.** Solve T(n) = T(√n) + 1. What technique do you use and what is the result?

<details>
<summary>Answer</summary>
Change of variable: let m = log₂(n), so n = 2^m and √n = 2^(m/2). Then S(m) = T(2^m) = S(m/2) + 1. Apply Master Theorem: a=1, b=2, f=1 → Case 2 → S(m) = Θ(log m). Back-substitute: T(n) = Θ(log(log n)). This is extremely fast — even for n = 2⁶⁴, only about 6 operations!
</details>

---

[← Previous: Space Complexity](../05-Space-Complexity/05-space-complexity.md) | [Back to README](../README.md) | [Next: Mathematical Foundations →](../07-Mathematical-Foundations/07-mathematical-foundations.md)
