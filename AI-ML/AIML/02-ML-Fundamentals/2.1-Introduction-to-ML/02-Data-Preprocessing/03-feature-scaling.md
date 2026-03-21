# ⚖️ Chapter 3: Feature Scaling

> **Module:** Data Preprocessing · **Topic:** Normalization, Standardization & Robust Scaling
> **Objective:** Understand why and how to scale features, and correctly apply scaling in ML pipelines.

---

## 📌 1. Why Feature Scaling Matters

Many ML algorithms compute **distances** or use **gradient descent** — features on different scales dominate calculations unfairly.

```
  Without Scaling                      With Scaling
  ┌─────────────────────────┐          ┌─────────────────────────┐
  │ Income: 20,000 - 150,000│          │ Income:  0.0  -  1.0    │
  │ Age:         18 - 65    │          │ Age:     0.0  -  1.0    │
  └─────────────────────────┘          └─────────────────────────┘
  Distance dominated by Income!        Both features contribute equally
```

| Problem Without Scaling | Affected Algorithms |
|---|---|
| Gradient descent converges slowly | Linear/Logistic Regression, Neural Nets |
| Distance calculations biased | KNN, K-Means, SVM (RBF) |
| Regularization penalizes unevenly | Ridge, Lasso, Elastic Net |
| Feature importance misleading | PCA (variance-based) |

---

## 📌 2. Normalization (Min-Max Scaling)

Rescales features to **[0, 1]**.

### Formula
```
                X  -  X_min
  X_norm  =  ─────────────────
               X_max  -  X_min
```

### Step-by-Step Example
```
  Feature "Age":  [18, 25, 35, 45, 65]    X_min=18, X_max=65

  Age=18 → (18-18)/(65-18) = 0/47  = 0.000
  Age=25 → (25-18)/(65-18) = 7/47  = 0.149
  Age=35 → (35-18)/(65-18) = 17/47 = 0.362
  Age=45 → (45-18)/(65-18) = 27/47 = 0.574
  Age=65 → (65-18)/(65-18) = 47/47 = 1.000
```

```python
from sklearn.preprocessing import MinMaxScaler
scaler = MinMaxScaler()
X_train_scaled = scaler.fit_transform(X_train)   # fit + transform on train
X_test_scaled  = scaler.transform(X_test)         # transform ONLY on test
```

**Use when:** Neural networks (sigmoid/tanh), image pixels (0-255→0-1), bounded range needed.
**Weakness:** Sensitive to outliers — they compress the majority of values.

---

## 📌 3. Standardization (Z-Score Scaling)

Centers data to **mean = 0**, scales to **std = 1**.

### Formula
```
               X  -  μ
  X_std  =  ───────────
                 σ
```

### Step-by-Step Example
```
  Feature "Salary": [30000, 45000, 50000, 55000, 70000]

  μ = 50000
  σ = √[((−20000)² + (−5000)² + 0² + 5000² + 20000²) / 5] ≈ 13038.4

  30000 → (30000-50000)/13038.4 = -1.534
  45000 → (45000-50000)/13038.4 = -0.384
  50000 → (50000-50000)/13038.4 =  0.000
  55000 → (55000-50000)/13038.4 =  0.384
  70000 → (70000-50000)/13038.4 =  1.534
```

```python
from sklearn.preprocessing import StandardScaler
scaler = StandardScaler()
X_train_scaled = scaler.fit_transform(X_train)
X_test_scaled  = scaler.transform(X_test)
print("Mean:", scaler.mean_, "Std:", scaler.scale_)
```

**Use when:** Linear/Logistic Regression, SVM, PCA, regularized models (Ridge, Lasso).
**Output range:** Unbounded (typically -3 to +3 for normal data).

---

## 📌 4. Robust Scaling

Uses **median** and **IQR** — resistant to outliers.

### Formula
```
                  X  -  median
  X_robust  =  ─────────────────
                     IQR            where IQR = Q3 - Q1
```

### Step-by-Step Example
```
  Feature "Income": [25000, 40000, 48000, 55000, 200000]  ← outlier
  Median=48000, Q1=40000, Q3=55000, IQR=15000

  25000  → (25000-48000)/15000  = -1.533
  40000  → (40000-48000)/15000  = -0.533
  48000  → (48000-48000)/15000  =  0.000
  55000  → (55000-48000)/15000  =  0.467
  200000 → (200000-48000)/15000 = 10.133  ← outlier preserved, not compressed
```

```python
from sklearn.preprocessing import RobustScaler
scaler = RobustScaler()
X_train_scaled = scaler.fit_transform(X_train)
X_test_scaled  = scaler.transform(X_test)
```

**Use when:** Significant outliers you want to keep, heavy-tailed distributions.

---

## 📌 5. Comparison Table

| Method | Formula | Range | Outlier Robust? | Best Use Case |
|---|---|---|---|---|
| **Min-Max** | `(X-Xmin)/(Xmax-Xmin)` | [0, 1] | ❌ No | Neural nets, bounded inputs |
| **Z-Score** | `(X-μ)/σ` | ~[-3, +3] | ❌ No | Linear models, PCA, SVM |
| **Robust** | `(X-median)/IQR` | Unbounded | ✅ Yes | Data with outliers |

### Visual Impact — Data: `[1, 2, 3, 4, 100]` (outlier at 100)
- **Min-Max:** `[0.00, 0.01, 0.02, 0.03, 1.00]` — everything squashed near 0
- **Z-Score:** `[-0.76, -0.73, -0.71, -0.69, 2.89]` — outlier distorts mean/std
- **Robust:** `[-0.50, 0.00, 0.50, 1.00, 49.00]` — majority well-separated

---

## 📌 6. Algorithms — Scaling Needed vs Not Needed

| Needs Scaling ⚠️ | Does NOT Need Scaling ✅ |
|---|---|
| Linear / Logistic Regression | Decision Trees |
| SVM (all kernels) | Random Forest |
| KNN, K-Means | XGBoost, LightGBM, CatBoost |
| Neural Networks, PCA | Naive Bayes |
| Ridge / Lasso / Elastic Net | Rule-based models |

**Why tree models skip scaling:** They split on thresholds, not distances or gradients.

---

## 📌 7. Critical Rule — Fit on Train, Transform Both

```
  ⚠️  scaler.fit(X_train)       ← learn params from train ONLY
      scaler.transform(X_train) ← apply to train
      scaler.transform(X_test)  ← apply SAME params to test
      NEVER fit on test data or full dataset — causes data leakage!
```

---

## 📌 8. Effect on Gradient Descent

```
  Without Scaling (elongated)        With Scaling (circular)
       w2                                 w2
       │   ╱──────╲                       │    ╱─╲
       │  ╱  ╱──╲  ╲ ← slow zigzag       │   ╱ · ╲ ← direct path
       │ ╱  ╱ ·  ╲  ╲                     │   ╲   ╱
       └────────────────── w1             └────────────── w1
  Many iterations                    Fewer iterations
```

---

## 📌 9. Complete Pipeline Example

```python
from sklearn.model_selection import train_test_split
from sklearn.preprocessing import StandardScaler, MinMaxScaler, RobustScaler
from sklearn.pipeline import Pipeline
from sklearn.linear_model import LogisticRegression

X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2, random_state=42)

# Pipeline ensures fit happens ONLY on training data — no leakage
pipeline = Pipeline([('scaler', StandardScaler()), ('model', LogisticRegression(max_iter=1000))])
pipeline.fit(X_train, y_train)
print(f"Accuracy: {pipeline.score(X_test, y_test):.4f}")
```

---

## 📌 10. Real-World ML Applications

| Domain | Scaling Choice | Reason |
|---|---|---|
| **Image Classification** | Min-Max (÷ 255) | Pixels naturally bounded [0, 255] |
| **Credit Scoring** | Robust Scaler | Income/debt contain outliers |
| **Recommendation Systems** | Standard Scaler | Ratings need zero-centering for SVD |
| **Medical Diagnosis** | Standard or Robust | Lab values vary in scale |

---

## 📋 Summary Table

| Concept | Key Takeaway |
|---|---|
| Why scale | Equal feature contribution; faster convergence |
| Min-Max | Maps to [0,1]; sensitive to outliers |
| Z-Score | Centers at 0, std=1; good for ~normal data |
| Robust | Uses median/IQR; resistant to outliers |
| Tree models | Don't need scaling (split-based) |
| Fit on train only | Prevents data leakage; use `Pipeline` |

---

## ❓ Revision Questions

1. **Why does KNN require scaling but Random Forest does not?** Explain the fundamental difference.
2. **Given [10, 20, 30, 40, 50], compute the Min-Max scaled values.** Show work step by step.
3. **A feature has μ=100, σ=25. Value=150.** What is the Z-score? Is it unusual?
4. **Why is RobustScaler preferred over StandardScaler** when outliers are present? Illustrate.
5. **Explain data leakage in feature scaling.** What is the correct procedure to avoid it?
6. **Building an SVM on features: age (18-65), income (20k-500k), children (0-5).** Which scaler?

---

## 🧭 Navigation

| ⬅️ Previous | ➡️ Next |
|---|---|
| [02 · Handling Missing Values](02-handling-missing-values.md) | [04 · Encoding Categorical Variables](04-encoding-categorical-variables.md) |

> **Pro Tip:** Always use `sklearn.pipeline.Pipeline` to chain scaler and model — this guarantees
> scaling params are learned from training data only, even during cross-validation.
