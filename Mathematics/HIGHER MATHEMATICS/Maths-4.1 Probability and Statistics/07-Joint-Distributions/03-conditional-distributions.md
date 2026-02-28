# Chapter 7.3: Conditional Distributions

[← Previous: Marginal Distributions](02-marginal-distributions.md) | [Back to README](../README.md) | [Next: Independence of Random Variables →](04-independence.md)

---

## Overview

Just as conditional probability refines our knowledge given an event, **conditional distributions** describe how one random variable behaves given a specific value of another.

---

## 1. Conditional PMF (Discrete Case)

$$\boxed{p_{Y|X}(y|x) = \frac{p_{XY}(x, y)}{p_X(x)}, \quad p_X(x) > 0}$$

This is a valid PMF in $y$: $\sum_y p_{Y|X}(y|x) = 1$.

### Example

| | $Y=0$ | $Y=1$ | $p_X$ |
|:-:|:-:|:-:|:-:|
| $X=0$ | 0.10 | 0.20 | 0.30 |
| $X=1$ | 0.30 | 0.40 | 0.70 |

**Conditional PMF of $Y$ given $X = 0$:**

$$p_{Y|X}(0|0) = \frac{0.10}{0.30} = \frac{1}{3}, \quad p_{Y|X}(1|0) = \frac{0.20}{0.30} = \frac{2}{3}$$

**Conditional PMF of $Y$ given $X = 1$:**

$$p_{Y|X}(0|1) = \frac{0.30}{0.70} = \frac{3}{7}, \quad p_{Y|X}(1|1) = \frac{0.40}{0.70} = \frac{4}{7}$$

```
  P(Y|X=0)                P(Y|X=1)
  
  2/3 │     █              4/7 │     █
      │     █                  │     █
  1/3 │ █   █              3/7 │ █   █
      │ █   █                  │ █   █
      └─┴───┴── Y             └─┴───┴── Y
        0   1                    0   1
  
  Different conditional distributions → X and Y are dependent
```

---

## 2. Conditional PDF (Continuous Case)

$$\boxed{f_{Y|X}(y|x) = \frac{f_{XY}(x,y)}{f_X(x)}, \quad f_X(x) > 0}$$

This is a valid PDF in $y$: $\int_{-\infty}^{\infty} f_{Y|X}(y|x)\,dy = 1$.

### Example

$$f(x,y) = 6(1-y), \quad 0 < x < y < 1$$

From Chapter 7.2: $f_X(x) = 3(1-x)^2$ for $0 < x < 1$.

$$f_{Y|X}(y|x) = \frac{6(1-y)}{3(1-x)^2} = \frac{2(1-y)}{(1-x)^2}, \quad x < y < 1$$

**Verify:** $\int_x^1 \frac{2(1-y)}{(1-x)^2}\,dy = \frac{2}{(1-x)^2}\left[(y - \frac{y^2}{2})\right]_x^1 = \frac{2}{(1-x)^2}\cdot\frac{(1-x)^2}{2} = 1$ ✓

---

## 3. Conditional Expectation

### Definition

$$\boxed{E[Y|X=x] = \begin{cases} \sum_y y \cdot p_{Y|X}(y|x) & \text{discrete} \\ \int_{-\infty}^{\infty} y \cdot f_{Y|X}(y|x)\,dy & \text{continuous} \end{cases}}$$

### Example (Discrete)

Using the PMF table above:

$$E[Y|X=0] = 0 \cdot \frac{1}{3} + 1 \cdot \frac{2}{3} = \frac{2}{3}$$

$$E[Y|X=1] = 0 \cdot \frac{3}{7} + 1 \cdot \frac{4}{7} = \frac{4}{7}$$

---

## 4. Law of Total Expectation (Tower Property)

$$\boxed{E[Y] = E[E[Y|X]]}$$

- $E[Y|X]$ is a **function of $X$** (a random variable itself)
- Taking its expectation over $X$ gives $E[Y]$

```
  The Tower Property — Iterated Expectation
  
  E[Y] ← Overall average
    │
    ├── E[Y|X=x₁] · P(X=x₁)    ← Weighted average
    ├── E[Y|X=x₂] · P(X=x₂)       of conditional
    ├── E[Y|X=x₃] · P(X=x₃)       expectations
    └── ...
  
  Discrete:   E[Y] = Σ_x E[Y|X=x] · p_X(x)
  Continuous:  E[Y] = ∫ E[Y|X=x] · f_X(x) dx
```

### Verification with our Example

$$E[Y] = E[Y|X=0] \cdot P(X=0) + E[Y|X=1] \cdot P(X=1)$$

$$= \frac{2}{3}(0.30) + \frac{4}{7}(0.70) = \frac{2}{10} + \frac{4}{10} = 0.2 + 0.4 = 0.6$$

**Check directly:** $E[Y] = 0 \cdot p_Y(0) + 1 \cdot p_Y(1) = 0(0.40) + 1(0.60) = 0.60$ ✓

---

## 5. Law of Total Variance

$$\boxed{\text{Var}(Y) = E[\text{Var}(Y|X)] + \text{Var}(E[Y|X])}$$

| Component | Name | Interpretation |
|-----------|------|----------------|
| $E[\text{Var}(Y\|X)]$ | Within-group variance | Average variance within each group |
| $\text{Var}(E[Y\|X])$ | Between-group variance | Variance of the group means |

```
  Total Variance Decomposition
  
  Var(Y) = E[Var(Y|X)]  +  Var(E[Y|X])
           ════════════     ════════════
           "Unexplained"   "Explained"
           variation        variation
           (within groups)  (between groups)
```

### Application: Random Sum

If $N$ is a random count and $X_1, X_2, \ldots$ are iid:

$$S = X_1 + X_2 + \cdots + X_N$$

$$E[S] = E[N] \cdot E[X_1]$$

$$\text{Var}(S) = E[N] \cdot \text{Var}(X_1) + (E[X_1])^2 \cdot \text{Var}(N)$$

---

## 6. Conditional Variance

$$\text{Var}(Y|X=x) = E[Y^2|X=x] - (E[Y|X=x])^2$$

---

## 7. Bayes' Rule for Distributions

### Discrete

$$p_{X|Y}(x|y) = \frac{p_{Y|X}(y|x) \cdot p_X(x)}{p_Y(y)}$$

### Continuous

$$f_{X|Y}(x|y) = \frac{f_{Y|X}(y|x) \cdot f_X(x)}{f_Y(y)}$$

This is the foundation of **Bayesian inference**:

$$\text{Posterior} \propto \text{Likelihood} \times \text{Prior}$$

---

## 8. Worked Example: Insurance Claims

An insurance company has two types of policyholders:
- Type A (60%): Claims follow Poisson($\lambda = 1$)
- Type B (40%): Claims follow Poisson($\lambda = 3$)

**Expected number of claims per policyholder:**

$$E[N] = E[E[N|T]] = E[N|T=A] \cdot P(T=A) + E[N|T=B] \cdot P(T=B)$$

$$= 1(0.6) + 3(0.4) = 0.6 + 1.2 = 1.8$$

**Variance of claims:**

$$\text{Var}(N) = E[\text{Var}(N|T)] + \text{Var}(E[N|T])$$

$$= [1(0.6) + 3(0.4)] + [(1-1.8)^2(0.6) + (3-1.8)^2(0.4)]$$

$$= 1.8 + [0.384 + 0.576] = 1.8 + 0.96 = 2.76$$

Note: $\text{Var}(N) = 2.76 > 1.8 = E[N]$ → overdispersion relative to Poisson!

---

## Summary Table

| Concept | Formula | Key Property |
|---------|---------|--------------|
| Conditional PMF | $p_{Y\|X}(y\|x) = p(x,y)/p_X(x)$ | Sums to 1 over $y$ |
| Conditional PDF | $f_{Y\|X}(y\|x) = f(x,y)/f_X(x)$ | Integrates to 1 over $y$ |
| Conditional expectation | $E[Y\|X=x]$ | Function of $x$ |
| Tower property | $E[Y] = E[E[Y\|X]]$ | Iterated expectation |
| Law of total variance | $\text{Var}(Y) = E[\text{Var}(Y\|X)] + \text{Var}(E[Y\|X])$ | Within + Between |
| Bayes for distributions | $f_{X\|Y} \propto f_{Y\|X} \cdot f_X$ | Posterior ∝ Likelihood × Prior |

---

## Quick Revision Questions

1. From the table: $p(1,1)=0.1$, $p(1,2)=0.2$, $p(2,1)=0.3$, $p(2,2)=0.4$. Find $P(Y=1|X=2)$.

2. For $f(x,y) = 2$ on $0 < x < y < 1$, find $f_{Y|X}(y|x)$ and identify the conditional distribution.

3. State the Law of Total Expectation and use it to find $E[Y]$ when $E[Y|X=0] = 3$, $E[Y|X=1] = 7$, $P(X=0) = 0.4$.

4. Decompose the variance using the Law of Total Variance for the insurance example if Type A has $\lambda = 2$ and Type B has $\lambda = 5$.

5. Explain why $E[Y|X]$ is a random variable, not a number.

---

[← Previous: Marginal Distributions](02-marginal-distributions.md) | [Back to README](../README.md) | [Next: Independence of Random Variables →](04-independence.md)
