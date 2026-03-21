# MobileNet — Efficient CNNs for Mobile & Edge Deployment

> **Unit 6 · Modern CNN Architectures · Topic 2/6**

---

## Table of Contents

1. [Overview & Motivation](#1-overview--motivation)
2. [Depthwise Separable Convolutions](#2-depthwise-separable-convolutions)
3. [MobileNetV1 Architecture](#3-mobilenetv1-architecture)
4. [Width & Resolution Multipliers](#4-width--resolution-multipliers)
5. [MobileNetV2 — Inverted Residuals & Linear Bottleneck](#5-mobilenetv2--inverted-residuals--linear-bottleneck)
6. [MobileNetV3 — NAS + NetAdapt](#6-mobilenetv3--nas--netadapt)
7. [Mathematical Analysis](#7-mathematical-analysis)
8. [Worked Examples](#8-worked-examples)
9. [PyTorch Implementation](#9-pytorch-implementation)
10. [Mobile Deployment Considerations](#10-mobile-deployment-considerations)
11. [Summary Table](#11-summary-table)
12. [Revision Questions](#12-revision-questions)
13. [Navigation](#13-navigation)

---

## 1. Overview & Motivation

MobileNets (Howard et al., 2017) are a family of lightweight CNN architectures designed for **mobile and embedded vision applications** where computational resources are limited.

**Core challenge:** Standard CNNs (VGG, ResNet) require billions of FLOPs and hundreds of millions of parameters — far too expensive for smartphones, IoT devices, drones, and AR glasses.

**Key insight:** Replace standard convolutions with **depthwise separable convolutions** to dramatically reduce computation while maintaining accuracy.

```
Resource Comparison:
┌──────────────┬────────────┬───────────┬──────────────┐
│    Model     │ Params (M) │ FLOPs (M) │ Top-1 Acc(%) │
├──────────────┼────────────┼───────────┼──────────────┤
│ VGG-16       │    138     │  15,470   │    71.5      │
│ ResNet-50    │     25     │   4,100   │    76.0      │
│ MobileNetV1  │    4.2     │    569    │    70.6      │
│ MobileNetV2  │    3.4     │    300    │    72.0      │
│ MobileNetV3-L│    5.4     │    219    │    75.2      │
└──────────────┴────────────┴───────────┴──────────────┘
```

---

## 2. Depthwise Separable Convolutions

The fundamental innovation: **factorize** a standard convolution into two simpler operations.

### Standard Convolution

```
Standard Conv: K × K × Cᵢₙ × Cₒᵤₜ

Input           Filter              Output
(H×W×Cᵢₙ)     (K×K×Cᵢₙ×Cₒᵤₜ)     (H×W×Cₒᵤₜ)
┌──────┐       ┌───┐ × Cₒᵤₜ       ┌──────┐
│      │       │K×K│               │      │
│ Cᵢₙ  │  ★   │×  │  ───────►    │ Cₒᵤₜ │
│      │       │Cᵢₙ│               │      │
└──────┘       └───┘               └──────┘

Each output channel requires a K×K×Cᵢₙ filter
→ Mixes spatial AND channel information simultaneously
→ Cost: H × W × K² × Cᵢₙ × Cₒᵤₜ
```

### Depthwise Separable Convolution = Depthwise + Pointwise

```
STEP 1: Depthwise Convolution (spatial filtering per channel)
─────────────────────────────────────────────────────────────
Input           DW Filters          Intermediate
(H×W×Cᵢₙ)     K×K × 1 (per ch)     (H×W×Cᵢₙ)
┌──────┐       ┌───┐               ┌──────┐
│ ch 1 │  ★    │f₁ │   ─────►     │ ch 1 │
│ ch 2 │  ★    │f₂ │   ─────►     │ ch 2 │
│ ch 3 │  ★    │f₃ │   ─────►     │ ch 3 │
│  ... │       │...│               │  ... │
│ ch M │  ★    │fₘ│   ─────►     │ ch M │
└──────┘       └───┘               └──────┘

★ Each filter applies to ONE channel only (groups = Cᵢₙ)
→ Cost: H × W × K² × Cᵢₙ

STEP 2: Pointwise Convolution (channel mixing)
─────────────────────────────────────────────────
Intermediate    PW Filter           Output
(H×W×Cᵢₙ)     (1×1×Cᵢₙ×Cₒᵤₜ)     (H×W×Cₒᵤₜ)
┌──────┐       ┌─┐ × Cₒᵤₜ         ┌──────┐
│      │       │1│                 │      │
│ Cᵢₙ  │  ★   │×│   ─────►       │ Cₒᵤₜ │
│      │       │1│                 │      │
└──────┘       └─┘                 └──────┘

★ Standard 1×1 convolution — mixes channels
→ Cost: H × W × Cᵢₙ × Cₒᵤₜ
```

### Computational Savings

```
Standard Conv Cost:      H · W · K² · M · N

Depthwise Sep. Cost:     H · W · K² · M          (depthwise)
                       + H · W · M · N            (pointwise)
                       = H · W · M · (K² + N)

Ratio = Depthwise Sep. / Standard
      = (K² + N) / (K² · N)
      = 1/N + 1/K²

For K=3, N=256:
  Ratio = 1/256 + 1/9 ≈ 0.115 → ~8.5× fewer computations!

For K=3, N=512:
  Ratio = 1/512 + 1/9 ≈ 0.113 → ~8.9× fewer computations!
```

---

## 3. MobileNetV1 Architecture

```
MobileNetV1 Architecture:
┌────────────────────────────────────────────────────────────┐
│ Layer           │ Filter/Stride │ Input Size  │ Output Size│
├────────────────────────────────────────────────────────────┤
│ Conv / s2       │ 3×3×3×32      │ 224×224×3   │ 112×112×32│
│ DW + PW / s1    │ 3×3×32 dw     │ 112×112×32  │ 112×112×64│
│                 │ 1×1×32×64 pw  │             │           │
│ DW + PW / s2    │ 3×3×64 dw     │ 112×112×64  │ 56×56×128 │
│                 │ 1×1×64×128 pw │             │           │
│ DW + PW / s1    │ 3×3×128 dw    │ 56×56×128   │ 56×56×128 │
│ DW + PW / s2    │ 3×3×128 dw    │ 56×56×128   │ 28×28×256 │
│ DW + PW / s1    │ 3×3×256 dw    │ 28×28×256   │ 28×28×256 │
│ DW + PW / s2    │ 3×3×256 dw    │ 28×28×256   │ 14×14×512 │
│ 5× DW+PW / s1  │ 3×3×512 dw    │ 14×14×512   │ 14×14×512 │
│ DW + PW / s2    │ 3×3×512 dw    │ 14×14×512   │ 7×7×1024  │
│ DW + PW / s1    │ 3×3×1024 dw   │ 7×7×1024    │ 7×7×1024  │
│ AvgPool / s1    │ 7×7           │ 7×7×1024    │ 1×1×1024  │
│ FC / s1         │ 1024×1000     │ 1×1×1024    │ 1×1×1000  │
│ Softmax         │               │ 1×1×1000    │ 1×1×1000  │
└────────────────────────────────────────────────────────────┘

Each DW + PW block:
  Input → [3×3 DW Conv → BN → ReLU6] → [1×1 PW Conv → BN → ReLU6] → Output
```

### ReLU6 Activation

```
ReLU6(x) = min(max(0, x), 6)

       6 ┤─────────────────────
         │                    /
         │                   /
         │                  /
       0 ├─────────────────/
         │                │
         └────────────────┴─── x
              -2   0   6   8

Purpose: Clips activations at 6 for better fixed-point (quantized) inference
```

---

## 4. Width & Resolution Multipliers

Two hyperparameters for trading accuracy vs speed:

### Width Multiplier α

Thins the network by scaling channel counts:

```
New channel count = α × Original channel count
α ∈ {1.0, 0.75, 0.5, 0.25}

FLOPs ∝ α²  (channels appear in both input and output)

Example: α = 0.75, layer with 512 → 1024 channels
  Becomes: 384 → 768 channels
```

### Resolution Multiplier ρ

Reduces input image resolution:

```
New resolution = ρ × 224
ρ ∈ {1.0, 224/224, 192/224, 160/224, 128/224}

Resolutions: 224, 192, 160, 128
FLOPs ∝ ρ²  (spatial dimensions H × W)

Combined: FLOPs ∝ α² × ρ²
```

```
Trade-off Table:
┌────────┬─────┬───────────┬───────────┬──────────┐
│  α     │  ρ  │ Resolution│ MFLOPs    │ Top-1(%) │
├────────┼─────┼───────────┼───────────┼──────────┤
│  1.0   │ 1.0 │   224     │   569     │  70.6    │
│  0.75  │ 1.0 │   224     │   325     │  68.4    │
│  0.5   │ 1.0 │   224     │   150     │  63.7    │
│  0.25  │ 1.0 │   224     │    41     │  50.6    │
│  1.0   │ 192 │   192     │   418     │  69.1    │
│  1.0   │ 160 │   160     │   291     │  67.2    │
│  1.0   │ 128 │   128     │   186     │  64.4    │
└────────┴─────┴───────────┴───────────┴──────────┘
```

---

## 5. MobileNetV2 — Inverted Residuals & Linear Bottleneck

MobileNetV2 (Sandler et al., 2018) introduced two key innovations:

### Innovation 1: Inverted Residual Block

```
Standard Residual (ResNet):         Inverted Residual (MobileNetV2):
  Wide → Narrow → Wide                Narrow → Wide → Narrow

  Input (256-d)                        Input (24-d)
    │                                    │
    ▼                                    ▼
  1×1 Conv (64-d)  ← Reduce           1×1 Conv (144-d)  ← EXPAND (×6)
    │                                    │
    ▼                                    ▼
  3×3 Conv (64-d)                      3×3 DW Conv (144-d)  ← Depthwise
    │                                    │
    ▼                                    ▼
  1×1 Conv (256-d) ← Restore          1×1 Conv (24-d)  ← PROJECT (reduce)
    │                                    │
    + ← Residual                         + ← Residual (in LOW-dim space)
    │                                    │
  Output (256-d)                       Output (24-d)

Key insight: Residual connects the BOTTLENECK layers
→ Memory efficient: shortcut in low-dimensional space
```

### Innovation 2: Linear Bottleneck

```
Why no ReLU on the final 1×1 projection?

ReLU destroys information in low-dimensional spaces:

  High-dim (e.g., 144-d):  ReLU safe — enough dimensions to preserve info
  Low-dim (e.g., 24-d):    ReLU harmful — zeroing out dimensions loses info

  ┌──────────────────────────────────────────────┐
  │  With ReLU on bottleneck:                     │
  │  24-d → ReLU → many dims zeroed → info lost  │
  │                                                │
  │  Without ReLU (linear):                        │
  │  24-d → linear projection → info preserved    │
  └──────────────────────────────────────────────┘
```

### MobileNetV2 Architecture

```
MobileNetV2 Inverted Residual Block:

Input (H×W×k)
    │
    ├──────────────────────────── Skip connection
    │                             (only if stride=1 AND in_ch=out_ch)
    ▼
┌──────────────────┐
│  1×1 Conv (expand)│  k → tk  (expansion factor t, typically 6)
│  BN + ReLU6      │
└────────┬─────────┘
         ▼
┌──────────────────┐
│  3×3 DW Conv     │  tk → tk  (depthwise, stride 1 or 2)
│  BN + ReLU6      │
└────────┬─────────┘
         ▼
┌──────────────────┐
│  1×1 Conv (proj.) │  tk → k'  (LINEAR — no activation!)
│  BN              │
└────────┬─────────┘
         │
    ┌────┘
    + ← Add residual
    │
  Output (H'×W'×k')


Full Architecture:
┌───────┬────────────┬───┬────────┬──────────┬────────┬─────────┐
│ Input │  Operator  │ t │ Output │ Stride   │ Repeat │ Resolution│
├───────┼────────────┼───┼────────┼──────────┼────────┼─────────┤
│ 224²×3│ conv2d     │ - │   32   │    2     │   1    │  112²   │
│112²×32│ bottleneck │ 1 │   16   │    1     │   1    │  112²   │
│112²×16│ bottleneck │ 6 │   24   │    2     │   2    │   56²   │
│ 56²×24│ bottleneck │ 6 │   32   │    2     │   3    │   28²   │
│ 28²×32│ bottleneck │ 6 │   64   │    2     │   4    │   14²   │
│ 14²×64│ bottleneck │ 6 │   96   │    1     │   3    │   14²   │
│14²×96 │ bottleneck │ 6 │  160   │    2     │   3    │    7²   │
│7²×160 │ bottleneck │ 6 │  320   │    1     │   1    │    7²   │
│7²×320 │ conv2d 1×1 │ - │ 1280   │    1     │   1    │    7²   │
│7²×1280│ avgpool 7×7│ - │   -    │    -     │   1    │    1²   │
│1²×1280│ conv2d 1×1 │ - │   k    │    -     │   1    │    1²   │
└───────┴────────────┴───┴────────┴──────────┴────────┴─────────┘
t = expansion factor,  k = number of classes
```

---

## 6. MobileNetV3 — NAS + NetAdapt

MobileNetV3 (Howard et al., 2019) combines:

### Architecture Search Pipeline

```
┌─────────────────────────────────┐
│  Step 1: NAS (MnasNet-style)   │  → Find global architecture
│  Platform-aware search          │    (block types, layers, channels)
│  Reward: ACC(m) × [LAT(m)/T]^w │
└──────────────┬──────────────────┘
               ▼
┌─────────────────────────────────┐
│  Step 2: NetAdapt              │  → Fine-tune per-layer
│  Progressive layer reduction    │    (filter count per layer)
└──────────────┬──────────────────┘
               ▼
┌─────────────────────────────────┐
│  Step 3: Manual Refinements    │  → Human-designed improvements
│  - h-swish activation          │
│  - Efficient last stage        │
│  - Efficient first layer       │
└─────────────────────────────────┘
```

### Key Improvements

**h-swish Activation (hardware-friendly Swish):**

```
swish(x) = x · σ(x)         ← requires sigmoid (expensive on mobile)

h-swish(x) = x · ReLU6(x + 3) / 6    ← piecewise linear approximation

             │         ╱──────
             │        ╱
         0   ├───────╱
             │      ╱
             │─────╱
             └──────────────── x
              -3    0    3

Almost identical results, much cheaper on mobile hardware
```

**Efficient Last Stage:**

```
MobileNetV2 last stage:        MobileNetV3 last stage:
┌──────────────────┐           ┌──────────────────┐
│ 7×7 Conv→1280    │           │ AvgPool 7×7      │ ← Pool FIRST
│ AvgPool 7×7      │           │ 1×1 Conv→1280    │ ← Conv on 1×1
│ 1×1 Conv→k       │           │ 1×1 Conv→k       │
└──────────────────┘           └──────────────────┘
  7×7 at 1280ch = expensive     1×1 at 1280ch = ~49× cheaper!
```

**SE Attention** integrated into selected blocks (with reduced ratio for efficiency).

### MobileNetV3 Variants

```
┌──────────────────┬──────────┬───────────┬──────────┐
│  Model           │ Params(M)│ MFLOPs    │ Top-1(%) │
├──────────────────┼──────────┼───────────┼──────────┤
│ MobileNetV3-Large│   5.4    │   219     │  75.2    │
│ MobileNetV3-Small│   2.9    │    56     │  67.4    │
└──────────────────┴──────────┴───────────┴──────────┘
```

---

## 7. Mathematical Analysis

### FLOP Count: Standard vs Depthwise Separable

```
Given: Input H×W×M, Output H×W×N, Kernel K×K

Standard Convolution:
  FLOPs = H · W · K² · M · N
  Parameters = K² · M · N

Depthwise Separable Convolution:
  FLOPs_dw = H · W · K² · M
  FLOPs_pw = H · W · M · N
  FLOPs_total = H · W · M · (K² + N)
  
  Parameters_dw = K² · M
  Parameters_pw = M · N
  Parameters_total = M · (K² + N)

Savings ratio:
  FLOPs_ratio = (K² + N) / (K² · N) = 1/N + 1/K²
  Param_ratio = (K² + N) / (K² · N) = 1/N + 1/K²
```

### Memory Analysis of Inverted Residuals

```
Standard Residual:
  Memory for shortcut: H × W × Cₒᵤₜ (large, e.g., 256)
  Peak activation memory: max(H×W×Cₒᵤₜ, H×W×Cᵦₒₜₜₗₑ)

Inverted Residual:
  Memory for shortcut: H × W × Cᵦₒₜₜₗₑ (small, e.g., 24)
  Peak activation memory: max(H×W×Cᵦₒₜₜₗₑ, H×W×Cₑₓₚₐₙᵈₑᵈ)

  → Shortcut memory is ~10× smaller!
  → Important for mobile devices with limited RAM
```

---

## 8. Worked Examples

### Example 1: Computational Savings

**Problem:** Compute FLOPs for a layer with input 14×14×512, output 14×14×512, kernel 3×3 for both standard and depthwise separable convolution.

```
Standard Convolution:
  FLOPs = 14 × 14 × 3² × 512 × 512
        = 196 × 9 × 262,144
        = 462,422,016
        ≈ 462M FLOPs

Depthwise Separable:
  DW: 14 × 14 × 3² × 512 = 196 × 9 × 512 = 903,168 ≈ 0.9M
  PW: 14 × 14 × 512 × 512 = 196 × 262,144 = 51,380,224 ≈ 51.4M
  Total = 52,283,392 ≈ 52.3M FLOPs

Savings = 462M / 52.3M ≈ 8.84× fewer FLOPs ✓

Verify with formula: 1/N + 1/K² = 1/512 + 1/9 ≈ 0.113 → 8.85× ✓
```

### Example 2: MobileNetV2 Block FLOPs

**Problem:** Compute FLOPs for an inverted residual block: input 56×56×24, expansion=6, output channels=24, kernel 3×3, stride=1.

```
Step 1: Expansion (1×1 PW)
  24 → 144 channels
  FLOPs = 56 × 56 × 24 × 144 = 10,838,016 ≈ 10.8M

Step 2: Depthwise (3×3 DW)
  144 channels, 3×3 kernel
  FLOPs = 56 × 56 × 9 × 144 = 4,064,256 ≈ 4.1M

Step 3: Projection (1×1 PW)
  144 → 24 channels
  FLOPs = 56 × 56 × 144 × 24 = 10,838,016 ≈ 10.8M

Total = 25,740,288 ≈ 25.7M FLOPs

Compare with standard 3×3 conv (24→24):
  56 × 56 × 9 × 24 × 24 = 16,257,024 ≈ 16.3M
  
The inverted residual costs MORE per block than a plain 3×3,
BUT the network needs fewer blocks total and the bottleneck
structure keeps memory low.
```

---

## 9. PyTorch Implementation

### Depthwise Separable Conv

```python
import torch
import torch.nn as nn

class DepthwiseSeparableConv(nn.Module):
    """Depthwise separable convolution: DW + PW."""
    def __init__(self, in_ch, out_ch, kernel_size=3, stride=1, padding=1):
        super().__init__()
        self.depthwise = nn.Sequential(
            nn.Conv2d(in_ch, in_ch, kernel_size, stride, padding,
                      groups=in_ch, bias=False),  # groups=in_ch → depthwise
            nn.BatchNorm2d(in_ch),
            nn.ReLU6(inplace=True),
        )
        self.pointwise = nn.Sequential(
            nn.Conv2d(in_ch, out_ch, 1, bias=False),  # 1×1 conv
            nn.BatchNorm2d(out_ch),
            nn.ReLU6(inplace=True),
        )

    def forward(self, x):
        x = self.depthwise(x)
        x = self.pointwise(x)
        return x
```

### MobileNetV2 Inverted Residual Block

```python
class InvertedResidual(nn.Module):
    def __init__(self, in_ch, out_ch, stride, expand_ratio):
        super().__init__()
        mid_ch = in_ch * expand_ratio
        self.use_residual = (stride == 1 and in_ch == out_ch)

        layers = []
        # Expansion (skip if ratio is 1)
        if expand_ratio != 1:
            layers.extend([
                nn.Conv2d(in_ch, mid_ch, 1, bias=False),
                nn.BatchNorm2d(mid_ch),
                nn.ReLU6(inplace=True),
            ])

        # Depthwise
        layers.extend([
            nn.Conv2d(mid_ch, mid_ch, 3, stride, 1,
                      groups=mid_ch, bias=False),
            nn.BatchNorm2d(mid_ch),
            nn.ReLU6(inplace=True),
        ])

        # Pointwise linear (NO activation)
        layers.extend([
            nn.Conv2d(mid_ch, out_ch, 1, bias=False),
            nn.BatchNorm2d(out_ch),
        ])

        self.conv = nn.Sequential(*layers)

    def forward(self, x):
        if self.use_residual:
            return x + self.conv(x)
        return self.conv(x)


# Test
block = InvertedResidual(in_ch=24, out_ch=24, stride=1, expand_ratio=6)
x = torch.randn(1, 24, 56, 56)
print(block(x).shape)  # torch.Size([1, 24, 56, 56])
print(f"Params: {sum(p.numel() for p in block.parameters()):,}")
```

### Using Pretrained MobileNets

```python
import torchvision.models as models

# MobileNetV2
mobilenet_v2 = models.mobilenet_v2(weights=models.MobileNet_V2_Weights.IMAGENET1K_V1)
# Modify classifier
mobilenet_v2.classifier = nn.Sequential(
    nn.Dropout(0.2),
    nn.Linear(1280, 10),
)

# MobileNetV3
mobilenet_v3_large = models.mobilenet_v3_large(
    weights=models.MobileNet_V3_Large_Weights.IMAGENET1K_V1
)
mobilenet_v3_small = models.mobilenet_v3_small(
    weights=models.MobileNet_V3_Small_Weights.IMAGENET1K_V1
)

# With timm
import timm
model = timm.create_model('mobilenetv3_large_100', pretrained=True, num_classes=10)
```

### Full MobileNetV1 (Simplified)

```python
class MobileNetV1(nn.Module):
    def __init__(self, num_classes=1000, width_mult=1.0):
        super().__init__()

        def _make_layer(in_ch, out_ch, stride):
            in_ch = int(in_ch * width_mult)
            out_ch = int(out_ch * width_mult)
            return DepthwiseSeparableConv(in_ch, out_ch, 3, stride, 1)

        w = width_mult
        self.model = nn.Sequential(
            # First standard conv
            nn.Sequential(
                nn.Conv2d(3, int(32*w), 3, 2, 1, bias=False),
                nn.BatchNorm2d(int(32*w)),
                nn.ReLU6(inplace=True),
            ),
            _make_layer(32, 64, 1),
            _make_layer(64, 128, 2),
            _make_layer(128, 128, 1),
            _make_layer(128, 256, 2),
            _make_layer(256, 256, 1),
            _make_layer(256, 512, 2),
            # 5× repeated
            *[_make_layer(512, 512, 1) for _ in range(5)],
            _make_layer(512, 1024, 2),
            _make_layer(1024, 1024, 1),
            nn.AdaptiveAvgPool2d(1),
        )
        self.classifier = nn.Linear(int(1024*w), num_classes)

    def forward(self, x):
        x = self.model(x)
        x = x.flatten(1)
        return self.classifier(x)

model = MobileNetV1(num_classes=10, width_mult=0.75)
print(f"Params: {sum(p.numel() for p in model.parameters()):,}")
```

---

## 10. Mobile Deployment Considerations

### Quantization

```
Full Precision (FP32):     32 bits per weight → baseline
Half Precision (FP16):     16 bits → 2× smaller, ~same accuracy
INT8 Quantization:          8 bits → 4× smaller, minimal accuracy loss
INT4 Quantization:          4 bits → 8× smaller, some accuracy loss

MobileNetV2 with Quantization:
┌────────────┬────────────┬───────────┬──────────┐
│  Precision │  Size (MB) │ Latency   │ Top-1(%) │
├────────────┼────────────┼───────────┼──────────┤
│   FP32     │   14.0     │  1.0×     │  72.0    │
│   FP16     │    7.0     │  0.6×     │  72.0    │
│   INT8     │    3.5     │  0.3×     │  71.8    │
└────────────┴────────────┴───────────┴──────────┘
```

### Deployment Frameworks

```
Training          Export              Deployment
┌──────────┐     ┌──────────┐       ┌──────────────────┐
│ PyTorch  │ ──► │  ONNX    │ ──►  │ ONNX Runtime     │
│          │     │  TorchScript│    │ TensorRT (NVIDIA)│
│          │     │  Core ML  │     │ Core ML (Apple)  │
│          │     │  TFLite   │     │ TFLite (Android) │
└──────────┘     └──────────┘       └──────────────────┘

# Export to ONNX
torch.onnx.export(model, dummy_input, "mobilenet.onnx",
                  input_names=['image'], output_names=['output'])

# Export to TorchScript
scripted = torch.jit.script(model)
scripted.save("mobilenet.pt")
```

---

## 11. Summary Table

| Concept | Key Idea |
|---|---|
| **Depthwise separable conv** | Factor standard conv into depthwise (spatial) + pointwise (channel) |
| **Computation savings** | ~8-9× fewer FLOPs (1/N + 1/K²) |
| **MobileNetV1** | Stack of DW-sep convs + width/resolution multipliers |
| **MobileNetV2** | Inverted residuals (narrow→wide→narrow) + linear bottleneck |
| **Linear bottleneck** | No ReLU after projection to preserve info in low-dim space |
| **MobileNetV3** | NAS + NetAdapt + h-swish + efficient last stage + SE attention |
| **Width multiplier α** | Thin all channels by factor α; FLOPs ∝ α² |
| **Resolution multiplier ρ** | Reduce input resolution; FLOPs ∝ ρ² |
| **ReLU6** | min(max(0,x), 6) — quantization-friendly |
| **h-swish** | x·ReLU6(x+3)/6 — hardware-friendly Swish approximation |

---

## 12. Revision Questions

1. **Explain depthwise separable convolution** with a diagram. Derive the computational savings ratio `1/N + 1/K²`. For a 5×5 kernel with 256 output channels, what is the savings factor?

2. **What is an "inverted" residual block?** Compare with a standard ResNet bottleneck. Why is the residual connection in the narrow (bottleneck) dimension? How does this save memory?

3. **Why does MobileNetV2 use a linear bottleneck** (no activation after the final 1×1 conv)? What happens to information in low-dimensional spaces when ReLU is applied?

4. **Compare MobileNetV1, V2, and V3.** List the key innovation of each version. Which version achieves the best accuracy-latency trade-off and why?

5. **Explain h-swish** and why it's preferred over standard Swish on mobile devices. Write the formula. Why is it faster to compute?

6. **A mobile model processes 56×56×96 features** through a depthwise separable conv to produce 56×56×96 output with 3×3 kernel. Compute exact FLOPs for both standard and depthwise separable convolutions.

---

## 13. Navigation

| Previous | Up | Next |
|---|---|---|
| [← EfficientNet](./01-efficientnet.md) | [Unit 6 Index](../06-Modern-CNN-Architectures/) | [ShuffleNet →](./03-shufflenet.md) |

---

> **References:**  
> - Howard, A. et al. (2017). *MobileNets: Efficient CNNs for Mobile Vision Applications.*  
> - Sandler, M. et al. (2018). *MobileNetV2: Inverted Residuals and Linear Bottlenecks.* CVPR.  
> - Howard, A. et al. (2019). *Searching for MobileNetV3.* ICCV.
