# 📦 Batch Processing

> **Chapter 2.5 — Single Sample, Mini-Batch, and Full-Batch Training**

| Topic | Details |
|---|---|
| **Subject** | Forward Propagation — Batch Processing |
| **Prerequisites** | Chapter 2.4 (Matrix Notation) |
| **Key Insight** | Mini-batch processing balances the noise of single-sample updates with the memory cost of full-batch computation |

---

## 📌 Overview

Real neural network training never processes the entire dataset at once (too much memory) nor one sample at a time (too slow). Instead, we divide the dataset into **mini-batches** — groups of samples processed together. Understanding batch processing is critical for practical deep learning: it affects training speed, convergence, generalization, and GPU utilization.

---

## 1. Three Processing Modes

### 1.1 Comparison

```
    ┌───────────────────────────────────────────────────────────────────┐
    │                  TRAINING DATA (m samples)                        │
    │  ┌──┬──┬──┬──┬──┬──┬──┬──┬──┬──┬──┬──┬──┬──┬──┬──┬──┬──┬──┬──┐ │
    │  │x₁│x₂│x₃│x₄│x₅│x₆│x₇│x₈│x₉│..│..│..│..│..│..│..│..│..│..│xₘ│ │
    │  └──┴──┴──┴──┴──┴──┴──┴──┴──┴──┴──┴──┴──┴──┴──┴──┴──┴──┴──┴──┘ │
    │                                                                   │
    │  ┌─────────────────── STOCHASTIC (SGD) ──────────────────────┐   │
    │  │  Process ONE sample at a time                              │   │
    │  │  |x₁| → update, |x₂| → update, |x₃| → update, ...       │   │
    │  │  m updates per epoch                                       │   │
    │  └───────────────────────────────────────────────────────────┘   │
    │                                                                   │
    │  ┌─────────────── MINI-BATCH (most common) ─────────────────┐   │
    │  │  Process B samples at a time                               │   │
    │  │  |x₁..xB| → update, |xB+1..x2B| → update, ...           │   │
    │  │  ⌈m/B⌉ updates per epoch                                  │   │
    │  └───────────────────────────────────────────────────────────┘   │
    │                                                                   │
    │  ┌────────────────── BATCH (full dataset) ──────────────────┐   │
    │  │  Process ALL m samples at once                             │   │
    │  │  |x₁, x₂, x₃, ..., xₘ| → one update                     │   │
    │  │  1 update per epoch                                        │   │
    │  └───────────────────────────────────────────────────────────┘   │
    └───────────────────────────────────────────────────────────────────┘
```

### 1.2 Detailed Comparison

| Aspect | Stochastic (B=1) | Mini-Batch (B=32-512) | Full Batch (B=m) |
|---|---|---|---|
| **Batch size** | 1 | 32, 64, 128, 256, 512 | m (entire dataset) |
| **Updates/epoch** | m | ⌈m/B⌉ | 1 |
| **Gradient estimate** | Very noisy | Moderate noise | Exact gradient |
| **Convergence** | Oscillates heavily | Smooth but with noise | Smoothest |
| **Speed per update** | Fast (tiny computation) | Medium | Slow (huge matrix) |
| **GPU utilization** | Poor (underutilized) | Good (parallel) | Best (if fits in memory) |
| **Memory** | Minimal | Moderate | Maximum (may not fit!) |
| **Generalization** | Good (noise = regularization) | Best balance | May overfit |
| **In practice** | Rarely used alone | **Standard approach** | Only for tiny datasets |

### 1.3 Gradient Noise Visualization

```
    Loss vs. Training Progress

    Stochastic (B=1):               Mini-Batch (B=64):           Full Batch:
    Loss                            Loss                         Loss
    │╲ ╱╲   ╱╲                      │╲                            │╲
    │ ╳  ╲╱╱  ╲╱╲                   │ ╲╱╲                         │ ╲
    │╱ ╲    ╲   ╲╱╲                 │   ╲╲╱╲                      │  ╲
    │   ╲╱╲  ╲   ╲                  │    ╲  ╲                     │   ╲
    │    ╲  ╲  ╲╱  ╲                │     ╲╱─╲                    │    ╲
    │     ╲  ╲╱     ╲─              │       ╲──╲──               │     ╲───────
    ┼───────────────── iter         ┼───────────────── iter       ┼──────────── iter

    Very noisy              Moderate noise             Smooth but slow
    but explores well       BEST TRADEOFF              and may overfit
```

---

## 2. Constructing Mini-Batches

### 2.1 The Process

```
    MINI-BATCH CONSTRUCTION (each epoch):
    
    Step 1: Shuffle the dataset randomly
    ┌──┬──┬──┬──┬──┬──┬──┬──┬──┬──┐
    │x₃│x₇│x₁│x₉│x₅│x₂│x₈│x₄│x₆│x₁₀│
    └──┴──┴──┴──┴──┴──┴──┴──┴──┴──┘
    
    Step 2: Divide into mini-batches of size B
    (B = 3 in this example)
    
    ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌────┐
    │x₃│x₇│x₁ │  │x₉│x₅│x₂ │  │x₈│x₄│x₆ │  │x₁₀│
    └──────────┘  └──────────┘  └──────────┘  └────┘
     Mini-batch 1  Mini-batch 2  Mini-batch 3   MB 4
                                               (smaller!)
    
    Step 3: For each mini-batch:
    - Forward pass
    - Compute loss
    - Backward pass
    - Update parameters
    
    Step 4: Repeat (new shuffle each epoch)
```

### 2.2 Why Shuffle?

```
Without shuffling:
    Epoch 1: [x₁,x₂,x₃] → [x₄,x₅,x₆] → [x₇,x₈,x₉] → [x₁₀]
    Epoch 2: [x₁,x₂,x₃] → [x₄,x₅,x₆] → [x₇,x₈,x₉] → [x₁₀]  ← SAME!
    
    Problem: Network sees same groupings every epoch → biased gradients
    
With shuffling:
    Epoch 1: [x₃,x₇,x₁] → [x₉,x₅,x₂] → [x₈,x₄,x₆] → [x₁₀]
    Epoch 2: [x₅,x₁,x₈] → [x₃,x₁₀,x₆] → [x₂,x₉,x₇] → [x₄]  ← DIFFERENT!
    
    Better: Different groupings each epoch → reduces gradient bias
```

---

## 3. Mini-Batch Forward Pass

### 3.1 Batched Matrix Operations

```
    For a mini-batch X_batch of size B:
    
    X_batch = [x⁽ⁱ⁾, x⁽ⁱ⁺¹⁾, ..., x⁽ⁱ⁺ᴮ⁻¹⁾]
    Shape: (n_features, B)
    
    Forward pass is identical to full-batch, just with B columns instead of m:
    
    Z^[1] = W^[1] · X_batch + b^[1]     Shape: (n¹, B)
    A^[1] = g^[1](Z^[1])                Shape: (n¹, B)
    Z^[2] = W^[2] · A^[1] + b^[2]       Shape: (n², B)
    A^[2] = g^[2](Z^[2])                Shape: (n², B)
    ...
    ŷ = A^[L]                            Shape: (n^L, B)
    
    The weight matrices W and biases b are the SAME for all batches!
    Only X changes between mini-batches.
```

---

## 4. Memory Considerations

### 4.1 What Consumes GPU Memory

```
    ┌──────────────────────────────────────────────────────┐
    │             GPU MEMORY BREAKDOWN                      │
    ├──────────────────────────────────────────────────────┤
    │                                                      │
    │  1. Model Parameters (W, b)         ~ Fixed          │
    │     • Stored once, shared across batches             │
    │     • Example: ResNet-50 = 25M params = 100 MB       │
    │                                                      │
    │  2. Activations (Z, A cache)        ~ Scales with B  │
    │     • Stored for EACH layer, EACH sample in batch    │
    │     • Example: 256 layers × 1024 neurons × B         │
    │     • This is usually the LARGEST consumer!          │
    │                                                      │
    │  3. Gradients (∂L/∂W, ∂L/∂b)       ~ Same as params │
    │     • Same size as parameters                        │
    │                                                      │
    │  4. Optimizer State                  ~ 1-2× params   │
    │     • Adam: stores m and v for each param            │
    │     • SGD+momentum: stores velocity                  │
    │                                                      │
    │  Memory ∝ Batch Size (due to activation caching)     │
    └──────────────────────────────────────────────────────┘
```

### 4.2 Batch Size and Memory

```
    Batch Size     Activation Memory     GPU Memory
    ─────────────────────────────────────────────────
    B = 1          ~50 MB                ~300 MB
    B = 16         ~800 MB               ~1.1 GB
    B = 32         ~1.6 GB               ~1.9 GB
    B = 64         ~3.2 GB               ~3.5 GB
    B = 128        ~6.4 GB               ~6.7 GB
    B = 256        ~12.8 GB              ~13.1 GB
    B = 512        ~25.6 GB              ~25.9 GB  ← might not fit!
    
    (Approximate values for a medium-sized CNN on ImageNet)
    
    Rule of thumb: If you get CUDA Out of Memory, reduce batch size!
```

---

## 5. Choosing Batch Size

### 5.1 Practical Guidelines

```
    ┌──────────────────────────────────────────────────────────┐
    │              BATCH SIZE SELECTION GUIDE                    │
    ├──────────────────────────────────────────────────────────┤
    │                                                          │
    │  START with B = 32 or 64 (good default)                  │
    │                                                          │
    │  INCREASE batch size if:                                 │
    │  • Training is too slow (more parallelism)               │
    │  • Loss is too noisy                                     │
    │  • You have GPU memory available                         │
    │  • Using batch normalization (needs B ≥ 16)              │
    │                                                          │
    │  DECREASE batch size if:                                 │
    │  • Out of GPU memory                                     │
    │  • Want better generalization (more noise)               │
    │  • Training plateaus (noise helps escape local minima)   │
    │  • Using very large models (memory constrained)          │
    │                                                          │
    │  USE powers of 2: 16, 32, 64, 128, 256, 512             │
    │  (GPU hardware is optimized for these sizes)             │
    │                                                          │
    └──────────────────────────────────────────────────────────┘
```

### 5.2 Batch Size vs. Learning Rate

```
    Important relationship: Larger batch → Use larger learning rate

    B = 32,  lr = 0.001    (baseline)
    B = 64,  lr = 0.002    (double both)
    B = 128, lr = 0.004    (linear scaling rule)
    B = 256, lr = 0.008

    Linear Scaling Rule (Goyal et al., 2017):
    If you multiply batch size by k, multiply learning rate by k.
    
    With warmup: Gradually increase lr over first few epochs
    to avoid divergence with large batches.
```

---

## 6. PyTorch DataLoader — The Standard Tool

```python
import torch
from torch.utils.data import DataLoader, TensorDataset
import numpy as np

# ──── Create synthetic dataset ────
np.random.seed(42)
n_samples = 1000
n_features = 20
n_classes = 5

X = np.random.randn(n_samples, n_features).astype(np.float32)
y = np.random.randint(0, n_classes, n_samples)

# Convert to PyTorch tensors
X_tensor = torch.from_numpy(X)
y_tensor = torch.from_numpy(y).long()

# Create Dataset and DataLoader
dataset = TensorDataset(X_tensor, y_tensor)

# ──── Different batch configurations ────
configs = {
    'Stochastic (B=1)':   DataLoader(dataset, batch_size=1, shuffle=True),
    'Small batch (B=16)': DataLoader(dataset, batch_size=16, shuffle=True),
    'Medium (B=64)':      DataLoader(dataset, batch_size=64, shuffle=True),
    'Large (B=256)':      DataLoader(dataset, batch_size=256, shuffle=True),
    'Full batch':         DataLoader(dataset, batch_size=len(dataset), shuffle=False),
}

for name, loader in configs.items():
    n_batches = len(loader)
    last_batch_size = len(dataset) % loader.batch_size or loader.batch_size
    print(f"{name:25s} │ Batches/epoch: {n_batches:5d} │ Last batch: {last_batch_size}")

# ──── Training loop skeleton ────
import torch.nn as nn

model = nn.Sequential(
    nn.Linear(n_features, 64),
    nn.ReLU(),
    nn.Linear(64, 32),
    nn.ReLU(),
    nn.Linear(32, n_classes)
)
optimizer = torch.optim.Adam(model.parameters(), lr=0.001)
criterion = nn.CrossEntropyLoss()

train_loader = DataLoader(dataset, batch_size=64, shuffle=True, drop_last=False)

# One epoch of training
model.train()
epoch_loss = 0
n_correct = 0
n_total = 0

for batch_idx, (X_batch, y_batch) in enumerate(train_loader):
    # Forward pass
    logits = model(X_batch)
    loss = criterion(logits, y_batch)
    
    # Backward pass
    optimizer.zero_grad()
    loss.backward()
    optimizer.step()
    
    # Track metrics
    epoch_loss += loss.item() * len(X_batch)
    predictions = torch.argmax(logits, dim=1)
    n_correct += (predictions == y_batch).sum().item()
    n_total += len(y_batch)

avg_loss = epoch_loss / n_total
accuracy = n_correct / n_total
print(f"\nEpoch summary: Loss={avg_loss:.4f}, Accuracy={accuracy:.2%}")
print(f"Batches processed: {batch_idx + 1}")
```

**Expected Output:**
```
Stochastic (B=1)          │ Batches/epoch:  1000 │ Last batch: 1
Small batch (B=16)        │ Batches/epoch:    63 │ Last batch: 8
Medium (B=64)             │ Batches/epoch:    16 │ Last batch: 40
Large (B=256)             │ Batches/epoch:     4 │ Last batch: 232
Full batch                │ Batches/epoch:     1 │ Last batch: 1000

Epoch summary: Loss=1.6384, Accuracy=19.70%
Batches processed: 16
```

---

## 7. Advanced Batch Techniques

### 7.1 Gradient Accumulation

When the batch size you want doesn't fit in GPU memory:

```python
# Effective batch size = 256, but GPU can only handle 64
accumulation_steps = 4  # 64 × 4 = 256 effective batch size

optimizer.zero_grad()
for i, (X_batch, y_batch) in enumerate(train_loader):
    logits = model(X_batch)                     # batch_size = 64
    loss = criterion(logits, y_batch) / accumulation_steps
    loss.backward()                              # Accumulate gradients
    
    if (i + 1) % accumulation_steps == 0:
        optimizer.step()                         # Update with accumulated gradient
        optimizer.zero_grad()                    # Reset gradients
```

### 7.2 `drop_last` Option

```python
# With 1000 samples and batch_size=64:
# 1000 / 64 = 15 full batches + 1 batch of 40 samples

DataLoader(dataset, batch_size=64, drop_last=False)  # 16 batches (last has 40)
DataLoader(dataset, batch_size=64, drop_last=True)   # 15 batches (drop the 40)

# drop_last=True is useful for:
# • Batch Normalization (needs consistent batch size)
# • When last small batch causes unstable gradients
```

---

## 📊 Summary Table

| Concept | Key Point |
|---|---|
| **Stochastic (B=1)** | Very noisy gradients, acts as regularization, rarely used alone |
| **Mini-batch** | Standard approach; B = 32-512; balances speed and noise |
| **Full batch** | Exact gradients but slow, memory-heavy, may overfit |
| **Shuffling** | Shuffle data each epoch to avoid biased gradients |
| **Memory** | Activations scale linearly with batch size — main memory consumer |
| **GPU efficiency** | Use powers of 2 for batch size (hardware optimization) |
| **Learning rate** | Scale linearly with batch size (linear scaling rule) |
| **Gradient accumulation** | Simulate large batches when GPU memory is limited |
| **DataLoader** | PyTorch utility for automatic batching, shuffling, multi-process loading |
| **drop_last** | Optionally drop the last incomplete mini-batch |

---

## ❓ Revision Questions

1. **Compare stochastic, mini-batch, and full-batch gradient descent. When would you prefer each? What are the tradeoffs?**

2. **Why is shuffling the data before each epoch important? What problem can arise without shuffling?**

3. **You have 50,000 training samples and a batch size of 128. How many mini-batches are in one epoch? What is the size of the last mini-batch?**

4. **Explain why batch size affects GPU memory usage. Which component of the training process consumes the most memory?**

5. **What is gradient accumulation? When would you use it, and how does it achieve a larger effective batch size?**

6. **Your model trains well with batch_size=32 and lr=0.001. You want to increase to batch_size=256 for speed. What learning rate should you use, and what precaution should you take?**

---

## 🧭 Navigation

| Previous | Up | Next |
|---|---|---|
| [← Matrix Notation](04-matrix-notation.md) | [🏠 Neural Networks Fundamentals](../README.md) | [Regression Losses →](../03-Loss-Functions/01-regression-losses.md) |

---

*© 2024 AIML Study Notes. Built for deep understanding, not shallow memorization.*
