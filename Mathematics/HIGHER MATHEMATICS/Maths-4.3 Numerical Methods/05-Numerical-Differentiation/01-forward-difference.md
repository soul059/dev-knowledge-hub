# Chapter 5.1: Forward Difference Differentiation

[← Previous: Spline Interpolation](../04-Interpolation/07-spline-interpolation.md) | [Next: Backward Difference Differentiation →](02-backward-difference.md)

---

## Overview

**Numerical differentiation** approximates derivatives of a function using tabulated values, without requiring an explicit formula for $f(x)$. Forward difference formulas use data points **ahead** of the point of interest and are best suited near the **beginning** of a data table.

---

## 1. Derivation from Newton's Forward Formula

Starting with Newton's forward interpolation formula ($p = \frac{x - x_0}{h}$):

$$f(x) = f_0 + p \Delta f_0 + \frac{p(p-1)}{2!} \Delta^2 f_0 + \frac{p(p-1)(p-2)}{3!} \Delta^3 f_0 + \cdots$$

Differentiating with respect to $x$ (using $\frac{dp}{dx} = \frac{1}{h}$):

$$\boxed{f'(x) = \frac{1}{h}\left[\Delta f_0 + \frac{2p-1}{2}\Delta^2 f_0 + \frac{3p^2 - 6p + 2}{6}\Delta^3 f_0 + \cdots\right]}$$

---

## 2. Derivatives at $x = x_0$ (i.e., $p = 0$)

### First Derivative

$$\boxed{f'(x_0) = \frac{1}{h}\left[\Delta f_0 - \frac{1}{2}\Delta^2 f_0 + \frac{1}{3}\Delta^3 f_0 - \frac{1}{4}\Delta^4 f_0 + \cdots\right]}$$

### Second Derivative

Differentiating again:

$$\boxed{f''(x_0) = \frac{1}{h^2}\left[\Delta^2 f_0 - \Delta^3 f_0 + \frac{11}{12}\Delta^4 f_0 - \cdots\right]}$$

### Third Derivative

$$f'''(x_0) = \frac{1}{h^3}\left[\Delta^3 f_0 - \frac{3}{2}\Delta^4 f_0 + \cdots\right]$$

---

## 3. Simple Forward Difference Approximations

| Derivative | Formula | Error |
|-----------|---------|-------|
| $f'(x_0)$ | $\frac{f_1 - f_0}{h}$ | $O(h)$ |
| $f'(x_0)$ | $\frac{-3f_0 + 4f_1 - f_2}{2h}$ | $O(h^2)$ |
| $f''(x_0)$ | $\frac{f_2 - 2f_1 + f_0}{h^2}$ | $O(h)$ |

---

## 4. Worked Example

Given:

| $x$ | 1.0 | 1.2 | 1.4 | 1.6 | 1.8 |
|-----|-----|-----|-----|-----|-----|
| $f(x)$ | 0.0000 | 0.1823 | 0.3365 | 0.4700 | 0.5878 |

Find $f'(1.0)$ and $f''(1.0)$ using forward differences. ($h = 0.2$)

### Step 1: Build the Forward Difference Table

| $x$ | $f$ | $\Delta f$ | $\Delta^2 f$ | $\Delta^3 f$ | $\Delta^4 f$ |
|-----|-----|-----------|--------------|--------------|--------------|
| 1.0 | 0.0000 | | | | |
| | | 0.1823 | | | |
| 1.2 | 0.1823 | | −0.0281 | | |
| | | 0.1542 | | 0.0054 | |
| 1.4 | 0.3365 | | −0.0227 | | −0.0007 |
| | | 0.1335 | | 0.0047 | |
| 1.6 | 0.4700 | | −0.0180 | | |
| | | 0.1178 | | | |
| 1.8 | 0.5878 | | | | |

### Step 2: Apply the Formulas

**First derivative at $x_0 = 1.0$:**

$$f'(1.0) = \frac{1}{0.2}\left[0.1823 - \frac{0.0281}{2} + \frac{0.0054}{3} - \frac{0.0007}{4}\right]$$

$$= 5\left[0.1823 - 0.01405 + 0.0018 - 0.000175\right]$$

$$= 5 \times 0.169875 = 0.8494$$

**Second derivative at $x_0 = 1.0$:**

$$f''(1.0) = \frac{1}{0.04}\left[-0.0281 - (-0.0054) + \frac{11}{12}(-0.0007)\right]$$

$$= 25\left[-0.0281 + 0.0054 - 0.000642\right]$$

$$= 25 \times (-0.02334) = -0.5835$$

---

## 5. Truncation vs Roundoff Error Trade-off

```
  Error
    │
    │ ╲  Truncation error
    │  ╲   (decreases as h → 0)
    │   ╲
    │    ╲        ╱ Roundoff error
    │     ╲      ╱   (increases as h → 0)
    │      ╲    ╱
    │       ╲  ╱
    │        ●    ← Optimal h
    │       ╱ ╲
    │      ╱   ╲
    │               Total error
    └────────────────────────── h
          h_opt
```

**Optimal step size** for first derivative: $h_{\text{opt}} \approx 2\sqrt{\epsilon_{\text{machine}}}$

---

## Summary Table

| Item | Value |
|------|-------|
| **Best for** | Differentiating near the beginning of a table |
| **Input** | Forward difference table ($\Delta, \Delta^2, \ldots$) |
| **$f'(x_0)$ formula** | $\frac{1}{h}[\Delta f_0 - \frac{1}{2}\Delta^2 f_0 + \frac{1}{3}\Delta^3 f_0 - \cdots]$ |
| **$f''(x_0)$ formula** | $\frac{1}{h^2}[\Delta^2 f_0 - \Delta^3 f_0 + \frac{11}{12}\Delta^4 f_0 - \cdots]$ |
| **Order of error** | $O(h)$ for simple 2-point, $O(h^2)$ for 3-point |

---

## Quick Revision Questions

1. **Derive $f'(x_0)$ using Newton's forward difference formula.**

2. **Find $f'(0)$ and $f''(0)$ from: $(0,1), (0.1,1.1052), (0.2,1.2214), (0.3,1.3499)$.**

3. **Why does forward difference differentiation work best near the start of a table?**

4. **Explain the trade-off between truncation error and roundoff error in numerical differentiation.**

5. **Write the $O(h^2)$ forward difference formula for $f'(x_0)$ using three points.**

6. **What is the optimal step size for numerical differentiation and how does it depend on machine precision?**

---

[← Previous: Spline Interpolation](../04-Interpolation/07-spline-interpolation.md) | [Next: Backward Difference Differentiation →](02-backward-difference.md)
