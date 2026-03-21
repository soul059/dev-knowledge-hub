# Deformable Convolutions

[← Previous: Dilated/Atrous Convolutions](02-dilated-atrous-conv.md) | [Next: Attention in CNNs →](04-attention-in-cnns.md)

---

## Overview

Standard convolutions sample from a **fixed rectangular grid**. Deformable convolutions learn **spatial offsets** for each sampling location, allowing the network to adaptively adjust its receptive field to handle geometric transformations like **scale, rotation, and deformation** — without explicit data augmentation.

---

## 1. The Problem with Fixed Grids

Standard convolution always samples a rigid K×K grid:

```
Standard 3×3 grid (fixed):

    ■ ■ ■
    ■ ■ ■          Always the same pattern,
    ■ ■ ■          regardless of content
```

This is problematic for:
- Objects at **different scales** (small bird vs large airplane)
- **Rotated** or **deformed** objects
- Objects with **varying aspect ratios**

Traditional solutions (multi-scale pyramids, data augmentation) are hand-crafted. Deformable convolutions **learn** the right sampling pattern.

---

## 2. Deformable Convolution Concept

Add a **learnable 2D offset** (Δpₖ) to each sampling position:

```
Standard sampling:              Deformable sampling:

■ ■ ■                           ■   ■     ■
■ ■ ■     →  Learn offsets →        ■  ■
■ ■ ■                             ■    ■  ■

Fixed grid                      Adaptive grid (learned per pixel!)
```

### Mathematical Formulation

**Standard convolution** at output position **p₀**:

```
y(p₀) = Σₖ w(pₖ) · x(p₀ + pₖ)
```

where `pₖ ∈ {(-1,-1), (-1,0), ..., (1,1)}` for a 3×3 filter.

**Deformable convolution:**

```
y(p₀) = Σₖ w(pₖ) · x(p₀ + pₖ + Δpₖ)
```

where `Δpₖ` is the **learned offset** for each sampling position.

Since offsets are typically **fractional**, use **bilinear interpolation**:

```
x(p) = Σ_q G(q, p) · x(q)

where G(q, p) = max(0, 1-|qₓ-pₓ|) · max(0, 1-|q_y-p_y|)
```

---

## 3. Architecture

```
Input Feature Map
    │
    ├──────────────────────────┐
    │                          │
    ▼                          ▼
┌──────────────┐      ┌───────────────┐
│  Offset Conv │      │ Standard Conv │
│  (3×3 conv)  │      │  (3×3 conv)   │
│  Output: 2N  │      │  with learned │
│  (dx,dy per  │─────►│  offsets      │
│   position)  │      │               │
└──────────────┘      └───────────────┘
                           │
                           ▼
                    Output Feature Map
```

The **offset conv** is a regular convolution that produces **2K² channels** (x and y offset for each of K² sampling positions).

---

## 4. Deformable Convolution v2 (DCNv2)

DCNv2 adds **modulation scalars** that learn to weight each sampling point:

```
y(p₀) = Σₖ w(pₖ) · Δmₖ · x(p₀ + pₖ + Δpₖ)
```

where `Δmₖ ∈ [0, 1]` is a learned modulation factor (via sigmoid).

This allows the network to not only **shift** sampling points but also **suppress** irrelevant ones.

```
Offset conv now outputs: 3K² channels
  - 2K² for offsets (dx, dy)
  - K²  for modulation weights
```

---

## 5. PyTorch Implementation

```python
import torch
import torch.nn as nn
from torchvision.ops import DeformConv2d

class DeformableConvBlock(nn.Module):
    def __init__(self, in_channels, out_channels, kernel_size=3, 
                 stride=1, padding=1):
        super().__init__()
        
        self.kernel_size = kernel_size
        offset_channels = 2 * kernel_size * kernel_size  # 2 * K²
        
        # Conv to learn offsets
        self.offset_conv = nn.Conv2d(
            in_channels, offset_channels, kernel_size,
            stride=stride, padding=padding
        )
        nn.init.zeros_(self.offset_conv.weight)
        nn.init.zeros_(self.offset_conv.bias)
        
        # Deformable conv
        self.deform_conv = DeformConv2d(
            in_channels, out_channels, kernel_size,
            stride=stride, padding=padding
        )
        
        self.bn = nn.BatchNorm2d(out_channels)
        self.relu = nn.ReLU(inplace=True)
    
    def forward(self, x):
        offsets = self.offset_conv(x)
        out = self.deform_conv(x, offsets)
        return self.relu(self.bn(out))

# Usage
deform_block = DeformableConvBlock(64, 128)
x = torch.randn(2, 64, 32, 32)
out = deform_block(x)
print(f"Output: {out.shape}")  # [2, 128, 32, 32]
```

---

## 6. Visualizing Learned Offsets

What the network actually learns:

```
Object at center:          Object at edge:           Elongated object:
  
    · ■ ·                      · · ■                  ■ · · · · · ■
    ■ ■ ■                      · ■ ■                  · · · ■ · · ·
    · ■ ·                      ■ ■ ■                  ■ · · · · · ■
    
 Focuses on center        Shifts towards edge       Adapts to shape

Small object:              Large object:
    · · · · ·                  ■ · · · ■
    · ■ ■ ■ ·                  · · · · ·
    · ■ ■ ■ ·                  · · · · ·
    · ■ ■ ■ ·                  · · · · ·
    · · · · ·                  ■ · · · ■
    
 Contracts RF              Expands RF
```

The network learns to **focus sampling on relevant regions** of the input.

---

## 7. Deformable RoI Pooling

Applied to region-of-interest pooling in object detection:

```
Standard RoI Pooling:           Deformable RoI Pooling:

┌─────────────┐                 ┌─────────────┐
│ ■ ■ │ ■ ■   │                 │  ■  │   ■   │
│ ■ ■ │ ■ ■   │  Fixed grid     │■   ■│ ■   ■ │  Learned offsets
├─────┼──────  │   →             ├─────┼─────  │   per bin
│ ■ ■ │ ■ ■   │                 │ ■ ■ │  ■  ■ │
│ ■ ■ │ ■ ■   │                 │■    │    ■  │
└─────────────┘                 └─────────────┘
```

Better handles objects that don't align with rectangular bounding boxes.

---

## 8. Where Deformable Convolutions Help

| Application | Benefit |
|---|---|
| **Object Detection** | Better handle varying object shapes/poses |
| **Semantic Segmentation** | Adapt to object boundaries more precisely |
| **Pose Estimation** | Handle articulated body parts |
| **Action Recognition** | Adapt to temporal deformations in video |

### Results Improvement

In Faster R-CNN (ResNet-50 backbone) on COCO:
- Standard: 37.5 mAP
- With Deformable Conv: **41.2 mAP** (+3.7)

---

## 9. Comparison

| | Standard Conv | Dilated Conv | Deformable Conv |
|---|---|---|---|
| **Sampling pattern** | Fixed grid | Fixed grid with gaps | Learned offsets |
| **Adaptivity** | None | None | Per-pixel learned |
| **Extra parameters** | None | None | Offset conv layer |
| **Extra compute** | None | None | ~10-20% overhead |
| **Geometric handling** | None | Larger RF only | Scale, rotation, deformation |
| **Implementation** | Built-in | Built-in (dilation param) | `torchvision.ops.DeformConv2d` |

---

## Summary Table

| Concept | Description |
|---|---|
| **Core idea** | Learn 2D offsets for each sampling position in convolution |
| **Offsets** | Predicted by a parallel conv layer (2K² output channels) |
| **Interpolation** | Bilinear interpolation for fractional offsets |
| **DCNv2** | Adds modulation scalars (3K² channels) to weight samples |
| **Benefit** | Handles geometric transformations adaptively |
| **Cost** | ~10-20% compute overhead from offset prediction |
| **Init** | Offset conv initialized to zeros (starts as standard conv) |

---

## Quick Revision Questions

1. **What does a deformable convolution learn that standard conv doesn't?**
   - 2D spatial offsets for each sampling position, allowing adaptive receptive field shapes

2. **How are fractional offsets handled?**
   - Bilinear interpolation over the 4 nearest integer positions

3. **What improvement does DCNv2 add?**
   - Modulation scalars (learned weights per sampling point) that can suppress irrelevant positions

4. **How many extra channels does the offset conv produce?**
   - 2K² for DCNv1 (x,y per position), 3K² for DCNv2 (+ modulation weights)

5. **Why initialize offset conv weights to zero?**
   - So the network starts as a standard convolution and gradually learns offsets during training

6. **What is deformable RoI pooling?**
   - Adds learned offsets to each bin in RoI pooling, better handling non-rectangular objects

---

[← Previous: Dilated/Atrous Convolutions](02-dilated-atrous-conv.md) | [Next: Attention in CNNs →](04-attention-in-cnns.md)
