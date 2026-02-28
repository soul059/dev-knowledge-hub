# Chapter 2: Square Root and Power Problems

[← Previous: BS on Answer Concept](01-concept.md) | [Next: Capacity Problems →](03-capacity-problems.md)

---

## Overview

Square root and nth-root problems are the **simplest** examples of binary search on answer. They help build intuition before tackling harder problems.

---

## Problem 1: Integer Square Root

> Given a non-negative integer `x`, return `floor(sqrt(x))`.

```
   sqrt(8) = 2.828...  → floor = 2
   sqrt(16) = 4.0      → floor = 4
   sqrt(0) = 0         → floor = 0
```

### Search Space

```
   Answer:     0    1    2    3    4    5    ...
   mid² ≤ x?   T    T    T    F    F    F    (for x = 8)
                          ↑
                    last True = 2 = answer
   
   We want the LARGEST mid where mid * mid ≤ x
```

### Solution (Java)

```java
public int mySqrt(int x) {
    if (x < 2) return x;
    long lo = 1, hi = x / 2;   // sqrt(x) ≤ x/2 for x ≥ 4
    
    while (lo < hi) {
        long mid = lo + (hi - lo + 1) / 2;   // right-biased (lo = mid)
        if (mid * mid <= x) {
            lo = mid;           // mid works, try larger
        } else {
            hi = mid - 1;       // mid too big
        }
    }
    return (int) lo;
}
```

### Solution (Python)

```python
def mySqrt(x):
    if x < 2:
        return x
    lo, hi = 1, x // 2
    while lo < hi:
        mid = lo + (hi - lo + 1) // 2    # right-biased
        if mid * mid <= x:
            lo = mid
        else:
            hi = mid - 1
    return lo
```

### Trace

```
   x = 17
   lo=1 hi=8
   
   mid=5: 25 > 17 → hi=4
   mid=3: 9 ≤ 17  → lo=3
   mid=4: 16 ≤ 17 → lo=4
   lo=4 hi=4 → return 4 ✓  (√17 = 4.12...)
```

---

## Problem 2: Valid Perfect Square

> Given a positive integer `num`, return `true` if it's a perfect square.

### Solution

```python
def isPerfectSquare(num):
    lo, hi = 1, num
    while lo <= hi:
        mid = lo + (hi - lo) // 2
        sq = mid * mid
        if sq == num:
            return True
        elif sq < num:
            lo = mid + 1
        else:
            hi = mid - 1
    return False
```

---

## Problem 3: Nth Root of M

> Given `n` and `m`, find `floor(m^(1/n))`.

```
   n=3, m=27 → cube_root(27) = 3
   n=3, m=30 → cube_root(30) = 3.107... → floor = 3
   n=4, m=69 → fourth_root(69) = 2.88... → floor = 2
```

### Solution (Java)

```java
public int nthRoot(int n, int m) {
    int lo = 1, hi = (int) Math.pow(m, 1.0 / n) + 2;
    
    while (lo < hi) {
        int mid = lo + (hi - lo + 1) / 2;
        long power = 1;
        boolean overflow = false;
        
        for (int i = 0; i < n; i++) {
            power *= mid;
            if (power > m) { overflow = true; break; }
        }
        
        if (!overflow && power <= m) {
            lo = mid;
        } else {
            hi = mid - 1;
        }
    }
    return lo;
}
```

**Overflow protection:** Instead of computing `mid^n` which can overflow, check during multiplication.

---

## Problem 4: Floating Point Square Root

> Find `sqrt(x)` to a given precision.

```python
def sqrtFloat(x, precision=1e-7):
    if x < 0:
        return -1   # undefined for negative
    
    lo, hi = 0, max(1, x)   # for x < 1, sqrt(x) > x
    
    while hi - lo > precision:
        mid = (lo + hi) / 2
        if mid * mid <= x:
            lo = mid
        else:
            hi = mid
    
    return lo
```

```
   sqrtFloat(2):
   
   lo=0     hi=2     mid=1.0    1.0 ≤ 2  → lo=1.0
   lo=1.0   hi=2     mid=1.5    2.25 > 2 → hi=1.5
   lo=1.0   hi=1.5   mid=1.25   1.5625≤2 → lo=1.25
   lo=1.25  hi=1.5   mid=1.375  1.89≤2   → lo=1.375
   lo=1.375 hi=1.5   mid=1.4375 2.07>2   → hi=1.4375
   ...converges to 1.4142135... ✓
```

### Key Differences for Floating Point

```
   ┌────────────────────────────────────────────────────────────┐
   │ Integer BS:                                                │
   │   - Loop: lo < hi or lo <= hi                             │
   │   - Terminates: lo == hi or lo > hi                       │
   │   - Updates: mid ± 1                                      │
   │                                                            │
   │ Floating Point BS:                                         │
   │   - Loop: hi - lo > epsilon                               │
   │   - Terminates: when range is small enough                │
   │   - Updates: lo = mid or hi = mid (no ±1!)               │
   │   - Alternative: fixed iteration count (e.g., 100 times) │
   └────────────────────────────────────────────────────────────┘
```

---

## Problem 5: Power(x, n)

> Compute `x^n` using binary search / fast exponentiation.

This isn't exactly BS-on-answer but uses the **same halving principle**.

```java
public double myPow(double x, int n) {
    long N = n;
    if (N < 0) {
        x = 1 / x;
        N = -N;
    }
    
    double result = 1.0;
    while (N > 0) {
        if (N % 2 == 1) {
            result *= x;
        }
        x *= x;
        N /= 2;
    }
    return result;
}
```

```
   2^10:
   10 → even: x = 2*2 = 4,    N = 5
   5  → odd:  result = 1*4=4,  x = 4*4 = 16,  N = 2
   2  → even: x = 16*16 = 256, N = 1
   1  → odd:  result = 4*256 = 1024, N = 0
   
   Answer: 1024 ✓
```

---

## Complexity Comparison

| Problem | Time | Space |
|---------|------|-------|
| Integer sqrt | O(log x) | O(1) |
| Perfect square | O(log n) | O(1) |
| Nth root | O(n × log m) | O(1) |
| Float sqrt | O(log((hi-lo)/ε)) | O(1) |
| Power(x,n) | O(log n) | O(1) |

---

## Summary Pattern

```
   All square root / power problems follow the same pattern:
   
   1. Define range: [0, x] or [1, x/2]
   2. Check: mid^k ≤ x?
   3. If seeking LARGEST valid: right-biased, lo = mid
   4. If seeking SMALLEST valid: left-biased, hi = mid
   5. Watch for overflow in mid^k computation
```

---

## Quick Revision Questions

1. **Why do we use `x/2` as the upper bound for sqrt(x)?**
2. **Why is right-biased mid needed for "find largest" problems?**
3. **How does floating-point BS differ from integer BS in termination?**
4. **How do you avoid overflow when computing `mid^n`?**
5. **What is the time complexity of integer square root?**
6. **Trace `mySqrt(26)` step by step.**

---

[← Previous: BS on Answer Concept](01-concept.md) | [Next: Capacity Problems →](03-capacity-problems.md)
