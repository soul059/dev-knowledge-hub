# Chapter 2: Sampling and Estimation

> **Unit 4 — Statistics for ML** | From populations to predictions

---

## 2.1 Overview

We rarely have access to the entire population. Instead, we draw **samples**
and use them to **estimate** population parameters. This chapter builds the
bridge between statistics and the train/test/validation paradigm of ML.

| Concept | ML Parallel |
|---|---|
| Population | Full data universe |
| Sample | Training set |
| Sampling distribution | Distribution of metric across CV folds |
| Standard error | Uncertainty of model metric |

---

## 2.2 Population vs Sample

| | Population | Sample |
|---|---|---|
| Definition | All items of interest | Subset drawn from population |
| Size symbol | N | n |
| Mean symbol | μ | x̄ |
| Std dev symbol | σ | s |
| Feasibility | Often impossible to observe fully | Practical, measurable |

**Key idea:** We use sample statistics (x̄, s) to *estimate* population
parameters (μ, σ).

---

## 2.3 Sampling Methods

### 2.3.1 Simple Random Sampling

Every member has equal probability of selection.

```
Population:  [A B C D E F G H I J]
Random draw (n=4): {C, F, A, I}
```

```python
import random
population = list(range(1, 101))
sample = random.sample(population, k=20)
```

### 2.3.2 Stratified Sampling

Divide population into strata (subgroups) and sample proportionally from each.

```
Population (100 items):
  ┌─────────────────┐
  │ Class A: 70     │ → sample 14 (20%)
  │ Class B: 30     │ → sample  6 (20%)
  └─────────────────┘
  Total sample: 20
```

**ML use:** `train_test_split(..., stratify=y)` in scikit-learn preserves class
proportions — critical for imbalanced datasets.

```python
from sklearn.model_selection import train_test_split
X_train, X_test, y_train, y_test = train_test_split(
    X, y, test_size=0.2, stratify=y, random_state=42
)
```

### 2.3.3 Systematic Sampling

Select every k-th element after a random start.

```
k = N / n = 100 / 20 = 5
Random start: 3
Selected: 3, 8, 13, 18, 23, ...
```

Simple to implement but risky if data has periodic patterns.

### 2.3.4 Comparison Table

| Method | Pros | Cons |
|---|---|---|
| Simple random | Unbiased, easy | May miss rare subgroups |
| Stratified | Preserves subgroup ratios | Requires subgroup knowledge |
| Systematic | Easy, good coverage | Fails with periodicity |
| Cluster | Cost-effective for geo data | Higher variance |

---

## 2.4 Sampling Distribution

If we repeatedly draw samples of size n and compute their means, those means
form a **sampling distribution**.

```
Population (μ, σ)
      │
      ▼
 Draw sample₁ → x̄₁
 Draw sample₂ → x̄₂
 Draw sample₃ → x̄₃         ──► Distribution of x̄ values
        ⋮                         = Sampling Distribution
 Draw sampleₖ → x̄ₖ
```

### Central Limit Theorem (CLT)

For sufficiently large n (rule of thumb: n ≥ 30):

```
  x̄  ~  N(μ,  σ² / n)

  regardless of the shape of the population distribution.
```

This is why so many ML evaluation metrics are approximately normal when
averaged over enough folds or bootstrap samples.

---

## 2.5 Standard Error (SE)

The standard deviation of the sampling distribution:

```
         σ         s
  SE = ─────  ≈  ─────   (when σ unknown)
        √n        √n
```

| n increases → | SE decreases → | Estimate more precise |
|---|---|---|

### Example 1

```
Sample: n = 64, s = 8

SE = 8 / √64 = 8 / 8 = 1.0

Interpretation: The sample mean is expected to vary by about ±1 unit
from the true population mean.
```

---

## 2.6 Point Estimation

A **point estimator** is a single number used to estimate a population
parameter.

| Parameter | Point Estimator |
|---|---|
| μ (population mean) | x̄ (sample mean) |
| σ² (population variance) | s² (sample variance) |
| p (population proportion) | p̂ = x / n |

---

## 2.7 Properties of Good Estimators

### 2.7.1 Unbiasedness

```
E[θ̂] = θ
```

The expected value of the estimator equals the true parameter.
- x̄ is an unbiased estimator of μ.
- s² (with n−1) is an unbiased estimator of σ².

### 2.7.2 Consistency

As n → ∞, the estimator converges to the true value.

```
θ̂ₙ  ──P──►  θ   as  n → ∞
```

### 2.7.3 Efficiency

Among unbiased estimators, the one with the **smallest variance** is most
efficient. The Cramér-Rao lower bound gives the theoretical minimum variance.

### 2.7.4 Summary Diagram

```
Good Estimator
  ├── Unbiased:   E[θ̂] = θ        (hits the bullseye on average)
  ├── Consistent: θ̂ → θ as n → ∞  (improves with more data)
  └── Efficient:  Var(θ̂) is min    (tightest grouping)
```

---

## 2.8 Maximum Likelihood Estimation (MLE) — Brief

MLE finds the parameter value that maximises the likelihood of observing the
data:

```
θ̂_MLE = argmax  L(θ | x₁, x₂, ..., xₙ)
           θ

       = argmax  Π P(xᵢ | θ)
           θ

In practice, we maximise the log-likelihood:
  ℓ(θ) = Σ log P(xᵢ | θ)
```

Many ML models are MLE under the hood:
- Logistic regression maximises log-likelihood.
- Linear regression with MSE loss = MLE assuming Gaussian noise.

---

## 2.9 The Bootstrap Method

A resampling technique to estimate the sampling distribution when theory is
hard to apply.

### Algorithm

```
1. From original sample of size n, draw n observations WITH replacement.
2. Compute the statistic of interest (mean, median, etc.).
3. Repeat B times (B = 1000–10000).
4. The distribution of B statistics ≈ sampling distribution.
```

### Python Example

```python
import numpy as np

data = np.array([12, 15, 14, 10, 18, 22, 20, 16, 13, 19])
B = 10000
boot_means = np.array([
    np.mean(np.random.choice(data, size=len(data), replace=True))
    for _ in range(B)
])

print(f"Bootstrap mean:  {boot_means.mean():.2f}")
print(f"Bootstrap SE:    {boot_means.std():.2f}")
print(f"95% CI:          ({np.percentile(boot_means, 2.5):.2f}, "
      f"{np.percentile(boot_means, 97.5):.2f})")
```

### Bootstrap in ML

| Use case | How |
|---|---|
| Confidence interval for accuracy | Bootstrap test predictions |
| Bagging (Random Forest) | Train on bootstrap samples |
| Out-of-bag (OOB) error | ~36.8 % samples not drawn → free validation |

---

## 2.10 Connection to Train/Test Splits

```
Full Dataset (≈ "Population")
  │
  ├── Training Set (≈ "Sample")
  │     └── Used to estimate model parameters (≈ point estimation)
  │
  ├── Validation Set
  │     └── Used to tune hyperparameters
  │
  └── Test Set
        └── Used to estimate generalisation error (≈ population parameter)

Cross-validation ≈ Repeated sampling → sampling distribution of metric
```

| Statistics Concept | ML Equivalent |
|---|---|
| Population parameter μ | True generalisation error |
| Sample statistic x̄ | Test-set accuracy |
| Standard error | Std of CV fold scores |
| Bootstrap | Bagging, OOB estimation |
| Stratified sampling | Stratified K-fold |

---

## 2.11 Real-World ML Applications

1. **Stratified K-Fold CV** ensures each fold mirrors class distribution.
2. **Bootstrap aggregating (Bagging)** reduces variance via ensemble of
   bootstrap samples.
3. **Standard error of accuracy** tells you if 92 % vs 91 % is noise or real.
4. **MLE** is the optimisation target for logistic regression, neural nets
   (cross-entropy loss).

---

## Summary Table

| Concept | Key Formula / Idea |
|---|---|
| Standard Error | SE = s / √n |
| CLT | x̄ ~ N(μ, σ²/n) for large n |
| Unbiased estimator | E[θ̂] = θ |
| Consistent estimator | θ̂ → θ as n → ∞ |
| Efficient estimator | Minimum variance among unbiased |
| Bootstrap | Resample with replacement B times |
| MLE | argmax Π P(xᵢ|θ) |

---

## Quick Revision Questions

1. Why does the Central Limit Theorem matter for ML evaluation metrics?
2. What is the difference between standard deviation and standard error?
3. Why do we use (n − 1) in the sample variance formula?
4. Explain how stratified sampling relates to `train_test_split(stratify=y)`.
5. In bootstrap, approximately what fraction of data is left out on each
   resample, and how is this used in Random Forests?
6. How is logistic regression connected to Maximum Likelihood Estimation?

---

*← Previous: [Descriptive Statistics](01-descriptive-statistics.md) | Next: [Hypothesis Testing](03-hypothesis-testing.md) →*
