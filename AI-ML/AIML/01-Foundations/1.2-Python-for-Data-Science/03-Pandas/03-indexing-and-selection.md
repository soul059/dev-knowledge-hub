# Chapter 3: Indexing and Selection

> **Unit 3 – Pandas** | Precisely accessing, filtering, and modifying data in DataFrames.

---

## 1. Overview

Pandas offers multiple ways to select data. Choosing the right method depends on whether you're selecting by **label**, **position**, or **condition**. Mastering `.loc`, `.iloc`, and boolean indexing is essential for efficient data manipulation.

```python
import pandas as pd
import numpy as np

df = pd.DataFrame({
    'name': ['Alice', 'Bob', 'Charlie', 'Diana', 'Eve'],
    'age': [25, 30, 35, 28, 32],
    'city': ['NYC', 'LA', 'NYC', 'Chicago', 'LA'],
    'salary': [50000, 60000, 70000, 55000, 65000]
}, index=['a', 'b', 'c', 'd', 'e'])
```

---

## 2. Column Selection

```python
# Single column — returns Series
df['name']
df.name           # Dot notation (avoid if name clashes with DataFrame method)

# Multiple columns — returns DataFrame
df[['name', 'age']]

# Select columns by dtype
df.select_dtypes(include='number')      # Only numeric columns
df.select_dtypes(include='object')      # Only string/object columns
df.select_dtypes(exclude='int64')       # Exclude specific dtype
```

---

## 3. Label-Based Selection — `.loc[]`

`.loc` selects by **index labels** and **column names**. Both endpoints are **inclusive**.

```python
# Single row by label
df.loc['a']                        # Returns Series for row 'a'

# Multiple rows
df.loc[['a', 'c', 'e']]           # Specific rows

# Row and column
df.loc['a', 'name']               # Single value: 'Alice'
df.loc['a':'c', 'name':'city']    # Slice rows a-c, columns name-city (inclusive!)

# All rows, specific columns
df.loc[:, ['name', 'salary']]

# Boolean mask with .loc
df.loc[df['age'] > 30]
df.loc[df['city'] == 'NYC', ['name', 'salary']]  # Filter rows, select columns
```

---

## 4. Position-Based Selection — `.iloc[]`

`.iloc` selects by **integer position**. End positions are **exclusive** (like Python slicing).

```python
# Single row by position
df.iloc[0]                         # First row (Series)

# Multiple rows
df.iloc[[0, 2, 4]]                 # Rows at positions 0, 2, 4

# Row and column by position
df.iloc[0, 1]                     # Row 0, Column 1 → 25
df.iloc[0:3, 1:3]                 # Rows 0-2, Columns 1-2 (exclusive end)

# Last row
df.iloc[-1]

# Every other row
df.iloc[::2]
```

---

## 5. `.loc` vs `.iloc` Comparison

| Feature | `.loc` | `.iloc` |
|---|---|---|
| Selection by | Labels | Integer positions |
| Slice end | **Inclusive** | **Exclusive** |
| Accepts | Labels, boolean arrays | Integers, boolean arrays |
| Example | `df.loc['a':'c']` → rows a, b, c | `df.iloc[0:3]` → rows 0, 1, 2 |

```python
# Key difference in slicing
df.loc['a':'c']     # Includes 'c' → 3 rows
df.iloc[0:2]        # Excludes position 2 → 2 rows
```

---

## 6. Scalar Access — `.at[]` and `.iat[]`

Optimized for accessing a **single value**. Faster than `.loc`/`.iloc` for scalar lookups.

```python
# .at — by label
df.at['a', 'name']        # 'Alice'

# .iat — by position
df.iat[0, 0]              # 'Alice'

# Setting a single value
df.at['a', 'salary'] = 55000
df.iat[0, 3] = 55000
```

---

## 7. Boolean Indexing (Conditional Selection)

```python
# Single condition
df[df['age'] > 30]

# Multiple conditions — use & (and), | (or), ~ (not)
# IMPORTANT: Each condition must be wrapped in parentheses
df[(df['age'] > 25) & (df['city'] == 'NYC')]
df[(df['salary'] > 55000) | (df['age'] < 28)]
df[~(df['city'] == 'LA')]       # NOT LA

# .isin() for matching multiple values
df[df['city'].isin(['NYC', 'Chicago'])]

# .between() for range filtering
df[df['age'].between(25, 32)]   # Inclusive on both ends

# String-based filtering
df[df['name'].str.startswith('A')]
df[df['name'].str.contains('li', case=False)]
```

---

## 8. The `.query()` Method

A more readable alternative to boolean indexing, especially for complex conditions.

```python
# Basic query
df.query('age > 30')

# Multiple conditions
df.query('age > 25 and city == "NYC"')
df.query('salary > 55000 or age < 28')

# Using variables with @
min_age = 28
df.query('age >= @min_age')

# Column names with spaces — use backticks
# df.query('`column name` > 10')
```

---

## 9. Multi-Index (Hierarchical Indexing)

```python
# Create a MultiIndex DataFrame
arrays = [
    ['East', 'East', 'West', 'West'],
    ['Q1', 'Q2', 'Q1', 'Q2']
]
index = pd.MultiIndex.from_arrays(arrays, names=['region', 'quarter'])
df_multi = pd.DataFrame({
    'revenue': [100, 150, 200, 250],
    'profit': [20, 30, 50, 60]
}, index=index)

# Select by first level
df_multi.loc['East']

# Select by both levels
df_multi.loc[('East', 'Q1')]

# Cross-section with xs
df_multi.xs('Q1', level='quarter')   # All Q1 data across regions

# Reset multi-index back to columns
df_multi.reset_index()
```

---

## 10. Setting Values

```python
# Set values with .loc
df.loc['a', 'salary'] = 52000
df.loc[df['city'] == 'LA', 'salary'] = 70000   # Conditional update

# Set values with .iloc
df.iloc[0, 3] = 53000

# Add a new column
df['bonus'] = df['salary'] * 0.1

# Conditional column assignment with np.where
df['senior'] = np.where(df['age'] >= 30, 'Yes', 'No')

# Multiple conditions with np.select
conditions = [df['age'] < 28, df['age'] < 35, df['age'] >= 35]
choices = ['Junior', 'Mid', 'Senior']
df['level'] = np.select(conditions, choices, default='Unknown')
```

---

## 11. ⚠️ Chained Indexing Warning

```python
# BAD — chained indexing can lead to SettingWithCopyWarning
df[df['age'] > 30]['salary'] = 99999   # May not modify df!

# GOOD — use .loc for setting values
df.loc[df['age'] > 30, 'salary'] = 99999   # Always modifies df

# WHY? Chained indexing creates an intermediate copy.
# .loc accesses the original DataFrame directly.
```

---

## 12. Best Practices

| Practice | Recommendation |
|---|---|
| Label-based selection | Use `.loc` |
| Position-based selection | Use `.iloc` |
| Single value access | Use `.at` / `.iat` for speed |
| Setting values | Always use `.loc` — never chain `[][]` |
| Complex conditions | Use `.query()` for readability |
| Copy safety | Use `df.copy()` when creating subsets you'll modify |

---

## 13. Revision Questions

1. What is the key difference between `.loc` and `.iloc` regarding slice endpoints?
2. Why does `df[df['x'] > 5]['y'] = 10` produce a warning, and how do you fix it?
3. Write code to select all rows where `city` is either 'NYC' or 'LA' and `salary` is above 55,000.
4. How do you use `.query()` with an external variable?
5. What are `.at` and `.iat`, and when should you prefer them over `.loc`/`.iloc`?
6. How do you select data from a specific level in a MultiIndex DataFrame?

---

[← Previous: Data Loading](02-data-loading.md) | [Next → Chapter 4: Data Cleaning](04-data-cleaning.md)
