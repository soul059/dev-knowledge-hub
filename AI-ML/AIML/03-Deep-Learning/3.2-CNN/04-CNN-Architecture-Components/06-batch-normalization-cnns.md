# 📊 Batch Normalization in CNNs

[← Previous: 05 — Flatten Operation](05-flatten-operation.md) | [📑 Unit Overview](../README.md)

---

## 📋 Chapter Overview

**Batch Normalization (BatchNorm)** is one of the most impactful techniques in deep learning. In CNNs, `BatchNorm2d` normalizes activations **per channel** across the batch and spatial dimensions, stabilizing training, enabling higher learning rates, and acting as a regularizer. Understanding its mechanics — especially the critical difference between training and evaluation modes — is essential for building and deploying CNN models correctly.

**What you'll learn:**
- How BatchNorm2d normalizes per channel across batch and spatial dimensions
- The four learnable/tracked parameters: γ, β, running_mean, running_var
- Training mode vs eval mode behavior (critical for deployment!)
- Where to place BatchNorm in a CNN architecture
- How BatchNorm affects training dynamics
- Complete PyTorch implementation with `nn.BatchNorm2d`

---

## 1️⃣ What Problem Does BatchNorm Solve?

### Internal Covariate Shift

As training progresses, the distribution of each layer's inputs changes because the weights of previous layers are being updated. This **internal covariate shift** forces each layer to continuously adapt to new input distributions, slowing training.

```
Epoch 1:                              Epoch 100:
Layer input distribution              Layer input distribution
shifted and rescaled                  shifted again!

    ┌───────┐                             ┌───────┐
    │  ╱╲   │                             │   ╱╲  │
    │ ╱  ╲  │                             │  ╱  ╲ │
    │╱    ╲ │                             │ ╱    ╲│
    └───────┘                             └───────┘
    mean=2.3, var=5.1                     mean=-1.2, var=12.7

Each layer must constantly re-adapt → slow, unstable training
```

### BatchNorm Solution

```
Before BatchNorm:                     After BatchNorm:
Activations vary wildly              Activations normalized to ~N(0,1)
across training                       then shifted/scaled by learned params

    ╱╲    ╱╲      ╱╲                        ╱╲
   ╱  ╲  ╱  ╲    ╱  ╲                      ╱  ╲
  ╱    ╲╱    ╲  ╱    ╲                     ╱    ╲
 ╱              ╲     ╲                   ╱      ╲
──────────────────────────              ──────────────
mean shifts, var changes              mean≈0, var≈1 (before learned shift)
```

---

## 2️⃣ BatchNorm2d — How It Works

### The Key Insight: Normalize Per Channel

In CNNs, each channel of a feature map represents a **different learned feature** (e.g., edge detector, texture detector). BatchNorm2d computes statistics **per channel** across all spatial positions and all samples in the batch.

```
Input tensor: (N, C, H, W)  — N images, C channels, H×W spatial

For each channel c (independently):
  Collect ALL values across batch (N) and spatial (H×W)
  Total values per channel: N × H × W

  ┌──────────────────────────────────────────────┐
  │ Batch dimension (N images)                    │
  │ ┌────────┐ ┌────────┐ ┌────────┐             │
  │ │ Image1 │ │ Image2 │ │ Image3 │  ...        │
  │ │ Ch. c  │ │ Ch. c  │ │ Ch. c  │             │
  │ │ ┌────┐ │ │ ┌────┐ │ │ ┌────┐ │             │
  │ │ │H×W │ │ │ │H×W │ │ │ │H×W │ │             │
  │ │ │vals│ │ │ │vals│ │ │ │vals│ │             │
  │ │ └────┘ │ │ └────┘ │ │ └────┘ │             │
  │ └────────┘ └────────┘ └────────┘             │
  │                                               │
  │ All these values → compute μ_c, σ²_c         │
  │ Total: N × H × W values per channel          │
  └──────────────────────────────────────────────┘
```

### Mathematical Formulation

**Step 1: Compute channel statistics**

```
For channel c:

    μ_c = (1 / (N·H·W)) × Σ_n Σ_h Σ_w  x[n, c, h, w]

    σ²_c = (1 / (N·H·W)) × Σ_n Σ_h Σ_w  (x[n, c, h, w] - μ_c)²
```

**Step 2: Normalize**

```
    x̂[n, c, h, w] = (x[n, c, h, w] - μ_c) / √(σ²_c + ε)

    ε = 1e-5 (small constant for numerical stability)
```

**Step 3: Scale and shift (learnable)**

```
    y[n, c, h, w] = γ_c · x̂[n, c, h, w] + β_c

    γ_c = learnable scale (initialized to 1)
    β_c = learnable shift (initialized to 0)
```

### Why γ and β?

If normalization always forced activations to N(0,1), it would **limit** the network's representational power. The learnable γ and β allow each channel to learn its **optimal** distribution:

```
If the network "decides" normalization isn't helpful for channel c,
it can learn γ_c = σ_c and β_c = μ_c to undo the normalization!

y = γ · (x - μ) / σ + β

If γ = σ and β = μ:
  y = σ · (x - μ) / σ + μ = x - μ + μ = x    (identity!)
```

---

## 3️⃣ Worked Example — Manual BatchNorm2d

```
Input: (N=2, C=2, H=2, W=2)    ← 2 images, 2 channels, 2×2 spatial

Image 1, Ch 0:    Image 1, Ch 1:    Image 2, Ch 0:    Image 2, Ch 1:
┌────┬────┐       ┌────┬────┐       ┌────┬────┐       ┌────┬────┐
│  1 │  2 │       │  5 │  6 │       │  3 │  4 │       │  7 │  8 │
├────┼────┤       ├────┼────┤       ├────┼────┤       ├────┼────┤
│  3 │  4 │       │  7 │  8 │       │  5 │  6 │       │  9 │ 10 │
└────┴────┘       └────┴────┘       └────┴────┘       └────┴────┘

Step 1: Channel statistics
  Channel 0 values: [1,2,3,4, 3,4,5,6] = 8 values
    μ₀ = (1+2+3+4+3+4+5+6)/8 = 28/8 = 3.5
    σ²₀ = ((1-3.5)²+(2-3.5)²+...+(6-3.5)²)/8
         = (6.25+2.25+0.25+0.25+0.25+0.25+2.25+6.25)/8
         = 18/8 = 2.25
    σ₀ = √2.25 = 1.5

  Channel 1 values: [5,6,7,8, 7,8,9,10] = 8 values
    μ₁ = (5+6+7+8+7+8+9+10)/8 = 60/8 = 7.5
    σ²₁ = 2.25  (same spread)
    σ₁ = 1.5

Step 2: Normalize (ε ≈ 0 for simplicity)
  Channel 0: x̂ = (x - 3.5) / 1.5

  Image 1, Ch 0:                Image 2, Ch 0:
  ┌──────────┬──────────┐       ┌──────────┬──────────┐
  │ -1.667   │ -1.000   │       │ -0.333   │  0.333   │
  ├──────────┼──────────┤       ├──────────┼──────────┤
  │ -0.333   │  0.333   │       │  1.000   │  1.667   │
  └──────────┴──────────┘       └──────────┴──────────┘

Step 3: Scale and shift (γ=1, β=0 initially)
  y = 1 · x̂ + 0 = x̂  (no change at initialization)
```

### Verify with PyTorch

```python
import torch
import torch.nn as nn

# Create input
x = torch.tensor([
    [[[1., 2.], [3., 4.]],   # Image 1, Channel 0
     [[5., 6.], [7., 8.]]],  # Image 1, Channel 1
    [[[3., 4.], [5., 6.]],   # Image 2, Channel 0
     [[7., 8.], [9., 10.]]]  # Image 2, Channel 1
])

print(f"Input shape: {x.shape}")  # [2, 2, 2, 2]

# BatchNorm2d
bn = nn.BatchNorm2d(num_features=2)  # 2 channels

# Forward pass (training mode)
bn.train()
y = bn(x)
print(f"\nOutput (training):")
print(y)

# Verify channel 0 statistics
ch0_values = x[:, 0, :, :].flatten()
print(f"\nChannel 0 mean: {ch0_values.mean():.4f}")  # 3.5
print(f"Channel 0 std:  {ch0_values.std(unbiased=False):.4f}")  # 1.5

# After BN, each channel should have mean≈0, std≈1
print(f"\nBN output Ch0 mean: {y[:, 0].mean():.4f}")  # ≈ 0
print(f"BN output Ch0 std:  {y[:, 0].std(unbiased=False):.4f}")  # ≈ 1
```

---

## 4️⃣ BatchNorm2d Parameters — Full Reference

```python
torch.nn.BatchNorm2d(
    num_features,       # C — number of channels (must match input channel dim)
    eps=1e-05,          # ε — small constant for numerical stability
    momentum=0.1,       # Exponential moving average factor for running stats
    affine=True,        # If True, learn γ and β (scale and shift)
    track_running_stats=True  # If True, maintain running mean and variance
)
```

### Parameter Inventory

```
For nn.BatchNorm2d(C):

Learnable parameters (updated by optimizer):
  γ (weight):  C values  — per-channel scale (init: 1.0)
  β (bias):    C values  — per-channel shift (init: 0.0)
  Total learnable: 2C

Running statistics (updated by forward pass, not optimizer):
  running_mean:  C values  — exponential moving average of μ (init: 0.0)
  running_var:   C values  — exponential moving average of σ² (init: 1.0)
  num_batches_tracked: 1 value — count of forward passes

Total state: 2C learnable + 2C running + 1 counter
```

### Verify in PyTorch

```python
bn = nn.BatchNorm2d(64)

# Learnable parameters
print(f"gamma (weight): {bn.weight.shape}")  # [64]
print(f"beta (bias):    {bn.bias.shape}")    # [64]

# Running statistics (buffers, not parameters)
print(f"running_mean:   {bn.running_mean.shape}")  # [64]
print(f"running_var:    {bn.running_var.shape}")   # [64]

# Parameter count
learnable = sum(p.numel() for p in bn.parameters())
print(f"Learnable params: {learnable}")  # 128 (64 γ + 64 β)
# Note: running stats are NOT in parameters() — they're buffers
```

---

## 5️⃣ Training vs Eval Mode — CRITICAL!

This is the **most common source of bugs** with BatchNorm.

### Training Mode (`model.train()`)

```
Uses BATCH statistics:
  μ_c = mean of current batch (N × H × W values per channel)
  σ²_c = variance of current batch

Also UPDATES running statistics:
  running_mean = (1 - momentum) × running_mean + momentum × μ_batch
  running_var  = (1 - momentum) × running_var  + momentum × σ²_batch

  (momentum=0.1 by default)
```

### Eval Mode (`model.eval()`)

```
Uses RUNNING statistics (accumulated during training):
  μ_c = running_mean_c    (fixed, not computed from current input)
  σ²_c = running_var_c    (fixed, not computed from current input)

Does NOT update running statistics.
γ and β are used as-is (no gradient though, if in torch.no_grad()).
```

### Why This Matters

```
Training with batch stats:                Eval with running stats:
  ✓ Batch stats estimated from           ✓ Uses stable, accumulated stats
    many samples (N × H × W)             ✓ Deterministic output
  ✓ Running stats accumulate               (same input → same output)
    gradually                             ✓ Works for single images
  ✗ Different batches → different         ✓ Required for deployment
    stats → slightly different
    normalization

If you forget model.eval() during inference:
  ✗ Uses batch stats from a single image → wildly incorrect normalization!
  ✗ Updates running stats → corrupts the model's accumulated statistics!
  ✗ Non-deterministic predictions
```

### ASCII Diagram: Train vs Eval Flow

```
TRAINING MODE:
                          ┌─── compute μ_batch, σ²_batch ───┐
                          │                                   │
  Input ──► Normalize ────┤                                   │
  (N,C,H,W)   using      │     update running stats:         │
            batch stats   │     running_mean ← EMA(μ_batch)  │
                          │     running_var  ← EMA(σ²_batch) │
                          └───────────────────────────────────┘
                   │
                   ▼
            Scale & Shift (γ, β)
                   │
                   ▼
              Output (N,C,H,W)


EVAL MODE:
                          ┌─── use running_mean, running_var ─┐
                          │    (accumulated during training)   │
  Input ──► Normalize ────┤                                    │
  (N,C,H,W)   using      │    NO updates to running stats     │
          running stats   │                                    │
                          └────────────────────────────────────┘
                   │
                   ▼
            Scale & Shift (γ, β)
                   │
                   ▼
              Output (N,C,H,W)
```

### PyTorch Demonstration

```python
import torch
import torch.nn as nn

bn = nn.BatchNorm2d(2)

# Training mode
bn.train()
x1 = torch.randn(4, 2, 8, 8)  # batch of 4
y1 = bn(x1)
print(f"After train batch 1:")
print(f"  running_mean: {bn.running_mean}")
print(f"  running_var:  {bn.running_var}")

x2 = torch.randn(4, 2, 8, 8)  # different batch
y2 = bn(x2)
print(f"After train batch 2:")
print(f"  running_mean: {bn.running_mean}")  # updated!
print(f"  running_var:  {bn.running_var}")    # updated!

# Eval mode
bn.eval()
y_eval1 = bn(x1)
y_eval2 = bn(x1)  # same input
print(f"\nEval outputs equal: {torch.equal(y_eval1, y_eval2)}")  # True!

# Check running stats didn't change
print(f"running_mean unchanged in eval: {bn.running_mean}")  # same as after training

# ⚠️ Critical: always use model.eval() for inference!
# ⚠️ Critical: always use model.train() when resuming training!
```

---

## 6️⃣ Placement in CNN Architecture

### Standard: Conv → BN → ReLU

```python
# The most common pattern in modern CNNs
block = nn.Sequential(
    nn.Conv2d(64, 128, 3, padding=1, bias=False),  # bias=False since BN has β
    nn.BatchNorm2d(128),                             # normalize per channel
    nn.ReLU(inplace=True),                           # non-linearity
)
```

### Why `bias=False` in Conv Before BN?

```
Conv output:  z = W*x + b     (b is conv bias)
BN step 1:   ẑ = (z - μ) / σ = (W*x + b - μ) / σ

The bias b is absorbed into the mean μ:
  μ = mean(W*x + b) = mean(W*x) + b

So:  ẑ = (W*x + b - mean(W*x) - b) / σ = (W*x - mean(W*x)) / σ

The bias b cancels out! It has no effect on the normalized output.
BN's own β parameter serves as the effective bias.

Therefore: bias=False saves C parameters with zero accuracy impact.
```

### Full Block Patterns

```python
# Pattern 1: Conv-BN-ReLU (ResNet, VGG-BN, most standard models)
def conv_bn_relu(in_ch, out_ch, **kwargs):
    return nn.Sequential(
        nn.Conv2d(in_ch, out_ch, bias=False, **kwargs),
        nn.BatchNorm2d(out_ch),
        nn.ReLU(inplace=True),
    )

# Pattern 2: BN-ReLU-Conv (Pre-activation ResNet v2)
def bn_relu_conv(in_ch, out_ch, **kwargs):
    return nn.Sequential(
        nn.BatchNorm2d(in_ch),   # normalize input channels
        nn.ReLU(inplace=True),
        nn.Conv2d(in_ch, out_ch, bias=False, **kwargs),
    )

# Pattern 3: Conv-BN-ReLU with residual
class ResBlock(nn.Module):
    def __init__(self, channels):
        super().__init__()
        self.block = nn.Sequential(
            nn.Conv2d(channels, channels, 3, padding=1, bias=False),
            nn.BatchNorm2d(channels),
            nn.ReLU(inplace=True),
            nn.Conv2d(channels, channels, 3, padding=1, bias=False),
            nn.BatchNorm2d(channels),
        )
        self.relu = nn.ReLU(inplace=True)
    
    def forward(self, x):
        return self.relu(self.block(x) + x)
```

### Where BatchNorm Appears in an Architecture

```
Typical CNN Architecture:

Input (3, 224, 224)
  │
  ├─ Conv2d(3, 64, 7, stride=2, padding=3)    ← No BN before first conv
  ├─ BatchNorm2d(64)                            ← BN after first conv
  ├─ ReLU
  ├─ MaxPool2d(3, stride=2, padding=1)
  │
  ├─ Conv2d(64, 64, 3, padding=1)
  ├─ BatchNorm2d(64)                            ← BN after every conv
  ├─ ReLU
  ├─ Conv2d(64, 64, 3, padding=1)
  ├─ BatchNorm2d(64)                            ← BN after every conv
  │  (+residual)
  ├─ ReLU
  │   ...
  ├─ AdaptiveAvgPool2d(1)
  ├─ Flatten
  ├─ Linear(512, 1000)                          ← No BN after final FC
  │
  Output (1000)

Rule: BN after EVERY conv layer, but NOT after the final linear layer.
```

---

## 7️⃣ Effect on Training

### Benefits

```
┌────────────────────────────────────────────────────────────────┐
│  1. FASTER CONVERGENCE                                        │
│     • Can use 10-30× higher learning rates                    │
│     • Normalized gradients prevent exploding/vanishing         │
│                                                                │
│  2. REGULARIZATION EFFECT                                      │
│     • Batch statistics add noise (mini-batch sampling)         │
│     • Often reduces need for Dropout in conv layers            │
│     • Each sample normalized differently depending on batch    │
│                                                                │
│  3. INITIALIZATION ROBUSTNESS                                  │
│     • Less sensitive to weight initialization                  │
│     • Bad init → BN normalizes activations anyway              │
│                                                                │
│  4. SMOOTHER LOSS LANDSCAPE                                    │
│     • Normalization makes optimization surface more predictable│
│     • Larger valid range for hyperparameters                   │
└────────────────────────────────────────────────────────────────┘
```

### Training Curves Comparison (Conceptual)

```
Loss                Without BN           With BN
  │                   ╲                    ╲
  │                    ╲                    ╲
  │                     ╲╱╲                  ╲
  │                      ╱  ╲╱╲              ╲───────
  │                     ╱      ╲╱╲                 ↑
  │                    ╱          ╲╱╲           Converges faster
  │                   ╱              ╲╱╲       and to lower loss
  │                  ╱                  ╲╱───
  └────────────────────────────────────────────► Epoch
     With BN converges in ~3× fewer epochs
     With BN reaches better final loss
     With BN is more stable (less oscillation)
```

### Learning Rate Sensitivity

```python
# Without BN: must use small learning rate
optimizer_no_bn = torch.optim.SGD(model_no_bn.parameters(), lr=0.01)

# With BN: can use much higher learning rate
optimizer_with_bn = torch.optim.SGD(model_with_bn.parameters(), lr=0.1)
# 10× higher LR → 10× faster training!
```

---

## 8️⃣ Momentum and Running Statistics

### Exponential Moving Average (EMA)

```
After each training batch:

  running_mean = (1 - momentum) × running_mean + momentum × batch_mean
  running_var  = (1 - momentum) × running_var  + momentum × batch_var

Default momentum = 0.1:
  running_mean = 0.9 × running_mean + 0.1 × batch_mean

This gives recent batches more weight, but old statistics still influence.
```

### Momentum Values

```
momentum = 0.1  (PyTorch default)
  → running stats update slowly
  → More stable but slower to adapt
  → Good for most cases

momentum = 0.01
  → Very slow updates
  → Extremely stable running stats
  → Use for very noisy batches or long training

momentum = 0.5
  → Fast updates
  → Running stats closely track recent batches
  → Risky for small batch sizes
```

### Visualizing EMA

```python
import torch
import torch.nn as nn

bn = nn.BatchNorm2d(1, momentum=0.1)
bn.train()

print(f"Initial running_mean: {bn.running_mean.item():.4f}")  # 0.0

# Simulate batches with different means
for batch_idx in range(10):
    x = torch.randn(32, 1, 8, 8) + batch_idx  # increasing mean
    bn(x)
    print(f"Batch {batch_idx}: batch_mean={x.mean():.2f}, "
          f"running_mean={bn.running_mean.item():.4f}")

# running_mean gradually approaches the batch means but with lag
# After 10 batches with increasing mean:
# running_mean ≈ weighted average, more influenced by recent batches
```

---

## 9️⃣ BatchNorm2d vs Other Normalizations

### Comparison of Normalization Strategies

```
Input shape: (N, C, H, W)

BatchNorm2d:          Normalize across N, H, W (per channel)
                      ┌─────┬─────┬─────┐
                      │ Img1│ Img2│ Img3│  ← across batch
                      │ Ch c│ Ch c│ Ch c│  ← same channel
                      └─────┴─────┴─────┘
                      Stats: C means, C variances

LayerNorm:            Normalize across C, H, W (per image)
                      ┌─────────────────┐
                      │ All channels of  │  ← single image
                      │ one image        │  ← all channels
                      └─────────────────┘
                      Stats: N means, N variances

InstanceNorm:         Normalize across H, W (per channel, per image)
                      ┌───────┐
                      │One ch │  ← single image, single channel
                      │of one │
                      │image  │
                      └───────┘
                      Stats: N×C means, N×C variances

GroupNorm:            Normalize across H, W and channel groups
                      ┌─────────────────┐
                      │Group of channels │  ← single image
                      │of one image      │  ← subset of channels
                      └─────────────────┘
                      Stats: N×G means, N×G variances
```

```python
import torch.nn as nn

C = 64  # channels
# All normalize to similar distributions but compute stats differently:
bn = nn.BatchNorm2d(C)         # per-channel across batch — standard for CNNs
ln = nn.LayerNorm([C, 32, 32]) # per-sample across all — standard for transformers
gn = nn.GroupNorm(8, C)        # per-group (8 groups) — when batch is small
in_norm = nn.InstanceNorm2d(C) # per-instance-per-channel — style transfer
```

### When to Use What

| Normalization | Best For | Batch Size Dependent? |
|--------------|---------|----------------------|
| **BatchNorm2d** | Standard CNN training, large batches | Yes (needs N ≥ 16) |
| **GroupNorm** | Small batches, detection/segmentation | No |
| **LayerNorm** | Transformers, RNNs | No |
| **InstanceNorm** | Style transfer, image generation | No |

---

## 🔟 Complete Code Example

```python
import torch
import torch.nn as nn

class BatchNormDemoCNN(nn.Module):
    """CNN demonstrating BatchNorm placement and behavior."""
    
    def __init__(self, num_classes=10, use_batchnorm=True):
        super().__init__()
        
        self.use_bn = use_batchnorm
        
        # Feature extractor
        layers = []
        in_channels = 3
        for out_channels in [32, 64, 128]:
            layers.append(nn.Conv2d(in_channels, out_channels, 3, 
                                     padding=1, bias=not use_batchnorm))
            if use_batchnorm:
                layers.append(nn.BatchNorm2d(out_channels))
            layers.append(nn.ReLU(inplace=True))
            layers.append(nn.MaxPool2d(2, 2))
            in_channels = out_channels
        
        self.features = nn.Sequential(*layers)
        
        # Classifier
        self.classifier = nn.Sequential(
            nn.AdaptiveAvgPool2d(1),
            nn.Flatten(),
            nn.Linear(128, num_classes),
        )
    
    def forward(self, x):
        return self.classifier(self.features(x))

# Compare models with and without BN
model_bn = BatchNormDemoCNN(use_batchnorm=True)
model_no_bn = BatchNormDemoCNN(use_batchnorm=False)

x = torch.randn(8, 3, 32, 32)

# Training mode
model_bn.train()
model_no_bn.train()
out_bn = model_bn(x)
out_no_bn = model_no_bn(x)

print(f"With BN params:    {sum(p.numel() for p in model_bn.parameters()):,}")
print(f"Without BN params: {sum(p.numel() for p in model_no_bn.parameters()):,}")

# Print BN layer statistics
print("\nBatchNorm layers:")
for name, module in model_bn.named_modules():
    if isinstance(module, nn.BatchNorm2d):
        print(f"  {name}:")
        print(f"    channels: {module.num_features}")
        print(f"    weight (γ): mean={module.weight.mean():.3f}, "
              f"shape={module.weight.shape}")
        print(f"    bias (β):   mean={module.bias.mean():.3f}")
        print(f"    running_mean: {module.running_mean.mean():.4f}")
        print(f"    running_var:  {module.running_var.mean():.4f}")

# CRITICAL: Switch to eval mode for inference
model_bn.eval()
with torch.no_grad():
    # These two calls produce IDENTICAL results (deterministic)
    pred1 = model_bn(x)
    pred2 = model_bn(x)
    print(f"\nEval deterministic: {torch.equal(pred1, pred2)}")  # True
```

### Training Loop with Proper Mode Switching

```python
import torch
import torch.nn as nn
import torch.optim as optim

model = BatchNormDemoCNN(num_classes=10)
optimizer = optim.Adam(model.parameters(), lr=0.001)
criterion = nn.CrossEntropyLoss()

def train_epoch(model, dataloader, optimizer, criterion):
    model.train()  # ← CRITICAL: enables BN training mode
    total_loss = 0
    for inputs, targets in dataloader:
        optimizer.zero_grad()
        outputs = model(inputs)
        loss = criterion(outputs, targets)
        loss.backward()
        optimizer.step()
        total_loss += loss.item()
    return total_loss / len(dataloader)

def evaluate(model, dataloader):
    model.eval()  # ← CRITICAL: enables BN eval mode
    correct = 0
    total = 0
    with torch.no_grad():  # no gradient computation
        for inputs, targets in dataloader:
            outputs = model(inputs)
            _, predicted = outputs.max(1)
            total += targets.size(0)
            correct += predicted.eq(targets).sum().item()
    return correct / total

# Training loop
# for epoch in range(num_epochs):
#     train_loss = train_epoch(model, train_loader, optimizer, criterion)
#     model.eval()  # switch to eval BEFORE validation
#     val_acc = evaluate(model, val_loader)
#     model.train()  # switch back to train for next epoch
```

### Freezing BatchNorm During Fine-Tuning

```python
def freeze_bn(model):
    """Freeze all BatchNorm layers — keep using running stats."""
    for module in model.modules():
        if isinstance(module, (nn.BatchNorm2d, nn.BatchNorm1d)):
            module.eval()  # use running stats
            for param in module.parameters():
                param.requires_grad = False  # don't update γ, β

# Usage: when fine-tuning with very small batches
# (batch stats would be unreliable)
model.train()
freeze_bn(model)  # BN layers stay in eval mode even though model is training
```

---

## 📊 Summary Table

| Property | Details |
|----------|---------|
| **PyTorch** | `nn.BatchNorm2d(num_features=C)` |
| **Normalizes over** | Batch (N) and spatial (H, W) dimensions — **per channel** |
| **Learnable params** | γ (scale, init=1) and β (shift, init=0) — 2C total |
| **Running stats** | running_mean, running_var — updated via EMA during training |
| **Training mode** | Uses **batch statistics**, updates running stats |
| **Eval mode** | Uses **running statistics**, no updates |
| **Placement** | After Conv2d, before ReLU: `Conv → BN → ReLU` |
| **bias=False** | Set in preceding Conv2d (BN's β replaces conv bias) |
| **Benefits** | 10-30× higher LR, faster convergence, regularization |
| **Momentum** | 0.1 default — EMA update factor for running stats |
| **Num BN params** | 2C learnable + 2C buffers (for C channels) |
| **Alternative** | GroupNorm (small batches), LayerNorm (transformers) |

---

## ❓ Revision Questions

### Q1: Parameter Count
**A CNN has BatchNorm2d layers after conv layers producing 64, 128, 256, and 512 channels. How many total learnable parameters do the BN layers contribute?**

<details>
<summary>Answer</summary>

Each BN layer has 2C learnable parameters (γ and β):
- BN(64): 2 × 64 = 128
- BN(128): 2 × 128 = 256
- BN(256): 2 × 256 = 512
- BN(512): 2 × 512 = 1,024

Total: 128 + 256 + 512 + 1,024 = **1,920 learnable parameters**

Note: There are also 2C running statistics (buffers) per layer, but these are NOT learnable parameters.
</details>

### Q2: Training vs Eval Bug
**A model with BatchNorm achieves 95% accuracy during training but only 60% during inference. What is the most likely cause?**

<details>
<summary>Answer</summary>

The most likely cause is **forgetting to call `model.eval()`** before inference.

When the model stays in training mode during inference:
1. BN uses **batch statistics** from the current (single) input instead of the accumulated **running statistics**
2. For a single image (batch size 1), the "batch mean" equals the image mean and "batch variance" is zero — leading to division by near-zero and wildly incorrect normalization
3. Even for larger inference batches, the batch statistics don't match the training distribution

Fix: Always call `model.eval()` before inference and `model.train()` before training.

```python
# Before inference:
model.eval()
with torch.no_grad():
    predictions = model(test_data)
```
</details>

### Q3: Why Per-Channel?
**Why does BatchNorm2d compute separate statistics for each channel rather than normalizing across all channels together?**

<details>
<summary>Answer</summary>

Each channel in a CNN feature map represents a **different learned feature** (e.g., edge detector, texture detector, color pattern). These features have inherently different distributions — an edge detector might output large values where edges exist and zero elsewhere, while a color channel might be uniformly active.

Normalizing across channels would:
- Mix semantically different features together
- Force all features to the same scale, potentially destroying meaningful differences
- Remove the distinction between highly activated features and inactive ones

Per-channel normalization preserves the **relative activation patterns within each channel** while ensuring each channel individually has stable statistics. The learned γ and β allow each channel to maintain its own optimal scale and offset.
</details>

### Q4: Conv Bias with BN
**Prove mathematically that the bias in `nn.Conv2d` is redundant when followed by `nn.BatchNorm2d`.**

<details>
<summary>Answer</summary>

Let the conv output be z = Wx + b (where b is the conv bias, broadcast across spatial dims).

BN normalizes:
```
ẑ = (z - μ_z) / σ_z
  = (Wx + b - μ_{Wx+b}) / σ_{Wx+b}
  = (Wx + b - μ_{Wx} - b) / σ_{Wx}     [since mean(Wx + b) = mean(Wx) + b]
  = (Wx - μ_{Wx}) / σ_{Wx}              [b cancels!]
```

The bias b has **no effect** on the normalized output because:
1. It shifts all values by a constant
2. The mean subtraction removes that constant
3. The variance is unaffected by a constant shift

Since BN then applies its own learned shift β:
```
y = γ · ẑ + β
```

β serves as the effective bias, making the conv bias completely redundant. Setting `bias=False` saves C parameters with no loss of expressiveness.
</details>

### Q5: Batch Size Effect
**Why does BatchNorm perform poorly with very small batch sizes (e.g., batch_size=1 or 2)? What alternative would you use?**

<details>
<summary>Answer</summary>

With small batches, the **batch statistics** (mean and variance computed from N samples) are **noisy and unreliable** estimates of the true population statistics:

- Batch size 1: Mean = the single sample's mean, Variance = 0 → division by zero!
- Batch size 2: Very high variance in the estimated statistics between batches

This leads to:
1. Unstable training (loss oscillates wildly)
2. Poor running statistics (accumulated from noisy estimates)
3. Large gap between training and eval performance

**Alternatives for small batches:**
- **GroupNorm**: Normalizes across groups of channels within each sample (batch-independent)
- **LayerNorm**: Normalizes across all channels per sample (batch-independent)
- **InstanceNorm**: Normalizes per channel per sample (batch-independent)

```python
# GroupNorm: divide 64 channels into 8 groups of 8
gn = nn.GroupNorm(num_groups=8, num_channels=64)  # works with batch_size=1!
```
</details>

### Q6: Running Statistics
**After training for 100 epochs, you save the model and load it on a new machine. The running_mean and running_var are loaded correctly, but you notice slightly different predictions compared to the original machine. What could cause this?**

<details>
<summary>Answer</summary>

Possible causes:

1. **Forgot `model.eval()` on one of the machines** → one uses batch stats, the other uses running stats

2. **Different floating-point precision** → different GPU architectures (e.g., NVIDIA A100 vs V100) may have slightly different floating-point behavior, causing tiny numerical differences that propagate through the network

3. **Different PyTorch versions** → numerical behavior can vary between versions

4. **CUDA non-determinism** → by default, CUDA operations are non-deterministic for performance. Use `torch.backends.cudnn.deterministic = True` for reproducibility

5. **Different batch composition during the final training batches** → if training wasn't fully completed, the running stats might not have fully converged, and the final running_mean/running_var depend on the exact sequence of batches seen

The fix: Ensure identical environment, use `model.eval()`, and enable deterministic mode:
```python
torch.backends.cudnn.deterministic = True
torch.backends.cudnn.benchmark = False
```
</details>

---

[← Previous: 05 — Flatten Operation](05-flatten-operation.md) | [📑 Unit Overview](../README.md)
