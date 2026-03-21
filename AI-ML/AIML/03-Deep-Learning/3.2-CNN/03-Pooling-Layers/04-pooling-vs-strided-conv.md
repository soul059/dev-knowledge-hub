# 4 · Pooling vs Strided Convolution

> **Unit 3.2 – Convolutional Neural Networks / 03 – Pooling Layers**
>
> [← Previous: Global Pooling](./03-global-pooling.md) | [Next: Spatial Dimension Reduction →](./05-spatial-dimension-reduction.md)
>
> [↑ Back to Pooling Layers Overview](./README.md)

---

## 4.1 Chapter Overview

Both pooling and strided convolutions achieve the same goal: **reducing spatial dimensions**. But they differ fundamentally in how they do it:

| Approach | Mechanism | Parameters |
|----------|-----------|------------|
| **Pooling** | Fixed aggregation (max, avg) | 0 |
| **Strided Convolution** | Learned filters applied with stride > 1 | Learnable |

The deep learning community has seen a clear **trend away from pooling and toward strided convolutions** in modern architectures. This chapter examines why, when each approach is appropriate, and the trade-offs involved.

---

## 4.2 How Each Method Downsamples

### Pooling — Fixed Aggregation

```
  Max Pool 2×2, stride 2:

  ┌────┬────┬────┬────┐         ┌────┬────┐
  │ 1  │ 3  │ 5  │ 2  │         │ 7  │ 5  │
  ├────┼────┼────┼────┤   ──►   ├────┼────┤
  │ 7  │ 2  │ 1  │ 4  │         │ 9  │ 8  │
  ├────┼────┼────┼────┤         └────┴────┘
  │ 6  │ 9  │ 3  │ 8  │
  ├────┼────┼────┼────┤    Aggregation function: max()
  │ 4  │ 1  │ 5  │ 2  │    Parameters: 0
  └────┴────┴────┴────┘    Learned: NO
```

### Strided Convolution — Learned Downsampling

```
  Conv2d 3×3, stride 2, padding 1:

  ┌────┬────┬────┬────┐         ┌────┬────┐
  │ 1  │ 3  │ 5  │ 2  │         │ a  │ b  │
  ├────┼────┼────┼────┤   ──►   ├────┼────┤
  │ 7  │ 2  │ 1  │ 4  │         │ c  │ d  │
  ├────┼────┼────┼────┤         └────┴────┘
  │ 6  │ 9  │ 3  │ 8  │
  ├────┼────┼────┼────┤    Each output = weighted sum using 3×3 kernel
  │ 4  │ 1  │ 5  │ 2  │    Parameters: 3×3×C_in×C_out + C_out
  └────┴────┴────┴────┘    Learned: YES
```

### Side-by-Side Comparison

```
  INPUT (4×4)                     POOLING                STRIDED CONV
                               (fixed function)        (learned weights)
  ┌────┬────┬────┬────┐       ┌────────────────┐     ┌────────────────────┐
  │    │    │    │    │       │  Select max or  │     │  Apply learned 3×3 │
  │    │    │    │    │  ──►  │  compute avg    │     │  filter at stride 2│
  │    │    │    │    │       │  No learning    │     │  Weights optimized │
  │    │    │    │    │       │                 │     │  via backprop      │
  └────┴────┴────┴────┘       └────────┬───────┘     └─────────┬──────────┘
                                       │                       │
                                       ▼                       ▼
                                  ┌────┬────┐            ┌────┬────┐
                                  │    │    │            │    │    │
                                  │    │    │            │    │    │
                                  └────┴────┘            └────┴────┘
                                   (2×2)                   (2×2)
                               Same output size!        Same output size!
```

---

## 4.3 Mathematical Formulations

### Pooling (No Parameters)

**Max Pooling:**
```
  Y[c, i, j] = max          X[c, i·s + m, j·s + n]
              0≤m<k, 0≤n<k

  Parameters: 0
  FLOPs per output element: k² comparisons
```

**Average Pooling:**
```
                  1     k-1  k-1
  Y[c, i, j] = ─────   Σ    Σ   X[c, i·s + m, j·s + n]
                k × k  m=0  n=0

  Parameters: 0
  FLOPs per output element: k² additions + 1 division
```

### Strided Convolution (Learned Parameters)

```
                C_in-1  k-1  k-1
  Y[c_o, i, j] =  Σ     Σ    Σ   W[c_o, c_i, m, n] · X[c_i, i·s + m, j·s + n]  +  b[c_o]
                c_i=0   m=0  n=0

  Parameters: k × k × C_in × C_out + C_out
  FLOPs per output element: k² × C_in multiply-adds
```

### Key Difference

```
  POOLING:                          STRIDED CONV:
  ┌─────────────────────────┐      ┌─────────────────────────┐
  │  f(x) = aggregate(x)   │      │  f(x) = Wx + b          │
  │                         │      │                         │
  │  • fixed function       │      │  • W is learned          │
  │  • operates per-channel │      │  • mixes channels       │
  │  • 0 parameters         │      │  • k²·C_in·C_out params │
  └─────────────────────────┘      └─────────────────────────┘
```

---

## 4.4 Strided Convolution in Detail

### How Stride > 1 Downsamples

A convolution with stride `s` skips `s-1` positions between applications:

```
  Stride 1 (standard):              Stride 2 (downsampling):
  ┌───┬───┬───┬───┬───┐            ┌───┬───┬───┬───┬───┐
  │ * │ * │ * │   │   │            │ * │ * │ * │   │   │
  ├───┼───┼───┼───┼───┤            ├───┼───┼───┼───┼───┤
  │ * │ * │ * │   │   │            │ * │ * │ * │   │   │
  ├───┼───┼───┼───┼───┤            ├───┼───┼───┼───┼───┤
  │ * │ * │ * │   │   │            │ * │ * │ * │   │   │   Position (0,0)
  ├───┼───┼───┼───┼───┤            ├───┼───┼───┼───┼───┤   then jump to
  │   │   │   │   │   │            │   │   │   │   │   │   position (0,2)
  ├───┼───┼───┼───┼───┤            ├───┼───┼───┼───┼───┤   (skip 1 col)
  │   │   │   │   │   │            │   │   │   │   │   │
  └───┴───┴───┴───┴───┘            └───┴───┴───┴───┴───┘
  Output: 3×3                       Output: 2×2

  With stride 2, the filter is applied at every other position,
  naturally halving the spatial dimensions.
```

### Output Size Formula (Same as Pooling)

```
            ⌊ H_in + 2p - k ⌋
  H_out  =  ⎢ ─────────────── ⎥ + 1
            ⌊       s         ⌋
```

For `stride=2, kernel=3, padding=1` (common in ResNet):
```
  H_out = ⌊(H_in + 2 - 3) / 2⌋ + 1 = ⌊(H_in - 1) / 2⌋ + 1

  Example: H_in = 32 → H_out = ⌊31/2⌋ + 1 = 15 + 1 = 16 ✓ (halved)
```

---

## 4.5 Detailed Comparison

### Feature-by-Feature

| Feature | Max Pool | Avg Pool | Strided Conv |
|---------|----------|----------|-------------|
| **Parameters** | 0 | 0 | k²·C_in·C_out |
| **Learnable** | No | No | Yes |
| **Channel mixing** | No | No | Yes |
| **Can change channels** | No | No | Yes (C_in → C_out) |
| **Translation equivariance** | Approximate | Approximate | Approximate |
| **Information preservation** | Lossy (max only) | Lossy (averaged) | Less lossy (learned) |
| **Gradient flow** | Sparse | Uniform | Standard backprop |
| **Computation** | Very cheap | Very cheap | More expensive |
| **Memory** | Low | Low | Weights + gradients |
| **Aliasing** | Can alias | Less aliasing | Can alias |

### Computational Cost Comparison

```
  Example: Input (64, 32, 32), Output (128, 16, 16)

  MAX POOL (after Conv to change channels):
  ──────────────────────────────────────────
  Conv 3×3: 64→128, stride=1, pad=1    → 3×3×64×128 = 73,728 params
  MaxPool 2×2, stride=2                → 0 params
  Total: 73,728 params

  STRIDED CONV (combined):
  ──────────────────────────────────────────
  Conv 3×3: 64→128, stride=2, pad=1    → 3×3×64×128 = 73,728 params
  Total: 73,728 params

  Same parameter count! But strided conv is ONE operation instead of TWO.
  ⟹ Strided conv can be more computationally efficient (better GPU utilization).
```

---

## 4.6 The Modern Trend: Away from Pooling

### Historical Timeline

```
  2012 ─── AlexNet ───── Max pooling (overlapping 3×3, stride 2)
    │
  2014 ─── VGGNet ────── Max pooling 2×2, stride 2 (5 pool layers)
    │
  2014 ─── GoogLeNet ─── Mixed: max pool + strided conv in inception modules
    │
  2014 ─── NiN ────────── Global Average Pooling (replaces FC layers)
    │
  2015 ─── ResNet ────── Mix: initial max pool, then strided conv for downsample
    │
  2016 ─── "Strided      Springenberg et al.: "Striving for Simplicity"
    │      Conv" paper ── Replace ALL pooling with strided conv → same accuracy
    │
  2017 ─── DenseNet ──── Strided conv in transition layers (+ avg pool option)
    │
  2019 ─── EfficientNet ─ Primarily strided conv, GAP at the end only
    │
  2020 ─── ConvNeXt ──── Pure strided conv downsampling (no pooling at all)
    │
  2021 ─── Vision        Patch embedding = strided conv (no pooling)
    │      Transformer
    │
  2023+── Modern CNNs ── Strided conv is the default; pooling is rare
```

### The "Striving for Simplicity" Paper

Springenberg et al. (2015) conducted a systematic study replacing pooling with strided convolutions:

```
  Experiment:
  ┌────────────────────────────┬────────────────────────────┐
  │  ORIGINAL (with pooling)  │  ALL-CONV (no pooling)     │
  ├────────────────────────────┼────────────────────────────┤
  │  Conv 5×5, stride 1       │  Conv 5×5, stride 1        │
  │  MaxPool 3×3, stride 2    │  Conv 3×3, stride 2  ← replaces pool
  │  Conv 5×5, stride 1       │  Conv 5×5, stride 1        │
  │  MaxPool 3×3, stride 2    │  Conv 3×3, stride 2  ← replaces pool
  │  Conv 3×3                 │  Conv 3×3                  │
  │  FC → softmax             │  GAP → softmax             │
  ├────────────────────────────┼────────────────────────────┤
  │  CIFAR-10: 92.5%          │  CIFAR-10: 92.7%  ← better │
  │  Parameters: fewer        │  Parameters: more           │
  └────────────────────────────┴────────────────────────────┘

  Key finding: Strided convolutions can fully replace pooling
  with no loss (and sometimes a gain) in accuracy.
```

---

## 4.7 When to Use Each

### Decision Framework

```
  START
    │
    ├── Need to CHANGE number of channels?
    │   │
    │   YES ──► Strided Convolution (pooling can't change channels)
    │   │
    │   NO
    │   │
    ├── Is this the FIRST downsampling after input?
    │   │
    │   YES ──► Often MaxPool (traditional) or Strided Conv (modern)
    │   │        ResNet uses MaxPool here; ConvNeXt uses Strided Conv
    │   │
    │   NO
    │   │
    ├── Is this the FINAL layer before classifier?
    │   │
    │   YES ──► Global Average Pooling (always)
    │   │
    │   NO
    │   │
    ├── Is model size / parameter count critical?
    │   │
    │   YES ──► Pooling (0 parameters)
    │   │
    │   NO
    │   │
    ├── Need MAXIMUM control over downsampling?
    │   │
    │   YES ──► Strided Convolution (learned, optimal downsampling)
    │   │
    │   NO
    │   │
    └── Default ──► Strided Convolution (modern default)
                    or MaxPool (if following a traditional architecture)
```

### Specific Recommendations

```
  USE POOLING when:
  ┌────────────────────────────────────────────────────────────────┐
  │  • Reproducing classic architectures (VGG, AlexNet)           │
  │  • Parameter budget is extremely tight (embedded / mobile)     │
  │  • You need built-in translation invariance (max pool)         │
  │  • Global aggregation before classifier (GAP — always use)    │
  │  • Quick prototyping without tuning extra conv layers          │
  └────────────────────────────────────────────────────────────────┘

  USE STRIDED CONVOLUTION when:
  ┌────────────────────────────────────────────────────────────────┐
  │  • Designing new architectures (modern best practice)          │
  │  • You need to change channels AND downsample simultaneously   │
  │  • Maximum accuracy is the priority                            │
  │  • Using generative models (GANs — avoids checkerboard)       │
  │  • Dense prediction (segmentation, detection)                  │
  │  • Following ResNet/ConvNeXt/EfficientNet patterns            │
  └────────────────────────────────────────────────────────────────┘
```

---

## 4.8 Worked Example — Replacing Pool with Strided Conv

### VGG-Style Block (Original)

```
  Input: (64, 32, 32)

  Conv2d(64, 128, 3, padding=1)   → (128, 32, 32)   params: 3×3×64×128+128 = 73,856
  ReLU                            → (128, 32, 32)
  Conv2d(128, 128, 3, padding=1)  → (128, 32, 32)   params: 3×3×128×128+128 = 147,584
  ReLU                            → (128, 32, 32)
  MaxPool2d(2, 2)                 → (128, 16, 16)   params: 0
                                                     ──────────
                                     Total params:   221,440
```

### All-Conv Replacement

```
  Input: (64, 32, 32)

  Conv2d(64, 128, 3, padding=1)   → (128, 32, 32)   params: 73,856
  ReLU                            → (128, 32, 32)
  Conv2d(128, 128, 3, stride=2,   → (128, 16, 16)   params: 147,584  ← REPLACES MaxPool
         padding=1)
  ReLU                            → (128, 16, 16)
                                                     ──────────
                                     Total params:   221,440  (same!)
```

The key insight: by changing the last conv's stride from 1 to 2, we combine feature extraction and downsampling into a single operation, eliminating the pooling layer entirely.

---

## 4.9 Python / PyTorch Implementation

### Pooling-Based Downsampling

```python
import torch
import torch.nn as nn

class PoolDownBlock(nn.Module):
    """Traditional: Conv → BN → ReLU → MaxPool"""
    def __init__(self, in_ch, out_ch):
        super().__init__()
        self.block = nn.Sequential(
            nn.Conv2d(in_ch, out_ch, 3, stride=1, padding=1),
            nn.BatchNorm2d(out_ch),
            nn.ReLU(inplace=True),
            nn.MaxPool2d(2, 2),  # Separate downsampling
        )

    def forward(self, x):
        return self.block(x)
```

### Strided Conv Downsampling

```python
class StridedConvDownBlock(nn.Module):
    """Modern: Conv(stride=2) → BN → ReLU"""
    def __init__(self, in_ch, out_ch):
        super().__init__()
        self.block = nn.Sequential(
            nn.Conv2d(in_ch, out_ch, 3, stride=2, padding=1),  # Combined!
            nn.BatchNorm2d(out_ch),
            nn.ReLU(inplace=True),
        )

    def forward(self, x):
        return self.block(x)
```

### Direct Comparison

```python
# Compare both approaches
pool_block = PoolDownBlock(64, 128)
strided_block = StridedConvDownBlock(64, 128)

x = torch.randn(1, 64, 32, 32)

y_pool = pool_block(x)
y_strided = strided_block(x)

print(f"Input:          {x.shape}")         # [1, 64, 32, 32]
print(f"Pool output:    {y_pool.shape}")    # [1, 128, 16, 16]
print(f"Strided output: {y_strided.shape}") # [1, 128, 16, 16]

# Parameter counts
pool_params = sum(p.numel() for p in pool_block.parameters())
strided_params = sum(p.numel() for p in strided_block.parameters())
print(f"\nPool block params:    {pool_params:,}")     # 74,112
print(f"Strided block params: {strided_params:,}")    # 74,112 (same!)
```

### ResNet-Style Downsample Block

```python
class ResNetDownsample(nn.Module):
    """ResNet uses strided conv in the first conv of each stage transition.
    The shortcut also needs a strided 1×1 conv to match dimensions."""
    def __init__(self, in_ch, out_ch, stride=2):
        super().__init__()
        self.conv1 = nn.Conv2d(in_ch, out_ch, 3, stride=stride, padding=1, bias=False)
        self.bn1 = nn.BatchNorm2d(out_ch)
        self.conv2 = nn.Conv2d(out_ch, out_ch, 3, stride=1, padding=1, bias=False)
        self.bn2 = nn.BatchNorm2d(out_ch)
        self.relu = nn.ReLU(inplace=True)

        # Shortcut: 1×1 strided conv to match spatial + channel dims
        self.shortcut = nn.Sequential(
            nn.Conv2d(in_ch, out_ch, 1, stride=stride, bias=False),
            nn.BatchNorm2d(out_ch),
        )

    def forward(self, x):
        identity = self.shortcut(x)
        out = self.relu(self.bn1(self.conv1(x)))
        out = self.bn2(self.conv2(out))
        out += identity
        out = self.relu(out)
        return out

block = ResNetDownsample(64, 128, stride=2)
x = torch.randn(1, 64, 32, 32)
y = block(x)
print(f"ResNet downsample: {x.shape} → {y.shape}")
# [1, 64, 32, 32] → [1, 128, 16, 16]
```

### ConvNeXt-Style Downsample (Pure Strided Conv)

```python
class ConvNeXtDownsample(nn.Module):
    """ConvNeXt uses LayerNorm + 2×2 strided conv for downsampling.
    No pooling at all — purely learned downsampling."""
    def __init__(self, in_ch, out_ch):
        super().__init__()
        self.norm = nn.LayerNorm(in_ch)
        self.down = nn.Conv2d(in_ch, out_ch, kernel_size=2, stride=2)

    def forward(self, x):
        # LayerNorm expects (B, H, W, C) — permute
        x = x.permute(0, 2, 3, 1)    # [B, H, W, C]
        x = self.norm(x)
        x = x.permute(0, 3, 1, 2)    # [B, C, H, W]
        x = self.down(x)
        return x

down = ConvNeXtDownsample(96, 192)
x = torch.randn(1, 96, 56, 56)
y = down(x)
print(f"ConvNeXt downsample: {x.shape} → {y.shape}")
# [1, 96, 56, 56] → [1, 192, 28, 28]
```

### Benchmark: Speed Comparison

```python
import time

def benchmark(block, x, name, n_runs=1000):
    # Warmup
    for _ in range(100):
        _ = block(x)
    # Timed runs
    start = time.perf_counter()
    for _ in range(n_runs):
        _ = block(x)
    elapsed = (time.perf_counter() - start) / n_runs * 1000
    print(f"{name}: {elapsed:.3f} ms/forward")

x = torch.randn(8, 64, 32, 32)
pool_block = PoolDownBlock(64, 128).eval()
strided_block = StridedConvDownBlock(64, 128).eval()

with torch.no_grad():
    benchmark(pool_block, x, "Pool + Conv")
    benchmark(strided_block, x, "Strided Conv")
# Strided conv is often faster (one fused op vs two separate ops)
```

---

## 4.10 Aliasing and Anti-Aliasing

### The Aliasing Problem

Both pooling and strided convolutions can suffer from **aliasing** — when high-frequency spatial features are undersampled:

```
  HIGH-FREQUENCY PATTERN:       AFTER STRIDE-2 SUBSAMPLE:
  ┌───┬───┬───┬───┬───┬───┐    ┌───┬───┬───┐
  │ 1 │ 0 │ 1 │ 0 │ 1 │ 0 │    │ 1 │ 1 │ 1 │   ← Aliased! Looks constant
  ├───┼───┼───┼───┼───┼───┤    ├───┼───┼───┤      instead of alternating
  │ 0 │ 1 │ 0 │ 1 │ 0 │ 1 │    │ 0 │ 0 │ 0 │
  ├───┼───┼───┼───┼───┼───┤    ├───┼───┼───┤
  │ 1 │ 0 │ 1 │ 0 │ 1 │ 0 │    │ 1 │ 1 │ 1 │
  └───┴───┴───┴───┴───┴───┘    └───┴───┴───┘
```

### Solution: Blur Before Subsample

Zhang (2019) proposed adding a low-pass filter (blur) before strided downsampling:

```
  Anti-aliased pipeline:
  Input → Conv(stride=1) → BlurPool(stride=2) → Output

  BlurPool = Average/Gaussian blur + Stride-2 subsample
```

```python
class BlurPool(nn.Module):
    """Anti-aliased downsampling: blur then subsample."""
    def __init__(self, channels, stride=2):
        super().__init__()
        # Simple box blur kernel
        kernel = torch.ones(1, 1, 3, 3) / 9.0
        kernel = kernel.repeat(channels, 1, 1, 1)
        self.register_buffer('kernel', kernel)
        self.stride = stride
        self.channels = channels

    def forward(self, x):
        # Depthwise blur
        x = torch.nn.functional.conv2d(x, self.kernel, stride=self.stride,
                                        padding=1, groups=self.channels)
        return x

blur = BlurPool(64, stride=2)
x = torch.randn(1, 64, 32, 32)
y = blur(x)
print(f"BlurPool: {x.shape} → {y.shape}")  # [1, 64, 32, 32] → [1, 64, 16, 16]
```

---

## 4.11 Applications

### Architecture Survey

| Architecture | Downsampling Method | Notes |
|-------------|-------------------|-------|
| **AlexNet (2012)** | Max Pool 3×3, stride 2 | Overlapping pooling |
| **VGG (2014)** | Max Pool 2×2, stride 2 | Between conv blocks |
| **GoogLeNet (2014)** | Max Pool + Strided Conv | Mixed in inception |
| **ResNet (2015)** | Max Pool (initial) + Strided Conv (stages) | Hybrid approach |
| **DenseNet (2017)** | Avg Pool in transition layers | Between dense blocks |
| **MobileNetV2 (2018)** | Strided Depthwise Conv | Efficient mobile design |
| **EfficientNet (2019)** | Strided Depthwise Conv | Compound scaling |
| **ConvNeXt (2022)** | Strided Conv 2×2 | Purely learned |
| **ViT (2021)** | Strided Conv 16×16 (patch embed) | Aggressive learned |

---

## 4.12 Summary Table

| Aspect | Pooling | Strided Convolution |
|--------|---------|-------------------|
| **Type** | Fixed aggregation | Learned transformation |
| **Parameters** | 0 | k²·C_in·C_out + C_out |
| **Channel change** | Not possible | Yes |
| **Computation** | Cheaper per operation | More FLOPs |
| **GPU efficiency** | Two ops (conv + pool) | One fused op |
| **Gradient flow** | Sparse (max) / uniform (avg) | Standard, dense |
| **Flexibility** | None — fixed behavior | Full — learned optimal downsampling |
| **Modern trend** | Declining (except GAP) | Increasing (default choice) |
| **Best for** | Quick prototyping, GAP | Production architectures |

---

## 4.13 Revision Questions

**Q1.** A strided convolution with `kernel_size=3, stride=2, padding=1` is applied to a feature map of shape `(1, 64, 32, 32)` with 128 output channels. What is the output shape and parameter count?

<details><summary>Answer</summary>

Output shape: `(1, 128, 16, 16)` — spatial dims halved, channels changed from 64 to 128.

Parameters: 3 × 3 × 64 × 128 + 128 = 73,728 + 128 = 73,856.

A MaxPool2d(2,2) followed by Conv2d(64,128,3,padding=1) would give the same output shape but requires the conv to operate on already-downsampled features.

</details>

**Q2.** Why did the paper "Striving for Simplicity" (Springenberg et al., 2015) advocate replacing pooling with strided convolutions?

<details><summary>Answer</summary>

They showed experimentally that an all-convolutional network (replacing all pooling layers with strided convolutions) matched or exceeded the accuracy of pooling-based architectures on CIFAR-10 and ImageNet. The argument is that learned downsampling can adapt to the data distribution, while fixed pooling operations (max, avg) impose arbitrary inductive biases. The simplicity of having only one type of operation (convolutions) also makes architecture design and analysis easier.

</details>

**Q3.** In what situation would pooling be strictly preferred over strided convolution?

<details><summary>Answer</summary>

When the parameter budget is extremely constrained (e.g., tiny embedded devices) and you cannot afford additional convolution weights. Also, Global Average Pooling before the classifier is almost universally preferred over alternatives — it's the one pooling operation that has not been replaced by strided convolutions in modern architectures.

</details>

**Q4.** Explain why strided convolutions in GANs are preferred over pooling for the discriminator.

<details><summary>Answer</summary>

In GANs, the discriminator must learn to distinguish real from generated images. Max pooling's fixed aggregation can create artifacts and doesn't provide learnable gradients that help the generator improve. Strided convolutions provide smooth, learned downsampling with dense gradient flow, which stabilizes GAN training. The DCGAN paper (Radford et al., 2016) established this as a best practice: strided convolutions in the discriminator, strided transposed convolutions in the generator.

</details>

**Q5.** How does ConvNeXt handle downsampling differently from ResNet, and why?

<details><summary>Answer</summary>

ResNet uses MaxPool2d(3,2,1) after the first conv layer, then strided 3×3 convolutions at stage transitions. ConvNeXt eliminates all pooling and uses a separate "downsample layer" consisting of LayerNorm followed by a 2×2 strided convolution (stride=2). This is inspired by the Swin Transformer's patch merging and provides cleaner, purely learned spatial reduction. It also enables the use of LayerNorm (instead of BatchNorm) for normalization, aligning CNNs with Transformer best practices.

</details>

**Q6.** Compare the gradient flow differences between max pooling and strided convolution during backpropagation.

<details><summary>Answer</summary>

Max pooling routes the entire gradient to only the maximum element in each window (sparse gradients — 75% zeros in 2×2). Strided convolution distributes gradients to all input positions covered by the kernel, weighted by the learned filter values (dense gradients). Dense gradients mean that all input features receive learning signals, which can lead to more efficient training. However, the sparsity of max pooling can act as an implicit regularizer, similar to dropout.

</details>

---

> [← Previous: Global Pooling](./03-global-pooling.md) | [Next: Spatial Dimension Reduction →](./05-spatial-dimension-reduction.md)
>
> [↑ Back to Pooling Layers Overview](./README.md)
