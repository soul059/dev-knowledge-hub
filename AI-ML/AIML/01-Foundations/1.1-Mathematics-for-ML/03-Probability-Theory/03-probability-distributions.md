# Chapter 3: Probability Distributions

> **Unit 3: Probability Theory** | **Topic**: Common Discrete & Continuous Distributions Overview

---

## 1. Overview

A probability distribution describes how the values of a random variable are spread or distributed.
Choosing the right distribution to model data is a core skill in machine learning — it affects
everything from model selection to loss function design. This chapter surveys the most important
discrete and continuous distributions, their formulas, shapes, and use cases.

---

## 2. Discrete Distributions

### 2.1 Discrete Uniform Distribution

Every outcome has equal probability.

```
P(X = x) = 1/n   for x ∈ {x₁, x₂, ..., xₙ}

Mean:     μ = (a + b) / 2
Variance: σ² = (n² - 1) / 12     (for values 1 to n)
```

**Example**: Fair die → P(X = k) = 1/6 for k = 1,...,6

```
p(x)
1/6 │  █    █    █    █    █    █
    └──1────2────3────4────5────6──→ x
```

**ML Use**: Random initialization, random class assignment baseline.

---

### 2.2 Bernoulli Distribution

A single trial with two outcomes (success/failure).

```
X ~ Bernoulli(p)

P(X = 1) = p        (success)
P(X = 0) = 1 - p    (failure)

Compact: P(X = x) = pˣ(1-p)¹⁻ˣ   for x ∈ {0, 1}

Mean:     μ = p
Variance: σ² = p(1-p)
```

**ML Use**: Binary classification labels, coin flip model, logistic regression output.

---

### 2.3 Binomial Distribution

Number of successes in **n** independent Bernoulli trials.

```
X ~ Binomial(n, p)

P(X = k) = C(n,k) · pᵏ · (1-p)ⁿ⁻ᵏ    for k = 0, 1, ..., n

where C(n,k) = n! / (k!(n-k)!)

Mean:     μ = np
Variance: σ² = np(1-p)
```

**Example**: n=10 coin flips, p=0.5

```
P(X=k) for Binomial(10, 0.5):

p(k)
0.25│         █
    │       █ █ █
0.20│     █ █ █ █ █
    │   █ █ █ █ █ █
0.10│ █ █ █ █ █ █ █ █
    │ █ █ █ █ █ █ █ █ █
    └─0─1─2─3─4─5─6─7─8─9─10→ k
```

**ML Use**: Number of correct predictions in a batch, A/B testing.

---

### 2.4 Poisson Distribution

Models the count of events in a fixed interval (time/space).

```
X ~ Poisson(λ)

P(X = k) = (λᵏ · e⁻λ) / k!    for k = 0, 1, 2, ...

Mean:     μ = λ
Variance: σ² = λ
```

**Key property**: Mean = Variance = λ

```
Poisson(λ=3):

p(k)
0.22│     █
    │   █ █
0.15│ █ █ █ █
    │ █ █ █ █ █
0.05│ █ █ █ █ █ █ █
    └─0─1─2─3─4─5─6─7─8→ k
```

**ML Use**: Modeling rare events — click counts, word frequencies, network failures.

---

### 2.5 Geometric Distribution

Number of trials until the **first** success.

```
X ~ Geometric(p)

P(X = k) = (1-p)ᵏ⁻¹ · p    for k = 1, 2, 3, ...

Mean:     μ = 1/p
Variance: σ² = (1-p)/p²
```

**ML Use**: Modeling time-to-event, number of attempts before convergence.

---

## 3. Continuous Distributions

### 3.1 Continuous Uniform Distribution

```
X ~ Uniform(a, b)

f(x) = 1/(b-a)    for a ≤ x ≤ b
       0           otherwise

Mean:     μ = (a+b)/2
Variance: σ² = (b-a)²/12
```

```
f(x)  Uniform(0, 1)
  1 │ ┌──────────────────┐
    │ │██████████████████│
  0 └─┴──────────────────┴──→ x
    0                     1
```

**ML Use**: Random weight initialization (uniform variant), prior distribution.

---

### 3.2 Normal (Gaussian) Distribution

The most important distribution in statistics and ML.

```
X ~ N(μ, σ²)

f(x) = (1 / (σ√(2π))) · exp(-(x-μ)² / (2σ²))

Mean:     μ
Variance: σ²
```

#### The 68-95-99.7 Rule

```
f(x)     Standard Normal N(0,1)
    │            ╱╲
    │           ╱  ╲
    │          ╱    ╲
    │         ╱  68% ╲
    │        ╱   ┊    ╲
    │      ╱─────┊─────╲
    │    ╱──── 95% ───────╲
    │  ╱────── 99.7% ────────╲
    └──┴──┴──┴──┴──┴──┴──┴──┴──→ x
      -3σ -2σ -1σ  μ  1σ  2σ  3σ
```

**ML Use**: Assumed distribution for errors, weight initialization, Gaussian Naive Bayes,
Gaussian processes, normalization layers.

---

### 3.3 Exponential Distribution

Models the time between events in a Poisson process.

```
X ~ Exponential(λ)

f(x) = λ · e⁻λˣ     for x ≥ 0

Mean:     μ = 1/λ
Variance: σ² = 1/λ²
```

```
f(x)  Exponential (different λ)
    │╲
    │ ╲  λ=2
    │  ╲
    │   ╲── λ=1
    │    ╲────
    │     ╲──────  λ=0.5
    │      ╲────────────
    └──────────────────────→ x
    0
```

**Memoryless property**: P(X > s + t | X > s) = P(X > t)

**ML Use**: Modeling time between user actions, survival analysis, decay rates.

---

## 4. Choosing the Right Distribution

```
Is the variable...

Discrete?
  ├─ Two outcomes? ─────────→ Bernoulli
  ├─ # successes in n trials? → Binomial
  ├─ Count of rare events? ──→ Poisson
  ├─ Trials until success? ──→ Geometric
  └─ Equal probability? ─────→ Discrete Uniform

Continuous?
  ├─ Bounded interval? ──────→ Continuous Uniform
  ├─ Bell-shaped / symmetric? → Normal (Gaussian)
  ├─ Time between events? ───→ Exponential
  └─ Positive, right-skewed? → Gamma or Log-Normal
```

---

## 5. Python Code: Distribution Exploration

```python
import numpy as np
from scipy import stats

# --- Binomial ---
n, p = 10, 0.3
X = stats.binom(n, p)
print(f"Binomial({n},{p}): Mean={X.mean():.2f}, Var={X.var():.2f}")
print(f"P(X=3) = {X.pmf(3):.4f}")
print(f"P(X≤3) = {X.cdf(3):.4f}")

# --- Poisson ---
lam = 5
Y = stats.poisson(lam)
print(f"\nPoisson({lam}): Mean={Y.mean():.2f}, Var={Y.var():.2f}")
print(f"P(Y=3) = {Y.pmf(3):.4f}")

# --- Normal ---
mu, sigma = 0, 1
Z = stats.norm(mu, sigma)
print(f"\nNormal({mu},{sigma}²): P(-1 < Z < 1) = {Z.cdf(1)-Z.cdf(-1):.4f}")  # ~0.6827
print(f"P(-2 < Z < 2) = {Z.cdf(2)-Z.cdf(-2):.4f}")                          # ~0.9545

# --- Exponential ---
lam_exp = 2.0
W = stats.expon(scale=1/lam_exp)
print(f"\nExponential(λ={lam_exp}): Mean={W.mean():.2f}, Var={W.var():.4f}")
```

```python
# Generate and compare samples
samples_normal = np.random.normal(0, 1, 10000)
samples_poisson = np.random.poisson(5, 10000)

print(f"Normal samples  — Mean: {samples_normal.mean():.3f}, Std: {samples_normal.std():.3f}")
print(f"Poisson samples — Mean: {samples_poisson.mean():.3f}, Std: {samples_poisson.std():.3f}")
```

---

## 6. Real-World ML Applications

| Distribution  | ML Application                                          |
|---------------|--------------------------------------------------------|
| Bernoulli     | Binary classification labels, dropout mask              |
| Binomial      | Batch accuracy counts, A/B test analysis               |
| Poisson       | NLP word counts, event-driven features                 |
| Gaussian      | Weight init, noise modeling, Gaussian processes, GMMs  |
| Exponential   | Session duration modeling, survival analysis           |
| Uniform       | Random search hyperparameter tuning                    |

---

## 7. Summary Table

| Distribution | Type     | PMF / PDF                              | Mean   | Variance    | Support        |
|-------------|----------|----------------------------------------|--------|-------------|----------------|
| Uniform(n)  | Discrete | 1/n                                    | (n+1)/2| (n²−1)/12   | {1,...,n}       |
| Bernoulli(p)| Discrete | pˣ(1−p)¹⁻ˣ                            | p      | p(1−p)      | {0, 1}         |
| Binomial    | Discrete | C(n,k)pᵏ(1−p)ⁿ⁻ᵏ                     | np     | np(1−p)     | {0,...,n}       |
| Poisson(λ)  | Discrete | λᵏe⁻λ/k!                              | λ      | λ           | {0,1,2,...}     |
| Geometric(p)| Discrete | (1−p)ᵏ⁻¹p                             | 1/p    | (1−p)/p²    | {1,2,3,...}     |
| Uniform(a,b)| Contin.  | 1/(b−a)                                | (a+b)/2| (b−a)²/12   | [a, b]         |
| Normal      | Contin.  | (1/σ√2π)exp(−(x−μ)²/2σ²)              | μ      | σ²          | (−∞, ∞)        |
| Exponential | Contin.  | λe⁻λˣ                                 | 1/λ    | 1/λ²        | [0, ∞)         |

---

## 8. Quick Revision Questions

1. **Which distribution models the number of heads in 20 fair coin flips?** State its parameters.
2. **In Poisson distribution, what is special about the mean and variance?**
3. **What is the memoryless property?** Which distribution has it?
4. **Write the PDF of N(5, 4).** What is P(X = 5) for this distribution?
5. **A website gets 3 hits/minute on average.** Which distribution models hits per minute?
6. **Why is the Gaussian distribution so prevalent in ML?** (Hint: CLT)

---

## 9. Navigation

| ← Previous | Next → |
|:-----------:|:------:|
| [Chapter 2: Random Variables](02-random-variables.md) | [Chapter 4: Expectation & Variance](04-expectation-and-variance.md) |

---

*Unit 3 · Probability Theory · Chapter 3 of 9*
