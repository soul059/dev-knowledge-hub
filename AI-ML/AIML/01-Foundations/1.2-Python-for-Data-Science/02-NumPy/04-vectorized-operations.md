# Chapter 4: Vectorized Operations

> **Unit 2 — NumPy** | Replace Python loops with fast, element-wise array operations — the heart of NumPy's performance.

---

## 4.1 Element-Wise Arithmetic

All standard operators work element-wise on arrays of the same shape (or broadcastable shapes).

```python
import numpy as np

a = np.array([1, 2, 3, 4])
b = np.array([10, 20, 30, 40])

a + b    # [11 22 33 44]
a - b    # [-9 -18 -27 -36]
a * b    # [10 40 90 160]
a / b    # [0.1 0.1 0.1 0.1]
a ** 2   # [1 4 9 16]
a % 3    # [1 2 0 1]
a // 2   # [0 1 1 2]
```

> All of these call optimized C routines under the hood — no Python loop overhead.

---

## 4.2 Universal Functions (ufuncs)

Ufuncs are NumPy functions that operate element-wise and return a new array.

```python
x = np.array([0, np.pi/4, np.pi/2])

# Trigonometric
np.sin(x)     # [0.   0.707 1.  ]
np.cos(x)     # [1.   0.707 0.  ]

# Exponential & logarithmic
np.exp(np.array([0, 1, 2]))       # [1.    2.718 7.389]
np.log(np.array([1, np.e, 100]))  # [0. 1. 4.605]
np.log10(np.array([1, 10, 100]))  # [0. 1. 2.]

# Rounding
np.floor(np.array([1.7, 2.3]))    # [1. 2.]
np.ceil(np.array([1.7, 2.3]))     # [2. 3.]
np.round(np.array([1.75, 2.35]), decimals=1)  # [1.8 2.4]

# Absolute value
np.abs(np.array([-3, -1, 2]))     # [3 1 2]

# Clipping
np.clip(np.array([-2, 3, 7]), 0, 5)  # [0 3 5]
```

---

## 4.3 Aggregation Functions

Aggregations reduce an array (or an axis of an array) to a single value.

```python
a = np.array([[1, 2, 3],
              [4, 5, 6]])

np.sum(a)          # 21          — total
np.sum(a, axis=0)  # [5 7 9]    — sum down rows (per column)
np.sum(a, axis=1)  # [6 15]     — sum across columns (per row)

np.mean(a)         # 3.5
np.std(a)          # 1.707...
np.var(a)          # 2.916...

np.min(a)          # 1
np.max(a, axis=1)  # [3 6]

np.prod(a, axis=1) # [6 120]    — product along rows

# Cumulative
np.cumsum(a, axis=1)  # [[1 3 6] [4 9 15]]
np.cumprod(a, axis=1) # [[1 2 6] [4 20 120]]
```

### Axis Parameter Visual

```
axis=0: ↓ collapse rows     (result has shape of a single row)
axis=1: → collapse columns  (result has shape of a single column)
```

---

## 4.4 Comparison Operations

Comparisons return boolean arrays — perfect for masking.

```python
a = np.array([1, 5, 3, 8, 2])

a > 3            # [False True False True False]
a == 5           # [False True False False False]
a != 2           # [True True True True False]

# Element-wise comparison of two arrays
b = np.array([2, 4, 3, 7, 3])
a > b            # [False True False True False]

# Useful checks
np.all(a > 0)    # True  — every element positive?
np.any(a > 7)    # True  — at least one > 7?

# Count matches
np.sum(a > 3)    # 2  — True counts as 1
```

---

## 4.5 Logical Operations

```python
x = np.array([True, False, True, True])
y = np.array([True, True, False, True])

np.logical_and(x, y)   # [True False False True]
np.logical_or(x, y)    # [True True True True]
np.logical_not(x)      # [False True False False]
np.logical_xor(x, y)   # [False True True False]

# Shorthand with operators (on boolean arrays)
x & y    # same as logical_and
x | y    # same as logical_or
~x       # same as logical_not
```

---

## 4.6 `np.apply_along_axis`

When you need a custom function applied along a specific axis:

```python
m = np.array([[3, 1, 4],
              [1, 5, 9],
              [2, 6, 5]])

# Apply range (max - min) along each row
np.apply_along_axis(lambda row: row.max() - row.min(), axis=1, arr=m)
# [3, 8, 4]

# Apply along columns
np.apply_along_axis(lambda col: np.median(col), axis=0, arr=m)
# [2. 5. 5.]
```

> **Note:** `apply_along_axis` is convenient but slower than pure vectorized code. Use built-in aggregations when possible.

---

## 4.7 Vectorize vs Loops — Timing Comparison

```python
import time

n = 1_000_000
a = np.random.rand(n)
b = np.random.rand(n)

# --- Python loop ---
start = time.perf_counter()
result_loop = [a[i] + b[i] for i in range(n)]
loop_time = time.perf_counter() - start

# --- Vectorized ---
start = time.perf_counter()
result_vec = a + b
vec_time = time.perf_counter() - start

print(f"Loop:       {loop_time:.4f}s")
print(f"Vectorized: {vec_time:.6f}s")
print(f"Speedup:    {loop_time / vec_time:.0f}x")
# Typical output: Loop ~0.3s, Vectorized ~0.001s → ~300x speedup
```

### Why Is Vectorization Faster?

| Factor | Loop | Vectorized |
|--------|------|------------|
| Type checking | Every iteration | Once |
| Memory access | Random (Python objects) | Contiguous (C array) |
| Instruction-level parallelism | None | SIMD / CPU vectorization |
| Interpreter overhead | Per element | Per array |

---

## 4.8 Practical ML Examples

### Euclidean Distance Between Two Vectors

```python
p = np.array([1.0, 2.0, 3.0])
q = np.array([4.0, 5.0, 6.0])

dist = np.sqrt(np.sum((p - q) ** 2))   # 5.196...
# or equivalently
dist = np.linalg.norm(p - q)
```

### Min-Max Normalization (Feature Scaling)

```python
X = np.array([[100, 0.5],
              [200, 0.8],
              [150, 0.3]])

X_min = X.min(axis=0)
X_max = X.max(axis=0)
X_norm = (X - X_min) / (X_max - X_min)
# All values now in [0, 1]
```

### Softmax Function

```python
def softmax(z):
    exp_z = np.exp(z - np.max(z))   # subtract max for numerical stability
    return exp_z / exp_z.sum()

logits = np.array([2.0, 1.0, 0.1])
print(softmax(logits))   # [0.659 0.242 0.099]
```

### Sigmoid Activation

```python
def sigmoid(x):
    return 1 / (1 + np.exp(-x))

x = np.linspace(-6, 6, 7)
print(sigmoid(x))   # values between 0 and 1
```

### Mean Squared Error

```python
y_true = np.array([3.0, -0.5, 2.0, 7.0])
y_pred = np.array([2.5, 0.0, 2.0, 8.0])

mse = np.mean((y_true - y_pred) ** 2)   # 0.375
```

---

## 4.9 Sorting

```python
a = np.array([3, 1, 4, 1, 5, 9])

np.sort(a)              # [1 1 3 4 5 9]  — returns sorted copy
a.sort()                # sorts in-place

# Sort indices (useful for ranking)
idx = np.argsort(a)     # indices that would sort the array

# Sort 2-D along an axis
m = np.array([[3, 1], [2, 4]])
np.sort(m, axis=0)      # sort each column: [[2,1],[3,4]]
np.sort(m, axis=1)      # sort each row:    [[1,3],[2,4]]
```

---

## Best Practices

| Practice | Why |
|----------|-----|
| Replace loops with ufuncs / operators | 100-1000× speedup |
| Always specify `axis` in aggregations | Avoid accidental full-array reduction |
| Subtract max before `exp` (softmax) | Prevents overflow |
| Use `np.linalg.norm` for distances | Cleaner and handles edge cases |
| Prefer built-in aggregations over `apply_along_axis` | Much faster |

---

## Summary Table

| Category | Key Functions |
|----------|---------------|
| Arithmetic | `+`, `-`, `*`, `/`, `**`, `%` |
| Ufuncs | `np.sin`, `np.exp`, `np.log`, `np.abs`, `np.clip` |
| Aggregation | `np.sum`, `np.mean`, `np.std`, `np.min`, `np.max` |
| Cumulative | `np.cumsum`, `np.cumprod` |
| Comparison | `>`, `<`, `==`, `np.all`, `np.any` |
| Logical | `&`, `|`, `~`, `np.logical_and/or/not` |
| Sorting | `np.sort`, `np.argsort` |

---

## Revision Questions

1. What is the difference between `np.sum(a)` and `np.sum(a, axis=0)` for a 2-D array?
2. Write a vectorized one-liner to compute the ReLU function: `max(0, x)` for each element.
3. Why is subtracting `np.max(z)` important in the softmax implementation?
4. Given arrays `y_true` and `y_pred`, write vectorized code for Mean Absolute Error.
5. Explain why `np.any()` and `np.all()` are useful for data validation.
6. Approximately how much faster is vectorized addition compared to a Python loop for 1 million elements?

---

[← Chapter 3: Broadcasting](03-broadcasting.md) | [Next → Chapter 5: Linear Algebra Operations](05-linear-algebra-operations.md)
