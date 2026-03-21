# ShuffleNet — Channel Shuffle for Efficient Group Convolutions

> **Unit 6 · Modern CNN Architectures · Topic 3/6**

---

## Table of Contents

1. [Overview & Motivation](#1-overview--motivation)
2. [Group Convolution Revisited](#2-group-convolution-revisited)
3. [Channel Shuffle Operation](#3-channel-shuffle-operation)
4. [ShuffleNet Unit (V1)](#4-shufflenet-unit-v1)
5. [ShuffleNetV1 Architecture](#5-shufflenetv1-architecture)
6. [ShuffleNetV2 — Practical Guidelines](#6-shufflenetv2--practical-guidelines)
7. [Mathematical Analysis](#7-mathematical-analysis)
8. [Worked Examples](#8-worked-examples)
9. [PyTorch Implementation](#9-pytorch-implementation)
10. [Comparison with MobileNet](#10-comparison-with-mobilenet)
11. [Summary Table](#11-summary-table)
12. [Revision Questions](#12-revision-questions)
13. [Navigation](#13-navigation)

---

## 1. Overview & Motivation

ShuffleNet (Zhang et al., 2018) addresses a fundamental limitation of **group convolutions**: groups don't share information with each other. The solution is elegantly simple — **shuffle the channels** between groups.

**Key problem with group convolutions:**

```
Without Channel Shuffle:

  Group 1    Group 2    Group 3      Group 1    Group 2    Group 3
  ┌────┐    ┌────┐    ┌────┐       ┌────┐    ┌────┐    ┌────┐
  │ a₁ │    │ b₁ │    │ c₁ │       │ a₁'│    │ b₁'│    │ c₁'│
  │ a₂ │    │ b₂ │    │ c₂ │  ──►  │ a₂'│    │ b₂'│    │ c₂'│
  │ a₃ │    │ b₃ │    │ c₃ │       │ a₃'│    │ b₃'│    │ c₃'│
  └────┘    └────┘    └────┘       └────┘    └────┘    └────┘
    Layer i                           Layer i+1

  Problem: Each group only sees its OWN channels!
  → Information is trapped within groups
  → Features from different groups never interact
  → Representational power is severely limited
```

---

## 2. Group Convolution Revisited

### Standard Convolution vs Group Convolution

```
Standard Convolution (1 group):
  Every output channel sees ALL input channels
  FLOPs = H × W × K² × Cᵢₙ × Cₒᵤₜ
  
  Input (Cᵢₙ=12)  ──all──►  Output (Cₒᵤₜ=12)

Group Convolution (g=3 groups):
  Each output channel sees only Cᵢₙ/g input channels
  FLOPs = H × W × K² × (Cᵢₙ/g) × (Cₒᵤₜ/g) × g
        = H × W × K² × Cᵢₙ × Cₒᵤₜ / g
        = Standard_FLOPs / g

  Input              Output
  ┌────┐ g₁ ──►    ┌────┐ g₁
  │ch 1│            │ch 1│
  │ch 2│            │ch 2│
  │ch 3│            │ch 3│
  │ch 4│            │ch 4│
  ├────┤ g₂ ──►    ├────┤ g₂
  │ch 5│            │ch 5│
  │ch 6│            │ch 6│
  │ch 7│            │ch 7│
  │ch 8│            │ch 8│
  ├────┤ g₃ ──►    ├────┤ g₃
  │ch 9│            │ch 9│
  │ch10│            │ch10│
  │ch11│            │ch11│
  │ch12│            │ch12│
  └────┘            └────┘
  
  g× fewer FLOPs, but NO cross-group interaction!
```

### Pointwise Group Convolution (1×1 Group Conv)

Standard 1×1 convolutions are the computational bottleneck in architectures like ResNeXt:

```
1×1 Conv with groups:
  FLOPs = H × W × (Cᵢₙ/g) × (Cₒᵤₜ/g) × g = H × W × Cᵢₙ × Cₒᵤₜ / g

For Cᵢₙ=Cₒᵤₜ=240, H=W=14, g=3:
  Standard 1×1:  14 × 14 × 240 × 240 = 11,289,600
  Grouped 1×1:   14 × 14 × 240 × 240 / 3 = 3,763,200  (3× cheaper)
```

---

## 3. Channel Shuffle Operation

The key innovation: **rearrange channels** after group convolution so that subsequent group convolutions see channels from different groups.

```
Channel Shuffle (g=3 groups, n=4 channels per group, total=12):

Before shuffle:                After shuffle:
Group 1: [a₁ a₂ a₃ a₄]       Group 1: [a₁ b₁ c₁ a₂]
Group 2: [b₁ b₂ b₃ b₄]  ──►  Group 2: [b₂ c₂ a₃ b₃]
Group 3: [c₁ c₂ c₃ c₄]       Group 3: [c₃ a₄ b₄ c₄]

Algorithm: Reshape → Transpose → Reshape
  
  Step 1: (C,) → (g, n)        Step 2: Transpose → (n, g)     Step 3: Flatten → (C,)
  ┌─────────────────┐          ┌─────────────────┐            ┌─────────────────┐
  │ a₁ a₂ a₃ a₄    │          │ a₁ b₁ c₁        │            │ a₁ b₁ c₁ a₂    │
  │ b₁ b₂ b₃ b₄    │   ──►    │ a₂ b₂ c₂        │   ──►      │ b₂ c₂ a₃ b₃    │
  │ c₁ c₂ c₃ c₄    │          │ a₃ b₃ c₃        │            │ c₃ a₄ b₄ c₄    │
  │    (3, 4)       │          │ a₄ b₄ c₄        │            │    (12,)        │
  └─────────────────┘          │    (4, 3)        │            └─────────────────┘
                               └─────────────────┘

Now each new "group" has channels from ALL original groups!
→ Cross-group information flow is restored
→ Zero computation cost (just memory reorder)
```

### Mathematical Formulation

```
Given tensor X of shape (batch, C, H, W) and g groups:

1. Reshape:    X → (batch, g, C/g, H, W)
2. Transpose:  → (batch, C/g, g, H, W)
3. Flatten:    → (batch, C, H, W)

This is a FREE operation — no parameters, no multiply-adds!
```

---

## 4. ShuffleNet Unit (V1)

### Basic ShuffleNet Unit (stride=1)

```
Input (H×W×C)
    │
    ├──────────────────────────── Shortcut (identity)
    │                                     │
    ▼                                     │
┌──────────────────┐                      │
│  1×1 GConv       │  Group pointwise     │
│  BN + ReLU       │  C → m (reduce)      │
└────────┬─────────┘                      │
         ▼                                │
┌──────────────────┐                      │
│  Channel Shuffle │  Rearrange channels  │
└────────┬─────────┘                      │
         ▼                                │
┌──────────────────┐                      │
│  3×3 DW Conv     │  Depthwise spatial   │
│  BN              │                      │
└────────┬─────────┘                      │
         ▼                                │
┌──────────────────┐                      │
│  1×1 GConv       │  Group pointwise     │
│  BN              │  m → C (expand)      │
└────────┬─────────┘                      │
         │                                │
         └────────── Add ◄────────────────┘
                      │
                      ▼
                  ReLU + Output
```

### ShuffleNet Unit with Stride=2 (Downsampling)

```
Input (H×W×C)
    │
    ├──────────────────────────── Shortcut
    │                             │
    ▼                             ▼
┌──────────────────┐     ┌──────────────────┐
│  1×1 GConv       │     │  3×3 AvgPool     │
│  BN + ReLU       │     │  stride=2        │
└────────┬─────────┘     └────────┬─────────┘
         ▼                        │
┌──────────────────┐              │
│  Channel Shuffle │              │
└────────┬─────────┘              │
         ▼                        │
┌──────────────────┐              │
│  3×3 DW Conv     │              │
│  stride=2        │              │
│  BN              │              │
└────────┬─────────┘              │
         ▼                        │
┌──────────────────┐              │
│  1×1 GConv       │              │
│  BN              │              │
└────────┬─────────┘              │
         │                        │
         └────── Concat ◄─────────┘  (NOT add — concatenate!)
                   │
                   ▼
               ReLU + Output (H/2 × W/2 × 2C)

Note: Concat doubles channels → "free" channel expansion
```

---

## 5. ShuffleNetV1 Architecture

```
┌──────────┬──────────┬──────────┬──────────┬──────────┐
│  Layer   │ Output   │ g=1      │ g=2      │ g=3      │
│          │ Size     │ channels │ channels │ channels │
├──────────┼──────────┼──────────┼──────────┼──────────┤
│ Conv1    │ 112×112  │ 24       │ 24       │ 24       │
│ MaxPool  │  56×56   │ -        │ -        │ -        │
├──────────┼──────────┼──────────┼──────────┼──────────┤
│ Stage2   │  28×28   │ 144      │ 200      │ 240      │
│ (×4)     │          │          │          │          │
├──────────┼──────────┼──────────┼──────────┼──────────┤
│ Stage3   │  14×14   │ 288      │ 400      │ 480      │
│ (×8)     │          │          │          │          │
├──────────┼──────────┼──────────┼──────────┼──────────┤
│ Stage4   │   7×7    │ 576      │ 800      │ 960      │
│ (×4)     │          │          │          │          │
├──────────┼──────────┼──────────┼──────────┼──────────┤
│ GAP + FC │   1×1    │ 1000     │ 1000     │ 1000     │
└──────────┴──────────┴──────────┴──────────┴──────────┘

More groups → more channels for same FLOPs budget
                 → better accuracy
```

---

## 6. ShuffleNetV2 — Practical Guidelines

ShuffleNetV2 (Ma et al., 2018) argued that **FLOPs alone don't determine speed** — memory access cost (MAC) matters too.

### Four Practical Guidelines

```
G1: Equal channel widths minimize MAC
──────────────────────────────────────
  Bad:  128 → 512 → 128    (unbalanced)
  Good: 256 → 256 → 256    (balanced)
  
  MAC ∝ hw(c₁ + c₂) + c₁c₂
  Minimized when c₁ = c₂ (by AM-GM inequality)

G2: Excessive group convolution increases MAC
──────────────────────────────────────────────
  More groups → more memory reads for small tensors
  FLOPs same, but g increases MAC:
  MAC = hw(cᵢ + cₒ) + cᵢcₒ/g  
  ↑ First term DOMINATES for small group sizes

G3: Network fragmentation reduces parallelism
──────────────────────────────────────────────
  Many small ops (multi-path, branching) → poor GPU utilization
  Prefer a single large operation over many small ones

G4: Element-wise operations are non-negligible
──────────────────────────────────────────────
  ReLU, add, bias, etc. have near-zero FLOPs
  but significant memory access → non-trivial latency
```

### ShuffleNetV2 Unit

```
ShuffleNetV2 Unit (stride=1):

Input (H×W×C)
    │
    ├── Channel Split ── Left branch (C/2 channels)
    │                         │
    │                         │ (identity — no computation)
    │                         │
    Right branch (C/2)        │
    │                         │
    ▼                         │
┌──────────────────┐          │
│  1×1 Conv        │  (G1: c_in = c_out = C/2)
│  BN + ReLU       │
└────────┬─────────┘          │
         ▼                    │
┌──────────────────┐          │
│  3×3 DW Conv     │  (G2: no group conv for 1×1)
│  BN              │
└────────┬─────────┘          │
         ▼                    │
┌──────────────────┐          │
│  1×1 Conv        │
│  BN + ReLU       │
└────────┬─────────┘          │
         │                    │
         └── Concat ◄─────────┘
               │
         Channel Shuffle
               │
               ▼
           Output (H×W×C)

Key differences from V1:
- Channel split instead of group conv at start (G2)
- Equal I/O channels in each branch (G1)
- No group conv on 1×1 layers (G2)
- Channel shuffle AFTER concat (more efficient)
- ReLU only on right branch, not on concat output (G4)


ShuffleNetV2 Unit (stride=2 — downsampling):

Input (H×W×C)
    │
    ├──────────────────────────── Right branch
    │                                  │
    Left branch                        ▼
    │                         ┌──────────────────┐
    ▼                         │  3×3 DW Conv s=2 │
┌──────────────────┐          │  BN              │
│  1×1 Conv        │          └────────┬─────────┘
│  BN + ReLU       │                   ▼
└────────┬─────────┘          ┌──────────────────┐
         ▼                    │  1×1 Conv        │
┌──────────────────┐          │  BN + ReLU       │
│  3×3 DW Conv s=2 │          └────────┬─────────┘
│  BN              │                   │
└────────┬─────────┘                   │
         ▼                             │
┌──────────────────┐                   │
│  1×1 Conv        │                   │
│  BN + ReLU       │                   │
└────────┬─────────┘                   │
         │                             │
         └────── Concat ◄──────────────┘
                   │
             Channel Shuffle
                   │
                   ▼
             Output (H/2 × W/2 × 2C)
```

---

## 7. Mathematical Analysis

### FLOPs Comparison

```
For input H×W×C, output H×W×C, kernel K×K:

Standard Conv:           H·W·K²·C²
Group Conv (g groups):   H·W·K²·C²/g
DW Sep Conv:             H·W·(K²·C + C²)  = H·W·C·(K² + C)
ShuffleNet Unit (V1):    H·W·(2·C²/g + K²·C)
ShuffleNet Unit (V2):    H·W·(C²/2 + K²·C/2)  ← each branch handles C/2

Channel Shuffle:         0 FLOPs (pure memory operation)
```

### Memory Access Cost (MAC)

```
Conv operation with input c₁, output c₂, spatial hw:

MAC = hw·c₁ (read input) + hw·c₂ (write output) + c₁·c₂ (read weights)
    = hw(c₁ + c₂) + c₁·c₂

For 1×1 conv with B FLOPs = hw·c₁·c₂:
  MAC = hw(c₁ + c₂) + c₁·c₂

By AM-GM: c₁ + c₂ ≥ 2√(c₁·c₂)
  MAC ≥ 2·hw·√(B/hw) + B/hw
  Equality when c₁ = c₂ → Guideline G1
```

---

## 8. Worked Examples

### Example 1: Channel Shuffle

**Problem:** Apply channel shuffle to a tensor with 12 channels and g=3 groups.

```
Original order: [0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11]
Groups:         [  G0    ] [  G1    ] [   G2     ]

Step 1: Reshape to (g, C/g) = (3, 4):
  [[0,  1,  2,  3],   ← G0
   [4,  5,  6,  7],   ← G1
   [8,  9, 10, 11]]   ← G2

Step 2: Transpose to (C/g, g) = (4, 3):
  [[0,  4,  8],
   [1,  5,  9],
   [2,  6, 10],
   [3,  7, 11]]

Step 3: Flatten:
  [0, 4, 8, 1, 5, 9, 2, 6, 10, 3, 7, 11]

New groups after next group conv:
  G0: [0, 4, 8, 1]   ← channels from ALL original groups ✓
  G1: [5, 9, 2, 6]   ← channels from ALL original groups ✓
  G2: [10, 3, 7, 11]  ← channels from ALL original groups ✓
```

### Example 2: FLOPs Comparison

**Problem:** Compare FLOPs for processing 14×14×480 features with different approaches.

```
Standard 3×3 Conv (480→480):
  FLOPs = 14 × 14 × 9 × 480 × 480 = 406,425,600 ≈ 406M

Group Conv (g=3):
  FLOPs = 406M / 3 = 135,475,200 ≈ 135M

Depthwise Separable Conv:
  DW:  14 × 14 × 9 × 480 = 846,720 ≈ 0.85M
  PW:  14 × 14 × 480 × 480 = 45,158,400 ≈ 45.2M
  Total ≈ 46M

ShuffleNet V2 unit (C/2 per branch):
  Branch: 240 channels
  1×1: 14 × 14 × 240 × 240 = 11,289,600 ≈ 11.3M
  DW:  14 × 14 × 9 × 240 = 423,360 ≈ 0.4M
  1×1: 14 × 14 × 240 × 240 = 11,289,600 ≈ 11.3M
  Total ≈ 23M  (plus identity branch costs nothing)
```

---

## 9. PyTorch Implementation

### Channel Shuffle

```python
import torch
import torch.nn as nn

def channel_shuffle(x: torch.Tensor, groups: int) -> torch.Tensor:
    """Shuffle channels across groups.
    
    Args:
        x: (batch, channels, height, width)
        groups: number of groups
    """
    batch, channels, height, width = x.shape
    channels_per_group = channels // groups

    # Reshape: (B, C, H, W) → (B, g, C/g, H, W)
    x = x.view(batch, groups, channels_per_group, height, width)

    # Transpose: (B, g, C/g, H, W) → (B, C/g, g, H, W)
    x = x.transpose(1, 2).contiguous()

    # Flatten: (B, C/g, g, H, W) → (B, C, H, W)
    x = x.view(batch, channels, height, width)

    return x

# Test
x = torch.arange(12).float().view(1, 12, 1, 1)
print("Before:", x.view(-1).tolist())
print("After: ", channel_shuffle(x, groups=3).view(-1).tolist())
# Before: [0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11]
# After:  [0, 4, 8, 1, 5, 9, 2, 6, 10, 3, 7, 11]
```

### ShuffleNetV1 Unit

```python
class ShuffleNetV1Unit(nn.Module):
    def __init__(self, in_ch, out_ch, groups, stride=1):
        super().__init__()
        self.stride = stride
        mid_ch = out_ch // 4

        if stride == 2:
            out_ch = out_ch - in_ch  # concat doubles channels

        # 1×1 group conv + shuffle
        self.compress = nn.Sequential(
            nn.Conv2d(in_ch, mid_ch, 1, groups=groups, bias=False),
            nn.BatchNorm2d(mid_ch),
            nn.ReLU(inplace=True),
        )
        self.groups = groups

        # 3×3 depthwise
        self.depthwise = nn.Sequential(
            nn.Conv2d(mid_ch, mid_ch, 3, stride, 1,
                      groups=mid_ch, bias=False),
            nn.BatchNorm2d(mid_ch),
        )

        # 1×1 group conv
        self.expand = nn.Sequential(
            nn.Conv2d(mid_ch, out_ch, 1, groups=groups, bias=False),
            nn.BatchNorm2d(out_ch),
        )

        if stride == 2:
            self.shortcut = nn.AvgPool2d(3, stride=2, padding=1)
        self.relu = nn.ReLU(inplace=True)

    def forward(self, x):
        out = self.compress(x)
        out = channel_shuffle(out, self.groups)
        out = self.depthwise(out)
        out = self.expand(out)

        if self.stride == 2:
            shortcut = self.shortcut(x)
            out = torch.cat([out, shortcut], dim=1)  # concat
        else:
            out = out + x  # add

        return self.relu(out)
```

### ShuffleNetV2 Unit

```python
class ShuffleNetV2Unit(nn.Module):
    def __init__(self, in_ch, out_ch, stride=1):
        super().__init__()
        self.stride = stride
        branch_ch = out_ch // 2

        if stride == 1:
            assert in_ch == out_ch
            self.branch_main = nn.Sequential(
                # 1×1 conv
                nn.Conv2d(branch_ch, branch_ch, 1, bias=False),
                nn.BatchNorm2d(branch_ch),
                nn.ReLU(inplace=True),
                # 3×3 depthwise
                nn.Conv2d(branch_ch, branch_ch, 3, 1, 1,
                          groups=branch_ch, bias=False),
                nn.BatchNorm2d(branch_ch),
                # 1×1 conv
                nn.Conv2d(branch_ch, branch_ch, 1, bias=False),
                nn.BatchNorm2d(branch_ch),
                nn.ReLU(inplace=True),
            )
        else:  # stride == 2
            # Main branch
            self.branch_main = nn.Sequential(
                nn.Conv2d(in_ch, branch_ch, 1, bias=False),
                nn.BatchNorm2d(branch_ch),
                nn.ReLU(inplace=True),
                nn.Conv2d(branch_ch, branch_ch, 3, stride, 1,
                          groups=branch_ch, bias=False),
                nn.BatchNorm2d(branch_ch),
                nn.Conv2d(branch_ch, branch_ch, 1, bias=False),
                nn.BatchNorm2d(branch_ch),
                nn.ReLU(inplace=True),
            )
            # Shortcut branch (also downsamples)
            self.branch_shortcut = nn.Sequential(
                nn.Conv2d(in_ch, in_ch, 3, stride, 1,
                          groups=in_ch, bias=False),
                nn.BatchNorm2d(in_ch),
                nn.Conv2d(in_ch, branch_ch, 1, bias=False),
                nn.BatchNorm2d(branch_ch),
                nn.ReLU(inplace=True),
            )

    def forward(self, x):
        if self.stride == 1:
            # Channel split
            x1, x2 = x.chunk(2, dim=1)
            out = torch.cat([x1, self.branch_main(x2)], dim=1)
        else:
            out = torch.cat([self.branch_shortcut(x),
                             self.branch_main(x)], dim=1)

        return channel_shuffle(out, groups=2)


# Test
unit_s1 = ShuffleNetV2Unit(64, 64, stride=1)
unit_s2 = ShuffleNetV2Unit(64, 128, stride=2)

x = torch.randn(1, 64, 28, 28)
print(unit_s1(x).shape)  # (1, 64, 28, 28)
print(unit_s2(x).shape)  # (1, 128, 14, 14)
```

### Using Pretrained ShuffleNet

```python
from torchvision.models import shufflenet_v2_x1_0, ShuffleNet_V2_X1_0_Weights

model = shufflenet_v2_x1_0(weights=ShuffleNet_V2_X1_0_Weights.IMAGENET1K_V1)

# Available variants:
# shufflenet_v2_x0_5  — 0.5× width
# shufflenet_v2_x1_0  — 1.0× width
# shufflenet_v2_x1_5  — 1.5× width
# shufflenet_v2_x2_0  — 2.0× width

# Modify for custom task
model.fc = nn.Linear(1024, 10)

x = torch.randn(1, 3, 224, 224)
print(model(x).shape)  # (1, 10)
print(f"Params: {sum(p.numel() for p in model.parameters()):,}")
```

---

## 10. Comparison with MobileNet

```
┌─────────────────┬──────────┬──────────┬──────────┬──────────┬──────────┐
│     Model       │ Params(M)│ MFLOPs   │ Top-1(%) │ Latency  │ Key Idea │
├─────────────────┼──────────┼──────────┼──────────┼──────────┼──────────┤
│ MobileNetV1     │   4.2    │   569    │  70.6    │  1.0×    │ DW-sep   │
│ MobileNetV2     │   3.4    │   300    │  72.0    │  0.75×   │ Inv.Res. │
│ ShuffleNet V1   │   ~2.4   │   140    │  67.4    │  0.70×   │ Ch.Shuff │
│ ShuffleNetV2 1× │   2.3    │   146    │  69.4    │  0.60×   │ Practical│
│ ShuffleNetV2 2× │   7.4    │   591    │  74.9    │  1.20×   │ G1-G4   │
└─────────────────┴──────────┴──────────┴──────────┴──────────┴──────────┘

Design Philosophy Comparison:
┌──────────────────┬──────────────────────────┬──────────────────────────┐
│   Aspect         │  MobileNet               │  ShuffleNet              │
├──────────────────┼──────────────────────────┼──────────────────────────┤
│ 1×1 convolutions │ Standard (expensive)     │ Group (cheap) + shuffle  │
│ Spatial conv     │ Depthwise                │ Depthwise                │
│ Efficiency metric│ FLOPs (V1/V2)            │ Speed (V2) — MAC-aware   │
│ Residual design  │ Inverted (V2)            │ Concat-based (doubles ch)│
│ NAS usage        │ V3 only                  │ No NAS                   │
│ Best use case    │ General mobile           │ Ultra-low compute budget │
└──────────────────┴──────────────────────────┴──────────────────────────┘
```

---

## 11. Summary Table

| Concept | Key Idea |
|---|---|
| **Group convolution** | Divide channels into groups, each processed independently; g× cheaper |
| **Channel shuffle** | Rearrange channels between groups: reshape → transpose → flatten |
| **ShuffleNet V1 unit** | 1×1 GConv → shuffle → 3×3 DW → 1×1 GConv (+ add or concat) |
| **Stride-2 unit** | Uses concat instead of add → doubles channels for free |
| **ShuffleNetV2 G1** | Equal channel widths minimize memory access cost |
| **ShuffleNetV2 G2** | Excessive groups increase MAC despite same FLOPs |
| **ShuffleNetV2 G3** | Avoid network fragmentation for GPU parallelism |
| **ShuffleNetV2 G4** | Element-wise ops (ReLU, add) have non-trivial cost |
| **V2 unit** | Channel split → identity + conv branch → concat → shuffle |
| **Key advantage** | Faster actual speed than MobileNet at same FLOPs |

---

## 12. Revision Questions

1. **Explain channel shuffle.** Given 18 channels and 3 groups, show the channel order before and after shuffle. Why is this operation computationally free?

2. **What problem does channel shuffle solve** for group convolutions? Draw a diagram showing information flow with and without shuffle.

3. **List and explain the four practical guidelines** from ShuffleNetV2. Why do FLOPs alone not determine actual speed?

4. **Compare ShuffleNetV1 and V2 units.** What changed and why? How does channel split in V2 relate to guidelines G1 and G2?

5. **Compare ShuffleNet with MobileNet.** When would you prefer one over the other? Which achieves better latency on actual hardware?

6. **Implement channel shuffle in PyTorch** using tensor operations. Verify with a 12-channel example and 4 groups.

---

## 13. Navigation

| Previous | Up | Next |
|---|---|---|
| [← MobileNet](./02-mobilenet.md) | [Unit 6 Index](../06-Modern-CNN-Architectures/) | [NASNet →](./04-nasnet.md) |

---

> **References:**  
> - Zhang, X. et al. (2018). *ShuffleNet: An Extremely Efficient CNN for Mobile Devices.* CVPR.  
> - Ma, N. et al. (2018). *ShuffleNet V2: Practical Guidelines for Efficient CNN Architecture Design.* ECCV.
