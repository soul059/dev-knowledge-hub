# EfficientNet — Compound Scaling for Superior Accuracy & Efficiency

> **Unit 6 · Modern CNN Architectures · Topic 1/6**

---

## Table of Contents

1. [Overview & Motivation](#1-overview--motivation)
2. [The Scaling Problem](#2-the-scaling-problem)
3. [Compound Scaling Method](#3-compound-scaling-method)
4. [EfficientNet-B0 Baseline Architecture](#4-efficientnet-b0-baseline-architecture)
5. [MBConv Block — The Core Building Block](#5-mbconv-block--the-core-building-block)
6. [Squeeze-and-Excitation (SE) Integration](#6-squeeze-and-excitation-se-integration)
7. [EfficientNet Family: B0 → B7](#7-efficientnet-family-b0--b7)
8. [Mathematical Formulation](#8-mathematical-formulation)
9. [Worked Example — Compound Scaling Computation](#9-worked-example--compound-scaling-computation)
10. [PyTorch / timm Implementation](#10-pytorch--timm-implementation)
11. [Accuracy vs Efficiency Analysis](#11-accuracy-vs-efficiency-analysis)
12. [Summary Table](#12-summary-table)
13. [Revision Questions](#13-revision-questions)
14. [Navigation](#14-navigation)

---

## 1. Overview & Motivation

Traditional CNN design scaled networks along **one dimension at a time** — deeper (more layers), wider (more channels), or higher resolution (bigger inputs). EfficientNet (Tan & Le, 2019) demonstrated that **scaling all three dimensions simultaneously** with a principled ratio yields dramatically better accuracy-efficiency trade-offs.

**Key insight:** If the input image is bigger, the network needs more layers (to increase receptive field) and more channels (to capture finer-grained patterns at the higher resolution).

```
Traditional Scaling (one dimension):
┌─────────────────────────────────────────────┐
│  Depth Only     Width Only    Resolution Only│
│                                              │
│  ┌──┐          ┌──────┐      ┌──┐           │
│  │  │          │      │      │  │  224→300   │
│  │  │          │      │      │  │           │
│  │  │ +layers  │      │ +ch  │  │           │
│  │  │          │      │      └──┘           │
│  │  │          └──────┘                      │
│  └──┘                                        │
│                                              │
│  Compound Scaling (all three):               │
│  ┌────────┐                                  │
│  │        │  +layers, +channels, +resolution │
│  │        │                                  │
│  │        │  → Best accuracy/FLOP trade-off  │
│  │        │                                  │
│  └────────┘                                  │
└─────────────────────────────────────────────┘
```

---

## 2. The Scaling Problem

A CNN can be represented as a sequence of stages, where each stage `i` has:
- **Depth** `dᵢ` (number of layers)
- **Width** `wᵢ` (number of channels)
- **Resolution** `rᵢ` (spatial size of input feature maps)

The goal is to maximize accuracy for a given resource budget (FLOPs or latency):

```
max     Accuracy(N(d, w, r))
d,w,r

subject to: N(d, w, r).FLOPs ≤ target_FLOPs
            N(d, w, r).Memory ≤ target_Memory
```

### Why Single-Dimension Scaling Fails

| Scaling Dimension | Benefit | Problem |
|---|---|---|
| **Depth** (more layers) | Captures richer features | Vanishing gradients, diminishing returns past ~100 layers |
| **Width** (more channels) | Captures finer features | Fails to capture high-level features if too shallow |
| **Resolution** (bigger input) | Captures fine details | Diminishing accuracy gains without depth/width increase |

---

## 3. Compound Scaling Method

EfficientNet introduces a **compound coefficient φ** that uniformly scales all three dimensions:

```
depth:      d = α^φ
width:      w = β^φ  
resolution: r = γ^φ

Subject to:  α · β² · γ² ≈ 2    (resource constraint)
             α ≥ 1,  β ≥ 1,  γ ≥ 1
```

### Why α · β² · γ² ≈ 2?

- **Depth** scales FLOPs linearly (doubling layers ≈ doubles FLOPs)
- **Width** scales FLOPs quadratically (width appears in both input and output channels)
- **Resolution** scales FLOPs quadratically (spatial dimensions H × W)

So total FLOP increase ≈ `(α · β² · γ²)^φ ≈ 2^φ`

```
Compound Scaling Visualization:
                                                    
  φ=0 (B0)     φ=1 (B1)      φ=2 (B2)     ...    φ=7 (B7)
  ┌───┐        ┌─────┐       ┌───────┐           ┌───────────┐
  │   │        │     │       │       │           │           │
  │   │ 224    │     │ 240   │       │ 260       │           │ 600
  │   │        │     │       │       │           │           │
  └───┘        │     │       │       │           │           │
               └─────┘       │       │           │           │
                              └───────┘           │           │
                                                   └───────────┘
  d=1.0        d=1.2         d=1.44               d=4.3
  w=1.0        w=1.1         w=1.21               w=2.14
  r=224        r=240         r=260                 r=600
```

### Optimal Coefficients (found via grid search on B0)

| Parameter | Symbol | Value |
|---|---|---|
| Depth coefficient | α | 1.2 |
| Width coefficient | β | 1.1 |
| Resolution coefficient | γ | 1.15 |
| Constraint check | α · β² · γ² | 1.2 × 1.21 × 1.3225 ≈ 1.92 ≈ 2 |

---

## 4. EfficientNet-B0 Baseline Architecture

The baseline architecture was found using **Neural Architecture Search (NAS)** optimizing for both accuracy and FLOPs:

```
EfficientNet-B0 Architecture:
┌──────────────────────────────────────────────────────────────┐
│ Stage │ Operator       │ Resolution │ #Channels │ #Layers   │
├──────────────────────────────────────────────────────────────┤
│   1   │ Conv3×3        │ 224×224    │    32     │    1      │
│   2   │ MBConv1, k3×3  │ 112×112    │    16     │    1      │
│   3   │ MBConv6, k3×3  │ 112×112    │    24     │    2      │
│   4   │ MBConv6, k5×5  │  56×56     │    40     │    2      │
│   5   │ MBConv6, k3×3  │  28×28     │    80     │    3      │
│   6   │ MBConv6, k5×5  │  14×14     │   112     │    3      │
│   7   │ MBConv6, k5×5  │  14×14     │   192     │    4      │
│   8   │ MBConv6, k3×3  │   7×7      │   320     │    1      │
│   9   │ Conv1×1 + Pool │   7×7      │  1280     │    1      │
│       │ + FC           │   1×1      │  1000     │    1      │
└──────────────────────────────────────────────────────────────┘
  MBConv{t} = Mobile Inverted Bottleneck with expansion factor t
  k{n}×{n}  = kernel size
```

---

## 5. MBConv Block — The Core Building Block

MBConv (Mobile Inverted Bottleneck Convolution) is derived from MobileNetV2's inverted residual block, enhanced with SE attention.

```
MBConv Block (expansion factor t=6):

Input (H×W×C)
    │
    ▼
┌──────────────────┐
│  1×1 Conv (expand)│  C → t·C (e.g., 6C)
│  BatchNorm + Swish│
└────────┬─────────┘
         │
         ▼
┌──────────────────┐
│  k×k DepthwiseConv│  t·C → t·C  (groups = t·C)
│  BatchNorm + Swish│
└────────┬─────────┘
         │
         ▼
┌──────────────────┐
│  Squeeze-Excite  │  Channel attention (ratio = 1/4)
└────────┬─────────┘
         │
         ▼
┌──────────────────┐
│  1×1 Conv (proj.) │  t·C → C'
│  BatchNorm       │  (NO activation — linear bottleneck)
└────────┬─────────┘
         │
    ┌────┴──── Skip Connection (if stride=1 AND C=C')
    │    │     + Stochastic Depth (drop path)
    ▼    ▼
   Add(input, output)
         │
         ▼
    Output (H'×W'×C')
```

### Key Design Choices

| Feature | Rationale |
|---|---|
| **Inverted residual** | Expand → depthwise → project. Bottleneck at I/O, wide in the middle |
| **Linear bottleneck** | No activation on projection preserves information in low-dim space |
| **Swish activation** | `x · σ(x)` — smoother than ReLU, better gradient flow |
| **Stochastic depth** | Drop entire block with probability that increases with depth |

### Swish Activation Function

```
Swish(x) = x · σ(βx) = x · sigmoid(βx)

For EfficientNet, β = 1 (SiLU):

f(x) = x · sigmoid(x) = x / (1 + e^(-x))

           │        ╱
        1  │       ╱
           │      ╱
        0  ├─────╱──────── x
           │   ╱
      -0.3 │──╱  (minimum ≈ -0.278 at x ≈ -1.28)
           │╱
           │
```

---

## 6. Squeeze-and-Excitation (SE) Integration

SE modules learn **channel-wise attention weights**, allowing the network to emphasize informative channels and suppress less useful ones.

```
Squeeze-and-Excitation Module:

Input Feature Map: U ∈ ℝ^(H×W×C)
    │
    ▼
┌─────────────────────────┐
│  Global Average Pooling  │  (H×W×C) → (1×1×C)
│  z_c = (1/H·W) Σᵢ Σⱼ u_c(i,j)
└───────────┬─────────────┘
            │  "Squeeze" — aggregate spatial info
            ▼
┌─────────────────────────┐
│  FC₁ (C → C/r)         │  Reduce (r=4 in EfficientNet)
│  + Swish                │
├─────────────────────────┤
│  FC₂ (C/r → C)         │  Restore
│  + Sigmoid              │
└───────────┬─────────────┘
            │  "Excitation" — learn channel weights
            ▼
      s ∈ ℝ^C  (values between 0 and 1)
            │
            ▼
┌─────────────────────────┐
│  Scale: x̃_c = s_c · u_c │  Channel-wise multiplication
└───────────┬─────────────┘
            ▼
    Output: x̃ ∈ ℝ^(H×W×C)
```

### Mathematical Formulation

```
z = F_sq(u) = (1/(H·W)) · Σᵢ₌₁ᴴ Σⱼ₌₁ᵂ u(i, j)      — Global Average Pooling

s = F_ex(z, W) = σ(W₂ · δ(W₁ · z))                     — Two FC layers
    where W₁ ∈ ℝ^(C/r × C),  W₂ ∈ ℝ^(C × C/r)
    δ = Swish,  σ = Sigmoid

x̃_c = s_c · u_c                                          — Rescale
```

---

## 7. EfficientNet Family: B0 → B7

Each variant is obtained by applying compound scaling with increasing φ:

```
┌──────────┬─────┬───────────┬────────┬──────────┬────────────┬──────────┐
│  Model   │  φ  │Resolution │ Width  │  Depth   │ #Params(M) │ Top-1(%) │
├──────────┼─────┼───────────┼────────┼──────────┼────────────┼──────────┤
│ B0       │  0  │   224     │  1.0   │   1.0    │     5.3    │   77.1   │
│ B1       │  1  │   240     │  1.0   │   1.1    │     7.8    │   79.1   │
│ B2       │  2  │   260     │  1.1   │   1.2    │     9.2    │   80.1   │
│ B3       │  3  │   300     │  1.2   │   1.4    │    12.0    │   81.6   │
│ B4       │  4  │   380     │  1.4   │   1.8    │    19.0    │   82.9   │
│ B5       │  5  │   456     │  1.6   │   2.2    │    30.0    │   83.6   │
│ B6       │  6  │   528     │  1.8   │   2.6    │    43.0    │   84.0   │
│ B7       │  7  │   600     │  2.0   │   3.1    │    66.0    │   84.3   │
└──────────┴─────┴───────────┴────────┴──────────┴────────────┴──────────┘

Accuracy vs Parameters:

Top-1 %  84 │                              ·B7
         83 │                         ·B6
         82 │                    ·B5
         81 │               ·B4
         80 │          ·B3
         79 │     ·B2
         78 │  ·B1
         77 │·B0
             └─────────────────────────────
              5   10  15  20  30  40  50  66  #Params(M)
```

### EfficientNetV2 Improvements

EfficientNetV2 (Tan & Le, 2021) introduced:
- **Fused-MBConv** in early stages (replace depthwise + 1×1 with regular 3×3 conv for small feature maps)
- **Progressive learning** — start with small images, increase during training
- **Smaller expansion ratio** and **smaller kernel sizes** (3×3 instead of 5×5)
- **3-8× faster training** than B0-B7

---

## 8. Mathematical Formulation

### FLOP Computation for MBConv

For an MBConv block with expansion `t`, input channels `C`, output channels `C'`, spatial size `H×W`, kernel `k`:

```
FLOPs_expand    = H · W · 1 · 1 · C · (t · C)                = H·W·t·C²
FLOPs_depthwise = H · W · k · k · (t · C) · 1                = H·W·k²·t·C
FLOPs_SE        ≈ 2 · (t·C) · (t·C/r)                        ≈ 2·t²·C²/r  (negligible)
FLOPs_project   = H · W · 1 · 1 · (t · C) · C'               = H·W·t·C·C'

Total FLOPs ≈ H·W·C · (t·C + k²·t + t·C')
```

### Comparison: Standard Conv vs MBConv

```
Standard Conv:  H · W · k² · C · C'
MBConv:         H · W · (t·C² + k²·t·C + t·C·C')

Ratio = MBConv / Standard = t·(C/C' + k²/C' + 1) / k²

For typical values (C=C'=64, k=3, t=6):
  Standard:  H·W·9·64·64     = H·W·36,864
  MBConv:    H·W·(24576+3456+24576) = H·W·52,608

  → MBConv with t=6 can be MORE FLOPs per block
  → BUT EfficientNet uses fewer blocks overall,
     and the bottleneck structure reduces memory
```

---

## 9. Worked Example — Compound Scaling Computation

**Problem:** Given EfficientNet-B0 has depth `d₀=1`, width `w₀=1`, resolution `r₀=224`, and scaling coefficients α=1.2, β=1.1, γ=1.15, compute B3 (φ=3) specifications.

**Solution:**

```
Step 1: Compute depth multiplier
  d = α^φ = 1.2³ = 1.728
  Round to: 1.8 (nearest practical value)
  
Step 2: Compute width multiplier  
  w = β^φ = 1.1³ = 1.331
  Round to: 1.2 (nearest practical value)

Step 3: Compute resolution multiplier
  r_mult = γ^φ = 1.15³ = 1.521
  r = 224 × 1.521 = 340.7
  Round to nearest multiple of 8: 300 (official value)

Step 4: Verify resource constraint
  FLOPs increase ≈ (α · β² · γ²)^φ
                  = (1.2 × 1.21 × 1.3225)^3
                  = (1.919)^3
                  ≈ 7.07
  
  So B3 uses ~7× the FLOPs of B0

Step 5: Apply to each stage of B0
  Stage 3 of B0: 2 layers, 24 channels
  Stage 3 of B3: ceil(2 × 1.8) = 4 layers
                  round(24 × 1.2) = 28 → round to nearest 8 = 32 channels
```

**Result: B3 → 300×300 input, ~1.8× depth, ~1.2× width, ~12M params**

---

## 10. PyTorch / timm Implementation

### Using timm (recommended)

```python
import timm
import torch

# --- Load pretrained EfficientNet variants ---
model_b0 = timm.create_model('efficientnet_b0', pretrained=True, num_classes=10)
model_b3 = timm.create_model('efficientnet_b3', pretrained=True, num_classes=10)
model_v2 = timm.create_model('efficientnetv2_s', pretrained=True, num_classes=10)

# --- Check model specs ---
print(f"B0 params: {sum(p.numel() for p in model_b0.parameters()):,}")
print(f"B3 params: {sum(p.numel() for p in model_b3.parameters()):,}")

# --- Inference ---
x_b0 = torch.randn(1, 3, 224, 224)  # B0 resolution
x_b3 = torch.randn(1, 3, 300, 300)  # B3 resolution

with torch.no_grad():
    out_b0 = model_b0(x_b0)  # (1, 10)
    out_b3 = model_b3(x_b3)  # (1, 10)
```

### Using torchvision

```python
from torchvision.models import efficientnet_b0, EfficientNet_B0_Weights
from torchvision.models import efficientnet_b7, EfficientNet_B7_Weights

# Pretrained with latest weights
model = efficientnet_b0(weights=EfficientNet_B0_Weights.IMAGENET1K_V1)

# Modify classifier for custom task
import torch.nn as nn
model.classifier = nn.Sequential(
    nn.Dropout(p=0.2, inplace=True),
    nn.Linear(1280, 10),  # 10 classes
)
```

### Custom MBConv Block Implementation

```python
import torch
import torch.nn as nn

class SqueezeExcitation(nn.Module):
    def __init__(self, in_channels, reduction=4):
        super().__init__()
        reduced = max(1, in_channels // reduction)
        self.se = nn.Sequential(
            nn.AdaptiveAvgPool2d(1),
            nn.Conv2d(in_channels, reduced, 1),
            nn.SiLU(inplace=True),
            nn.Conv2d(reduced, in_channels, 1),
            nn.Sigmoid(),
        )

    def forward(self, x):
        return x * self.se(x)


class MBConvBlock(nn.Module):
    def __init__(self, in_ch, out_ch, kernel_size=3, stride=1,
                 expand_ratio=6, se_ratio=4, drop_path_rate=0.0):
        super().__init__()
        mid_ch = in_ch * expand_ratio
        self.use_residual = (stride == 1 and in_ch == out_ch)
        padding = kernel_size // 2

        layers = []
        # Expansion phase (skip if expand_ratio == 1)
        if expand_ratio != 1:
            layers.extend([
                nn.Conv2d(in_ch, mid_ch, 1, bias=False),
                nn.BatchNorm2d(mid_ch),
                nn.SiLU(inplace=True),
            ])

        # Depthwise convolution
        layers.extend([
            nn.Conv2d(mid_ch, mid_ch, kernel_size, stride, padding,
                      groups=mid_ch, bias=False),
            nn.BatchNorm2d(mid_ch),
            nn.SiLU(inplace=True),
        ])

        # Squeeze-and-Excitation
        layers.append(SqueezeExcitation(mid_ch, se_ratio))

        # Pointwise projection (linear bottleneck — no activation)
        layers.extend([
            nn.Conv2d(mid_ch, out_ch, 1, bias=False),
            nn.BatchNorm2d(out_ch),
        ])

        self.block = nn.Sequential(*layers)

        # Stochastic depth
        self.drop_path_rate = drop_path_rate

    def stochastic_depth(self, x):
        if not self.training or self.drop_path_rate == 0.0:
            return x
        keep_prob = 1 - self.drop_path_rate
        shape = (x.shape[0],) + (1,) * (x.ndim - 1)
        mask = torch.rand(shape, device=x.device) < keep_prob
        return x * mask / keep_prob

    def forward(self, x):
        out = self.block(x)
        if self.use_residual:
            out = self.stochastic_depth(out)
            out = out + x
        return out


# --- Test ---
block = MBConvBlock(in_ch=32, out_ch=32, kernel_size=3,
                     stride=1, expand_ratio=6)
x = torch.randn(2, 32, 56, 56)
print(block(x).shape)  # torch.Size([2, 32, 56, 56])
```

### Fine-Tuning EfficientNet

```python
import torch
import torch.nn as nn
import timm
from torch.optim import AdamW
from torch.optim.lr_scheduler import CosineAnnealingLR

# 1. Load pretrained model
model = timm.create_model('efficientnet_b3', pretrained=True, num_classes=100)

# 2. Freeze early layers (optional)
for name, param in model.named_parameters():
    if 'blocks.0' in name or 'blocks.1' in name or 'blocks.2' in name:
        param.requires_grad = False

# 3. Discriminative learning rates
backbone_params = [p for n, p in model.named_parameters()
                   if 'classifier' not in n and p.requires_grad]
head_params = [p for n, p in model.named_parameters()
               if 'classifier' in n]

optimizer = AdamW([
    {'params': backbone_params, 'lr': 1e-4},
    {'params': head_params, 'lr': 1e-3},
], weight_decay=0.01)

scheduler = CosineAnnealingLR(optimizer, T_max=50)

# 4. Training loop
criterion = nn.CrossEntropyLoss(label_smoothing=0.1)

for epoch in range(50):
    model.train()
    # for images, labels in train_loader:
    #     optimizer.zero_grad()
    #     outputs = model(images)
    #     loss = criterion(outputs, labels)
    #     loss.backward()
    #     optimizer.step()
    # scheduler.step()
```

---

## 11. Accuracy vs Efficiency Analysis

```
Accuracy vs FLOPs (ImageNet Top-1):

Top-1 %
  85 │                                          ·EfficientNet-B7
     │                                    ·B6
  84 │                              ·B5
     │
  83 │                        ·B4          ·NASNet-A
     │
  82 │                  ·B3        ·AmoebaNet-A
     │
  81 │            ·B2         ·ResNet-152
     │
  80 │       ·B1        ·Inception-v4
     │
  79 │  ·B0        ·ResNeXt-101
     │
  78 │        ·DenseNet-201
     │
  77 │   ·ResNet-50
     └──────────────────────────────────────────
       0.4  1   2   4   8   16  32  64   GFLOPs
       
Key takeaway: EfficientNet achieves the SAME accuracy as 
larger models with 4-8× fewer FLOPs
```

### Why EfficientNet Wins

| Factor | Explanation |
|---|---|
| **Compound scaling** | Balanced growth prevents bottlenecks |
| **NAS-optimized baseline** | B0 is already efficient (unlike scaling a suboptimal architecture) |
| **SE attention** | Cheap channel attention adds accuracy with minimal cost |
| **Swish activation** | Better gradient landscape than ReLU |
| **Stochastic depth** | Regularization that also speeds up training |

---

## 12. Summary Table

| Concept | Key Idea |
|---|---|
| **Core innovation** | Compound scaling — scale depth, width, resolution together |
| **Scaling formula** | d=α^φ, w=β^φ, r=γ^φ with α·β²·γ²≈2 |
| **Optimal coefficients** | α=1.2, β=1.1, γ=1.15 |
| **Baseline (B0)** | NAS-found, 5.3M params, 77.1% ImageNet top-1 |
| **Building block** | MBConv — inverted residual + SE + Swish |
| **Best model (B7)** | 66M params, 84.3% top-1 (8× fewer params than GPipe) |
| **Key activation** | Swish(x) = x · σ(x) |
| **Regularization** | Stochastic depth + dropout |
| **EfficientNetV2** | Fused-MBConv + progressive learning → 3-8× faster training |
| **Libraries** | `timm`, `torchvision.models` |

---

## 13. Revision Questions

1. **Explain compound scaling.** Why is scaling depth, width, and resolution simultaneously better than scaling a single dimension? What constraint does α·β²·γ²≈2 enforce?

2. **Describe the MBConv block** in EfficientNet. What is an "inverted residual"? Why does the projection layer have no activation function (linear bottleneck)?

3. **How does Squeeze-and-Excitation (SE) work** inside EfficientNet? Write the mathematical formulation. Why is the computational overhead negligible?

4. **Given α=1.2, β=1.1, γ=1.15 and φ=5 (B5):** compute the depth multiplier, width multiplier, resolution, and approximate FLOP increase over B0.

5. **Compare EfficientNet-B0 vs ResNet-50** in terms of parameters, FLOPs, and ImageNet accuracy. Why does a smaller model outperform a larger one?

6. **What improvements does EfficientNetV2 introduce** over the original EfficientNet? Why is Fused-MBConv used in early stages?

---

## 14. Navigation

| Previous | Up | Next |
|---|---|---|
| ← [Classic CNN Architectures](../05-Classic-CNN-Architectures/) | [Unit 6 Index](../06-Modern-CNN-Architectures/) | [MobileNet →](./02-mobilenet.md) |

---

> **References:**  
> - Tan, M. & Le, Q. V. (2019). *EfficientNet: Rethinking Model Scaling for CNNs.* ICML.  
> - Tan, M. & Le, Q. V. (2021). *EfficientNetV2: Smaller Models and Faster Training.* ICML.  
> - Hu, J. et al. (2018). *Squeeze-and-Excitation Networks.* CVPR.
