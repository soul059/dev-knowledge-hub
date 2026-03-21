# Attention Mechanisms in CNNs

[← Previous: Deformable Convolutions](03-deformable-conv.md) | [Next: Neural Style Transfer →](05-neural-style-transfer.md)

---

## Overview

Attention mechanisms allow CNNs to **selectively focus** on important features — by channel (what features matter), by spatial location (where to look), or both. This chapter covers **Squeeze-and-Excitation (SE)**, **CBAM**, **Non-Local Networks**, and how attention bridges the gap between CNNs and Transformers.

---

## 1. Why Attention in CNNs?

Standard convolution treats all channels and spatial locations **equally**. But:
- Not all feature channels are equally informative for a given input
- Not all spatial regions are equally important
- Attention allows the network to **amplify relevant features** and **suppress irrelevant ones**

```
Standard Conv:                     Conv + Attention:

All channels weighted equally      Important channels amplified
┌──┬──┬──┬──┬──┬──┐               ┌──┬──┬══┬──┬══┬──┐
│  │  │  │  │  │  │               │  │  ║██║  ║██║  │
└──┴──┴──┴──┴──┴──┘               └──┴──┴══┴──┴══┴──┘
                                    ║██║ = high attention weight
```

---

## 2. Channel Attention: Squeeze-and-Excitation (SE)

### SE-Net (Hu et al., 2018) — Won ImageNet 2017

**Idea:** Learn to recalibrate channel-wise feature responses.

```
Input Feature Map U (H × W × C)
         │
         ▼
┌─────────────────────┐
│  Global Avg Pool    │  ← Squeeze: H×W×C → 1×1×C
│  (per channel)      │     Aggregate spatial info
└─────────────────────┘
         │  (1×1×C)
         ▼
┌─────────────────────┐
│  FC → ReLU → FC     │  ← Excitation: learn channel
│  (C → C/r → C)      │     importance weights
│  → Sigmoid           │
└─────────────────────┘
         │  (1×1×C) — channel weights s
         ▼
┌─────────────────────┐
│  Scale: U × s       │  ← Rescale: multiply each
│  (channel-wise)     │     channel by its weight
└─────────────────────┘
         │
         ▼
Output (H × W × C)
```

### PyTorch Implementation

```python
import torch
import torch.nn as nn

class SEBlock(nn.Module):
    """Squeeze-and-Excitation Block."""
    
    def __init__(self, channels, reduction=16):
        super().__init__()
        self.squeeze = nn.AdaptiveAvgPool2d(1)  # H×W → 1×1
        self.excitation = nn.Sequential(
            nn.Linear(channels, channels // reduction, bias=False),
            nn.ReLU(inplace=True),
            nn.Linear(channels // reduction, channels, bias=False),
            nn.Sigmoid()
        )
    
    def forward(self, x):
        b, c, _, _ = x.shape
        # Squeeze: [B, C, H, W] → [B, C]
        s = self.squeeze(x).view(b, c)
        # Excitation: [B, C] → [B, C] (channel weights)
        s = self.excitation(s).view(b, c, 1, 1)
        # Scale: channel-wise multiplication
        return x * s

# SE-ResNet Block
class SEResBlock(nn.Module):
    def __init__(self, channels):
        super().__init__()
        self.conv = nn.Sequential(
            nn.Conv2d(channels, channels, 3, padding=1, bias=False),
            nn.BatchNorm2d(channels), nn.ReLU(inplace=True),
            nn.Conv2d(channels, channels, 3, padding=1, bias=False),
            nn.BatchNorm2d(channels)
        )
        self.se = SEBlock(channels)
        self.relu = nn.ReLU(inplace=True)
    
    def forward(self, x):
        residual = x
        out = self.conv(x)
        out = self.se(out)       # Apply channel attention
        return self.relu(out + residual)
```

**Parameter overhead:** Only `2 × C²/r` extra parameters per block (~minimal).

---

## 3. Spatial Attention

Focus on **where** in the spatial domain to attend.

```
Input (H × W × C)
    │
    ▼
┌────────────────────┐
│ Max Pool + Avg Pool│  ← Aggregate across channels
│ along channel axis │     → 2 maps of H × W
├────────────────────┤
│ Conv 7×7 → Sigmoid│  ← Learn spatial weights
└────────────────────┘
    │  (H × W × 1) — spatial attention map
    ▼
Scale: Input × spatial weights (broadcast across channels)
```

```python
class SpatialAttention(nn.Module):
    def __init__(self, kernel_size=7):
        super().__init__()
        padding = kernel_size // 2
        self.conv = nn.Sequential(
            nn.Conv2d(2, 1, kernel_size, padding=padding, bias=False),
            nn.Sigmoid()
        )
    
    def forward(self, x):
        # Channel-wise pooling: [B, C, H, W] → [B, 2, H, W]
        max_pool = torch.max(x, dim=1, keepdim=True)[0]
        avg_pool = torch.mean(x, dim=1, keepdim=True)
        pooled = torch.cat([max_pool, avg_pool], dim=1)
        
        # Spatial attention map: [B, 1, H, W]
        attention = self.conv(pooled)
        return x * attention
```

---

## 4. CBAM: Convolutional Block Attention Module

**CBAM** (Woo et al., 2018) combines **channel + spatial** attention sequentially:

```
Input
  │
  ▼
┌─────────────────┐
│ Channel Attention│  ← "What" to focus on
│ (SE-style)       │
└─────────────────┘
  │
  ▼
┌─────────────────┐
│ Spatial Attention│  ← "Where" to focus
│ (7×7 conv)       │
└─────────────────┘
  │
  ▼
Output (refined features)
```

```python
class CBAM(nn.Module):
    def __init__(self, channels, reduction=16, spatial_kernel=7):
        super().__init__()
        # Channel attention
        self.channel_att = nn.Sequential(
            nn.AdaptiveAvgPool2d(1),
        )
        self.mlp = nn.Sequential(
            nn.Linear(channels, channels // reduction, bias=False),
            nn.ReLU(inplace=True),
            nn.Linear(channels // reduction, channels, bias=False)
        )
        self.max_pool = nn.AdaptiveMaxPool2d(1)
        
        # Spatial attention
        self.spatial_att = nn.Sequential(
            nn.Conv2d(2, 1, spatial_kernel, padding=spatial_kernel//2, bias=False),
            nn.Sigmoid()
        )
    
    def forward(self, x):
        b, c, _, _ = x.shape
        
        # Channel attention (avg + max pooling, shared MLP)
        avg_out = self.mlp(self.channel_att(x).view(b, c))
        max_out = self.mlp(self.max_pool(x).view(b, c))
        channel_weights = torch.sigmoid(avg_out + max_out).view(b, c, 1, 1)
        x = x * channel_weights
        
        # Spatial attention
        max_pool = torch.max(x, dim=1, keepdim=True)[0]
        avg_pool = torch.mean(x, dim=1, keepdim=True)
        spatial_weights = self.spatial_att(
            torch.cat([max_pool, avg_pool], dim=1)
        )
        x = x * spatial_weights
        
        return x
```

---

## 5. Self-Attention / Non-Local Neural Networks

**Non-Local Networks** (Wang et al., 2018) apply **self-attention** to capture long-range dependencies:

```
Input X (H×W×C)
    │
    ├── Query: Wq·X  (1×1 conv)
    ├── Key:   Wk·X  (1×1 conv)  
    └── Value: Wv·X  (1×1 conv)
    
    Attention = softmax(Q^T · K / √d)
    Output = Attention · V
    
    Final = Wz · Output + X  (residual)
```

```python
class NonLocalBlock(nn.Module):
    def __init__(self, channels, reduction=2):
        super().__init__()
        inter_channels = channels // reduction
        
        self.query = nn.Conv2d(channels, inter_channels, 1)
        self.key = nn.Conv2d(channels, inter_channels, 1)
        self.value = nn.Conv2d(channels, inter_channels, 1)
        self.out = nn.Conv2d(inter_channels, channels, 1)
        
        nn.init.zeros_(self.out.weight)
        nn.init.zeros_(self.out.bias)
    
    def forward(self, x):
        B, C, H, W = x.shape
        
        q = self.query(x).view(B, -1, H*W).permute(0, 2, 1)  # [B, HW, C']
        k = self.key(x).view(B, -1, H*W)                      # [B, C', HW]
        v = self.value(x).view(B, -1, H*W).permute(0, 2, 1)   # [B, HW, C']
        
        attn = torch.softmax(q @ k / (q.shape[-1] ** 0.5), dim=-1)  # [B, HW, HW]
        out = (attn @ v).permute(0, 2, 1).view(B, -1, H, W)         # [B, C', H, W]
        
        return x + self.out(out)  # Residual connection
```

**Cost:** O(H²W²) — expensive for high-resolution feature maps. Typically used in deeper (smaller spatial) layers.

---

## 6. Comparison of Attention Mechanisms

| Mechanism | Attention Type | Extra Params | Compute Cost | Key Benefit |
|---|---|:---:|:---:|---|
| **SE-Net** | Channel | ~2C²/r | Negligible | Channel recalibration |
| **Spatial Att.** | Spatial | Minimal | Negligible | Spatial focus |
| **CBAM** | Channel + Spatial | ~2C²/r + conv | Negligible | Both what and where |
| **Non-Local** | Self-attention | ~3C²/r | O(H²W²) | Long-range dependencies |
| **ECA-Net** | Channel (efficient) | K params | Negligible | No FC, 1D conv instead |

---

## 7. ECA-Net: Efficient Channel Attention

Replaces SE's FC layers with a 1D convolution — even more lightweight:

```python
class ECABlock(nn.Module):
    def __init__(self, channels, gamma=2, b=1):
        super().__init__()
        # Adaptive kernel size
        import math
        k = int(abs(math.log2(channels) / gamma + b / gamma))
        k = k if k % 2 else k + 1
        
        self.avg_pool = nn.AdaptiveAvgPool2d(1)
        self.conv = nn.Conv1d(1, 1, kernel_size=k, padding=k//2, bias=False)
        self.sigmoid = nn.Sigmoid()
    
    def forward(self, x):
        y = self.avg_pool(x).squeeze(-1).transpose(-1, -2)  # [B, 1, C]
        y = self.sigmoid(self.conv(y)).transpose(-1, -2).unsqueeze(-1)  # [B, C, 1, 1]
        return x * y
```

---

## Summary Table

| Concept | Description |
|---|---|
| **Channel Attention** | Learn which feature channels are important (SE-Net) |
| **Spatial Attention** | Learn which spatial locations are important |
| **CBAM** | Sequential channel → spatial attention |
| **Non-Local/Self-Attention** | Capture long-range dependencies (expensive) |
| **Key Trend** | Attention bridges CNNs → Transformers (ViT) |

---

## Quick Revision Questions

1. **What does SE-Net's "squeeze" operation do?**
   - Global average pooling to compress H×W×C → 1×1×C, aggregating spatial information per channel

2. **How does CBAM combine channel and spatial attention?**
   - Sequentially: first apply channel attention (what), then spatial attention (where)

3. **What is the computational bottleneck of non-local blocks?**
   - O(H²W²) attention computation — quadratic in spatial resolution

4. **What does the reduction ratio `r` control in SE-Net?**
   - The bottleneck size in the excitation MLP (C → C/r → C), balancing expressiveness vs parameters

5. **Why initialize the output conv of non-local blocks to zero?**
   - So the block starts as an identity mapping (residual connection only), then gradually learns attention

6. **How does ECA-Net improve on SE-Net?**
   - Replaces FC layers with 1D convolution across channels — fewer parameters, comparable or better performance

---

[← Previous: Deformable Convolutions](03-deformable-conv.md) | [Next: Neural Style Transfer →](05-neural-style-transfer.md)
