# Chapter 7.4: Independence of Random Variables

[← Previous: Conditional Distributions](03-conditional-distributions.md) | [Back to README](../README.md) | [Next: Covariance and Correlation →](05-covariance-and-correlation.md)

---

## Overview

Two random variables are **independent** if knowing the value of one gives no information about the other. This is the multivariate extension of event independence and has powerful computational consequences.

---

## 1. Definition of Independence

$X$ and $Y$ are independent if and only if:

$$\boxed{f_{XY}(x,y) = f_X(x) \cdot f_Y(y) \quad \text{for all } (x,y)}$$

Equivalently (any ONE suffices):

| Criterion | Discrete | Continuous |
|-----------|----------|------------|
| Joint = Product of marginals | $p(x,y) = p_X(x) \cdot p_Y(y)$ for all $x,y$ | $f(x,y) = f_X(x) \cdot f_Y(y)$ for all $x,y$ |
| CDF factorizes | $F(x,y) = F_X(x) \cdot F_Y(y)$ for all $x,y$ | Same |
| Conditional = Marginal | $p_{Y\|X}(y\|x) = p_Y(y)$ for all $x,y$ | $f_{Y\|X}(y\|x) = f_Y(y)$ for all $x,y$ |
| MGFs multiply | $M_{XY}(s,t) = M_X(s) \cdot M_Y(t)$ | Same |

```
  Independent                    Dependent
  
  ┌─────────────────┐            ┌─────────────────┐
  │ ■ ■ ■ ■ ■ ■ ■ ■ │            │           ■ ■ ■ │
  │ ■ ■ ■ ■ ■ ■ ■ ■ │            │       ■ ■ ■     │
  │ ■ ■ ■ ■ ■ ■ ■ ■ │            │   ■ ■ ■         │
  │ ■ ■ ■ ■ ■ ■ ■ ■ │            │ ■ ■ ■           │
  └─────────────────┘            └─────────────────┘
  f(x,y) = f_X(x)·f_Y(y)        f(x,y) ≠ f_X(x)·f_Y(y)
  "Rectangular" support           Non-rectangular support
```

---

## 2. Testing Independence (Discrete)

### Step-by-Step Method

1. Compute all marginals $p_X(x)$ and $p_Y(y)$
2. For EVERY cell, check if $p(x,y) = p_X(x) \cdot p_Y(y)$
3. If even ONE cell fails → **dependent**

### Example: Independent

| | $Y=0$ | $Y=1$ | $p_X$ |
|:-:|:-:|:-:|:-:|
| $X=0$ | 0.12 | 0.18 | 0.30 |
| $X=1$ | 0.28 | 0.42 | 0.70 |
| $p_Y$ | 0.40 | 0.60 | 1.00 |

Check: $p(0,0) = 0.12 = 0.30 \times 0.40 = p_X(0) \cdot p_Y(0)$ ✓

Check all cells: $0.18 = 0.30 \times 0.60$ ✓, $0.28 = 0.70 \times 0.40$ ✓, $0.42 = 0.70 \times 0.60$ ✓

**Independent!**

### Example: Dependent

| | $Y=0$ | $Y=1$ | $p_X$ |
|:-:|:-:|:-:|:-:|
| $X=0$ | 0.50 | 0.00 | 0.50 |
| $X=1$ | 0.00 | 0.50 | 0.50 |
| $p_Y$ | 0.50 | 0.50 | 1.00 |

Check: $p(0,0) = 0.50 \neq 0.50 \times 0.50 = 0.25$ ✗

**Dependent!** (Knowing $X$ completely determines $Y$.)

---

## 3. Testing Independence (Continuous)

### The Factorization Criterion

$X$ and $Y$ are independent if and only if:

$$f_{XY}(x,y) = g(x) \cdot h(y)$$

for some non-negative functions $g$ and $h$ (they don't need to be proper densities).

### Quick Check: Support Must Be Rectangular

If the support of $(X,Y)$ is **not** a rectangle (product of intervals), then $X$ and $Y$ are **dependent**.

```
  Independent support         Dependent support
  (rectangular)               (non-rectangular)
  
  y                           y
  │ ┌──────┐                  │  ╱╲
  │ │      │                  │ ╱  ╲
  │ │      │                  │╱    ╲
  │ └──────┘                  └──────► x
  └──────────► x              
  {a ≤ x ≤ b} × {c ≤ y ≤ d}  Triangle → DEPENDENT
```

### Example: Independent

$$f(x,y) = e^{-x} \cdot e^{-y} = e^{-(x+y)}, \quad x > 0, y > 0$$

- Support is $(0,\infty) \times (0,\infty)$ — rectangular ✓
- $f(x,y) = \underbrace{e^{-x}}_{g(x)} \cdot \underbrace{e^{-y}}_{h(y)}$ — factorizes ✓
- **Independent!** Both are $\text{Exp}(1)$.

### Example: Dependent

$$f(x,y) = 2, \quad 0 < x < y < 1$$

- Support: triangle (not rectangular) → **Dependent!**

---

## 4. Consequences of Independence

If $X$ and $Y$ are independent:

| Property | Formula |
|----------|---------|
| $E[g(X)h(Y)]$ factorizes | $E[g(X)h(Y)] = E[g(X)] \cdot E[h(Y)]$ |
| $E[XY]$ factorizes | $E[XY] = E[X] \cdot E[Y]$ |
| Covariance is zero | $\text{Cov}(X,Y) = 0$ |
| Variances add | $\text{Var}(X + Y) = \text{Var}(X) + \text{Var}(Y)$ |
| MGFs multiply | $M_{X+Y}(t) = M_X(t) \cdot M_Y(t)$ |

### Critical Warning

$$\text{Cov}(X,Y) = 0 \quad \not\Rightarrow \quad X, Y \text{ independent}$$

**Counterexample:** $X \sim \text{Uniform}(-1,1)$, $Y = X^2$.

$\text{Cov}(X,Y) = E[XY] - E[X]E[Y] = E[X^3] - 0 \cdot E[X^2] = 0$ (by symmetry)

But $Y = X^2$ so $Y$ is completely determined by $X$ → maximally dependent!

```
  ZERO COVARIANCE vs INDEPENDENCE
  
  Independent ──────────────► Zero Covariance ✓ (always)
  
  Zero Covariance ────╳────► Independent ✗ (not always)
  
  Exception: For jointly NORMAL RVs,
  zero covariance ⟺ independence ✓
```

---

## 5. Functions of Independent Variables

If $X \perp Y$ (independent), then:

$$g(X) \perp h(Y) \quad \text{for any functions } g, h$$

Examples:
- $X^2 \perp e^Y$
- $\sin(X) \perp Y^3 + 1$
- $|X| \perp \mathbb{1}(Y > 0)$

---

## 6. Mutual Independence (Multiple Variables)

$X_1, X_2, \ldots, X_n$ are **mutually independent** if:

$$f(x_1, \ldots, x_n) = \prod_{i=1}^n f_{X_i}(x_i) \quad \text{for all } (x_1, \ldots, x_n)$$

### Pairwise vs Mutual

$$\text{Mutual independence} \Rightarrow \text{Pairwise independence}$$

$$\text{Pairwise independence} \not\Rightarrow \text{Mutual independence}$$

(Same distinction as with events — see Chapter 1.5.)

---

## 7. IID Random Variables

$X_1, X_2, \ldots, X_n$ are **iid** (independent and identically distributed) if:

1. They are mutually independent
2. They all have the same distribution

This is the standard assumption for random sampling!

$$\text{iid} = \text{independent} + \text{same distribution}$$

---

## 8. Sums of Independent Variables

If $X \perp Y$:

| Distribution | $X + Y$ |
|-------------|---------|
| $X \sim \text{Bin}(n,p), Y \sim \text{Bin}(m,p)$ | $X+Y \sim \text{Bin}(n+m, p)$ |
| $X \sim \text{Poi}(\lambda_1), Y \sim \text{Poi}(\lambda_2)$ | $X+Y \sim \text{Poi}(\lambda_1 + \lambda_2)$ |
| $X \sim N(\mu_1, \sigma_1^2), Y \sim N(\mu_2, \sigma_2^2)$ | $X+Y \sim N(\mu_1+\mu_2, \sigma_1^2+\sigma_2^2)$ |
| $X \sim \text{Gamma}(\alpha_1,\beta), Y \sim \text{Gamma}(\alpha_2,\beta)$ | $X+Y \sim \text{Gamma}(\alpha_1+\alpha_2, \beta)$ |
| $X \sim \chi^2(k_1), Y \sim \chi^2(k_2)$ | $X+Y \sim \chi^2(k_1+k_2)$ |

These are proven easily via MGFs!

---

## Summary Table

| Concept | Key Idea |
|---------|----------|
| Independence | $f(x,y) = f_X(x) \cdot f_Y(y)$ for ALL $(x,y)$ |
| Discrete check | Verify $p(x,y) = p_X(x)p_Y(y)$ for every cell |
| Continuous quick check | Support must be rectangular |
| Factorization | $f(x,y) = g(x)h(y)$ ⟹ independent |
| Cov = 0 ⟹ independent? | **No!** (except for joint Normal) |
| Independent ⟹ Cov = 0? | **Yes!** Always |
| Sum distributions | Closed-form for many families |
| iid | Independent + identically distributed |

---

## Quick Revision Questions

1. $p(0,0)=0.06$, $p(0,1)=0.14$, $p(1,0)=0.24$, $p(1,1)=0.56$. Are $X$ and $Y$ independent?

2. $f(x,y) = 4xy$ for $0 < x < 1, 0 < y < 1$. Are $X$ and $Y$ independent? Find $E[XY]$.

3. Give an example where $\text{Cov}(X,Y)=0$ but $X$ and $Y$ are dependent.

4. If $X \sim N(0,1)$ and $Y \sim N(0,1)$ are independent, find the distribution of $3X - 2Y + 5$.

5. Why does the triangular support $0 < x < y < 1$ immediately imply dependence?

6. If $X_1, \ldots, X_n$ are iid $\text{Exp}(\lambda)$, what is the distribution of $\sum X_i$?

---

[← Previous: Conditional Distributions](03-conditional-distributions.md) | [Back to README](../README.md) | [Next: Covariance and Correlation →](05-covariance-and-correlation.md)
