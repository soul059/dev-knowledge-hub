# Dilated / Atrous Convolutions

[← Previous: Depthwise Separable Convolutions](01-depthwise-separable-conv.md) | [Next: Deformable Convolutions →](03-deformable-conv.md)

---

## Overview

Dilated (atrous) convolutions insert **gaps (holes)** between filter elements, expanding the **receptive field exponentially** without increasing parameters or reducing spatial resolution. They are crucial for **semantic segmentation** (DeepLab) and **sequence modeling** (WaveNet).

---

## 1. What Is Dilation?

Standard convolution samples contiguous positions. Dilated convolution introduces a **dilation rate `d`** that spreads the filter elements apart.

```
Standard Conv (d=1):        Dilated Conv (d=2):         Dilated Conv (d=4):
3×3 filter, RF = 3×3        3×3 filter, RF = 5×5        3×3 filter, RF = 9×9

■ ■ ■                       ■ · ■ · ■                   ■ · · · ■ · · · ■
■ ■ ■                       · · · · ·                   · · · · · · · · ·
■ ■ ■                       ■ · ■ · ■                   · · · · · · · · ·
                            · · · · ·                   · · · · · · · · ·
                            ■ · ■ · ■                   ■ · · · ■ · · · ■
                                                        · · · · · · · · ·
■ = sampled position                                    · · · · · · · · ·
· = skipped position                                    · · · · · · · · ·
                                                        ■ · · · ■ · · · ■
```

**Effective receptive field size:** `K + (K-1)(d-1)` or equivalently `d(K-1) + 1`

| Dilation Rate | 3×3 Filter RF | Parameters |
|:---:|:---:|:---:|
| d=1 | 3×3 | 9 |
| d=2 | 5×5 | 9 |
| d=4 | 9×9 | 9 |
| d=8 | 17×17 | 9 |

**Same 9 parameters**, exponentially growing receptive field!

---

## 2. Mathematical Formulation

For a 1D dilated convolution with dilation rate `d`:

```
(F *_d x)[i] = Σₖ F[k] · x[i + d·k]
```

For 2D:
```
(F *_d x)[i,j] = Σₘ Σₙ F[m,n] · x[i + d·m, j + d·n]
```

When d=1, this reduces to standard convolution.

---

## 3. Output Size Formula

```
Output = floor((Input + 2·Padding - Dilation·(K-1) - 1) / Stride + 1)
```

For **same padding** with dilation:
```
Padding = dilation × (kernel_size - 1) / 2
```

---

## 4. Why Dilated Convolutions?

### The Problem: Segmentation Needs Large RF + Full Resolution

```
Standard approach:          Dilated approach:
Conv → Pool → Conv → Pool   Conv(d=1) → Conv(d=2) → Conv(d=4)
→ Upsample                 
                            
Resolution: ↓↓ then ↑↑      Resolution: stays SAME
Information loss!            No information loss!
```

| Approach | Receptive Field | Resolution | Info Loss |
|---|:---:|:---:|:---:|
| Stacking standard convs | Grows linearly | Maintained | None (but slow RF growth) |
| Pooling + upsampling | Large after pool | Reduced, then recovered | Yes |
| **Dilated convolutions** | **Grows exponentially** | **Maintained** | **None** |

---

## 5. Exponential Receptive Field Growth

Stack dilated convolutions with exponentially increasing rates:

```
Layer 1: d=1,  RF = 3
Layer 2: d=2,  RF = 7     (3 + 2×2)
Layer 3: d=4,  RF = 15    (7 + 2×4)
Layer 4: d=8,  RF = 31    (15 + 2×8)
Layer 5: d=16, RF = 63    (31 + 2×16)

After 5 layers: 63×63 receptive field with only 5×9 = 45 parameters!
vs standard 3×3: would need 31 layers for same RF
```

---

## 6. Atrous Spatial Pyramid Pooling (ASPP)

Used in **DeepLab** for multi-scale feature extraction:

```
                    ┌─── Conv d=1  (local) ───┐
                    │                          │
Input ──────────────├─── Conv d=6  (medium) ───├──► Concatenate ──► 1×1 Conv
                    │                          │
                    ├─── Conv d=12 (wide)  ────┤
                    │                          │
                    └─── Conv d=18 (wider) ────┘
                    │                          │
                    └─── Global Avg Pool   ────┘
```

Captures features at multiple scales simultaneously!

```python
import torch
import torch.nn as nn
import torch.nn.functional as F

class ASPP(nn.Module):
    def __init__(self, in_channels, out_channels=256):
        super().__init__()
        
        # 1×1 convolution
        self.conv1x1 = nn.Sequential(
            nn.Conv2d(in_channels, out_channels, 1, bias=False),
            nn.BatchNorm2d(out_channels), nn.ReLU(inplace=True)
        )
        
        # Atrous convolutions at different rates
        rates = [6, 12, 18]
        self.atrous_convs = nn.ModuleList([
            nn.Sequential(
                nn.Conv2d(in_channels, out_channels, 3, 
                          padding=r, dilation=r, bias=False),
                nn.BatchNorm2d(out_channels), nn.ReLU(inplace=True)
            ) for r in rates
        ])
        
        # Global average pooling branch
        self.global_pool = nn.Sequential(
            nn.AdaptiveAvgPool2d(1),
            nn.Conv2d(in_channels, out_channels, 1, bias=False),
            nn.BatchNorm2d(out_channels), nn.ReLU(inplace=True)
        )
        
        # Projection after concatenation
        self.project = nn.Sequential(
            nn.Conv2d(out_channels * 5, out_channels, 1, bias=False),
            nn.BatchNorm2d(out_channels), nn.ReLU(inplace=True),
            nn.Dropout(0.5)
        )
    
    def forward(self, x):
        size = x.shape[2:]
        
        features = [self.conv1x1(x)]
        features += [conv(x) for conv in self.atrous_convs]
        features.append(
            F.interpolate(self.global_pool(x), size=size, mode='bilinear')
        )
        
        return self.project(torch.cat(features, dim=1))
```

---

## 7. PyTorch Dilated Convolution

```python
# Standard convolution
conv_standard = nn.Conv2d(64, 64, kernel_size=3, padding=1, dilation=1)
# Effective RF: 3×3

# Dilated convolution
conv_dilated = nn.Conv2d(64, 64, kernel_size=3, padding=2, dilation=2)
# Effective RF: 5×5, same parameters!

conv_dilated4 = nn.Conv2d(64, 64, kernel_size=3, padding=4, dilation=4)
# Effective RF: 9×9, still same parameters!

# Verify output size maintained
x = torch.randn(1, 64, 32, 32)
print(conv_standard(x).shape)   # [1, 64, 32, 32]
print(conv_dilated(x).shape)    # [1, 64, 32, 32]
print(conv_dilated4(x).shape)   # [1, 64, 32, 32]
```

---

## 8. The Gridding Problem

With large dilation rates, a checkerboard/gridding artifact can occur:

```
d=2: ■ · ■ · ■     Only samples even positions!
     · · · · ·     Misses odd positions entirely
     ■ · ■ · ■
```

**Solutions:**
- **Hybrid Dilated Convolution (HDC):** Use non-power-of-2 rates (e.g., d=1,2,3 instead of 1,2,4)
- **Multi-scale aggregation:** ASPP combines multiple rates
- **Mix dilated with standard convolutions**

---

## 9. Applications

| Application | Architecture | Dilation Use |
|---|---|---|
| **Semantic Segmentation** | DeepLabV3/V3+ | ASPP with d=6,12,18 |
| **Audio Generation** | WaveNet | Causal dilated 1D conv (d=1,2,4,...,512) |
| **Text Classification** | Dilated CNN | Large RF for long sequences |
| **Medical Imaging** | Various U-Nets | Dense prediction without resolution loss |

---

## Summary Table

| Aspect | Standard Conv | Dilated Conv |
|---|---|---|
| **Receptive Field** | K×K | (d(K-1)+1)² |
| **Parameters** | K²·Cᵢₙ·Cₒᵤₜ | Same! |
| **Resolution** | Maintained (or reduced with stride) | Maintained |
| **RF Growth (stacked)** | Linear | Exponential |
| **Key Use** | General feature extraction | Segmentation, large RF tasks |

---

## Quick Revision Questions

1. **What does dilation rate d=4 on a 3×3 filter produce?**
   - An effective receptive field of 9×9 (d(K-1)+1 = 4×2+1 = 9) with only 9 parameters

2. **How does dilated conv differ from simply using a larger filter?**
   - Same parameter count as small filter, but covers larger area by skipping positions

3. **What is the gridding problem?**
   - Large dilation rates sample only specific positions, missing intermediate information

4. **What is ASPP?**
   - Atrous Spatial Pyramid Pooling — parallel dilated convolutions at multiple rates to capture multi-scale features

5. **How do you specify dilation in PyTorch?**
   - `nn.Conv2d(..., dilation=d, padding=d)` — also adjust padding to maintain spatial size

6. **Why are dilated convs preferred over pooling for segmentation?**
   - They increase receptive field without losing spatial resolution, avoiding information loss from downsampling

---

[← Previous: Depthwise Separable Convolutions](01-depthwise-separable-conv.md) | [Next: Deformable Convolutions →](03-deformable-conv.md)
