# Chapter 2: Control Flow

> **Overview:** Control flow determines the order in which statements execute. Mastering conditionals, loops, and iteration patterns is essential for writing efficient data-processing code in Python.

---

## 1. Conditional Statements — if / elif / else

```python
score = 85

if score >= 90:
    grade = "A"
elif score >= 80:
    grade = "B"
elif score >= 70:
    grade = "C"
else:
    grade = "F"

print(grade)  # B
```

### Truthiness rules
```python
# Falsy: 0, 0.0, "", [], {}, set(), None, False
# Everything else is truthy.

data = []
if data:
    print("Has data")
else:
    print("Empty")       # ← prints this
```

---

## 2. Ternary (Conditional) Expression

```python
age = 20
status = "adult" if age >= 18 else "minor"
print(status)  # adult

# Nested (use sparingly)
label = "high" if score > 80 else ("mid" if score > 50 else "low")
```

---

## 3. for Loops

### Iterating over sequences
```python
fruits = ["apple", "banana", "cherry"]
for fruit in fruits:
    print(fruit)
```

### range()
```python
for i in range(5):          # 0, 1, 2, 3, 4
    print(i)

for i in range(2, 10, 3):   # 2, 5, 8
    print(i)
```

### enumerate() — index + value
```python
names = ["Alice", "Bob", "Charlie"]
for idx, name in enumerate(names, start=1):
    print(f"{idx}. {name}")
# 1. Alice
# 2. Bob
# 3. Charlie
```

### zip() — parallel iteration
```python
features = ["height", "weight", "age"]
values   = [175, 70, 30]

for feat, val in zip(features, values):
    print(f"{feat}: {val}")

# Unequal lengths → stops at shortest; use itertools.zip_longest for padding
```

### Iterating over dictionaries
```python
config = {"lr": 0.01, "epochs": 100, "batch": 32}

for key in config:                    # keys only
    print(key)

for key, val in config.items():       # key-value pairs
    print(f"{key} = {val}")

for val in config.values():           # values only
    print(val)
```

---

## 4. while Loops

```python
count = 0
while count < 5:
    print(count)
    count += 1

# Sentinel pattern — reading until condition
data = []
while len(data) < 3:
    data.append(len(data) * 10)
print(data)  # [0, 10, 20]
```

### Infinite loop with break
```python
import random
while True:
    num = random.randint(1, 10)
    if num == 7:
        print("Found 7!")
        break
```

---

## 5. Loop Control — break, continue, pass

```python
# break — exit the loop immediately
for n in range(10):
    if n == 5:
        break
    print(n)        # 0 1 2 3 4

# continue — skip to next iteration
for n in range(6):
    if n % 2 == 0:
        continue
    print(n)        # 1 3 5

# pass — placeholder, does nothing
for n in range(3):
    pass            # TODO: implement later
```

### for-else pattern
```python
# The else block runs only if the loop completes WITHOUT a break.
primes = [2, 3, 5, 7, 11]
target = 4
for p in primes:
    if p == target:
        print("Found!")
        break
else:
    print(f"{target} not in list")   # ← prints this
```

---

## 6. Nested Loops

```python
matrix = [[1, 2, 3],
          [4, 5, 6],
          [7, 8, 9]]

for row in matrix:
    for val in row:
        print(val, end=" ")
    print()
# 1 2 3
# 4 5 6
# 7 8 9
```

### Flattening a matrix
```python
flat = [val for row in matrix for val in row]
print(flat)  # [1, 2, 3, 4, 5, 6, 7, 8, 9]
```

---

## 7. match-case (Structural Pattern Matching — Python 3.10+)

```python
def classify(status_code):
    match status_code:
        case 200:
            return "OK"
        case 301 | 302:
            return "Redirect"
        case 404:
            return "Not Found"
        case 500:
            return "Server Error"
        case _:
            return "Unknown"

print(classify(404))  # Not Found
```

### Pattern matching with destructuring
```python
point = (3, 0)
match point:
    case (0, 0):
        print("Origin")
    case (x, 0):
        print(f"On X-axis at {x}")
    case (0, y):
        print(f"On Y-axis at {y}")
    case (x, y):
        print(f"Point({x}, {y})")
# On X-axis at 3
```

---

## 8. Common Loop Patterns for Data Science

### Accumulator pattern
```python
numbers = [10, 20, 30, 40]
total = 0
for n in numbers:
    total += n
print(total)  # 100  (or simply use sum(numbers))
```

### Building a frequency table
```python
words = ["cat", "dog", "cat", "bird", "dog", "cat"]
freq = {}
for w in words:
    freq[w] = freq.get(w, 0) + 1
print(freq)  # {'cat': 3, 'dog': 2, 'bird': 1}
```

### Filtering with enumeration
```python
scores = [88, 42, 95, 67, 55, 91]
passed = [(i, s) for i, s in enumerate(scores) if s >= 60]
print(passed)  # [(0, 88), (2, 95), (3, 67), (5, 91)]
```

### Pairwise iteration
```python
data = [1, 4, 6, 9, 15]
diffs = [data[i+1] - data[i] for i in range(len(data) - 1)]
print(diffs)  # [3, 2, 3, 6]
```

### Sliding window
```python
def moving_avg(data, window=3):
    return [sum(data[i:i+window]) / window
            for i in range(len(data) - window + 1)]

print(moving_avg([1, 3, 5, 7, 9]))  # [3.0, 5.0, 7.0]
```

---

## Summary Table

| Construct          | Purpose                                  | Key Syntax              |
|--------------------|------------------------------------------|-------------------------|
| `if/elif/else`     | Branching logic                          | `if cond: ... else: ...`|
| `for`              | Iterate over sequences                   | `for x in iterable:`    |
| `while`            | Loop until condition is false            | `while cond:`           |
| `range()`          | Generate integer sequences               | `range(start, stop, step)` |
| `enumerate()`      | Index + value pairs                      | `enumerate(seq, start)` |
| `zip()`            | Parallel iteration                       | `zip(iter1, iter2)`     |
| `break`            | Exit loop immediately                    | `break`                 |
| `continue`         | Skip to next iteration                   | `continue`              |
| `pass`             | No-op placeholder                        | `pass`                  |
| `match-case`       | Structural pattern matching (3.10+)      | `match val: case p:`    |
| Ternary            | Inline conditional                       | `x if cond else y`      |

---

## Best Practices

1. **Prefer `for` over `while`** when the number of iterations is known.
2. **Use `enumerate()`** instead of manual index counters.
3. **Use `zip()`** to iterate over parallel sequences cleanly.
4. **Avoid deeply nested loops** — refactor into functions or use comprehensions.
5. **Use `for-else`** judiciously — most developers find it confusing; add a comment.
6. **Prefer built-in functions** (`sum`, `min`, `max`, `any`, `all`) over manual loops.

---

## Revision Questions

1. What values does Python consider *falsy*? List at least five.
2. Explain the difference between `break` and `continue` with an example.
3. When does the `else` block of a `for` loop execute?
4. Write a `while` loop that computes the factorial of a given number.
5. How does `zip()` handle iterables of different lengths? How can you override this?
6. Rewrite nested loops that flatten a 2D list using a single list comprehension.

---

| [◀️ Previous: Data Types](./01-data-types-and-structures.md) | [🏠 Home](../README.md) | [Next: Functions & Lambda ▶️](./03-functions-and-lambda.md) |
|:---|:---:|---:|
