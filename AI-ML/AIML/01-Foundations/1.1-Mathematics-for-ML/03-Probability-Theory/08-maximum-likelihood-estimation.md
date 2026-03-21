# Chapter 8: Maximum Likelihood Estimation (MLE)

> **Unit 3: Probability Theory** | **Topic**: Likelihood, Log-Likelihood, MLE Derivations, Connection to ML

---

## 1. Overview

Maximum Likelihood Estimation (MLE) is the most widely used method for estimating the parameters
of a statistical model. Given observed data, MLE finds the parameter values that make the data
**most probable**. Nearly every ML model — from logistic regression to neural networks — is trained
using a principle that traces back to MLE.

---

## 2. The Likelihood Function

### 2.1 Definition

Given data D = {x₁, x₂, ..., xₙ} and a model with parameter θ:

```
Likelihood:  L(θ) = P(D | θ) = ∏ᵢ₌₁ⁿ P(xᵢ | θ)

(assuming i.i.d. — independent and identically distributed data)
```

> ⚠️ **Key distinction**: The PDF/PMF is a function of **x** with fixed θ.
> The likelihood is the SAME formula but viewed as a function of **θ** with fixed data.

### 2.2 Likelihood vs Probability

```
P(x | θ) as function of x  →  Probability distribution (integrates/sums to 1)
P(x | θ) as function of θ  →  Likelihood function (does NOT integrate to 1)
```

---

## 3. Log-Likelihood

Products are numerically unstable and hard to differentiate. The **log-likelihood** converts
products into sums:

```
ℓ(θ) = ln L(θ) = ln ∏ᵢ P(xᵢ|θ) = Σᵢ₌₁ⁿ ln P(xᵢ|θ)
```

**Why log?**
1. Log is monotonically increasing → maximizing log-likelihood = maximizing likelihood
2. Products → Sums (numerically stable, easier calculus)
3. Connects to information theory (cross-entropy)

---

## 4. MLE Procedure

```
Step 1: Write the likelihood L(θ) = ∏ P(xᵢ|θ)
Step 2: Take the log → ℓ(θ) = Σ ln P(xᵢ|θ)
Step 3: Differentiate with respect to θ
Step 4: Set derivative = 0 and solve for θ_MLE
Step 5: Verify it's a maximum (second derivative < 0)

                   θ̂_MLE = argmax_θ  ℓ(θ)
```

---

## 5. MLE for Bernoulli Distribution — Full Derivation

**Setup**: Data D = {x₁, ..., xₙ} where xᵢ ∈ {0, 1}, model X ~ Bernoulli(p).

```
Step 1: Likelihood
  L(p) = ∏ᵢ pˣⁱ(1-p)¹⁻ˣⁱ

Step 2: Log-likelihood
  ℓ(p) = Σᵢ [xᵢ ln(p) + (1-xᵢ) ln(1-p)]
        = k·ln(p) + (n-k)·ln(1-p)

  where k = Σᵢ xᵢ = number of successes

Step 3: Differentiate
  dℓ/dp = k/p - (n-k)/(1-p)

Step 4: Set to zero
  k/p = (n-k)/(1-p)
  k(1-p) = p(n-k)
  k - kp = np - kp
  k = np

  ┌─────────────────────────┐
  │  p̂_MLE = k/n = x̄       │
  │  (sample proportion)    │
  └─────────────────────────┘

Step 5: Second derivative
  d²ℓ/dp² = -k/p² - (n-k)/(1-p)² < 0  ✓  (it's a maximum)
```

**Example**: 7 heads in 10 coin flips → p̂ = 7/10 = 0.7

---

## 6. MLE for Gaussian Distribution — Full Derivation

**Setup**: Data D = {x₁, ..., xₙ}, model X ~ N(μ, σ²). Estimate both μ and σ².

```
Step 1: Likelihood
  L(μ, σ²) = ∏ᵢ (1/(σ√(2π))) exp(-(xᵢ-μ)²/(2σ²))

Step 2: Log-likelihood
  ℓ(μ, σ²) = -n/2 ln(2π) - n/2 ln(σ²) - 1/(2σ²) Σᵢ(xᵢ-μ)²

Step 3a: Differentiate w.r.t. μ
  ∂ℓ/∂μ = 1/σ² · Σᵢ(xᵢ - μ) = 0
  Σᵢ xᵢ - nμ = 0

  ┌───────────────────────┐
  │  μ̂_MLE = (1/n)Σxᵢ = x̄ │  (sample mean)
  └───────────────────────┘

Step 3b: Differentiate w.r.t. σ²
  Let τ = σ². Then:
  ∂ℓ/∂τ = -n/(2τ) + 1/(2τ²) Σᵢ(xᵢ-μ̂)² = 0

  n/(2τ) = Σᵢ(xᵢ-μ̂)² / (2τ²)
  nτ = Σᵢ(xᵢ-μ̂)²

  ┌───────────────────────────────────┐
  │  σ̂²_MLE = (1/n)Σ(xᵢ - x̄)²       │  (sample variance — biased!)
  └───────────────────────────────────┘

Note: MLE gives the BIASED sample variance (divides by n, not n-1).
The unbiased estimator uses n-1 (Bessel's correction).
```

---

## 7. MLE for Poisson Distribution

```
Data: {x₁, ..., xₙ}, model X ~ Poisson(λ)

ℓ(λ) = Σᵢ [xᵢ ln(λ) - λ - ln(xᵢ!)]

dℓ/dλ = Σᵢ xᵢ/λ - n = 0

┌────────────────────────┐
│  λ̂_MLE = (1/n)Σxᵢ = x̄  │  (sample mean, again!)
└────────────────────────┘
```

---

## 8. Properties of MLE

| Property         | Description                                                  |
|------------------|--------------------------------------------------------------|
| **Consistency**  | As n → ∞, θ̂_MLE → θ_true (converges to true value)         |
| **Asymptotic normality** | For large n: θ̂ ~ N(θ_true, 1/nI(θ)) approximately  |
| **Efficiency**   | Achieves the Cramér-Rao lower bound (minimum variance)       |
| **Invariance**   | If θ̂ is MLE of θ, then g(θ̂) is MLE of g(θ)                |
| **Bias**         | MLE can be biased for finite samples (e.g., σ² estimate)     |

### Fisher Information

```
I(θ) = -E[d²ℓ/dθ²]    (curvature of the log-likelihood)

Higher curvature → more information → more precise estimate
```

---

## 9. Connection to Loss Functions in ML

### 9.1 MLE = Minimizing Negative Log-Likelihood (NLL)

```
θ̂_MLE = argmax_θ  ℓ(θ)
       = argmin_θ  -ℓ(θ)
       = argmin_θ  NLL(θ)
```

### 9.2 Cross-Entropy Loss IS Negative Log-Likelihood

For classification with Bernoulli model:

```
NLL = -Σᵢ [yᵢ ln(p̂ᵢ) + (1-yᵢ) ln(1-p̂ᵢ)]

This is exactly the Binary Cross-Entropy (BCE) loss!
```

For multi-class with Multinomial (softmax):

```
NLL = -Σᵢ Σⱼ yᵢⱼ ln(p̂ᵢⱼ)

This is exactly the Categorical Cross-Entropy loss!
```

### 9.3 MSE Loss = Gaussian MLE

```
If Y|X ~ N(f(X;θ), σ²), then maximizing log-likelihood:

ℓ(θ) = -n/2 ln(2πσ²) - 1/(2σ²) Σᵢ(yᵢ - f(xᵢ;θ))²

Maximizing ℓ ⟺ Minimizing Σ(yᵢ - f(xᵢ;θ))² = MSE Loss!
```

### Summary of Connections

```
┌──────────────────────────────────────────────────┐
│                                                  │
│   Assumed Model      MLE Objective    ML Loss    │
│   ─────────────      ─────────────    ───────    │
│   Gaussian noise  →  Max ℓ(θ)     =  Min MSE    │
│   Bernoulli       →  Max ℓ(p)     =  Min BCE    │
│   Multinomial     →  Max ℓ(p)     =  Min CCE    │
│   Poisson         →  Max ℓ(λ)     =  Min Poisson│
│                                       Loss       │
│                                                  │
└──────────────────────────────────────────────────┘
```

---

## 10. Python Implementation

```python
import numpy as np
from scipy.optimize import minimize_scalar
from scipy.stats import norm

# --- MLE for Bernoulli ---
data_bern = np.array([1, 1, 0, 1, 1, 1, 0, 1, 0, 1])
p_mle = data_bern.mean()
print(f"Bernoulli MLE: p̂ = {p_mle:.4f}")

# --- MLE for Gaussian ---
data_gauss = np.random.normal(5.0, 2.0, size=100)
mu_mle = data_gauss.mean()
sigma2_mle = np.mean((data_gauss - mu_mle)**2)  # biased (MLE)
sigma2_unbiased = np.var(data_gauss, ddof=1)      # unbiased

print(f"Gaussian MLE:  μ̂ = {mu_mle:.4f}, σ̂² = {sigma2_mle:.4f}")
print(f"Unbiased:      σ² = {sigma2_unbiased:.4f}")

# --- MLE for Poisson ---
data_pois = np.random.poisson(3.5, size=200)
lambda_mle = data_pois.mean()
print(f"Poisson MLE:   λ̂ = {lambda_mle:.4f}")
```

```python
# --- Visualize Likelihood Function ---
import matplotlib.pyplot as plt

# Bernoulli: observe 7 heads in 10 flips
n, k = 10, 7
p_range = np.linspace(0.01, 0.99, 200)

# Log-likelihood as function of p
log_lik = k * np.log(p_range) + (n - k) * np.log(1 - p_range)

plt.figure(figsize=(8, 4))
plt.plot(p_range, log_lik, 'b-', lw=2)
plt.axvline(k/n, color='r', ls='--', label=f'MLE: p̂ = {k/n}')
plt.xlabel('p'); plt.ylabel('Log-Likelihood ℓ(p)')
plt.title('Log-Likelihood for Bernoulli (7 heads in 10 flips)')
plt.legend(); plt.grid(True)
plt.show()
```

```python
# --- MLE = Minimizing Cross-Entropy in Logistic Regression ---
from sklearn.linear_model import LogisticRegression
from sklearn.datasets import make_classification

X, y = make_classification(n_samples=200, n_features=2, random_state=42)
model = LogisticRegression()
model.fit(X, y)

# The model internally minimizes negative log-likelihood (cross-entropy)
probs = model.predict_proba(X)
nll = -np.mean(y * np.log(probs[:,1]) + (1-y) * np.log(probs[:,0]))
print(f"Negative Log-Likelihood (cross-entropy): {nll:.4f}")
```

---

## 11. MLE Limitations

| Limitation              | Description                                              |
|-------------------------|----------------------------------------------------------|
| Overfitting              | With small data, MLE can overfit (no regularization)    |
| No uncertainty           | Gives point estimate, no confidence measure             |
| Degenerate solutions     | Can give extreme values (e.g., p̂=0 or p̂=1)            |
| Biased for small n       | Variance estimate is biased (÷n instead of ÷(n-1))     |
| Requires model choice    | MLE is only as good as the assumed distribution         |

→ These limitations motivate **MAP estimation** (next chapter) and full **Bayesian inference**.

---

## 12. Summary Table

| Concept           | Formula / Key Point                                    |
|-------------------|--------------------------------------------------------|
| Likelihood        | L(θ) = ∏ P(xᵢ\|θ)                                    |
| Log-likelihood    | ℓ(θ) = Σ ln P(xᵢ\|θ)                                 |
| MLE               | θ̂ = argmax ℓ(θ)                                       |
| Bernoulli MLE     | p̂ = k/n (sample proportion)                           |
| Gaussian μ MLE    | μ̂ = x̄ (sample mean)                                   |
| Gaussian σ² MLE   | σ̂² = (1/n)Σ(xᵢ−x̄)² (biased)                         |
| Poisson MLE       | λ̂ = x̄ (sample mean)                                   |
| MSE loss          | = Gaussian NLL (up to constants)                       |
| Cross-entropy     | = Bernoulli/Multinomial NLL                            |
| Consistency       | θ̂ → θ_true as n → ∞                                   |

---

## 13. Quick Revision Questions

1. **What is the difference between likelihood and probability?**
2. **Derive the MLE for Bernoulli(p)** given n observations with k successes.
3. **Why do we use log-likelihood instead of likelihood?** Give two reasons.
4. **Show that minimizing MSE is equivalent to MLE** under Gaussian noise assumption.
5. **Why is the MLE estimate of variance biased?** What is Bessel's correction?
6. **What is Fisher Information,** and how does it relate to the precision of MLE?

---

## 14. Navigation

| ← Previous | Next → |
|:-----------:|:------:|
| [Chapter 7: Common Distributions](07-common-distributions.md) | [Chapter 9: MAP Estimation](09-maximum-a-posteriori.md) |

---

*Unit 3 · Probability Theory · Chapter 8 of 9*
