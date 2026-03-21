# 🌊 Pooling Layers — Spatial Downsampling in CNNs

[← Previous: 02 — Activation Layer](02-activation-layer.md) | [📑 Unit Overview](../README.md) | [Next → 04 — Fully Connected Layer](04-fully-connected-layer.md)

---

## 📋 Chapter Overview

Pooling layers **reduce the spatial dimensions** of feature maps, making the representation smaller and more manageable while retaining the most important information. They are a fundamental component of CNN architectures, appearing after convolutional + activation blocks to progressively shrink spatial resolution while increasing the number of channels.

**What you'll learn:**
- MaxPool2d and AvgPool2d — standard spatial pooling
- AdaptiveMaxPool2d and AdaptiveAvgPool2d — output-size-based pooling
- Why pooling has zero learnable parameters
- Where to place pooling in an architecture
- How backpropagation works through pooling
- Practical PyTorch code and worked examples

---

## 1️⃣ Why Pooling?

### Goals of Pooling

```
┌──────────────────────────────────────────────────────────────────┐
│  1. REDUCE SPATIAL DIMENSIONS  →  fewer computations downstream │
│  2. INCREASE RECEPTIVE FIELD   →  each neuron "sees" more input │
│  3. PROVIDE INVARIANCE         →  small shifts don't change out │
│  4. PREVENT OVERFITTING        →  fewer parameters in FC layers │
└──────────────────────────────────────────────────────────────────┘
```

### Visual: What Pooling Does

```
Input Feature Map (8×8)                    After 2×2 MaxPool (4×4)
┌──┬──┬──┬──┬──┬──┬──┬──┐                 ┌──┬──┬──┬──┐
│  │  │  │  │  │  │  │  │                 │  │  │  │  │
├──┼──┼──┼──┼──┼──┼──┼──┤                 ├──┼──┼──┼──┤
│  │  │  │  │  │  │  │  │    MaxPool      │  │  │  │  │
├──┼──┼──┼──┼──┼──┼──┼──┤   ────────►     ├──┼──┼──┼──┤
│  │  │  │  │  │  │  │  │    2×2, s=2     │  │  │  │  │
├──┼──┼──┼──┼──┼──┼──┼──┤                 ├──┼──┼──┼──┤
│  │  │  │  │  │  │  │  │                 │  │  │  │  │
├──┼──┼──┼──┼──┼──┼──┼──┤                 └──┴──┴──┴──┘
│  │  │  │  │  │  │  │  │
├──┼──┼──┼──┼──┼──┼──┼──┤                 Spatial dimensions halved!
│  │  │  │  │  │  │  │  │                 Number of channels unchanged.
├──┼──┼──┼──┼──┼──┼──┼──┤                 Memory reduced by 4×.
│  │  │  │  │  │  │  │  │
├──┼──┼──┼──┼──┼──┼──┼──┤
│  │  │  │  │  │  │  │  │
└──┴──┴──┴──┴──┴──┴──┴──┘
```

---

## 2️⃣ MaxPool2d — Maximum Pooling

### How It Works

For each pooling window, take the **maximum** value:

```
Input (4×4):                    MaxPool2d(kernel_size=2, stride=2)
┌────┬────┬────┬────┐          ┌────┬────┐
│  1 │  3 │  2 │  4 │          │    │    │
├────┼────┼────┼────┤          │  5 │  4 │
│  5 │  2 │  1 │  0 │   ───►   ├────┼────┤
├────┼────┼────┼────┤          │    │    │
│  3 │  1 │  7 │  2 │          │  3 │  7 │
├────┼────┼────┼────┤          └────┴────┘
│  0 │  2 │  4 │  6 │
└────┴────┴────┴────┘

Window [1,3,5,2] → max = 5     Window [2,4,1,0] → max = 4
Window [3,1,0,2] → max = 3     Window [7,2,4,6] → max = 7
```

### PyTorch API

```python
torch.nn.MaxPool2d(
    kernel_size,      # Size of pooling window (k or (k_h, k_w))
    stride=None,      # Step size (defaults to kernel_size if None)
    padding=0,        # Zero-padding before pooling
    dilation=1,       # Spacing between pooling window elements
    return_indices=False,  # Return indices of max values (for unpooling)
    ceil_mode=False    # Use ceil instead of floor for output size
)
```

### Output Shape Formula

Same as convolution (with dilation=1):

```
H_out = floor( (H_in + 2p - k) / s + 1 )
W_out = floor( (W_in + 2p - k) / s + 1 )
```

### Common Configurations

```python
# Standard 2×2 max pooling (halves spatial dims)
pool = nn.MaxPool2d(kernel_size=2, stride=2)
# Input: (N, C, 32, 32) → Output: (N, C, 16, 16)

# 3×3 max pooling with stride 2 and padding (also halves)
pool = nn.MaxPool2d(kernel_size=3, stride=2, padding=1)
# Input: (N, C, 32, 32) → Output: (N, C, 16, 16)

# Overlapping pooling (AlexNet)
pool = nn.MaxPool2d(kernel_size=3, stride=2)
# Slightly overlapping windows — reduces overfitting
```

### Worked Example with PyTorch

```python
import torch
import torch.nn as nn

x = torch.tensor([[[[1., 3., 2., 4.],
                     [5., 2., 1., 0.],
                     [3., 1., 7., 2.],
                     [0., 2., 4., 6.]]]])

pool = nn.MaxPool2d(kernel_size=2, stride=2)
out = pool(x)
print(out)
# tensor([[[[5., 4.],
#           [3., 7.]]]])

# With return_indices (needed for MaxUnpool2d)
pool_idx = nn.MaxPool2d(kernel_size=2, stride=2, return_indices=True)
out, indices = pool_idx(x)
print(f"Output: {out}")
print(f"Indices: {indices}")
# Indices tell you WHERE the max came from (flattened position in each window)
# tensor([[[[4, 3],
#           [0, 2]]]])  → positions of max values
```

---

## 3️⃣ AvgPool2d — Average Pooling

### How It Works

For each pooling window, take the **arithmetic mean**:

```
Input (4×4):                    AvgPool2d(kernel_size=2, stride=2)
┌────┬────┬────┬────┐          ┌──────┬──────┐
│  1 │  3 │  2 │  4 │          │      │      │
├────┼────┼────┼────┤          │ 2.75 │ 1.75 │
│  5 │  2 │  1 │  0 │   ───►   ├──────┼──────┤
├────┼────┼────┼────┤          │      │      │
│  3 │  1 │  7 │  2 │          │ 1.50 │ 4.75 │
├────┼────┼────┼────┤          └──────┴──────┘
│  0 │  2 │  4 │  6 │
└────┴────┴────┴────┘

Window [1,3,5,2] → mean = (1+3+5+2)/4 = 2.75
Window [2,4,1,0] → mean = (2+4+1+0)/4 = 1.75
Window [3,1,0,2] → mean = (3+1+0+2)/4 = 1.50
Window [7,2,4,6] → mean = (7+2+4+6)/4 = 4.75
```

### PyTorch API

```python
torch.nn.AvgPool2d(
    kernel_size,        # Size of pooling window
    stride=None,        # Defaults to kernel_size
    padding=0,          # Zero-padding
    ceil_mode=False,    # Use ceil for output size
    count_include_pad=True,  # Include padding zeros in average calculation
    divisor_override=None    # Force a specific divisor for the average
)
```

### Max Pooling vs Average Pooling

```
MaxPool: Takes the strongest activation in each window
  → Preserves the most prominent features
  → More translation invariant
  → Standard in feature extraction layers

AvgPool: Averages all activations in each window
  → Preserves overall activation level
  → Smoother, less aggressive
  → More common as Global Average Pooling at the end
```

```python
import torch
import torch.nn as nn

x = torch.tensor([[[[1., 3., 2., 4.],
                     [5., 2., 1., 0.],
                     [3., 1., 7., 2.],
                     [0., 2., 4., 6.]]]])

max_pool = nn.MaxPool2d(2, 2)
avg_pool = nn.AvgPool2d(2, 2)

print(f"MaxPool: {max_pool(x)}")
# tensor([[[[5., 4.],
#           [3., 7.]]]])

print(f"AvgPool: {avg_pool(x)}")
# tensor([[[[2.7500, 1.7500],
#           [1.5000, 4.7500]]]])
```

---

## 4️⃣ AdaptiveMaxPool2d — Output-Size Pooling

### The Problem with Fixed Pooling

Standard pooling requires you to manually calculate kernel sizes to get the desired output dimensions. For different input sizes, you'd need different pooling configurations.

### Adaptive Pooling Solution

Specify the **desired output size**, and PyTorch automatically computes the kernel size and stride:

```python
torch.nn.AdaptiveMaxPool2d(output_size)  # output_size: int or (H, W)

# Examples:
pool = nn.AdaptiveMaxPool2d((4, 4))    # Always outputs 4×4 regardless of input
pool = nn.AdaptiveMaxPool2d(1)          # Global Max Pooling → outputs 1×1
pool = nn.AdaptiveMaxPool2d((None, 4))  # Keep height, pool width to 4
```

### How It Works Internally

```
For input size H_in and desired output H_out:
  stride = floor(H_in / H_out)
  kernel = H_in - (H_out - 1) * stride

Example: H_in=7, H_out=3
  stride = floor(7/3) = 2
  kernel = 7 - 2*2 = 3
  → Uses 3×3 kernel with stride 2 → output: floor((7-3)/2 + 1) = 3 ✓
```

### Practical Use: Handling Variable Input Sizes

```python
import torch
import torch.nn as nn

class FlexibleCNN(nn.Module):
    """CNN that accepts any input spatial size."""
    
    def __init__(self, num_classes=10):
        super().__init__()
        self.features = nn.Sequential(
            nn.Conv2d(3, 64, 3, padding=1),
            nn.ReLU(inplace=True),
            nn.Conv2d(64, 128, 3, padding=1),
            nn.ReLU(inplace=True),
        )
        # Adaptive pooling ensures fixed-size output regardless of input
        self.pool = nn.AdaptiveMaxPool2d((4, 4))
        self.classifier = nn.Linear(128 * 4 * 4, num_classes)
    
    def forward(self, x):
        x = self.features(x)     # variable spatial size
        x = self.pool(x)         # always (N, 128, 4, 4)
        x = x.view(x.size(0), -1)
        return self.classifier(x)

model = FlexibleCNN()

# Works with any input size!
for size in [32, 64, 128, 224]:
    x = torch.randn(1, 3, size, size)
    out = model(x)
    print(f"Input: {size}×{size} → Output: {out.shape}")
    # All produce: torch.Size([1, 10])
```

---

## 5️⃣ AdaptiveAvgPool2d — Global Average Pooling (GAP)

### The Most Important Adaptive Pooling: GAP

**Global Average Pooling** is `AdaptiveAvgPool2d(1)` — it averages the entire spatial extent of each channel down to a single value.

```
Input: (N, C, H, W)  ──►  AdaptiveAvgPool2d(1)  ──►  Output: (N, C, 1, 1)

For each channel c:
    output[c] = mean(input[c, :, :])  =  (1/HW) Σ_i Σ_j  input[c, i, j]
```

### ASCII Diagram

```
Feature Map (1 channel, 4×4):          After GAP (1×1):
┌────┬────┬────┬────┐                  ┌──────┐
│  2 │  4 │  6 │  8 │                  │      │
├────┼────┼────┼────┤                  │  5.0 │
│  1 │  3 │  5 │  7 │   GAP            │      │
├────┼────┼────┼────┤  ────────►       └──────┘
│  8 │  6 │  4 │  2 │
├────┼────┼────┼────┤   mean(all 16 values)
│  7 │  5 │  3 │  9 │   = (2+4+6+8+1+3+5+7+8+6+4+2+7+5+3+9)/16
└────┴────┴────┴────┘   = 80/16 = 5.0
```

### Why GAP Is Revolutionary (Network in Network, GoogLeNet, ResNet)

```
Traditional Classifier:                Modern Classifier (with GAP):
  Conv features: (N, 512, 7, 7)          Conv features: (N, 512, 7, 7)
       ↓                                      ↓
  Flatten: (N, 25088)                    GAP: (N, 512, 1, 1)
       ↓                                      ↓
  FC 25088→4096: 102M params!            Squeeze: (N, 512)
       ↓                                      ↓
  FC 4096→4096: 16.7M params            FC 512→1000: 0.5M params
       ↓                                      ↓
  FC 4096→1000: 4M params               Output: (N, 1000)
       ↓
  Output: (N, 1000)                      Total FC params: ~0.5M
                                         vs VGG: ~123M FC params
  Total FC params: ~123M!               >>> ~246× fewer parameters! <<<
```

### PyTorch GAP

```python
import torch
import torch.nn as nn

# Method 1: Module
gap = nn.AdaptiveAvgPool2d(1)
x = torch.randn(8, 512, 7, 7)
out = gap(x)                    # (8, 512, 1, 1)
out = out.view(out.size(0), -1) # (8, 512) — ready for FC layer

# Method 2: Functional
out = torch.nn.functional.adaptive_avg_pool2d(x, 1)

# Method 3: Manual (equivalent)
out = x.mean(dim=[2, 3])  # average over spatial dims → (8, 512)

# Modern classifier head
classifier = nn.Sequential(
    nn.AdaptiveAvgPool2d(1),  # (N, C, 1, 1)
    nn.Flatten(),              # (N, C)
    nn.Dropout(0.2),
    nn.Linear(512, 1000),      # (N, 1000)
)
```

---

## 6️⃣ Zero Learnable Parameters

### Key Property of Pooling Layers

Pooling operations are **completely deterministic** — they have no weights or biases to learn.

```
Layer Type           Learnable Parameters
─────────────────────────────────────────
Conv2d(64, 128, 3)   → 73,856 params
BatchNorm2d(128)     → 256 params
ReLU                 → 0 params
MaxPool2d(2, 2)      → 0 params     ← no parameters!
AvgPool2d(2, 2)      → 0 params     ← no parameters!
AdaptiveAvgPool2d(1) → 0 params     ← no parameters!
```

### Verify in PyTorch

```python
import torch.nn as nn

layers = {
    'MaxPool2d':         nn.MaxPool2d(2, 2),
    'AvgPool2d':         nn.AvgPool2d(2, 2),
    'AdaptiveMaxPool2d': nn.AdaptiveMaxPool2d(4),
    'AdaptiveAvgPool2d': nn.AdaptiveAvgPool2d(1),
}

for name, layer in layers.items():
    params = sum(p.numel() for p in layer.parameters())
    print(f"{name}: {params} learnable parameters")

# MaxPool2d: 0 learnable parameters
# AvgPool2d: 0 learnable parameters
# AdaptiveMaxPool2d: 0 learnable parameters
# AdaptiveAvgPool2d: 0 learnable parameters
```

### Implication

Since pooling has no parameters:
- **No learning happens** in pooling layers — they're fixed operations
- **No gradient w.r.t. parameters** — but gradients still flow through them to earlier layers
- Adding/removing pooling doesn't change model size, only computation and spatial resolution

---

## 7️⃣ Placement in Architecture

### Classic CNN Pattern

```
Input → [Conv → BN → ReLU → Conv → BN → ReLU → Pool] × N → GAP → FC → Output
         ─────────── Feature Block ────────────   ↓
                                                Spatial
                                               reduction

Typical spatial dimension progression:
  224 → 112 → 56 → 28 → 14 → 7 → 1 (GAP)
         ↑     ↑    ↑    ↑    ↑    ↑
       Pool  Pool  Pool  Pool Pool  GAP
       or     or    or    or   or
      stride stride stride stride stride
```

### Placement Rules of Thumb

| Guideline | Rationale |
|-----------|-----------|
| Pool **after** Conv+ReLU, not before | Let convolution extract features first |
| Pool after every **1-3 conv blocks** | Gradual reduction; too frequent = information loss |
| Use **MaxPool** in body (feature extraction) | Preserves strongest activations |
| Use **GAP** at the end (before classifier) | Eliminates FC parameter explosion |
| **Double channels** when pooling halves spatial dims | Keeps computational budget balanced |

### Example: VGG-16 Architecture Pooling Placement

```
Layer                  Output Shape     Pool?
──────────────────────────────────────────────
Input                  3 × 224 × 224
Conv3-64 × 2           64 × 224 × 224
MaxPool                64 × 112 × 112   ← Pool 1
Conv3-128 × 2          128 × 112 × 112
MaxPool                128 × 56 × 56    ← Pool 2
Conv3-256 × 3          256 × 56 × 56
MaxPool                256 × 28 × 28    ← Pool 3
Conv3-512 × 3          512 × 28 × 28
MaxPool                512 × 14 × 14    ← Pool 4
Conv3-512 × 3          512 × 14 × 14
MaxPool                512 × 7 × 7      ← Pool 5
Flatten + FC           4096
FC                     4096
FC                     1000
```

### Modern Trend: Strided Convolution Instead of Pooling

```python
# Traditional: Conv + Pool
traditional = nn.Sequential(
    nn.Conv2d(64, 128, 3, padding=1),  # spatial dims unchanged
    nn.ReLU(inplace=True),
    nn.MaxPool2d(2, 2),                 # halve spatial dims
)

# Modern: Strided Conv (ResNet, EfficientNet)
modern = nn.Sequential(
    nn.Conv2d(64, 128, 3, stride=2, padding=1),  # halve AND convolve
    nn.BatchNorm2d(128),
    nn.ReLU(inplace=True),
)
# Advantages: learnable downsampling, no information discarded by fixed rule
```

---

## 8️⃣ Backpropagation Through Pooling

### Max Pooling Backward Pass

The gradient flows **only to the position that had the maximum value** in the forward pass. All other positions receive zero gradient.

```
Forward (MaxPool, 2×2, stride 2):

Input:                          Output:
┌─────┬─────┬─────┬─────┐      ┌─────┬─────┐
│  1  │  3  │  2  │  4  │      │     │     │
├─────┼─────┼─────┼─────┤      │  5  │  4  │
│ [5] │  2  │  1  │  0  │ ───► ├─────┼─────┤
├─────┼─────┼─────┼─────┤      │     │     │
│  3  │  1  │ [7] │  2  │      │  3  │  7  │
├─────┼─────┼─────┼─────┤      └─────┴─────┘
│  0  │  2  │  4  │  6  │
└─────┴─────┴─────┴─────┘      [x] = max positions

Backward (gradient from output):

dL/d(output):                   dL/d(input):
┌─────┬─────┐                   ┌─────┬─────┬─────┬─────┐
│     │     │                   │  0  │  0  │  0  │ g₂  │
│ g₁  │ g₂  │                   ├─────┼─────┼─────┼─────┤
├─────┼─────┤       ───►        │ g₁  │  0  │  0  │  0  │
│     │     │                   ├─────┼─────┼─────┼─────┤
│ g₃  │ g₄  │                   │ g₃  │  0  │ g₄  │  0  │
└─────┴─────┘                   ├─────┼─────┼─────┼─────┤
                                │  0  │  0  │  0  │  0  │
Only max positions get          └─────┴─────┴─────┴─────┘
non-zero gradients!
```

### Average Pooling Backward Pass

The gradient is **divided equally** among all positions in the window:

```
Forward (AvgPool, 2×2, stride 2):

Input:                          Output:
┌─────┬─────┬─────┬─────┐      ┌──────┬──────┐
│  1  │  3  │  2  │  4  │      │      │      │
├─────┼─────┼─────┼─────┤      │ 2.75 │ 1.75 │
│  5  │  2  │  1  │  0  │ ───► ├──────┼──────┤
├─────┼─────┼─────┼─────┤      │      │      │
│  3  │  1  │  7  │  2  │      │ 1.50 │ 4.75 │
├─────┼─────┼─────┼─────┤      └──────┴──────┘
│  0  │  2  │  4  │  6  │
└─────┴─────┴─────┴────-┘

Backward:

dL/d(output):                   dL/d(input):
┌──────┬──────┐                 ┌──────┬──────┬──────┬──────┐
│      │      │                 │ g₁/4 │ g₁/4 │ g₂/4 │ g₂/4 │
│  g₁  │  g₂  │                ├──────┼──────┼──────┼──────┤
├──────┼──────┤     ───►        │ g₁/4 │ g₁/4 │ g₂/4 │ g₂/4 │
│      │      │                 ├──────┼──────┼──────┼──────┤
│  g₃  │  g₄  │                │ g₃/4 │ g₃/4 │ g₄/4 │ g₄/4 │
└──────┴──────┘                 ├──────┼──────┼──────┼──────┤
                                │ g₃/4 │ g₃/4 │ g₄/4 │ g₄/4 │
Each input position gets        └──────┴──────┴──────┴──────┘
gradient / (pool_size²)
```

### Mathematical Formulation

```
MaxPool backward:
    ∂L/∂x[i,j] = ┌ ∂L/∂y[m,n]    if x[i,j] was the max in window (m,n)
                  └ 0              otherwise

AvgPool backward:
    ∂L/∂x[i,j] = ∂L/∂y[m,n] / (k_h × k_w)    for all (i,j) in window (m,n)

Key difference:
  MaxPool → sparse gradient (only 1 path per window)
  AvgPool → distributed gradient (all paths equally)
```

### Verify Gradients in PyTorch

```python
import torch
import torch.nn as nn

# MaxPool gradient
x = torch.tensor([[[[1., 3., 2., 4.],
                     [5., 2., 1., 0.],
                     [3., 1., 7., 2.],
                     [0., 2., 4., 6.]]]], requires_grad=True)

max_pool = nn.MaxPool2d(2, 2)
out = max_pool(x)
loss = out.sum()
loss.backward()
print("MaxPool gradient:")
print(x.grad)
# tensor([[[[0., 0., 0., 1.],
#           [1., 0., 0., 0.],
#           [1., 0., 1., 0.],
#           [0., 0., 0., 0.]]]])
# Only max positions get gradient = 1

# AvgPool gradient
x2 = torch.tensor([[[[1., 3., 2., 4.],
                      [5., 2., 1., 0.],
                      [3., 1., 7., 2.],
                      [0., 2., 4., 6.]]]], requires_grad=True)

avg_pool = nn.AvgPool2d(2, 2)
out2 = avg_pool(x2)
loss2 = out2.sum()
loss2.backward()
print("\nAvgPool gradient:")
print(x2.grad)
# tensor([[[[0.25, 0.25, 0.25, 0.25],
#           [0.25, 0.25, 0.25, 0.25],
#           [0.25, 0.25, 0.25, 0.25],
#           [0.25, 0.25, 0.25, 0.25]]]])
# All positions get gradient = 1/4
```

---

## 9️⃣ Complete Worked Example — Pooling in a CNN

```python
import torch
import torch.nn as nn

class PoolingDemoCNN(nn.Module):
    """CNN demonstrating all pooling types."""
    
    def __init__(self, num_classes=10, pool_type='max'):
        super().__init__()
        
        # Feature extraction with configurable pooling
        if pool_type == 'max':
            Pool = lambda: nn.MaxPool2d(2, 2)
        elif pool_type == 'avg':
            Pool = lambda: nn.AvgPool2d(2, 2)
        else:
            raise ValueError(f"Unknown pool type: {pool_type}")
        
        self.features = nn.Sequential(
            # Block 1: 32×32 → 16×16
            nn.Conv2d(3, 32, 3, padding=1, bias=False),
            nn.BatchNorm2d(32),
            nn.ReLU(inplace=True),
            nn.Conv2d(32, 32, 3, padding=1, bias=False),
            nn.BatchNorm2d(32),
            nn.ReLU(inplace=True),
            Pool(),
            
            # Block 2: 16×16 → 8×8
            nn.Conv2d(32, 64, 3, padding=1, bias=False),
            nn.BatchNorm2d(64),
            nn.ReLU(inplace=True),
            nn.Conv2d(64, 64, 3, padding=1, bias=False),
            nn.BatchNorm2d(64),
            nn.ReLU(inplace=True),
            Pool(),
            
            # Block 3: 8×8 → 4×4
            nn.Conv2d(64, 128, 3, padding=1, bias=False),
            nn.BatchNorm2d(128),
            nn.ReLU(inplace=True),
            Pool(),
        )
        
        # Classifier with GAP
        self.classifier = nn.Sequential(
            nn.AdaptiveAvgPool2d(1),   # (N, 128, 1, 1)
            nn.Flatten(),              # (N, 128)
            nn.Linear(128, num_classes),
        )
    
    def forward(self, x):
        x = self.features(x)
        x = self.classifier(x)
        return x

# Test
model = PoolingDemoCNN(num_classes=10, pool_type='max')
x = torch.randn(4, 3, 32, 32)
out = model(x)
print(f"Output shape: {out.shape}")  # [4, 10]

# Count parameters (pooling adds ZERO)
total_params = sum(p.numel() for p in model.parameters())
print(f"Total parameters: {total_params:,}")

# Print layer-by-layer shapes
def print_shapes(model, x):
    print(f"{'Layer':<40} {'Output Shape':<25}")
    print("=" * 65)
    for name, layer in model.named_modules():
        if isinstance(layer, (nn.Conv2d, nn.MaxPool2d, nn.AvgPool2d, 
                              nn.AdaptiveAvgPool2d, nn.Flatten, nn.Linear)):
            x = layer(x)
            print(f"{name:<40} {str(list(x.shape)):<25}")
    return x

# This prints the spatial dimension reduction at each stage
```

### Output Shape Tracking

```
For CIFAR-10 (32×32 input):

Layer               Output Shape          Params      Pool?
────────────────────────────────────────────────────────────
Input               (N, 3, 32, 32)        -
Conv3-32            (N, 32, 32, 32)       864
Conv3-32            (N, 32, 32, 32)       9,216
MaxPool 2×2         (N, 32, 16, 16)       0           ← Pool
Conv3-64            (N, 64, 16, 16)       18,432
Conv3-64            (N, 64, 16, 16)       36,864
MaxPool 2×2         (N, 64, 8, 8)         0           ← Pool
Conv3-128           (N, 128, 8, 8)        73,728
MaxPool 2×2         (N, 128, 4, 4)        0           ← Pool
AdaptiveAvgPool     (N, 128, 1, 1)        0           ← GAP
Flatten             (N, 128)              -
Linear 128→10       (N, 10)               1,290
```

---

## 🔟 Pooling vs Strided Convolution

### Comparison

| Feature | MaxPool2d | Strided Conv2d |
|---------|----------|----------------|
| **Parameters** | 0 | C_out × C_in × k² |
| **Learnable** | No | Yes |
| **Information loss** | Discards non-max values | Learns what to keep |
| **Gradient flow** | Sparse (max only) | Dense (all positions) |
| **Usage** | Classic CNNs (VGG, early ResNet) | Modern CNNs (ResNetv2, EfficientNet) |
| **Compute cost** | Very low | Same as regular conv |

### When to Use Each

```
Use MaxPool when:
  ✓ You want a simple, parameter-free downsampling
  ✓ Building a classic architecture (VGG-style)
  ✓ You need translation invariance
  ✓ Memory/compute is very limited

Use Strided Conv when:
  ✓ You want the network to LEARN how to downsample
  ✓ Building modern architectures
  ✓ You have enough data to train the extra parameters
  ✓ You want smoother gradient flow

Use GAP when:
  ✓ Replacing the heavy FC layers at the end
  ✓ You want spatial invariance in the classifier
  ✓ Building lightweight models
```

---

## 📊 Summary Table

| Pooling Type | Operation | Parameters | Output Formula | Use Case |
|-------------|-----------|------------|----------------|----------|
| **MaxPool2d** | max in window | 0 | `⌊(H+2p−k)/s + 1⌋` | Feature extraction body |
| **AvgPool2d** | mean in window | 0 | `⌊(H+2p−k)/s + 1⌋` | Less common in body |
| **AdaptiveMaxPool2d** | max, auto-sized | 0 | Specified by user | Flexible input sizes |
| **AdaptiveAvgPool2d** | mean, auto-sized | 0 | Specified by user | **GAP** — modern classifier |
| **Strided Conv** | learned downsampling | C²k² | `⌊(H+2p−k)/s + 1⌋` | Modern alternative to pool |

| Property | MaxPool | AvgPool |
|----------|---------|---------|
| Gradient flow | Sparse (max only) | Uniform (divided equally) |
| Feature preserved | Strongest activation | Average activation |
| Invariance | More translation invariant | Less invariant |
| Typical placement | Body of CNN | End of CNN (GAP) |

---

## ❓ Revision Questions

### Q1: Output Shape Calculation
**Input shape is `(1, 128, 14, 14)`. What is the output shape after `nn.MaxPool2d(kernel_size=2, stride=2)`?**

<details>
<summary>Answer</summary>

H_out = ⌊(14 + 0 − 2) / 2 + 1⌋ = ⌊12/2 + 1⌋ = 7
W_out = 7 (same calculation)
Channels unchanged.

Output shape: **(1, 128, 7, 7)**
</details>

### Q2: MaxPool vs AvgPool Gradients
**During backpropagation through a 2×2 MaxPool window, how many of the 4 input positions receive a non-zero gradient? How about AvgPool?**

<details>
<summary>Answer</summary>

**MaxPool**: Only **1 out of 4** positions receives a non-zero gradient — the position that held the maximum value during the forward pass. The gradient at that position equals the upstream gradient; the other 3 positions get zero.

**AvgPool**: All **4 out of 4** positions receive a non-zero gradient. Each position receives `upstream_gradient / 4` (the gradient is divided equally among all positions in the window).
</details>

### Q3: GAP vs Flatten+FC
**A feature map of shape `(N, 512, 7, 7)` is fed into a classifier. Compare the parameter count of (a) Flatten + Linear(25088, 1000) vs (b) AdaptiveAvgPool2d(1) + Linear(512, 1000).**

<details>
<summary>Answer</summary>

**(a) Flatten + FC:**
- Flatten: 512 × 7 × 7 = 25,088 features
- Linear: 25,088 × 1,000 + 1,000 = **25,089,000 parameters**

**(b) GAP + FC:**
- GAP: 512 × 1 × 1 = 512 features (0 params for GAP itself)
- Linear: 512 × 1,000 + 1,000 = **513,000 parameters**

GAP reduces classifier parameters by **~49×** (25M → 0.5M). This is why modern architectures universally use GAP.
</details>

### Q4: Adaptive Pooling
**Why is `AdaptiveAvgPool2d` useful for transfer learning when the input image size differs from the original training size?**

<details>
<summary>Answer</summary>

The convolutional layers of a pretrained model (e.g., ResNet trained on 224×224 images) produce feature maps whose spatial size depends on the input. If you feed a 320×320 image, the feature maps will be larger than expected, and a fixed `MaxPool2d` or `Flatten` would produce the wrong dimensions for the FC layer.

`AdaptiveAvgPool2d(1)` **always** outputs `(N, C, 1, 1)` regardless of the input spatial size. This means:
- The classifier (FC layer) always receives a fixed-size vector
- Any input resolution works without architecture changes
- The pretrained convolutional weights remain valid
</details>

### Q5: Pooling Parameter Count
**A CNN has 10 MaxPool2d layers and 5 AdaptiveAvgPool2d layers. How many learnable parameters do all these pooling layers contribute?**

<details>
<summary>Answer</summary>

**Zero.** Pooling layers have no learnable parameters — they perform fixed operations (max or average). All 15 pooling layers together contribute exactly **0 parameters** to the model.
</details>

### Q6: Architecture Design
**In a ResNet-style architecture, pooling is used only twice: once at the beginning (MaxPool after the first 7×7 conv) and once at the end (GAP). All other spatial reductions use stride-2 convolutions. Why is this design preferred over using multiple MaxPool layers?**

<details>
<summary>Answer</summary>

1. **Learnable downsampling**: Stride-2 convolutions learn *how* to reduce spatial dimensions, rather than using a fixed max/avg rule. The network can learn to preserve important information.

2. **Better gradient flow**: MaxPool creates sparse gradients (only the max position receives gradient). Stride-2 conv has dense gradients flowing to all positions, enabling better training of deep networks.

3. **Combined operation**: Stride-2 conv performs feature extraction and downsampling simultaneously, reducing the total number of layers and making the network more computationally efficient.

4. **Residual connections**: In ResNet, the shortcut connections need spatial matching. Using stride-2 conv makes it easy to add a matching 1×1 stride-2 conv on the shortcut path.
</details>

---

[← Previous: 02 — Activation Layer](02-activation-layer.md) | [📑 Unit Overview](../README.md) | [Next → 04 — Fully Connected Layer](04-fully-connected-layer.md)
