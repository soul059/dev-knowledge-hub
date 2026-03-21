# Stride & Padding — Controlling the Output Dimensions

> **Chapter 2.3 · Convolution Operation**
> Learn how stride and padding control the spatial dimensions of convolution outputs.

---

## Table of Contents

1. [The Output Size Problem](#1-the-output-size-problem)
2. [Stride — Step Size of the Sliding Window](#2-stride--step-size-of-the-sliding-window)
3. [Padding — Adding Borders to the Input](#3-padding--adding-borders-to-the-input)
4. [The Master Formula](#4-the-master-formula)
5. [Padding Types in Detail](#5-padding-types-in-detail)
6. [Comprehensive Output Size Table](#6-comprehensive-output-size-table)
7. [Worked Examples](#7-worked-examples)
8. [Python & PyTorch Implementation](#8-python--pytorch-implementation)
9. [Applications & Design Guidelines](#9-applications--design-guidelines)
10. [Summary Table](#10-summary-table)
11. [Revision Questions](#11-revision-questions)

---

## 1. The Output Size Problem

When we convolve an input of size `I × I` with a kernel of size `K × K`, the output
shrinks:

```
    Input (7×7)                      Output (5×5)
    ┌───┬───┬───┬───┬───┬───┬───┐   ┌───┬───┬───┬───┬───┐
    │   │   │   │   │   │   │   │   │   │   │   │   │   │
    ├───┼───┼───┼───┼───┼───┼───┤   ├───┼───┼───┼───┼───┤
    │   │   │   │   │   │   │   │   │   │   │   │   │   │
    ├───┼───┼───┼───┼───┼───┼───┤   ├───┼───┼───┼───┼───┤
    │   │   │   │   │   │   │   │   │   │   │   │   │   │
    ├───┼───┼───┼───┼───┼───┼───┤ → ├───┼───┼───┼───┼───┤
    │   │   │   │   │   │   │   │   │   │   │   │   │   │
    ├───┼───┼───┼───┼───┼───┼───┤   ├───┼───┼───┼───┼───┤
    │   │   │   │   │   │   │   │   │   │   │   │   │   │
    ├───┼───┼───┼───┼───┼───┼───┤   └───┴───┴───┴───┴───┘
    │   │   │   │   │   │   │   │
    ├───┼───┼───┼───┼───┼───┼───┤   Lost 2 rows and 2 columns!
    │   │   │   │   │   │   │   │
    └───┴───┴───┴───┴───┴───┴───┘
    
    With a 3×3 kernel: 7 - 3 + 1 = 5
```

### The Two Problems

```
Problem 1: SPATIAL SHRINKAGE
    After many convolution layers, the feature map becomes tiny
    7×7 → 5×5 → 3×3 → 1×1 → Can't convolve anymore!
    
    Solution → PADDING (add border pixels)

Problem 2: COMPUTATIONAL COST
    Large feature maps are expensive to process
    224×224 → We want to reduce this gradually
    
    Solution → STRIDE (skip positions)
```

---

## 2. Stride — Step Size of the Sliding Window

**Stride (S)** controls how many pixels the kernel moves between positions.

### Stride = 1 (Default)

```
    Input (5×5), Kernel (3×3), Stride = 1
    
    The kernel moves 1 pixel at a time:
    
    Position 1:     Position 2:     Position 3:
    ╔═══╗───┬───    ┌─╔═══╗──┐     ┌───╔═══╗
    ║   ║   │   │   │ ║   ║  │     │   ║   ║
    ╠═══╣───┼───    ├─╠═══╣──┤     ├───╠═══╣
    ║   ║   │   │   │ ║   ║  │     │   ║   ║
    ╠═══╣───┼───    ├─╠═══╣──┤     ├───╠═══╣
    ║   ║   │   │   │ ║   ║  │     │   ║   ║
    ╚═══╝───┼───    └─╚═══╝──┘     └───╚═══╝
    │   │   │   │
    ├───┼───┼───┤
    │   │   │   │
    └───┴───┴───┘
    
    Output: (5-3)/1 + 1 = 3 positions per dimension → 3×3 output
```

### Stride = 2

```
    Input (5×5), Kernel (3×3), Stride = 2
    
    The kernel moves 2 pixels at a time:
    
    Position 1:     Position 2:     (skipped middle!)
    ╔═══╗───┬───    ┌───╔═══╗
    ║   ║   │   │   │   ║   ║
    ╠═══╣───┼───    ├───╠═══╣
    ║   ║   │   │   │   ║   ║
    ╠═══╣───┼───    ├───╠═══╣
    ║   ║   │   │   │   ║   ║
    ╚═══╝───┼───    └───╚═══╝
    │   │   │   │
    ├───┼───┼───┤
    │   │   │   │
    └───┴───┴───┘
    
    Output: (5-3)/2 + 1 = 2 positions per dimension → 2×2 output
```

### Stride = 2 on Larger Input

```
    Input (6×6), Kernel (3×3), Stride = 2
    
    Row positions:
    ┌─────────────┐
    │ ╔═══╗       │  Position (0,0)
    │ ╚═══╝       │
    │   ↓ skip 1  │
    │     ╔═══╗   │  Position (0,1) — moved 2 pixels right
    │     ╚═══╝   │
    └─────────────┘
    
    Output: ⌊(6-3)/2⌋ + 1 = ⌊1.5⌋ + 1 = 1 + 1 = 2 → 2×2 output
```

### Visual Comparison: Stride 1 vs Stride 2

```
    Input (7×7) + Kernel (3×3):
    
    Stride = 1:                          Stride = 2:
    Output = (7-3)/1 + 1 = 5×5          Output = (7-3)/2 + 1 = 3×3
    
    ┌───┬───┬───┬───┬───┐              ┌───┬───┬───┐
    │ o │ o │ o │ o │ o │              │ o │ o │ o │
    ├───┼───┼───┼───┼───┤              ├───┼───┼───┤
    │ o │ o │ o │ o │ o │              │ o │ o │ o │
    ├───┼───┼───┼───┼───┤              ├───┼───┼───┤
    │ o │ o │ o │ o │ o │              │ o │ o │ o │
    ├───┼───┼───┼───┼───┤              └───┴───┴───┘
    │ o │ o │ o │ o │ o │
    ├───┼───┼───┼───┼───┤              Downsampled by 2×!
    │ o │ o │ o │ o │ o │              Fewer computations.
    └───┴───┴───┴───┴───┘
    
    25 output values                    9 output values
```

### Why Use Stride > 1?

| Purpose | Explanation |
|---------|-------------|
| **Downsampling** | Reduces spatial dimensions (like pooling) |
| **Computational efficiency** | Fewer output pixels = less computation |
| **Increase receptive field** | Each subsequent layer sees more of the input |
| **Replace pooling** | Some modern architectures use stride-2 conv instead of max-pool |

---

## 3. Padding — Adding Borders to the Input

**Padding (P)** adds extra pixels around the border of the input before convolution.

### Why Pad?

```
Problem 1: Output shrinks               Problem 2: Border pixels underused
                                         
Input:  ┌──────────────┐               Center pixel: used in 9 convolutions
        │              │               Corner pixel: used in only 1 convolution
        │     7 × 7    │               
        │              │               ┌───┬───┬───┬───┬───┐
        └──────────────┘               │ 1 │ 2 │ 3 │ 2 │ 1 │
             ↓ 3×3 conv               ├───┼───┼───┼───┼───┤
Output: ┌──────────┐                   │ 2 │ 4 │ 6 │ 4 │ 2 │
        │          │                   ├───┼───┼───┼───┼───┤
        │   5 × 5  │                   │ 3 │ 6 │ 9 │ 6 │ 3 │
        │          │                   ├───┼───┼───┼───┼───┤
        └──────────┘                   │ 2 │ 4 │ 6 │ 4 │ 2 │
                                       ├───┼───┼───┼───┼───┤
Lost info at borders!                  │ 1 │ 2 │ 3 │ 2 │ 1 │
                                       └───┴───┴───┴───┴───┘
                                       ↑ Times each pixel contributes
```

### Zero Padding Visualization

```
    Original Input (4×4):              After Padding P=1 (6×6):
    
    ┌───┬───┬───┬───┐                ┌───┬───┬───┬───┬───┬───┐
    │ 1 │ 2 │ 3 │ 4 │                │ 0 │ 0 │ 0 │ 0 │ 0 │ 0 │
    ├───┼───┼───┼───┤                ├───┼───┼───┼───┼───┼───┤
    │ 5 │ 6 │ 7 │ 8 │                │ 0 │ 1 │ 2 │ 3 │ 4 │ 0 │
    ├───┼───┼───┼───┤    ──pad──►    ├───┼───┼───┼───┼───┼───┤
    │ 9 │ 0 │ 1 │ 2 │                │ 0 │ 5 │ 6 │ 7 │ 8 │ 0 │
    ├───┼───┼───┼───┤                ├───┼───┼───┼───┼───┼───┤
    │ 3 │ 4 │ 5 │ 6 │                │ 0 │ 9 │ 0 │ 1 │ 2 │ 0 │
    └───┴───┴───┴───┘                ├───┼───┼───┼───┼───┼───┤
                                     │ 0 │ 3 │ 4 │ 5 │ 6 │ 0 │
                                     ├───┼───┼───┼───┼───┼───┤
                                     │ 0 │ 0 │ 0 │ 0 │ 0 │ 0 │
                                     └───┴───┴───┴───┴───┴───┘
    
    Now with 3×3 kernel: Output = (6-3+1) = 4×4 ← Same as input!
```

---

## 4. The Master Formula

The output size for any combination of input size, kernel size, padding, and stride:

```
    ┌─────────────────────────────────────────────────────┐
    │                                                     │
    │         O = ⌊ (I - K + 2P) / S ⌋ + 1               │
    │                                                     │
    │   Where:                                            │
    │     O = Output size (height or width)               │
    │     I = Input size                                  │
    │     K = Kernel size                                 │
    │     P = Padding (pixels added on each side)         │
    │     S = Stride (step size)                          │
    │                                                     │
    │   ⌊ ⌋ = Floor function (round down)                 │
    │                                                     │
    └─────────────────────────────────────────────────────┘
```

### Formula Derivation

```
    After padding, effective input size = I + 2P
    
    Space available for kernel to slide = (I + 2P) - K
    
    Number of steps at stride S = (I + 2P - K) / S
    
    Total positions = steps + 1 (for the starting position)
    
    O = ⌊(I + 2P - K) / S⌋ + 1
    
    Rearranging: O = ⌊(I - K + 2P) / S⌋ + 1
```

### Special Cases

```
    Case 1: No padding, Stride 1
        O = I - K + 1
        Example: I=7, K=3 → O = 7 - 3 + 1 = 5
    
    Case 2: "Same" padding (output = input), Stride 1
        I = I - K + 2P + 1
        P = (K - 1) / 2
        For K=3: P = 1
        For K=5: P = 2
        For K=7: P = 3
    
    Case 3: Stride 2, No padding
        O = ⌊(I - K) / 2⌋ + 1
        Example: I=8, K=3 → O = ⌊5/2⌋ + 1 = 2 + 1 = 3

    Case 4: Stride 2, "Same" padding
        O = ⌈I / S⌉ = ⌈I / 2⌉
        Example: I=8 → O = 4
```

---

## 5. Padding Types in Detail

### 5.1 Valid Padding (No Padding)

```
    P = 0
    
    Input (5×5), Kernel (3×3):
    
    ┌───┬───┬───┬───┬───┐        Output (3×3):
    │   │   │   │   │   │        ┌───┬───┬───┐
    ├───┼───┼───┼───┼───┤        │   │   │   │
    │   │   │   │   │   │   →    ├───┼───┼───┤
    ├───┼───┼───┼───┼───┤        │   │   │   │
    │   │   │   │   │   │        ├───┼───┼───┤
    ├───┼───┼───┼───┼───┤        │   │   │   │
    │   │   │   │   │   │        └───┴───┴───┘
    ├───┼───┼───┼───┼───┤
    │   │   │   │   │   │        O = 5 - 3 + 1 = 3
    └───┴───┴───┴───┴───┘
    
    ✅ No artificial values introduced
    ❌ Output shrinks
    ❌ Border pixels underrepresented
```

### 5.2 Same Padding (Preserve Dimensions)

```
    P = (K-1)/2  (for odd K with stride 1)
    
    For K=3: P=1        For K=5: P=2        For K=7: P=3
    
    Input (5×5) + P=1 → Padded (7×7):
    
    ┌───┬───┬───┬───┬───┬───┬───┐    Output (5×5):
    │ 0 │ 0 │ 0 │ 0 │ 0 │ 0 │ 0 │    ┌───┬───┬───┬───┬───┐
    ├───┼───┼───┼───┼───┼───┼───┤    │   │   │   │   │   │
    │ 0 │   │   │   │   │   │ 0 │    ├───┼───┼───┼───┼───┤
    ├───┼───┼───┼───┼───┼───┼───┤    │   │   │   │   │   │
    │ 0 │   │ input │   │   │ 0 │ →  ├───┼───┼───┼───┼───┤
    ├───┼───┼───┼───┼───┼───┼───┤    │   │   │   │   │   │
    │ 0 │   │   │   │   │   │ 0 │    ├───┼───┼───┼───┼───┤
    ├───┼───┼───┼───┼───┼───┼───┤    │   │   │   │   │   │
    │ 0 │   │   │   │   │   │ 0 │    ├───┼───┼───┼───┼───┤
    ├───┼───┼───┼───┼───┼───┼───┤    │   │   │   │   │   │
    │ 0 │ 0 │ 0 │ 0 │ 0 │ 0 │ 0 │    └───┴───┴───┴───┴───┘
    └───┴───┴───┴───┴───┴───┴───┘
                                      O = (5 - 3 + 2×1)/1 + 1 = 5 ✓
    
    ✅ Output size = Input size
    ✅ Border pixels participate more
    ❌ Introduces artificial zero values at borders
```

### 5.3 Full Padding

```
    P = K - 1
    
    For K=3: P=2
    
    Input (3×3) + P=2 → Padded (7×7):
    
    ┌───┬───┬───┬───┬───┬───┬───┐    Output (5×5):
    │ 0 │ 0 │ 0 │ 0 │ 0 │ 0 │ 0 │    ┌───┬───┬───┬───┬───┐
    ├───┼───┼───┼───┼───┼───┼───┤    │   │   │   │   │   │
    │ 0 │ 0 │ 0 │ 0 │ 0 │ 0 │ 0 │    ├───┼───┼───┼───┼───┤
    ├───┼───┼───┼───┼───┼───┼───┤    │   │   │   │   │   │
    │ 0 │ 0 │   │   │   │ 0 │ 0 │ →  ├───┼───┼───┼───┼───┤
    ├───┼───┼───┼───┼───┼───┼───┤    │   │   │   │   │   │
    │ 0 │ 0 │  input  │  │ 0 │ 0 │    ├───┼───┼───┼───┼───┤
    ├───┼───┼───┼───┼───┼───┼───┤    │   │   │   │   │   │
    │ 0 │ 0 │   │   │   │ 0 │ 0 │    ├───┼───┼───┼───┼───┤
    ├───┼───┼───┼───┼───┼───┼───┤    │   │   │   │   │   │
    │ 0 │ 0 │ 0 │ 0 │ 0 │ 0 │ 0 │    └───┴───┴───┴───┴───┘
    ├───┼───┼───┼───┼───┼───┼───┤
    │ 0 │ 0 │ 0 │ 0 │ 0 │ 0 │ 0 │    O = (3 - 3 + 2×2)/1 + 1 = 5
    └───┴───┴───┴───┴───┴───┴───┘    Output > Input!
    
    ✅ Every input pixel contributes to K×K output positions
    ❌ Many artificial zeros, rarely used in CNNs
```

### 5.4 Zero Padding vs Other Padding Modes

```
    Original:  [a  b  c  d  e]

    Zero:      [0  0 │ a  b  c  d  e │ 0  0]
               Adds zeros — most common, simple

    Reflect:   [c  b │ a  b  c  d  e │ d  c]
               Mirrors values — preserves statistics

    Replicate: [a  a │ a  b  c  d  e │ e  e]
               Repeats edge — avoids sharp transitions

    Circular:  [d  e │ a  b  c  d  e │ a  b]
               Wraps around — assumes periodic signal
```

### 2D Padding Modes Comparison

```
    Original (3×3):          Zero Padded (P=1):
    ┌───┬───┬───┐           ┌───┬───┬───┬───┬───┐
    │ a │ b │ c │           │ 0 │ 0 │ 0 │ 0 │ 0 │
    ├───┼───┼───┤           ├───┼───┼───┼───┼───┤
    │ d │ e │ f │           │ 0 │ a │ b │ c │ 0 │
    ├───┼───┼───┤           ├───┼───┼───┼───┼───┤
    │ g │ h │ i │           │ 0 │ d │ e │ f │ 0 │
    └───┴───┴───┘           ├───┼───┼───┼───┼───┤
                            │ 0 │ g │ h │ i │ 0 │
                            ├───┼───┼───┼───┼───┤
                            │ 0 │ 0 │ 0 │ 0 │ 0 │
                            └───┴───┴───┴───┴───┘

    Reflect Padded (P=1):    Replicate Padded (P=1):
    ┌───┬───┬───┬───┬───┐   ┌───┬───┬───┬───┬───┐
    │ e │ d │ e │ f │ e │   │ a │ a │ b │ c │ c │
    ├───┼───┼───┼───┼───┤   ├───┼───┼───┼───┼───┤
    │ b │ a │ b │ c │ b │   │ a │ a │ b │ c │ c │
    ├───┼───┼───┼───┼───┤   ├───┼───┼───┼───┼───┤
    │ e │ d │ e │ f │ e │   │ d │ d │ e │ f │ f │
    ├───┼───┼───┼───┼───┤   ├───┼───┼───┼───┼───┤
    │ h │ g │ h │ i │ h │   │ g │ g │ h │ i │ i │
    ├───┼───┼───┼───┼───┤   ├───┼───┼───┼───┼───┤
    │ e │ d │ e │ f │ e │   │ g │ g │ h │ i │ i │
    └───┴───┴───┴───┴───┘   └───┴───┴───┴───┴───┘
```

| Padding Mode | PyTorch Argument | Best For |
|-------------|-----------------|----------|
| Zero | `padding_mode='zeros'` | Most CNN layers (default) |
| Reflect | `padding_mode='reflect'` | Image generation, style transfer |
| Replicate | `padding_mode='replicate'` | Boundary-sensitive tasks |
| Circular | `padding_mode='circular'` | Periodic signals, panoramic images |

---

## 6. Comprehensive Output Size Table

### Table: Input × Kernel × Stride × Padding → Output

| Input (I) | Kernel (K) | Stride (S) | Padding (P) | Output (O) | Formula Check |
|-----------|-----------|------------|-------------|------------|---------------|
| 5 | 3 | 1 | 0 | **3** | ⌊(5-3+0)/1⌋+1 = 3 |
| 5 | 3 | 1 | 1 | **5** | ⌊(5-3+2)/1⌋+1 = 5 (same) |
| 5 | 3 | 2 | 0 | **2** | ⌊(5-3+0)/2⌋+1 = 2 |
| 5 | 3 | 2 | 1 | **3** | ⌊(5-3+2)/2⌋+1 = 3 |
| 7 | 3 | 1 | 0 | **5** | ⌊(7-3+0)/1⌋+1 = 5 |
| 7 | 3 | 1 | 1 | **7** | ⌊(7-3+2)/1⌋+1 = 7 (same) |
| 7 | 3 | 2 | 0 | **3** | ⌊(7-3+0)/2⌋+1 = 3 |
| 7 | 3 | 2 | 1 | **4** | ⌊(7-3+2)/2⌋+1 = 4 |
| 7 | 5 | 1 | 0 | **3** | ⌊(7-5+0)/1⌋+1 = 3 |
| 7 | 5 | 1 | 2 | **7** | ⌊(7-5+4)/1⌋+1 = 7 (same) |
| 7 | 5 | 2 | 0 | **2** | ⌊(7-5+0)/2⌋+1 = 2 |
| 7 | 5 | 2 | 2 | **4** | ⌊(7-5+4)/2⌋+1 = 4 |
| 8 | 3 | 1 | 0 | **6** | ⌊(8-3+0)/1⌋+1 = 6 |
| 8 | 3 | 2 | 0 | **3** | ⌊(8-3+0)/2⌋+1 = 3 |
| 8 | 3 | 2 | 1 | **4** | ⌊(8-3+2)/2⌋+1 = 4 |
| 28 | 3 | 1 | 1 | **28** | ⌊(28-3+2)/1⌋+1 = 28 (same) |
| 28 | 5 | 1 | 2 | **28** | ⌊(28-5+4)/1⌋+1 = 28 (same) |
| 28 | 5 | 2 | 2 | **14** | ⌊(28-5+4)/2⌋+1 = 14 |
| 32 | 3 | 1 | 1 | **32** | ⌊(32-3+2)/1⌋+1 = 32 (same) |
| 32 | 5 | 2 | 2 | **16** | ⌊(32-5+4)/2⌋+1 = 16 |
| 224 | 3 | 1 | 1 | **224** | ⌊(224-3+2)/1⌋+1 = 224 (same) |
| 224 | 7 | 2 | 3 | **112** | ⌊(224-7+6)/2⌋+1 = 112 |
| 224 | 11 | 4 | 2 | **55** | ⌊(224-11+4)/4⌋+1 = 55 |

### Quick Reference: "Same" Padding Values

| Kernel Size | Padding for "Same" (S=1) |
|-------------|-------------------------|
| 1×1 | P = 0 |
| 3×3 | P = 1 |
| 5×5 | P = 2 |
| 7×7 | P = 3 |
| 9×9 | P = 4 |
| K×K (odd) | P = (K-1)/2 |

---

## 7. Worked Examples

### Example 1: Standard Conv Layer (ResNet-style)

```
    Input:   56 × 56
    Kernel:  3 × 3
    Stride:  1
    Padding: 1

    O = ⌊(56 - 3 + 2×1) / 1⌋ + 1
      = ⌊(56 - 3 + 2) / 1⌋ + 1
      = ⌊55 / 1⌋ + 1
      = 55 + 1
      = 56 ← Same as input ✓
```

### Example 2: Downsampling Conv (Stride 2)

```
    Input:   56 × 56
    Kernel:  3 × 3
    Stride:  2
    Padding: 1

    O = ⌊(56 - 3 + 2×1) / 2⌋ + 1
      = ⌊55 / 2⌋ + 1
      = 27 + 1
      = 28 ← Exactly half! ✓
```

### Example 3: AlexNet First Layer

```
    Input:   224 × 224
    Kernel:  11 × 11
    Stride:  4
    Padding: 2

    O = ⌊(224 - 11 + 2×2) / 4⌋ + 1
      = ⌊(224 - 11 + 4) / 4⌋ + 1
      = ⌊217 / 4⌋ + 1
      = 54 + 1
      = 55
```

### Example 4: VGGNet Layer + Max Pool

```
    Input:   224 × 224
    Conv 3×3, S=1, P=1:  → 224 × 224  (same padding)
    Conv 3×3, S=1, P=1:  → 224 × 224  (same padding)
    MaxPool 2×2, S=2:    → 112 × 112  (halved)
    
    Conv 3×3, S=1, P=1:  → 112 × 112  (same padding)
    Conv 3×3, S=1, P=1:  → 112 × 112  (same padding)
    MaxPool 2×2, S=2:    → 56 × 56    (halved)
```

### Example 5: Fractional Output (Floor Division)

```
    Input:   6 × 6
    Kernel:  3 × 3
    Stride:  2
    Padding: 0

    O = ⌊(6 - 3 + 0) / 2⌋ + 1
      = ⌊3 / 2⌋ + 1
      = 1 + 1
      = 2

    Note: The rightmost column and bottom row are NOT covered!
    
    ┌───┬───┬───┬───┬───┬───┐
    │ × │ × │ × │   │   │   │   Position (0,0): columns 0-2
    ├───┼───┼───┼───┼───┼───┤
    │ × │ × │ × │   │   │   │
    ├───┼───┼───┼───┼───┼───┤
    │ × │ × │ × │   │   │   │   Position (0,1): columns 2-4
    ├───┼───┼───┼───┼───┼───┤   → Column 5 is never touched!
    │   │   │   │   │   │   │
    ├───┼───┼───┼───┼───┼───┤
    │   │   │   │   │   │   │
    ├───┼───┼───┼───┼───┼───┤
    │   │   │   │ ← │ n │ e │   ← These pixels are dropped
    └───┴───┴───┴───┴───┴───┘       by floor division
```

### Example 6: Computing Padding for Target Output Size

```
    Problem: I want Output = 14 from Input = 28, using K=5, S=2

    O = ⌊(I - K + 2P) / S⌋ + 1
    14 = ⌊(28 - 5 + 2P) / 2⌋ + 1
    13 = ⌊(23 + 2P) / 2⌋
    
    For 13 = ⌊(23 + 2P) / 2⌋:
    26 ≤ 23 + 2P < 28
    3 ≤ 2P < 5
    1.5 ≤ P < 2.5
    
    P = 2 ✓
    
    Verify: ⌊(28 - 5 + 4) / 2⌋ + 1 = ⌊27/2⌋ + 1 = 13 + 1 = 14 ✓
```

---

## 8. Python & PyTorch Implementation

### Output Size Calculator

```python
import math

def conv_output_size(input_size, kernel_size, stride=1, padding=0):
    """Calculate the output size of a convolution operation."""
    return math.floor((input_size - kernel_size + 2 * padding) / stride) + 1

def same_padding(kernel_size):
    """Calculate padding needed for 'same' output with stride=1."""
    return (kernel_size - 1) // 2

# ── Examples ──
print("=== Output Size Examples ===")
examples = [
    (224, 7, 2, 3),   # ResNet first layer
    (224, 11, 4, 2),  # AlexNet first layer
    (56, 3, 1, 1),    # Standard conv
    (56, 3, 2, 1),    # Downsampling conv
    (28, 5, 1, 2),    # 5×5 same conv
    (28, 5, 2, 2),    # 5×5 stride-2 conv
]

for I, K, S, P in examples:
    O = conv_output_size(I, K, S, P)
    print(f"  I={I:3d}, K={K:2d}, S={S}, P={P} → O={O:3d}")

# Output:
#   I=224, K= 7, S=2, P=3 → O=112
#   I=224, K=11, S=4, P=2 → O= 55
#   I= 56, K= 3, S=1, P=1 → O= 56
#   I= 56, K= 3, S=2, P=1 → O= 28
#   I= 28, K= 5, S=1, P=2 → O= 28
#   I= 28, K= 5, S=2, P=2 → O= 14

print("\n=== Same Padding Values ===")
for k in [1, 3, 5, 7, 9, 11]:
    print(f"  Kernel {k:2d}×{k:2d} → Padding = {same_padding(k)}")
```

### PyTorch: Stride & Padding in Action

```python
import torch
import torch.nn as nn

# Create a sample input: batch=1, channels=1, height=7, width=7
x = torch.randn(1, 1, 7, 7)

# Different stride/padding combinations
configs = [
    {"kernel_size": 3, "stride": 1, "padding": 0},  # valid
    {"kernel_size": 3, "stride": 1, "padding": 1},  # same
    {"kernel_size": 3, "stride": 2, "padding": 0},  # downsample
    {"kernel_size": 3, "stride": 2, "padding": 1},  # downsample with padding
    {"kernel_size": 5, "stride": 1, "padding": 2},  # 5×5 same
    {"kernel_size": 5, "stride": 2, "padding": 2},  # 5×5 downsample
]

print("Input shape:", x.shape)
print(f"{'K':>3s} {'S':>3s} {'P':>3s} → {'Output Shape':<20s}")
print("-" * 35)

for cfg in configs:
    conv = nn.Conv2d(1, 1, **cfg, bias=False)
    out = conv(x)
    k, s, p = cfg['kernel_size'], cfg['stride'], cfg['padding']
    print(f"{k:3d} {s:3d} {p:3d} → {str(out.shape):<20s}")

# Output:
#   K   S   P → Output Shape
# -----------------------------------
#   3   1   0 → torch.Size([1, 1, 5, 5])
#   3   1   1 → torch.Size([1, 1, 7, 7])
#   3   2   0 → torch.Size([1, 1, 3, 3])
#   3   2   1 → torch.Size([1, 1, 4, 4])
#   5   1   2 → torch.Size([1, 1, 7, 7])
#   5   2   2 → torch.Size([1, 1, 4, 4])
```

### PyTorch: Different Padding Modes

```python
import torch
import torch.nn as nn

x = torch.tensor([[[[1., 2., 3.],
                     [4., 5., 6.],
                     [7., 8., 9.]]]])

# Zero padding (default)
conv_zero = nn.Conv2d(1, 1, 3, padding=1, padding_mode='zeros', bias=False)
with torch.no_grad():
    conv_zero.weight.fill_(1.0 / 9.0)  # Box blur

# Reflect padding
conv_reflect = nn.Conv2d(1, 1, 3, padding=1, padding_mode='reflect', bias=False)
with torch.no_grad():
    conv_reflect.weight.fill_(1.0 / 9.0)

# Replicate padding
conv_replicate = nn.Conv2d(1, 1, 3, padding=1, padding_mode='replicate', bias=False)
with torch.no_grad():
    conv_replicate.weight.fill_(1.0 / 9.0)

print("Zero pad output:\n", conv_zero(x).squeeze().detach())
print("Reflect pad output:\n", conv_reflect(x).squeeze().detach())
print("Replicate pad output:\n", conv_replicate(x).squeeze().detach())
```

### Convolution with Stride — NumPy Implementation

```python
import numpy as np

def conv2d_stride_pad(input_matrix, kernel, stride=1, padding=0):
    """
    2D convolution with configurable stride and zero padding.
    """
    # Add zero padding
    if padding > 0:
        input_matrix = np.pad(input_matrix, padding, mode='constant')
    
    H, W = input_matrix.shape
    k = kernel.shape[0]
    
    out_h = (H - k) // stride + 1
    out_w = (W - k) // stride + 1
    
    output = np.zeros((out_h, out_w))
    
    for i in range(out_h):
        for j in range(out_w):
            row = i * stride
            col = j * stride
            patch = input_matrix[row:row+k, col:col+k]
            output[i, j] = np.sum(patch * kernel)
    
    return output


# ── Test all combinations ──
inp = np.arange(1, 50).reshape(7, 7).astype(float)
kernel = np.ones((3, 3)) / 9  # box blur

configs = [
    (1, 0, "valid"),
    (1, 1, "same"),
    (2, 0, "stride-2"),
    (2, 1, "stride-2 + pad-1"),
]

for stride, pad, name in configs:
    out = conv2d_stride_pad(inp, kernel, stride=stride, padding=pad)
    print(f"{name:20s} → Input 7×7 → Output {out.shape[0]}×{out.shape[1]}")

# Output:
# valid                → Input 7×7 → Output 5×5
# same                 → Input 7×7 → Output 7×7
# stride-2             → Input 7×7 → Output 3×3
# stride-2 + pad-1     → Input 7×7 → Output 4×4
```

---

## 9. Applications & Design Guidelines

### Common Architecture Patterns

```
    Pattern 1: "Same" Conv + MaxPool (VGGNet style)
    ┌────────────────┐  ┌────────────────┐  ┌────────────────┐
    │ Conv 3×3, S=1  │→ │ Conv 3×3, S=1  │→ │ MaxPool 2×2    │
    │ P=1 (same)     │  │ P=1 (same)     │  │ S=2            │
    │ 224→224        │  │ 224→224        │  │ 224→112        │
    └────────────────┘  └────────────────┘  └────────────────┘
    
    Pattern 2: Stride-2 Conv (Modern style — replaces pooling)
    ┌────────────────┐  ┌────────────────┐
    │ Conv 3×3, S=1  │→ │ Conv 3×3, S=2  │
    │ P=1 (same)     │  │ P=1            │
    │ 224→224        │  │ 224→112        │
    └────────────────┘  └────────────────┘
    
    Pattern 3: Aggressive First Layer (AlexNet, ResNet)
    ┌────────────────┐
    │ Conv 7×7, S=2  │
    │ P=3             │
    │ 224→112         │
    └────────────────┘
```

### Design Guidelines

| Guideline | Recommendation |
|-----------|---------------|
| **Preserve spatial dims** | Use K=3, S=1, P=1 (or K=5, S=1, P=2) |
| **Halve spatial dims** | Use K=3, S=2, P=1 (output ≈ input/2) |
| **First layer (large receptive field)** | K=7, S=2, P=3 (ResNet) or K=11, S=4, P=2 (AlexNet) |
| **Padding mode** | Use zero padding by default; reflect for generative models |
| **Odd kernel sizes** | Always prefer odd K (1, 3, 5, 7) — enables symmetric padding |

---

## 10. Summary Table

| Concept | Description |
|---------|-------------|
| **Stride** | Step size of the kernel; S=2 downsamples by ~2× |
| **Padding** | Extra border pixels; prevents spatial shrinkage |
| **Valid padding** | P=0; output shrinks; no artificial values |
| **Same padding** | P=(K-1)/2 with S=1; output = input size |
| **Full padding** | P=K-1; output > input; every pixel contributes K² times |
| **Output formula** | `O = ⌊(I - K + 2P) / S⌋ + 1` |
| **Zero padding** | Most common; fills with 0s |
| **Reflect padding** | Mirrors border values; better for generation tasks |
| **Replicate padding** | Copies edge values; avoids artificial zeros |
| **Floor division** | Drops incomplete positions at borders |

---

## 11. Revision Questions

**Q1.** Calculate the output size for: Input=32×32, Kernel=5×5, Stride=1, Padding=2.
Is this a "same" convolution?

<details>
<summary>Answer</summary>

```
O = ⌊(32 - 5 + 2×2) / 1⌋ + 1 = ⌊31/1⌋ + 1 = 32
```
Yes, this is "same" convolution because O = I = 32. For K=5, P=(5-1)/2=2 gives "same" output.
</details>

**Q2.** An input of size 112×112 is convolved with a 3×3 kernel, stride=2, padding=1.
What is the output size? What fraction of the original resolution is preserved?

<details>
<summary>Answer</summary>

```
O = ⌊(112 - 3 + 2) / 2⌋ + 1 = ⌊111/2⌋ + 1 = 55 + 1 = 56
```
The output is 56×56, which is exactly half of 112×112 (1/4 of the pixels).
</details>

**Q3.** Why are odd-sized kernels (3×3, 5×5, 7×7) preferred over even-sized
(2×2, 4×4, 6×6)?

<details>
<summary>Answer</summary>

Odd kernels have a well-defined center pixel, enabling symmetric padding on both
sides. With K=3, P=(3-1)/2=1 gives an integer. With K=4, P=(4-1)/2=1.5 is not
an integer, making symmetric "same" padding impossible.
</details>

**Q4.** Design a convolution configuration that takes a 224×224 input and produces
a 56×56 output in a single layer. What kernel, stride, and padding would you use?

<details>
<summary>Answer</summary>

Need O = 56 from I = 224. That's a 4× reduction, so S=4 is a good start.

```
56 = ⌊(224 - K + 2P) / 4⌋ + 1
55 = ⌊(224 - K + 2P) / 4⌋
220 = 224 - K + 2P
K - 2P = 4
```

Options: K=8, P=2 or K=6, P=1 or K=4, P=0. Since odd kernels preferred: use
K=7, S=4, P=3 → ⌊(224-7+6)/4⌋+1 = ⌊223/4⌋+1 = 55+1 = 56 ✓ (close, but
actually K=7,S=4,P=4 → 57, so we'd need K=8,S=4,P=2 for exact 56).

Most practical: K=7, S=4, P=2 → ⌊(224-7+4)/4⌋+1 = ⌊221/4⌋+1 = 55+1 = 56 ✓
</details>

**Q5.** Explain the difference between using stride=2 for downsampling vs
using stride=1 followed by max pooling. What are the tradeoffs?

<details>
<summary>Answer</summary>

| Approach | Stride-2 Conv | Conv + MaxPool |
|----------|--------------|----------------|
| Parameters | Fewer (one layer) | More (conv + pool) |
| Information loss | Learned what to discard | Max value preserved |
| Computation | Less (fewer output positions) | More |
| Modern preference | ✅ Increasingly popular | Classic approach |

Stride-2 conv learns what information to discard, while max pooling always keeps the maximum value. Stride-2 is more parameter-efficient but may lose important details.
</details>

**Q6.** For an input of size 13×13, kernel 3×3, stride 2, padding 0: what is the
output? Are any input pixels at the borders not covered by any kernel position?

<details>
<summary>Answer</summary>

```
O = ⌊(13 - 3 + 0) / 2⌋ + 1 = ⌊10/2⌋ + 1 = 5 + 1 = 6
```

Positions along one axis: 0, 2, 4, 6, 8, 10. Each covers 3 pixels.
Last position covers pixels 10, 11, 12. So all 13 pixels (0–12) are covered. ✓
No pixels are missed in this case.
</details>

---

## Navigation

| | |
|---|---|
| ⬅️ **Previous** | [Kernel & Filter Concept](./02-kernel-filter-concept.md) |
| ➡️ **Next** | [Feature Maps](./04-feature-maps.md) |
| 🏠 **Home** | [CNN Overview](../README.md) |

---

> *"Stride and padding are the architect's tools — they let you control exactly
> how information flows and transforms through the network's layers."*
