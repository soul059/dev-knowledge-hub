# Unit 2: Algorithm Analysis Basics

[← Previous: Introduction to DSA](../01-Introduction-to-DSA/01-introduction-to-dsa.md) | [Back to README](../README.md) | [Next: Big O Notation →](../03-Big-O-Notation/03-big-o-notation.md)

---

## Chapter Overview

Now that we know what data structures and algorithms are, we need a way to **measure** and **compare** them. This unit introduces the core framework for analyzing algorithm efficiency — time complexity, space complexity, and asymptotic analysis.

---

## 2.1 Why Analyze Algorithms?

### The Core Problem

For any given problem, there are usually **multiple algorithms** that solve it. How do we decide which one is **better**?

```
 Problem: Sort an array of n numbers

 Algorithm A: Bubble Sort    →  ~n² comparisons
 Algorithm B: Merge Sort     →  ~n log n comparisons
 Algorithm C: Quick Sort     →  ~n log n comparisons (avg)

 For n = 1,000,000:
   Bubble Sort:   1,000,000,000,000 operations  (~16 minutes at 10⁹ ops/sec)
   Merge Sort:    20,000,000 operations          (~0.02 seconds)
                                                  
   Same problem, VASTLY different performance!
```

### Two Ways to Measure Performance

```
┌─────────────────────────────────┬─────────────────────────────────┐
│    A Posteriori Analysis        │    A Priori Analysis            │
│    (After implementation)       │    (Before implementation)      │
├─────────────────────────────────┼─────────────────────────────────┤
│ • Run the program               │ • Analyze the algorithm         │
│ • Measure actual time/memory    │ • Count operations as f(n)     │
│ • Results depend on:            │ • Results are:                  │
│   - Hardware (CPU, RAM)         │   - Hardware independent        │
│   - Programming language        │   - Language independent        │
│   - Input data                  │   - Based on input size n       │
│   - Other running processes     │   - Mathematical & objective    │
│                                 │                                 │
│ ✗ Not reliable for comparison   │ ✓ Standard way to compare       │
└─────────────────────────────────┴─────────────────────────────────┘
```

> **We always use A Priori (theoretical) analysis** because it gives us hardware-independent, objective results.

### What Do We Count?

We count **basic operations** (also called elementary operations):

```
Basic Operations:
  • Assignments      (x = 5)
  • Comparisons      (if x > y)
  • Arithmetic ops   (x + y, x * y)
  • Array access     (arr[i])
  • Return statement (return x)

Each basic operation takes CONSTANT time → we call it O(1).
```

---

## 2.2 Time Complexity

### Definition

**Time complexity** describes the amount of time an algorithm takes as a **function of the input size n**.

> We don't measure actual seconds — we count the **number of basic operations** relative to `n`.

### Example 1: Constant Time — O(1)

```
ALGORITHM GetFirst(A)
    RETURN A[0]         ← 1 operation, regardless of array size

 n = 10      → 1 operation
 n = 1000    → 1 operation
 n = 1000000 → 1 operation

 Time = constant = O(1)
```

### Example 2: Linear Time — O(n)

```
ALGORITHM Sum(A, n)
    total ← 0                   ← 1 operation
    FOR i ← 0 TO n-1 DO         ← loop runs n times
        total ← total + A[i]    ← 2 operations per iteration (access + add)
    RETURN total                 ← 1 operation

 Total operations = 1 + n×2 + 1 = 2n + 2

 As n grows large, constants don't matter → O(n)
```

**Counting operations in detail:**

```
Statement                    # of times executed
─────────────────────────────────────────────────
total ← 0                   1
i ← 0                       1
i < n (comparison)           n + 1  (last check fails)
i++ (increment)              n
total ← total + A[i]        n
RETURN total                 1
─────────────────────────────────────────────────
Total                        3n + 4  →  O(n)
```

### Example 3: Quadratic Time — O(n²)

```
ALGORITHM PrintPairs(A, n)
    FOR i ← 0 TO n-1 DO              ← outer loop: n times
        FOR j ← 0 TO n-1 DO          ← inner loop: n times (per outer)
            PRINT A[i], A[j]          ← 1 operation

 Total = n × n = n²  →  O(n²)

 Trace for n=3, A=[1,2,3]:
 i=0: (1,1) (1,2) (1,3)     ← 3 prints
 i=1: (2,1) (2,2) (2,3)     ← 3 prints
 i=2: (3,1) (3,2) (3,3)     ← 3 prints
                              Total: 9 = 3² prints
```

### Visualizing Growth Rates

```
 Operations
    ▲
 100│                                          * n²
    │                                     *
    │                                *
 75 │                           *
    │                      *
    │                 *
 50 │            *                        · n
    │       *                        ·         
    │  *                        ·
 25 │ *                    ·
    │*                ·
    │*           ·                    - log n
    │*      · - - - - - - - - - - - -
    │*  · - - - - - - - - - - - - - -  ___ 1
    │·- - - - - - - - - - - - - - - -
    └────────────────────────────────────► n
    0    2    4    6    8   10
```

---

## 2.3 Space Complexity

### Definition

**Space complexity** measures the total **memory** an algorithm needs as a function of input size n.

```
Total Space = Input Space + Auxiliary Space
                  │              │
                  ▼              ▼
         Space for input   Extra space needed
         data itself       by the algorithm
```

### Example: In-place vs Extra Space

```
ALGORITHM SumArray(A, n)        ALGORITHM CopyAndSort(A, n)
    total ← 0                      B ← new Array[n]        ← n extra space!
    FOR i ← 0 TO n-1               FOR i ← 0 TO n-1
        total ← total + A[i]           B[i] ← A[i]
    RETURN total                    Sort(B)
                                    RETURN B
    
    Auxiliary space: O(1)           Auxiliary space: O(n)
    (only 'total' and 'i')         (entire copy of array)
```

### Space Complexity Examples

| Scenario | Space Used | Complexity |
|----------|-----------|------------|
| Single variable | `int x = 5;` | O(1) |
| Fixed number of variables | `int a, b, c;` | O(1) |
| Array of size n | `int arr[n];` | O(n) |
| 2D matrix n×n | `int matrix[n][n];` | O(n²) |
| Recursive call with depth n | Call stack grows to n | O(n) |
| Recursive call with depth log n | Call stack grows to log n | O(log n) |

---

## 2.4 Best, Average, Worst Case

### Why Three Cases?

An algorithm's performance can **vary** depending on the specific input, not just the input size.

### Example: Linear Search

```
ALGORITHM LinearSearch(A, n, target)
    FOR i ← 0 TO n-1
        IF A[i] = target THEN
            RETURN i
    RETURN -1

A = [5, 3, 8, 1, 9, 2, 7]
```

```
┌──────────────┬────────────────────────────────┬──────────────┐
│    Case      │         Scenario               │ Comparisons  │
├──────────────┼────────────────────────────────┼──────────────┤
│ BEST CASE    │ Target is first element (5)     │     1        │
│              │ Found immediately               │   O(1)       │
├──────────────┼────────────────────────────────┼──────────────┤
│ AVERAGE CASE │ Target is somewhere in middle   │    n/2       │
│              │ On average, check half the list │   O(n)       │
├──────────────┼────────────────────────────────┼──────────────┤
│ WORST CASE   │ Target is last element (7)      │     n        │
│              │ OR target is not in array       │   O(n)       │
└──────────────┴────────────────────────────────┴──────────────┘
```

### Visual: The Three Cases

```
Best Case: Target at index 0
 [5, 3, 8, 1, 9, 2, 7]    Search for 5
  ↑ Found! (1 comparison)


Average Case: Target somewhere in middle
 [5, 3, 8, 1, 9, 2, 7]    Search for 1
  ✗  ✗  ✗  ↑ Found! (4 comparisons ≈ n/2)


Worst Case: Target at end or not present
 [5, 3, 8, 1, 9, 2, 7]    Search for 7
  ✗  ✗  ✗  ✗  ✗  ✗  ↑ Found! (7 comparisons = n)

 [5, 3, 8, 1, 9, 2, 7]    Search for 4
  ✗  ✗  ✗  ✗  ✗  ✗  ✗  Not found! (7 comparisons = n)
```

### Which Case Do We Usually Care About?

```
┌──────────────┬────────────────────────────────────────────────┐
│    Case      │         Usage                                  │
├──────────────┼────────────────────────────────────────────────┤
│ Best Case    │ Rarely useful (too optimistic)                 │
│ Average Case │ Useful but hard to compute (need probability)  │
│ Worst Case   │ MOST COMMONLY USED — guarantees upper bound    │ ◄── Default
└──────────────┴────────────────────────────────────────────────┘

When someone says "This algorithm is O(n²)" they typically mean
the WORST CASE is O(n²).
```

### Example: Insertion Sort — All Three Cases

```
Best Case: Already sorted array
 [1, 2, 3, 4, 5]  → Each element compared once with predecessor
 Comparisons: n - 1 = O(n)

Average Case: Random order
 [3, 1, 4, 5, 2]  → On average n²/4 comparisons
 Comparisons: O(n²)

Worst Case: Reverse sorted array
 [5, 4, 3, 2, 1]  → Each element compared with ALL predecessors
 
 Pass 1: [4,5,3,2,1]  → 1 comparison
 Pass 2: [3,4,5,2,1]  → 2 comparisons
 Pass 3: [2,3,4,5,1]  → 3 comparisons
 Pass 4: [1,2,3,4,5]  → 4 comparisons
 Total: 1+2+3+4 = 10 = n(n-1)/2 = O(n²)
```

---

## 2.5 Asymptotic Analysis

### What is Asymptotic Analysis?

It's the study of how an algorithm's performance scales as input size **approaches infinity** (n → ∞).

> We care about the **growth rate**, not the exact count.

### The Key Idea

```
 Exact count:  T(n) = 5n² + 3n + 7

 As n → ∞:
   n=10:     5(100) + 3(10) + 7     = 537        (n² dominates: 93%)
   n=100:    5(10000) + 3(100) + 7  = 50,307     (n² dominates: 99.4%)
   n=1000:   5(1000000) + 3(1000)+7 = 5,003,007  (n² dominates: 99.94%)
   
   The 3n + 7 part becomes NEGLIGIBLE.
   So T(n) ≈ 5n² → asymptotically O(n²)
```

### Why Drop Constants?

```
 Algorithm A: T(n) = 100n      → O(n)
 Algorithm B: T(n) = 2n²       → O(n²)

 Small n:   A is slower (100×10=1000 vs 2×100=200)
 n=100:     A: 10,000  vs  B: 20,000  → About equal
 n=1000:    A: 100,000 vs  B: 2,000,000  → A is 20x faster!
 n=10000:   A: 1,000,000 vs B: 200,000,000 → A is 200x faster!!

 The GROWTH RATE always wins for large n.
```

```
 Time
  ▲
  │                              ╱ 2n²
  │                           ╱
  │                        ╱
  │                     ╱
  │                  ╱
  │               ╱            ╱ 100n  
  │            ╱           ╱
  │         ╱          ╱
  │      ╱ ────────╱───── Crossover point
  │   ╱    ╱   
  │╱   ╱
  │ ╱   
  └────────────────────────────────► n
  
  After the crossover, O(n) ALWAYS beats O(n²)
```

### The Three Asymptotic Notations (Preview)

```
┌─────────────────────────────────────────────────────────────┐
│                                                             │
│  Big O (O)     →  Upper bound   →  "At most this fast"     │
│                    Worst case guarantee                      │
│                                                             │
│  Big Omega (Ω) →  Lower bound   →  "At least this fast"    │
│                    Best case guarantee                       │
│                                                             │
│  Big Theta (Θ) →  Tight bound   →  "Exactly this fast"     │
│                    Both upper and lower                      │
│                                                             │
└─────────────────────────────────────────────────────────────┘

 f(n) = O(g(n))  means f grows NO FASTER than g
 f(n) = Ω(g(n))  means f grows NO SLOWER than g
 f(n) = Θ(g(n))  means f grows AT THE SAME RATE as g
```

### Formal Definition of Big O (Intuitive)

```
 f(n) = O(g(n))  if there exist constants c > 0 and n₀ ≥ 0
                  such that f(n) ≤ c·g(n) for all n ≥ n₀

 Example: Show 3n + 5 = O(n)
 
   We need: 3n + 5 ≤ c·n for some c, for large n
   
   Choose c = 4:  3n + 5 ≤ 4n  →  5 ≤ n  →  true for n ≥ 5
   
   So with c = 4, n₀ = 5:  3n + 5 ≤ 4n for all n ≥ 5  ✓
   
   Therefore 3n + 5 = O(n)  ✓
```

---

## 2.6 Comparing Algorithms

### Framework for Comparison

```
When comparing algorithms, consider:

┌──────────────────────────────────┐
│  1. TIME COMPLEXITY              │  ← Most important
│     (How many operations?)       │
├──────────────────────────────────┤
│  2. SPACE COMPLEXITY             │  ← Second most important
│     (How much memory?)           │
├──────────────────────────────────┤
│  3. IMPLEMENTATION COMPLEXITY    │
│     (How hard to code/maintain?) │
├──────────────────────────────────┤
│  4. STABILITY (for sorting)      │
│     (Does it preserve order?)    │
├──────────────────────────────────┤
│  5. ADAPTIVITY                   │
│     (Faster on partially sorted?)│
└──────────────────────────────────┘
```

### Example: Comparing Search Algorithms

```
┌──────────────────┬──────────────┬──────────────┬───────────────┐
│    Algorithm     │  Best Case   │ Average Case │  Worst Case   │
├──────────────────┼──────────────┼──────────────┼───────────────┤
│ Linear Search    │    O(1)      │    O(n)      │    O(n)       │
│ Binary Search    │    O(1)      │  O(log n)    │  O(log n)     │
│ Hash Table       │    O(1)      │    O(1)      │    O(n)       │
├──────────────────┼──────────────┼──────────────┼───────────────┤
│ Requirement      │    None      │  Sorted data │  Hash function│
│ Space            │    O(1)      │    O(1)      │    O(n)       │
└──────────────────┴──────────────┴──────────────┴───────────────┘
```

### Example: Comparing Sorting Algorithms

```
┌─────────────────┬──────────┬──────────┬──────────┬────────┬────────┐
│   Algorithm     │  Best    │ Average  │  Worst   │ Space  │ Stable │
├─────────────────┼──────────┼──────────┼──────────┼────────┼────────┤
│ Bubble Sort     │  O(n)    │  O(n²)   │  O(n²)   │ O(1)   │  Yes   │
│ Selection Sort  │  O(n²)   │  O(n²)   │  O(n²)   │ O(1)   │  No    │
│ Insertion Sort  │  O(n)    │  O(n²)   │  O(n²)   │ O(1)   │  Yes   │
│ Merge Sort      │O(n log n)│O(n log n)│O(n log n)│ O(n)   │  Yes   │
│ Quick Sort      │O(n log n)│O(n log n)│  O(n²)   │O(log n)│  No    │
│ Heap Sort       │O(n log n)│O(n log n)│O(n log n)│ O(1)   │  No    │
└─────────────────┴──────────┴──────────┴──────────┴────────┴────────┘
```

### Practical Decision Making

```
Scenario 1: Small input (n < 50)
  → Use simple algorithms (Insertion Sort)
  → Constants matter more than growth rates
  → Easier to implement and debug

Scenario 2: Large input (n > 10,000)
  → Growth rate dominates
  → Use efficient algorithms (Merge Sort, Quick Sort)
  → O(n²) becomes unacceptable

Scenario 3: Memory-constrained
  → Prefer in-place algorithms (O(1) extra space)
  → Use Quick Sort over Merge Sort
  → Avoid creating copies of data

Scenario 4: Nearly sorted data
  → Use adaptive algorithms (Insertion Sort is O(n) for sorted data)
  → Avoid algorithms that don't benefit (Selection Sort is always O(n²))
```

### Running Time Comparison Table

```
 Operations performed in 1 second (assuming 10⁸ operations/sec):

 ┌──────────┬──────────┬──────────┬──────────┬──────────┬──────────┐
 │   n      │  O(n)    │O(n log n)│  O(n²)   │  O(n³)   │  O(2ⁿ)   │
 ├──────────┼──────────┼──────────┼──────────┼──────────┼──────────┤
 │   10     │ instant  │ instant  │ instant  │ instant  │ instant  │
 │   100    │ instant  │ instant  │ instant  │  0.001s  │  10²² yr │
 │   1,000  │ instant  │ instant  │  0.01s   │  10s     │    ∞     │
 │  10,000  │ instant  │  0.001s  │  1s      │  2.7 hr  │    ∞     │
 │ 100,000  │  0.001s  │  0.002s  │  100s    │ 115 days │    ∞     │
 │ 1,000,000│  0.01s   │  0.02s   │  2.7 hr  │  31 yr   │    ∞     │
 └──────────┴──────────┴──────────┴──────────┴──────────┴──────────┘
```

---

## Summary Table

| Concept | Key Points |
|---------|-----------|
| A Priori Analysis | Theoretical analysis independent of hardware; count operations |
| Time Complexity | Number of operations as function of input size n |
| Space Complexity | Total memory needed = Input space + Auxiliary space |
| Best Case | Minimum operations (most favorable input) |
| Average Case | Expected operations over all possible inputs |
| Worst Case | Maximum operations (least favorable input); **most commonly used** |
| Asymptotic Analysis | Study of growth rate as n → ∞; ignore constants and lower-order terms |
| Basic Operations | Assignments, comparisons, arithmetic — each counts as O(1) |
| Growth Rate | n → n² → n³ → 2ⁿ; higher growth = slower algorithm for large n |

---

## Quick Revision Questions

**Q1.** Why do we prefer theoretical (a priori) analysis over empirical (a posteriori) analysis?

<details>
<summary>Answer</summary>
Theoretical analysis is hardware-independent, language-independent, and based on input size n. Empirical analysis varies with hardware, language, and specific inputs, making it unreliable for objective comparison.
</details>

**Q2.** An algorithm has T(n) = 7n³ + 2n² + 5n + 100. What is its time complexity in Big O notation?

<details>
<summary>Answer</summary>
O(n³). We keep only the highest-order term and drop constants. As n → ∞, 7n³ dominates all other terms.
</details>

**Q3.** For Linear Search, what input causes the best case and what causes the worst case?

<details>
<summary>Answer</summary>
Best case: Target is the first element → O(1). Worst case: Target is the last element or not present → O(n), requiring n comparisons.
</details>

**Q4.** What is the difference between time complexity and space complexity? Can an algorithm be fast but use a lot of memory?

<details>
<summary>Answer</summary>
Time complexity measures operations; space complexity measures memory. Yes, an algorithm can be fast but memory-heavy — for example, using a hash table for O(1) lookups requires O(n) extra space to store all elements. This is a space-time trade-off.
</details>

**Q5.** If Algorithm A is O(n) and Algorithm B is O(n²), is A ALWAYS faster than B?

<details>
<summary>Answer</summary>
Not always! For small n, B could be faster if its constants are smaller (e.g., T_A(n) = 1000n vs T_B(n) = n²; for n < 1000, B is faster). However, for sufficiently large n, A will always be faster. Asymptotic analysis describes behavior for LARGE n.
</details>

**Q6.** Why is worst-case analysis more commonly used than average-case or best-case?

<details>
<summary>Answer</summary>
Worst-case gives a guaranteed upper bound — the algorithm will NEVER take longer than this. Best case is too optimistic. Average case requires knowledge of input distribution, which is often unknown. Worst case provides a safe performance guarantee.
</details>

---

[← Previous: Introduction to DSA](../01-Introduction-to-DSA/01-introduction-to-dsa.md) | [Back to README](../README.md) | [Next: Big O Notation →](../03-Big-O-Notation/03-big-o-notation.md)
