# Chapter 5: Missing Data Handling

> **Unit 3 – Pandas** | Detecting, understanding, and handling missing values — a critical skill for ML pipelines.

---

## 1. Overview

Missing data is inevitable in real-world datasets. How you handle it directly impacts model performance. Pandas represents missing data as `NaN` (float), `None` (object), or `pd.NaT` (datetime). The strategy you choose (drop, fill, or impute) depends on the **type** and **pattern** of missingness.

```python
import pandas as pd
import numpy as np

df = pd.DataFrame({
    'name': ['Alice', 'Bob', None, 'Diana', 'Eve'],
    'age': [25, np.nan, 35, 28, np.nan],
    'salary': [50000, 60000, np.nan, np.nan, 65000],
    'dept': ['HR', 'IT', 'HR', None, 'IT']
})
```

---

## 2. Types of Missing Data

Understanding **why** data is missing determines the correct handling strategy.

| Type | Full Name | Meaning | Example |
|---|---|---|---|
| **MCAR** | Missing Completely At Random | Missingness has no relationship to any data | Sensor randomly fails |
| **MAR** | Missing At Random | Missingness depends on observed data | Younger people skip salary question |
| **MNAR** | Missing Not At Random | Missingness depends on the missing value itself | High earners don't report income |

- **MCAR** → Safe to drop rows (no bias introduced).
- **MAR** → Impute using related observed variables.
- **MNAR** → Requires domain knowledge; simple imputation can introduce bias.

---

## 3. Detecting Missing Values

```python
# Check for missing values — returns DataFrame of booleans
df.isnull()       # Same as df.isna()
df.notnull()      # Same as df.notna()

# Count missing per column
df.isnull().sum()

# Count missing per row
df.isnull().sum(axis=1)

# Total missing values in entire DataFrame
df.isnull().sum().sum()

# Percentage of missing per column
(df.isnull().sum() / len(df) * 100).round(2)

# Columns with any missing values
df.columns[df.isnull().any()]

# Rows with any missing values
df[df.isnull().any(axis=1)]

# Visual summary (useful in notebooks)
def missing_summary(df):
    missing = df.isnull().sum()
    percent = (missing / len(df) * 100).round(2)
    summary = pd.DataFrame({'missing': missing, 'percent': percent})
    return summary[summary['missing'] > 0].sort_values('percent', ascending=False)

missing_summary(df)
```

---

## 4. Dropping Missing Values — `dropna()`

```python
# Drop rows with ANY missing value
df_dropped = df.dropna()

# Drop rows where ALL values are missing
df_dropped = df.dropna(how='all')

# Drop rows that have fewer than N non-null values
df_dropped = df.dropna(thresh=3)   # Keep rows with at least 3 non-null values

# Drop based on specific columns only
df_dropped = df.dropna(subset=['name', 'age'])

# Drop columns (instead of rows) with missing values
df_dropped = df.dropna(axis=1)

# Combine parameters
df_dropped = df.dropna(subset=['salary'], how='any')
```

### When to Drop

- Small percentage of missing data (<5%).
- Data is MCAR.
- You have enough data that losing rows won't hurt model performance.

---

## 5. Filling Missing Values — `fillna()`

```python
# Fill with a constant
df['salary'] = df['salary'].fillna(0)
df['dept'] = df['dept'].fillna('Unknown')

# Fill with column statistics
df['age'] = df['age'].fillna(df['age'].mean())
df['salary'] = df['salary'].fillna(df['salary'].median())
df['dept'] = df['dept'].fillna(df['dept'].mode()[0])

# Forward fill — propagate last valid value forward
df['salary'] = df['salary'].fillna(method='ffill')
# or equivalently: df['salary'] = df['salary'].ffill()

# Backward fill — propagate next valid value backward
df['salary'] = df['salary'].fillna(method='bfill')
# or equivalently: df['salary'] = df['salary'].bfill()

# Limit the number of consecutive fills
df['salary'] = df['salary'].fillna(method='ffill', limit=2)

# Fill with a Series or dictionary (different value per column)
fill_values = {'age': 30, 'salary': 55000, 'dept': 'General'}
df = df.fillna(fill_values)

# Fill with another DataFrame (same shape)
df = df.fillna(df.mean(numeric_only=True))
```

---

## 6. Interpolation

Estimates missing values using surrounding data points. Ideal for **time series** and **ordered data**.

```python
# Linear interpolation (default)
df['temperature'] = df['temperature'].interpolate()

# Method options
df['value'].interpolate(method='linear')       # Default — straight line
df['value'].interpolate(method='time')         # For datetime index
df['value'].interpolate(method='polynomial', order=2)  # Polynomial fit
df['value'].interpolate(method='spline', order=3)      # Spline fit

# Limit interpolation to a max number of consecutive NaNs
df['value'].interpolate(limit=3)

# Direction control
df['value'].interpolate(limit_direction='forward')
df['value'].interpolate(limit_direction='backward')
df['value'].interpolate(limit_direction='both')

# Example
s = pd.Series([1, np.nan, np.nan, 4, 5, np.nan, 7])
s.interpolate()
# 0    1.0
# 1    2.0    ← filled
# 2    3.0    ← filled
# 3    4.0
# 4    5.0
# 5    6.0    ← filled
# 6    7.0
```

---

## 7. Imputation Strategies

### 7.1 Simple Imputation (Pandas-only)

```python
# Mean imputation (numeric) — biases variance downward
df['age'] = df['age'].fillna(df['age'].mean())

# Median imputation (numeric) — robust to outliers
df['salary'] = df['salary'].fillna(df['salary'].median())

# Mode imputation (categorical)
df['dept'] = df['dept'].fillna(df['dept'].mode()[0])

# Group-wise imputation — more accurate than global
df['salary'] = df.groupby('dept')['salary'].transform(
    lambda x: x.fillna(x.median())
)
```

### 7.2 Scikit-learn Imputers

```python
from sklearn.impute import SimpleImputer, KNNImputer

# Simple Imputer
imputer = SimpleImputer(strategy='median')  # 'mean', 'median', 'most_frequent', 'constant'
df[['age', 'salary']] = imputer.fit_transform(df[['age', 'salary']])

# KNN Imputer — uses K nearest neighbors to estimate missing values
knn_imputer = KNNImputer(n_neighbors=5)
df[['age', 'salary']] = knn_imputer.fit_transform(df[['age', 'salary']])
```

### 7.3 Strategy Selection Guide

| Strategy | Best For | Pros | Cons |
|---|---|---|---|
| Mean | Normally distributed numeric | Simple | Sensitive to outliers |
| Median | Skewed numeric | Robust to outliers | Ignores relationships |
| Mode | Categorical | Simple | Over-represents common value |
| Forward/Back fill | Time series | Preserves trends | Inappropriate for non-ordered data |
| Interpolation | Ordered numeric | Smooth estimates | Assumes linearity |
| KNN | Structured numeric | Uses feature relationships | Computationally expensive |
| Group-wise | Grouped data | Context-aware | Requires meaningful groups |

---

## 8. Impact on ML Models

```python
# ❌ Most ML models CANNOT handle NaN
from sklearn.linear_model import LinearRegression
# LinearRegression().fit(X_with_nans, y)  → ValueError!

# ✅ Always handle missing data BEFORE model training

# Strategy 1: Drop (only if very few missing values)
df_clean = df.dropna()

# Strategy 2: Impute in a pipeline (recommended)
from sklearn.pipeline import Pipeline
from sklearn.impute import SimpleImputer
from sklearn.linear_model import LogisticRegression

pipeline = Pipeline([
    ('imputer', SimpleImputer(strategy='median')),
    ('model', LogisticRegression())
])
pipeline.fit(X_train, y_train)

# Strategy 3: Add a missing indicator column
df['salary_missing'] = df['salary'].isnull().astype(int)
df['salary'] = df['salary'].fillna(df['salary'].median())
# The indicator tells the model that the value was imputed
```

### Key Considerations

- **Tree-based models** (XGBoost, LightGBM) can handle NaN natively.
- **Linear models** require imputation — NaN breaks them.
- **Always impute AFTER train/test split** to avoid data leakage.
- **Fit imputer on training data only**, then transform both train and test.

---

## 9. Best Practices

| Practice | Recommendation |
|---|---|
| First step | Always compute `missing_summary()` before choosing a strategy |
| Small % missing | Drop rows if MCAR and <5% missing |
| Numeric columns | Use median (robust) or group-wise mean |
| Categorical | Use mode or a dedicated "Unknown" category |
| Time series | Use interpolation or forward fill |
| ML pipeline | Impute inside a `Pipeline` to prevent data leakage |
| Feature engineering | Add `_missing` indicator columns for important features |

---

## 10. Summary Table

| Method | Function | Use Case |
|---|---|---|
| Detect | `isnull()`, `notnull()`, `isna()` | Find missing values |
| Count | `isnull().sum()` | Quantify missingness |
| Drop | `dropna(how, thresh, subset)` | Remove rows/columns with NaN |
| Fill constant | `fillna(value)` | Replace with specific value |
| Fill forward/back | `ffill()`, `bfill()` | Propagate valid values |
| Interpolate | `interpolate(method)` | Estimate from neighbors |
| Impute (sklearn) | `SimpleImputer`, `KNNImputer` | ML-ready imputation |

---

## 11. Revision Questions

1. What are the three types of missing data (MCAR, MAR, MNAR)? Give a real-world example of each.
2. When is it appropriate to simply drop rows with missing values?
3. Why is median imputation preferred over mean imputation for skewed data?
4. Write code to fill missing `salary` values with the median salary of each `department`.
5. What is data leakage in the context of imputation, and how do you prevent it?
6. How does adding a `_missing` indicator column help ML models?

---

[← Previous: Data Cleaning](04-data-cleaning.md) | [Next → Chapter 6: Grouping & Aggregation](06-grouping-and-aggregation.md)
