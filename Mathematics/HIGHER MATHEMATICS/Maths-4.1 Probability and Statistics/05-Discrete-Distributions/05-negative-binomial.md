# Chapter 5.5: Negative Binomial Distribution

[← Previous: Geometric Distribution](04-geometric-distribution.md) | [Back to README](../README.md) | [Next: Hypergeometric Distribution →](06-hypergeometric-distribution.md)

---

## Overview

The **Negative Binomial distribution** generalizes the Geometric distribution. While the Geometric counts trials until the **first** success, the Negative Binomial counts trials until the **$r$-th** success.

---

## 1. Definition

$X$ = number of trials needed to get $r$ successes.

$$\boxed{P(X = k) = \binom{k-1}{r-1} p^r (1-p)^{k-r}, \quad k = r, r+1, r+2, \ldots}$$

### Interpretation

$$P(X = k) = \underbrace{\binom{k-1}{r-1}}_{\substack{\text{ways to place} \\ r-1 \text{ successes in} \\ \text{first } k-1 \text{ trials}}} \times \underbrace{p^r}_{\substack{r \text{ total} \\ \text{successes}}} \times \underbrace{(1-p)^{k-r}}_{\substack{k-r \\ \text{failures}}}$$

The last trial (the $k$-th) **must** be a success. The remaining $r-1$ successes are distributed among the first $k-1$ trials.

### Notation

$$X \sim \text{NegBin}(r, p) \quad \text{or} \quad X \sim \text{NB}(r, p)$$

---

## 2. Alternative Parameterization

### Version 2: $Y$ = number of failures before $r$-th success

$$P(Y = k) = \binom{k+r-1}{k} p^r (1-p)^k, \quad k = 0, 1, 2, \ldots$$

$Y = X - r$ where $X$ is from Version 1.

```
  Example: NegBin(r=3, p=0.4) — waiting for 3rd success
  
  Trial:  F  S  F  F  S  F  S
          1  2  3  4  5  6  7
          ×  ✓  ×  ×  ✓  ×  ✓ ← 3rd success on trial 7
  
  X = 7 (total trials)
  Y = 4 (total failures)
```

---

## 3. Key Properties

| Property | Version 1 (total trials) |
|----------|:--:|
| Mean | $E[X] = r/p$ |
| Variance | $\text{Var}(X) = r(1-p)/p^2$ |
| MGF | $\left(\frac{pe^t}{1-(1-p)e^t}\right)^r$ |
| Mode | $1 + \lfloor (r-1)(1-p)/p \rfloor$ for $r > 1$ |

### Derivation of Mean

$X = X_1 + X_2 + \cdots + X_r$ where $X_i \sim \text{Geometric}(p)$ independent.

$$E[X] = \sum_{i=1}^r E[X_i] = r \cdot \frac{1}{p} = \frac{r}{p}$$

$$\text{Var}(X) = \sum_{i=1}^r \text{Var}(X_i) = r \cdot \frac{1-p}{p^2}$$

---

## 4. Special Cases

| Case | Distribution |
|------|:---:|
| $r = 1$ | **Geometric($p$)** |
| $r = n$, as Binomial complement | Fixed successes, count trials |

```
  Negative Binomial Family
  
  r = 1:  Geometric  X₁
  r = 2:  Sum of two Geometrics  X₁ + X₂
  r = 3:  Sum of three Geometrics  X₁ + X₂ + X₃
    ⋮
  r = r:  Sum of r Geometrics  X₁ + X₂ + ... + Xᵣ
  
  ┌─────────────────────────────────────────────┐
  │  NegBin(r, p) = sum of r i.i.d. Geom(p)    │
  └─────────────────────────────────────────────┘
```

---

## 5. Binomial vs. Negative Binomial

| Feature | Binomial | Negative Binomial |
|---------|----------|------------------|
| Fixed | Number of trials $n$ | Number of successes $r$ |
| Random | Number of successes | Number of trials |
| Question | "How many successes in $n$ trials?" | "How many trials for $r$ successes?" |
| Support | $\{0, 1, \ldots, n\}$ | $\{r, r+1, r+2, \ldots\}$ |
| Mean | $np$ | $r/p$ |
| Variance | $np(1-p)$ | $r(1-p)/p^2$ |

```
  Complementary perspectives:
  
  Binomial(n, p):
  Fixed ─────n trials─────►  Random: count successes
  
  NegBin(r, p):
  Random ──?? trials──►  Fixed: stop at r successes
```

---

## 6. Worked Examples

### Example 1: Recruiting

You need to hire $r = 3$ engineers. Each candidate has $P(\text{hire}) = 0.2$.

$$E[\text{interviews}] = \frac{3}{0.2} = 15$$

$$\text{SD} = \sqrt{\frac{3 \times 0.8}{0.04}} = \sqrt{60} \approx 7.75$$

$$P(X = 3) = \binom{2}{2}(0.2)^3(0.8)^0 = 0.008 \quad (\text{all 3 are hires})$$

$$P(X = 5) = \binom{4}{2}(0.2)^3(0.8)^2 = 6 \times 0.008 \times 0.64 = 0.03072$$

### Example 2: Clinical Trials

A drug has 70% success rate. How many patients until 10 successes?

$X \sim \text{NegBin}(10, 0.7)$

$$E[X] = \frac{10}{0.7} \approx 14.3 \text{ patients}$$

$$\text{Var}(X) = \frac{10 \times 0.3}{0.49} \approx 6.12$$

### Example 3: World Series

In baseball's World Series, the first team to win 4 games wins. If teams are evenly matched ($p = 0.5$):

Number of games = $\text{NegBin}(4, 0.5)$ (from one team's perspective)

$$E[\text{games}] = \frac{4}{0.5} = 8 \quad \text{(but it's capped at 7, so this is approximate)}$$

---

## 7. Negative Binomial for Overdispersion

In count data, when **variance > mean** (overdispersion), the Negative Binomial often fits better than Poisson.

| Model | Mean–Variance Relationship |
|-------|---------------------------|
| Poisson($\lambda$) | $\text{Var} = \text{Mean} = \lambda$ |
| NegBin($r, p$) | $\text{Var} = \text{Mean}/p > \text{Mean}$ |

> The Poisson is a special case of the NegBin as $r \to \infty$.

---

## 8. Real-World Applications

| Application | $r$ | $p$ |
|-------------|-----|-----|
| Oil well drilling: trials until $r$ productive wells | 3 | 0.1 |
| Sales: calls until $r$ orders | 5 | 0.15 |
| Genetics: offspring until $r$ with trait | 2 | 0.25 |
| Customer service: interactions until $r$ complaints | 10 | 0.05 |
| Sports: games until $r$ wins | 4 | 0.5 |

---

## Summary Table

| Concept | Formula |
|---------|---------|
| PMF | $\binom{k-1}{r-1}p^r(1-p)^{k-r}$, $k \geq r$ |
| Mean | $r/p$ |
| Variance | $r(1-p)/p^2$ |
| Relationship | Sum of $r$ i.i.d. Geometric($p$) |
| Special case $r=1$ | Geometric distribution |
| MGF | $(pe^t/(1-(1-p)e^t))^r$ |

---

## Quick Revision Questions

1. You roll a fair die repeatedly. Find the expected number of rolls until you've rolled three 6's.

2. If $X \sim \text{NegBin}(5, 0.3)$, find $E[X]$, $\text{Var}(X)$, and $P(X = 5)$.

3. Show that $\text{NegBin}(1, p) = \text{Geometric}(p)$ by simplifying the PMF.

4. A salesperson closes deals with probability 0.2. How many calls are expected to close 4 deals? What is the probability of closing all 4 in exactly 10 calls?

5. Explain why the Negative Binomial has larger variance than the Poisson for the same mean, and why this makes it useful for modeling overdispersed count data.

6. Derive $E[X] = r/p$ using the decomposition into Geometric random variables and linearity of expectation.

---

[← Previous: Geometric Distribution](04-geometric-distribution.md) | [Back to README](../README.md) | [Next: Hypergeometric Distribution →](06-hypergeometric-distribution.md)
