# Convolution Arithmetic — Complete Formula Reference

> **Chapter 2.5 · Convolution Operation**
> A comprehensive reference for all convolution output size formulas, parameter
> counts, and special convolution types.

---

## Table of Contents

1. [Standard Convolution Output Size](#1-standard-convolution-output-size)
2. [Transposed Convolution (Deconvolution)](#2-transposed-convolution-deconvolution)
3. [1×1 Convolution (Pointwise)](#3-1×1-convolution-pointwise)
4. [Dilated (Atrous) Convolution](#4-dilated-atrous-convolution)
5. [Depthwise Separable Convolution](#5-depthwise-separable-convolution)
6. [Parameter Count Formulas](#6-parameter-count-formulas)
7. [Receptive Field Calculation](#7-receptive-field-calculation)
8. [Complete Formula Reference Card](#8-complete-formula-reference-card)
9. [Python & PyTorch Implementation](#9-python--pytorch-implementation)
10. [Summary Table](#10-summary-table)
11. [Revision Questions](#11-revision-questions)

---

## 1. Standard Convolution Output Size

### The Master Formula

```
    ┌──────────────────────────────────────────────────────────┐
    │                                                          │
    │       O = ⌊ (I - K + 2P) / S ⌋ + 1                      │
    │                                                          │
    │   I = Input spatial dimension (height or width)          │
    │   K = Kernel size                                        │
    │   P = Padding (on each side)                             │
    │   S = Stride                                             │
    │   O = Output spatial dimension                           │
    │                                                          │
    └──────────────────────────────────────────────────────────┘
```

### All Common Configurations

| Configuration | Formula | Example (I=224) |
|---------------|---------|-----------------|
| No padding, S=1 | `O = I - K + 1` | K=3 → O=222 |
| Same padding, S=1 | `O = I` (P=(K-1)/2) | K=3, P=1 → O=224 |
| No padding, S=2 | `O = ⌊(I-K)/2⌋ + 1` | K=3 → O=111 |
| Same padding, S=2 | `O = ⌈I/2⌉` | O=112 |
| Full padding, S=1 | `O = I + K - 1` (P=K-1) | K=3 → O=226 |

### Worked Examples — Every Combination

```
    I=7, K=3:
    ┌────────┬────────┬─────────────────────┬───────┐
    │ Stride │Padding │ Calculation         │Output │
    ├────────┼────────┼─────────────────────┼───────┤
    │   1    │   0    │ ⌊(7-3+0)/1⌋+1 = 5  │  5×5  │
    │   1    │   1    │ ⌊(7-3+2)/1⌋+1 = 7  │  7×7  │
    │   1    │   2    │ ⌊(7-3+4)/1⌋+1 = 9  │  9×9  │
    │   2    │   0    │ ⌊(7-3+0)/2⌋+1 = 3  │  3×3  │
    │   2    │   1    │ ⌊(7-3+2)/2⌋+1 = 4  │  4×4  │
    │   3    │   0    │ ⌊(7-3+0)/3⌋+1 = 2  │  2×2  │
    └────────┴────────┴─────────────────────┴───────┘
    
    I=8, K=3:
    ┌────────┬────────┬─────────────────────┬───────┐
    │ Stride │Padding │ Calculation         │Output │
    ├────────┼────────┼─────────────────────┼───────┤
    │   1    │   0    │ ⌊(8-3+0)/1⌋+1 = 6  │  6×6  │
    │   1    │   1    │ ⌊(8-3+2)/1⌋+1 = 8  │  8×8  │
    │   2    │   0    │ ⌊(8-3+0)/2⌋+1 = 3  │  3×3  │
    │   2    │   1    │ ⌊(8-3+2)/2⌋+1 = 4  │  4×4  │
    │   3    │   0    │ ⌊(8-3+0)/3⌋+1 = 2  │  2×2  │
    │   3    │   1    │ ⌊(8-3+2)/3⌋+1 = 3  │  3×3  │
    └────────┴────────┴─────────────────────┴───────┘
```

### Real Architecture Output Sizes

```
    ResNet-50 spatial dimensions:
    
    Layer          Config              Input    → Output
    ─────────────────────────────────────────────────────
    conv1          7×7, S=2, P=3      224×224  → 112×112
    max_pool       3×3, S=2, P=1      112×112  → 56×56
    res_block_2    3×3, S=2, P=1      56×56    → 28×28
    res_block_3    3×3, S=2, P=1      28×28    → 14×14
    res_block_4    3×3, S=2, P=1      14×14    → 7×7
    avg_pool       7×7, S=1, P=0      7×7      → 1×1
```

---

## 2. Transposed Convolution (Deconvolution)

Transposed convolution **upsamples** the feature map — it increases spatial
dimensions (the opposite of standard convolution).

### Why "Transposed"?

```
    Standard Convolution:
    Large input ───► Small output    (downsampling)
    
    Transposed Convolution:
    Small input ───► Large output    (upsampling)
    
    Mathematically: if standard conv can be expressed as matrix multiplication
    y = Cx, then transposed conv computes x' = C^T y (transpose of the matrix)
```

### Transposed Convolution Output Formula

```
    ┌──────────────────────────────────────────────────────────┐
    │                                                          │
    │       O = (I - 1) × S - 2P + K + output_padding         │
    │                                                          │
    │   I = Input spatial dimension                            │
    │   K = Kernel size                                        │
    │   P = Padding                                            │
    │   S = Stride                                             │
    │   output_padding = extra pixels added (0 or 1, typically)│
    │                                                          │
    └──────────────────────────────────────────────────────────┘
```

### How Transposed Convolution Works

```
    Input (2×2):              Insert zeros (stride=2):       Convolve with 3×3 kernel:
    ┌───┬───┐                ┌───┬───┬───┬───┬───┐         ┌───┬───┬───┬───┬───┐
    │ a │ b │                │ a │ 0 │ b │ 0 │ 0 │         │   │   │   │   │   │
    ├───┼───┤     ═══►       ├───┼───┼───┼───┼───┤  ═══►   ├───┼───┼───┼───┼───┤
    │ c │ d │                │ 0 │ 0 │ 0 │ 0 │ 0 │         │   │   │   │   │   │
    └───┴───┘                ├───┼───┼───┼───┼───┤         ├───┼───┼───┼───┼───┤
                             │ c │ 0 │ d │ 0 │ 0 │         │   │   │   │   │   │
    2×2 input                ├───┼───┼───┼───┼───┤         ├───┼───┼───┼───┼───┤
                             │ 0 │ 0 │ 0 │ 0 │ 0 │         │   │   │   │   │   │
                             ├───┼───┼───┼───┼───┤         ├───┼───┼───┼───┼───┤
                             │ 0 │ 0 │ 0 │ 0 │ 0 │         │   │   │   │   │   │
                             └───┴───┴───┴───┴───┘         └───┴───┴───┴───┴───┘
                             
                             Insert S-1 zeros                5×5 output!
                             between each input element      (upsampled)
```

### Transposed Convolution Examples

```
    Example 1: K=3, S=2, P=1
    ─────────────────────────
    Input: 2×2
    O = (2-1)×2 - 2×1 + 3 = 2 - 2 + 3 = 3
    Output: 3×3 (1.5× upsampling, but we get exactly 3×3)
    
    Example 2: K=3, S=2, P=1, output_padding=1
    ────────────────────────────────────────────
    Input: 2×2
    O = (2-1)×2 - 2×1 + 3 + 1 = 2 - 2 + 3 + 1 = 4
    Output: 4×4 (2× upsampling)
    
    Example 3: K=4, S=2, P=1
    ─────────────────────────
    Input: 7×7
    O = (7-1)×2 - 2×1 + 4 = 12 - 2 + 4 = 14
    Output: 14×14 (2× upsampling)
    
    Example 4: K=2, S=2, P=0
    ─────────────────────────
    Input: 7×7
    O = (7-1)×2 - 0 + 2 = 12 + 2 = 14
    Output: 14×14 (2× upsampling)
```

### Common Upsampling Configurations

| Input | Target Output | K | S | P | output_pad | Check |
|-------|---------------|---|---|---|------------|-------|
| 7×7 | 14×14 | 4 | 2 | 1 | 0 | (7-1)×2-2+4=14 ✓ |
| 14×14 | 28×28 | 4 | 2 | 1 | 0 | (14-1)×2-2+4=28 ✓ |
| 28×28 | 56×56 | 4 | 2 | 1 | 0 | (28-1)×2-2+4=56 ✓ |
| 7×7 | 14×14 | 2 | 2 | 0 | 0 | (7-1)×2+2=14 ✓ |
| 7×7 | 14×14 | 3 | 2 | 1 | 1 | (7-1)×2-2+3+1=14 ✓ |

### Applications of Transposed Convolution

```
    1. Semantic Segmentation (U-Net, FCN)
       ┌────┐     ┌──────┐     ┌──────────┐
       │7×7 │ ──► │14×14 │ ──► │ 28×28    │  → Pixel-level labels
       └────┘     └──────┘     └──────────┘
       
    2. Image Generation (GANs)
       ┌────┐     ┌──────┐     ┌──────────┐     ┌────────────┐
       │1×1 │ ──► │ 4×4  │ ──► │  16×16   │ ──► │   64×64    │ → Generated image
       └────┘     └──────┘     └──────────┘     └────────────┘
       
    3. Super-Resolution
       ┌──────┐     ┌──────────┐
       │ Low  │ ──► │  High    │  → Upscaled image
       │ Res  │     │  Res     │
       └──────┘     └──────────┘
```

---

## 3. 1×1 Convolution (Pointwise)

A 1×1 convolution uses a kernel of size 1×1. Despite having no spatial extent,
it's surprisingly useful.

### What Does 1×1 Convolution Do?

```
    Input: H × W × C_in              Output: H × W × C_out
    
    ┌──────────┐                     ┌──────────┐
    │          │                     │          │
    │  H × W  │  ──► 1×1 conv ──►   │  H × W  │
    │          │  C_out filters      │          │
    │          │                     │          │
    └──────────┘                     └──────────┘
    C_in channels                    C_out channels
    
    Spatial dimensions UNCHANGED!
    Only the number of channels changes.
```

### Mathematical Operation

At each spatial position (i, j):

```
    Output(i, j, k) = Σ Input(i, j, c) × W(c, k) + b(k)
                      c=1 to C_in
    
    This is just a fully connected layer applied independently
    at every spatial position!
    
    Equivalent to:
    - A dot product across channels at each pixel
    - A linear combination of all input channels
    - Channel-wise "mixing"
```

### Visualization

```
    Input (4×4×3):                    Kernel (1×1×3):      Output (4×4×1):
    
    Channel R:  Channel G:  Channel B:    ┌─────┐
    ┌──────┐   ┌──────┐   ┌──────┐       │w_R  │         ┌──────┐
    │r₀₀r₀₁│   │g₀₀g₀₁│   │b₀₀b₀₁│     │w_G  │   →     │ o₀₀  │
    │r₁₀r₁₁│   │g₁₀g₁₁│   │b₁₀b₁₁│     │w_B  │         │ o₁₀  │
    └──────┘   └──────┘   └──────┘       └─────┘         └──────┘
    
    o₀₀ = r₀₀ × w_R + g₀₀ × w_G + b₀₀ × w_B + bias
    
    Same formula applied at EVERY position independently
```

### Use Cases for 1×1 Convolution

| Use Case | Description | Example Architecture |
|----------|-------------|---------------------|
| **Channel reduction** | Reduce C_in to smaller C_out (bottleneck) | ResNet, Inception |
| **Channel expansion** | Increase C_in to larger C_out | MobileNet |
| **Cross-channel interaction** | Mix information across channels | Network-in-Network |
| **Add non-linearity** | 1×1 conv + ReLU adds expressive power | GoogLeNet |
| **Dimensionality control** | Adjust feature dimensions cheaply | All modern CNNs |

### Bottleneck Example (ResNet)

```
    Without 1×1 conv:
    ┌──────────┐     ┌──────────┐
    │ 256 ch   │ ──► │ 3×3 conv │ ──► 256 ch
    │          │     │ 256→256  │
    └──────────┘     └──────────┘
    Parameters: 3×3×256×256 = 589,824
    
    With 1×1 bottleneck:
    ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐
    │ 256 ch   │→ │ 1×1 conv │→ │ 3×3 conv │→ │ 1×1 conv │→ 256 ch
    │          │  │ 256→64   │  │ 64→64    │  │ 64→256   │
    └──────────┘  └──────────┘  └──────────┘  └──────────┘
    Parameters: 1×1×256×64 + 3×3×64×64 + 1×1×64×256 = 16,384 + 36,864 + 16,384 = 69,632
    
    Savings: 589,824 → 69,632 = 88% reduction!
```

---

## 4. Dilated (Atrous) Convolution

Dilated convolution inserts **gaps** (zeros) between kernel elements, expanding the
effective receptive field without increasing parameters.

### Dilation Rate

```
    Standard (d=1):        Dilated (d=2):          Dilated (d=3):
    
    ┌───┬───┬───┐         ┌───┬───┬───┬───┬───┐   ┌───┬───┬───┬───┬───┬───┬───┐
    │ × │ × │ × │         │ × │   │ × │   │ × │   │ × │   │   │ × │   │   │ × │
    ├───┼───┼───┤         ├───┼───┼───┼───┼───┤   ├───┼───┼───┼───┼───┼───┼───┤
    │ × │ × │ × │         │   │   │   │   │   │   │   │   │   │   │   │   │   │
    ├───┼───┼───┤         ├───┼───┼───┼───┼───┤   ├───┼───┼───┼───┼───┼───┼───┤
    │ × │ × │ × │         │ × │   │ × │   │ × │   │   │   │   │   │   │   │   │
    └───┴───┴───┘         ├───┼───┼───┼───┼───┤   ├───┼───┼───┼───┼───┼───┼───┤
                          │   │   │   │   │   │   │ × │   │   │ × │   │   │ × │
    Receptive: 3×3        ├───┼───┼───┼───┼───┤   ├───┼───┼───┼───┼───┼───┼───┤
    Params: 9             │ × │   │ × │   │ × │   │   │   │   │   │   │   │   │
                          └───┴───┴───┴───┴───┘   ├───┼───┼───┼───┼───┼───┼───┤
                                                   │   │   │   │   │   │   │   │
                          Receptive: 5×5           ├───┼───┼───┼───┼───┼───┼───┤
                          Params: 9 (same!)        │ × │   │   │ × │   │   │ × │
                                                   └───┴───┴───┴───┴───┴───┴───┘
                                                   
                                                   Receptive: 7×7
                                                   Params: 9 (same!)
```

### Effective Kernel Size

```
    K_eff = K + (K - 1)(d - 1)

    Where:
    - K = original kernel size
    - d = dilation rate
    - K_eff = effective kernel size (receptive field)

    Examples:
    K=3, d=1: K_eff = 3 + 2×0 = 3
    K=3, d=2: K_eff = 3 + 2×1 = 5
    K=3, d=3: K_eff = 3 + 2×2 = 7
    K=3, d=4: K_eff = 3 + 2×3 = 9
```

### Dilated Convolution Output Formula

```
    ┌──────────────────────────────────────────────────────────┐
    │                                                          │
    │   O = ⌊ (I - K_eff + 2P) / S ⌋ + 1                      │
    │                                                          │
    │   Where K_eff = K + (K-1)(d-1)                           │
    │                                                          │
    │   Or equivalently:                                       │
    │   O = ⌊ (I - (K-1)·d - 1 + 2P) / S ⌋ + 1               │
    │                                                          │
    └──────────────────────────────────────────────────────────┘
```

### Dilated Convolution Examples

```
    I=15, K=3, S=1:
    ┌──────────┬────────────────────────────────┬───────┐
    │ Dilation │ Calculation                    │Output │
    ├──────────┼────────────────────────────────┼───────┤
    │  d=1     │ K_eff=3;  (15-3+0)/1+1  = 13  │ 13×13 │
    │  d=2     │ K_eff=5;  (15-5+0)/1+1  = 11  │ 11×11 │
    │  d=3     │ K_eff=7;  (15-7+0)/1+1  = 9   │  9×9  │
    │  d=4     │ K_eff=9;  (15-9+0)/1+1  = 7   │  7×7  │
    │  d=6     │ K_eff=13; (15-13+0)/1+1 = 3   │  3×3  │
    │  d=7     │ K_eff=15; (15-15+0)/1+1 = 1   │  1×1  │
    └──────────┴────────────────────────────────┴───────┘
    
    Same parameters (9), but receptive field grows from 3 to 15!
```

### Multi-Scale Feature Extraction with Dilation

```
    Used in DeepLab for semantic segmentation:
    
    ┌────────────┐
    │   Input    │
    │  Feature   │
    │    Map     │
    └─────┬──────┘
          │
    ┌─────┼──────────────────────────┐
    │     │                          │
    ▼     ▼          ▼               ▼
  d=1    d=6        d=12           d=18
  ┌──┐  ┌──┐       ┌──┐           ┌──┐
  │  │  │  │       │  │           │  │
  └──┘  └──┘       └──┘           └──┘
    │     │          │               │
    └─────┴──────────┴───────────────┘
                     │
              ┌──────┴──────┐
              │  Concatenate │  → Multi-scale features
              └─────────────┘
    
    Atrous Spatial Pyramid Pooling (ASPP)
```

---

## 5. Depthwise Separable Convolution

A factorization of standard convolution into two steps that dramatically reduces
computation.

### Standard vs Depthwise Separable

```
    Standard Convolution:
    ┌──────────┐     ┌─────────────┐     ┌──────────┐
    │ H×W×C_in │ ──► │K×K×C_in×C_out│ ──► │H'×W'×C_out│
    └──────────┘     └─────────────┘     └──────────┘
    
    Params: K² × C_in × C_out
    FLOPs:  K² × C_in × C_out × H' × W'
    
    
    Depthwise Separable Convolution:
    
    Step 1: Depthwise (one K×K filter per channel)
    ┌──────────┐     ┌─────────┐     ┌──────────┐
    │ H×W×C_in │ ──► │ K×K×1×C_in │ ──► │H'×W'×C_in│
    └──────────┘     └─────────┘     └──────────┘
    
    Step 2: Pointwise (1×1 conv to mix channels)
    ┌──────────┐     ┌───────────────┐     ┌──────────┐
    │H'×W'×C_in│ ──► │1×1×C_in×C_out│ ──► │H'×W'×C_out│
    └──────────┘     └───────────────┘     └──────────┘
    
    Params: K² × C_in + C_in × C_out
    
    Savings ratio: 1/C_out + 1/K²
    For K=3, C_out=256: saves ~89% of parameters!
```

---

## 6. Parameter Count Formulas

### Complete Parameter Count Reference

```
    ┌──────────────────────────────────────────────────────────────────┐
    │                 PARAMETER COUNT FORMULAS                        │
    │                                                                  │
    │ Standard Conv:                                                   │
    │   Params = K × K × C_in × C_out + C_out (bias)                  │
    │                                                                  │
    │ 1×1 Conv:                                                        │
    │   Params = 1 × 1 × C_in × C_out + C_out = C_in × C_out + C_out │
    │                                                                  │
    │ Depthwise Conv:                                                  │
    │   Params = K × K × C_in + C_in (bias)                           │
    │                                                                  │
    │ Depthwise Separable:                                             │
    │   Params = K × K × C_in + C_in × C_out + 2×C_in (bias)         │
    │                                                                  │
    │ Transposed Conv:                                                 │
    │   Params = K × K × C_in × C_out + C_out (same as standard!)     │
    │                                                                  │
    │ Dilated Conv:                                                    │
    │   Params = K × K × C_in × C_out + C_out (same as standard!)     │
    │                                                                  │
    │ Grouped Conv (groups=G):                                         │
    │   Params = K × K × (C_in/G) × C_out + C_out                     │
    │                                                                  │
    └──────────────────────────────────────────────────────────────────┘
```

### Worked Examples

```
    Example 1: First layer of VGG16
    ────────────────────────────────
    Conv2d(3, 64, kernel_size=3, padding=1)
    Params = 3 × 3 × 3 × 64 + 64 = 1,728 + 64 = 1,792
    
    Example 2: Middle layer of VGG16
    ─────────────────────────────────
    Conv2d(256, 512, kernel_size=3, padding=1)
    Params = 3 × 3 × 256 × 512 + 512 = 1,179,648 + 512 = 1,180,160
    
    Example 3: ResNet bottleneck
    ────────────────────────────
    Conv2d(256, 64, kernel_size=1)    → 1×1×256×64 + 64 = 16,448
    Conv2d(64, 64, kernel_size=3)     → 3×3×64×64 + 64 = 36,928
    Conv2d(64, 256, kernel_size=1)    → 1×1×64×256 + 256 = 16,640
    Total: 16,448 + 36,928 + 16,640 = 70,016
    
    vs Direct: Conv2d(256, 256, kernel_size=3) = 3×3×256×256 + 256 = 590,080
    
    Savings: 590,080 / 70,016 = 8.4× fewer parameters!
    
    Example 4: Depthwise separable (MobileNet)
    ───────────────────────────────────────────
    Standard: Conv2d(128, 256, kernel_size=3)
    Params = 3×3×128×256 + 256 = 295,168
    
    Depthwise: Conv2d(128, 128, kernel_size=3, groups=128) → 3×3×128 + 128 = 1,280
    Pointwise: Conv2d(128, 256, kernel_size=1)              → 1×1×128×256 + 256 = 33,024
    Total: 1,280 + 33,024 = 34,304
    
    Savings: 295,168 / 34,304 = 8.6× fewer parameters!
```

### FLOPs (Floating Point Operations)

```
    Standard Conv FLOPs:
    FLOPs = 2 × K × K × C_in × C_out × H_out × W_out
    
    (Factor of 2: one multiply + one add per element)
    
    Example: Conv2d(64, 128, 3), input 56×56:
    FLOPs = 2 × 3 × 3 × 64 × 128 × 56 × 56 = 924,844,032 ≈ 925M FLOPs
```

---

## 7. Receptive Field Calculation

The **receptive field** is the region of the input that influences a particular
output neuron.

### Formula for Sequential Layers

```
    For L layers with kernel sizes k_1, k_2, ..., k_L and strides s_1, s_2, ..., s_L:
    
    R_L = R_{L-1} + (k_L - 1) × ∏(s_i, i=1 to L-1)
    
    Starting with R_0 = 1 (a single pixel)
    
    Simplified (all strides = 1):
    R = 1 + Σ(k_i - 1) for i=1 to L
    
    Example: Three 3×3 conv layers, all stride 1:
    R = 1 + (3-1) + (3-1) + (3-1) = 1 + 2 + 2 + 2 = 7
    → Equivalent receptive field of one 7×7 kernel!
```

### Receptive Field Table

```
    Layer-by-layer growth (all 3×3 kernels, stride 1):
    
    Layer │ Kernel │ Receptive Field │ Equivalent Single Kernel
    ──────┼────────┼─────────────────┼────────────────────────
      1   │  3×3   │      3×3        │        3×3
      2   │  3×3   │      5×5        │        5×5
      3   │  3×3   │      7×7        │        7×7
      4   │  3×3   │      9×9        │        9×9
      5   │  3×3   │     11×11       │       11×11
    
    With stride-2 pooling every 2 layers:
    
    Layer │ Kernel │ Stride │ RF   │ Notes
    ──────┼────────┼────────┼──────┼──────────
      1   │  3×3   │   1    │  3   │
      2   │  3×3   │   1    │  5   │
    pool  │  2×2   │   2    │  6   │ +1, ×2 for subsequent
      3   │  3×3   │   1    │ 10   │ (3-1)×2 = 4, +6 = 10
      4   │  3×3   │   1    │ 14   │ (3-1)×2 = 4, +10 = 14
    pool  │  2×2   │   2    │ 16   │
      5   │  3×3   │   1    │ 24   │ (3-1)×4 = 8, +16 = 24
```

---

## 8. Complete Formula Reference Card

```
┌──────────────────────────────────────────────────────────────────────────┐
│                    CONVOLUTION ARITHMETIC CHEAT SHEET                    │
├──────────────────────────────────────────────────────────────────────────┤
│                                                                          │
│  STANDARD CONVOLUTION                                                    │
│  ─────────────────────                                                   │
│  Output size:  O = ⌊(I - K + 2P) / S⌋ + 1                              │
│  Parameters:   K² × C_in × C_out + C_out                                │
│  FLOPs:        2 × K² × C_in × C_out × O²                              │
│                                                                          │
│  TRANSPOSED CONVOLUTION                                                  │
│  ──────────────────────                                                  │
│  Output size:  O = (I-1) × S - 2P + K + output_padding                 │
│  Parameters:   K² × C_in × C_out + C_out                                │
│                                                                          │
│  DILATED (ATROUS) CONVOLUTION                                            │
│  ────────────────────────────                                            │
│  Effective K:  K_eff = K + (K-1)(d-1)                                   │
│  Output size:  O = ⌊(I - K_eff + 2P) / S⌋ + 1                          │
│  Parameters:   K² × C_in × C_out + C_out  (same as standard)            │
│                                                                          │
│  1×1 (POINTWISE) CONVOLUTION                                             │
│  ───────────────────────────                                             │
│  Output size:  O = I  (spatial dimensions unchanged)                     │
│  Parameters:   C_in × C_out + C_out                                      │
│                                                                          │
│  DEPTHWISE SEPARABLE                                                     │
│  ───────────────────                                                     │
│  Depthwise:    K² × C_in + C_in                                         │
│  Pointwise:    C_in × C_out + C_out                                      │
│  Total:        K² × C_in + C_in × C_out + C_in + C_out                  │
│                                                                          │
│  GROUPED CONVOLUTION (G groups)                                          │
│  ──────────────────────────────                                          │
│  Parameters:   K² × (C_in/G) × C_out + C_out                            │
│                                                                          │
│  RECEPTIVE FIELD (sequential layers)                                     │
│  ──────────────────────────────────                                      │
│  R_l = R_{l-1} + (K_l - 1) × ∏(S_i, i=1..l-1)                         │
│                                                                          │
│  "SAME" PADDING                                                         │
│  ──────────────                                                          │
│  P = (K - 1) / 2   (for odd K, stride 1)                                │
│                                                                          │
└──────────────────────────────────────────────────────────────────────────┘
```

---

## 9. Python & PyTorch Implementation

### Output Size Calculator (All Types)

```python
import math

def standard_conv_output(I, K, S=1, P=0):
    """Standard convolution output size."""
    return math.floor((I - K + 2*P) / S) + 1

def transposed_conv_output(I, K, S=1, P=0, output_padding=0):
    """Transposed convolution output size."""
    return (I - 1) * S - 2*P + K + output_padding

def dilated_conv_output(I, K, S=1, P=0, d=1):
    """Dilated convolution output size."""
    K_eff = K + (K - 1) * (d - 1)
    return math.floor((I - K_eff + 2*P) / S) + 1

def param_count(K, C_in, C_out, bias=True, groups=1):
    """Parameter count for convolution."""
    params = K * K * (C_in // groups) * C_out
    if bias:
        params += C_out
    return params

# ── Demonstrations ──
print("=== Standard Convolution ===")
for I, K, S, P in [(224, 7, 2, 3), (56, 3, 1, 1), (28, 3, 2, 1)]:
    O = standard_conv_output(I, K, S, P)
    print(f"  I={I:3d}, K={K}, S={S}, P={P} → O={O}")

print("\n=== Transposed Convolution ===")
for I, K, S, P in [(7, 4, 2, 1), (14, 4, 2, 1), (7, 2, 2, 0)]:
    O = transposed_conv_output(I, K, S, P)
    print(f"  I={I:2d}, K={K}, S={S}, P={P} → O={O}")

print("\n=== Dilated Convolution ===")
for d in [1, 2, 3, 4]:
    O = dilated_conv_output(15, 3, 1, 0, d)
    K_eff = 3 + 2*(d-1)
    print(f"  I=15, K=3, d={d} → K_eff={K_eff}, O={O}")

print("\n=== Parameter Counts ===")
configs = [
    ("VGG first",     3, 3, 64,   1),
    ("VGG middle",    3, 256, 512, 1),
    ("ResNet 1x1",    1, 256, 64,  1),
    ("Depthwise 3x3", 3, 128, 128, 128),
]
for name, K, Cin, Cout, G in configs:
    p = param_count(K, Cin, Cout, groups=G)
    print(f"  {name:15s}: K={K}, C_in={Cin:3d}, C_out={Cout:3d}, G={G:3d} → {p:>10,d} params")
```

### PyTorch: All Convolution Types

```python
import torch
import torch.nn as nn

x = torch.randn(1, 64, 28, 28)  # (batch, channels, height, width)

# Standard convolution
conv = nn.Conv2d(64, 128, kernel_size=3, stride=1, padding=1)
print(f"Standard:   {x.shape} → {conv(x).shape}")
print(f"  Params: {sum(p.numel() for p in conv.parameters()):,}")

# Transposed convolution (upsampling)
deconv = nn.ConvTranspose2d(64, 128, kernel_size=4, stride=2, padding=1)
print(f"\nTransposed: {x.shape} → {deconv(x).shape}")
print(f"  Params: {sum(p.numel() for p in deconv.parameters()):,}")

# 1x1 convolution
conv1x1 = nn.Conv2d(64, 32, kernel_size=1)
print(f"\n1×1 Conv:   {x.shape} → {conv1x1(x).shape}")
print(f"  Params: {sum(p.numel() for p in conv1x1.parameters()):,}")

# Dilated convolution
dilated = nn.Conv2d(64, 128, kernel_size=3, padding=2, dilation=2)
print(f"\nDilated:    {x.shape} → {dilated(x).shape}")
print(f"  Params: {sum(p.numel() for p in dilated.parameters()):,}")

# Depthwise separable convolution
dw_conv = nn.Conv2d(64, 64, kernel_size=3, padding=1, groups=64)
pw_conv = nn.Conv2d(64, 128, kernel_size=1)
out = pw_conv(dw_conv(x))
dw_params = sum(p.numel() for p in dw_conv.parameters())
pw_params = sum(p.numel() for p in pw_conv.parameters())
print(f"\nDepthwise Sep: {x.shape} → {out.shape}")
print(f"  Params: {dw_params:,} (DW) + {pw_params:,} (PW) = {dw_params+pw_params:,}")

# Grouped convolution
grouped = nn.Conv2d(64, 128, kernel_size=3, padding=1, groups=4)
print(f"\nGrouped(4): {x.shape} → {grouped(x).shape}")
print(f"  Params: {sum(p.numel() for p in grouped.parameters()):,}")

# Output:
# Standard:   torch.Size([1, 64, 28, 28]) → torch.Size([1, 128, 28, 28])
#   Params: 73,856
# 
# Transposed: torch.Size([1, 64, 28, 28]) → torch.Size([1, 128, 56, 56])
#   Params: 131,200
# 
# 1×1 Conv:   torch.Size([1, 64, 28, 28]) → torch.Size([1, 32, 28, 28])
#   Params: 2,080
# 
# Dilated:    torch.Size([1, 64, 28, 28]) → torch.Size([1, 128, 28, 28])
#   Params: 73,856
# 
# Depthwise Sep: torch.Size([1, 64, 28, 28]) → torch.Size([1, 128, 28, 28])
#   Params: 640 (DW) + 8,320 (PW) = 8,960
#
# Grouped(4): torch.Size([1, 64, 28, 28]) → torch.Size([1, 128, 28, 28])
#   Params: 18,560
```

### Building a Complete Network Architecture

```python
import torch
import torch.nn as nn

class ConvArithmeticDemo(nn.Module):
    """Demonstrates spatial dimension changes through a CNN."""
    
    def __init__(self):
        super().__init__()
        
        # Encoder (downsampling)
        self.enc1 = nn.Conv2d(3, 64, 7, stride=2, padding=3)     # 224→112
        self.enc2 = nn.Conv2d(64, 128, 3, stride=2, padding=1)   # 112→56
        self.enc3 = nn.Conv2d(128, 256, 3, stride=2, padding=1)  # 56→28
        self.enc4 = nn.Conv2d(256, 512, 3, stride=2, padding=1)  # 28→14
        
        # Bottleneck (1x1 for channel reduction)
        self.bottleneck = nn.Conv2d(512, 128, 1)                  # 14→14
        
        # Decoder (upsampling with transposed conv)
        self.dec1 = nn.ConvTranspose2d(128, 64, 4, stride=2, padding=1)  # 14→28
        self.dec2 = nn.ConvTranspose2d(64, 32, 4, stride=2, padding=1)   # 28→56
        self.dec3 = nn.ConvTranspose2d(32, 16, 4, stride=2, padding=1)   # 56→112
        self.dec4 = nn.ConvTranspose2d(16, 3, 4, stride=2, padding=1)    # 112→224
    
    def forward(self, x):
        shapes = [f"Input: {x.shape}"]
        
        x = torch.relu(self.enc1(x)); shapes.append(f"Enc1:  {x.shape}")
        x = torch.relu(self.enc2(x)); shapes.append(f"Enc2:  {x.shape}")
        x = torch.relu(self.enc3(x)); shapes.append(f"Enc3:  {x.shape}")
        x = torch.relu(self.enc4(x)); shapes.append(f"Enc4:  {x.shape}")
        x = torch.relu(self.bottleneck(x)); shapes.append(f"BN:    {x.shape}")
        x = torch.relu(self.dec1(x)); shapes.append(f"Dec1:  {x.shape}")
        x = torch.relu(self.dec2(x)); shapes.append(f"Dec2:  {x.shape}")
        x = torch.relu(self.dec3(x)); shapes.append(f"Dec3:  {x.shape}")
        x = torch.sigmoid(self.dec4(x)); shapes.append(f"Dec4:  {x.shape}")
        
        for s in shapes:
            print(f"  {s}")
        return x

model = ConvArithmeticDemo()
x = torch.randn(1, 3, 224, 224)
out = model(x)

# Output:
#   Input: torch.Size([1, 3, 224, 224])
#   Enc1:  torch.Size([1, 64, 112, 112])
#   Enc2:  torch.Size([1, 128, 56, 56])
#   Enc3:  torch.Size([1, 256, 28, 28])
#   Enc4:  torch.Size([1, 512, 14, 14])
#   BN:    torch.Size([1, 128, 14, 14])
#   Dec1:  torch.Size([1, 64, 28, 28])
#   Dec2:  torch.Size([1, 32, 56, 56])
#   Dec3:  torch.Size([1, 16, 112, 112])
#   Dec4:  torch.Size([1, 3, 224, 224])

total_params = sum(p.numel() for p in model.parameters())
print(f"\nTotal parameters: {total_params:,}")
```

---

## 10. Summary Table

| Convolution Type | Output Formula | Parameters | Key Use |
|-----------------|----------------|------------|---------|
| **Standard** | ⌊(I-K+2P)/S⌋+1 | K²·C_in·C_out+C_out | General feature extraction |
| **Transposed** | (I-1)·S-2P+K+op | K²·C_in·C_out+C_out | Upsampling |
| **1×1 Pointwise** | I (unchanged) | C_in·C_out+C_out | Channel mixing, bottleneck |
| **Dilated** | ⌊(I-K_eff+2P)/S⌋+1 | K²·C_in·C_out+C_out | Large receptive field |
| **Depthwise** | ⌊(I-K+2P)/S⌋+1 | K²·C_in+C_in | Per-channel spatial features |
| **DW Separable** | ⌊(I-K+2P)/S⌋+1 | K²·C_in+C_in·C_out | Efficient mobile networks |
| **Grouped** | ⌊(I-K+2P)/S⌋+1 | K²·(C_in/G)·C_out+C_out | Parallel processing |

---

## 11. Revision Questions

**Q1.** Calculate the output size for a transposed convolution with: Input=7×7,
K=4, S=2, P=1. What is the upsampling factor?

<details>
<summary>Answer</summary>

```
O = (7-1)×2 - 2×1 + 4 = 12 - 2 + 4 = 14
Upsampling factor: 14/7 = 2×
```
</details>

**Q2.** A dilated convolution has K=3, d=4. What is the effective kernel size?
How does this compare to a standard 11×11 kernel in terms of parameters?

<details>
<summary>Answer</summary>

```
K_eff = 3 + (3-1)(4-1) = 3 + 6 = 9

Dilated 3×3 (d=4): 9 parameters per channel pair
Standard 9×9:       81 parameters per channel pair

The dilated convolution has the same receptive field as a 9×9 kernel but
with only 9/81 = 11% of the parameters!
```
</details>

**Q3.** Compare the parameter count of a standard Conv2d(128, 256, 3) vs a
depthwise separable equivalent. Show your calculations.

<details>
<summary>Answer</summary>

```
Standard: 3×3×128×256 + 256 = 294,912 + 256 = 295,168

Depthwise: 3×3×128 + 128 = 1,152 + 128 = 1,280
Pointwise: 1×1×128×256 + 256 = 32,768 + 256 = 33,024
Total DW Sep: 1,280 + 33,024 = 34,304

Ratio: 295,168 / 34,304 ≈ 8.6× fewer parameters
```
</details>

**Q4.** Explain the purpose of 1×1 convolution in the ResNet bottleneck block.
Draw the architecture and compute parameter savings.

<details>
<summary>Answer</summary>

The bottleneck uses 1×1 to reduce channels before the expensive 3×3 conv, then 1×1 to restore.

```
Direct: Conv(256, 256, 3×3) = 3×3×256×256 = 589,824

Bottleneck:
  1×1: 256→64  = 16,384
  3×3: 64→64   = 36,864
  1×1: 64→256  = 16,384
  Total: 69,632

Savings: 8.5× fewer parameters
```
</details>

**Q5.** A network has 5 consecutive 3×3 conv layers with stride=1. What is the
total receptive field? How many parameters would a single equivalent kernel require?

<details>
<summary>Answer</summary>

```
RF = 1 + 5×(3-1) = 1 + 10 = 11

Five 3×3 layers: 5 × 9 = 45 parameters (per channel pair)
One 11×11 kernel: 121 parameters (per channel pair)

Five stacked 3×3 filters are 121/45 = 2.7× more parameter-efficient
and have 5 non-linearities instead of 1.
```
</details>

**Q6.** Design a decoder path that upsamples from 7×7 to 224×224 using transposed
convolutions. Specify K, S, P for each layer.

<details>
<summary>Answer</summary>

```
Layer 1: 7→14   : ConvTranspose2d(K=4, S=2, P=1)  → (7-1)×2-2+4 = 14 ✓
Layer 2: 14→28  : ConvTranspose2d(K=4, S=2, P=1)  → (14-1)×2-2+4 = 28 ✓
Layer 3: 28→56  : ConvTranspose2d(K=4, S=2, P=1)  → (28-1)×2-2+4 = 56 ✓
Layer 4: 56→112 : ConvTranspose2d(K=4, S=2, P=1)  → (56-1)×2-2+4 = 112 ✓
Layer 5: 112→224: ConvTranspose2d(K=4, S=2, P=1)  → (112-1)×2-2+4 = 224 ✓

Five transposed conv layers with K=4, S=2, P=1 each: 7 → 14 → 28 → 56 → 112 → 224
```
</details>

---

## Navigation

| | |
|---|---|
| ⬅️ **Previous** | [Feature Maps](./04-feature-maps.md) |
| ➡️ **Next** | [Multiple Input Channels](./06-multiple-input-channels.md) |
| 🏠 **Home** | [CNN Overview](../README.md) |

---

> *"Mastering convolution arithmetic is like learning the grammar of CNNs —
> once you know the rules, you can design any architecture with confidence."*
