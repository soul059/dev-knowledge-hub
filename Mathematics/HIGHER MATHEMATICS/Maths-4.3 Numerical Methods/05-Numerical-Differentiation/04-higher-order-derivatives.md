# Chapter 5.4: Higher Order Derivatives

[← Previous: Central Difference](03-central-difference.md) | [Next: Error Estimation →](05-error-estimation.md)

---

## Overview

Higher order numerical derivatives ($f'', f''', f^{(4)}, \ldots$) are needed in beam deflection, vibration analysis, and solving PDEs. These formulas generalise the techniques from previous chapters.

---

## 1. Second Derivative Formulas

### Central (most common)

$$\boxed{f''(x_0) = \frac{f_{-1} - 2f_0 + f_1}{h^2} + O(h^2)}$$

### Forward

$$f''(x_0) = \frac{f_0 - 2f_1 + f_2}{h^2} + O(h)$$

### Backward

$$f''(x_0) = \frac{f_{-2} - 2f_{-1} + f_0}{h^2} + O(h)$$

### Higher accuracy central (5-point)

$$f''(x_0) = \frac{-f_{-2} + 16f_{-1} - 30f_0 + 16f_1 - f_2}{12h^2} + O(h^4)$$

---

## 2. Third Derivative Formulas

### Central (4-point)

$$f'''(x_0) = \frac{f_2 - 2f_1 + 2f_{-1} - f_{-2}}{2h^3} + O(h^2)$$

### Forward

$$f'''(x_0) = \frac{-f_0 + 3f_1 - 3f_2 + f_3}{h^3} + O(h)$$

### Backward

$$f'''(x_0) = \frac{f_0 - 3f_{-1} + 3f_{-2} - f_{-3}}{h^3} + O(h)$$

---

## 3. Fourth Derivative

### Central (5-point)

$$\boxed{f^{(4)}(x_0) = \frac{f_{-2} - 4f_{-1} + 6f_0 - 4f_1 + f_2}{h^4} + O(h^2)}$$

### Forward

$$f^{(4)}(x_0) = \frac{f_0 - 4f_1 + 6f_2 - 4f_3 + f_4}{h^4} + O(h)$$

---

## 4. Pattern — Binomial Coefficients

The coefficients follow the binomial pattern with alternating signs:

```
  f'   :    ─1    +1                    (2 points)
  f''  :    +1    ─2    +1              (3 points; ÷h²)
  f''' :    ─1    +3    ─3    +1        (4 points; ÷h³)
  f⁽⁴⁾:    +1    ─4    +6    ─4    +1  (5 points; ÷h⁴)

  Coefficients of: (−1)ⁿ × Row n of Pascal's triangle / hⁿ
```

General $n$-th order **forward** difference:

$$f^{(n)}(x_0) \approx \frac{1}{h^n} \sum_{k=0}^{n} (-1)^k \binom{n}{k} f_{n-k}$$

---

## 5. Worked Example

Given:

| $x$ | 0 | 1 | 2 | 3 | 4 |
|-----|---|---|---|---|---|
| $f(x)$ | 1 | 0 | 1 | 8 | 27 |

Find $f''(2)$ and $f^{(4)}(2)$. ($h = 1$)

**Second derivative (central):**

$$f''(2) = \frac{f(1) - 2f(2) + f(3)}{1^2} = \frac{0 - 2 + 8}{1} = 6$$

**Fourth derivative (central):**

$$f^{(4)}(2) = \frac{f(0) - 4f(1) + 6f(2) - 4f(3) + f(4)}{1^4}$$

$$= \frac{1 - 0 + 6 - 32 + 27}{1} = 2$$

---

## 6. Using the Difference Table

For the $n$-th derivative using forward differences at the beginning of a table:

$$f^{(n)}(x_0) = \frac{\Delta^n f_0}{h^n} + O(h)$$

More accurate:

$$f^{(n)}(x_0) = \frac{1}{h^n}\left[\Delta^n f_0 - \frac{n}{2}\Delta^{n+1} f_0 + \cdots\right]$$

For central differences at an interior point:

$$f^{(n)}(x_0) = \frac{\delta^n f_0}{h^n} + O(h^2)$$

---

## 7. Maximum Derivative vs Data Points

| Data Points | Maximum Derivative |
|-------------|-------------------|
| 2 | $f'$ |
| 3 | $f''$ |
| 4 | $f'''$ |
| 5 | $f^{(4)}$ |
| $n+1$ | $f^{(n)}$ |

> **Rule**: To compute the $n$-th derivative, you need at least $n+1$ data points.

---

## 8. Error Amplification

```
  Derivative    Points    Roundoff amplification
  Order         Needed    factor (worst case)
  ──────────    ──────    ──────────────────────
  f'            2          2ε/h
  f''           3          4ε/h²
  f'''          4          8ε/h³
  f⁽⁴⁾         5          16ε/h⁴

  Higher derivatives amplify roundoff MUCH more severely!
```

---

## Summary Table

| Derivative | Central Formula | Error | Min Points |
|-----------|----------------|-------|------------|
| $f'$ | $(f_1 - f_{-1})/(2h)$ | $O(h^2)$ | 3 |
| $f''$ | $(f_{-1} - 2f_0 + f_1)/h^2$ | $O(h^2)$ | 3 |
| $f'''$ | $(f_2 - 2f_1 + 2f_{-1} - f_{-2})/(2h^3)$ | $O(h^2)$ | 5 |
| $f^{(4)}$ | $(f_{-2} - 4f_{-1} + 6f_0 - 4f_1 + f_2)/h^4$ | $O(h^2)$ | 5 |

---

## Quick Revision Questions

1. **Write the central difference formula for $f''(x_0)$ and $f^{(4)}(x_0)$.**

2. **How do the coefficients of finite difference formulas relate to Pascal's triangle?**

3. **Given $(0,1), (1,4), (2,15), (3,40), (4,85)$, find $f''(2)$ and $f'''(2)$.**

4. **Why do higher-order numerical derivatives suffer more from roundoff error?**

5. **How many data points are needed to compute the third derivative?**

6. **Write the general formula $f^{(n)}(x_0) \approx \frac{1}{h^n}\sum_{k=0}^{n}(-1)^k \binom{n}{k}f_{n-k}$. Which type of difference is this?**

---

[← Previous: Central Difference](03-central-difference.md) | [Next: Error Estimation →](05-error-estimation.md)
