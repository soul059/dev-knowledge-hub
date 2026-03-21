# Chapter 3: Mini-Batch Gradient Descent

> **Unit 6 · Optimization · Module 3 of 7**

---

## 3.1 Overview

Mini-batch gradient descent is the **practical sweet spot** between batch GD and stochastic GD. It computes the gradient on a small random subset (mini-batch) of the training data at each step. This is the default training method for virtually all modern deep learning systems.

---

## 3.2 The Compromise

```
  ◄──────────────────────────────────────────────────────►
  Batch GD                Mini-Batch GD                SGD
  (all m samples)         (B samples)            (1 sample)

  ✅ Stable gradient       ✅ Balanced                ✅ Fast updates
  ❌ Slow per step         ✅ GPU-friendly            ❌ Very noisy
  ❌ No GPU benefit        ✅ Some noise (good!)      ❌ No vectorization
```

---

## 3.3 Mini-Batch Update Rule

At each step, sample a mini-batch `B = {(x¹,y¹), ..., (xᴮ,yᴮ)}` of size B:

```
g_B = (1/B) Σⱼ₌₁ᴮ ∇θ L(f(xʲ; θ), yʲ)

θ_(t+1) = θ_t − α · g_B
```

### Algorithm

```
1. Initialize θ
2. For each epoch:
     a. Shuffle training data
     b. Partition into mini-batches of size B
     c. For each mini-batch B_k:
          i.   g = (1/B) Σ_{j ∈ B_k} ∇θ L(xʲ, yʲ; θ)
          ii.  θ = θ − α · g
3. Repeat until convergence
4. Return θ
```

---

## 3.4 Effect of Batch Size on Convergence

### Gradient Variance vs Batch Size

```
  Gradient
  Variance
     |
     | ·
     |  ·
     |    ·
     |       ·
     |            ·
     |                  ·  ·  ·  ·  ·  ·  ·  ·
     +——————————————————————————————————————→ Batch Size
     1    8   32  64  128  256  512  1024

  Variance ∝ σ² / B   (decreases as 1/B)
```

### Loss Curves by Batch Size

```
  Loss
   |
   | \  /\                           B = 1 (SGD) — very noisy
   |  \/  \  /\ /\
   |       \/   \_______________
   |
   | \                               B = 32 — moderate noise
   |   \_  _
   |     \/ \___________________
   |
   | \                               B = m (Batch GD) — smooth
   |   \____
   |        \___________________
   +———————————————————————————→ Iterations
```

---

## 3.5 Batch Size Selection

### Common Batch Sizes

| Batch Size | Characteristics |
|-----------|-----------------|
| **1** | SGD — maximum noise, no vectorization |
| **16–32** | Good for small models, high noise (regularizing) |
| **64–128** | Most common default, good balance |
| **256** | Common for vision tasks, moderate noise |
| **512–1024** | Large-scale training, requires LR scaling |
| **8192+** | Distributed training, needs warmup + LR scaling |

### Rules of Thumb

1. **Start with 32 or 64** — works well for most problems
2. **Batch size should be a power of 2** — optimizes GPU memory alignment
3. **Larger batch → scale learning rate**: `α_new = α_base × (B_new / B_base)`
4. **Larger batch = less noise** → may need explicit regularization
5. **Memory constraint**: `max_batch_size = GPU_memory / (model_size + activations_per_sample)`

---

## 3.6 Vectorized Computation Benefits

Mini-batches exploit **hardware parallelism** via matrix operations:

```
Single sample:     y_hat = x · θ           → O(n)  sequential

Mini-batch (B):    Y_hat = X_B · θ         → O(B·n)  but parallelized on GPU
                   (B×n) · (n×1) = (B×1)
```

### GPU Utilization vs Batch Size

```
  GPU Util (%)
  100 |                    ·  ·  ·  ·  ·  ·  ·
      |                 ·
      |              ·
      |          ·
      |       ·
      |    ·
   0  | ·
      +—————————————————————————————————————→ Batch Size
        1    8   32  64 128 256 512 1024

  Sweet spot: GPU saturates around B = 64–256 for most architectures
```

---

## 3.7 Step-by-Step Example

**Data**: 8 samples for `y = 2x + 1`

| i | x | y |
|---|---|---|
| 1 | 0.5 | 2.0 |
| 2 | 1.0 | 3.0 |
| 3 | 1.5 | 4.0 |
| 4 | 2.0 | 5.0 |
| 5 | 2.5 | 6.0 |
| 6 | 3.0 | 7.0 |
| 7 | 3.5 | 8.0 |
| 8 | 4.0 | 9.0 |

**Settings**: B = 4, α = 0.01, θ = [0, 0]

**Epoch 1, Mini-batch 1** (samples 1–4 after shuffle):

```
X_B = [[1, 0.5], [1, 1.0], [1, 1.5], [1, 2.0]]
y_B = [2.0, 3.0, 4.0, 5.0]

Predictions: [0, 0, 0, 0]
Errors: [-2.0, -3.0, -4.0, -5.0]

∂J/∂θ₀ = mean(errors) = -3.50
∂J/∂θ₁ = mean(errors · x) = -5.375

θ₀ = 0 - 0.01 × (-3.50)  = 0.0350
θ₁ = 0 - 0.01 × (-5.375) = 0.0538
```

**Epoch 1, Mini-batch 2** (samples 5–8): continues similarly...

---

## 3.8 Python Implementation

```python
import numpy as np

def mini_batch_gd(X, y, batch_size=32, lr=0.01, n_epochs=100):
    """
    Mini-batch gradient descent for linear regression.
    X: (m, n) feature matrix with bias
    y: (m,) target vector
    """
    m, n = X.shape
    theta = np.zeros(n)
    history = []

    for epoch in range(n_epochs):
        # Shuffle
        indices = np.random.permutation(m)
        X_shuf = X[indices]
        y_shuf = y[indices]

        # Process mini-batches
        for start in range(0, m, batch_size):
            end = min(start + batch_size, m)
            X_batch = X_shuf[start:end]
            y_batch = y_shuf[start:end]
            B = len(y_batch)

            # Gradient on mini-batch
            y_hat = X_batch @ theta
            gradient = (1 / B) * (X_batch.T @ (y_hat - y_batch))

            # Update
            theta = theta - lr * gradient

        # Track epoch cost
        cost = (1 / (2 * m)) * np.sum((X @ theta - y) ** 2)
        history.append(cost)

    return theta, history


# Example
np.random.seed(42)
X_raw = np.linspace(0.5, 4.0, 100)
y = 2 * X_raw + 1 + np.random.randn(100) * 0.3
X = np.column_stack([np.ones(100), X_raw])

theta, hist = mini_batch_gd(X, y, batch_size=32, lr=0.01, n_epochs=200)
print(f"θ₀ = {theta[0]:.4f}, θ₁ = {theta[1]:.4f}")
# Expected: θ₀ ≈ 1.0, θ₁ ≈ 2.0
```

### Using PyTorch DataLoader

```python
import torch
from torch.utils.data import DataLoader, TensorDataset

X_tensor = torch.FloatTensor(X)
y_tensor = torch.FloatTensor(y)
dataset = TensorDataset(X_tensor, y_tensor)

# DataLoader handles shuffling and batching automatically
loader = DataLoader(dataset, batch_size=32, shuffle=True)

model = torch.nn.Linear(2, 1, bias=False)
optimizer = torch.optim.SGD(model.parameters(), lr=0.01)

for epoch in range(100):
    for X_batch, y_batch in loader:
        y_hat = model(X_batch).squeeze()
        loss = torch.nn.functional.mse_loss(y_hat, y_batch)
        optimizer.zero_grad()
        loss.backward()
        optimizer.step()
```

---

## 3.9 Comparison Table: All Three GD Variants

| Property | Batch GD | Mini-Batch GD | SGD |
|----------|----------|---------------|-----|
| **Samples per update** | All m | B (e.g., 32) | 1 |
| **Updates per epoch** | 1 | ⌈m/B⌉ | m |
| **Gradient noise** | None | Low–Moderate | High |
| **Convergence path** | Smooth | Slightly noisy | Very noisy |
| **GPU utilization** | Good | Excellent | Poor |
| **Memory** | High | Moderate | Low |
| **Convergence speed** | Slow (few updates) | Fast | Fast (but noisy) |
| **Learning rate** | Can be larger | Moderate | Must be smaller |
| **Parallelism** | Full batch | Vectorized mini-batch | No parallelism |
| **Default in practice** | Rarely used | ✅ Standard | Rarely pure SGD |

---

## 3.10 Real-World ML Applications

| Application | Typical Batch Size | Why |
|------------|-------------------|-----|
| **Image Classification (ResNet)** | 256 | Large models, GPU memory limit |
| **NLP (BERT fine-tuning)** | 16–32 | Long sequences consume memory |
| **GPT training** | 512–2048 (effective) | Gradient accumulation across GPUs |
| **Object Detection (YOLO)** | 64 | High-resolution images |
| **Recommendation Systems** | 128–512 | Embedding tables are memory-heavy |

---

## 3.11 Practical Recommendations

1. **Default batch size**: Start with **32** (good for most tasks)
2. **Scale LR with batch size**: Linear scaling rule `α ∝ B`
3. **Use gradient accumulation** if batch doesn't fit in memory
4. **Monitor training loss** per iteration (not just per epoch)
5. **Last mini-batch** may be smaller — handle with `drop_last=True` or accept variable size
6. **Shuffle every epoch** — critical for unbiased estimates

---

## 3.12 Summary Table

| Concept | Detail |
|---------|--------|
| Update rule | `θ = θ − α · (1/B) Σ ∇L` on mini-batch |
| Typical batch sizes | 32, 64, 128, 256 |
| Key advantage | GPU-friendly + noise for regularization |
| Gradient variance | `σ²/B` — controllable via B |
| Industry standard | ✅ This is the default for deep learning |
| Key hyperparameter | Batch size B (affects LR choice) |

---

## 3.13 Quick Revision Questions

1. **Why is mini-batch GD preferred over pure SGD in practice?**
   → It leverages GPU parallelism via matrix operations while still providing regularizing noise.

2. **How does gradient variance relate to batch size?**
   → Variance decreases as O(1/B) — doubling batch size halves the variance.

3. **Why should batch sizes be powers of 2?**
   → It aligns with GPU memory architecture, maximizing hardware utilization.

4. **What is the linear scaling rule for learning rate?**
   → When you increase batch size by k, multiply the learning rate by k.

5. **What is gradient accumulation?**
   → Accumulating gradients over multiple forward passes before updating, simulating a larger batch size when memory is limited.

6. **How many parameter updates occur per epoch with m=10,000 and B=100?**
   → 10,000 / 100 = 100 updates per epoch.

---

| [← Previous: Stochastic Gradient Descent](02-stochastic-gradient-descent.md) | [Next Chapter: Momentum →](04-momentum.md) |
|:---|---:|
