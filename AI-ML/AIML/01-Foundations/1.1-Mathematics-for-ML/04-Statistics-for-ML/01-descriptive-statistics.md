# Chapter 1: Descriptive Statistics

> **Unit 4 — Statistics for ML** | Summarizing data before feeding it to models

---

## 1.1 Overview

Descriptive statistics condense raw data into a handful of numbers that reveal
shape, centre, and spread. Every ML pipeline begins here — understanding your
features before training prevents garbage-in-garbage-out disasters.

| What you will learn | Why it matters for ML |
|---|---|
| Central tendency (mean, median, mode) | Feature scaling, imputation |
| Spread (range, IQR, variance, SD) | Normalisation, outlier detection |
| Skewness & kurtosis | Distribution assumptions |
| Percentiles & box plots | EDA, feature engineering |

---

## 1.2 Measures of Central Tendency

### Mean (Arithmetic Average)

```
         1   n
  x̄  =  ─── Σ  xᵢ
         n  i=1
```

- Sensitive to outliers.
- Used in least-squares loss (MSE minimises distance to the mean).

### Median

The middle value when data is sorted. For even n, it is the average of the two
middle values.

```
Sorted:  2  4  [5  7]  9  12      ← n = 6 (even)
Median = (5 + 7) / 2 = 6
```

- Robust to outliers — preferred for skewed data (e.g., income, house prices).

### Mode

The most frequently occurring value. Can be multi-modal.

- Useful for categorical features (e.g., most common category).

### Example 1 — Compute All Three

```
Data: 3, 7, 7, 2, 9, 1, 7

Mean   = (3+7+7+2+9+1+7) / 7 = 36 / 7 ≈ 5.14
Median = sorted → 1 2 3 [7] 7 7 9 → 7
Mode   = 7 (appears 3 times)
```

### Python

```python
import numpy as np
from scipy import stats

data = [3, 7, 7, 2, 9, 1, 7]
print("Mean:  ", np.mean(data))       # 5.142...
print("Median:", np.median(data))     # 7.0
print("Mode:  ", stats.mode(data, keepdims=False).mode)  # 7
```

---

## 1.3 Measures of Spread (Dispersion)

### Range

```
Range = max(x) − min(x)
```

Simple but heavily influenced by outliers.

### Interquartile Range (IQR)

```
IQR = Q3 − Q1
```

Where Q1 = 25th percentile, Q3 = 75th percentile. Captures the middle 50 % of
data; basis for the box-plot outlier rule.

### Variance

```
           1    n
  σ²  =  ───  Σ  (xᵢ − x̄)²        ← population variance
           n  i=1

           1     n
  s²  =  ─────  Σ  (xᵢ − x̄)²      ← sample variance (Bessel's correction)
          n−1   i=1
```

### Standard Deviation

```
σ = √σ²       s = √s²
```

Same units as the data — more interpretable than variance.

### Example 2 — Spread

```
Data: 4, 8, 6, 5, 3

Mean     = 26/5 = 5.2
Deviations: −1.2, 2.8, 0.8, −0.2, −2.2
Squared:     1.44, 7.84, 0.64, 0.04, 4.84  → Sum = 14.80

Population variance σ² = 14.80 / 5 = 2.96
Sample variance     s² = 14.80 / 4 = 3.70
Sample SD           s  = √3.70 ≈ 1.92
```

---

## 1.4 Percentiles & Quartiles

The **p-th percentile** is the value below which p % of observations fall.

| Percentile | Name |
|---|---|
| 25th | Q1 (first quartile) |
| 50th | Q2 (median) |
| 75th | Q3 (third quartile) |

```python
import numpy as np
data = [12, 15, 18, 22, 25, 30, 35, 40, 45, 50]
print(np.percentile(data, [25, 50, 75]))  # [17.25, 27.5, 41.25]
```

---

## 1.5 Skewness & Kurtosis

### Skewness — Asymmetry of the distribution

```
             1   n  ⎛ xᵢ − x̄ ⎞³
  Skew  =   ─── Σ  ⎜────────⎟
             n  i=1 ⎝   s    ⎠
```

| Value | Interpretation |
|---|---|
| Skew ≈ 0 | Symmetric |
| Skew > 0 | Right-skewed (long right tail) |
| Skew < 0 | Left-skewed (long left tail) |

**ML tip:** Many algorithms (linear regression, PCA) assume roughly symmetric
features. Apply `log` or `sqrt` transforms to reduce positive skew.

### Kurtosis — Tailedness

```
             1   n  ⎛ xᵢ − x̄ ⎞⁴
  Kurt  =   ─── Σ  ⎜────────⎟   − 3      (excess kurtosis)
             n  i=1 ⎝   s    ⎠
```

| Value | Shape |
|---|---|
| Kurt ≈ 0 | Mesokurtic (Normal-like tails) |
| Kurt > 0 | Leptokurtic (heavy tails, more outliers) |
| Kurt < 0 | Platykurtic (light tails, fewer outliers) |

---

## 1.6 Box Plot Interpretation (ASCII)

```
  Outlier          Q1    Median    Q3          Outlier
    o         ┌─────────┬─────────┐
  ──┼─────────┤         │         ├─────────┼──
    o         └─────────┴─────────┘
         ◄─ Whisker ─►      ◄─ Whisker ─►
              1.5×IQR            1.5×IQR

  │◄──── Lower fence ────►│◄──── Upper fence ────►│

  Lower fence = Q1 − 1.5 × IQR
  Upper fence = Q3 + 1.5 × IQR
  Points beyond fences = outliers
```

### Example 3 — Box Plot Values

```
Sorted data: 1, 3, 5, 7, 8, 12, 14, 16, 45

Q1 = 4,  Q3 = 15,  IQR = 11
Lower fence = 4 − 16.5 = −12.5
Upper fence = 15 + 16.5 = 31.5

Outlier: 45 (above upper fence)
```

```python
import matplotlib.pyplot as plt
data = [1, 3, 5, 7, 8, 12, 14, 16, 45]
plt.boxplot(data, vert=False)
plt.title("Box Plot Example")
plt.show()
```

---

## 1.7 Data Summarization for ML

| Task | Statistic Used |
|---|---|
| Missing-value imputation | Mean (symmetric) or Median (skewed) |
| Feature scaling (StandardScaler) | Mean & SD |
| Feature scaling (MinMaxScaler) | Min & Max |
| Outlier detection | IQR rule, z-score (> 3 SD) |
| Feature selection | Variance threshold |
| Target analysis | Skewness → log-transform |

### Python — Quick EDA Summary

```python
import pandas as pd

df = pd.read_csv("data.csv")
print(df.describe())         # count, mean, std, min, 25%, 50%, 75%, max
print(df.skew())              # skewness per column
print(df.kurtosis())          # kurtosis per column
```

---

## 1.8 Real-World ML Applications

1. **Feature engineering:** Knowing the spread helps decide normalisation
   strategy (StandardScaler vs RobustScaler).
2. **Outlier removal:** IQR or z-score filters before training tree-based or
   linear models.
3. **Target transform:** Skewed regression targets (house prices) are
   log-transformed to satisfy normality assumptions.
4. **Class imbalance detection:** Mode of target variable reveals dominant
   class in classification.

---

## Summary Table

| Measure | Formula / Key Idea | Sensitive to Outliers? |
|---|---|---|
| Mean | Σxᵢ / n | Yes |
| Median | Middle value | No |
| Mode | Most frequent | No |
| Range | max − min | Yes |
| IQR | Q3 − Q1 | No |
| Variance | Σ(xᵢ−x̄)² / (n−1) | Yes |
| Std Dev | √Variance | Yes |
| Skewness | 3rd standardised moment | Somewhat |
| Kurtosis | 4th standardised moment − 3 | Somewhat |

---

## Quick Revision Questions

1. Why do we divide by (n − 1) instead of n when computing sample variance?
2. A dataset has mean = 50, median = 35. Is it left-skewed or right-skewed?
3. What is the IQR rule for detecting outliers?
4. When should you impute missing values with the median instead of the mean?
5. How does `StandardScaler` use descriptive statistics internally?
6. A feature has zero variance — what does this imply and what should you do?

---

*← Previous: [Probability Distributions](../03-Probability-and-Distributions/) | Next: [Sampling & Estimation](02-sampling-and-estimation.md) →*
