# Chapter 4: Encoding Categorical Variables

> **Why Encoding?** Machine learning models operate on numerical computations — they cannot interpret text labels like `"Red"`, `"Male"`, or `"Mumbai"`. Encoding transforms categorical features into numeric representations that algorithms can process while preserving the information the categories carry.

---

## 4.1 Types of Categorical Data

```
Categorical Data
├── Nominal (No order)  ──▶  Color: {Red, Blue, Green}
│                             City:  {Delhi, Mumbai, Pune}
│
└── Ordinal (Has order) ──▶  Size:      {S < M < L < XL}
                              Education: {High School < Bachelor < Master < PhD}
```

| Property | Nominal | Ordinal |
|----------|---------|---------|
| Natural order | ✗ No | ✓ Yes |
| Example | Blood group (A, B, O) | Rating (Low, Med, High) |
| Safe encodings | One-Hot, Frequency, Target | Label, Ordinal, One-Hot |

---

## 4.2 Label Encoding

Assigns a unique integer (0, 1, 2, …) to each category **alphabetically** by default.

```
Category  ──▶  Integer
  Apple   ──▶    0
  Banana  ──▶    1
  Cherry  ──▶    2
```

> ⚠️ **Pitfall:** Creates a false ordinal relationship — the model may interpret `Cherry(2) > Banana(1) > Apple(0)`. Use **only for ordinal data** or tree-based models that split on thresholds.

```python
from sklearn.preprocessing import LabelEncoder

data = ['cat', 'dog', 'bird', 'cat', 'bird']

le = LabelEncoder()
encoded = le.fit_transform(data)
print(encoded)                    # [1, 2, 0, 1, 0]
print(le.classes_)                # ['bird', 'cat', 'dog']
print(le.inverse_transform([0]))  # ['bird']
```

---

## 4.3 Ordinal Encoding

Like Label Encoding but you **explicitly define the rank order** so the integers reflect the true hierarchy.

```
Education Level    Rank
─────────────────────────
High School   ──▶   0
Bachelor      ──▶   1
Master        ──▶   2
PhD           ──▶   3

Meaning: PhD(3) > Master(2) > Bachelor(1) — order is meaningful
```

```python
from sklearn.preprocessing import OrdinalEncoder

# Define categories in ascending order
categories = [['High School', 'Bachelor', 'Master', 'PhD']]

oe = OrdinalEncoder(categories=categories)
data = [['Bachelor'], ['PhD'], ['High School'], ['Master']]

encoded = oe.fit_transform(data)
print(encoded)  # [[1.], [3.], [0.], [2.]]
```

---

## 4.4 One-Hot Encoding (OHE)

Creates a **separate binary column** (0 or 1) for each category — safest for **nominal data** (no artificial ordering).

```
 Color         Color_Red  Color_Blue  Color_Green
  Red     ──▶     1          0           0
  Blue    ──▶     0          1           0
  Green   ──▶     0          0           1
```

### 4.4.1 The Dummy Variable Trap

With *k* categories, only *k − 1* columns are needed — the last is **linearly dependent**. Including all *k* causes **multicollinearity** in linear models.

```
If Color_Red = 0 AND Color_Blue = 0  ⟹  Color_Green = 1  (redundant)
```

> Use `drop='first'` (sklearn) or `drop_first=True` (pandas) to avoid this.

```python
import pandas as pd
from sklearn.preprocessing import OneHotEncoder

df = pd.DataFrame({'Color': ['Red', 'Blue', 'Green', 'Red', 'Blue']})

# Method 1: pandas get_dummies
ohe_pd = pd.get_dummies(df, columns=['Color'], drop_first=True, dtype=int)

# Method 2: sklearn OneHotEncoder
encoder = OneHotEncoder(drop='first', sparse_output=False)
encoded = encoder.fit_transform(df[['Color']])
print(encoder.get_feature_names_out())  # ['Color_Green', 'Color_Red']
```

---

## 4.5 Binary Encoding

Converts the integer label into its **binary representation**, then splits each bit into a separate column. Creates only **⌈log₂(k)⌉** columns vs *k* for One-Hot.

```
Category   Integer   Binary   Bit_2  Bit_1  Bit_0
─────────────────────────────────────────────────
Delhi        0        000       0      0      0
Mumbai       1        001       0      0      1
Pune         2        010       0      1      0
Chennai      3        011       0      1      1
Kolkata      4        100       1      0      0

Only 3 columns for 5 categories  (vs 5 for OHE)
```

```python
# pip install category_encoders
import category_encoders as ce
import pandas as pd

df = pd.DataFrame({'City': ['Delhi', 'Mumbai', 'Pune', 'Chennai', 'Kolkata']})
encoder = ce.BinaryEncoder(cols=['City'])
encoded = encoder.fit_transform(df)
print(encoded)
#    City_0  City_1  City_2
# 0       0       0       1
# 1       0       1       0
# 2       0       1       1
# 3       1       0       0
# 4       1       0       1
```

---

## 4.6 Target Encoding (Mean Encoding)

Replaces each category with the **mean of the target variable** for that category.

```
City       Target(Price)     Encoded (Mean Target)
──────────────────────────────────────────────────
Mumbai        80                    75.0
Delhi         60                    55.0
Mumbai        70                    75.0
Delhi         50                    55.0

Mean_Mumbai = (80 + 70) / 2 = 75.0
Mean_Delhi  = (60 + 50) / 2 = 55.0
```

> ⚠️ **Risk of Overfitting:** Categories with few samples get noisy estimates. Use **smoothing** or **K-fold target encoding** to mitigate.

```
                   count(category) × mean(category) + m × global_mean
Smoothed value  =  ─────────────────────────────────────────────────────
                              count(category) + m

where m = smoothing factor (higher m → more regularization)
```

```python
import category_encoders as ce
import pandas as pd

df = pd.DataFrame({
    'City':   ['Mumbai', 'Delhi', 'Mumbai', 'Delhi', 'Pune', 'Pune'],
    'Price':  [80, 60, 70, 50, 40, 45]
})

encoder = ce.TargetEncoder(cols=['City'], smoothing=1.0)
df['City_encoded'] = encoder.fit_transform(df['City'], df['Price'])
print(df[['City', 'City_encoded']])
```

---

## 4.7 Frequency Encoding

Replaces each category with its **count or proportion** in the dataset. Simple, avoids creating extra columns, and captures popularity.

```python
import pandas as pd

df = pd.DataFrame({'Browser': ['Chrome', 'Firefox', 'Chrome', 'Safari',
                                'Chrome', 'Firefox']})

freq_map = df['Browser'].value_counts(normalize=True)
df['Browser_freq'] = df['Browser'].map(freq_map)
# Chrome→0.50, Firefox→0.33, Safari→0.17
```

> ⚠️ **Limitation:** Two different categories with the **same frequency** get the **same encoded value** — the model cannot distinguish them.

---

## 4.8 Handling High Cardinality Features

High cardinality = a feature with many unique categories (e.g., ZIP codes, user IDs).

```
Problem: One-Hot Encoding 10,000 ZIP codes → 10,000 new columns → memory explosion

Solutions:
┌───────────────────────────────────────────────────────┐
│ 1. Frequency / Target Encoding (1 column only)        │
│ 2. Binary Encoding (log₂(k) columns)                 │
│ 3. Grouping: rare categories → "Other"                │
│ 4. Feature hashing (HashingVectorizer)                │
│ 5. Embedding layers (deep learning)                   │
└───────────────────────────────────────────────────────┘
```

```python
# Group rare categories into 'Other'
threshold = 0.01  # categories appearing < 1%
freq = df['ZipCode'].value_counts(normalize=True)
rare = freq[freq < threshold].index
df['ZipCode_grouped'] = df['ZipCode'].replace(rare, 'Other')
```

---

## 4.9 When to Use Each Encoding — Decision Table

| Encoding | Columns Added | Ordinal Safe? | Nominal Safe? | High Cardinality? | Best For |
|----------|:---:|:---:|:---:|:---:|----------|
| **Label** | 0 | ✓ | ✗ | ✓ | Ordinal data, tree models |
| **Ordinal** | 0 | ✓ | ✗ | ✓ | Ordinal data (explicit order) |
| **One-Hot** | k − 1 | ✓ | ✓ | ✗ | Nominal data, low cardinality |
| **Binary** | ⌈log₂k⌉ | ~ | ✓ | ✓ | Medium–high cardinality |
| **Target** | 0 | ✓ | ✓ | ✓ | Supervised tasks, careful tuning |
| **Frequency** | 0 | ~ | ✓ | ✓ | Quick baseline, popularity signal |

```
Decision Flowchart:
                   Is the feature ordinal?
                     /            \
                   Yes             No
                   /                \
          Ordinal Encoding    How many unique values?
                                /          \
                            ≤ 10          > 10
                            /                \
                     One-Hot Encoding    Target / Binary / Frequency
```

---

## 4.10 Complete Pipeline Example

```python
import pandas as pd
from sklearn.preprocessing import OrdinalEncoder, OneHotEncoder
from sklearn.compose import ColumnTransformer

df = pd.DataFrame({
    'Size':  ['S', 'M', 'L', 'XL', 'M'],        # ordinal
    'Color': ['Red', 'Blue', 'Red', 'Green', 'Blue'],  # nominal
    'Price': [10, 20, 15, 25, 18]
})

preprocessor = ColumnTransformer([
    ('ordinal', OrdinalEncoder(categories=[['S', 'M', 'L', 'XL']]), ['Size']),
    ('onehot',  OneHotEncoder(drop='first', sparse_output=False),   ['Color']),
], remainder='passthrough')

result = preprocessor.fit_transform(df)
```

---

## 4.11 Summary Table

| Concept | Key Point |
|---------|-----------|
| Why encode | ML models require numeric input |
| Label Encoding | Integer per category; risk of false order |
| Ordinal Encoding | Explicit rank mapping; ordinal data only |
| One-Hot Encoding | Binary columns; use `drop_first` to avoid multicollinearity |
| Binary Encoding | Bit-level split; ⌈log₂k⌉ columns; memory efficient |
| Target Encoding | Mean of target; powerful but prone to overfitting |
| Frequency Encoding | Count/proportion; simple one-column mapping |
| High Cardinality | Group rare values, use hashing, or embeddings |
| Dummy Variable Trap | k categories need only k − 1 columns in linear models |

---

## 4.12 Revision Questions

1. **Why can't we directly feed categorical text labels into a linear regression model?**
2. **Explain the dummy variable trap. How does `drop_first=True` solve it, and which model types are affected?**
3. **A dataset has a `Temperature` column with values `{Cold, Warm, Hot}`. Which encoding would you choose and why?**
4. **Compare One-Hot Encoding and Binary Encoding for a feature with 1,000 unique categories in terms of column count and information loss.**
5. **What is the main risk of Target Encoding, and how does smoothing help reduce it?**
6. **You have a `Country` column with 195 unique values, and you're training a logistic regression. Recommend an encoding strategy with justification.**

---

## 4.13 Real-World ML Applications

| Domain | Feature | Encoding Used |
|--------|---------|---------------|
| E-commerce | Product category (1000+ items) | Target / Frequency |
| Healthcare | Blood type (A, B, AB, O) | One-Hot |
| Finance | Education level (HS → PhD) | Ordinal |
| NLP | Word tokens (huge vocab) | Embeddings / Hashing |
| Ride-sharing | Pickup zone (high cardinality) | Binary / Target |

---

[⬅️ Previous: 03-feature-scaling.md](03-feature-scaling.md) | [➡️ Next: 05-feature-engineering.md](05-feature-engineering.md)
