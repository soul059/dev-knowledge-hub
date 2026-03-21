# Chapter 7: Merging and Joining DataFrames

> **Unit 3 – Pandas** | Combining multiple DataFrames — the Pandas equivalent of SQL JOINs.

---

## 1. Overview

Real-world analysis often requires combining data from multiple sources. Pandas provides three main tools:
- **`pd.merge()`** — SQL-style joins on columns or indices.
- **`pd.concat()`** — Stack DataFrames vertically or horizontally.
- **`df.join()`** — Join on index (convenience wrapper around merge).

```python
import pandas as pd
import numpy as np

# Sample data
employees = pd.DataFrame({
    'emp_id': [1, 2, 3, 4, 5],
    'name': ['Alice', 'Bob', 'Charlie', 'Diana', 'Eve'],
    'dept_id': [10, 20, 10, 30, 20]
})

departments = pd.DataFrame({
    'dept_id': [10, 20, 40],
    'dept_name': ['HR', 'IT', 'Finance']
})

salaries = pd.DataFrame({
    'emp_id': [1, 2, 3, 6],
    'salary': [50000, 60000, 55000, 70000]
})
```

---

## 2. `pd.merge()` — SQL-Style Joins

### 2.1 Basic Merge

```python
# Merge on a common column (auto-detected if names match)
result = pd.merge(employees, departments, on='dept_id')

# Explicit left and right keys (when column names differ)
result = pd.merge(employees, salaries, left_on='emp_id', right_on='emp_id')
```

### 2.2 Join Types — `how` Parameter

```python
# INNER JOIN (default) — only matching rows from both
inner = pd.merge(employees, departments, on='dept_id', how='inner')
# emp_id 4 (dept_id=30) dropped — no match in departments
# dept_id 40 (Finance) dropped — no match in employees

# LEFT JOIN — all rows from left, matching from right
left = pd.merge(employees, departments, on='dept_id', how='left')
# All employees kept; Diana gets NaN for dept_name

# RIGHT JOIN — all rows from right, matching from left
right = pd.merge(employees, departments, on='dept_id', how='right')
# All departments kept; Finance gets NaN for name

# OUTER JOIN — all rows from both sides
outer = pd.merge(employees, departments, on='dept_id', how='outer')
# All employees AND all departments; NaN where no match

# CROSS JOIN — Cartesian product (every combination)
cross = pd.merge(employees, departments, how='cross')
```

### 2.3 SQL Join Analogy Table

| Pandas `how=` | SQL Equivalent | Keeps |
|---|---|---|
| `'inner'` | `INNER JOIN` | Only matching rows |
| `'left'` | `LEFT OUTER JOIN` | All left rows + matching right |
| `'right'` | `RIGHT OUTER JOIN` | All right rows + matching left |
| `'outer'` | `FULL OUTER JOIN` | All rows from both sides |
| `'cross'` | `CROSS JOIN` | Cartesian product |

---

## 3. Handling Duplicate Column Names — `suffixes`

```python
df1 = pd.DataFrame({'id': [1, 2], 'value': [100, 200]})
df2 = pd.DataFrame({'id': [1, 2], 'value': [300, 400]})

# Default suffixes are _x and _y
result = pd.merge(df1, df2, on='id')
# Columns: id, value_x, value_y

# Custom suffixes
result = pd.merge(df1, df2, on='id', suffixes=('_left', '_right'))
# Columns: id, value_left, value_right

result = pd.merge(df1, df2, on='id', suffixes=('_2023', '_2024'))
```

---

## 4. Merge Indicator — `indicator`

Track which rows matched during the merge.

```python
result = pd.merge(employees, departments, on='dept_id', how='outer', indicator=True)
# New column '_merge' with values:
#   'both'       — matched in both DataFrames
#   'left_only'  — only in left DataFrame
#   'right_only' — only in right DataFrame

# Custom indicator name
result = pd.merge(employees, departments, on='dept_id', how='outer', indicator='source')

# Use indicator to find unmatched rows
unmatched_left = result[result['_merge'] == 'left_only']
unmatched_right = result[result['_merge'] == 'right_only']
```

---

## 5. Merge Validation — `validate`

Catch unexpected duplicates and many-to-many relationships.

```python
# Expect one-to-one relationship
pd.merge(df1, df2, on='id', validate='one_to_one')    # Raises if duplicates on either side

# Expect many-to-one (left can have dups, right must be unique)
pd.merge(employees, departments, on='dept_id', validate='many_to_one')

# Expect one-to-many
pd.merge(departments, employees, on='dept_id', validate='one_to_many')

# Options: 'one_to_one', 'one_to_many', 'many_to_one', 'many_to_many'
```

---

## 6. Merging on Index

```python
# Merge using the index of one or both DataFrames
df_a = pd.DataFrame({'value_a': [1, 2, 3]}, index=['x', 'y', 'z'])
df_b = pd.DataFrame({'value_b': [4, 5, 6]}, index=['x', 'y', 'w'])

# Both on index
result = pd.merge(df_a, df_b, left_index=True, right_index=True, how='outer')

# Left on column, right on index
result = pd.merge(employees, departments.set_index('dept_id'),
                  left_on='dept_id', right_index=True, how='left')
```

---

## 7. `pd.concat()` — Stacking DataFrames

### 7.1 Vertical Concatenation (Stacking Rows)

```python
df1 = pd.DataFrame({'A': [1, 2], 'B': [3, 4]})
df2 = pd.DataFrame({'A': [5, 6], 'B': [7, 8]})

# Stack vertically (default axis=0)
result = pd.concat([df1, df2])
# Index: 0, 1, 0, 1 — duplicated!

# Reset index
result = pd.concat([df1, df2], ignore_index=True)
# Index: 0, 1, 2, 3

# Add keys to track source
result = pd.concat([df1, df2], keys=['batch1', 'batch2'])
# Creates a MultiIndex
```

### 7.2 Horizontal Concatenation (Adding Columns)

```python
# Side by side
result = pd.concat([df1, df2], axis=1)
```

### 7.3 Handling Mismatched Columns

```python
df_a = pd.DataFrame({'A': [1, 2], 'B': [3, 4]})
df_b = pd.DataFrame({'A': [5, 6], 'C': [7, 8]})

# Outer join (default) — keep all columns, fill NaN
result = pd.concat([df_a, df_b], join='outer', ignore_index=True)
# Columns: A, B, C — B and C have NaN where not present

# Inner join — keep only common columns
result = pd.concat([df_a, df_b], join='inner', ignore_index=True)
# Columns: A only
```

### 7.4 Concatenating Many DataFrames

```python
# From a list (e.g., reading multiple CSV files)
import glob

# files = glob.glob('data/*.csv')
# dfs = [pd.read_csv(f) for f in files]
# combined = pd.concat(dfs, ignore_index=True)
```

---

## 8. `df.join()` — Index-Based Join

A convenience method that merges on the **index** of the calling DataFrame.

```python
df_a = pd.DataFrame({'value_a': [1, 2, 3]}, index=['x', 'y', 'z'])
df_b = pd.DataFrame({'value_b': [4, 5, 6]}, index=['x', 'y', 'w'])

# Default is left join on index
result = df_a.join(df_b)            # Left join
result = df_a.join(df_b, how='outer')

# Join multiple DataFrames at once
df_c = pd.DataFrame({'value_c': [7, 8, 9]}, index=['x', 'z', 'w'])
result = df_a.join([df_b, df_c], how='outer')

# Join on a column of the left DataFrame with index of the right
result = employees.join(departments.set_index('dept_id'), on='dept_id')
```

---

## 9. `merge_asof()` — Approximate Matching

Joins on the nearest key rather than exact match. Useful for **time series** data.

```python
# Stock trades and quotes — match each trade to the most recent quote
trades = pd.DataFrame({
    'time': pd.to_datetime(['10:01', '10:03', '10:05']),
    'ticker': ['AAPL', 'AAPL', 'AAPL'],
    'quantity': [100, 200, 150]
})

quotes = pd.DataFrame({
    'time': pd.to_datetime(['10:00', '10:02', '10:04']),
    'ticker': ['AAPL', 'AAPL', 'AAPL'],
    'price': [150.0, 151.5, 152.0]
})

# Both DataFrames must be sorted by the merge key
result = pd.merge_asof(trades, quotes, on='time', by='ticker')
# Each trade gets the most recent quote price (backward looking)

# With tolerance — only match within a time window
result = pd.merge_asof(trades, quotes, on='time', tolerance=pd.Timedelta('2min'))
```

---

## 10. Choosing the Right Method

| Scenario | Method |
|---|---|
| Join on common column(s) | `pd.merge()` |
| Stack rows from multiple sources | `pd.concat(axis=0)` |
| Add columns side by side | `pd.concat(axis=1)` |
| Join on index | `df.join()` or `pd.merge(left_index=True, right_index=True)` |
| Nearest-match join (time series) | `pd.merge_asof()` |
| Combine many similar files | `pd.concat([list_of_dfs])` |

---

## 11. Best Practices

| Practice | Recommendation |
|---|---|
| Always specify `how` | Don't rely on the default — be explicit about join type |
| Use `validate` | Catch unexpected many-to-many merges early |
| Use `indicator` | Debug merges by checking which rows matched |
| Check shape after merge | Compare before/after row counts to detect unexpected duplicates |
| Use `suffixes` | Give meaningful names to overlapping columns |
| Reset index after concat | Use `ignore_index=True` to avoid duplicate indices |

---

## 12. Summary Table

| Function | Key Parameters | Returns |
|---|---|---|
| `pd.merge()` | `on`, `how`, `left_on`, `right_on`, `suffixes`, `validate`, `indicator` | Merged DataFrame |
| `pd.concat()` | `axis`, `join`, `keys`, `ignore_index` | Concatenated DataFrame |
| `df.join()` | `on`, `how` | Joined DataFrame |
| `pd.merge_asof()` | `on`, `by`, `tolerance`, `direction` | Approximate-match join |

---

## 13. Revision Questions

1. What are the four join types in `pd.merge()`? Draw a Venn diagram for each.
2. What is the difference between `pd.merge()` and `pd.concat()`?
3. How do you handle duplicate column names when merging two DataFrames that share a column?
4. Write code to merge `employees` and `departments`, keeping all employees even if their department doesn't exist.
5. What does the `indicator` parameter in `pd.merge()` do, and how can it help debug merges?
6. When would you use `pd.merge_asof()` instead of a regular merge?

---

[← Previous: Grouping & Aggregation](06-grouping-and-aggregation.md) | [Next → Chapter 8: Time Series Basics](08-time-series-basics.md)
