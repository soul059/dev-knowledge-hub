# 🔄 1.5 — Translation Invariance & Equivariance

> **Unit:** 01 – Introduction to CNNs
> **Chapter:** 5 of 5 — Position-Independent Feature Detection

---

[← Previous: Parameter Sharing](./04-parameter-sharing.md) · [🏠 Unit Overview](./README.md)

---

## 📋 Chapter Overview

Translation invariance — the ability to recognise an object regardless of where it appears in the image — is one of the most desirable properties of a vision system. This chapter rigorously defines translation equivariance and invariance, shows how convolutions provide equivariance, how pooling provides invariance, and how data augmentation complements these architectural properties.

**After this chapter you will be able to:**

1. Distinguish between translation **equivariance** and translation **invariance**.
2. Prove mathematically that convolution is equivariant to translation.
3. Explain how pooling layers convert equivariance into approximate invariance.
4. Describe how data augmentation complements architectural invariance.
5. Analyse the trade-offs between invariance and spatial precision.

---

## 1 — Definitions: Equivariance vs Invariance

### 1.1 — Translation Equivariance

A function **f** is **equivariant** to translation if shifting the input causes the same shift in the output:

```
f(translate(x)) = translate(f(x))

If the input shifts by (Δh, Δw), the output shifts by (Δh, Δw) too.
The response MOVES WITH the input.
```

**Analogy:** A shadow is equivariant to your movement — if you step left, your shadow steps left.

### 1.2 — Translation Invariance

A function **f** is **invariant** to translation if shifting the input does NOT change the output:

```
f(translate(x)) = f(x)

The output is THE SAME regardless of where the input feature appears.
```

**Analogy:** A smoke detector is invariant to fire position — it beeps regardless of where in the room the fire is.

### 1.3 — ASCII Diagram: Equivariance vs Invariance

```
EQUIVARIANCE (Convolution):

  Input A:              Input B (shifted right):
  🐱 · · · ·            · · 🐱 · ·
  
  Conv Output A:         Conv Output B:
  ★ · · · ·             · · ★ · ·         ← Output SHIFTS with input
  
  ★ = "cat feature detected here"

────────────────────────────────────────────────

INVARIANCE (Convolution + Global Pooling → Classification):

  Input A:              Input B (shifted right):
  🐱 · · · ·            · · 🐱 · ·
  
  Classification A:      Classification B:
  "cat" (0.95)           "cat" (0.95)       ← Output UNCHANGED
```

### 1.4 — Why Both Matter

| Property | Where Used | Purpose |
|---|---|---|
| **Equivariance** | Conv layers (intermediate) | Feature map accurately reflects WHERE features are |
| **Invariance** | Final classification | Output label doesn't depend on object position |

For tasks like **object detection** and **segmentation**, we need equivariance (to know WHERE things are). For **classification**, we need invariance (to know WHAT things are, regardless of position).

---

## 2 — Convolution Is Equivariant: The Proof

### 2.1 — Mathematical Setup

Let:
- **x** = input image (2-D for simplicity)
- **w** = convolutional filter
- **T_{Δ}** = translation operator that shifts by Δ
- **(x * w)** = convolution of x with w

### 2.2 — The Translation Operator

```
T_{(Δh, Δw)}[x](i, j) = x(i - Δh, j - Δw)

This shifts the image by (Δh, Δw).
```

### 2.3 — Proof of Equivariance

We need to show: **(T_{Δ}[x]) * w = T_{Δ}[x * w]**

```
Left side: Convolve THEN observe the output
  (T_{Δ}[x] * w)[m, n] = Σ_k Σ_l w[k, l] × (T_{Δ}[x])[m+k, n+l]
                        = Σ_k Σ_l w[k, l] × x[m+k-Δh, n+l-Δw]

Right side: Observe the convolution output THEN shift
  T_{Δ}[x * w][m, n]   = (x * w)[m-Δh, n-Δw]
                        = Σ_k Σ_l w[k, l] × x[(m-Δh)+k, (n-Δw)+l]
                        = Σ_k Σ_l w[k, l] × x[m+k-Δh, n+l-Δw]

Left side = Right side  ✓

QED: Convolution is equivariant to translation.
```

### 2.4 — Intuitive Explanation

```
Why does this work?

Because of WEIGHT SHARING:
  - The filter w has the SAME values at every position
  - If the input feature moves, the filter sees the same local pattern
    at a different position → produces the same activation at the new position
  
If weights were NOT shared (FC or locally connected):
  - Different weights at different positions
  - Moving the feature activates DIFFERENT weights
  - Output changes unpredictably → NOT equivariant
```

### 2.5 — PyTorch Code: Demonstrating Equivariance

```python
import torch
import torch.nn as nn
import torch.nn.functional as F

# Create a conv layer (with fixed weights for reproducibility)
conv = nn.Conv2d(1, 1, 3, padding=1, bias=False)
torch.manual_seed(42)
conv.weight.data = torch.randn(1, 1, 3, 3)

# Create input with a feature at position (2, 2)
x = torch.zeros(1, 1, 8, 8)
x[0, 0, 1:4, 1:4] = torch.tensor([
    [1, 2, 1],
    [2, 4, 2],
    [1, 2, 1],
], dtype=torch.float32)

# Create shifted input: same feature at position (2, 5)
x_shifted = torch.zeros(1, 1, 8, 8)
x_shifted[0, 0, 1:4, 4:7] = torch.tensor([
    [1, 2, 1],
    [2, 4, 2],
    [1, 2, 1],
], dtype=torch.float32)

# Apply convolution
y = conv(x)
y_shifted = conv(x_shifted)

print("Output for original input:")
print(y.squeeze().detach().numpy().round(2))
print("\nOutput for shifted input:")
print(y_shifted.squeeze().detach().numpy().round(2))

# Verify equivariance: shift(conv(x)) should equal conv(shift(x))
# Shift the original output by (0, +3)
y_then_shift = torch.zeros_like(y)
y_then_shift[0, 0, :, 3:] = y[0, 0, :, :5]

print("\nShifted output of original (equivariance prediction):")
print(y_then_shift.squeeze().detach().numpy().round(2))

# Compare
diff = (y_shifted - y_then_shift).abs().max().item()
print(f"\nMax difference: {diff:.6f}")  # Should be ~0 (numerical precision)
print(f"Equivariance verified: {diff < 1e-5}")
```

---

## 3 — FC Layers Are NOT Equivariant

### 3.1 — Why FC Lacks Equivariance

In an FC layer, each output neuron has **unique weights** for each input position. Moving a feature changes which weights it activates:

```
FC Layer: y_j = Σ_i w_ji × x_i + b_j

Feature at position 5:   y_j uses w_j5 × (feature value)
Feature at position 50:  y_j uses w_j50 × (feature value)

w_j5 ≠ w_j50 in general → different output → NOT equivariant
```

### 3.2 — Visual Comparison

```
CONV — Same filter at every position:
                                               
Position A: [w₁ w₂ w₃] · [a b c] = w₁a + w₂b + w₃c = z₁
Position B: [w₁ w₂ w₃] · [a b c] = w₁a + w₂b + w₃c = z₁  ← SAME output!

FC — Different weights at every position:

Position A: [α₁ α₂ α₃] · [a b c] = α₁a + α₂b + α₃c = z₁
Position B: [β₁ β₂ β₃] · [a b c] = β₁a + β₂b + β₃c = z₂  ← DIFFERENT output!

Unless α = β (which FC does not guarantee), z₁ ≠ z₂
```

### 3.3 — PyTorch Demonstration

```python
import torch
import torch.nn as nn

# FC layer for a 1×8 "image"
fc = nn.Linear(8, 4, bias=False)

# Feature at position 2-3
x1 = torch.zeros(1, 8)
x1[0, 2:4] = torch.tensor([5.0, 3.0])

# Same feature at position 5-6
x2 = torch.zeros(1, 8)
x2[0, 5:7] = torch.tensor([5.0, 3.0])

y1 = fc(x1)
y2 = fc(x2)

print(f"Feature at pos 2-3: output = {y1.detach().numpy().round(3)}")
print(f"Feature at pos 5-6: output = {y2.detach().numpy().round(3)}")
print(f"Same output? {torch.allclose(y1, y2, atol=1e-5)}")  # False

# Now try with Conv1d (equivariant)
conv = nn.Conv1d(1, 1, 3, padding=1, bias=False)

x1_conv = torch.zeros(1, 1, 8)
x1_conv[0, 0, 2:4] = torch.tensor([5.0, 3.0])

x2_conv = torch.zeros(1, 1, 8)
x2_conv[0, 0, 5:7] = torch.tensor([5.0, 3.0])

y1_conv = conv(x1_conv)
y2_conv = conv(x2_conv)

print(f"\nConv: Feature at 2-3: {y1_conv.squeeze().detach().numpy().round(3)}")
print(f"Conv: Feature at 5-6: {y2_conv.squeeze().detach().numpy().round(3)}")
print("Note: The activation pattern SHIFTS with the feature (equivariance)")
```

---

## 4 — From Equivariance to Invariance: The Role of Pooling

### 4.1 — Why We Need Invariance for Classification

Equivariance alone is not enough for classification:

```
Equivariant conv output for "cat at left":    [0.9, 0.1, 0.0, 0.0]
Equivariant conv output for "cat at right":   [0.0, 0.0, 0.1, 0.9]

These are DIFFERENT vectors → a classifier would treat them differently!

We need: regardless of position → same output → "cat"
```

### 4.2 — Pooling Provides Approximate Invariance

**Pooling** aggregates features over a spatial region, making the output less sensitive to exact position:

```
Max Pooling (2×2, stride 2):

Feature map (before pooling):
  Case A (feature at left):        Case B (feature shifted right by 1):
  ┌─────────────────┐              ┌─────────────────┐
  │ 9.0  0.5 │ 0.1  0.0 │         │ 0.5  9.0 │ 0.5  0.0 │
  │ 8.0  0.3 │ 0.0  0.0 │         │ 0.3  8.0 │ 0.3  0.0 │
  └─────────────────┘              └─────────────────┘

After 2×2 max pooling:
  Case A: [9.0, 0.1]              Case B: [9.0, 0.5]
           ↑                                ↑
        Same max value!              Same max value!

The max operation absorbs small shifts → approximate invariance.
```

### 4.3 — Types of Pooling and Their Invariance Properties

| Pooling Type | Operation | Invariance Level |
|---|---|---|
| **Max Pooling** | Take maximum in window | High (any shift within window) |
| **Average Pooling** | Average values in window | Moderate (smooths shifts) |
| **Global Average Pooling** | Average entire feature map → 1 value | Complete spatial invariance |
| **Global Max Pooling** | Max of entire feature map → 1 value | Complete spatial invariance |

### 4.4 — ASCII Diagram: Progressive Invariance Through Pooling

```
Input (16×16):     Cat at different positions
  ┌────────────┐   ┌────────────┐
  │ 🐱          │   │          🐱│
  │            │   │            │
  └────────────┘   └────────────┘

After Conv (16×16 feature maps):    Different activation locations
  ┌────────────┐   ┌────────────┐
  │ ★★          │   │          ★★│
  │ ★★          │   │          ★★│
  └────────────┘   └────────────┘

After Pool 2×2 (8×8):              Still different, but less so
  ┌──────┐         ┌──────┐
  │ ★     │         │     ★│
  │       │         │      │
  └──────┘         └──────┘

After Conv+Pool (4×4):              Getting closer
  ┌────┐           ┌────┐
  │ ★   │           │   ★│
  └────┘           └────┘

After Conv+Pool (2×2):              Very similar
  ┌──┐             ┌──┐
  │★ │             │ ★│
  └──┘             └──┘

After Global Average Pool (1×1):    IDENTICAL
  [0.73]           [0.73]          ← Same value! Position invariant!
```

### 4.5 — Mathematical View

```
Global Average Pooling (GAP):

  GAP(Y) = (1 / (H × W)) × Σ_m Σ_n Y[m, n]

For an equivariant feature map Y:
  If Y' = translate(Y), then Y' has the same values, just shifted.
  The sum over all positions is the same:
  
  Σ_m Σ_n Y'[m, n] = Σ_m Σ_n Y[m, n]
  
  → GAP(Y') = GAP(Y)    ← Perfect translation invariance!

Global Average Pooling achieves EXACT invariance because
summation over all positions is inherently order-independent.
```

### 4.6 — PyTorch Code: Pooling Provides Invariance

```python
import torch
import torch.nn as nn

# Build a simple CNN with Global Average Pooling
class InvariantCNN(nn.Module):
    def __init__(self):
        super().__init__()
        self.features = nn.Sequential(
            nn.Conv2d(1, 16, 3, padding=1), nn.ReLU(),
            nn.Conv2d(16, 32, 3, padding=1), nn.ReLU(),
            nn.AdaptiveAvgPool2d(1),  # Global Average Pooling
        )
        self.classifier = nn.Linear(32, 2)
    
    def forward(self, x):
        x = self.features(x)
        x = x.view(x.size(0), -1)  # (batch, 32)
        return self.classifier(x)

model = InvariantCNN()
model.eval()

# Create input with a bright patch at position (2, 2)
x1 = torch.zeros(1, 1, 16, 16)
x1[0, 0, 1:4, 1:4] = 5.0

# Same patch at position (10, 10)
x2 = torch.zeros(1, 1, 16, 16)
x2[0, 0, 9:12, 9:12] = 5.0

with torch.no_grad():
    y1 = model(x1)
    y2 = model(x2)

print(f"Output for feature at (2,2):   {y1.numpy().round(4)}")
print(f"Output for feature at (10,10): {y2.numpy().round(4)}")
print(f"Difference: {(y1 - y2).abs().max().item():.6f}")
print(f"Approximately invariant: {(y1 - y2).abs().max().item() < 0.5}")

# Compare with model WITHOUT global pooling
class NonInvariantCNN(nn.Module):
    def __init__(self):
        super().__init__()
        self.features = nn.Sequential(
            nn.Conv2d(1, 16, 3, padding=1), nn.ReLU(),
            nn.Conv2d(16, 32, 3, padding=1), nn.ReLU(),
        )
        self.classifier = nn.Linear(32 * 16 * 16, 2)
    
    def forward(self, x):
        x = self.features(x)
        x = x.view(x.size(0), -1)
        return self.classifier(x)

model2 = NonInvariantCNN()
model2.eval()

with torch.no_grad():
    z1 = model2(x1)
    z2 = model2(x2)

print(f"\nWithout GAP:")
print(f"Output for feature at (2,2):   {z1.numpy().round(4)}")
print(f"Output for feature at (10,10): {z2.numpy().round(4)}")
print(f"Difference: {(z1 - z2).abs().max().item():.6f}")
print(f"NOT invariant: outputs differ significantly")
```

---

## 5 — Invariance vs Equivariance: When You Need Which

### 5.1 — Task Requirements

```
╔═══════════════════════════╦═══════════════╦════════════════════════╗
║ Task                      ║ Needs         ║ Why                    ║
╠═══════════════════════════╬═══════════════╬════════════════════════╣
║ Image Classification      ║ Invariance    ║ Label is same anywhere ║
║ Object Detection          ║ Equivariance  ║ Must know WHERE object ║
║ Semantic Segmentation     ║ Equivariance  ║ Must label EVERY pixel ║
║ Image Retrieval           ║ Invariance    ║ Match regardless of pos║
║ Pose Estimation           ║ Equivariance  ║ Must localise keypoints║
║ Style Transfer            ║ Equivariance  ║ Style applied locally  ║
╚═══════════════════════════╩═══════════════╩════════════════════════╝
```

### 5.2 — The Invariance–Precision Trade-off

```
More pooling → More invariance → Less spatial precision

                  Invariance ↑
                       │
                  GAP  │  ████████████      Perfect invariance,
                       │  ████████████      no spatial info
                       │
            2×2 pool   │  ██████████
            ×3 layers  │  ██████████
                       │
            2×2 pool   │  ████████
            ×2 layers  │  ████████
                       │
            2×2 pool   │  ██████
            ×1 layer   │  ██████
                       │
            No pool    │  ████               Full equivariance,
                       │  ████               full spatial precision
                       └──────────────────
                         Spatial Precision →

Object detection needs: moderate invariance + good precision
Classification needs:   maximum invariance
Segmentation needs:     minimal invariance + maximum precision
```

### 5.3 — Architectural Solutions for Different Needs

```
Classification (ResNet):
  Conv → Pool → Conv → Pool → ... → GAP → FC → class
  ✓ Full invariance at output

Object Detection (Faster R-CNN):
  Conv → Conv → ... → Feature Map → Region Proposals → positions
  ✓ Equivariant feature maps preserve location info

Segmentation (U-Net):
  Encoder (conv+pool) → Bottleneck → Decoder (conv+upsample)
      ↓                                    ↑
      └──── Skip connections (preserve spatial detail) ────┘
  ✓ Combines deep features with fine spatial info
```

---

## 6 — Data Augmentation: Complementing Architectural Invariance

### 6.1 — Why Architecture Alone Is Not Enough

CNNs provide **approximate** translation invariance through weight sharing + pooling. But they do NOT provide invariance to:
- Rotation
- Scaling (zoom in/out)
- Flipping
- Colour changes
- Occlusion

### 6.2 — Data Augmentation Provides Learned Invariance

By showing the network transformed versions of the same image during training, we teach it to be invariant to those transformations:

```
Original Image:    Augmented Copies (all labelled "cat"):

  ┌──────┐         ┌──────┐  ┌──────┐  ┌──────┐  ┌──────┐
  │  🐱  │   →     │ 🐱    │  │    🐱│  │ 🐱   │  │  🐱  │
  │      │         │      │  │      │  │rotated│  │zoomed│
  └──────┘         └──────┘  └──────┘  └──────┘  └──────┘
                   shifted   shifted   rotated   scaled
                    left      right     15°       0.8×
```

### 6.3 — Common Augmentation Techniques

| Augmentation | Invariance It Provides | Typical Range |
|---|---|---|
| **Random Horizontal Flip** | Left-right mirror invariance | p = 0.5 |
| **Random Crop** | Translation invariance (stronger) | ±10-20% |
| **Random Rotation** | Rotation invariance | ±15-30° |
| **Random Scale** | Scale invariance | 0.8× – 1.2× |
| **Colour Jitter** | Lighting/colour invariance | brightness, contrast, saturation |
| **Random Erasing** | Occlusion robustness | 0-20% of image area |
| **Gaussian Blur** | Frequency invariance | σ = 0.1–2.0 |
| **Mixup** | Interpolation between classes | α = 0.2 |
| **CutMix** | Regional occlusion + mixing | α = 1.0 |

### 6.4 — PyTorch Code: Comprehensive Data Augmentation

```python
import torch
from torchvision import transforms

# Training augmentation pipeline
train_transform = transforms.Compose([
    # Spatial augmentations (provide geometric invariance)
    transforms.RandomResizedCrop(224, scale=(0.8, 1.0)),   # Scale + translation
    transforms.RandomHorizontalFlip(p=0.5),                # Mirror invariance
    transforms.RandomRotation(15),                          # Rotation invariance
    
    # Colour augmentations (provide photometric invariance)
    transforms.ColorJitter(
        brightness=0.2,
        contrast=0.2,
        saturation=0.2,
        hue=0.1,
    ),
    
    # Convert to tensor
    transforms.ToTensor(),
    
    # Normalise (ImageNet stats)
    transforms.Normalize(
        mean=[0.485, 0.456, 0.406],
        std=[0.229, 0.224, 0.225],
    ),
    
    # Random erasing (occlusion robustness)
    transforms.RandomErasing(p=0.25, scale=(0.02, 0.15)),
])

# Validation/test: NO augmentation (only resize + normalise)
val_transform = transforms.Compose([
    transforms.Resize(256),
    transforms.CenterCrop(224),
    transforms.ToTensor(),
    transforms.Normalize(
        mean=[0.485, 0.456, 0.406],
        std=[0.229, 0.224, 0.225],
    ),
])

# Example usage
from torchvision.datasets import CIFAR10
from torch.utils.data import DataLoader

train_dataset = CIFAR10(
    root='./data', train=True, download=True, transform=train_transform
)
val_dataset = CIFAR10(
    root='./data', train=False, download=True, transform=val_transform
)

train_loader = DataLoader(train_dataset, batch_size=64, shuffle=True)
val_loader = DataLoader(val_dataset, batch_size=64, shuffle=False)

# Verify augmentation produces diverse views
sample_img = train_dataset.data[0]  # Get a single image
from PIL import Image
import numpy as np

img_pil = Image.fromarray(sample_img)

print("10 augmented versions of the same image:")
for i in range(10):
    augmented = train_transform(img_pil)
    print(f"  Version {i+1}: shape={augmented.shape}, "
          f"mean={augmented.mean():.3f}, std={augmented.std():.3f}")
```

### 6.5 — Augmentation vs Architectural Invariance

```
╔════════════════════════════╦══════════════════════╦═══════════════════════╗
║ Aspect                     ║ Architecture (Conv)  ║ Data Augmentation     ║
╠════════════════════════════╬══════════════════════╬═══════════════════════╣
║ Translation invariance     ║ ✅ Built-in          ║ ✅ Reinforced          ║
║ Rotation invariance        ║ ❌ Not built-in      ║ ✅ Learned             ║
║ Scale invariance           ║ ❌ Not built-in      ║ ✅ Learned             ║
║ Colour invariance          ║ ❌ Not built-in      ║ ✅ Learned             ║
║ Cost                       ║ No extra compute     ║ Extra training time   ║
║ Guaranteed?                ║ Yes (mathematical)   ║ Approximate (learned) ║
║ Generalisation to unseen   ║ Perfect for shifts   ║ Depends on aug range  ║
╚════════════════════════════╩══════════════════════╩═══════════════════════╝

Best practice: Use BOTH architectural invariance AND data augmentation.
```

---

## 7 — Beyond Translation: Other Invariance Types in CNNs

### 7.1 — Scale Invariance

CNNs are NOT inherently scale-invariant. Solutions:

```
Multi-scale approaches:

1. Image Pyramid:
   Process the image at multiple scales:
   ┌──────────────┐
   │ Full size    │ → CNN → features
   └──────────────┘
   ┌──────────┐
   │ 0.75×    │ → CNN → features      → Combine all
   └──────────┘
   ┌──────┐
   │ 0.5× │ → CNN → features
   └──────┘

2. Feature Pyramid Network (FPN):
   Use features from multiple conv layers (different RF sizes):
   
   Layer 1 features ─────→ Detect small objects (high res, small RF)
   Layer 3 features ─────→ Detect medium objects
   Layer 5 features ─────→ Detect large objects (low res, large RF)
```

### 7.2 — Rotation Invariance

Standard CNNs only have invariance to 0° and 180° rotations (via flipping augmentation). Specialised approaches:

| Approach | How It Works |
|---|---|
| **Data augmentation** | Train with random rotations |
| **Rotated filters** | Apply each filter at multiple orientations |
| **Group equivariant CNNs** | Filters that are mathematically equivariant to rotations |
| **Spatial Transformer Networks** | Learnable geometric transformations |

### 7.3 — Deformation Invariance

```
Ideal:   Recognise objects despite deformations

  😊  →  😄  →  🤪  →  😊
  (all should be "face")

Solutions:
  - Deformable convolutions: learn filter offsets
  - Capsule networks: encode pose information
  - Data augmentation: elastic deformations
```

---

## 8 — Practical Experiment: Measuring Invariance

### 8.1 — PyTorch Code: Quantifying Translation Invariance

```python
import torch
import torch.nn as nn
import numpy as np

class MeasureInvariance:
    """Measure how invariant a model's output is to input translations."""
    
    def __init__(self, model):
        self.model = model
        self.model.eval()
    
    def translate_image(self, x, shift_h, shift_w):
        """Translate image by (shift_h, shift_w) pixels with zero padding."""
        translated = torch.zeros_like(x)
        _, _, H, W = x.shape
        
        # Compute source and destination ranges
        src_h_start = max(0, -shift_h)
        src_h_end   = min(H, H - shift_h)
        src_w_start = max(0, -shift_w)
        src_w_end   = min(W, W - shift_w)
        
        dst_h_start = max(0, shift_h)
        dst_w_start = max(0, shift_w)
        
        h_len = src_h_end - src_h_start
        w_len = src_w_end - src_w_start
        
        translated[:, :, dst_h_start:dst_h_start+h_len, 
                        dst_w_start:dst_w_start+w_len] = \
            x[:, :, src_h_start:src_h_start+h_len, 
                    src_w_start:src_w_start+w_len]
        
        return translated
    
    def measure(self, x, max_shift=10):
        """Measure output consistency across translations."""
        with torch.no_grad():
            base_output = self.model(x)
            
            differences = []
            for dh in range(-max_shift, max_shift + 1, 2):
                for dw in range(-max_shift, max_shift + 1, 2):
                    if dh == 0 and dw == 0:
                        continue
                    x_shifted = self.translate_image(x, dh, dw)
                    shifted_output = self.model(x_shifted)
                    diff = (base_output - shifted_output).abs().mean().item()
                    differences.append(diff)
            
            return {
                'mean_difference': np.mean(differences),
                'max_difference': np.max(differences),
                'min_difference': np.min(differences),
                'std_difference': np.std(differences),
            }

# Compare invariance of CNN with/without pooling
class CNNWithPooling(nn.Module):
    def __init__(self):
        super().__init__()
        self.net = nn.Sequential(
            nn.Conv2d(1, 16, 3, padding=1), nn.ReLU(),
            nn.MaxPool2d(2),
            nn.Conv2d(16, 32, 3, padding=1), nn.ReLU(),
            nn.MaxPool2d(2),
            nn.AdaptiveAvgPool2d(1),
            nn.Flatten(),
            nn.Linear(32, 10),
        )
    def forward(self, x):
        return self.net(x)

class CNNWithoutPooling(nn.Module):
    def __init__(self):
        super().__init__()
        self.net = nn.Sequential(
            nn.Conv2d(1, 16, 3, padding=1), nn.ReLU(),
            nn.Conv2d(16, 32, 3, padding=1), nn.ReLU(),
            nn.Flatten(),
            nn.Linear(32 * 32 * 32, 10),
        )
    def forward(self, x):
        return self.net(x)

# Test
x = torch.randn(1, 1, 32, 32)

for model_cls, name in [(CNNWithPooling, "With Pooling"), 
                         (CNNWithoutPooling, "Without Pooling")]:
    model = model_cls()
    measurer = MeasureInvariance(model)
    results = measurer.measure(x, max_shift=5)
    
    print(f"\n{name}:")
    print(f"  Mean output difference: {results['mean_difference']:.4f}")
    print(f"  Max output difference:  {results['max_difference']:.4f}")
    print(f"  Std output difference:  {results['std_difference']:.4f}")
```

---

## 9 — Modern Perspectives on Invariance

### 9.1 — Vision Transformers (ViT) and Translation Invariance

Unlike CNNs, Vision Transformers do NOT have built-in translation equivariance:

```
CNN:  Translation equivariance from weight sharing (architectural)
ViT:  No inherent equivariance — must LEARN it from data

However:
  - ViTs trained on large datasets learn approximate invariance
  - Positional encodings help ViTs understand spatial relationships
  - Hybrid models (CNN stem + Transformer) get the best of both worlds
```

### 9.2 — Anti-Aliasing for True Invariance

Modern research (Zhang, 2019 — "Making Convolutional Networks Shift-Invariant Again") showed that standard pooling (especially stride > 1) actually **breaks** perfect shift invariance due to aliasing:

```
Standard Max Pooling with stride 2:

Input A:     [1, 5, 2, 8, 3]  → MaxPool(2,s=2) → [5, 8]
Input A+1:   [0, 1, 5, 2, 8]  → MaxPool(2,s=2) → [1, 8]  ← Different!

The shift changed which values land in which pooling windows.

Fix: Anti-aliased pooling (blur before subsampling):
Input A:     [1, 5, 2, 8, 3] → Blur → [3, 3.5, 5, 4.3, 5.5] → Subsample → [3, 5]
Input A+1:   [0, 1, 5, 2, 8] → Blur → [0.5, 2, 2.7, 5, 5]   → Subsample → [0.5, 5]

Still not perfect, but much more consistent.
```

### 9.3 — PyTorch Code: Anti-Aliased Pooling

```python
import torch
import torch.nn as nn
import torch.nn.functional as F

class BlurPool2d(nn.Module):
    """
    Anti-aliased downsampling: blur then subsample.
    From Zhang (2019): "Making Convolutional Networks Shift-Invariant Again"
    """
    def __init__(self, channels, stride=2):
        super().__init__()
        self.stride = stride
        # Binomial filter [1,2,1]/4 for anti-aliasing
        kernel_1d = torch.tensor([1, 2, 1], dtype=torch.float32) / 4
        kernel_2d = kernel_1d.unsqueeze(0) * kernel_1d.unsqueeze(1)  # outer product
        kernel_2d = kernel_2d.unsqueeze(0).unsqueeze(0)  # (1,1,3,3)
        kernel_2d = kernel_2d.repeat(channels, 1, 1, 1)  # (C,1,3,3)
        self.register_buffer('kernel', kernel_2d)
        self.channels = channels
    
    def forward(self, x):
        # Apply blur filter (depthwise convolution)
        x = F.pad(x, (1, 1, 1, 1), mode='reflect')
        x = F.conv2d(x, self.kernel, stride=self.stride, groups=self.channels)
        return x

class ShiftInvariantCNN(nn.Module):
    """CNN with anti-aliased pooling for better shift invariance."""
    def __init__(self):
        super().__init__()
        self.net = nn.Sequential(
            nn.Conv2d(1, 32, 3, padding=1), nn.ReLU(),
            nn.MaxPool2d(2, stride=1),    # Max without subsampling
            BlurPool2d(32, stride=2),      # Anti-aliased subsampling
            nn.Conv2d(32, 64, 3, padding=1), nn.ReLU(),
            nn.MaxPool2d(2, stride=1),
            BlurPool2d(64, stride=2),
            nn.AdaptiveAvgPool2d(1),
            nn.Flatten(),
            nn.Linear(64, 10),
        )
    
    def forward(self, x):
        return self.net(x)

# Test shift consistency
model = ShiftInvariantCNN()
model.eval()

x = torch.randn(1, 1, 32, 32)

with torch.no_grad():
    y1 = model(x)
    # Shift input by 1 pixel
    x_shifted = torch.zeros_like(x)
    x_shifted[:, :, 1:, 1:] = x[:, :, :-1, :-1]
    y2 = model(x_shifted)

print(f"Output diff after 1-pixel shift: {(y1-y2).abs().mean():.6f}")
```

---

## 10 — Complete Pipeline: Invariance in a Real CNN

### 10.1 — End-to-End Example

```python
import torch
import torch.nn as nn
import torch.optim as optim
from torchvision import datasets, transforms
from torch.utils.data import DataLoader

# Full pipeline demonstrating all invariance concepts
class InvariantClassifier(nn.Module):
    """
    CNN demonstrating:
    1. Translation equivariance (conv layers)
    2. Approximate invariance (pooling)  
    3. Complete spatial invariance (global average pooling)
    """
    def __init__(self, num_classes=10):
        super().__init__()
        
        # Equivariant feature extraction
        self.conv_block1 = nn.Sequential(
            nn.Conv2d(1, 32, 3, padding=1),   # Equivariant
            nn.BatchNorm2d(32),
            nn.ReLU(),
            nn.Conv2d(32, 32, 3, padding=1),  # Equivariant
            nn.BatchNorm2d(32),
            nn.ReLU(),
            nn.MaxPool2d(2, 2),                # Adds partial invariance
        )
        
        self.conv_block2 = nn.Sequential(
            nn.Conv2d(32, 64, 3, padding=1),
            nn.BatchNorm2d(64),
            nn.ReLU(),
            nn.Conv2d(64, 64, 3, padding=1),
            nn.BatchNorm2d(64),
            nn.ReLU(),
            nn.MaxPool2d(2, 2),
        )
        
        # Complete spatial invariance
        self.global_pool = nn.AdaptiveAvgPool2d(1)
        
        # Position-independent classification
        self.classifier = nn.Sequential(
            nn.Flatten(),
            nn.Linear(64, 128),
            nn.ReLU(),
            nn.Dropout(0.5),
            nn.Linear(128, num_classes),
        )
    
    def forward(self, x):
        x = self.conv_block1(x)   # (B, 32, 14, 14) — equivariant
        x = self.conv_block2(x)   # (B, 64, 7, 7)   — equivariant
        x = self.global_pool(x)   # (B, 64, 1, 1)   — invariant!
        x = self.classifier(x)    # (B, 10)
        return x

# Data augmentation for additional invariance
train_transform = transforms.Compose([
    transforms.RandomAffine(
        degrees=10,           # Rotation invariance
        translate=(0.1, 0.1), # Stronger translation invariance
        scale=(0.9, 1.1),     # Scale invariance
    ),
    transforms.ToTensor(),
    transforms.Normalize((0.1307,), (0.3081,)),
])

test_transform = transforms.Compose([
    transforms.ToTensor(),
    transforms.Normalize((0.1307,), (0.3081,)),
])

# Load MNIST
train_data = datasets.MNIST('./data', train=True, download=True, transform=train_transform)
test_data  = datasets.MNIST('./data', train=False, transform=test_transform)

train_loader = DataLoader(train_data, batch_size=128, shuffle=True)
test_loader  = DataLoader(test_data,  batch_size=128)

# Train
model = InvariantClassifier()
optimizer = optim.Adam(model.parameters(), lr=1e-3)
criterion = nn.CrossEntropyLoss()

for epoch in range(3):
    model.train()
    correct, total = 0, 0
    for images, labels in train_loader:
        optimizer.zero_grad()
        outputs = model(images)
        loss = criterion(outputs, labels)
        loss.backward()
        optimizer.step()
        correct += (outputs.argmax(1) == labels).sum().item()
        total += labels.size(0)
    
    # Test
    model.eval()
    test_correct, test_total = 0, 0
    with torch.no_grad():
        for images, labels in test_loader:
            outputs = model(images)
            test_correct += (outputs.argmax(1) == labels).sum().item()
            test_total += labels.size(0)
    
    print(f"Epoch {epoch+1}: Train={100*correct/total:.1f}% Test={100*test_correct/test_total:.1f}%")

print(f"\nModel params: {sum(p.numel() for p in model.parameters()):,}")
```

---

## 📊 Summary Table

| Concept | Key Idea |
|---|---|
| Translation Equivariance | Output shifts when input shifts. Convolution provides this via weight sharing. |
| Translation Invariance | Output stays the same when input shifts. Achieved through pooling. |
| Mathematical Proof | Conv(Translate(x)) = Translate(Conv(x)) because same weights at every position |
| FC Layers | NOT equivariant — different weights at different positions |
| Max Pooling | Provides approximate invariance within pooling window |
| Global Average Pooling | Provides complete spatial invariance (sums over all positions) |
| Invariance–Precision Trade-off | More invariance = less spatial detail (classification vs detection) |
| Data Augmentation | Provides learned invariance to rotations, scaling, colour changes |
| Anti-Aliased Pooling | Fixes aliasing in strided operations for better shift consistency |
| Task-Dependent Choice | Classification needs invariance; detection/segmentation need equivariance |

---

## ❓ Revision Questions

### Q1: Equivariance vs Invariance
Explain the difference between translation equivariance and translation invariance. Give one CNN component that provides each.

<details>
<summary>Answer</summary>

**Translation equivariance:** If the input shifts by (Δh, Δw), the output shifts by the same amount. The output "follows" the input.
→ **Convolutional layers** provide equivariance because the same filter weights are applied everywhere.

**Translation invariance:** The output remains unchanged regardless of where the input feature appears.
→ **Global Average Pooling** provides invariance because it sums over all spatial positions, producing the same result regardless of feature location.

Key relationship: **Equivariance + Pooling → Approximate Invariance**
</details>

### Q2: The Proof
Sketch the proof that convolution is equivariant to translation. What property of convolution makes this possible?

<details>
<summary>Answer</summary>

**Proof sketch:**

Let T_Δ be translation by Δ, * be convolution:

```
Conv(T_Δ[x])[m,n] = Σ_k,l w[k,l] × x[m+k-Δh, n+l-Δw]

T_Δ[Conv(x)][m,n] = Conv(x)[m-Δh, n-Δw]
                   = Σ_k,l w[k,l] × x[(m-Δh)+k, (n-Δw)+l]
                   = Σ_k,l w[k,l] × x[m+k-Δh, n+l-Δw]
```

Both expressions are identical. ∎

The key property is **weight sharing** — the filter w[k,l] is the same at every position. If weights varied by position (as in FC layers), the two expressions would not be equal.
</details>

### Q3: Pooling and Invariance
Why does max pooling provide only *approximate* translation invariance? Give a concrete counter-example.

<details>
<summary>Answer</summary>

Max pooling provides approximate invariance only for shifts **smaller than the pooling window**. Shifts larger than the window move features into different pooling regions.

**Counter-example with 2×2 max pool, stride 2:**
```
Input A: [1, 8 | 2, 3]  → Max pool → [8, 3]
Input B: [1, 2 | 8, 3]  → Max pool → [2, 8]  ← Different!
```

Shifting the value 8 by one position moved it across the pooling boundary, changing the output. Additionally, even within a window, max pooling with stride > 1 can cause aliasing — the subsampled output depends on which values align with the sampling grid.
</details>

### Q4: Task-Specific Design
You're building two models: (a) an image classifier, and (b) a semantic segmentation model. How would their architectures differ regarding invariance?

<details>
<summary>Answer</summary>

**(a) Classifier:**
- Use aggressive pooling to reduce spatial dimensions
- End with **Global Average Pooling** for complete translation invariance
- Architecture: Conv → Pool → Conv → Pool → ... → GAP → FC → class
- Example: ResNet-50

**(b) Segmentation model:**
- Need to **preserve spatial detail** (output must be same resolution as input)
- Use **equivariant** layers throughout (conv without heavy pooling)
- Use **skip connections** to recover fine spatial info lost to pooling
- End with a Conv layer (not GAP!) that outputs per-pixel predictions
- Architecture: Encoder (conv+pool) → Decoder (conv+upsample) + skip connections
- Example: U-Net
</details>

### Q5: Augmentation Gaps
List three types of invariance that CNNs do NOT provide architecturally and explain how data augmentation addresses each.

<details>
<summary>Answer</summary>

1. **Rotation invariance:** Standard conv filters are axis-aligned; rotating an image activates different weights.
   → **Random rotation augmentation** (±15–30°) trains the network on rotated versions, teaching it to recognise rotated features.

2. **Scale invariance:** A conv filter has a fixed receptive field; objects at different scales activate different amounts of the RF.
   → **Random resized crop / random zoom** shows the network objects at various scales during training.

3. **Colour/lighting invariance:** Conv filters are sensitive to absolute pixel values, which change with lighting conditions.
   → **Colour jitter** (brightness, contrast, saturation, hue variations) teaches the network to recognise objects under different lighting.

These are "learned invariances" rather than "guaranteed invariances" — they work well in practice but only for transformations within the augmentation range.
</details>

### Q6: Anti-Aliasing
Explain why standard stride-2 pooling breaks strict translation equivariance. How does anti-aliased pooling (blur + subsample) help?

<details>
<summary>Answer</summary>

**Why stride-2 breaks equivariance:**
Stride-2 subsampling selects every other value. A 1-pixel shift changes which values are selected:

```
Signal:         [a, b, c, d, e, f, g, h]
Subsample(s=2): [a,    c,    e,    g]

Shifted signal: [-, a, b, c, d, e, f, g]
Subsample(s=2): [-,    b,    d,    f]    ← Completely different values!
```

This is the **Nyquist aliasing** problem — subsampling without low-pass filtering causes high-frequency components to alias.

**Anti-aliased pooling:** Applies a blur (low-pass filter) before subsampling. This smooths the signal so that small shifts don't drastically change the subsampled values:

```
Signal:         [a, b, c, d, e, f, g, h]
Blur:           [a', b', c', d', e', f', g', h']  (smooth)
Subsample:      [a',     c',     e',     g']

Shifted:        [-, a, b, c, d, e, f, g]
Blur:           [-', a'', b'', c'', d'', e'', f'', g'']
Subsample:      [-',      b'',      d'',      f'']

Now the values change gradually with shift (smooth → similar subsamples).
```
</details>

---

## 🔗 Navigation

| | |
|---|---|
| ← Previous | [Parameter Sharing](./04-parameter-sharing.md) |
| → Next | *This is the last chapter in this unit* |
| 🏠 Unit | [01 – Introduction to CNNs](./README.md) |

---

*© 2025 AIML Study Notes. For educational use.*
