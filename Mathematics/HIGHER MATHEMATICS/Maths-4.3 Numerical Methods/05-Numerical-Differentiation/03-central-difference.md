# Chapter 5.3: Central Difference Differentiation

[← Previous: Backward Difference](02-backward-difference.md) | [Next: Higher Order Derivatives →](04-higher-order-derivatives.md)

---

## Overview

**Central difference formulas** use data points on **both sides** of the point of interest. They provide significantly higher accuracy than forward or backward formulas of the same order, making them the preferred choice whenever data is available on both sides.

---

## 1. Central Difference Operators

Recall the central difference operator:

$$\delta f_i = f_{i+1/2} - f_{i-1/2}$$

and the averaging operator:

$$\mu f_i = \frac{1}{2}(f_{i+1/2} + f_{i-1/2})$$

---

## 2. Key Central Difference Formulas

### First Derivative

**2-point (simplest):**

$$\boxed{f'(x_0) = \frac{f_1 - f_{-1}}{2h}} \quad [O(h^2)]$$

**4-point (higher accuracy):**

$$f'(x_0) = \frac{-f_2 + 8f_1 - 8f_{-1} + f_{-2}}{12h} \quad [O(h^4)]$$

### Second Derivative

**3-point:**

$$\boxed{f''(x_0) = \frac{f_1 - 2f_0 + f_{-1}}{h^2}} \quad [O(h^2)]$$

**5-point:**

$$f''(x_0) = \frac{-f_2 + 16f_1 - 30f_0 + 16f_{-1} - f_{-2}}{12h^2} \quad [O(h^4)]$$

---

## 3. Why Central Differences Are More Accurate

```
  Forward:   uses f₀, f₁  ──► Error O(h)
             ●────●
             x₀   x₁

  Backward:  uses f₋₁, f₀ ──► Error O(h)
             ●────●
            x₋₁   x₀

  Central:   uses f₋₁, f₁ ──► Error O(h²) ← BETTER!
             ●────●────●
            x₋₁   x₀   x₁
                   ↑
             Symmetric about x₀ cancels odd-order error terms
```

The symmetry causes **odd-order error terms to cancel**, giving $O(h^2)$ accuracy instead of $O(h)$.

---

## 4. Stirling's Central Difference Formula

Using $\mu\delta$ notation for derivatives at $x = x_0$:

$$f'(x_0) = \frac{1}{h}\left[\mu\delta f_0 - \frac{1}{6}\mu\delta^3 f_0 + \frac{1}{30}\mu\delta^5 f_0 - \cdots\right]$$

where $\mu\delta f_0 = \frac{1}{2}(\Delta f_0 + \Delta f_{-1}) = \frac{f_1 - f_{-1}}{2}$

---

## 5. Worked Example

Given $f(x)$:

| $x$ | 0.1 | 0.2 | 0.3 | 0.4 | 0.5 |
|-----|-----|-----|-----|-----|-----|
| $f(x)$ | 0.0998 | 0.1987 | 0.2955 | 0.3894 | 0.4794 |

Find $f'(0.3)$ and $f''(0.3)$ using central differences. ($h = 0.1$)

**First derivative (central 2-point):**

$$f'(0.3) = \frac{f(0.4) - f(0.2)}{2(0.1)} = \frac{0.3894 - 0.1987}{0.2} = \frac{0.1907}{0.2} = 0.9535$$

**First derivative (central 4-point):**

$$f'(0.3) = \frac{-f(0.5) + 8f(0.4) - 8f(0.2) + f(0.1)}{12(0.1)}$$

$$= \frac{-0.4794 + 3.1152 - 1.5896 + 0.0998}{1.2} = \frac{1.1460}{1.2} = 0.9550$$

**Second derivative (central 3-point):**

$$f''(0.3) = \frac{f(0.4) - 2f(0.3) + f(0.2)}{(0.1)^2} = \frac{0.3894 - 0.5910 + 0.1987}{0.01}$$

$$= \frac{-0.0029}{0.01} = -0.290$$

**If $f(x) = \sin(x)$**: $f'(0.3) = \cos(0.3) = 0.9553$ and $f''(0.3) = -\sin(0.3) = -0.2955$. Good agreement!

---

## 6. Richardson Extrapolation

To improve accuracy, compute the derivative with step sizes $h$ and $h/2$, then extrapolate:

$$D_{\text{improved}} = \frac{4D(h/2) - D(h)}{3}$$

This boosts a $O(h^2)$ formula to $O(h^4)$ accuracy.

### Example

If $f'_{h} = 0.9535$ (with $h = 0.1$) and $f'_{h/2} = 0.9549$ (with $h = 0.05$):

$$f'_{\text{improved}} = \frac{4(0.9549) - 0.9535}{3} = \frac{3.8196 - 0.9535}{3} = \frac{2.8661}{3} = 0.9554$$

---

## 7. Complete Family of Central Formulas

| Derivative | Points | Formula | Error |
|-----------|--------|---------|-------|
| $f'$ | 3 | $\frac{f_1 - f_{-1}}{2h}$ | $O(h^2)$ |
| $f'$ | 5 | $\frac{-f_2 + 8f_1 - 8f_{-1} + f_{-2}}{12h}$ | $O(h^4)$ |
| $f''$ | 3 | $\frac{f_1 - 2f_0 + f_{-1}}{h^2}$ | $O(h^2)$ |
| $f''$ | 5 | $\frac{-f_2 + 16f_1 - 30f_0 + 16f_{-1} - f_{-2}}{12h^2}$ | $O(h^4)$ |
| $f'''$ | 5 | $\frac{f_2 - 2f_1 + 2f_{-1} - f_{-2}}{2h^3}$ | $O(h^2)$ |
| $f^{(4)}$ | 5 | $\frac{f_2 - 4f_1 + 6f_0 - 4f_{-1} + f_{-2}}{h^4}$ | $O(h^2)$ |

---

## Summary Table

| Property | Value |
|----------|-------|
| **Best for** | Interior points (data on both sides) |
| **$f'$ (basic)** | $\frac{f_1 - f_{-1}}{2h}$, error $O(h^2)$ |
| **$f''$ (basic)** | $\frac{f_1 - 2f_0 + f_{-1}}{h^2}$, error $O(h^2)$ |
| **Accuracy gain** | Odd-error-term cancellation via symmetry |
| **Enhancement** | Richardson extrapolation: $O(h^2) \to O(h^4)$ |

---

## Quick Revision Questions

1. **Write the central difference formulas for $f'(x_0)$ and $f''(x_0)$ with error $O(h^2)$.**

2. **Explain why central differences are more accurate than forward/backward differences.**

3. **Given: $f(0.9)=0.7833, f(1.0)=0.8415, f(1.1)=0.8912$, find $f'(1.0)$ and $f''(1.0)$.**

4. **What is Richardson extrapolation and how does it improve accuracy?**

5. **Write the 5-point central difference formula for $f'(x_0)$.**

6. **When can you NOT use central difference formulas?**

---

[← Previous: Backward Difference](02-backward-difference.md) | [Next: Higher Order Derivatives →](04-higher-order-derivatives.md)
