# Unit 3: Big O Notation

[← Previous: Algorithm Analysis Basics](../02-Algorithm-Analysis-Basics/02-algorithm-analysis-basics.md) | [Back to README](../README.md) | [Next: Big Omega and Big Theta →](../04-Big-Omega-and-Big-Theta/04-big-omega-and-big-theta.md)

---

## Chapter Overview

Big O notation is the **most important tool** in algorithm analysis. It provides a standardized way to describe the **upper bound** of an algorithm's growth rate. Mastering Big O is essential — you will use it every single day in DSA.

---

## 3.1 What is Big O?

### Intuitive Definition

Big O describes the **worst-case growth rate** of an algorithm. It answers:

> "As the input size grows, how does the number of operations grow **at most**?"

### Formal Definition

```
f(n) = O(g(n))  if and only if there exist positive constants c and n₀
                 such that:

         f(n) ≤ c · g(n)    for all n ≥ n₀


 In plain English:
 "f(n) grows no faster than g(n), up to a constant factor,
  for sufficiently large n."
```

### Visual Understanding

```
 Operations
    ▲
    │           c·g(n) ╱
    │               ╱
    │            ╱
    │         ╱  ← f(n) is ALWAYS below c·g(n) after n₀
    │       ╱ ·
    │     ╱·    · f(n)
    │   ╱·   ·
    │  ╱ · ·
    │╱··
    │·    n₀
    └──────┼──────────────────────► n

 After n₀, f(n) never exceeds c·g(n).
 Therefore f(n) = O(g(n))
```

### Example Proof

**Prove:** $3n + 5 = O(n)$

```
We need to find c > 0 and n₀ ≥ 0 such that:
    3n + 5 ≤ c·n    for all n ≥ n₀

Step 1: Choose c = 4

    3n + 5 ≤ 4n
    5 ≤ n
    n ≥ 5

Step 2: So with c = 4 and n₀ = 5:

    For n=5:  3(5)+5 = 20 ≤ 4(5) = 20  ✓
    For n=6:  3(6)+5 = 23 ≤ 4(6) = 24  ✓
    For n=10: 3(10)+5 = 35 ≤ 4(10) = 40 ✓
    ... always holds for n ≥ 5  ✓

Therefore 3n + 5 = O(n)  ✓
```

---

## 3.2 Common Complexities

### The Complexity Hierarchy

```
 Speed: FAST ──────────────────────────────────────► SLOW

  O(1)  <  O(log n)  <  O(n)  <  O(n log n)  <  O(n²)  <  O(2ⁿ)  <  O(n!)

  ─────    ─────────    ─────    ───────────    ──────    ──────    ──────
  Constant Logarithmic  Linear  Linearithmic  Quadratic  Expo.   Factorial
```

### 1. O(1) — Constant Time

**The operation count does NOT depend on input size.**

```
ALGORITHM GetElement(A, index)
    RETURN A[index]

ALGORITHM Swap(a, b)
    temp ← a
    a ← b
    b ← temp

ALGORITHM IsEven(n)
    RETURN n % 2 == 0
```

```
 Operations
    ▲
  5 │ ────────────────────────────── (always the same)
    │
    └──────────────────────────────► n
    
 Examples:
 • Array access by index
 • Push/Pop on stack
 • Insert/Remove at head of linked list
 • Hash table lookup (average case)
```

### 2. O(log n) — Logarithmic Time

**The problem size is halved (or reduced by a fraction) at each step.**

```
ALGORITHM BinarySearch(A, n, target)
    low ← 0
    high ← n - 1
    WHILE low ≤ high DO
        mid ← (low + high) / 2
        IF A[mid] == target THEN
            RETURN mid
        ELSE IF A[mid] < target THEN
            low ← mid + 1        ← eliminate left half
        ELSE
            high ← mid - 1       ← eliminate right half
    RETURN -1

Step-by-step trace:
A = [2, 5, 8, 12, 16, 23, 38, 56, 72, 91], target = 23

Step 1: low=0, high=9, mid=4 → A[4]=16 < 23 → low=5
        [2, 5, 8, 12, 16, | 23, 38, 56, 72, 91]
                             ▲ search here

Step 2: low=5, high=9, mid=7 → A[7]=56 > 23 → high=6
        [23, 38, 56, | 72, 91]  → [23, 38]
         ▲ search here

Step 3: low=5, high=6, mid=5 → A[5]=23 == 23 → FOUND at index 5!

Only 3 steps for 10 elements (vs 6 for linear search)!
```

```
  n elements       Steps needed
  ──────────       ────────────
       1                0
       2                1
       4                2
       8                3
      16                4
      32                5
    1024               10
 1048576               20
 
 Pattern: Steps = log₂(n)
```

```
 Why log n? Because we HALVE the search space each step:
 
 n → n/2 → n/4 → n/8 → ... → 1
 
 How many times can we divide n by 2 until we reach 1?
 
 n / 2^k = 1  →  n = 2^k  →  k = log₂(n)
```

### 3. O(n) — Linear Time

**Visit every element exactly once (or a constant number of times).**

```
ALGORITHM FindMax(A, n)
    max ← A[0]
    FOR i ← 1 TO n-1
        IF A[i] > max
            max ← A[i]
    RETURN max

Every element checked once → n operations → O(n)
```

```
 Operations
    ▲
    │                              ╱
    │                           ╱
    │                        ╱
    │                     ╱
    │                  ╱
    │               ╱
    │            ╱
    │         ╱
    │      ╱
    │   ╱
    │╱
    └──────────────────────────────► n

 Examples:
 • Linear search
 • Finding min/max
 • Counting elements
 • Traversing a linked list
 • Single pass through array
```

### 4. O(n log n) — Linearithmic Time

**Typically: divide the problem in half, do linear work at each level.**

```
ALGORITHM MergeSort(A, left, right)
    IF left < right THEN
        mid ← (left + right) / 2
        MergeSort(A, left, mid)          ← sort left half
        MergeSort(A, mid+1, right)       ← sort right half
        Merge(A, left, mid, right)       ← merge two halves: O(n)

 Visualization for A = [38, 27, 43, 3, 9, 82, 10]:
 
 Level 0:  [38, 27, 43, 3, 9, 82, 10]           Split → O(1)
                    /              \
 Level 1:  [38, 27, 43, 3]    [9, 82, 10]        Split → O(1)
              /       \          /      \
 Level 2:  [38,27]  [43,3]   [9,82]   [10]       Split → O(1)
            / \      / \      / \       |
 Level 3: [38][27] [43][3]  [9][82]   [10]       Base case
            \ /      \ /      \ /       |
 Level 2:  [27,38]  [3,43]   [9,82]   [10]       Merge → O(n)
              \       /          \      /
 Level 1:  [3, 27, 38, 43]    [9, 10, 82]        Merge → O(n)
                    \              /
 Level 0:  [3, 9, 10, 27, 38, 43, 82]            Merge → O(n)

 log n levels × O(n) work per level = O(n log n)
```

### 5. O(n²) — Quadratic Time

**Nested iteration over the data — every pair of elements is considered.**

```
ALGORITHM BubbleSort(A, n)
    FOR i ← 0 TO n-2
        FOR j ← 0 TO n-i-2
            IF A[j] > A[j+1]
                SWAP(A[j], A[j+1])

Trace for A = [5, 3, 8, 1]:

Pass i=0: [5,3,8,1] → [3,5,8,1] → [3,5,8,1] → [3,5,1,8]  (3 comparisons)
Pass i=1: [3,5,1,8] → [3,5,1,8] → [3,1,5,8]               (2 comparisons)
Pass i=2: [3,1,5,8] → [1,3,5,8]                             (1 comparison)
                                                    Total: 6 = n(n-1)/2

For n=4:  4×3/2 = 6 comparisons
For n=10: 10×9/2 = 45 comparisons
For n=100: 100×99/2 = 4950 comparisons
→ Quadratic growth!
```

```
 Operations
    ▲
    │                                    *
    │                                 *
    │                              *
    │                          *
    │                      *
    │                  *
    │             *
    │         *
    │     *
    │  *
    │*
    └──────────────────────────────────► n

 Examples:
 • Bubble Sort, Selection Sort, Insertion Sort
 • Checking all pairs: two nested loops
 • Simple matrix operations
```

### 6. O(2ⁿ) — Exponential Time

**The number of operations DOUBLES with each additional input element.**

```
ALGORITHM Fibonacci(n)
    IF n ≤ 1
        RETURN n
    RETURN Fibonacci(n-1) + Fibonacci(n-2)

Call tree for Fibonacci(5):

                        fib(5)
                       /      \
                   fib(4)      fib(3)
                  /    \       /    \
              fib(3)  fib(2) fib(2) fib(1)
              /  \    /  \    /  \
          fib(2) fib(1) fib(1) fib(0) fib(1) fib(0)
          /  \
      fib(1) fib(0)

Total calls: 15 ≈ 2⁵ = 32 (roughly doubles each level)

 n=10:  ~1,024 calls
 n=20:  ~1,048,576 calls
 n=30:  ~1,073,741,824 calls  (over 1 BILLION!)
 n=50:  ~10¹⁵ calls  (would take YEARS)
```

### 7. O(n!) — Factorial Time

```
ALGORITHM GeneratePermutations(A, n)
    // Generates all possible arrangements of n elements
    // n=3: 6 permutations
    // n=5: 120 permutations
    // n=10: 3,628,800 permutations
    // n=20: 2,432,902,008,176,640,000 permutations!

Example: Permutations of [1, 2, 3]:
  [1,2,3] [1,3,2] [2,1,3] [2,3,1] [3,1,2] [3,2,1]
  → 3! = 6 permutations
```

### Growth Comparison Table

```
┌──────┬────────┬─────────┬────────┬───────────┬──────────┬──────────┬─────────┐
│  n   │  O(1)  │O(log n) │  O(n)  │O(n log n) │  O(n²)   │  O(2ⁿ)   │  O(n!)  │
├──────┼────────┼─────────┼────────┼───────────┼──────────┼──────────┼─────────┤
│   1  │    1   │    0    │    1   │     0     │      1   │      2   │      1  │
│   5  │    1   │    2    │    5   │    12     │     25   │     32   │    120  │
│  10  │    1   │    3    │   10   │    33     │    100   │   1024   │ 3.6×10⁶ │
│  20  │    1   │    4    │   20   │    86     │    400   │  1.0×10⁶ │ 2.4×10¹⁸│
│  50  │    1   │    6    │   50   │   282     │  2,500   │  1.1×10¹⁵│   ∞     │
│ 100  │    1   │    7    │  100   │   664     │ 10,000   │  1.3×10³⁰│   ∞     │
│1000  │    1   │   10    │ 1000   │  9,966    │1,000,000 │    ∞     │   ∞     │
└──────┴────────┴─────────┴────────┴───────────┴──────────┴──────────┴─────────┘
```

---

## 3.3 Rules for Calculating Big O

### Rule 1: Drop Constants

```
T(n) = 5n → O(n)         (drop the 5)
T(n) = 100n² → O(n²)     (drop the 100)
T(n) = n/2 → O(n)        (1/2 is a constant)

WHY? Constants don't affect the growth RATE.
      5n and n both grow linearly.
```

### Rule 2: Drop Non-Dominant Terms

```
T(n) = n² + n → O(n²)           (n is dominated by n²)
T(n) = n³ + n² + n → O(n³)      (n³ dominates everything)
T(n) = n + log n → O(n)         (n dominates log n)
T(n) = 2ⁿ + n³ → O(2ⁿ)         (2ⁿ dominates n³)

WHY? As n → ∞, only the largest term matters.

 n=1000:
   n² = 1,000,000
   n  = 1,000        ← only 0.1% of n²
   
   n² + n ≈ n²  for large n
```

### Rule 3: Different Inputs → Different Variables

```
ALGORITHM PrintPairs(A, B)
    FOR i ← 0 TO len(A)-1          ← a iterations
        FOR j ← 0 TO len(B)-1      ← b iterations
            PRINT A[i], B[j]

 This is O(a × b), NOT O(n²)!
 
 If A and B are different inputs with different sizes,
 use different variables.

 O(n²) would only be correct if a == b == n.
```

### Rule 4: Sequential Steps ADD

```
ALGORITHM DoTwoThings(A, n)
    // Step 1: O(n)
    FOR i ← 0 TO n-1
        PRINT A[i]
    
    // Step 2: O(n²)
    FOR i ← 0 TO n-1
        FOR j ← 0 TO n-1
            PRINT A[i] + A[j]

 Total = O(n) + O(n²) = O(n²)    (drop the non-dominant O(n))
```

### Rule 5: Nested Steps MULTIPLY

```
ALGORITHM NestedLoops(A, n)
    FOR i ← 0 TO n-1              ← outer: n times
        FOR j ← 0 TO n-1          ← inner: n times (for each outer)
            PRINT A[i] * A[j]

 Total = n × n = O(n²)
```

### Summary of Rules

```
┌─────────────────────────────────────────────────┐
│           Big O Calculation Rules               │
├─────────────────────────────────────────────────┤
│ 1. Drop constants:  5n → O(n)                   │
│ 2. Drop lower terms: n² + n → O(n²)            │
│ 3. Different inputs = different variables        │
│ 4. Sequential (one after another) → ADD          │
│ 5. Nested (one inside another) → MULTIPLY        │
│ 6. Log base doesn't matter: log₂n = O(log n)   │
│ 7. Conditional → take the WORST branch           │
└─────────────────────────────────────────────────┘
```

---

## 3.4 Drop Constants and Non-Dominant Terms — Deep Dive

### Why Constants Don't Matter Asymptotically

```
 Compare: f(n) = 100n  vs  g(n) = n²

  n=1:     100  vs  1        → f is 100x slower
  n=10:    1000 vs  100      → f is 10x slower
  n=100:   10000 vs 10000    → EQUAL
  n=1000:  100000 vs 1000000 → g is 10x slower
  n=10000: 1000000 vs 10⁸    → g is 100x slower!

 The constant (100) only matters for small n.
 For large n, the GROWTH RATE (linear vs quadratic) always wins.
```

### Common Simplifications

```
Expression                  Simplified Big O
────────────────────────── ──────────────────
3n + 7                      O(n)
n² + 1000n + 5000          O(n²)
n/2                         O(n)
5 × 2ⁿ + n³                O(2ⁿ)
log(n²)                     O(log n)  [since log(n²) = 2·log(n)]
√n                          O(√n)  or O(n^0.5)
n + n/2 + n/4 + ...         O(n)   [geometric series converges to 2n]
```

### Practice: Simplify These

```
1. T(n) = 3n² + 7n + 42
   Answer: O(n²)

2. T(n) = n × log(n) + n
   Answer: O(n log n)  [n log n dominates n]

3. T(n) = 2ⁿ + n⁵ + 3n
   Answer: O(2ⁿ)  [exponential dominates polynomial]

4. T(n) = log(n³)
   Answer: O(log n)  [log(n³) = 3·log(n)]

5. T(n) = n! + 2ⁿ
   Answer: O(n!)  [factorial dominates exponential]
```

---

## 3.5 Big O of Nested Loops

### Basic Nested Loops

```
// Pattern 1: Both loops go to n → O(n²)
FOR i ← 0 TO n-1
    FOR j ← 0 TO n-1
        // O(1) work

 Total: n × n = n²

 i=0: j runs n times  → n operations
 i=1: j runs n times  → n operations
 ...
 i=n-1: j runs n times → n operations
 Total: n × n = n²
```

### Dependent Inner Loop

```
// Pattern 2: j depends on i → Still O(n²) but ~ n²/2
FOR i ← 0 TO n-1
    FOR j ← 0 TO i
        // O(1) work

 i=0: j runs 1 time     → 1
 i=1: j runs 2 times    → 2
 i=2: j runs 3 times    → 3
 ...
 i=n-1: j runs n times  → n

 Total = 1 + 2 + 3 + ... + n = n(n+1)/2 = n²/2 + n/2 → O(n²)
```

### Triple Nested Loop

```
// Pattern 3: Three nested loops → O(n³)
FOR i ← 0 TO n-1
    FOR j ← 0 TO n-1
        FOR k ← 0 TO n-1
            // O(1) work

 Total: n × n × n = n³
```

### Loop with Multiplication (Logarithmic)

```
// Pattern 4: i doubles each iteration → O(log n)
i ← 1
WHILE i < n
    // O(1) work
    i ← i * 2

 Trace for n=16:
 i = 1, 2, 4, 8, 16 → STOP
 Iterations: 4 = log₂(16)

 i = 1 → 2 → 4 → 8 → 16 → ... → n
 Iterations = log₂(n)
```

```
// Pattern 5: i halves each iteration → O(log n)
i ← n
WHILE i > 1
    // O(1) work
    i ← i / 2

 Same idea: n → n/2 → n/4 → ... → 1
 Iterations = log₂(n)
```

### Nested Loop with Logarithmic Inner

```
// Pattern 6: O(n) outer × O(log n) inner = O(n log n)
FOR i ← 0 TO n-1
    j ← 1
    WHILE j < n
        // O(1) work
        j ← j * 2

 Outer: n iterations
 Inner: log(n) iterations
 Total: n × log(n) = O(n log n)
```

### Tricky Patterns

```
// Pattern 7: Two sequential loops → O(n) + O(n) = O(n)
FOR i ← 0 TO n-1
    // O(1) work
FOR j ← 0 TO n-1
    // O(1) work
    
 Total: n + n = 2n → O(n)  (ADD, don't multiply!)


// Pattern 8: Loop skips elements → Still O(n)
FOR i ← 0 TO n-1 STEP 3     ← i increments by 3
    // O(1) work

 Iterations: n/3 → O(n)  (constants don't matter)


// Pattern 9: Two independent inputs
FOR i ← 0 TO a-1
    FOR j ← 0 TO b-1
        // O(1) work

 Total: a × b = O(a·b)  (NOT O(n²) unless a=b=n)
```

### Big O Nested Loops — Decision Flowchart

```
┌───────────────────────────────────┐
│ Is the loop sequential or nested? │
└─────────┬─────────────────┬───────┘
          │                 │
     Sequential          Nested
          │                 │
          ▼                 ▼
      ADD them         MULTIPLY them
    O(a) + O(b)        O(a) × O(b)
    = O(max(a,b))      = O(a·b)

┌───────────────────────────────────┐
│  How does the loop variable       │
│  change each iteration?           │
└──┬────────────┬───────────┬───────┘
   │            │           │
  i++         i*=2        i*=k
 (add 1)     (double)    (multiply by k)
   │            │           │
   ▼            ▼           ▼
  O(n)       O(log n)    O(log n)
```

---

## 3.6 Big O of Recursive Functions

### Basic Recursion — O(n)

```
ALGORITHM Factorial(n)
    IF n ≤ 1 THEN RETURN 1
    RETURN n × Factorial(n-1)

Call trace for n=5:
  Factorial(5) → 5 × Factorial(4)
                      → 4 × Factorial(3)
                            → 3 × Factorial(2)
                                  → 2 × Factorial(1)
                                        → RETURN 1
                                  → RETURN 2
                            → RETURN 6
                      → RETURN 24
                → RETURN 120

 Depth of recursion: n
 Work at each level: O(1)
 Total: n × O(1) = O(n)

 Recurrence: T(n) = T(n-1) + O(1)
 Solution:   T(n) = O(n)
```

### Binary Recursion — O(2ⁿ)

```
ALGORITHM Fibonacci(n)
    IF n ≤ 1 THEN RETURN n
    RETURN Fibonacci(n-1) + Fibonacci(n-2)

Call tree for n=4:
                    fib(4)
                   /      \
               fib(3)      fib(2)
              /    \       /    \
          fib(2)  fib(1) fib(1) fib(0)
          /  \
      fib(1) fib(0)

 Each call spawns 2 more calls (roughly)
 Level 0: 1 call
 Level 1: 2 calls
 Level 2: 4 calls
 Level k: 2^k calls

 Total ≈ 2⁰ + 2¹ + 2² + ... + 2ⁿ = 2^(n+1) - 1 → O(2ⁿ)

 Recurrence: T(n) = T(n-1) + T(n-2) + O(1)
 Solution:   T(n) = O(2ⁿ)  (actually closer to O(1.618ⁿ))
```

### Divide and Conquer — O(n log n)

```
ALGORITHM MergeSort(A, lo, hi)
    IF lo < hi THEN
        mid ← (lo + hi) / 2
        MergeSort(A, lo, mid)         ← T(n/2)
        MergeSort(A, mid+1, hi)       ← T(n/2)
        Merge(A, lo, mid, hi)         ← O(n)

 Recurrence: T(n) = 2T(n/2) + O(n)

 Recursion tree:
 
 Level 0:       [n]                    → n work
                /   \
 Level 1:    [n/2]  [n/2]             → n work (n/2 + n/2)
             / \     / \
 Level 2: [n/4][n/4][n/4][n/4]        → n work (4 × n/4)
           ...
 Level k: [1][1]...[1][1] (n leaves)  → n work

 Levels: log₂(n)
 Work per level: O(n)
 Total: O(n) × log(n) = O(n log n)
```

### Binary Search Recursion — O(log n)

```
ALGORITHM BinarySearch(A, lo, hi, target)
    IF lo > hi THEN RETURN -1
    mid ← (lo + hi) / 2
    IF A[mid] = target THEN RETURN mid
    IF A[mid] < target THEN
        RETURN BinarySearch(A, mid+1, hi, target)    ← T(n/2)
    ELSE
        RETURN BinarySearch(A, lo, mid-1, target)    ← T(n/2)

 Recurrence: T(n) = T(n/2) + O(1)

 Only ONE recursive call (not two!) → halve the problem each time
 
 Level 0: problem size n     → O(1) work
 Level 1: problem size n/2   → O(1) work
 Level 2: problem size n/4   → O(1) work
 ...
 Level k: problem size 1     → O(1) work
 
 Levels: log₂(n)
 Work per level: O(1)
 Total: O(1) × log(n) = O(log n)
```

### Common Recursion Patterns Summary

```
┌──────────────────────────────────────┬────────────────────┬──────────┐
│           Pattern                    │    Recurrence      │  Big O   │
├──────────────────────────────────────┼────────────────────┼──────────┤
│ Process & recurse on n-1            │ T(n)=T(n-1)+O(1)  │  O(n)    │
│ (factorial, list traversal)          │                    │          │
├──────────────────────────────────────┼────────────────────┼──────────┤
│ Process & recurse on n-1 with O(n)  │ T(n)=T(n-1)+O(n)  │  O(n²)   │
│ work (selection sort recursive)      │                    │          │
├──────────────────────────────────────┼────────────────────┼──────────┤
│ Split in half, one side only        │ T(n)=T(n/2)+O(1)  │ O(log n) │
│ (binary search)                      │                    │          │
├──────────────────────────────────────┼────────────────────┼──────────┤
│ Split in half, both sides + O(n)    │ T(n)=2T(n/2)+O(n) │O(n log n)│
│ (merge sort)                         │                    │          │
├──────────────────────────────────────┼────────────────────┼──────────┤
│ Two recursive calls on n-1          │ T(n)=2T(n-1)+O(1) │  O(2ⁿ)   │
│ (naive fibonacci, power set)         │                    │          │
├──────────────────────────────────────┼────────────────────┼──────────┤
│ Split in half, both sides + O(1)    │ T(n)=2T(n/2)+O(1) │  O(n)    │
│ (tree traversal)                     │                    │          │
└──────────────────────────────────────┴────────────────────┴──────────┘
```

---

## Common Patterns & Problem-Solving Techniques

### Identifying Big O at a Glance

```
Clue                              Likely Big O
────────────────────────          ──────────
Single loop over n elements       O(n)
Two nested loops over n           O(n²)
Halving at each step              O(log n)
Loop over n × halving inside      O(n log n)
Two choices at each step, depth n O(2ⁿ)
Generating all permutations       O(n!)
Constant work, no loops           O(1)
```

### Amortized Analysis (Brief Intro)

```
Some operations are USUALLY cheap but OCCASIONALLY expensive.

Example: Dynamic Array (ArrayList)
  - Most insertions: O(1)
  - When full, resize (copy all elements): O(n)
  - But resizing happens rarely!

  Insert 1:  [1, _, _, _]     cost: 1
  Insert 2:  [1, 2, _, _]     cost: 1
  Insert 3:  [1, 2, 3, _]     cost: 1
  Insert 4:  [1, 2, 3, 4]     cost: 1
  Insert 5:  [1,2,3,4,5,_,_,_] cost: 5 (copy 4 old + insert 1)
  
  Total for 5 inserts: 9
  Amortized per insert: 9/5 ≈ 2 → O(1) amortized!
```

---

## Summary Table

| Concept | Key Points |
|---------|-----------|
| Big O | Upper bound on growth rate; worst-case guarantee |
| O(1) | Constant — operations don't depend on n |
| O(log n) | Logarithmic — problem halved each step (binary search) |
| O(n) | Linear — process each element once |
| O(n log n) | Linearithmic — divide-and-conquer with linear merge |
| O(n²) | Quadratic — nested loops, all pairs |
| O(2ⁿ) | Exponential — two choices per element |
| O(n!) | Factorial — all permutations |
| Drop constants | 5n → O(n); constants don't affect growth rate |
| Drop lower terms | n² + n → O(n²); dominant term only |
| Sequential → ADD | O(n) + O(n²) = O(n²) |
| Nested → MULTIPLY | n outer × n inner = O(n²) |
| Recursive | Count depth × work per level, or solve recurrence |

---

## Quick Revision Questions

**Q1.** What is the Big O of the following code?
```
for i = 0 to n:
    for j = 0 to n:
        for k = 0 to n:
            print(i, j, k)
```

<details>
<summary>Answer</summary>
O(n³). Three nested loops, each running n times: n × n × n = n³.
</details>

**Q2.** What is the Big O of binary search and WHY?

<details>
<summary>Answer</summary>
O(log n). At each step, the search space is halved. Starting with n elements: n → n/2 → n/4 → ... → 1. The number of halvings is log₂(n).
</details>

**Q3.** Simplify: T(n) = 3n³ + 2n² + 100n + 500. What is the Big O?

<details>
<summary>Answer</summary>
O(n³). Drop constants (3) and non-dominant terms (2n², 100n, 500). Only the highest-degree term matters.
</details>

**Q4.** Two functions process two different arrays A (size a) and B (size b) in nested loops. Is the complexity O(n²)?

<details>
<summary>Answer</summary>
No! It's O(a × b). Since A and B are different inputs with potentially different sizes, we must use different variables. It's only O(n²) if a = b = n.
</details>

**Q5.** Why is the naive recursive Fibonacci O(2ⁿ) but iterative Fibonacci O(n)?

<details>
<summary>Answer</summary>
Recursive Fibonacci makes two calls for each n (branching tree), creating ~2ⁿ total calls with massive redundant computation. Iterative Fibonacci uses a single loop from 1 to n, computing each value exactly once, hence O(n).
</details>

**Q6.** What is the Big O of this code?
```
i = n
while i > 0:
    print(i)
    i = i // 2
```

<details>
<summary>Answer</summary>
O(log n). The variable i is halved each iteration: n → n/2 → n/4 → ... → 1. Halving n until reaching 1 takes log₂(n) steps.
</details>

---

[← Previous: Algorithm Analysis Basics](../02-Algorithm-Analysis-Basics/02-algorithm-analysis-basics.md) | [Back to README](../README.md) | [Next: Big Omega and Big Theta →](../04-Big-Omega-and-Big-Theta/04-big-omega-and-big-theta.md)
