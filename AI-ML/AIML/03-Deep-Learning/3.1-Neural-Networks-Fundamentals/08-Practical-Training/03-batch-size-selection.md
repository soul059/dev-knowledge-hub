# 3. Batch Size Selection

> **Unit 8 · Practical Training** — Choosing the right batch size for convergence, generalization, and GPU efficiency

---

## Chapter Overview

Batch size is one of the most under-appreciated hyperparameters in deep learning. It affects **convergence speed**, **generalization ability**, **GPU utilization**, and **memory consumption** — all at the same time. Recent research has revealed a deep connection between batch size and the geometry of the loss landscape: small batches find **flat minima** that generalize better, while large batches tend toward **sharp minima** that don't. This chapter covers the theory, the practical tradeoffs, gradient accumulation for effective large batches, and clear recommendations based on current best practices.

---

## 1. What is Batch Size?

### Mini-Batch Gradient Descent

```
  Full Dataset: N = 60,000 samples
  
  ┌──────────────────────────────────────────────────────────────┐
  │  Batch GD (batch_size = N = 60,000)                         │
  │  • 1 update per epoch                                        │
  │  • Exact gradient (no noise)                                 │
  │  • Very slow, huge memory                                    │
  │                                                              │
  │  Mini-Batch GD (batch_size = 128)                            │
  │  • 469 updates per epoch (60000/128)                         │
  │  • Approximate gradient (noisy — this is useful!)            │
  │  • Standard approach in deep learning                        │
  │                                                              │
  │  Stochastic GD (batch_size = 1)                              │
  │  • 60,000 updates per epoch                                  │
  │  • Very noisy gradient                                       │
  │  • Poor GPU utilization                                      │
  └──────────────────────────────────────────────────────────────┘
  
  Gradient estimate quality:
  
  g_B = (1/B) Σᵢ ∇L(xᵢ, θ)     ← average over B samples
  
  Var(g_B) = Var(g₁) / B         ← variance decreases with batch size
  
  Large B → low variance (accurate gradient) but fewer updates per epoch
  Small B → high variance (noisy gradient) but many updates per epoch
```

---

## 2. Effect on Convergence and Generalization

### Sharp vs Flat Minima

```
  THE KEY INSIGHT (Keskar et al., 2017):
  
  Large Batch → Sharp Minimum          Small Batch → Flat Minimum
  
  Loss ↑                               Loss ↑
       │     ╱╲                              │     ╱─────╲
       │    ╱  ╲                             │    ╱       ╲
       │   ╱    ╲                            │   ╱         ╲
       │  ╱      ╲                           │  ╱           ╲
       │ ╱        ╲                          │ ╱             ╲
       │╱    ●     ╲                         │╱      ●       ╲
       └──────────────→                      └──────────────────→
            Train                                  Train
            minimum                                minimum
  
  Why sharp minima generalize WORSE:
  
  Loss ↑                               Loss ↑
       │     ╱╲                              │     ╱─────╲
       │    ╱  ╲                             │    ╱   ····╲····  test
       │   ╱ ···╲···· test loss             │   ╱   ·     ╲  ·  loss
       │  ╱ ·    ╲  · (shifted)             │  ╱   ·       ╲ ·  (shifted)
       │ ╱ ·      ╲·                        │ ╱   ·         ╲·
       │╱  · ●     ╲                        │╱    ·   ●      ╲
       └──────────────→                     └──────────────────→
       
       Test loss at ● is MUCH               Test loss at ● is CLOSE
       higher than train loss!              to train loss!
       → Poor generalization               → Good generalization
  
  Small distribution shift between train and test data
  causes large loss increase in sharp minima.
```

### Why Small Batches Find Flat Minima

```
  Small batch gradient = True gradient + NOISE
  
  g_B = g_true + ε      where ε ~ N(0, σ²/B)
  
  ┌──────────────────────────────────────────────────────────────┐
  │  The noise acts as IMPLICIT REGULARIZATION:                  │
  │                                                              │
  │  • In sharp minima: noise pushes parameters OUT              │
  │    (the well is narrow, noise escapes easily)                │
  │                                                              │
  │  • In flat minima: noise keeps parameters IN                 │
  │    (the basin is wide, noise can't escape)                   │
  │                                                              │
  │  → SGD with small batches naturally selects flat minima!     │
  └──────────────────────────────────────────────────────────────┘
  
  Analogy: Ball on a landscape
  
  Sharp valley:          Flat valley:
  ╱╲                     ╱────╲
  │●│ ← ball bounces    │ ● │ ← ball stays
  │ │   out easily!     │    │   comfortably
  ╲╱                     ╲────╱
```

---

## 3. The Linear Scaling Rule

When increasing batch size, you should increase the learning rate proportionally.

```
  LINEAR SCALING RULE (Goyal et al., 2017 — Facebook/Meta):
  
  If batch size B₁ uses learning rate α₁,
  then batch size B₂ should use learning rate:
  
  α₂ = α₁ × (B₂ / B₁)
  
  Example:
  B₁ = 128,  α₁ = 0.01
  B₂ = 1024, α₂ = 0.01 × (1024/128) = 0.08
  
  WHY? Each step processes B samples. If you use 8× more
  samples per step, you get 8× fewer steps per epoch.
  To make the same total "progress" per epoch, each step
  needs to be 8× larger → 8× higher LR.
  
  
  ⚠ IMPORTANT: This rule requires LR WARMUP!
  
  LR ↑
     │         ╱───────────── target LR = 0.08
     │        ╱
     │       ╱
     │      ╱    warmup for first 5 epochs
     │     ╱
     │    ╱
     │   ╱
     │──╱  start at α₁ = 0.01
     └──────────────────────────────→ Epochs
        0  1  2  3  4  5  6  7  ...
  
  Without warmup, the large LR causes training to diverge
  because early gradients are large and noisy.
```

---

## 4. GPU Utilization

### Batch Size and Throughput

```
  Throughput (samples/sec) vs Batch Size
  
  Throughput ↑
             │
   10000     │                     ●────●────●
             │                   ╱
    8000     │                 ╱         Saturated
             │               ╱           (GPU fully utilized)
    6000     │             ╱
             │           ╱   Sweet spot
    4000     │         ╱     (good utilization)
             │       ╱
    2000     │     ╱
             │   ╱   Underutilized
    1000     │ ╱     (GPU cores idle)
             │╱
             └────┬────┬────┬────┬────┬────┬────→ Batch Size
                  1    8   32  128  512 2048 8192
  
  ┌──────────────────────────────────────────────────────────────┐
  │  GPU parallelism means:                                      │
  │  • Batch size 1 → 1 sample at a time (1% GPU usage)        │
  │  • Batch size 32 → 32 samples in parallel (~50% GPU usage) │
  │  • Batch size 256 → near-full GPU utilization               │
  │  • Batch size > GPU memory → OUT OF MEMORY ERROR!           │
  └──────────────────────────────────────────────────────────────┘
  
  Training time per EPOCH:
  
  Time ↑
       │ ●
       │  ●
       │   ●●
       │     ●●●
       │        ●●●●
       │            ●●●●●●●●●●──────  ← diminishing returns
       └────────────────────────────→ Batch Size
         1   8  32 128 512 2048
  
  Bigger batch → fewer gradient updates → faster epochs
  BUT also fewer updates → may need more epochs to converge!
```

### Memory Consumption

```
  GPU Memory Usage = Model + Activations + Gradients + Optimizer States
  
  ┌─────────────────────────────────────────────────────────┐
  │  Component          │ Scales with Batch Size?           │
  ├─────────────────────┼───────────────────────────────────┤
  │  Model weights      │ NO (fixed)                        │
  │  Activations        │ YES (linear with batch size!)    │
  │  Gradients          │ NO (same size as weights)         │
  │  Optimizer states   │ NO (Adam: 2× weight size)        │
  └─────────────────────┴───────────────────────────────────┘
  
  Example: ResNet-50 on RTX 3090 (24 GB)
  
  Batch Size    Memory      Fits?
  ─────────────────────────────────
  8             3.5 GB      ✓
  32            8.2 GB      ✓
  64            14.1 GB     ✓
  128           25.8 GB     ✗ OOM!
  
  → Maximum batch size is limited by GPU memory
```

---

## 5. Gradient Accumulation

**Problem:** You want a large effective batch size but don't have enough GPU memory.

**Solution:** Accumulate gradients over multiple small batches before updating.

```
  Gradient Accumulation: Simulate large batch with small GPU memory
  
  Effective batch size = micro_batch_size × accumulation_steps
  
  Example: Want batch_size=256 but GPU only fits 64
  → Use micro_batch=64, accumulate=4
  
  ┌──────────────────────────────────────────────────────────────┐
  │  Step 1: Forward + backward on micro-batch 1 (64 samples)  │
  │          gradients accumulate in .grad                       │
  │                                                              │
  │  Step 2: Forward + backward on micro-batch 2 (64 samples)  │
  │          gradients ADD to existing .grad                     │
  │                                                              │
  │  Step 3: Forward + backward on micro-batch 3 (64 samples)  │
  │          gradients ADD to existing .grad                     │
  │                                                              │
  │  Step 4: Forward + backward on micro-batch 4 (64 samples)  │
  │          gradients ADD to existing .grad                     │
  │                                                              │
  │  Step 5: optimizer.step()  ← NOW update weights!            │
  │          optimizer.zero_grad()                               │
  │                                                              │
  │  Total: 4 × 64 = 256 samples contributed to this update    │
  │  Memory used: only 64 samples worth!                        │
  └──────────────────────────────────────────────────────────────┘
```

### PyTorch Implementation

```python
import torch
import torch.nn as nn
import torch.optim as optim

model = MyModel().to(device)
optimizer = optim.AdamW(model.parameters(), lr=3e-4)
criterion = nn.CrossEntropyLoss()

# Configuration
micro_batch_size = 64       # what fits in GPU memory
accumulation_steps = 4      # number of micro-batches to accumulate
effective_batch_size = micro_batch_size * accumulation_steps  # = 256

train_loader = DataLoader(dataset, batch_size=micro_batch_size, shuffle=True)

for epoch in range(num_epochs):
    model.train()
    optimizer.zero_grad()  # zero gradients at the start
    
    for batch_idx, (X, y) in enumerate(train_loader):
        X, y = X.to(device), y.to(device)
        
        # Forward pass
        outputs = model(X)
        loss = criterion(outputs, y)
        
        # Scale loss by accumulation steps
        # (so accumulated gradient = average over effective batch)
        loss = loss / accumulation_steps
        
        # Backward pass — gradients ACCUMULATE in .grad
        loss.backward()
        
        # Update weights every accumulation_steps mini-batches
        if (batch_idx + 1) % accumulation_steps == 0:
            # Optional: gradient clipping
            torch.nn.utils.clip_grad_norm_(model.parameters(), max_norm=1.0)
            
            optimizer.step()
            optimizer.zero_grad()
    
    # Handle remaining batches that don't fill a full accumulation cycle
    if (batch_idx + 1) % accumulation_steps != 0:
        optimizer.step()
        optimizer.zero_grad()
```

### Gradient Accumulation with LR Scheduler

```python
# When using a scheduler, step it based on OPTIMIZER steps, not batches
scheduler = optim.lr_scheduler.CosineAnnealingLR(
    optimizer, 
    T_max=len(train_loader) // accumulation_steps * num_epochs
)

for epoch in range(num_epochs):
    optimizer.zero_grad()
    for batch_idx, (X, y) in enumerate(train_loader):
        X, y = X.to(device), y.to(device)
        loss = criterion(model(X), y) / accumulation_steps
        loss.backward()
        
        if (batch_idx + 1) % accumulation_steps == 0:
            optimizer.step()
            scheduler.step()  # step scheduler on optimizer steps
            optimizer.zero_grad()
```

---

## 6. Research Findings

### Key Papers and Their Findings

```
  ┌────────────────────────────────────────────────────────────────────┐
  │  PAPER                              KEY FINDING                   │
  ├────────────────────────────────────────────────────────────────────┤
  │  Keskar et al. (2017)              Large batch → sharp minima     │
  │  "On Large-Batch Training"         → worse generalization         │
  │                                                                    │
  │  Goyal et al. (2017)               Linear scaling rule + warmup   │
  │  "Accurate, Large Minibatch SGD"   → batch 8192 matches batch 256│
  │                                                                    │
  │  Smith & Le (2018)                  Increasing batch size ≈        │
  │  "Don't Decay the LR,              decaying learning rate!        │
  │   Increase the Batch Size"         (same noise-to-signal ratio)   │
  │                                                                    │
  │  McCandlish et al. (2018)          "Critical batch size" beyond   │
  │  "An Empirical Model of            which larger batches give      │
  │   Large-Batch Training"            diminishing returns            │
  │                                                                    │
  │  Hoffer et al. (2017)              Large batch CAN generalize     │
  │  "Train Longer, Generalize         well if trained long enough    │
  │   Better"                          (+ BN and LR scaling)          │
  └────────────────────────────────────────────────────────────────────┘
```

### The Noise Scale Perspective

```
  Smith & Le (2018) showed that what matters is the 
  NOISE SCALE of SGD:
  
  Noise Scale ∝ (ε · N) / B
  
  where:
    ε = learning rate
    N = dataset size
    B = batch size
  
  ┌──────────────────────────────────────────────────────────────┐
  │  EQUIVALENT OPERATIONS (same noise scale):                   │
  │                                                              │
  │  1. Decay LR by 2×     ↔  Increase batch size by 2×        │
  │  2. LR = 0.01, B = 128 ↔  LR = 0.02, B = 256              │
  │                                                              │
  │  Implication: Instead of LR decay schedules,                │
  │  you can INCREASE batch size during training!               │
  │                                                              │
  │  Advantage: Larger batches → better GPU utilization         │
  │  → faster wall-clock time per sample                        │
  └──────────────────────────────────────────────────────────────┘
```

### Critical Batch Size

```
  Critical batch size = the batch size beyond which
  you get diminishing returns from parallelism.
  
  Compute    │
  Efficiency │●●●●●●
  (samples   │      ●●●
   per GPU   │         ●●●
   second)   │            ●●●●●●●●●●●── diminishing returns
             │
             └────────────────────────────→ Batch Size
                    ↑
              Critical batch size (B_crit)
  
  B_crit depends on:
  • Model size (larger models → larger B_crit)
  • Dataset size
  • Task difficulty
  
  Typical B_crit values:
  • MNIST: ~512
  • CIFAR-10: ~1024
  • ImageNet: ~8192
  • GPT-3: ~3.2 million tokens!
```

---

## 7. Practical Recommendations

```
  ┌──────────────────────────────────────────────────────────────────┐
  │  PRACTICAL BATCH SIZE RECOMMENDATIONS                           │
  ├──────────────────────────────────────────────────────────────────┤
  │                                                                  │
  │  1. START with batch_size = 32 or 64                            │
  │     → Good default, well-studied, usually generalizes well      │
  │                                                                  │
  │  2. INCREASE if training is too slow                            │
  │     → 128, 256 — check if generalization holds                  │
  │     → Apply linear scaling rule for LR                          │
  │                                                                  │
  │  3. Use LARGEST batch that fits in GPU memory                   │
  │     → Only if you apply proper LR scaling + warmup              │
  │     → Use gradient accumulation if needed                       │
  │                                                                  │
  │  4. For BEST GENERALIZATION                                     │
  │     → Batch size 32-128 with SGD + momentum                    │
  │     → Or small batch + one-cycle policy                         │
  │                                                                  │
  │  5. For FASTEST TRAINING TIME                                   │
  │     → Largest batch with linear LR scaling + warmup             │
  │     → Use mixed precision to fit larger batches                 │
  │                                                                  │
  │  6. Powers of 2 for GPU efficiency                              │
  │     → 32, 64, 128, 256, 512, 1024                              │
  │     → GPU cores work best with sizes that are multiples of 32   │
  │                                                                  │
  │  7. Task-specific guidance:                                     │
  │     • Classification: 32-256                                    │
  │     • Object detection: 2-16 (large images)                     │
  │     • NLP / Transformers: 16-64 (long sequences)               │
  │     • GANs: 32-128                                              │
  │     • RL: task-dependent                                        │
  └──────────────────────────────────────────────────────────────────┘
```

### Decision Flowchart

```
  START: Choose batch size
  │
  ├─ Q: Does your model fit in GPU memory with batch_size=32?
  │  ├─ NO → Reduce model size, use gradient checkpointing,
  │  │       or use mixed precision
  │  └─ YES ↓
  │
  ├─ Q: Is training TOO SLOW?
  │  ├─ NO → Use batch_size=32 (good generalization)
  │  └─ YES ↓
  │
  ├─ Try batch_size=128 with LR × 4
  │  Add LR warmup for 5 epochs
  │
  ├─ Q: Does generalization match batch_size=32?
  │  ├─ YES → Use batch_size=128 (faster!)
  │  └─ NO → Stick with batch_size=32
  │          or try batch_size=64 as compromise
  │
  └─ Q: Need EVEN FASTER?
     ├─ Try batch_size=256 or 512 with linear LR scaling
     ├─ Use gradient accumulation if GPU memory limited
     └─ Use mixed precision (torch.cuda.amp)
```

---

## 8. Complete Example: Comparing Batch Sizes

```python
import torch
import torch.nn as nn
import torch.optim as optim
from torch.utils.data import DataLoader
from torchvision import datasets, transforms
import time

# ── Model ──
def create_model():
    return nn.Sequential(
        nn.Flatten(),
        nn.Linear(784, 256), nn.ReLU(), nn.Dropout(0.2),
        nn.Linear(256, 128), nn.ReLU(), nn.Dropout(0.2),
        nn.Linear(128, 10)
    )

# ── Data ──
transform = transforms.Compose([
    transforms.ToTensor(),
    transforms.Normalize((0.1307,), (0.3081,))
])
train_data = datasets.MNIST('./data', train=True, download=True, transform=transform)
test_data = datasets.MNIST('./data', train=False, transform=transform)
test_loader = DataLoader(test_data, batch_size=1024)

def train_with_batch_size(batch_size, base_lr=0.01, base_batch=32, epochs=10):
    """Train with given batch size using linear scaling rule."""
    torch.manual_seed(42)
    model = create_model()
    
    # Linear scaling rule
    lr = base_lr * (batch_size / base_batch)
    
    optimizer = optim.SGD(model.parameters(), lr=lr, momentum=0.9)
    criterion = nn.CrossEntropyLoss()
    train_loader = DataLoader(train_data, batch_size=batch_size, shuffle=True)
    
    # LR warmup for large batches
    warmup_epochs = 5 if batch_size > 128 else 0
    
    start_time = time.time()
    history = {'train_loss': [], 'test_acc': []}
    
    for epoch in range(epochs):
        # Warmup
        if epoch < warmup_epochs:
            warmup_lr = base_lr + (lr - base_lr) * (epoch / warmup_epochs)
            for pg in optimizer.param_groups:
                pg['lr'] = warmup_lr
        
        model.train()
        total_loss = 0
        for X, y in train_loader:
            optimizer.zero_grad()
            loss = criterion(model(X.view(-1, 784)), y)
            loss.backward()
            optimizer.step()
            total_loss += loss.item()
        
        # Evaluate
        model.eval()
        correct = 0
        with torch.no_grad():
            for X, y in test_loader:
                correct += (model(X.view(-1, 784)).argmax(1) == y).sum().item()
        
        test_acc = correct / len(test_data)
        avg_loss = total_loss / len(train_loader)
        history['train_loss'].append(avg_loss)
        history['test_acc'].append(test_acc)
    
    elapsed = time.time() - start_time
    print(f"  Batch {batch_size:4d} | LR {lr:.4f} | "
          f"Final acc {test_acc:.4f} | Time {elapsed:.1f}s")
    return history

# ── Compare batch sizes ──
print("Batch Size Comparison (10 epochs):")
print("=" * 60)
for bs in [32, 64, 128, 256, 512]:
    train_with_batch_size(bs)
```

Expected output pattern:
```
  Batch Size Comparison (10 epochs):
  ============================================================
    Batch   32 | LR 0.0100 | Final acc 0.9782 | Time 45.2s
    Batch   64 | LR 0.0200 | Final acc 0.9778 | Time 28.1s
    Batch  128 | LR 0.0400 | Final acc 0.9765 | Time 18.3s
    Batch  256 | LR 0.0800 | Final acc 0.9741 | Time 13.7s
    Batch  512 | LR 0.1600 | Final acc 0.9698 | Time 11.2s
    
  Observation: Smaller batches generalize slightly better,
  but larger batches train faster. The sweet spot is often
  batch_size=64 or 128 for a good speed/accuracy tradeoff.
```

---

## 9. Summary Table

| Batch Size | Gradient Noise | Generalization | GPU Utilization | Training Speed | Memory |
|:----------:|:--------------:|:--------------:|:---------------:|:--------------:|:------:|
| **1** | Very high | Unstable | Very poor | Slowest | Minimal |
| **32** | High | Best | Low-medium | Slow | Low |
| **64** | Medium-high | Very good | Medium | Medium | Medium |
| **128** | Medium | Good | Good | Fast | Medium |
| **256** | Low-medium | Good (with LR scaling) | Very good | Very fast | High |
| **512+** | Low | Needs careful tuning | Near-max | Fastest | Very high |

---

## 10. Revision Questions

1. **Explain the connection between batch size and the sharpness of the loss minimum.** Why do small batches tend to find flat minima? Use the noise/regularization argument.

2. **State the linear scaling rule and explain why it is necessary when increasing batch size.** Why does it also require learning rate warmup?

3. **What is gradient accumulation and when would you use it?** Write the key lines of PyTorch code that implement it, and explain why the loss must be divided by the number of accumulation steps.

4. **Describe the "noise scale" perspective from Smith & Le (2018).** How does it show that increasing batch size and decaying the learning rate are equivalent?

5. **What is the "critical batch size" and how does it determine the optimal batch size for a given task?** How does it differ between MNIST and ImageNet?

6. **A researcher has a model that fits batch_size=32 on a single GPU. They want to simulate batch_size=512 using gradient accumulation. Describe the exact procedure, including any learning rate adjustments needed.**

---

| [← 02 Learning Rate Finder](02-learning-rate-finder.md) | [Unit 8 Home](README.md) | [04 → Training Monitoring](04-training-monitoring.md) |
|:---|:---:|---:|
