# Chapter 7: Performance Optimization

> **Unit 2 — NumPy** | Write faster code with vectorization, memory-aware layouts, einsum, and profiling techniques.

---

## 7.1 Vectorization vs Loops

The single most impactful optimization: replace Python loops with NumPy operations.

```python
import numpy as np
import time

n = 1_000_000
a = np.random.rand(n)

# ❌ Python loop
start = time.perf_counter()
result = np.empty(n)
for i in range(n):
    result[i] = a[i] ** 2 + 2 * a[i] + 1
loop_time = time.perf_counter() - start

# ✅ Vectorized
start = time.perf_counter()
result_vec = a ** 2 + 2 * a + 1
vec_time = time.perf_counter() - start

print(f"Loop: {loop_time:.3f}s | Vectorized: {vec_time:.5f}s | Speedup: {loop_time/vec_time:.0f}x")
# Typical: ~200-500x speedup
```

### Why the Speedup?

| Factor | Python Loop | Vectorized |
|--------|-------------|------------|
| Type dispatch | Every element | Once per array |
| Memory layout | Scattered PyObjects | Contiguous C array |
| CPU optimization | Interpreter overhead | SIMD instructions |
| Cache efficiency | Poor | Excellent |

---

## 7.2 Memory Layout: C Order vs Fortran Order

NumPy arrays can be stored **row-major (C)** or **column-major (Fortran)**.

```python
a = np.arange(12).reshape(3, 4)           # default: C order (row-major)
b = np.asfortranarray(a)                    # Fortran order (column-major)

print(a.flags['C_CONTIGUOUS'])   # True
print(b.flags['F_CONTIGUOUS'])   # True

# Access pattern matters for performance:
# C order  → iterating over columns within a row is fast (a[i, :])
# F order  → iterating over rows within a column is fast (a[:, j])
```

### Performance Impact

```python
n = 5000
C_arr = np.ones((n, n), order='C')
F_arr = np.ones((n, n), order='F')

# Row sum — C order wins
start = time.perf_counter()
_ = C_arr.sum(axis=1)
c_time = time.perf_counter() - start

start = time.perf_counter()
_ = F_arr.sum(axis=1)
f_time = time.perf_counter() - start

print(f"Row sum → C: {c_time:.4f}s, F: {f_time:.4f}s")
# C order is typically 2-4x faster for row operations
```

---

## 7.3 Contiguous Arrays and Copies

```python
a = np.arange(20).reshape(4, 5)

# Slicing can create non-contiguous views
col = a[:, 2]                     # every 5th element in memory
print(col.flags['C_CONTIGUOUS'])  # False

# Force contiguous layout for performance-critical operations
col_contig = np.ascontiguousarray(col)
print(col_contig.flags['C_CONTIGUOUS'])  # True

# Check if array shares memory with another
print(np.shares_memory(a, col))          # True (it's a view)
```

---

## 7.4 `np.einsum` — Einstein Summation

`einsum` is a powerful, concise way to express many array operations using index notation.

```python
A = np.random.rand(3, 4)
B = np.random.rand(4, 5)
v = np.random.rand(4)

# Matrix multiplication: C_ij = Σ_k A_ik * B_kj
C = np.einsum('ik,kj->ij', A, B)        # same as A @ B

# Dot product: Σ_i a_i * b_i
a = np.random.rand(5)
b = np.random.rand(5)
np.einsum('i,i->', a, b)                 # same as np.dot(a, b)

# Trace: Σ_i A_ii
M = np.random.rand(4, 4)
np.einsum('ii->', M)                     # same as np.trace(M)

# Transpose
np.einsum('ij->ji', A)                   # same as A.T

# Outer product
np.einsum('i,j->ij', a[:3], b[:4])       # same as np.outer(a[:3], b[:4])

# Batch matrix multiply: for each batch b, C[b] = A[b] @ B[b]
A_batch = np.random.rand(10, 3, 4)
B_batch = np.random.rand(10, 4, 5)
C_batch = np.einsum('bij,bjk->bik', A_batch, B_batch)  # (10, 3, 5)

# Element-wise multiply then sum over axis
np.einsum('ij,ij->i', A, A)             # row-wise sum of squares
```

### When to Use `einsum`

| Use Case | `einsum` | Alternative |
|----------|----------|-------------|
| Complex tensor contractions | ✅ Clear | Multiple reshapes + matmuls |
| Simple matrix multiply | Overkill | `@` is clearer |
| Batch operations | ✅ Elegant | Loops or `np.matmul` with broadcasting |

---

## 7.5 Memory-Mapped Arrays (`np.memmap`)

Work with arrays larger than RAM by mapping them to disk files.

```python
# Create a memory-mapped file
large = np.memmap('data.dat', dtype='float32', mode='w+', shape=(10000, 1000))
large[:] = np.random.rand(10000, 1000).astype('float32')
large.flush()   # write to disk
del large

# Read back — only pages accessed into RAM
mapped = np.memmap('data.dat', dtype='float32', mode='r', shape=(10000, 1000))
print(mapped[0, :5])          # only this row is loaded into memory
print(mapped.mean(axis=0)[:3])  # processes in chunks automatically

# Clean up
import os
del mapped
os.remove('data.dat')
```

> **Use Case:** Processing datasets that exceed available RAM (e.g., large image or genomics datasets).

---

## 7.6 Structured Arrays

Store heterogeneous, column-like data in a single NumPy array.

```python
dt = np.dtype([('name', 'U10'), ('age', 'i4'), ('score', 'f8')])
students = np.array([
    ('Alice', 23, 88.5),
    ('Bob',   21, 92.0),
    ('Carol', 25, 79.3)
], dtype=dt)

# Access by field name
students['name']      # ['Alice' 'Bob' 'Carol']
students['score']     # [88.5 92.  79.3]

# Boolean mask on a field
students[students['age'] > 22]  # Alice and Carol

# Sort by score
np.sort(students, order='score')
```

> **Note:** For most tabular data, Pandas is more ergonomic. Use structured arrays when you need NumPy speed without the Pandas dependency.

---

## 7.7 Avoiding Unnecessary Copies

Every copy doubles memory usage and wastes time. Know what creates copies:

```python
a = np.arange(1_000_000)

# ✅ Views — no copy
b = a[::2]            # slicing
c = a.reshape(1000, 1000)
d = a.ravel()

# ❌ Copies — allocates new memory
e = a.copy()
f = a.flatten()
g = a[np.array([0, 1, 2])]   # fancy indexing
h = a[a > 5]                  # boolean indexing

# Check if it's a view
print(b.base is a)   # True → view
print(e.base is a)   # False → copy
```

### Tips to Minimize Copies

```python
# ❌ Creates intermediate arrays
result = np.sqrt(np.sum(a ** 2))

# ✅ In-place operations where possible
temp = a.copy()        # only if mutation is acceptable
np.square(temp, out=temp)         # square in-place
total = temp.sum()
result = np.sqrt(total)

# Use the `out` parameter for ufuncs
out = np.empty_like(a, dtype='float64')
np.multiply(a, 2, out=out)       # no intermediate allocation
```

---

## 7.8 Profiling with `%timeit`

Use IPython/Jupyter's `%timeit` magic for reliable micro-benchmarks.

```python
# In IPython or Jupyter:
# %timeit np.sum(a)
# %timeit a.sum()
# %timeit np.einsum('i->', a)

# In a script, use timeit module:
import timeit

a = np.random.rand(1_000_000)

t1 = timeit.timeit(lambda: np.sum(a ** 2), number=100)
t2 = timeit.timeit(lambda: np.einsum('i,i->', a, a), number=100)

print(f"sum(a**2):    {t1:.4f}s")
print(f"einsum:       {t2:.4f}s")
# einsum is often faster — avoids creating a**2 intermediate array
```

### Profiling Guidelines

| Tool | Best For |
|------|----------|
| `%timeit` | Micro-benchmarks (single expressions) |
| `cProfile` | Full-program profiling |
| `line_profiler` | Line-by-line analysis |
| `memory_profiler` | Memory usage per line |

---

## 7.9 Numba JIT Compilation

When pure NumPy isn't enough, Numba compiles Python to machine code.

```python
# pip install numba
from numba import njit

@njit
def pairwise_distance_numba(X):
    n = X.shape[0]
    D = np.empty((n, n))
    for i in range(n):
        for j in range(n):
            d = 0.0
            for k in range(X.shape[1]):
                d += (X[i, k] - X[j, k]) ** 2
            D[i, j] = np.sqrt(d)
    return D

X = np.random.rand(500, 3)

# First call: compilation overhead (~1s)
D = pairwise_distance_numba(X)

# Subsequent calls: near-C speed
# %timeit pairwise_distance_numba(X)  # ~10ms vs ~seconds in pure Python
```

### When to Use Numba

| Scenario | Recommendation |
|----------|----------------|
| Vectorizable with NumPy | Use NumPy (simpler) |
| Complex element-wise logic with branches | Use Numba |
| GPU acceleration needed | Use Numba CUDA or CuPy |
| Need compatibility without compilation | Stick with NumPy |

---

## 7.10 When to Use Alternatives

| Tool | Best For |
|------|----------|
| **NumPy** | General array computation on CPU |
| **CuPy** | GPU-accelerated NumPy (drop-in replacement) |
| **Numba** | JIT-compiled loops, custom kernels |
| **JAX** | Auto-differentiation + JIT + GPU/TPU |
| **Dask** | Parallel/distributed arrays larger than RAM |
| **PyTorch/TF** | Deep learning with autograd |

```python
# CuPy example (requires NVIDIA GPU + CUDA)
# import cupy as cp
# a_gpu = cp.array([1, 2, 3])
# b_gpu = cp.array([4, 5, 6])
# c_gpu = a_gpu + b_gpu        # runs on GPU
# c_cpu = cp.asnumpy(c_gpu)    # transfer back to CPU
```

---

## 7.11 Optimization Checklist

```
1. ✅ Replace loops with vectorized operations
2. ✅ Use appropriate dtype (float32 instead of float64 when precision allows)
3. ✅ Avoid unnecessary copies (use views, in-place ops, `out` parameter)
4. ✅ Match memory layout to access pattern (C order for row access)
5. ✅ Use einsum for complex contractions
6. ✅ Profile before optimizing — measure, don't guess
7. ✅ Consider Numba for irreducibly loopy code
8. ✅ Use memmap for datasets larger than RAM
9. ✅ Use float32 for ML training when full precision isn't needed
```

---

## Best Practices

| Practice | Why |
|----------|-----|
| Always vectorize first | Biggest performance gain for least effort |
| Profile before micro-optimizing | Avoid premature optimization |
| Use `out` parameter in ufuncs | Eliminates intermediate array allocation |
| Choose `float32` for large ML datasets | Half the memory, faster computation |
| Prefer `np.einsum` for multi-axis operations | Often faster and avoids temporaries |
| Use `np.ascontiguousarray` before hot loops | Ensures cache-friendly access |

---

## Summary Table

| Technique | Typical Speedup | Complexity |
|-----------|----------------|------------|
| Vectorization | 100-500× | Low |
| Correct memory order | 2-4× | Low |
| `np.einsum` | 1.5-3× | Medium |
| `out` parameter | 1.2-2× (memory savings) | Low |
| `float32` vs `float64` | 1.5-2× | Low |
| Numba `@njit` | 50-200× over Python | Medium |
| CuPy (GPU) | 10-100× over CPU | Medium |
| `np.memmap` | Enables > RAM datasets | Low |

---

## Revision Questions

1. Why is vectorized NumPy code faster than equivalent Python loops? Name at least 3 reasons.
2. What is the difference between C-order and Fortran-order memory layout? When does it matter?
3. Write an `np.einsum` expression for batch matrix multiplication of shapes `(B, M, K)` and `(B, K, N)`.
4. How does `np.memmap` allow processing arrays larger than available RAM?
5. When should you use Numba instead of pure NumPy? Give a concrete example.
6. List 3 ways to avoid unnecessary array copies in NumPy code.

---

[← Chapter 6: Random Number Generation](06-random-number-generation.md)
