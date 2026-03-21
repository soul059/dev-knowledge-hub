# Chapter 4: Expectation and Variance

> **Unit 3: Probability Theory** | **Topic**: Expected Value, Variance, Covariance, LLN, CLT

---

## 1. Overview

Expectation and variance are the two most fundamental summary statistics of a probability distribution.
The expected value tells us the "center" of a distribution, while variance measures its "spread." These
concepts are deeply embedded in ML — from defining loss functions (expected risk) to understanding
optimization (gradient variance) and convergence guarantees.

---

## 2. Expected Value (Mean)

### 2.1 Definition

The **expected value** (or mean) of a random variable X is its long-run average:

```
Discrete:    E[X] = Σ_x  x · P(X = x)
Continuous:  E[X] = ∫_{-∞}^{∞} x · f(x) dx
```

### 2.2 Example: Fair Die

```
E[X] = 1·(1/6) + 2·(1/6) + 3·(1/6) + 4·(1/6) + 5·(1/6) + 6·(1/6)
     = 21/6
     = 3.5
```

Note: E[X] = 3.5, which is NOT a possible outcome. The expected value need not be a value X can take.

### 2.3 Expected Value of a Function

```
E[g(X)] = Σ_x g(x) · P(X = x)          (discrete)
E[g(X)] = ∫ g(x) · f(x) dx             (continuous)
```

---

## 3. Properties of Expectation

```
1. Linearity:        E[aX + b]  = a·E[X] + b
2. Sum:              E[X + Y]   = E[X] + E[Y]           (ALWAYS, even if dependent!)
3. Product (indep.): E[X · Y]   = E[X] · E[Y]           (only if X ⊥ Y)
4. Constant:         E[c]       = c
```

### Linearity of Expectation — Powerful Tool

**Problem**: What is the expected number of fixed points in a random permutation of n elements?

```
Let Xᵢ = 1 if element i is in position i, 0 otherwise.
E[Xᵢ] = 1/n

Total fixed points: X = X₁ + X₂ + ... + Xₙ
E[X] = E[X₁] + E[X₂] + ... + E[Xₙ] = n · (1/n) = 1

Answer: 1 fixed point on average, regardless of n!
```

---

## 4. Variance

### 4.1 Definition

Variance measures how far values deviate from the mean:

```
Var(X) = E[(X - μ)²] = E[X²] - (E[X])²

Discrete:    Var(X) = Σ_x (x - μ)² · P(X = x)
Continuous:  Var(X) = ∫ (x - μ)² · f(x) dx
```

The **computational formula** is often easier:

```
Var(X) = E[X²] - (E[X])²
```

### 4.2 Example: Fair Die

```
E[X²] = 1²·(1/6) + 2²·(1/6) + ... + 6²·(1/6)
       = (1 + 4 + 9 + 16 + 25 + 36)/6
       = 91/6 ≈ 15.167

Var(X) = E[X²] - (E[X])²
       = 91/6 - (3.5)²
       = 91/6 - 12.25
       = 2.917
```

---

## 5. Properties of Variance

```
1. Var(X) ≥ 0                              (always non-negative)
2. Var(c) = 0                              (constant has no spread)
3. Var(aX + b) = a² · Var(X)               (scaling squares, shift ignored)
4. Var(X + Y) = Var(X) + Var(Y)            (only if X ⊥ Y)
5. Var(X + Y) = Var(X) + Var(Y) + 2Cov(X,Y)  (general case)
```

---

## 6. Standard Deviation

```
σ = √Var(X) = √(E[(X - μ)²])
```

Standard deviation has the **same units** as X, making it more interpretable than variance.

---

## 7. Covariance

### 7.1 Definition

Covariance measures how two random variables vary together:

```
Cov(X, Y) = E[(X - μₓ)(Y - μᵧ)] = E[XY] - E[X]·E[Y]
```

### 7.2 Interpretation

```
Cov(X,Y) > 0  →  X and Y tend to increase together
Cov(X,Y) < 0  →  When X increases, Y tends to decrease
Cov(X,Y) = 0  →  No linear relationship (NOT necessarily independent!)
```

### 7.3 Correlation Coefficient

Normalized covariance (dimensionless, range [-1, 1]):

```
ρ(X,Y) = Cov(X,Y) / (σₓ · σᵧ)

ρ = +1  →  perfect positive linear relationship
ρ = -1  →  perfect negative linear relationship
ρ =  0  →  no linear relationship
```

### 7.4 Properties

```
Cov(X, X) = Var(X)
Cov(X, Y) = Cov(Y, X)
Cov(aX, bY) = ab · Cov(X, Y)
Cov(X + Z, Y) = Cov(X, Y) + Cov(Z, Y)
```

---

## 8. Law of Large Numbers (LLN)

As the number of samples n → ∞, the sample mean converges to the expected value:

```
X̄ₙ = (X₁ + X₂ + ... + Xₙ) / n  →  E[X]   as n → ∞
```

### Intuition

```
n=10:    X̄ = 3.2    (noisy)
n=100:   X̄ = 3.48   (closer)
n=1000:  X̄ = 3.507  (very close)
n=10000: X̄ = 3.4998 (≈ 3.5)     ← True E[X] for fair die
```

**ML Relevance**: Justifies using empirical averages (training loss, accuracy) as estimates
of true expected values.

---

## 9. Central Limit Theorem (CLT)

The sum (or average) of many independent random variables tends toward a **normal distribution**,
regardless of the original distribution:

```
If X₁, X₂, ..., Xₙ are i.i.d. with mean μ and variance σ²:

X̄ₙ ≈ N(μ, σ²/n)   for large n

Equivalently: (X̄ₙ - μ) / (σ/√n) → N(0, 1)
```

### Why CLT Matters in ML

```
┌──────────────────────────────────────────────────┐
│  Original distribution         Sampling dist.    │
│  (any shape!)                  of X̄ₙ             │
│                                                  │
│  ████                              ╱╲            │
│  ██████                          ╱    ╲          │
│  ████████        n→∞           ╱  N(μ, ╲        │
│  ██████████      ────→       ╱  σ²/n)   ╲       │
│  ████████████                ──────────────      │
│                                                  │
│  (Uniform, Exponential,      Always Gaussian!    │
│   Poisson, anything!)                            │
└──────────────────────────────────────────────────┘
```

---

## 10. Python Simulations

### 10.1 Law of Large Numbers

```python
import numpy as np
import matplotlib.pyplot as plt

np.random.seed(42)
# Rolling a fair die — E[X] = 3.5
rolls = np.random.randint(1, 7, size=10000)
running_mean = np.cumsum(rolls) / np.arange(1, 10001)

plt.figure(figsize=(10, 4))
plt.plot(running_mean, label='Running Mean')
plt.axhline(y=3.5, color='r', linestyle='--', label='E[X]=3.5')
plt.xlabel('Number of Rolls')
plt.ylabel('Sample Mean')
plt.title('Law of Large Numbers: Die Rolls')
plt.legend()
plt.show()
```

### 10.2 Central Limit Theorem

```python
np.random.seed(42)
# Exponential distribution (clearly non-normal) with λ=1 → mean=1, var=1
sample_means = []
for _ in range(5000):
    sample = np.random.exponential(1.0, size=50)
    sample_means.append(np.mean(sample))

plt.figure(figsize=(8, 4))
plt.hist(sample_means, bins=50, density=True, alpha=0.7, label='Sample Means')

# Overlay theoretical normal
from scipy.stats import norm
x = np.linspace(0.5, 1.5, 200)
plt.plot(x, norm.pdf(x, 1.0, 1.0/np.sqrt(50)), 'r-', lw=2, label='N(1, 1/50)')
plt.title('CLT: Means of Exponential Samples → Normal')
plt.legend()
plt.show()
```

### 10.3 Covariance and Correlation

```python
np.random.seed(42)
n = 1000
x = np.random.randn(n)
y_pos = 2*x + np.random.randn(n) * 0.5    # Positive correlation
y_neg = -1.5*x + np.random.randn(n) * 0.5 # Negative correlation
y_none = np.random.randn(n)                # No correlation

print(f"Positive: Cov={np.cov(x, y_pos)[0,1]:.3f}, ρ={np.corrcoef(x, y_pos)[0,1]:.3f}")
print(f"Negative: Cov={np.cov(x, y_neg)[0,1]:.3f}, ρ={np.corrcoef(x, y_neg)[0,1]:.3f}")
print(f"None:     Cov={np.cov(x, y_none)[0,1]:.3f}, ρ={np.corrcoef(x, y_none)[0,1]:.3f}")
```

---

## 11. Real-World ML Applications

| Concept       | ML Application                                              |
|---------------|-------------------------------------------------------------|
| E[X]          | Expected loss (risk minimization is minimizing E[Loss])     |
| Var(X)        | Gradient variance → affects optimizer convergence           |
| Covariance    | Feature correlation → PCA uses covariance matrix            |
| LLN           | Mini-batch gradient is unbiased estimator of full gradient  |
| CLT           | Justifies Gaussian assumptions for averaged quantities      |
| Linearity     | Expected loss of a mixture = mixture of expected losses     |

---

## 12. Summary Table

| Concept              | Formula                              | Key Property                           |
|----------------------|--------------------------------------|----------------------------------------|
| E[X] (discrete)      | Σ x·P(X=x)                         | Linear: E[aX+b] = aE[X]+b            |
| E[X] (continuous)    | ∫ x·f(x)dx                         | E[X+Y] = E[X]+E[Y] always            |
| Var(X)               | E[X²] − (E[X])²                    | Var(aX+b) = a²Var(X)                 |
| Std Dev              | σ = √Var(X)                         | Same units as X                       |
| Cov(X,Y)             | E[XY] − E[X]E[Y]                   | = 0 if independent (not converse)     |
| Correlation          | Cov(X,Y)/(σₓσᵧ)                    | ρ ∈ [−1, 1]                          |
| LLN                  | X̄ₙ → E[X]                          | Sample mean → population mean        |
| CLT                  | X̄ₙ ~ N(μ, σ²/n)                    | Works for any distribution            |

---

## 13. Quick Revision Questions

1. **Compute E[X] and Var(X)** for X with P(X=0)=0.3, P(X=1)=0.5, P(X=2)=0.2.
2. **If E[X]=4 and Var(X)=9,** what are E[3X+2] and Var(3X+2)?
3. **Can Cov(X,Y) = 0 while X and Y are dependent?** Give an example.
4. **Why does the mini-batch SGD loss fluctuate** while full-batch loss decreases smoothly?
5. **State the Central Limit Theorem** and explain why it makes the Gaussian so important.
6. **Using linearity of expectation,** find the expected number of pairs when drawing 5 cards.

---

## 14. Navigation

| ← Previous | Next → |
|:-----------:|:------:|
| [Chapter 3: Probability Distributions](03-probability-distributions.md) | [Chapter 5: Joint & Conditional Probability](05-joint-and-conditional-probability.md) |

---

*Unit 3 · Probability Theory · Chapter 4 of 9*
