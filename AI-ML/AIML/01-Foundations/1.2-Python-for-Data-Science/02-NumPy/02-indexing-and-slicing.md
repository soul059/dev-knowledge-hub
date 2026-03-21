# Chapter 2: Indexing and Slicing

> **Unit 2 — NumPy** | Access, filter, and modify array elements with basic, boolean, and fancy indexing.

---

## 2.1 Basic Indexing (1-D)

```python
import numpy as np

a = np.array([10, 20, 30, 40, 50])

a[0]      # 10   — first element
a[-1]     # 50   — last element
a[-2]     # 40   — second-to-last
```

---

## 2.2 Slicing — `start:stop:step`

```python
a = np.arange(10)   # [0 1 2 3 4 5 6 7 8 9]

a[2:7]       # [2 3 4 5 6]      — stop is exclusive
a[:5]        # [0 1 2 3 4]      — from beginning
a[5:]        # [5 6 7 8 9]      — to end
a[::2]       # [0 2 4 6 8]      — every 2nd element
a[::-1]      # [9 8 7 6 5 4 3 2 1 0]  — reversed
a[1:8:3]     # [1 4 7]          — start=1, step=3
```

> **Key:** Slicing returns a **view** — modifications propagate back to the original.

```python
s = a[3:6]
s[0] = 99
print(a[3])   # 99  — original changed!
```

---

## 2.3 Multi-Dimensional Indexing

```python
m = np.array([[1, 2, 3],
              [4, 5, 6],
              [7, 8, 9]])

m[0, 2]       # 3       — row 0, col 2
m[1]          # [4 5 6] — entire row 1
m[:, 1]       # [2 5 8] — entire col 1

# Slicing sub-matrix
m[0:2, 1:3]   # [[2 3]
               #  [5 6]]

# Mix of integer and slice — reduces dimensionality
m[1, :]        # shape (3,) — 1-D
m[1:2, :]      # shape (1, 3) — still 2-D
```

---

## 2.4 Negative Indexing in N-D

```python
m = np.arange(20).reshape(4, 5)

m[-1]          # last row
m[:, -1]       # last column
m[-2:, -3:]    # bottom-right 2×3 submatrix
```

---

## 2.5 Boolean Indexing (Masking)

Boolean indexing selects elements where the mask is `True`. It always returns a **copy**.

```python
a = np.array([15, 22, 8, 31, 5, 19])

# Create boolean mask
mask = a > 10
print(mask)    # [True True False True False True]

# Apply mask
a[mask]        # [15 22 31 19]

# Inline — most common style
a[a > 10]      # same result

# Compound conditions (use & | ~, NOT and/or/not)
a[(a > 10) & (a < 25)]   # [15 22 19]
a[~(a > 10)]              # [8 5]  — negation
```

### Modifying via Boolean Mask

```python
scores = np.array([45, 78, 92, 38, 61])
scores[scores < 50] = 50   # clip minimum to 50
print(scores)               # [50 78 92 50 61]
```

---

## 2.6 Fancy Indexing (Integer Array Indexing)

Pass an array of indices to select specific elements in any order. Returns a **copy**.

```python
a = np.array([10, 20, 30, 40, 50])

idx = np.array([0, 3, 4])
a[idx]                     # [10 40 50]

# Duplicate indices are allowed
a[np.array([1, 1, 3, 3])] # [20 20 40 40]
```

### Fancy Indexing in 2-D

```python
m = np.arange(12).reshape(3, 4)
# [[0  1  2  3]
#  [4  5  6  7]
#  [8  9 10 11]]

# Select specific elements: (0,1), (1,2), (2,3)
m[np.array([0, 1, 2]), np.array([1, 2, 3])]   # [1 6 11]

# Select specific rows
m[np.array([0, 2])]         # rows 0 and 2 → shape (2, 4)
```

---

## 2.7 `np.where()` — Conditional Selection

```python
a = np.array([1, -2, 3, -4, 5])

# Indices where condition is True
np.where(a > 0)             # (array([0, 2, 4]),)

# Ternary-style: where(condition, x, y)
np.where(a > 0, a, 0)       # [1 0 3 0 5]  — replace negatives with 0

# Practical: ReLU activation
def relu(x):
    return np.where(x > 0, x, 0)

relu(np.array([-3, -1, 0, 2, 5]))   # [0 0 0 2 5]
```

---

## 2.8 `np.argmax`, `np.argmin`, `np.nonzero`

```python
a = np.array([3, 1, 4, 1, 5, 9, 2, 6])

np.argmax(a)          # 5  — index of max value (9)
np.argmin(a)          # 1  — index of min value (1)

# Along an axis in 2-D
m = np.array([[1, 7, 3],
              [4, 2, 8]])
np.argmax(m, axis=0)  # [1 0 1] — row index of max in each column
np.argmax(m, axis=1)  # [1 2]   — col index of max in each row

# Indices of non-zero elements
x = np.array([0, 5, 0, 3, 0])
np.nonzero(x)         # (array([1, 3]),)
```

---

## 2.9 Combining Index Types

```python
m = np.arange(20).reshape(4, 5)

# Boolean rows + slice columns
row_mask = np.array([True, False, True, False])
m[row_mask, 1:4]      # rows 0 & 2, columns 1-3

# Fancy rows + single column
m[np.array([0, 3]), 2]  # elements (0,2) and (3,2)
```

---

## 2.10 Modifying Arrays via Indexing

```python
a = np.zeros(5)

# Scalar assignment
a[2] = 10

# Slice assignment
a[1:4] = [7, 8, 9]        # [0. 7. 8. 9. 0.]

# Fancy index assignment — beware of repeated indices
a = np.zeros(5)
idx = np.array([1, 1, 3])
a[idx] += 1
print(a)                   # [0. 1. 0. 1. 0.] — NOT [0. 2. 0. 1. 0.]!

# Use np.add.at for repeated-index accumulation
a = np.zeros(5)
np.add.at(a, idx, 1)
print(a)                   # [0. 2. 0. 1. 0.] — correct
```

---

## Best Practices

| Practice | Why |
|----------|-----|
| Prefer boolean masks over loops | Vectorized and 10-100× faster |
| Use parentheses around compound conditions | `&` / `|` have higher precedence than `>` / `<` |
| Remember: slicing → view, fancy → copy | Prevents subtle mutation bugs |
| Use `np.where` for conditional assignment | Cleaner and faster than loops |
| Use `np.add.at` for repeated-index updates | `+=` with duplicates doesn't accumulate |

---

## Summary Table

| Technique | Syntax | Returns |
|-----------|--------|---------|
| Basic indexing | `a[i]`, `a[i, j]` | scalar / view |
| Slicing | `a[start:stop:step]` | view |
| Boolean indexing | `a[mask]` | copy |
| Fancy indexing | `a[idx_array]` | copy |
| `np.where(cond)` | indices of True | tuple of arrays |
| `np.where(cond, x, y)` | ternary select | new array |
| `np.argmax/argmin` | index of extremum | int / array |
| `np.nonzero` | indices of non-zeros | tuple of arrays |

---

## Revision Questions

1. What is the output of `np.arange(10)[7:2:-2]`?
2. Why must you use `&` instead of `and` when combining boolean masks?
3. Given a 2-D array `m`, write one expression to extract all rows where the value in column 0 exceeds 5.
4. Explain why `a[[1,1,1]] += 1` only increments `a[1]` by 1 instead of 3. How do you fix it?
5. What is the difference between `m[1, :]` and `m[1:2, :]` in terms of shape?
6. Write a one-liner using `np.where` to replace all NaN values in an array with 0.

---

[← Chapter 1: Array Creation](01-array-creation-and-manipulation.md) | [Next → Chapter 3: Broadcasting](03-broadcasting.md)
