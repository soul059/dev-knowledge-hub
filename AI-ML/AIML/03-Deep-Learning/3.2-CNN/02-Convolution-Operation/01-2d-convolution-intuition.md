# 2D Convolution — Intuition & Mechanics

> **Chapter 2.1 · Convolution Operation**
> Master the foundational operation behind every CNN — the 2D convolution.

---

## Table of Contents

1. [What Is 2D Convolution?](#1-what-is-2d-convolution)
2. [The Sliding Window Analogy](#2-the-sliding-window-analogy)
3. [Step-by-Step: Element-wise Multiply & Sum](#3-step-by-step-element-wise-multiply--sum)
4. [ASCII Animation — Kernel Sliding Across Input](#4-ascii-animation--kernel-sliding-across-input)
5. [The Kernel as a Feature Detector](#5-the-kernel-as-a-feature-detector)
6. [Edge Detection with the Sobel Filter](#6-edge-detection-with-the-sobel-filter)
7. [Worked Examples](#7-worked-examples)
8. [Python & PyTorch Implementation](#8-python--pytorch-implementation)
9. [Applications](#9-applications)
10. [Summary Table](#10-summary-table)
11. [Revision Questions](#11-revision-questions)

---

## 1. What Is 2D Convolution?

In image processing and deep learning, **2D convolution** is a mathematical operation
that takes two inputs:

| Component | Symbol | Description |
|-----------|--------|-------------|
| **Input** | `I` | A 2D matrix (e.g., a grayscale image) of size `H × W` |
| **Kernel (Filter)** | `K` | A small 2D matrix of size `k × k` (e.g., 3×3, 5×5) |

The operation produces an **output (feature map)** by sliding the kernel across the
input, computing an element-wise product and summing the results at each position.

### Mathematical Definition

For an input `I` of size `H × W` and kernel `K` of size `k × k`, the output `O` at
position `(i, j)` is:

```
              k-1  k-1
O(i, j)  =   Σ    Σ    I(i + m, j + n) · K(m, n)
             m=0  n=0
```

> **Note:** This is technically *cross-correlation*, not mathematical convolution
> (which flips the kernel). In deep learning, we always use cross-correlation but
> call it "convolution" by convention.

### Convolution vs Cross-Correlation

```
Cross-correlation (used in CNNs):
    O(i,j) = Σ Σ I(i+m, j+n) · K(m, n)

True convolution (flipped kernel):
    O(i,j) = Σ Σ I(i+m, j+n) · K(k-1-m, k-1-n)
```

Since the kernel is **learned**, the distinction doesn't matter — the network simply
learns the flipped version if needed.

---

## 2. The Sliding Window Analogy

Think of convolution as looking at a photograph through a small magnifying glass:

```
┌─────────────────────────────────┐
│                                 │
│   Input Image (H × W)          │
│                                 │
│       ┌───────┐                 │
│       │ Kernel│──► Slide right  │
│       │ (k×k) │                 │
│       └───────┘                 │
│           │                     │
│           ▼ Slide down          │
│                                 │
└─────────────────────────────────┘

At each position → multiply & sum → one output value
```

**Key properties of the sliding window:**

1. **Local receptive field** — The kernel only "sees" a small patch at a time
2. **Weight sharing** — The same kernel weights are used at every position
3. **Translation equivariance** — A shifted input produces a correspondingly shifted output

---

## 3. Step-by-Step: Element-wise Multiply & Sum

Let's walk through one position of the convolution operation.

### Setup

```
Input I (4×4):              Kernel K (3×3):

┌────┬────┬────┬────┐      ┌────┬────┬────┐
│  1 │  2 │  3 │  0 │      │  1 │  0 │ -1 │
├────┼────┼────┼────┤      ├────┼────┼────┤
│  4 │  5 │  6 │  1 │      │  1 │  0 │ -1 │
├────┼────┼────┼────┤      ├────┼────┼────┤
│  7 │  8 │  9 │  2 │      │  1 │  0 │ -1 │
├────┼────┼────┼────┤      └────┴────┴────┘
│  3 │  2 │  1 │  0 │
└────┴────┴────┴────┘
```

### Position (0, 0) — Top-Left Corner

```
Step 1: Overlay the kernel on the top-left 3×3 patch of the input

    Input patch:            Kernel:
    ┌───┬───┬───┐          ┌────┬────┬────┐
    │ 1 │ 2 │ 3 │          │  1 │  0 │ -1 │
    ├───┼───┼───┤          ├────┼────┼────┤
    │ 4 │ 5 │ 6 │          │  1 │  0 │ -1 │
    ├───┼───┼───┤          ├────┼────┼────┤
    │ 7 │ 8 │ 9 │          │  1 │  0 │ -1 │
    └───┴───┴───┘          └────┴────┴────┘

Step 2: Element-wise multiply

    ┌─────────┬─────────┬──────────┐
    │ 1×1 = 1 │ 2×0 = 0 │ 3×(-1)=-3│
    ├─────────┼─────────┼──────────┤
    │ 4×1 = 4 │ 5×0 = 0 │ 6×(-1)=-6│
    ├─────────┼─────────┼──────────┤
    │ 7×1 = 7 │ 8×0 = 0 │ 9×(-1)=-9│
    └─────────┴─────────┴──────────┘

Step 3: Sum all values

    O(0,0) = 1 + 0 + (-3) + 4 + 0 + (-6) + 7 + 0 + (-9)
           = (1 + 4 + 7) + (0 + 0 + 0) + (-3 - 6 - 9)
           = 12 + 0 + (-18)
           = -6
```

### Position (0, 1) — Slide Right by 1

```
    Input patch:            Kernel:
    ┌───┬───┬───┐          ┌────┬────┬────┐
    │ 2 │ 3 │ 0 │          │  1 │  0 │ -1 │
    ├───┼───┼───┤          ├────┼────┼────┤
    │ 5 │ 6 │ 1 │          │  1 │  0 │ -1 │
    ├───┼───┼───┤          ├────┼────┼────┤
    │ 8 │ 9 │ 2 │          │  1 │  0 │ -1 │
    └───┴───┴───┘          └────┴────┴────┘

    O(0,1) = 2×1 + 3×0 + 0×(-1) + 5×1 + 6×0 + 1×(-1) + 8×1 + 9×0 + 2×(-1)
           = 2 + 0 + 0 + 5 + 0 + (-1) + 8 + 0 + (-2)
           = 12
```

---

## 4. ASCII Animation — Kernel Sliding Across Input

Here is the **complete convolution** of a 4×4 input with a 3×3 kernel (stride=1, no padding).

Output size = `(4 - 3 + 1) × (4 - 3 + 1) = 2 × 2`

```
Input I (4×4):                    Kernel K (3×3):
┌────┬────┬────┬────┐            ┌────┬────┬────┐
│  1 │  2 │  3 │  0 │            │  1 │  0 │ -1 │
├────┼────┼────┼────┤            ├────┼────┼────┤
│  4 │  5 │  6 │  1 │            │  1 │  0 │ -1 │
├────┼────┼────┼────┤            ├────┼────┼────┤
│  7 │  8 │  9 │  2 │            │  1 │  0 │ -1 │
├────┼────┼────┼────┤            └────┴────┴────┘
│  3 │  2 │  1 │  0 │
└────┴────┴────┴────┘
```

### Frame 1: Position (0, 0) — Top-Left

```
    ╔════╦════╦════╗────┐
    ║  1 ║  2 ║  3 ║  0 │        O(0,0) = 1(1)+2(0)+3(-1)
    ╠════╬════╬════╣────┤                + 4(1)+5(0)+6(-1)
    ║  4 ║  5 ║  6 ║  1 │                + 7(1)+8(0)+9(-1)
    ╠════╬════╬════╣────┤
    ║  7 ║  8 ║  9 ║  2 │        O(0,0) = 12 - 18 = -6
    ╚════╩════╩════╝────┤
    │  3 │  2 │  1 │  0 │
    └────┴────┴────┴────┘

    Output so far:  ┌─────┬─────┐
                    │  -6 │  ?  │
                    ├─────┼─────┤
                    │  ?  │  ?  │
                    └─────┴─────┘
```

### Frame 2: Position (0, 1) — Top-Right

```
    ┌────╔════╦════╦════╗
    │  1 ║  2 ║  3 ║  0 ║        O(0,1) = 2(1)+3(0)+0(-1)
    ├────╠════╬════╬════╣                + 5(1)+6(0)+1(-1)
    │  4 ║  5 ║  6 ║  1 ║                + 8(1)+9(0)+2(-1)
    ├────╠════╬════╬════╣
    │  7 ║  8 ║  9 ║  2 ║        O(0,1) = 15 - 3 = 12
    ├────╚════╩════╩════╝
    │  3 │  2 │  1 │  0 │
    └────┴────┴────┴────┘

    Output so far:  ┌─────┬─────┐
                    │  -6 │  12 │
                    ├─────┼─────┤
                    │  ?  │  ?  │
                    └─────┴─────┘
```

### Frame 3: Position (1, 0) — Bottom-Left

```
    ┌────┬────┬────┬────┐
    │  1 │  2 │  3 │  0 │
    ╠════╬════╬════╣────┤        O(1,0) = 4(1)+5(0)+6(-1)
    ║  4 ║  5 ║  6 ║  1 │                + 7(1)+8(0)+9(-1)
    ╠════╬════╬════╣────┤                + 3(1)+2(0)+1(-1)
    ║  7 ║  8 ║  9 ║  2 │
    ╠════╬════╬════╣────┤        O(1,0) = 14 - 16 = -2
    ║  3 ║  2 ║  1 ║  0 │
    ╚════╩════╩════╝────┘

    Output so far:  ┌─────┬─────┐
                    │  -6 │  12 │
                    ├─────┼─────┤
                    │  -2 │  ?  │
                    └─────┴─────┘
```

### Frame 4: Position (1, 1) — Bottom-Right

```
    ┌────┬────┬────┬────┐
    │  1 │  2 │  3 │  0 │
    ├────╠════╬════╬════╣        O(1,1) = 5(1)+6(0)+1(-1)
    │  4 ║  5 ║  6 ║  1 ║                + 8(1)+9(0)+2(-1)
    ├────╠════╬════╬════╣                + 2(1)+1(0)+0(-1)
    │  7 ║  8 ║  9 ║  2 ║
    ├────╠════╬════╬════╣        O(1,1) = 15 - 3 = 12
    │  3 ║  2 ║  1 ║  0 ║
    └────╚════╩════╩════╝

    Output so far:  ┌─────┬─────┐
                    │  -6 │  12 │
                    ├─────┼─────┤
                    │  -2 │  12 │
                    └─────┴─────┘
```

### Final Output

```
    Input (4×4)          Kernel (3×3)         Output (2×2)
    ┌───┬───┬───┬───┐   ┌────┬────┬────┐    ┌─────┬─────┐
    │ 1 │ 2 │ 3 │ 0 │   │  1 │  0 │ -1 │    │  -6 │  12 │
    ├───┼───┼───┼───┤   ├────┼────┼────┤  → ├─────┼─────┤
    │ 4 │ 5 │ 6 │ 1 │   │  1 │  0 │ -1 │    │  -2 │  12 │
    ├───┼───┼───┼───┤   ├────┼────┼────┤    └─────┴─────┘
    │ 7 │ 8 │ 9 │ 2 │   │  1 │  0 │ -1 │
    ├───┼───┼───┼───┤   └────┴────┴────┘
    │ 3 │ 2 │ 1 │ 0 │
    └───┴───┴───┴───┘
```

**Interpretation:** The vertical-edge-detection kernel produces high-magnitude
outputs where there are strong intensity differences between the left and right
sides of each patch.

---

## 5. The Kernel as a Feature Detector

A kernel acts as a **template matcher** or **feature detector**. Each kernel is
designed (or learned) to respond strongly to a specific pattern.

### How Detection Works

```
Strong response (high output value):
    Input patch matches the kernel pattern
    → large element-wise products → large sum

Weak response (output near zero):
    Input patch is unrelated to the kernel pattern
    → products cancel each other → small sum

Negative response (large negative output):
    Input patch is the "opposite" of the kernel pattern
    → large negative products → large negative sum
```

### Analogy

```
    Kernel = "Question"            Input Patch = "Answer"

    ┌────┬────┬────┐              ┌────┬────┬────┐
    │  1 │  0 │ -1 │              │  9 │  5 │  1 │
    ├────┼────┼────┤    matches?  ├────┼────┼────┤
    │  1 │  0 │ -1 │  ──────────► │  8 │  4 │  0 │
    ├────┼────┼────┤              ├────┼────┼────┤
    │  1 │  0 │ -1 │              │  7 │  3 │  0 │
    └────┴────┴────┘              └────┴────┴────┘

    "Is there a vertical edge       "Yes! Bright on left,
     (bright→dark, left→right)?"     dark on right"
                                     → HIGH OUTPUT VALUE
```

### Common Feature Detectors

| Kernel Purpose | What It Detects | Response |
|----------------|-----------------|----------|
| Vertical edge | Left-right intensity change | High at vertical edges |
| Horizontal edge | Top-bottom intensity change | High at horizontal edges |
| Gaussian blur | Nothing specific (smooths) | Average of neighborhood |
| Sharpen | Fine details | Enhances differences |
| Corner detector | L-shaped patterns | High at corners |

---

## 6. Edge Detection with the Sobel Filter

The **Sobel filter** is a classic edge-detection kernel used in computer vision.

### Sobel Kernels

```
    Horizontal edges (Sobel-Y):       Vertical edges (Sobel-X):

    ┌────┬────┬────┐                  ┌────┬────┬────┐
    │ -1 │ -2 │ -1 │                  │ -1 │  0 │  1 │
    ├────┼────┼────┤                  ├────┼────┼────┤
    │  0 │  0 │  0 │                  │ -2 │  0 │  2 │
    ├────┼────┼────┤                  ├────┼────┼────┤
    │  1 │  2 │  1 │                  │ -1 │  0 │  1 │
    └────┴────┴────┘                  └────┴────┴────┘
```

### Worked Example: Horizontal Edge Detection

```
Input (5×5) — bright top, dark bottom:

    ┌─────┬─────┬─────┬─────┬─────┐
    │ 200 │ 200 │ 200 │ 200 │ 200 │
    ├─────┼─────┼─────┼─────┼─────┤
    │ 200 │ 200 │ 200 │ 200 │ 200 │
    ├─────┼─────┼─────┼─────┼─────┤
    │  50 │  50 │  50 │  50 │  50 │   ← Edge here
    ├─────┼─────┼─────┼─────┼─────┤
    │  50 │  50 │  50 │  50 │  50 │
    ├─────┼─────┼─────┼─────┼─────┤
    │  50 │  50 │  50 │  50 │  50 │
    └─────┴─────┴─────┴─────┴─────┘
```

**At position (1,1)** — kernel straddles the edge:

```
    Patch:               Sobel-Y:
    ┌─────┬─────┬─────┐ ┌────┬────┬────┐
    │ 200 │ 200 │ 200 │ │ -1 │ -2 │ -1 │
    ├─────┼─────┼─────┤ ├────┼────┼────┤
    │ 200 │ 200 │ 200 │ │  0 │  0 │  0 │
    ├─────┼─────┼─────┤ ├────┼────┼────┤
    │  50 │  50 │  50 │ │  1 │  2 │  1 │
    └─────┴─────┴─────┘ └────┴────┴────┘

    O(1,1) = 200(-1) + 200(-2) + 200(-1)
           + 200(0)  + 200(0)  + 200(0)
           + 50(1)   + 50(2)   + 50(1)

           = -200 - 400 - 200 + 0 + 0 + 0 + 50 + 100 + 50
           = -800 + 200
           = -600
```

**At position (0,1)** — kernel entirely in uniform region:

```
    Patch (all 200s):
    ┌─────┬─────┬─────┐
    │ 200 │ 200 │ 200 │
    ├─────┼─────┼─────┤
    │ 200 │ 200 │ 200 │
    ├─────┼─────┼─────┤
    │ 200 │ 200 │ 200 │
    └─────┴─────┴─────┘

    O(0,1) = 200(-1-2-1+0+0+0+1+2+1) = 200 × 0 = 0
```

**Result:** Strong response at edge, zero in uniform regions ✓

### Edge Magnitude

The combined edge strength at each position is:

```
    G = √(Gx² + Gy²)

    where Gx = output of Sobel-X kernel
          Gy = output of Sobel-Y kernel
```

---

## 7. Worked Examples

### Example 1: Identity Kernel

The identity kernel passes the input through unchanged (only the center pixel):

```
    Kernel:                 Input (3×3):
    ┌───┬───┬───┐          ┌───┬───┬───┐
    │ 0 │ 0 │ 0 │          │ 5 │ 3 │ 8 │
    ├───┼───┼───┤          ├───┼───┼───┤
    │ 0 │ 1 │ 0 │          │ 2 │ 7 │ 1 │
    ├───┼───┼───┤          ├───┼───┼───┤
    │ 0 │ 0 │ 0 │          │ 4 │ 6 │ 9 │
    └───┴───┴───┘          └───┴───┴───┘

    Output (1×1):
    O(0,0) = 0(5)+0(3)+0(8)+0(2)+1(7)+0(1)+0(4)+0(6)+0(9) = 7
```

### Example 2: Averaging (Box Blur) Kernel

```
    Kernel (3×3):           Input (4×4):
    ┌──────┬──────┬──────┐  ┌────┬────┬────┬────┐
    │ 1/9  │ 1/9  │ 1/9  │  │ 10 │ 20 │ 30 │ 40 │
    ├──────┼──────┼──────┤  ├────┼────┼────┼────┤
    │ 1/9  │ 1/9  │ 1/9  │  │ 50 │ 60 │ 70 │ 80 │
    ├──────┼──────┼──────┤  ├────┼────┼────┼────┤
    │ 1/9  │ 1/9  │ 1/9  │  │ 90 │ 10 │ 20 │ 30 │
    └──────┴──────┴──────┘  ├────┼────┼────┼────┤
                            │ 40 │ 50 │ 60 │ 70 │
                            └────┴────┴────┴────┘

    O(0,0) = (1/9)(10+20+30+50+60+70+90+10+20) = 360/9 = 40.0
    O(0,1) = (1/9)(20+30+40+60+70+80+10+20+30) = 360/9 = 40.0
    O(1,0) = (1/9)(50+60+70+90+10+20+40+50+60) = 450/9 = 50.0
    O(1,1) = (1/9)(60+70+80+10+20+30+50+60+70) = 450/9 = 50.0

    Output (2×2):
    ┌──────┬──────┐
    │ 40.0 │ 40.0 │
    ├──────┼──────┤
    │ 50.0 │ 50.0 │
    └──────┴──────┘
```

### Example 3: 5×5 Input with 3×3 Kernel (Full Computation)

```
    Input (5×5):                            Kernel (3×3):
    ┌───┬───┬───┬───┬───┐                  ┌────┬────┬────┐
    │ 1 │ 0 │ 2 │ 1 │ 0 │                  │  0 │  1 │  0 │
    ├───┼───┼───┼───┼───┤                  ├────┼────┼────┤
    │ 0 │ 3 │ 1 │ 0 │ 2 │                  │  1 │ -4 │  1 │
    ├───┼───┼───┼───┼───┤                  ├────┼────┼────┤
    │ 1 │ 2 │ 0 │ 3 │ 1 │                  │  0 │  1 │  0 │
    ├───┼───┼───┼───┼───┤                  └────┴────┴────┘
    │ 0 │ 1 │ 2 │ 1 │ 0 │           (Laplacian — detects all edges)
    ├───┼───┼───┼───┼───┤
    │ 2 │ 0 │ 1 │ 0 │ 1 │
    └───┴───┴───┴───┴───┘

    Output (3×3):
    O(0,0) = 0(0)+1(1)+0(2) + 0(1)+3(-4)+1(1) + 0(1)+2(1)+0(0)
           = 0+1+0+0-12+1+0+2+0 = -8

    O(0,1) = 0(0)+1(0)+2(1) + 3(1)+1(-4)+0(1) + 2(0)+0(1)+3(0)
           = 0+0+2+3-4+0+0+0+0 = 1

    O(0,2) = 0(2)+1(1)+0(0) + 1(1)+0(-4)+2(1) + 0(0)+3(1)+1(0)
           = 0+1+0+1+0+2+0+3+0 = 7

    O(1,0) = 0(0)+3(1)+1(0) + 0(1)+2(-4)+0(1) + 0(0)+1(1)+2(0)
           = 0+3+0+0-8+0+0+1+0 = -4

    O(1,1) = 3(0)+1(1)+0(0) + 2(1)+0(-4)+3(1) + 1(0)+2(1)+1(0)
           = 0+1+0+2+0+3+0+2+0 = 8

    O(1,2) = 1(0)+0(1)+2(0) + 0(1)+3(-4)+1(1) + 2(0)+1(1)+0(0)
           = 0+0+0+0-12+1+0+1+0 = -10

    O(2,0) = 0(0)+2(1)+0(0) + 1(1)+1(-4)+2(1) + 0(2)+0(1)+1(0)
           = 0+2+0+1-4+2+0+0+0 = 1

    O(2,1) = 2(0)+0(1)+3(0) + 1(1)+2(-4)+1(1) + 0(0)+1(1)+0(0)
           = 0+0+0+1-8+1+0+1+0 = -5

    O(2,2) = 0(0)+3(1)+1(0) + 2(1)+1(-4)+0(1) + 1(0)+0(1)+1(0)
           = 0+3+0+2-4+0+0+0+0 = 1

    Final Output:
    ┌─────┬─────┬─────┐
    │  -8 │   1 │   7 │
    ├─────┼─────┼─────┤
    │  -4 │   8 │ -10 │
    ├─────┼─────┼─────┤
    │   1 │  -5 │   1 │
    └─────┴─────┴─────┘
```

---

## 8. Python & PyTorch Implementation

### Pure Python (NumPy) Implementation

```python
import numpy as np

def conv2d_numpy(input_matrix, kernel):
    """
    Perform 2D convolution (cross-correlation) using NumPy.
    
    Args:
        input_matrix: 2D numpy array of shape (H, W)
        kernel: 2D numpy array of shape (k, k)
    
    Returns:
        output: 2D numpy array of shape (H-k+1, W-k+1)
    """
    H, W = input_matrix.shape
    k = kernel.shape[0]
    
    # Output dimensions (valid convolution — no padding)
    out_h = H - k + 1
    out_w = W - k + 1
    
    output = np.zeros((out_h, out_w))
    
    for i in range(out_h):
        for j in range(out_w):
            # Extract the patch
            patch = input_matrix[i:i+k, j:j+k]
            # Element-wise multiply and sum
            output[i, j] = np.sum(patch * kernel)
    
    return output


# ── Example usage ──
input_matrix = np.array([
    [1, 2, 3, 0],
    [4, 5, 6, 1],
    [7, 8, 9, 2],
    [3, 2, 1, 0]
], dtype=np.float32)

kernel = np.array([
    [ 1, 0, -1],
    [ 1, 0, -1],
    [ 1, 0, -1]
], dtype=np.float32)

output = conv2d_numpy(input_matrix, kernel)
print("Output:\n", output)
# Output:
#  [[ -6.  12.]
#   [ -2.  12.]]
```

### PyTorch Implementation

```python
import torch
import torch.nn.functional as F

# Create input tensor — shape: (batch, channels, height, width)
input_tensor = torch.tensor([
    [1, 2, 3, 0],
    [4, 5, 6, 1],
    [7, 8, 9, 2],
    [3, 2, 1, 0]
], dtype=torch.float32).unsqueeze(0).unsqueeze(0)  # (1, 1, 4, 4)

# Create kernel tensor — shape: (out_channels, in_channels, kH, kW)
kernel = torch.tensor([
    [ 1, 0, -1],
    [ 1, 0, -1],
    [ 1, 0, -1]
], dtype=torch.float32).unsqueeze(0).unsqueeze(0)  # (1, 1, 3, 3)

# Apply convolution
output = F.conv2d(input_tensor, kernel, padding=0, stride=1)
print(f"Input shape:  {input_tensor.shape}")   # torch.Size([1, 1, 4, 4])
print(f"Kernel shape: {kernel.shape}")          # torch.Size([1, 1, 3, 3])
print(f"Output shape: {output.shape}")          # torch.Size([1, 1, 2, 2])
print(f"Output:\n{output.squeeze()}")
# tensor([[ -6.,  12.],
#         [ -2.,  12.]])
```

### Using nn.Conv2d (Learnable Convolution Layer)

```python
import torch
import torch.nn as nn

# Define a learnable convolution layer
conv_layer = nn.Conv2d(
    in_channels=1,
    out_channels=1,
    kernel_size=3,
    stride=1,
    padding=0,
    bias=False
)

# Manually set the weights to our kernel
with torch.no_grad():
    conv_layer.weight.copy_(
        torch.tensor([[[[ 1, 0, -1],
                         [ 1, 0, -1],
                         [ 1, 0, -1]]]], dtype=torch.float32)
    )

# Forward pass
input_tensor = torch.tensor([
    [1, 2, 3, 0],
    [4, 5, 6, 1],
    [7, 8, 9, 2],
    [3, 2, 1, 0]
], dtype=torch.float32).reshape(1, 1, 4, 4)

output = conv_layer(input_tensor)
print(output)
# tensor([[[[ -6.,  12.],
#           [ -2.,  12.]]]])
```

### Sobel Edge Detection with OpenCV

```python
import cv2
import numpy as np

# Load image in grayscale
image = cv2.imread('photo.jpg', cv2.IMREAD_GRAYSCALE)

# Sobel filters
sobel_x = cv2.Sobel(image, cv2.CV_64F, 1, 0, ksize=3)  # Vertical edges
sobel_y = cv2.Sobel(image, cv2.CV_64F, 0, 1, ksize=3)  # Horizontal edges

# Combined edge magnitude
edges = np.sqrt(sobel_x**2 + sobel_y**2)
edges = np.uint8(np.clip(edges, 0, 255))

cv2.imwrite('edges.jpg', edges)
```

---

## 9. Applications

| Domain | Application | How Convolution Helps |
|--------|-------------|----------------------|
| **Computer Vision** | Image classification | Detects visual features (edges, textures, shapes) |
| **Medical Imaging** | Tumor detection in MRI/CT scans | Identifies abnormal tissue patterns |
| **Autonomous Driving** | Lane detection, object recognition | Detects road markings, pedestrians, vehicles |
| **NLP** | Text classification with 1D convolution | Extracts n-gram features from text sequences |
| **Audio Processing** | Speech recognition | Detects spectral patterns in spectrograms |
| **Video Analysis** | Action recognition (3D convolution) | Captures spatial + temporal features |
| **Astronomy** | Galaxy classification | Identifies galactic morphologies in telescope images |

### Why Convolution Beats Fully Connected Layers for Images

```
Fully Connected Layer (for 224×224 image):
    Parameters = 224 × 224 × 224 × 224 = 2.5 BILLION
    ❌ Massive memory, easy overfitting, no spatial awareness

Convolutional Layer (64 filters, 3×3):
    Parameters = 64 × 3 × 3 × 1 = 576
    ✅ Parameter sharing, spatial structure preserved, translation equivariant
```

---

## 10. Summary Table

| Concept | Description |
|---------|-------------|
| **2D Convolution** | Sliding a kernel across an input, multiply & sum at each position |
| **Kernel / Filter** | Small weight matrix (e.g., 3×3) that detects a specific feature |
| **Feature Map** | Output of convolution — a map showing where the feature was detected |
| **Output Size** | `O = (I - K) + 1` for valid convolution (no padding, stride=1) |
| **Cross-Correlation** | What CNNs actually compute (kernel is not flipped) |
| **Weight Sharing** | Same kernel weights used at every spatial position |
| **Local Connectivity** | Each output neuron connects to only a small patch of input |
| **Translation Equivariance** | Shifting the input shifts the output correspondingly |
| **Sobel Filter** | Classic handcrafted kernel for edge detection |
| **Element-wise Multiply & Sum** | Core computation: `O(i,j) = Σ Σ I(i+m,j+n)·K(m,n)` |

---

## 11. Revision Questions

**Q1.** Given a 6×6 input and a 3×3 kernel with stride=1 and no padding, what is the
output size? Show your calculation.

<details>
<summary>Answer</summary>

```
O = (I - K) / S + 1 = (6 - 3) / 1 + 1 = 4
Output size: 4 × 4
```
</details>

**Q2.** Why do CNNs use cross-correlation instead of true mathematical convolution?
What is the difference?

<details>
<summary>Answer</summary>

True convolution flips the kernel (rotates 180°) before sliding; cross-correlation
does not flip. Since kernels are learned, the network can learn the flipped version
if needed, making the distinction irrelevant in practice.
</details>

**Q3.** A vertical edge detection kernel `[[1,0,-1],[1,0,-1],[1,0,-1]]` is applied
to a patch where all pixel values are 128. What is the output? Why?

<details>
<summary>Answer</summary>

```
Output = 128(1+0+(-1)+1+0+(-1)+1+0+(-1)) = 128 × 0 = 0
```
The output is 0 because there is no intensity change — the region is uniform.
Edge detectors only respond to differences.
</details>

**Q4.** Explain why convolution achieves "parameter sharing." How does this differ
from a fully connected layer?

<details>
<summary>Answer</summary>

In convolution, the same kernel weights are applied at every spatial position of
the input. A 3×3 kernel has only 9 parameters regardless of input size. A fully
connected layer would need a separate weight for every input-output pair, leading
to millions of parameters for even modest image sizes.
</details>

**Q5.** Perform the convolution of the following input with the given kernel:

```
Input (3×3):     Kernel (2×2):
┌───┬───┬───┐    ┌───┬───┐
│ 1 │ 2 │ 3 │    │ 1 │ 0 │
├───┼───┼───┤    ├───┼───┤
│ 4 │ 5 │ 6 │    │ 0 │ 1 │
├───┼───┼───┤    └───┴───┘
│ 7 │ 8 │ 9 │
└───┴───┴───┘
```

<details>
<summary>Answer</summary>

```
O(0,0) = 1(1) + 2(0) + 4(0) + 5(1) = 6
O(0,1) = 2(1) + 3(0) + 5(0) + 6(1) = 8
O(1,0) = 4(1) + 5(0) + 7(0) + 8(1) = 12
O(1,1) = 5(1) + 6(0) + 8(0) + 9(1) = 14

Output (2×2):
┌────┬────┐
│  6 │  8 │
├────┼────┤
│ 12 │ 14 │
└────┴────┘
```
</details>

**Q6.** Why is the convolution output smaller than the input when no padding is used?
Derive the output size formula.

<details>
<summary>Answer</summary>

The kernel cannot extend beyond the input boundaries. At stride=1, the kernel
can be placed at `(I - K + 1)` positions along each dimension. For a 7×7 input
with a 3×3 kernel: `7 - 3 + 1 = 5`, giving a 5×5 output. The general formula is:

```
O = ⌊(I - K + 2P) / S⌋ + 1
```

With P=0 and S=1: `O = I - K + 1`
</details>

---

## Navigation

| | |
|---|---|
| ⬅️ **Previous** | [Translation Invariance](../01-Introduction-to-CNNs/05-translation-invariance.md) |
| ➡️ **Next** | [Kernel & Filter Concept](./02-kernel-filter-concept.md) |
| 🏠 **Home** | [CNN Overview](../README.md) |

---

> *"The convolution operation is the backbone of every CNN — a beautifully simple
> idea that captures local patterns through sliding, multiplying, and summing."*
