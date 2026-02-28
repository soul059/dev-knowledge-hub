# Chapter 2: Ternary Search on Unimodal Functions

[← Previous: Concept](01-concept.md) | [Next: Comparison with Binary Search →](03-comparison.md)

---

## Overview

The primary use of ternary search is finding the **maximum or minimum** of a **unimodal function** — a function that increases then decreases (or vice versa).

---

## What Is a Unimodal Function?

```
   Unimodal (one peak):           Unimodal (one valley):
   
         ●                             ●               ●
       ●   ●                            ●           ●
     ●       ●                            ●       ●
   ●           ●                            ●   ●
                 ●                            ●
   
   f increases then decreases      f decreases then increases
   → find MAXIMUM                  → find MINIMUM
```

### Examples in Real Life

- Projectile trajectory (height vs time) — peak height
- Profit function (price vs revenue) — maximum profit
- Distance function — closest point to a target

---

## Algorithm: Find Maximum of Unimodal Function

```
   TERNARY-MAX(f, lo, hi):
       while hi - lo > EPSILON:      // or iterate fixed times
           mid1 = lo + (hi - lo) / 3
           mid2 = hi - (hi - lo) / 3
           
           if f(mid1) < f(mid2):
               lo = mid1        // peak is right of mid1
           else:
               hi = mid2        // peak is left of mid2
       
       return (lo + hi) / 2     // converged to peak
```

### Why This Works

```
   Case 1: f(mid1) < f(mid2)
   
        f(mid2) ●
   f(mid1) ●       ●
         ●           ●         Peak must be between mid1 and hi
   ──────┬──┬──────┬──┬──      (mid1 is on the ascending side)
       lo  m1     m2  hi
                                → eliminate [lo, mid1]
   
   Case 2: f(mid1) > f(mid2)
   
   f(mid1) ●
        ●     ● f(mid2)
                  ●             Peak must be between lo and mid2
   ──────┬──┬──────┬──┬──      (mid2 is on the descending side)
       lo  m1     m2  hi
                                → eliminate [mid2, hi]
   
   Case 3: f(mid1) == f(mid2)
   Peak is between mid1 and mid2 — can shrink from both sides.
```

---

## Integer Version: Find Peak in Discrete Array

```python
def findPeak(arr):
    lo, hi = 0, len(arr) - 1
    while hi - lo > 2:
        mid1 = lo + (hi - lo) // 3
        mid2 = hi - (hi - lo) // 3
        if arr[mid1] < arr[mid2]:
            lo = mid1 + 1
        else:
            hi = mid2 - 1
    # Linear search on remaining 1-3 elements
    peak = lo
    for i in range(lo, hi + 1):
        if arr[i] > arr[peak]:
            peak = i
    return peak
```

---

## Floating-Point Version

### Find Maximum

```python
def ternarySearchMax(f, lo, hi, iterations=200):
    for _ in range(iterations):
        mid1 = lo + (hi - lo) / 3
        mid2 = hi - (hi - lo) / 3
        if f(mid1) < f(mid2):
            lo = mid1
        else:
            hi = mid2
    return (lo + hi) / 2
```

### Find Minimum

```python
def ternarySearchMin(f, lo, hi, iterations=200):
    for _ in range(iterations):
        mid1 = lo + (hi - lo) / 3
        mid2 = hi - (hi - lo) / 3
        if f(mid1) > f(mid2):     # Reversed comparison!
            lo = mid1
        else:
            hi = mid2
    return (lo + hi) / 2
```

---

## Example: Maximum of f(x) = -x² + 4x + 5

```
   f(x) = -x² + 4x + 5
   
   f'(x) = -2x + 4 = 0 → x = 2 (calculus answer)
   f(2) = -4 + 8 + 5 = 9
   
   Ternary search on [-10, 10]:
   
   Iter 1: lo=-10 hi=10 mid1=-3.33 mid2=3.33
           f(-3.33)=-6.77 < f(3.33)=7.89 → lo=-3.33
   
   Iter 2: lo=-3.33 hi=10 mid1=1.11 mid2=5.56
           f(1.11)=8.21 > f(5.56)=-3.30 → hi=5.56
   
   Iter 3: lo=-3.33 hi=5.56 mid1=-0.37 mid2=2.60
           f(-0.37)=6.34 < f(2.60)=8.84 → lo=-0.37
   
   ... converges to x ≈ 2.0, f(x) ≈ 9.0 ✓
```

---

## Java Implementation

```java
public double ternaryMax(DoubleUnaryOperator f, double lo, double hi) {
    for (int i = 0; i < 200; i++) {
        double mid1 = lo + (hi - lo) / 3;
        double mid2 = hi - (hi - lo) / 3;
        if (f.applyAsDouble(mid1) < f.applyAsDouble(mid2)) {
            lo = mid1;
        } else {
            hi = mid2;
        }
    }
    return (lo + hi) / 2;
}

// Usage:
double peak = ternaryMax(x -> -x * x + 4 * x + 5, -10, 10);
// peak ≈ 2.0
```

---

## C++ Implementation

```cpp
double ternaryMax(function<double(double)> f, double lo, double hi) {
    for (int i = 0; i < 200; i++) {
        double mid1 = lo + (hi - lo) / 3;
        double mid2 = hi - (hi - lo) / 3;
        if (f(mid1) < f(mid2))
            lo = mid1;
        else
            hi = mid2;
    }
    return (lo + hi) / 2;
}

// Usage:
auto f = [](double x) { return -x*x + 4*x + 5; };
double peak = ternaryMax(f, -10, 10);
```

---

## Epsilon vs Fixed Iterations

```
   ┌────────────────────────────────────────────────────────────┐
   │  Epsilon-based:                                            │
   │    while (hi - lo > 1e-9)                                  │
   │    + Intuitive precision control                           │
   │    - Risk of infinite loop if precision > floating point   │
   │                                                            │
   │  Fixed iterations:                                         │
   │    for (int i = 0; i < 200; i++)                          │
   │    + Guaranteed termination                                │
   │    + 200 iters: range shrinks by (2/3)^200 ≈ 10^-35      │
   │    - Might over/under-iterate                              │
   │                                                            │
   │  Recommendation: Use 100-300 fixed iterations              │
   └────────────────────────────────────────────────────────────┘
```

---

## Convergence Rate

```
   Each iteration reduces range by factor 2/3:
   
   After k iterations: range × (2/3)^k
   
   For range [0, 10^9] to converge to 10^-9:
   10^9 × (2/3)^k < 10^-9
   (2/3)^k < 10^-18
   k × log(2/3) < -18 × log(10)
   k > 18 × 2.302 / 0.405 ≈ 102
   
   ~102 iterations for 18 digits of precision
   200 iterations is more than enough!
```

---

## Quick Revision Questions

1. **What is a unimodal function?**
2. **How does comparing f(mid1) and f(mid2) determine which third to eliminate?**
3. **Why can't binary search find the peak of a unimodal function directly?**
4. **What's the difference between finding max vs min using ternary search?**
5. **How many iterations are needed for 10^-9 precision on range [0, 10^9]?**
6. **Why are fixed iterations preferred over epsilon for floating-point ternary search?**

---

[← Previous: Concept](01-concept.md) | [Next: Comparison with Binary Search →](03-comparison.md)
