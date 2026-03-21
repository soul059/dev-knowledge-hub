# Multiple Output Channels — Building the Feature Volume

> **Chapter 2.7 · Convolution Operation**
> Understand how multiple filters create multiple output channels, forming the
> complete output volume of a convolution layer.

---

## Table of Contents

1. [From One Filter to Many](#1-from-one-filter-to-many)
2. [How Multiple Filters Work](#2-how-multiple-filters-work)
3. [Output Volume Shape](#3-output-volume-shape)
4. [Parameter Count](#4-parameter-count)
5. [Worked Example: Multiple Filters](#5-worked-example-multiple-filters)
6. [The Full Convolution Layer Pipeline](#6-the-full-convolution-layer-pipeline)
7. [PyTorch nn.Conv2d Deep Dive](#7-pytorch-nnconv2d-deep-dive)
8. [Real Architecture Analysis](#8-real-architecture-analysis)
9. [Applications](#9-applications)
10. [Summary Table](#10-summary-table)
11. [Revision Questions](#11-revision-questions)

---

## 1. From One Filter to Many

In the previous chapter, we saw that one 3D filter (K×K×C_in) produces one 2D
feature map. But a single feature map can only detect **one type of feature**.

To detect **multiple features** (edges, corners, textures, colors, etc.),
we need **multiple filters**.

```
    ONE filter → ONE feature map
    
    ┌──────────┐     ┌────────┐     ┌──────────┐
    │ Input    │ ⊛   │ Filter │  =  │Feature   │
    │ H×W×C_in│     │K×K×C_in│     │Map (H'×W')│
    └──────────┘     └────────┘     └──────────┘
    
    
    C_out filters → C_out feature maps (STACKED INTO A VOLUME)
    
    ┌──────────┐     ┌────────┐     ┌──────────┐
    │          │ ⊛   │Filter 1│ →   │  Map 1   │─┐
    │          │ ⊛   │Filter 2│ →   │  Map 2   │ │
    │ Input    │ ⊛   │Filter 3│ →   │  Map 3   │ ├── H' × W' × C_out
    │ H×W×C_in│ ⊛   │  ...   │ →   │  ...     │ │
    │          │ ⊛   │Filter N│ →   │  Map N   │─┘
    └──────────┘     └────────┘     └──────────┘
```

---

## 2. How Multiple Filters Work

### Each Filter Is Independent

```
    Input (H × W × C_in)
         │
         ├──── ⊛ Filter 1 (K×K×C_in) + bias1 → Feature Map 1 (H'×W')
         │         detects vertical edges
         │
         ├──── ⊛ Filter 2 (K×K×C_in) + bias2 → Feature Map 2 (H'×W')
         │         detects horizontal edges
         │
         ├──── ⊛ Filter 3 (K×K×C_in) + bias3 → Feature Map 3 (H'×W')
         │         detects diagonal edges
         │
         ├──── ⊛ Filter 4 (K×K×C_in) + bias4 → Feature Map 4 (H'×W')
         │         detects color blobs
         │
         ⋮
         │
         └──── ⊛ Filter N (K×K×C_in) + biasN → Feature Map N (H'×W')
                   detects some learned pattern
         
    ALL filters share the SAME input
    Each filter has its OWN set of weights
    Each filter produces its OWN feature map
```

### Stacking Feature Maps

```
    Feature Map 1:   Feature Map 2:   Feature Map 3:   ...   Feature Map N:
    ┌──────────┐     ┌──────────┐     ┌──────────┐          ┌──────────┐
    │ vert.    │     │ horiz.   │     │ diag.    │          │ pattern  │
    │ edges    │     │ edges    │     │ edges    │          │ N        │
    │          │     │          │     │          │          │          │
    │ H' × W' │     │ H' × W' │     │ H' × W' │          │ H' × W' │
    └──────────┘     └──────────┘     └──────────┘          └──────────┘
         │                │                │                      │
         └────────────────┴────────────────┴──────────────────────┘
                                    │
                                    ▼
                          ┌───────────────────┐
                          │                   │
                          │  Output Volume    │
                          │  H' × W' × C_out │
                          │                   │
                          └───────────────────┘
                          
                          This becomes the INPUT
                          to the next conv layer!
```

---

## 3. Output Volume Shape

### Complete Shape Specification

```
    ┌────────────────────────────────────────────────────────────────┐
    │                                                                │
    │  Input:   (N, C_in, H, W)     — batch of N images             │
    │  Filters: C_out filters, each (C_in, K, K)                    │
    │  Output:  (N, C_out, H', W')  — batch of N feature volumes    │
    │                                                                │
    │  Where:                                                        │
    │    H' = ⌊(H - K + 2P) / S⌋ + 1                               │
    │    W' = ⌊(W - K + 2P) / S⌋ + 1                               │
    │    C_out = number of filters                                   │
    │                                                                │
    └────────────────────────────────────────────────────────────────┘
```

### Shape Flow Diagram

```
    Input Tensor:                Weight Tensor:               Output Tensor:
    ┌──────────────────┐        ┌──────────────────┐        ┌──────────────────┐
    │ N × C_in × H × W │  ⊛    │C_out×C_in×K×K    │  →     │N×C_out×H'×W'     │
    └──────────────────┘        └──────────────────┘        └──────────────────┘
    
    Example:
    (1, 3, 224, 224)    ⊛    (64, 3, 3, 3)    →    (1, 64, 224, 224)
    batch=1              64 filters             64 feature maps
    RGB input            each 3×3×3             each 224×224
    224×224              matching 3 input ch     (with padding=1)
```

### Common Configurations

| Layer | Input Shape | Filters | Kernel | S | P | Output Shape |
|-------|------------|---------|--------|---|---|-------------|
| Conv1 | 224×224×3 | 64 | 3×3 | 1 | 1 | 224×224×64 |
| Conv2 | 224×224×64 | 128 | 3×3 | 1 | 1 | 224×224×128 |
| Conv3 (downsample) | 224×224×128 | 256 | 3×3 | 2 | 1 | 112×112×256 |
| Conv4 | 112×112×256 | 512 | 3×3 | 1 | 1 | 112×112×512 |
| Conv5 (downsample) | 112×112×512 | 512 | 3×3 | 2 | 1 | 56×56×512 |

---

## 4. Parameter Count

### Formula

```
    ┌──────────────────────────────────────────────────────────────┐
    │                                                              │
    │  Total Parameters = K × K × C_in × C_out + C_out            │
    │                      \_________________/   \____/            │
    │                         weights              biases          │
    │                                                              │
    │  Weight tensor shape: (C_out, C_in, K, K)                    │
    │  Bias tensor shape:   (C_out,)                               │
    │                                                              │
    └──────────────────────────────────────────────────────────────┘
```

### Derivation

```
    Each filter has:
        K × K × C_in weights + 1 bias = K² × C_in + 1 parameters
    
    With C_out filters:
        Total = C_out × (K² × C_in + 1)
              = K² × C_in × C_out + C_out
    
    ┌──────────────────────────────────────────────────────┐
    │  Filter 1: K×K×C_in weights + 1 bias                │
    │  Filter 2: K×K×C_in weights + 1 bias                │
    │  Filter 3: K×K×C_in weights + 1 bias                │
    │  ...                                                 │
    │  Filter C_out: K×K×C_in weights + 1 bias             │
    │                                                      │
    │  Total weights: K×K×C_in×C_out                       │
    │  Total biases:  C_out                                │
    │  Grand total:   K×K×C_in×C_out + C_out               │
    └──────────────────────────────────────────────────────┘
```

### Parameter Count Examples

```
    ┌──────────────────────────────────────────────────────────────────┐
    │ Layer                    │ K │ C_in │ C_out │ Weights │ Bias │Total│
    ├──────────────────────────┼───┼──────┼───────┼─────────┼──────┼─────┤
    │ Conv1 (first layer)      │ 3 │   3  │   64  │  1,728  │  64  │1,792│
    │ Conv2 (early)            │ 3 │  64  │   64  │ 36,864  │  64  │36,928│
    │ Conv3 (mid)              │ 3 │  64  │  128  │ 73,728  │ 128  │73,856│
    │ Conv4 (mid)              │ 3 │ 128  │  256  │294,912  │ 256  │295,168│
    │ Conv5 (deep)             │ 3 │ 256  │  512  │1,179,648│ 512  │1,180,160│
    │ Conv6 (deep)             │ 3 │ 512  │  512  │2,359,296│ 512  │2,359,808│
    ├──────────────────────────┼───┼──────┼───────┼─────────┼──────┼─────┤
    │ ResNet 7×7 first layer   │ 7 │   3  │   64  │  9,408  │  64  │9,472│
    │ 1×1 bottleneck (reduce)  │ 1 │ 256  │   64  │ 16,384  │  64  │16,448│
    │ 1×1 bottleneck (expand)  │ 1 │  64  │  256  │ 16,384  │ 256  │16,640│
    └──────────────────────────┴───┴──────┴───────┴─────────┴──────┴─────┘
    
    Notice: Parameters grow with C_in × C_out (quadratic in channels!)
```

### FLOPs per Layer

```
    FLOPs = 2 × K² × C_in × C_out × H' × W'
    
    Example: Conv(64, 128, 3) with 56×56 output:
    FLOPs = 2 × 9 × 64 × 128 × 56 × 56
          = 2 × 9 × 64 × 128 × 3,136
          = 462,422,016 ≈ 462M FLOPs
```

---

## 5. Worked Example: Multiple Filters

### Setup

```
    Input (3×3×2 — 2 channels):
    
    Channel 0:          Channel 1:
    ┌───┬───┬───┐      ┌───┬───┬───┐
    │ 1 │ 2 │ 0 │      │ 0 │ 1 │ 2 │
    ├───┼───┼───┤      ├───┼───┼───┤
    │ 3 │ 1 │ 2 │      │ 1 │ 0 │ 1 │
    ├───┼───┼───┤      ├───┼───┼───┤
    │ 0 │ 2 │ 1 │      │ 2 │ 1 │ 0 │
    └───┴───┴───┘      └───┴───┴───┘
    
    Three filters (2×2×2 each), no padding, stride 1:
    
    Filter 1:                   Filter 2:                   Filter 3:
    Ch0:     Ch1:               Ch0:     Ch1:               Ch0:     Ch1:
    ┌───┬───┐┌───┬───┐         ┌───┬───┐┌───┬───┐         ┌───┬───┐┌───┬───┐
    │ 1 │ 0 ││ 0 │ 1 │         │ 0 │ 1 ││ 1 │ 0 │         │ 1 │ 1 ││ 1 │ 1 │
    ├───┼───┤├───┼───┤         ├───┼───┤├───┼───┤         ├───┼───┤├───┼───┤
    │ 0 │ 1 ││ 1 │ 0 │         │ 1 │ 0 ││ 0 │ 1 │         │ 1 │ 1 ││ 1 │ 1 │
    └───┴───┘└───┴───┘         └───┴───┘└───┴───┘         └───┴───┘└───┴───┘
    
    Bias: b1=0, b2=0, b3=0
```

### Output Size

```
    O = (3 - 2) / 1 + 1 = 2
    Output: 2 × 2 × 3  (3 feature maps, each 2×2)
```

### Filter 1 Computation

```
    Position (0,0):
    Ch0 patch: [1,2; 3,1]  ⊙  [1,0; 0,1] = 1+0+0+1 = 2
    Ch1 patch: [0,1; 1,0]  ⊙  [0,1; 1,0] = 0+1+1+0 = 2
    O₁(0,0) = 2 + 2 = 4
    
    Position (0,1):
    Ch0 patch: [2,0; 1,2]  ⊙  [1,0; 0,1] = 2+0+0+2 = 4
    Ch1 patch: [1,2; 0,1]  ⊙  [0,1; 1,0] = 0+2+0+0 = 2
    O₁(0,1) = 4 + 2 = 6
    
    Position (1,0):
    Ch0 patch: [3,1; 0,2]  ⊙  [1,0; 0,1] = 3+0+0+2 = 5
    Ch1 patch: [1,0; 2,1]  ⊙  [0,1; 1,0] = 0+0+2+0 = 2
    O₁(1,0) = 5 + 2 = 7
    
    Position (1,1):
    Ch0 patch: [1,2; 2,1]  ⊙  [1,0; 0,1] = 1+0+0+1 = 2
    Ch1 patch: [0,1; 1,0]  ⊙  [0,1; 1,0] = 0+1+1+0 = 2
    O₁(1,1) = 2 + 2 = 4
    
    Feature Map 1:
    ┌───┬───┐
    │ 4 │ 6 │
    ├───┼───┤
    │ 7 │ 4 │
    └───┴───┘
```

### Filter 2 Computation

```
    Position (0,0):
    Ch0: [1,2; 3,1] ⊙ [0,1; 1,0] = 0+2+3+0 = 5
    Ch1: [0,1; 1,0] ⊙ [1,0; 0,1] = 0+0+0+0 = 0
    O₂(0,0) = 5 + 0 = 5
    
    Position (0,1):
    Ch0: [2,0; 1,2] ⊙ [0,1; 1,0] = 0+0+1+0 = 1
    Ch1: [1,2; 0,1] ⊙ [1,0; 0,1] = 1+0+0+1 = 2
    O₂(0,1) = 1 + 2 = 3
    
    Position (1,0):
    Ch0: [3,1; 0,2] ⊙ [0,1; 1,0] = 0+1+0+0 = 1
    Ch1: [1,0; 2,1] ⊙ [1,0; 0,1] = 1+0+0+1 = 2
    O₂(1,0) = 1 + 2 = 3
    
    Position (1,1):
    Ch0: [1,2; 2,1] ⊙ [0,1; 1,0] = 0+2+2+0 = 4
    Ch1: [0,1; 1,0] ⊙ [1,0; 0,1] = 0+0+0+0 = 0
    O₂(1,1) = 4 + 0 = 4
    
    Feature Map 2:
    ┌───┬───┐
    │ 5 │ 3 │
    ├───┼───┤
    │ 3 │ 4 │
    └───┴───┘
```

### Filter 3 Computation (Averaging Filter — all 1s)

```
    Position (0,0):
    Ch0: [1,2; 3,1] ⊙ [1,1; 1,1] = 1+2+3+1 = 7
    Ch1: [0,1; 1,0] ⊙ [1,1; 1,1] = 0+1+1+0 = 2
    O₃(0,0) = 7 + 2 = 9
    
    Position (0,1):
    Ch0: [2,0; 1,2] ⊙ [1,1; 1,1] = 2+0+1+2 = 5
    Ch1: [1,2; 0,1] ⊙ [1,1; 1,1] = 1+2+0+1 = 4
    O₃(0,1) = 5 + 4 = 9
    
    Position (1,0):
    Ch0: [3,1; 0,2] ⊙ [1,1; 1,1] = 3+1+0+2 = 6
    Ch1: [1,0; 2,1] ⊙ [1,1; 1,1] = 1+0+2+1 = 4
    O₃(1,0) = 6 + 4 = 10
    
    Position (1,1):
    Ch0: [1,2; 2,1] ⊙ [1,1; 1,1] = 1+2+2+1 = 6
    Ch1: [0,1; 1,0] ⊙ [1,1; 1,1] = 0+1+1+0 = 2
    O₃(1,1) = 6 + 2 = 8
    
    Feature Map 3:
    ┌────┬───┐
    │  9 │ 9 │
    ├────┼───┤
    │ 10 │ 8 │
    └────┴───┘
```

### Final Output Volume

```
    Three feature maps stacked into the output volume:
    
    Map 1:        Map 2:        Map 3:         Stacked:
    ┌───┬───┐    ┌───┬───┐    ┌────┬───┐      ┌───┬───┐
    │ 4 │ 6 │    │ 5 │ 3 │    │  9 │ 9 │      │   │   │╲
    ├───┼───┤    ├───┼───┤    ├────┼───┤      ├───┼───┤ ╲
    │ 7 │ 4 │    │ 3 │ 4 │    │ 10 │ 8 │      │   │   │  │ C_out=3
    └───┴───┘    └───┴───┘    └────┴───┘      └───┴───┘  │
                                                └───┴───┘╱
    
    Output shape: 2 × 2 × 3
    
    Parameters: 3 filters × (2×2×2 weights + 1 bias) = 3 × 9 = 27
    Or: K²×C_in×C_out + C_out = 4×2×3 + 3 = 24 + 3 = 27
```

---

## 6. The Full Convolution Layer Pipeline

### Complete Data Flow

```
    ┌───────────────────────────────────────────────────────────────────┐
    │                    CONV LAYER PIPELINE                           │
    │                                                                   │
    │  Input           Convolution      Bias         Activation         │
    │  (N,C_in,H,W) → (weight tensor) → (+bias) → (ReLU) → Output     │
    │                                                    (N,C_out,H',W')│
    │                                                                   │
    │  Weight shape: (C_out, C_in, K, K)                               │
    │  Bias shape:   (C_out,)                                          │
    │                                                                   │
    │  Each of the C_out filters independently:                        │
    │  1. Convolves with all C_in input channels                       │
    │  2. Sums across channels                                         │
    │  3. Adds its bias                                                │
    │  4. ReLU zeros out negatives                                     │
    │  5. Produces one H'×W' feature map                               │
    └───────────────────────────────────────────────────────────────────┘
```

### Layer-to-Layer Connection

```
    Layer 1                    Layer 2                    Layer 3
    ┌────────────────┐        ┌────────────────┐        ┌────────────────┐
    │                │        │                │        │                │
    │ In: 3 channels │        │ In: 64 channels│        │ In: 128 ch.   │
    │ 64 filters     │        │ 128 filters    │        │ 256 filters   │
    │ Each: 3×3×3    │───────►│ Each: 3×3×64   │───────►│ Each: 3×3×128 │
    │                │        │                │        │                │
    │ Out: 64 ch.    │        │ Out: 128 ch.   │        │ Out: 256 ch.  │
    │                │        │                │        │                │
    └────────────────┘        └────────────────┘        └────────────────┘
    
    Params:                   Params:                   Params:
    3×3×3×64+64              3×3×64×128+128            3×3×128×256+256
    = 1,792                  = 73,856                  = 295,168
```

---

## 7. PyTorch nn.Conv2d Deep Dive

### Constructor Parameters

```python
torch.nn.Conv2d(
    in_channels,   # C_in: number of input channels
    out_channels,  # C_out: number of output channels (= number of filters)
    kernel_size,   # K: spatial size of kernel (int or tuple)
    stride=1,      # S: stride (int or tuple)
    padding=0,     # P: zero-padding (int or tuple)
    dilation=1,    # d: dilation rate
    groups=1,      # G: grouped convolution
    bias=True,     # whether to include bias
    padding_mode='zeros'  # 'zeros', 'reflect', 'replicate', 'circular'
)
```

### Internal Tensor Shapes

```python
import torch
import torch.nn as nn

conv = nn.Conv2d(in_channels=3, out_channels=64, kernel_size=3, 
                 stride=1, padding=1, bias=True)

# Weight tensor: (C_out, C_in, K_h, K_w)
print(f"Weight shape: {conv.weight.shape}")
# torch.Size([64, 3, 3, 3])

# Bias tensor: (C_out,)
print(f"Bias shape: {conv.bias.shape}")
# torch.Size([64])

# Total parameters
total = conv.weight.numel() + conv.bias.numel()
print(f"Total params: {total}")
# 64 × 3 × 3 × 3 + 64 = 1,728 + 64 = 1,792
```

### Understanding the Weight Tensor Layout

```
    conv.weight.shape = (C_out, C_in, K_h, K_w)
    
    ┌───────────────────────────────────────────────────────┐
    │  conv.weight[0]  →  Filter 0: shape (C_in, K, K)    │
    │  conv.weight[1]  →  Filter 1: shape (C_in, K, K)    │
    │  conv.weight[2]  →  Filter 2: shape (C_in, K, K)    │
    │  ...                                                  │
    │  conv.weight[63] →  Filter 63: shape (C_in, K, K)   │
    └───────────────────────────────────────────────────────┘
    
    conv.weight[f, c, :, :] = The 2D slice of filter f for channel c
    
    Example:
    conv.weight[0, 0, :, :] → Filter 0, Red channel slice (3×3)
    conv.weight[0, 1, :, :] → Filter 0, Green channel slice (3×3)
    conv.weight[0, 2, :, :] → Filter 0, Blue channel slice (3×3)
```

### Complete Working Example

```python
import torch
import torch.nn as nn

# ── Create a complete convolution layer ──
conv = nn.Conv2d(
    in_channels=3,    # RGB input
    out_channels=8,   # 8 different filters
    kernel_size=3,    # 3×3 spatial kernel
    stride=1,
    padding=1,        # "same" padding
    bias=True
)

# Create a sample RGB image batch
batch = torch.randn(4, 3, 32, 32)  # 4 images, RGB, 32×32

# Forward pass
output = conv(batch)

print(f"Input shape:  {batch.shape}")    # (4, 3, 32, 32)
print(f"Output shape: {output.shape}")   # (4, 8, 32, 32)
print(f"Weight shape: {conv.weight.shape}")  # (8, 3, 3, 3)
print(f"Bias shape:   {conv.bias.shape}")    # (8,)

# ── Parameter count ──
weights = conv.weight.numel()    # 8 × 3 × 3 × 3 = 216
biases = conv.bias.numel()       # 8
print(f"\nWeights: {weights}")
print(f"Biases:  {biases}")
print(f"Total:   {weights + biases}")   # 224
```

### Manual Multi-Output Convolution

```python
import torch
import torch.nn.functional as F

# Input: 4×4 with 2 channels
x = torch.tensor([
    [[1, 2, 0, 1], [3, 1, 2, 0], [0, 2, 1, 3], [1, 0, 2, 1]],  # Ch 0
    [[0, 1, 2, 0], [1, 0, 1, 2], [2, 1, 0, 1], [0, 2, 1, 0]],  # Ch 1
], dtype=torch.float32).unsqueeze(0)  # (1, 2, 4, 4)

# 3 filters, each 3×3×2
filters = torch.tensor([
    # Filter 0 (vertical edge detector)
    [[[ 1, 0, -1], [ 1, 0, -1], [ 1, 0, -1]],   # Ch 0 slice
     [[ 1, 0, -1], [ 1, 0, -1], [ 1, 0, -1]]],  # Ch 1 slice
    # Filter 1 (horizontal edge detector)
    [[[ 1,  1,  1], [ 0,  0,  0], [-1, -1, -1]],
     [[ 1,  1,  1], [ 0,  0,  0], [-1, -1, -1]]],
    # Filter 2 (identity-like, center only)
    [[[ 0, 0, 0], [ 0, 1, 0], [ 0, 0, 0]],
     [[ 0, 0, 0], [ 0, 1, 0], [ 0, 0, 0]]],
], dtype=torch.float32)  # (3, 2, 3, 3)

bias = torch.zeros(3)

output = F.conv2d(x, filters, bias=bias)

print(f"Input shape:   {x.shape}")       # (1, 2, 4, 4)
print(f"Filters shape: {filters.shape}") # (3, 2, 3, 3)
print(f"Output shape:  {output.shape}")  # (1, 3, 2, 2)

print("\nFeature Map 0 (vertical edges):")
print(output[0, 0])
print("\nFeature Map 1 (horizontal edges):")
print(output[0, 1])
print("\nFeature Map 2 (center values):")
print(output[0, 2])
```

### Layer Analysis Utility

```python
import torch
import torch.nn as nn

def analyze_conv_layer(layer: nn.Conv2d, input_shape: tuple):
    """Analyze a conv layer's shapes and parameter count."""
    N, C_in_input, H, W = input_shape
    
    C_out = layer.out_channels
    C_in = layer.in_channels
    K = layer.kernel_size[0] if isinstance(layer.kernel_size, tuple) else layer.kernel_size
    S = layer.stride[0] if isinstance(layer.stride, tuple) else layer.stride
    P = layer.padding[0] if isinstance(layer.padding, tuple) else layer.padding
    
    H_out = (H - K + 2*P) // S + 1
    W_out = (W - K + 2*P) // S + 1
    
    weights = K * K * C_in * C_out
    biases = C_out if layer.bias is not None else 0
    flops = 2 * K * K * C_in * C_out * H_out * W_out
    
    print(f"┌{'─'*50}┐")
    print(f"│ Conv2d({C_in}, {C_out}, {K}×{K}, S={S}, P={P})")
    print(f"├{'─'*50}┤")
    print(f"│ Input shape:    ({N}, {C_in_input}, {H}, {W})")
    print(f"│ Output shape:   ({N}, {C_out}, {H_out}, {W_out})")
    print(f"│ Weight shape:   ({C_out}, {C_in}, {K}, {K})")
    print(f"│ Filter shape:   ({C_in}, {K}, {K}) × {C_out} filters")
    print(f"│ Parameters:     {weights:,} weights + {biases:,} biases = {weights+biases:,}")
    print(f"│ FLOPs:          {flops:,} ({flops/1e6:.1f}M)")
    print(f"│ Output volume:  {C_out} × {H_out} × {W_out} = {C_out*H_out*W_out:,} values")
    print(f"└{'─'*50}┘")
    
    return (N, C_out, H_out, W_out)

# ── Analyze a sequence of layers ──
print("=== VGG-style Network Analysis ===\n")
layers = [
    nn.Conv2d(3, 64, 3, padding=1),
    nn.Conv2d(64, 64, 3, padding=1),
    nn.Conv2d(64, 128, 3, stride=2, padding=1),
    nn.Conv2d(128, 128, 3, padding=1),
    nn.Conv2d(128, 256, 3, stride=2, padding=1),
    nn.Conv2d(256, 256, 3, padding=1),
    nn.Conv2d(256, 512, 3, stride=2, padding=1),
]

shape = (1, 3, 224, 224)
for i, layer in enumerate(layers):
    print(f"\nLayer {i+1}:")
    shape = analyze_conv_layer(layer, shape)
```

---

## 8. Real Architecture Analysis

### VGG-16 Parameter Breakdown

```
    Layer          Config            Params          Cumulative
    ──────────────────────────────────────────────────────────
    conv1_1        Conv(3,64,3)        1,792            1,792
    conv1_2        Conv(64,64,3)      36,928           38,720
    ── MaxPool ──
    conv2_1        Conv(64,128,3)     73,856          112,576
    conv2_2        Conv(128,128,3)   147,584          260,160
    ── MaxPool ──
    conv3_1        Conv(128,256,3)   295,168          555,328
    conv3_2        Conv(256,256,3)   590,080        1,145,408
    conv3_3        Conv(256,256,3)   590,080        1,735,488
    ── MaxPool ──
    conv4_1        Conv(256,512,3) 1,180,160        2,915,648
    conv4_2        Conv(512,512,3) 2,359,808        5,275,456
    conv4_3        Conv(512,512,3) 2,359,808        7,635,264
    ── MaxPool ──
    conv5_1        Conv(512,512,3) 2,359,808        9,995,072
    conv5_2        Conv(512,512,3) 2,359,808       12,354,880
    conv5_3        Conv(512,512,3) 2,359,808       14,714,688
    ──────────────────────────────────────────────────────────
    FC layers                     123,642,880      138,357,568
    
    Total: ~138M parameters
    Conv layers: ~14.7M (only 11% of total!)
    FC layers: ~123.6M (89% of total!)
```

### ResNet-50 Channel Progression

```
    ┌──────────────────────────────────────────────────────────┐
    │ Stage   │ Input     │ Output    │ Spatial │ C_out        │
    ├─────────┼───────────┼───────────┼─────────┼──────────────┤
    │ Input   │ 224×224×3 │           │ 224×224 │ 3            │
    │ Conv1   │ 224×224×3 │ 112×112×64│ 112×112 │ 64           │
    │ Pool    │ 112×112×64│ 56×56×64  │ 56×56   │ 64           │
    │ Stage 2 │ 56×56×64  │ 56×56×256 │ 56×56   │ 256 ←──────┐│
    │ Stage 3 │ 56×56×256 │ 28×28×512 │ 28×28   │ 512   ×2   ││
    │ Stage 4 │ 28×28×512 │ 14×14×1024│ 14×14   │ 1024  ×2   ││
    │ Stage 5 │ 14×14×1024│ 7×7×2048  │ 7×7     │ 2048  ×2   ││
    │ AvgPool │ 7×7×2048  │ 1×1×2048  │ 1×1     │ 2048       ││
    │ FC      │ 2048      │ 1000      │ -       │ 1000       ││
    └─────────┴───────────┴───────────┴─────────┴──────────────┘
    
    Pattern: Spatial dims halve, channels double at each stage
    
    Spatial:  224 → 112 → 56 → 28 → 14 → 7 → 1
    Channels: 3  →  64  → 256 → 512 → 1024 → 2048
```

---

## 9. Applications

| Application | Typical C_out Values | Why |
|-------------|---------------------|-----|
| **MNIST classification** | 32, 64 | Simple patterns, small images |
| **CIFAR-10** | 64, 128, 256 | More complex patterns |
| **ImageNet classification** | 64, 128, 256, 512, 1024, 2048 | Rich visual hierarchy |
| **Object detection** | 256, 512 | Need diverse feature maps |
| **Semantic segmentation** | 64→512 (encoder), 512→1 (decoder) | Pixel-level features |
| **Style transfer** | 64, 128, 256, 512 | Content + style features |
| **Super-resolution** | 64, 128, 64, 3 | Upsample with learned features |

### How Many Filters to Use?

```
    General Guidelines:
    
    ┌────────────────────────────────────────────────────────┐
    │ 1. Double channels when halving spatial dimensions     │
    │    224×224×64 → 112×112×128 → 56×56×256               │
    │                                                        │
    │ 2. Start with 32-64 filters in the first layer         │
    │    (don't need too many for simple edge detection)     │
    │                                                        │
    │ 3. Cap at 512-2048 in the deepest layers               │
    │    (more doesn't help much; too many → overfitting)    │
    │                                                        │
    │ 4. Use powers of 2 for hardware efficiency             │
    │    32, 64, 128, 256, 512, 1024, 2048                  │
    │                                                        │
    │ 5. For lighter models, use fewer:                      │
    │    MobileNet: 32, 64, 128, 256, 512, 1024             │
    │    EfficientNet: scaled by compound coefficient        │
    └────────────────────────────────────────────────────────┘
```

### Information Flow in CNNs

```
    Input Image                           Output Classes
    ┌────────────┐                        ┌──────────┐
    │ 224×224×3  │                        │  1000    │
    │ = 150,528  │                        │ classes  │
    │ raw pixels │                        │          │
    └──────┬─────┘                        └──────┬───┘
           │                                     │
    ┌──────▼──────────────────────────────────────▼───┐
    │ INFORMATION COMPRESSION THROUGH THE NETWORK     │
    │                                                  │
    │ Stage 1: 224×224×64  = 3,211,264 values         │
    │ Stage 2:  56×56×256  =   802,816 values         │
    │ Stage 3:  28×28×512  =   401,408 values         │
    │ Stage 4:  14×14×1024 =   200,704 values         │
    │ Stage 5:   7×7×2048  =   100,352 values         │
    │ AvgPool:   1×1×2048  =     2,048 values         │
    │ Output:         1000 =     1,000 values         │
    │                                                  │
    │ From 150K raw pixels → 1K class probabilities   │
    │ = 150× compression of information               │
    └──────────────────────────────────────────────────┘
```

---

## 10. Summary Table

| Concept | Description |
|---------|-------------|
| **Multiple output channels** | Multiple filters = multiple feature maps |
| **C_out** | Number of filters = number of output channels |
| **Output volume shape** | H' × W' × C_out |
| **Weight tensor shape** | (C_out, C_in, K, K) in PyTorch |
| **Total parameters** | K² × C_in × C_out + C_out |
| **One filter** | Shape (C_in, K, K), produces one 2D feature map |
| **Filter bank** | C_out filters, each (C_in, K, K) |
| **Channel progression** | Typically doubles when spatial dims halve |
| **Powers of 2** | C_out = 32, 64, 128, 256, 512 for efficiency |
| **Layer connection** | C_out of layer N = C_in of layer N+1 |
| **nn.Conv2d** | `Conv2d(in_channels, out_channels, kernel_size)` |

---

## 11. Revision Questions

**Q1.** A Conv2d layer has in_channels=128, out_channels=256, kernel_size=3,
with bias. Calculate the total number of parameters.

<details>
<summary>Answer</summary>

```
Weights: 3 × 3 × 128 × 256 = 294,912
Biases:  256
Total:   294,912 + 256 = 295,168
```
</details>

**Q2.** An input tensor of shape (8, 64, 56, 56) passes through
Conv2d(64, 128, 3, stride=2, padding=1). What is the output shape?

<details>
<summary>Answer</summary>

```
Batch: 8 (unchanged)
Channels: 128 (= out_channels)
Spatial: ⌊(56 - 3 + 2×1) / 2⌋ + 1 = ⌊55/2⌋ + 1 = 27 + 1 = 28

Output shape: (8, 128, 28, 28)
```
</details>

**Q3.** Explain why the weight tensor in PyTorch's Conv2d has shape
(C_out, C_in, K, K). What does each dimension represent?

<details>
<summary>Answer</summary>

```
Dimension 0: C_out — Number of filters (one per output channel)
Dimension 1: C_in  — Each filter has C_in slices (matching input channels)
Dimension 2: K     — Kernel height
Dimension 3: K     — Kernel width

weight[f] = The complete 3D filter f, shape (C_in, K, K)
weight[f, c] = The 2D slice of filter f for input channel c, shape (K, K)
weight[f, c, i, j] = One specific weight value
```
</details>

**Q4.** VGG-16 has Conv(512, 512, 3) repeated three times. What are the parameters
for each such layer? Why does VGG-16 have so many parameters?

<details>
<summary>Answer</summary>

```
Each layer: 3×3×512×512 + 512 = 2,359,808 parameters
Three layers: 3 × 2,359,808 = 7,079,424

VGG-16 has many parameters because:
1. It uses 512 channels in deep layers (512² = 262,144 channel pairs)
2. It has 3 fully connected layers with 4096 neurons each (~120M params)
3. No bottleneck (1×1) layers to reduce channels
```
</details>

**Q5.** Design a 3-layer CNN that takes a 32×32×3 input and produces a
4×4×256 output. Specify in_channels, out_channels, kernel_size, stride,
and padding for each layer.

<details>
<summary>Answer</summary>

```
Layer 1: Conv2d(3, 64, 3, stride=2, padding=1)
         → ⌊(32-3+2)/2⌋+1 = 16 → Output: (64, 16, 16)

Layer 2: Conv2d(64, 128, 3, stride=2, padding=1)
         → ⌊(16-3+2)/2⌋+1 = 8  → Output: (128, 8, 8)

Layer 3: Conv2d(128, 256, 3, stride=2, padding=1)
         → ⌊(8-3+2)/2⌋+1 = 4   → Output: (256, 4, 4) ✓

Total params:
  Layer 1: 3×3×3×64+64 = 1,792
  Layer 2: 3×3×64×128+128 = 73,856
  Layer 3: 3×3×128×256+256 = 295,168
  Total: 370,816
```
</details>

**Q6.** What is the relationship between C_out and the number of unique
features a conv layer can detect? Can you have too many filters?

<details>
<summary>Answer</summary>

C_out determines the maximum number of distinct features the layer can detect.
Each filter learns to respond to a different pattern.

Yes, you can have too many filters:
- **Overfitting:** Too many filters on small datasets memorize noise
- **Redundancy:** Some filters may learn near-identical features
- **Computation:** More filters = more parameters and FLOPs
- **Diminishing returns:** Beyond a threshold, accuracy plateaus

Best practice: start with proven architectures (e.g., ResNet, EfficientNet)
and scale using techniques like compound scaling (EfficientNet).
</details>

---

## Navigation

| | |
|---|---|
| ⬅️ **Previous** | [Multiple Input Channels](./06-multiple-input-channels.md) |
| ➡️ **Next** | [Max Pooling](../03-Pooling-Layers/01-max-pooling.md) |
| 🏠 **Home** | [CNN Overview](../README.md) |

---

> *"Multiple output channels are what give CNNs their power — each filter a
> different question asked of the image, together painting a complete picture
> of visual understanding."*
