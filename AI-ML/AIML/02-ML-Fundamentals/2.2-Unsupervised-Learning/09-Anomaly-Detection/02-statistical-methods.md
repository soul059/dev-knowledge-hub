# 📊 Statistical Methods for Anomaly Detection

> **Chapter 9.2 — Z-Score, IQR, Grubbs' Test, and Gaussian Models**

---

## 📌 Overview

Statistical methods are the **simplest and most interpretable** approaches to anomaly detection. They work by assuming a statistical distribution for normal data and flagging points that have **low probability** under that distribution. These methods are fast, require no training, and serve as excellent baselines before deploying more complex algorithms.

---

## 📐 1. Z-Score Method

### Definition

The Z-score measures how many **standard deviations** a point is from the mean:

```
        xᵢ - μ
zᵢ  =  ─────────
           σ

where:  μ = mean of the data
        σ = standard deviation
```

### Decision Rule

```
┌──────────────────────────────────────────────────────────────┐
│                                                              │
│  If |zᵢ| > threshold → ANOMALY                              │
│                                                              │
│  Common thresholds:                                          │
│  • |z| > 2    → ~5% of data flagged (lenient)                │
│  • |z| > 2.5  → ~1.2% flagged                               │
│  • |z| > 3    → ~0.3% flagged (strict)                       │
│                                                              │
│  Distribution:                                               │
│                                                              │
│    ◄─ Anomaly ─►  ◄──── Normal ────►  ◄─ Anomaly ─►         │
│                   │                │                          │
│   ▁▂▃▅█████████████████████████████▅▃▂▁                      │
│                   │                │                          │
│         -3σ  -2σ  -σ   μ   +σ  +2σ  +3σ                     │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

### Modified Z-Score (Robust)

The standard Z-score is sensitive to outliers (mean and std are affected). Use the **Modified Z-Score** with median and MAD:

```
                  0.6745 × (xᵢ - median)
Modified zᵢ  =  ─────────────────────────
                         MAD

where MAD = median(|xᵢ - median(X)|)   (Median Absolute Deviation)
```

Threshold: |Modified z| > 3.5 (recommended by Iglewicz & Hoaglin)

### Python Implementation

```python
import numpy as np

def zscore_anomaly_detection(data, threshold=3.0):
    """Standard Z-score anomaly detection."""
    mean = np.mean(data)
    std = np.std(data)
    z_scores = np.abs((data - mean) / std)
    anomalies = z_scores > threshold
    return anomalies, z_scores

def modified_zscore_anomaly_detection(data, threshold=3.5):
    """Modified Z-score using median and MAD (robust to outliers)."""
    median = np.median(data)
    mad = np.median(np.abs(data - median))
    if mad == 0:
        mad = np.mean(np.abs(data - median))  # Fallback
    modified_z = 0.6745 * (data - median) / mad
    anomalies = np.abs(modified_z) > threshold
    return anomalies, modified_z

# Example
np.random.seed(42)
normal_data = np.random.normal(50, 5, 100)
data = np.append(normal_data, [100, 10, 95])  # Add outliers

anomalies_z, z_scores = zscore_anomaly_detection(data, threshold=3)
anomalies_mz, mz_scores = modified_zscore_anomaly_detection(data, threshold=3.5)

print(f"Z-score anomalies: {np.where(anomalies_z)[0]}")
print(f"Modified Z-score anomalies: {np.where(anomalies_mz)[0]}")
```

---

## 📐 2. IQR (Interquartile Range) Method

### Definition

```
  Q1 = 25th percentile (first quartile)
  Q3 = 75th percentile (third quartile)
  IQR = Q3 - Q1

  Lower Bound = Q1 - 1.5 × IQR
  Upper Bound = Q3 + 1.5 × IQR

  Any point outside [Lower Bound, Upper Bound] → ANOMALY
```

### Visual Explanation (Box Plot)

```
     Anomalies   ◄── Normal Range ──►   Anomalies
         ★        │                │        ★   ★
         │        │                │        │   │
  ───────┼────────┼════════════════┼────────┼───┼──────
         │     Lower    Q1    Q3   Upper    │   │
         │     Fence    │  IQR │   Fence    │   │
         │              ├──────┤            │   │
         │              │      │            │   │
    Q1-1.5·IQR          │      │      Q3+1.5·IQR
                        │      │
                        ▼      ▼
                     Box Plot Whiskers
```

### Why 1.5 × IQR?

```
For normal distribution:
  IQR ≈ 1.35σ
  1.5 × IQR ≈ 2.025σ
  
  Q1 - 1.5·IQR ≈ μ - 2.7σ
  Q3 + 1.5·IQR ≈ μ + 2.7σ
  
  → Captures ~99.3% of data (flags ~0.7%)

For extreme outliers, use 3 × IQR (flags ~0.002%)
```

### Python Implementation

```python
import numpy as np

def iqr_anomaly_detection(data, factor=1.5):
    """IQR-based anomaly detection."""
    Q1 = np.percentile(data, 25)
    Q3 = np.percentile(data, 75)
    IQR = Q3 - Q1
    
    lower_bound = Q1 - factor * IQR
    upper_bound = Q3 + factor * IQR
    
    anomalies = (data < lower_bound) | (data > upper_bound)
    
    print(f"Q1={Q1:.2f}, Q3={Q3:.2f}, IQR={IQR:.2f}")
    print(f"Bounds: [{lower_bound:.2f}, {upper_bound:.2f}]")
    
    return anomalies, lower_bound, upper_bound

# Example
anomalies_iqr, lb, ub = iqr_anomaly_detection(data)
print(f"IQR anomalies: indices {np.where(anomalies_iqr)[0]}")
print(f"Anomaly values: {data[anomalies_iqr]}")
```

---

## 📐 3. Grubbs' Test

### Definition

Tests whether the **most extreme value** is an outlier, assuming data follows a normal distribution:

```
         max|xᵢ - x̄|
G  =  ─────────────────
             s

where x̄ = sample mean, s = sample std

Critical value: Compare G with tabulated values at significance α
```

### Python Implementation

```python
from scipy import stats
import numpy as np

def grubbs_test(data, alpha=0.05):
    """
    Grubbs' test for a single outlier.
    Returns: is_outlier (bool), outlier_index, G_statistic, critical_value
    """
    n = len(data)
    mean = np.mean(data)
    std = np.std(data, ddof=1)
    
    # Find the point furthest from mean
    abs_deviations = np.abs(data - mean)
    max_idx = np.argmax(abs_deviations)
    G = abs_deviations[max_idx] / std
    
    # Critical value (two-sided)
    t_critical = stats.t.ppf(1 - alpha / (2 * n), n - 2)
    G_critical = ((n - 1) / np.sqrt(n)) * np.sqrt(t_critical**2 / (n - 2 + t_critical**2))
    
    is_outlier = G > G_critical
    
    return is_outlier, max_idx, G, G_critical

# Example
is_outlier, idx, G, G_crit = grubbs_test(data)
print(f"Most extreme value: {data[idx]:.2f} at index {idx}")
print(f"G statistic: {G:.4f}, Critical value: {G_crit:.4f}")
print(f"Is outlier: {is_outlier}")
```

---

## 📐 4. Mahalanobis Distance (Multivariate)

### The Problem with Euclidean Distance

```
  Euclidean Distance:                Mahalanobis Distance:
  ┌──────────────────┐              ┌──────────────────┐
  │     ●            │              │     ●            │
  │     │ d=3        │              │     │ d_M=1.2    │
  │     ▼            │              │     ▼            │
  │  ●●●●●●●●●      │              │  ●●●●●●●●●      │
  │  ●●●●●●●●●      │              │  ╱●●●●●●●●●╲    │
  │  ●●●●●●●●●      │              │ ╱ ●●●●●●●●● ╲   │
  │  ●●●●●●●●●      │              │  ╲●●●●●●●●●╱    │
  │     ▲            │              │   ╲●●●●●●●╱     │
  │     │ d=3        │              │     ▲            │
  │     ●            │              │     │ d_M=3.5    │
  │                  │              │     ●            │
  │  Same Euclidean  │              │  Different       │
  │  distance, but   │              │  Mahalanobis     │
  │  different       │              │  distances —     │
  │  "outlierness"   │              │  CORRECT!        │
  └──────────────────┘              └──────────────────┘

  Mahalanobis accounts for the SHAPE and CORRELATION of the data!
```

### Formula

```
d_M(x) = √[(x - μ)ᵀ Σ⁻¹ (x - μ)]

where:  μ = mean vector (center of data)
        Σ = covariance matrix (shape of data)
        Σ⁻¹ = inverse covariance matrix
```

### Threshold

For d-dimensional data assuming multivariate Gaussian:

```
d_M² follows a Chi-squared distribution with d degrees of freedom

Threshold: d_M² > χ²_{d, α}

Example (d=2, α=0.01): threshold = χ²(2, 0.01) = 9.21
  → d_M > √9.21 ≈ 3.03
```

### Python Implementation

```python
import numpy as np
from scipy.spatial.distance import mahalanobis
from scipy import stats

def mahalanobis_anomaly_detection(X, alpha=0.01):
    """
    Mahalanobis distance-based anomaly detection.
    Assumes multivariate Gaussian distribution.
    """
    n, d = X.shape
    mean = np.mean(X, axis=0)
    cov = np.cov(X, rowvar=False)
    cov_inv = np.linalg.inv(cov)
    
    # Compute Mahalanobis distance for each point
    distances = np.array([
        mahalanobis(x, mean, cov_inv) for x in X
    ])
    
    # Chi-squared threshold
    threshold = np.sqrt(stats.chi2.ppf(1 - alpha, d))
    anomalies = distances > threshold
    
    return anomalies, distances, threshold

# Example: 2D data
np.random.seed(42)
X_normal = np.random.multivariate_normal([0, 0], [[1, 0.8], [0.8, 1]], 200)
X_outliers = np.array([[3, -2], [-3, 3], [4, 4]])
X = np.vstack([X_normal, X_outliers])

anomalies, distances, threshold = mahalanobis_anomaly_detection(X, alpha=0.01)
print(f"Threshold: {threshold:.3f}")
print(f"Anomalies detected: {anomalies.sum()}")
print(f"Anomaly indices: {np.where(anomalies)[0]}")
```

### Robust Mahalanobis (sklearn)

```python
from sklearn.covariance import EllipticEnvelope

# EllipticEnvelope fits a robust Gaussian and uses Mahalanobis distance
detector = EllipticEnvelope(contamination=0.05, random_state=42)
y_pred = detector.fit_predict(X)  # 1=normal, -1=anomaly

n_anomalies = (y_pred == -1).sum()
print(f"Anomalies detected: {n_anomalies}")
```

---

## 📊 Comparison of Statistical Methods

```
┌──────────────────────┬────────────┬───────────┬─────────────┬──────────┐
│ Method               │ Univariate │ Multi-    │ Robust to   │ Assumes  │
│                      │            │ variate   │ Outliers    │ Normal   │
├──────────────────────┼────────────┼───────────┼─────────────┼──────────┤
│ Z-Score              │ ✅         │ ❌        │ ❌          │ ✅       │
│ Modified Z-Score     │ ✅         │ ❌        │ ✅          │ ~        │
│ IQR                  │ ✅         │ ❌        │ ✅          │ ❌       │
│ Grubbs' Test         │ ✅         │ ❌        │ ❌          │ ✅       │
│ Mahalanobis Distance │ ❌         │ ✅        │ ❌          │ ✅       │
│ Elliptic Envelope    │ ❌         │ ✅        │ ✅          │ ✅       │
└──────────────────────┴────────────┴───────────┴─────────────┴──────────┘
```

---

## 📋 Summary Table

| Method | Formula | When to Use |
|--------|---------|-------------|
| **Z-Score** | (x - μ) / σ, threshold: \|z\| > 3 | Quick univariate check, Gaussian data |
| **Modified Z-Score** | 0.6745(x - median) / MAD | Univariate, robust to existing outliers |
| **IQR** | Outside Q1-1.5·IQR to Q3+1.5·IQR | Univariate, no distribution assumption |
| **Grubbs' Test** | max\|xᵢ - x̄\| / s | Test single most extreme point |
| **Mahalanobis** | √(x-μ)ᵀΣ⁻¹(x-μ) | Multivariate Gaussian data |
| **Elliptic Envelope** | Robust Mahalanobis | Multivariate, robust to contamination |

---

## ❓ Revision Questions

1. **Compare Z-score and Modified Z-score. Why is the modified version more robust? When would you prefer each?**
   Discuss the effect of outliers on mean/std vs. median/MAD.

2. **Given data [2, 3, 4, 5, 5, 6, 7, 8, 50], identify outliers using both the Z-score (threshold=3) and IQR (factor=1.5) methods. Do they agree?**
   Show all calculations step by step.

3. **Why does Euclidean distance fail for anomaly detection in correlated multivariate data? How does Mahalanobis distance fix this?**
   Use a visual example with elongated (correlated) clusters.

4. **Derive the relationship between the IQR method and the Z-score method for normally distributed data.**
   Show that Q1-1.5·IQR ≈ μ - 2.7σ for normal distribution.

5. **A dataset has heavy-tailed (non-Gaussian) distribution. Which statistical methods would still work? Which would fail?**
   Discuss assumptions and distribution-free methods.

6. **Implement a complete anomaly detection pipeline using Mahalanobis distance on a 5-dimensional dataset. Include visualization of the results.**
   Address covariance estimation, threshold selection, and how to plot results.

---

## 🧭 Navigation

| Previous | Up | Next |
|----------|-----|------|
| [← Types of Anomalies](./01-types-of-anomalies.md) | [📂 Unsupervised Learning](../README.md) | [Isolation Forest →](./03-isolation-forest.md) |

---

> **Key Takeaway:** Statistical methods provide fast, interpretable baselines for anomaly detection. Use Z-score or IQR for univariate data, Mahalanobis distance for multivariate Gaussian data, and robust variants (Modified Z-score, Elliptic Envelope) when the data may already contain outliers. These methods work well as first-pass filters before applying more complex ML methods.
