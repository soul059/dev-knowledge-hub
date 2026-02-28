# Chapter 8.3: Central Limit Theorem

[← Previous: Sampling Distributions](02-sampling-distributions.md) | [Back to README](../README.md) | [Next: Point Estimation →](04-point-estimation.md)

---

## Overview

The **Central Limit Theorem (CLT)** is arguably the most important theorem in statistics. It states that the sum (or average) of a large number of independent random variables is approximately Normal, *regardless of the original distribution*.

---

## 1. Statement of the CLT

Let $X_1, X_2, \ldots, X_n$ be iid with $E[X_i] = \mu$ and $\text{Var}(X_i) = \sigma^2 < \infty$.

Then as $n \to \infty$:

$$\boxed{\frac{\bar{X} - \mu}{\sigma/\sqrt{n}} \xrightarrow{d} N(0, 1)}$$

Equivalently:

$$\boxed{\bar{X} \approx N\left(\mu, \frac{\sigma^2}{n}\right) \quad \text{for large } n}$$

Or for the sum $S_n = X_1 + \cdots + X_n$:

$$S_n \approx N(n\mu, n\sigma^2)$$

---

## 2. Visual Demonstration

```
  Original Distribution          Distribution of X̄
  (can be ANYTHING)              (approaches Normal)
  
  Uniform:                       n=2     n=5     n=30
  ┌──────────┐                   ╱╲      ╱╲       ╱╲
  │██████████│                  ╱  ╲    ╱╱╲╲    ╱╱│╲╲
  └──────────┘               ──╱────╲──╱╱──╲╲──╱╱─┼─╲╲──
  
  Exponential:                   n=2     n=5     n=30
  │╲                             ╱╲      ╱╲       ╱╲
  │ ╲                           ╱╲ ╲    ╱╱╲╲    ╱╱│╲╲
  │  ╲___                    ──╱╱──╲╲──╱╱──╲╲──╱╱─┼─╲╲──
  
  Bimodal:                       n=2     n=5     n=30
  ╱╲  ╱╲                        ╱╲      ╱╲       ╱╲
  ╱  ╲╱  ╲                     ╱  ╲    ╱╱╲╲    ╱╱│╲╲
  ────────                   ──╱────╲──╱╱──╲╲──╱╱─┼─╲╲──
  
  ALL converge to Normal shape as n increases!
```

---

## 3. How Large Must $n$ Be?

| Original Distribution | $n$ Needed | Reason |
|-----------------------|-----------|--------|
| Normal | $n = 1$ | Already Normal! |
| Symmetric, unimodal | $n \geq 5$ | Close to Normal already |
| Mildly skewed | $n \geq 15$ | Moderate correction needed |
| Heavily skewed | $n \geq 30$ | Standard rule of thumb |
| Very skewed (e.g., Exponential) | $n \geq 40-50$ | More averaging needed |
| Discrete with few values | $n \geq 30$ | With continuity correction |

**Rule of Thumb:** $n \geq 30$ is usually sufficient.

---

## 4. Applying the CLT

### Step-by-Step Procedure

1. Identify $\mu = E[X_i]$ and $\sigma^2 = \text{Var}(X_i)$
2. Standardize: $Z = \frac{\bar{X} - \mu}{\sigma / \sqrt{n}}$
3. Use the standard Normal table

### Example 1: Average Test Score

Scores have $\mu = 72$, $\sigma = 15$. For $n = 100$ students:

$$\bar{X} \approx N\left(72, \frac{225}{100}\right) = N(72, 2.25)$$

$$P(\bar{X} > 75) = P\left(Z > \frac{75 - 72}{1.5}\right) = P(Z > 2) = 0.0228$$

### Example 2: Sum of Dice

Roll $n = 100$ fair dice. $X_i \sim \text{Uniform}\{1,\ldots,6\}$, $\mu = 3.5$, $\sigma^2 = 35/12$.

$$S_{100} = \sum_{i=1}^{100} X_i \approx N(350, 100 \times 35/12) = N(350, 291.67)$$

$$P(S_{100} > 370) = P\left(Z > \frac{370 - 350}{\sqrt{291.67}}\right) = P(Z > 1.17) = 0.121$$

---

## 5. CLT for Proportions

If $X_1, \ldots, X_n \overset{\text{iid}}{\sim} \text{Bernoulli}(p)$:

$$\hat{p} = \bar{X} = \frac{\sum X_i}{n} \approx N\left(p, \frac{p(1-p)}{n}\right)$$

**Conditions:** $np \geq 5$ and $n(1-p) \geq 5$.

### Example 3: Election Polling

Poll $n = 1000$ voters, true support $p = 0.52$.

$$\hat{p} \approx N\left(0.52, \frac{0.52 \times 0.48}{1000}\right) = N(0.52, 0.0002496)$$

$$\text{SE}(\hat{p}) = \sqrt{0.0002496} \approx 0.0158$$

$$P(\hat{p} > 0.50) = P\left(Z > \frac{0.50 - 0.52}{0.0158}\right) = P(Z > -1.27) = 0.898$$

---

## 6. Continuity Correction

When using CLT for discrete distributions, add/subtract 0.5:

$$P(X \leq k) \approx P\left(Z \leq \frac{k + 0.5 - \mu}{\sigma}\right)$$

$$P(X \geq k) \approx P\left(Z \geq \frac{k - 0.5 - \mu}{\sigma}\right)$$

### Example 4

$X \sim \text{Bin}(100, 0.3)$: $\mu = 30$, $\sigma = \sqrt{21} \approx 4.58$.

$$P(X \leq 25\text{ — exact}) \approx P\left(Z \leq \frac{25.5 - 30}{4.58}\right) = P(Z \leq -0.98) = 0.1635$$

```
  Continuity Correction
  
  Discrete bars:    ▒ ▒ ▒ ▒ ▒ ▒ ▒ ▒
                    24 25 26 27 28 29 30 31
  
  P(X ≤ 25) includes all bars UP TO AND INCLUDING 25
  
  Each bar occupies [k-0.5, k+0.5]:
  Bar at 25: [24.5, 25.5]
  
  So P(X ≤ 25) ≈ P(continuous Z ≤ 25.5)
                                   ↑
                              add 0.5
```

---

## 7. Why the CLT Works (Intuition)

```
  The CLT via MGFs (sketch)
  
  M_X̄(t) = [M_X(t/√n)]ⁿ
  
  For large n, expand M_X(t/√n) in Taylor series:
  
  M_X(t/√n) ≈ 1 + μ(t/√n) + (μ²+σ²)(t/√n)²/2 + ...
  
  After standardizing and taking limit:
  
  lim M_{Z_n}(t) = e^{t²/2}  ← MGF of N(0,1)!
  n→∞
  
  Convergence of MGFs ⟹ Convergence in distribution
```

---

## 8. Limitations

| Limitation | Explanation |
|-----------|-------------|
| Finite variance required | CLT fails for Cauchy (infinite variance) |
| Independence required | Dependent data needs modified CLT |
| Rate of convergence | Berry-Esseen: error $\leq C \cdot E|X^3|/(\sigma^3\sqrt{n})$ |
| Heavy tails | Slower convergence |

---

## Summary Table

| Concept | Formula/Result |
|---------|---------------|
| CLT for $\bar{X}$ | $\bar{X} \approx N(\mu, \sigma^2/n)$ |
| CLT for $S_n$ | $S_n \approx N(n\mu, n\sigma^2)$ |
| CLT for $\hat{p}$ | $\hat{p} \approx N(p, p(1-p)/n)$ |
| Standardize | $Z = (\bar{X} - \mu)/(\sigma/\sqrt{n})$ |
| Rule of thumb | $n \geq 30$ (less if symmetric) |
| Continuity correction | Add $\pm 0.5$ for discrete → continuous |
| Key requirement | $\text{Var}(X) < \infty$ and independence |

---

## Quick Revision Questions

1. State the CLT. What are its key assumptions?

2. Lightbulb lifetimes have $\mu = 1000$ hours, $\sigma = 200$ hours. For $n = 64$ bulbs, find $P(\bar{X} > 1050)$.

3. A coin has $P(\text{heads}) = 0.6$. In 400 flips, find $P(\hat{p} < 0.55)$ using the CLT.

4. Why does the CLT fail for the Cauchy distribution?

5. Apply continuity correction: If $X \sim \text{Bin}(200, 0.5)$, find $P(X \geq 110)$.

6. How does the CLT justify using $Z$-tests and $t$-tests even for non-Normal populations?

---

[← Previous: Sampling Distributions](02-sampling-distributions.md) | [Back to README](../README.md) | [Next: Point Estimation →](04-point-estimation.md)
