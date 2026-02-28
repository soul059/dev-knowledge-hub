# Chapter 3.4: Continuous Random Variables

[← Previous: Cumulative Distribution Function (CDF)](03-cdf.md) | [Back to README](../README.md) | [Next: Probability Density Function (PDF) →](05-pdf.md)

---

## Overview

A **continuous random variable** can take on any value within an interval (or union of intervals) of real numbers. Unlike discrete random variables, the probability at any single exact point is zero — probabilities are defined over intervals using integration.

---

## 1. Definition

A random variable $X$ is **continuous** if there exists a non-negative function $f_X(x)$ (called the **probability density function**) such that:

$$P(a \leq X \leq b) = \int_a^b f_X(x) \, dx$$

and the CDF is:

$$F_X(x) = P(X \leq x) = \int_{-\infty}^{x} f_X(t)\, dt$$

### Key Distinction from Discrete

| Property | Discrete | Continuous |
|----------|:--------:|:----------:|
| Values taken | Countable (list them) | Uncountable (intervals) |
| $P(X = x_0)$ | Can be $> 0$ | Always $= 0$ |
| Described by | PMF: $P(X = x)$ | PDF: $f(x)$ |
| Total prob | $\sum p(x) = 1$ | $\int f(x)\,dx = 1$ |
| $P(a < X < b)$ | $\sum_{a < x < b} p(x)$ | $\int_a^b f(x)\,dx$ |

```
  Discrete RV                    Continuous RV
  
  P(X=x)                         f(x)
  │                               │     ╱╲
  │  ▮                            │    ╱  ╲
  │  ▮  ▮                         │   ╱    ╲
  │  ▮  ▮  ▮                      │  ╱      ╲
  │  ▮  ▮  ▮  ▮                   │ ╱ Area=1  ╲
  └──┴──┴──┴──┴──► x              └────────────────► x
  Sum of heights = 1              Area under curve = 1
```

---

## 2. Why $P(X = x_0) = 0$ for Continuous RVs

For a continuous random variable:

$$P(X = x_0) = P(x_0 \leq X \leq x_0) = \int_{x_0}^{x_0} f(x)\, dx = 0$$

The integral over a single point (zero-width interval) is always zero.

### Consequence

$$P(a \leq X \leq b) = P(a < X < b) = P(a \leq X < b) = P(a < X \leq b)$$

Whether endpoints are included or excluded **doesn't matter** for continuous RVs!

```
  P(X = exact value) = 0  →  Why?
  
  Think of it this way:
  
  │        ╱╲
  │       ╱  ╲        The area of an infinitely
  │      ╱    ╲       thin line is ZERO
  │     ╱  ║   ╲
  │    ╱   ║    ╲     No matter how tall f(x₀) is,
  │   ╱    ║     ╲    the "width" is 0
  │  ╱     ║      ╲
  └────────║────────► x             
           x₀
           
  Area = height × width = f(x₀) × 0 = 0
```

---

## 3. CDF for Continuous RVs

The CDF is **continuous** (no jumps) and is the integral of the PDF:

$$F_X(x) = \int_{-\infty}^{x} f_X(t)\, dt$$

### Properties

| Property | Statement |
|----------|-----------|
| Continuous | $F_X(x)$ has no jumps |
| Smooth | $F_X'(x) = f_X(x)$ wherever $f$ is continuous |
| Limits | $F(-\infty) = 0$, $F(+\infty) = 1$ |
| Non-decreasing | $a < b \implies F(a) \leq F(b)$ |

### CDF → PDF

If $F_X(x)$ is differentiable:

$$f_X(x) = \frac{d}{dx} F_X(x) = F_X'(x)$$

```
  Relationship Between PDF and CDF (Continuous)
  
  PDF: f(x)                      CDF: F(x)
  │      ╱╲                      │                ─────── 1
  │     ╱  ╲                     │              ╱
  │    ╱    ╲                    │            ╱
  │   ╱      ╲                   │          ╱
  │  ╱        ╲                  │        ╱
  │ ╱          ╲                 │      ╱
  └──────────────► x             │    ╱
                                 │  ╱
  f(x) = F'(x)                  │╱
  Area = 1                       └──────────────► x
                                 F(x) = ∫f(t)dt  (from -∞ to x)
```

---

## 4. Examples of Continuous Random Variables

### Example 1: Uniform Distribution on $[a, b]$

$$f_X(x) = \begin{cases} \frac{1}{b-a} & a \leq x \leq b \\ 0 & \text{otherwise} \end{cases}$$

$$F_X(x) = \begin{cases} 0 & x < a \\ \frac{x-a}{b-a} & a \leq x \leq b \\ 1 & x > b \end{cases}$$

```
  Uniform PDF on [0, 1]          Uniform CDF on [0, 1]
  
  f(x)                           F(x)
  1.0 ┌──────────┐               1.0 │          ╱────
      │          │                    │        ╱
      │          │                    │      ╱
      │          │                    │    ╱
      │          │                    │  ╱
  0   └──────────┴──► x           0   │╱──────────► x
      0          1                    0          1
```

**Example:** $X \sim \text{Uniform}(0, 1)$. Find $P(0.2 \leq X \leq 0.7)$.

$$P(0.2 \leq X \leq 0.7) = \int_{0.2}^{0.7} 1 \, dx = 0.7 - 0.2 = 0.5$$

### Example 2: Exponential Distribution

$$f_X(x) = \begin{cases} \lambda e^{-\lambda x} & x \geq 0 \\ 0 & x < 0 \end{cases}$$

$$F_X(x) = \begin{cases} 1 - e^{-\lambda x} & x \geq 0 \\ 0 & x < 0 \end{cases}$$

### Example 3: Normal Distribution

$$f_X(x) = \frac{1}{\sigma\sqrt{2\pi}} e^{-\frac{(x-\mu)^2}{2\sigma^2}}, \quad -\infty < x < \infty$$

The famous bell curve — covered in detail in Unit 6.

---

## 5. Checking Validity of a PDF

A function $f(x)$ is a valid PDF if and only if:

1. $f(x) \geq 0$ for all $x$
2. $\int_{-\infty}^{\infty} f(x)\, dx = 1$

### Example 4

Is $f(x) = 3x^2$ for $0 \leq x \leq 1$ (and 0 otherwise) a valid PDF?

1. $f(x) = 3x^2 \geq 0$ for $x \in [0,1]$ ✓
2. $\int_0^1 3x^2 \, dx = [x^3]_0^1 = 1 - 0 = 1$ ✓

Yes, it's a valid PDF.

### Example 5

Is $f(x) = 2x$ for $0 \leq x \leq 2$ a valid PDF?

$\int_0^2 2x\, dx = [x^2]_0^2 = 4 \neq 1$ ✗ — **Not valid.**

To fix: $f(x) = x/2$ for $0 \leq x \leq 2$ → $\int_0^2 x/2\, dx = [x^2/4]_0^2 = 1$ ✓

---

## 6. Mixed Random Variables

Some random variables are **neither purely discrete nor purely continuous** — they have both "point mass" (discrete) and "continuous density" components.

**Example:** Insurance claims where $P(X = 0)$ = 0.8 (no claim) and $X$ has a continuous distribution for $X > 0$ (claim amount).

```
  Mixed Distribution CDF
  
  F(x)
  1.0 │                    ────────
      │                  ╱
  0.8 │───────●         ╱         ← Jump of 0.8 at x = 0
      │      ╱╱        ╱             (discrete component)
      │     ╱╱       ╱
      │    ╱╱      ╱              ← Smooth increase for x > 0
      │   ╱╱    ╱                    (continuous component)
  0.0 │──○╱──────────────────► x
         0
```

---

## 7. Real-World Applications

| Domain | Continuous Random Variable |
|--------|---------------------------|
| **Physics** | Measurement errors (normal), radioactive decay (exponential) |
| **Engineering** | Component lifetimes, signal strength |
| **Finance** | Stock returns, interest rate changes |
| **Biology** | Height, weight, blood pressure |
| **Manufacturing** | Part dimensions (target ± tolerance) |
| **Transportation** | Waiting time at traffic light |

---

## Summary Table

| Concept | Formula / Key Fact |
|---------|-------------------|
| Continuous RV | Takes values in an interval |
| PDF | $f(x) \geq 0$, $\int f(x)dx = 1$ |
| CDF | $F(x) = \int_{-\infty}^{x} f(t)dt$ |
| $P(a \leq X \leq b)$ | $\int_a^b f(x)dx = F(b) - F(a)$ |
| $P(X = x_0)$ | $= 0$ always |
| PDF from CDF | $f(x) = F'(x)$ |
| CDF shape | Continuous (no jumps), S-curve |
| Endpoints don't matter | $P(a \leq X \leq b) = P(a < X < b)$ |

---

## Quick Revision Questions

1. Why is $P(X = 5) = 0$ for a continuous random variable, even if $f(5) > 0$? Explain both mathematically and intuitively.

2. Given $f(x) = cx^3$ for $0 \leq x \leq 2$ (0 otherwise), find $c$ so that $f$ is a valid PDF.

3. For $f(x) = 2(1-x)$ on $[0,1]$, find $F(x)$ and compute $P(0.25 \leq X \leq 0.75)$.

4. A continuous RV has CDF $F(x) = 1 - e^{-2x}$ for $x \geq 0$. Find the PDF and $P(X > 1)$.

5. How does the CDF of a continuous RV differ visually from the CDF of a discrete RV?

6. Give an example of a real-world quantity that is best modeled as (a) discrete, (b) continuous, (c) mixed.

---

[← Previous: Cumulative Distribution Function (CDF)](03-cdf.md) | [Back to README](../README.md) | [Next: Probability Density Function (PDF) →](05-pdf.md)
