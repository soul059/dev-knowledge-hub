# Unit 4: Big Omega and Big Theta

[← Previous: Big O Notation](../03-Big-O-Notation/03-big-o-notation.md) | [Back to README](../README.md) | [Next: Space Complexity →](../05-Space-Complexity/05-space-complexity.md)

---

## Chapter Overview

Big O gives us the **upper bound** — the worst case. But sometimes we want to talk about the **lower bound** (best case guarantee) or the **exact growth rate** (tight bound). This unit completes the picture of asymptotic notations.

---

## 4.1 Big Omega (Ω) — Lower Bound

### Intuitive Definition

Big Omega answers: **"What is the MINIMUM number of operations this algorithm will perform?"**

> f(n) = Ω(g(n)) means f grows **at least as fast** as g(n).

### Formal Definition

```
f(n) = Ω(g(n))  if and only if there exist positive constants c and n₀
                 such that:

         f(n) ≥ c · g(n)    for all n ≥ n₀


 In plain English:
 "f(n) grows no slower than g(n), up to a constant factor,
  for sufficiently large n."
```

### Visual Understanding

```
 Operations
    ▲
    │     f(n)
    │    ·
    │   ·
    │  ·            ← f(n) is ALWAYS above c·g(n) after n₀
    │ · 
    │·   c·g(n)
    │·  ╱
    │· ╱
    │·╱
    │╱    n₀
    └──────┼──────────────────────► n

 After n₀, f(n) is always >= c·g(n).
 Therefore f(n) = Ω(g(n))
```

### Example Proof

**Prove:** $3n + 5 = Ω(n)$

```
We need to find c > 0 and n₀ ≥ 0 such that:
    3n + 5 ≥ c·n    for all n ≥ n₀

Step 1: Choose c = 1

    3n + 5 ≥ 1·n
    3n + 5 ≥ n    → always true for all n ≥ 0

Step 2: So with c = 1 and n₀ = 0:

    For n=0:  0 + 5 = 5 ≥ 0  ✓
    For n=1:  3 + 5 = 8 ≥ 1  ✓
    For n=10: 35 ≥ 10         ✓
    ... always holds  ✓

Therefore 3n + 5 = Ω(n)  ✓
```

### What Big Omega Tells Us

```
Algorithm: Linear Search

Best Case:   Target at index 0  →  1 comparison  →  Ω(1)
Worst Case:  Target not found   →  n comparisons →  O(n)

Combined: Linear Search is Ω(1) and O(n)

                ┌─────────────────────────────────┐
                │         Linear Search            │
                │                                  │
                │   Ω(1) ◄── At BEST, 1 check     │
                │                                  │
                │   O(n) ◄── At WORST, n checks    │
                │                                  │
                │   Performance lies somewhere     │
                │   between Ω(1) and O(n)          │
                └─────────────────────────────────┘
```

### Big Omega for Comparison-Based Sorting

```
Important theorem:

  Any comparison-based sorting algorithm has a lower bound of Ω(n log n)
  in the worst case.

  This means NO comparison sort can do better than n log n in the worst case.
  
  ┌─────────────────────────────────────────────────────────┐
  │  Merge Sort:    O(n log n) worst case                   │
  │  Quick Sort:    O(n²) worst case, O(n log n) average    │
  │  Bubble Sort:   O(n²) worst case                        │
  │  Heap Sort:     O(n log n) worst case                   │
  │                                                         │
  │  Merge Sort and Heap Sort are OPTIMAL comparison sorts  │
  │  because they match the Ω(n log n) lower bound!         │
  └─────────────────────────────────────────────────────────┘
```

---

## 4.2 Big Theta (Θ) — Tight Bound

### Intuitive Definition

Big Theta answers: **"What is the EXACT growth rate of this algorithm?"**

> f(n) = Θ(g(n)) means f grows **at the same rate** as g(n).

It's the **sandwich** — f(n) is squeezed between two constant multiples of g(n).

### Formal Definition

```
f(n) = Θ(g(n))  if and only if there exist positive constants c₁, c₂ and n₀
                 such that:

         c₁·g(n) ≤ f(n) ≤ c₂·g(n)    for all n ≥ n₀


 Equivalently:
     f(n) = Θ(g(n))  ⟺  f(n) = O(g(n)) AND f(n) = Ω(g(n))
```

### Visual Understanding — The Sandwich

```
 Operations
    ▲
    │         c₂·g(n)
    │        ╱
    │      ╱    · f(n) is sandwiched
    │    ╱  · ·    between c₁·g(n) and c₂·g(n)
    │  ╱ ·  ╱
    │╱ · ╱ c₁·g(n)
    │·╱
    │╱
    │     n₀
    └──────┼──────────────────────► n

 After n₀, c₁·g(n) ≤ f(n) ≤ c₂·g(n)
 f(n) is "sandwiched" → f(n) = Θ(g(n))
```

### Example Proof

**Prove:** $3n² + 5n + 2 = Θ(n²)$

```
We need: c₁·n² ≤ 3n² + 5n + 2 ≤ c₂·n²  for all n ≥ n₀

UPPER BOUND (Big O part):
    3n² + 5n + 2 ≤ 3n² + 5n² + 2n²    (for n ≥ 1: n ≤ n², 2 ≤ 2n²)
                  = 10n²
    So c₂ = 10 works.

LOWER BOUND (Big Omega part):
    3n² + 5n + 2 ≥ 3n²                 (since 5n + 2 ≥ 0 for n ≥ 0)
    So c₁ = 3 works.

With c₁ = 3, c₂ = 10, n₀ = 1:
    3·n² ≤ 3n² + 5n + 2 ≤ 10·n²  for all n ≥ 1  ✓

Therefore 3n² + 5n + 2 = Θ(n²)  ✓
```

### When Can We Use Theta?

```
┌─────────────────────────────────────────────────────────────────┐
│  Θ can be used when BEST CASE = WORST CASE (same growth rate)  │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  Merge Sort:                                                    │
│    Best case:  O(n log n)                                       │
│    Worst case: O(n log n)                                       │
│    → Merge Sort is Θ(n log n)  ✓                                │
│                                                                 │
│  Selection Sort:                                                │
│    Best case:  O(n²)                                            │
│    Worst case: O(n²)                                            │
│    → Selection Sort is Θ(n²)  ✓                                 │
│                                                                 │
│  Quick Sort:                                                    │
│    Best case:  O(n log n)                                       │
│    Worst case: O(n²)                                            │
│    → Quick Sort is NOT Θ(anything) for all cases                │
│    → We say: Best Θ(n log n), Worst Θ(n²)                      │
│                                                                 │
│  Insertion Sort:                                                │
│    Best case:  O(n)                                             │
│    Worst case: O(n²)                                            │
│    → NOT Θ(n²) overall (best case is faster)                    │
│    → Worst case is Θ(n²), Best case is Θ(n)                    │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 4.3 Little o and Little omega

### Little o — Strict Upper Bound

```
f(n) = o(g(n))  means f grows STRICTLY SLOWER than g(n).

 Formal: For ALL constants c > 0, there exists n₀ such that
         f(n) < c·g(n) for all n ≥ n₀

 Alternatively:  lim(n→∞) f(n)/g(n) = 0

 Big O:    f(n) ≤ c·g(n)     "f grows at most as fast as g"
 Little o: f(n) < c·g(n)     "f grows strictly slower than g"
```

### Examples of Little o

```
 n = o(n²)          ✓  n grows slower than n²
   lim(n→∞) n/n² = lim(n→∞) 1/n = 0  ✓

 n² = o(n³)         ✓  n² grows slower than n³
   lim(n→∞) n²/n³ = lim(n→∞) 1/n = 0  ✓

 n = o(n)           ✗  n does NOT grow strictly slower than n
   lim(n→∞) n/n = 1 ≠ 0  ✗
   (n = O(n) but n ≠ o(n))

 log n = o(n)       ✓  log n grows strictly slower than n
 n = o(2ⁿ)          ✓  n grows strictly slower than 2ⁿ
 100n = o(n²)       ✓  linear is strictly slower than quadratic
```

### Little ω — Strict Lower Bound

```
f(n) = ω(g(n))  means f grows STRICTLY FASTER than g(n).

 Formal: For ALL constants c > 0, there exists n₀ such that
         f(n) > c·g(n) for all n ≥ n₀

 Alternatively:  lim(n→∞) f(n)/g(n) = ∞

 Big Ω:      f(n) ≥ c·g(n)     "f grows at least as fast as g"
 Little ω:   f(n) > c·g(n)     "f grows strictly faster than g"
```

### Examples of Little ω

```
 n² = ω(n)          ✓  n² grows faster than n
   lim(n→∞) n²/n = lim(n→∞) n = ∞  ✓

 n³ = ω(n²)         ✓
 2ⁿ = ω(n³)         ✓  exponential grows faster than any polynomial
 n = ω(log n)       ✓
 n = ω(n)           ✗  n does NOT grow strictly faster than n
```

### Relationship Summary

```
┌─────────┬────────────┬─────────────────────────────────────────┐
│Notation │  Analogy   │              Meaning                    │
├─────────┼────────────┼─────────────────────────────────────────┤
│ o(g)    │  f < g     │ f grows strictly slower than g           │
│ O(g)    │  f ≤ g     │ f grows no faster than g (upper bound)  │
│ Θ(g)    │  f = g     │ f grows at the same rate as g           │
│ Ω(g)    │  f ≥ g     │ f grows no slower than g (lower bound)  │
│ ω(g)    │  f > g     │ f grows strictly faster than g           │
└─────────┴────────────┴─────────────────────────────────────────┘

 Mathematical analogy:
   o  →  <    (strictly less than)
   O  →  ≤    (less than or equal)
   Θ  →  =    (equal)
   Ω  →  ≥    (greater than or equal)
   ω  →  >    (strictly greater than)
```

---

## 4.4 When to Use Which Notation

### Decision Guide

```
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  "This algorithm takes AT MOST n² steps"                        │
│   → Use Big O: O(n²)                                           │
│   → Upper bound / worst-case guarantee                          │
│   → MOST COMMON in practice                                    │
│                                                                 │
│  "This algorithm takes AT LEAST n steps"                        │
│   → Use Big Ω: Ω(n)                                            │
│   → Lower bound / best-case guarantee                           │
│   → Common in proving optimality                                │
│                                                                 │
│  "This algorithm takes EXACTLY n log n steps (asymptotically)"  │
│   → Use Big Θ: Θ(n log n)                                      │
│   → Tight bound / both upper and lower                          │
│   → Most precise, but requires same best & worst growth         │
│                                                                 │
│  "This algorithm is STRICTLY faster than n²"                    │
│   → Use little o: o(n²)                                        │
│   → Strict upper bound                                          │
│   → Common in theoretical CS papers                             │
│                                                                 │
│  "This problem requires STRICTLY more than n steps"             │
│   → Use little ω: ω(n)                                         │
│   → Strict lower bound                                          │
│   → Rare in practice                                            │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### In Practice: What People Actually Use

```
┌──────────────────────────────────────────────────────────────┐
│                                                              │
│  In industry / interviews:                                   │
│  ─────────────────────────                                   │
│  • Almost always Big O                                       │
│  • "This is O(n log n)" = "worst case is n log n"           │
│  • Technically imprecise but universally understood           │
│                                                              │
│  In academia / research:                                     │
│  ─────────────────────────                                   │
│  • All notations used precisely                              │
│  • Θ preferred when tight bound is known                    │
│  • Ω used to prove algorithm optimality                     │
│  • o and ω used in proofs and comparisons                   │
│                                                              │
│  Tip: If someone says "the time complexity is O(n²)",        │
│  they usually mean this is the worst-case upper bound.       │
│  If the algorithm is always n² (even best case), then        │
│  Θ(n²) would be more precise.                               │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## 4.5 Practical Implications

### Why Lower Bounds Matter

```
Problem: Sorting n numbers using comparisons

 Known lower bound: Ω(n log n)

 This means:
 ┌────────────────────────────────────────────────────────────┐
 │ No matter how clever you are, you CANNOT sort n numbers   │
 │ using comparisons in fewer than n log n steps              │
 │ (in the worst case).                                       │
 │                                                            │
 │ So if your sorting algorithm is O(n log n) worst case,     │
 │ you've achieved the OPTIMAL complexity!                    │
 │                                                            │
 │ Examples of optimal sorts: Merge Sort, Heap Sort           │
 └────────────────────────────────────────────────────────────┘
```

### Proving Optimality

```
To prove an algorithm is OPTIMAL:

 Step 1: Show the algorithm is O(f(n))        ← Upper bound
 Step 2: Show the PROBLEM has a lower bound Ω(f(n))  ← Can't do better
 Step 3: Since O(f(n)) = Ω(f(n)), the algorithm is Θ(f(n)) — optimal!

 Example:
   Merge Sort is O(n log n)                    ← We proved this
   Sorting requires Ω(n log n) comparisons     ← Decision tree argument
   Therefore Merge Sort is optimal: Θ(n log n) ✓
```

### Comparison of All Notations on One Function

```
 Let f(n) = 3n² + 5n + 2

 True statements:                 False statements:
 ──────────────                   ────────────────
 f(n) = O(n²)    ✓                f(n) = O(n)     ✗  (n² grows faster than n)
 f(n) = O(n³)    ✓  (loose)       f(n) = Θ(n)     ✗  (not tight)
 f(n) = O(2ⁿ)   ✓  (very loose)  f(n) = Ω(n³)    ✗  (n² doesn't grow ≥ n³)
 f(n) = Ω(n²)   ✓                f(n) = o(n²)    ✗  (not strictly less)
 f(n) = Ω(n)    ✓  (loose)
 f(n) = Θ(n²)   ✓  (tight!)
 f(n) = o(n³)   ✓  (strictly less)
 f(n) = ω(n)    ✓  (strictly more)

 The TIGHTEST correct bound is Θ(n²).
```

### Common Mistakes

```
┌─────────────────────────────────────────────────────────────────┐
│                     Common Mistakes                             │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│ ✗ "Big O is the worst case"                                    │
│   → Big O is an upper bound. You CAN say:                      │
│     "The best case is O(1)" — that's valid!                    │
│     Big O bounds a function, not a specific case.              │
│                                                                 │
│ ✗ "O(n) means exactly n operations"                            │
│   → O(n) means AT MOST c·n operations for large n.            │
│     Could be n, 5n, or n/2 — all are O(n).                    │
│                                                                 │
│ ✗ "O(n²) is bad" / "O(n) is good"                             │
│   → It depends on the problem! O(n²) might be optimal         │
│     for some problems. Always compare against the lower bound. │
│                                                                 │
│ ✗ Using O when Θ is more precise                               │
│   → "Merge Sort is O(n log n)" is correct but loose.          │
│     "Merge Sort is Θ(n log n)" is more precise.               │
│                                                                 │
│ ✗ Confusing Big O of an algorithm vs Big O of a problem        │
│   → Algorithm: "This algorithm runs in O(n²)"                 │
│   → Problem: "This problem requires Ω(n log n)"              │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## Summary Table

| Notation | Name | Symbol | Meaning | Analogy |
|----------|------|--------|---------|---------|
| Big O | Upper Bound | O(g(n)) | f(n) ≤ c·g(n) for large n | f ≤ g |
| Big Omega | Lower Bound | Ω(g(n)) | f(n) ≥ c·g(n) for large n | f ≥ g |
| Big Theta | Tight Bound | Θ(g(n)) | c₁·g(n) ≤ f(n) ≤ c₂·g(n) | f = g |
| Little o | Strict Upper | o(g(n)) | f(n) < c·g(n) for ALL c | f < g |
| Little omega | Strict Lower | ω(g(n)) | f(n) > c·g(n) for ALL c | f > g |
| Most Common | Industry usage | O(·) | Upper bound / worst case | — |
| Most Precise | Academic usage | Θ(·) | Tight bound when possible | — |
| Optimality | Proving optimal | Ω(·) | Problem lower bound | — |

---

## Quick Revision Questions

**Q1.** What is the difference between Big O and Big Theta? When would you prefer one over the other?

<details>
<summary>Answer</summary>
Big O provides an upper bound (at most this fast), while Big Theta provides a tight bound (exactly this fast asymptotically). Use Θ when the best and worst cases have the same growth rate (e.g., Merge Sort is Θ(n log n)). Use O when best and worst differ (e.g., Quick Sort is O(n²) worst, but Θ notation can't capture both cases in one statement).
</details>

**Q2.** Is 5n² = o(n²)? Is 5n² = O(n²)? Explain the difference.

<details>
<summary>Answer</summary>
5n² = O(n²) ✓ — because 5n² ≤ c·n² for c=5. 5n² = o(n²) ✗ — because o means "strictly slower growth." 5n² and n² grow at the same rate (both are quadratic), so 5n² is NOT strictly slower than n². Little o requires the limit f(n)/g(n) → 0 as n → ∞, but 5n²/n² = 5 ≠ 0.
</details>

**Q3.** If an algorithm's best case is O(n) and worst case is O(n²), can we say it's Θ(n²)?

<details>
<summary>Answer</summary>
No. Θ(n²) means the algorithm ALWAYS takes n² (for large n), including the best case. Since the best case is O(n) — which is better than n² — the algorithm is NOT Θ(n²) overall. We can say the worst case is Θ(n²) and the best case is Θ(n), but not a single Θ for the whole algorithm.
</details>

**Q4.** Why does the Ω(n log n) lower bound for comparison sorting matter?

<details>
<summary>Answer</summary>
It tells us that no comparison-based sorting algorithm can ever do better than n log n in the worst case. This means algorithms like Merge Sort and Heap Sort (which are O(n log n)) are provably optimal — we can't improve them. Any claimed comparison sort "faster" than n log n must be wrong. (Non-comparison sorts like Counting Sort can beat this, but have different constraints.)
</details>

**Q5.** Classify true or false: n³ = O(n²), n = Ω(log n), 2ⁿ = Θ(3ⁿ).

<details>
<summary>Answer</summary>
n³ = O(n²): FALSE — n³ grows faster than n², so n² is not an upper bound for n³.
n = Ω(log n): TRUE — n grows at least as fast as log n.
2ⁿ = Θ(3ⁿ): FALSE — 2ⁿ grows strictly slower than 3ⁿ. lim(n→∞) 2ⁿ/3ⁿ = lim (2/3)ⁿ = 0, so 2ⁿ = o(3ⁿ), not Θ(3ⁿ).
</details>

**Q6.** In a job interview, someone says "Binary Search is O(n)." Are they wrong? Explain.

<details>
<summary>Answer</summary>
Technically, they're not wrong — but they're being imprecise. Binary Search IS O(n) because O gives an upper bound, and log n ≤ n. It's also O(n²), O(n³), O(2ⁿ) — all are valid upper bounds! But the tightest correct statement is O(log n) or better yet Θ(log n). In an interview, always give the tightest bound.
</details>

---

[← Previous: Big O Notation](../03-Big-O-Notation/03-big-o-notation.md) | [Back to README](../README.md) | [Next: Space Complexity →](../05-Space-Complexity/05-space-complexity.md)
