# Chapter 4.3: Newton's Backward Difference Interpolation

[← Previous: Newton's Forward Difference](02-newton-forward.md) | [Next: Lagrange Interpolation →](04-lagrange-interpolation.md)

---

## Overview

Newton's Backward Difference formula is the mirror of the forward formula. It uses **backward differences** and is most accurate when interpolating near the **end** of the data table.

---

## 1. The Formula

Let $x = x_n + sh$ where $s = \frac{x - x_n}{h}$ (note: $s$ is typically negative or small positive):

$$\boxed{f(x) = f_n + s\nabla f_n + \frac{s(s+1)}{2!}\nabla^2 f_n + \frac{s(s+1)(s+2)}{3!}\nabla^3 f_n + \cdots}$$

---

## 2. When to Use

```
  Data:  x₀    x₁    x₂    x₃    x₄    x₅
         ●─────●─────●─────●─────●─────●
                                        ↑
                                  Best accuracy
                                  for backward
                                  interpolation

  Forward: best near x₀ (beginning)
  Backward: best near xₙ (end)
```

---

## 3. Worked Example

Given:

| $x$ | 1 | 2 | 3 | 4 | 5 |
|:---:|:-:|:-:|:-:|:-:|:-:|
| $f(x)$ | 1 | 8 | 27 | 64 | 125 |

Find $f(4.5)$.

### Backward difference table (built from the bottom)

| $x$ | $f$ | $\nabla f$ | $\nabla^2 f$ | $\nabla^3 f$ | $\nabla^4 f$ |
|:---:|:---:|:----------:|:-------------:|:-------------:|:-------------:|
| 1 | 1 | | | | |
| 2 | 8 | 7 | | | |
| 3 | 27 | 19 | 12 | | |
| 4 | 64 | 37 | 18 | 6 | |
| 5 | 125 | 61 | 24 | 6 | 0 |

Here $x_n = 5$, $h = 1$, $s = (4.5 - 5)/1 = -0.5$

$$f(4.5) = 125 + (-0.5)(61) + \frac{(-0.5)(0.5)}{2}(24) + \frac{(-0.5)(0.5)(1.5)}{6}(6) + 0$$

$$= 125 - 30.5 - 3.0 - 0.375 = 91.125$$

**Check**: $f(x) = x^3$, so $f(4.5) = 91.125$ ✓

---

## 4. Forward vs. Backward: When to Choose

| Factor | Forward | Backward |
|--------|:-------:|:--------:|
| Interpolation near | Beginning ($x_0$) | End ($x_n$) |
| Uses differences from | Top of table | Bottom of table |
| $s$ parameter | $(x - x_0)/h$ (usually $0 < s < 1$) | $(x - x_n)/h$ (usually $-1 < s < 0$) |
| Extrapolation | Right side | Left side |

---

## 5. Derivation

Using $E^{-1} = 1 - \nabla$:

$$f(x_n + sh) = E^s f_n = (1-\nabla)^{-s} f_n$$

Expanding:

$$(1 - \nabla)^{-s} = \sum_{k=0}^{n} \binom{-s}{k}(-\nabla)^k = \sum_{k=0}^{n} \frac{s(s+1)\cdots(s+k-1)}{k!}\nabla^k$$

---

## Summary Table

| Property | Value |
|----------|-------|
| **Formula** | $f(x) = \sum_{k=0}^{n} \binom{-s}{k}(-1)^k \nabla^k f_n$ |
| **$s$ parameter** | $s = (x - x_n)/h$ |
| **Best for** | Interpolation near end of table |
| **Requires** | Equally spaced data |
| **Uses** | Backward differences from bottom |

---

## Quick Revision Questions

1. **State Newton's backward difference formula and explain when it is preferred.**

2. **Using the data $f(1) = 1, f(2) = 4, f(3) = 9, f(4) = 16, f(5) = 25$, find $f(4.8)$ using backward interpolation.**

3. **Show the relationship between forward and backward differences.**

4. **Compare error behavior of forward and backward formulas for interpolation near the middle of a table.**

5. **Can backward interpolation be used for extrapolation beyond $x_n$? Explain.**

6. **Derive the backward formula from the operator $E^{-1} = 1 - \nabla$.**

---

[← Previous: Newton's Forward Difference](02-newton-forward.md) | [Next: Lagrange Interpolation →](04-lagrange-interpolation.md)
