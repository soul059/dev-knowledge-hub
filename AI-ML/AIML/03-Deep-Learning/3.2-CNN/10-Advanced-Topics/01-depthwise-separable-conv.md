# Depthwise Separable Convolutions

[← Previous: Understanding What CNNs Learn](../../09-Visualizing-CNNs/06-understanding-cnns-learn.md) | [Next: Dilated/Atrous Convolutions →](02-dilated-atrous-conv.md)

---

## Overview

Depthwise separable convolutions decompose a standard convolution into two simpler operations — **depthwise convolution** and **pointwise convolution** — dramatically reducing computation and parameters while maintaining representational power. They are the core building block of lightweight architectures like **MobileNet** and **Xception**.

---

## 1. Standard Convolution Recap

A standard convolution with:
- Input: `Dₖ × Dₖ × M` (spatial size Dₖ, M input channels)
- Filter: `K × K × M` (one filter spans ALL input channels)
- N output filters

```
Standard Conv Computation:

Input (Dₖ × Dₖ × M)
    │
    ▼
┌─────────────────┐
│  N filters of   │   Each filter: K × K × M
│  size K×K×M     │   Slides across spatial dims
└─────────────────┘
    │
    ▼
Output (Dₖ × Dₖ × N)
```

**Computational cost:** `K² · M · N · Dₖ²`

**Parameters:** `K² · M · N + N` (bias)

For a 3×3 conv with 256 input and 256 output channels on 14×14:
- FLOPs = 3² × 256 × 256 × 14² = **115.6M**
- Params = 3² × 256 × 256 = **589,824**

---

## 2. Depthwise Convolution

Apply **one filter per input channel** independently. No cross-channel interaction.

```
Depthwise Convolution:

Channel 1: [K×K filter₁] → Feature Map 1
Channel 2: [K×K filter₂] → Feature Map 2
Channel 3: [K×K filter₃] → Feature Map 3
   ...          ...              ...
Channel M: [K×K filterₘ] → Feature Map M

Input channels = Output channels = M
```

**Each channel is convolved independently with its own K×K filter.**

**Computational cost:** `K² · M · Dₖ²`

**Parameters:** `K² · M`

### PyTorch Implementation

```python
import torch.nn as nn

# Depthwise conv: groups = in_channels
depthwise = nn.Conv2d(
    in_channels=32,
    out_channels=32,       # Same as in_channels
    kernel_size=3,
    padding=1,
    groups=32              # KEY: groups = in_channels
)
# Parameters: 3 × 3 × 32 = 288 (vs 3 × 3 × 32 × 32 = 9,216 for standard)
```

---

## 3. Pointwise Convolution (1×1 Convolution)

A **1×1 convolution** that combines information across channels.

```
Pointwise Convolution:

All M feature maps
    │
    ▼
┌──────────────────┐
│  N filters of    │   Each filter: 1 × 1 × M
│  size 1×1×M      │   Linear combination of channels
└──────────────────┘
    │
    ▼
N output feature maps
```

**Computational cost:** `M · N · Dₖ²`

**Parameters:** `M · N`

```python
# Pointwise conv: 1×1 convolution
pointwise = nn.Conv2d(
    in_channels=32,
    out_channels=64,
    kernel_size=1          # 1×1 convolution
)
# Parameters: 1 × 1 × 32 × 64 = 2,048
```

---

## 4. Depthwise Separable = Depthwise + Pointwise

```
┌─────────────┐     ┌───────────────┐     ┌──────────────┐
│   Input     │────►│   Depthwise   │────►│  Pointwise   │────► Output
│  Dₖ×Dₖ×M   │     │   K×K conv    │     │  1×1 conv    │     Dₖ×Dₖ×N
│             │     │  (per channel)│     │ (cross chan) │
└─────────────┘     └───────────────┘     └──────────────┘
                     Cost: K²·M·Dₖ²        Cost: M·N·Dₖ²

Total cost: K²·M·Dₖ² + M·N·Dₖ² = M·Dₖ²·(K² + N)
```

### Computational Savings

```
Standard cost:     K² · M · N · Dₖ²
Separable cost:    K² · M · Dₖ²  +  M · N · Dₖ²

Ratio = Separable / Standard
      = (K² + N) / (K² · N)
      = 1/N + 1/K²
```

**For K=3, N=256:**
```
Ratio = 1/256 + 1/9 ≈ 0.115
```

**~8.5x fewer computations!** (and similar parameter reduction)

### Numerical Example

| | Standard 3×3 | Depthwise Separable |
|---|---|---|
| Input | 14×14×256 | 14×14×256 |
| Output | 14×14×256 | 14×14×256 |
| **FLOPs** | **115.6M** | **13.6M** |
| **Parameters** | **589,824** | **68,096** |
| **Reduction** | 1× | **~8.5×** |

---

## 5. Complete PyTorch Implementation

```python
import torch
import torch.nn as nn

class DepthwiseSeparableConv(nn.Module):
    """Depthwise Separable Convolution block."""
    
    def __init__(self, in_channels, out_channels, kernel_size=3, 
                 stride=1, padding=1):
        super().__init__()
        
        # Depthwise: groups=in_channels, out_channels=in_channels
        self.depthwise = nn.Conv2d(
            in_channels, in_channels, kernel_size,
            stride=stride, padding=padding, groups=in_channels, bias=False
        )
        self.bn1 = nn.BatchNorm2d(in_channels)
        
        # Pointwise: 1×1 convolution
        self.pointwise = nn.Conv2d(
            in_channels, out_channels, kernel_size=1, bias=False
        )
        self.bn2 = nn.BatchNorm2d(out_channels)
        
        self.relu = nn.ReLU(inplace=True)
    
    def forward(self, x):
        x = self.relu(self.bn1(self.depthwise(x)))
        x = self.relu(self.bn2(self.pointwise(x)))
        return x

# Usage
dw_conv = DepthwiseSeparableConv(64, 128)
x = torch.randn(1, 64, 32, 32)
out = dw_conv(x)
print(f"Output shape: {out.shape}")  # [1, 128, 32, 32]

# Parameter comparison
standard = nn.Conv2d(64, 128, 3, padding=1)
print(f"Standard params:  {sum(p.numel() for p in standard.parameters()):,}")
print(f"DW-Sep params:    {sum(p.numel() for p in dw_conv.parameters()):,}")
# Standard params:  73,856
# DW-Sep params:    9,024
```

---

## 6. Where Depthwise Separable Convolutions Are Used

| Architecture | Usage |
|---|---|
| **MobileNetV1** | All conv layers replaced with depthwise separable |
| **MobileNetV2** | Inverted residuals with depthwise separable |
| **Xception** | Modified Inception with depthwise separable |
| **EfficientNet** | MBConv blocks (inverted residual + depthwise separable) |
| **ShuffleNet** | Combined with channel shuffle |

---

## 7. Inverted Residual Block (MobileNetV2)

```
Input (thin)
    │
    ▼
┌───────────────┐
│ 1×1 Conv      │  ← Expansion (increase channels by factor t)
│ BN + ReLU6    │
├───────────────┤
│ 3×3 Depthwise │  ← Depthwise (spatial filtering)
│ BN + ReLU6    │
├───────────────┤
│ 1×1 Conv      │  ← Projection (reduce channels back)
│ BN (no ReLU!) │     Linear bottleneck
└───────────────┘
    │
    ▼
Output (thin) + Residual connection (if stride=1 and same dims)
```

```python
class InvertedResidual(nn.Module):
    def __init__(self, in_ch, out_ch, stride, expand_ratio):
        super().__init__()
        hidden = in_ch * expand_ratio
        self.use_residual = (stride == 1 and in_ch == out_ch)
        
        layers = []
        if expand_ratio != 1:
            layers += [nn.Conv2d(in_ch, hidden, 1, bias=False),
                       nn.BatchNorm2d(hidden), nn.ReLU6(inplace=True)]
        layers += [
            nn.Conv2d(hidden, hidden, 3, stride, 1, groups=hidden, bias=False),
            nn.BatchNorm2d(hidden), nn.ReLU6(inplace=True),
            nn.Conv2d(hidden, out_ch, 1, bias=False),
            nn.BatchNorm2d(out_ch)
        ]
        self.conv = nn.Sequential(*layers)
    
    def forward(self, x):
        if self.use_residual:
            return x + self.conv(x)
        return self.conv(x)
```

---

## Summary Table

| Aspect | Standard Conv | Depthwise Separable |
|---|---|---|
| **Operation** | Single K×K×M×N | Depthwise K×K + Pointwise 1×1 |
| **FLOPs** | K²·M·N·Dₖ² | M·Dₖ²·(K²+N) |
| **Reduction** | 1× | ~8-9× (for K=3) |
| **Cross-channel** | Yes (built in) | Only in pointwise step |
| **Accuracy** | Higher | Slightly lower (but close) |
| **Use case** | Standard models | Mobile/edge deployment |

---

## Quick Revision Questions

1. **What two operations make up a depthwise separable convolution?**
   - Depthwise convolution (per-channel K×K) and pointwise convolution (1×1 cross-channel)

2. **How do you set up depthwise conv in PyTorch?**
   - Set `groups=in_channels` in `nn.Conv2d`

3. **What is the computational reduction ratio for K=3, N=128?**
   - 1/N + 1/K² = 1/128 + 1/9 ≈ 0.119, so ~8.4× reduction

4. **Why is pointwise conv needed after depthwise?**
   - Depthwise doesn't mix information between channels; pointwise creates cross-channel combinations

5. **What is the "inverted residual" in MobileNetV2?**
   - Expand channels with 1×1, apply depthwise 3×3, project back with 1×1 (thin→wide→thin, opposite of ResNet bottleneck)

6. **Why does the inverted residual use linear activation in the projection?**
   - ReLU on low-dimensional features destroys information; linear bottleneck preserves it

---

[← Previous: Understanding What CNNs Learn](../../09-Visualizing-CNNs/06-understanding-cnns-learn.md) | [Next: Dilated/Atrous Convolutions →](02-dilated-atrous-conv.md)
