# 3 · Global Pooling

> **Unit 3.2 – Convolutional Neural Networks / 03 – Pooling Layers**
>
> [← Previous: Average Pooling](./02-average-pooling.md) | [Next: Pooling vs Strided Convolution →](./04-pooling-vs-strided-conv.md)
>
> [↑ Back to Pooling Layers Overview](./README.md)

---

## 3.1 Chapter Overview

Global Average Pooling (GAP) is one of the most consequential ideas in modern CNN design. Instead of using a small sliding window, GAP pools across the **entire spatial extent** of each channel, collapsing an `(C, H, W)` feature map into a `(C, 1, 1)` vector — **one scalar per channel**.

This simple operation:

- **Eliminates fully connected layers** — removing millions of parameters.
- **Acts as structural regularization** — the network must learn one meaningful feature per channel.
- **Enables arbitrary input sizes** — no fixed spatial dimension requirement.
- **Powers modern architectures** — used in GoogLeNet, ResNet, DenseNet, EfficientNet, Vision Transformers, and nearly every competitive CNN since 2014.

GAP was introduced by **Lin et al. (2014)** in the **Network in Network (NiN)** paper, which argued that fully connected layers are prone to overfitting and can be replaced entirely by global average pooling.

---

## 3.2 Mathematical Definition

### Global Average Pooling (GAP)

Given input feature map **X** of shape `(C, H, W)`:

```
                     1     H-1  W-1
  Y[c]  =  ─────────────  Σ    Σ   X[c, i, j]
            H × W         i=0  j=0

  Output shape: (C,)  or equivalently  (C, 1, 1)
```

Each output scalar is the **spatial mean** of the corresponding channel.

### Global Max Pooling (GMP)

The max variant takes the **maximum** across the spatial extent:

```
  Y[c]  =   max    X[c, i, j]
           0≤i<H
           0≤j<W

  Output shape: (C,)  or equivalently  (C, 1, 1)
```

### Comparison

| Operation | Formula | Interpretation |
|-----------|---------|---------------|
| **GAP** | Mean over H×W | "How strongly is feature c present *on average*?" |
| **GMP** | Max over H×W | "Is feature c present *anywhere*?" |

---

## 3.3 ASCII Diagram — Global Average Pooling

```
  INPUT FEATURE MAP: 3 channels × 4×4 spatial

  Channel 0:              Channel 1:              Channel 2:
  ┌────┬────┬────┬────┐   ┌────┬────┬────┬────┐   ┌────┬────┬────┬────┐
  │ 1  │ 2  │ 3  │ 4  │   │ 0  │ 1  │ 0  │ 1  │   │ 5  │ 5  │ 5  │ 5  │
  ├────┼────┼────┼────┤   ├────┼────┼────┼────┤   ├────┼────┼────┼────┤
  │ 5  │ 6  │ 7  │ 8  │   │ 2  │ 0  │ 2  │ 0  │   │ 5  │ 5  │ 5  │ 5  │
  ├────┼────┼────┼────┤   ├────┼────┼────┼────┤   ├────┼────┼────┼────┤
  │ 1  │ 2  │ 3  │ 4  │   │ 0  │ 1  │ 0  │ 1  │   │ 5  │ 5  │ 5  │ 5  │
  ├────┼────┼────┼────┤   ├────┼────┼────┼────┤   ├────┼────┼────┼────┤
  │ 5  │ 6  │ 7  │ 8  │   │ 2  │ 0  │ 2  │ 0  │   │ 5  │ 5  │ 5  │ 5  │
  └────┴────┴────┴────┘   └────┴────┴────┴────┘   └────┴────┴────┴────┘

              │                    │                      │
              ▼                    ▼                      ▼
        sum = 72              sum = 12               sum = 80
        /16 = 4.5             /16 = 0.75             /16 = 5.0

  OUTPUT VECTOR: [4.5, 0.75, 5.0]     shape: (3,)

  From (3, 4, 4) = 48 values  →  (3,) = 3 values
  Compression ratio: 16:1 per channel
```

### The Big Picture — GAP Replaces FC Layers

```
  TRADITIONAL (AlexNet/VGG style):
  ┌──────────┐     ┌─────────┐     ┌──────────────┐     ┌──────────┐
  │ Conv     │     │ Conv    │     │ Flatten      │     │ FC       │
  │ layers   │ ──► │ output  │ ──► │ 512×7×7      │ ──► │ 25088→   │
  │          │     │ 512×7×7 │     │ = 25,088     │     │ 4096     │
  └──────────┘     └─────────┘     └──────────────┘     └──────────┘
                                                          │
                                   Parameters: 25,088 × 4,096 = 102,760,448 !!
                                                          │
                                                          ▼
                                                    ┌──────────┐
                                                    │ FC 4096  │
                                                    │ → 1000   │
                                                    └──────────┘

  MODERN (ResNet/GoogLeNet style):
  ┌──────────┐     ┌─────────┐     ┌──────────────┐     ┌──────────┐
  │ Conv     │     │ Conv    │     │ GAP          │     │ FC       │
  │ layers   │ ──► │ output  │ ──► │ 512×7×7      │ ──► │ 512 →   │
  │          │     │ 512×7×7 │     │ → 512×1×1    │     │ 1000     │
  └──────────┘     └─────────┘     └──────────────┘     └──────────┘
                                                          │
                                   Parameters: 512 × 1,000 = 512,000
                                                          │
                                   Reduction: 102M → 0.5M = ~200× fewer params!
```

---

## 3.4 The Network in Network (NiN) Contribution

### Historical Context

Before NiN (2014), the standard CNN architecture ended with:

```
  Conv layers → Flatten → FC(4096) → FC(4096) → FC(num_classes)
```

These FC layers contained **>90% of the total parameters** in networks like AlexNet and VGG.

### The NiN Insight

Lin et al. proposed two key ideas:

1. **1×1 Convolutions** ("mlpconv") — Add non-linear transformations within each spatial position (this later influenced Inception and ResNet bottlenecks).

2. **Global Average Pooling** — Replace the FC classifier entirely:

```
  NiN Architecture:
  ┌───────────┐     ┌───────────┐     ┌───────────┐     ┌──────┐
  │ MLPConv   │     │ MLPConv   │     │ MLPConv   │     │ GAP  │
  │ Block 1   │ ──► │ Block 2   │ ──► │ Block 3   │ ──► │      │ ──► predictions
  │           │     │           │     │ out_ch =  │     │      │
  │           │     │           │     │ num_class │     │      │
  └───────────┘     └───────────┘     └───────────┘     └──────┘

  The last conv layer has exactly num_classes output channels.
  GAP collapses each channel to a single value → class logits.
  No FC layer needed!
```

### Why GAP Works as a Classifier

```
  If the final conv layer has C = num_classes channels:

  Channel 0 feature map → "How much does the input look like class 0?"
  Channel 1 feature map → "How much does the input look like class 1?"
  ...
  Channel k feature map → "How much does the input look like class k?"

  GAP averages each map into a single confidence score.
  This enforces a meaningful, interpretable structure:
  each channel becomes a CLASS ACTIVATION MAP (CAM).
```

---

## 3.5 Backpropagation Through Global Average Pooling

### Gradient Computation

```
  Forward:  Y[c] = (1 / H·W) × Σ X[c, i, j]

  Backward:
      ∂L           1       ∂L
      ──────── = ─────── · ────
      ∂X[c,i,j]   H × W   ∂Y[c]

  Every spatial position in channel c receives the SAME gradient,
  scaled by 1/(H×W).
```

### ASCII Diagram

```
  Backward through GAP (channel c):

  Upstream gradient: ∂L/∂Y[c] = g

  Distributed to input:
  ┌────────┬────────┬────────┬────────┐
  │ g/H·W  │ g/H·W  │ g/H·W  │ g/H·W  │
  ├────────┼────────┼────────┼────────┤
  │ g/H·W  │ g/H·W  │ g/H·W  │ g/H·W  │     For 4×4: each gets g/16
  ├────────┼────────┼────────┼────────┤
  │ g/H·W  │ g/H·W  │ g/H·W  │ g/H·W  │
  ├────────┼────────┼────────┼────────┤
  │ g/H·W  │ g/H·W  │ g/H·W  │ g/H·W  │
  └────────┴────────┴────────┴────────┘

  Uniform gradient → equal contribution from all spatial locations.
```

---

## 3.6 Worked Example — GAP for Classification

### Problem

A CNN produces a final feature map of shape `(num_classes=3, 3, 3)`. Apply GAP and compute class predictions.

```
  Channel 0 ("cat"):       Channel 1 ("dog"):       Channel 2 ("bird"):
  ┌─────┬─────┬─────┐     ┌─────┬─────┬─────┐     ┌─────┬─────┬─────┐
  │ 8.0 │ 7.0 │ 6.0 │     │ 1.0 │ 2.0 │ 1.0 │     │ 3.0 │ 4.0 │ 3.0 │
  ├─────┼─────┼─────┤     ├─────┼─────┼─────┤     ├─────┼─────┼─────┤
  │ 9.0 │ 8.5 │ 7.0 │     │ 0.5 │ 1.5 │ 0.5 │     │ 2.0 │ 5.0 │ 2.0 │
  ├─────┼─────┼─────┤     ├─────┼─────┼─────┤     ├─────┼─────┼─────┤
  │ 6.0 │ 7.5 │ 5.0 │     │ 1.0 │ 2.0 │ 0.5 │     │ 3.0 │ 4.0 │ 1.0 │
  └─────┴─────┴─────┘     └─────┴─────┴─────┘     └─────┴─────┴─────┘
```

### Step 1: Compute GAP

```
  GAP[0] = (8+7+6+9+8.5+7+6+7.5+5) / 9 = 64.0/9 ≈ 7.11   ("cat")
  GAP[1] = (1+2+1+0.5+1.5+0.5+1+2+0.5) / 9 = 10.0/9 ≈ 1.11   ("dog")
  GAP[2] = (3+4+3+2+5+2+3+4+1) / 9 = 27.0/9 = 3.00   ("bird")

  Logits = [7.11, 1.11, 3.00]
```

### Step 2: Apply Softmax

```
  exp(7.11) = 1225.1
  exp(1.11) =    3.03
  exp(3.00) =   20.09

  sum = 1248.2

  P("cat")  = 1225.1 / 1248.2 = 0.981
  P("dog")  =    3.03 / 1248.2 = 0.002
  P("bird") =   20.09 / 1248.2 = 0.016

  Prediction: CAT (98.1% confidence) ✓
```

### Step 3: Interpretation

Channel 0 has consistently high activations across the spatial map → strong "cat" presence detected everywhere. This makes the GAP output high for the cat class. The feature map itself serves as a **Class Activation Map (CAM)**, showing where the network detects "cat-like" features.

---

## 3.7 Parameter Reduction Analysis

### Case Study: VGG-16 vs VGG-16 with GAP

```
  VGG-16 Original:
  ────────────────────────────────────────────────────────
  Final conv output:  512 × 7 × 7 = 25,088 features
  FC1: 25,088 → 4,096    →  25,088 × 4,096 = 102,760,448 params
  FC2: 4,096 → 4,096      →   4,096 × 4,096 =  16,777,216 params
  FC3: 4,096 → 1,000      →   4,096 × 1,000 =   4,096,000 params
                              ─────────────────────────────
  Total FC params:                              123,633,664

  VGG-16 with GAP:
  ────────────────────────────────────────────────────────
  Final conv output:  512 × 7 × 7
  GAP: 512 × 7 × 7 → 512   →          0 params (no weights!)
  FC:  512 → 1,000          →  512 × 1,000 =     512,000 params
                              ─────────────────────────────
  Total classifier params:                          512,000

  SAVINGS: 123.6M → 0.5M = 241× reduction!
  ────────────────────────────────────────────────────────

  VGG-16 total:    ~138M parameters  (89% in FC layers!)
  VGG-16 + GAP:     ~15M parameters  (96% in conv layers)
```

### Broader Comparison

| Architecture | Without GAP | With GAP | Reduction |
|-------------|-------------|----------|-----------|
| AlexNet | ~60M | ~3.7M | 16× |
| VGG-16 | ~138M | ~15M | 9× |
| GoogLeNet | Used GAP from start — only 5M total | — |
| ResNet-50 | Used GAP from start — only 25.6M total | — |

---

## 3.8 Python / PyTorch Implementation

### Using `nn.AdaptiveAvgPool2d(1)`

The key PyTorch class for GAP is `AdaptiveAvgPool2d` with output size `(1, 1)`:

```python
import torch
import torch.nn as nn

# ─── Global Average Pooling ────────────────────────────────
gap = nn.AdaptiveAvgPool2d(1)  # output_size = (1, 1)

# Input: batch=2, channels=512, H=7, W=7
x = torch.randn(2, 512, 7, 7)

y = gap(x)
print(f"Input shape:  {x.shape}")   # [2, 512, 7, 7]
print(f"Output shape: {y.shape}")   # [2, 512, 1, 1]

# Squeeze spatial dims for the classifier
y_flat = y.view(y.size(0), -1)     # [2, 512]
print(f"Flattened:    {y_flat.shape}")
```

### Why `AdaptiveAvgPool2d` Instead of `AvgPool2d`?

```python
# AdaptiveAvgPool2d AUTOMATICALLY computes kernel_size and stride
# to produce the desired output size, regardless of input size.

gap = nn.AdaptiveAvgPool2d(1)

# Works with ANY spatial size:
for size in [7, 14, 28, 32, 224]:
    x = torch.randn(1, 64, size, size)
    y = gap(x)
    print(f"  Input {size}×{size} → Output {y.shape[-2]}×{y.shape[-1]}")
# All produce 1×1 output!

# AvgPool2d would need different kernel_size for each input size:
# AvgPool2d(7) for 7×7, AvgPool2d(14) for 14×14, etc.
```

### Complete CNN with GAP (ResNet-Style)

```python
class SimpleResNetStyle(nn.Module):
    """Simplified ResNet-style classifier using GAP."""
    def __init__(self, num_classes=10):
        super().__init__()
        # Feature extractor (simplified)
        self.features = nn.Sequential(
            nn.Conv2d(3, 64, 7, stride=2, padding=3),   # /2
            nn.BatchNorm2d(64),
            nn.ReLU(inplace=True),
            nn.MaxPool2d(3, stride=2, padding=1),        # /2
            nn.Conv2d(64, 128, 3, stride=2, padding=1),  # /2
            nn.BatchNorm2d(128),
            nn.ReLU(inplace=True),
            nn.Conv2d(128, 256, 3, stride=2, padding=1), # /2
            nn.BatchNorm2d(256),
            nn.ReLU(inplace=True),
            nn.Conv2d(256, 512, 3, stride=2, padding=1), # /2
            nn.BatchNorm2d(512),
            nn.ReLU(inplace=True),
        )

        # GAP + classifier
        self.gap = nn.AdaptiveAvgPool2d(1)
        self.classifier = nn.Linear(512, num_classes)

    def forward(self, x):
        x = self.features(x)       # [B, 512, H', W']
        x = self.gap(x)            # [B, 512, 1, 1]
        x = x.view(x.size(0), -1)  # [B, 512]
        x = self.classifier(x)     # [B, num_classes]
        return x

model = SimpleResNetStyle(num_classes=1000)
x = torch.randn(1, 3, 224, 224)
y = model(x)
print(f"Output: {y.shape}")  # [1, 1000]

# Count parameters
total = sum(p.numel() for p in model.parameters())
classifier = sum(p.numel() for p in model.classifier.parameters())
print(f"Total params:      {total:,}")       # ~2.5M
print(f"Classifier params: {classifier:,}")  # 513,000 (512*1000 + 1000)
print(f"Classifier share:  {classifier/total*100:.1f}%")
```

### Global Max Pooling

```python
# Global Max Pooling variant
gmp = nn.AdaptiveMaxPool2d(1)

x = torch.randn(1, 256, 14, 14)
y = gmp(x)
print(f"GMP output: {y.shape}")  # [1, 256, 1, 1]

# Compare GAP vs GMP
gap = nn.AdaptiveAvgPool2d(1)
y_avg = gap(x)
y_max = gmp(x)

print(f"GAP mean value: {y_avg.mean():.4f}")
print(f"GMP mean value: {y_max.mean():.4f}")  # Higher, since it picks peaks
```

### Class Activation Map (CAM) Visualization

```python
import torch
import torch.nn as nn
import torch.nn.functional as F

class CAMModel(nn.Module):
    """Model that supports Class Activation Map extraction."""
    def __init__(self, num_classes=10):
        super().__init__()
        self.features = nn.Sequential(
            nn.Conv2d(3, 64, 3, padding=1),
            nn.ReLU(),
            nn.MaxPool2d(2),
            nn.Conv2d(64, 128, 3, padding=1),
            nn.ReLU(),
            nn.MaxPool2d(2),
            nn.Conv2d(128, num_classes, 3, padding=1),
            nn.ReLU(),
        )
        self.gap = nn.AdaptiveAvgPool2d(1)
        # No FC layer — GAP output IS the logits

    def forward(self, x):
        feature_maps = self.features(x)   # [B, num_classes, H', W']
        logits = self.gap(feature_maps)    # [B, num_classes, 1, 1]
        logits = logits.view(logits.size(0), -1)  # [B, num_classes]
        return logits, feature_maps

# Usage
model = CAMModel(num_classes=10)
x = torch.randn(1, 3, 32, 32)
logits, feature_maps = model(x)

pred_class = logits.argmax(dim=1).item()
cam = feature_maps[0, pred_class]  # The CAM for the predicted class

print(f"Predicted class: {pred_class}")
print(f"CAM shape: {cam.shape}")  # [8, 8] — shows WHERE the class is detected
# Upsample CAM to input size for visualization:
cam_upsampled = F.interpolate(cam.unsqueeze(0).unsqueeze(0),
                               size=(32, 32), mode='bilinear',
                               align_corners=False)
print(f"Upsampled CAM: {cam_upsampled.shape}")  # [1, 1, 32, 32]
```

---

## 3.9 Advantages and Limitations

### Advantages

| Advantage | Explanation |
|-----------|-------------|
| **Massive param reduction** | Eliminates FC layers (often 90%+ of parameters) |
| **Regularization** | Fewer params → less overfitting, may not need dropout |
| **Input size flexibility** | No fixed spatial dimensions required at inference |
| **Interpretability** | Feature maps become class activation maps (CAMs) |
| **Simpler architecture** | One less hyperparameter choice (FC layer width) |

### Limitations

| Limitation | Explanation |
|------------|-------------|
| **Information loss** | Entire spatial map compressed to one number per channel |
| **Requires enough channels** | Network needs sufficient channels to encode all class info |
| **May underperform for localization** | Averaging loses precise spatial information |
| **Assumes distributed features** | Poor if class evidence is concentrated in a small region |

---

## 3.10 Summary Table

| Aspect | Detail |
|--------|--------|
| **Operation** | Average (or max) over entire spatial extent per channel |
| **Input Shape** | `(B, C, H, W)` — any H, W |
| **Output Shape** | `(B, C, 1, 1)` |
| **Parameters** | 0 |
| **Key Benefit** | Replaces FC layers, reduces parameters by 100-200× |
| **Introduced By** | Lin et al., "Network in Network" (NiN), 2014 |
| **PyTorch Class** | `nn.AdaptiveAvgPool2d(1)` or `nn.AdaptiveMaxPool2d(1)` |
| **Used In** | GoogLeNet, ResNet, DenseNet, EfficientNet, MobileNet, ViT |
| **Enables** | Class Activation Maps (CAMs) for interpretability |

---

## 3.11 Revision Questions

**Q1.** What is the output shape when `nn.AdaptiveAvgPool2d(1)` is applied to a tensor of shape `(16, 2048, 7, 7)`?

<details><summary>Answer</summary>

`(16, 2048, 1, 1)` — Each of the 2048 channels is averaged across its 7×7=49 spatial positions to produce a single scalar. The batch dimension is preserved.

</details>

**Q2.** Why does GAP act as a form of regularization compared to fully connected layers?

<details><summary>Answer</summary>

GAP has zero learnable parameters, while FC layers add millions (e.g., 25,088 × 4,096 = 102M in VGG-16). Fewer parameters means less capacity to memorize training data, reducing overfitting. Additionally, GAP enforces that each feature map channel captures a meaningful spatial pattern, preventing the FC layer from learning arbitrary mappings.

</details>

**Q3.** How does GAP enable Class Activation Maps (CAMs)?

<details><summary>Answer</summary>

When the last convolutional layer has `num_classes` channels followed by GAP, each channel's feature map represents the spatial detection of that class. The GAP output for class k is literally the spatial average of channel k's feature map. By upsampling channel k's feature map to the input resolution, we get a heatmap showing where the network detects class k — this is the Class Activation Map.

</details>

**Q4.** Why do we use `AdaptiveAvgPool2d(1)` instead of `AvgPool2d(kernel_size=H)` in PyTorch?

<details><summary>Answer</summary>

`AdaptiveAvgPool2d(1)` automatically computes the required kernel size to produce a 1×1 output, regardless of the input spatial dimensions. `AvgPool2d(H)` requires knowing the exact spatial size at construction time and would break if the input size changes. This makes `AdaptiveAvgPool2d` essential for models that accept variable input sizes.

</details>

**Q5.** In ResNet-50, the last conv layer outputs `(B, 2048, 7, 7)`. Compare the parameter count of a GAP + Linear(2048, 1000) classifier vs a Flatten + Linear(2048×7×7, 1000) classifier.

<details><summary>Answer</summary>

- **GAP + Linear:** 0 + (2048 × 1000 + 1000) = 2,049,000 parameters
- **Flatten + Linear:** 0 + (2048 × 7 × 7 × 1000 + 1000) = 100,353,000 parameters

GAP reduces classifier parameters by ~49×. The GAP version is also more robust to input size changes and less prone to overfitting.

</details>

**Q6.** A student claims that Global Max Pooling is always better than Global Average Pooling because "max captures the strongest feature." Evaluate this claim.

<details><summary>Answer</summary>

The claim is incorrect in general. GAP is preferred in most architectures because: (1) it considers the entire spatial distribution, not just the peak — a feature present everywhere is more reliable than a single strong activation that might be noise; (2) GAP provides smoother gradients (all positions updated equally); (3) GAP is more robust to adversarial perturbations that could artificially create a strong peak; (4) empirically, ResNet, EfficientNet, and most SOTA architectures use GAP. GMP can be useful when features are truly sparse and localized.

</details>

---

> [← Previous: Average Pooling](./02-average-pooling.md) | [Next: Pooling vs Strided Convolution →](./04-pooling-vs-strided-conv.md)
>
> [↑ Back to Pooling Layers Overview](./README.md)
