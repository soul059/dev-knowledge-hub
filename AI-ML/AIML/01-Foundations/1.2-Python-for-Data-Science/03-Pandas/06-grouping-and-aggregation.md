# Chapter 6: Grouping and Aggregation

> **Unit 3 – Pandas** | Split-Apply-Combine — the most powerful pattern for summarizing and analyzing data.

---

## 1. Overview

`groupby()` splits a DataFrame into groups based on column values, applies a function to each group, and combines the results. This **split-apply-combine** paradigm is the backbone of exploratory data analysis (EDA).

```python
import pandas as pd
import numpy as np

df = pd.DataFrame({
    'dept': ['HR', 'IT', 'HR', 'IT', 'Sales', 'Sales', 'IT', 'HR'],
    'name': ['Alice', 'Bob', 'Charlie', 'Diana', 'Eve', 'Frank', 'Grace', 'Hank'],
    'salary': [50000, 80000, 55000, 75000, 60000, 62000, 90000, 48000],
    'experience': [2, 5, 3, 4, 6, 7, 8, 1],
    'rating': [4.2, 3.8, 4.5, 4.0, 3.9, 4.1, 4.7, 3.5]
})
```

---

## 2. Basic `groupby()` Mechanics

```python
# Create a GroupBy object (lazy — no computation yet)
grouped = df.groupby('dept')

# The object stores group information
grouped.groups        # Dict: {'HR': [0, 2, 7], 'IT': [1, 3, 6], 'Sales': [4, 5]}
grouped.ngroups       # 3
grouped.size()        # Count per group

# Iterate over groups
for name, group_df in grouped:
    print(f"\n{name}:")
    print(group_df)

# Get a specific group
grouped.get_group('IT')
```

---

## 3. Single-Column Aggregation

```python
# Mean salary per department
df.groupby('dept')['salary'].mean()

# Common aggregation functions
df.groupby('dept')['salary'].sum()
df.groupby('dept')['salary'].min()
df.groupby('dept')['salary'].max()
df.groupby('dept')['salary'].median()
df.groupby('dept')['salary'].std()
df.groupby('dept')['salary'].count()     # Non-null count
df.groupby('dept')['salary'].nunique()   # Unique values
df.groupby('dept')['salary'].first()     # First value in each group
df.groupby('dept')['salary'].last()      # Last value in each group
```

---

## 4. Multi-Column GroupBy

```python
# Group by multiple columns
df_ext = df.copy()
df_ext['location'] = ['East', 'West', 'East', 'West', 'East', 'West', 'East', 'West']

df_ext.groupby(['dept', 'location'])['salary'].mean()
# Returns a Series with MultiIndex

# Reset index to get a flat DataFrame
df_ext.groupby(['dept', 'location'])['salary'].mean().reset_index()
```

---

## 5. Multiple Aggregations with `agg()`

```python
# Multiple functions on one column
df.groupby('dept')['salary'].agg(['mean', 'median', 'std', 'count'])

# Different functions for different columns
df.groupby('dept').agg({
    'salary': ['mean', 'max'],
    'experience': 'mean',
    'rating': ['mean', 'min']
})
# Returns a DataFrame with MultiIndex columns

# Named aggregation (Pandas >= 0.25) — clean column names
df.groupby('dept').agg(
    avg_salary=('salary', 'mean'),
    max_salary=('salary', 'max'),
    avg_experience=('experience', 'mean'),
    avg_rating=('rating', 'mean'),
    headcount=('name', 'count')
)
# Returns flat column names — much cleaner for downstream use!
```

---

## 6. Custom Aggregation Functions

```python
# Lambda functions
df.groupby('dept')['salary'].agg(lambda x: x.max() - x.min())

# Named function
def salary_range(series):
    return series.max() - series.min()

df.groupby('dept')['salary'].agg(salary_range)

# Multiple custom functions
def iqr(series):
    return series.quantile(0.75) - series.quantile(0.25)

def coefficient_of_variation(series):
    return series.std() / series.mean()

df.groupby('dept')['salary'].agg([salary_range, iqr, coefficient_of_variation])

# Custom function with named aggregation
df.groupby('dept').agg(
    salary_range=('salary', salary_range),
    salary_iqr=('salary', iqr)
)
```

---

## 7. `transform()` — Broadcast Back to Original Shape

`transform()` returns a result that has the **same index** as the input — perfect for adding group-level statistics as new columns.

```python
# Add group mean as a new column
df['dept_avg_salary'] = df.groupby('dept')['salary'].transform('mean')

# Normalize salary within each department (z-score)
df['salary_zscore'] = df.groupby('dept')['salary'].transform(
    lambda x: (x - x.mean()) / x.std()
)

# Rank within each group
df['dept_salary_rank'] = df.groupby('dept')['salary'].transform('rank', ascending=False)

# Percentage of group total
df['salary_pct_of_dept'] = df['salary'] / df.groupby('dept')['salary'].transform('sum')

# Fill missing values with group median
df['salary'] = df.groupby('dept')['salary'].transform(
    lambda x: x.fillna(x.median())
)
```

---

## 8. `filter()` — Keep/Remove Entire Groups

```python
# Keep only departments with more than 2 employees
df_filtered = df.groupby('dept').filter(lambda x: len(x) > 2)

# Keep groups where average salary > 60000
df_filtered = df.groupby('dept').filter(lambda x: x['salary'].mean() > 60000)

# Keep groups with low variance
df_filtered = df.groupby('dept').filter(lambda x: x['salary'].std() < 15000)
```

---

## 9. `pivot_table()`

A high-level interface for creating summary tables — similar to Excel pivot tables.

```python
# Basic pivot table
pd.pivot_table(df, values='salary', index='dept', aggfunc='mean')

# Multiple aggregation functions
pd.pivot_table(df, values='salary', index='dept', aggfunc=['mean', 'sum', 'count'])

# Two-dimensional pivot (index + columns)
df['level'] = ['Junior', 'Senior', 'Junior', 'Senior', 'Junior', 'Senior', 'Senior', 'Junior']

pd.pivot_table(
    df,
    values='salary',
    index='dept',
    columns='level',
    aggfunc='mean',
    fill_value=0       # Fill NaN cells
)

# Multiple values
pd.pivot_table(
    df,
    values=['salary', 'rating'],
    index='dept',
    aggfunc={'salary': 'mean', 'rating': 'max'}
)

# Add margins (totals)
pd.pivot_table(df, values='salary', index='dept', aggfunc='sum', margins=True)
```

---

## 10. `crosstab()`

Computes frequency tables (cross-tabulations) between two or more categorical variables.

```python
# Basic frequency table
pd.crosstab(df['dept'], df['level'])

# Normalized (proportions)
pd.crosstab(df['dept'], df['level'], normalize=True)         # Of total
pd.crosstab(df['dept'], df['level'], normalize='index')      # Row-wise
pd.crosstab(df['dept'], df['level'], normalize='columns')    # Column-wise

# With values and aggregation
pd.crosstab(df['dept'], df['level'], values=df['salary'], aggfunc='mean')

# Add margins
pd.crosstab(df['dept'], df['level'], margins=True)
```

---

## 11. Practical EDA Patterns

```python
# Pattern 1: Top N per group
top_2_per_dept = (df.groupby('dept')
                   .apply(lambda x: x.nlargest(2, 'salary'))
                   .reset_index(drop=True))

# Pattern 2: Group summary report
summary = df.groupby('dept').agg(
    headcount=('name', 'count'),
    avg_salary=('salary', 'mean'),
    min_salary=('salary', 'min'),
    max_salary=('salary', 'max'),
    avg_rating=('rating', 'mean')
).round(2)

# Pattern 3: Cumulative sum within groups
df['cumulative_salary'] = df.sort_values('experience').groupby('dept')['salary'].cumsum()

# Pattern 4: Percentage change within groups
df['salary_pct_change'] = df.sort_values('experience').groupby('dept')['salary'].pct_change()

# Pattern 5: Flag groups meeting a condition
dept_avg = df.groupby('dept')['salary'].transform('mean')
df['above_dept_avg'] = df['salary'] > dept_avg
```

---

## 12. Best Practices

| Practice | Recommendation |
|---|---|
| Named aggregation | Use `agg(new_name=('col', 'func'))` for clean output |
| Adding group stats | Use `transform()`, not merge after groupby |
| Filtering groups | Use `filter()` to keep/remove entire groups |
| Avoid `apply` if possible | `agg` and `transform` are faster |
| Reset index | Use `.reset_index()` after groupby for flat DataFrames |
| Sort results | Chain `.sort_values()` for presentation |

---

## 13. Summary Table

| Method | Returns | Use Case |
|---|---|---|
| `groupby().mean()` | Reduced (one row per group) | Simple single-function aggregation |
| `agg()` | Reduced | Multiple functions, named columns |
| `transform()` | Same shape as input | Add group stats back to original |
| `filter()` | Subset of original | Keep/remove entire groups |
| `apply()` | Flexible | Complex group-wise operations |
| `pivot_table()` | Reshaped summary | 2D summary tables |
| `crosstab()` | Frequency table | Category counts |

---

## 14. Revision Questions

1. Explain the split-apply-combine pattern with a concrete example.
2. What is the difference between `agg()` and `transform()`? When would you use each?
3. Write code to compute the mean, median, and standard deviation of `salary` per `dept` using named aggregation.
4. How do you add each employee's department average salary as a new column without using merge?
5. Write code to keep only departments that have more than 3 employees using `filter()`.
6. Create a pivot table showing average salary by `dept` (rows) and `level` (columns).

---

[← Previous: Missing Data Handling](05-missing-data-handling.md) | [Next → Chapter 7: Merging & Joining](07-merging-and-joining.md)
