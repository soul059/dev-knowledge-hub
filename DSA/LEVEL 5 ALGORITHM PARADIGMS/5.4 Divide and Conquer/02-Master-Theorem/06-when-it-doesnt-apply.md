# Chapter 6: When the Master Theorem Doesn't Apply

## Overview

The Master Theorem is powerful but not universal. This chapter identifies the situations where it fails and provides alternative methods for those cases.

---

## Conditions for the Master Theorem

```
The Master Theorem requires ALL of these:

┌─────────────────────────────────────────────────────┐
│  1. a ≥ 1              (at least 1 subproblem)     │
│  2. b > 1              (problems must shrink)      │
│  3. a and b are CONSTANTS (not functions of n)      │
│  4. All subproblems have EQUAL size (n/b)           │
│  5. f(n) is polynomially comparable to n^(log_b a) │
│  6. For Case 3: regularity condition holds          │
└─────────────────────────────────────────────────────┘

If ANY condition is violated → Master Theorem FAILS
```

---

## Failure Case 1: Unequal Subproblem Sizes

```
T(n) = T(n/3) + T(2n/3) + n

Why it fails:
  Subproblems have sizes n/3 and 2n/3 — NOT both n/b.
  Can't write as T(n) = a·T(n/b) + f(n)

Solution approach: Akra-Bazzi or Recursion Tree
Answer: T(n) = Θ(n log n)

Common algorithms with this pattern:
  • Unbalanced Quick Sort
  • Randomized algorithms (expected case)
```

---

## Failure Case 2: a or b Depends on n

```
T(n) = n · T(n/2) + 1

Why it fails:
  a = n is NOT a constant — it depends on input size.

T(n) = T(√n) + 1

Why it fails:
  Subproblem size = √n ≠ n/b for any constant b.
  (Here the problem shrinks by a square root, not a fraction)

Solution for T(n) = T(√n) + 1:
  Let m = log n, so n = 2^m
  T(2^m) = T(2^(m/2)) + 1
  Let S(m) = T(2^m)
  S(m) = S(m/2) + 1  ← NOW Master Theorem applies!
  a=1, b=2, f(m)=1 → Case 2 → S(m) = Θ(log m)
  T(n) = Θ(log log n)
```

---

## Failure Case 3: Non-Polynomial Gap

The Master Theorem requires that `f(n)` is **polynomially** smaller, equal, or larger than `n^(log_b a)`.

```
T(n) = 2T(n/2) + n/log n

c_crit = log₂(2) = 1
f(n) = n/log n

Is n/log n = O(n^(1-ε)) for some ε > 0?
  NO! n/log n grows faster than n^(1-ε) for any ε > 0.

Is n/log n = Θ(n)?
  NO! n/log n grows slower than n.

The gap between n/log n and n is only LOGARITHMIC, not polynomial.
→ Falls in the "gap" between Case 1 and Case 2.
→ Master Theorem DOES NOT APPLY!

Solution: By recursion tree analysis:
T(n) = Θ(n · log log n)
```

```
Visual: The gap where MT fails

      n^(1-ε)    n/log n    n^1      n · log n    n^(1+ε)
         │          │        │           │            │
         ├──────────┤        │           │            │
         │  Gap!    │        │           │            │
         │  MT fails│        │           │            │
         └──────────┘        │           │            │
                        Case 2 boundary
```

---

## Failure Case 4: Regularity Condition Fails (Case 3)

```
For Case 3, we need: a · f(n/b) ≤ k · f(n) for some k < 1

This usually holds for polynomial f(n), but can fail
for oscillating or pathological functions.

Example (theoretical):
T(n) = T(n/2) + n · (2 - cos(n))

f(n) = n · (2 - cos(n))  oscillates between n and 3n
The regularity condition may not hold uniformly.
→ Master Theorem technically inapplicable.

In practice: This is rare for real algorithms.
             All standard D&C algorithms satisfy regularity.
```

---

## Failure Case 5: Subtractive Recurrences

```
T(n) = T(n-1) + n          ← Subtractive, not multiplicative!

This is NOT of the form T(n) = aT(n/b) + f(n)
because (n-1) ≠ n/b for any constant b.

Solution approach: Unroll the recurrence
T(n) = T(n-1) + n
     = T(n-2) + (n-1) + n
     = T(n-3) + (n-2) + (n-1) + n
     = ... 
     = 1 + 2 + ... + n
     = n(n+1)/2 = Θ(n²)

Other subtractive examples:
T(n) = T(n-1) + 1         → Θ(n)
T(n) = T(n-1) + log n     → Θ(n log n)
T(n) = 2T(n-1) + 1        → Θ(2^n)    [Exponential!]
```

---

## Failure Case 6: Multiple Distinct Recursions

```
T(n) = T(n/2) + T(n/4) + n

Why it fails:
  Two subproblems but different sizes (n/2 and n/4)
  Not in form a·T(n/b) — both a and b vary.

Solution: Akra-Bazzi
  Find p: (1/2)^p + (1/4)^p = 1
  Try p = 0.695... (solve numerically)
  T(n) = Θ(n^0.695... · (1 + ∫...))

  Or by recursion tree: T(n) = Θ(n)
  (Heavy root work dominates)
```

---

## Alternative Methods Summary

### Method 1: Recursion Tree
```
Best for: Visual understanding, unequal splits
Process:
  1. Draw the tree with costs at each node
  2. Sum costs per level
  3. Sum across all levels
  4. Identify if geometric series → first/last/all terms
```

### Method 2: Substitution Method
```
Best for: Verification, tight bounds
Process:
  1. Guess the answer (educated guess)
  2. Assume T(k) ≤ f(k) for k < n (inductive hypothesis)
  3. Prove T(n) ≤ f(n) using the recurrence
  4. Verify base case
```

### Method 3: Akra-Bazzi
```
Best for: Unequal subproblem sizes
Process:
  1. Write T(n) = Σ aᵢ · T(bᵢ · n) + f(n)
  2. Solve Σ aᵢ · bᵢ^p = 1 for p
  3. T(n) = Θ(n^p · (1 + ∫₁ⁿ f(u)/u^(p+1) du))
```

### Method 4: Change of Variables
```
Best for: Non-standard subproblem sizes (√n, log n)
Process:
  1. Substitute m = log n (or other transform)
  2. Rewrite recurrence in terms of m
  3. Apply Master Theorem to the new recurrence
  4. Substitute back
```

---

## Change of Variables: Detailed Example

### T(n) = 2T(√n) + log n

```
Step 1: Let m = log₂ n   (so n = 2^m, √n = 2^(m/2))

Step 2: T(2^m) = 2T(2^(m/2)) + m

Step 3: Let S(m) = T(2^m)
        S(m) = 2S(m/2) + m

Step 4: Apply Master Theorem to S(m):
        a = 2, b = 2, f(m) = m
        c_crit = log₂(2) = 1
        f(m) = m → c = 1 = c_crit
        Case 2: S(m) = Θ(m log m)

Step 5: Convert back:
        m = log n
        T(n) = Θ(log n · log log n)
```

---

## Decision Guide

```
Given a recurrence T(n) = ..., which method to use?

Is it T(n) = aT(n/b) + f(n)?
├── YES → Is f(n) polynomially comparable to n^(log_b a)?
│         ├── YES → Use MASTER THEOREM
│         └── NO  → Use RECURSION TREE or SUBSTITUTION
│
├── Unequal splits (T(n/3) + T(2n/3))?
│   └── Use AKRA-BAZZI or RECURSION TREE
│
├── Subtractive (T(n-1), T(n-2))?
│   └── UNROLL the recurrence
│
├── Non-standard size (T(√n), T(log n))?
│   └── Use CHANGE OF VARIABLES
│
└── Other
    └── SUBSTITUTION METHOD (guess and verify)
```

---

## Summary Table

| Failure Type | Example | Why MT Fails | Alternative Method |
|-------------|---------|-------------|-------------------|
| Unequal splits | T(n/3) + T(2n/3) | Not aT(n/b) form | Akra-Bazzi |
| Variable a or b | n·T(n/2) | a not constant | Case-specific |
| Non-polynomial gap | n/log n vs n | Gap is logarithmic | Recursion tree |
| Regularity fails | Oscillating f(n) | Case 3 condition | Substitution |
| Subtractive | T(n-1) + n | Not n/b form | Unrolling |
| Non-standard size | T(√n) | √n ≠ n/b | Change of variables |

---

## Quick Revision Questions

1. **Give three reasons why the Master Theorem might not apply to a recurrence.**
2. **Solve T(n) = T(√n) + 1 using change of variables.**
3. **How does the Akra-Bazzi theorem handle T(n) = T(n/5) + T(7n/10) + n?**
4. **Solve the subtractive recurrence T(n) = 2T(n-1) + 1. Why is this exponential?**
5. **What is the non-polynomial gap, and why does it prevent the Master Theorem from working?**
6. **When would you choose the substitution method over the recursion tree method?**

---

[← Previous: Extended Master Theorem](05-extended-master-theorem.md) | [Next Unit: Binary Search →](../03-Binary-Search/01-binary-search-concept.md)
