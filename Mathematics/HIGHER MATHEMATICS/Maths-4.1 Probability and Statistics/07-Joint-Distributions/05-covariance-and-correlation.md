# Chapter 7.5: Covariance and Correlation

[← Previous: Independence of Random Variables](04-independence.md) | [Back to README](../README.md) | [Next: Population and Sample →](../08-Sampling-and-Estimation/01-population-and-sample.md)

---

## Overview

**Covariance** measures the linear association between two random variables — whether they tend to move together or in opposite directions. **Correlation** is the standardized version, always between $-1$ and $+1$.

---

## 1. Covariance

### Definition

$$\boxed{\text{Cov}(X, Y) = E[(X - \mu_X)(Y - \mu_Y)] = E[XY] - E[X]E[Y]}$$

The second form (shortcut) is almost always easier to compute.

### Sign Interpretation

```
  Cov(X,Y) > 0              Cov(X,Y) < 0              Cov(X,Y) = 0
  (Positive association)     (Negative association)      (No linear association)
  
  Y │        ·· ·            Y │ · ·                    Y │  · ·  ·
    │     · ··                 │   · ·                    │ ·  · ·
    │  · ··                    │     · ·                  │  · · ·
    │· ·                       │       · ·                │ ·  · ·
    └──────────► X             └──────────► X             └──────────► X
  
  X↑ ⟹ Y↑ tends            X↑ ⟹ Y↓ tends            No linear pattern
```

---

## 2. Properties of Covariance

| Property | Formula |
|----------|---------|
| Symmetry | $\text{Cov}(X,Y) = \text{Cov}(Y,X)$ |
| Self-covariance | $\text{Cov}(X,X) = \text{Var}(X)$ |
| Constants | $\text{Cov}(X,c) = 0$ |
| Scaling | $\text{Cov}(aX, bY) = ab\,\text{Cov}(X,Y)$ |
| Bilinearity | $\text{Cov}(X+Y, Z) = \text{Cov}(X,Z) + \text{Cov}(Y,Z)$ |
| Independence | $X \perp Y \Rightarrow \text{Cov}(X,Y) = 0$ |
| Converse fails | $\text{Cov}(X,Y) = 0 \not\Rightarrow X \perp Y$ |

### Variance of a Sum

$$\boxed{\text{Var}(X + Y) = \text{Var}(X) + \text{Var}(Y) + 2\text{Cov}(X,Y)}$$

For $n$ variables:

$$\text{Var}\left(\sum_{i=1}^n X_i\right) = \sum_{i=1}^n \text{Var}(X_i) + 2\sum_{i<j} \text{Cov}(X_i, X_j)$$

If all $X_i$ are **independent**: $\text{Var}\left(\sum X_i\right) = \sum \text{Var}(X_i)$.

---

## 3. Correlation Coefficient

### Definition

$$\boxed{\rho(X, Y) = \frac{\text{Cov}(X, Y)}{\sigma_X \sigma_Y} = \frac{\text{Cov}(X,Y)}{\sqrt{\text{Var}(X)\text{Var}(Y)}}}$$

### Key Properties

$$-1 \leq \rho(X,Y) \leq 1$$

| Value | Meaning |
|-------|---------|
| $\rho = +1$ | Perfect positive linear relationship: $Y = aX + b$, $a > 0$ |
| $\rho = -1$ | Perfect negative linear relationship: $Y = aX + b$, $a < 0$ |
| $\rho = 0$ | No linear association (uncorrelated) |
| $0 < \rho < 1$ | Positive linear tendency |
| $-1 < \rho < 0$ | Negative linear tendency |

```
  ρ = -1         ρ ≈ -0.7       ρ = 0          ρ ≈ 0.7        ρ = 1
  
  · ·              · ·           ·  · ·          · ·               · ·
    · ·          ·  · ·         ·· ·· ··        · · ·            · ·
      · ·         · · ·        · ·· · ·         · · ·          · ·
        · ·        · · ·        · ·  · ·         · ·         · ·
          · ·       · ·                            · ·     · ·
  
  Perfect       Strong          None           Strong        Perfect
  negative      negative                       positive      positive
```

---

## 4. Worked Examples

### Example 1: Discrete

| | $Y=0$ | $Y=1$ | $p_X$ |
|:-:|:-:|:-:|:-:|
| $X=0$ | 0.10 | 0.20 | 0.30 |
| $X=1$ | 0.30 | 0.40 | 0.70 |

**Step 1:** $E[X] = 0(0.3) + 1(0.7) = 0.7$, $E[Y] = 0(0.4) + 1(0.6) = 0.6$

**Step 2:** $E[XY] = \sum\sum xy \cdot p(x,y) = 0 + 0 + 0 + 1 \cdot 1 \cdot 0.4 = 0.4$

**Step 3:** $\text{Cov}(X,Y) = E[XY] - E[X]E[Y] = 0.4 - 0.7(0.6) = 0.4 - 0.42 = -0.02$

**Step 4:** $\text{Var}(X) = E[X^2] - (E[X])^2 = 0.7 - 0.49 = 0.21$

$\text{Var}(Y) = E[Y^2] - (E[Y])^2 = 0.6 - 0.36 = 0.24$

**Step 5:** $\rho(X,Y) = \frac{-0.02}{\sqrt{0.21 \times 0.24}} = \frac{-0.02}{\sqrt{0.0504}} = \frac{-0.02}{0.2245} \approx -0.089$

Weak negative correlation.

### Example 2: Continuous

$f(x,y) = 6(1-y)$ for $0 < x < y < 1$.

**Compute expectations:**

$E[X] = \int_0^1 x \cdot 3(1-x)^2\,dx = 3\int_0^1 (x - 2x^2 + x^3)\,dx = 3\left[\frac{1}{2} - \frac{2}{3} + \frac{1}{4}\right] = 3 \cdot \frac{1}{12} = \frac{1}{4}$

$E[Y] = \int_0^1 y \cdot 6y(1-y)\,dy = 6\int_0^1 (y^2 - y^3)\,dy = 6\left[\frac{1}{3} - \frac{1}{4}\right] = 6 \cdot \frac{1}{12} = \frac{1}{2}$

$E[XY] = \int_0^1\int_0^y 6xy(1-y)\,dx\,dy = \int_0^1 6y(1-y)\frac{y^2}{2}\,dy = 3\int_0^1 (y^3 - y^4)\,dy$

$= 3\left[\frac{1}{4} - \frac{1}{5}\right] = 3 \cdot \frac{1}{20} = \frac{3}{20}$

$\text{Cov}(X,Y) = \frac{3}{20} - \frac{1}{4}\cdot\frac{1}{2} = \frac{3}{20} - \frac{1}{8} = \frac{6-5}{40} = \frac{1}{40}$

Positive covariance — as expected, since $X < Y$ forces them to move together.

---

## 5. Covariance Matrix

For random vector $(X_1, X_2, \ldots, X_n)$:

$$\Sigma = \begin{pmatrix} \text{Var}(X_1) & \text{Cov}(X_1,X_2) & \cdots & \text{Cov}(X_1,X_n) \\ \text{Cov}(X_2,X_1) & \text{Var}(X_2) & \cdots & \text{Cov}(X_2,X_n) \\ \vdots & \vdots & \ddots & \vdots \\ \text{Cov}(X_n,X_1) & \text{Cov}(X_n,X_2) & \cdots & \text{Var}(X_n) \end{pmatrix}$$

Properties:
- Symmetric: $\Sigma_{ij} = \Sigma_{ji}$
- Positive semi-definite: $\mathbf{a}^T \Sigma \mathbf{a} \geq 0$ for all vectors $\mathbf{a}$
- Diagonal entries = variances, off-diagonal = covariances

### Correlation Matrix

$$R = \begin{pmatrix} 1 & \rho_{12} & \cdots & \rho_{1n} \\ \rho_{21} & 1 & \cdots & \rho_{2n} \\ \vdots & \vdots & \ddots & \vdots \\ \rho_{n1} & \rho_{n2} & \cdots & 1 \end{pmatrix}$$

Diagonal entries always 1.

---

## 6. Variance of Linear Combinations

For $W = a_1X_1 + a_2X_2 + \cdots + a_nX_n$:

$$\text{Var}(W) = \sum_{i=1}^n a_i^2 \text{Var}(X_i) + 2\sum_{i<j} a_i a_j \text{Cov}(X_i, X_j)$$

In matrix form: $\text{Var}(W) = \mathbf{a}^T \Sigma \mathbf{a}$

### Special Case: Sample Mean

If $X_1, \ldots, X_n$ are iid with variance $\sigma^2$:

$$\text{Var}(\bar{X}) = \text{Var}\left(\frac{1}{n}\sum X_i\right) = \frac{1}{n^2} \cdot n\sigma^2 = \frac{\sigma^2}{n}$$

---

## 7. Cauchy-Schwarz Inequality

$$\boxed{|\text{Cov}(X,Y)| \leq \sigma_X \sigma_Y}$$

equivalently: $|\rho(X,Y)| \leq 1$

Equality holds if and only if $Y = aX + b$ for constants $a \neq 0, b$.

---

## 8. Common Pitfalls

```
  CORRELATION ≠ CAUSATION
  
  ┌───────────────────────────────────────────────────┐
  │                                                   │
  │  Strong correlation between ice cream sales       │
  │  and drowning deaths does NOT mean ice cream      │
  │  causes drowning!                                 │
  │                                                   │
  │  Confounding variable: Temperature (summer)       │
  │                                                   │
  │  Hot day → More ice cream AND More swimming       │
  │                                                   │
  └───────────────────────────────────────────────────┘
  
  Also: ρ only captures LINEAR relationships!
  
  ρ = 0 does NOT mean "no relationship"
  (e.g., X ~ Uniform, Y = X² has ρ = 0 but strong dependence)
```

---

## Summary Table

| Concept | Formula | Range |
|---------|---------|-------|
| Covariance | $\text{Cov}(X,Y) = E[XY] - E[X]E[Y]$ | $(-\infty, \infty)$ |
| Correlation | $\rho = \text{Cov}(X,Y)/(\sigma_X\sigma_Y)$ | $[-1, 1]$ |
| $\text{Var}(X+Y)$ | $\text{Var}(X) + \text{Var}(Y) + 2\text{Cov}(X,Y)$ | — |
| $\rho = \pm 1$ | $Y = aX + b$ exactly | Perfect linear |
| $\rho = 0$ | Uncorrelated (not necessarily independent) | No linear pattern |
| Cauchy-Schwarz | $\|\text{Cov}(X,Y)\| \leq \sigma_X\sigma_Y$ | — |
| Covariance matrix | $\Sigma_{ij} = \text{Cov}(X_i, X_j)$ | Pos. semi-definite |

---

## Quick Revision Questions

1. Compute $\text{Cov}(X,Y)$ and $\rho(X,Y)$ for: $p(0,0) = 0.25, p(0,1) = 0.25, p(1,0) = 0.25, p(1,1) = 0.25$.

2. If $\text{Var}(X) = 4$, $\text{Var}(Y) = 9$, and $\rho(X,Y) = 0.5$, find $\text{Var}(2X - 3Y + 1)$.

3. Prove that $\text{Cov}(X + Y, X - Y) = \text{Var}(X) - \text{Var}(Y)$ using bilinearity.

4. Give an example of two random variables with $\rho = 0$ that are not independent.

5. If $X$ and $Y$ are independent with $\text{Var}(X) = \text{Var}(Y) = \sigma^2$, find $\rho(X+Y, X-Y)$.

6. Explain why correlation does not imply causation. Provide an example of a spurious correlation.

---

[← Previous: Independence of Random Variables](04-independence.md) | [Back to README](../README.md) | [Next: Population and Sample →](../08-Sampling-and-Estimation/01-population-and-sample.md)
