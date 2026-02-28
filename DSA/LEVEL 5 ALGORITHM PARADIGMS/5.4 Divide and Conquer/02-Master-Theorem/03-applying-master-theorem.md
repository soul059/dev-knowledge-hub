# Chapter 3: Applying the Master Theorem

## Overview

This chapter provides a systematic, step-by-step method for applying the Master Theorem to any recurrence. Follow this procedure consistently and you'll never get the analysis wrong.

---

## The 5-Step Procedure

```
┌─────────────────────────────────────────────────────────────┐
│           HOW TO APPLY THE MASTER THEOREM                  │
│                                                             │
│  Step 1: Verify the recurrence is in standard form         │
│          T(n) = a·T(n/b) + f(n)                           │
│                                                             │
│  Step 2: Extract a, b, and f(n)                            │
│                                                             │
│  Step 3: Compute c_crit = log_b(a)                         │
│                                                             │
│  Step 4: Determine the exponent c of f(n)                  │
│          (i.e., f(n) = Θ(n^c) or Θ(n^c · log^k n))       │
│                                                             │
│  Step 5: Compare c with c_crit to identify the case        │
│          • c < c_crit → Case 1                             │
│          • c = c_crit → Case 2                             │
│          • c > c_crit → Case 3 (+ check regularity)       │
│                                                             │
│  Step 6: Write the answer                                  │
└─────────────────────────────────────────────────────────────┘
```

---

## Worked Example 1: Merge Sort

**Recurrence**: `T(n) = 2T(n/2) + n`

```
Step 1: Standard form? ✓  T(n) = a·T(n/b) + f(n)

Step 2: Extract parameters
  a = 2  (two subproblems)
  b = 2  (each is half the size)
  f(n) = n

Step 3: c_crit = log_b(a) = log₂(2) = 1

Step 4: f(n) = n = n¹  → c = 1

Step 5: Compare c with c_crit
  c = 1 = c_crit = 1  → CASE 2 (k = 0)

Step 6: T(n) = Θ(n^c_crit · log n) = Θ(n · log n) = Θ(n log n) ✓
```

---

## Worked Example 2: Binary Search

**Recurrence**: `T(n) = T(n/2) + 1`

```
Step 1: Standard form? ✓

Step 2: a = 1, b = 2, f(n) = 1 = n⁰

Step 3: c_crit = log₂(1) = 0

Step 4: f(n) = 1 = n⁰ → c = 0

Step 5: c = 0 = c_crit = 0 → CASE 2 (k = 0)

Step 6: T(n) = Θ(n⁰ · log n) = Θ(log n) ✓
```

---

## Worked Example 3: Strassen's Algorithm

**Recurrence**: `T(n) = 7T(n/2) + n²`

```
Step 1: Standard form? ✓

Step 2: a = 7, b = 2, f(n) = n²

Step 3: c_crit = log₂(7) ≈ 2.807

Step 4: f(n) = n² → c = 2

Step 5: c = 2 < c_crit = 2.807 → CASE 1

Step 6: T(n) = Θ(n^(log₂ 7)) = Θ(n^2.807) ✓
```

---

## Worked Example 4: A Root-Heavy Example

**Recurrence**: `T(n) = 2T(n/2) + n²`

```
Step 1: Standard form? ✓

Step 2: a = 2, b = 2, f(n) = n²

Step 3: c_crit = log₂(2) = 1

Step 4: f(n) = n² → c = 2

Step 5: c = 2 > c_crit = 1 → CASE 3
  
  Regularity check: a·f(n/b) ≤ k·f(n)?
  2 · (n/2)² = 2 · n²/4 = n²/2 ≤ ½ · n²  ✓  (k = ½ < 1)

Step 6: T(n) = Θ(f(n)) = Θ(n²) ✓
```

---

## Worked Example 5: Non-Standard f(n)

**Recurrence**: `T(n) = 2T(n/4) + √n`

```
Step 1: Standard form? ✓

Step 2: a = 2, b = 4, f(n) = √n = n^(1/2)

Step 3: c_crit = log₄(2) = 1/2 = 0.5

Step 4: f(n) = n^(1/2) → c = 0.5

Step 5: c = 0.5 = c_crit = 0.5 → CASE 2 (k = 0)

Step 6: T(n) = Θ(n^(1/2) · log n) = Θ(√n · log n) ✓
```

---

## Worked Example 6: Three-Way Split

**Recurrence**: `T(n) = 3T(n/3) + n`

```
Step 1: Standard form? ✓

Step 2: a = 3, b = 3, f(n) = n

Step 3: c_crit = log₃(3) = 1

Step 4: f(n) = n → c = 1

Step 5: c = 1 = c_crit = 1 → CASE 2 (k = 0)

Step 6: T(n) = Θ(n · log n) ✓

Note: Same asymptotic result as Merge Sort!
      Because a/b ratio and f(n) balance the same way.
```

---

## Worked Example 7: Leaf-Heavy

**Recurrence**: `T(n) = 4T(n/2) + n`

```
Step 1: Standard form? ✓

Step 2: a = 4, b = 2, f(n) = n

Step 3: c_crit = log₂(4) = 2

Step 4: f(n) = n → c = 1

Step 5: c = 1 < c_crit = 2 → CASE 1

Step 6: T(n) = Θ(n²) ✓

Visualization:
Level 0: n
Level 1: 4·(n/2) = 2n          ← doubled
Level 2: 16·(n/4) = 4n         ← doubled
Level 3: 64·(n/8) = 8n         ← keeps growing!
Leaves dominate → Θ(n²)
```

---

## Common Pitfalls and How to Avoid Them

### Pitfall 1: Floor/Ceiling in n/b

```
T(n) = 2T(⌊n/2⌋) + n

This is FINE — the Master Theorem still applies!
Floor and ceiling don't affect asymptotic results.
```

### Pitfall 2: Forgetting the Regularity Condition in Case 3

```
Always check: a · f(n/b) ≤ k · f(n) for some k < 1

For polynomial f(n) = n^c, regularity almost always holds.
For non-polynomial f(n), be careful!
```

### Pitfall 3: f(n) is not polynomial

```
T(n) = 2T(n/2) + n log n

f(n) = n log n, c_crit = log₂(2) = 1
Is n log n = Θ(n^c)?
  n log n > n¹ but n log n < n^(1+ε) for any ε > 0
  
This is Case 2 with k = 1!
f(n) = Θ(n¹ · log¹ n) → T(n) = Θ(n · log² n)
```

### Pitfall 4: Non-Standard Recurrence

```
T(n) = T(n/3) + T(2n/3) + n

This doesn't fit T(n) = a·T(n/b) + f(n) because
subproblems have DIFFERENT sizes!

Master Theorem CANNOT be applied directly.
→ Use Akra-Bazzi or recursion tree method instead.
```

---

## Practice Problems with Solutions

### Problem 1: T(n) = 9T(n/3) + n

```
a = 9, b = 3, f(n) = n
c_crit = log₃(9) = 2
c = 1 < 2 → Case 1
T(n) = Θ(n²)
```

### Problem 2: T(n) = T(n/2) + n

```
a = 1, b = 2, f(n) = n
c_crit = log₂(1) = 0
c = 1 > 0 → Case 3
Regularity: 1 · f(n/2) = n/2 ≤ ½ · n ✓
T(n) = Θ(n)
```

### Problem 3: T(n) = 3T(n/4) + n log n

```
a = 3, b = 4, f(n) = n log n
c_crit = log₄(3) ≈ 0.793
f(n) = n log n = Ω(n^1), and 1 > 0.793 → Case 3
Regularity: 3 · (n/4)log(n/4) = (3n/4)(log n - log 4) ≤ (3/4) · n log n ✓ (for large n)
T(n) = Θ(n log n)
```

### Problem 4: T(n) = 2T(n/2) + n/log n

```
a = 2, b = 2, f(n) = n/log n
c_crit = 1
f(n) = n/log n → Is this n^c for some c?
  n/log n < n¹ but n/log n > n^(1-ε) for any ε > 0
  
  f(n) is NOT polynomially smaller or larger!
  ★ MASTER THEOREM DOES NOT APPLY ★
  → Must use other methods (substitution, Akra-Bazzi)
  Answer: T(n) = Θ(n · log log n)
```

---

## Quick Decision Table

```
Given T(n) = a·T(n/b) + n^c:

If c < log_b(a): → Θ(n^(log_b a))          [Leaf wins]
If c = log_b(a): → Θ(n^c · log n)           [Tie]  
If c > log_b(a): → Θ(n^c)                   [Root wins]
```

---

## Summary Table

| Step | Action | Key Check |
|------|--------|-----------|
| 1 | Verify standard form | T(n) = a·T(n/b) + f(n) |
| 2 | Extract a, b, f(n) | a ≥ 1, b > 1 |
| 3 | Compute c_crit | c_crit = log_b(a) |
| 4 | Find c from f(n) | f(n) = Θ(n^c · log^k n) |
| 5 | Compare c vs c_crit | Determines the case |
| 6 | Apply the formula | Write Θ notation |

---

## Quick Revision Questions

1. **Apply the Master Theorem to T(n) = 16T(n/4) + n. What is T(n)?**
2. **For T(n) = 2T(n/2) + n³, show all steps of applying the theorem.**
3. **Why does T(n) = 2T(n/2) + n/log n fail the Master Theorem?**
4. **What is the time complexity of T(n) = 3T(n/3) + n² ?**
5. **For T(n) = 5T(n/5) + n, identify the case and solve.**
6. **Can the Master Theorem be applied to T(n) = T(n/3) + T(2n/3) + n? Why or why not?**

---

[← Previous: Three Cases](02-three-cases.md) | [Next: Worked Examples →](04-worked-examples.md)
