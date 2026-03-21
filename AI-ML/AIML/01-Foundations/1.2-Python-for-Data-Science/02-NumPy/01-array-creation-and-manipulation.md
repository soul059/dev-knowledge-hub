# Chapter 1: Array Creation and Manipulation

> **Unit 2 — NumPy** | Create, reshape, combine, and copy arrays — the building blocks of every numerical pipeline.

---

## 1.1 Creating Arrays from Python Objects

```python
import numpy as np

# From a list
a = np.array([1, 2, 3])                    # shape (3,)

# From nested lists → 2-D
b = np.array([[1, 2], [3, 4], [5, 6]])     # shape (3, 2)

# Specifying dtype explicitly
c = np.array([1, 2, 3], dtype=np.float64)  # [1. 2. 3.]
d = np.array([1, 0, 1], dtype=np.bool_)    # [True, False, True]
```

### Common dtypes

| dtype | Description |
|-------|-------------|
| `np.int32` / `np.int64` | 32/64-bit integers |
| `np.float32` / `np.float64` | Single/double precision floats |
| `np.bool_` | Boolean |
| `np.complex128` | Complex numbers |
| `np.str_` | Fixed-length strings |

---

## 1.2 Built-in Array Constructors

```python
# Zeros & ones
np.zeros((2, 3))          # 2×3 matrix of 0.0
np.ones((4,))             # [1. 1. 1. 1.]

# Fill with a constant
np.full((3, 3), 7)        # 3×3 matrix filled with 7

# Identity matrix
np.eye(4)                 # 4×4 identity matrix

# Uninitialized (fast allocation, garbage values)
np.empty((2, 2))          # values are undefined — use with care

# zeros_like / ones_like / full_like — match shape & dtype of another array
template = np.array([[1, 2], [3, 4]])
np.zeros_like(template)   # same shape, filled with 0
```

---

## 1.3 Range-Based Constructors

```python
# arange: like Python range but returns ndarray
np.arange(0, 10, 2)       # [0 2 4 6 8]  — start, stop (exclusive), step

# linspace: N evenly spaced points (inclusive of both endpoints)
np.linspace(0, 1, 5)      # [0.   0.25 0.5  0.75 1.  ]

# logspace: points spaced evenly on a log scale
np.logspace(0, 3, 4)      # [1. 10. 100. 1000.]
```

> **Tip:** Prefer `linspace` when you need a specific *count* of points and `arange` when you need a specific *step*.

---

## 1.4 Shape, Reshape, and Dimensionality

```python
a = np.arange(12)         # shape (12,)

# Reshape to 3×4 — total elements must match
b = a.reshape(3, 4)       # shape (3, 4)

# Use -1 to let NumPy infer one dimension
c = a.reshape(2, -1)      # shape (2, 6)

# Key attributes
print(b.shape)             # (3, 4)
print(b.ndim)              # 2
print(b.size)              # 12
print(b.dtype)             # int64 (platform-dependent)
```

### Flatten vs Ravel

```python
mat = np.array([[1, 2], [3, 4]])

# ravel() — returns a view when possible (no copy)
flat_view = mat.ravel()

# flatten() — always returns a copy
flat_copy = mat.flatten()

flat_view[0] = 99
print(mat[0, 0])   # 99  — ravel shared memory
```

---

## 1.5 Adding and Removing Dimensions

```python
a = np.array([1, 2, 3])   # shape (3,)

# np.newaxis — adds a length-1 axis
row = a[np.newaxis, :]     # shape (1, 3)
col = a[:, np.newaxis]     # shape (3, 1)

# np.expand_dims — equivalent but more explicit
row2 = np.expand_dims(a, axis=0)   # shape (1, 3)

# np.squeeze — removes all length-1 axes
x = np.array([[[1, 2, 3]]])        # shape (1, 1, 3)
print(np.squeeze(x).shape)         # (3,)
```

---

## 1.6 Joining Arrays

```python
a = np.array([1, 2, 3])
b = np.array([4, 5, 6])

# concatenate along an existing axis
np.concatenate([a, b])                  # [1 2 3 4 5 6]

# For 2-D arrays
m1 = np.ones((2, 3))
m2 = np.zeros((2, 3))
np.concatenate([m1, m2], axis=0)        # (4, 3) — row-wise
np.concatenate([m1, m2], axis=1)        # (2, 6) — column-wise

# stack creates a NEW axis
np.stack([a, b])                        # shape (2, 3)
np.stack([a, b], axis=1)               # shape (3, 2)

# Convenience shortcuts
np.vstack([m1, m2])                     # same as axis=0 concatenate
np.hstack([m1, m2])                     # same as axis=1 concatenate
```

---

## 1.7 Splitting Arrays

```python
x = np.arange(12).reshape(3, 4)

# Split into equal parts
a, b, c = np.split(x, 3, axis=0)       # 3 arrays of shape (1, 4)

# Split at specified indices
left, right = np.split(x, [2], axis=1) # shapes (3,2) and (3,2)

# vsplit / hsplit shortcuts
top, bottom = np.vsplit(x, [1])         # split after row 0
```

---

## 1.8 Copy vs View

```python
a = np.array([1, 2, 3, 4, 5])

# Slicing returns a VIEW — shared memory
view = a[1:4]
view[0] = 99
print(a)             # [ 1 99  3  4  5]

# Explicit copy — independent memory
copy = a.copy()
copy[0] = -1
print(a[0])          # 1 (unchanged)
```

> **Rule of Thumb:** Slicing and `reshape` return views; `fancy indexing` and `flatten()` return copies. Use `.base` to check: if `arr.base is not None`, it is a view.

---

## Best Practices

| Practice | Why |
|----------|-----|
| Specify `dtype` at creation time | Avoids implicit upcasting and saves memory |
| Use `reshape(-1, 1)` for column vectors | Required by scikit-learn estimators |
| Prefer `ravel()` over `flatten()` | Avoids unnecessary copies |
| Use `np.empty` only when you will fill every element | Uninitialized memory contains garbage |
| Use `.copy()` when you need to mutate without side effects | Prevents accidental modification of the source |

---

## Summary Table

| Function | Purpose | Example Shape |
|----------|---------|---------------|
| `np.array()` | Create from list | varies |
| `np.zeros()` | All zeros | `(m, n)` |
| `np.ones()` | All ones | `(m, n)` |
| `np.full()` | Constant fill | `(m, n)` |
| `np.eye()` | Identity matrix | `(n, n)` |
| `np.arange()` | Range with step | `(k,)` |
| `np.linspace()` | N evenly spaced | `(N,)` |
| `reshape()` | Change shape | any compatible |
| `ravel()` / `flatten()` | To 1-D | `(k,)` |
| `np.concatenate()` | Join along axis | varies |
| `np.stack()` | Join on new axis | varies |

---

## Revision Questions

1. What is the difference between `np.arange(0, 1, 0.1)` and `np.linspace(0, 1, 10)` in terms of endpoints and element count?
2. Explain why modifying a slice of an array changes the original array. How do you prevent this?
3. Given `a = np.arange(24)`, write one expression to reshape it into shape `(2, 3, 4)`.
4. What happens if you call `np.reshape(np.arange(10), (3, 4))`? Why?
5. How does `np.newaxis` differ from `np.expand_dims`? When would you prefer one over the other?
6. Write code to create a 5×5 matrix with 1 on the border and 0 inside using slicing and `np.ones`.

---

[Next → Chapter 2: Indexing and Slicing](02-indexing-and-slicing.md)
