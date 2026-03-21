# Chapter 6: Random Number Generation

> **Unit 2 — NumPy** | Generate random data for simulations, sampling, initialization, and reproducibility in ML workflows.

---

## 6.1 Setting the Seed (Legacy API)

```python
import numpy as np

# Legacy global seed — ensures reproducibility
np.random.seed(42)
print(np.random.rand(3))   # [0.3745 0.9507 0.7320] — same every run
```

> **Note:** The legacy `np.random.seed()` sets a *global* state. For better practice, use the new Generator API (§6.7).

---

## 6.2 Basic Random Functions (Legacy API)

### Uniform [0, 1)

```python
np.random.rand(3)          # 1-D: 3 values in [0, 1)
np.random.rand(2, 4)       # 2-D: shape (2, 4)
```

### Standard Normal (mean=0, std=1)

```python
np.random.randn(5)         # 5 samples from N(0, 1)
np.random.randn(3, 3)      # shape (3, 3)
```

### Random Integers

```python
np.random.randint(0, 10)           # single int in [0, 10)
np.random.randint(0, 10, size=5)   # array of 5 ints in [0, 10)
np.random.randint(1, 7, size=(3, 3))  # 3×3 dice rolls
```

### Random Floats in Custom Range

```python
# Uniform in [a, b): a + (b - a) * rand()
low, high = 5.0, 15.0
samples = low + (high - low) * np.random.rand(100)
```

---

## 6.3 Random Choice and Shuffling

```python
labels = np.array(['cat', 'dog', 'bird', 'fish'])

# Random choice — with or without replacement
np.random.choice(labels, size=3)                          # 3 random picks
np.random.choice(labels, size=3, replace=False)           # no repeats
np.random.choice(labels, size=3, p=[0.5, 0.3, 0.1, 0.1]) # weighted

# Shuffle in-place
deck = np.arange(52)
np.random.shuffle(deck)     # modifies deck

# Permutation — returns a new shuffled copy (original unchanged)
perm = np.random.permutation(52)
```

---

## 6.4 Common Probability Distributions

```python
# Uniform distribution — U(low, high)
np.random.uniform(low=2.0, high=8.0, size=5)

# Normal (Gaussian) — N(mean, std)
np.random.normal(loc=100, scale=15, size=1000)   # IQ scores

# Binomial — B(n, p)
np.random.binomial(n=10, p=0.5, size=5)  # coin flips: successes in 10 trials

# Poisson — P(λ)
np.random.poisson(lam=5, size=10)  # events per interval

# Exponential — Exp(1/β)
np.random.exponential(scale=2.0, size=10)  # time between events

# Beta distribution — commonly used for probability estimates
np.random.beta(a=2, b=5, size=10)

# Multivariate Normal — for correlated features
mean = [0, 0]
cov = [[1, 0.8],
       [0.8, 1]]
samples = np.random.multivariate_normal(mean, cov, size=100)  # (100, 2)
```

---

## 6.5 Distribution Comparison Table

| Distribution | Function | Use Case |
|-------------|----------|----------|
| Uniform | `np.random.uniform(a, b)` | Random initialization, jittering |
| Normal | `np.random.normal(μ, σ)` | Weight initialization, noise |
| Binomial | `np.random.binomial(n, p)` | Success/failure trials |
| Poisson | `np.random.poisson(λ)` | Count data (clicks, arrivals) |
| Exponential | `np.random.exponential(β)` | Wait times |
| Beta | `np.random.beta(α, β)` | Probabilities, Bayesian priors |
| Multivariate Normal | `np.random.multivariate_normal` | Correlated features, GMMs |

---

## 6.6 Reproducibility Deep Dive

```python
# Method 1: Global seed (legacy) — simple but has pitfalls
np.random.seed(0)
a = np.random.rand(3)

np.random.seed(0)          # reset
b = np.random.rand(3)
print(np.array_equal(a, b))  # True

# Pitfall: any other library or function call that uses np.random
# will advance the global state unpredictably
```

---

## 6.7 New Generator API (Recommended since NumPy 1.17)

The new API uses explicit `Generator` objects — no global state, better statistical properties.

```python
# Create a generator with a seed
rng = np.random.default_rng(seed=42)

# Same functions, cleaner interface
rng.random(5)                      # uniform [0, 1), replaces rand()
rng.standard_normal((3, 3))        # N(0,1), replaces randn()
rng.integers(0, 10, size=5)        # replaces randint()
rng.choice(labels, size=3)         # same as before
rng.shuffle(deck)                  # in-place shuffle
rng.permutation(52)                # permuted copy

# Distributions
rng.normal(loc=0, scale=1, size=100)
rng.uniform(low=0, high=1, size=100)
rng.binomial(n=10, p=0.5, size=100)
rng.poisson(lam=5, size=100)
```

### Why Prefer the New API?

| Feature | Legacy (`np.random`) | New (`default_rng`) |
|---------|---------------------|---------------------|
| State | Global (shared) | Per-generator (isolated) |
| Algorithm | Mersenne Twister | PCG64 (faster, better stats) |
| Thread safety | No | Yes |
| Reproducibility | Fragile | Robust |
| Bit generation | 32-bit | 64-bit |

---

## 6.8 ML Application: Train/Test Split

```python
rng = np.random.default_rng(seed=42)

X = np.arange(100).reshape(20, 5)   # 20 samples, 5 features
y = np.random.randint(0, 2, 20)     # binary labels

# Shuffle indices
indices = rng.permutation(len(X))

# 80/20 split
split = int(0.8 * len(X))
train_idx, test_idx = indices[:split], indices[split:]

X_train, X_test = X[train_idx], X[test_idx]
y_train, y_test = y[train_idx], y[test_idx]

print(f"Train: {X_train.shape}, Test: {X_test.shape}")
# Train: (16, 5), Test: (4, 5)
```

---

## 6.9 ML Application: Bootstrap Sampling

```python
rng = np.random.default_rng(seed=0)

data = np.array([12, 15, 14, 10, 13, 11, 16, 14, 12, 15])

n_bootstrap = 1000
bootstrap_means = np.empty(n_bootstrap)

for i in range(n_bootstrap):
    sample = rng.choice(data, size=len(data), replace=True)  # sample WITH replacement
    bootstrap_means[i] = sample.mean()

print(f"Mean estimate: {bootstrap_means.mean():.2f}")
print(f"95% CI: [{np.percentile(bootstrap_means, 2.5):.2f}, "
      f"{np.percentile(bootstrap_means, 97.5):.2f}]")
```

---

## 6.10 ML Application: Random Weight Initialization

```python
rng = np.random.default_rng(seed=42)

# Xavier/Glorot initialization for a layer with fan_in=128, fan_out=64
fan_in, fan_out = 128, 64
limit = np.sqrt(6 / (fan_in + fan_out))
W = rng.uniform(-limit, limit, size=(fan_in, fan_out))

# He initialization (for ReLU networks)
std = np.sqrt(2 / fan_in)
W_he = rng.normal(0, std, size=(fan_in, fan_out))

print(f"Xavier W range: [{W.min():.4f}, {W.max():.4f}]")
print(f"He W std: {W_he.std():.4f} (target: {std:.4f})")
```

---

## 6.11 Generating Reproducible Random Streams

```python
# Spawn independent generators from a parent (for parallel work)
parent_rng = np.random.default_rng(seed=42)
child_rngs = parent_rng.spawn(4)   # 4 independent generators

for i, child in enumerate(child_rngs):
    print(f"Child {i}: {child.random(3)}")
# Each child produces a unique, reproducible stream
```

---

## Best Practices

| Practice | Why |
|----------|-----|
| Use `default_rng(seed)` over `np.random.seed()` | Isolated state, better algorithm |
| Always set a seed in experiments | Reproducibility for debugging and papers |
| Use `rng.spawn()` for parallel streams | Guarantees statistical independence |
| Use `replace=False` for sampling without replacement | Prevents duplicates in train/test splits |
| Use `rng.permutation` over `rng.shuffle` | Doesn't modify original array |

---

## Summary Table

| Legacy API | New API (`rng = default_rng()`) | Purpose |
|------------|--------------------------------|---------|
| `np.random.rand()` | `rng.random()` | Uniform [0, 1) |
| `np.random.randn()` | `rng.standard_normal()` | Standard normal |
| `np.random.randint()` | `rng.integers()` | Random integers |
| `np.random.choice()` | `rng.choice()` | Random selection |
| `np.random.shuffle()` | `rng.shuffle()` | In-place shuffle |
| `np.random.permutation()` | `rng.permutation()` | Shuffled copy |
| `np.random.normal()` | `rng.normal()` | Gaussian samples |
| `np.random.seed(n)` | `default_rng(n)` | Set seed |

---

## Revision Questions

1. What is the difference between `np.random.seed(42)` and `np.random.default_rng(42)`?
2. How do you generate 100 samples from a normal distribution with mean 50 and std 10?
3. Explain the difference between `shuffle` and `permutation`.
4. Write code to perform stratified sampling: randomly select 5 items from each of 3 categories.
5. Why is Xavier initialization preferred over simple `randn` for neural network weights?
6. How would you create 4 independent random streams for parallel processing?

---

[← Chapter 5: Linear Algebra](05-linear-algebra-operations.md) | [Next → Chapter 7: Performance Optimization](07-performance-optimization.md)
