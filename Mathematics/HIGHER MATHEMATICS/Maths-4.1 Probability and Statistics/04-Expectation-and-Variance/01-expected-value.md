# Chapter 4.1: Expected Value

[← Previous: Probability Density Function (PDF)](../03-Random-Variables/05-pdf.md) | [Back to README](../README.md) | [Next: Properties of Expectation →](02-properties-of-expectation.md)

---

## Overview

The **expected value** (or **expectation** or **mean**) of a random variable is its long-run average. It is the single most important summary number for a probability distribution — the "center of gravity" of the distribution.

---

## 1. Definition

### Discrete Random Variable

$$\boxed{E[X] = \mu_X = \sum_{x \in R_X} x \cdot p_X(x) = \sum_{x} x \cdot P(X = x)}$$

### Continuous Random Variable

$$\boxed{E[X] = \mu_X = \int_{-\infty}^{\infty} x \cdot f_X(x)\, dx}$$

### Intuition

The expected value is the **weighted average** of all possible values, weighted by their probabilities.

```
  Expected Value = "Center of Gravity"
  
  Discrete:                    Continuous:
  
  p(x)                         f(x)
  │  ▮                         │      ╱╲
  │  ▮  ▮                      │    ╱    ╲
  │  ▮  ▮  ▮                   │  ╱   ▲    ╲
  │  ▮  ▮  ▮  ▮                │╱     │      ╲
  └──┴──┴──┴──┴──► x           └──────┼────────► x
        ▲                             │
        │                             E[X]
       E[X]                     Center of mass
  Weighted average               of the curve
```

---

## 2. Examples

### Example 1: Fair Die

$X$ = die roll, $P(X = k) = 1/6$ for $k = 1, 2, 3, 4, 5, 6$

$$E[X] = \sum_{k=1}^{6} k \cdot \frac{1}{6} = \frac{1+2+3+4+5+6}{6} = \frac{21}{6} = 3.5$$

Note: $E[X] = 3.5$ is NOT a possible outcome! The expected value doesn't have to be a realizable value.

### Example 2: Bernoulli Trial

$X \in \{0, 1\}$ with $P(X = 1) = p$, $P(X = 0) = 1-p$

$$E[X] = 0 \cdot (1-p) + 1 \cdot p = p$$

### Example 3: Exponential Distribution

$f(x) = \lambda e^{-\lambda x}$ for $x \geq 0$

$$E[X] = \int_0^{\infty} x \cdot \lambda e^{-\lambda x}\, dx = \frac{1}{\lambda}$$

(Using integration by parts)

### Example 4: Uniform Distribution on $[a, b]$

$$E[X] = \int_a^b x \cdot \frac{1}{b-a}\, dx = \frac{1}{b-a} \cdot \frac{x^2}{2}\Bigg|_a^b = \frac{b^2 - a^2}{2(b-a)} = \frac{a + b}{2}$$

---

## 3. Expected Value of a Function of $X$ (LOTUS)

**Law of the Unconscious Statistician (LOTUS):**

### Discrete

$$E[g(X)] = \sum_{x} g(x) \cdot p_X(x)$$

### Continuous

$$E[g(X)] = \int_{-\infty}^{\infty} g(x) \cdot f_X(x)\, dx$$

> You do **NOT** need to find the distribution of $Y = g(X)$ first — just apply $g$ inside the expectation.

### Example 5

$X$ = die roll. Find $E[X^2]$.

$$E[X^2] = \sum_{k=1}^{6} k^2 \cdot \frac{1}{6} = \frac{1+4+9+16+25+36}{6} = \frac{91}{6} \approx 15.17$$

---

## 4. The Gambling Interpretation

Expected value tells you the **fair price** of a game:

- If $E[\text{winnings}] > \text{cost}$, the game favors you
- If $E[\text{winnings}] < \text{cost}$, the house has the edge
- If $E[\text{winnings}] = \text{cost}$, it's a "fair game"

### Example 6: Lottery

A lottery ticket costs \$2. Prizes:
- \$1,000,000 with probability 1/10,000,000
- \$100 with probability 1/100,000
- \$0 otherwise

$$E[\text{winnings}] = 1{,}000{,}000 \cdot \frac{1}{10{,}000{,}000} + 100 \cdot \frac{1}{100{,}000} = 0.10 + 0.001 = \$0.101$$

Since $E = \$0.101 < \$2.00$ (cost), the expected loss is $\$1.899$ per ticket.

---

## 5. Existence of Expected Value

The expected value exists only if the sum/integral converges absolutely:

$$\sum |x| \cdot p(x) < \infty \quad \text{or} \quad \int |x| \cdot f(x)\, dx < \infty$$

### Cauchy Distribution — No Mean!

$$f(x) = \frac{1}{\pi(1+x^2)}, \quad x \in \mathbb{R}$$

$E[X] = \int_{-\infty}^{\infty} \frac{x}{\pi(1+x^2)} dx$ does **not converge absolutely** → $E[X]$ is undefined.

---

## 6. Expected Value for Common Distributions

| Distribution | $E[X]$ |
|:------------|:------:|
| Bernoulli($p$) | $p$ |
| Binomial($n, p$) | $np$ |
| Poisson($\lambda$) | $\lambda$ |
| Geometric($p$) | $1/p$ |
| Uniform($a, b$) discrete | $(a+b)/2$ |
| Uniform($a, b$) continuous | $(a+b)/2$ |
| Exponential($\lambda$) | $1/\lambda$ |
| Normal($\mu, \sigma^2$) | $\mu$ |
| Gamma($\alpha, \beta$) | $\alpha/\beta$ |

---

## 7. Real-World Applications

| Domain | Use of Expected Value |
|--------|----------------------|
| **Insurance** | Expected claim = basis for premium pricing |
| **Finance** | Expected return on investment |
| **Gambling** | House edge calculation |
| **Manufacturing** | Expected defects per batch |
| **Decision Theory** | Expected utility of each option |
| **Queuing Theory** | Expected waiting time |

---

## Summary Table

| Concept | Formula |
|---------|---------|
| $E[X]$ (discrete) | $\sum x \cdot p(x)$ |
| $E[X]$ (continuous) | $\int x \cdot f(x)\, dx$ |
| LOTUS | $E[g(X)] = \sum g(x) p(x)$ or $\int g(x) f(x) dx$ |
| Fair die | $E[X] = 3.5$ |
| Bernoulli | $E[X] = p$ |
| Exponential | $E[X] = 1/\lambda$ |
| Fair game | $E[\text{profit}] = 0$ |
| Existence | Requires absolute convergence |

---

## Quick Revision Questions

1. A random variable has the distribution: $P(X = -1) = 0.2$, $P(X = 0) = 0.3$, $P(X = 1) = 0.4$, $P(X = 2) = 0.1$. Find $E[X]$ and $E[X^2]$.

2. For $X \sim \text{Uniform}(0, 10)$, find $E[X]$, $E[X^2]$, and $E[3X + 5]$.

3. A game costs \$5 to play. You roll a die: if you get 6, you win \$20; otherwise you win nothing. Should you play? What is the expected profit?

4. **Derive** $E[X] = 1/\lambda$ for the exponential distribution using integration by parts.

5. Why does the expected value of a fair die (3.5) not correspond to any actual outcome? Is this a problem?

6. The Cauchy distribution has no mean. Give an intuitive explanation of why its "average" doesn't settle down even with millions of samples.

---

[← Previous: Probability Density Function (PDF)](../03-Random-Variables/05-pdf.md) | [Back to README](../README.md) | [Next: Properties of Expectation →](02-properties-of-expectation.md)
