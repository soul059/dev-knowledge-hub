# Chapter 7: File I/O

> **Overview:** Data science workflows begin and end with files — reading raw data, saving processed results, serializing models. This chapter covers text files, CSV, JSON, pickle, pathlib, handling large files, and common data science file formats.

---

## 1. Reading and Writing Text Files

### Writing
```python
with open("output.txt", "w", encoding="utf-8") as f:
    f.write("Line 1\n")
    f.write("Line 2\n")
    f.writelines(["Line 3\n", "Line 4\n"])

# Append mode
with open("output.txt", "a") as f:
    f.write("Line 5\n")
```

### Reading
```python
# Read entire file
with open("output.txt", "r", encoding="utf-8") as f:
    content = f.read()
    print(content)

# Read line by line (memory efficient)
with open("output.txt") as f:
    for line in f:
        print(line.strip())

# Read into a list of lines
with open("output.txt") as f:
    lines = f.readlines()

# Read first N lines
with open("output.txt") as f:
    first_3 = [next(f).strip() for _ in range(3)]
```

### File modes reference

| Mode  | Description                          |
|-------|--------------------------------------|
| `"r"` | Read (default) — file must exist     |
| `"w"` | Write — creates or truncates         |
| `"a"` | Append — creates or appends          |
| `"x"` | Exclusive create — fails if exists   |
| `"b"` | Binary mode (e.g., `"rb"`, `"wb"`)   |
| `"+"` | Read + write (e.g., `"r+"`)          |

---

## 2. CSV Files

### Using the csv module
```python
import csv

# Writing CSV
data = [
    ["name", "age", "score"],
    ["Alice", 25, 92.5],
    ["Bob", 30, 87.3],
    ["Charlie", 22, 95.1],
]

with open("students.csv", "w", newline="", encoding="utf-8") as f:
    writer = csv.writer(f)
    writer.writerows(data)

# Reading CSV
with open("students.csv", "r", encoding="utf-8") as f:
    reader = csv.reader(f)
    header = next(reader)
    for row in reader:
        name, age, score = row[0], int(row[1]), float(row[2])
        print(f"{name}: age={age}, score={score}")
```

### DictReader / DictWriter — column names as keys
```python
# Writing with DictWriter
records = [
    {"name": "Alice", "age": 25, "score": 92.5},
    {"name": "Bob",   "age": 30, "score": 87.3},
]

with open("students.csv", "w", newline="") as f:
    writer = csv.DictWriter(f, fieldnames=["name", "age", "score"])
    writer.writeheader()
    writer.writerows(records)

# Reading with DictReader
with open("students.csv") as f:
    reader = csv.DictReader(f)
    for row in reader:
        print(row)  # {'name': 'Alice', 'age': '25', 'score': '92.5'}
```

---

## 3. JSON Files

```python
import json

# Python dict → JSON file
config = {
    "model": "random_forest",
    "hyperparams": {"n_estimators": 100, "max_depth": 10},
    "features": ["age", "income", "score"]
}

# Write JSON
with open("config.json", "w") as f:
    json.dump(config, f, indent=2)

# Read JSON
with open("config.json") as f:
    loaded = json.load(f)
    print(loaded["hyperparams"]["max_depth"])  # 10
```

### String conversion
```python
json_str = json.dumps(config, indent=2)   # dict → JSON string
parsed   = json.loads(json_str)            # JSON string → dict
```

### Handling non-serializable types
```python
from datetime import datetime

class CustomEncoder(json.JSONEncoder):
    def default(self, obj):
        if isinstance(obj, datetime):
            return obj.isoformat()
        return super().default(obj)

data = {"event": "training", "timestamp": datetime.now()}
print(json.dumps(data, cls=CustomEncoder, indent=2))
```

---

## 4. Pickle — Python Object Serialization

```python
import pickle

# Serialize (save)
model_data = {
    "weights": [0.5, -0.3, 0.8],
    "bias": 0.1,
    "accuracy": 0.95
}

with open("model.pkl", "wb") as f:
    pickle.dump(model_data, f)

# Deserialize (load)
with open("model.pkl", "rb") as f:
    loaded_model = pickle.load(f)
    print(loaded_model["accuracy"])  # 0.95
```

> ⚠️ **Security warning:** Never unpickle data from untrusted sources — it can execute arbitrary code.

### Pickle vs JSON

| Feature        | pickle                    | JSON                     |
|----------------|---------------------------|--------------------------|
| Format         | Binary                    | Human-readable text      |
| Python-only    | ✅ Yes                    | ❌ Language-agnostic     |
| Speed          | Fast                      | Slower                   |
| Security       | ⚠️ Unsafe with untrusted | ✅ Safe                  |
| Types          | Any Python object         | str, int, float, list, dict, bool, None |

---

## 5. pathlib — Modern Path Handling

```python
from pathlib import Path

# Create path objects
data_dir = Path("data")
file_path = data_dir / "raw" / "dataset.csv"   # / operator joins paths

print(file_path)            # data/raw/dataset.csv
print(file_path.name)       # dataset.csv
print(file_path.stem)       # dataset
print(file_path.suffix)     # .csv
print(file_path.parent)     # data/raw
print(file_path.exists())   # True/False
print(file_path.is_file())  # True/False
```

### Common operations
```python
# Create directories
Path("output/results").mkdir(parents=True, exist_ok=True)

# List files
for f in Path("data").glob("*.csv"):
    print(f)

# Recursive glob
for f in Path(".").rglob("*.py"):
    print(f)

# Read/write shortcuts
path = Path("notes.txt")
path.write_text("Hello, pathlib!", encoding="utf-8")
content = path.read_text(encoding="utf-8")

# Rename / delete
path.rename("notes_backup.txt")
# path.unlink()  # delete file
```

### Building paths dynamically
```python
from pathlib import Path
from datetime import date

output_dir = Path("results") / str(date.today())
output_dir.mkdir(parents=True, exist_ok=True)
output_file = output_dir / "metrics.json"
```

---

## 6. Working with Large Files

### Chunk reading
```python
def process_large_file(filepath, chunk_size=8192):
    """Read a large file in chunks to limit memory usage."""
    with open(filepath, "r") as f:
        while True:
            chunk = f.read(chunk_size)
            if not chunk:
                break
            process(chunk)
```

### Line-by-line streaming
```python
def count_lines(filepath):
    """Count lines without loading entire file."""
    count = 0
    with open(filepath) as f:
        for _ in f:
            count += 1
    return count
```

### Processing large CSVs in batches
```python
import csv

def process_csv_batches(filepath, batch_size=1000):
    """Yield batches of rows from a large CSV."""
    with open(filepath) as f:
        reader = csv.DictReader(f)
        batch = []
        for row in reader:
            batch.append(row)
            if len(batch) >= batch_size:
                yield batch
                batch = []
        if batch:
            yield batch

# Usage
for batch in process_csv_batches("large_data.csv", batch_size=500):
    print(f"Processing batch of {len(batch)} rows")
```

---

## 7. Binary Files

```python
# Writing binary
data = bytes([0x48, 0x65, 0x6C, 0x6C, 0x6F])  # "Hello"
with open("binary.dat", "wb") as f:
    f.write(data)

# Reading binary
with open("binary.dat", "rb") as f:
    raw = f.read()
    print(raw)           # b'Hello'
    print(list(raw))     # [72, 101, 108, 108, 111]
```

### struct — pack/unpack binary data
```python
import struct

# Pack: int (4 bytes) + float (4 bytes)
packed = struct.pack("if", 42, 3.14)
with open("record.bin", "wb") as f:
    f.write(packed)

# Unpack
with open("record.bin", "rb") as f:
    data = f.read()
    num, val = struct.unpack("if", data)
    print(num, val)  # 42  3.140000104904175
```

---

## 8. Common Data Science File Formats

| Format   | Module / Library         | Use Case                              |
|----------|--------------------------|---------------------------------------|
| CSV      | `csv`, `pandas`          | Tabular data (universal)              |
| JSON     | `json`                   | Config, API responses, nested data    |
| Pickle   | `pickle`, `joblib`       | Python objects, trained models        |
| Parquet  | `pyarrow`, `pandas`      | Columnar storage (big data)           |
| HDF5     | `h5py`, `pandas`         | Large numerical datasets              |
| Feather  | `pyarrow`                | Fast DataFrame interchange            |
| YAML     | `pyyaml`                 | Configuration files                   |
| SQLite   | `sqlite3`                | Embedded relational database          |
| NPY/NPZ  | `numpy`                 | NumPy array storage                   |

### Quick pandas example
```python
# pandas handles many formats with a consistent API
import pandas as pd

# Read
df = pd.read_csv("data.csv")
df = pd.read_json("data.json")
df = pd.read_parquet("data.parquet")

# Write
df.to_csv("output.csv", index=False)
df.to_json("output.json", orient="records", indent=2)
df.to_parquet("output.parquet")
```

---

## 9. Context Managers for Files — Why They Matter

```python
# WITHOUT context manager — risk of resource leak
f = open("data.txt")
try:
    data = f.read()
finally:
    f.close()

# WITH context manager — clean and safe
with open("data.txt") as f:
    data = f.read()
# File is guaranteed closed, even if an exception occurs
```

### Temporary files
```python
import tempfile

with tempfile.NamedTemporaryFile(mode="w", suffix=".csv", delete=False) as tmp:
    tmp.write("a,b,c\n1,2,3\n")
    print(f"Temp file: {tmp.name}")
# File persists because delete=False; clean up manually
```

---

## Summary Table

| Task                    | Recommended Approach                              |
|-------------------------|---------------------------------------------------|
| Read/write text         | `open()` with `with` statement                    |
| CSV read/write          | `csv.DictReader` / `csv.DictWriter`               |
| JSON read/write         | `json.load()` / `json.dump()`                     |
| Serialize Python objects| `pickle.dump()` / `pickle.load()`                 |
| Path manipulation       | `pathlib.Path`                                    |
| Large files             | Line-by-line iteration or chunk reading           |
| Binary data             | `"rb"` / `"wb"` modes + `struct`                  |
| DataFrames              | `pandas.read_csv()` / `df.to_csv()`               |
| Temp files              | `tempfile.NamedTemporaryFile`                     |

---

## Best Practices

1. **Always use `with` statements** for file operations — prevents resource leaks.
2. **Specify encoding** explicitly (`encoding="utf-8"`) to avoid platform-dependent behavior.
3. **Use `pathlib`** instead of `os.path` for cleaner, cross-platform path handling.
4. **Stream large files** line-by-line or in chunks — never load GBs into memory.
5. **Use JSON for configs**, pickle for model serialization, CSV/Parquet for tabular data.
6. **Add `newline=""`** when writing CSV on Windows to prevent extra blank lines.
7. **Never unpickle untrusted data** — it's a security risk.

---

## Revision Questions

1. What is the difference between `"r"`, `"w"`, `"a"`, and `"x"` file modes?
2. Why should you use `csv.DictReader` over `csv.reader` in most cases?
3. How do you handle a JSON file that contains `datetime` objects?
4. Explain why `pathlib.Path` is preferred over string concatenation for file paths.
5. Write a function that reads a large CSV in batches of N rows using a generator.
6. What are the security concerns with using `pickle`? When should you use JSON instead?

---

| [◀️ Previous: Exception Handling](./06-exception-handling.md) | [🏠 Home](../README.md) |
|:---|---:|
