# Chapter 7.2: Marginal Distributions

[← Previous: Joint PMF and PDF](01-joint-pmf-and-pdf.md) | [Back to README](../README.md) | [Next: Conditional Distributions →](03-conditional-distributions.md)

---

## Overview

Given a joint distribution of $(X, Y)$, we can recover the individual distributions of $X$ and $Y$ by **marginalizing** — summing or integrating out the other variable. These individual distributions are called **marginal distributions**.

---

## 1. Marginal PMF (Discrete Case)

$$\boxed{p_X(x) = \sum_{\text{all } y} p_{XY}(x, y)}$$

$$\boxed{p_Y(y) = \sum_{\text{all } x} p_{XY}(x, y)}$$

### Reading Marginals from a Table

| | $Y=0$ | $Y=1$ | $Y=2$ | $\mathbf{p_X(x)}$ |
|:-:|:-:|:-:|:-:|:-:|
| $X=0$ | 0.10 | 0.15 | 0.05 | **0.30** ← row sum |
| $X=1$ | 0.20 | 0.25 | 0.10 | **0.55** ← row sum |
| $X=2$ | 0.05 | 0.05 | 0.05 | **0.15** ← row sum |
| $\mathbf{p_Y(y)}$ | **0.35** | **0.45** | **0.20** | 1.00 |
| | ↑ col sum | ↑ col sum | ↑ col sum | |

```
  Why "Marginal"? — The name comes from the margins of the table!
  
  ┌─────────────────────────┐
  │  Joint probabilities    │ ← Marginal
  │  live in the interior   │    of X
  │  of the table           │    (right
  │                         │    margin)
  ├─────────────────────────┤
  │  Marginal of Y          │
  │  (bottom margin)        │
  └─────────────────────────┘
```

---

## 2. Marginal PDF (Continuous Case)

$$\boxed{f_X(x) = \int_{-\infty}^{\infty} f_{XY}(x, y)\,dy}$$

$$\boxed{f_Y(y) = \int_{-\infty}^{\infty} f_{XY}(x, y)\,dx}$$

### Geometric Interpretation

```
  f(x,y)         "Slice" at fixed x, integrate out y
  │  ╱╲
  │╱╱  ╲╲        → gives f_X(x) = height of marginal at x
  │      ╲╲
  └──────────► x      f_X(x)
  ╱                    │  ╱╲
 y                     │╱╱  ╲╲
                       └──────► x
  
  Like projecting a shadow onto the x-axis
```

---

## 3. Worked Examples

### Example 1: Marginal from Joint PMF

$(X, Y)$ has joint PMF: $p(0,0) = 0.2$, $p(0,1) = 0.1$, $p(1,0) = 0.3$, $p(1,1) = 0.4$.

**Marginal of X:**
- $p_X(0) = p(0,0) + p(0,1) = 0.2 + 0.1 = 0.3$
- $p_X(1) = p(1,0) + p(1,1) = 0.3 + 0.4 = 0.7$

**Marginal of Y:**
- $p_Y(0) = p(0,0) + p(1,0) = 0.2 + 0.3 = 0.5$
- $p_Y(1) = p(0,1) + p(1,1) = 0.1 + 0.4 = 0.5$

### Example 2: Marginal from Joint PDF

$$f(x,y) = 6(1 - y), \quad 0 < x < y < 1$$

**Region of integration:**

```
  y
  1┌──────────╱
   │        ╱  ← Line y = x
   │      ╱
   │    ╱  Region: 0 < x < y < 1
   │  ╱    (above the diagonal)
   │╱
  0└──────────► x
   0          1
```

**Marginal of X:**

For fixed $x \in (0,1)$, $y$ ranges from $x$ to $1$:

$$f_X(x) = \int_x^1 6(1-y)\,dy = 6\left[y - \frac{y^2}{2}\right]_x^1 \cdot (-1) = 6\left[\left(-y + \frac{y^2}{2}\right)\right]_x^1$$

Wait, let me redo:

$$f_X(x) = \int_x^1 6(1-y)\,dy = 6\left[(y - \frac{y^2}{2})\right]_x^1 = 6\left[\left(1-\frac{1}{2}\right) - \left(x - \frac{x^2}{2}\right)\right] = 6\left[\frac{1}{2} - x + \frac{x^2}{2}\right]$$

$$= 3(1 - 2x + x^2) = 3(1-x)^2, \quad 0 < x < 1$$

**Marginal of Y:**

For fixed $y \in (0,1)$, $x$ ranges from $0$ to $y$:

$$f_Y(y) = \int_0^y 6(1-y)\,dx = 6y(1-y), \quad 0 < y < 1$$

This is a $\text{Beta}(2, 2)$ distribution!

### Example 3: Bivariate Uniform

$(X,Y)$ uniform on the unit square: $f(x,y) = 1$ for $0 < x < 1, 0 < y < 1$.

$$f_X(x) = \int_0^1 1\,dy = 1, \quad 0 < x < 1$$

$$f_Y(y) = \int_0^1 1\,dx = 1, \quad 0 < y < 1$$

Both marginals are $\text{Uniform}(0,1)$.

---

## 4. Important Warning: Marginals Don't Determine the Joint

**Different joint distributions can have the same marginals!**

Consider two joint PMFs with $X, Y \in \{0, 1\}$:

**Joint A (Independent):**

| | $Y=0$ | $Y=1$ | $p_X$ |
|:-:|:-:|:-:|:-:|
| $X=0$ | 0.25 | 0.25 | 0.50 |
| $X=1$ | 0.25 | 0.25 | 0.50 |
| $p_Y$ | 0.50 | 0.50 | 1.00 |

**Joint B (Dependent):**

| | $Y=0$ | $Y=1$ | $p_X$ |
|:-:|:-:|:-:|:-:|
| $X=0$ | 0.50 | 0.00 | 0.50 |
| $X=1$ | 0.00 | 0.50 | 0.50 |
| $p_Y$ | 0.50 | 0.50 | 1.00 |

Same marginals, very different joint behavior!

```
  Joint A:               Joint B:
  ┌───┬───┐              ┌───┬───┐
  │ ■ │ ■ │  Uniform     │ ■ │   │  Concentrated
  ├───┼───┤  spread      ├───┼───┤  on diagonal
  │ ■ │ ■ │              │   │ ■ │
  └───┴───┘              └───┴───┘
  (independent)          (perfectly dependent)
```

---

## 5. Marginals and Expectation

From marginals alone:

$$E[X] = \sum_x x \cdot p_X(x) = \sum_x x \sum_y p_{XY}(x,y)$$

$$E[Y] = \sum_y y \cdot p_Y(y) = \sum_y y \sum_x p_{XY}(x,y)$$

$$E[X + Y] = E[X] + E[Y] \quad \text{(ALWAYS — no independence needed!)}$$

But for products: $E[XY] \neq E[X] \cdot E[Y]$ in general.

---

## 6. Marginal CDF

$$F_X(x) = \lim_{y \to \infty} F_{XY}(x, y) = F_{XY}(x, \infty)$$

$$F_Y(y) = \lim_{x \to \infty} F_{XY}(x, y) = F_{XY}(\infty, y)$$

---

## Summary Table

| Concept | Discrete | Continuous |
|---------|----------|------------|
| Marginal of $X$ | $p_X(x) = \sum_y p(x,y)$ | $f_X(x) = \int f(x,y)\,dy$ |
| Marginal of $Y$ | $p_Y(y) = \sum_x p(x,y)$ | $f_Y(y) = \int f(x,y)\,dx$ |
| Table interpretation | Row sums / Column sums | Project/integrate out |
| $E[X]$ from marginal | $\sum x \cdot p_X(x)$ | $\int x \cdot f_X(x)\,dx$ |
| Uniqueness | Marginals do NOT determine joint | Same |

---

## Quick Revision Questions

1. Given the joint PMF $p(1,1) = 0.1$, $p(1,2) = 0.2$, $p(2,1) = 0.3$, $p(2,2) = 0.4$: find both marginal PMFs and verify they sum to 1.

2. For $f(x,y) = 2$ on $0 < x < y < 1$: find $f_X(x)$ and $f_Y(y)$.

3. Give an example of two different joint distributions that have the same marginals but different correlation.

4. If $f(x,y) = e^{-x-y}$ for $x > 0, y > 0$: find both marginals. What familiar distributions are they?

5. Explain why $E[X+Y] = E[X] + E[Y]$ holds even when $X$ and $Y$ are dependent.

---

[← Previous: Joint PMF and PDF](01-joint-pmf-and-pdf.md) | [Back to README](../README.md) | [Next: Conditional Distributions →](03-conditional-distributions.md)
