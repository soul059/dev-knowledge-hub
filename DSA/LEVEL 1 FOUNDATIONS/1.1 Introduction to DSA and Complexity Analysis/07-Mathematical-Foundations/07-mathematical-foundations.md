# Unit 7: Mathematical Foundations

[← Previous: Recurrence Relations](../06-Recurrence-Relations/06-recurrence-relations.md) | [Back to README](../README.md) | [Next: Problem-Solving Approach →](../08-Problem-Solving-Approach/08-problem-solving-approach.md)

---

## Chapter Overview

Mathematics is the **language of algorithm analysis**. This unit covers the essential mathematical tools you'll need throughout your DSA journey — logarithms, series, combinatorics, probability, proof techniques, and modular arithmetic.

---

## 7.1 Logarithms Review

### Definition

The logarithm answers: **"To what power must we raise the base to get this number?"**

$$\log_b(n) = k \iff b^k = n$$

```
 log₂(8) = 3     because  2³ = 8
 log₂(16) = 4    because  2⁴ = 16
 log₂(1024) = 10 because  2¹⁰ = 1024
 log₁₀(1000) = 3 because  10³ = 1000
```

### Why Log Base 2 Dominates DSA

```
 In DSA, log usually means log₂ (binary logarithm)

 WHY? Because we constantly HALVE things:
 
 Binary Search:     n → n/2 → n/4 → ... → 1      (halving)
 Merge Sort:        Split array in half each time
 Binary Tree:       Each level doubles the nodes
 Balanced BST:      Height = log₂(n)
 
 "How many times can you halve n until you reach 1?"
  → That's log₂(n)
  
  n / 2^k = 1  →  k = log₂(n)
```

### Essential Logarithm Properties

```
┌────────────────────────────────────────────────────────────┐
│                 Logarithm Properties                       │
├────────────────────────────────────────────────────────────┤
│                                                            │
│  1. Product Rule:   log(a × b) = log(a) + log(b)         │
│                                                            │
│  2. Quotient Rule:  log(a / b) = log(a) - log(b)         │
│                                                            │
│  3. Power Rule:     log(aⁿ) = n × log(a)                 │
│                                                            │
│  4. Change of Base: log_b(n) = log_c(n) / log_c(b)       │
│                                                            │
│  5. Identity:       log_b(b) = 1                          │
│                                                            │
│  6. Identity:       log_b(1) = 0                          │
│                                                            │
│  7. Inverse:        b^(log_b(n)) = n                      │
│                                                            │
│  8. Base swap:      log_a(b) = 1 / log_b(a)              │
│                                                            │
└────────────────────────────────────────────────────────────┘
```

### Why Log Base Doesn't Matter in Big O

```
log₂(n) = log₁₀(n) / log₁₀(2) = log₁₀(n) × 3.32...

So log₂(n) = constant × log₁₀(n)

Since Big O drops constants:
  O(log₂ n) = O(log₁₀ n) = O(ln n) = O(log n)

We just write O(log n) — base doesn't matter!
```

### Common Log Values to Know

```
┌──────────┬──────────┬────────────────────────────┐
│    n     │ log₂(n)  │          Context           │
├──────────┼──────────┼────────────────────────────┤
│    1     │    0     │ Base case                   │
│    2     │    1     │                             │
│    4     │    2     │                             │
│    8     │    3     │                             │
│   16     │    4     │                             │
│   32     │    5     │                             │
│   64     │    6     │                             │
│  128     │    7     │                             │
│  256     │    8     │ Number of ASCII characters  │
│ 1024     │   10     │ ~1 thousand (1 KB)          │
│ 10⁶      │   ~20   │ 1 million                   │
│ 10⁹      │   ~30   │ 1 billion                   │
│ 10¹²     │   ~40   │ 1 trillion                  │
│ 10¹⁸     │   ~60   │ Long max                    │
└──────────┴──────────┴────────────────────────────┘

Key insight: Even for ENORMOUS inputs, log n is tiny!
```

---

## 7.2 Arithmetic and Geometric Series

### Arithmetic Series

A series where each term increases by a **constant difference** d.

```
a, a+d, a+2d, a+3d, ..., a+(n-1)d

Sum = n × (first + last) / 2
    = n × (2a + (n-1)d) / 2
```

### Most Important Arithmetic Sum

```
1 + 2 + 3 + ... + n = n(n+1)/2

 WHY this matters in DSA:

 Nested loop with dependent inner loop:
 FOR i = 1 TO n
     FOR j = 1 TO i     ← j runs i times
         // work

 Total iterations = 1 + 2 + 3 + ... + n = n(n+1)/2 = O(n²)

 Visual proof:
 
 ┌───┬───┬───┬───┬───┐
 │ * │   │   │   │   │  i=1: 1 step
 ├───┼───┼───┼───┼───┤
 │ * │ * │   │   │   │  i=2: 2 steps
 ├───┼───┼───┼───┼───┤
 │ * │ * │ * │   │   │  i=3: 3 steps
 ├───┼───┼───┼───┼───┤
 │ * │ * │ * │ * │   │  i=4: 4 steps
 ├───┼───┼───┼───┼───┤
 │ * │ * │ * │ * │ * │  i=5: 5 steps
 └───┴───┴───┴───┴───┘
 
 Stars = half the rectangle = n×n/2 = n(n+1)/2
```

### Other Useful Arithmetic Sums

```
Sum of first n odd numbers:
 1 + 3 + 5 + ... + (2n-1) = n²

Sum of first n even numbers:
 2 + 4 + 6 + ... + 2n = n(n+1)

Sum of squares:
 1² + 2² + 3² + ... + n² = n(n+1)(2n+1)/6  ≈  n³/3  →  O(n³)

Sum of cubes:
 1³ + 2³ + 3³ + ... + n³ = [n(n+1)/2]²  →  O(n⁴)
```

### Geometric Series

A series where each term is **multiplied** by a constant ratio r.

```
a, ar, ar², ar³, ..., ar^(n-1)

Sum = a × (rⁿ - 1) / (r - 1)     if r ≠ 1
```

### Geometric Series in DSA

```
1 + 2 + 4 + 8 + ... + 2^k = 2^(k+1) - 1 ≈ 2 × 2^k

 WHY this matters:

 1. Complete binary tree: nodes at each level double
    Level 0:  1 node
    Level 1:  2 nodes
    Level 2:  4 nodes
    Level k:  2^k nodes
    Total:    2^(k+1) - 1 ≈ 2n (if n = 2^k leaves)

 2. Dynamic array resizing:
    Resize costs: 1 + 2 + 4 + 8 + ... + n = 2n - 1 ≈ 2n
    
    Total cost for n insertions ≈ 2n → amortized O(1) per insert!

 3. When r > 1: The LAST TERM dominates
    1 + 2 + 4 + 8 + 16 = 31 ≈ 2 × 16
    → Geometric series with ratio > 1 is dominated by its last term
```

### Decreasing Geometric Series

```
When |r| < 1: The FIRST TERM dominates and series converges

 n + n/2 + n/4 + n/8 + ... = n × (1 + 1/2 + 1/4 + ...) 
                             = n × 2 = 2n → O(n)

 In DSA: If work DECREASES by constant factor at each level,
 total work = O(root's work)

 Example: T(n) = T(n/2) + n
   n + n/2 + n/4 + ... = 2n → O(n)
```

### Series Quick Reference

```
┌────────────────────────────┬────────────────────┬────────────┐
│          Series            │       Sum          │   Big O    │
├────────────────────────────┼────────────────────┼────────────┤
│ 1 + 1 + ... + 1  (n times)│        n           │   O(n)     │
│ 1 + 2 + 3 + ... + n       │    n(n+1)/2        │   O(n²)    │
│ 1² + 2² + ... + n²        │  n(n+1)(2n+1)/6    │   O(n³)    │
│ 1 + 2 + 4 + ... + 2^k     │    2^(k+1) - 1     │   O(2^k)   │
│ n + n/2 + n/4 + ...        │      2n            │   O(n)     │
│ 1 + 1/2 + 1/3 + ... + 1/n │     ≈ ln(n)        │   O(log n) │
│ log1 + log2 + ... + logn   │   log(n!) ≈ nlogn  │  O(n log n)│
└────────────────────────────┴────────────────────┴────────────┘
```

### Harmonic Series

```
H(n) = 1 + 1/2 + 1/3 + 1/4 + ... + 1/n ≈ ln(n) + γ

 Where γ ≈ 0.5772 (Euler-Mascheroni constant)

 H(n) = Θ(log n)

 Where it appears:
 • Expected time for hashing with chaining
 • Coupon collector problem
 • Analysis of randomized algorithms
```

---

## 7.3 Permutations and Combinations

### Permutations — Order Matters

```
P(n, r) = n! / (n-r)!

"How many ways to arrange r items from n items?"

Example: Arrange 3 letters from {A, B, C, D}
 P(4, 3) = 4! / 1! = 24

 All permutations: ABC, ACB, BAC, BCA, CAB, CBA,
                   ABD, ADB, BAD, BDA, DAB, DBA,
                   ACD, ADC, CAD, CDA, DAC, DCA,
                   BCD, BDC, CBD, CDB, DBC, DCB  → 24 ✓
```

### Combinations — Order Doesn't Matter

```
C(n, r) = n! / (r! × (n-r)!)
        also written as: (n choose r) or ⁿCᵣ

"How many ways to choose r items from n items?"

Example: Choose 2 from {A, B, C, D}
 C(4, 2) = 4! / (2! × 2!) = 24 / 4 = 6

 All combinations: {A,B}, {A,C}, {A,D}, {B,C}, {B,D}, {C,D}  → 6 ✓
```

### DSA Applications

```
┌──────────────────────────────┬────────────────────────────────┐
│ Concept                      │ DSA Connection                 │
├──────────────────────────────┼────────────────────────────────┤
│ n! (all permutations)        │ Brute force: try every order   │
│                              │ Time: O(n!)                    │
├──────────────────────────────┼────────────────────────────────┤
│ 2ⁿ (all subsets)             │ Power set: include/exclude     │
│                              │ each element                   │
│                              │ Time: O(2ⁿ)                   │
├──────────────────────────────┼────────────────────────────────┤
│ C(n, r) (choosing r from n)  │ Number of ways to select       │
│                              │ items; combinatorial problems  │
├──────────────────────────────┼────────────────────────────────┤
│ Catalan numbers              │ Number of valid parentheses,   │
│                              │ BST shapes, etc.               │
└──────────────────────────────┴────────────────────────────────┘
```

### Factorials and Growth

```
n!  grows FASTER than 2ⁿ for large n:

 n      n!            2ⁿ
 5      120           32
 10     3,628,800     1,024
 15     1.3 × 10¹²    32,768
 20     2.4 × 10¹⁸    1,048,576

 Stirling's approximation: n! ≈ √(2πn) × (n/e)ⁿ
 log(n!) ≈ n·log(n)  →  useful in analysis
```

### Power Set (All Subsets)

```
Set {A, B, C} has 2³ = 8 subsets:

 Subset    Binary  Include?
 {}        000     none
 {A}       001     A
 {B}       010     B
 {A,B}     011     A,B
 {C}       100     C
 {A,C}     101     A,C
 {B,C}     110     B,C
 {A,B,C}   111     all

 Each element: 2 choices (include or exclude)
 n elements: 2 × 2 × ... × 2 = 2ⁿ subsets

 This is why subset enumeration is O(2ⁿ)
```

---

## 7.4 Probability Basics

### Relevance to DSA

Probability helps analyze **average-case** performance and **randomized algorithms**.

### Key Concepts

```
P(event) = favorable outcomes / total outcomes

Expected Value:
 E[X] = Σ xᵢ × P(xᵢ)

 "The average result over many trials"
```

### Example: Average Case of Linear Search

```
Assume target is equally likely to be at any position (or absent):

 If found: probability of being at index i = 1/n for each i
 
 Expected comparisons (if target exists):
 E = 1×(1/n) + 2×(1/n) + 3×(1/n) + ... + n×(1/n)
   = (1 + 2 + 3 + ... + n) / n
   = n(n+1)/(2n)
   = (n+1)/2
   ≈ n/2
   = O(n)

 So on average, linear search checks about HALF the elements.
```

### Example: Quick Sort — Expected Partitioning

```
Quick Sort picks a random pivot. 

Good pivot: splits array into ≥ 25% and ≤ 75%
Bad pivot:  everything else

 Probability of good pivot = 50% (middle half of values)

 ┌────────────────────────────────────────────┐
 │  Sorted array: [1, 2, 3, 4, 5, 6, 7, 8]  │
 │                                            │
 │  Bad        Good pivots (3-6)     Bad      │
 │  [1,2]     [3, 4, 5, 6]         [7,8]     │
 │  25%           50%               25%       │
 └────────────────────────────────────────────┘

 Even with 50% bad pivots, expected depth = O(log n)
 because on average, we get good splits enough.
 
 Expected time of Quick Sort = O(n log n)
```

### Example: Birthday Paradox (Hashing Collisions)

```
In a group of n people, what's the probability that 2 share a birthday?

 Surprisingly: only 23 people needed for >50% probability!

 This relates to HASH COLLISIONS:
 With m hash buckets and n keys:
 
 Expected collisions start when n ≈ √m
 (Birthday paradox: n ≈ √365 ≈ 19)

 This is why hash tables are sized to be
 significantly larger than the expected number of entries.
```

---

## 7.5 Proof Techniques

### Why Proofs in DSA?

```
We use proofs to:
 1. Prove correctness of algorithms (loop invariants)
 2. Prove time/space complexity bounds
 3. Prove lower bounds (impossibility results)
 4. Prove properties of data structures
```

### Technique 1: Mathematical Induction

```
Like climbing a ladder:
 1. BASE: Show it's true for n = 1 (or n = 0)
 2. STEP: Assume true for k, prove for k + 1
 3. CONCLUSION: True for all n ≥ 1

Example: Prove 1 + 2 + 3 + ... + n = n(n+1)/2

 Base (n=1): 1 = 1(2)/2 = 1  ✓

 Inductive step: Assume true for k:
   1 + 2 + ... + k = k(k+1)/2

 Prove for k+1:
   1 + 2 + ... + k + (k+1)
   = k(k+1)/2 + (k+1)             (by assumption)
   = k(k+1)/2 + 2(k+1)/2
   = (k+1)(k+2)/2
   = (k+1)((k+1)+1)/2            ✓

 Therefore true for all n ≥ 1.  □
```

### Technique 2: Proof by Contradiction

```
To prove statement S:
 1. Assume S is FALSE (¬S)
 2. Derive a logical contradiction
 3. Since ¬S leads to contradiction, S must be TRUE

Example: Prove √2 is irrational

 Assume √2 = p/q (rational, in lowest terms, gcd(p,q) = 1)
 Then 2 = p²/q²   →   p² = 2q²
 So p² is even, therefore p is even → p = 2k
 Then (2k)² = 2q²  →  4k² = 2q²  →  q² = 2k²
 So q² is even, therefore q is even
 Both p and q are even → they share factor 2
 But we said gcd(p,q) = 1 → CONTRADICTION!
 Therefore √2 is irrational.  □
```

### Technique 3: Loop Invariant

```
A loop invariant is a property that:
 1. Is TRUE before the loop starts  (INITIALIZATION)
 2. If true before iteration i, remains true after  (MAINTENANCE)
 3. When the loop ends, gives us the desired result  (TERMINATION)

Example: Prove insertion sort correctness

ALGORITHM InsertionSort(A, n)
    FOR i ← 1 TO n-1
        key ← A[i]
        j ← i - 1
        WHILE j ≥ 0 AND A[j] > key
            A[j+1] ← A[j]
            j ← j - 1
        A[j+1] ← key

 Invariant: "Before iteration i, A[0..i-1] is sorted."

 INITIALIZATION: Before i=1, A[0..0] has 1 element → sorted ✓
 
 MAINTENANCE: In iteration i, we insert A[i] into its correct
              position in A[0..i-1]. After insertion, A[0..i] is sorted. ✓
 
 TERMINATION: When i = n, invariant says A[0..n-1] is sorted. ✓
              The entire array is sorted!  □
```

### Technique 4: Pigeonhole Principle

```
"If n+1 pigeons are placed in n holes, at least one hole has 2+ pigeons."

DSA Applications:
 • If a hash table has n slots and n+1 keys → at least one collision
 • In an array of n+1 integers from [1,n] → at least one duplicate
 • In a sequence of n+1 elements from {1,...,n} → cycle exists

Example Problem:
 Array of n+1 integers, each in range [1, n]. Prove a duplicate exists.
 
 n values (1 through n) = n "holes"
 n+1 elements = n+1 "pigeons"
 By pigeonhole principle: at least one value appears twice.  □
```

---

## 7.6 Modular Arithmetic

### Definition

```
a mod m = remainder when a is divided by m

 17 mod 5 = 2    (17 = 3×5 + 2)
 25 mod 7 = 4    (25 = 3×7 + 4)
 12 mod 4 = 0    (12 = 3×4 + 0)
 -3 mod 5 = 2    (in math: -3 = -1×5 + 2)
```

### Properties of Modular Arithmetic

```
┌──────────────────────────────────────────────────────────────┐
│              Modular Arithmetic Properties                   │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│ (a + b) mod m = ((a mod m) + (b mod m)) mod m               │
│                                                              │
│ (a - b) mod m = ((a mod m) - (b mod m) + m) mod m           │
│                                                              │
│ (a × b) mod m = ((a mod m) × (b mod m)) mod m               │
│                                                              │
│ (a^n) mod m   = use fast exponentiation (repeated squaring)  │
│                                                              │
│ Division: (a / b) mod m ≠ ((a mod m) / (b mod m)) mod m     │
│   Instead: multiply by modular inverse of b                  │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

### Why Modular Arithmetic in DSA?

```
1. HASHING:
   Hash function: h(key) = key mod table_size
   Distributes keys across table slots

2. OVERFLOW PREVENTION:
   Computing large results (e.g., Fibonacci(1000))
   Without mod: number has hundreds of digits
   With mod m: result stays within int range

3. COMPETITIVE PROGRAMMING:
   "Give answer modulo 10⁹ + 7"
   → Prevents overflow, allows comparison

4. CRYPTOGRAPHY:
   RSA encryption uses modular exponentiation
   a^e mod n
```

### Fast Exponentiation (Modular)

```
Problem: Compute a^n mod m efficiently

Naive: multiply a, n times → O(n)
Fast: repeated squaring   → O(log n)

ALGORITHM FastPower(a, n, m)
    result ← 1
    a ← a mod m
    WHILE n > 0
        IF n is odd
            result ← (result × a) mod m
        n ← n / 2  (integer division)
        a ← (a × a) mod m
    RETURN result

Trace: 3^13 mod 7

 n=13 (odd):  result = 1×3 = 3     a = 3²=9≡2(mod7)   n=6
 n=6  (even): result = 3            a = 2²=4            n=3
 n=3  (odd):  result = 3×4=12≡5     a = 4²=16≡2         n=1
 n=1  (odd):  result = 5×2=10≡3     a = 2²=4            n=0

 3^13 mod 7 = 3 ✓  (3^13 = 1,594,323; 1,594,323/7 = 227,760 R 3)

Only 4 iterations instead of 13!  (log₂(13) ≈ 4)
```

### GCD and Euclidean Algorithm

```
GCD(a, b) = largest number that divides both a and b

Euclidean Algorithm:
 GCD(a, b) = GCD(b, a mod b),  with GCD(a, 0) = a

Example: GCD(48, 18)
 GCD(48, 18) = GCD(18, 48 mod 18) = GCD(18, 12)
 GCD(18, 12) = GCD(12, 18 mod 12) = GCD(12, 6)
 GCD(12, 6)  = GCD(6, 12 mod 6)   = GCD(6, 0)
 GCD(6, 0)   = 6

 Answer: GCD(48, 18) = 6  ✓

Time Complexity: O(log(min(a, b)))
 — one of the oldest known algorithms (300 BC, Euclid)!
```

---

## Summary Table

| Topic | Key Formula / Concept | DSA Application |
|-------|----------------------|-----------------|
| log₂(n) | "How many times halve n to reach 1" | Binary search, tree height, divide & conquer |
| Arithmetic sum | 1+2+...+n = n(n+1)/2 | Nested loops: O(n²) |
| Geometric sum (r>1) | Last term dominates | Binary tree total nodes, amortized analysis |
| Geometric sum (r<1) | First term dominates | Decreasing work per level = O(root) |
| Harmonic sum | 1+1/2+...+1/n ≈ ln(n) | Average hash chain length |
| n! | n × (n-1) × ... × 1 | All permutations: O(n!) |
| 2ⁿ | All subsets of n elements | Subset enumeration: O(2ⁿ) |
| C(n,r) | n! / (r!(n-r)!) | Choosing r items from n |
| Induction | Base + Step → all n | Proving algorithm correctness, complexity proofs |
| Contradiction | Assume false → derive impossible | Lower bound proofs |
| Loop Invariant | Init + Maintain + Terminate | Algorithm correctness |
| Pigeonhole | n+1 items in n slots → collision | Duplicate finding, hash collisions |
| a mod m | Remainder after division | Hashing, overflow prevention |
| Fast exponentiation | Repeated squaring: O(log n) | Large power computation |
| GCD (Euclid) | GCD(a,b) = GCD(b, a mod b) | O(log n), number theory |

---

## Quick Revision Questions

**Q1.** Why don't we care about the base of logarithm in Big O notation?

<details>
<summary>Answer</summary>
Because changing the base of a logarithm just multiplies by a constant: log_a(n) = log_b(n) / log_b(a). Since Big O drops constants, O(log₂ n) = O(log₁₀ n) = O(ln n) = O(log n). They're all the same asymptotically.
</details>

**Q2.** What is the sum 1 + 2 + 4 + 8 + ... + 2^k and why does it matter for dynamic arrays?

<details>
<summary>Answer</summary>
Sum = 2^(k+1) - 1 ≈ 2 × 2^k. For dynamic arrays: when the array is full, we double its size and copy elements. Over n insertions, total copy cost = 1 + 2 + 4 + ... + n ≈ 2n. So n insertions cost O(2n) total → O(1) amortized per insertion, even though individual resizes are expensive.
</details>

**Q3.** A set of 10 elements has how many subsets? How many permutations?

<details>
<summary>Answer</summary>
Subsets: 2¹⁰ = 1024. Each element is either included or not (2 choices per element). Permutations: 10! = 3,628,800. All possible orderings of 10 elements. Note: 10! >> 2¹⁰, confirming n! grows faster than 2ⁿ.
</details>

**Q4.** Prove using induction that 1 + 3 + 5 + ... + (2n-1) = n².

<details>
<summary>Answer</summary>
Base: n=1: 1 = 1² = 1 ✓. Inductive step: Assume 1+3+...+(2k-1) = k². Then 1+3+...+(2k-1)+(2(k+1)-1) = k² + (2k+1) = (k+1)². ✓. By induction, true for all n ≥ 1.
</details>

**Q5.** Compute 2^100 mod 7 using the pattern-finding approach.

<details>
<summary>Answer</summary>
Find the pattern: 2¹ mod 7 = 2, 2² mod 7 = 4, 2³ mod 7 = 1, 2⁴ mod 7 = 2, 2⁵ mod 7 = 4, 2⁶ mod 7 = 1. Pattern repeats every 3: {2, 4, 1}. 100 mod 3 = 1, so 2^100 mod 7 = 2¹ mod 7 = 2.
</details>

**Q6.** What is the Pigeonhole Principle and how does it relate to hash table collisions?

<details>
<summary>Answer</summary>
Pigeonhole Principle: if n+1 items are placed into n containers, at least one container has 2+ items. For hash tables: if we have m slots and insert m+1 keys, at least one slot must have a collision (two keys mapping to same slot), regardless of how good the hash function is. This is why collision handling (chaining or open addressing) is always necessary.
</details>

---

[← Previous: Recurrence Relations](../06-Recurrence-Relations/06-recurrence-relations.md) | [Back to README](../README.md) | [Next: Problem-Solving Approach →](../08-Problem-Solving-Approach/08-problem-solving-approach.md)
