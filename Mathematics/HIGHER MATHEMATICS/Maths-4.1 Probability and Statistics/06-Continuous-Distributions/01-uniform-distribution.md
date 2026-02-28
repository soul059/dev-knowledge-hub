# Chapter 6.1: Uniform Distribution (Continuous)

[← Previous: Hypergeometric Distribution](../05-Discrete-Distributions/06-hypergeometric-distribution.md) | [Back to README](../README.md) | [Next: Exponential Distribution →](02-exponential-distribution.md)

---

## Overview

The **Continuous Uniform distribution** models equally-likely outcomes over a continuous interval $[a, b]$. It represents **maximum ignorance** — when we know the range but nothing else about where a value will fall.

---

## 1. Definition

$$X \sim \text{Uniform}(a, b) \quad \text{or} \quad X \sim U(a, b)$$

### PDF

$$\boxed{f(x) = \begin{cases} \frac{1}{b-a} & a \leq x \leq b \\ 0 & \text{otherwise} \end{cases}}$$

### CDF

$$\boxed{F(x) = \begin{cases} 0 & x < a \\ \frac{x-a}{b-a} & a \leq x \leq b \\ 1 & x > b \end{cases}}$$

```
  PDF                               CDF
  
  f(x)                              F(x)
  │    ┌──────────┐                  1│              ●━━━━━
  │    │          │                   │            ╱
  1/(b-a)│          │                   │          ╱
  │    │          │                   │        ╱
  │    │          │                   │      ╱
  │────┘          └────              0│━━━━╱
  └────┼──────────┼──► x             └────┼──────────┼──► x
       a          b                       a          b
  
  Constant height                    Linear ramp
```

---

## 2. Key Properties

| Property | Value |
|----------|-------|
| Mean | $E[X] = \frac{a+b}{2}$ |
| Variance | $\text{Var}(X) = \frac{(b-a)^2}{12}$ |
| Std Dev | $\sigma = \frac{b-a}{\sqrt{12}} = \frac{b-a}{2\sqrt{3}}$ |
| Median | $(a+b)/2$ |
| Mode | Any point in $[a, b]$ |
| MGF | $\frac{e^{tb}-e^{ta}}{t(b-a)}$ for $t \neq 0$ |
| Skewness | 0 (symmetric) |
| Kurtosis | $9/5$ (excess = $-6/5$) |

### Standard Uniform: $U(0, 1)$

$$E[X] = 0.5, \quad \text{Var}(X) = 1/12 \approx 0.0833$$

---

## 3. Probability Calculations

For $U(a,b)$, probabilities are proportional to interval length:

$$\boxed{P(c \leq X \leq d) = \frac{d - c}{b - a}}$$

### Example 1

$X \sim U(2, 8)$

$$P(3 \leq X \leq 5) = \frac{5-3}{8-2} = \frac{2}{6} = \frac{1}{3}$$

$$P(X > 6) = \frac{8-6}{8-2} = \frac{1}{3}$$

$$E[X] = \frac{2+8}{2} = 5, \quad \text{Var}(X) = \frac{36}{12} = 3$$

---

## 4. Generating Other Distributions

### Inverse Transform Method

If $U \sim U(0,1)$ and $F$ is a CDF with inverse $F^{-1}$, then:

$$X = F^{-1}(U) \sim F$$

This is the basis for **random number generation** in computer simulations.

### Example 2: Generating Exponential from Uniform

Set $U \sim U(0,1)$. Then $X = -\frac{1}{\lambda}\ln(1-U) \sim \text{Exponential}(\lambda)$

**Proof**: $F_X(x) = 1 - e^{-\lambda x}$, so $F^{-1}(u) = -\frac{1}{\lambda}\ln(1-u)$ ✓

```
  Inverse Transform Sampling
  
  U ~ Uniform(0,1)
  │
  ▼
  ┌───────────────┐
  │ X = F⁻¹(U)   │
  └───────┬───────┘
          │
          ▼
  X ~ Any distribution with CDF F
```

---

## 5. Order Statistics from Uniform

If $U_1, U_2, \ldots, U_n \sim U(0,1)$ i.i.d., and $U_{(1)} \leq U_{(2)} \leq \cdots \leq U_{(n)}$ are the sorted values:

$$U_{(k)} \sim \text{Beta}(k, n-k+1)$$

$$E[U_{(k)}] = \frac{k}{n+1}, \quad \text{Var}(U_{(k)}] = \frac{k(n-k+1)}{(n+1)^2(n+2)}$$

---

## 6. Real-World Applications

| Application | Interval | Justification |
|-------------|----------|---------------|
| Random number generators | $[0, 1]$ | Foundation of all simulation |
| Rounding errors | $[-0.5, 0.5]$ | Equal chance of any rounding |
| Arrival within an hour | $[0, 60]$ | Equally likely any minute |
| Phase of a signal | $[0, 2\pi]$ | Random phase assumption |
| Random position on a segment | $[a, b]$ | No preferred location |

---

## Summary Table

| Concept | Formula |
|---------|---------|
| PDF | $1/(b-a)$ on $[a,b]$ |
| CDF | $(x-a)/(b-a)$ |
| $P(c \leq X \leq d)$ | $(d-c)/(b-a)$ |
| Mean | $(a+b)/2$ |
| Variance | $(b-a)^2/12$ |
| Inverse transform | $X = F^{-1}(U)$ generates $X \sim F$ |

---

## Quick Revision Questions

1. If $X \sim U(3, 9)$, find $E[X]$, $\text{Var}(X)$, and $P(4 < X < 7)$.

2. A bus arrives at a stop every 15 minutes. You arrive at a random time. What is the expected waiting time? Model this with a Uniform distribution.

3. Show that $f(x) = 1/(b-a)$ integrates to 1 over $[a,b]$.

4. Using the inverse transform method, show how to generate a sample from $U(3, 7)$ given $U \sim U(0,1)$.

5. If $X \sim U(0, 10)$, find $P(|X - 5| > 3)$.

6. Two people agree to meet between 12:00 and 1:00 PM. Each arrives at a uniformly random time and waits 15 minutes. What is the probability they meet? (Hint: geometric probability with $U(0,60)$.)

---

[← Previous: Hypergeometric Distribution](../05-Discrete-Distributions/06-hypergeometric-distribution.md) | [Back to README](../README.md) | [Next: Exponential Distribution →](02-exponential-distribution.md)
