# Chapter 4.4: Moments and Moment Generating Functions

[← Previous: Variance and Standard Deviation](03-variance-and-standard-deviation.md) | [Back to README](../README.md) | [Next: Chebyshev's Inequality →](05-chebyshev-inequality.md)

---

## Overview

**Moments** generalize mean and variance by extending to higher powers. The **Moment Generating Function (MGF)** is a powerful tool that encodes ALL moments in a single function, uniquely identifies distributions, and simplifies work with sums of independent variables.

---

## 1. Moments

### Raw Moments (about the origin)

$$\boxed{\mu_k' = E[X^k]}$$

| Moment | Formula | Meaning |
|--------|---------|---------|
| 1st raw moment | $E[X]$ | Mean (center) |
| 2nd raw moment | $E[X^2]$ | Used in variance |
| 3rd raw moment | $E[X^3]$ | Used in skewness |
| 4th raw moment | $E[X^4]$ | Used in kurtosis |

### Central Moments (about the mean)

$$\boxed{\mu_k = E[(X - \mu)^k]}$$

| Moment | Formula | Meaning |
|--------|---------|---------|
| 1st central | $E[X - \mu] = 0$ | Always zero |
| 2nd central | $E[(X-\mu)^2]$ | **Variance** |
| 3rd central | $E[(X-\mu)^3]$ | Related to **Skewness** |
| 4th central | $E[(X-\mu)^4]$ | Related to **Kurtosis** |

---

## 2. Skewness and Kurtosis

### Skewness (3rd standardized moment)

$$\boxed{\gamma_1 = \frac{E[(X-\mu)^3]}{\sigma^3} = \frac{\mu_3}{\sigma^3}}$$

```
  Negative Skew           Symmetric              Positive Skew
  (Left-skewed)           (γ₁ = 0)               (Right-skewed)
  
       ╱╲                    ╱╲                       ╱╲
     ╱    ╲                ╱    ╲                   ╱    ╲
   ╱        ╲            ╱        ╲              ╱        ╲
  ╱────────────╲       ╱──────────────╲       ╱──────────────╲
  ◄─ tail    mean   tail ─► mean ◄─ tail     mean    tail ─►
  
  γ₁ < 0                γ₁ = 0                 γ₁ > 0
  
  Mean < Median         Mean = Median          Mean > Median
```

### Kurtosis (4th standardized moment)

$$\boxed{\kappa = \frac{E[(X-\mu)^4]}{\sigma^4} = \frac{\mu_4}{\sigma^4}}$$

**Excess Kurtosis** = $\kappa - 3$ (Normal distribution has $\kappa = 3$)

```
  Platykurtic             Mesokurtic              Leptokurtic
  (κ < 3)                 (κ = 3)                 (κ > 3)
  
      ╱───╲                  ╱╲                      ╱╲
    ╱       ╲              ╱    ╲                   ╱    ╲
  ╱           ╲          ╱        ╲               ╱  ╱  ╲  ╲
  ─────────────        ╱────────────╲          ╱╱──────────╲╲
  
  Light tails            Normal tails           Heavy tails
  Flat top               Reference              Sharp peak
```

---

## 3. Moment Generating Function (MGF)

$$\boxed{M_X(t) = E[e^{tX}]}$$

### Discrete

$$M_X(t) = \sum_x e^{tx} \cdot p_X(x)$$

### Continuous

$$M_X(t) = \int_{-\infty}^{\infty} e^{tx} \cdot f_X(x)\, dx$$

### Why "Moment Generating"?

Expand $e^{tX}$ as a Taylor series:

$$M_X(t) = E[e^{tX}] = E\left[1 + tX + \frac{(tX)^2}{2!} + \frac{(tX)^3}{3!} + \cdots\right]$$
$$= 1 + tE[X] + \frac{t^2}{2!}E[X^2] + \frac{t^3}{3!}E[X^3] + \cdots$$

Therefore:

$$\boxed{E[X^k] = M_X^{(k)}(0) = \frac{d^k}{dt^k} M_X(t)\Bigg|_{t=0}}$$

The $k$-th derivative of the MGF at $t=0$ gives the $k$-th raw moment!

---

## 4. MGFs for Common Distributions

| Distribution | MGF $M_X(t)$ | Valid for |
|:------------|:-------------|:----------|
| Bernoulli($p$) | $1-p+pe^t$ | All $t$ |
| Binomial($n,p$) | $(1-p+pe^t)^n$ | All $t$ |
| Poisson($\lambda$) | $e^{\lambda(e^t - 1)}$ | All $t$ |
| Geometric($p$) | $\frac{pe^t}{1-(1-p)e^t}$ | $t < -\ln(1-p)$ |
| Uniform($a,b$) | $\frac{e^{tb}-e^{ta}}{t(b-a)}$ | $t \neq 0$ |
| Exponential($\lambda$) | $\frac{\lambda}{\lambda - t}$ | $t < \lambda$ |
| Normal($\mu,\sigma^2$) | $e^{\mu t + \sigma^2 t^2/2}$ | All $t$ |
| Gamma($\alpha,\beta$) | $\left(\frac{\beta}{\beta-t}\right)^\alpha$ | $t < \beta$ |

---

## 5. Extracting Moments — Worked Examples

### Example 1: Exponential Distribution

$M_X(t) = \frac{\lambda}{\lambda - t}$

$$M_X'(t) = \frac{\lambda}{(\lambda-t)^2}, \quad M_X'(0) = \frac{1}{\lambda} = E[X] \checkmark$$

$$M_X''(t) = \frac{2\lambda}{(\lambda-t)^3}, \quad M_X''(0) = \frac{2}{\lambda^2} = E[X^2]$$

$$\text{Var}(X) = E[X^2] - (E[X])^2 = \frac{2}{\lambda^2} - \frac{1}{\lambda^2} = \frac{1}{\lambda^2} \checkmark$$

### Example 2: Normal Distribution

$M_X(t) = e^{\mu t + \sigma^2 t^2/2}$

$$M_X'(t) = (\mu + \sigma^2 t) e^{\mu t + \sigma^2 t^2/2}$$

$$M_X'(0) = \mu = E[X] \checkmark$$

$$M_X''(t) = (\sigma^2 + (\mu + \sigma^2 t)^2) e^{\mu t + \sigma^2 t^2/2}$$

$$M_X''(0) = \sigma^2 + \mu^2 = E[X^2]$$

$$\text{Var}(X) = \sigma^2 + \mu^2 - \mu^2 = \sigma^2 \checkmark$$

---

## 6. Key Properties of MGFs

### Uniqueness Theorem

$$\text{If } M_X(t) = M_Y(t) \text{ for all } t \text{ in an interval around 0, then } X \overset{d}{=} Y$$

> Two distributions are the same if and only if they have the same MGF.

### Linear Transformation

$$M_{aX+b}(t) = e^{bt} \cdot M_X(at)$$

### Sum of Independent Variables

$$\boxed{M_{X+Y}(t) = M_X(t) \cdot M_Y(t) \quad \text{if } X \perp Y}$$

More generally: $M_{X_1 + \cdots + X_n}(t) = \prod_{i=1}^n M_{X_i}(t)$ if independent.

```
  MGF of Sum of Independents
  
  ┌───────────┐
  │  X₁ ~ F₁  │──── M_X₁(t)
  │  X₂ ~ F₂  │──── M_X₂(t)     MULTIPLY     M_S(t) = M_X₁(t)·M_X₂(t)·M_X₃(t)
  │  X₃ ~ F₃  │──── M_X₃(t)  ─────────────►  Identify the distribution of S
  └───────────┘
  
  S = X₁ + X₂ + X₃
```

---

## 7. Using MGFs to Identify Distributions

### Example 3: Sum of Independent Poissons

$X \sim \text{Poisson}(\lambda_1)$, $Y \sim \text{Poisson}(\lambda_2)$, $X \perp Y$

$$M_{X+Y}(t) = e^{\lambda_1(e^t-1)} \cdot e^{\lambda_2(e^t-1)} = e^{(\lambda_1+\lambda_2)(e^t-1)}$$

This is the MGF of $\text{Poisson}(\lambda_1 + \lambda_2)$!

$$\therefore X + Y \sim \text{Poisson}(\lambda_1 + \lambda_2)$$

### Example 4: Sum of Independent Normals

$X \sim N(\mu_1, \sigma_1^2)$, $Y \sim N(\mu_2, \sigma_2^2)$, $X \perp Y$

$$M_{X+Y}(t) = e^{\mu_1 t + \sigma_1^2 t^2/2} \cdot e^{\mu_2 t + \sigma_2^2 t^2/2} = e^{(\mu_1+\mu_2)t + (\sigma_1^2+\sigma_2^2)t^2/2}$$

$$\therefore X + Y \sim N(\mu_1 + \mu_2, \sigma_1^2 + \sigma_2^2)$$

### Example 5: Sum of Independent Binomials (same $p$)

$X \sim \text{Bin}(n_1, p)$, $Y \sim \text{Bin}(n_2, p)$, $X \perp Y$

$$M_{X+Y}(t) = (1-p+pe^t)^{n_1} \cdot (1-p+pe^t)^{n_2} = (1-p+pe^t)^{n_1+n_2}$$

$$\therefore X + Y \sim \text{Bin}(n_1 + n_2, p)$$

---

## 8. Cumulant Generating Function

$$K_X(t) = \ln M_X(t)$$

Cumulants $\kappa_n$ are extracted similarly:

| Cumulant | Value |
|----------|-------|
| $\kappa_1 = K'(0)$ | Mean |
| $\kappa_2 = K''(0)$ | Variance |
| $\kappa_3 = K'''(0)$ | Related to skewness |
| $\kappa_4 = K^{(4)}(0)$ | Related to excess kurtosis |

> For the Normal distribution: $K(t) = \mu t + \sigma^2 t^2/2$, so only $\kappa_1$ and $\kappa_2$ are nonzero.

---

## 9. When MGF Doesn't Exist

The MGF may not exist if $E[e^{tX}]$ diverges for all $t \neq 0$.

**Example**: Cauchy distribution, Log-normal distribution (MGF exists only at $t=0$).

**Alternative**: Use the **characteristic function** $\varphi_X(t) = E[e^{itX}]$ which **always** exists.

---

## Summary Table

| Concept | Formula / Key Idea |
|---------|-------------------|
| $k$-th raw moment | $\mu_k' = E[X^k]$ |
| $k$-th central moment | $\mu_k = E[(X-\mu)^k]$ |
| Skewness | $\gamma_1 = \mu_3/\sigma^3$ |
| Kurtosis | $\kappa = \mu_4/\sigma^4$ |
| MGF definition | $M_X(t) = E[e^{tX}]$ |
| Extract moments | $E[X^k] = M^{(k)}(0)$ |
| Uniqueness | Same MGF $\Rightarrow$ same distribution |
| Sum of indep. | $M_{X+Y}(t) = M_X(t) M_Y(t)$ |
| Linear transform | $M_{aX+b}(t) = e^{bt} M_X(at)$ |

---

## Quick Revision Questions

1. Given $M_X(t) = e^{3t + 8t^2}$, find $E[X]$ and $\text{Var}(X)$.

2. **Derive** the MGF of the Bernoulli($p$) distribution from the definition, and verify $E[X] = p$ using $M'(0)$.

3. $X_1, X_2, X_3$ are independent $\text{Exponential}(\lambda)$. Use MGFs to find the distribution of $S = X_1 + X_2 + X_3$.

4. Is a distribution with skewness $\gamma_1 = 0$ necessarily symmetric? Give a reason or counterexample.

5. A distribution has $E[X] = 2$, $E[X^2] = 8$, $E[X^3] = 26$. Find its skewness.

6. Why can't we use MGFs for the Cauchy distribution? What alternative function always exists?

---

[← Previous: Variance and Standard Deviation](03-variance-and-standard-deviation.md) | [Back to README](../README.md) | [Next: Chebyshev's Inequality →](05-chebyshev-inequality.md)
