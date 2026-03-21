# Chapter 5: Feature Engineering

> **Feature Engineering** is the process of creating new input features — or transforming existing ones — to improve model performance. It is often the single most impactful step in a machine learning pipeline, leveraging domain knowledge to extract signals that raw data hides.

---

## 5.1 Why Feature Engineering Matters

```
Raw Data  ──▶  Engineered Features  ──▶  Better Model Performance

"Applied ML is basically feature engineering."  — Andrew Ng
```

| Aspect | Without FE | With FE |
|--------|-----------|---------|
| Model accuracy | Baseline | Significantly higher |
| Training time | May be longer (noisy features) | Often shorter (cleaner signal) |
| Interpretability | Low | Higher (meaningful features) |

---

## 5.2 Domain Knowledge-Driven Features

The most powerful features come from **understanding the problem domain**, not from algorithms.

```
┌──────────────────────────────────────────────────────────────┐
│  Domain            Raw Columns          Engineered Feature   │
├──────────────────────────────────────────────────────────────┤
│  Real Estate       length, width        area = l × w         │
│  Finance           income, debt         debt_to_income ratio │
│  E-commerce        purchase_date,       days_since_purchase  │
│                    current_date                              │
│  Healthcare        weight(kg),          BMI = w / h²         │
│                    height(m)                                 │
│  Ride-sharing      pickup_time          is_rush_hour (0/1)   │
└──────────────────────────────────────────────────────────────┘
```

```python
import pandas as pd

df = pd.DataFrame({
    'weight_kg': [70, 85, 60],
    'height_m':  [1.75, 1.80, 1.60]
})

# Domain-driven: Body Mass Index
df['BMI'] = df['weight_kg'] / (df['height_m'] ** 2)
print(df['BMI'].round(1))  # [22.9, 26.2, 23.4]
```

---

## 5.3 Polynomial Features

Captures **non-linear relationships** by generating powers and cross-products of features.

```
For features x₁, x₂ with degree 2:
  Generated:  x₁,  x₂,  x₁²,  x₁·x₂,  x₂²
  Count = C(n + d, d) = (n + d)! / (d! × n!)
```

```python
from sklearn.preprocessing import PolynomialFeatures
import numpy as np

X = np.array([[2, 3], [4, 5]])

poly = PolynomialFeatures(degree=2, include_bias=False)
X_poly = poly.fit_transform(X)
print(poly.get_feature_names_out())
# ['x0', 'x1', 'x0^2', 'x0 x1', 'x1^2']
```

> ⚠️ High-degree polynomials cause **combinatorial explosion** and overfitting. Degree 2 is most common.

---

## 5.4 Interaction Features

Captures the **combined effect** of two features that neither captures alone.

```python
import pandas as pd

df = pd.DataFrame({
    'age': [25, 45, 30], 'income': [40000, 90000, 55000], 'is_urban': [1, 0, 1]
})

df['age_x_income']   = df['age'] * df['income']
df['income_x_urban'] = df['income'] * df['is_urban']
```

---

## 5.5 Date/Time Features

Raw timestamps are rarely useful on their own — extracting calendar and cyclical components unlocks powerful patterns.

```
Timestamp: 2024-07-15 14:30:00
                │
    ┌───────────┼───────────────────────────────┐
    ▼           ▼           ▼         ▼         ▼
  year=2024  month=7    day=15   hour=14   minute=30
              │                    │
         quarter=3           is_afternoon=1
         is_summer=1         is_rush_hour=0
                             day_of_week=0 (Mon)
                             is_weekend=0
```

```python
import pandas as pd

df = pd.DataFrame({
    'timestamp': pd.to_datetime([
        '2024-01-15 08:30', '2024-07-20 22:15',
        '2024-12-25 14:00', '2024-03-10 06:45'
    ])
})

df['year']        = df['timestamp'].dt.year
df['month']       = df['timestamp'].dt.month
df['day_of_week'] = df['timestamp'].dt.dayofweek     # Mon=0, Sun=6
df['hour']        = df['timestamp'].dt.hour
df['is_weekend']  = (df['day_of_week'] >= 5).astype(int)
df['quarter']     = df['timestamp'].dt.quarter

season_map = {1:'Winter', 2:'Winter', 3:'Spring', 4:'Spring',
              5:'Summer', 6:'Summer', 7:'Monsoon', 8:'Monsoon',
              9:'Autumn', 10:'Autumn', 11:'Winter', 12:'Winter'}
df['season'] = df['month'].map(season_map)
```

### Cyclical Encoding for Time Features

Hours and months are **cyclical** — hour 23 is close to hour 0. Encode using sine/cosine:

```
x_sin = sin(2π × value / max_value)
x_cos = cos(2π × value / max_value)
```

```python
import numpy as np

df['hour_sin'] = np.sin(2 * np.pi * df['hour'] / 24)
df['hour_cos'] = np.cos(2 * np.pi * df['hour'] / 24)
```

---

## 5.6 Text Features

When a column contains free text, engineer numeric features before feeding it to a model.

```
Text: "Machine learning is amazing!"
       │
       ├── char_count   = 30
       ├── word_count    = 4
       ├── avg_word_len  = 6.5
       ├── has_exclam    = 1
       └── TF-IDF vector = [0.0, 0.41, 0.41, 0.41, 0.41]
```

```python
import pandas as pd
from sklearn.feature_extraction.text import TfidfVectorizer

df = pd.DataFrame({'review': [
    'Great product, loved it!', 'Terrible quality, very bad.',
    'Average product, okay value.'
]})

df['char_count'] = df['review'].str.len()
df['word_count'] = df['review'].str.split().str.len()
df['has_exclam'] = df['review'].str.contains('!').astype(int)

# TF-IDF(t,d) = TF(t,d) × log(N / DF(t))
tfidf = TfidfVectorizer(max_features=5)
tfidf_matrix = tfidf.fit_transform(df['review'])
print(tfidf.get_feature_names_out())
```

---

## 5.7 Aggregation Features (Group-By Statistics)

Compute **statistics within groups** — especially useful for transactional data.

```
Customer C1: 3 txns → mean_amt=66.7, max_amt=120, std=47.3
Customer C2: 2 txns → mean_amt=140,  max_amt=200, std=84.9
```

```python
import pandas as pd

txn = pd.DataFrame({
    'customer_id': ['C1', 'C1', 'C1', 'C2', 'C2'],
    'amount':      [50, 120, 30, 200, 80]
})

agg_features = txn.groupby('customer_id')['amount'].agg(
    txn_count='count', mean_amt='mean', max_amt='max', std_amt='std'
).reset_index()

result = txn.merge(agg_features, on='customer_id', how='left')
```

---

## 5.8 Log / Square Root Transformations

Many real-world features (income, price, population) are **right-skewed**. Log/sqrt compresses the tail toward a Gaussian shape.

```
Common Transforms:  log(x), √x, x^(1/3), Box-Cox(x, λ)
Use log1p(x) = log(1+x) when data contains zeros
```

```python
import numpy as np
import pandas as pd

df = pd.DataFrame({'salary': [30000, 45000, 120000, 500000, 1200000]})

# log1p handles zero values: log1p(x) = log(1 + x)
df['salary_log']  = np.log1p(df['salary'])
df['salary_sqrt'] = np.sqrt(df['salary'])

# Box-Cox (requires positive values)
from scipy.stats import boxcox
df['salary_boxcox'], best_lambda = boxcox(df['salary'])
```

---

## 5.9 Automated Feature Engineering

For large relational datasets, tools like **Featuretools** auto-generate features via Deep Feature Synthesis (DFS).

```python
# Conceptual example (pip install featuretools)
import featuretools as ft

es = ft.EntitySet(id='customers')
es.add_dataframe(dataframe=transactions_df, dataframe_name='transactions',
                 index='txn_id', time_index='date')

feature_matrix, feature_defs = ft.dfs(
    entityset=es, target_dataframe_name='transactions', max_depth=2)
```

> 🔑 Automated FE is a starting point — always **validate** with domain knowledge and remove noisy features.

---

## 5.10 Best Practices and Common Pitfalls

| ✅ DO | ✗ AVOID |
|-------|---------|
| Start with domain understanding | Blindly generating all combos |
| Validate with feature importance | Including leaky features |
| Use cross-validation to test impact | Polynomial degree > 3 |
| Fit transforms on train set only | Fitting transformers on test set |
| Keep features interpretable | Over-engineering (curse of dimensionality) |

### Data Leakage Warning

```
  WRONG: fit_transform(ALL data) → split     RIGHT: split → fit(train) → transform(both)
```

```python
from sklearn.model_selection import train_test_split
from sklearn.preprocessing import PolynomialFeatures

X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2)

poly = PolynomialFeatures(degree=2)
X_train_poly = poly.fit_transform(X_train)   # fit on train
X_test_poly  = poly.transform(X_test)        # only transform test
```

---

## 5.11 Summary Table

| Technique | When to Use | Watch Out For |
|-----------|-------------|---------------|
| **Domain features** | Always — first step | Requires domain expertise |
| **Polynomial** | Non-linear relationships | Explosion with high degree |
| **Interaction** | Combined variable effects | Grows as O(n²) pairs |
| **Date/Time** | Temporal patterns | Cyclical nature (use sin/cos) |
| **Text features** | NLP / text columns | High dimensionality with TF-IDF |
| **Aggregations** | Transactional / grouped data | Leakage if not time-aware |
| **Log/Sqrt** | Right-skewed distributions | Cannot apply log to negatives |
| **Automated (DFS)** | Large relational datasets | Generates noisy features |

---

## 5.12 Revision Questions

1. **What is feature engineering, and why does Andrew Ng call it the most important part of applied ML?**
2. **Given features `length` and `width`, write code to generate all polynomial features up to degree 2. How many features are produced?**
3. **Explain why cyclical encoding (sin/cos) is better than raw integer encoding for the `hour` feature.**
4. **You have a `timestamp` column for retail sales. List at least 5 features you would engineer from it and explain the intuition behind each.**
5. **What is data leakage in feature engineering? Give an example and explain how to prevent it.**
6. **A dataset has a highly right-skewed `income` column with some zero values. Which transformation would you use and why?**

---

## 5.13 Real-World ML Applications

| Domain | Raw Data | Engineered Feature | Impact |
|--------|----------|--------------------|--------|
| Fraud Detection | transaction_time | hour, is_night, txn_frequency_1h | Night transactions flag fraud |
| Recommendation | user_id, item_id | avg_rating_per_user, items_viewed_count | Captures user behavior |
| Autonomous Driving | GPS coordinates | speed, heading_change, distance_to_stop | Driving pattern signals |
| Retail Forecasting | sale_date | day_of_week, is_holiday, days_to_payday | Seasonal demand patterns |
| Credit Scoring | loan_amount, income | debt_to_income_ratio | Core risk indicator |

---

[⬅️ Previous: 04-encoding-categorical-variables.md](04-encoding-categorical-variables.md) | [➡️ Next: 06-feature-selection.md](06-feature-selection.md)
