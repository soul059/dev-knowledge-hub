# Chapter 4: Confidence Intervals

> **Unit 4 — Statistics for ML** | Quantifying uncertainty in our estimates

---

## 4.1 Overview

A point estimate (e.g., x̄ = 0.92) tells you *one number*. A confidence
interval tells you the **range of plausible values** for the true parameter,
capturing the uncertainty inherent in sampling.

```
Point estimate:       ────── • ──────
                              x̄

95% Confidence Interval:
                  ├──────── • ────────┤
                 Lower      x̄       Upper
```

---

## 4.2 Definition and Interpretation

A **95 % confidence interval** means: if we repeated the experiment many
times, approximately 95 % of the constructed intervals would contain the true
population parameter.

> ⚠️ It does **NOT** mean "there is a 95 % probability that μ lies in this
> interval." The parameter is fixed; the interval is random.

```
Experiment 1:  ├────────── • ──────────┤  ✓ contains μ
Experiment 2:  ├──── • ────┤              ✓ contains μ
Experiment 3:         ├──── • ────┤       ✗ misses μ
Experiment 4:  ├─────────── • ─────────┤  ✓ contains μ
                           ▲
                           μ (true value, unknown)
```

---

## 4.3 Constructing a CI for the Mean

### Case 1: σ Known (z-interval)

```
  CI = x̄  ±  z_{α/2} × (σ / √n)
```

| Confidence Level | z_{α/2} |
|---|---|
| 90 % | 1.645 |
| 95 % | 1.960 |
| 99 % | 2.576 |

### Case 2: σ Unknown (t-interval)

```
  CI = x̄  ±  t_{α/2, df} × (s / √n)       df = n − 1
```

Use the t-distribution when σ is unknown and/or n is small (< 30).

### Example 1 — 95 % CI for Mean

```
Given: n = 25, x̄ = 82, s = 10, α = 0.05

df = 24,  t_{0.025, 24} = 2.064

Margin of Error = 2.064 × (10 / √25) = 2.064 × 2 = 4.128

CI = (82 − 4.128,  82 + 4.128)
   = (77.87,  86.13)

Interpretation: We are 95% confident the true population mean
lies between 77.87 and 86.13.
```

### Python

```python
import numpy as np
from scipy import stats

data = np.random.normal(loc=82, scale=10, size=25)
n = len(data)
x_bar = np.mean(data)
s = np.std(data, ddof=1)
se = s / np.sqrt(n)

# 95% CI using t-distribution
t_crit = stats.t.ppf(0.975, df=n-1)
ci_lower = x_bar - t_crit * se
ci_upper = x_bar + t_crit * se
print(f"95% CI: ({ci_lower:.2f}, {ci_upper:.2f})")

# Or use scipy directly
ci = stats.t.interval(0.95, df=n-1, loc=x_bar, scale=se)
print(f"95% CI: ({ci[0]:.2f}, {ci[1]:.2f})")
```

---

## 4.4 CI for a Proportion

```
  p̂ ± z_{α/2} × √(p̂(1 − p̂) / n)
```

### Example 2 — CI for Proportion

```
A classifier correctly predicts 460 out of 500 test samples.
p̂ = 460 / 500 = 0.92

SE = √(0.92 × 0.08 / 500) = √(0.000147) ≈ 0.01213

95% CI = 0.92 ± 1.96 × 0.01213
       = (0.896, 0.944)

We are 95% confident the true accuracy lies between 89.6% and 94.4%.
```

---

## 4.5 Margin of Error

```
  ME = z_{α/2} × SE
```

The margin of error is the "±" part of the confidence interval. It depends on:

```
  ME ∝ z  (confidence level ↑ → wider interval)
  ME ∝ s  (more variability → wider interval)
  ME ∝ 1/√n (more data → narrower interval)
```

### Visualisation — Effect of n on CI Width

```
  n = 10   ├──────────────── • ────────────────┤
  n = 50   ├──────── • ────────┤
  n = 200  ├──── • ────┤
  n = 1000 ├── • ──┤

  More data → tighter confidence interval
```

---

## 4.6 Sample Size Determination

To achieve a desired margin of error ME with confidence level z:

### For a Mean

```
         ⎛ z_{α/2} × σ ⎞²
  n  =   ⎜──────────────⎟
         ⎝     ME       ⎠
```

### For a Proportion

```
         z_{α/2}² × p̂(1 − p̂)
  n  =  ──────────────────────
                ME²
```

If p̂ is unknown, use p̂ = 0.5 (worst case → largest n).

### Example 3 — Required Sample Size

```
Want: 95% CI for proportion with ME = 0.03
Assume p̂ = 0.5 (worst case)

n = (1.96² × 0.5 × 0.5) / 0.03²
  = (3.8416 × 0.25) / 0.0009
  = 0.9604 / 0.0009
  ≈ 1068

Need at least 1068 test samples.
```

---

## 4.7 Relationship to Hypothesis Testing

Confidence intervals and hypothesis tests are two sides of the same coin:

```
  ┌─────────────────────────────────────────────────┐
  │  If μ₀ falls OUTSIDE the 95% CI → Reject H₀    │
  │  If μ₀ falls INSIDE  the 95% CI → Fail to reject│
  └─────────────────────────────────────────────────┘
```

### Equivalence Diagram

```
  Hypothesis Test (α = 0.05)        Confidence Interval (95%)
  ─────────────────────────         ─────────────────────────
  p-value < 0.05 → reject     ⟺    μ₀ outside CI
  p-value ≥ 0.05 → fail       ⟺    μ₀ inside CI
```

**Advantage of CI over p-value:** CI gives the direction and magnitude of the
effect, not just a binary decision.

---

## 4.8 Confidence Intervals in Model Evaluation

### 4.8.1 CI for Cross-Validation Scores

```python
from sklearn.model_selection import cross_val_score
from sklearn.ensemble import RandomForestClassifier
import numpy as np

clf = RandomForestClassifier(n_estimators=100, random_state=42)
scores = cross_val_score(clf, X, y, cv=10, scoring='accuracy')

mean_acc = scores.mean()
se = scores.std() / np.sqrt(len(scores))
ci = (mean_acc - 1.96 * se, mean_acc + 1.96 * se)

print(f"Mean Accuracy: {mean_acc:.4f}")
print(f"95% CI: ({ci[0]:.4f}, {ci[1]:.4f})")
```

### 4.8.2 Bootstrap CI for Model Performance

```python
from sklearn.utils import resample
from sklearn.metrics import accuracy_score

n_iterations = 1000
boot_scores = []

for _ in range(n_iterations):
    X_boot, y_boot = resample(X_test, y_test, random_state=None)
    score = accuracy_score(y_boot, model.predict(X_boot))
    boot_scores.append(score)

lower = np.percentile(boot_scores, 2.5)
upper = np.percentile(boot_scores, 97.5)
print(f"Bootstrap 95% CI: ({lower:.4f}, {upper:.4f})")
```

### 4.8.3 Comparing Two Models

```
Model A:  95% CI = (0.88, 0.93)
Model B:  95% CI = (0.91, 0.96)

  Model A   ├──────── • ────────┤
  Model B         ├──────── • ────────┤
                   ▲ overlap zone

Overlapping CIs suggest the difference may NOT be significant.
For a definitive answer, compute CI for the DIFFERENCE (A − B).
```

---

## 4.9 Real-World ML Applications

| Application | How CI is used |
|---|---|
| Model selection | CI on accuracy across CV folds |
| A/B testing | CI for difference in conversion rates |
| Error estimation | CI around RMSE/MAE of regression |
| Hyperparameter tuning | Overlapping CIs → no significant difference |
| Reporting results | Papers should report CI, not just point accuracy |

---

## Summary Table

| Concept | Formula / Key Idea |
|---|---|
| CI for mean (σ known) | x̄ ± z × (σ/√n) |
| CI for mean (σ unknown) | x̄ ± t × (s/√n) |
| CI for proportion | p̂ ± z × √(p̂(1−p̂)/n) |
| Margin of error | ME = z × SE |
| Sample size (mean) | n = (zσ/ME)² |
| Sample size (proportion) | n = z²p̂(1−p̂)/ME² |
| CI ↔ Hypothesis test | μ₀ outside CI ⟺ reject H₀ |
| Wider CI causes | Higher confidence, more variability, smaller n |

---

## Quick Revision Questions

1. A 99 % CI is wider than a 95 % CI for the same data. Why?
2. You compute a 95 % CI of (0.88, 0.94) for model accuracy. Can you say
   "there is a 95 % chance the true accuracy is in this range"? Why or why not?
3. How many test samples do you need for a ±2 % margin of error at 95 %
   confidence (assume p̂ = 0.5)?
4. Two models have overlapping 95 % CIs. Does that guarantee their means are
   not significantly different?
5. Why is a bootstrap CI useful when the sampling distribution is unknown?
6. How does increasing the sample size affect the margin of error?

---

*← Previous: [Hypothesis Testing](03-hypothesis-testing.md) | Next: [Correlation & Covariance](05-correlation-and-covariance.md) →*
