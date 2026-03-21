# Chapter 3: Broadcasting

> **Unit 2 — NumPy** | Understand how NumPy automatically expands dimensions so arrays of different shapes can work together without explicit loops or copies.

---

## 3.1 What Is Broadcasting?

Broadcasting is the mechanism by which NumPy performs element-wise operations on arrays of **different shapes** by virtually stretching the smaller array to match the larger one — *without copying data*.

```python
import numpy as np

a = np.array([1, 2, 3])   # shape (3,)
b = 10                      # scalar — shape ()

a + b   # [11 12 13]  — scalar is "broadcast" across all elements
```

---

## 3.2 Broadcasting Rules (Step by Step)

When operating on two arrays, NumPy compares shapes **right to left**:

| Rule | Description |
|------|-------------|
| **1. Pad with 1s** | If arrays differ in `ndim`, prepend 1s to the smaller shape |
| **2. Stretch** | Dimensions of size 1 are stretched to match the other array |
| **3. Error** | If sizes differ and *neither* is 1 → **incompatible** |

### Example walkthrough

```
A shape: (4, 3)
B shape:    (3,)

Step 1 — pad B:   (1, 3)
Step 2 — stretch: (4, 3)  ← B is virtually repeated 4 times along axis 0
Result shape:     (4, 3)  ✓
```

---

## 3.3 Compatible Shape Examples

```python
# 2-D + 1-D
m = np.ones((3, 4))       # (3, 4)
v = np.array([1, 2, 3, 4])# (4,) → (1, 4) → (3, 4)
print((m + v).shape)       # (3, 4) ✓

# 2-D + column vector
col = np.array([[10], [20], [30]])  # (3, 1)
print((m + col).shape)              # (3, 4) ✓

# 3-D + 2-D
a = np.ones((2, 3, 4))    # (2, 3, 4)
b = np.ones((3, 4))       # (3, 4) → (1, 3, 4) → (2, 3, 4)
print((a + b).shape)       # (2, 3, 4) ✓

# 3-D + 1-D
c = np.ones((4,))         # (4,) → (1, 1, 4) → (2, 3, 4)
print((a + c).shape)       # (2, 3, 4) ✓
```

---

## 3.4 Broadcasting Errors

```python
a = np.ones((3, 4))
b = np.ones((3,))

# a.shape = (3, 4)
# b.shape =    (3,)  → (1, 3)
# Rightmost: 4 vs 3 → NEITHER is 1 → ERROR

try:
    a + b
except ValueError as e:
    print(e)
# "operands could not be broadcast together with shapes (3,4) (3,)"

# Fix: make b a column vector
b_col = b[:, np.newaxis]   # (3, 1)
print((a + b_col).shape)   # (3, 4) ✓
```

---

## 3.5 Outer Product via Broadcasting

Broadcasting elegantly computes outer products without `np.outer`:

```python
x = np.array([1, 2, 3])          # (3,)
y = np.array([10, 20, 30, 40])   # (4,)

# Reshape x to column, then multiply
outer = x[:, np.newaxis] * y     # (3,1) * (4,) → (3,1) * (1,4) → (3,4)
print(outer)
# [[10 20 30 40]
#  [20 40 60 80]
#  [30 60 90 120]]
```

---

## 3.6 Common ML Patterns Using Broadcasting

### Subtracting the Mean (Centering Data)

```python
# X: (n_samples, n_features)
X = np.array([[1.0, 200],
              [2.0, 400],
              [3.0, 600]])

mean = X.mean(axis=0)        # shape (2,) — one mean per feature
X_centered = X - mean        # (3,2) - (2,) → broadcasts along axis 0
print(X_centered)
# [[-1. -200.]
#  [ 0.    0.]
#  [ 1.  200.]]
```

### Z-Score Normalization

```python
std = X.std(axis=0)                # (2,)
X_normalized = (X - mean) / std    # element-wise, fully broadcast
```

### Min-Max Scaling

```python
X_min = X.min(axis=0)
X_max = X.max(axis=0)
X_scaled = (X - X_min) / (X_max - X_min)   # all values in [0, 1]
```

### Adding Bias to All Samples

```python
weights = np.random.randn(5)       # (5,)
bias = 0.1                         # scalar
output = X @ weights[:2] + bias    # (3,) + scalar → broadcasts
```

---

## 3.7 Pairwise Distance Matrix (Advanced)

Compute Euclidean distances between every pair of points:

```python
# points: (N, D)
points = np.array([[0, 0],
                    [1, 0],
                    [0, 1]])

# Expand dims so subtraction broadcasts to (N, N, D)
diff = points[:, np.newaxis, :] - points[np.newaxis, :, :]  # (3,1,2) - (1,3,2)
dist = np.sqrt((diff ** 2).sum(axis=-1))                     # (3, 3)
print(np.round(dist, 2))
# [[0.   1.   1.  ]
#  [1.   0.   1.41]
#  [1.   1.41 0.  ]]
```

---

## 3.8 Performance Benefits

Broadcasting avoids:
- **Explicit loops** — operations stay in optimized C/Fortran code.
- **Data copies** — the stretched dimension is *virtual*; no extra memory is allocated.

```python
# Slow: explicit tiling then adding
tiled = np.tile(v, (3, 1))       # actually allocates (3, 4) memory
result_slow = m + tiled

# Fast: broadcasting — same result, zero extra memory
result_fast = m + v              # ← preferred
```

> **Rule:** If you find yourself calling `np.tile` or `np.repeat` just to match shapes before an operation, broadcasting can almost always replace it.

---

## 3.9 Quick Shape Compatibility Checker

```python
def broadcastable(shape_a, shape_b):
    """Return the result shape or raise ValueError."""
    return np.broadcast_shapes(shape_a, shape_b)

print(broadcastable((3, 4), (4,)))     # (3, 4)
print(broadcastable((2, 1), (1, 5)))   # (2, 5)
# broadcastable((3, 4), (3,))          # ValueError
```

---

## Best Practices

| Practice | Why |
|----------|-----|
| Keep trailing dimensions aligned | Broadcasting matches right to left |
| Use `[:, np.newaxis]` to add axes | Explicitly control which dimension stretches |
| Avoid `np.tile` / `np.repeat` for shape matching | Broadcasting is free in memory |
| Print `.shape` during debugging | Most broadcasting bugs are shape mismatches |
| Use `np.broadcast_shapes()` to verify | Catches errors before computation |

---

## Summary Table

| Shape A | Shape B | Result | Valid? |
|---------|---------|--------|--------|
| `(3, 4)` | `(4,)` | `(3, 4)` | ✓ |
| `(3, 4)` | `(3, 1)` | `(3, 4)` | ✓ |
| `(2, 3, 4)` | `(3, 4)` | `(2, 3, 4)` | ✓ |
| `(5, 1, 3)` | `(1, 4, 3)` | `(5, 4, 3)` | ✓ |
| `(3, 4)` | `(3,)` | — | ✗ |
| `(2, 3)` | `(2, 4)` | — | ✗ |

---

## Revision Questions

1. State the three broadcasting rules in your own words.
2. What shapes are compatible: `(5, 3)` and `(1, 3)`? What is the result shape?
3. Why does `(4, 3) + (4,)` fail? How would you fix it?
4. Write a broadcasting expression to compute the multiplication table from 1 to 10 (a 10×10 matrix).
5. Explain why broadcasting is more memory-efficient than `np.tile`.
6. Given `X` of shape `(100, 5)`, write one line to subtract the column means and divide by column standard deviations.

---

[← Chapter 2: Indexing and Slicing](02-indexing-and-slicing.md) | [Next → Chapter 4: Vectorized Operations](04-vectorized-operations.md)
