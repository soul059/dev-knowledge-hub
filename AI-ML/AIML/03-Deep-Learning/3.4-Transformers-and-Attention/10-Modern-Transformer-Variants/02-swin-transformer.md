# Swin Transformer: Shifted Window Attention

> **Navigation:**
> Previous: [← Vision Transformer](01-vision-transformer.md) | Next: [Sparse Attention →](03-sparse-attention.md)

---

## Overview

The Swin Transformer, introduced by Liu et al. in 2021, addresses two critical limitations of the
original Vision Transformer (ViT): its quadratic computational complexity with respect to image
size and its lack of multi-scale feature maps. Swin achieves this through two key innovations —
**window-based self-attention** that restricts attention to local windows for linear complexity, and
**shifted windows** that enable cross-window information flow without extra cost. Combined with a
**hierarchical architecture** using patch merging, Swin produces multi-scale feature maps akin to
CNNs, making it suitable as a general-purpose backbone for tasks like detection and segmentation
that require dense predictions at multiple resolutions.

---

## 1. Motivation: Limitations of ViT

```
Standard ViT Attention:
  Every patch attends to EVERY other patch
  Complexity: O(N²) where N = number of patches

  For 224×224 image, P=16:  N = 196    → manageable
  For 1024×1024 image, P=4: N = 65,536 → O(N²) = 4.3 BILLION operations!

Swin Transformer Solution:
  Attention within LOCAL WINDOWS only
  Complexity: O(N · M²) where M = window size (fixed, e.g., 7)
  This is LINEAR in image size!
```

---

## 2. Window-Based Self-Attention (W-MSA)

### 2.1 Window Partitioning

The feature map of size **(H/P × W/P)** is divided into non-overlapping windows of size **M × M**:

```
Number of windows = (H/P × W/P) / M²

For a 56×56 feature map with M = 7:
    Windows = (56 × 56) / (7 × 7) = 64 windows
    Each window contains 49 tokens
```

### 2.2 Attention Within Windows

Self-attention is computed **independently within each window**:

```
Attention(Q, K, V) = softmax(Q K^T / √d + B) · V

where B ∈ ℝ^(M² × M²) is a learnable relative position bias
```

### 2.3 Complexity Analysis

```
Standard Global MSA:
    Ω(MSA) = 4hwC² + 2(hw)²C

Window-based MSA (W-MSA):
    Ω(W-MSA) = 4hwC² + 2M²hwC

where:
    h × w  = feature map spatial resolution
    C      = embedding dimension
    M      = window size

The key difference:
    (hw)²C   →  M²hwC
    Quadratic     Linear in hw (since M is fixed)
```

---

## 3. Shifted Window Mechanism (SW-MSA)

### 3.1 The Cross-Window Problem

Window-based attention isolates each window — tokens in adjacent windows cannot communicate.

### 3.2 Solution: Shift the Window Grid

In alternating layers, the window partition is **shifted by (⌊M/2⌋, ⌊M/2⌋) pixels**:

```
Layer ℓ:     Regular Windows (W-MSA)
Layer ℓ+1:   Shifted Windows (SW-MSA)

┌────────┬────────┬────────┬────────┐
│        │        │        │        │   Layer ℓ: Regular windows
│  W1    │  W2    │  W3    │  W4    │   Each window computes
│        │        │        │        │   self-attention independently
├────────┼────────┼────────┼────────┤
│        │        │        │        │
│  W5    │  W6    │  W7    │  W8    │
│        │        │        │        │
├────────┼────────┼────────┼────────┤
│        │        │        │        │
│  W9    │  W10   │  W11   │  W12   │
│        │        │        │        │
├────────┼────────┼────────┼────────┤
│        │        │        │        │
│  W13   │  W14   │  W15   │  W16   │
│        │        │        │        │
└────────┴────────┴────────┴────────┘

   ┌─────┬───────────┬───────────┬──────┐
   │     │           │           │      │   Layer ℓ+1: SHIFTED windows
   │     │           │           │      │   Shifted by (M/2, M/2)
   ├─────┼───────────┼───────────┼──────┤   Tokens from DIFFERENT original
   │     │           │           │      │   windows now share a window
   │     │  SW1      │  SW2      │      │   → enables cross-window
   │     │           │           │      │   communication!
   ├─────┼───────────┼───────────┼──────┤
   │     │           │           │      │
   │     │  SW3      │  SW4      │      │
   │     │           │           │      │
   ├─────┼───────────┼───────────┼──────┤
   │     │           │           │      │
   └─────┴───────────┴───────────┴──────┘
```

### 3.3 Efficient Cyclic Shift

To avoid creating variable-sized windows at borders, Swin uses **cyclic shifting** with
**attention masking**:

```
Step 1: Cyclically shift the feature map by (-M/2, -M/2)
Step 2: Compute window attention with masking
        (mask prevents attention between non-adjacent regions)
Step 3: Reverse the cyclic shift

This keeps the number of windows constant and avoids padding.
```

---

## 4. Hierarchical Architecture

### 4.1 Patch Merging

To create multi-scale feature maps, Swin uses **patch merging** between stages:

```
Stage 1: (H/4 × W/4) tokens, dim C          (e.g., 56×56, C=96)
           ↓ Patch Merging: 2×2 neighbors → concat → Linear
Stage 2: (H/8 × W/8) tokens, dim 2C         (e.g., 28×28, C=192)
           ↓ Patch Merging
Stage 3: (H/16 × W/16) tokens, dim 4C       (e.g., 14×14, C=384)
           ↓ Patch Merging
Stage 4: (H/32 × W/32) tokens, dim 8C       (e.g., 7×7, C=768)
```

### 4.2 Full Architecture Diagram

```
INPUT IMAGE (224 × 224 × 3)
         │
         ▼
┌──────────────────────────────────┐
│  Patch Partition (4×4 patches)   │
│  → (56 × 56) tokens, dim 48     │
│  Linear Embedding → dim C=96    │
└──────────────────────────────────┘
         │
         ▼
┌──────────────────────────────────┐
│  STAGE 1: 2× Swin Blocks        │
│  W-MSA → SW-MSA (window M=7)    │
│  Resolution: 56 × 56, dim 96    │
└──────────────────────────────────┘
         │ Patch Merging (2× downsample)
         ▼
┌──────────────────────────────────┐
│  STAGE 2: 2× Swin Blocks        │
│  W-MSA → SW-MSA                 │
│  Resolution: 28 × 28, dim 192   │
└──────────────────────────────────┘
         │ Patch Merging
         ▼
┌──────────────────────────────────┐
│  STAGE 3: 6× Swin Blocks        │
│  W-MSA → SW-MSA (×3 pairs)      │
│  Resolution: 14 × 14, dim 384   │
└──────────────────────────────────┘
         │ Patch Merging
         ▼
┌──────────────────────────────────┐
│  STAGE 4: 2× Swin Blocks        │
│  W-MSA → SW-MSA                 │
│  Resolution: 7 × 7, dim 768     │
└──────────────────────────────────┘
         │
         ▼
    Global Average Pool → Classification Head
```

---

## 5. Relative Position Bias

Unlike ViT's absolute position embeddings, Swin uses **relative position bias**:

```
Attention(Q, K, V) = softmax(QK^T / √d  +  B) · V

B is indexed from a bias table B̂ ∈ ℝ^((2M-1) × (2M-1))

For M = 7:
    Relative positions range from -6 to +6 in each axis
    Bias table size: 13 × 13 = 169 learnable parameters per head

Advantage:
    - Generalizes better across window sizes
    - Encodes relative spatial relationships
    - More flexible than absolute position embeddings
```

---

## 6. Swin Model Variants

| Model     | C   | Layers          | Heads         | Params | FLOPs  |
|-----------|-----|-----------------|---------------|--------|--------|
| Swin-T    | 96  | [2, 2, 6, 2]   | [3, 6, 12, 24]| 29M   | 4.5G   |
| Swin-S    | 96  | [2, 2, 18, 2]  | [3, 6, 12, 24]| 50M   | 8.7G   |
| Swin-B    | 128 | [2, 2, 18, 2]  | [4, 8, 16, 32]| 88M   | 15.4G  |
| Swin-L    | 192 | [2, 2, 18, 2]  | [6, 12, 24, 48]| 197M  | 34.5G  |

---

## 7. Worked Example

**Setting:** Swin-T, input 224×224×3, window size M = 7.

```
Step 1: Patch partition (4×4)
    Feature map: (56, 56), dim = 48
    Linear embed → dim C = 96
    Tokens: 56 × 56 = 3,136

Step 2: Stage 1 — W-MSA + SW-MSA
    Windows: 3136 / 49 = 64 windows, each 7×7 = 49 tokens
    Attention per window: 49 × 49 = 2,401 (vs global: 3136² = 9.8M)
    Savings: 4,096× fewer attention computations!

Step 3: Patch Merging after Stage 1
    Group 2×2 neighbors → concat: dim = 4 × 96 = 384
    Linear: 384 → 192
    New resolution: 28 × 28, dim 192

Step 4: Stage 2 → Stage 3 → Stage 4
    28×28 → 14×14 → 7×7
    Dim: 192 → 384 → 768

Step 5: Output
    7 × 7 = 49 tokens, dim 768
    Global average pool → (768,) → Classification head
```

---

## 8. PyTorch Implementation

```python
import torch
import torch.nn as nn
import torch.nn.functional as F


def window_partition(x, window_size):
    """Partition feature map into non-overlapping windows.

    Args:
        x: (B, H, W, C)
        window_size: int (M)
    Returns:
        windows: (num_windows * B, M, M, C)
    """
    B, H, W, C = x.shape
    x = x.view(B, H // window_size, window_size, W // window_size, window_size, C)
    windows = x.permute(0, 1, 3, 2, 4, 5).contiguous()
    windows = windows.view(-1, window_size, window_size, C)
    return windows


def window_reverse(windows, window_size, H, W):
    """Reverse window partition back to feature map.

    Args:
        windows: (num_windows * B, M, M, C)
        window_size: int
        H, W: original spatial dimensions
    Returns:
        x: (B, H, W, C)
    """
    B = int(windows.shape[0] / (H * W / window_size / window_size))
    x = windows.view(B, H // window_size, W // window_size,
                      window_size, window_size, -1)
    x = x.permute(0, 1, 3, 2, 4, 5).contiguous().view(B, H, W, -1)
    return x


class WindowAttention(nn.Module):
    """Window-based multi-head self-attention with relative position bias."""

    def __init__(self, dim, window_size, num_heads):
        super().__init__()
        self.dim = dim
        self.window_size = window_size  # (M, M)
        self.num_heads = num_heads
        head_dim = dim // num_heads
        self.scale = head_dim ** -0.5

        # Relative position bias table
        self.relative_position_bias_table = nn.Parameter(
            torch.zeros((2 * window_size - 1) * (2 * window_size - 1), num_heads)
        )
        nn.init.trunc_normal_(self.relative_position_bias_table, std=0.02)

        # Compute relative position index
        coords_h = torch.arange(window_size)
        coords_w = torch.arange(window_size)
        coords = torch.stack(torch.meshgrid(coords_h, coords_w, indexing='ij'))  # (2, M, M)
        coords_flatten = torch.flatten(coords, 1)  # (2, M*M)

        relative_coords = coords_flatten[:, :, None] - coords_flatten[:, None, :]  # (2, M², M²)
        relative_coords = relative_coords.permute(1, 2, 0).contiguous()
        relative_coords[:, :, 0] += window_size - 1
        relative_coords[:, :, 1] += window_size - 1
        relative_coords[:, :, 0] *= 2 * window_size - 1
        relative_position_index = relative_coords.sum(-1)  # (M², M²)
        self.register_buffer("relative_position_index", relative_position_index)

        self.qkv = nn.Linear(dim, dim * 3)
        self.proj = nn.Linear(dim, dim)

    def forward(self, x, mask=None):
        B_, N, C = x.shape  # N = M*M
        qkv = self.qkv(x).reshape(B_, N, 3, self.num_heads, C // self.num_heads)
        qkv = qkv.permute(2, 0, 3, 1, 4)
        q, k, v = qkv.unbind(0)

        q = q * self.scale
        attn = q @ k.transpose(-2, -1)

        # Add relative position bias
        relative_position_bias = self.relative_position_bias_table[
            self.relative_position_index.view(-1)
        ].view(N, N, -1)
        relative_position_bias = relative_position_bias.permute(2, 0, 1).contiguous()
        attn = attn + relative_position_bias.unsqueeze(0)

        if mask is not None:
            attn = attn.masked_fill(mask == 0, float('-inf'))

        attn = F.softmax(attn, dim=-1)
        x = (attn @ v).transpose(1, 2).reshape(B_, N, C)
        x = self.proj(x)
        return x


class SwinBlock(nn.Module):
    """A single Swin Transformer block with W-MSA or SW-MSA."""

    def __init__(self, dim, num_heads, window_size=7, shift_size=0, mlp_ratio=4.0):
        super().__init__()
        self.shift_size = shift_size
        self.window_size = window_size

        self.norm1 = nn.LayerNorm(dim)
        self.attn = WindowAttention(dim, window_size, num_heads)
        self.norm2 = nn.LayerNorm(dim)
        mlp_hidden = int(dim * mlp_ratio)
        self.mlp = nn.Sequential(
            nn.Linear(dim, mlp_hidden),
            nn.GELU(),
            nn.Linear(mlp_hidden, dim),
        )

    def forward(self, x, H, W):
        B, L, C = x.shape
        shortcut = x
        x = self.norm1(x)
        x = x.view(B, H, W, C)

        # Cyclic shift
        if self.shift_size > 0:
            x = torch.roll(x, shifts=(-self.shift_size, -self.shift_size), dims=(1, 2))

        # Window partition
        x_windows = window_partition(x, self.window_size)  # (nW*B, M, M, C)
        x_windows = x_windows.view(-1, self.window_size * self.window_size, C)

        # W-MSA / SW-MSA
        attn_windows = self.attn(x_windows)

        # Merge windows
        attn_windows = attn_windows.view(-1, self.window_size, self.window_size, C)
        x = window_reverse(attn_windows, self.window_size, H, W)

        # Reverse cyclic shift
        if self.shift_size > 0:
            x = torch.roll(x, shifts=(self.shift_size, self.shift_size), dims=(1, 2))

        x = x.view(B, H * W, C)
        x = shortcut + x

        # MLP
        x = x + self.mlp(self.norm2(x))
        return x


# --- Quick test ---
if __name__ == "__main__":
    B, H, W, C = 2, 56, 56, 96
    x = torch.randn(B, H * W, C)

    block_w = SwinBlock(dim=96, num_heads=3, window_size=7, shift_size=0)
    block_sw = SwinBlock(dim=96, num_heads=3, window_size=7, shift_size=3)

    out = block_w(x, H, W)
    out = block_sw(out, H, W)
    print(f"Input:  ({B}, {H*W}, {C})")
    print(f"Output: {out.shape}")  # (2, 3136, 96) — same shape
```

---

## 9. Swin vs ViT Comparison

| Feature                | ViT                    | Swin Transformer          |
|------------------------|------------------------|---------------------------|
| Attention scope        | Global (all patches)   | Local (within windows)    |
| Complexity             | O(N²)                 | O(N · M²) — linear in N  |
| Feature maps           | Single-scale           | Multi-scale (hierarchical)|
| Position encoding      | Absolute (learned 1D)  | Relative position bias    |
| Cross-window comms     | N/A (global attention) | Shifted windows           |
| Dense prediction tasks | Requires adaptation    | Natively supported        |
| Backbone flexibility   | Classification mainly  | Classification + Det + Seg|

---

## 10. Real-World Applications

| Application                | Description                                           |
|----------------------------|-------------------------------------------------------|
| **Image Classification**   | 87.3% top-1 on ImageNet-1K (Swin-L, 22K pre-train)  |
| **Object Detection**       | COCO: 58.7 box AP with Cascade Mask R-CNN + Swin-L   |
| **Semantic Segmentation**  | ADE20K: 53.5 mIoU with UperNet + Swin-L              |
| **Instance Segmentation**  | COCO: 51.1 mask AP                                    |
| **Video Recognition**      | Video Swin Transformer for action recognition         |
| **3D Medical Imaging**     | Swin UNETR for volumetric segmentation                |

---

## 11. Summary Table

| Concept              | Details                                                     |
|----------------------|-------------------------------------------------------------|
| Paper                | Liu et al., *Swin Transformer* (ICCV 2021, Best Paper)     |
| Key innovation       | Window attention + shifted windows                          |
| Window size          | Typically M = 7                                             |
| Attention complexity | O(N · M²) — linear in image size                           |
| Hierarchical design  | Patch merging creates multi-scale feature maps              |
| Position encoding    | Relative position bias (per-head, learnable)                |
| Cyclic shift trick   | Efficient implementation of shifted windows                 |
| Strengths            | Multi-scale, linear complexity, versatile backbone          |
| Swin v2              | Adds cosine attention, log-spaced CPB, res-post-norm        |

---

## 12. Revision Questions

1. **Why does window-based attention achieve linear complexity?** Show the derivation comparing
   O(N²) global attention to O(N · M²) window attention, and explain why M is constant.

2. **How do shifted windows enable cross-window communication?** Illustrate with a concrete
   example showing which tokens from different windows can now interact.

3. **Explain the cyclic shift trick.** Why is it more efficient than naively computing attention
   on variable-sized shifted windows?

4. **How does patch merging create a hierarchical representation?** Describe the exact operation
   and compare it to pooling in CNNs.

5. **Why does Swin use relative position bias instead of absolute position embeddings?** Discuss
   advantages for transferability across different resolutions.

6. **Compare Swin Transformer and ViT for object detection.** Why is Swin's multi-scale design
   important for tasks requiring dense predictions at multiple scales?

---

> **Navigation:**
> Previous: [← Vision Transformer](01-vision-transformer.md) | Next: [Sparse Attention →](03-sparse-attention.md)
