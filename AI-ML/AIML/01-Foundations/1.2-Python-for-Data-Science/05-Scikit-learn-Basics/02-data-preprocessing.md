# 📘 Chapter 2: Data Preprocessing

> **Unit 5: Scikit-learn Basics** · Foundations of AI/ML

---

## 🎯 Overview

Raw data rarely comes in a format suitable for machine learning. **Data preprocessing** transforms raw features into a form that algorithms can work with effectively. This chapter covers feature scaling, categorical encoding, and combining transformations using `ColumnTransformer`.

---

## 1. Feature Scaling

### Why Scale?

- **Distance-based algorithms** (KNN, SVM, K-Means) are sensitive to feature magnitudes
- **Gradient descent** converges faster when features are on similar scales
- **Tree-based models** (Random Forest, XGBoost) generally do NOT require scaling

### 1.1 StandardScaler — Z-score Normalization

Transforms features to have **mean=0** and **std=1**: `z = (x - μ) / σ`

```python
from sklearn.preprocessing import StandardScaler
import numpy as np

X = np.array([[1, 100], [2, 200], [3, 300], [4, 400]])

scaler = StandardScaler()
X_scaled = scaler.fit_transform(X)

print(X_scaled.mean(axis=0))  # [0., 0.]  — zero mean
print(X_scaled.std(axis=0))   # [1., 1.]  — unit variance

# Learned attributes
print(scaler.mean_)   # [2.5, 250.]
print(scaler.scale_)  # [1.118, 111.803]
```

### 1.2 MinMaxScaler — Range [0, 1]

Scales features to a given range (default 0–1): `x_scaled = (x - x_min) / (x_max - x_min)`

```python
from sklearn.preprocessing import MinMaxScaler

scaler = MinMaxScaler(feature_range=(0, 1))
X_scaled = scaler.fit_transform(X)

print(X_scaled.min(axis=0))  # [0., 0.]
print(X_scaled.max(axis=0))  # [1., 1.]

# Custom range
scaler_custom = MinMaxScaler(feature_range=(-1, 1))
X_custom = scaler_custom.fit_transform(X)
```

### 1.3 RobustScaler — Median / IQR (Outlier-Resistant)

Uses **median** and **interquartile range (IQR)** — robust to outliers.

```python
from sklearn.preprocessing import RobustScaler

# Data with outliers
X_outliers = np.array([[1], [2], [3], [4], [1000]])

robust = RobustScaler()
X_robust = robust.fit_transform(X_outliers)

print(robust.center_)  # [3.]  — median
print(robust.scale_)   # [2.]  — IQR (Q3 - Q1)

# Compare with StandardScaler (badly affected by outlier)
standard = StandardScaler()
X_standard = standard.fit_transform(X_outliers)
print(f"Robust range:   {X_robust.min():.2f} to {X_robust.max():.2f}")
print(f"Standard range: {X_standard.min():.2f} to {X_standard.max():.2f}")
```

### 1.4 MaxAbsScaler — Scale by Maximum Absolute Value

Scales each feature by its maximum absolute value. Preserves sparsity.

```python
from sklearn.preprocessing import MaxAbsScaler

scaler = MaxAbsScaler()
X_scaled = scaler.fit_transform(X)
# Each feature is now in [-1, 1]; good for sparse data
```

### 1.5 Comparison Table

| Scaler | Formula | Best For | Handles Outliers? |
|---|---|---|---|
| `StandardScaler` | `(x - μ) / σ` | General purpose, Gaussian-like data | ❌ No |
| `MinMaxScaler` | `(x - min) / (max - min)` | Neural networks, bounded ranges | ❌ No |
| `RobustScaler` | `(x - median) / IQR` | Data with outliers | ✅ Yes |
| `MaxAbsScaler` | `x / max(\|x\|)` | Sparse data | ❌ No |

---

## 2. Advanced Transformations

### 2.1 PowerTransformer — Make Data More Gaussian

```python
from sklearn.preprocessing import PowerTransformer

# Box-Cox (requires strictly positive data)
pt_boxcox = PowerTransformer(method='box-cox')

# Yeo-Johnson (handles zero and negative values)
pt_yj = PowerTransformer(method='yeo-johnson', standardize=True)
X_transformed = pt_yj.fit_transform(X)

print(pt_yj.lambdas_)  # learned lambda for each feature
```

### 2.2 QuantileTransformer — Uniform or Gaussian Output

```python
from sklearn.preprocessing import QuantileTransformer

# Transform to uniform distribution [0, 1]
qt_uniform = QuantileTransformer(output_distribution='uniform', random_state=42)

# Transform to Gaussian distribution
qt_normal = QuantileTransformer(output_distribution='normal', random_state=42)
X_normal = qt_normal.fit_transform(X)
```

---

## 3. Categorical Encoding

### 3.1 LabelEncoder — Encode Target Labels

```python
from sklearn.preprocessing import LabelEncoder

le = LabelEncoder()
labels = ['cat', 'dog', 'bird', 'cat', 'dog']

y_encoded = le.fit_transform(labels)
print(y_encoded)         # [1, 2, 0, 1, 2]
print(le.classes_)       # ['bird', 'cat', 'dog']

# Inverse transform
y_original = le.inverse_transform(y_encoded)
print(y_original)        # ['cat', 'dog', 'bird', 'cat', 'dog']
```

> ⚠️ **Note:** `LabelEncoder` is for target variables only, not features. For features, use `OrdinalEncoder` or `OneHotEncoder`.

### 3.2 OrdinalEncoder — Ordinal Features

```python
from sklearn.preprocessing import OrdinalEncoder

# Data with a natural order
X_size = np.array([['small'], ['medium'], ['large'], ['medium']])

# Specify the order explicitly
oe = OrdinalEncoder(categories=[['small', 'medium', 'large']])
X_encoded = oe.fit_transform(X_size)
print(X_encoded)  # [[0.], [1.], [2.], [1.]]
```

### 3.3 OneHotEncoder — Nominal Features

```python
from sklearn.preprocessing import OneHotEncoder

X_color = np.array([['red'], ['blue'], ['green'], ['red']])

# sparse_output=False returns a dense array
ohe = OneHotEncoder(sparse_output=False, handle_unknown='ignore')
X_encoded = ohe.fit_transform(X_color)
print(X_encoded)
# [[0. 0. 1.]   ← red
#  [1. 0. 0.]   ← blue
#  [0. 1. 0.]   ← green
#  [0. 0. 1.]]  ← red

print(ohe.categories_)       # [array(['blue', 'green', 'red'])]
print(ohe.get_feature_names_out())  # ['x0_blue', 'x0_green', 'x0_red']

# Handle unseen categories at prediction time
X_new = np.array([['yellow']])
print(ohe.transform(X_new))  # [[0. 0. 0.]] — all zeros with handle_unknown='ignore'
```

### 3.4 Drop First (Avoid Multicollinearity)

```python
ohe_drop = OneHotEncoder(sparse_output=False, drop='first')
X_encoded = ohe_drop.fit_transform(X_color)
# One fewer column per feature (k-1 columns for k categories)
```

---

## 4. ColumnTransformer — Mixed Feature Types

Real datasets have both numerical and categorical columns. `ColumnTransformer` applies different transformers to different columns.

```python
import pandas as pd
from sklearn.compose import ColumnTransformer
from sklearn.preprocessing import StandardScaler, OneHotEncoder

# Sample DataFrame with mixed types
df = pd.DataFrame({
    'age': [25, 30, 35, 40],
    'salary': [50000, 60000, 70000, 80000],
    'city': ['NYC', 'LA', 'NYC', 'SF'],
    'gender': ['M', 'F', 'M', 'F']
})

X = df.values  # or pass DataFrame directly

# Define transformations for each column group
preprocessor = ColumnTransformer(
    transformers=[
        ('num', StandardScaler(), ['age', 'salary']),
        ('cat', OneHotEncoder(sparse_output=False), ['city', 'gender'])
    ],
    remainder='passthrough'  # or 'drop' (default)
)

# When using DataFrame, pass column names
X_processed = preprocessor.fit_transform(df)
print(preprocessor.get_feature_names_out())
```

### Using Column Indices Instead of Names

```python
preprocessor = ColumnTransformer(
    transformers=[
        ('num', StandardScaler(), [0, 1]),        # first two columns
        ('cat', OneHotEncoder(), [2, 3])           # last two columns
    ]
)
```

### Selecting Columns by dtype

```python
from sklearn.compose import make_column_selector

preprocessor = ColumnTransformer(
    transformers=[
        ('num', StandardScaler(), make_column_selector(dtype_include=np.number)),
        ('cat', OneHotEncoder(), make_column_selector(dtype_include=object))
    ]
)
```

---

## 5. Golden Rule: Fit on Train Only

```python
from sklearn.model_selection import train_test_split

X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2)

scaler = StandardScaler()

# ✅ CORRECT: fit on training data, transform both
X_train_scaled = scaler.fit_transform(X_train)
X_test_scaled  = scaler.transform(X_test)

# ❌ WRONG: fitting on test data causes data leakage
# X_test_scaled = scaler.fit_transform(X_test)  # NEVER do this!

# ❌ WRONG: fitting on all data before split
# scaler.fit(X)  # leaks test information into scaling params
```

---

## 6. When to Scale (Quick Guide)

| Algorithm | Scaling Needed? | Reason |
|---|---|---|
| Linear/Logistic Regression | ✅ Yes | Regularization depends on scale |
| SVM | ✅ Yes | Distance-based kernel calculations |
| KNN | ✅ Yes | Distance calculations |
| K-Means | ✅ Yes | Distance-based clustering |
| Neural Networks | ✅ Yes | Gradient descent convergence |
| Decision Trees | ❌ No | Splits are invariant to scale |
| Random Forest | ❌ No | Tree-based — scale invariant |
| XGBoost / LightGBM | ❌ No | Tree-based — scale invariant |
| Naive Bayes | ❌ No | Probability-based |

---

## 📋 Summary Table

| Tool | Purpose | Key Parameters |
|---|---|---|
| `StandardScaler` | Z-score normalization | `with_mean`, `with_std` |
| `MinMaxScaler` | Scale to [0, 1] | `feature_range` |
| `RobustScaler` | Outlier-robust scaling | `with_centering`, `quantile_range` |
| `LabelEncoder` | Encode target labels | — |
| `OrdinalEncoder` | Encode ordinal features | `categories` |
| `OneHotEncoder` | Binary columns per category | `sparse_output`, `handle_unknown`, `drop` |
| `ColumnTransformer` | Different transforms per column | `transformers`, `remainder` |
| `PowerTransformer` | Make data Gaussian-like | `method` (`box-cox`, `yeo-johnson`) |

---

## ❓ Revision Questions

1. **When would you choose `RobustScaler` over `StandardScaler`?**
2. **What does `handle_unknown='ignore'` do in `OneHotEncoder`?**
3. **Why should you only `fit` the scaler on training data and not on the entire dataset?**
4. **What is the difference between `LabelEncoder` and `OrdinalEncoder`?**
5. **How does `ColumnTransformer` help when your dataset has both numerical and categorical features?**
6. **When would you use `PowerTransformer` instead of `StandardScaler`?**

---

## 🧭 Navigation

| Previous | Up | Next |
|---|---|---|
| [← 01 - Estimator API](01-estimator-api.md) | [Unit 5: Scikit-learn Basics](../README.md) | [03 - Train-Test Split →](03-train-test-split.md) |
