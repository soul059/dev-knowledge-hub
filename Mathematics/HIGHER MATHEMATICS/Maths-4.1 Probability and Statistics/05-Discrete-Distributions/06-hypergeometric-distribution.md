# Chapter 5.6: Hypergeometric Distribution

[← Previous: Negative Binomial Distribution](05-negative-binomial.md) | [Back to README](../README.md) | [Next: Uniform Distribution →](../06-Continuous-Distributions/01-uniform-distribution.md)

---

## Overview

The **Hypergeometric distribution** models the number of successes in draws **without replacement** from a finite population. Unlike the Binomial (with replacement / independent trials), the Hypergeometric captures the changing probabilities when items are removed from the pool.

---

## 1. Definition

A finite population of $N$ items contains $K$ successes and $N - K$ failures. Draw $n$ items **without replacement**. Let $X$ = number of successes drawn.

$$\boxed{P(X = k) = \frac{\binom{K}{k}\binom{N-K}{n-k}}{\binom{N}{n}}}$$

### Valid range

$$\max(0, n - N + K) \leq k \leq \min(n, K)$$

### Parameters

| Symbol | Meaning |
|--------|---------|
| $N$ | Population size |
| $K$ | Number of successes in population |
| $n$ | Number of draws (sample size) |

### Interpretation

$$P(X=k) = \frac{\text{ways to choose } k \text{ successes}}{\text{ways to choose } n \text{ items}} \times \text{ways to choose } n-k \text{ failures}$$

$$= \frac{\binom{K}{k} \times \binom{N-K}{n-k}}{\binom{N}{n}}$$

---

## 2. Key Properties

| Property | Value |
|----------|-------|
| Mean | $E[X] = n \cdot \frac{K}{N}$ |
| Variance | $\text{Var}(X) = n \cdot \frac{K}{N} \cdot \frac{N-K}{N} \cdot \frac{N-n}{N-1}$ |
| Support | $\max(0, n+K-N)$ to $\min(n, K)$ |

### Finite Population Correction Factor

$$\text{Var}(X) = \underbrace{n \cdot \frac{K}{N} \cdot \frac{N-K}{N}}_{\text{Binomial variance}} \times \underbrace{\frac{N-n}{N-1}}_{\text{FPC}}$$

The **FPC** (Finite Population Correction) is always $\leq 1$, so sampling without replacement has **less variability** than with replacement.

```
  Sampling WITH vs WITHOUT replacement
  
  With replacement (Binomial):
  ┌──────────────────────┐
  │ ● ● ● ○ ○ ○ ○ ○ ○ ○ │ → Draw → Replace → Draw → ...
  │                      │   Probabilities stay the same
  └──────────────────────┘
  
  Without replacement (Hypergeometric):
  ┌──────────────────────┐
  │ ● ● ● ○ ○ ○ ○ ○ ○ ○ │ → Draw → Remove → Draw → ...
  │                      │   Probabilities CHANGE
  └──────────────────────┘
  
  ● = success   ○ = failure
```

---

## 3. Worked Examples

### Example 1: Card Game

A standard deck (52 cards, 13 hearts). Draw 5 cards without replacement. Find $P(\text{exactly 3 hearts})$.

$N = 52$, $K = 13$, $n = 5$, $k = 3$

$$P(X=3) = \frac{\binom{13}{3}\binom{39}{2}}{\binom{52}{5}} = \frac{286 \times 741}{2{,}598{,}960} = \frac{211{,}926}{2{,}598{,}960} \approx 0.0815$$

### Example 2: Quality Inspection

A lot of 100 items contains 5 defectives. An inspector draws 10 without replacement.

$N = 100$, $K = 5$, $n = 10$

$$E[X] = 10 \times \frac{5}{100} = 0.5$$

$$P(X = 0) = \frac{\binom{5}{0}\binom{95}{10}}{\binom{100}{10}} = \frac{\binom{95}{10}}{\binom{100}{10}} \approx 0.5838$$

$$P(X \geq 1) = 1 - P(X=0) \approx 0.4162$$

### Example 3: Lottery

A lottery draws 6 numbers from 1–49. You pick 6 numbers. What is $P(\text{all 6 match})$?

$$P(X=6) = \frac{\binom{6}{6}\binom{43}{0}}{\binom{49}{6}} = \frac{1}{13{,}983{,}816} \approx 7.15 \times 10^{-8}$$

---

## 4. Hypergeometric vs. Binomial

| Feature | Hypergeometric | Binomial |
|---------|:-:|:-:|
| Sampling | **Without** replacement | **With** replacement |
| Trials | Dependent | Independent |
| Probability | Changes each draw | Constant $p$ |
| Population | Finite $N$ | Infinite (or with replacement) |
| Mean | $nK/N$ | $np$ |
| Variance | $npq \cdot \frac{N-n}{N-1}$ | $npq$ |

### Binomial Approximation

When $N$ is much larger than $n$ (rule of thumb: $n/N < 0.05$):

$$\text{Hypergeometric}(N, K, n) \approx \text{Binomial}(n, K/N)$$

The FPC factor $\frac{N-n}{N-1} \approx 1$ when $N \gg n$.

```
  Approximation convergence:
  
  n/N = 0.50  →  FPC = 0.505   (very different from Binomial)
  n/N = 0.10  →  FPC = 0.909   (moderate difference)
  n/N = 0.05  →  FPC = 0.953   (close to Binomial)
  n/N = 0.01  →  FPC = 0.990   (essentially Binomial)
  n/N → 0     →  FPC → 1.000   (exact Binomial)
```

---

## 5. Fisher's Exact Test

The Hypergeometric is the basis for **Fisher's Exact Test** — testing association in a 2×2 contingency table.

### Setup

|  | Success | Failure | Total |
|--|:-------:|:-------:|:-----:|
| Group 1 | $a$ | $b$ | $a+b$ |
| Group 2 | $c$ | $d$ | $c+d$ |
| Total | $a+c$ | $b+d$ | $N$ |

Under $H_0$ (no association):

$$P(a) = \frac{\binom{a+c}{a}\binom{b+d}{b}}{\binom{N}{a+b}}$$

This follows the Hypergeometric with $N = a+b+c+d$, $K = a+c$, $n = a+b$.

---

## 6. Multivariate Hypergeometric

When there are **more than two categories**:

$N$ items with $K_1$ of type 1, $K_2$ of type 2, ..., $K_m$ of type $m$.

Draw $n$ without replacement. $(X_1, X_2, \ldots, X_m)$ = counts of each type drawn.

$$P(X_1=k_1, \ldots, X_m=k_m) = \frac{\binom{K_1}{k_1}\binom{K_2}{k_2}\cdots\binom{K_m}{k_m}}{\binom{N}{n}}$$

### Example 4

A bag has 5 red, 7 blue, 3 green marbles. Draw 4 without replacement.

$$P(2 \text{ red}, 1 \text{ blue}, 1 \text{ green}) = \frac{\binom{5}{2}\binom{7}{1}\binom{3}{1}}{\binom{15}{4}} = \frac{10 \times 7 \times 3}{1365} = \frac{210}{1365} \approx 0.1538$$

---

## 7. Real-World Applications

| Scenario | $N$ | $K$ | $n$ |
|----------|-----|-----|-----|
| Card games | 52 | suits/ranks | hand size |
| Quality control | lot size | defectives | sample |
| Ecology (capture-recapture) | population | tagged | recaptured |
| Jury selection | eligible pool | minorities | jury size |
| Drug testing | tested athletes | dopers | selected |

---

## Summary Table

| Concept | Formula |
|---------|---------|
| PMF | $\frac{\binom{K}{k}\binom{N-K}{n-k}}{\binom{N}{n}}$ |
| Mean | $nK/N$ |
| Variance | $\frac{nK(N-K)(N-n)}{N^2(N-1)}$ |
| FPC | $(N-n)/(N-1)$ |
| Binomial approx. | When $n/N < 0.05$ |
| Fisher's test | Based on Hypergeometric |

---

## Quick Revision Questions

1. An urn has 8 red and 12 blue balls. You draw 5 without replacement. Find $P(\text{exactly 3 red})$.

2. A committee of 4 is randomly selected from 6 men and 9 women. Find the probability that the committee has exactly 2 men.

3. Compare $P(X = 2)$ for Hypergeometric($N=100, K=10, n=5$) and Binomial($n=5, p=0.1$). How close are they?

4. Explain why the FPC $(N-n)/(N-1)$ makes variance smaller for sampling without replacement. What happens when $n = N$?

5. In a capture-recapture study, 50 fish are tagged and released. Later, 30 fish are caught, and 8 are tagged. Estimate the total population $N$.

6. A shipment of 200 light bulbs contains 15 defective ones. An inspector tests 20 bulbs. What is the expected number of defective bulbs found? What is $P(\text{none defective})$?

---

[← Previous: Negative Binomial Distribution](05-negative-binomial.md) | [Back to README](../README.md) | [Next: Uniform Distribution →](../06-Continuous-Distributions/01-uniform-distribution.md)
