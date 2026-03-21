# 🔲 Convolutional Layer — The Core Building Block of CNNs

[📑 Unit Overview](../README.md) | **Next →** [02 — Activation Layer](02-activation-layer.md)

---

## 📋 Chapter Overview

The **convolutional layer** is the heart of every CNN. It applies a set of learnable filters (kernels) to the input, producing **feature maps** that capture spatial patterns — edges, textures, shapes, and eventually high-level object parts. Understanding its parameters, arithmetic, and design trade-offs is essential for building and debugging any vision model.

**What you'll learn:**
- Every parameter of `nn.Conv2d` and what it controls
- How to compute output shapes and parameter counts by hand
- Receptive field theory and why it matters
- Grouped convolutions and their efficiency gains
- Practical PyTorch code with worked examples

---

## 1️⃣ What Does a Convolutional Layer Do?

A convolutional layer slides a small learnable filter across the spatial dimensions of its input, computing element-wise multiplications and summing the results at each position. This produces a 2D activation map for each filter.

```
INPUT (C_in × H × W)          KERNEL (C_in × k × k)          OUTPUT (1 × H' × W')
┌───────────────────┐          ┌───────────┐                   ┌────────────────┐
│ . . . . . . . . . │          │ w w w     │     slide &       │ o o o o o o    │
│ . . . . . . . . . │   ×      │ w w w     │  ──────────►      │ o o o o o o    │
│ . . . . . . . . . │          │ w w w     │     sum            │ o o o o o o    │
│ . . . . . . . . . │          └───────────┘                   │ o o o o o o    │
│ . . . . . . . . . │                                          └────────────────┘
└───────────────────┘
        ↕                              ↕                              ↕
  All C_in channels           Filter spans all              One output channel
  of the input                C_in input channels           (repeat for C_out filters)
```

**Key Insight:** Each output channel is produced by one filter. If we want `C_out` output channels, we need `C_out` independent filters, each of size `C_in × k_h × k_w`.

---

## 2️⃣ `nn.Conv2d` — Full Parameter Reference

```python
torch.nn.Conv2d(
    in_channels,   # C_in  — number of channels in the input
    out_channels,  # C_out — number of filters (= number of output channels)
    kernel_size,   # k or (k_h, k_w) — spatial size of each filter
    stride=1,      # s or (s_h, s_w) — step size when sliding
    padding=0,     # p or (p_h, p_w) — zero-padding added to input borders
    dilation=1,    # d or (d_h, d_w) — spacing between kernel elements
    groups=1,      # g — split input/output channels into g groups
    bias=True,     # whether to add a learnable bias per output channel
    padding_mode='zeros'  # 'zeros', 'reflect', 'replicate', 'circular'
)
```

### 2.1 `in_channels` (C_in)

The number of channels in the input tensor. For the first layer this matches the image channels (1 for grayscale, 3 for RGB). For subsequent layers it equals the `out_channels` of the previous convolutional layer.

### 2.2 `out_channels` (C_out)

The number of filters the layer learns. Each filter produces one output feature map. More filters = richer feature extraction but more parameters and compute.

```
Typical progression in a CNN:
Layer 1:  3 → 64    (RGB to 64 feature maps)
Layer 2:  64 → 128  (double channels, halve spatial)
Layer 3:  128 → 256
Layer 4:  256 → 512
```

### 2.3 `kernel_size`

The spatial extent of each filter. Can be an integer (square kernel) or a tuple (rectangular kernel).

```
kernel_size=3  →  3×3 filter   (most common; VGG, ResNet)
kernel_size=5  →  5×5 filter   (LeNet, early layers)
kernel_size=7  →  7×7 filter   (ResNet first layer)
kernel_size=1  →  1×1 filter   (channel mixing; Inception, bottleneck)
kernel_size=(1,7)              (factorized; Inception v3)
```

**Why 3×3 is king:** Two stacked 3×3 convolutions have the same receptive field as one 5×5 but fewer parameters (2 × 9 = 18 vs 25) and two non-linearities, making the function more expressive (VGGNet insight).

### 2.4 `stride`

Controls how many pixels the filter moves per step. `stride=1` produces the densest output; `stride=2` halves spatial dimensions (used instead of pooling in some modern architectures).

```
stride = 1                        stride = 2
┌─────────────────┐               ┌─────────────────┐
│[x x x]. . . . .│               │[x x x]. . . . .│
│[x x x]. . . . .│               │[x x x]. . . . .│
│[x x x]. . . . .│               │[x x x]. . . . .│
│ . . . . . . . .│               │ . . . . . . . .│
│ . . . . . . . .│               │ . . . . . . . .│
└─────────────────┘               └─────────────────┘
   ↓ next position                   ↓ next position (skip 1)
┌─────────────────┐               ┌─────────────────┐
│ .[x x x]. . . .│               │ . .[x x x]. . .│
│ .[x x x]. . . .│               │ . .[x x x]. . .│
│ .[x x x]. . . .│               │ . .[x x x]. . .│
│ . . . . . . . .│               │ . . . . . . . .│
│ . . . . . . . .│               │ . . . . . . . .│
└─────────────────┘               └─────────────────┘
  Moves 1 pixel right               Moves 2 pixels right
```

### 2.5 `padding`

Adds zeros (or reflected/replicated values) around the input border so the filter can process edge pixels.

```
padding=0  (valid convolution)  → output shrinks
padding=k//2  (same convolution, when stride=1)  → output same size as input
    For k=3: padding=1
    For k=5: padding=2
    For k=7: padding=3
```

**"Same" padding formula (stride=1):**

```
p = (k - 1) / 2     (works for odd k)
```

### 2.6 `dilation`

Inserts gaps between kernel elements, expanding the receptive field without increasing parameters.

```
dilation=1 (standard)         dilation=2 (atrous)
┌───────┐                     ┌─────────────┐
│ w . w . w │ ← effective     │ w . . w . . w │
│ . . . . . │    3×3 kernel   │ . . . . . . . │
│ w . w . w │    but covers   │ . . . . . . . │
│ . . . . . │    5×5 area     │ w . . w . . w │
│ w . w . w │                 │ . . . . . . . │
└───────┘                     │ . . . . . . . │
                              │ w . . w . . w │
                              └─────────────────┘
  Effective kernel size = k + (k-1)(d-1)
  For k=3, d=2: effective = 3 + 2×1 = 5
```

### 2.7 `bias`

When `True` (default), adds one learnable scalar per output channel. In practice, bias is often set to `False` when using Batch Normalization right after the conv layer, because BN has its own learnable shift parameter (β) that makes the conv bias redundant.

```python
# Common pattern: Conv → BN → ReLU
nn.Conv2d(64, 128, 3, padding=1, bias=False),  # no bias needed
nn.BatchNorm2d(128),
nn.ReLU(inplace=True),
```

### 2.8 `groups`

Splits both input and output channels into `g` separate groups. Each group is convolved independently.

```
groups=1 (standard):
  Every output channel sees ALL input channels
  Filter shape: C_out × C_in × k × k

groups=C_in=C_out (depthwise):
  Each channel convolved independently
  Filter shape: C_out × 1 × k × k
  Used in MobileNet, EfficientNet

groups=2 (AlexNet split):
  Input split into 2 halves → convolved separately → concatenated
```

```
Standard Conv (groups=1)            Depthwise Conv (groups=C)
┌──────────────────────┐            ┌──────────────────────┐
│  All C_in channels   │            │  Ch1   Ch2   Ch3     │
│  ──────────────────  │            │  ───   ───   ───     │
│   ↓    ↓    ↓    ↓   │            │   ↓     ↓     ↓      │
│  [F1] [F2] [F3] [F4] │            │  [F1]  [F2]  [F3]   │
│   ↓    ↓    ↓    ↓   │            │   ↓     ↓     ↓      │
│  Out1 Out2 Out3 Out4 │            │  Out1  Out2  Out3   │
└──────────────────────┘            └──────────────────────┘
Each filter sees all channels       Each filter sees 1 channel
```

### 2.9 `padding_mode`

| Mode | Description | Use Case |
|------|-------------|----------|
| `'zeros'` | Pad with 0s (default) | General purpose |
| `'reflect'` | Mirror-pad from edge | Reduces border artifacts |
| `'replicate'` | Repeat edge pixels | Image inpainting |
| `'circular'` | Wrap-around padding | Periodic signals |

---

## 3️⃣ Output Shape Formula

For an input of shape `(N, C_in, H_in, W_in)`:

```
              ┌ H_in + 2p - d(k - 1) - 1     ┐
    H_out  =  │ ─────────────────────────── + 1│
              └            s                   ┘  (floor)

              ┌ W_in + 2p - d(k - 1) - 1     ┐
    W_out  =  │ ─────────────────────────── + 1│
              └            s                   ┘  (floor)
```

**Simplified (dilation=1):**

```
    H_out = floor( (H_in + 2p - k) / s + 1 )
```

**Quick reference for common configs (dilation=1):**

| kernel | stride | padding | Output size (from H_in) |
|--------|--------|---------|-------------------------|
| 3 | 1 | 0 | H_in − 2 |
| 3 | 1 | 1 | H_in (same) |
| 3 | 2 | 1 | ⌊(H_in − 1)/2⌋ + 1 ≈ H_in/2 |
| 5 | 1 | 2 | H_in (same) |
| 7 | 2 | 3 | ⌊(H_in − 1)/2⌋ + 1 ≈ H_in/2 |
| 1 | 1 | 0 | H_in (always same) |

---

## 4️⃣ Parameter Count

### With bias:

```
Parameters = C_out × (C_in / g × k_h × k_w) + C_out
             ─────   ──────────────────────    ─────
             filters  weights per filter        biases
```

### Without bias:

```
Parameters = C_out × (C_in / g × k_h × k_w)
```

### Worked Example 1: Standard Convolution

```
nn.Conv2d(in_channels=3, out_channels=64, kernel_size=3, bias=True)

Weights: 64 × (3 × 3 × 3) = 64 × 27 = 1,728
Biases:  64
Total:   1,728 + 64 = 1,792 parameters
```

### Worked Example 2: Depthwise Convolution

```
nn.Conv2d(in_channels=64, out_channels=64, kernel_size=3, groups=64, bias=False)

Weights: 64 × (64/64 × 3 × 3) = 64 × 9 = 576
Biases:  0
Total:   576 parameters

Compare standard: 64 × (64 × 3 × 3) = 36,864  →  64× fewer params!
```

### Worked Example 3: 1×1 Convolution (Pointwise)

```
nn.Conv2d(in_channels=256, out_channels=64, kernel_size=1, bias=False)

Weights: 64 × (256 × 1 × 1) = 16,384
Total:   16,384 parameters

This is essentially a linear projection across channels at each spatial location.
```

### Verify with PyTorch:

```python
import torch.nn as nn

conv = nn.Conv2d(3, 64, kernel_size=3, bias=True)
total = sum(p.numel() for p in conv.parameters())
print(f"Total parameters: {total}")  # 1792

# Breakdown
print(f"Weight shape: {conv.weight.shape}")  # [64, 3, 3, 3]
print(f"Bias shape:   {conv.bias.shape}")    # [64]
print(f"Weight params: {conv.weight.numel()}")  # 1728
print(f"Bias params:   {conv.bias.numel()}")    # 64
```

---

## 5️⃣ Receptive Field

The **receptive field** of a neuron is the region in the original input image that influences its activation.

### Why it matters

- Early layers have small receptive fields → detect local features (edges, corners)
- Deeper layers have larger receptive fields → detect global patterns (objects, scenes)
- The receptive field must be large enough to "see" the objects you want to classify

### Computing Receptive Field

For a stack of L layers, each with kernel size `k_l` and stride `s_l`:

```
Receptive Field at layer L:

    RF_L = RF_{L-1} + (k_L - 1) × ∏_{i=1}^{L-1} s_i

Starting: RF_0 = 1 (single pixel)
```

**Simplified (all strides = 1):**

```
RF_L = RF_{L-1} + (k_L - 1)

For N layers of 3×3 conv (stride 1):
    RF = 1 + N × (3 - 1) = 1 + 2N

    3 layers of 3×3 → RF = 7×7    (same as one 7×7)
    5 layers of 3×3 → RF = 11×11
```

### Worked Example: VGG-style Stack

```
Layer 1: Conv 3×3, stride 1    RF = 1 + (3-1)×1  = 3
Layer 2: Conv 3×3, stride 1    RF = 3 + (3-1)×1  = 5
Pool:    MaxPool 2×2, stride 2  RF = 5 + (2-1)×1  = 6  (but jump = 2 now)
Layer 3: Conv 3×3, stride 1    RF = 6 + (3-1)×2  = 10
Layer 4: Conv 3×3, stride 1    RF = 10 + (3-1)×2 = 14
Pool:    MaxPool 2×2, stride 2  RF = 14 + (2-1)×2 = 16 (jump = 4 now)

After 4 conv + 2 pool: each neuron "sees" a 16×16 region of the original image.
```

### ASCII Diagram — Growing Receptive Field

```
Original Input (pixels)
┌──────────────────────────────────────┐
│                                      │
│    ┌────────────────────────┐        │
│    │   Layer 3 RF (10×10)   │        │
│    │                        │        │
│    │   ┌──────────────┐     │        │
│    │   │ Layer 2 RF   │     │        │
│    │   │   (5×5)      │     │        │
│    │   │  ┌──────┐    │     │        │
│    │   │  │L1 RF │    │     │        │
│    │   │  │(3×3) │    │     │        │
│    │   │  └──────┘    │     │        │
│    │   └──────────────┘     │        │
│    └────────────────────────┘        │
│                                      │
└──────────────────────────────────────┘
```

---

## 6️⃣ The Convolution Math — Forward Pass

For a single output pixel at position (i, j) in output channel `m`:

```
                C_in   k_h   k_w
    y[m, i, j] = Σ     Σ     Σ   W[m, c, p, q] · x[c, i·s+p, j·s+q]  +  b[m]
                c=0   p=0   q=0
```

Where:
- `W` is the weight tensor of shape `(C_out, C_in/g, k_h, k_w)`
- `x` is the (padded) input of shape `(C_in, H_in, W_in)`
- `b` is the bias vector of shape `(C_out,)`
- `s` is the stride

### Worked Example: Tiny 2D Convolution

```
Input (1 channel, 4×4):          Kernel (3×3):
┌──────────────┐                 ┌─────────┐
│ 1  2  3  0  │                 │ 1  0  1 │
│ 0  1  2  1  │                 │ 0  1  0 │
│ 1  0  1  2  │                 │ 1  0  1 │
│ 2  1  0  1  │                 └─────────┘
└──────────────┘

stride=1, padding=0 → output is (4-3)/1 + 1 = 2×2

Position (0,0):
  1×1 + 2×0 + 3×1 + 0×0 + 1×1 + 2×0 + 1×1 + 0×0 + 1×1 = 1+3+1+1+1 = 7

Position (0,1):
  2×1 + 3×0 + 0×1 + 1×0 + 2×1 + 1×0 + 0×1 + 1×0 + 2×1 = 2+0+2+2 = 6

Position (1,0):
  0×1 + 1×0 + 2×1 + 1×0 + 0×1 + 1×0 + 2×1 + 1×0 + 0×1 = 0+2+0+2+0 = 4

Position (1,1):
  1×1 + 2×0 + 1×1 + 0×0 + 1×1 + 2×0 + 1×1 + 0×0 + 1×1 = 1+1+1+1+1 = 5

Output (2×2):
┌──────┐
│ 7  6 │
│ 4  5 │
└──────┘
```

### Verify with PyTorch:

```python
import torch
import torch.nn as nn

# Input: batch=1, channels=1, height=4, width=4
x = torch.tensor([[[[1., 2., 3., 0.],
                     [0., 1., 2., 1.],
                     [1., 0., 1., 2.],
                     [2., 1., 0., 1.]]]])

conv = nn.Conv2d(1, 1, kernel_size=3, stride=1, padding=0, bias=False)

# Manually set weights
with torch.no_grad():
    conv.weight.copy_(torch.tensor([[[[1., 0., 1.],
                                       [0., 1., 0.],
                                       [1., 0., 1.]]]]))

output = conv(x)
print(output)
# tensor([[[[7., 6.],
#           [4., 5.]]]])
```

---

## 7️⃣ Groups Parameter — Deep Dive

### Standard Convolution (groups=1)

```
Input: C_in channels ──► ALL channels feed into EACH filter ──► C_out channels

Params = C_out × C_in × k × k
```

### Grouped Convolution (groups=g)

```
Split input into g groups of C_in/g channels each.
Split output into g groups of C_out/g channels each.
Each group is convolved independently.

Params = C_out × (C_in/g) × k × k    →    1/g of standard params
```

### Visual: groups=2

```
Input (6 channels)           Filters                    Output (4 channels)
┌───┬───┬───┬───┬───┬───┐                              ┌───┬───┬───┬───┐
│ 1 │ 2 │ 3 │ 4 │ 5 │ 6 │                              │ 1 │ 2 │ 3 │ 4 │
└───┴───┴───┴───┴───┴───┘                              └───┴───┴───┴───┘
 Group 1     Group 2           Group 1     Group 2       Grp 1    Grp 2
 Ch 1-3      Ch 4-6           2 filters    2 filters    Out 1-2  Out 3-4
    ↓           ↓              (3×k×k)     (3×k×k)
 [F1, F2]    [F3, F4]           ↓            ↓
    ↓           ↓            Out 1,2      Out 3,4
    └─────┬─────┘
          ↓
   Concatenate → 4 output channels
```

### Depthwise Convolution (groups=C_in)

Each channel gets its own filter — no cross-channel interaction.

```python
# Depthwise conv: each of 64 channels filtered independently
dw_conv = nn.Conv2d(64, 64, kernel_size=3, padding=1, groups=64, bias=False)
print(dw_conv.weight.shape)  # [64, 1, 3, 3] — note C_in/g = 1

# Params: 64 × 1 × 3 × 3 = 576
# Standard would be: 64 × 64 × 3 × 3 = 36,864
```

### Depthwise Separable Convolution (MobileNet)

```python
# Step 1: Depthwise — spatial filtering per channel
dw = nn.Conv2d(64, 64, 3, padding=1, groups=64, bias=False)  # 576 params

# Step 2: Pointwise — channel mixing with 1×1 conv
pw = nn.Conv2d(64, 128, 1, bias=False)  # 64×128 = 8,192 params

# Total: 576 + 8,192 = 8,768
# Standard 64→128, 3×3: 64×128×3×3 = 73,728
# Reduction: ~8.4× fewer parameters!
```

---

## 8️⃣ Complete Worked Example — Building a Conv Block

```python
import torch
import torch.nn as nn

class ConvBlock(nn.Module):
    """Standard Conv → BN → ReLU block used in most modern CNNs."""
    
    def __init__(self, in_ch, out_ch, kernel_size=3, stride=1, padding=1):
        super().__init__()
        self.conv = nn.Conv2d(in_ch, out_ch, kernel_size, stride, padding, bias=False)
        self.bn = nn.BatchNorm2d(out_ch)
        self.relu = nn.ReLU(inplace=True)
    
    def forward(self, x):
        return self.relu(self.bn(self.conv(x)))

# Create a sample input: batch=2, channels=3, height=32, width=32
x = torch.randn(2, 3, 32, 32)

# Stack multiple conv blocks
block1 = ConvBlock(3, 64)
block2 = ConvBlock(64, 128, stride=2)  # downsamples by 2
block3 = ConvBlock(128, 256, stride=2)  # downsamples by 2 again

# Forward pass
out1 = block1(x)
out2 = block2(out1)
out3 = block3(out2)

print(f"Input:    {x.shape}")      # [2, 3, 32, 32]
print(f"Block 1:  {out1.shape}")   # [2, 64, 32, 32]
print(f"Block 2:  {out2.shape}")   # [2, 128, 16, 16]
print(f"Block 3:  {out3.shape}")   # [2, 256, 8, 8]

# Parameter counts
for name, block in [("Block1", block1), ("Block2", block2), ("Block3", block3)]:
    params = sum(p.numel() for p in block.parameters())
    print(f"{name}: {params:,} parameters")
# Block1: 1,856 params   (conv: 3×64×3×3=1,728  BN: 128)
# Block2: 74,112 params  (conv: 64×128×3×3=73,728  BN: 256+128... wait)
# Block3: 295,680 params
```

---

## 9️⃣ 1×1 Convolutions — Channel Mixing & Bottlenecks

A 1×1 convolution operates only across channels at each spatial position. Think of it as a fully-connected layer applied independently at every pixel.

```
1×1 Conv: C_in → C_out at each spatial location

Input:   (C_in,  H, W)
Weight:  (C_out, C_in, 1, 1)
Output:  (C_out, H, W)

At pixel (i,j):  out[m, i, j] = Σ_c W[m, c, 0, 0] · x[c, i, j] + b[m]
```

**Uses:**
1. **Dimensionality reduction** (bottleneck): 256→64 channels before expensive 3×3 conv
2. **Dimensionality expansion**: 64→256 channels after bottleneck
3. **Adding non-linearity** between layers without changing spatial dims
4. **Cross-channel interaction** after depthwise convolution

```python
# Bottleneck block (ResNet-style)
bottleneck = nn.Sequential(
    nn.Conv2d(256, 64, kernel_size=1, bias=False),   # reduce: 256→64
    nn.BatchNorm2d(64),
    nn.ReLU(inplace=True),
    nn.Conv2d(64, 64, kernel_size=3, padding=1, bias=False),  # spatial: 64→64
    nn.BatchNorm2d(64),
    nn.ReLU(inplace=True),
    nn.Conv2d(64, 256, kernel_size=1, bias=False),   # expand: 64→256
    nn.BatchNorm2d(256),
)
# Without bottleneck: 256×256×3×3 = 589,824 params
# With bottleneck: 16,384 + 36,864 + 16,384 = 69,632 params  (~8.5× fewer!)
```

---

## 🔟 Applications & Design Guidelines

### Where Convolutional Layers Are Used

| Application | Typical First Conv | Notes |
|------------|-------------------|-------|
| ImageNet Classification | 7×7, stride 2, 64 out | ResNet, large input 224×224 |
| CIFAR-10 | 3×3, stride 1, 16/32 out | Small 32×32 images |
| Object Detection | Pretrained backbone | YOLO, Faster R-CNN |
| Semantic Segmentation | Dilated convolutions | DeepLab series |
| Super Resolution | 3×3 or 5×5, many channels | Sub-pixel conv at end |
| GANs (Generator) | Transposed conv (deconv) | Upsampling path |

### Design Principles

1. **Double channels when halving spatial dimensions** — keeps computation roughly constant per layer
2. **Use stride-2 conv instead of pooling** in modern designs (avoids information loss)
3. **Set `bias=False` when using BatchNorm** — redundant parameters
4. **Prefer small kernels (3×3)** and stack them for larger receptive fields
5. **Use 1×1 convolutions** for efficient channel transformations

---

## 📊 Summary Table

| Property | Details |
|----------|---------|
| **Purpose** | Extract spatial features via learned filters |
| **Key params** | `in_channels`, `out_channels`, `kernel_size`, `stride`, `padding` |
| **Weight shape** | `(C_out, C_in/g, k_h, k_w)` |
| **Bias shape** | `(C_out,)` or None |
| **Output shape** | `⌊(H + 2p − d(k−1) − 1) / s + 1⌋` |
| **Param count** | `C_out × (C_in/g × k_h × k_w) + C_out` (with bias) |
| **groups=1** | Standard — all input channels to all output channels |
| **groups=C_in** | Depthwise — independent per-channel filtering |
| **Common kernel** | 3×3 (best accuracy/param trade-off) |
| **Receptive field** | Grows with depth: RF = 1 + Σ (k_i − 1) × Π s_j |
| **"Same" padding** | `p = (k − 1) / 2` for stride 1 |

---

## ❓ Revision Questions

### Q1: Parameter Count Calculation
**A Conv2d layer has `in_channels=128`, `out_channels=256`, `kernel_size=3`, `bias=True`. How many learnable parameters does it have?**

<details>
<summary>Answer</summary>

Weights = 256 × 128 × 3 × 3 = 294,912
Biases = 256
Total = 294,912 + 256 = **295,168 parameters**
</details>

### Q2: Output Shape
**Input shape is `(1, 3, 224, 224)`. Apply `nn.Conv2d(3, 64, kernel_size=7, stride=2, padding=3)`. What is the output shape?**

<details>
<summary>Answer</summary>

H_out = ⌊(224 + 2×3 − 7) / 2 + 1⌋ = ⌊(224 + 6 − 7) / 2 + 1⌋ = ⌊223/2 + 1⌋ = ⌊111.5 + 1⌋ = 112

Output shape: **(1, 64, 112, 112)**

This is exactly the first conv layer of ResNet!
</details>

### Q3: Groups and Efficiency
**Compare the parameter count of `nn.Conv2d(64, 64, 3, groups=1)` vs `nn.Conv2d(64, 64, 3, groups=64)`. By what factor is the grouped version more efficient?**

<details>
<summary>Answer</summary>

Standard (groups=1): 64 × 64 × 3 × 3 = 36,864 params
Depthwise (groups=64): 64 × 1 × 3 × 3 = 576 params

Factor = 36,864 / 576 = **64×** more efficient (equal to in_channels)
</details>

### Q4: Receptive Field
**Three consecutive Conv2d layers with `kernel_size=3, stride=1, padding=1`. What is the receptive field after all three layers?**

<details>
<summary>Answer</summary>

RF after layer 1: 1 + (3−1)×1 = 3
RF after layer 2: 3 + (3−1)×1 = 5
RF after layer 3: 5 + (3−1)×1 = **7×7**

Three 3×3 layers give the same receptive field as one 7×7 layer, but with fewer parameters (3×9C² = 27C² vs 49C²) and more non-linearities.
</details>

### Q5: Why bias=False with BatchNorm?
**Explain why `nn.Conv2d(..., bias=False)` is used when followed by BatchNorm.**

<details>
<summary>Answer</summary>

BatchNorm normalizes the output to zero mean and unit variance, then applies learned shift (β) and scale (γ). The conv bias adds a constant per channel, but BN's normalization subtracts the mean — which removes the effect of any constant bias. BN's own β parameter then serves as the effective bias. So the conv bias is **redundant** and wastes parameters.

Mathematically: BN(Conv(x) + b) = BN(Conv(x)) since the mean subtraction cancels b.
</details>

### Q6: 1×1 Convolution
**What is the computational purpose of a 1×1 convolution, and why is it used in bottleneck architectures like ResNet?**

<details>
<summary>Answer</summary>

A 1×1 convolution performs **channel-wise linear combination** at each spatial location — it mixes information across channels without spatial filtering. In bottleneck designs:
1. **Reduce** channels (256→64) with 1×1 conv to make the subsequent 3×3 conv cheaper
2. Apply expensive **3×3 spatial conv** on fewer channels (64→64)
3. **Expand** channels back (64→256) with another 1×1 conv

This reduces computation by ~8× compared to a direct 256→256 3×3 conv, with minimal accuracy loss due to the added non-linearities.
</details>

---

[📑 Unit Overview](../README.md) | **Next →** [02 — Activation Layer](02-activation-layer.md)
