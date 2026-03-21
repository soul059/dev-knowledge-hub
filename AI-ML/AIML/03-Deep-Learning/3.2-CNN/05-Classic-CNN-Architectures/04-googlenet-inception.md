# 🔬 GoogLeNet / Inception: Thinking in Parallel

[← Previous: VGGNet](03-vggnet.md) | [Back to Unit Overview](../README.md) | [Next: ResNet →](05-resnet.md)

---

## 📋 Table of Contents

- [Overview](#overview)
- [Historical Context](#historical-context)
- [The Inception Module](#the-inception-module)
- [1×1 Convolutions as Bottlenecks](#1×1-convolutions-as-bottlenecks)
- [GoogLeNet Architecture](#googlenet-architecture)
- [ASCII Architecture Diagrams](#ascii-architecture-diagrams)
- [Auxiliary Classifiers](#auxiliary-classifiers)
- [Parameter Calculations](#parameter-calculations)
- [Mathematical Foundations](#mathematical-foundations)
- [Worked Example](#worked-example)
- [PyTorch Implementation](#pytorch-implementation)
- [Inception v2, v3, v4](#inception-v2-v3-v4)
- [Applications](#applications)
- [Summary Table](#summary-table)
- [Revision Questions](#revision-questions)

---

## Overview

**GoogLeNet** (also known as **Inception v1**) was developed by **Christian Szegedy et al.** at Google and won the **ILSVRC-2014** classification challenge. Its key innovation is the **Inception module** — a parallel multi-branch architecture that processes information at multiple spatial scales simultaneously.

### Key Facts

| Property | Value |
|----------|-------|
| **Year** | 2014 |
| **Authors** | Szegedy, Liu, Jia, Sermanet, Reed, Anguelov, Erhan, Vanhoucke, Rabinovich |
| **Paper** | Going Deeper with Convolutions |
| **Competition** | ILSVRC-2014 winner (6.67% top-5 error) |
| **Input Size** | 224 × 224 × 3 |
| **Parameters** | ~6.8 million (only!) |
| **Depth** | 22 layers (with parameters) |
| **Key Innovation** | Inception module, 1×1 conv bottleneck |
| **Named After** | The movie "Inception" — "We need to go deeper" |

### The Efficiency Revolution

```
Parameters Comparison:
══════════════════════

  AlexNet (2012):  │████████████████████████████████████████│  60M params
  VGG-16  (2014):  │████████████████████████████████████████████████████████████████████│  138M params
  GoogLeNet(2014): │███│  6.8M params  ← 20× fewer than VGG!

  Yet GoogLeNet BEAT VGG in classification accuracy!
  This proved: more parameters ≠ better accuracy.
  Smart architecture design > brute force scaling.
```

---

## Historical Context

### The Problem with Naïve Scaling

By 2014, it was clear that deeper and wider networks performed better. But naïve scaling had problems:

1. **More parameters → more overfitting** (VGG-16: 138M params)
2. **More computation → slower training/inference** (VGG-16: 15.5B FLOPs)
3. **Uniform filter size → missed multi-scale features** (what if the relevant pattern is 1×1 in one place and 5×5 in another?)

### Google's Solution: Compute Efficiently

Instead of going wider uniformly, GoogLeNet asked: **"What if the network could choose the right filter size at each location?"**

The answer was the **Inception module** — use ALL filter sizes in parallel and let the network learn which ones are important.

---

## The Inception Module

### Naïve Inception (Conceptual Version)

The basic idea: apply multiple filter sizes in parallel and concatenate the results.

```
                    Naïve Inception Module
                    ═════════════════════

                         ┌── Previous Layer ──┐
                         │    (28×28×256)      │
                         └─────────┬───────────┘
                    ┌──────────────┼──────────────┐──────────────┐
                    ▼              ▼              ▼              ▼
              ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐
              │ 1×1 conv │  │ 3×3 conv │  │ 5×5 conv │  │ 3×3 max  │
              │ 128 filt │  │ 192 filt │  │  96 filt │  │  pool    │
              │          │  │  pad=1   │  │  pad=2   │  │  pad=1   │
              └────┬─────┘  └────┬─────┘  └────┬─────┘  └────┬─────┘
                   │              │              │              │
              [28×28×128]   [28×28×192]   [28×28×96]    [28×28×256]
                   │              │              │              │
                   └──────────────┼──────────────┼──────────────┘
                                  ▼
                         Channel Concatenation
                         [28×28×(128+192+96+256)]
                          = [28×28×672]

  Problem: Output has 672 channels!
           → Computation explodes with depth
           → 5×5 conv on 256 channels is very expensive
```

### Why Naïve Inception is Too Expensive

```
Computation for the naïve 5×5 branch alone:
  Input:  28×28×256
  Filter: 5×5×256×96

  FLOPs = 2 × 5 × 5 × 256 × 96 × 28 × 28
        = 2 × 25 × 256 × 96 × 784
        ≈ 965 million FLOPs (just this one branch!)

  And output channels keep growing: 672 → even more in next module
  → Computationally infeasible!
```

### Inception Module with Dimensionality Reduction

The solution: **1×1 convolutions** as bottlenecks before expensive operations.

```
            Inception Module (with 1×1 bottleneck reduction)
            ═══════════════════════════════════════════════

                         ┌── Previous Layer ──┐
                         │    (28×28×256)      │
                         └─────────┬───────────┘
                    ┌──────────────┼──────────────┐──────────────┐
                    ▼              ▼              ▼              ▼
              ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐
              │ 1×1 conv │  │ 1×1 conv │  │ 1×1 conv │  │ 3×3 max  │
              │  64 filt │  │  96 filt │  │  16 filt │  │  pool    │
              │          │  │(bottleneck)│  │(bottleneck)│  │  pad=1   │
              └────┬─────┘  └────┬─────┘  └────┬─────┘  └────┬─────┘
                   │              ▼              ▼              ▼
                   │        ┌──────────┐  ┌──────────┐  ┌──────────┐
                   │        │ 3×3 conv │  │ 5×5 conv │  │ 1×1 conv │
                   │        │ 128 filt │  │  32 filt │  │  32 filt │
                   │        │  pad=1   │  │  pad=2   │  │(reduce ch)│
                   │        └────┬─────┘  └────┬─────┘  └────┬─────┘
                   │              │              │              │
              [28×28×64]   [28×28×128]    [28×28×32]    [28×28×32]
                   │              │              │              │
                   └──────────────┴──────────────┴──────────────┘
                                       ▼
                              Channel Concatenation
                              [28×28×(64+128+32+32)]
                               = [28×28×256]

  Now output is 256 channels (same as input!) — controllable!
```

### Computation Savings

```
5×5 branch comparison:
═══════════════════════

Naïve (no bottleneck):
  Input: 28×28×256
  5×5 conv: 256→96
  FLOPs: 2 × 25 × 256 × 96 × 784 = 965M

With bottleneck:
  Input: 28×28×256
  1×1 conv: 256→16  → FLOPs: 2 × 1 × 256 × 16 × 784 = 6.4M
  5×5 conv: 16→32   → FLOPs: 2 × 25 × 16 × 32 × 784 = 20.1M
  Total: 26.5M

Savings: 965M / 26.5M ≈ 36× fewer FLOPs!
```

---

## 1×1 Convolutions as Bottlenecks

### What is a 1×1 Convolution?

A 1×1 convolution operates on each spatial position independently, combining information across channels:

```
1×1 Convolution = "Cross-Channel Pooling" or "Network in Network"
═════════════════════════════════════════════════════════════════

Input: H × W × C_in
                                    1×1 conv
                                   C_out filters
  Channel dim                         ▼
      │
      ▼                    ┌─────────────────────┐
  ┌───┬───┬───┐           │  For EACH (i,j):    │
  │c1 │c2 │c3 │──────────►│  y[i,j,k] = Σ_c     │──► ┌───┬───┐
  │   │   │   │ at (i,j)  │    w[c,k]·x[i,j,c]  │    │k1 │k2 │
  └───┴───┴───┘           │    + b[k]            │    └───┴───┘
  3 channels               └─────────────────────┘    2 channels

  It's essentially a linear layer applied to the channel dimension
  at each spatial position independently.
```

### Three Roles of 1×1 Convolutions

```
Role 1: Dimensionality Reduction (C_out < C_in)
─────────────────────────────────────────────────
  [H×W×256] ──1×1 conv, 64 filters──► [H×W×64]
  
  Reduces 256 channels to 64 before an expensive 3×3 or 5×5 conv.
  Saves ~75% of computation in the subsequent layer.


Role 2: Dimensionality Expansion (C_out > C_in)
─────────────────────────────────────────────────
  [H×W×64] ──1×1 conv, 256 filters──► [H×W×256]
  
  Expands channels back up after processing.


Role 3: Adding Non-linearity
─────────────────────────────
  [H×W×256] ──1×1 conv, 256 filters──► [H×W×256]
  
  Same number of channels, but adds a ReLU non-linearity.
  Like an extra "hidden layer" at each spatial position.
```

### Mathematical View

```
For a 1×1 convolution with C_in input channels and C_out output channels:

                 C_in-1
y[i, j, k] = b_k + ∑    w[c, k] · x[i, j, c]
                  c=0

This is equivalent to a fully connected layer (linear transform)
applied independently at each spatial position (i, j):

  y_{ij} = W · x_{ij} + b

Where:
  x_{ij} ∈ R^{C_in}   (vector of C_in channel values at position (i,j))
  W ∈ R^{C_out × C_in} (weight matrix)
  b ∈ R^{C_out}        (bias vector)
  y_{ij} ∈ R^{C_out}   (output vector at position (i,j))

Parameters: C_in × C_out + C_out
FLOPs:      2 × C_in × C_out × H × W
```

---

## GoogLeNet Architecture

### Complete Architecture

GoogLeNet consists of:
- Initial convolution/pooling stem
- 9 Inception modules (in 3 groups)
- 2 auxiliary classifiers (training only)
- Global average pooling (replaces FC layers!)
- Single linear layer for output

```
GoogLeNet Layer-by-Layer:
═════════════════════════

Layer               Output Size     Parameters
──────────────────────────────────────────────
Conv 7×7, stride 2   112×112×64       9,472
MaxPool 3×3, s=2      56×56×64           0
Conv 1×1              56×56×64       4,160
Conv 3×3, pad=1       56×56×192     110,784
MaxPool 3×3, s=2      28×28×192          0
──────────────────────────────────────────────
Inception (3a)        28×28×256
Inception (3b)        28×28×480
MaxPool 3×3, s=2      14×14×480          0
──────────────────────────────────────────────
Inception (4a)        14×14×512
Inception (4b)        14×14×512
Inception (4c)        14×14×512
Inception (4d)        14×14×528
Inception (4e)        14×14×832
MaxPool 3×3, s=2       7×7×832           0
──────────────────────────────────────────────
Inception (5a)         7×7×832
Inception (5b)         7×7×1024
──────────────────────────────────────────────
AvgPool 7×7            1×1×1024          0
Dropout (40%)          1×1×1024          0
Linear                 1000          1,025,000
Softmax                1000              0
──────────────────────────────────────────────
Total:                           ~6.8 million
```

### Inception Module Configurations

```
Module  │ 1×1  │ 3×3_reduce │ 3×3  │ 5×5_reduce │ 5×5 │ pool_proj │ Output
────────┼──────┼────────────┼──────┼────────────┼─────┼───────────┼────────
3a      │  64  │     96     │ 128  │     16     │  32 │    32     │  256
3b      │ 128  │    128     │ 192  │     32     │  96 │    64     │  480
4a      │ 192  │     96     │ 208  │     16     │  48 │    64     │  512
4b      │ 160  │    112     │ 224  │     24     │  64 │    64     │  512
4c      │ 128  │    128     │ 256  │     24     │  64 │    64     │  512
4d      │ 112  │    144     │ 288  │     32     │  64 │    64     │  528
4e      │ 256  │    160     │ 320  │     32     │ 128 │   128     │  832
5a      │ 256  │    160     │ 320  │     32     │ 128 │   128     │  832
5b      │ 384  │    192     │ 384  │     48     │ 128 │   128     │ 1024

Output = 1×1 + 3×3 + 5×5 + pool_proj
  e.g., 3a: 64 + 128 + 32 + 32 = 256
```

---

## ASCII Architecture Diagrams

### Full GoogLeNet Architecture

```
                          GoogLeNet / Inception v1
                          ════════════════════════

  INPUT                                             ┌──────────────┐
 224×224×3                                          │ Aux Classifier│
    │                                               │ (at 4a)      │
    ▼                                               └──────┬───────┘
┌────────────┐                                             │
│ Conv 7×7   │ 112×112×64                                  │
│ stride 2   │                                             │
│ + MaxPool  │ 56×56×64                                    │
│ Conv 1×1   │ 56×56×64                                    │
│ Conv 3×3   │ 56×56×192                                   │
│ + MaxPool  │ 28×28×192                                   │
└─────┬──────┘                                             │
      │                                                    │
      ▼                                                    │
┌─────────────┐                                            │
│ Inception 3a│ 28×28×256                                  │
│ Inception 3b│ 28×28×480                                  │
│ + MaxPool   │ 14×14×480                                  │
└─────┬───────┘                                            │
      │                                                    │
      ▼                                                    │
┌─────────────┐ ───────────── Aux Classifier 1 ───────────►│
│ Inception 4a│ 14×14×512                                  │
│ Inception 4b│ 14×14×512                                  │
│ Inception 4c│ 14×14×512                                  │
│ Inception 4d│ 14×14×528 ── Aux Classifier 2 ───────────►│
│ Inception 4e│ 14×14×832                                  │
│ + MaxPool   │  7×7×832                                   │
└─────┬───────┘                                            │
      │                                                    │
      ▼                                                    │
┌─────────────┐                                            │
│ Inception 5a│  7×7×832                                   │
│ Inception 5b│  7×7×1024                                  │
└─────┬───────┘                                            │
      │                                                    │
      ▼                                                    │
┌─────────────┐           Total Loss:                      │
│ AvgPool 7×7 │ 1×1×1024  L = L_main + 0.3×L_aux1         │
│ Dropout(0.4)│           + 0.3×L_aux2                     │
│ FC → 1000   │                                            │
│ Softmax     │◄───────────────────────────────────────────┘
└─────────────┘
```

### Inception Module Internals (Detailed)

```
                    Inception Module (e.g., 3a)
     ═══════════════════════════════════════════════════

     Input: 28×28×192

     ┌─────────────────────────────────────────────────────────┐
     │                                                         │
     ▼              ▼                ▼                ▼        │
┌─────────┐   ┌─────────┐     ┌─────────┐     ┌──────────┐   │
│ 1×1     │   │ 1×1     │     │ 1×1     │     │ 3×3 max  │   │
│ conv    │   │ conv    │     │ conv    │     │ pool     │   │
│ 64 filt │   │ 96 filt │     │ 16 filt │     │ stride=1 │   │
│         │   │(reduce) │     │(reduce) │     │ pad=1    │   │
└────┬────┘   └────┬────┘     └────┬────┘     └────┬─────┘   │
     │              │              │                │         │
     │              ▼              ▼                ▼         │
     │        ┌─────────┐   ┌─────────┐      ┌─────────┐    │
     │        │ 3×3     │   │ 5×5     │      │ 1×1     │    │
     │        │ conv    │   │ conv    │      │ conv    │    │
     │        │128 filt │   │ 32 filt │      │ 32 filt │    │
     │        │ pad=1   │   │ pad=2   │      │(project)│    │
     │        └────┬────┘   └────┬────┘      └────┬────┘    │
     │              │              │                │         │
  28×28×64    28×28×128      28×28×32          28×28×32      │
     │              │              │                │         │
     └──────────────┴──────────────┴────────────────┘        │
                          │                                   │
                    Concatenate                               │
                    along channels                            │
                          │                                   │
                    28×28×256                                  │
                    (64+128+32+32)                             │
                          │                                   │
                          ▼                                   │
                    Output: 28×28×256                          │
     └─────────────────────────────────────────────────────────┘
```

---

## Auxiliary Classifiers

### Purpose

Deep networks can suffer from **vanishing gradients** — the gradient signal weakens as it propagates backward through many layers. Auxiliary classifiers inject additional gradient signal at intermediate points.

```
Auxiliary Classifier Architecture:
══════════════════════════════════

  Input from Inception module (e.g., 14×14×512)
       │
       ▼
  AvgPool 5×5, stride 3  → 4×4×512
       │
  Conv 1×1, 128 filters  → 4×4×128
       │
  Flatten               → 2,048
       │
  FC 1024 + ReLU
       │
  Dropout (70%)
       │
  FC 1000 (num_classes)
       │
  Softmax
       │
  Loss (weighted 0.3)

Total loss during training:
  L = L_main + 0.3 × L_aux1 + 0.3 × L_aux2

At inference: auxiliary classifiers are removed (only main output used)
```

### Why Auxiliary Classifiers Help

```
Gradient flow WITHOUT auxiliary classifiers:
────────────────────────────────────────────

  Loss → FC → Inc5b → Inc5a → Inc4e → Inc4d → Inc4c → Inc4b → Inc4a → ...
    ▼                                                                    ▼
  Strong                                                              Weak
  gradient                    gradient diminishes                   gradient

Gradient flow WITH auxiliary classifiers:
─────────────────────────────────────────

  Loss_main → FC → Inc5b → Inc5a → Inc4e → Inc4d → ...
                                              ↑
  Loss_aux2 ──────────────────────────────────┘
                        Additional gradient!

  Loss_aux1 → Inc4a → Inc3b → Inc3a → ...
                  ↑
                  └─── Direct gradient at layer 4a!
```

**Note:** Later research (Inception v3 paper) showed that auxiliary classifiers act more as **regularizers** than as gradient highways. They slightly improve the final accuracy but are not essential.

---

## Parameter Calculations

### Inception Module 3a (Detailed)

```
Input: 28×28×192

Branch 1: 1×1 conv, 64 filters
  Params: 1×1×192×64 + 64 = 12,352

Branch 2: 1×1 conv (96) → 3×3 conv (128)
  1×1: 1×1×192×96 + 96  = 18,528
  3×3: 3×3×96×128 + 128  = 110,720
  Subtotal:               129,248

Branch 3: 1×1 conv (16) → 5×5 conv (32)
  1×1: 1×1×192×16 + 16  = 3,088
  5×5: 5×5×16×32 + 32    = 12,832
  Subtotal:               15,920

Branch 4: 3×3 maxpool → 1×1 conv (32)
  Pool: 0 params
  1×1: 1×1×192×32 + 32 = 6,176

Module 3a Total: 12,352 + 129,248 + 15,920 + 6,176 = 163,696
```

### Complete GoogLeNet Parameter Count

```
Component              Parameters
──────────────────────────────────
Stem (initial layers)    ~125,000
Inception 3a             163,696
Inception 3b             388,736
Inception 4a             376,176
Inception 4b             449,160
Inception 4c             510,104
Inception 4d             605,376
Inception 4e           1,444,080
Inception 5a           1,444,080
Inception 5b           2,048,560
──────────────────────────────────
Subtotal (features)   ~7,555,000
Aux classifier 1       ~3,000,000
Aux classifier 2       ~3,000,000
Final FC (1024→1000)   1,025,000
──────────────────────────────────
Training total:       ~14,500,000
Inference total:       ~6,800,000  (no aux classifiers)

Compare:
  VGG-16:    138,000,000  → 20× more!
  AlexNet:    60,000,000  → 9× more!
  GoogLeNet:   6,800,000
```

### Why So Few Parameters?

```
Three key design choices:

1. 1×1 bottleneck convolutions
   → Reduce channels before expensive 3×3/5×5 operations
   → Dramatically fewer weights

2. Global Average Pooling (instead of FC layers)
   → Input: 7×7×1024  → AvgPool → 1×1×1024
   → Eliminates the massive 7×7×1024 → 4096 FC layer
   → VGG FC1: 25,088 × 4,096 = 102M params
   → GoogLeNet: AvgPool → Linear 1024 → 1000 = 1M params

3. Factored convolutions
   → Split computation into parallel branches
   → Each branch processes fewer channels
```

---

## Mathematical Foundations

### Global Average Pooling

```
Instead of flattening and using FC layers:
  Flatten: [7×7×1024] → [50,176] → FC → [4096] → FC → [1000]
  Params: 50,176 × 4,096 + 4,096 × 1,000 ≈ 209M

Global Average Pooling:
  For each channel c:
              1    H-1  W-1
  y[c] = ──────── ∑    ∑    x[i, j, c]
           H × W  i=0  j=0

  [7×7×1024] → [1×1×1024] → FC → [1000]
  Params: 1,024 × 1,000 + 1,000 ≈ 1M

  Savings: 209M → 1M = 209× fewer parameters!

Benefits:
  1. Massive parameter reduction
  2. Acts as structural regularizer
  3. More interpretable (each channel = one "feature score")
  4. Allows variable input sizes at test time
```

### Concatenation Along Channels

```
Given N branches with outputs o_1, o_2, ..., o_N:
  o_i has shape [H × W × C_i]

Concatenation:
  output = concat(o_1, o_2, ..., o_N, dim=channels)
  output shape: [H × W × (C_1 + C_2 + ... + C_N)]

All branches must produce the SAME spatial dimensions (H × W).
This is ensured by appropriate padding in each branch.
```

### FLOPs of Inception Module 3a

```
Branch 1: 1×1 conv (192→64)
  2 × 1×1 × 192 × 64 × 28×28 = 19.3M

Branch 2: 1×1 (192→96) + 3×3 (96→128)
  2 × 1 × 192 × 96 × 784    = 28.9M
  2 × 9 × 96 × 128 × 784    = 173.4M
  Subtotal: 202.3M

Branch 3: 1×1 (192→16) + 5×5 (16→32)
  2 × 1 × 192 × 16 × 784    = 4.8M
  2 × 25 × 16 × 32 × 784    = 20.1M
  Subtotal: 24.9M

Branch 4: pool + 1×1 (192→32)
  Pool: ~negligible
  2 × 1 × 192 × 32 × 784    = 9.6M

Total: 256.1M FLOPs (for one module)

Without bottleneck (naïve version with same outputs):
  5×5 branch: 2 × 25 × 192 × 32 × 784 = 241M (10× more for this branch alone!)
```

---

## Worked Example

### Tracing Through Inception Module 3a

```
Input: 28×28×192 (feature maps from the stem)

Branch 1: 1×1 Convolution (direct)
─────────────────────────────────────
  Input:  [28, 28, 192]
  Filter: 1×1×192, 64 filters
  At position (5, 10):
    y[5,10,k] = Σ_{c=0}^{191} w[c,k] · x[5,10,c] + b[k]
    = 192 multiply-adds per output channel
    × 64 output channels = 12,288 operations at this position
    × 784 positions = 9.6M total operations
  Output: [28, 28, 64]


Branch 2: 1×1 → 3×3 Convolution
────────────────────────────────
  Step 2a: 1×1 conv, 192→96 (reduce)
    [28, 28, 192] → [28, 28, 96]
    Reduces channel dim from 192 to 96

  Step 2b: 3×3 conv, 96→128 (spatial processing)
    [28, 28, 96] → [28, 28, 128]
    At position (5, 10):
      y[5,10,k] = Σ_{c=0}^{95} Σ_{p=-1}^{1} Σ_{q=-1}^{1}
                     w[p+1,q+1,c,k] · x[5+p,10+q,c] + b[k]
      = 9 × 96 = 864 multiply-adds per output channel
      × 128 output channels = 110,592 operations at this position
  Output: [28, 28, 128]


Branch 3: 1×1 → 5×5 Convolution
────────────────────────────────
  Step 3a: 1×1 conv, 192→16 (aggressively reduce)
    [28, 28, 192] → [28, 28, 16]

  Step 3b: 5×5 conv, 16→32
    [28, 28, 16] → [28, 28, 32]
    Now the expensive 5×5 conv operates on only 16 channels!
  Output: [28, 28, 32]


Branch 4: MaxPool → 1×1 Convolution
─────────────────────────────────────
  Step 4a: 3×3 max pool, stride 1, pad 1
    [28, 28, 192] → [28, 28, 192]  (same spatial size)

  Step 4b: 1×1 conv, 192→32 (reduce channels)
    [28, 28, 192] → [28, 28, 32]
  Output: [28, 28, 32]


Concatenation:
──────────────
  Branch 1: [28, 28,  64]
  Branch 2: [28, 28, 128]
  Branch 3: [28, 28,  32]
  Branch 4: [28, 28,  32]
  ─────────────────────────
  Concat:   [28, 28, 256]   (64+128+32+32 = 256 channels)
```

---

## PyTorch Implementation

### Inception Module

```python
import torch
import torch.nn as nn
import torch.nn.functional as F


class InceptionModule(nn.Module):
    """
    Single Inception module with 1×1 bottleneck dimensionality reduction.

    Args:
        in_channels: Number of input channels
        ch1x1: Output channels for 1×1 branch
        ch3x3_reduce: Bottleneck channels before 3×3 conv
        ch3x3: Output channels for 3×3 branch
        ch5x5_reduce: Bottleneck channels before 5×5 conv
        ch5x5: Output channels for 5×5 branch
        pool_proj: Output channels for pool branch
    """
    def __init__(self, in_channels, ch1x1, ch3x3_reduce, ch3x3,
                 ch5x5_reduce, ch5x5, pool_proj):
        super(InceptionModule, self).__init__()

        # Branch 1: 1×1 conv
        self.branch1 = nn.Sequential(
            nn.Conv2d(in_channels, ch1x1, kernel_size=1),
            nn.ReLU(inplace=True),
        )

        # Branch 2: 1×1 reduce → 3×3 conv
        self.branch2 = nn.Sequential(
            nn.Conv2d(in_channels, ch3x3_reduce, kernel_size=1),
            nn.ReLU(inplace=True),
            nn.Conv2d(ch3x3_reduce, ch3x3, kernel_size=3, padding=1),
            nn.ReLU(inplace=True),
        )

        # Branch 3: 1×1 reduce → 5×5 conv
        self.branch3 = nn.Sequential(
            nn.Conv2d(in_channels, ch5x5_reduce, kernel_size=1),
            nn.ReLU(inplace=True),
            nn.Conv2d(ch5x5_reduce, ch5x5, kernel_size=5, padding=2),
            nn.ReLU(inplace=True),
        )

        # Branch 4: 3×3 maxpool → 1×1 conv
        self.branch4 = nn.Sequential(
            nn.MaxPool2d(kernel_size=3, stride=1, padding=1),
            nn.Conv2d(in_channels, pool_proj, kernel_size=1),
            nn.ReLU(inplace=True),
        )

    def forward(self, x):
        b1 = self.branch1(x)
        b2 = self.branch2(x)
        b3 = self.branch3(x)
        b4 = self.branch4(x)

        # Concatenate along channel dimension
        return torch.cat([b1, b2, b3, b4], dim=1)
```

### Auxiliary Classifier

```python
class AuxiliaryClassifier(nn.Module):
    """Auxiliary classifier for intermediate supervision."""
    def __init__(self, in_channels, num_classes=1000):
        super(AuxiliaryClassifier, self).__init__()

        self.pool = nn.AvgPool2d(kernel_size=5, stride=3)
        self.conv = nn.Conv2d(in_channels, 128, kernel_size=1)
        self.relu = nn.ReLU(inplace=True)
        self.fc1 = nn.Linear(128 * 4 * 4, 1024)
        self.dropout = nn.Dropout(p=0.7)
        self.fc2 = nn.Linear(1024, num_classes)

    def forward(self, x):
        x = self.pool(x)               # [batch, C, 4, 4]
        x = self.relu(self.conv(x))     # [batch, 128, 4, 4]
        x = x.view(x.size(0), -1)      # [batch, 2048]
        x = self.relu(self.fc1(x))      # [batch, 1024]
        x = self.dropout(x)
        x = self.fc2(x)                # [batch, num_classes]
        return x
```

### Complete GoogLeNet

```python
class GoogLeNet(nn.Module):
    """
    GoogLeNet / Inception v1.

    Input:  224×224×3 RGB image
    Output: 1000 class probabilities
    Params: ~6.8M (inference), ~14.5M (training with aux classifiers)
    """
    def __init__(self, num_classes=1000, use_aux=True):
        super(GoogLeNet, self).__init__()
        self.use_aux = use_aux

        # ─── Stem ───
        self.conv1 = nn.Sequential(
            nn.Conv2d(3, 64, kernel_size=7, stride=2, padding=3),
            nn.ReLU(inplace=True),
            nn.MaxPool2d(3, stride=2, padding=1),
        )
        self.conv2 = nn.Sequential(
            nn.Conv2d(64, 64, kernel_size=1),
            nn.ReLU(inplace=True),
            nn.Conv2d(64, 192, kernel_size=3, padding=1),
            nn.ReLU(inplace=True),
            nn.MaxPool2d(3, stride=2, padding=1),
        )

        # ─── Inception modules ───
        #                    in   1×1  3r  3×3  5r  5×5  pool
        self.inc3a = InceptionModule(192,  64,  96, 128, 16,  32, 32)   # → 256
        self.inc3b = InceptionModule(256, 128, 128, 192, 32,  96, 64)   # → 480

        self.maxpool3 = nn.MaxPool2d(3, stride=2, padding=1)

        self.inc4a = InceptionModule(480, 192,  96, 208, 16,  48, 64)   # → 512
        self.inc4b = InceptionModule(512, 160, 112, 224, 24,  64, 64)   # → 512
        self.inc4c = InceptionModule(512, 128, 128, 256, 24,  64, 64)   # → 512
        self.inc4d = InceptionModule(512, 112, 144, 288, 32,  64, 64)   # → 528
        self.inc4e = InceptionModule(528, 256, 160, 320, 32, 128, 128)  # → 832

        self.maxpool4 = nn.MaxPool2d(3, stride=2, padding=1)

        self.inc5a = InceptionModule(832, 256, 160, 320, 32, 128, 128)  # → 832
        self.inc5b = InceptionModule(832, 384, 192, 384, 48, 128, 128)  # → 1024

        # ─── Auxiliary classifiers ───
        if self.use_aux:
            self.aux1 = AuxiliaryClassifier(512, num_classes)
            self.aux2 = AuxiliaryClassifier(528, num_classes)

        # ─── Final classifier ───
        self.avgpool = nn.AdaptiveAvgPool2d((1, 1))
        self.dropout = nn.Dropout(p=0.4)
        self.fc = nn.Linear(1024, num_classes)

    def forward(self, x):
        # Stem
        x = self.conv1(x)               # [batch, 64, 56, 56]
        x = self.conv2(x)               # [batch, 192, 28, 28]

        # Inception 3
        x = self.inc3a(x)               # [batch, 256, 28, 28]
        x = self.inc3b(x)               # [batch, 480, 28, 28]
        x = self.maxpool3(x)            # [batch, 480, 14, 14]

        # Inception 4
        x = self.inc4a(x)               # [batch, 512, 14, 14]

        aux_out1 = None
        if self.training and self.use_aux:
            aux_out1 = self.aux1(x)

        x = self.inc4b(x)               # [batch, 512, 14, 14]
        x = self.inc4c(x)               # [batch, 512, 14, 14]
        x = self.inc4d(x)               # [batch, 528, 14, 14]

        aux_out2 = None
        if self.training and self.use_aux:
            aux_out2 = self.aux2(x)

        x = self.inc4e(x)               # [batch, 832, 14, 14]
        x = self.maxpool4(x)            # [batch, 832, 7, 7]

        # Inception 5
        x = self.inc5a(x)               # [batch, 832, 7, 7]
        x = self.inc5b(x)               # [batch, 1024, 7, 7]

        # Classifier
        x = self.avgpool(x)             # [batch, 1024, 1, 1]
        x = x.view(x.size(0), -1)       # [batch, 1024]
        x = self.dropout(x)
        x = self.fc(x)                  # [batch, num_classes]

        if self.training and self.use_aux:
            return x, aux_out1, aux_out2
        return x


# ─── Verify ───
model = GoogLeNet(num_classes=1000, use_aux=False)
total = sum(p.numel() for p in model.parameters())
print(f"GoogLeNet Parameters (inference): {total:,}")

x = torch.randn(2, 3, 224, 224)
output = model(x)
print(f"Input:  {x.shape}")
print(f"Output: {output.shape}")

# With auxiliary classifiers
model_aux = GoogLeNet(num_classes=1000, use_aux=True)
model_aux.train()
total_train = sum(p.numel() for p in model_aux.parameters())
print(f"\nGoogLeNet Parameters (training): {total_train:,}")
out_main, aux1, aux2 = model_aux(x)
print(f"Main output: {out_main.shape}")
print(f"Aux1 output: {aux1.shape}")
print(f"Aux2 output: {aux2.shape}")
```

### Training with Auxiliary Losses

```python
import torch.optim as optim

model = GoogLeNet(num_classes=1000, use_aux=True)
criterion = nn.CrossEntropyLoss()
optimizer = optim.SGD(model.parameters(), lr=0.01, momentum=0.9,
                      weight_decay=0.0002)

# Training step
model.train()
images = torch.randn(32, 3, 224, 224)
labels = torch.randint(0, 1000, (32,))

main_out, aux1_out, aux2_out = model(images)

# Combined loss with auxiliary weights
loss_main = criterion(main_out, labels)
loss_aux1 = criterion(aux1_out, labels)
loss_aux2 = criterion(aux2_out, labels)

total_loss = loss_main + 0.3 * loss_aux1 + 0.3 * loss_aux2

optimizer.zero_grad()
total_loss.backward()
optimizer.step()

print(f"Main loss: {loss_main.item():.4f}")
print(f"Aux1 loss: {loss_aux1.item():.4f}")
print(f"Aux2 loss: {loss_aux2.item():.4f}")
print(f"Total loss: {total_loss.item():.4f}")
```

---

## Inception v2, v3, v4

### Inception v2 (2015) — Batch Normalization

```
Key changes from v1:
━━━━━━━━━━━━━━━━━━━

1. Batch Normalization after every conv layer
2. Replace 5×5 convs with two stacked 3×3 convs (VGG insight)
3. 4.8% top-5 error (vs. 6.67% for v1)

  Before:  Conv → ReLU
  After:   Conv → BatchNorm → ReLU
```

### Inception v3 (2015) — Factorized Convolutions

```
Key changes:
━━━━━━━━━━━━

1. Factorize n×n convolutions into 1×n and n×1:
   ┌────────┐            ┌────────┐    ┌────────┐
   │ 7×7    │   →→→→→    │ 1×7    │ →  │ 7×1    │
   │ conv   │            │ conv   │    │ conv   │
   └────────┘            └────────┘    └────────┘
   49 operations          7 + 7 = 14 operations (3.5× fewer)

2. Three types of Inception modules:
   Module A: Standard (at 35×35 resolution)
   Module B: Factorized (at 17×17) — n×n → 1×n + n×1
   Module C: Expanded (at 8×8) — wider branches

3. Label smoothing regularization
4. 3.58% top-5 error

Label Smoothing:
  Instead of hard labels [0, 0, 1, 0, 0]:
  Use soft labels [ε/K, ε/K, 1-ε+ε/K, ε/K, ε/K]
  Where ε = 0.1, K = number of classes
```

### Inception v4 and Inception-ResNet (2016)

```
Inception v4: Simplified and uniform Inception module design
Inception-ResNet: Combines Inception modules with residual connections

  Standard Inception:
    output = concat(branch1, branch2, branch3, branch4)

  Inception-ResNet:
    output = input + concat(branch1, branch2, branch3)
                      ↑ residual connection!

  Results:
    Inception v4:          3.08% top-5 error
    Inception-ResNet-v2:   3.07% top-5 error
```

---

## Applications

### 1. ImageNet Classification
- **ILSVRC-2014 winner**: 6.67% top-5 error
- Proved efficient architectures can beat brute-force scaling

### 2. Efficient Inference
- Only 6.8M parameters → deployable on mobile/edge devices
- Much faster inference than VGG-16 (1.5B FLOPs vs. 15.5B)

### 3. Multi-Scale Feature Processing
- Inception modules naturally capture features at multiple scales
- Useful for tasks where objects appear at different sizes

### 4. Medical Imaging
- Google used Inception for **diabetic retinopathy detection**
- Inception v3 for **skin cancer classification** (matching dermatologists)

### 5. Foundation for Neural Architecture Search
- The idea of "modules" with multiple parallel paths inspired NASNet
- Automated architecture design builds on Inception's modular approach

---

## Summary Table

| Aspect | Details |
|--------|---------|
| **Architecture** | Stem + 9 Inception modules + GAP + FC |
| **Input Size** | 224 × 224 × 3 |
| **Parameters** | ~6.8M (20× fewer than VGG!) |
| **Depth** | 22 layers (with parameters) |
| **Key Module** | Inception (parallel 1×1, 3×3, 5×5, pool) |
| **Bottleneck** | 1×1 conv for dimensionality reduction |
| **Pooling** | Global Average Pooling (no FC layers!) |
| **Aux Classifiers** | 2 (at Inception 4a and 4d, weight 0.3) |
| **Top-5 Error** | 6.67% (ILSVRC-2014 winner) |
| **Key Innovation** | Multi-scale parallel processing + efficiency |
| **Evolution** | v1 → v2 (BN) → v3 (factorized) → v4 (ResNet) |

---

## Revision Questions

### Q1: The Inception Module
**Describe the four branches of an Inception module and explain why each one is needed.**

<details>
<summary>Answer</summary>

1. **1×1 conv branch**: Captures per-pixel cross-channel correlations. Equivalent to a "point-wise" feature transform. Low cost.

2. **1×1 → 3×3 conv branch**: The 1×1 reduces channels (bottleneck), then 3×3 captures medium-range spatial patterns. Balances spatial and channel processing.

3. **1×1 → 5×5 conv branch**: Similar but with larger receptive field for coarser spatial patterns. The aggressive 1×1 bottleneck (e.g., 192→16) keeps cost manageable.

4. **MaxPool → 1×1 branch**: Max pooling preserves the strongest activations; the 1×1 conv reduces channels to a controllable size. Provides a different "view" of the data.

The key insight is that **different features exist at different scales**, and the network shouldn't have to "choose" — it can process all scales in parallel and let training determine the right mixture.
</details>

### Q2: 1×1 Convolution
**Explain what a 1×1 convolution does and why it is essential for making Inception modules computationally feasible.**

<details>
<summary>Answer</summary>

A 1×1 convolution performs a linear transformation across channels at each spatial position independently. It maps C_in channels to C_out channels: y = Wx + b at each (i,j).

In Inception modules, 1×1 convolutions serve as **bottlenecks** that reduce the channel dimension before expensive 3×3 or 5×5 convolutions. Example:

Without bottleneck: 5×5 conv on 192 channels → 2×25×192×32×784 = 241M FLOPs
With bottleneck: 1×1 (192→16) + 5×5 (16→32) → 4.8M + 20.1M = 24.9M FLOPs

That's a **~10× reduction** in computation for the 5×5 branch!
</details>

### Q3: Global Average Pooling
**Why did GoogLeNet use Global Average Pooling instead of fully connected layers? What are the benefits?**

<details>
<summary>Answer</summary>

GAP replaces the expensive FC layers by averaging each feature map into a single value:
- [7×7×1024] → [1×1×1024] (via averaging) → FC → [1000]
- vs. VGG: [7×7×512] → flatten [25,088] → FC → [4096] → FC → [4096] → FC → [1000]

Benefits:
1. **Massive parameter reduction**: ~1M vs. ~124M for VGG's FC layers
2. **Regularization**: No huge weight matrices to overfit
3. **Spatial invariance**: Averages out spatial information, focusing on "what" not "where"
4. **Variable input sizes**: Works with any spatial resolution at test time
</details>

### Q4: Auxiliary Classifiers
**What problem do auxiliary classifiers address, and how are they used during training vs. inference?**

<details>
<summary>Answer</summary>

Auxiliary classifiers address the **vanishing gradient problem** in deep networks by injecting gradient signal at intermediate layers (Inception 4a and 4d).

**During training:**
- Total loss = L_main + 0.3 × L_aux1 + 0.3 × L_aux2
- Extra gradient flows back from auxiliary loss → helps early layers train

**During inference:**
- Auxiliary classifiers are completely removed
- Only the main classifier output is used
- This reduces model size from ~14.5M to ~6.8M parameters

Later research showed they act more as **regularizers** than gradient highways.
</details>

### Q5: Efficiency Comparison
**GoogLeNet has 6.8M parameters while VGG-16 has 138M, yet GoogLeNet achieves lower error. How is this possible?**

<details>
<summary>Answer</summary>

Three architectural innovations explain this:

1. **1×1 bottleneck convolutions**: Compress channels before expensive operations, drastically reducing parameters while preserving information.

2. **Multi-scale parallel processing**: Inception modules process information at 1×1, 3×3, and 5×5 scales simultaneously, capturing richer features than a single filter size.

3. **Global Average Pooling**: Eliminates the massive FC layers that account for ~89% of VGG's parameters (124M out of 138M). GoogLeNet's final classifier is just a 1024→1000 linear layer (~1M params).

These innovations show that **architectural efficiency** (smart design) can beat **brute force scaling** (more parameters).
</details>

### Q6: Inception Evolution
**Trace the key improvements from Inception v1 through v4.**

<details>
<summary>Answer</summary>

| Version | Year | Key Improvement | Top-5 Error |
|---------|------|-----------------|-------------|
| **v1** (GoogLeNet) | 2014 | Inception module, 1×1 bottleneck, GAP | 6.67% |
| **v2** | 2015 | + Batch Normalization after every conv | 4.8% |
| **v3** | 2015 | + Factorized convolutions (n×n → 1×n + n×1), label smoothing, 3 module types | 3.58% |
| **v4** | 2016 | + Cleaner, simplified module design | 3.08% |
| **Inception-ResNet** | 2016 | + Residual connections within Inception modules | 3.07% |

Key theme: Each version improved efficiency and accuracy through better factorization and regularization techniques.
</details>

---

## Key Takeaways

```
╔══════════════════════════════════════════════════════════════════════╗
║  GoogLeNet / Inception Key Takeaways                                ║
║                                                                      ║
║  1. Inception module: parallel 1×1, 3×3, 5×5, pool branches        ║
║  2. 1×1 conv bottleneck: reduce channels before expensive ops       ║
║  3. Only 6.8M params yet beats 138M-param VGG → efficiency wins    ║
║  4. Global Average Pooling replaces FC layers (209M → 1M params)   ║
║  5. Auxiliary classifiers help train deep networks (act as reg.)    ║
║  6. Multi-scale processing captures diverse feature patterns        ║
║  7. Evolved: v1 → v2 (BN) → v3 (factorized) → v4 (residual)      ║
╚══════════════════════════════════════════════════════════════════════╝
```

---

[← Previous: VGGNet](03-vggnet.md) | [Back to Unit Overview](../README.md) | [Next: ResNet →](05-resnet.md)
