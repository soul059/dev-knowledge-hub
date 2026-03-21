# Chapter 4: List Comprehensions and Generator Expressions

> **Overview:** Comprehensions provide a concise, readable, and often faster way to create collections in Python. This chapter covers list, dictionary, and set comprehensions, generator expressions, and practical data science examples.

---

## 1. Basic List Comprehension

**Syntax:** `[expression for item in iterable]`

```python
# Traditional loop
squares = []
for x in range(1, 6):
    squares.append(x ** 2)

# Comprehension — same result, one line
squares = [x ** 2 for x in range(1, 6)]
print(squares)  # [1, 4, 9, 16, 25]
```

---

## 2. Conditional Comprehension

### Filtering (if)
```python
nums = range(1, 21)
evens = [x for x in nums if x % 2 == 0]
print(evens)  # [2, 4, 6, 8, 10, 12, 14, 16, 18, 20]
```

### Conditional expression (if-else)
```python
labels = ["even" if x % 2 == 0 else "odd" for x in range(1, 6)]
print(labels)  # ['odd', 'even', 'odd', 'even', 'odd']
```

### Multiple conditions
```python
# Numbers divisible by both 3 and 5
fizzbuzz = [x for x in range(1, 31) if x % 3 == 0 if x % 5 == 0]
print(fizzbuzz)  # [15, 30]
```

---

## 3. Nested Comprehension

### Flattening a 2D list
```python
matrix = [[1, 2, 3],
          [4, 5, 6],
          [7, 8, 9]]

flat = [val for row in matrix for val in row]
print(flat)  # [1, 2, 3, 4, 5, 6, 7, 8, 9]
```

> **Reading order:** Read nested comprehensions left-to-right, matching the equivalent nested `for` loop order.

### Creating a 2D list
```python
# 3×3 identity-like matrix
grid = [[1 if i == j else 0 for j in range(3)] for i in range(3)]
# [[1, 0, 0], [0, 1, 0], [0, 0, 1]]
```

### Cartesian product
```python
colors = ["red", "blue"]
sizes  = ["S", "M", "L"]
combos = [(c, s) for c in colors for s in sizes]
# [('red', 'S'), ('red', 'M'), ('red', 'L'),
#  ('blue', 'S'), ('blue', 'M'), ('blue', 'L')]
```

---

## 4. Dictionary Comprehension

**Syntax:** `{key_expr: val_expr for item in iterable}`

```python
words = ["hello", "world", "python"]
word_lengths = {w: len(w) for w in words}
print(word_lengths)  # {'hello': 5, 'world': 5, 'python': 6}

# Inverting a dictionary
original = {"a": 1, "b": 2, "c": 3}
inverted = {v: k for k, v in original.items()}
print(inverted)  # {1: 'a', 2: 'b', 3: 'c'}

# Filtering dictionary entries
scores = {"Alice": 92, "Bob": 67, "Charlie": 85, "Diana": 45}
passed = {name: score for name, score in scores.items() if score >= 70}
print(passed)  # {'Alice': 92, 'Charlie': 85}
```

---

## 5. Set Comprehension

**Syntax:** `{expression for item in iterable}`

```python
sentence = "hello world hello python world"
unique_lengths = {len(word) for word in sentence.split()}
print(unique_lengths)  # {5, 6}

# Remove duplicates with transformation
names = ["Alice", "BOB", "alice", "Charlie", "bob"]
unique_names = {name.lower() for name in names}
print(unique_names)  # {'alice', 'bob', 'charlie'}
```

---

## 6. Generator Expressions

Generator expressions look like list comprehensions but use parentheses `()`. They produce values **lazily** — one at a time — saving memory.

```python
# List comprehension — builds entire list in memory
sum_sq_list = sum([x**2 for x in range(1_000_000)])

# Generator expression — computes on the fly, no list in memory
sum_sq_gen = sum(x**2 for x in range(1_000_000))

# Both give the same result, but the generator uses O(1) memory.
```

### Checking conditions lazily
```python
data = [2, 4, 6, 8, 10]
all_even = all(x % 2 == 0 for x in data)   # True — short-circuits
any_neg  = any(x < 0 for x in data)         # False
```

### Converting to other types
```python
# Generator → list
gen = (x**2 for x in range(5))
print(list(gen))  # [0, 1, 4, 9, 16]

# Generator → tuple
t = tuple(x * 10 for x in range(4))
print(t)  # (0, 10, 20, 30)
```

---

## 7. Comprehension vs Loop — Performance

```python
import timeit

n = 100_000

# Loop approach
def loop_squares():
    result = []
    for i in range(n):
        result.append(i ** 2)
    return result

# Comprehension approach
def comp_squares():
    return [i ** 2 for i in range(n)]

loop_time = timeit.timeit(loop_squares, number=100)
comp_time = timeit.timeit(comp_squares, number=100)

print(f"Loop:          {loop_time:.3f}s")
print(f"Comprehension: {comp_time:.3f}s")
# Comprehension is typically 10-30% faster due to optimized bytecode.
```

### Why comprehensions are faster
1. No repeated `.append()` method lookups.
2. Python optimizes the bytecode for comprehensions internally.
3. The intent is clearer, so CPython can apply optimizations.

---

## 8. Practical Data Science Examples

### Feature extraction — word count vector
```python
documents = [
    "machine learning is great",
    "deep learning is a subset of machine learning",
    "great progress in deep learning"
]

vocab = sorted({word for doc in documents for word in doc.split()})

vectors = [
    [doc.split().count(word) for word in vocab]
    for doc in documents
]
# Each row is a bag-of-words vector
for doc, vec in zip(documents, vectors):
    print(vec)
```

### Data filtering — cleaning dataset rows
```python
raw_data = [
    {"name": "Alice", "age": 25, "score": 88},
    {"name": "Bob",   "age": None, "score": 72},
    {"name": "Charlie", "age": 30, "score": None},
    {"name": "Diana", "age": 22, "score": 95},
]

# Keep only complete records
clean = [row for row in raw_data
         if all(v is not None for v in row.values())]
print(clean)
# [{'name': 'Alice', 'age': 25, 'score': 88},
#  {'name': 'Diana', 'age': 22, 'score': 95}]
```

### One-hot encoding
```python
categories = ["cat", "dog", "bird", "cat", "bird"]
unique = sorted(set(categories))

one_hot = [
    [1 if cat == u else 0 for u in unique]
    for cat in categories
]
print(f"Columns: {unique}")
for row in one_hot:
    print(row)
# Columns: ['bird', 'cat', 'dog']
# [0, 1, 0]
# [0, 0, 1]
# [1, 0, 0]
# [0, 1, 0]
# [1, 0, 0]
```

### Normalization with comprehension
```python
data = [10, 20, 30, 40, 50]
lo, hi = min(data), max(data)
normalized = [(x - lo) / (hi - lo) for x in data]
print(normalized)  # [0.0, 0.25, 0.5, 0.75, 1.0]
```

---

## Summary Table

| Type                  | Syntax                                    | Returns       | Memory    |
|-----------------------|-------------------------------------------|---------------|-----------|
| List comprehension    | `[expr for x in iter]`                    | `list`        | Eager     |
| Dict comprehension    | `{k: v for x in iter}`                    | `dict`        | Eager     |
| Set comprehension     | `{expr for x in iter}`                    | `set`         | Eager     |
| Generator expression  | `(expr for x in iter)`                    | `generator`   | **Lazy**  |
| Nested comprehension  | `[expr for x in iter1 for y in iter2]`    | `list`        | Eager     |
| Filtered comprehension| `[expr for x in iter if cond]`            | `list`        | Eager     |

---

## Best Practices

1. **Keep comprehensions short** — if it exceeds ~80 characters, use a regular loop.
2. **Avoid side effects** inside comprehensions (e.g., `print`, file writes).
3. **Use generator expressions** with `sum()`, `any()`, `all()`, `min()`, `max()` to save memory.
4. **Prefer comprehensions over `map`/`filter`** for readability.
5. **Don't nest more than two levels** — readability degrades quickly.
6. **Name complex expressions** — extract helper functions for clarity.

---

## Revision Questions

1. Rewrite `list(filter(lambda x: x > 0, nums))` as a list comprehension.
2. What is the difference between a list comprehension and a generator expression? When should you use each?
3. Write a dictionary comprehension that maps characters to their ASCII values for `a-z`.
4. Flatten a 3D list `[[[1,2],[3,4]],[[5,6],[7,8]]]` using nested comprehensions.
5. Why are list comprehensions generally faster than equivalent `for` loops with `.append()`?
6. Write a set comprehension that extracts all unique file extensions from a list of filenames.

---

| [◀️ Previous: Functions & Lambda](./03-functions-and-lambda.md) | [🏠 Home](../README.md) | [Next: OOP Basics ▶️](./05-oop-basics.md) |
|:---|:---:|---:|
