# 🔧 Chapter 2: Handling Missing Values

> **Module:** Data Preprocessing · **Topic:** Missing Data Detection, Analysis & Imputation
> **Objective:** Understand types of missingness, choose the right strategy, and implement it in Python.

---

## 📌 1. Why Missing Data Matters

Missing values can **bias** predictions, **reduce** sample size, and **break** algorithms that cannot
handle NaN (SVM, KNN, most sklearn estimators).

```
  Complete Dataset          With Missing Values
  ┌───┬───┬───┐            ┌───┬───┬───┐
  │ A │ B │ C │            │ A │ B │ C │
  ├───┼───┼───┤            ├───┼───┼───┤
  │ 1 │ 5 │ 9 │            │ 1 │ ? │ 9 │  ← missing
  │ 2 │ 6 │ 8 │            │ 2 │ 6 │ ? │  ← missing
  │ 3 │ 7 │ 7 │            │ ? │ 7 │ 7 │  ← missing
  └───┴───┴───┘            └───┴───┴───┘
```

## 📌 2. Types of Missing Data

| Type | Full Name | Meaning | Example |
|---|---|---|---|
| **MCAR** | Missing Completely At Random | No relationship to any variable | Lab equipment randomly fails |
| **MAR** | Missing At Random | Depends on **observed** variables | Young patients skip BP readings |
| **MNAR** | Missing Not At Random | Depends on the **missing value itself** | High earners refuse to report salary |

```
  MCAR ──▶ Safe to delete rows (no bias)
  MAR  ──▶ Impute using related observed features
  MNAR ──▶ Requires domain knowledge; imputation may still bias
```

---

## 📌 3. Detecting Missing Values

```python
import pandas as pd
df = pd.read_csv('data.csv')

print(df.isnull().sum())                              # count per column
print((df.isnull().sum() / len(df) * 100).round(2))   # percentage
print("Total missing:", df.isnull().sum().sum())
```

```python
import missingno as msno
msno.matrix(df)       # white gaps = missing
msno.bar(df)          # bar chart of non-null counts
msno.heatmap(df)      # correlation of missingness between columns
```

---

## 📌 4. Deletion Strategies

### 4.1 Listwise Deletion (Complete Case)
```python
df_clean = df.dropna()              # drop rows with ANY missing
df_clean = df.dropna(thresh=3)      # keep rows with ≥3 non-null values
```

| Pros | Cons |
|---|---|
| Simple to implement | Significant data loss |
| No imputation bias | Only valid under MCAR |

### 4.2 Column Deletion
```python
# Drop columns with more than 50% missing
df_clean = df.loc[:, df.isnull().mean() < 0.5]
```
**Rule of thumb:** If >60-70% missing, consider dropping unless critically important.

---

## 📌 5. Imputation Strategies

### 5.1 Simple Statistical Imputation

| Method | Best For | Formula |
|---|---|---|
| **Mean** | Normally distributed numerical data | `x_fill = (Σxᵢ) / n` |
| **Median** | Skewed numerical data (robust to outliers) | Middle value |
| **Mode** | Categorical data | Most frequent value |

```python
df['age'].fillna(df['age'].mean(), inplace=True)
df['salary'].fillna(df['salary'].median(), inplace=True)
df['city'].fillna(df['city'].mode()[0], inplace=True)
```

### 5.2 Forward / Backward Fill (Time Series)
```python
df['temp'].fillna(method='ffill', inplace=True)   # propagate last valid forward
df['temp'].fillna(method='bfill', inplace=True)   # use next valid to fill back
```

### 5.3 Sklearn SimpleImputer
```python
from sklearn.impute import SimpleImputer

num_imputer = SimpleImputer(strategy='median')
df[['age', 'salary']] = num_imputer.fit_transform(df[['age', 'salary']])

cat_imputer = SimpleImputer(strategy='most_frequent')
df[['city']] = cat_imputer.fit_transform(df[['city']])
```

---

## 📌 6. Advanced Imputation

### 6.1 KNN Imputer
Fills missing values using the **k nearest neighbors** based on available features.
```
  Row with missing 'salary':  age=30, exp=5, salary=?
    ├── Neighbor 1: age=31, exp=5, salary=55000  (dist=1.0)
    ├── Neighbor 2: age=29, exp=4, salary=52000  (dist=1.4)
    └── Neighbor 3: age=30, exp=6, salary=58000  (dist=1.0)
  Imputed salary = mean(55000, 52000, 58000) = 55,000
```
```python
from sklearn.impute import KNNImputer
knn_imputer = KNNImputer(n_neighbors=5, weights='distance')
df_imputed = pd.DataFrame(knn_imputer.fit_transform(df), columns=df.columns)
```

### 6.2 MICE (Multiple Imputation by Chained Equations)
Iteratively models each feature with missing values as a function of the others.
```
  Iter 1:  Impute A using B,C → Impute B using A,C → Impute C using A,B
  Iter 2:  Re-impute A using updated B,C → ...repeat until convergence
```
```python
from sklearn.experimental import enable_iterative_imputer
from sklearn.impute import IterativeImputer
mice = IterativeImputer(max_iter=10, random_state=42)
df_imputed = pd.DataFrame(mice.fit_transform(df), columns=df.columns)
```

---

## 📌 7. Impact on ML Models

| Model | Handles NaN? | Recommended Strategy |
|---|---|---|
| **Linear / Logistic Regression** | ❌ | Mean/median or MICE |
| **SVM** | ❌ | KNN imputer |
| **KNN** | ❌ | Median imputation |
| **Decision Trees** | ⚠️ Some | Can handle, but imputation helps |
| **XGBoost / LightGBM** | ✅ | Built-in handling; evaluate imputation |
| **Neural Networks** | ❌ | MICE or KNN imputer |

---

## 📌 8. Decision Flowchart

```
                ┌──────────────────────┐
                │ Column > 60% missing?│
                └─────────┬────────────┘
                     Yes/ \No
                       /   \
            ┌─────────┐     ┌──────────────────┐
            │  DROP    │     │ Rows missing < 5%?│
            │  column  │     └────────┬─────────┘
            └─────────┘         Yes/ \No
                                  /   \
                       ┌─────────┐     ┌──────────────┐
                       │ Listwise│     │ Data is MAR? │
                       │ deletion│     └──────┬───────┘
                       └─────────┘       Yes/ \No(MCAR)
                                          /   \
                              ┌──────────┐     ┌────────┐
                              │ KNN /    │     │ Mean / │
                              │ MICE     │     │ Median │
                              └──────────┘     └────────┘
```

---

## 📌 9. Complete Pipeline Example

```python
from sklearn.model_selection import train_test_split
from sklearn.impute import KNNImputer
from sklearn.pipeline import Pipeline
from sklearn.linear_model import LogisticRegression

df = pd.read_csv('patients.csv')
X = df.drop('diagnosis', axis=1)
y = df['diagnosis']

# ⚠️ Split BEFORE imputation to prevent data leakage
X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2, random_state=42)

pipeline = Pipeline([
    ('imputer', KNNImputer(n_neighbors=5)),
    ('classifier', LogisticRegression(max_iter=1000))
])
pipeline.fit(X_train, y_train)
print(f"Accuracy: {pipeline.score(X_test, y_test):.4f}")
```

---

## 📋 Summary Table

| Strategy | Type | Best When | Watch Out For |
|---|---|---|---|
| Listwise deletion | Deletion | MCAR, <5% missing | Data loss |
| Column deletion | Deletion | >60% column missing | Info loss |
| Mean / Median | Simple | Quick baseline | Reduces variance |
| Mode | Simple | Categorical features | Over-represents dominant class |
| Forward/Backward fill | Sequential | Time-series data | Not for non-sequential data |
| KNN Imputer | Advanced | MAR, local structure | Expensive on large data |
| MICE | Advanced | Multiple correlated missing | Slow; assumes linearity |

---

## ❓ Revision Questions

1. **Explain MCAR, MAR, and MNAR** with a real-world example of each.
2. **Why does mean imputation reduce variance** of the imputed feature? What downstream effect?
3. **Column `income` has 8% missing values and is right-skewed.** Which imputation method and why?
4. **What is data leakage in imputation?** Give an example of incorrect vs correct procedure.
5. **Compare KNN Imputer and MICE.** When would you prefer one over the other?
6. **A time-series column has 3% MCAR missing values.** What strategy per the decision flowchart?

---

## 🧭 Navigation

| ⬅️ Previous | ➡️ Next |
|---|---|
| [01 · Data Collection and Exploration](01-data-collection-and-exploration.md) | [03 · Feature Scaling](03-feature-scaling.md) |

---
> **Pro Tip:** Before choosing any imputation method, visualize the missing data pattern using
> `missingno.matrix()`. The pattern often reveals whether data is MCAR, MAR, or MNAR.
