# Chapter 1: Factorial

## Overview
Factorial is the **"Hello World" of recursion** — the simplest and most classic example. Understanding factorial thoroughly builds the foundation for all recursive thinking.

---

## Mathematical Definition

```
n! = n × (n-1) × (n-2) × ... × 2 × 1

Examples:
  0! = 1  (by definition)
  1! = 1
  3! = 3 × 2 × 1 = 6
  5! = 5 × 4 × 3 × 2 × 1 = 120

Recursive definition:
  n! = n × (n-1)!     for n > 0
  0! = 1               base case
```

---

## Recursive Solution

### Pseudocode
```
function factorial(n):
    if n <= 1:              // Base case
        return 1
    return n * factorial(n - 1)   // Recursive case
```

### Applying the 5-Step Framework

```
Step 1: DEFINE   factorial(n) returns n!
Step 2: BASE     factorial(0) = 1, factorial(1) = 1
Step 3: TRUST    factorial(n-1) correctly computes (n-1)!
Step 4: BUILD    factorial(n) = n * factorial(n-1)
Step 5: VERIFY   (see trace below)
```

---

## Step-by-Step Trace: factorial(5)

```
factorial(5)
│  n=5, not base case
│  return 5 * factorial(4)
│                │
│                factorial(4)
│                │  n=4, not base case
│                │  return 4 * factorial(3)
│                │                │
│                │                factorial(3)
│                │                │  n=3, not base case
│                │                │  return 3 * factorial(2)
│                │                │                │
│                │                │                factorial(2)
│                │                │                │  n=2, not base case
│                │                │                │  return 2 * factorial(1)
│                │                │                │                │
│                │                │                │                factorial(1)
│                │                │                │                │  n=1, BASE CASE
│                │                │                │                │  return 1
│                │                │                │                │
│                │                │                │  return 2 * 1 = 2
│                │                │                │
│                │                │  return 3 * 2 = 6
│                │                │
│                │  return 4 * 6 = 24
│                │
│  return 5 * 24 = 120
```

### Call Stack at Maximum Depth

```
┌─────────────────┐
│ factorial(1)    │ ← TOP (Base case, returns 1)
│ n=1, return 1   │
├─────────────────┤
│ factorial(2)    │
│ n=2, waiting    │
├─────────────────┤
│ factorial(3)    │
│ n=3, waiting    │
├─────────────────┤
│ factorial(4)    │
│ n=4, waiting    │
├─────────────────┤
│ factorial(5)    │ ← BOTTOM
│ n=5, waiting    │
└─────────────────┘
  Max depth = 5 frames
```

---

## Iterative Solution (Comparison)

```
function factorial_iterative(n):
    result = 1
    for i = 2 to n:
        result = result * i
    return result
```

```
Trace for n=5:
  i=2: result = 1 * 2 = 2
  i=3: result = 2 * 3 = 6
  i=4: result = 6 * 4 = 24
  i=5: result = 24 * 5 = 120
  return 120
```

---

## Tail-Recursive Version

```
function factorial_tail(n, accumulator=1):
    if n <= 1:
        return accumulator
    return factorial_tail(n - 1, n * accumulator)
```

```
Trace for n=5:
  factorial_tail(5, 1)
  factorial_tail(4, 5)      // 5 * 1 = 5
  factorial_tail(3, 20)     // 4 * 5 = 20
  factorial_tail(2, 60)     // 3 * 20 = 60
  factorial_tail(1, 120)    // 2 * 60 = 120
  return 120                 // Base case

[*] Computation happens on the way DOWN, not on the way back up!
```

---

## Complexity Analysis

```
┌──────────────────────────────────────────────┐
│           COMPLEXITY ANALYSIS                │
│                                              │
│  Recursive:                                  │
│    Time:  O(n) — n multiplications           │
│    Space: O(n) — n stack frames              │
│                                              │
│  Iterative:                                  │
│    Time:  O(n) — n multiplications           │
│    Space: O(1) — just one variable           │
│                                              │
│  Tail-recursive (with TCO):                  │
│    Time:  O(n)                               │
│    Space: O(1) — compiler optimizes          │
│                                              │
│  Recurrence: T(n) = T(n-1) + O(1)           │
│              T(n) = O(n)                     │
└──────────────────────────────────────────────┘
```

---

## Edge Cases

| Input | Output | Note |
|-------|--------|------|
| 0 | 1 | By mathematical convention |
| 1 | 1 | Simplest case |
| Negative | Undefined | Add guard: if n < 0, error |
| Large n (>20) | Overflow | Use BigInteger/long |

---

## Real-World Applications

- **Combinatorics**: nCr = n! / (r! × (n-r)!)
- **Probability**: Arrangements and permutations
- **Taylor series**: sin(x) = x - x³/3! + x⁵/5! - ...
- **Algorithm analysis**: Permutation complexity = O(n!)

---

## Summary Table

| Aspect | Detail |
|--------|--------|
| Problem | Compute n! |
| Base case | n ≤ 1 → return 1 |
| Recursive case | n × factorial(n-1) |
| Time complexity | O(n) |
| Space complexity | O(n) recursive, O(1) iterative |
| Recurrence | T(n) = T(n-1) + O(1) |
| Key insight | Classic example of linear recursion |

---

## Quick Revision Questions

1. **What is the base case** for factorial and why is 0! = 1?
2. **Write the recurrence relation** for factorial.
3. **What is the space complexity** difference between recursive and iterative factorial?
4. **Convert factorial to tail-recursive** form. What's the advantage?
5. **What happens** if we call factorial(-1) with no guard?
6. **How many stack frames** exist at peak for factorial(10)?

---

[← Previous: Thinking Recursively](../01-Recursion-Fundamentals/06-thinking-recursively.md) | [Next: Fibonacci →](02-fibonacci.md)

[← Back to README](../README.md)
