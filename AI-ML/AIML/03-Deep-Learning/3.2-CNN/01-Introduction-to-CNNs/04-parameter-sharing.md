# 🔗 1.4 — Parameter Sharing in CNNs

> **Unit:** 01 – Introduction to CNNs
> **Chapter:** 4 of 5 — The Weight Sharing Revolution

---

[← Previous: Local Connectivity](./03-local-connectivity.md) · [🏠 Unit Overview](./README.md) · [Next → Translation Invariance](./05-translation-invariance.md)

---

## 📋 Chapter Overview

Parameter sharing is the second pillar of CNN design (alongside local connectivity). While local connectivity reduces connections by making them local, parameter sharing goes further — **the same filter weights are used at every spatial position**. This chapter explores why this works, the dramatic parameter savings it produces, and how it enables CNNs to learn position-independent feature detectors.

**After this chapter you will be able to:**

1. Explain the difference between locally connected and convolutional layers.
2. Calculate exact parameter counts for FC, locally connected, and convolutional layers.
3. Articulate why a shared filter is a universal feature detector.
4. Describe what each filter in a convolutional layer "learns."
5. Implement and visualise weight sharing in PyTorch.

---

## 1 — What Is Parameter Sharing?

### 1.1 — Core Concept

In a standard convolution, a **single filter** (a small matrix of learnable weights) is slid across the entire input image. At every position, the **same weights** are used to compute the output.

```
Filter W (3×3):
    ┌─────────┐
    │ w₁ w₂ w₃│
    │ w₄ w₅ w₆│         Same 9 weights used EVERYWHERE
    │ w₇ w₈ w₉│
    └─────────┘

Position (0,0):                  Position (0,1):
    ┌─────────────────┐              ┌─────────────────┐
    │[w₁ w₂ w₃]x x x │              │ x[w₁ w₂ w₃]x x │
    │[w₄ w₅ w₆]x x x │              │ x[w₄ w₅ w₆]x x │
    │[w₇ w₈ w₉]x x x │              │ x[w₇ w₈ w₉]x x │
    │ x  x  x  x x x │              │ x  x  x  x x x │
    │ x  x  x  x x x │              │ x  x  x  x x x │
    └─────────────────┘              └─────────────────┘

    Same w₁...w₉ at both positions!
```

### 1.2 — Three Levels of Connectivity

| Type | Connections | Parameter Sharing? | Params (for one output map) |
|---|---|---|---|
| **Fully Connected** | Every input → every output | ❌ No | H_in × W_in × C_in |
| **Locally Connected** | Local region → output | ❌ No (unique weights per position) | K² × C_in × H_out × W_out |
| **Convolutional** | Local region → output | ✅ Yes (same weights everywhere) | K² × C_in |

### 1.3 — ASCII Diagram: Three Connectivity Types

```
FULLY CONNECTED (all-to-all, unique weights):

Input: a b c d e f g h i     (9 values)
        ╲│╲│╲│╲│╱│╱│╱│╱
         ╲│ ╲│ ╲│╱ │╱        ← 9 unique weights per output neuron
          ●   ●   ●

Parameters per output: 9    Total for 3 outputs: 27 weights

────────────────────────────────────────────────

LOCALLY CONNECTED (local, unique weights per position):

Input: a b c d e f g h i
        ╲│╱     ╲│╱   ╲│╱
        {α}     {β}   {γ}    ← DIFFERENT weights at each position
         ●       ●     ●

α = {α₁,α₂,α₃}, β = {β₁,β₂,β₃}, γ = {γ₁,γ₂,γ₃}
Parameters: 3 + 3 + 3 = 9 weights (each position has own weights)

────────────────────────────────────────────────

CONVOLUTIONAL (local, SHARED weights):

Input: a b c d e f g h i
        ╲│╱     ╲│╱   ╲│╱
        {w}     {w}   {w}    ← SAME weights at every position
         ●       ●     ●

w = {w₁, w₂, w₃}
Parameters: 3 weights total (shared across all positions)
```

---

## 2 — Parameter Count Comparison: The Numbers

### 2.1 — The Mathematical Formulation

For an input of size **H_in × W_in × C_in** producing an output of size **H_out × W_out × C_out** using kernel size **K × K**:

```
Fully Connected Layer:
  Params = (H_in × W_in × C_in) × (H_out × W_out × C_out) + (H_out × W_out × C_out)
  
Locally Connected Layer:
  Params = (K × K × C_in) × (H_out × W_out × C_out) + (H_out × W_out × C_out)
  
Convolutional Layer:
  Params = (K × K × C_in) × C_out + C_out
```

> **The key difference:** Conv params depend only on K, C_in, C_out — independent of spatial dimensions.

### 2.2 — Worked Example: CIFAR-10 (32×32×3)

```
Input: 32 × 32 × 3 = 3,072 values
Output: 32 × 32 × 64 (64 feature maps, same spatial size with padding)
Kernel: 3 × 3

╔═══════════════════════╦═══════════════════════╦═══════════════╗
║ Layer Type            ║ Parameter Formula     ║ Count         ║
╠═══════════════════════╬═══════════════════════╬═══════════════╣
║ Fully Connected       ║ 3072 × 65536 + 65536 ║ 201,392,128   ║
║ Locally Connected     ║ 27 × 65536 + 65536   ║ 1,835,008     ║
║ Convolutional         ║ 27 × 64 + 64         ║ 1,792         ║
╠═══════════════════════╬═══════════════════════╬═══════════════╣
║ FC / Conv ratio       ║                       ║ 112,400×      ║
║ Local / Conv ratio    ║                       ║ 1,024×        ║
╚═══════════════════════╩═══════════════════════╩═══════════════╝
```

### 2.3 — Worked Example: ImageNet (224×224×3)

```
Input: 224 × 224 × 3 = 150,528 values
First layer: 64 filters of 3×3 (output: 224 × 224 × 64 with padding)
H_out × W_out = 224 × 224 = 50,176

╔═══════════════════════╦══════════════════════════════════╦════════════════╗
║ Layer Type            ║ Calculation                      ║ Parameters     ║
╠═══════════════════════╬══════════════════════════════════╬════════════════╣
║ Fully Connected       ║ 150,528 × 3,211,264 + 3,211,264 ║ ~483 BILLION   ║
║ Locally Connected     ║ 27 × 3,211,264 + 3,211,264      ║ ~89.9 million  ║
║ Convolutional         ║ 27 × 64 + 64                     ║ 1,792          ║
╠═══════════════════════╬══════════════════════════════════╬════════════════╣
║ FC / Conv ratio       ║                                  ║ ~269 million×  ║
║ Local / Conv ratio    ║                                  ║ ~50,176×       ║
╚═══════════════════════╩══════════════════════════════════╩════════════════╝

The locally connected layer has as many weights as positions (50,176)
times the conv weights. Parameter sharing reduces this by a factor
equal to the number of output spatial positions.
```

### 2.4 — PyTorch Code: Parameter Count Verification

```python
import torch
import torch.nn as nn

# Input dimensions
C_in, H, W = 3, 32, 32
C_out = 64
K = 3

# CONVOLUTIONAL layer (weight sharing)
conv = nn.Conv2d(C_in, C_out, K, padding=1)
conv_params = sum(p.numel() for p in conv.parameters())

# FULLY CONNECTED layer (no sharing, no locality)
fc = nn.Linear(C_in * H * W, C_out * H * W)
fc_params = sum(p.numel() for p in fc.parameters())

# LOCALLY CONNECTED — PyTorch doesn't have a built-in layer,
# but we can calculate it:
locally_connected_params = K * K * C_in * C_out * H * W + C_out * H * W

print(f"Input: {C_in}×{H}×{W} → Output: {C_out}×{H}×{W}")
print(f"Kernel: {K}×{K}")
print(f"{'─' * 50}")
print(f"Fully Connected  : {fc_params:>15,} parameters")
print(f"Locally Connected: {locally_connected_params:>15,} parameters")
print(f"Convolutional    : {conv_params:>15,} parameters")
print(f"{'─' * 50}")
print(f"FC / Conv ratio  : {fc_params / conv_params:,.0f}×")
print(f"LC / Conv ratio  : {locally_connected_params / conv_params:,.0f}×")

# Inspect conv layer internals
print(f"\nConv weight shape: {conv.weight.shape}")  # (64, 3, 3, 3)
print(f"Conv bias shape  : {conv.bias.shape}")      # (64,)
print(f"Weight tensor: {K}×{K}×{C_in}×{C_out} = {K*K*C_in*C_out}")
print(f"Bias vector:   {C_out}")
print(f"Total: {K*K*C_in*C_out + C_out}")
```

---

## 3 — How Weight Sharing Works: Step by Step

### 3.1 — The Convolution Operation

For a single filter W of size K×K applied to a single-channel input X to produce output Y:

```
Y[m, n] = Σ_k Σ_l W[k, l] × X[m+k, n+l] + b

Where:
  m, n = output spatial position
  k, l = filter indices (0 to K-1)
  b    = bias (one scalar per filter)
```

### 3.2 — Worked Example: 2-D Convolution

```
Input X (5×5):                    Filter W (3×3):
┌──────────────────────┐          ┌───────────┐
│  1   2   3   4   5   │          │  1   0  -1 │
│  6   7   8   9  10   │          │  1   0  -1 │
│ 11  12  13  14  15   │          │  1   0  -1 │
│ 16  17  18  19  20   │          └───────────┘
│ 21  22  23  24  25   │          Bias b = 0
└──────────────────────┘

Output Y (3×3):

Y[0,0] = 1×1 + 2×0 + 3×(-1) + 6×1 + 7×0 + 8×(-1) + 11×1 + 12×0 + 13×(-1)
       = 1 + 0 - 3 + 6 + 0 - 8 + 11 + 0 - 13
       = -6

Y[0,1] = 2×1 + 3×0 + 4×(-1) + 7×1 + 8×0 + 9×(-1) + 12×1 + 13×0 + 14×(-1)
       = 2 + 0 - 4 + 7 + 0 - 9 + 12 + 0 - 14
       = -6

Y[0,2] = 3×1 + 4×0 + 5×(-1) + 8×1 + 9×0 + 10×(-1) + 13×1 + 14×0 + 15×(-1)
       = 3 + 0 - 5 + 8 + 0 - 10 + 13 + 0 - 15
       = -6

Full output Y:
┌──────────────┐
│ -6   -6   -6 │       ← Constant! This filter detects vertical edges.
│ -6   -6   -6 │          Since the input has a uniform left-to-right
│ -6   -6   -6 │          gradient, the response is uniform.
└──────────────┘

THE SAME W is used for ALL 9 output positions.
```

### 3.3 — Multi-Channel Convolution

For C_in input channels and C_out output filters:

```
For output channel j:
  Y_j[m, n] = Σ(c=0 to C_in-1) Σ_k Σ_l W_j[c, k, l] × X[c, m+k, n+l] + b_j

Each filter W_j has shape (C_in, K, K):
  - It spans ALL input channels
  - But only a K×K spatial region

Total weights for all C_out filters:
  = C_out × C_in × K × K + C_out (biases)
```

### 3.4 — ASCII Diagram: Multi-Channel Convolution

```
Input X (3 channels):           Filter W₁ (3 channels):         Output Y₁:
┌──────┐ ┌──────┐ ┌──────┐     ┌────┐ ┌────┐ ┌────┐
│  R   │ │  G   │ │  B   │     │ Wr │ │ Wg │ │ Wb │          ┌──────────┐
│ 5×5  │ │ 5×5  │ │ 5×5  │  *  │3×3 │ │3×3 │ │3×3 │   →      │  3×3     │
│      │ │      │ │      │     └────┘ └────┘ └────┘           │ feature  │
└──────┘ └──────┘ └──────┘                                     │   map    │
                                One filter produces             └──────────┘
                                ONE output channel.

For 64 output channels:
  64 filters × (3 input channels × 3×3 kernel) = 64 × 27 = 1,728 weights
  + 64 biases = 1,792 total parameters

Each of the 64 filters detects a DIFFERENT feature,
but each filter is shared across ALL spatial positions.
```

### 3.5 — PyTorch Code: Convolution Step by Step

```python
import torch
import torch.nn as nn
import torch.nn.functional as F

# Single channel 5×5 input
X = torch.tensor([
    [1,  2,  3,  4,  5],
    [6,  7,  8,  9,  10],
    [11, 12, 13, 14, 15],
    [16, 17, 18, 19, 20],
    [21, 22, 23, 24, 25],
], dtype=torch.float32).reshape(1, 1, 5, 5)  # (N, C, H, W)

# Vertical edge detection filter
W = torch.tensor([
    [1,  0, -1],
    [1,  0, -1],
    [1,  0, -1],
], dtype=torch.float32).reshape(1, 1, 3, 3)  # (C_out, C_in, kH, kW)

# Manual computation
print("Manual computation:")
for m in range(3):
    for n in range(3):
        patch = X[0, 0, m:m+3, n:n+3]
        val = (patch * W[0, 0]).sum().item()
        print(f"  Y[{m},{n}] = {val:.0f}")

# PyTorch conv2d
Y = F.conv2d(X, W)
print(f"\nPyTorch F.conv2d output:\n{Y.squeeze()}")

# Using nn.Conv2d with manual weights
conv = nn.Conv2d(1, 1, 3, bias=False)
conv.weight.data = W.clone()
Y_module = conv(X)
print(f"\nnn.Conv2d output:\n{Y_module.squeeze().detach()}")

# Verify that the same weights are used everywhere
print(f"\nFilter weights:\n{conv.weight.data.squeeze()}")
print(f"Number of parameters: {sum(p.numel() for p in conv.parameters())}")
```

---

## 4 — What Does Each Filter Learn?

### 4.1 — Filter as Feature Detector

Each filter in a convolutional layer learns to detect **one specific pattern**. Thanks to weight sharing, it detects that pattern **everywhere** in the image.

```
Filter #1: Vertical edge detector        Filter #2: Horizontal edge detector
    ┌─────────┐                              ┌─────────┐
    │ -1  0  1│                              │ -1 -1 -1│
    │ -1  0  1│                              │  0  0  0│
    │ -1  0  1│                              │  1  1  1│
    └─────────┘                              └─────────┘

Filter #3: Diagonal edge detector         Filter #4: Blob/spot detector
    ┌─────────┐                              ┌─────────┐
    │  0  0  1│                              │ -1 -1 -1│
    │  0  1  0│                              │ -1  8 -1│
    │  1  0  0│                              │ -1 -1 -1│
    └─────────┘                              └─────────┘
```

### 4.2 — Feature Maps: The Output of Filter Application

Applying a filter to an image produces a **feature map** (also called an activation map). It highlights where in the image the learned pattern appears.

```
Input Image:                Feature Map (vertical edges):
┌────────────────┐          ┌────────────────┐
│ █████░░░░░░░░░ │          │ ○○○○●○○○○○○○○○ │
│ █████░░░░░░░░░ │   ──→    │ ○○○○●○○○○○○○○○ │
│ █████░░░░░░░░░ │          │ ○○○○●○○○○○○○○○ │
│ █████░░░░░░░░░ │          │ ○○○○●○○○○○○○○○ │
│ █████░░░░░░░░░ │          │ ○○○○●○○○○○○○○○ │
└────────────────┘          └────────────────┘
                             ● = high activation (edge detected)
                             ○ = low activation (no edge)
```

### 4.3 — Multiple Filters = Multiple Feature Maps

```
                                    ┌─────────────┐
                              ──→   │ Feature Map 1│  (vertical edges)
                             │      └─────────────┘
                             │      ┌─────────────┐
Input Image ──→ 64 filters ──┤──→   │ Feature Map 2│  (horizontal edges)
                             │      └─────────────┘
                             │      ┌─────────────┐
                             │──→   │ Feature Map 3│  (diagonal edges)
                             │      └─────────────┘
                             │           ...
                             │      ┌──────────────┐
                              ──→   │ Feature Map 64│  (some other pattern)
                                    └──────────────┘

Input:  (3, 224, 224)    →    Output: (64, 222, 222)
         C_in                          C_out = number of filters
```

### 4.4 — PyTorch Code: Visualising Learned Filters

```python
import torch
import torch.nn as nn
import torchvision.models as models

# Load a pretrained ResNet and inspect first layer filters
resnet = models.resnet18(pretrained=True)
first_conv = resnet.conv1  # Conv2d(3, 64, 7, stride=2, padding=3)

weights = first_conv.weight.data  # Shape: (64, 3, 7, 7)

print(f"First conv layer: {first_conv}")
print(f"Weight shape: {weights.shape}")
print(f"Number of filters: {weights.shape[0]}")
print(f"Each filter shape: ({weights.shape[1]}, {weights.shape[2]}, {weights.shape[3]})")
print(f"Parameters: {weights.numel() + first_conv.bias.data.numel() if first_conv.bias is not None else weights.numel()}")

# Show a few filter patterns
for i in [0, 1, 2, 3]:
    filt = weights[i]  # (3, 7, 7)
    # Average across input channels for visualisation
    filt_avg = filt.mean(dim=0)  # (7, 7)
    
    print(f"\nFilter {i} (channel-averaged, 7×7):")
    for row in filt_avg:
        line = ""
        for val in row:
            v = val.item()
            if v > 0.05:
                line += " █"
            elif v < -0.05:
                line += " ░"
            else:
                line += " ·"
        print(f"  {line}")
```

---

## 5 — Why Weight Sharing Works: Stationarity Assumption

### 5.1 — The Stationarity Assumption

Weight sharing is justified by the **stationarity assumption** of natural images:

> **Stationarity:** The statistical properties of an image are approximately the same regardless of spatial position. A vertical edge looks the same whether it appears in the top-left or bottom-right of the image.

### 5.2 — When Stationarity Holds (Most Cases)

```
A vertical edge at position (10, 10):      A vertical edge at position (200, 200):

  120  120  200  200                         120  120  200  200
  120  120  200  200                         120  120  200  200
  120  120  200  200                         120  120  200  200

  Same local pattern! Same filter should detect both.
```

### 5.3 — When Stationarity Partially Breaks Down

| Scenario | Issue | Mitigation |
|---|---|---|
| Face detection | Eyes always above nose | Data augmentation, position encoding |
| Document OCR | Text at top ≠ text at bottom | Locally connected layers, attention |
| Satellite imaging | Scale varies with altitude | Multi-scale features, FPN |
| Medical imaging | Anatomy has fixed structure | Position-aware models |

> Even when stationarity partially breaks down, the **parameter savings** from weight sharing are so significant that convolutions still outperform locally connected layers in practice.

---

## 6 — Weight Sharing in Forward and Backward Pass

### 6.1 — Forward Pass

Same weights applied at every position — this is straightforward:

```
For output position (m, n):
  Y[m, n] = Σ_k Σ_l W[k, l] × X[m+k, n+l] + b
  
  W and b are IDENTICAL for all (m, n).
```

### 6.2 — Backward Pass: Gradient Aggregation

During backpropagation, the gradient for each shared weight is the **sum of gradients from all positions** where it was used:

```
∂L/∂W[k, l] = Σ_m Σ_n (∂L/∂Y[m, n]) × X[m+k, n+l]

The gradient is SUMMED over all spatial positions (m, n).
This means:
  - If a pattern appears many times → large gradient → strong learning signal
  - If a pattern appears rarely → small gradient → less update
```

### 6.3 — Intuition

```
Filter detects vertical edges.
Image has vertical edges at 50 positions.

During backprop:
  - Gradient contributions from all 50 positions are summed
  - This gives a STRONG learning signal for vertical edge detection
  - The filter gets optimised efficiently because it sees 50 examples per image

vs. Locally connected layer:
  - Each position's weights get gradient from only 1 location
  - Much weaker learning signal
  - Needs much more data to learn the same feature
```

### 6.4 — PyTorch Code: Verifying Gradient Sharing

```python
import torch
import torch.nn as nn

# Small example: 1 filter, 1 channel
conv = nn.Conv2d(1, 1, 3, bias=False, padding=1)

# Set known weights
conv.weight.data = torch.tensor([
    [[[1, 0, -1],
      [1, 0, -1],
      [1, 0, -1]]]
], dtype=torch.float32)

# Input with vertical edges at multiple positions
x = torch.tensor([[
    [[100, 100, 200, 200, 100, 100, 200, 200],
     [100, 100, 200, 200, 100, 100, 200, 200],
     [100, 100, 200, 200, 100, 100, 200, 200],
     [100, 100, 200, 200, 100, 100, 200, 200]]
]], dtype=torch.float32)

# Forward
y = conv(x)

# Backward (sum of all outputs)
loss = y.sum()
loss.backward()

# The gradient is the SUM of contributions from all positions
print("Weight gradient:")
print(conv.weight.grad.squeeze())
print(f"\nGradient shape: {conv.weight.grad.shape}")
print(f"Number of spatial positions: {y.shape[2] * y.shape[3]}")
print(f"Gradient is sum of all {y.shape[2] * y.shape[3]} position contributions")
```

---

## 7 — Deep Dive: FC vs Conv Parameter Scaling

### 7.1 — Scaling with Image Resolution

```
Fixed architecture: → 64 output channels, 3×3 kernel, 3 input channels

Resolution      FC Params          Conv Params      Ratio
───────────────────────────────────────────────────────────
28 × 28         150,528 × 64       1,792            84×
                = 9,633,792
32 × 32         196,608 × 64       1,792            112×
                = 12,582,912
64 × 64         786,432 × 64       1,792            448×
                = 50,331,648
128 × 128       3,145,728 × 64     1,792            1,792×
                = 201,326,592
224 × 224       9,633,792 × 64     1,792            5,376×
                = 616,562,688
512 × 512       50,331,648 × 64    1,792            28,000×
                = 3,221,225,472
```

### 7.2 — Scaling with Depth (Typical CNN)

```
Layer     C_in    C_out   K    Params (Conv)        Cumulative
──────────────────────────────────────────────────────────────
Conv1     3       64      3    3×3×3×64+64=1,792    1,792
Conv2     64      64      3    3×3×64×64+64=36,928  38,720
Conv3     64      128     3    3×3×64×128+128=73,856 112,576
Conv4     128     128     3    3×3×128×128+128=147,584 260,160
Conv5     128     256     3    3×3×128×256+256=295,168 555,328
Conv6     256     256     3    3×3×256×256+256=590,080 1,145,408
Conv7     256     512     3    3×3×256×512+512=1,180,160 2,325,568
Conv8     512     512     3    3×3×512×512+512=2,359,808 4,685,376

Total: ~4.7M parameters for 8 conv layers

FC equivalent for first layer alone: ~617M parameters
Conv achieves 8 layers for 0.76% of FC's first-layer cost!
```

### 7.3 — PyTorch Code: Full Network Parameter Breakdown

```python
import torch
import torch.nn as nn

class ParameterEfficientCNN(nn.Module):
    def __init__(self, num_classes=10):
        super().__init__()
        self.features = nn.Sequential(
            # Block 1: 3 → 64
            nn.Conv2d(3, 64, 3, padding=1), nn.BatchNorm2d(64), nn.ReLU(),
            nn.Conv2d(64, 64, 3, padding=1), nn.BatchNorm2d(64), nn.ReLU(),
            nn.MaxPool2d(2, 2),
            
            # Block 2: 64 → 128
            nn.Conv2d(64, 128, 3, padding=1), nn.BatchNorm2d(128), nn.ReLU(),
            nn.Conv2d(128, 128, 3, padding=1), nn.BatchNorm2d(128), nn.ReLU(),
            nn.MaxPool2d(2, 2),
            
            # Block 3: 128 → 256
            nn.Conv2d(128, 256, 3, padding=1), nn.BatchNorm2d(256), nn.ReLU(),
            nn.Conv2d(256, 256, 3, padding=1), nn.BatchNorm2d(256), nn.ReLU(),
            nn.MaxPool2d(2, 2),
            
            # Global Average Pooling
            nn.AdaptiveAvgPool2d(1),
        )
        self.classifier = nn.Linear(256, num_classes)
    
    def forward(self, x):
        x = self.features(x)
        x = x.view(x.size(0), -1)
        return self.classifier(x)

model = ParameterEfficientCNN()

# Parameter breakdown
print(f"{'Layer':<35} {'Shape':<25} {'Params':>10}")
print("─" * 72)
total = 0
for name, param in model.named_parameters():
    print(f"{name:<35} {str(list(param.shape)):<25} {param.numel():>10,}")
    total += param.numel()
print("─" * 72)
print(f"{'TOTAL':<35} {'':25} {total:>10,}")

# Compare with equivalent FC
fc_first_layer = 3 * 32 * 32 * 64  # Assuming 32×32 input
print(f"\nFC first layer alone: {fc_first_layer:,} params")
print(f"Entire CNN: {total:,} params")
print(f"CNN uses {total / fc_first_layer:.1f}× the params of FC's first layer")
print(f"  ... but has 6 conv layers, 6 BN layers, and a classifier!")
```

---

## 8 — Locally Connected Layers: The Middle Ground

### 8.1 — When You Might Want Unique Weights Per Position

Locally connected layers (no weight sharing) are rarely used, but have niche applications:

| Application | Why | Example |
|---|---|---|
| **Face recognition** | Face parts have fixed locations (eyes at top, mouth at bottom) | DeepFace (Facebook, 2014) |
| **Coordinate-aware tasks** | Position carries semantic meaning | CoordConv |
| **Very small feature maps** | Few positions → sharing saves little | Late layers of some architectures |

### 8.2 — PyTorch Code: Simulating a Locally Connected Layer

```python
import torch
import torch.nn as nn

class LocallyConnected2d(nn.Module):
    """
    Locally connected layer: same as conv but WITHOUT weight sharing.
    Each spatial position has its own unique set of weights.
    """
    def __init__(self, in_channels, out_channels, kernel_size, 
                 input_height, input_width):
        super().__init__()
        self.kernel_size = kernel_size
        self.out_h = input_height - kernel_size + 1
        self.out_w = input_width - kernel_size + 1
        
        # Unique weights for each output position
        self.weight = nn.Parameter(torch.randn(
            self.out_h, self.out_w, 
            out_channels, in_channels, 
            kernel_size, kernel_size
        ) * 0.01)
        self.bias = nn.Parameter(torch.zeros(out_channels))
    
    def forward(self, x):
        batch_size = x.size(0)
        out = torch.zeros(batch_size, self.weight.size(2), 
                         self.out_h, self.out_w, device=x.device)
        
        for i in range(self.out_h):
            for j in range(self.out_w):
                patch = x[:, :, i:i+self.kernel_size, j:j+self.kernel_size]
                # Use UNIQUE weights for position (i, j)
                w = self.weight[i, j]  # (out_ch, in_ch, k, k)
                out[:, :, i, j] = torch.einsum('bckl,ockl->bo', patch, w)
        
        return out + self.bias.view(1, -1, 1, 1)

# Compare parameter counts
H, W = 16, 16
conv = nn.Conv2d(3, 64, 3)
local = LocallyConnected2d(3, 64, 3, H, W)

print(f"Conv2d parameters:           {sum(p.numel() for p in conv.parameters()):>10,}")
print(f"LocallyConnected parameters: {sum(p.numel() for p in local.parameters()):>10,}")
print(f"Ratio: {sum(p.numel() for p in local.parameters()) / sum(p.numel() for p in conv.parameters()):.0f}×")
```

---

## 9 — Applications of Parameter Sharing

### 9.1 — Beyond Images

The weight-sharing principle extends far beyond 2-D image convolutions:

| Domain | Convolution Type | Sharing Dimension |
|---|---|---|
| **Images** | 2-D Conv | Spatial (H, W) |
| **Video** | 3-D Conv | Spatial + Temporal (H, W, T) |
| **Audio / Time Series** | 1-D Conv | Temporal (T) |
| **3-D Point Clouds** | 3-D Conv / PointConv | Spatial (X, Y, Z) |
| **Text (NLP)** | 1-D Conv | Sequential position |
| **Graphs** | Graph Conv | Node neighbourhood |

### 9.2 — 1-D Convolution Example (Time Series)

```python
import torch
import torch.nn as nn

# 1-D convolution for time series
# Input: batch of 8 sequences, 3 channels, 100 time steps
x = torch.randn(8, 3, 100)

conv1d = nn.Conv1d(in_channels=3, out_channels=16, kernel_size=5, padding=2)
y = conv1d(x)

print(f"Input shape:  {x.shape}")   # (8, 3, 100)
print(f"Output shape: {y.shape}")   # (8, 16, 100)
print(f"Parameters:   {sum(p.numel() for p in conv1d.parameters()):,}")
# 5 × 3 × 16 + 16 = 256 parameters — shared across all 100 time steps!
```

---

## 📊 Summary Table

| Concept | Key Idea |
|---|---|
| Parameter Sharing | Same filter weights applied at every spatial position |
| FC vs Conv | Conv has ~5,000× fewer params for ImageNet-sized inputs |
| Locally Connected | Middle ground: local but NOT shared (rarely used) |
| Filter = Feature Detector | Each filter learns one pattern, detects it everywhere |
| Feature Map | Output of applying one filter to the input; highlights where pattern appears |
| Stationarity Assumption | Visual patterns look the same regardless of position |
| Gradient Aggregation | Shared weights receive summed gradients from all positions |
| Scaling | Conv params are O(1) w.r.t. image resolution; FC params are O(H×W) |
| Multi-Channel Conv | Filter spans all input channels but only local spatial region |
| Generalisation | Weight sharing acts as regulariser — fewer params, less overfitting |

---

## ❓ Revision Questions

### Q1: Parameter Calculation
A Conv2d layer has 3 input channels, 128 output channels, and a 5×5 kernel. Calculate the total number of parameters (including biases).

<details>
<summary>Answer</summary>

```
Weights = K × K × C_in × C_out = 5 × 5 × 3 × 128 = 9,600
Biases  = C_out = 128
Total   = 9,600 + 128 = 9,728 parameters

These 9,728 parameters are shared across ALL spatial positions.
For a 224×224 input, the same 9,728 values are used at each of
the 220×220 = 48,400 output positions.
```
</details>

### Q2: Sharing Factor
For a Conv2d layer processing a 64×64 input with 3×3 kernel (stride=1, no padding), what is the "sharing factor" — how many times is each weight used in the forward pass?

<details>
<summary>Answer</summary>

```
Output size: (64-3+1) × (64-3+1) = 62 × 62 = 3,844 positions

Each weight in the 3×3 filter is used at all 3,844 positions.
Sharing factor = 3,844

For a 224×224 input: sharing factor = 222 × 222 = 49,284

This is why conv layers are so parameter-efficient: each weight
contributes to thousands of outputs.
```
</details>

### Q3: FC Equivalent
If you had to create an FC layer that produces the exact same output as a Conv2d(3, 64, 3) layer on a 32×32 input (with the constraint of using the same weight values), how many of the FC layer's weights would be zero?

<details>
<summary>Answer</summary>

```
Conv output: 30 × 30 × 64 = 57,600 neurons
FC input: 32 × 32 × 3 = 3,072 neurons
FC weight matrix: 3,072 × 57,600 = 176,947,200 weights

Each conv output neuron connects to only 3×3×3 = 27 inputs.
Non-zero weights: 57,600 × 27 = 1,555,200

But weight sharing means only 27 × 64 = 1,728 unique values are used.
Many of the "non-zero" FC weights have identical values.

Zero weights: 176,947,200 - 1,555,200 = 175,392,000

That's 99.12% zeros! The FC matrix is extremely sparse and redundant.
The conv layer compresses this into just 1,728 + 64 = 1,792 params.
```
</details>

### Q4: Gradient Accumulation
During backpropagation, how is the gradient computed for a shared convolutional weight? Why does this help learning?

<details>
<summary>Answer</summary>

The gradient for each shared weight is the **sum of partial derivatives from every spatial position**:

```
∂L/∂W[k,l] = Σ_m Σ_n (∂L/∂Y[m,n]) × X[m+k, n+l]
```

This **aggregation** helps learning because:
1. **Stronger signal:** The gradient accumulates evidence from all positions, giving a clearer learning signal than a single position would.
2. **Built-in averaging:** It's like having mini-batch gradient descent within each image — each position is a "sample" of the pattern.
3. **Noise reduction:** Random noise at individual positions tends to cancel out across the sum.
4. **Data efficiency:** One training image provides H_out × W_out gradient samples for the filter weights.
</details>

### Q5: Stationarity
Explain why parameter sharing might be suboptimal for face recognition. Why are convolutions still used in practice?

<details>
<summary>Answer</summary>

**Why suboptimal:** Faces have a fixed spatial layout — eyes are always above the nose, mouth is below the nose. A filter detecting "eye" features at the top of the image doesn't need to detect "eyes" at the bottom. The stationarity assumption (features are position-independent) partially breaks down.

**Why convolutions are still used:**
1. The parameter savings are too significant to give up (locally connected layers would have ~50,000× more parameters).
2. Low-level features (edges, textures) ARE stationary even in faces.
3. Data augmentation (random crops, rotations) forces the network to handle position variation anyway.
4. Deeper layers naturally learn position-dependent features through the combination of position-independent filters.
5. Modern approaches (attention mechanisms, positional encodings) can add position awareness to conv-based models.
</details>

### Q6: Multi-Channel
A Conv2d(64, 128, 3) layer processes an input of shape (1, 64, 56, 56). What is the shape of (a) a single filter, (b) the full weight tensor, (c) the output?

<details>
<summary>Answer</summary>

```
(a) Single filter shape: (64, 3, 3) = (C_in, K_h, K_w)
    Each filter spans ALL 64 input channels.

(b) Full weight tensor: (128, 64, 3, 3)
    = (C_out, C_in, K_h, K_w)
    128 filters, each of shape (64, 3, 3)
    Total weights: 128 × 64 × 3 × 3 = 73,728
    + 128 biases = 73,856 parameters

(c) Output shape: (1, 128, 54, 54)
    = (N, C_out, H_out, W_out)
    H_out = 56 - 3 + 1 = 54 (no padding)
    W_out = 56 - 3 + 1 = 54
```
</details>

---

## 🔗 Navigation

| | |
|---|---|
| ← Previous | [Local Connectivity](./03-local-connectivity.md) |
| → Next | [Translation Invariance](./05-translation-invariance.md) |
| 🏠 Unit | [01 – Introduction to CNNs](./README.md) |

---

*© 2025 AIML Study Notes. For educational use.*
