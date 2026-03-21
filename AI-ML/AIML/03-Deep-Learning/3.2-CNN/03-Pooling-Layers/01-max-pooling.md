# 1 · Max Pooling

> **Unit 3.2 – Convolutional Neural Networks / 03 – Pooling Layers**
>
> ← Previous: *Start of Unit* | [Next: Average Pooling →](./02-average-pooling.md)
>
> [↑ Back to Pooling Layers Overview](./README.md)

---

## 1.1 Chapter Overview

Max Pooling is the most widely used pooling operation in convolutional neural networks. It slides a fixed-size window across the spatial dimensions of a feature map and **retains only the maximum value** within each window. The operation performs aggressive, non-linear downsampling that:

- **Reduces spatial dimensions** — cutting computation in deeper layers.
- **Provides translation invariance** — small shifts in the input do not change the pooled output.
- **Selects dominant features** — the maximum activation indicates the strongest presence of a learned feature.

Max pooling was popularized by AlexNet (2012) and remains a staple in architectures like VGG, ResNet, and Inception.

---

## 1.2 Mathematical Definition

### Formal Notation

Given an input feature map **X** of shape `(C, H_in, W_in)`, a pooling window of size `k × k`, stride `s`, and padding `p`:

```
Output spatial dimensions:

            ⌊ H_in + 2p - k ⌋
  H_out  =  ⎢ ─────────────── ⎥ + 1
            ⌊       s         ⌋

            ⌊ W_in + 2p - k ⌋
  W_out  =  ⎢ ─────────────── ⎥ + 1
            ⌊       s         ⌋
```

For the most common configuration — **k = 2, s = 2, p = 0** — the formula simplifies to:

```
  H_out = H_in / 2
  W_out = W_in / 2
```

### Pointwise Operation

For each output position `(i, j)` and channel `c`:

```
  Y[c, i, j] = max      X[c, i·s + m, j·s + n]
              0≤m<k
              0≤n<k
```

Key properties:

| Property         | Value                                    |
|------------------|------------------------------------------|
| Learnable params | **0** — no weights, no bias              |
| Non-linearity    | **Yes** — `max` is a non-linear function |
| Channel mixing   | **No** — operates independently per channel |
| Typical config   | k=2, s=2, p=0 (halves spatial dims)      |

---

## 1.3 ASCII Diagram — Max Pool 2×2, Stride 2

```
  INPUT FEATURE MAP (4×4)                 OUTPUT (2×2)
 ┌─────┬─────┬─────┬─────┐
 │  1  │  3  │  2  │  8  │        ┌─────┬─────┐
 ├─────┼─────┼─────┼─────┤        │  6  │  8  │   ← max(1,3,5,6)=6
 │  5  │  6  │  4  │  1  │   ──►  ├─────┼─────┤     max(2,8,4,1)=8
 ├─────┼─────┼─────┼─────┤        │  9  │  7  │   ← max(3,9,2,1)=9
 │  3  │  9  │  5  │  7  │        └─────┴─────┘     max(5,7,6,3)=7
 ├─────┼─────┼─────┼─────┤
 │  2  │  1  │  6  │  3  │
 └─────┴─────┴─────┴─────┘

  Window 1 (top-left):             Window 2 (top-right):
  ┌─────┬─────┐                    ┌─────┬─────┐
  │  1  │  3  │  max = 6           │  2  │  8  │  max = 8
  │  5  │ [6] │  ← winner          │  4  │  1  │
  └─────┴─────┘                    └─────┴─────┘
                                        ↑ [8] is winner

  Window 3 (bottom-left):          Window 4 (bottom-right):
  ┌─────┬─────┐                    ┌─────┬─────┐
  │  3  │ [9] │  max = 9           │  5  │ [7] │  max = 7
  │  2  │  1  │                    │  6  │  3  │
  └─────┴─────┘                    └─────┴─────┘
```

### Max Pool with Overlapping Windows (k=3, s=1, p=1)

When stride < kernel size, windows overlap, producing smoother outputs with the same spatial size:

```
  INPUT (4×4)               OUTPUT (4×4) ← same spatial size
 ┌────┬────┬────┬────┐     ┌────┬────┬────┬────┐
 │ 1  │ 3  │ 2  │ 8  │     │ 6  │ 6  │ 8  │ 8  │
 ├────┼────┼────┼────┤     ├────┼────┼────┼────┤
 │ 5  │ 6  │ 4  │ 1  │ ──► │ 9  │ 9  │ 8  │ 8  │
 ├────┼────┼────┼────┤     ├────┼────┼────┼────┤
 │ 3  │ 9  │ 5  │ 7  │     │ 9  │ 9  │ 9  │ 7  │
 ├────┼────┼────┼────┤     ├────┼────┼────┼────┤
 │ 2  │ 1  │ 6  │ 3  │     │ 9  │ 9  │ 7  │ 7  │
 └────┴────┴────┴────┘     └────┴────┴────┴────┘
  (zero-padded borders)
```

---

## 1.4 Translation Invariance

Max pooling provides **local translation invariance**: if a feature (high activation) shifts by a small amount within the pooling window, the output remains the same.

```
  Original:                   Shifted 1px right:
  ┌─────┬─────┐              ┌─────┬─────┐
  │  1  │ [9] │  max = 9     │ [9] │  1  │  max = 9  ← SAME OUTPUT
  │  3  │  2  │              │  2  │  3  │
  └─────┴─────┘              └─────┴─────┘
```

**Why this matters:**

1. **Robustness** — The network becomes less sensitive to exact spatial positions of features.
2. **Object recognition** — A cat's ear detected at pixel (10, 10) or (11, 10) should produce the same classification.
3. **Hierarchical abstraction** — Successive pooling layers build increasingly position-invariant representations.

> **Caveat:** Full translation invariance requires invariance at every layer. Max pooling provides only *local* invariance within each window. Global invariance typically requires data augmentation or specialized architectures.

---

## 1.5 Backpropagation Through Max Pooling — Gradient Routing

Max pooling has **no learnable parameters**, but gradients must still flow through it during backpropagation so that upstream convolutional layers can update their weights.

### The Gradient Rule

During the forward pass, max pooling records **which element was the maximum** (the "switch" or "mask"). During backpropagation, the gradient is **routed entirely to that winning element**; all other elements receive zero gradient.

```
  Forward Pass:                     Backward Pass:
  ┌─────┬─────┐                     ┌─────┬─────┐
  │  1  │  3  │                     │  0  │  0  │
  │  5  │ [6] │ → max = 6          │  0  │ dL  │ ← gradient goes only
  └─────┴─────┘                     └─────┴─────┘   to the max element

  If ∂L/∂Y = dL  (gradient from upstream),

  Then:
      ∂L/∂X[m,n] = dL    if X[m,n] was the max (winner)
      ∂L/∂X[m,n] = 0     otherwise
```

### Formal Expression

```
  ∂L        ∂L
  ──── = ── ──── · 𝟙(X[c, i·s+m, j·s+n] = Y[c, i, j])
  ∂X[c, i·s+m, j·s+n]   ∂Y[c, i, j]

  where 𝟙(·) is the indicator function (1 if true, 0 if false)
```

### ASCII Diagram — Full Backward Pass

```
  FORWARD:
  Input (4×4)                    Output (2×2)
  ┌────┬────┬────┬────┐         ┌────┬────┐
  │ 1  │ 3  │ 2  │[8] │         │ 6  │ 8  │
  ├────┼────┼────┼────┤   ──►   ├────┼────┤
  │ 5  │[6] │ 4  │ 1  │         │ 9  │ 7  │
  ├────┼────┼────┼────┤         └────┴────┘
  │ 3  │[9] │ 5  │[7] │
  ├────┼────┼────┼────┤    [·] = local max (switch positions)
  │ 2  │ 1  │ 6  │ 3  │
  └────┴────┴────┴────┘

  BACKWARD (upstream gradient dL/dY):
  ┌─────┬─────┐              Routed to input:
  │ g1  │ g2  │              ┌─────┬─────┬─────┬─────┐
  ├─────┼─────┤              │  0  │  0  │  0  │ g2  │
  │ g3  │ g4  │              ├─────┼─────┼─────┼─────┤
  └─────┴─────┘              │  0  │ g1  │  0  │  0  │
                             ├─────┼─────┼─────┼─────┤
                             │  0  │ g3  │  0  │ g4  │
                             ├─────┼─────┼─────┼─────┤
                             │  0  │  0  │  0  │  0  │
                             └─────┴─────┴─────┴─────┘
```

**Implication:** Only the "winning" neurons get updated. This creates a form of **sparse gradient flow** similar to ReLU, which can be beneficial for learning sharp, discriminative features.

---

## 1.6 Worked Example — Step by Step

### Problem

Apply **Max Pool 2×2, stride 2** to the following single-channel feature map:

```
  Input X (6×6):
  ┌────┬────┬────┬────┬────┬────┐
  │  4 │  2 │  7 │  1 │  3 │  5 │
  ├────┼────┼────┼────┼────┼────┤
  │  6 │  8 │  3 │  9 │  2 │  4 │
  ├────┼────┼────┼────┼────┼────┤
  │  1 │  5 │ 11 │  0 │  8 │  6 │
  ├────┼────┼────┼────┼────┼────┤
  │  3 │  7 │  2 │  4 │  1 │  9 │
  ├────┼────┼────┼────┼────┼────┤
  │  0 │  3 │  5 │  8 │ 10 │  2 │
  ├────┼────┼────┼────┼────┼────┤
  │  1 │  6 │  4 │  7 │  3 │ 12 │
  └────┴────┴────┴────┴────┴────┘
```

### Step 1: Compute Output Dimensions

```
  H_out = ⌊(6 + 0 - 2) / 2⌋ + 1 = ⌊4/2⌋ + 1 = 2 + 1 = 3
  W_out = ⌊(6 + 0 - 2) / 2⌋ + 1 = ⌊4/2⌋ + 1 = 2 + 1 = 3

  Output shape: 3 × 3
```

### Step 2: Slide Windows and Take Max

```
  Window (0,0): rows 0-1, cols 0-1    Window (0,1): rows 0-1, cols 2-3
  │ 4  2 │                            │ 7  1 │
  │ 6 [8]│ → max = 8                  │ 3 [9]│ → max = 9
                                       
  Window (0,2): rows 0-1, cols 4-5    Window (1,0): rows 2-3, cols 0-1
  │ 3 [5]│                            │ 1  5 │
  │ 2  4 │ → max = 5                  │ 3 [7]│ → max = 7
                                       
  Window (1,1): rows 2-3, cols 2-3    Window (1,2): rows 2-3, cols 4-5
  │[11] 0│                            │ 8  6 │
  │ 2   4│ → max = 11                 │ 1 [9]│ → max = 9
                                       
  Window (2,0): rows 4-5, cols 0-1    Window (2,1): rows 4-5, cols 2-3
  │ 0  3 │                            │ 5 [8]│
  │ 1 [6]│ → max = 6                  │ 4  7 │ → max = 8
                                       
  Window (2,2): rows 4-5, cols 4-5
  │ 10  2 │
  │  3 [12]│ → max = 12
```

### Step 3: Assemble Output

```
  Output Y (3×3):
  ┌────┬────┬────┐
  │  8 │  9 │  5 │
  ├────┼────┼────┤
  │  7 │ 11 │  9 │
  ├────┼────┼────┤
  │  6 │  8 │ 12 │
  └────┴────┴────┘
```

### Step 4: Backward Pass (given upstream gradient)

Suppose `dL/dY` is:

```
  ┌──────┬──────┬──────┐
  │ 0.5  │ 1.0  │ 0.3  │
  ├──────┼──────┼──────┤
  │ 0.7  │ 0.2  │ 0.8  │
  ├──────┼──────┼──────┤
  │ 0.1  │ 0.4  │ 0.6  │
  └──────┴──────┴──────┘
```

Route each gradient to the position of the max in the input:

```
  dL/dX (6×6):
  ┌──────┬──────┬──────┬──────┬──────┬──────┐
  │  0   │  0   │  0   │  0   │  0   │ 0.3  │
  ├──────┼──────┼──────┼──────┼──────┼──────┤
  │  0   │ 0.5  │  0   │ 1.0  │  0   │  0   │
  ├──────┼──────┼──────┼──────┼──────┼──────┤
  │  0   │  0   │ 0.2  │  0   │  0   │  0   │
  ├──────┼──────┼──────┼──────┼──────┼──────┤
  │  0   │ 0.7  │  0   │  0   │  0   │ 0.8  │
  ├──────┼──────┼──────┼──────┼──────┼──────┤
  │  0   │  0   │  0   │  0   │  0   │  0   │
  ├──────┼──────┼──────┼──────┼──────┼──────┤
  │  0   │ 0.1  │  0   │  0   │  0   │ 0.6  │
  └──────┴──────┴──────┴──────┴──────┴──────┘
```

---

## 1.7 Python / PyTorch Implementation

### Basic Usage — `nn.MaxPool2d`

```python
import torch
import torch.nn as nn

# ─── Define the layer ───────────────────────────────────────
pool = nn.MaxPool2d(kernel_size=2, stride=2)

# ─── Create a sample input: batch=1, channels=1, H=4, W=4 ─
x = torch.tensor([[[[1., 3., 2., 8.],
                     [5., 6., 4., 1.],
                     [3., 9., 5., 7.],
                     [2., 1., 6., 3.]]]])

print(f"Input shape:  {x.shape}")   # torch.Size([1, 1, 4, 4])

y = pool(x)
print(f"Output shape: {y.shape}")   # torch.Size([1, 1, 2, 2])
print(f"Output:\n{y}")
# tensor([[[[ 6.,  8.],
#           [ 9.,  7.]]]])
```

### Extracting Max Indices (for Gradient Visualization)

```python
# return_indices=True gives the flattened index of each max element
pool_with_idx = nn.MaxPool2d(kernel_size=2, stride=2, return_indices=True)

y, indices = pool_with_idx(x)
print(f"Max values:\n{y}")
print(f"Max indices (flattened):\n{indices}")
# indices tensor([[[[ 5,  3],
#                   [ 9, 11]]]])
# 5 → row=1, col=1 (value 6) in the 4×4 input
# 3 → row=0, col=3 (value 8)
# 9 → row=2, col=1 (value 9)
# 11 → row=2, col=3 (value 7)
```

### Max Unpooling (Inverse Operation)

```python
# MaxUnpool2d uses stored indices to place values back
unpool = nn.MaxUnpool2d(kernel_size=2, stride=2)

x_reconstructed = unpool(y, indices, output_size=x.shape)
print(f"Unpooled:\n{x_reconstructed}")
# tensor([[[[ 0.,  0.,  0.,  8.],
#           [ 0.,  6.,  0.,  0.],
#           [ 0.,  9.,  0.,  7.],
#           [ 0.,  0.,  0.,  0.]]]])
```

### Max Pooling in a CNN Block

```python
class ConvBlock(nn.Module):
    """Conv → BatchNorm → ReLU → MaxPool"""
    def __init__(self, in_ch, out_ch):
        super().__init__()
        self.block = nn.Sequential(
            nn.Conv2d(in_ch, out_ch, kernel_size=3, padding=1),
            nn.BatchNorm2d(out_ch),
            nn.ReLU(inplace=True),
            nn.MaxPool2d(kernel_size=2, stride=2)
        )

    def forward(self, x):
        return self.block(x)

# Trace shapes through the block
model = ConvBlock(3, 64)
x = torch.randn(1, 3, 32, 32)
y = model(x)
print(f"Input:  {x.shape}")   # [1, 3, 32, 32]
print(f"Output: {y.shape}")   # [1, 64, 16, 16]  ← spatial halved
```

### Verifying Backprop Gradient Routing

```python
x = torch.tensor([[[[1., 3., 2., 8.],
                     [5., 6., 4., 1.],
                     [3., 9., 5., 7.],
                     [2., 1., 6., 3.]]]], requires_grad=True)

pool = nn.MaxPool2d(kernel_size=2, stride=2)
y = pool(x)

# Simulate an upstream gradient
y.backward(torch.ones_like(y))

print("Input gradient (gradient routing):")
print(x.grad)
# tensor([[[[0., 0., 0., 1.],
#           [0., 1., 0., 0.],
#           [0., 1., 0., 1.],
#           [0., 0., 0., 0.]]]])
# Gradients appear only at max positions
```

---

## 1.8 Common Configurations

| Configuration | Kernel | Stride | Padding | Effect |
|---------------|--------|--------|---------|--------|
| **Standard** | 2×2 | 2 | 0 | Halves spatial dims (most common) |
| **Overlapping** | 3×3 | 2 | 0 | Reduces dims with overlap (AlexNet) |
| **Same-size** | 3×3 | 1 | 1 | Smooths features, keeps dims |
| **Aggressive** | 4×4 | 4 | 0 | Quarters spatial dims |

---

## 1.9 Applications and Practical Insights

### Where Max Pooling Excels

1. **Image Classification** — VGG-16 uses 5 max-pool layers, each halving spatial dims from 224→112→56→28→14→7.
2. **Feature Pyramid Networks** — Max pooling in top-down pathways.
3. **Encoder-Decoder Architectures** — SegNet stores max-pool indices for precise upsampling.
4. **Texture Recognition** — Max pooling preserves strong texture responses.

### When to Be Cautious

- **Dense prediction tasks** (segmentation, depth estimation): Max pooling discards spatial information. Consider dilated convolutions or strided convolutions instead.
- **Very small feature maps**: Applying max pooling to a 2×2 map produces 1×1 — too aggressive.
- **Tiny objects**: Important features might be in non-max positions and get discarded.

### Historical Note

> Max pooling was used as early as Fukushima's **Neocognitron** (1980). It gained mainstream adoption through Krizhevsky et al.'s **AlexNet** (2012), which used overlapping max pooling (3×3, stride 2) and showed it reduced overfitting compared to non-overlapping pooling.

---

## 1.10 Summary Table

| Aspect | Detail |
|--------|--------|
| **Operation** | Take the maximum value in each pooling window |
| **Typical Config** | 2×2 kernel, stride 2, no padding |
| **Spatial Effect** | Halves H and W |
| **Channel Effect** | None — operates per-channel independently |
| **Parameters** | 0 (no learnable weights) |
| **Gradient Rule** | Route gradient only to the max element; others get 0 |
| **Key Benefit** | Translation invariance + dimensionality reduction |
| **Key Drawback** | Discards non-max information (lossy) |
| **PyTorch Class** | `nn.MaxPool2d(kernel_size, stride, padding)` |
| **Inverse** | `nn.MaxUnpool2d` (requires stored indices) |

---

## 1.11 Revision Questions

**Q1.** Given a feature map of size `(1, 64, 28, 28)`, what is the output shape after `nn.MaxPool2d(kernel_size=2, stride=2)`?

<details><summary>Answer</summary>

`(1, 64, 14, 14)` — Spatial dims halved, batch and channels unchanged.

</details>

**Q2.** Max pooling has zero learnable parameters. How does gradient still flow backwards through it?

<details><summary>Answer</summary>

During the forward pass, the index of the maximum element in each window is recorded. During backpropagation, the upstream gradient is routed entirely to that maximum element's position, while all other positions receive zero gradient. This is analogous to how ReLU routes gradients.

</details>

**Q3.** Why does max pooling provide translation invariance? Give a concrete 2×2 example.

<details><summary>Answer</summary>

If a strong activation (say value 9) shifts from position (0,1) to (1,0) within a 2×2 window, `max(1, 9, 3, 2) = 9` and `max(1, 3, 9, 2) = 9` — the output is identical regardless of the shift. The network cannot distinguish small translations within the pooling window.

</details>

**Q4.** What is the difference between `nn.MaxPool2d(3, stride=2)` and `nn.MaxPool2d(2, stride=2)`? When might you prefer the overlapping version?

<details><summary>Answer</summary>

The 3×3 version uses overlapping windows (each element is covered by multiple windows), producing slightly larger output and smoother downsampling. AlexNet used this configuration and found it reduced top-1 error by ~0.4% compared to non-overlapping pooling, likely because it captures more context.

</details>

**Q5.** In a segmentation network like SegNet, why are max-pool indices stored during the forward pass?

<details><summary>Answer</summary>

SegNet uses an encoder-decoder architecture where the decoder must upsample feature maps back to the original resolution. By storing the indices of maximum elements during encoding, the decoder can use `MaxUnpool2d` to place activations at their original spatial positions, producing sharper segmentation boundaries than naive upsampling (e.g., bilinear interpolation).

</details>

**Q6.** A colleague proposes using max pooling with `kernel_size=2, stride=1, padding=0` on a `(1, 32, 8, 8)` tensor. What is the output shape, and what does this configuration achieve?

<details><summary>Answer</summary>

Output: `(1, 32, 7, 7)` — spatial dims reduce by 1. This configuration provides local maximum smoothing with minimal downsampling. It's rarely used in practice because it doesn't significantly reduce spatial dimensions but adds computational overhead. The more common `stride=2` halves the dimensions.

</details>

---

> ← Previous: *Start of Unit* | [Next: Average Pooling →](./02-average-pooling.md)
>
> [↑ Back to Pooling Layers Overview](./README.md)
