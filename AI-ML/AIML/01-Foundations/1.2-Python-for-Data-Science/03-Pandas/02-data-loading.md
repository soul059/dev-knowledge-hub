# Chapter 2: Data Loading and Export

> **Unit 3 – Pandas** | Reading data from various file formats and writing results back.

---

## 1. Overview

Real-world data lives in files, databases, and APIs. Pandas provides `read_*` functions to ingest data and `to_*` methods to export. The most common format is CSV, but Pandas supports JSON, Excel, SQL, Parquet, and more.

```python
import pandas as pd
```

---

## 2. Reading CSV Files — `pd.read_csv()`

### 2.1 Basic Usage

```python
df = pd.read_csv('data.csv')
```

### 2.2 Key Parameters

```python
# Custom separator (TSV, pipe-delimited, etc.)
df = pd.read_csv('data.tsv', sep='\t')
df = pd.read_csv('data.txt', sep='|')

# No header row — provide column names manually
df = pd.read_csv('data.csv', header=None, names=['col1', 'col2', 'col3'])

# Use a specific row as header
df = pd.read_csv('data.csv', header=2)  # Third row becomes header

# Read only specific columns
df = pd.read_csv('data.csv', usecols=['name', 'age'])
df = pd.read_csv('data.csv', usecols=[0, 2, 4])  # By position

# Force specific dtypes
df = pd.read_csv('data.csv', dtype={'zipcode': str, 'price': float})

# Parse date columns automatically
df = pd.read_csv('data.csv', parse_dates=['date_col'])
df = pd.read_csv('data.csv', parse_dates=[['year', 'month', 'day']])  # Combine columns

# Read only first N rows (useful for previewing large files)
df = pd.read_csv('data.csv', nrows=100)

# Skip rows
df = pd.read_csv('data.csv', skiprows=5)          # Skip first 5 rows
df = pd.read_csv('data.csv', skipfooter=3, engine='python')  # Skip last 3

# Set a column as index while reading
df = pd.read_csv('data.csv', index_col='id')

# Handle missing value markers
df = pd.read_csv('data.csv', na_values=['NA', 'N/A', '-', '?', 'missing'])
```

### 2.3 Handling Large Files with `chunksize`

```python
# Process file in chunks of 10,000 rows
chunks = pd.read_csv('large_file.csv', chunksize=10_000)

results = []
for chunk in chunks:
    # Process each chunk independently
    filtered = chunk[chunk['sales'] > 1000]
    results.append(filtered)

df_final = pd.concat(results, ignore_index=True)
```

### 2.4 Encoding Issues

```python
# Common encodings: utf-8 (default), latin-1, cp1252, iso-8859-1
df = pd.read_csv('data.csv', encoding='latin-1')
df = pd.read_csv('data.csv', encoding='cp1252')

# If unsure, try latin-1 — it rarely fails (accepts all byte values)
```

---

## 3. Reading JSON — `pd.read_json()`

```python
# Standard JSON (array of records)
df = pd.read_json('data.json')

# Nested JSON — use orient parameter
df = pd.read_json('data.json', orient='records')
# orient options: 'split', 'records', 'index', 'columns', 'values'

# JSON Lines format (one JSON object per line)
df = pd.read_json('data.jsonl', lines=True)

# From a JSON string
import json
json_str = '[{"name": "Alice", "age": 25}, {"name": "Bob", "age": 30}]'
df = pd.read_json(json_str)

# Nested JSON — flatten with json_normalize
data = [
    {'name': 'Alice', 'address': {'city': 'NYC', 'zip': '10001'}},
    {'name': 'Bob', 'address': {'city': 'LA', 'zip': '90001'}}
]
df = pd.json_normalize(data)
# Columns: name, address.city, address.zip
```

---

## 4. Reading Excel — `pd.read_excel()`

```python
# Requires openpyxl (for .xlsx) or xlrd (for .xls)
# pip install openpyxl

df = pd.read_excel('data.xlsx')

# Specific sheet
df = pd.read_excel('data.xlsx', sheet_name='Sales')
df = pd.read_excel('data.xlsx', sheet_name=1)  # By index

# Read all sheets — returns a dict of DataFrames
all_sheets = pd.read_excel('data.xlsx', sheet_name=None)
for sheet_name, df in all_sheets.items():
    print(f"{sheet_name}: {df.shape}")

# Skip rows and select columns (same params as read_csv)
df = pd.read_excel('data.xlsx', usecols='A:D', skiprows=2)
```

---

## 5. Reading from SQL — `pd.read_sql()`

```python
import sqlite3

# Create a connection
conn = sqlite3.connect('database.db')

# Read entire table
df = pd.read_sql('SELECT * FROM customers', conn)

# Parameterized query
df = pd.read_sql(
    'SELECT * FROM orders WHERE amount > ?',
    conn,
    params=[100]
)

# Read table directly
df = pd.read_sql_table('customers', conn)

# With SQLAlchemy (recommended for production)
from sqlalchemy import create_engine
engine = create_engine('sqlite:///database.db')
df = pd.read_sql('SELECT * FROM sales', engine)

conn.close()
```

---

## 6. Reading Parquet — `pd.read_parquet()`

```python
# Parquet: columnar storage, fast reads, preserves dtypes
# pip install pyarrow  (or fastparquet)

df = pd.read_parquet('data.parquet')

# Read specific columns (extremely fast with columnar format)
df = pd.read_parquet('data.parquet', columns=['name', 'age'])
```

---

## 7. Exporting Data

### 7.1 To CSV

```python
df.to_csv('output.csv', index=False)

# Custom separator
df.to_csv('output.tsv', sep='\t', index=False)

# Specific columns only
df.to_csv('output.csv', columns=['name', 'age'], index=False)

# Handle encoding
df.to_csv('output.csv', encoding='utf-8-sig', index=False)  # BOM for Excel
```

### 7.2 To JSON

```python
df.to_json('output.json', orient='records', indent=2)
df.to_json('output.jsonl', orient='records', lines=True)
```

### 7.3 To Excel

```python
df.to_excel('output.xlsx', sheet_name='Results', index=False)

# Multiple sheets
with pd.ExcelWriter('output.xlsx') as writer:
    df1.to_excel(writer, sheet_name='Sales', index=False)
    df2.to_excel(writer, sheet_name='Inventory', index=False)
```

### 7.4 To Parquet

```python
df.to_parquet('output.parquet', index=False)
```

---

## 8. Best Practices

| Scenario | Recommendation |
|---|---|
| Large files (>1 GB) | Use `chunksize` or Parquet format |
| Preserve dtypes | Use Parquet instead of CSV |
| Date columns | Always use `parse_dates` when reading |
| Memory optimization | Specify `dtype` and `usecols` |
| Encoding errors | Try `encoding='latin-1'` as a fallback |
| Production databases | Use SQLAlchemy engine over raw connections |

---

## 9. Summary Table

| Format | Read Function | Write Method | Best For |
|---|---|---|---|
| CSV | `pd.read_csv()` | `df.to_csv()` | Universal interchange |
| JSON | `pd.read_json()` | `df.to_json()` | API data, nested structures |
| Excel | `pd.read_excel()` | `df.to_excel()` | Business reports |
| SQL | `pd.read_sql()` | `df.to_sql()` | Database integration |
| Parquet | `pd.read_parquet()` | `df.to_parquet()` | Big data, ML pipelines |

---

## 10. Revision Questions

1. What are the key parameters of `pd.read_csv()` you should always consider?
2. How do you process a 10 GB CSV file that doesn't fit in memory?
3. What is the advantage of Parquet over CSV for storing DataFrames?
4. How do you read a JSON file where each line is a separate JSON object?
5. Write code to read an Excel file with multiple sheets into separate DataFrames.
6. How do you handle a CSV file with a non-UTF-8 encoding that causes `UnicodeDecodeError`?

---

[← Previous: Series & DataFrames](01-series-and-dataframes.md) | [Next → Chapter 3: Indexing & Selection](03-indexing-and-selection.md)
