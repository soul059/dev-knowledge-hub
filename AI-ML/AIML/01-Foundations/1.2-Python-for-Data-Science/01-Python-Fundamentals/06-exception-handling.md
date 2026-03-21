# Chapter 6: Exception Handling

> **Overview:** Robust data pipelines must anticipate and handle errors gracefully. This chapter covers Python's exception mechanism — try/except/else/finally, raising and creating custom exceptions, the exception hierarchy, context managers, and common patterns for data loading and processing.

---

## 1. Basic try / except

```python
try:
    result = 10 / 0
except ZeroDivisionError:
    print("Cannot divide by zero!")
# Output: Cannot divide by zero!
```

### Catching multiple exceptions
```python
try:
    value = int("abc")
except (ValueError, TypeError) as e:
    print(f"Error: {e}")
# Error: invalid literal for int() with base 10: 'abc'
```

### Catching all exceptions (use sparingly)
```python
try:
    risky_operation()
except Exception as e:
    print(f"Unexpected error: {type(e).__name__}: {e}")
```

> ⚠️ Avoid bare `except:` — it catches `SystemExit` and `KeyboardInterrupt` too.

---

## 2. else and finally

```python
try:
    f = open("data.txt", "r")
    content = f.read()
except FileNotFoundError:
    print("File not found!")
else:
    # Runs ONLY if no exception was raised
    print(f"Read {len(content)} characters")
finally:
    # ALWAYS runs — cleanup code goes here
    print("Cleanup complete")
```

### Flow summary
| Block     | Runs when…                                |
|-----------|-------------------------------------------|
| `try`     | Always — code to attempt                  |
| `except`  | Only if an exception matches              |
| `else`    | Only if NO exception occurred             |
| `finally` | ALWAYS — cleanup, closing resources, etc. |

---

## 3. Raising Exceptions

```python
def validate_age(age):
    if not isinstance(age, int):
        raise TypeError(f"Expected int, got {type(age).__name__}")
    if age < 0 or age > 150:
        raise ValueError(f"Age {age} is out of valid range (0-150)")
    return True

try:
    validate_age(-5)
except ValueError as e:
    print(e)  # Age -5 is out of valid range (0-150)
```

### Re-raising exceptions
```python
try:
    data = load_data("file.csv")
except FileNotFoundError:
    print("Logging: file not found")
    raise   # re-raise the original exception
```

---

## 4. Custom Exceptions

```python
class DataValidationError(Exception):
    """Raised when data fails validation checks."""
    def __init__(self, column, message):
        self.column = column
        self.message = message
        super().__init__(f"Column '{column}': {message}")

class MissingValueError(DataValidationError):
    """Raised when required values are missing."""
    pass

# Usage
def validate_column(name, values):
    if any(v is None for v in values):
        raise MissingValueError(name, "contains None values")

try:
    validate_column("age", [25, None, 30])
except MissingValueError as e:
    print(e)           # Column 'age': contains None values
    print(e.column)    # age
```

---

## 5. Built-in Exception Hierarchy

```
BaseException
├── SystemExit
├── KeyboardInterrupt
├── GeneratorExit
└── Exception
    ├── ArithmeticError
    │   ├── ZeroDivisionError
    │   ├── OverflowError
    │   └── FloatingPointError
    ├── LookupError
    │   ├── IndexError
    │   └── KeyError
    ├── ValueError
    ├── TypeError
    ├── AttributeError
    ├── FileNotFoundError
    ├── IOError / OSError
    ├── ImportError
    │   └── ModuleNotFoundError
    ├── StopIteration
    └── RuntimeError
```

### Common exceptions in data science

| Exception            | Common Cause                              |
|----------------------|-------------------------------------------|
| `FileNotFoundError`  | Missing data file or wrong path           |
| `ValueError`         | Wrong data format, e.g., `int("abc")`     |
| `KeyError`           | Missing dictionary key / DataFrame column |
| `IndexError`         | Out-of-range list index                   |
| `TypeError`          | Wrong argument type                       |
| `ImportError`        | Missing library                           |
| `MemoryError`        | Dataset too large for available RAM       |
| `UnicodeDecodeError` | Wrong file encoding                       |

---

## 6. Context Managers — the `with` Statement

Context managers ensure resources are properly acquired and released.

```python
# File handling — automatically closes file
with open("output.txt", "w") as f:
    f.write("Hello, World!")
# f is automatically closed here, even if an error occurs

# Multiple context managers
with open("in.txt") as src, open("out.txt", "w") as dst:
    dst.write(src.read())
```

### Writing a custom context manager (class-based)
```python
class Timer:
    """Context manager to time a code block."""
    import time

    def __enter__(self):
        self.start = __import__("time").perf_counter()
        return self

    def __exit__(self, exc_type, exc_val, exc_tb):
        self.elapsed = __import__("time").perf_counter() - self.start
        print(f"Elapsed: {self.elapsed:.4f}s")
        return False   # don't suppress exceptions

with Timer():
    total = sum(range(1_000_000))
# Elapsed: 0.0234s
```

### Using contextlib
```python
from contextlib import contextmanager

@contextmanager
def managed_resource(name):
    print(f"Acquiring {name}")
    try:
        yield name
    finally:
        print(f"Releasing {name}")

with managed_resource("database") as r:
    print(f"Using {r}")
# Acquiring database
# Using database
# Releasing database
```

---

## 7. Common Patterns for Data Loading

### Safe file reading with fallback
```python
def read_data(filepath, default=None):
    try:
        with open(filepath, "r", encoding="utf-8") as f:
            return f.read()
    except FileNotFoundError:
        print(f"Warning: {filepath} not found, using default")
        return default
    except UnicodeDecodeError:
        with open(filepath, "r", encoding="latin-1") as f:
            return f.read()
```

### Parsing rows with error collection
```python
def parse_csv_rows(rows):
    """Parse numeric rows, collecting errors instead of stopping."""
    parsed = []
    errors = []
    for i, row in enumerate(rows):
        try:
            parsed.append([float(x) for x in row.split(",")])
        except ValueError as e:
            errors.append({"row": i, "data": row, "error": str(e)})
    return parsed, errors

data_rows = ["1.0,2.0,3.0", "4.0,bad,6.0", "7.0,8.0,9.0"]
good, bad = parse_csv_rows(data_rows)
print(f"Parsed: {len(good)} rows, Errors: {len(bad)} rows")
# Parsed: 2 rows, Errors: 1 rows
```

### Retry pattern for flaky operations
```python
import time

def retry(func, max_attempts=3, delay=1.0):
    """Retry a function up to max_attempts times."""
    for attempt in range(1, max_attempts + 1):
        try:
            return func()
        except Exception as e:
            print(f"Attempt {attempt} failed: {e}")
            if attempt == max_attempts:
                raise
            time.sleep(delay)
```

### Error handling in data pipelines
```python
def etl_pipeline(source_path, dest_path):
    """Extract-Transform-Load with comprehensive error handling."""
    # Extract
    try:
        with open(source_path) as f:
            raw = f.readlines()
    except FileNotFoundError:
        raise FileNotFoundError(f"Source file not found: {source_path}")

    # Transform
    processed = []
    for i, line in enumerate(raw):
        try:
            values = [float(x) for x in line.strip().split(",")]
            processed.append(values)
        except ValueError:
            print(f"Skipping malformed row {i}: {line.strip()}")

    if not processed:
        raise ValueError("No valid data rows found after transformation")

    # Load
    try:
        with open(dest_path, "w") as f:
            for row in processed:
                f.write(",".join(str(v) for v in row) + "\n")
    except PermissionError:
        raise PermissionError(f"Cannot write to {dest_path}")

    return len(processed)
```

---

## 8. EAFP vs LBYL

Python favors **EAFP** (Easier to Ask Forgiveness than Permission):

```python
# LBYL (Look Before You Leap) — checks first
if "key" in my_dict:
    value = my_dict["key"]

# EAFP (Pythonic) — try and handle
try:
    value = my_dict["key"]
except KeyError:
    value = default_value

# Even better — use .get()
value = my_dict.get("key", default_value)
```

---

## Summary Table

| Concept             | Syntax / Pattern                          | When to Use                      |
|---------------------|-------------------------------------------|----------------------------------|
| `try/except`        | `try: ... except Error: ...`              | Catch specific known errors      |
| `else`              | After `except`, before `finally`          | Code that needs no exception     |
| `finally`           | `finally: ...`                            | Cleanup (close files, connections)|
| `raise`             | `raise ValueError("msg")`                | Signal an error condition        |
| Custom exception    | `class MyError(Exception):`               | Domain-specific error types      |
| Context manager     | `with resource() as r:`                   | Auto resource management         |
| `@contextmanager`   | `from contextlib import contextmanager`   | Lightweight custom context mgr   |
| Retry pattern       | Loop with try/except                      | Flaky I/O, network calls         |
| Error collection    | Append errors to list, continue           | Batch data processing            |

---

## Best Practices

1. **Catch specific exceptions** — never use bare `except:`.
2. **Keep try blocks small** — only wrap the code that might fail.
3. **Use `else`** for code that should run only on success.
4. **Always use `with`** for files, database connections, and locks.
5. **Create custom exceptions** for domain-specific errors in larger projects.
6. **Log errors** before re-raising in production pipelines.
7. **Collect errors** instead of failing fast when processing batches of data.

---

## Revision Questions

1. What is the difference between `except Exception` and bare `except:`?
2. When does the `else` block execute in a try/except/else/finally structure?
3. Write a custom exception class `InvalidFeatureError` with column name and reason.
4. Explain EAFP vs LBYL with a practical example.
5. Write a context manager using `@contextmanager` that temporarily changes the working directory.
6. Design an error-collecting CSV parser that returns both valid rows and a list of errors.

---

| [◀️ Previous: OOP Basics](./05-oop-basics.md) | [🏠 Home](../README.md) | [Next: File I/O ▶️](./07-file-io.md) |
|:---|:---:|---:|
