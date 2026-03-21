# 📈 Regression Metrics

## Chapter Overview
Regression metrics measure how close predicted **continuous values** are to actual values. Unlike classification (right or wrong), regression errors exist on a spectrum. This chapter covers all essential regression metrics, their mathematical formulations, trade-offs, and practical Python implementation.

---

## 1. Mean Squared Error (MSE)
```
MSE = (1/n) Σᵢ₌₁ⁿ (yᵢ - ŷᵢ)²
```
| Property | Detail |
|----------|--------|
| Range | [0, ∞) — Units: squared target — Best: 0 |

- Squares each error → **penalizes large errors disproportionately**
- Differentiable everywhere → ideal for gradient-based optimization
- Sensitive to outliers (a single large error dominates the sum)
```
  Error:   1    2    1    1   10
  Squared: 1    4    1    1  100   ← outlier dominates
  MSE = (1+4+1+1+100)/5 = 21.4
```

---

## 2. Root Mean Squared Error (RMSE)
```
RMSE = √MSE = √[ (1/n) Σᵢ₌₁ⁿ (yᵢ - ŷᵢ)² ]
```
| Property | Detail |
|----------|--------|
| Range | [0, ∞) — Units: **same as target** — Best: 0 |

- RMSE is interpretable: if predicting house prices in dollars, RMSE is also in dollars
- Still penalizes large errors (inherits from MSE)
- Most commonly reported regression metric in practice

---

## 3. Mean Absolute Error (MAE)
```
MAE = (1/n) Σᵢ₌₁ⁿ |yᵢ - ŷᵢ|
```
| Property | Detail |
|----------|--------|
| Range | [0, ∞) — Units: same as target — Best: 0 |

- Treats all errors equally (linear penalty) — **more robust to outliers** than MSE/RMSE
- Not differentiable at zero (use sub-gradients in optimization)
```
  Errors:  [1, 2, 1, 1, 10]
  MAE  = (1+2+1+1+10)/5  =  3.0
  MSE  = (1+4+1+1+100)/5 = 21.4   ← 7x larger due to outlier
```

---

## 4. R² — Coefficient of Determination
```
R² = 1 - (SS_res / SS_tot)
  SS_res = Σᵢ (yᵢ - ŷᵢ)²       (Residual Sum of Squares)
  SS_tot = Σᵢ (yᵢ - ȳ)²        (Total Sum of Squares, ȳ = mean of y)
```
| Property | Detail |
|----------|--------|
| Range | (-∞, 1] — Best: 1 — Baseline: 0 (predicting the mean) |

**Interpretation:**
- **R² = 1.0** → model explains all variance (perfect prediction)
- **R² = 0.0** → model is no better than predicting the mean
- **R² < 0** → model is **worse** than predicting the mean
```
  Total Variance (SS_tot)
  ├──────────────────────────────────┤
  │  Explained by model  │ Residual │
  │     (SS_tot - SS_res)│ (SS_res) │
  ├──────────────────────┤──────────┤
                         ↑
                   R² = explained fraction
```
> **Caution:** R² always increases when adding features, even irrelevant ones. Use **Adjusted R²**.

---

## 5. Adjusted R²
```
Adjusted R² = 1 - [(1 - R²)(n - 1) / (n - p - 1)]
  n = number of samples,  p = number of features (predictors)
```
Adding an irrelevant feature slightly increases R² but the penalty term grows, potentially **decreasing** Adjusted R². This helps with **feature selection**.

| Scenario | R² | Adjusted R² | Interpretation |
|----------|----|-------------|----------------|
| Good feature added | ↑ | ↑ | Feature helps |
| Useless feature added | ↑ slightly | ↓ | Feature is noise |

---

## 6. Mean Absolute Percentage Error (MAPE)
```
MAPE = (100/n) Σᵢ₌₁ⁿ |yᵢ - ŷᵢ| / |yᵢ|
```
| Property | Detail |
|----------|--------|
| Range | [0, ∞) — Units: % — Best: 0% |

**Advantages:** Scale-independent → easy to communicate ("model is off by 5% on average").

**⚠️ Limitations:** Undefined when yᵢ = 0 (division by zero). Asymmetric — penalizes over-predictions more than under-predictions. Not suitable for targets crossing zero.

---

## 7. Explained Variance Score
```
Explained Variance = 1 - Var(y - ŷ) / Var(y)
```
**Difference from R²:** Does **not** account for systematic bias. If predictions are shifted by a constant, R² penalizes this but Explained Variance does not.
```
  y_true = [3, 5, 7, 9],  y_pred = [5, 7, 9, 11]  ← shifted by +2
  R² < 1  (penalizes offset)  |  Explained Variance = 1.0 (pattern is perfect)
```

---

## 8. Comparing Metrics — Decision Guide
```
  ┌──────────────┬───────────────────────────────────┐
  │ Outliers?    │  YES → MAE    │  NO → RMSE        │
  │ Need %?      │  YES → MAPE (if no zeros)         │
  │ Compare?     │  YES → R² / Adjusted R²           │
  │ Feature sel? │  YES → Adjusted R²                │
  │ Loss func?   │  → MSE (smooth gradient)          │
  └──────────────┴───────────────────────────────────┘
```
| Metric | Outlier Robust | Interpretable Units | Differentiable | Scale-Free |
|--------|:-:|:-:|:-:|:-:|
| MSE | ✗ | ✗ (squared) | ✓ | ✗ |
| RMSE | ✗ | ✓ | ✓ | ✗ |
| MAE | ✓ | ✓ | ✗ (at 0) | ✗ |
| R² | ✗ | — | ✓ | ✓ |
| MAPE | ✓ | ✓ (%) | ✗ (at 0) | ✓ |

---

## 9. Step-by-Step Calculation Example
| i | yᵢ (actual) | ŷᵢ (predicted) | Error (yᵢ - ŷᵢ) |
|---|-------------|----------------|-------------------|
| 1 | 3.0 | 2.5 | 0.5 |
| 2 | 5.0 | 4.8 | 0.2 |
| 3 | 7.0 | 7.5 | -0.5 |
| 4 | 9.0 | 8.0 | 1.0 |
| 5 | 2.0 | 2.1 | -0.1 |

```
MAE  = (0.5 + 0.2 + 0.5 + 1.0 + 0.1) / 5                    = 0.46
MSE  = (0.25 + 0.04 + 0.25 + 1.00 + 0.01) / 5                = 0.31
RMSE = √0.31                                                  = 0.5568

ȳ = (3+5+7+9+2)/5 = 5.2
SS_tot = (3-5.2)² + (5-5.2)² + (7-5.2)² + (9-5.2)² + (2-5.2)²
       = 4.84 + 0.04 + 3.24 + 14.44 + 10.24                  = 32.80
SS_res = 1.55
R²     = 1 - 1.55/32.80 = 1 - 0.0473                         = 0.9527
```
> The model explains **95.27%** of the variance in the target variable.

---

## 10. Python Implementation
```python
from sklearn.metrics import (
    mean_squared_error, mean_absolute_error,
    r2_score, explained_variance_score,
    mean_absolute_percentage_error
)
import numpy as np

y_true = np.array([3.0, 5.0, 7.0, 9.0, 2.0])
y_pred = np.array([2.5, 4.8, 7.5, 8.0, 2.1])

mse  = mean_squared_error(y_true, y_pred)
rmse = np.sqrt(mse)
mae  = mean_absolute_error(y_true, y_pred)
r2   = r2_score(y_true, y_pred)
evs  = explained_variance_score(y_true, y_pred)
mape = mean_absolute_percentage_error(y_true, y_pred)

print(f"MSE:  {mse:.4f}   RMSE: {rmse:.4f}   MAE: {mae:.4f}")
print(f"R²:   {r2:.4f}   Expl.Var: {evs:.4f}   MAPE: {mape:.4f}")

# Adjusted R²
n, p = len(y_true), 2
adj_r2 = 1 - ((1 - r2) * (n - 1)) / (n - p - 1)
print(f"Adjusted R²: {adj_r2:.4f}")

# Comparing two models
y_pred_b = np.array([3.1, 5.2, 6.8, 8.5, 2.3])
for name, preds in [("Model A", y_pred), ("Model B", y_pred_b)]:
    print(f"{name}: RMSE={np.sqrt(mean_squared_error(y_true, preds)):.4f}, "
          f"R²={r2_score(y_true, preds):.4f}, "
          f"MAE={mean_absolute_error(y_true, preds):.4f}")
```

---

## 11. Real-World ML Applications
| Domain | Metric | Reason |
|--------|--------|--------|
| House price prediction | RMSE, R² | Interpretable error in dollars |
| Stock price forecasting | MAPE | Percentage error across price scales |
| Weather temperature | MAE | Robust to occasional extreme errors |
| Energy consumption | RMSE | Penalize large forecast misses |
| Drug dosage prediction | MAE | Every unit of error matters equally |
| Model comparison / papers | R², Adjusted R² | Standardized, scale-free comparison |

---

## 📋 Summary Table
| Metric | Formula | Units | Outlier Sensitive | Key Use Case |
|--------|---------|-------|:-:|----------------|
| MSE | (1/n)Σ(y-ŷ)² | Squared | ✓ | Loss function |
| RMSE | √MSE | Same as y | ✓ | Standard reporting |
| MAE | (1/n)Σ\|y-ŷ\| | Same as y | ✗ | Robust evaluation |
| R² | 1-SS_res/SS_tot | Unitless | ✓ | Variance explained |
| Adj. R² | Penalized R² | Unitless | ✓ | Feature selection |
| MAPE | (100/n)Σ\|e/y\| | % | ✗ | Scale-free comparison |
| Expl. Var. | 1-Var(e)/Var(y) | Unitless | ✓ | Bias-unaware fit |

---

## ❓ Revision Questions
1. **Why does MSE penalize large errors more than MAE? Show with a numerical example using errors [1, 2, 10].**
2. **A model has R²=0.85. Another has R²=0.87 but uses 15 additional features. Which would you prefer and why? What metric helps decide?**
3. **Calculate RMSE and R² for: y_true=[10, 20, 30, 40], y_pred=[12, 18, 33, 37]. Show all steps.**
4. **Why is MAPE undefined or misleading when actual values are near zero? Give an example.**
5. **A model predicts house prices with MAE=$15,000 and RMSE=$45,000. What does the large gap between MAE and RMSE tell you about the error distribution?**
6. **Explain why R² can be negative. What does a negative R² mean about your model compared to a simple baseline?**

---

## 🧭 Navigation
| | Link |
|------|------|
| ⬅️ Previous | [Classification Metrics](01-classification-metrics.md) |
| ➡️ Next | [Confusion Matrix](03-confusion-matrix.md) |
| 📂 Up | [Model Evaluation](README.md) |
