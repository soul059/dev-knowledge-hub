# Kernels & Filters — The Feature Detectors of CNNs

> **Chapter 2.2 · Convolution Operation**
> Understand what kernels are, how common filters work, and how CNNs learn them automatically.

---

## Table of Contents

1. [What Is a Kernel (Filter)?](#1-what-is-a-kernel-filter)
2. [Learnable Filters — The Key Insight](#2-learnable-filters--the-key-insight)
3. [Common Handcrafted Kernels](#3-common-handcrafted-kernels)
4. [Kernel Size Tradeoffs: 3×3 vs 5×5 vs 7×7](#4-kernel-size-tradeoffs-3×3-vs-5×5-vs-7×7)
5. [Filter as Template Matcher](#5-filter-as-template-matcher)
6. [Filter Visualization](#6-filter-visualization)
7. [Handcrafted vs Learned Filters](#7-handcrafted-vs-learned-filters)
8. [Python & PyTorch Code](#8-python--pytorch-code)
9. [Applications](#9-applications)
10. [Summary Table](#10-summary-table)
11. [Revision Questions](#11-revision-questions)

---

## 1. What Is a Kernel (Filter)?

A **kernel** (also called a **filter**) is a small matrix of weights that slides
across the input to detect specific features through the convolution operation.

```
    Kernel / Filter
    ┌─────┬─────┬─────┐
    │ w00 │ w01 │ w02 │     • Small matrix (typically 3×3, 5×5, 7×7)
    ├─────┼─────┼─────┤     • Contains learnable weights
    │ w10 │ w11 │ w12 │     • Shared across ALL spatial positions
    ├─────┼─────┼─────┤     • Each kernel detects ONE type of feature
    │ w20 │ w21 │ w22 │
    └─────┴─────┴─────┘
         9 parameters
```

### Terminology Clarification

| Term | Meaning |
|------|---------|
| **Kernel** | The 2D weight matrix (e.g., 3×3) |
| **Filter** | Often used synonymously with kernel; in multi-channel contexts, a filter is a 3D block (K×K×C_in) |
| **Feature detector** | Functional description — the kernel detects a specific feature |
| **Weight matrix** | Mathematical description — the kernel contains the weights |

---

## 2. Learnable Filters — The Key Insight

### Traditional Computer Vision

```
    Human expert designs kernels manually:
    
    Sobel-X → detects vertical edges
    Sobel-Y → detects horizontal edges
    Gaussian → blurs the image
    Laplacian → detects all edges
    
    ❌ Limited to known features
    ❌ Requires domain expertise
    ❌ Cannot adapt to new problems
```

### Deep Learning Revolution

```
    CNN LEARNS kernels automatically via backpropagation:
    
    1. Initialize kernel weights randomly
    2. Forward pass: apply convolution
    3. Compute loss (e.g., classification error)
    4. Backward pass: compute ∂Loss/∂w for each kernel weight
    5. Update weights: w ← w - α · ∂Loss/∂w
    6. Repeat until convergence
    
    ✅ Discovers optimal features for the task
    ✅ No human design needed
    ✅ Adapts to any domain
```

### Learning Process Visualization

```
    Epoch 0 (Random):          Epoch 50:              Epoch 200 (Converged):
    ┌──────┬──────┬──────┐    ┌──────┬──────┬──────┐  ┌──────┬──────┬──────┐
    │ 0.12 │-0.34 │ 0.56 │    │ 0.65 │ 0.02 │-0.58 │  │ 1.00 │ 0.00 │-1.00 │
    ├──────┼──────┼──────┤    ├──────┼──────┼──────┤  ├──────┼──────┼──────┤
    │-0.78 │ 0.90 │-0.11 │ →  │ 0.78 │-0.03 │-0.72 │ → │ 1.00 │ 0.00 │-1.00 │
    ├──────┼──────┼──────┤    ├──────┼──────┼──────┤  ├──────┼──────┼──────┤
    │ 0.45 │-0.23 │ 0.67 │    │ 0.59 │ 0.01 │-0.61 │  │ 1.00 │ 0.00 │-1.00 │
    └──────┴──────┴──────┘    └──────┴──────┴──────┘  └──────┴──────┴──────┘
                                                       ↑ Learned vertical
                                                         edge detector!
```

---

## 3. Common Handcrafted Kernels

### 3.1 Identity Kernel

Passes the input through unchanged (center pixel only):

```
    ┌───┬───┬───┐           Input pixel value
    │ 0 │ 0 │ 0 │           passes through
    ├───┼───┼───┤           unchanged
    │ 0 │ 1 │ 0 │ ◄─────
    ├───┼───┼───┤
    │ 0 │ 0 │ 0 │
    └───┴───┴───┘
```

### 3.2 Box Blur (Mean Filter)

Averages all neighbors — produces a smooth/blurry output:

```
    ┌─────┬─────┬─────┐
    │ 1/9 │ 1/9 │ 1/9 │     Effect: Each output pixel = average
    ├─────┼─────┼─────┤     of its 3×3 neighborhood
    │ 1/9 │ 1/9 │ 1/9 │     
    ├─────┼─────┼─────┤     Use: Noise reduction, smoothing
    │ 1/9 │ 1/9 │ 1/9 │
    └─────┴─────┴─────┘
    
    Sum of all weights = 1 (preserves brightness)
```

### 3.3 Gaussian Blur

Weighted average — center pixels contribute more than corners:

```
    3×3 Gaussian (σ ≈ 1):           5×5 Gaussian (σ ≈ 1):
    
         1  ┌───┬───┬───┐               1   ┌───┬───┬───┬───┬───┐
        ── · │ 1 │ 2 │ 1 │             ─── · │ 1 │ 4 │  7│  4│  1│
        16  ├───┼───┼───┤             273   ├───┼───┼───┼───┼───┤
            │ 2 │ 4 │ 2 │                   │ 4 │16 │ 26│ 16│  4│
            ├───┼───┼───┤                   ├───┼───┼───┼───┼───┤
            │ 1 │ 2 │ 1 │                   │ 7 │26 │ 41│ 26│  7│
            └───┴───┴───┘                   ├───┼───┼───┼───┼───┤
                                            │ 4 │16 │ 26│ 16│  4│
                                            ├───┼───┼───┼───┼───┤
                                            │ 1 │ 4 │  7│  4│  1│
                                            └───┴───┴───┴───┴───┘
    
    Gaussian formula: G(x,y) = (1 / 2πσ²) · e^(-(x²+y²) / 2σ²)
```

### 3.4 Sharpen Kernel

Enhances edges and fine details by subtracting a blurred version:

```
    ┌────┬────┬────┐
    │  0 │ -1 │  0 │         Sharpened = Original + α(Original - Blurred)
    ├────┼────┼────┤
    │ -1 │  5 │ -1 │         Center weight > 1 amplifies the pixel
    ├────┼────┼────┤         Negative neighbors subtract the blur
    │  0 │ -1 │  0 │
    └────┴────┴────┘
    
    Sum of weights = 5 - 4 = 1 (preserves brightness)
```

### 3.5 Edge Detection Kernels

#### Vertical Edge Detection

```
    ┌────┬────┬────┐         Detects transitions from
    │  1 │  0 │ -1 │         bright-to-dark (or dark-to-bright)
    ├────┼────┼────┤         going LEFT → RIGHT
    │  1 │  0 │ -1 │
    ├────┼────┼────┤         Responds to: │ bright │ dark │
    │  1 │  0 │ -1 │                      ↕ vertical edge
    └────┴────┴────┘
```

#### Horizontal Edge Detection

```
    ┌────┬────┬────┐         Detects transitions from
    │  1 │  1 │  1 │         bright-to-dark (or dark-to-bright)
    ├────┼────┼────┤         going TOP → BOTTOM
    │  0 │  0 │  0 │
    ├────┼────┼────┤         Responds to: ── bright ──
    │ -1 │ -1 │ -1 │                      ── dark ──
    └────┴────┴────┘                      ↔ horizontal edge
```

#### Diagonal Edge Detection

```
    Top-left to Bottom-right:        Top-right to Bottom-left:

    ┌────┬────┬────┐                 ┌────┬────┬────┐
    │  0 │  1 │  2 │                 │  2 │  1 │  0 │
    ├────┼────┼────┤                 ├────┼────┼────┤
    │ -1 │  0 │  1 │                 │  1 │  0 │ -1 │
    ├────┼────┼────┤                 ├────┼────┼────┤
    │ -2 │ -1 │  0 │                 │  0 │ -1 │ -2 │
    └────┴────┴────┘                 └────┴────┴────┘

    Responds to: ╲ diagonal edge     Responds to: ╱ diagonal edge
```

### 3.6 Sobel Filters (Gradient Estimation)

```
    Sobel-X (vertical edges):        Sobel-Y (horizontal edges):

    ┌────┬────┬────┐                 ┌────┬────┬────┐
    │ -1 │  0 │  1 │                 │ -1 │ -2 │ -1 │
    ├────┼────┼────┤                 ├────┼────┼────┤
    │ -2 │  0 │  2 │                 │  0 │  0 │  0 │
    ├────┼────┼────┤                 ├────┼────┼────┤
    │ -1 │  0 │  1 │                 │  1 │  2 │  1 │
    └────┴────┴────┘                 └────┴────┴────┘
    
    Sobel = Smoothing × Derivative
    
    Sobel-X = [1]   × [-1  0  1]   = [-1  0  1]
              [2]                     [-2  0  2]
              [1]                     [-1  0  1]
```

### 3.7 Laplacian (Second Derivative)

```
    4-connected:                     8-connected:

    ┌────┬────┬────┐                 ┌────┬────┬────┐
    │  0 │  1 │  0 │                 │  1 │  1 │  1 │
    ├────┼────┼────┤                 ├────┼────┼────┤
    │  1 │ -4 │  1 │                 │  1 │ -8 │  1 │
    ├────┼────┼────┤                 ├────┼────┼────┤
    │  0 │  1 │  0 │                 │  1 │  1 │  1 │
    └────┴────┴────┘                 └────┴────┴────┘
    
    Detects ALL edges regardless of direction
    Sum of weights = 0 (no response in uniform regions)
```

### 3.8 Emboss Kernel

```
    ┌────┬────┬────┐
    │ -2 │ -1 │  0 │         Creates a 3D embossed effect
    ├────┼────┼────┤         by highlighting edges from
    │ -1 │  1 │  1 │         one direction
    ├────┼────┼────┤
    │  0 │  1 │  2 │
    └────┴────┴────┘
```

### Complete Kernel Reference

| Kernel | Matrix | Sum | Use Case |
|--------|--------|-----|----------|
| Identity | `[[0,0,0],[0,1,0],[0,0,0]]` | 1 | No change |
| Box Blur | `(1/9)·[[1,1,1],[1,1,1],[1,1,1]]` | 1 | Uniform smoothing |
| Gaussian 3×3 | `(1/16)·[[1,2,1],[2,4,2],[1,2,1]]` | 1 | Better smoothing |
| Sharpen | `[[0,-1,0],[-1,5,-1],[0,-1,0]]` | 1 | Detail enhancement |
| Vertical edge | `[[1,0,-1],[1,0,-1],[1,0,-1]]` | 0 | Detect `│` edges |
| Horizontal edge | `[[1,1,1],[0,0,0],[-1,-1,-1]]` | 0 | Detect `──` edges |
| Sobel-X | `[[-1,0,1],[-2,0,2],[-1,0,1]]` | 0 | Gradient in X |
| Sobel-Y | `[[-1,-2,-1],[0,0,0],[1,2,1]]` | 0 | Gradient in Y |
| Laplacian | `[[0,1,0],[1,-4,1],[0,1,0]]` | 0 | All edges |
| Emboss | `[[-2,-1,0],[-1,1,1],[0,1,2]]` | 1 | 3D relief effect |

> **Rule of thumb:** Edge-detection kernels sum to 0; blur/smooth kernels sum to 1.

---

## 4. Kernel Size Tradeoffs: 3×3 vs 5×5 vs 7×7

### Receptive Field Comparison

```
    3×3 kernel:         5×5 kernel:            7×7 kernel:
    ┌───┬───┬───┐      ┌───┬───┬───┬───┬───┐  ┌───┬───┬───┬───┬───┬───┬───┐
    │   │   │   │      │   │   │   │   │   │  │   │   │   │   │   │   │   │
    ├───┼───┼───┤      ├───┼───┼───┼───┼───┤  ├───┼───┼───┼───┼───┼───┼───┤
    │   │ ● │   │      │   │   │   │   │   │  │   │   │   │   │   │   │   │
    ├───┼───┼───┤      ├───┼───┼───┼───┼───┤  ├───┼───┼───┼───┼───┼───┼───┤
    │   │   │   │      │   │   │ ● │   │   │  │   │   │   │   │   │   │   │
    └───┴───┴───┘      ├───┼───┼───┼───┼───┤  ├───┼───┼───┼───┼───┼───┼───┤
                       │   │   │   │   │   │  │   │   │   │ ● │   │   │   │
    9 params           ├───┼───┼───┼───┼───┤  ├───┼───┼───┼───┼───┼───┼───┤
    Sees 3×3 area      │   │   │   │   │   │  │   │   │   │   │   │   │   │
                       └───┴───┴───┴───┴───┘  ├───┼───┼───┼───┼───┼───┼───┤
                                              │   │   │   │   │   │   │   │
                       25 params              ├───┼───┼───┼───┼───┼───┼───┤
                       Sees 5×5 area          │   │   │   │   │   │   │   │
                                              └───┴───┴───┴───┴───┴───┴───┘

                                              49 params
                                              Sees 7×7 area
```

### Key Insight: Two 3×3 ≈ One 5×5

```
    Two stacked 3×3 convolutions:
    
    Layer 1: 3×3 conv → Each output pixel sees 3×3 of input
    Layer 2: 3×3 conv → Each output pixel sees 3×3 of Layer 1
                      → Effectively sees 5×5 of original input!
    
    Parameters: 2 × (3×3) = 18  vs  1 × (5×5) = 25
    
    ✅ Fewer parameters (18 < 25)
    ✅ Two non-linearities (more expressive)
    ✅ Same effective receptive field
```

### Detailed Comparison Table

| Property | 3×3 | 5×5 | 7×7 |
|----------|-----|-----|-----|
| **Parameters** | 9 | 25 | 49 |
| **Receptive field** | 3×3 | 5×5 | 7×7 |
| **Computation** | Low | Medium | High |
| **Detail capture** | Fine | Medium | Coarse |
| **Common use** | Most layers | Some layers | First layer only |
| **Two stacked 3×3 equivalent** | — | ✅ (18 params) | Three 3×3 (27 params) |
| **Modern usage** | ★★★★★ | ★★☆☆☆ | ★☆☆☆☆ |

### Why 3×3 Dominates Modern CNNs (VGGNet Insight)

```
    VGGNet (2014) showed that:
    
    ┌────────────────────────────────────────────────────────┐
    │  Stack of small (3×3) filters > One large filter       │
    │                                                        │
    │  3 layers of 3×3 = receptive field of 7×7              │
    │  Parameters: 3 × 9 = 27  vs  49 for one 7×7           │
    │  Non-linearities: 3 ReLU  vs  1 ReLU                  │
    │                                                        │
    │  Result: Better accuracy with fewer parameters!         │
    └────────────────────────────────────────────────────────┘
```

---

## 5. Filter as Template Matcher

A kernel acts like a **template** — it responds most strongly when the input patch
looks like the kernel pattern.

### Mathematical Intuition

The convolution output is related to the **dot product** between the kernel vector
and the input patch vector. In vector space:

```
    output = kernel · patch = |kernel| × |patch| × cos(θ)
    
    When θ = 0° → cos(θ) = 1  → Maximum response (perfect match)
    When θ = 90° → cos(θ) = 0 → Zero response (orthogonal)
    When θ = 180° → cos(θ) = -1 → Maximum negative response (opposite)
```

### Template Matching Example

```
    Kernel (looking for "┘" corner):    Input Patch A (has corner):
    ┌────┬────┬────┐                    ┌─────┬─────┬─────┐
    │  1 │  1 │  0 │                    │ 200 │ 200 │  10 │
    ├────┼────┼────┤                    ├─────┼─────┼─────┤
    │  1 │  1 │  0 │                    │ 200 │ 200 │  10 │
    ├────┼────┼────┤                    ├─────┼─────┼─────┤
    │  0 │  0 │  0 │                    │  10 │  10 │  10 │
    └────┴────┴────┘                    └─────┴─────┴─────┘
    
    Output = 200+200+0 + 200+200+0 + 0+0+0 = 800  ← STRONG MATCH ✓

    Input Patch B (no corner):
    ┌─────┬─────┬─────┐
    │ 100 │ 100 │ 100 │
    ├─────┼─────┼─────┤
    │ 100 │ 100 │ 100 │
    ├─────┼─────┼─────┤
    │ 100 │ 100 │ 100 │
    └─────┴─────┴─────┘
    
    Output = 100(1+1+0+1+1+0+0+0+0) = 100 × 4 = 400  ← MODERATE

    Input Patch C (opposite corner):
    ┌─────┬─────┬─────┐
    │  10 │  10 │ 200 │
    ├─────┼─────┼─────┤
    │  10 │  10 │ 200 │
    ├─────┼─────┼─────┤
    │ 200 │ 200 │ 200 │
    └─────┴─────┴─────┘
    
    Output = 10+10+0 + 10+10+0 + 0+0+0 = 40  ← WEAK MATCH ✗
```

---

## 6. Filter Visualization

### What Learned Filters Look Like

```
    Layer 1 filters (first convolution layer):
    ┌─────────────────────────────────────────────┐
    │  Learns simple, interpretable patterns:      │
    │                                              │
    │  ───  Horizontal edges                       │
    │  │    Vertical edges                         │
    │  ╱ ╲  Diagonal edges                         │
    │  ●    Blobs / color spots                    │
    │  ○    Opponent colors                        │
    └─────────────────────────────────────────────┘
    
    Layer 2-3 filters:
    ┌─────────────────────────────────────────────┐
    │  Learns combinations of Layer 1 features:    │
    │                                              │
    │  ┘ └  Corners                                │
    │  ⊤ ⊥  T-junctions                           │
    │  ╳    Crosses                                │
    │  ≋    Textures (gratings, grids)             │
    └─────────────────────────────────────────────┘
    
    Deep layer filters:
    ┌─────────────────────────────────────────────┐
    │  Learns high-level, abstract concepts:       │
    │                                              │
    │  👁  Eye detectors                            │
    │  🔵  Wheel detectors                          │
    │  ✿  Flower petal detectors                   │
    │  📝  Text region detectors                    │
    └─────────────────────────────────────────────┘
```

### Feature Hierarchy

```
    Input Image
         │
         ▼
    ┌─────────┐    Layer 1: Edges, colors
    │ Conv 1  │ →  ── │ ╱ ╲ ●
    └────┬────┘
         │
    ┌─────────┐    Layer 2: Corners, textures
    │ Conv 2  │ →  ┘ └ ≋ ⊤ ○
    └────┬────┘
         │
    ┌─────────┐    Layer 3: Parts of objects
    │ Conv 3  │ →  Eye, Wheel, Nose, Window
    └────┬────┘
         │
    ┌─────────┐    Layer 4: Whole objects
    │ Conv 4  │ →  Face, Car, Dog, Building
    └────┬────┘
         │
         ▼
    Classification
```

---

## 7. Handcrafted vs Learned Filters

| Aspect | Handcrafted Filters | Learned Filters (CNN) |
|--------|--------------------|-----------------------|
| **Design** | Human expert designs | Network learns via backpropagation |
| **Flexibility** | Fixed, task-specific | Adapts to any task |
| **Optimality** | Good for known features | Optimal for training data |
| **Interpretability** | Easy to understand | Early layers interpretable, deep layers abstract |
| **Examples** | Sobel, Gaussian, Laplacian | ImageNet-trained CNN filters |
| **Domain transfer** | Must redesign for new domain | Fine-tune for new domain |
| **Historical era** | Pre-2012 computer vision | Post-2012 deep learning |
| **Feature hierarchy** | Single-level features | Multi-level feature hierarchy |

### The Paradigm Shift

```
    BEFORE Deep Learning (pre-2012):
    ┌─────────────┐    ┌──────────────┐    ┌─────────────┐
    │ Hand-design │ →  │ Extract      │ →  │ Train SVM / │
    │ filters     │    │ features     │    │ classifier  │
    └─────────────┘    └──────────────┘    └─────────────┘
         Human              Fixed              Learned
    
    AFTER Deep Learning (post-2012):
    ┌────────────────────────────────────────────────────┐
    │              CNN (End-to-End Learning)              │
    │  Input → [Conv→ReLU→Pool] × N → FC → Output       │
    │                                                    │
    │  Everything is learned from data!                  │
    └────────────────────────────────────────────────────┘
```

---

## 8. Python & PyTorch Code

### Applying Common Kernels with NumPy

```python
import numpy as np
from scipy.signal import convolve2d

# Sample grayscale image (8×8)
image = np.array([
    [100, 100, 100, 100,  50,  50,  50,  50],
    [100, 100, 100, 100,  50,  50,  50,  50],
    [100, 100, 100, 100,  50,  50,  50,  50],
    [100, 100, 100, 100,  50,  50,  50,  50],
    [100, 100, 100, 100,  50,  50,  50,  50],
    [100, 100, 100, 100,  50,  50,  50,  50],
    [100, 100, 100, 100,  50,  50,  50,  50],
    [100, 100, 100, 100,  50,  50,  50,  50],
], dtype=np.float32)

# Define kernels
kernels = {
    'identity':     np.array([[0, 0, 0], [0, 1, 0], [0, 0, 0]]),
    'box_blur':     np.ones((3, 3)) / 9,
    'gaussian':     np.array([[1, 2, 1], [2, 4, 2], [1, 2, 1]]) / 16,
    'sharpen':      np.array([[0, -1, 0], [-1, 5, -1], [0, -1, 0]]),
    'vertical_edge': np.array([[1, 0, -1], [1, 0, -1], [1, 0, -1]]),
    'horizontal_edge': np.array([[1, 1, 1], [0, 0, 0], [-1, -1, -1]]),
    'sobel_x':      np.array([[-1, 0, 1], [-2, 0, 2], [-1, 0, 1]]),
    'laplacian':    np.array([[0, 1, 0], [1, -4, 1], [0, 1, 0]]),
}

# Apply each kernel
for name, kernel in kernels.items():
    result = convolve2d(image, kernel, mode='valid')
    print(f"{name:20s} → max={result.max():.1f}, min={result.min():.1f}")

# Output:
# identity             → max=100.0, min=50.0
# box_blur             → max=100.0, min=50.0
# gaussian             → max=100.0, min=50.0
# sharpen              → max=150.0, min=0.0
# vertical_edge        → max=0.0, min=-150.0
# horizontal_edge      → max=0.0, min=0.0
# sobel_x              → max=0.0, min=-200.0
# laplacian            → max=0.0, min=-100.0
```

### Visualizing Kernels Applied to Real Images (PyTorch)

```python
import torch
import torch.nn.functional as F
import torchvision.transforms as transforms
from PIL import Image
import matplotlib.pyplot as plt

# Load and preprocess image
image = Image.open('sample.jpg').convert('L')  # Grayscale
transform = transforms.ToTensor()
img_tensor = transform(image).unsqueeze(0)  # (1, 1, H, W)

# Define kernels as tensors
kernels = {
    'Vertical Edge': torch.tensor([[[[ 1., 0., -1.],
                                      [ 1., 0., -1.],
                                      [ 1., 0., -1.]]]]),
    'Horizontal Edge': torch.tensor([[[[ 1.,  1.,  1.],
                                        [ 0.,  0.,  0.],
                                        [-1., -1., -1.]]]]),
    'Sharpen': torch.tensor([[[[ 0., -1.,  0.],
                                [-1.,  5., -1.],
                                [ 0., -1.,  0.]]]]),
    'Gaussian Blur': torch.tensor([[[[1., 2., 1.],
                                      [2., 4., 2.],
                                      [1., 2., 1.]]]]) / 16,
    'Laplacian': torch.tensor([[[[ 0.,  1.,  0.],
                                  [ 1., -4.,  1.],
                                  [ 0.,  1.,  0.]]]]),
}

# Apply each kernel and visualize
fig, axes = plt.subplots(2, 3, figsize=(15, 10))
axes = axes.flatten()

axes[0].imshow(img_tensor.squeeze(), cmap='gray')
axes[0].set_title('Original')

for idx, (name, kernel) in enumerate(kernels.items(), 1):
    output = F.conv2d(img_tensor, kernel, padding=1)
    axes[idx].imshow(output.squeeze().detach().numpy(), cmap='gray')
    axes[idx].set_title(name)

for ax in axes:
    ax.axis('off')
plt.tight_layout()
plt.savefig('kernel_effects.png', dpi=150)
plt.show()
```

### Inspecting Learned Filters from a Pretrained CNN

```python
import torch
import torchvision.models as models
import matplotlib.pyplot as plt
import numpy as np

# Load pretrained model
model = models.vgg16(pretrained=True)

# Get first convolutional layer weights
first_layer = model.features[0]  # Conv2d(3, 64, kernel_size=3, padding=1)
weights = first_layer.weight.data.cpu()

print(f"First layer shape: {weights.shape}")
# torch.Size([64, 3, 3, 3]) → 64 filters, each 3×3×3 (RGB)

# Visualize first 16 filters
fig, axes = plt.subplots(4, 4, figsize=(8, 8))
for i, ax in enumerate(axes.flatten()):
    if i < 16:
        # Normalize filter to [0, 1] for display
        filt = weights[i].permute(1, 2, 0).numpy()  # (3, 3, 3) → RGB
        filt = (filt - filt.min()) / (filt.max() - filt.min())
        ax.imshow(filt)
        ax.set_title(f'Filter {i}')
    ax.axis('off')
plt.suptitle('First 16 Learned Filters (VGG16 Layer 1)')
plt.tight_layout()
plt.savefig('learned_filters.png', dpi=150)
plt.show()
```

### Creating a Custom Kernel Layer

```python
import torch
import torch.nn as nn

class CustomKernelConv(nn.Module):
    """Apply a fixed (non-learnable) kernel as a convolution."""
    
    def __init__(self, kernel):
        super().__init__()
        # Register as buffer (not a parameter — won't be updated by optimizer)
        self.register_buffer('weight', 
                             kernel.unsqueeze(0).unsqueeze(0).float())
    
    def forward(self, x):
        return torch.nn.functional.conv2d(x, self.weight, padding=1)

# Example: Sobel-X edge detector as a layer
sobel_x = torch.tensor([[-1, 0, 1],
                         [-2, 0, 2],
                         [-1, 0, 1]], dtype=torch.float32)

edge_detector = CustomKernelConv(sobel_x)

# Apply to a batch of images
dummy_input = torch.randn(4, 1, 28, 28)  # batch of 4 grayscale images
edges = edge_detector(dummy_input)
print(f"Input shape:  {dummy_input.shape}")  # (4, 1, 28, 28)
print(f"Output shape: {edges.shape}")        # (4, 1, 28, 28)
```

---

## 9. Applications

| Application | Kernel Type | Purpose |
|-------------|-------------|---------|
| **Photo enhancement** | Sharpen kernel | Make details crisper |
| **Noise removal** | Gaussian blur | Smooth out noise |
| **Edge detection** | Sobel, Canny | Find object boundaries |
| **Feature extraction** | Learned kernels | Automatic feature discovery |
| **Medical imaging** | Specialized kernels | Detect tumors, lesions |
| **Satellite imagery** | Edge + texture | Land use classification |
| **OCR** | Stroke detectors | Recognize characters |
| **Face detection** | Hierarchical features | Eyes, nose, mouth → face |

### Real-World Impact

```
    Traditional Pipeline:
    Image → [Sobel] → [Gabor] → [SIFT] → [HOG] → Classifier
                    Manual feature engineering
    
    CNN Pipeline:
    Image → [Conv1→ReLU] → [Conv2→ReLU] → ... → Classifier
              Automatic feature learning
    
    Result: CNN achieves 95%+ accuracy where traditional methods plateau at 75%
```

---

## 10. Summary Table

| Concept | Key Takeaway |
|---------|-------------|
| **Kernel** | Small weight matrix that detects a specific feature |
| **Learnable filters** | CNN learns optimal kernels via backpropagation |
| **Blur kernels** | Sum to 1, average neighbors, reduce noise |
| **Edge kernels** | Sum to 0, detect intensity changes |
| **3×3 dominance** | Modern CNNs use stacked 3×3 filters (VGGNet insight) |
| **Template matching** | Kernel responds strongly when patch matches its pattern |
| **Feature hierarchy** | Edges → Textures → Parts → Objects across layers |
| **Weight sharing** | Same kernel applied at every position = efficiency |
| **Handcrafted vs learned** | Learned filters consistently outperform handcrafted |
| **Filter visualization** | First-layer filters are interpretable; deep layers are abstract |

---

## 11. Revision Questions

**Q1.** Why do edge detection kernels always sum to zero? What would happen if the
sum were non-zero?

<details>
<summary>Answer</summary>

Edge kernels sum to zero so they produce zero output in uniform regions (where
all pixels are the same value). If the sum were non-zero, the kernel would produce
a non-zero response even in uniform regions, making it act partly as a brightness
detector rather than a pure edge detector.
</details>

**Q2.** Explain why two stacked 3×3 convolutions are preferred over one 5×5
convolution. State the parameter counts for both.

<details>
<summary>Answer</summary>

Two stacked 3×3 convolutions have the same effective receptive field (5×5) but:
- Fewer parameters: 2 × 9 = 18 vs 25
- More non-linearities: 2 ReLU activations vs 1
- More expressive function approximation

This was a key insight from VGGNet (2014).
</details>

**Q3.** Given the sharpen kernel `[[0,-1,0],[-1,5,-1],[0,-1,0]]`, what is the output
when applied to a completely uniform 3×3 patch where all values are 100?

<details>
<summary>Answer</summary>

```
O = 100(0) + 100(-1) + 100(0) + 100(-1) + 100(5) + 100(-1) + 100(0) + 100(-1) + 100(0)
  = 100 × (0 - 1 + 0 - 1 + 5 - 1 + 0 - 1 + 0)
  = 100 × 1
  = 100
```

The output equals the input value (100) because the kernel weights sum to 1.
In uniform regions, the sharpen kernel acts like identity.
</details>

**Q4.** What is the Gaussian kernel formula? Why is it preferred over box blur?

<details>
<summary>Answer</summary>

Formula: `G(x,y) = (1 / 2πσ²) · e^(-(x²+y²) / 2σ²)`

Gaussian blur is preferred because it gives more weight to nearby pixels and less
to distant ones, producing a more natural-looking blur. Box blur weights all
pixels equally, which can create boxy artifacts.
</details>

**Q5.** A CNN's first convolutional layer has learned 64 filters of size 3×3 applied
to an RGB input. How many total parameters does this layer have (including bias)?

<details>
<summary>Answer</summary>

```
Each filter: 3×3 × 3 channels = 27 weights
64 filters: 64 × 27 = 1,728 weights
Bias: 64 (one per filter)
Total: 1,728 + 64 = 1,792 parameters
```
</details>

**Q6.** Explain the concept of a "feature hierarchy" in CNNs. Give examples of what
each layer level might detect.

<details>
<summary>Answer</summary>

A feature hierarchy is the progressive building of increasingly complex features:

- **Layer 1:** Simple edges and color blobs (horizontal, vertical, diagonal lines)
- **Layer 2:** Combinations of edges → corners, textures, simple shapes
- **Layer 3:** Parts of objects → eyes, wheels, windows
- **Layer 4-5:** Complete objects → faces, cars, dogs

Each layer builds on features detected by the previous layer, creating a hierarchy
from simple to complex.
</details>

---

## Navigation

| | |
|---|---|
| ⬅️ **Previous** | [2D Convolution Intuition](./01-2d-convolution-intuition.md) |
| ➡️ **Next** | [Stride & Padding](./03-stride-and-padding.md) |
| 🏠 **Home** | [CNN Overview](../README.md) |

---

> *"Kernels are the eyes of a CNN — each one looking for a different pattern,
> together forming a complete visual understanding of the input."*
