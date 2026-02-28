# Chapter 6.4: Boole's Rule and Higher Newton-Cotes

[← Previous: Simpson's 3/8 Rule](03-simpsons-three-eighth.md) | [Next: Weddle's Rule →](05-weddles-rule.md)

---

## Overview

**Boole's Rule** (also spelled Bode's Rule) uses 5 points (4 subintervals) and fits a degree-4 polynomial. Along with higher Newton-Cotes formulas, it provides $O(h^6)$ accuracy but is rarely used in practice due to negative weights appearing at higher orders.

---

## 1. Boole's Rule

$$\boxed{\int_{x_0}^{x_4} f(x)\,dx \approx \frac{2h}{45}[7f_0 + 32f_1 + 12f_2 + 32f_3 + 7f_4]}$$

- Uses 5 points, 4 subintervals
- Weights: $7, 32, 12, 32, 7$
- Error: $-\frac{8h^7}{945}f^{(6)}(\xi) = O(h^6)$
- Exact for polynomials of degree $\leq 5$

---

## 2. Newton-Cotes Family Summary

| Rule | Points | Degree | Weights | Error Order |
|------|--------|--------|---------|-------------|
| Trapezoidal | 2 | 1 | $1, 1$ | $O(h^2)$ |
| Simpson's 1/3 | 3 | 2 | $1, 4, 1$ | $O(h^4)$ |
| Simpson's 3/8 | 4 | 3 | $1, 3, 3, 1$ | $O(h^4)$ |
| **Boole's** | **5** | **4** | **$7, 32, 12, 32, 7$** | **$O(h^6)$** |
| 6-point | 6 | 5 | $19, 75, 50, 50, 75, 19$ | $O(h^6)$ |
| Weddle's | 7 | 6 | $1, 5, 1, 6, 1, 5, 1$ (simplified) | $O(h^8)$ |

---

## 3. Common Multipliers

```
  Rule         Multiplier    Weights
  ─────────────────────────────────────────
  Trapezoidal   h/2         1, 1
  Simpson 1/3   h/3         1, 4, 1
  Simpson 3/8   3h/8        1, 3, 3, 1
  Boole's       2h/45       7, 32, 12, 32, 7
```

---

## 4. Worked Example (Boole's Rule)

Evaluate $\int_0^4 e^x \, dx$ with $h = 1$:

| $x$ | 0 | 1 | 2 | 3 | 4 |
|-----|---|---|---|---|---|
| $f$ | 1.000 | 2.718 | 7.389 | 20.086 | 54.598 |

$$I \approx \frac{2(1)}{45}[7(1.000) + 32(2.718) + 12(7.389) + 32(20.086) + 7(54.598)]$$

$$= \frac{2}{45}[7.000 + 86.976 + 88.668 + 642.752 + 382.186]$$

$$= \frac{2}{45} \times 1207.582 = \frac{2415.164}{45} = 53.670$$

**Exact:** $e^4 - 1 = 53.598$. Error: $0.072$.

---

## 5. The Problem with Higher-Order Closed Newton-Cotes

For $n \geq 8$ points, some weights become **negative**, which causes:
- **Cancellation errors** (subtracting large nearly-equal numbers)
- **Instability** for oscillatory functions
- Divergence as $n \to \infty$ for some smooth functions

> **This is why Gaussian quadrature (open-node) methods are preferred for high accuracy over Newton-Cotes!**

---

## 6. Open vs Closed Newton-Cotes

| Type | Endpoints | Example |
|------|-----------|---------|
| **Closed** | Function evaluated at $a$ and $b$ | Trapezoidal, Simpson |
| **Open** | Function NOT evaluated at $a$, $b$ | Midpoint rule |

### Midpoint Rule (simplest open)

$$\int_a^b f(x)\,dx \approx (b-a) \cdot f\left(\frac{a+b}{2}\right), \quad E = \frac{h^3}{24}f''(\xi)$$

Surprisingly: error constant $1/24$ vs trapezoidal's $1/12$ — midpoint is **twice** as accurate!

---

## Summary Table

| Property | Value |
|----------|-------|
| **Boole's formula** | $\frac{2h}{45}[7f_0 + 32f_1 + 12f_2 + 32f_3 + 7f_4]$ |
| **Error** | $O(h^6)$, exact for degree $\leq 5$ |
| **Subintervals** | 4 per application |
| **Higher NC issue** | Negative weights for $n \geq 8$ |
| **Alternative** | Gaussian quadrature for high accuracy |

---

## Quick Revision Questions

1. **State Boole's rule with its weights and multiplier.**

2. **For which degree polynomials is Boole's rule exact?**

3. **Why are higher-order Newton-Cotes formulas ($n \geq 8$) rarely used in practice?**

4. **Compare the accuracy of the midpoint rule and the trapezoidal rule.**

5. **List the Newton-Cotes formulas from 2 to 7 points with their error orders.**

6. **What is the difference between open and closed Newton-Cotes formulas?**

---

[← Previous: Simpson's 3/8 Rule](03-simpsons-three-eighth.md) | [Next: Weddle's Rule →](05-weddles-rule.md)
