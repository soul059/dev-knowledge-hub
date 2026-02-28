# Chapter 5.2: Backward Difference Differentiation

[← Previous: Forward Difference](01-forward-difference.md) | [Next: Central Difference →](03-central-difference.md)

---

## Overview

**Backward difference differentiation** uses data points **behind** the point of interest. These formulas are best suited near the **end** of a data table, complementing the forward difference approach used at the beginning.

---

## 1. Derivation from Newton's Backward Formula

Starting with Newton's backward interpolation ($p = \frac{x - x_n}{h}$):

$$f(x) = f_n + p\nabla f_n + \frac{p(p+1)}{2!}\nabla^2 f_n + \frac{p(p+1)(p+2)}{3!}\nabla^3 f_n + \cdots$$

Differentiating with respect to $x$:

$$\boxed{f'(x) = \frac{1}{h}\left[\nabla f_n + \frac{2p+1}{2}\nabla^2 f_n + \frac{3p^2+6p+2}{6}\nabla^3 f_n + \cdots\right]}$$

---

## 2. Derivatives at $x = x_n$ (i.e., $p = 0$)

### First Derivative

$$\boxed{f'(x_n) = \frac{1}{h}\left[\nabla f_n + \frac{1}{2}\nabla^2 f_n + \frac{1}{3}\nabla^3 f_n + \frac{1}{4}\nabla^4 f_n + \cdots\right]}$$

### Second Derivative

$$\boxed{f''(x_n) = \frac{1}{h^2}\left[\nabla^2 f_n + \nabla^3 f_n + \frac{11}{12}\nabla^4 f_n + \cdots\right]}$$

### Third Derivative

$$f'''(x_n) = \frac{1}{h^3}\left[\nabla^3 f_n + \frac{3}{2}\nabla^4 f_n + \cdots\right]$$

---

## 3. Comparison of Coefficient Signs

```
  Forward:  f'(x₀) = 1/h [Δf₀ - ½Δ²f₀ + ⅓Δ³f₀ - ¼Δ⁴f₀ + ...]
                           +     -       +       -         (alternating)

  Backward: f'(xₙ) = 1/h [∇fₙ + ½∇²fₙ + ⅓∇³fₙ + ¼∇⁴fₙ + ...]
                           +     +       +       +         (all positive)
```

Note: The **magnitudes** of the coefficients are the same; only the **signs** differ!

---

## 4. Worked Example

Given:

| $x$ | 1.0 | 1.5 | 2.0 | 2.5 | 3.0 |
|-----|-----|-----|-----|-----|-----|
| $f(x)$ | 0.000 | 0.405 | 0.693 | 0.916 | 1.099 |

Find $f'(3.0)$ and $f''(3.0)$. ($h = 0.5$)

### Step 1: Build the Backward Difference Table

| $x$ | $f$ | $\nabla f$ | $\nabla^2 f$ | $\nabla^3 f$ | $\nabla^4 f$ |
|-----|------|-----------|--------------|--------------|--------------|
| 1.0 | 0.000 | | | | |
| | | 0.405 | | | |
| 1.5 | 0.405 | | −0.117 | | |
| | | 0.288 | | 0.046 | |
| 2.0 | 0.693 | | −0.071 | | −0.022 |
| | | 0.223 | | 0.024 | |
| 2.5 | 0.916 | | −0.040 | | |
| | | 0.183 | | | |
| 3.0 | 1.099 | | | | |

At $x_n = 3.0$:
- $\nabla f_n = 0.183$
- $\nabla^2 f_n = -0.040$ (wait — computed wrong direction; let me recompute)

The backward differences at $x_n = 3.0$:

$\nabla f_4 = f_4 - f_3 = 1.099 - 0.916 = 0.183$

$\nabla^2 f_4 = \nabla f_4 - \nabla f_3 = 0.183 - 0.223 = -0.040$

$\nabla^3 f_4 = \nabla^2 f_4 - \nabla^2 f_3 = -0.040 - (-0.071) = 0.031$

$\nabla^4 f_4 = \nabla^3 f_4 - \nabla^3 f_3 = 0.031 - 0.046 = -0.015$

### Step 2: Apply the Formulas

**First derivative:**

$$f'(3.0) = \frac{1}{0.5}\left[0.183 + \frac{1}{2}(-0.040) + \frac{1}{3}(0.031) + \frac{1}{4}(-0.015)\right]$$

$$= 2\left[0.183 - 0.020 + 0.01033 - 0.00375\right]$$

$$= 2 \times 0.16958 = 0.33917$$

**Exact value:** $f(x) = \ln x \Rightarrow f'(3) = 1/3 = 0.33333$

**Second derivative:**

$$f''(3.0) = \frac{1}{0.25}\left[-0.040 + 0.031 + \frac{11}{12}(-0.015)\right]$$

$$= 4\left[-0.040 + 0.031 - 0.01375\right]$$

$$= 4 \times (-0.02275) = -0.0910$$

**Exact value:** $f''(3) = -1/9 = -0.11111$

---

## 5. Simple Backward Difference Approximations

| Derivative | Formula | Error |
|-----------|---------|-------|
| $f'(x_n)$ | $\frac{f_n - f_{n-1}}{h}$ | $O(h)$ |
| $f'(x_n)$ | $\frac{3f_n - 4f_{n-1} + f_{n-2}}{2h}$ | $O(h^2)$ |
| $f''(x_n)$ | $\frac{f_n - 2f_{n-1} + f_{n-2}}{h^2}$ | $O(h)$ |

---

## 6. When to Use Which Formula

```
  Data table:   x₀    x₁    x₂    ...    xₙ₋₁    xₙ
                │                                  │
                │    FORWARD                       │
                │  formulas best    BACKWARD       │
                │  ◄──────────    formulas best    │
                │                  ──────────►     │
                │                                  │
                └──────────────────────────────────┘
```

---

## Summary Table

| Property | Value |
|----------|-------|
| **Best for** | Differentiating near the **end** of a table |
| **$f'(x_n)$** | $\frac{1}{h}[\nabla f_n + \frac{1}{2}\nabla^2 f_n + \frac{1}{3}\nabla^3 f_n + \cdots]$ |
| **$f''(x_n)$** | $\frac{1}{h^2}[\nabla^2 f_n + \nabla^3 f_n + \frac{11}{12}\nabla^4 f_n + \cdots]$ |
| **Signs** | All positive (unlike forward formulas which alternate) |
| **Accuracy** | Same as forward when same number of terms used |

---

## Quick Revision Questions

1. **Write the backward difference formulas for $f'(x_n)$ and $f''(x_n)$.**

2. **Given $f(3.0)=1.099, f(2.5)=0.916, f(2.0)=0.693$, estimate $f'(3.0)$ using the 3-point backward formula.**

3. **Compare the signs in forward and backward differentiation formulas.**

4. **When is backward difference differentiation preferred over forward difference?**

5. **Derive the $O(h^2)$ backward difference formula for $f'(x_n)$.**

6. **What happens if you apply backward difference formulas at the beginning of a table?**

---

[← Previous: Forward Difference](01-forward-difference.md) | [Next: Central Difference →](03-central-difference.md)
