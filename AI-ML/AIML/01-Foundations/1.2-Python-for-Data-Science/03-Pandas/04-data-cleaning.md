# Chapter 4: Data Cleaning

> **Unit 3 – Pandas** | Transforming messy real-world data into a clean, analysis-ready format.

---

## 1. Overview

Raw data is rarely clean. Data cleaning typically accounts for **60–80%** of a data scientist's time. Common issues include duplicates, inconsistent naming, wrong data types, dirty strings, and outliers. Pandas provides a rich toolkit for all of these.

```python
import pandas as pd
import numpy as np

df = pd.DataFrame({
    'Name': ['  Alice ', 'BOB', 'charlie', 'Alice ', 'Diana'],
    'Age': ['25', '30', 'thirty', '25', '28'],
    'City': ['new york', 'LA', 'new york', 'new york', 'chicago'],
    'Salary': [50000, 60000, 70000, 50000, None]
})
```

---

## 2. Identifying Dirty Data

```python
# Check shape, types, and nulls
df.info()
df.describe(include='all')

# Check for duplicates
df.duplicated()              # Boolean Series — True for duplicate rows
df.duplicated().sum()        # Count of duplicate rows

# Check unique values
df['City'].unique()
df['City'].nunique()
df['City'].value_counts()

# Check data types
df.dtypes
```

---

## 3. Removing Duplicates

```python
# Drop exact duplicate rows (keeps first occurrence by default)
df_clean = df.drop_duplicates()

# Keep last occurrence
df_clean = df.drop_duplicates(keep='last')

# Drop all duplicates (keep none)
df_clean = df.drop_duplicates(keep=False)

# Check duplicates based on specific columns
df_clean = df.drop_duplicates(subset=['Name', 'City'])

# In-place
df.drop_duplicates(inplace=True)
```

---

## 4. Renaming Columns and Index

```python
# Rename specific columns
df = df.rename(columns={'Name': 'name', 'Age': 'age', 'City': 'city'})

# Rename using a function
df.columns = df.columns.str.lower().str.strip().str.replace(' ', '_')

# Rename index values
df = df.rename(index={0: 'first', 1: 'second'})

# Common cleaning pipeline for column names
def clean_columns(df):
    df.columns = (df.columns
                  .str.strip()
                  .str.lower()
                  .str.replace(' ', '_')
                  .str.replace(r'[^\w]', '', regex=True))
    return df
```

---

## 5. Changing Data Types — `astype()`

```python
# Convert string to numeric (will fail if non-numeric strings exist)
df['age'] = df['age'].astype(int)       # Raises error on 'thirty'

# Safe conversion with pd.to_numeric
df['age'] = pd.to_numeric(df['age'], errors='coerce')  # 'thirty' → NaN

# errors='coerce' converts unparseable values to NaN
# errors='ignore' leaves the column unchanged
# errors='raise' (default) raises an error

# Convert to specific types
df['salary'] = df['salary'].astype('float64')
df['city'] = df['city'].astype('category')     # Memory-efficient for low-cardinality

# Convert to datetime
df['date'] = pd.to_datetime(df['date_col'], errors='coerce')

# Convert boolean-like strings
df['active'] = df['active'].map({'yes': True, 'no': False})
```

---

## 6. String Operations — `.str` Accessor

The `.str` accessor unlocks vectorized string methods on Series.

```python
# Case changes
df['name'] = df['name'].str.lower()
df['name'] = df['name'].str.upper()
df['name'] = df['name'].str.title()       # 'alice' → 'Alice'
df['name'] = df['name'].str.capitalize()

# Whitespace
df['name'] = df['name'].str.strip()       # Remove leading/trailing spaces
df['name'] = df['name'].str.lstrip()      # Left strip only
df['name'] = df['name'].str.rstrip()      # Right strip only

# Searching
df['name'].str.contains('ali', case=False)     # Boolean Series
df['name'].str.startswith('A')
df['name'].str.endswith('e')
df['name'].str.len()                           # Length of each string

# Replacing
df['city'] = df['city'].str.replace('new york', 'New York')
df['phone'] = df['phone'].str.replace(r'\D', '', regex=True)  # Remove non-digits

# Splitting
df[['first', 'last']] = df['full_name'].str.split(' ', n=1, expand=True)

# Extracting with regex
df['digits'] = df['code'].str.extract(r'(\d+)')

# Padding
df['id'] = df['id'].str.zfill(5)   # '42' → '00042'
```

---

## 7. Replacing Values

```python
# Replace specific values
df['city'] = df['city'].replace('LA', 'Los Angeles')

# Replace multiple values
df['city'] = df['city'].replace({'LA': 'Los Angeles', 'NYC': 'New York'})

# Replace across entire DataFrame
df = df.replace('?', np.nan)
df = df.replace({'-': np.nan, 'N/A': np.nan, '': np.nan})

# Replace with regex
df['text'] = df['text'].replace(r'\s+', ' ', regex=True)  # Collapse whitespace

# Clip values to a range
df['age'] = df['age'].clip(lower=0, upper=120)
```

---

## 8. Applying Functions — `apply()`, `map()`, `applymap()`

```python
# apply() — works on rows or columns of a DataFrame, or elements of a Series
df['age_group'] = df['age'].apply(lambda x: 'Young' if x < 30 else 'Senior')

# Apply across rows (axis=1)
df['summary'] = df.apply(lambda row: f"{row['name']} from {row['city']}", axis=1)

# Apply a named function
def categorize_salary(salary):
    if salary < 55000:
        return 'Low'
    elif salary < 65000:
        return 'Mid'
    return 'High'

df['salary_tier'] = df['salary'].apply(categorize_salary)

# map() — element-wise on a Series, also accepts dict for mapping
df['city_code'] = df['city'].map({'New York': 'NY', 'Los Angeles': 'LA', 'Chicago': 'CHI'})

# applymap() — element-wise on an entire DataFrame (Pandas <2.1)
# In Pandas >=2.1, use DataFrame.map() instead
df_str = df.select_dtypes('object').applymap(str.upper)

# Prefer vectorized operations over apply when possible
# SLOW: df['x'].apply(lambda v: v * 2)
# FAST: df['x'] * 2
```

---

## 9. Outlier Detection

```python
# IQR Method
Q1 = df['salary'].quantile(0.25)
Q3 = df['salary'].quantile(0.75)
IQR = Q3 - Q1

lower_bound = Q1 - 1.5 * IQR
upper_bound = Q3 + 1.5 * IQR

outliers = df[(df['salary'] < lower_bound) | (df['salary'] > upper_bound)]
df_no_outliers = df[(df['salary'] >= lower_bound) & (df['salary'] <= upper_bound)]

# Z-Score Method
from scipy import stats
z_scores = np.abs(stats.zscore(df['salary'].dropna()))
df_no_outliers = df[z_scores < 3]   # Keep values within 3 std devs

# Cap outliers instead of removing
df['salary'] = df['salary'].clip(lower=lower_bound, upper=upper_bound)
```

---

## 10. Data Validation

```python
# Assert no nulls
assert df['name'].notnull().all(), "Name column has null values!"

# Assert unique
assert df['id'].is_unique, "IDs are not unique!"

# Assert value range
assert (df['age'] >= 0).all() and (df['age'] <= 150).all(), "Invalid ages!"

# Assert expected values
assert df['status'].isin(['active', 'inactive']).all(), "Unexpected status values!"

# Comprehensive validation check
def validate_dataframe(df):
    issues = []
    if df.duplicated().any():
        issues.append(f"Found {df.duplicated().sum()} duplicate rows")
    for col in df.select_dtypes('number').columns:
        if df[col].isnull().any():
            issues.append(f"Column '{col}' has {df[col].isnull().sum()} null values")
    return issues
```

---

## 11. Best Practices

| Practice | Recommendation |
|---|---|
| Type conversion | Use `pd.to_numeric(errors='coerce')` over `astype()` for safety |
| String cleaning | Chain `.str.strip().str.lower()` early in the pipeline |
| Duplicates | Always check and document why duplicates exist before dropping |
| Apply vs vectorized | Prefer vectorized ops; use `apply()` only when necessary |
| Validation | Add assertions after each cleaning step in pipelines |

---

## 12. Summary Table

| Task | Method |
|---|---|
| Remove duplicates | `df.drop_duplicates()` |
| Rename columns | `df.rename(columns={})` or `df.columns = [...]` |
| Change types | `pd.to_numeric()`, `astype()`, `pd.to_datetime()` |
| String operations | `df['col'].str.method()` |
| Replace values | `df.replace()` |
| Apply functions | `apply()`, `map()`, `applymap()` |
| Detect outliers | IQR method, Z-score method |

---

## 13. Revision Questions

1. What is the difference between `df.drop_duplicates()` with `keep='first'`, `keep='last'`, and `keep=False`?
2. Why is `pd.to_numeric(errors='coerce')` safer than `astype(int)` for dirty data?
3. Write a pipeline that strips whitespace, lowercases, and replaces spaces with underscores in all column names.
4. When should you use `apply()` vs vectorized operations? Give an example of each.
5. Explain the IQR method for outlier detection. What does the 1.5 multiplier represent?
6. How do you split a `full_name` column into separate `first_name` and `last_name` columns?

---

[← Previous: Indexing & Selection](03-indexing-and-selection.md) | [Next → Chapter 5: Missing Data Handling](05-missing-data-handling.md)
