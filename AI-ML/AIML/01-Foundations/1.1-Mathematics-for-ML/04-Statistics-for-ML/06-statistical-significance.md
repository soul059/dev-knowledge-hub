# Chapter 6: Statistical Significance

> **Unit 4 — Statistics for ML** | When does a difference truly matter?

---

## 6.1 Overview

Getting a low p-value is only half the story. This chapter distinguishes
**statistical significance** (is the effect real?) from **practical
significance** (is the effect useful?) — a critical skill for responsible ML
experimentation.

| Question | Tool |
|---|---|
| Is the effect real (not just noise)? | p-value, hypothesis test |
| How large is the effect? | Effect size (Cohen's d) |
| Can I reliably detect a real effect? | Power analysis |
| Am I testing too many things? | Multiple testing correction |

---

## 6.2 Statistical vs Practical Significance

### Statistical Significance

The result is unlikely to have occurred by chance alone (p < α).

### Practical Significance

The result is large enough to matter in the real world.

```
  ┌──────────────────────────────────────────────────┐
  │  With enough data, even TINY differences become  │
  │  statistically significant.                      │
  │                                                  │
  │  Example:                                        │
  │  Model A accuracy: 92.10%                        │
  │  Model B accuracy: 92.15%                        │
  │  n = 1,000,000 → p = 0.001 (significant!)       │
  │                                                  │
  │  But is 0.05% worth the deployment cost?         │
  └──────────────────────────────────────────────────┘
```

**Rule of thumb:** Always report effect size alongside p-values.

---

## 6.3 Effect Size — Cohen's d

Measures the magnitude of the difference in standard deviation units.

### Formula

```
              x̄₁ − x̄₂
  d  =  ─────────────────
             s_pooled

  where s_pooled = √((s₁² + s₂²) / 2)
```

### Interpretation (Cohen's guidelines)

| |d| | Effect Size |
|---|---|
| 0.2 | Small |
| 0.5 | Medium |
| 0.8 | Large |

### Example 1

```
Model A: x̄₁ = 0.91, s₁ = 0.03
Model B: x̄₂ = 0.94, s₂ = 0.04

s_pooled = √((0.03² + 0.04²) / 2) = √(0.00125) ≈ 0.0354

d = (0.94 − 0.91) / 0.0354 ≈ 0.85  → Large effect
```

Even if p > 0.05 due to small sample, the large effect size suggests a
meaningful difference — collect more data.

### Python

```python
import numpy as np

def cohens_d(group1, group2):
    n1, n2 = len(group1), len(group2)
    var1, var2 = np.var(group1, ddof=1), np.var(group2, ddof=1)
    pooled_std = np.sqrt((var1 + var2) / 2)
    return (np.mean(group1) - np.mean(group2)) / pooled_std

scores_a = [0.89, 0.91, 0.92, 0.90, 0.93]
scores_b = [0.93, 0.95, 0.94, 0.96, 0.92]

d = cohens_d(scores_b, scores_a)
print(f"Cohen's d = {d:.3f}")
```

---

## 6.4 Power Analysis

**Statistical power** = P(reject H₀ | H₀ is false) = 1 − β.

Commonly, we want power ≥ 0.80 (80 % chance of detecting a real effect).

### The Four Linked Quantities

```
  ┌─────────────┐
  │ Sample size  │──────┐
  │     (n)      │      │
  └─────────────┘      │
  ┌─────────────┐      │     ┌─────────────┐
  │ Effect size  │──────┼────►│   Power     │
  │     (d)      │      │     │  (1 − β)    │
  └─────────────┘      │     └─────────────┘
  ┌─────────────┐      │
  │ Significance │──────┘
  │ level (α)    │
  └─────────────┘

  Fix any three → solve for the fourth.
```

### Example 2 — Sample Size Calculation

```python
from statsmodels.stats.power import TTestIndPower

analysis = TTestIndPower()

# How many samples per group to detect d=0.5 at α=0.05 with 80% power?
n = analysis.solve_power(effect_size=0.5, alpha=0.05, power=0.8,
                         alternative='two-sided')
print(f"Required n per group: {n:.0f}")  # ≈ 64
```

### Power Curve (ASCII)

```
  Power
  1.0 │                          ●●●●●●●●●●
      │                     ●●●●
  0.8 │- - - - - - - - -●●● - - - - - - - -  ← target power
      │              ●●●
  0.6 │           ●●●
      │        ●●●
  0.4 │      ●●
      │    ●●
  0.2 │  ●●
      │ ●
  0.0 │●
      └───────────────────────────────────── n
       10   30   50   70   90  110  130  150

  More samples → more power to detect a given effect.
```

---

## 6.5 Multiple Testing Problem

When you run many hypothesis tests, the chance of at least one false positive
grows rapidly.

### Family-Wise Error Rate (FWER)

```
  P(at least one Type I error) = 1 − (1 − α)^m

  m = number of tests

  m = 1:   P = 0.05
  m = 10:  P = 0.40
  m = 20:  P = 0.64
  m = 100: P = 0.994  ← almost guaranteed false positive!
```

### Bonferroni Correction

The simplest fix: divide α by the number of tests.

```
  α_adjusted = α / m
```

| m tests | Original α | Bonferroni α |
|---|---|---|
| 10 | 0.05 | 0.005 |
| 20 | 0.05 | 0.0025 |
| 100 | 0.05 | 0.0005 |

**Drawback:** Very conservative — increases Type II errors.

### Alternative: Benjamini-Hochberg (FDR)

Controls the **False Discovery Rate** instead of FWER. Less conservative,
more power. Preferred in high-dimensional settings (genomics, feature selection).

```python
from statsmodels.stats.multitest import multipletests

p_values = [0.01, 0.04, 0.03, 0.20, 0.50, 0.001]
reject, corrected_p, _, _ = multipletests(p_values, method='fdr_bh')
print("Reject:", reject)
print("Adjusted p:", corrected_p)
```

---

## 6.6 p-Hacking Dangers

**p-hacking** = manipulating analysis until p < 0.05.

### Common Forms

```
  ┌─────────────────────────────────────────────┐
  │  ✗ Try many features, report only the one   │
  │    that is "significant"                     │
  │  ✗ Remove outliers selectively to get p<0.05│
  │  ✗ Stop data collection when p < 0.05       │
  │  ✗ Try many models, report the best p-value │
  │  ✗ Change α after seeing results             │
  └─────────────────────────────────────────────┘
```

### ML Version of p-Hacking

- Testing dozens of hyperparameter configs and reporting only the best without
  adjusting for multiple comparisons.
- "Leaking" test data into training through repeated evaluation.
- Reporting performance on a cherry-picked random seed.

### Prevention

1. **Pre-register** your hypothesis and analysis plan.
2. Use **hold-out test sets** touched only once.
3. Apply **multiple testing corrections**.
4. Report **all** experiments, not just significant ones.
5. Use **cross-validation** instead of single splits.

---

## 6.7 Reproducibility Crisis

Many published results fail to replicate. Key causes:

| Cause | Fix |
|---|---|
| Small sample sizes | Power analysis before experiment |
| p-hacking | Pre-registration |
| Publication bias | Report null results |
| Overfitting to test set | Strict train/val/test protocol |
| Lack of code sharing | Open-source code + data |

### In ML Research

```
  "State-of-the-art" claims often fail to reproduce because:
  • Results depend on specific random seeds
  • Hyperparameters tuned on test data
  • Hardware/software differences
  • Missing implementation details

  Solution: Report mean ± std over multiple runs,
            share code, fix random seeds for reproducibility.
```

---

## 6.8 Significance in ML Model Comparison

### Best Practices

| Practice | Reason |
|---|---|
| Report mean ± std from K-fold CV | Shows variability |
| Use paired t-test on fold scores | Accounts for data pairing |
| Compute Cohen's d | Quantifies effect size |
| Use McNemar's test | For binary classifiers on same data |
| Report confidence intervals | More informative than p-values |

### Python — Comparing Two Models

```python
from scipy import stats
import numpy as np

# Accuracy on 10 CV folds
model_a_scores = np.array([0.91, 0.89, 0.93, 0.90, 0.88,
                           0.92, 0.91, 0.90, 0.89, 0.93])
model_b_scores = np.array([0.93, 0.92, 0.95, 0.91, 0.90,
                           0.94, 0.93, 0.92, 0.91, 0.94])

# Paired t-test
t_stat, p_value = stats.ttest_rel(model_a_scores, model_b_scores)
print(f"t = {t_stat:.3f}, p = {p_value:.4f}")

# Effect size
d = cohens_d(model_b_scores, model_a_scores)
print(f"Cohen's d = {d:.3f}")

# Decision
if p_value < 0.05 and abs(d) > 0.5:
    print("Statistically AND practically significant improvement")
```

---

## Summary Table

| Concept | Key Idea |
|---|---|
| Statistical significance | p < α → effect unlikely due to chance |
| Practical significance | Effect large enough to matter |
| Cohen's d | (x̄₁−x̄₂)/s_pooled; 0.2/0.5/0.8 = small/medium/large |
| Power (1−β) | P(detect real effect); target ≥ 0.80 |
| Bonferroni correction | α_adj = α/m |
| FDR (Benjamini-Hochberg) | Less conservative than Bonferroni |
| p-hacking | Manipulating analysis to get p < 0.05 |
| Reproducibility | Mean ± std, share code, pre-register |

---

## Quick Revision Questions

1. A p-value of 0.001 with Cohen's d of 0.1 — is this practically significant?
2. You test 20 features for significance at α = 0.05. How many false positives
   do you expect by chance?
3. What is the minimum sample size to detect a medium effect (d = 0.5) with
   80 % power at α = 0.05?
4. Explain the difference between FWER and FDR.
5. Give two examples of p-hacking in ML experiments.
6. Why should ML papers report standard deviations alongside mean accuracy?

---

*← Previous: [Correlation & Covariance](05-correlation-and-covariance.md) | Next: [Bias-Variance Tradeoff](07-bias-variance-tradeoff.md) →*
