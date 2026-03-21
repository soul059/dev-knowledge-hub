# Multiple Input Channels — Convolving Over RGB and Beyond

> **Chapter 2.6 · Convolution Operation**
> Learn how convolution extends from single-channel (grayscale) to multi-channel
> (RGB) inputs, with complete worked examples.

---

## Table of Contents

1. [From Grayscale to Color](#1-from-grayscale-to-color)
2. [3D Filter Structure](#2-3d-filter-structure)
3. [The Multi-Channel Convolution Operation](#3-the-multi-channel-convolution-operation)
4. [Step-by-Step Worked Example](#4-step-by-step-worked-example)
5. [General Formula](#5-general-formula)
6. [Visual Summary](#6-visual-summary)
7. [Multiple Input Channels Beyond RGB](#7-multiple-input-channels-beyond-rgb)
8. [Python & PyTorch Implementation](#8-python--pytorch-implementation)
9. [Applications](#9-applications)
10. [Summary Table](#10-summary-table)
11. [Revision Questions](#11-revision-questions)

---

## 1. From Grayscale to Color

### Grayscale: Single Channel

```
    Grayscale image → 1 channel
    
    ┌──────────────────┐
    │                  │
    │   H × W × 1     │    Each pixel: one intensity value (0-255)
    │                  │
    └──────────────────┘
    
    Filter: K × K × 1 (2D matrix)
    
    ┌───┬───┬───┐
    │   │   │   │        9 parameters for a 3×3 filter
    ├───┼───┼───┤
    │   │   │   │
    ├───┼───┼───┤
    │   │   │   │
    └───┴───┴───┘
```

### RGB: Three Channels

```
    RGB image → 3 channels
    
    ┌──────────┐
    │ Red      │ ─┐
    ├──────────┤  │
    │ Green    │ ─┼── H × W × 3    Each pixel: three values (R, G, B)
    ├──────────┤  │
    │ Blue     │ ─┘
    └──────────┘
    
    Filter: K × K × 3 (3D block!)
    
    ┌───┬───┬───┐  ┌───┬───┬───┐  ┌───┬───┬───┐
    │   │   │   │  │   │   │   │  │   │   │   │
    ├───┼───┼───┤  ├───┼───┼───┤  ├───┼───┼───┤
    │   │   │   │  │   │   │   │  │   │   │   │
    ├───┼───┼───┤  ├───┼───┼───┤  ├───┼───┼───┤
    │   │   │   │  │   │   │   │  │   │   │   │
    └───┴───┴───┘  └───┴───┴───┘  └───┴───┴───┘
      Red slice      Green slice     Blue slice
    
    27 parameters for a 3×3×3 filter
```

---

## 2. 3D Filter Structure

### Filter Shape Convention

```
    Filter shape: K × K × C_in
    
    Where:
    - K = kernel spatial size (height = width)
    - C_in = number of input channels
    
    The filter MUST have the same number of channels as the input!
    
    ┌──────────────────────────────────────────┐
    │  Input channels = Filter depth           │
    │                                          │
    │  RGB input (C_in=3) → Filter is K×K×3   │
    │  64-channel input   → Filter is K×K×64  │
    │  512-channel input  → Filter is K×K×512 │
    └──────────────────────────────────────────┘
```

### 3D Filter Visualization

```
    One 3×3×3 filter for RGB input:
    
         Channel 0 (Red)     Channel 1 (Green)    Channel 2 (Blue)
         ┌───┬───┬───┐       ┌───┬───┬───┐       ┌───┬───┬───┐
         │w00│w01│w02│       │w09│w10│w11│       │w18│w19│w20│
         ├───┼───┼───┤       ├───┼───┼───┤       ├───┼───┼───┤
         │w03│w04│w05│       │w12│w13│w14│       │w21│w22│w23│
         ├───┼───┼───┤       ├───┼───┼───┤       ├───┼───┼───┤
         │w06│w07│w08│       │w15│w16│w17│       │w24│w25│w26│
         └───┴───┴───┘       └───┴───┴───┘       └───┴───┴───┘
         
         Total parameters = 3 × 3 × 3 = 27 (+ 1 bias = 28)
```

### Isometric View

```
                    C_in channels
                   ◄──────────►
                ┌───┬───┬───┐
              ╱ │   │   │   │ ╲
            ╱   ├───┼───┼───┤   ╲
          K     │   │   │   │     K
            ╲   ├───┼───┼───┤   ╱
              ╲ │   │   │   │ ╱
                └───┴───┴───┘
                
    One filter = one 3D block of weights
    K × K spatial × C_in depth
```

---

## 3. The Multi-Channel Convolution Operation

### How It Works

The key insight: **convolve each channel separately, then SUM across channels**.

```
    Step 1: Convolve each input channel with the corresponding filter slice
    
    Input Ch.0   ⊛   Filter Slice 0   =   Partial Output 0
    Input Ch.1   ⊛   Filter Slice 1   =   Partial Output 1
    Input Ch.2   ⊛   Filter Slice 2   =   Partial Output 2
    
    Step 2: Sum all partial outputs + bias
    
    Output = Partial 0 + Partial 1 + Partial 2 + bias
    
    Result: ONE 2D feature map (regardless of C_in!)
```

### Diagram

```
    ┌─────────────┐      ┌────────────┐
    │ Input       │      │ Filter     │
    │ H × W × 3  │      │ K×K × 3    │
    │             │      │            │
    │ ┌─────────┐ │      │ ┌────────┐ │
    │ │  Red    │─┼──⊛───┼─│Slice R │─┼──► partial_R
    │ ├─────────┤ │      │ ├────────┤ │
    │ │  Green  │─┼──⊛───┼─│Slice G │─┼──► partial_G    ──► SUM + bias ──► Output
    │ ├─────────┤ │      │ ├────────┤ │                       (2D map)
    │ │  Blue   │─┼──⊛───┼─│Slice B │─┼──► partial_B
    │ └─────────┘ │      │ └────────┘ │
    └─────────────┘      └────────────┘
    
    C_in channel input + 1 filter (K×K×C_in) = 1 output feature map (2D)
```

### Mathematical Formula

```
    Output at position (i, j):
    
                    C_in-1   K-1  K-1
    O(i, j) = b  +   Σ       Σ    Σ    I(c, i+m, j+n) · W(c, m, n)
                    c=0     m=0  n=0
    
    Where:
    - I(c, i+m, j+n) = input value at channel c, position (i+m, j+n)
    - W(c, m, n) = filter weight at channel c, position (m, n)
    - b = bias term
```

---

## 4. Step-by-Step Worked Example

### Setup: 4×4×3 RGB Image with 3×3×3 Filter

```
    Input Image (4×4×3):
    
    Red Channel (4×4):          Green Channel (4×4):        Blue Channel (4×4):
    ┌────┬────┬────┬────┐      ┌────┬────┬────┬────┐      ┌────┬────┬────┬────┐
    │  1 │  2 │  0 │  1 │      │  0 │  1 │  2 │  0 │      │  2 │  0 │  1 │  2 │
    ├────┼────┼────┼────┤      ├────┼────┼────┼────┤      ├────┼────┼────┼────┤
    │  0 │  3 │  1 │  2 │      │  1 │  2 │  0 │  1 │      │  1 │  1 │  0 │  0 │
    ├────┼────┼────┼────┤      ├────┼────┼────┼────┤      ├────┼────┼────┼────┤
    │  2 │  1 │  0 │  0 │      │  0 │  3 │  1 │  2 │      │  0 │  2 │  3 │  1 │
    ├────┼────┼────┼────┤      ├────┼────┼────┼────┤      ├────┼────┼────┼────┤
    │  1 │  0 │  2 │  1 │      │  2 │  1 │  0 │  1 │      │  1 │  0 │  1 │  2 │
    └────┴────┴────┴────┘      └────┴────┴────┴────┘      └────┴────┴────┴────┘

    Filter (3×3×3):
    
    Red Slice:                  Green Slice:                Blue Slice:
    ┌────┬────┬────┐           ┌────┬────┬────┐           ┌────┬────┬────┐
    │  1 │  0 │ -1 │           │  0 │  1 │  0 │           │ -1 │  0 │  1 │
    ├────┼────┼────┤           ├────┼────┼────┤           ├────┼────┼────┤
    │  1 │  0 │ -1 │           │  0 │  1 │  0 │           │ -1 │  0 │  1 │
    ├────┼────┼────┤           ├────┼────┼────┤           ├────┼────┼────┤
    │  1 │  0 │ -1 │           │  0 │  1 │  0 │           │ -1 │  0 │  1 │
    └────┴────┴────┘           └────┴────┴────┘           └────┴────┴────┘

    Bias: b = 1
```

### Position (0, 0): Top-Left

```
    Red patch:     Green patch:    Blue patch:
    ┌───┬───┬───┐  ┌───┬───┬───┐  ┌───┬───┬───┐
    │ 1 │ 2 │ 0 │  │ 0 │ 1 │ 2 │  │ 2 │ 0 │ 1 │
    ├───┼───┼───┤  ├───┼───┼───┤  ├───┼───┼───┤
    │ 0 │ 3 │ 1 │  │ 1 │ 2 │ 0 │  │ 1 │ 1 │ 0 │
    ├───┼───┼───┤  ├───┼───┼───┤  ├───┼───┼───┤
    │ 2 │ 1 │ 0 │  │ 0 │ 3 │ 1 │  │ 0 │ 2 │ 3 │
    └───┴───┴───┘  └───┴───┴───┘  └───┴───┴───┘
    
    Red channel contribution:
        1×1 + 2×0 + 0×(-1) + 0×1 + 3×0 + 1×(-1) + 2×1 + 1×0 + 0×(-1)
      = 1 + 0 + 0 + 0 + 0 + (-1) + 2 + 0 + 0
      = 2
    
    Green channel contribution:
        0×0 + 1×1 + 2×0 + 1×0 + 2×1 + 0×0 + 0×0 + 3×1 + 1×0
      = 0 + 1 + 0 + 0 + 2 + 0 + 0 + 3 + 0
      = 6
    
    Blue channel contribution:
        2×(-1) + 0×0 + 1×1 + 1×(-1) + 1×0 + 0×1 + 0×(-1) + 2×0 + 3×1
      = -2 + 0 + 1 + (-1) + 0 + 0 + 0 + 0 + 3
      = 1
    
    Total at (0,0):
        O(0,0) = Red + Green + Blue + bias
               = 2 + 6 + 1 + 1
               = 10
```

### Position (0, 1): One Step Right

```
    Red patch:     Green patch:    Blue patch:
    ┌───┬───┬───┐  ┌───┬───┬───┐  ┌───┬───┬───┐
    │ 2 │ 0 │ 1 │  │ 1 │ 2 │ 0 │  │ 0 │ 1 │ 2 │
    ├───┼───┼───┤  ├───┼───┼───┤  ├───┼───┼───┤
    │ 3 │ 1 │ 2 │  │ 2 │ 0 │ 1 │  │ 1 │ 0 │ 0 │
    ├───┼───┼───┤  ├───┼───┼───┤  ├───┼───┼───┤
    │ 1 │ 0 │ 0 │  │ 3 │ 1 │ 2 │  │ 2 │ 3 │ 1 │
    └───┴───┴───┘  └───┴───┴───┘  └───┴───┴───┘
    
    Red:   2(1)+0(0)+1(-1) + 3(1)+1(0)+2(-1) + 1(1)+0(0)+0(-1) = 2-1+3-2+1 = 3
    Green: 1(0)+2(1)+0(0) + 2(0)+0(1)+1(0) + 3(0)+1(1)+2(0) = 2+0+1 = 3
    Blue:  0(-1)+1(0)+2(1) + 1(-1)+0(0)+0(1) + 2(-1)+3(0)+1(1) = 2-1-2+1 = 0
    
    O(0,1) = 3 + 3 + 0 + 1 = 7
```

### Position (1, 0): One Step Down

```
    Red patch:     Green patch:    Blue patch:
    ┌───┬───┬───┐  ┌───┬───┬───┐  ┌───┬───┬───┐
    │ 0 │ 3 │ 1 │  │ 1 │ 2 │ 0 │  │ 1 │ 1 │ 0 │
    ├───┼───┼───┤  ├───┼───┼───┤  ├───┼───┼───┤
    │ 2 │ 1 │ 0 │  │ 0 │ 3 │ 1 │  │ 0 │ 2 │ 3 │
    ├───┼───┼───┤  ├───┼───┼───┤  ├───┼───┼───┤
    │ 1 │ 0 │ 2 │  │ 2 │ 1 │ 0 │  │ 1 │ 0 │ 1 │
    └───┴───┴───┘  └───┴───┴───┘  └───┴───┴───┘
    
    Red:   0(1)+3(0)+1(-1) + 2(1)+1(0)+0(-1) + 1(1)+0(0)+2(-1) = -1+2+1-2 = 0
    Green: 1(0)+2(1)+0(0) + 0(0)+3(1)+1(0) + 2(0)+1(1)+0(0) = 2+3+1 = 6
    Blue:  1(-1)+1(0)+0(1) + 0(-1)+2(0)+3(1) + 1(-1)+0(0)+1(1) = -1+3-1+1 = 2
    
    O(1,0) = 0 + 6 + 2 + 1 = 9
```

### Position (1, 1): Bottom-Right

```
    Red patch:     Green patch:    Blue patch:
    ┌───┬───┬───┐  ┌───┬───┬───┐  ┌───┬───┬───┐
    │ 3 │ 1 │ 2 │  │ 2 │ 0 │ 1 │  │ 1 │ 0 │ 0 │
    ├───┼───┼───┤  ├───┼───┼───┤  ├───┼───┼───┤
    │ 1 │ 0 │ 0 │  │ 3 │ 1 │ 2 │  │ 2 │ 3 │ 1 │
    ├───┼───┼───┤  ├───┼───┼───┤  ├───┼───┼───┤
    │ 0 │ 2 │ 1 │  │ 1 │ 0 │ 1 │  │ 0 │ 1 │ 2 │
    └───┴───┴───┘  └───┴───┴───┘  └───┴───┴───┘
    
    Red:   3(1)+1(0)+2(-1) + 1(1)+0(0)+0(-1) + 0(1)+2(0)+1(-1) = 3-2+1-1 = 1
    Green: 2(0)+0(1)+1(0) + 3(0)+1(1)+2(0) + 1(0)+0(1)+1(0) = 0+1+0 = 1
    Blue:  1(-1)+0(0)+0(1) + 2(-1)+3(0)+1(1) + 0(-1)+1(0)+2(1) = -1-2+1+2 = 0
    
    O(1,1) = 1 + 1 + 0 + 1 = 3
```

### Final Output

```
    Input (4×4×3)       Filter (3×3×3)       Output (2×2×1)
    ┌────────────┐      ┌───────────┐        ┌──────┬──────┐
    │ R │ G │ B  │  ⊛   │ R │ G │ B │   =    │  10  │   7  │
    │ 4 │ 4 │ 4  │      │ 3 │ 3 │ 3 │        ├──────┼──────┤
    │ × │ × │ ×  │      │ × │ × │ × │        │   9  │   3  │
    │ 4 │ 4 │ 4  │      │ 3 │ 3 │ 3 │        └──────┴──────┘
    └────────────┘      └───────────┘
                        + bias = 1            ONE 2D feature map
    
    ┌──────────────────────────────────────────────────────────┐
    │  KEY INSIGHT:                                            │
    │                                                          │
    │  3D input (H×W×C_in) + 3D filter (K×K×C_in) = 2D output │
    │                                                          │
    │  The channel dimension is SUMMED AWAY!                   │
    │  One 3D filter always produces ONE 2D feature map.       │
    └──────────────────────────────────────────────────────────┘
```

---

## 5. General Formula

### Dimensions

```
    Input shape:   H × W × C_in
    Filter shape:  K × K × C_in     (C_in MUST match input channels!)
    Output shape:  H' × W' × 1      (always 1 channel per filter)
    
    Where:
    H' = ⌊(H - K + 2P) / S⌋ + 1
    W' = ⌊(W - K + 2P) / S⌋ + 1
```

### Parameters per Filter

```
    Weights = K × K × C_in
    Bias = 1
    Total = K × K × C_in + 1
    
    Examples:
    ┌────────────┬──────────┬─────────────────┬────────┐
    │ Kernel     │ Channels │ Calculation     │ Total  │
    ├────────────┼──────────┼─────────────────┼────────┤
    │ 3×3        │ 1        │ 9×1 + 1         │ 10     │
    │ 3×3        │ 3 (RGB)  │ 9×3 + 1         │ 28     │
    │ 3×3        │ 64       │ 9×64 + 1        │ 577    │
    │ 3×3        │ 256      │ 9×256 + 1       │ 2,305  │
    │ 3×3        │ 512      │ 9×512 + 1       │ 4,609  │
    │ 5×5        │ 3        │ 25×3 + 1        │ 76     │
    │ 7×7        │ 3        │ 49×3 + 1        │ 148    │
    └────────────┴──────────┴─────────────────┴────────┘
```

---

## 6. Visual Summary

### The Complete Multi-Channel Convolution

```
    INPUT (H × W × C_in)                        FILTER (K × K × C_in)
    
    ┌─────────────────┐                          ┌──────────┐
    │                 │                          │          │
    │   Channel 0     │──────── ⊛ ──────────────│ Slice 0  │──┐
    │                 │                          │          │  │
    ├─────────────────┤                          ├──────────┤  │
    │                 │                          │          │  │
    │   Channel 1     │──────── ⊛ ──────────────│ Slice 1  │──┼── Σ + b ──► 2D Feature Map
    │                 │                          │          │  │
    ├─────────────────┤                          ├──────────┤  │
    │                 │                          │          │  │
    │   Channel 2     │──────── ⊛ ──────────────│ Slice 2  │──┘
    │                 │                          │          │
    └─────────────────┘                          └──────────┘
    
    
    Summary:
    ┌──────────────────────────────────────────────────────┐
    │  For each position (i, j):                          │
    │                                                      │
    │  1. Extract K×K patch from EACH channel              │
    │  2. Multiply each patch with corresponding slice     │
    │  3. SUM all products across ALL channels             │
    │  4. Add bias                                         │
    │  5. Result = one scalar value at position (i, j)     │
    └──────────────────────────────────────────────────────┘
```

### Dimensionality Flow

```
    Layer 1:   (224 × 224 × 3)   →  Conv 3×3×3, 64 filters  →  (224 × 224 × 64)
    Layer 2:   (224 × 224 × 64)  →  Conv 3×3×64, 128 filters →  (224 × 224 × 128)
    Layer 3:   (112 × 112 × 128) →  Conv 3×3×128, 256 filters→  (112 × 112 × 256)
    
    ┌──────────────────────────────────────────────────────────────────┐
    │  The C_in of each layer = C_out of the previous layer!          │
    │                                                                  │
    │  Layer N output channels → Layer N+1 input channels             │
    │  Each filter in Layer N+1 has depth = C_out of Layer N          │
    └──────────────────────────────────────────────────────────────────┘
```

---

## 7. Multiple Input Channels Beyond RGB

### Where Multi-Channel Inputs Come From

```
    Source 1: RGB Images (3 channels)
    ┌───┬───┬───┐
    │ R │ G │ B │  → C_in = 3
    └───┴───┴───┘
    
    Source 2: Previous Conv Layer Output (N channels)
    ┌───┬───┬───┬───┬───────┐
    │F1 │F2 │F3 │...│ F64   │  → C_in = 64
    └───┴───┴───┴───┴───────┘
    
    Source 3: Medical Imaging (variable channels)
    ┌────┬────┬────┬────┐
    │ T1 │ T2 │FLAIR│DWI │  → C_in = 4 (MRI modalities)
    └────┴────┴────┴────┘
    
    Source 4: Satellite Imagery (many bands)
    ┌───┬───┬───┬───┬───┬────┬────────┐
    │ R │ G │ B │NIR│SWIR│Thermal│...  │  → C_in = 13 (Sentinel-2)
    └───┴───┴───┴───┴───┴────┴────────┘
    
    Source 5: Video Frames (stacked)
    ┌───┬───┬───┬───┬───┬───┬───┬───┬───┐
    │R₁ │G₁ │B₁ │R₂ │G₂ │B₂ │R₃ │G₃ │B₃ │  → C_in = 9 (3 RGB frames)
    └───┴───┴───┴───┴───┴───┴───┴───┴───┘
```

### Why Channels Are Summed (Not Kept Separate)

```
    Separate channels → detect features per channel only
    
    But real-world features SPAN channels:
    
    "Red car" = high Red + low Green + low Blue + specific edge pattern
    "Blue sky" = low Red + low Green + high Blue + smooth texture
    
    By summing across channels, the filter can learn combinations:
    
    Filter that detects "red edges":
    Red slice:   strong edge detector    [1, 0, -1; 1, 0, -1; 1, 0, -1]
    Green slice: near zero               [0, 0, 0; 0, 0, 0; 0, 0, 0]
    Blue slice:  near zero               [0, 0, 0; 0, 0, 0; 0, 0, 0]
    
    Filter that detects "any color edge":
    Red slice:   edge detector           [1, 0, -1; 1, 0, -1; 1, 0, -1]
    Green slice: edge detector           [1, 0, -1; 1, 0, -1; 1, 0, -1]
    Blue slice:  edge detector           [1, 0, -1; 1, 0, -1; 1, 0, -1]
```

---

## 8. Python & PyTorch Implementation

### NumPy: Multi-Channel Convolution from Scratch

```python
import numpy as np

def multi_channel_conv2d(input_3d, filter_3d, bias=0):
    """
    Multi-channel 2D convolution.
    
    Args:
        input_3d:  shape (C_in, H, W) — input with C_in channels
        filter_3d: shape (C_in, K, K) — filter with C_in slices
        bias:      scalar bias term
    
    Returns:
        output_2d: shape (H', W') — single 2D feature map
    """
    C_in, H, W = input_3d.shape
    _, K, _ = filter_3d.shape
    
    out_h = H - K + 1
    out_w = W - K + 1
    output = np.zeros((out_h, out_w))
    
    for i in range(out_h):
        for j in range(out_w):
            # Sum over all channels
            total = 0
            for c in range(C_in):
                patch = input_3d[c, i:i+K, j:j+K]
                total += np.sum(patch * filter_3d[c])
            output[i, j] = total + bias
    
    return output


# ── Worked Example ──
# 4×4×3 RGB input
input_rgb = np.array([
    # Red channel
    [[1, 2, 0, 1],
     [0, 3, 1, 2],
     [2, 1, 0, 0],
     [1, 0, 2, 1]],
    # Green channel
    [[0, 1, 2, 0],
     [1, 2, 0, 1],
     [0, 3, 1, 2],
     [2, 1, 0, 1]],
    # Blue channel
    [[2, 0, 1, 2],
     [1, 1, 0, 0],
     [0, 2, 3, 1],
     [1, 0, 1, 2]]
], dtype=np.float32)

# 3×3×3 filter
filter_rgb = np.array([
    # Red slice
    [[ 1,  0, -1],
     [ 1,  0, -1],
     [ 1,  0, -1]],
    # Green slice
    [[ 0,  1,  0],
     [ 0,  1,  0],
     [ 0,  1,  0]],
    # Blue slice
    [[-1,  0,  1],
     [-1,  0,  1],
     [-1,  0,  1]]
], dtype=np.float32)

output = multi_channel_conv2d(input_rgb, filter_rgb, bias=1)
print("Output (2×2):")
print(output)
# Output (2×2):
# [[10.  7.]
#  [ 9.  3.]]
```

### PyTorch: Multi-Channel Convolution

```python
import torch
import torch.nn.functional as F

# Create input: (batch=1, channels=3, height=4, width=4)
input_rgb = torch.tensor([
    [[1, 2, 0, 1], [0, 3, 1, 2], [2, 1, 0, 0], [1, 0, 2, 1]],  # R
    [[0, 1, 2, 0], [1, 2, 0, 1], [0, 3, 1, 2], [2, 1, 0, 1]],  # G
    [[2, 0, 1, 2], [1, 1, 0, 0], [0, 2, 3, 1], [1, 0, 1, 2]],  # B
], dtype=torch.float32).unsqueeze(0)  # Add batch dim → (1, 3, 4, 4)

# Create filter: (out_channels=1, in_channels=3, kH=3, kW=3)
filter_rgb = torch.tensor([
    [[ 1,  0, -1], [ 1,  0, -1], [ 1,  0, -1]],  # R slice
    [[ 0,  1,  0], [ 0,  1,  0], [ 0,  1,  0]],  # G slice
    [[-1,  0,  1], [-1,  0,  1], [-1,  0,  1]],  # B slice
], dtype=torch.float32).unsqueeze(0)  # Add out_channel dim → (1, 3, 3, 3)

# Create bias
bias = torch.tensor([1.0])

# Apply convolution
output = F.conv2d(input_rgb, filter_rgb, bias=bias)

print(f"Input shape:  {input_rgb.shape}")    # (1, 3, 4, 4)
print(f"Filter shape: {filter_rgb.shape}")   # (1, 3, 3, 3)
print(f"Output shape: {output.shape}")       # (1, 1, 2, 2)
print(f"Output:\n{output.squeeze()}")
# tensor([[10.,  7.],
#          [ 9.,  3.]])
```

### Using nn.Conv2d for Multi-Channel Input

```python
import torch
import torch.nn as nn

# Conv2d layer: 3 input channels (RGB), 16 output channels, 3×3 kernel
conv = nn.Conv2d(
    in_channels=3,
    out_channels=16,
    kernel_size=3,
    stride=1,
    padding=1,
    bias=True
)

print(f"Filter weight shape: {conv.weight.shape}")
# torch.Size([16, 3, 3, 3])
# 16 filters, each is 3×3×3 (matching 3 input channels)

print(f"Bias shape: {conv.bias.shape}")
# torch.Size([16])
# One bias per output channel

print(f"Parameters: {conv.weight.numel() + conv.bias.numel()}")
# 16 × 3 × 3 × 3 + 16 = 432 + 16 = 448

# Forward pass with RGB image
rgb_image = torch.randn(1, 3, 224, 224)  # (batch, RGB, H, W)
output = conv(rgb_image)
print(f"\nInput:  {rgb_image.shape}")      # (1, 3, 224, 224)
print(f"Output: {output.shape}")           # (1, 16, 224, 224)
# 16 feature maps, each 224×224
```

### Verifying Channel Summation

```python
import torch
import torch.nn as nn

# Simple example to verify multi-channel conv = sum of per-channel convs
input_tensor = torch.randn(1, 3, 5, 5)

conv = nn.Conv2d(3, 1, kernel_size=3, bias=False)

# Get the per-channel results
with torch.no_grad():
    # Full multi-channel convolution
    full_output = conv(input_tensor)
    
    # Manual: convolve each channel separately, then sum
    per_channel_sum = torch.zeros(1, 1, 3, 3)
    for c in range(3):
        single_ch_input = input_tensor[:, c:c+1, :, :]  # (1, 1, 5, 5)
        single_ch_filter = conv.weight[:, c:c+1, :, :]  # (1, 1, 3, 3)
        partial = torch.nn.functional.conv2d(single_ch_input, single_ch_filter)
        per_channel_sum += partial
    
    print("Multi-channel conv output:\n", full_output.squeeze())
    print("\nSum of per-channel convs:\n", per_channel_sum.squeeze())
    print("\nEqual?", torch.allclose(full_output, per_channel_sum, atol=1e-6))
    # Equal? True
```

### Visualizing Per-Channel Filter Responses

```python
import torch
import torchvision.models as models
import matplotlib.pyplot as plt

model = models.resnet18(pretrained=True)
first_conv = model.conv1  # Conv2d(3, 64, kernel_size=7, stride=2, padding=3)

weights = first_conv.weight.data  # Shape: (64, 3, 7, 7)

# Visualize filter 0: its R, G, B slices
fig, axes = plt.subplots(1, 4, figsize=(12, 3))
channel_names = ['Red Slice', 'Green Slice', 'Blue Slice', 'Combined (RGB)']

for c in range(3):
    slice_data = weights[0, c].numpy()  # (7, 7)
    axes[c].imshow(slice_data, cmap='RdBu_r', vmin=-0.5, vmax=0.5)
    axes[c].set_title(channel_names[c])
    axes[c].axis('off')

# Combined RGB view
combined = weights[0].permute(1, 2, 0).numpy()  # (7, 7, 3)
combined = (combined - combined.min()) / (combined.max() - combined.min())
axes[3].imshow(combined)
axes[3].set_title('Combined (RGB)')
axes[3].axis('off')

plt.suptitle('Filter 0 of ResNet-18 (7×7×3)', fontsize=14)
plt.tight_layout()
plt.savefig('multi_channel_filter.png', dpi=150)
plt.show()
```

---

## 9. Applications

| Application | Input Channels | Description |
|-------------|---------------|-------------|
| **RGB image classification** | 3 (R,G,B) | Standard computer vision |
| **RGBA with transparency** | 4 (R,G,B,A) | Image editing, compositing |
| **Medical MRI** | 4+ (T1, T2, FLAIR, DWI) | Brain tumor segmentation |
| **Satellite imagery** | 13 (Sentinel-2 bands) | Land use classification |
| **Depth + RGB** | 4 (R,G,B,D) | Autonomous driving, robotics |
| **Optical flow** | 2 (horizontal, vertical flow) | Video action recognition |
| **Multi-frame video** | 3×N (N stacked RGB frames) | Video understanding |
| **Feature maps from prior layer** | 64, 128, 256, 512 | All hidden CNN layers |

### Why Multi-Channel Matters

```
    Single-channel filter: can only detect spatial patterns
    
    Multi-channel filter:  can detect patterns that SPAN channels
    
    Example: Detecting a "sunset" requires:
    - Red channel: HIGH values at top
    - Blue channel: gradient from high to low
    - Green channel: LOW values
    
    A multi-channel filter naturally captures this:
    Red slice:   [0, 0, 0; 0, 1, 0; 0, 0, 0]   (amplify red)
    Green slice: [0, 0, 0; 0, -1, 0; 0, 0, 0]  (suppress green)
    Blue slice:  [0, 1, 0; 0, 0, 0; 0, -1, 0]  (detect blue gradient)
```

---

## 10. Summary Table

| Concept | Description |
|---------|-------------|
| **Multi-channel input** | Input with C_in channels (e.g., RGB has C_in=3) |
| **3D filter** | Filter shape: K × K × C_in (depth matches input channels) |
| **Channel summation** | Convolve each channel, then sum across channels |
| **Output dimensionality** | One 3D filter → one 2D feature map (channels collapse) |
| **Parameters per filter** | K × K × C_in + 1 (weights + bias) |
| **Channel matching** | Filter C_in MUST equal input C_in |
| **Cross-channel features** | Filter can learn patterns spanning multiple channels |
| **Layer chaining** | Output channels of layer N = input channels of layer N+1 |
| **First layer** | C_in = image channels (1 for grayscale, 3 for RGB) |
| **Hidden layers** | C_in = number of feature maps from previous layer |

---

## 11. Revision Questions

**Q1.** An RGB image (3 channels) is convolved with a 5×5 filter. What is the
shape of this filter? How many parameters does it have (including bias)?

<details>
<summary>Answer</summary>

```
Filter shape: 5 × 5 × 3
Parameters: 5 × 5 × 3 + 1 = 75 + 1 = 76
```
The filter must have the same depth (3) as the input channels.
</details>

**Q2.** Explain why a 3D filter (K×K×C_in) applied to a multi-channel input
produces a 2D output. What happens to the channel dimension?

<details>
<summary>Answer</summary>

At each spatial position, the filter computes element-wise products across ALL
channels simultaneously, then sums everything into a single scalar. The channel
dimension is "reduced" by summation. This is analogous to a dot product between
the filter (flattened to K×K×C_in vector) and the input patch (also flattened),
producing one scalar per position.
</details>

**Q3.** A conv layer has input shape 28×28×64 and uses 3×3 kernels. What is
the shape of each filter? How many parameters per filter?

<details>
<summary>Answer</summary>

```
Each filter shape: 3 × 3 × 64
Parameters per filter: 3 × 3 × 64 + 1 = 576 + 1 = 577
```
The filter depth (64) matches the 64 input channels from the previous layer.
</details>

**Q4.** In the worked example, verify that O(0,1) = 7 by performing the
element-wise multiplication and summation for all three channels.

<details>
<summary>Answer</summary>

```
Red patch:   [2,0,1; 3,1,2; 1,0,0] ⊙ [1,0,-1; 1,0,-1; 1,0,-1]
           = 2+0-1 + 3+0-2 + 1+0+0 = 3

Green patch: [1,2,0; 2,0,1; 3,1,2] ⊙ [0,1,0; 0,1,0; 0,1,0]
           = 0+2+0 + 0+0+0 + 0+1+0 = 3

Blue patch:  [0,1,2; 1,0,0; 2,3,1] ⊙ [-1,0,1; -1,0,1; -1,0,1]
           = 0+0+2 + (-1)+0+0 + (-2)+0+1 = 0

O(0,1) = 3 + 3 + 0 + bias(1) = 7 ✓
```
</details>

**Q5.** A CNN processes satellite images with 13 spectral bands. The first conv
layer has 32 filters of size 3×3. Calculate the total parameters for this layer.

<details>
<summary>Answer</summary>

```
Each filter: 3 × 3 × 13 = 117 weights + 1 bias = 118
Total: 32 × 118 = 3,776 parameters

Or: 3 × 3 × 13 × 32 + 32 = 3,744 + 32 = 3,776
```
</details>

**Q6.** Why can a multi-channel filter detect "red edges" but not a single-channel
filter? Explain with an example.

<details>
<summary>Answer</summary>

A single-channel filter only sees one channel at a time. It can detect edges but
cannot determine if the edge is red, blue, or green.

A multi-channel filter processes all channels simultaneously:
```
Red slice:   [1, 0, -1; 1, 0, -1; 1, 0, -1]  ← edge detector
Green slice: [0, 0, 0; 0, 0, 0; 0, 0, 0]     ← ignores green
Blue slice:  [0, 0, 0; 0, 0, 0; 0, 0, 0]     ← ignores blue
```
This filter responds strongly only when there's an edge AND high red values,
giving zero response to edges in green or blue regions.
</details>

---

## Navigation

| | |
|---|---|
| ⬅️ **Previous** | [Convolution Arithmetic](./05-convolution-arithmetic.md) |
| ➡️ **Next** | [Multiple Output Channels](./07-multiple-output-channels.md) |
| 🏠 **Home** | [CNN Overview](../README.md) |

---

> *"Multi-channel convolution is where CNNs truly shine — learning to combine
> spatial patterns across color channels, spectral bands, or feature maps."*
