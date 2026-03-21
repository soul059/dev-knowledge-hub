# 🔗 Fully Connected (FC) Layers — The Classifier Head

[← Previous: 03 — Pooling Layer](03-pooling-layer.md) | [📑 Unit Overview](../README.md) | [Next → 05 — Flatten Operation](05-flatten-operation.md)

---

## 📋 Chapter Overview

Fully Connected (FC) layers sit at the **end of a CNN**, acting as the **classifier head**. They take the high-level features extracted by convolutional layers and map them to the final output — class scores, regression values, or embeddings. While essential for classification, FC layers are the most **parameter-heavy** component of traditional CNNs, and modern architectures increasingly replace them with Global Average Pooling.

**What you'll learn:**
- The role of FC layers in CNN architectures
- The `Flatten → Linear → Activation → Linear` pattern
- Why FC layers are parameter-heavy and how to mitigate this
- The modern trend of replacing FC layers with GAP
- Dropout as regularization before FC layers
- `nn.Linear` in PyTorch with worked examples

---

## 1️⃣ What Is a Fully Connected Layer?

A fully connected (dense) layer connects **every input neuron** to **every output neuron**. Unlike convolutional layers which exploit spatial locality and weight sharing, FC layers have no spatial awareness — they treat the input as a flat vector.

```
Convolutional Layer:                    Fully Connected Layer:
  Each output neuron sees               Each output neuron sees
  a LOCAL region of input               the ENTIRE input

  ┌─────────┐                           ┌─────────┐
  │ x x x . │ ← local receptive        │ x x x x │ ← ALL inputs
  │ x x x . │    field (3×3)            │ x x x x │    connected
  │ x x x . │                           │ x x x x │
  │ . . . . │                           │ x x x x │
  └─────────┘                           └─────────┘
       ↓                                     ↓
   One output                           One output
   (shared weights)                     (unique weights for each connection)
```

### Mathematical Definition

```
y = Wx + b

Where:
  x ∈ ℝ^n      (input vector, n features)
  W ∈ ℝ^(m×n)  (weight matrix, m outputs × n inputs)
  b ∈ ℝ^m      (bias vector, m outputs)
  y ∈ ℝ^m      (output vector, m features)

Each output neuron:
  y_j = Σ_{i=1}^{n} W_{ji} · x_i + b_j
```

---

## 2️⃣ The Classic FC Pattern: Flatten → Linear → Activation → Linear

### Traditional CNN Classifier (VGG, AlexNet)

```
Feature Extraction (Conv layers)        Classifier (FC layers)
┌───────────────────────────┐           ┌──────────────────────────────────┐
│                           │           │                                  │
│  Conv→BN→ReLU→Pool       │           │  Flatten → FC → ReLU → Dropout  │
│  Conv→BN→ReLU→Pool       │───────►   │         → FC → ReLU → Dropout  │
│  Conv→BN→ReLU→Pool       │           │         → FC (output)           │
│  ...                     │           │                                  │
└───────────────────────────┘           └──────────────────────────────────┘
  Output: (N, C, H, W)                   Output: (N, num_classes)
  e.g., (N, 512, 7, 7)                   e.g., (N, 1000)
```

### Step-by-Step Data Flow

```
Step 1: Feature maps from last conv layer
  Shape: (N, 512, 7, 7)
  
Step 2: Flatten — convert 3D feature maps to 1D vector
  Shape: (N, 512 × 7 × 7) = (N, 25088)
  
Step 3: FC Layer 1 — reduce dimensionality
  Linear(25088, 4096)
  Shape: (N, 4096)
  
Step 4: ReLU activation
  Shape: (N, 4096) — unchanged
  
Step 5: Dropout (p=0.5)
  Shape: (N, 4096) — unchanged (some neurons zeroed)
  
Step 6: FC Layer 2 — further compression
  Linear(4096, 4096)
  Shape: (N, 4096)
  
Step 7: ReLU + Dropout
  Shape: (N, 4096)
  
Step 8: FC Layer 3 — final classification
  Linear(4096, 1000)  ← 1000 classes for ImageNet
  Shape: (N, 1000)
  
Step 9: Softmax (applied in loss function, not model)
  Shape: (N, 1000) — probabilities
```

### PyTorch Implementation (VGG-style)

```python
import torch
import torch.nn as nn

class VGGClassifier(nn.Module):
    """Classic VGG-style classifier head."""
    
    def __init__(self, num_classes=1000):
        super().__init__()
        self.classifier = nn.Sequential(
            nn.Linear(512 * 7 * 7, 4096),   # 25088 → 4096
            nn.ReLU(inplace=True),
            nn.Dropout(p=0.5),
            nn.Linear(4096, 4096),            # 4096 → 4096
            nn.ReLU(inplace=True),
            nn.Dropout(p=0.5),
            nn.Linear(4096, num_classes),     # 4096 → 1000
        )
    
    def forward(self, x):
        x = torch.flatten(x, 1)  # (N, 512, 7, 7) → (N, 25088)
        x = self.classifier(x)
        return x

# Test
classifier = VGGClassifier(num_classes=1000)
features = torch.randn(4, 512, 7, 7)
output = classifier(features)
print(f"Output shape: {output.shape}")  # [4, 1000]
```

---

## 3️⃣ Parameter Count — The FC Problem

FC layers are **overwhelmingly** the most parameter-heavy part of traditional CNNs.

### VGG-16 Parameter Breakdown

```
Layer                     Parameters      % of Total
──────────────────────────────────────────────────────
Conv layers (13 layers)   14,714,688       10.6%
FC1: 25088 → 4096        102,764,544      74.3%   ← !!!!
FC2: 4096 → 4096         16,781,312       12.1%
FC3: 4096 → 1000         4,097,000         3.0%
──────────────────────────────────────────────────────
Total                     138,357,544      100%

FC layers: 123,642,856 parameters = 89.4% of the entire model!
Conv layers: 14,714,688 parameters = 10.6%
```

### ASCII Visualization of Parameter Distribution

```
VGG-16 Parameters Distribution:

Conv Layers   ███░░░░░░░░░░░░░░░░░░░░░░░░░░░  10.6%
FC Layer 1    ████████████████████████████████  74.3%  ← Dominant!
FC Layer 2    ████████░░░░░░░░░░░░░░░░░░░░░░░  12.1%
FC Layer 3    █░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░   3.0%
```

### Why So Many Parameters?

```
FC1: Linear(25088, 4096)
  Weights: 25,088 × 4,096 = 102,760,448
  Biases:  4,096
  Total:   102,764,544

This is because:
  - Input is 512 × 7 × 7 = 25,088 dimensions (after flattening)
  - EVERY input connects to EVERY output
  - No weight sharing (unlike conv layers)
```

### Parameter Count Formula

```
nn.Linear(in_features, out_features, bias=True)

Parameters = in_features × out_features + out_features
             ─────────────────────────   ─────────────
                    weights                  biases

Without bias:
Parameters = in_features × out_features
```

---

## 4️⃣ `nn.Linear` — PyTorch's FC Layer

### API Reference

```python
torch.nn.Linear(
    in_features,    # Size of each input sample
    out_features,   # Size of each output sample
    bias=True       # If True, adds a learnable bias
)
```

### Weight Shape and Initialization

```python
import torch.nn as nn

fc = nn.Linear(512, 256, bias=True)

print(f"Weight shape: {fc.weight.shape}")  # [256, 512] — note: (out, in)
print(f"Bias shape:   {fc.bias.shape}")    # [256]
print(f"Parameters:   {fc.weight.numel() + fc.bias.numel()}")  # 131,328

# Default initialization: uniform distribution
# W ~ U(-1/√in_features, 1/√in_features)
# b ~ U(-1/√in_features, 1/√in_features)
```

### Worked Example: Forward Pass

```
Input:  x = [0.5, -1.0, 2.0]    (3 features)

Weight: W = [[0.2, -0.3, 0.1],   (2×3 matrix for 2 outputs)
             [0.4,  0.5, -0.2]]

Bias:   b = [0.1, -0.1]

Output: y = Wx + b

y[0] = 0.2×0.5 + (-0.3)×(-1.0) + 0.1×2.0 + 0.1
     = 0.1 + 0.3 + 0.2 + 0.1
     = 0.7

y[1] = 0.4×0.5 + 0.5×(-1.0) + (-0.2)×2.0 + (-0.1)
     = 0.2 - 0.5 - 0.4 - 0.1
     = -0.8

Output: y = [0.7, -0.8]
```

### Verify in PyTorch

```python
import torch
import torch.nn as nn

fc = nn.Linear(3, 2)

# Set weights manually
with torch.no_grad():
    fc.weight.copy_(torch.tensor([[0.2, -0.3, 0.1],
                                   [0.4,  0.5, -0.2]]))
    fc.bias.copy_(torch.tensor([0.1, -0.1]))

x = torch.tensor([[0.5, -1.0, 2.0]])
y = fc(x)
print(y)  # tensor([[ 0.7000, -0.8000]])
```

---

## 5️⃣ Dropout Before FC Layers — Regularization

### Why Dropout Is Critical for FC Layers

FC layers are prone to **overfitting** because they have so many parameters. Dropout randomly zeroes out neurons during training, forcing the network to learn redundant representations.

```
Without Dropout:                    With Dropout (p=0.5):
┌─────────────────┐                 ┌─────────────────┐
│ o o o o o o o o │                 │ o x o x o o x o │
│ │╲│╲│╲│╲│╲│╲│╲│ │   Training     │ │   │   │ │   │ │
│ o o o o o o o o │  ──────────►    │ o x o o x o o x │
│ │╲│╲│╲│╲│╲│╲│╲│ │                 │ │   │ │   │ │   │
│ o o o o o o o o │                 │ o o x o o x o o │
└─────────────────┘                 └─────────────────┘
All connections active              x = dropped neurons (output = 0)
                                    Each neuron dropped with prob p

At test time: ALL neurons active, weights scaled by (1-p)
PyTorch handles scaling automatically.
```

### Mathematical Formulation

```
Training:
  mask ~ Bernoulli(1 - p)    (each element independently)
  y = (x * mask) / (1 - p)   (inverted dropout — scale up to maintain expectation)

Testing:
  y = x                       (no dropout, no scaling needed)

E[y_train] = E[x * mask / (1-p)] = x · (1-p) / (1-p) = x = E[y_test]  ✓
```

### PyTorch Implementation

```python
import torch
import torch.nn as nn

class ClassifierWithDropout(nn.Module):
    def __init__(self, in_features=512, num_classes=10, dropout_rate=0.5):
        super().__init__()
        self.classifier = nn.Sequential(
            nn.Dropout(p=dropout_rate),      # dropout BEFORE first FC
            nn.Linear(in_features, 256),
            nn.ReLU(inplace=True),
            nn.Dropout(p=dropout_rate),      # dropout between FC layers
            nn.Linear(256, num_classes),     # no dropout before final output
        )
    
    def forward(self, x):
        return self.classifier(x)

model = ClassifierWithDropout()

# Training mode — dropout active
model.train()
x = torch.randn(4, 512)
out_train1 = model(x)
out_train2 = model(x)
print(torch.equal(out_train1, out_train2))  # False — different dropout masks

# Eval mode — dropout disabled
model.eval()
out_eval1 = model(x)
out_eval2 = model(x)
print(torch.equal(out_eval1, out_eval2))  # True — deterministic
```

### Dropout Placement Guidelines

```
Placement Strategy:

  ✓  Dropout BEFORE FC layers (common in modern practice)
  ✓  Dropout BETWEEN FC layers (traditional, VGG-style)
  ✗  Dropout AFTER the final FC layer (pointless — output needs all info)
  ✗  Dropout in conv layers (use spatial dropout or just BN instead)

Common rates:
  p = 0.5   Traditional default (VGG, AlexNet)
  p = 0.2   More conservative (modern models)
  p = 0.0   No dropout (when using heavy data augmentation + BN)
```

---

## 6️⃣ Modern Trend: Replace FC with GAP

### The Paradigm Shift

```
Traditional (VGG, AlexNet):
  Conv features → Flatten → FC → FC → FC → Output
  Problem: FC layers dominate parameter count

Modern (ResNet, EfficientNet, MobileNet):
  Conv features → GAP → Single FC → Output
  Benefit: Massive parameter reduction
```

### Comparison

```python
import torch.nn as nn

# ❌ Traditional: Flatten + Multiple FC layers
class TraditionalClassifier(nn.Module):
    def __init__(self):
        super().__init__()
        self.flatten = nn.Flatten()
        self.fc1 = nn.Linear(512 * 7 * 7, 4096)  # 102M params!
        self.fc2 = nn.Linear(4096, 4096)            # 16M params
        self.fc3 = nn.Linear(4096, 1000)            # 4M params
    # Total FC params: ~123M

# ✅ Modern: GAP + Single FC layer
class ModernClassifier(nn.Module):
    def __init__(self):
        super().__init__()
        self.gap = nn.AdaptiveAvgPool2d(1)
        self.flatten = nn.Flatten()
        self.fc = nn.Linear(512, 1000)              # 0.5M params
    # Total FC params: ~0.5M  (246× fewer!)
```

### Why GAP Works

```
GAP compresses each feature map to a single number by averaging:

Feature Map (512 × 7 × 7):
  Channel 1: 7×7 spatial map → average → 1 scalar
  Channel 2: 7×7 spatial map → average → 1 scalar
  ...
  Channel 512: 7×7 spatial map → average → 1 scalar

Result: 512-dimensional vector

Each channel represents a learned feature detector.
The average tells "how much of this feature is present in the image."
This is a natural, interpretable summary — no need for 25088→4096 FC.
```

### ResNet Classifier Head (Modern Standard)

```python
class ResNetHead(nn.Module):
    """Modern classifier head used in ResNet, EfficientNet, etc."""
    
    def __init__(self, in_channels=2048, num_classes=1000, dropout=0.0):
        super().__init__()
        self.avgpool = nn.AdaptiveAvgPool2d(1)
        self.dropout = nn.Dropout(p=dropout) if dropout > 0 else nn.Identity()
        self.fc = nn.Linear(in_channels, num_classes)
    
    def forward(self, x):
        x = self.avgpool(x)      # (N, 2048, H, W) → (N, 2048, 1, 1)
        x = torch.flatten(x, 1)  # (N, 2048)
        x = self.dropout(x)
        x = self.fc(x)           # (N, num_classes)
        return x

head = ResNetHead(in_channels=2048, num_classes=1000)
params = sum(p.numel() for p in head.parameters())
print(f"ResNet head parameters: {params:,}")
# ResNet head parameters: 2,049,000  (vs VGG's 123M!)
```

---

## 7️⃣ FC Layers for Different Tasks

### Classification

```python
# Multi-class classification (e.g., ImageNet)
classifier = nn.Linear(512, 1000)  # 1000 classes
# Loss: nn.CrossEntropyLoss()  (includes softmax internally)

# Binary classification
classifier = nn.Linear(512, 1)     # single output
# Loss: nn.BCEWithLogitsLoss()  (includes sigmoid internally)
```

### Regression

```python
# Single value regression (e.g., age estimation)
regressor = nn.Linear(512, 1)
# Loss: nn.MSELoss() or nn.L1Loss()

# Multi-value regression (e.g., bounding box: x, y, w, h)
regressor = nn.Linear(512, 4)
# Loss: nn.SmoothL1Loss()
```

### Embedding / Feature Extraction

```python
# Learn a 128-d embedding (e.g., face recognition)
embedder = nn.Sequential(
    nn.Linear(2048, 512),
    nn.ReLU(inplace=True),
    nn.Linear(512, 128),
    # No activation — raw embedding
)
# Loss: TripletMarginLoss or contrastive loss
```

### Multi-Task Head

```python
class MultiTaskHead(nn.Module):
    """Shared backbone → multiple FC heads for different tasks."""
    
    def __init__(self, in_features=512):
        super().__init__()
        self.classification_head = nn.Linear(in_features, 10)
        self.regression_head = nn.Linear(in_features, 4)
        self.embedding_head = nn.Linear(in_features, 128)
    
    def forward(self, features):
        cls_out = self.classification_head(features)
        reg_out = self.regression_head(features)
        emb_out = self.embedding_head(features)
        return cls_out, reg_out, emb_out
```

---

## 8️⃣ Complete Example — Full CNN with FC Classifier

```python
import torch
import torch.nn as nn

class SimpleCNN(nn.Module):
    """Complete CNN showing the transition from conv → FC layers."""
    
    def __init__(self, num_classes=10):
        super().__init__()
        
        # Feature extractor (convolutional layers)
        self.features = nn.Sequential(
            # Block 1
            nn.Conv2d(3, 32, 3, padding=1, bias=False),
            nn.BatchNorm2d(32),
            nn.ReLU(inplace=True),
            nn.MaxPool2d(2, 2),            # 32×32 → 16×16
            
            # Block 2
            nn.Conv2d(32, 64, 3, padding=1, bias=False),
            nn.BatchNorm2d(64),
            nn.ReLU(inplace=True),
            nn.MaxPool2d(2, 2),            # 16×16 → 8×8
            
            # Block 3
            nn.Conv2d(64, 128, 3, padding=1, bias=False),
            nn.BatchNorm2d(128),
            nn.ReLU(inplace=True),
            nn.MaxPool2d(2, 2),            # 8×8 → 4×4
        )
        
        # Classifier (FC layers)
        # After features: (N, 128, 4, 4) → flatten → (N, 2048)
        self.classifier = nn.Sequential(
            nn.Flatten(),
            nn.Dropout(0.5),
            nn.Linear(128 * 4 * 4, 256),   # 2048 → 256
            nn.ReLU(inplace=True),
            nn.Dropout(0.3),
            nn.Linear(256, num_classes),    # 256 → 10
        )
    
    def forward(self, x):
        x = self.features(x)
        x = self.classifier(x)
        return x

# Create and analyze model
model = SimpleCNN(num_classes=10)
x = torch.randn(4, 3, 32, 32)
output = model(x)
print(f"Output: {output.shape}")  # [4, 10]

# Parameter breakdown
conv_params = sum(p.numel() for n, p in model.named_parameters() if 'features' in n)
fc_params = sum(p.numel() for n, p in model.named_parameters() if 'classifier' in n)
total = conv_params + fc_params

print(f"\nParameter Breakdown:")
print(f"  Conv layers:   {conv_params:>10,} ({100*conv_params/total:.1f}%)")
print(f"  FC layers:     {fc_params:>10,} ({100*fc_params/total:.1f}%)")
print(f"  Total:         {total:>10,}")

# Output:
# Conv layers:      75,072 (12.2%)
# FC layers:       527,626 (85.8%) ← FC still dominates even in small model!
# (BN params account for remaining ~2%)
```

### Same Model with GAP (Modern Approach)

```python
class ModernCNN(nn.Module):
    """Same feature extractor but with GAP instead of flatten+FC."""
    
    def __init__(self, num_classes=10):
        super().__init__()
        
        self.features = nn.Sequential(
            nn.Conv2d(3, 32, 3, padding=1, bias=False),
            nn.BatchNorm2d(32),
            nn.ReLU(inplace=True),
            nn.MaxPool2d(2, 2),
            nn.Conv2d(32, 64, 3, padding=1, bias=False),
            nn.BatchNorm2d(64),
            nn.ReLU(inplace=True),
            nn.MaxPool2d(2, 2),
            nn.Conv2d(64, 128, 3, padding=1, bias=False),
            nn.BatchNorm2d(128),
            nn.ReLU(inplace=True),
            # No final MaxPool — GAP handles it
        )
        
        # Modern classifier: GAP + single FC
        self.classifier = nn.Sequential(
            nn.AdaptiveAvgPool2d(1),    # (N, 128, H, W) → (N, 128, 1, 1)
            nn.Flatten(),               # (N, 128)
            nn.Dropout(0.2),
            nn.Linear(128, num_classes), # (N, 10)
        )
    
    def forward(self, x):
        x = self.features(x)
        x = self.classifier(x)
        return x

model_modern = ModernCNN(num_classes=10)
fc_params_modern = sum(p.numel() for n, p in model_modern.named_parameters() 
                       if 'classifier' in n)
print(f"Modern FC params: {fc_params_modern:,}")
# Modern FC params: 1,290  (vs 527,626 in traditional → 409× fewer!)
```

---

## 9️⃣ Transfer Learning and FC Layers

When using pretrained models, you typically **replace only the FC head**:

```python
import torchvision.models as models

# Load pretrained ResNet-18
model = models.resnet18(weights='IMAGENET1K_V1')

# The original FC layer: Linear(512, 1000)
print(model.fc)  # Linear(in_features=512, out_features=1000)

# Replace for your task (e.g., 5 classes)
model.fc = nn.Sequential(
    nn.Dropout(0.3),
    nn.Linear(512, 5),  # your number of classes
)

# Freeze conv layers (optional — for feature extraction)
for param in model.parameters():
    param.requires_grad = False
# Unfreeze only the new FC head
for param in model.fc.parameters():
    param.requires_grad = True

# Now only the FC head trains
trainable = sum(p.numel() for p in model.parameters() if p.requires_grad)
total = sum(p.numel() for p in model.parameters())
print(f"Training {trainable:,} / {total:,} parameters ({100*trainable/total:.1f}%)")
# Training 2,565 / 11,179,077 parameters (0.02%!)
```

---

## 📊 Summary Table

| Property | Details |
|----------|---------|
| **Purpose** | Map extracted features to task output (classes, values, embeddings) |
| **PyTorch** | `nn.Linear(in_features, out_features, bias=True)` |
| **Weight shape** | `(out_features, in_features)` |
| **Param count** | `in_features × out_features + out_features` (with bias) |
| **Classic pattern** | `Flatten → FC → ReLU → Dropout → FC → ReLU → Dropout → FC` |
| **Modern pattern** | `GAP → Flatten → (Dropout) → FC` |
| **VGG FC params** | ~123M out of 138M total (~89%) |
| **ResNet FC params** | ~2M out of 25M total (~8%) — thanks to GAP |
| **Dropout** | Applied **before** or **between** FC layers (p=0.2–0.5) |
| **Transfer learning** | Replace final FC layer, freeze conv layers |
| **Not used in** | Fully convolutional networks (FCN) for segmentation |

---

## ❓ Revision Questions

### Q1: Parameter Count
**Calculate the total parameters in: `Linear(2048, 512)` → `ReLU` → `Linear(512, 100)`. Include biases.**

<details>
<summary>Answer</summary>

Layer 1: `Linear(2048, 512)`
- Weights: 2,048 × 512 = 1,048,576
- Biases: 512
- Subtotal: 1,049,088

ReLU: 0 parameters

Layer 2: `Linear(512, 100)`
- Weights: 512 × 100 = 51,200
- Biases: 100
- Subtotal: 51,300

**Total: 1,049,088 + 51,300 = 1,100,388 parameters**
</details>

### Q2: VGG vs ResNet FC
**Why does ResNet have ~246× fewer FC parameters than VGG despite having better accuracy?**

<details>
<summary>Answer</summary>

**VGG** uses Flatten + 3 FC layers:
- Flatten: 512×7×7 = 25,088 features
- FC1: 25,088→4,096 = ~103M params
- FC2: 4,096→4,096 = ~17M params
- FC3: 4,096→1,000 = ~4M params
- Total FC: ~123M

**ResNet** uses Global Average Pooling + 1 FC layer:
- GAP: 512→512 (no parameters)
- FC: 512→1,000 = ~0.5M params

The key difference is **GAP replaces Flatten**. GAP reduces each channel to a single scalar (averaging the spatial dimensions), resulting in only 512 features instead of 25,088. This eliminates the need for the massive FC1 layer. The spatial information is effectively summarized by the channel-wise averages.
</details>

### Q3: Dropout Behavior
**What happens if you forget to call `model.eval()` during inference? How does this affect predictions?**

<details>
<summary>Answer</summary>

If you forget `model.eval()`:
1. **Dropout stays active** — randomly zeroing neurons during inference, making predictions non-deterministic and generally worse
2. **BatchNorm uses batch statistics** instead of running statistics, causing inconsistent predictions especially for small batches

The predictions will be:
- **Non-deterministic**: Same input → different output each time (due to random dropout)
- **Noisy**: Some neurons are randomly disabled, degrading feature quality
- **Biased**: Though inverted dropout scales activations, the randomness still hurts

Always call `model.eval()` before inference and `model.train()` before training.
</details>

### Q4: Design Decision
**You're building a CNN for 32×32 images with 5 classes. Should you use the VGG-style classifier (Flatten+FC+FC+FC) or the modern approach (GAP+FC)? Justify your answer.**

<details>
<summary>Answer</summary>

Use the **modern approach (GAP + single FC)** because:
1. **Fewer parameters**: For small datasets (which often accompany small images), fewer parameters mean less overfitting
2. **Input size flexibility**: GAP works with any spatial resolution, making the model more robust
3. **Regularization**: GAP acts as structural regularization by forcing feature maps to be spatially meaningful
4. **Proven effectiveness**: Modern architectures (ResNet, EfficientNet, MobileNet) all use this approach

The VGG-style classifier with multiple FC layers would be massive overkill for a 5-class problem on 32×32 images — the FC layers would likely have more parameters than the conv layers, leading to severe overfitting.
</details>

### Q5: Transfer Learning FC
**When fine-tuning a pretrained ResNet for a new task with 20 classes, why do you replace only `model.fc` and not all FC layers?**

<details>
<summary>Answer</summary>

ResNet has only **one FC layer** (`model.fc`), which is the final classification layer mapping features to ImageNet's 1000 classes. All other layers are convolutional.

You replace this single `model.fc = nn.Linear(512, 20)` because:
- The **conv layers** have learned general features (edges, textures, shapes) that transfer well across tasks
- Only the **final FC layer** is task-specific (mapping features to class scores)
- The new FC layer needs to output 20 scores instead of 1000

If the new task is very different from ImageNet, you might fine-tune some of the later conv layers too, but the FC replacement is always the minimum change needed.
</details>

### Q6: FC in Segmentation
**Why are FC layers NOT used in Fully Convolutional Networks (FCNs) for semantic segmentation?**

<details>
<summary>Answer</summary>

FC layers produce a **fixed-size output vector** (e.g., 1000 class scores), but segmentation needs a **spatial output** — a class prediction for every pixel. FC layers destroy spatial information by flattening.

FCNs replace FC layers with **1×1 convolutions**, which:
- Preserve spatial dimensions (output is H×W, not a vector)
- Produce per-pixel class predictions
- Accept any input size (not fixed to 224×224)
- Are equivalent to FC layers when applied at a single spatial position, but generalize to arbitrary spatial sizes

The conversion: `nn.Linear(4096, 1000)` ↔ `nn.Conv2d(4096, 1000, 1)` when applied to a 1×1 spatial map.
</details>

---

[← Previous: 03 — Pooling Layer](03-pooling-layer.md) | [📑 Unit Overview](../README.md) | [Next → 05 — Flatten Operation](05-flatten-operation.md)
