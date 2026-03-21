# Chapter 3: Hypothesis Testing

> **Unit 4 — Statistics for ML** | Making data-driven decisions with rigour

---

## 3.1 Overview

Hypothesis testing is a formal framework for deciding whether observed data
provides enough evidence to reject a claim about a population. In ML, it
underpins A/B testing, feature selection, and model comparison.

| Step | Action |
|---|---|
| 1 | State H₀ and H₁ |
| 2 | Choose significance level α |
| 3 | Compute test statistic |
| 4 | Find p-value |
| 5 | Decision: reject or fail to reject H₀ |

---

## 3.2 Null and Alternative Hypotheses

| Symbol | Name | Meaning |
|---|---|---|
| H₀ | Null hypothesis | Default assumption (no effect / no difference) |
| H₁ (Hₐ) | Alternative hypothesis | What we want to show (effect exists) |

**Example:** "Does Model B have higher accuracy than Model A?"

```
H₀: μ_B − μ_A ≤ 0   (B is not better)
H₁: μ_B − μ_A > 0    (B is better)
```

---

## 3.3 Test Statistic

A value computed from the sample that measures how far the observed data
deviates from what H₀ predicts.

```
              Observed statistic − Expected under H₀
  Test stat = ────────────────────────────────────────
                      Standard error
```

---

## 3.4 p-Value

The probability of obtaining a test statistic **at least as extreme** as the
one observed, assuming H₀ is true.

```
  p-value small  →  data unlikely under H₀  →  reject H₀
  p-value large  →  data consistent with H₀  →  fail to reject H₀
```

### Decision Rule

```
  If p-value ≤ α  →  Reject H₀  (result is "statistically significant")
  If p-value > α  →  Fail to reject H₀
```

Common α values: 0.05, 0.01, 0.10.

---

## 3.5 Type I and Type II Errors

```
                        Reality
                  ┌──────────┬──────────┐
                  │  H₀ True │ H₀ False │
  ┌───────────────┼──────────┼──────────┤
  │ Reject H₀     │ Type I   │ Correct  │
  │               │ (α)      │ (Power)  │
  ├───────────────┼──────────┼──────────┤
  │ Fail to       │ Correct  │ Type II  │
  │ Reject H₀     │ (1 − α)  │ (β)      │
  └───────────────┴──────────┴──────────┘

  Type I  error (α): False positive — rejecting H₀ when it is true.
  Type II error (β): False negative — failing to reject H₀ when it is false.
  Power = 1 − β:     Probability of correctly rejecting a false H₀.
```

| Error | ML Analogy |
|---|---|
| Type I | Predicting positive when actually negative (FP) |
| Type II | Predicting negative when actually positive (FN) |

---

## 3.6 One-Tailed vs Two-Tailed Tests

```
Two-tailed (H₁: μ ≠ μ₀):

  Reject │           Reject
    ◄────┤   Fail to  ├────►
   α/2   │   reject   │  α/2
  ───────┴────────────┴───────

One-tailed right (H₁: μ > μ₀):

                       Reject
         Fail to      ├────►
         reject       │  α
  ───────────────────┴───────
```

Use **one-tailed** when you have a directional hypothesis (e.g., "new model is
better"). Use **two-tailed** when difference could go either way.

---

## 3.7 Common Tests

### 3.7.1 z-Test (known σ, large n)

```
         x̄ − μ₀
  z  =  ─────────
         σ / √n
```

### 3.7.2 t-Test (unknown σ, small n)

```
         x̄ − μ₀
  t  =  ─────────      df = n − 1
         s / √n
```

**Two-sample t-test** (independent):

```
            x̄₁ − x̄₂
  t  = ─────────────────
        √(s₁²/n₁ + s₂²/n₂)
```

### 3.7.3 Paired t-Test

When the same subjects are measured under two conditions (e.g., before/after):

```
  Compute dᵢ = x₁ᵢ − x₂ᵢ   for each pair
  Then apply one-sample t-test on the dᵢ values.
```

### 3.7.4 Chi-Squared Test (χ²)

Tests association between categorical variables.

```
         Σ  (Oᵢ − Eᵢ)²
  χ²  = ───────────────
              Eᵢ

  O = Observed frequency,  E = Expected frequency
  df = (rows − 1)(cols − 1)
```

### 3.7.5 ANOVA (Analysis of Variance)

Compares means of **3 or more** groups simultaneously.

```
  H₀: μ₁ = μ₂ = μ₃ = ... = μₖ
  H₁: At least one μᵢ differs

              Between-group variance (MSB)
  F-stat  =  ─────────────────────────────
              Within-group variance  (MSW)
```

If F is large → groups differ significantly.

---

## 3.8 Step-by-Step Hypothesis Test Example

**Problem:** A data scientist claims the mean inference time of a new model is
less than 50 ms. A sample of n = 36 runs gives x̄ = 47 ms, s = 9 ms. Test at
α = 0.05.

```
Step 1: H₀: μ ≥ 50     H₁: μ < 50   (left-tailed)

Step 2: α = 0.05

Step 3: t = (47 − 50) / (9 / √36)
           = −3 / 1.5
           = −2.0

Step 4: df = 35
        p-value ≈ 0.027  (from t-distribution table)

Step 5: 0.027 < 0.05 → Reject H₀

Conclusion: Sufficient evidence that the mean inference time is < 50 ms.
```

### Python

```python
from scipy import stats

# One-sample t-test (testing if mean < 50)
import numpy as np
np.random.seed(42)
sample = np.random.normal(loc=47, scale=9, size=36)

t_stat, p_value = stats.ttest_1samp(sample, popmean=50)
p_one_tail = p_value / 2  # one-tailed

print(f"t = {t_stat:.3f}, p (one-tailed) = {p_one_tail:.4f}")
if p_one_tail < 0.05:
    print("Reject H₀: evidence supports μ < 50 ms")
```

---

## 3.9 A/B Testing in ML

A/B testing is hypothesis testing applied to product decisions.

```
 ┌──────────────┐        ┌──────────────┐
 │  Control (A)  │        │ Treatment (B) │
 │  Old model    │        │ New model     │
 │  n₁ users     │        │ n₂ users      │
 └──────┬───────┘        └──────┬───────┘
        │                       │
        ▼                       ▼
   Metric: CTR_A            Metric: CTR_B
        │                       │
        └───────── Compare ─────┘
                     │
           Two-sample z/t test
                     │
              p-value → Decision
```

### Example — A/B Test for Click-Through Rate

```python
from scipy.stats import proportions_ztest

# Control: 500 clicks out of 10000
# Treatment: 550 clicks out of 10000
count = np.array([500, 550])
nobs  = np.array([10000, 10000])

z_stat, p_value = proportions_ztest(count, nobs, alternative='smaller')
print(f"z = {z_stat:.3f}, p = {p_value:.4f}")
```

### A/B Testing Checklist

1. Define metric (CTR, conversion, revenue).
2. Determine required sample size (power analysis).
3. Randomly assign users to A and B.
4. Run experiment until sample size is reached.
5. Perform hypothesis test. Do NOT peek early.
6. Consider practical significance, not just p-value.

---

## 3.10 Real-World ML Applications

| Application | Test Used |
|---|---|
| Compare two model accuracies | Paired t-test (across CV folds) |
| Feature independence check | Chi-squared test |
| Compare ≥ 3 models | ANOVA or Friedman test |
| Online model deployment | A/B test (z-test for proportions) |
| Check if residuals are normal | Shapiro-Wilk, K-S test |

---

## Summary Table

| Concept | Key Point |
|---|---|
| H₀ / H₁ | Null = no effect; Alternative = effect exists |
| p-value | P(data as extreme \| H₀ true) |
| α | Threshold (usually 0.05) |
| Type I error | False positive (reject true H₀) |
| Type II error | False negative (keep false H₀) |
| z-test | Known σ, large n |
| t-test | Unknown σ, small n |
| χ² test | Categorical association |
| ANOVA | Compare ≥ 3 group means |
| A/B test | Hypothesis test for product experiments |

---

## Quick Revision Questions

1. What does a p-value of 0.03 mean in plain English?
2. Why is "fail to reject H₀" different from "accept H₀"?
3. When would you use a paired t-test instead of an independent t-test?
4. In an A/B test, what happens if you check results too early?
5. How does ANOVA differ from running multiple t-tests?
6. Describe a Type I error in the context of deploying a new ML model.

---

*← Previous: [Sampling & Estimation](02-sampling-and-estimation.md) | Next: [Confidence Intervals](04-confidence-intervals.md) →*
