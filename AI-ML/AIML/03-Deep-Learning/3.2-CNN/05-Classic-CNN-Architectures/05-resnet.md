# 🔗 ResNet: Residual Learning and the Depth Breakthrough

[← Previous: GoogLeNet/Inception](04-googlenet-inception.md) | [Back to Unit Overview](../README.md) | [Next: DenseNet →](06-densenet.md)

---

## 📋 Table of Contents

- [Overview](#overview)
- [The Degradation Problem](#the-degradation-problem)
- [Residual Learning Framework](#residual-learning-framework)
- [Architecture Deep Dive](#architecture-deep-dive)
- [ASCII Architecture Diagrams](#ascii-architecture-diagrams)
- [Basic Block vs. Bottleneck Block](#basic-block-vs-bottleneck-block)
- [Identity vs. Projection Shortcuts](#identity-vs-projection-shortcuts)
- [ResNet Variants](#resnet-variants)
- [Parameter Calculations](#parameter-calculations)
- [Mathematical Foundations](#mathematical-foundations)
- [Worked Example](#worked-example)
- [PyTorch Implementation](#pytorch-implementation)
- [Applications](#applications)
- [Summary Table](#summary-table)
- [Revision Questions](#revision-questions)

---

## Overview

**ResNet** (Residual Network) was introduced by **Kaiming He, Xiangyu Zhang, Shaoqing Ren, and Jian Sun** from Microsoft Research in their 2015 paper *"Deep Residual Learning for Image Recognition."* ResNet won the **ILSVRC-2015** challenge and is considered one of the most influential architectures in deep learning history.

### Key Facts

| Property | Value |
|----------|-------|
| **Year** | 2015 |
| **Authors** | He, Zhang, Ren, Sun (Microsoft Research) |
| **Paper** | Deep Residual Learning for Image Recognition |
| **Competition** | ILSVRC-2015 winner (3.57% top-5 error) |
| **Input Size** | 224 × 224 × 3 |
| **Variants** | ResNet-18, 34, 50, 101, 152 |
| **Parameters (ResNet-50)** | ~25.6 million |
| **Depth** | Up to 152 layers (even 1000+ in experiments) |
| **Key Innovation** | Skip connections / Residual learning |
| **Top-5 Error** | 3.57% (surpassed human performance ~5.1%) |

### The Significance

```
ILSVRC Top-5 Error Rate:
════════════════════════

  30% │
      │ ██████████████████████████████  28.2% (2010)
  25% │ ████████████████████████████    25.8% (2011)
      │
  20% │
      │
  15% │ ███████████████████             15.3% (2012, AlexNet)
      │
  10% │
      │ █████████████                    7.3% (2014, VGG)
      │ ████████████                     6.7% (2014, GoogLeNet)
   5% │ ──── human performance ─────    ~5.1% (human)
      │ ███████                          3.57% (2015, ResNet) ← SUPER-HUMAN!
      │
   0% └──────────────────────────────────────────────

  ResNet was the first architecture to surpass human-level
  performance on ImageNet classification!
```

---

## The Degradation Problem

### The Unexpected Discovery

Before ResNet, the community assumed: "Deeper networks should always perform at least as well as shallower ones." The reasoning was:

> A 56-layer network could simply copy the first 20 layers of a good 20-layer network and set the remaining 36 layers to identity mappings. So it should be at least as good.

But experiments showed the **opposite**:

```
Training Error vs. Depth (on CIFAR-10):
═══════════════════════════════════════

  Error
   │
 8%│    ╲
   │     ╲
 7%│      ╲
   │       ╲      56-layer
   │        ╲    ╱
 6%│         ╲  ╱
   │          ╲╱     ← 56-layer has HIGHER error!
   │          ╱╲
 5%│         ╱  ╲
   │        ╱    20-layer
 4%│       ╱
   │      ╱
 3%│     ╱
   │
   └────────────────────► Epochs

  This is NOT overfitting (training error is worse too)!
  The problem is that deeper networks are harder to optimize.
```

### Why Identity Mapping is Hard to Learn

```
For a plain network, if the optimal function is identity (y = x),
the layer must learn:

  y = σ(Wx + b) = x

  This requires W = I (identity matrix) and b = 0,
  and σ(x) = x (no nonlinearity).

  With ReLU, random initialization, and gradient descent,
  learning this identity mapping is surprisingly difficult!

  The deeper the network, the more "identity layers" needed,
  and the harder optimization becomes.
```

### Degradation ≠ Overfitting

```
Key distinction:
  Overfitting:   Training error ↓, Test error ↑ (gap increases)
  Degradation:   Training error ↑, Test error ↑ (both worse!)

  Degradation is an OPTIMIZATION problem, not a capacity problem.
  The deeper network has enough capacity but can't find the solution.
```

---

## Residual Learning Framework

### The Core Idea

Instead of learning the desired mapping H(x) directly, learn the **residual** F(x) = H(x) - x:

```
Plain Network:                    Residual Network:
══════════════                    ═════════════════

  x                                x ─────────────────┐
  │                                │                   │
  ▼                                ▼                   │
┌──────────┐                    ┌──────────┐           │
│  Layer   │                    │  Layer   │           │
│  (conv)  │                    │  (conv)  │           │
└────┬─────┘                    └────┬─────┘           │
     │                               │                 │ (skip/shortcut
     ▼                               ▼                 │  connection)
┌──────────┐                    ┌──────────┐           │
│  Layer   │                    │  Layer   │           │
│  (conv)  │                    │  (conv)  │           │
└────┬─────┘                    └────┬─────┘           │
     │                               │                 │
     ▼                               ▼                 │
   H(x)                          F(x) + x ◄────────────┘
     │                               │
     ▼                               ▼
   ReLU                            ReLU

  Learns H(x) directly           Learns F(x) = H(x) - x
  If optimal is identity:        If optimal is identity:
    Must learn H(x) = x            Must learn F(x) = 0
    (hard!)                         (easy — just drive weights to 0)
```

### The Residual Block Equation

```
y = F(x, {W_i}) + x

Where:
  x  = input to the residual block
  F  = residual function (the layers in the block)
  y  = output of the block (before final ReLU)
  W_i = weights of the layers in F

Expanded for a 2-layer block:
  F(x) = W_2 · σ(W_1 · x + b_1) + b_2

  y = W_2 · σ(W_1 · x + b_1) + b_2 + x
                                      ↑
                                  skip connection

Then: y_activated = ReLU(y)
```

### Why Residual Learning Works

```
1. IDENTITY IS EASY TO LEARN
   If the optimal function is identity, the residual F(x) = 0.
   Driving weights to 0 is easy with weight decay and SGD.

2. GRADIENT FLOW
   Backpropagation through a residual block:
   
   ∂L/∂x = ∂L/∂y · ∂y/∂x = ∂L/∂y · (∂F/∂x + 1)
                                              ↑
                              The +1 ensures gradient ALWAYS flows!
                              Even if ∂F/∂x → 0, the gradient is ∂L/∂y.

   In a plain network:
   ∂L/∂x = ∂L/∂y · ∂H/∂x
   
   If ∂H/∂x → 0 (vanishing), gradient dies.
   
   In ResNet: gradient has a "highway" through skip connections!

3. ENSEMBLE INTERPRETATION
   A ResNet can be viewed as an ensemble of many paths of
   different lengths. With N residual blocks, there are 2^N paths:

   Block 1: x₁ = F₁(x₀) + x₀  (use F₁ or skip)
   Block 2: x₂ = F₂(x₁) + x₁  (use F₂ or skip)
   ...
   
   2^N possible combinations = exponential ensemble!
```

---

## Architecture Deep Dive

### Overall Structure

All ResNet variants follow the same high-level structure:

```
ResNet High-Level Architecture:
═══════════════════════════════

  Input: 224×224×3
       │
  ═══ STEM ══════════════════
  │ Conv 7×7, 64, stride 2   → 112×112×64
  │ BatchNorm + ReLU
  │ MaxPool 3×3, stride 2    → 56×56×64
  ═══════════════════════════
       │
  ═══ STAGE 1 (conv2_x) ════
  │ N₁ residual blocks        56×56×64 (or 256 for bottleneck)
  ═══════════════════════════
       │ (stride 2 downsample)
  ═══ STAGE 2 (conv3_x) ════
  │ N₂ residual blocks        28×28×128 (or 512)
  ═══════════════════════════
       │ (stride 2 downsample)
  ═══ STAGE 3 (conv4_x) ════
  │ N₃ residual blocks        14×14×256 (or 1024)
  ═══════════════════════════
       │ (stride 2 downsample)
  ═══ STAGE 4 (conv5_x) ════
  │ N₄ residual blocks        7×7×512 (or 2048)
  ═══════════════════════════
       │
  Global Average Pooling      1×1×512 (or 2048)
       │
  FC → num_classes
       │
  Softmax
```

---

## ASCII Architecture Diagrams

### ResNet-34 Architecture

```
                         ResNet-34 Architecture
                         ══════════════════════

  INPUT     STEM          STAGE 1        STAGE 2        STAGE 3        STAGE 4
 224×224×3               56×56×64      28×28×128      14×14×256       7×7×512

┌────────┐ ┌──────────┐ ┌──────────┐ ┌───────────┐ ┌───────────┐ ┌───────────┐
│        │ │ 7×7 conv │ │ [3×3,64] │ │ [3×3,128] │ │ [3×3,256] │ │ [3×3,512] │
│224×224 │►│ stride 2 │►│ [3×3,64] │►│ [3×3,128] │►│ [3×3,256] │►│ [3×3,512] │
│  RGB   │ │ maxpool  │ │ × 3      │ │ × 4       │ │ × 6       │ │ × 3       │
└────────┘ └──────────┘ │ blocks   │ │ blocks    │ │ blocks    │ │ blocks    │
                        │ (basic)  │ │ (basic)   │ │ (basic)   │ │ (basic)   │
                        └──────────┘ └───────────┘ └───────────┘ └───────────┘
                                                                      │
                                                                 AvgPool → FC
                                                                      │
                                                                    1000

  Total: 1 + (3+4+6+3)×2 = 1 + 32 = 33 conv layers + 1 FC = 34 layers
```

### Residual Block (Basic) — Detailed

```
                    Basic Residual Block
                    ═══════════════════

                         x (input)
                         │
              ┌──────────┤
              │          │
              │          ▼
              │    ┌───────────┐
              │    │ Conv 3×3  │
              │    │ BN + ReLU │
              │    └─────┬─────┘
              │          │
              │          ▼
              │    ┌───────────┐
              │    │ Conv 3×3  │
              │    │ BN        │  (no ReLU here!)
              │    └─────┬─────┘
              │          │
   identity   │          │  F(x)
   shortcut   │          │
              │          │
              └────►  (+)  ◄──── element-wise addition
                      │
                      ▼
                    ReLU     ← ReLU AFTER addition
                      │
                      ▼
                    output = ReLU(F(x) + x)
```

### Bottleneck Residual Block — Detailed

```
                  Bottleneck Residual Block
                  ════════════════════════
                  (used in ResNet-50, 101, 152)

                         x (input)
                         │
              ┌──────────┤
              │          │
              │          ▼
              │    ┌───────────┐
              │    │ Conv 1×1  │  ← reduce channels (256→64)
              │    │ BN + ReLU │
              │    └─────┬─────┘
              │          │
              │          ▼
              │    ┌───────────┐
              │    │ Conv 3×3  │  ← spatial convolution (64→64)
              │    │ BN + ReLU │
              │    └─────┬─────┘
              │          │
              │          ▼
              │    ┌───────────┐
              │    │ Conv 1×1  │  ← expand channels (64→256)
              │    │ BN        │  (no ReLU!)
              │    └─────┬─────┘
              │          │
   identity   │          │  F(x)
   shortcut   │          │
              │          │
              └────►  (+)  ◄──── element-wise addition
                      │
                      ▼
                    ReLU
                      │
                      ▼
                    output


  Channel flow: 256 → 64 → 64 → 256
                  ↑                ↑
               input           output (same as input!)
               
  The "bottleneck" (64) reduces computation of the 3×3 conv.
```

### Downsampling Residual Block

```
             Downsampling Block (stride 2)
             ═════════════════════════════

                         x (input)
                     [56×56×64]
                         │
              ┌──────────┤
              │          │
              │          ▼
              │    ┌───────────┐
              │    │ Conv 3×3  │  stride=2 (or 1×1 s=2 for bottleneck)
              │    │ 128 filt  │
              │    │ BN + ReLU │
              │    └─────┬─────┘
              │      [28×28×128]
              │          │
              │          ▼
              │    ┌───────────┐
              │    │ Conv 3×3  │
              │    │ 128 filt  │
              │    │ BN        │
              │    └─────┬─────┘
              │      [28×28×128]
              │          │
              │          │  F(x)
              │          │
              ▼          │
       ┌───────────┐     │
       │ Conv 1×1  │     │
       │ stride=2  │     │
       │ 128 filt  │     │     Projection shortcut!
       │ BN        │     │     (changes dimensions to match)
       └─────┬─────┘     │
         [28×28×128]     │
              │          │
              └────►  (+)  ◄─── Now dimensions match!
                      │
                      ▼
                    ReLU
                      │
                  [28×28×128]
```

---

## Basic Block vs. Bottleneck Block

### Comparison

```
Basic Block (ResNet-18, 34):         Bottleneck Block (ResNet-50, 101, 152):
════════════════════════════         ═══════════════════════════════════════

  x [H×W×C]                           x [H×W×4C]
  │                                    │
  ├─ shortcut ──────────┐              ├─ shortcut ──────────┐
  │                     │              │                     │
  Conv 3×3, C           │              Conv 1×1, C           │
  BN + ReLU             │              BN + ReLU             │
  │                     │              │                     │
  Conv 3×3, C           │              Conv 3×3, C           │
  BN                    │              BN + ReLU             │
  │                     │              │                     │
  (+) ◄─────────────────┘              Conv 1×1, 4C          │
  │                                    BN                    │
  ReLU                                 │                     │
  │                                    (+) ◄─────────────────┘
  output [H×W×C]                       │
                                       ReLU
                                       │
                                       output [H×W×4C]

  2 conv layers per block              3 conv layers per block
  Params: 2 × (3×3×C×C) = 18C²        Params: C×4C + 9×C×C + C×4C
                                              = 4C² + 9C² + 4C² = 17C²

  For C=64:  18×64² = 73,728          For C=64: 17×64² = 69,632
  Similar params, but bottleneck       The 1×1 convs handle channel
  adds a 3rd non-linearity!           changes cheaply
```

### Why Use Bottleneck?

```
For deeper networks (50+ layers), bottleneck blocks are preferred:

1. COMPUTATIONAL EFFICIENCY
   Basic block at 256 channels: 3×3×256×256 × 2 = 1,179,648 params
   Bottleneck at 256 channels:  1×1×256×64 + 3×3×64×64 + 1×1×64×256
                                = 16,384 + 36,864 + 16,384 = 69,632 params
   → 17× fewer parameters for the same channel width!

2. MORE NON-LINEARITIES
   3 layers = 3 ReLU activations vs. 2
   → More expressive without more computation

3. FLEXIBLE CHANNEL MANAGEMENT
   1×1 convs cheaply change channel dimensions
   → Enables wider networks (256→2048 channels in Stage 4)
```

---

## Identity vs. Projection Shortcuts

### When Dimensions Match: Identity Shortcut

```
y = F(x) + x

  x and F(x) have the SAME dimensions → just add them.
  This is a zero-parameter operation!

  Example: [56×56×64] → ResBlock → [56×56×64]
  Shortcut: just copy x unchanged.
```

### When Dimensions Don't Match: Projection Shortcut

```
When moving between stages (e.g., Stage 1 → Stage 2):
  - Spatial dims halve: 56×56 → 28×28
  - Channels increase: 64 → 128

Two options from the paper:

Option A: Zero-padding shortcut (no extra params)
  Pad the channel dimension with zeros and stride 2 for spatial:
  [56×56×64] → stride-2 subsample → [28×28×64] → pad channels → [28×28×128]
  
  y = F(x) + pad(subsample(x))

Option B: Projection shortcut (1×1 conv) — PREFERRED
  Use a 1×1 convolution with stride 2:
  [56×56×64] → Conv 1×1, 128 filters, stride 2 → [28×28×128]
  
  y = F(x) + W_s · x     (W_s is the projection matrix)

  Extra params: 1×1×64×128 = 8,192 (small)
```

### Shortcut Options Comparison (from the paper)

```
Option A: Identity shortcuts everywhere, zero-pad for dimension changes
Option B: Projection shortcuts only when dimensions change
Option C: Projection shortcuts everywhere (all blocks)

Results on ImageNet (ResNet-34):
  Option A: 25.03% top-1 error
  Option B: 24.52% top-1 error  ← Best trade-off!
  Option C: 24.19% top-1 error

Option B is used in practice — projection only where needed.
Option C adds parameters without significant benefit.
```

---

## ResNet Variants

### Configuration Table

```
Layer         Output    ResNet-18    ResNet-34    ResNet-50    ResNet-101   ResNet-152
              Size      (basic)      (basic)      (bottle)     (bottle)     (bottle)
─────────────────────────────────────────────────────────────────────────────────────
conv1         112×112   7×7, 64, stride 2   (same for all variants)
              56×56     3×3 max pool, stride 2
─────────────────────────────────────────────────────────────────────────────────────
conv2_x       56×56     [3×3, 64]    [3×3, 64]    [1×1, 64 ]   [1×1, 64 ]   [1×1, 64 ]
                        [3×3, 64]    [3×3, 64]    [3×3, 64 ]   [3×3, 64 ]   [3×3, 64 ]
                        × 2          × 3          [1×1, 256]   [1×1, 256]   [1×1, 256]
                                                  × 3          × 3          × 3
─────────────────────────────────────────────────────────────────────────────────────
conv3_x       28×28     [3×3, 128]   [3×3, 128]   [1×1, 128]   [1×1, 128]   [1×1, 128]
                        [3×3, 128]   [3×3, 128]   [3×3, 128]   [3×3, 128]   [3×3, 128]
                        × 2          × 4          [1×1, 512]   [1×1, 512]   [1×1, 512]
                                                  × 4          × 4          × 8
─────────────────────────────────────────────────────────────────────────────────────
conv4_x       14×14     [3×3, 256]   [3×3, 256]   [1×1, 256]   [1×1, 256]   [1×1, 256]
                        [3×3, 256]   [3×3, 256]   [3×3, 256]   [3×3, 256]   [3×3, 256]
                        × 2          × 6          [1×1,1024]   [1×1,1024]   [1×1,1024]
                                                  × 6          × 23         × 36
─────────────────────────────────────────────────────────────────────────────────────
conv5_x       7×7       [3×3, 512]   [3×3, 512]   [1×1, 512]   [1×1, 512]   [1×1, 512]
                        [3×3, 512]   [3×3, 512]   [3×3, 512]   [3×3, 512]   [3×3, 512]
                        × 2          × 3          [1×1,2048]   [1×1,2048]   [1×1,2048]
                                                  × 3          × 3          × 3
─────────────────────────────────────────────────────────────────────────────────────
              1×1       global average pooling
              1000      FC, softmax
─────────────────────────────────────────────────────────────────────────────────────
Total layers:           18           34           50           101          152
Parameters:             11.7M        21.8M        25.6M        44.5M        60.2M
FLOPs:                  1.8B         3.7B         4.1B         7.9B         11.6B
Top-1 error:            30.24%       26.73%       24.80%       23.60%       23.00%
Top-5 error:            10.92%       8.74%        7.80%        7.14%        6.71%
```

### Depth Counting

```
ResNet-18: 
  conv1 (1) + conv2_x (2×2=4) + conv3_x (2×2=4) + 
  conv4_x (2×2=4) + conv5_x (2×2=4) + FC (1) = 18

ResNet-50:
  conv1 (1) + conv2_x (3×3=9) + conv3_x (4×3=12) + 
  conv4_x (6×3=18) + conv5_x (3×3=9) + FC (1) = 50

ResNet-152:
  conv1 (1) + conv2_x (3×3=9) + conv3_x (8×3=24) + 
  conv4_x (36×3=108) + conv5_x (3×3=9) + FC (1) = 152
```

---

## Parameter Calculations

### ResNet-50 Detailed Count

```
Layer              Configuration                   Parameters
──────────────────────────────────────────────────────────────
conv1              7×7×3×64 + 64                      9,472
bn1                64×2 (γ, β)                          128
──────────────────────────────────────────────────────────────
Stage 1 (×3 bottleneck blocks, 64→256)
  Block 1:
    conv1: 1×1×64×64 + 64                              4,160
    conv2: 3×3×64×64 + 64                              36,928
    conv3: 1×1×64×256 + 256                            16,640
    bn×3: (64+64+256)×2                                   768
    proj:  1×1×64×256 + 256                            16,640
    bn_proj: 256×2                                        512
  Block 2-3: (no projection)
    each: 4,160+36,928+16,640+768                      58,496
    × 2 blocks                                        116,992
  Stage 1 total:                                      175,640
──────────────────────────────────────────────────────────────
Stage 2 (×4 bottleneck blocks, 128→512)
  Block 1 (with projection):                         ~214K
  Blocks 2-4:                          3 × ~230K  ≈  ~690K
  Stage 2 total:                                     ~904K
──────────────────────────────────────────────────────────────
Stage 3 (×6 bottleneck blocks, 256→1024)
  Block 1 (with projection):                        ~852K
  Blocks 2-6:                          5 × ~920K ≈  ~4.6M
  Stage 3 total:                                     ~5.5M
──────────────────────────────────────────────────────────────
Stage 4 (×3 bottleneck blocks, 512→2048)
  Block 1 (with projection):                        ~3.4M
  Blocks 2-3:                          2 × ~3.7M ≈  ~7.4M
  Stage 4 total:                                    ~10.8M
──────────────────────────────────────────────────────────────
FC: 2048×1000 + 1000                                ~2.05M
──────────────────────────────────────────────────────────────
Grand Total: ~25.6 million parameters

Compare:
  VGG-16:     138M  (5.4× more)
  GoogLeNet:  6.8M  (3.8× fewer, but less accurate)
  AlexNet:    60M   (2.3× more)
```

### Where Are the Parameters?

```
Stage         Channels    Params      % of Total
──────────────────────────────────────────────────
Stem (conv1)  64           10K         0.04%
Stage 1       64/256      176K         0.69%
Stage 2       128/512     904K         3.53%
Stage 3       256/1024    5.5M        21.5%
Stage 4       512/2048   10.8M        42.2%
FC            2048→1000   2.1M         8.0%
──────────────────────────────────────────────────

Key insight: Unlike VGG (89% in FC), ResNet's parameters are
distributed across conv stages. Stage 4 has the most because
of the 512-channel bottleneck within 2048-channel blocks.
```

---

## Mathematical Foundations

### Gradient Flow Analysis

```
Consider a network with L residual blocks:

  x_l = x_{l-1} + F(x_{l-1}, W_l)    for l = 1, 2, ..., L

Unrolling from layer l to layer L:

  x_L = x_l + Σ_{i=l}^{L-1} F(x_i, W_i)

Gradient of loss w.r.t. x_l:

  ∂L     ∂L     ∂x_L
  ── = ──── · ────
  ∂x_l  ∂x_L   ∂x_l

       ∂L              L-1
  = ──── · (1 + Σ      ∂F(x_i, W_i)/∂x_l)
     ∂x_L        i=l

       ∂L        ∂L       L-1
  = ──── + ──── · Σ      ∂F(x_i, W_i)/∂x_l
     ∂x_L    ∂x_L  i=l
     ↑                    ↑
     Direct               Residual gradient
     gradient path        (may vanish, but the
     (ALWAYS present!)    direct path saves us)

The "1" in the gradient ensures information flows directly
from loss to ANY layer — no matter how deep!
```

### Batch Normalization in ResNet

```
Every conv layer (except the last in each block) is followed by BN + ReLU:

  Conv → BatchNorm → ReLU

BatchNorm:
                x_i - μ_B
  x̂_i = ──────────────────
          √(σ²_B + ε)

  y_i = γ · x̂_i + β

Where:
  μ_B = mini-batch mean
  σ²_B = mini-batch variance
  γ, β = learnable scale and shift
  ε = small constant (1e-5)

In ResNet:
  Conv → BN → ReLU → Conv → BN → (+shortcut) → ReLU
  
  Note: The LAST BN in a block comes BEFORE the addition.
  ReLU is applied AFTER the addition (post-activation ResNet).
```

### Receptive Field

```
ResNet-50 receptive field:

  Layer           RF        Jump
  ─────────────────────────────────
  conv1 (7×7,s2)   7          2
  maxpool (3×3,s2) 11         4
  Stage1 (×3 blocks, each adds 2×3×3):
    After Stage1    75         4
  Stage2 (first block s=2, then ×3):
    After Stage2   203         8
  Stage3 (first block s=2, then ×5):
    After Stage3   459        16
  Stage4 (first block s=2, then ×2):
    After Stage4   483        32

  Final RF: 483 × 483 (much larger than 224×224 input!)
  → The network "sees" the full image context.
```

---

## Worked Example

### Forward Pass Through One Bottleneck Block

```
Input: x = [14, 14, 1024] (in Stage 3 of ResNet-50)

Step 1: 1×1 Conv (squeeze channels)
────────────────────────────────────
  Conv: 1×1×1024×256, no padding
  BN + ReLU

  x₁ = ReLU(BN(Conv1×1(x)))
  x₁ shape: [14, 14, 256]

  At position (7, 7):
    z[7,7,k] = Σ_{c=0}^{1023} w[c,k] · x[7,7,c] + b[k]
    (1024 multiply-adds per output channel × 256 channels)


Step 2: 3×3 Conv (spatial processing)
──────────────────────────────────────
  Conv: 3×3×256×256, padding=1
  BN + ReLU

  x₂ = ReLU(BN(Conv3×3(x₁)))
  x₂ shape: [14, 14, 256]

  At position (7, 7):
    z[7,7,k] = Σ_{c=0}^{255} Σ_{p,q} w[p,q,c,k] · x₁[7+p,7+q,c] + b[k]
    (9 × 256 = 2,304 multiply-adds per output channel × 256 channels)


Step 3: 1×1 Conv (expand channels)
────────────────────────────────────
  Conv: 1×1×256×1024, no padding
  BN (NO ReLU yet!)

  x₃ = BN(Conv1×1(x₂))
  x₃ shape: [14, 14, 1024]    ← same as input!


Step 4: Addition + ReLU
────────────────────────
  output = ReLU(x₃ + x)    ← element-wise add shortcut

  At each position (i, j) and channel c:
    output[i,j,c] = max(0, x₃[i,j,c] + x[i,j,c])

  output shape: [14, 14, 1024]


Summary of this block:
  [14,14,1024] → [14,14,256] → [14,14,256] → [14,14,1024]
       ↑              ↓             ↓              ↓
     input         squeeze       process         expand
                   (1×1)        (3×3)           (1×1)
       │                                          │
       └────────── skip connection ─ (+) ──────────┘
```

---

## PyTorch Implementation

### Basic Residual Block

```python
import torch
import torch.nn as nn


class BasicBlock(nn.Module):
    """Basic residual block for ResNet-18 and ResNet-34."""

    expansion = 1  # Output channels = in_channels × expansion

    def __init__(self, in_channels, out_channels, stride=1, downsample=None):
        super(BasicBlock, self).__init__()

        self.conv1 = nn.Conv2d(in_channels, out_channels,
                               kernel_size=3, stride=stride,
                               padding=1, bias=False)
        self.bn1 = nn.BatchNorm2d(out_channels)

        self.conv2 = nn.Conv2d(out_channels, out_channels,
                               kernel_size=3, stride=1,
                               padding=1, bias=False)
        self.bn2 = nn.BatchNorm2d(out_channels)

        self.relu = nn.ReLU(inplace=True)
        self.downsample = downsample  # Projection shortcut (if needed)

    def forward(self, x):
        identity = x

        out = self.relu(self.bn1(self.conv1(x)))
        out = self.bn2(self.conv2(out))

        if self.downsample is not None:
            identity = self.downsample(x)

        out += identity  # Skip connection
        out = self.relu(out)

        return out
```

### Bottleneck Residual Block

```python
class BottleneckBlock(nn.Module):
    """Bottleneck residual block for ResNet-50, 101, and 152."""

    expansion = 4  # Output channels = out_channels × 4

    def __init__(self, in_channels, out_channels, stride=1, downsample=None):
        super(BottleneckBlock, self).__init__()

        # 1×1 conv: squeeze channels
        self.conv1 = nn.Conv2d(in_channels, out_channels,
                               kernel_size=1, bias=False)
        self.bn1 = nn.BatchNorm2d(out_channels)

        # 3×3 conv: spatial processing
        self.conv2 = nn.Conv2d(out_channels, out_channels,
                               kernel_size=3, stride=stride,
                               padding=1, bias=False)
        self.bn2 = nn.BatchNorm2d(out_channels)

        # 1×1 conv: expand channels
        self.conv3 = nn.Conv2d(out_channels, out_channels * self.expansion,
                               kernel_size=1, bias=False)
        self.bn3 = nn.BatchNorm2d(out_channels * self.expansion)

        self.relu = nn.ReLU(inplace=True)
        self.downsample = downsample

    def forward(self, x):
        identity = x

        out = self.relu(self.bn1(self.conv1(x)))    # Squeeze
        out = self.relu(self.bn2(self.conv2(out)))   # Process
        out = self.bn3(self.conv3(out))              # Expand (no ReLU)

        if self.downsample is not None:
            identity = self.downsample(x)

        out += identity  # Skip connection
        out = self.relu(out)

        return out
```

### Complete ResNet

```python
class ResNet(nn.Module):
    """
    ResNet implementation supporting all standard variants.

    Args:
        block: BasicBlock or BottleneckBlock
        layers: list of block counts for each stage [N1, N2, N3, N4]
        num_classes: number of output classes
    """
    def __init__(self, block, layers, num_classes=1000):
        super(ResNet, self).__init__()
        self.in_channels = 64

        # ─── Stem ───
        self.conv1 = nn.Conv2d(3, 64, kernel_size=7, stride=2,
                               padding=3, bias=False)
        self.bn1 = nn.BatchNorm2d(64)
        self.relu = nn.ReLU(inplace=True)
        self.maxpool = nn.MaxPool2d(kernel_size=3, stride=2, padding=1)

        # ─── Residual Stages ───
        self.layer1 = self._make_layer(block, 64,  layers[0], stride=1)
        self.layer2 = self._make_layer(block, 128, layers[1], stride=2)
        self.layer3 = self._make_layer(block, 256, layers[2], stride=2)
        self.layer4 = self._make_layer(block, 512, layers[3], stride=2)

        # ─── Classifier ───
        self.avgpool = nn.AdaptiveAvgPool2d((1, 1))
        self.fc = nn.Linear(512 * block.expansion, num_classes)

        # ─── Weight Initialization ───
        for m in self.modules():
            if isinstance(m, nn.Conv2d):
                nn.init.kaiming_normal_(m.weight, mode='fan_out',
                                        nonlinearity='relu')
            elif isinstance(m, nn.BatchNorm2d):
                nn.init.constant_(m.weight, 1)
                nn.init.constant_(m.bias, 0)

    def _make_layer(self, block, out_channels, num_blocks, stride):
        """Build a residual stage with num_blocks blocks."""
        downsample = None

        # Need projection shortcut if dimensions change
        if stride != 1 or self.in_channels != out_channels * block.expansion:
            downsample = nn.Sequential(
                nn.Conv2d(self.in_channels, out_channels * block.expansion,
                          kernel_size=1, stride=stride, bias=False),
                nn.BatchNorm2d(out_channels * block.expansion),
            )

        layers = []

        # First block (may downsample)
        layers.append(block(self.in_channels, out_channels,
                           stride, downsample))
        self.in_channels = out_channels * block.expansion

        # Remaining blocks
        for _ in range(1, num_blocks):
            layers.append(block(self.in_channels, out_channels))

        return nn.Sequential(*layers)

    def forward(self, x):
        # Stem
        x = self.relu(self.bn1(self.conv1(x)))   # [batch, 64, 112, 112]
        x = self.maxpool(x)                       # [batch, 64, 56, 56]

        # Residual stages
        x = self.layer1(x)                        # [batch, 64/256, 56, 56]
        x = self.layer2(x)                        # [batch, 128/512, 28, 28]
        x = self.layer3(x)                        # [batch, 256/1024, 14, 14]
        x = self.layer4(x)                        # [batch, 512/2048, 7, 7]

        # Classifier
        x = self.avgpool(x)                       # [batch, C, 1, 1]
        x = torch.flatten(x, 1)                   # [batch, C]
        x = self.fc(x)                            # [batch, num_classes]

        return x


# ─── Factory Functions ───
def resnet18(num_classes=1000):
    return ResNet(BasicBlock, [2, 2, 2, 2], num_classes)

def resnet34(num_classes=1000):
    return ResNet(BasicBlock, [3, 4, 6, 3], num_classes)

def resnet50(num_classes=1000):
    return ResNet(BottleneckBlock, [3, 4, 6, 3], num_classes)

def resnet101(num_classes=1000):
    return ResNet(BottleneckBlock, [3, 4, 23, 3], num_classes)

def resnet152(num_classes=1000):
    return ResNet(BottleneckBlock, [3, 8, 36, 3], num_classes)


# ─── Verify All Variants ───
for name, fn in [('ResNet-18', resnet18), ('ResNet-34', resnet34),
                 ('ResNet-50', resnet50), ('ResNet-101', resnet101),
                 ('ResNet-152', resnet152)]:
    model = fn(num_classes=1000)
    params = sum(p.numel() for p in model.parameters())
    x = torch.randn(1, 3, 224, 224)
    out = model(x)
    print(f"{name:>10}: {params/1e6:6.1f}M params, output={out.shape}")
```

**Expected output:**
```
  ResNet-18:   11.7M params, output=torch.Size([1, 1000])
  ResNet-34:   21.8M params, output=torch.Size([1, 1000])
  ResNet-50:   25.6M params, output=torch.Size([1, 1000])
 ResNet-101:   44.5M params, output=torch.Size([1, 1000])
 ResNet-152:   60.2M params, output=torch.Size([1, 1000])
```

### Using Pretrained ResNet

```python
import torchvision.models as models

# Load pretrained ResNet-50
resnet = models.resnet50(weights='IMAGENET1K_V2')

# Use as feature extractor
features = nn.Sequential(*list(resnet.children())[:-2])  # Remove avgpool + fc
# features(x) → [batch, 2048, 7, 7]

# Fine-tune for 10 classes
resnet.fc = nn.Linear(2048, 10)

# Freeze early layers
for name, param in resnet.named_parameters():
    if 'layer4' not in name and 'fc' not in name:
        param.requires_grad = False

trainable = sum(p.numel() for p in resnet.parameters() if p.requires_grad)
total = sum(p.numel() for p in resnet.parameters())
print(f"Trainable: {trainable:,} / {total:,} ({100*trainable/total:.1f}%)")
```

---

## Applications

### 1. Image Classification
- **ILSVRC-2015 winner**: 3.57% top-5 error (super-human)
- The default backbone for most vision tasks since 2016

### 2. Object Detection
- **Faster R-CNN with ResNet** became the new standard
- **Feature Pyramid Networks (FPN)** built on ResNet stages
- ResNet-50 and ResNet-101 most common detection backbones

### 3. Semantic Segmentation
- **DeepLab** uses dilated ResNet as backbone
- **PSPNet** uses ResNet for scene parsing
- **U-Net variants** use ResNet encoders

### 4. Generative Models
- **StyleGAN** uses residual blocks
- **Super-resolution** networks (EDSR, RCAN) are built on ResBlocks
- **Diffusion models** heavily use residual connections

### 5. Beyond Vision
- **ResNet principles applied to NLP**: Transformer's residual connections
- **Audio processing**: residual blocks in speech models
- **Reinforcement learning**: residual networks for policy/value functions

### 6. Transfer Learning Standard
- ResNet-50 is the **most commonly used backbone** for transfer learning
- Pretrained on ImageNet, fine-tuned for virtually any vision task
- Available in every major framework (PyTorch, TensorFlow, JAX)

---

## Summary Table

| Aspect | Details |
|--------|---------|
| **Architecture** | Stem + 4 stages of residual blocks + GAP + FC |
| **Key Innovation** | Skip/residual connections: y = F(x) + x |
| **Problem Solved** | Degradation problem in deep networks |
| **Input Size** | 224 × 224 × 3 |
| **Block Types** | Basic (2-layer) and Bottleneck (3-layer) |
| **Shortcut Types** | Identity (dims match) and Projection (1×1 conv) |
| **Parameters (ResNet-50)** | 25.6M |
| **FLOPs (ResNet-50)** | 4.1B |
| **Top-5 Error** | 3.57% (ILSVRC-2015, surpassing humans) |
| **Normalization** | Batch Normalization after every conv |
| **Pooling** | Global Average Pooling (no FC bloat) |
| **Depth** | 18, 34, 50, 101, 152 (even 1000+ possible) |
| **Gradient Flow** | ∂L/∂x always has a direct path (the "+1") |
| **Legacy** | Most influential CNN; residual idea used everywhere |

---

## Revision Questions

### Q1: The Degradation Problem
**What is the degradation problem, and why is it different from overfitting? How do residual connections solve it?**

<details>
<summary>Answer</summary>

**Degradation**: Deeper plain networks achieve *higher training error* (not just test error) than shallower networks. This is NOT overfitting (which would show lower training error but higher test error).

**Cause**: It's an optimization difficulty — deeper networks are harder for SGD to optimize. Even though a deep network could theoretically learn identity mappings for extra layers, gradient-based optimization struggles to find this solution.

**Solution**: Residual connections reformulate the problem: instead of learning H(x), learn F(x) = H(x) - x. If identity is optimal, the network just needs F(x) = 0, which is easy (drive weights to zero). The skip connection provides a gradient "highway" — ∂L/∂x always includes a direct path component, preventing gradient vanishing.
</details>

### Q2: Residual Block Math
**Write the forward equations for a bottleneck residual block. Explain why ReLU is applied after addition.**

<details>
<summary>Answer</summary>

```
Forward pass:
  x₁ = ReLU(BN(Conv1×1(x)))      # squeeze channels
  x₂ = ReLU(BN(Conv3×3(x₁)))     # spatial processing  
  x₃ = BN(Conv1×1(x₂))           # expand channels (NO ReLU)
  y  = ReLU(x₃ + x)              # add shortcut, THEN ReLU

Equivalently: y = ReLU(F(x) + x)
```

ReLU is applied AFTER addition because:
1. If ReLU were applied BEFORE addition, the shortcut would pass through non-negative values only, limiting its expressiveness
2. Applying ReLU after means the skip connection carries the full (positive and negative) signal
3. The BN before addition can produce negative values, which combine with the shortcut before being thresholded
</details>

### Q3: Basic vs. Bottleneck
**Compare the parameter count of a basic block and a bottleneck block operating at the same effective channel width. When should you use each?**

<details>
<summary>Answer</summary>

For effective width C=64 (output channels of the 3×3 conv):

**Basic block** (in=64, out=64):
- Conv1: 3×3×64×64 = 36,864
- Conv2: 3×3×64×64 = 36,864
- Total: 73,728 params, output channels: 64

**Bottleneck block** (in=256, bottleneck=64, out=256):
- Conv1: 1×1×256×64 = 16,384
- Conv2: 3×3×64×64 = 36,864
- Conv3: 1×1×64×256 = 16,384
- Total: 69,632 params, output channels: 256

The bottleneck has fewer params but produces 4× more output channels! It also has 3 non-linearities vs. 2.

**Use basic blocks** for shallower networks (ResNet-18/34) where the overhead of 1×1 convolutions isn't worth it. **Use bottleneck blocks** for deeper networks (ResNet-50+) where the 4× channel expansion enables richer representations without excessive computation.
</details>

### Q4: Gradient Flow
**Mathematically show why gradients can flow through residual connections even in very deep networks.**

<details>
<summary>Answer</summary>

For a residual block: x_l = x_{l-1} + F(x_{l-1})

Unrolling from layer l to L:
  x_L = x_l + Σ_{i=l}^{L-1} F(x_i)

Taking the gradient:
  ∂L/∂x_l = ∂L/∂x_L · (1 + ∂/∂x_l Σ F(x_i))

The key is the **"1"** term — it provides a direct gradient path from the loss to any layer, regardless of depth. Even if all the ∂F/∂x terms vanish (saturated layers), the gradient ∂L/∂x_L flows directly to x_l unchanged.

In a plain network: ∂L/∂x_l = Π_{i=l}^{L-1} ∂H(x_i)/∂x_i — a product of many terms, which vanishes exponentially if any term < 1.
</details>

### Q5: Architecture Comparison
**Compare ResNet-50 with VGG-16 in terms of parameters, FLOPs, accuracy, and design philosophy.**

<details>
<summary>Answer</summary>

| Aspect | VGG-16 | ResNet-50 |
|--------|--------|-----------|
| Parameters | 138M | 25.6M (5.4× fewer) |
| FLOPs | 15.5B | 4.1B (3.8× fewer) |
| Top-5 Error | 7.3% | 7.8%* / 3.57%** |
| Depth | 16 layers | 50 layers (3.1× deeper) |
| FC params | 124M (89%) | 2.1M (8%) |
| Skip connections | No | Yes |
| Batch Norm | No | Yes |
| Bottleneck | No | Yes (1×1 → 3×3 → 1×1) |

*Single model; **Ensemble

VGG scales by brute force (more params); ResNet scales by depth with skip connections, achieving better or comparable accuracy with far fewer parameters.
</details>

### Q6: Practical Usage
**You need to classify images into 50 categories with a dataset of 10,000 images. Which ResNet variant would you choose and why? How would you set up training?**

<details>
<summary>Answer</summary>

**Model choice**: ResNet-18 or ResNet-34 with pretrained ImageNet weights.

**Reasoning**: 10K images is small, so a lighter model reduces overfitting risk. ResNet-50 would work too but has more parameters to potentially overfit.

**Training setup**:
1. Load pretrained ResNet-18/34 from torchvision
2. Replace the final FC layer: `model.fc = nn.Linear(512, 50)`
3. Freeze early layers (stages 1-2), fine-tune stages 3-4 + FC
4. Use aggressive data augmentation (random crops, flips, color jitter)
5. Learning rate: 1e-3 for FC, 1e-4 for fine-tuned conv layers
6. Optimizer: Adam or SGD with momentum
7. Scheduler: Cosine annealing or ReduceLROnPlateau
8. Train for 30-50 epochs with early stopping
</details>

---

## Key Takeaways

```
╔══════════════════════════════════════════════════════════════════════╗
║  ResNet Key Takeaways                                               ║
║                                                                      ║
║  1. Skip connections: y = F(x) + x solves the degradation problem  ║
║  2. Enables training of 100+ layer networks (even 1000+ in theory) ║
║  3. First to surpass human performance on ImageNet (3.57% top-5)   ║
║  4. Gradient highway: the "+1" ensures gradients always flow       ║
║  5. Bottleneck block: 1×1→3×3→1×1 is efficient for deep networks  ║
║  6. ResNet-50: only 25.6M params yet deeper than 138M-param VGG    ║
║  7. Most influential CNN ever — residual connections used EVERYWHERE║
║     (Transformers, GANs, diffusion models, speech, NLP, etc.)      ║
╚══════════════════════════════════════════════════════════════════════╝
```

---

[← Previous: GoogLeNet/Inception](04-googlenet-inception.md) | [Back to Unit Overview](../README.md) | [Next: DenseNet →](06-densenet.md)
