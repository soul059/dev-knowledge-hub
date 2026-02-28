# Chapter 5: Extended Master Theorem

## Overview

The standard Master Theorem handles `T(n) = aT(n/b) + f(n)` where `f(n) = Θ(n^c)`. But what about logarithmic factors, non-polynomial functions, or unequal subproblem sizes? This chapter covers extensions that handle these situations.

---

## The Extended Case 2 (With Logarithmic Factors)

The standard Case 2 handles `f(n) = Θ(n^c_crit)`. The extended version handles:

```
┌─────────────────────────────────────────────────────────────┐
│  EXTENDED CASE 2:                                          │
│                                                             │
│  If f(n) = Θ(n^(log_b(a)) · log^k(n))  for k ≥ 0         │
│                                                             │
│  Then T(n) = Θ(n^(log_b(a)) · log^(k+1)(n))               │
│                                                             │
│  The log exponent k increases by 1!                        │
└─────────────────────────────────────────────────────────────┘
```

### Examples

| Recurrence | k | T(n) |
|-----------|---|------|
| T(n) = 2T(n/2) + n | 0 | Θ(n log n) |
| T(n) = 2T(n/2) + n log n | 1 | Θ(n log²n) |
| T(n) = 2T(n/2) + n log²n | 2 | Θ(n log³n) |
| T(n) = 4T(n/2) + n² log n | 1 | Θ(n² log²n) |

### Trace: T(n) = 2T(n/2) + n log n

```
Level 0: n log n
Level 1: 2 · (n/2) · log(n/2) = n · (log n - 1)
Level 2: 4 · (n/4) · log(n/4) = n · (log n - 2)
...
Level k: n · (log n - k)

Total = Σ(k=0 to log n) n · (log n - k)
      = n · Σ(k=0 to log n) (log n - k)
      = n · (log²n / 2)
      = Θ(n log²n) ✓
```

---

## The Akra-Bazzi Theorem (Unequal Splits)

When subproblems have **different sizes**, the standard Master Theorem fails:

```
T(n) = T(n/3) + T(2n/3) + n    ← Unequal splits!
                                   Can't use standard form
```

### The Akra-Bazzi Method

```
┌──────────────────────────────────────────────────────────────┐
│  AKRA-BAZZI THEOREM                                         │
│                                                              │
│  Given: T(n) = Σᵢ aᵢ · T(n · bᵢ) + f(n)                   │
│         where 0 < bᵢ < 1 for all i                          │
│                                                              │
│  Step 1: Find p such that Σᵢ aᵢ · bᵢ^p = 1                 │
│                                                              │
│  Step 2: T(n) = Θ(n^p · (1 + ∫₁ⁿ f(u)/u^(p+1) du))       │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

### Example: T(n) = T(n/3) + T(2n/3) + n

```
Step 1: Find p where:
  1 · (1/3)^p + 1 · (2/3)^p = 1
  
  Try p = 1: (1/3)¹ + (2/3)¹ = 1/3 + 2/3 = 1 ✓
  
  So p = 1.

Step 2: T(n) = Θ(n¹ · (1 + ∫₁ⁿ u/u² du))
              = Θ(n · (1 + ∫₁ⁿ 1/u du))
              = Θ(n · (1 + ln n))
              = Θ(n log n)
```

---

## Recursion Tree Method (Alternative to Master Theorem)

When the Master Theorem doesn't apply, draw the recursion tree:

### Example: T(n) = T(n/3) + T(2n/3) + cn

```
                        cn                          Level 0: cn
                      /    \
                 cn/3       2cn/3                   Level 1: cn
                /   \       /     \
           cn/9   2cn/9  2cn/9   4cn/9             Level 2: cn
            ...     ...    ...     ...

Each level sums to AT MOST cn.
Height = log_{3/2}(n) (determined by the larger branch: 2n/3)

Because (2/3)^h · n = 1 → h = log_{3/2}(n)

Total work ≤ cn · log_{3/2}(n) = Θ(n log n)

Lower bound: Most nodes exist down to depth log₃(n)
             Each such level also has ≥ cn total work

Therefore: T(n) = Θ(n log n)
```

```
Visual: Recursion tree shape (unbalanced)

            cn
          /     \
       cn/3    2cn/3
       / \      / \
      ·   ·    ·   ·
     /\  /\   /\  /\
    ·  · · · ·  · · ·
    
    Left branch ends sooner (depth log₃ n)
    Right branch ends later (depth log_{3/2} n)
    
    But total work per level is still ≈ cn
```

---

## Substitution Method (For Verification)

Use this to verify Master Theorem results or handle non-standard cases:

### Example: Verify T(n) = 2T(n/2) + n gives T(n) = O(n log n)

```
Claim: T(n) ≤ cn log n for some constant c.

Proof by Strong Induction:
Assume T(k) ≤ ck log k for all k < n.

T(n) = 2T(n/2) + n
     ≤ 2 · c(n/2) · log(n/2) + n     [by inductive hypothesis]
     = cn · (log n - 1) + n
     = cn log n - cn + n
     = cn log n - (c-1)n
     ≤ cn log n                       [if c ≥ 1]

Therefore T(n) = O(n log n) ✓
```

### Example: T(n) = T(n/2) + c (Binary Search → O(log n))

```
Claim: T(n) ≤ d · log n

T(n) = T(n/2) + c
     ≤ d · log(n/2) + c
     = d · (log n - 1) + c
     = d · log n - d + c
     ≤ d · log n          [if d ≥ c]

Therefore T(n) = O(log n) ✓
```

---

## Special Cases Not Covered by Standard Master Theorem

### Case: f(n) = n^c_crit / log n

```
T(n) = 2T(n/2) + n/log n

c_crit = log₂(2) = 1
f(n) = n/log n

n/log n is between n^(1-ε) and n^1 for any ε > 0
→ NOT polynomially smaller, NOT equal
→ Standard Master Theorem FAILS

By recursion tree or substitution:
T(n) = Θ(n · log log n)
```

### Case: Exponential Number of Subproblems

```
T(n) = 2^n · T(n/2) + n

This doesn't fit Master Theorem at all!
a = 2^n is not a constant.

Must solve by other methods entirely.
```

### Case: Additive Subproblem Sizes

```
T(n) = T(n-1) + T(n-2) + 1    (Fibonacci-like)

Not in the form T(n) = aT(n/b) + f(n)
because n-1 ≠ n/b for any constant b.

Solution: T(n) = Θ(φ^n) where φ = (1+√5)/2
```

---

## Master Theorem Extensions Comparison

| Method | Handles | Complexity to Use |
|--------|---------|-------------------|
| **Standard Master Theorem** | T(n) = aT(n/b) + n^c | Easy — plug into formula |
| **Extended Case 2** | f(n) = n^c_crit · log^k n | Easy — add 1 to k |
| **Akra-Bazzi** | Unequal subproblem sizes | Medium — solve for p, integrate |
| **Substitution** | Any recurrence | Medium — guess and verify |
| **Recursion Tree** | Any recurrence | Visual but can be tricky |

---

## Summary Table

| Extension | When to Use | Key Formula |
|-----------|-------------|-------------|
| Extended Case 2 | f(n) has log factors | k → k+1 in the exponent |
| Akra-Bazzi | Different subproblem sizes | Solve Σ aᵢ bᵢ^p = 1 |
| Substitution | Verification or non-standard | Guess, prove by induction |
| Recursion Tree | Visual understanding | Sum work across all levels |

---

## Quick Revision Questions

1. **What is the result of T(n) = 2T(n/2) + n log²n using the extended Master Theorem?**
2. **For T(n) = T(n/3) + T(2n/3) + n, why can't the standard Master Theorem be used?**
3. **What value of p solves the Akra-Bazzi equation for T(n) = T(n/3) + T(2n/3) + n?**
4. **Use the substitution method to verify T(n) = T(n/2) + 1 = O(log n).**
5. **Why does T(n) = 2T(n/2) + n/log n not fit into any standard Master Theorem case?**
6. **Compare the recursion tree method vs the substitution method — when would you prefer each?**

---

[← Previous: Worked Examples](04-worked-examples.md) | [Next: When It Doesn't Apply →](06-when-it-doesnt-apply.md)
