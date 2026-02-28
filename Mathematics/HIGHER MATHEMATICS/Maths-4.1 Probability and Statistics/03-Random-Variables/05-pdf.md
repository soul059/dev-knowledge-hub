# Chapter 3.5: Probability Density Function (PDF)

[← Previous: Continuous Random Variables](04-continuous-random-variables.md) | [Back to README](../README.md) | [Next: Expected Value →](../04-Expectation-and-Variance/01-expected-value.md)

---

## Overview

The **Probability Density Function (PDF)** is the continuous analog of the PMF. While the PMF gives probabilities directly, the PDF gives **probability density** — probability per unit length. Probabilities are obtained by **integrating** the PDF over intervals.

---

## 1. Formal Definition

A function $f_X(x)$ is the PDF of continuous random variable $X$ if:

$$\boxed{P(a \leq X \leq b) = \int_a^b f_X(x)\, dx}$$

### Required Properties

| Property | Condition |
|----------|-----------|
| Non-negativity | $f_X(x) \geq 0$ for all $x$ |
| Total area = 1 | $\int_{-\infty}^{\infty} f_X(x)\, dx = 1$ |

### Important Notes

- $f(x)$ is **NOT** a probability — it can be $> 1$!
- $f(x)\, dx$ is an infinitesimal probability: $P(x \leq X \leq x + dx) \approx f(x)\, dx$
- Probability = area under the curve between two points

```
  PDF Interpretation
  
  f(x)
  2.0 │    ╱╲                f(x) can exceed 1!
      │   ╱  ╲               (it's density, not probability)
  1.5 │  ╱    ╲
      │ ╱      ╲
  1.0 │╱  ████  ╲            Shaded area = P(a ≤ X ≤ b)
      │   ████   ╲
  0.5 │   ████    ╲
      │   ████     ╲
  0.0 └───████──────╲──► x
          a    b
  
  P(a ≤ X ≤ b) = ∫ₐᵇ f(x) dx = Shaded Area
```

---

## 2. Relationship Between PDF and CDF

$$F_X(x) = \int_{-\infty}^{x} f_X(t)\, dt \qquad \Longleftrightarrow \qquad f_X(x) = \frac{dF_X}{dx}$$

```
  PDF ─────── Integration ────────► CDF
       (accumulate area)
       
  PDF ◄────── Differentiation ───── CDF
       (rate of change)
```

---

## 3. Standard PDF Examples

### 3.1 Uniform PDF on $[a, b]$

$$f(x) = \begin{cases} \frac{1}{b-a} & a \leq x \leq b \\ 0 & \text{otherwise}\end{cases}$$

```
  f(x)
  1/(b-a) │ ┌────────────┐
          │ │            │
          │ │  Area = 1  │
          │ │            │
      0   └─┴────────────┴──► x
            a              b
```

**Example:** $X \sim \text{Uniform}(2, 5)$

$f(x) = 1/(5-2) = 1/3$ for $2 \leq x \leq 5$

$P(3 \leq X \leq 4) = \int_3^4 \frac{1}{3} dx = \frac{1}{3}$

### 3.2 Exponential PDF

$$f(x) = \begin{cases} \lambda e^{-\lambda x} & x \geq 0 \\ 0 & x < 0\end{cases}$$

```
  f(x) for λ = 1
  
  1.0 │╲
      │ ╲
  0.8 │  ╲
      │   ╲
  0.6 │    ╲
      │     ╲
  0.4 │      ╲
      │       ╲╲
  0.2 │         ╲╲
      │           ╲╲╲
  0.0 └──────────────╲╲╲───► x
      0   1   2   3   4   5
  
  Rapid decay — models "time until event"
```

### 3.3 Normal (Gaussian) PDF

$$f(x) = \frac{1}{\sigma\sqrt{2\pi}} \exp\!\left(-\frac{(x-\mu)^2}{2\sigma^2}\right)$$

```
  Standard Normal PDF (μ=0, σ=1)
  
  f(x)
  0.4 │        ╱╲
      │       ╱  ╲
  0.3 │      ╱    ╲
      │     ╱      ╲
  0.2 │    ╱        ╲
      │   ╱          ╲
  0.1 │  ╱            ╲
      │ ╱              ╲
  0.0 └╱────────────────╲──► x
     -3  -2  -1   0   1   2   3
  
  Symmetric bell curve
  68% within ±1σ, 95% within ±2σ, 99.7% within ±3σ
```

### 3.4 Triangular PDF

$$f(x) = \begin{cases} \frac{2x}{b^2} & 0 \leq x \leq b \\ 0 & \text{otherwise}\end{cases}$$

```
  Triangular PDF on [0, b]
  
  f(x)
  2/b │           ╱
      │         ╱
      │       ╱
      │     ╱
      │   ╱   Area = 1
      │ ╱
  0   └╱─────────────► x
      0               b
```

---

## 4. Verifying a PDF

### Worked Example

**Given:** $f(x) = \begin{cases} c(4x - 2x^2) & 0 \leq x \leq 2 \\ 0 & \text{otherwise}\end{cases}$

**Step 1:** Check non-negativity.

$4x - 2x^2 = 2x(2-x) \geq 0$ for $x \in [0, 2]$ ✓

**Step 2:** Find $c$ from normalization.

$$\int_0^2 c(4x - 2x^2)\, dx = c\left[2x^2 - \frac{2x^3}{3}\right]_0^2 = c\left(8 - \frac{16}{3}\right) = c \cdot \frac{8}{3} = 1$$

$$c = \frac{3}{8}$$

**Step 3:** Find $P(1 \leq X \leq 2)$.

$$P(1 \leq X \leq 2) = \frac{3}{8}\int_1^2 (4x - 2x^2)\, dx = \frac{3}{8}\left[2x^2 - \frac{2x^3}{3}\right]_1^2$$

$$= \frac{3}{8}\left[\left(8 - \frac{16}{3}\right) - \left(2 - \frac{2}{3}\right)\right] = \frac{3}{8}\left[\frac{8}{3} - \frac{4}{3}\right] = \frac{3}{8} \cdot \frac{4}{3} = \frac{1}{2}$$

---

## 5. Mode, Median, and Percentiles from PDF

### Mode

The value $x$ where $f(x)$ is **maximum**:

$$f'(x_{\text{mode}}) = 0, \quad f''(x_{\text{mode}}) < 0$$

### Median

The value $m$ where:

$$\int_{-\infty}^{m} f(x)\, dx = 0.5 \quad \Longleftrightarrow \quad F(m) = 0.5$$

### $p$-th Percentile

$$F(x_p) = p$$

---

## 6. Transformations of Continuous RVs

If $Y = g(X)$ where $g$ is monotonic and differentiable:

### Monotonically Increasing $g$

$$f_Y(y) = f_X\!\left(g^{-1}(y)\right) \cdot \left|\frac{d}{dy}g^{-1}(y)\right|$$

### General Formula

$$\boxed{f_Y(y) = f_X(g^{-1}(y)) \cdot \left|\frac{dx}{dy}\right|}$$

### Example: Linear Transformation

If $Y = aX + b$ (where $a \neq 0$):

$$f_Y(y) = \frac{1}{|a|} f_X\!\left(\frac{y - b}{a}\right)$$

---

## 7. Comparison: PMF vs. PDF

| Feature | PMF (Discrete) | PDF (Continuous) |
|---------|:--------------:|:----------------:|
| Notation | $p_X(x) = P(X=x)$ | $f_X(x)$ |
| Value meaning | Probability directly | Density (prob per unit) |
| Can exceed 1? | No ($0 \leq p \leq 1$) | Yes ($f(x) > 1$ possible) |
| Probability at a point | $P(X=x) = p(x) > 0$ possible | $P(X=x) = 0$ always |
| Total | $\sum p(x) = 1$ | $\int f(x)dx = 1$ |
| Prob of interval | $\sum_{a \leq x \leq b} p(x)$ | $\int_a^b f(x)dx$ |
| CDF relationship | $F(x) = \sum_{t \leq x} p(t)$ | $F(x) = \int_{-\infty}^{x} f(t)dt$ |

---

## 8. Real-World Applications

| PDF Model | Application |
|-----------|------------|
| Uniform | Random number generators, round-off errors |
| Exponential | Time between events (calls, failures, arrivals) |
| Normal | Measurement errors, natural phenomena, grades |
| Log-normal | Stock prices, income distributions |
| Beta | Bayesian prior for probabilities |
| Weibull | Lifetime/reliability analysis |

---

## Summary Table

| Concept | Formula / Key Fact |
|---------|-------------------|
| PDF definition | $P(a \leq X \leq b) = \int_a^b f(x)dx$ |
| Valid PDF | $f(x) \geq 0$ and $\int f(x)dx = 1$ |
| $f(x)$ can be $>1$ | Yes — it's density, not probability |
| CDF from PDF | $F(x) = \int_{-\infty}^{x} f(t)dt$ |
| PDF from CDF | $f(x) = F'(x)$ |
| $P(X = x_0) = 0$ | Single points have zero probability |
| Transformation | $f_Y(y) = f_X(g^{-1}(y)) \cdot |dx/dy|$ |
| Median | $F(m) = 0.5$ |
| Mode | $f'(x) = 0$, $f''(x) < 0$ |

---

## Quick Revision Questions

1. A continuous RV has $f(x) = kx$ for $0 \leq x \leq 4$. Find $k$ and $P(X > 2)$.

2. Explain why $f(x) = 3$ for $0 \leq x \leq 1/3$ is a valid PDF even though $f(x) > 1$. What does $f(x) = 3$ mean probabilistically?

3. Given $f(x) = 2e^{-2x}$ for $x \geq 0$, find the CDF $F(x)$, the median, and $P(X > 1)$.

4. If $X \sim \text{Uniform}(0,1)$ and $Y = X^2$, find the PDF of $Y$.

5. **Prove** that for any valid PDF, $\int_{-\infty}^{\infty} f(x)dx = 1$ is sufficient (with $f \geq 0$) to guarantee all probability axioms are satisfied.

6. A PDF is given as $f(x) = a + bx$ for $0 \leq x \leq 1$. If $E[X] = 2/3$, find $a$ and $b$.

---

[← Previous: Continuous Random Variables](04-continuous-random-variables.md) | [Back to README](../README.md) | [Next: Expected Value →](../04-Expectation-and-Variance/01-expected-value.md)
