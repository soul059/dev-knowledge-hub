# 2 · Average Pooling

> **Unit 3.2 – Convolutional Neural Networks / 03 – Pooling Layers**
>
> [← Previous: Max Pooling](./01-max-pooling.md) | [Next: Global Pooling →](./03-global-pooling.md)
>
> [↑ Back to Pooling Layers Overview](./README.md)

---

## 2.1 Chapter Overview

Average Pooling computes the **arithmetic mean** of all values within each pooling window. Unlike max pooling, which selects the single strongest activation, average pooling produces a **smooth, blended downsampling** that retains information about the overall magnitude of activations in each region.

Key characteristics:

- **Preserves background information** — low activations still contribute to the output.
- **Smoother gradients** — every element in the window receives an equal share of the gradient.
- **Less aggressive** — doesn't discard non-max values, making it more suitable when all spatial responses matter.

Average pooling is less common than max pooling in modern classification networks but remains important in specific architectures and is the basis for **Global Average Pooling** (covered in the next chapter).

---

## 2.2 Mathematical Definition

### Formal Notation

Given input feature map **X** of shape `(C, H_in, W_in)`, pooling window `k × k`, stride `s`, padding `p`:

```
  Output dimensions (same formula as max pooling):

            ⌊ H_in + 2p - k ⌋
  H_out  =  ⎢ ─────────────── ⎥ + 1
            ⌊       s         ⌋
```

### Pointwise Operation

For each output position `(i, j)` and channel `c`:

```
                    1      k-1  k-1
  Y[c, i, j]  =  ─────   Σ    Σ   X[c, i·s + m, j·s + n]
                  k × k   m=0  n=0
```

For the standard **k=2, s=2** configuration:

```
                    1
  Y[c, i, j]  =  ─── ( X[c, 2i, 2j] + X[c, 2i, 2j+1]
                    4
                       + X[c, 2i+1, 2j] + X[c, 2i+1, 2j+1] )
```

| Property         | Value                                     |
|------------------|-------------------------------------------|
| Learnable params | **0** — no weights, no bias               |
| Non-linearity    | **No** — averaging is a linear operation  |
| Channel mixing   | **No** — operates independently per channel |
| Typical config   | k=2, s=2, p=0 (halves spatial dims)       |

---

## 2.3 ASCII Diagram — Average Pool 2×2, Stride 2

```
  INPUT FEATURE MAP (4×4)                  OUTPUT (2×2)
 ┌─────┬─────┬─────┬─────┐
 │  2  │  4  │  6  │  8  │         ┌──────┬──────┐
 ├─────┼─────┼─────┼─────┤         │ 4.0  │ 5.0  │
 │  6  │  4  │  2  │  4  │   ──►   ├──────┼──────┤
 ├─────┼─────┼─────┼─────┤         │ 5.0  │ 5.5  │
 │  8  │  2  │  4  │  6  │         └──────┴──────┘
 ├─────┼─────┼─────┼─────┤
 │  4  │  6  │  8  │  4  │
 └─────┴─────┴─────┴─────┘

  Window calculations:

  Top-left:  (2 + 4 + 6 + 4) / 4 = 16/4 = 4.0
  Top-right: (6 + 8 + 2 + 4) / 4 = 20/4 = 5.0
  Bot-left:  (8 + 2 + 4 + 6) / 4 = 20/4 = 5.0
  Bot-right: (4 + 6 + 8 + 4) / 4 = 22/4 = 5.5
```

### Side-by-Side: Max Pool vs Average Pool

```
  INPUT (4×4):               MAX POOL (2×2):        AVG POOL (2×2):
  ┌────┬────┬────┬────┐     ┌──────┬──────┐        ┌──────┬──────┐
  │ 1  │ 0  │ 3  │ 2  │     │  5   │  3   │        │ 1.75 │ 2.25 │
  ├────┼────┼────┼────┤     ├──────┼──────┤        ├──────┼──────┤
  │ 5  │ 1  │ 1  │ 2  │     │  8   │  9   │        │ 5.00 │ 4.00 │
  ├────┼────┼────┼────┤     └──────┴──────┘        └──────┴──────┘
  │ 2  │ 8  │ 9  │ 1  │
  ├────┼────┼────┼────┤      Max preserves          Avg smooths
  │ 4  │ 3  │ 2  │ 4  │      peak activations       across all values
  └────┴────┴────┴────┘

  Notice: Max pool top-left = 5 (the peak)
          Avg pool top-left = 1.75 (the mean — much lower due to zeros)
```

---

## 2.4 Backpropagation Through Average Pooling

### Gradient Distribution

Unlike max pooling, which routes the entire gradient to a single element, average pooling **distributes the gradient equally** to all elements in the window:

```
  Forward Pass:                     Backward Pass:
  ┌─────┬─────┐                     ┌────────┬────────┐
  │  2  │  4  │                     │ dL/4   │ dL/4   │
  │  6  │  4  │ → avg = 4.0        │ dL/4   │ dL/4   │
  └─────┴─────┘                     └────────┴────────┘

  If ∂L/∂Y = dL, then for a k×k window:

      ∂L                dL       1
      ────────────── = ───── = ───── · dL    for ALL (m,n) in window
      ∂X[c, i·s+m, j·s+n]   k²     k²
```

### Formal Expression

```
  ∂L                          1       ∂L
  ──────────────────────  =  ─────  · ────
  ∂X[c, i·s + m, j·s + n]    k²     ∂Y[c, i, j]

  for all 0 ≤ m < k, 0 ≤ n < k
```

### ASCII Diagram — Backward Pass Comparison

```
  MAX POOL backward:                AVG POOL backward:
  ┌─────┬─────┐                    ┌──────┬──────┐
  │  0  │  0  │ gradient           │ g/4  │ g/4  │ gradient
  │  0  │  g  │ → to max only      │ g/4  │ g/4  │ → split equally
  └─────┴─────┘                    └──────┴──────┘

  Max: sparse gradients             Avg: dense, uniform gradients
  → sharper feature learning        → smoother optimization landscape
```

---

## 2.5 Worked Example — Step by Step

### Problem

Apply **Average Pool 2×2, stride 2** to this feature map and compute the backward pass.

```
  Input X (4×4):
  ┌─────┬─────┬─────┬─────┐
  │ 10  │  2  │  6  │  8  │
  ├─────┼─────┼─────┼─────┤
  │  4  │ 12  │  2  │  4  │
  ├─────┼─────┼─────┼─────┤
  │  8  │  6  │ 14  │  2  │
  ├─────┼─────┼─────┼─────┤
  │  2  │  4  │  6  │ 16  │
  └─────┴─────┴─────┴─────┘
```

### Step 1: Forward Pass

```
  Window (0,0): (10 + 2 + 4 + 12) / 4 = 28/4 = 7.0
  Window (0,1): (6 + 8 + 2 + 4)   / 4 = 20/4 = 5.0
  Window (1,0): (8 + 6 + 2 + 4)   / 4 = 20/4 = 5.0
  Window (1,1): (14 + 2 + 6 + 16) / 4 = 38/4 = 9.5

  Output Y (2×2):
  ┌──────┬──────┐
  │ 7.0  │ 5.0  │
  ├──────┼──────┤
  │ 5.0  │ 9.5  │
  └──────┴──────┘
```

### Step 2: Backward Pass

Given upstream gradient `dL/dY`:

```
  ┌──────┬──────┐
  │ 0.8  │ 0.4  │
  ├──────┼──────┤
  │ 1.2  │ 0.6  │
  └──────┴──────┘
```

Each gradient is divided by k² = 4 and distributed to all window positions:

```
  dL/dX (4×4):
  ┌──────┬──────┬──────┬──────┐
  │ 0.2  │ 0.2  │ 0.1  │ 0.1  │   ← 0.8/4=0.2, 0.4/4=0.1
  ├──────┼──────┼──────┼──────┤
  │ 0.2  │ 0.2  │ 0.1  │ 0.1  │
  ├──────┼──────┼──────┼──────┤
  │ 0.3  │ 0.3  │ 0.15 │ 0.15 │   ← 1.2/4=0.3, 0.6/4=0.15
  ├──────┼──────┼──────┼──────┤
  │ 0.3  │ 0.3  │ 0.15 │ 0.15 │
  └──────┴──────┴──────┴──────┘
```

---

## 2.6 Max Pooling vs Average Pooling — Detailed Comparison

| Criterion | Max Pooling | Average Pooling |
|-----------|-------------|-----------------|
| **Operation** | Takes maximum | Takes arithmetic mean |
| **Information retained** | Strongest activation only | All activations (blended) |
| **Linearity** | Non-linear (`max` function) | Linear (weighted sum) |
| **Gradient flow** | Sparse — only to max element | Dense — equal to all elements |
| **Translation invariance** | Strong local invariance | Weaker invariance |
| **Sensitivity to outliers** | High — one large value dominates | Low — outliers are averaged out |
| **Noise handling** | Preserves noise if it's the max | Suppresses noise via averaging |
| **Feature type** | Best for sparse, peaky features | Best for smooth, distributed features |
| **Typical use** | Hidden layers of classifiers | Final layer (as GAP), early processing |
| **Dominant architecture** | VGG, ResNet | Inception (mixed), NiN (GAP) |

### When to Use Each

```
  USE MAX POOLING when:
  ┌─────────────────────────────────────────────────────────┐
  │  • Features are sparse (edges, corners, textures)       │
  │  • You want strong translation invariance               │
  │  • Classification task (discriminative features needed) │
  │  • Standard ConvNet architecture (VGG, ResNet-style)    │
  └─────────────────────────────────────────────────────────┘

  USE AVERAGE POOLING when:
  ┌─────────────────────────────────────────────────────────┐
  │  • Features are smooth / distributed                    │
  │  • You want to preserve overall activation magnitude    │
  │  • Global pooling before classifier (GAP)               │
  │  • Task requires sensitivity to all spatial regions     │
  │  • Anti-aliasing before downsampling                    │
  └─────────────────────────────────────────────────────────┘
```

---

## 2.7 Python / PyTorch Implementation

### Basic Usage — `nn.AvgPool2d`

```python
import torch
import torch.nn as nn

# ─── Define average pooling layer ───────────────────────────
pool = nn.AvgPool2d(kernel_size=2, stride=2)

# ─── Create sample input: batch=1, channels=1, H=4, W=4 ───
x = torch.tensor([[[[10., 2., 6., 8.],
                     [ 4., 12., 2., 4.],
                     [ 8., 6., 14., 2.],
                     [ 2., 4., 6., 16.]]]])

print(f"Input shape:  {x.shape}")   # torch.Size([1, 1, 4, 4])

y = pool(x)
print(f"Output shape: {y.shape}")   # torch.Size([1, 1, 2, 2])
print(f"Output:\n{y}")
# tensor([[[[ 7.0000,  5.0000],
#           [ 5.0000,  9.5000]]]])
```

### Handling Padding and `count_include_pad`

```python
# With padding, border elements may be zero-padded.
# count_include_pad controls whether padding zeros are counted in the average.

pool_include = nn.AvgPool2d(kernel_size=3, stride=1, padding=1,
                             count_include_pad=True)
pool_exclude = nn.AvgPool2d(kernel_size=3, stride=1, padding=1,
                             count_include_pad=False)

x = torch.tensor([[[[1., 2., 3.],
                     [4., 5., 6.],
                     [7., 8., 9.]]]])

y_include = pool_include(x)
y_exclude = pool_exclude(x)

print("count_include_pad=True:")
print(y_include)
# Corner (0,0): (0+0+0 + 0+1+2 + 0+4+5) / 9 = 12/9 = 1.333

print("\ncount_include_pad=False:")
print(y_exclude)
# Corner (0,0): (1+2 + 4+5) / 4 = 12/4 = 3.0  ← only real elements counted
```

### Verifying Gradient Distribution

```python
x = torch.tensor([[[[2., 4.],
                     [6., 4.]]]], requires_grad=True)

pool = nn.AvgPool2d(kernel_size=2, stride=2)
y = pool(x)  # → 4.0

y.backward(torch.tensor([[[[1.0]]]]))

print("Input gradient:")
print(x.grad)
# tensor([[[[0.2500, 0.2500],
#           [0.2500, 0.2500]]]])
# Each element gets 1/4 of the upstream gradient
```

### Comparing Max vs Avg in a CNN

```python
class DualPoolCNN(nn.Module):
    """CNN with both max and average pooling branches for comparison."""
    def __init__(self, in_ch=3, num_classes=10):
        super().__init__()
        self.conv = nn.Sequential(
            nn.Conv2d(in_ch, 32, 3, padding=1),
            nn.ReLU()
        )
        self.max_branch = nn.Sequential(
            nn.MaxPool2d(2, 2),
            nn.Flatten(),
        )
        self.avg_branch = nn.Sequential(
            nn.AvgPool2d(2, 2),
            nn.Flatten(),
        )

    def forward(self, x):
        features = self.conv(x)
        max_out = self.max_branch(features)
        avg_out = self.avg_branch(features)
        return max_out, avg_out

model = DualPoolCNN()
x = torch.randn(1, 3, 8, 8)
max_out, avg_out = model(x)
print(f"Max branch output shape: {max_out.shape}")  # [1, 32*4*4] = [1, 512]
print(f"Avg branch output shape: {avg_out.shape}")  # [1, 32*4*4] = [1, 512]

# The values will differ — max is peakier, avg is smoother
print(f"Max branch — mean: {max_out.mean():.3f}, std: {max_out.std():.3f}")
print(f"Avg branch — mean: {avg_out.mean():.3f}, std: {avg_out.std():.3f}")
```

### Anti-Aliased Downsampling (Advanced)

```python
class AntiAliasedDownsample(nn.Module):
    """Average pooling before max pooling to reduce aliasing artifacts.
    Inspired by Zhang (2019) 'Making Convolutional Networks Shift-Invariant Again'.
    """
    def __init__(self):
        super().__init__()
        self.blur = nn.AvgPool2d(kernel_size=2, stride=1, padding=0)
        self.subsample = nn.MaxPool2d(kernel_size=2, stride=2)

    def forward(self, x):
        x = self.blur(x)       # Smooth
        x = self.subsample(x)  # Then subsample
        return x

aa = AntiAliasedDownsample()
x = torch.randn(1, 16, 32, 32)
y = aa(x)
print(f"Anti-aliased output: {y.shape}")  # [1, 16, 15, 15]
```

---

## 2.8 Applications and Practical Insights

### Real-World Usage

1. **Inception / GoogLeNet** — Uses average pooling branches alongside max pooling in inception modules.
2. **Global Average Pooling** — The most impactful use of average pooling, used in nearly all modern architectures (see next chapter).
3. **Signal Processing** — Average pooling acts as a low-pass filter, suppressing high-frequency noise.
4. **Medical Imaging** — Smooth features (organ boundaries, tissue textures) often benefit from average pooling.

### Practical Tips

- **Default choice:** Use max pooling in hidden layers; it typically outperforms average pooling for classification.
- **Global context:** Use Global Average Pooling at the end (see Chapter 3).
- **Anti-aliasing:** Average pooling before strided operations improves shift invariance.
- **Feature visualization:** Average pooled feature maps show "what regions are activated," while max pooled maps show "where the strongest feature is."

---

## 2.9 Summary Table

| Aspect | Detail |
|--------|--------|
| **Operation** | Arithmetic mean of all values in the pooling window |
| **Typical Config** | 2×2 kernel, stride 2, no padding |
| **Spatial Effect** | Halves H and W (same as max pool) |
| **Linearity** | Linear — averaging is a linear operation |
| **Parameters** | 0 (no learnable weights) |
| **Gradient Rule** | Gradient divided equally among all k² window elements |
| **Key Benefit** | Smooth downsampling, preserves magnitude information |
| **Key Drawback** | Dilutes strong activations, weaker feature selection |
| **PyTorch Class** | `nn.AvgPool2d(kernel_size, stride, padding)` |
| **Special Param** | `count_include_pad` — whether padding zeros count in avg |

---

## 2.10 Revision Questions

**Q1.** Compute the output of Average Pool 2×2, stride 2 on:
```
┌───┬───┬───┬───┐
│ 4 │ 8 │ 2 │ 6 │
├───┼───┼───┼───┤
│ 2 │ 6 │ 4 │ 8 │
├───┼───┼───┼───┤
│ 1 │ 3 │ 5 │ 7 │
├───┼───┼───┼───┤
│ 9 │ 7 │ 3 │ 1 │
└───┴───┴───┴───┘
```

<details><summary>Answer</summary>

```
Top-left:  (4+8+2+6)/4 = 20/4 = 5.0
Top-right: (2+6+4+8)/4 = 20/4 = 5.0
Bot-left:  (1+3+9+7)/4 = 20/4 = 5.0
Bot-right: (5+7+3+1)/4 = 16/4 = 4.0

Output:
┌─────┬─────┐
│ 5.0 │ 5.0 │
├─────┼─────┤
│ 5.0 │ 4.0 │
└─────┴─────┘
```

</details>

**Q2.** Why is average pooling considered a linear operation while max pooling is non-linear?

<details><summary>Answer</summary>

Average pooling computes a weighted sum (each element multiplied by 1/k²) which satisfies linearity: `avg(aX + bY) = a·avg(X) + b·avg(Y)`. Max pooling uses the `max` function, which does not satisfy linearity: `max(X + Y) ≠ max(X) + max(Y)` in general. This distinction affects the optimization landscape — average pooling creates smoother loss surfaces.

</details>

**Q3.** What does `count_include_pad=False` do in `nn.AvgPool2d`, and when would you use it?

<details><summary>Answer</summary>

When `count_include_pad=False`, zero-padded positions at the borders are excluded from the denominator of the average. This prevents the average from being diluted near edges. Use it when border accuracy matters (e.g., in dense prediction tasks like segmentation) or when input data has meaningful edge content.

</details>

**Q4.** Compare the gradient flow of max pooling vs average pooling. Which creates more "dead" gradients?

<details><summary>Answer</summary>

Max pooling creates more dead gradients. In a 2×2 max pool, only 1 out of 4 elements receives gradient (75% get zero). Average pooling distributes gradient equally to all 4 elements (0% get zero). This means average pooling updates all upstream weights, while max pooling only updates weights contributing to the maximum activation.

</details>

**Q5.** In what scenario would average pooling outperform max pooling for a classification task?

<details><summary>Answer</summary>

Average pooling can outperform max pooling when the discriminative information is distributed across the spatial extent rather than concentrated in peaks. For example, classifying textures (fabric types, terrain) where the overall pattern matters more than any single strong activation. Also, Global Average Pooling at the final layer consistently outperforms max pooling by providing a natural regularization effect.

</details>

**Q6.** An input of shape `(8, 256, 14, 14)` passes through `nn.AvgPool2d(kernel_size=7, stride=7)`. What is the output shape and how many elements does each output value average over?

<details><summary>Answer</summary>

Output shape: `(8, 256, 2, 2)` — since ⌊(14-7)/7⌋ + 1 = 2. Each output value is the average of 7×7 = 49 elements. This is a fairly aggressive downsampling that produces a very compact representation.

</details>

---

> [← Previous: Max Pooling](./01-max-pooling.md) | [Next: Global Pooling →](./03-global-pooling.md)
>
> [↑ Back to Pooling Layers Overview](./README.md)
