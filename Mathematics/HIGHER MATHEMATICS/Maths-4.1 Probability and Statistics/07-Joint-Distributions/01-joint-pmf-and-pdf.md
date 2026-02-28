# Chapter 7.1: Joint PMF and PDF

[← Previous: F-Distribution](../06-Continuous-Distributions/08-f-distribution.md) | [Back to README](../README.md) | [Next: Marginal Distributions →](02-marginal-distributions.md)

---

## Overview

So far we've studied one random variable at a time. **Joint distributions** describe the simultaneous behavior of two or more random variables, capturing how they co-vary and interact.

---

## 1. Joint PMF (Discrete Case)

For discrete RVs $X$ and $Y$:

$$\boxed{p_{XY}(x, y) = P(X = x, Y = y)}$$

### Properties

1. $p_{XY}(x, y) \geq 0$ for all $(x, y)$
2. $\sum_x \sum_y p_{XY}(x, y) = 1$

### Example 1: Two Dice

$X$ = first die, $Y$ = second die.

$p_{XY}(i, j) = 1/36$ for $i, j \in \{1, 2, 3, 4, 5, 6\}$

### Example 2: Joint PMF Table

| | $Y=0$ | $Y=1$ | $Y=2$ | $p_X(x)$ |
|:-:|:-:|:-:|:-:|:-:|
| $X=0$ | 0.10 | 0.15 | 0.05 | 0.30 |
| $X=1$ | 0.20 | 0.25 | 0.10 | 0.55 |
| $X=2$ | 0.05 | 0.05 | 0.05 | 0.15 |
| $p_Y(y)$ | 0.35 | 0.45 | 0.20 | **1.00** |

---

## 2. Joint PDF (Continuous Case)

For continuous RVs $X$ and $Y$:

$$\boxed{f_{XY}(x, y) \geq 0 \quad \text{and} \quad \int_{-\infty}^{\infty}\int_{-\infty}^{\infty} f_{XY}(x,y)\,dx\,dy = 1}$$

### Probabilities from Joint PDF

$$P((X,Y) \in A) = \iint_A f_{XY}(x,y)\,dx\,dy$$

```
  Joint PDF visualized as a surface over the xy-plane:
  
  f(x,y)
  │    ╱╲
  │  ╱╱  ╲╲
  │╱╱      ╲╲    The volume under the surface = 1
  │          ╲╲
  │            ╲
  └────────────────► x
  ╱
 y
  
  P(region A) = Volume above A under the surface
```

---

## 3. Joint CDF

$$\boxed{F_{XY}(x, y) = P(X \leq x, Y \leq y)}$$

### Properties

1. $F(-\infty, y) = F(x, -\infty) = 0$
2. $F(\infty, \infty) = 1$
3. Non-decreasing in each argument
4. Right-continuous in each argument

### Recovering PMF/PDF

- Discrete: $p(x,y) = F(x,y) - F(x^-, y) - F(x, y^-) + F(x^-, y^-)$
- Continuous: $f(x,y) = \frac{\partial^2}{\partial x \partial y} F(x,y)$

---

## 4. Worked Examples

### Example 3: Continuous Joint PDF

$$f(x,y) = \begin{cases} 6(1-y) & 0 < x < y < 1 \\ 0 & \text{otherwise} \end{cases}$$

**Verify it integrates to 1:**

$$\int_0^1 \int_0^y 6(1-y)\,dx\,dy = \int_0^1 6y(1-y)\,dy = 6\left[\frac{y^2}{2} - \frac{y^3}{3}\right]_0^1 = 6\left(\frac{1}{2} - \frac{1}{3}\right) = 6 \cdot \frac{1}{6} = 1 \checkmark$$

**Find $P(X < 1/2, Y < 1/2)$:**

$$\int_0^{1/2}\int_0^y 6(1-y)\,dx\,dy = \int_0^{1/2} 6y(1-y)\,dy = 6\left[\frac{y^2}{2}-\frac{y^3}{3}\right]_0^{1/2} = 6\left(\frac{1}{8}-\frac{1}{24}\right) = \frac{1}{2}$$

### Example 4: Uniform on a Region

$(X,Y)$ uniform on the unit disc $x^2 + y^2 \leq 1$:

$$f(x,y) = \frac{1}{\text{Area}} = \frac{1}{\pi}, \quad x^2 + y^2 \leq 1$$

$$P(X > 0, Y > 0) = \frac{\text{Area of quarter disc}}{\text{Area of disc}} = \frac{1}{4}$$

---

## 5. Visualization

```
  Discrete Joint PMF                Continuous Joint PDF
  (3D bar chart)                    (3D surface)
  
  p(x,y)                            f(x,y)
  │  ▯                               │      ╱╲
  │  ▯ ▯                             │    ╱╱  ╲╲
  │  ▯ ▯ ▯                           │  ╱╱  ╱╲  ╲╲
  │  ▯ ▯ ▯                           │╱╱  ╱    ╲  ╲╲
  └──┴─┴─┴──► x                      └───╱────────╲───► x
  ╱                                   ╱
 y                                   y
  
  Heights = probabilities            Volume under surface = 1
  Sum of heights = 1
```

---

## 6. Functions of Two Random Variables

For $Z = g(X, Y)$:

$$E[g(X,Y)] = \begin{cases} \sum_x \sum_y g(x,y) \cdot p_{XY}(x,y) & \text{discrete} \\ \int\int g(x,y) \cdot f_{XY}(x,y)\,dx\,dy & \text{continuous} \end{cases}$$

---

## Summary Table

| Concept | Discrete | Continuous |
|---------|----------|------------|
| Joint distribution | $p(x,y) = P(X=x, Y=y)$ | $f(x,y)$ with $\iint f = 1$ |
| Probability of region | $\sum\sum p(x,y)$ | $\iint_A f(x,y)\,dx\,dy$ |
| CDF | $F(x,y) = P(X \leq x, Y \leq y)$ | Same |
| $E[g(X,Y)]$ | $\sum\sum g \cdot p$ | $\iint g \cdot f$ |
| Normalization | $\sum\sum p = 1$ | $\iint f = 1$ |

---

## Quick Revision Questions

1. From the joint PMF table in Example 2, find $P(X + Y \leq 2)$ and $E[XY]$.

2. Given $f(x,y) = 2$ for $0 < x < y < 1$, verify this is a valid PDF and find $P(Y > 0.5)$.

3. Explain the difference between a joint PMF table and a contingency table.

4. For $(X,Y)$ uniform on the triangle with vertices $(0,0)$, $(1,0)$, $(0,1)$: find the joint PDF and $P(X+Y < 0.5)$.

5. If $f(x,y) = e^{-x-y}$ for $x > 0, y > 0$, verify it's a valid PDF. Are $X$ and $Y$ independent?

6. Find $E[X+Y]$ for the joint PDF $f(x,y) = 6(1-y)$ on $0 < x < y < 1$.

---

[← Previous: F-Distribution](../06-Continuous-Distributions/08-f-distribution.md) | [Back to README](../README.md) | [Next: Marginal Distributions →](02-marginal-distributions.md)
