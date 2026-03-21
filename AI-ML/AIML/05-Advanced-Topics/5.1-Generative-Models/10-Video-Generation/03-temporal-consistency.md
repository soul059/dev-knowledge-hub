# Temporal Consistency

## Overview

Temporal consistency is the single most important quality factor that separates good video generation from bad. A temporally consistent video has smooth motion, stable appearance, and no flickering or sudden jumps between frames. This chapter covers why temporal consistency is hard, the mathematical formulations, and the techniques used to achieve it in modern video generation systems.

---

## What is Temporal Consistency?

```
  Temporally consistent video:
    • Objects maintain their appearance across frames
    • Motion is smooth and physically plausible
    • No flickering, popping, or jittering
    • Lighting and colors remain stable
    • Geometry doesn't warp unexpectedly

  Types of inconsistency:

  ┌──────────────────────────────────────────────────┐
  │                                                    │
  │  1. Flickering:                                   │
  │     Frame 1: 😊  Frame 2: 😊  Frame 3: 😐      │
  │     (face expression randomly changes)            │
  │                                                    │
  │  2. Jittering:                                    │
  │     Frame 1: ● at (10,10)                        │
  │     Frame 2: ● at (11,13)  ← should be (11,10)  │
  │     Frame 3: ● at (12,9)   ← should be (12,10)  │
  │     (position wobbles around correct trajectory)  │
  │                                                    │
  │  3. Identity Drift:                              │
  │     Frame 1: 👩 brown hair                       │
  │     Frame 50: 👩 blonde hair (gradual change!)   │
  │                                                    │
  │  4. Structural Warping:                          │
  │     Buildings bend, objects morph shape           │
  │                                                    │
  │  5. Texture Swimming:                            │
  │     Textures slide across surfaces               │
  │                                                    │
  └──────────────────────────────────────────────────┘
```

---

## Why is Temporal Consistency Hard?

```
  Root cause: frame-independent generation

  If each frame is generated independently:
    Frame t:   denoise(noise_t, text) → frame_t
    Frame t+1: denoise(noise_{t+1}, text) → frame_{t+1}
    
  Even with the same prompt, different noise → different details!

  ┌──────────────────────────────────────────────────┐
  │                                                    │
  │  Same prompt: "A red car on a road"              │
  │                                                    │
  │  Frame 1: 🚗 (3-door hatch, dark red)           │
  │  Frame 2: 🚗 (4-door sedan, bright red)         │
  │  Frame 3: 🚗 (SUV, maroon)                      │
  │                                                    │
  │  Each frame is individually correct,             │
  │  but together they're inconsistent!              │
  │                                                    │
  │  Solutions need to introduce inter-frame         │
  │  dependencies during generation.                 │
  │                                                    │
  └──────────────────────────────────────────────────┘
```

---

## Temporal Attention

```
  Most common approach: add temporal attention layers

  Standard spatial attention (per frame):
    Q, K, V from spatial positions within one frame
    Each pixel attends to other pixels in SAME frame

  Temporal attention (across frames):
    Q, K, V from same spatial position across frames
    Each position attends to same position in OTHER frames

  ┌──────────────────────────────────────────────────┐
  │                                                    │
  │  Spatial Attention:          Temporal Attention:  │
  │                                                    │
  │  Frame t:                    Position (i,j):     │
  │  ┌─────────┐                t=1: ─────┐          │
  │  │ ● ← → ● │                t=2: ──┐  │          │
  │  │ ↕     ↕ │                t=3: ┐  │  │          │
  │  │ ● ← → ● │                     ▼  ▼  ▼          │
  │  └─────────┘               [attend across time]  │
  │  (within frame)            (across frames)        │
  │                                                    │
  │  Both are needed for consistent video!           │
  │                                                    │
  └──────────────────────────────────────────────────┘

  Implementation:
    # Reshape (B, T, C, H, W) for spatial attention:
    #   → (B*T, H*W, C) → self-attention → reshape back
    
    # Reshape for temporal attention:
    #   → (B*H*W, T, C) → self-attention → reshape back
```

---

## Temporal Convolutions

```
  Add 1D convolutions along the time axis:

  After each 2D spatial conv block, add a temporal conv:
  
  Spatial conv:  (B*T, C, H, W)  → Conv2D → (B*T, C', H, W)
  Temporal conv: (B*H*W, C', T)  → Conv1D → (B*H*W, C', T)

  This mixes information across frames at every layer!

  ┌──────────────────────────────────────────────────┐
  │                                                    │
  │  Pseudo-3D approach:                             │
  │                                                    │
  │  Full 3D conv: expensive (K³ parameters)         │
  │  2D + 1D:     cheap (K² + K parameters)         │
  │                                                    │
  │  Decomposition:                                  │
  │  Conv3D(k,k,k) ≈ Conv2D(k,k) + Conv1D(k)      │
  │                                                    │
  │  Benefits:                                       │
  │  • Can initialize from pretrained 2D models      │
  │  • Much fewer parameters                         │
  │  • Often works just as well                      │
  │                                                    │
  └──────────────────────────────────────────────────┘
```

---

## Optical Flow for Consistency

```
  Optical flow: per-pixel motion vectors between frames

  Frame t → Frame t+1:
    Each pixel (x,y) maps to (x+dx, y+dy)

  Using flow for consistency:

  1. Flow-based warping loss:
     Warp frame t using flow → should match frame t+1
     L_warp = ||warp(frame_t, flow_{t→t+1}) - frame_{t+1}||

  2. Flow-guided attention:
     Instead of attending to same position across time,
     attend to flow-displaced positions
     (follow the object, not the grid position)

  ┌──────────────────────────────────────────────────┐
  │                                                    │
  │  Frame t:        Flow:        Frame t+1:         │
  │  ┌─────────┐   ┌─────────┐   ┌─────────┐       │
  │  │    ●    │   │    →    │   │       ●  │       │
  │  │         │   │    →    │   │         │       │
  │  │         │   │         │   │         │       │
  │  └─────────┘   └─────────┘   └─────────┘       │
  │                                                    │
  │  Object at (3,1) in t moves to (5,1) in t+1    │
  │  Flow vector: (dx=2, dy=0)                      │
  │  Warp frame t by flow → should align with t+1   │
  │                                                    │
  └──────────────────────────────────────────────────┘
```

---

## Noise Initialization Strategies

```
  Smart noise initialization for consistency:

  1. Shared noise (naïve):
     Use same noise for all frames
     → Identical frames (no motion)

  2. Independent noise:
     Different noise per frame
     → Inconsistent outputs

  3. Correlated noise (best!):
     noise_t = α * shared_noise + (1-α) * unique_noise_t
     
     Or use noise schedules that correlate across time

  4. Progressive noise:
     Frame 1: noise_1
     Frame 2: noise_1 + δ_1  (small perturbation)
     Frame 3: noise_1 + δ_1 + δ_2
     → Smooth transitions between frames

  ┌──────────────────────────────────────────────────┐
  │  FreeNoise (Qiu et al., 2023):                  │
  │                                                    │
  │  Sliding window with noise rescheduling:         │
  │                                                    │
  │  Window 1: [n₁, n₂, n₃, n₄]                    │
  │  Window 2:     [n₂, n₃, n₄, n₅]                │
  │  Window 3:         [n₃, n₄, n₅, n₆]            │
  │                                                    │
  │  Overlapping windows ensure consistent frames    │
  │  → Enables arbitrarily long video generation!    │
  └──────────────────────────────────────────────────┘
```

---

## Cross-Frame Attention Mechanisms

```
  Different strategies for attending across frames:

  1. Full temporal attention:
     Every frame attends to every other frame
     Cost: O(T² × HW)  — expensive for long videos!

  2. Causal temporal attention:
     Frame t only attends to frames 1..t
     Good for autoregressive generation
     Cost: O(T² × HW / 2)

  3. Sparse temporal attention:
     Attend to: [first frame, previous frame, current frame]
     Cost: O(3 × HW)  — constant regardless of T!

  4. Anchor frame attention:
     Pick key frames as anchors
     All frames attend to anchors + neighbors
     Good for long videos with consistent identity

  ┌──────────────────────────────────────────────────┐
  │                                                    │
  │  Sparse attention pattern (common in practice):  │
  │                                                    │
  │  Frame:  1  2  3  4  5  6  7  8                  │
  │          ↑        ↑  ↑                            │
  │          │        │  │                            │
  │          └────────┘  current                     │
  │         first    prev                             │
  │                                                    │
  │  Frame 5 attends to: frame 1 (anchor),           │
  │  frame 4 (prev), and frame 5 (self)              │
  │                                                    │
  └──────────────────────────────────────────────────┘
```

---

## Measuring Temporal Consistency

```
  ┌─────────────────────┬────────────────────────────────────┐
  │ Metric              │ How it works                       │
  ├─────────────────────┼────────────────────────────────────┤
  │ Warping Error       │ Warp frame t→t+1 using GT flow,  │
  │                     │ measure pixel difference           │
  ├─────────────────────┼────────────────────────────────────┤
  │ CLIP Consistency    │ Cosine similarity of CLIP          │
  │                     │ embeddings between consecutive     │
  │                     │ frames (should be high)            │
  ├─────────────────────┼────────────────────────────────────┤
  │ FVD                 │ Fréchet Video Distance —           │
  │                     │ measures both quality AND          │
  │                     │ temporal coherence                 │
  ├─────────────────────┼────────────────────────────────────┤
  │ Frame Consistency   │ Average LPIPS between all          │
  │ (FC)                │ consecutive frame pairs            │
  ├─────────────────────┼────────────────────────────────────┤
  │ Motion Smoothness   │ Acceleration of optical flow:      │
  │                     │ ||flow_t - 2*flow_{t+1}            │
  │                     │ + flow_{t+2}||                     │
  ├─────────────────────┼────────────────────────────────────┤
  │ Subject Consistency │ DINO feature similarity of the    │
  │                     │ main subject across frames         │
  └─────────────────────┴────────────────────────────────────┘
```

---

## Python: Temporal Attention Implementation

```python
import torch
import torch.nn as nn
import torch.nn.functional as F
from einops import rearrange


class TemporalAttention(nn.Module):
    """Temporal self-attention for video generation."""
    
    def __init__(self, dim, heads=8):
        super().__init__()
        self.heads = heads
        self.dim_head = dim // heads
        self.to_qkv = nn.Linear(dim, dim * 3, bias=False)
        self.to_out = nn.Linear(dim, dim)
        self.norm = nn.LayerNorm(dim)
    
    def forward(self, x):
        """
        x: (B, T, H, W, C) - video features
        """
        B, T, H, W, C = x.shape
        
        # Reshape: treat each spatial position independently
        x_flat = rearrange(x, 'b t h w c -> (b h w) t c')
        x_norm = self.norm(x_flat)
        
        # Compute Q, K, V
        qkv = self.to_qkv(x_norm).chunk(3, dim=-1)
        q, k, v = map(
            lambda t: rearrange(t, 'n t (h d) -> n h t d', h=self.heads),
            qkv
        )
        
        # Scaled dot-product attention across time
        scale = self.dim_head ** -0.5
        attn = torch.matmul(q, k.transpose(-2, -1)) * scale
        attn = F.softmax(attn, dim=-1)
        
        out = torch.matmul(attn, v)
        out = rearrange(out, 'n h t d -> n t (h d)')
        out = self.to_out(out)
        
        # Residual connection
        out = out + x_flat
        return rearrange(out, '(b h w) t c -> b t h w c', b=B, h=H, w=W)


class TemporalConv(nn.Module):
    """1D temporal convolution for frame-to-frame consistency."""
    
    def __init__(self, channels, kernel_size=3):
        super().__init__()
        self.conv = nn.Conv1d(
            channels, channels,
            kernel_size,
            padding=kernel_size // 2,
        )
        self.norm = nn.GroupNorm(8, channels)
    
    def forward(self, x):
        """x: (B, T, C, H, W)"""
        B, T, C, H, W = x.shape
        
        # Reshape: temporal conv per spatial position
        x_flat = rearrange(x, 'b t c h w -> (b h w) c t')
        out = self.conv(x_flat)
        out = self.norm(rearrange(out, '(b h w) c t -> (b t) c (h w)',
                                  b=B, h=H, w=W).reshape(B*T, C, H*W)
                       ).reshape(B, T, C, H, W)
        
        return out + x  # residual


class VideoBlock(nn.Module):
    """Combined spatial + temporal block for video generation."""
    
    def __init__(self, channels, heads=8):
        super().__init__()
        # Spatial processing (per frame)
        self.spatial_conv = nn.Sequential(
            nn.GroupNorm(8, channels),
            nn.SiLU(),
            nn.Conv2d(channels, channels, 3, padding=1),
        )
        # Temporal processing (across frames)
        self.temporal_attn = TemporalAttention(channels, heads)
        self.temporal_conv = TemporalConv(channels)
    
    def forward(self, x):
        """x: (B, T, C, H, W)"""
        B, T, C, H, W = x.shape
        
        # Spatial conv (per frame)
        x_flat = rearrange(x, 'b t c h w -> (b t) c h w')
        x_flat = self.spatial_conv(x_flat) + x_flat
        x = rearrange(x_flat, '(b t) c h w -> b t c h w', b=B)
        
        # Temporal conv
        x = self.temporal_conv(x)
        
        # Temporal attention
        x_perm = rearrange(x, 'b t c h w -> b t h w c')
        x_perm = self.temporal_attn(x_perm)
        x = rearrange(x_perm, 'b t h w c -> b t c h w')
        
        return x


# Measure temporal consistency
def compute_temporal_consistency(frames, feature_extractor):
    """
    Compute average CLIP cosine similarity between consecutive frames.
    frames: list of PIL Images
    feature_extractor: CLIP model
    """
    similarities = []
    for i in range(len(frames) - 1):
        feat_1 = feature_extractor(frames[i])
        feat_2 = feature_extractor(frames[i + 1])
        sim = F.cosine_similarity(feat_1, feat_2, dim=-1)
        similarities.append(sim.item())
    
    return sum(similarities) / len(similarities)
```

---

## Revision Questions

1. **What are the five types of temporal inconsistency in generated video?**
2. **How does temporal attention differ from spatial attention?**
3. **Why does shared noise across frames cause problems?**
4. **How does optical flow help enforce temporal consistency?**
5. **What is sparse temporal attention and why is it used for long videos?**
6. **Name three metrics used to measure temporal consistency.**

---

[Previous: 02-video-synthesis.md](02-video-synthesis.md) | [Next: 04-current-state-of-video-generation.md](04-current-state-of-video-generation.md)
