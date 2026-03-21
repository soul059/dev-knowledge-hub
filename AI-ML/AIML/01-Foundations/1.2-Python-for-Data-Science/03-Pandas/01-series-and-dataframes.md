# Chapter 1: Series and DataFrames

> **Unit 3 – Pandas** | The two fundamental data structures in Pandas that power all data manipulation.

---

## 1. Overview

Pandas provides two primary data structures:
- **Series** — A one-dimensional labeled array (like a column).
- **DataFrame** — A two-dimensional labeled table (like a spreadsheet or SQL table).

Understanding these structures is the foundation for every Pandas operation.

```python
import pandas as pd
import numpy as np
```

---

## 2. Series

### 2.1 Creating a Series

```python
# From a list
s1 = pd.Series([10, 20, 30, 40])

# From a list with custom index
s2 = pd.Series([10, 20, 30], index=['a', 'b', 'c'])

# From a dictionary
s3 = pd.Series({'x': 100, 'y': 200, 'z': 300})

# From a scalar (broadcasts to all indices)
s4 = pd.Series(5, index=['a', 'b', 'c'])

# From a NumPy array
s5 = pd.Series(np.random.randn(5), name='random_values')
```

### 2.2 Series Attributes

```python
s = pd.Series([10, 20, 30], index=['a', 'b', 'c'], name='scores')

s.index      # Index(['a', 'b', 'c'], dtype='object')
s.values     # array([10, 20, 30])
s.dtype      # int64
s.name       # 'scores'
s.shape      # (3,)
s.size       # 3
s.is_unique  # True — all values are unique
```

### 2.3 Series Operations

```python
# Vectorized arithmetic
s * 2          # Each element multiplied by 2
s + 100        # Each element increased by 100

# Boolean filtering
s[s > 15]      # Series with values > 15

# Alignment by index — missing indices become NaN
s_a = pd.Series([1, 2, 3], index=['a', 'b', 'c'])
s_b = pd.Series([4, 5, 6], index=['b', 'c', 'd'])
s_a + s_b      # a: NaN, b: 6, c: 8, d: NaN
```

---

## 3. DataFrame

### 3.1 Creating a DataFrame

```python
# From a dictionary of lists
df1 = pd.DataFrame({
    'name': ['Alice', 'Bob', 'Charlie'],
    'age': [25, 30, 35],
    'salary': [50000, 60000, 70000]
})

# From a list of dictionaries
df2 = pd.DataFrame([
    {'name': 'Alice', 'age': 25},
    {'name': 'Bob', 'age': 30},
    {'name': 'Charlie', 'age': 35}
])

# From a 2D NumPy array
df3 = pd.DataFrame(
    np.random.randn(4, 3),
    columns=['A', 'B', 'C'],
    index=['row1', 'row2', 'row3', 'row4']
)

# From a list of lists
df4 = pd.DataFrame(
    [['Alice', 25], ['Bob', 30]],
    columns=['name', 'age']
)

# From a dictionary of Series
df5 = pd.DataFrame({
    'height': pd.Series([5.5, 6.0], index=['Alice', 'Bob']),
    'weight': pd.Series([55, 70, 80], index=['Alice', 'Bob', 'Charlie'])
})
# Missing values filled with NaN automatically
```

### 3.2 DataFrame Attributes

```python
df = pd.DataFrame({
    'name': ['Alice', 'Bob', 'Charlie'],
    'age': [25, 30, 35],
    'salary': [50000.0, 60000.0, 70000.0]
})

df.shape      # (3, 3) — rows, columns
df.columns    # Index(['name', 'age', 'salary'])
df.index      # RangeIndex(start=0, stop=3, step=1)
df.dtypes     # name: object, age: int64, salary: float64
df.size       # 9 (total elements)
df.ndim       # 2
df.values     # Underlying NumPy array
df.T          # Transposed DataFrame
```

### 3.3 Accessing Columns

```python
# Bracket notation — always works
df['name']              # Returns a Series
df[['name', 'age']]     # Returns a DataFrame (list of columns)

# Dot notation — only for valid Python identifiers, no spaces
df.name                 # Returns a Series (avoid if column name clashes with method)
```

### 3.4 Quick Inspection Methods

```python
df.head(3)       # First 3 rows (default 5)
df.tail(2)       # Last 2 rows (default 5)
df.sample(2)     # 2 random rows

df.info()        # Column names, non-null counts, dtypes, memory usage
df.describe()    # Statistical summary for numeric columns
df.describe(include='all')  # Include non-numeric columns too

df.nunique()     # Number of unique values per column
df.value_counts('age')  # Frequency count of values in a column
```

### 3.5 Setting and Resetting the Index

```python
# Set an existing column as index
df_indexed = df.set_index('name')

# Reset index back to default integer
df_reset = df_indexed.reset_index()

# Set index in-place
df.set_index('name', inplace=True)

# Use drop=False to keep the column after setting as index
df.set_index('name', drop=False)
```

### 3.6 Renaming Columns

```python
# Using rename() with a dictionary
df_renamed = df.rename(columns={'name': 'full_name', 'age': 'years'})

# Overwrite .columns directly
df.columns = ['Name', 'Age', 'Salary']

# Rename with a function
df.rename(columns=str.upper)
df.rename(columns=lambda x: x.strip().lower().replace(' ', '_'))
```

---

## 4. Best Practices

| Practice | Recommendation |
|---|---|
| Column access | Use `df['col']` over `df.col` for safety |
| Index | Always set a meaningful index for faster lookups |
| Inspection | Run `df.info()` + `df.describe()` immediately after loading data |
| Naming | Use `snake_case` column names — rename early |
| Copies vs views | Use `.copy()` when you want an independent DataFrame |

---

## 5. Summary Table

| Feature | Series | DataFrame |
|---|---|---|
| Dimensions | 1D | 2D |
| Created from | list, dict, array, scalar | dict, list of dicts, 2D array |
| Key attributes | `index`, `values`, `dtype`, `name` | `shape`, `columns`, `dtypes`, `index` |
| Access element | `s['a']` or `s[0]` | `df['col']`, `df.loc[]`, `df.iloc[]` |
| Inspection | `head()`, `describe()` | `head()`, `info()`, `describe()` |

---

## 6. Revision Questions

1. What is the difference between a Series and a DataFrame?
2. Create a DataFrame from a dictionary with columns `product`, `price`, and `quantity`. Then set `product` as the index.
3. What happens when you add two Series with partially overlapping indices?
4. Why should you prefer `df['column']` over `df.column`?
5. How do `df.info()` and `df.describe()` differ in the information they provide?
6. Write code to rename all columns in a DataFrame to lowercase with underscores replacing spaces.

---

[Next → Chapter 2: Data Loading](02-data-loading.md)
