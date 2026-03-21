# 📐 Flatten Operation — Bridging Conv and FC Layers

[← Previous: 04 — Fully Connected Layer](04-fully-connected-layer.md) | [📑 Unit Overview](../README.md) | [Next → 06 — Batch Normalization in CNNs](06-batch-normalization-cnns.md)

---

## 📋 Chapter Overview

The **flatten operation** converts multi-dimensional feature maps (3D tensors of shape `C × H × W`) into a 1D vector, enabling them to be fed into fully connected (linear) layers. While conceptually simple, understanding the exact dimension calculations and the different ways to flatten in PyTorch is essential for building and debugging CNN architectures.

**What you'll learn:**
- Why flattening is necessary between conv and FC layers
- How 3D feature maps become 1D vectors
- `torch.flatten()` vs `nn.Flatten()` vs `.view()` vs `.reshape()`
- Dimension calculations for the subsequent linear layer
- Common pitfalls and debugging strategies
- When flatten is NOT needed (modern architectures with GAP)

---

## 1️⃣ Why Flatten Is Necessary

### The Shape Mismatch Problem

Convolutional layers output **3D tensors** (channels × height × width), but fully connected layers expect **1D vectors** (a flat list of numbers). Flatten bridges this gap.

```
Conv Layer Output                    Flatten                 FC Layer Input
(C × H × W)                         ────────►               (C * H * W)

┌─────────────────┐                                          ┌─────────────┐
│ Channel 0       │                                          │             │
│ ┌───┬───┬───┐   │                                          │  x₁         │
│ │ a │ b │ c │   │                                          │  x₂         │
│ ├───┼───┼───┤   │          Reshape into                    │  x₃         │
│ │ d │ e │ f │   │          single vector:                  │  ...        │
│ └───┴───┴───┘   │                                          │  x₉         │
│                 │          [a,b,c,d,e,f,                    │  x₁₀        │
│ Channel 1       │           g,h,i,j,k,l,                   │  x₁₁        │
│ ┌───┬───┬───┐   │           m,n,o,p,q,r]                   │  ...        │
│ │ g │ h │ i │   │                                          │  x₁₈        │
│ ├───┼───┼───┤   │          Total length:                   │             │
│ │ j │ k │ l │   │          2 × 3 × 3 = 18                  └─────────────┘
│ └───┴───┴───┘   │
│                 │
│ Channel 2       │          Shape transformation:
│  (not shown)    │          (N, 2, 3, 3) → (N, 18)
└─────────────────┘
```

### Without Flatten — Error!

```python
import torch
import torch.nn as nn

features = torch.randn(4, 128, 8, 8)  # batch=4, 128 channels, 8×8 spatial
fc = nn.Linear(128 * 8 * 8, 256)

# ❌ Direct connection fails
try:
    output = fc(features)
except RuntimeError as e:
    print(f"Error: {e}")
    # RuntimeError: mat1 and mat2 shapes cannot be multiplied (32x64 and 8192x256)

# ✅ Flatten first
features_flat = torch.flatten(features, start_dim=1)  # (4, 8192)
output = fc(features_flat)  # (4, 256) ✓
```

---

## 2️⃣ How Flatten Works — Memory Layout

### Tensor Memory Layout (Row-Major / C-Contiguous)

PyTorch tensors are stored in **contiguous memory** in row-major order. For a 4D tensor `(N, C, H, W)`, the memory layout is:

```
Memory layout of tensor with shape (1, 2, 3, 3):

Channel 0:               Channel 1:
┌───┬───┬───┐            ┌───┬───┬───┐
│ 1 │ 2 │ 3 │            │10 │11 │12 │
├───┼───┼───┤            ├───┼───┼───┤
│ 4 │ 5 │ 6 │            │13 │14 │15 │
├───┼───┼───┤            ├───┼───┼───┤
│ 7 │ 8 │ 9 │            │16 │17 │18 │
└───┴───┴───┘            └───┴───┴───┘

In memory: [1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, 16, 17, 18]
            ─────── Channel 0 ──────── ──────── Channel 1 ────────────────

Flatten just reinterprets this same memory as a 1D vector.
No data is copied — it's a view (reshape) operation!
```

### Verify in PyTorch

```python
import torch

x = torch.tensor([[[[1, 2, 3],
                     [4, 5, 6],
                     [7, 8, 9]],
                    [[10, 11, 12],
                     [13, 14, 15],
                     [16, 17, 18]]]])

print(f"Original shape: {x.shape}")  # [1, 2, 3, 3]

flat = torch.flatten(x, start_dim=1)
print(f"Flattened shape: {flat.shape}")  # [1, 18]
print(f"Flattened values: {flat}")
# tensor([[ 1,  2,  3,  4,  5,  6,  7,  8,  9, 10, 11, 12, 13, 14, 15, 16, 17, 18]])

# Channel 0 values first, then Channel 1 (row-major within each channel)
```

---

## 3️⃣ Methods to Flatten in PyTorch

### Method 1: `torch.flatten()` (Recommended)

```python
import torch

x = torch.randn(8, 256, 4, 4)  # batch=8, 256 channels, 4×4

# Flatten all dims except batch
flat = torch.flatten(x, start_dim=1)
print(flat.shape)  # [8, 4096]  (256 * 4 * 4 = 4096)

# Parameters:
# start_dim: first dim to flatten (1 = skip batch dim)
# end_dim:   last dim to flatten (-1 = last dim, default)

# Partial flatten examples:
flat_spatial = torch.flatten(x, start_dim=2)  # flatten only spatial dims
print(flat_spatial.shape)  # [8, 256, 16]  (4*4=16)

flat_all = torch.flatten(x, start_dim=0)  # flatten everything including batch
print(flat_all.shape)  # [32768]  (8*256*4*4)
```

### Method 2: `nn.Flatten()` (Module — Use in Sequential)

```python
import torch.nn as nn

# nn.Flatten is a module that can be used in nn.Sequential
model = nn.Sequential(
    nn.Conv2d(3, 64, 3, padding=1),
    nn.ReLU(inplace=True),
    nn.AdaptiveAvgPool2d(4),   # (N, 64, 4, 4)
    nn.Flatten(),               # (N, 1024)  ← default: start_dim=1
    nn.Linear(1024, 10),
)

x = torch.randn(2, 3, 32, 32)
out = model(x)
print(out.shape)  # [2, 10]

# nn.Flatten(start_dim, end_dim)
flat_module = nn.Flatten(start_dim=1, end_dim=-1)  # default behavior
```

### Method 3: `.view()` (Manual Reshape)

```python
x = torch.randn(8, 256, 4, 4)

# Using view — must specify exact new shape
flat = x.view(x.size(0), -1)  # -1 auto-computes: 256*4*4 = 4096
print(flat.shape)  # [8, 4096]

# ⚠️ view requires contiguous memory!
# If tensor is non-contiguous, view will fail
x_transposed = x.permute(0, 2, 3, 1)  # non-contiguous!
try:
    flat = x_transposed.view(x_transposed.size(0), -1)
except RuntimeError:
    print("view failed — tensor is not contiguous!")
    flat = x_transposed.contiguous().view(x_transposed.size(0), -1)  # fix
```

### Method 4: `.reshape()` (Safe View)

```python
x = torch.randn(8, 256, 4, 4)

# reshape works even for non-contiguous tensors (may copy data)
flat = x.reshape(x.size(0), -1)
print(flat.shape)  # [8, 4096]

# reshape = view when possible, copies when necessary
# Slightly less efficient than view but always works
```

### Comparison Table

| Method | Type | In Sequential? | Contiguous Required? | Typical Use |
|--------|------|----------------|---------------------|-------------|
| `torch.flatten(x, 1)` | Function | No | No | In `forward()` method |
| `nn.Flatten()` | Module | **Yes** | No | In `nn.Sequential` |
| `x.view(N, -1)` | Method | No | **Yes** | Legacy code |
| `x.reshape(N, -1)` | Method | No | No | When view might fail |

### Best Practice

```python
# In nn.Sequential — use nn.Flatten()
model = nn.Sequential(
    # ... conv layers ...
    nn.Flatten(),           # ← clean, module-based
    nn.Linear(2048, 10),
)

# In custom forward() — use torch.flatten()
class MyModel(nn.Module):
    def forward(self, x):
        x = self.features(x)
        x = torch.flatten(x, 1)  # ← explicit, readable
        x = self.fc(x)
        return x
```

---

## 4️⃣ Dimension Calculation — Critical Skill

### The Key Formula

```
After conv layers, feature map shape is: (N, C, H, W)

Flattened size = C × H × W

This MUST match the in_features of the following nn.Linear layer!
```

### Manual Calculation Example

```
Input image: (N, 3, 32, 32)        → 3 channels, 32×32 pixels

Conv2d(3, 32, 3, padding=1):       → (N, 32, 32, 32)    [same padding]
MaxPool2d(2, 2):                    → (N, 32, 16, 16)    [halved]
Conv2d(32, 64, 3, padding=1):      → (N, 64, 16, 16)    [same padding]
MaxPool2d(2, 2):                    → (N, 64, 8, 8)      [halved]
Conv2d(64, 128, 3, padding=1):     → (N, 128, 8, 8)     [same padding]
MaxPool2d(2, 2):                    → (N, 128, 4, 4)     [halved]

Flatten:                            → (N, 128 × 4 × 4) = (N, 2048)

Therefore: nn.Linear(2048, num_classes)
                     ^^^^
                     Must be 2048!
```

### Common Mistake and How to Debug

```python
# ❌ Wrong: guessing the flatten size
model = nn.Sequential(
    nn.Conv2d(3, 64, 3, padding=1),
    nn.ReLU(),
    nn.MaxPool2d(2, 2),
    nn.Conv2d(64, 128, 3, padding=1),
    nn.ReLU(),
    nn.MaxPool2d(2, 2),
    nn.Flatten(),
    nn.Linear(8192, 10),  # ← Is 8192 correct? Hard to verify mentally!
)

# ✅ Method 1: Use a dummy forward pass to determine the right size
def get_flatten_size(conv_model, input_shape=(1, 3, 32, 32)):
    """Run a dummy input through conv layers to find flatten size."""
    with torch.no_grad():
        dummy = torch.zeros(*input_shape)
        out = conv_model(dummy)
        return out.numel() // out.shape[0]  # total elements / batch size

# Build conv layers separately
conv_layers = nn.Sequential(
    nn.Conv2d(3, 64, 3, padding=1),
    nn.ReLU(),
    nn.MaxPool2d(2, 2),
    nn.Conv2d(64, 128, 3, padding=1),
    nn.ReLU(),
    nn.MaxPool2d(2, 2),
)

flatten_size = get_flatten_size(conv_layers, input_shape=(1, 3, 32, 32))
print(f"Flatten size: {flatten_size}")  # 128 * 8 * 8 = 8192

# ✅ Method 2: Use AdaptiveAvgPool to guarantee size
# Then you always know the flatten size = num_channels × output_h × output_w
```

### Automatic Flatten Size Detection in a Model

```python
import torch
import torch.nn as nn

class AutoFlattenCNN(nn.Module):
    """CNN that automatically computes the flatten size."""
    
    def __init__(self, input_shape=(3, 32, 32), num_classes=10):
        super().__init__()
        
        self.features = nn.Sequential(
            nn.Conv2d(3, 32, 3, padding=1),
            nn.BatchNorm2d(32),
            nn.ReLU(inplace=True),
            nn.MaxPool2d(2, 2),
            nn.Conv2d(32, 64, 3, padding=1),
            nn.BatchNorm2d(64),
            nn.ReLU(inplace=True),
            nn.MaxPool2d(2, 2),
            nn.Conv2d(64, 128, 3, padding=1),
            nn.BatchNorm2d(128),
            nn.ReLU(inplace=True),
            nn.MaxPool2d(2, 2),
        )
        
        # Automatically compute flatten size
        with torch.no_grad():
            dummy = torch.zeros(1, *input_shape)
            feat_out = self.features(dummy)
            self._flatten_size = feat_out.numel()  # for batch=1
        
        self.classifier = nn.Sequential(
            nn.Flatten(),
            nn.Linear(self._flatten_size, 256),
            nn.ReLU(inplace=True),
            nn.Linear(256, num_classes),
        )
    
    def forward(self, x):
        x = self.features(x)
        x = self.classifier(x)
        return x

# Works for any input shape!
model_32 = AutoFlattenCNN(input_shape=(3, 32, 32))
model_64 = AutoFlattenCNN(input_shape=(3, 64, 64))
model_rgb = AutoFlattenCNN(input_shape=(1, 28, 28))  # grayscale

print(f"32×32: flatten = {model_32._flatten_size}")   # 2048
print(f"64×64: flatten = {model_64._flatten_size}")   # 8192
print(f"28×28: flatten = {model_rgb._flatten_size}")  # 1152
```

---

## 5️⃣ Flatten Dimensions Explained

### What `start_dim` and `end_dim` Mean

```python
x = torch.randn(2, 3, 4, 5)   # shape: (2, 3, 4, 5)

# Flatten everything except batch
torch.flatten(x, start_dim=1)          # (2, 60)   3*4*5=60

# Flatten only spatial dims (keep channels)
torch.flatten(x, start_dim=2)          # (2, 3, 20)  4*5=20

# Flatten channels + height (keep width)
torch.flatten(x, start_dim=1, end_dim=2)  # (2, 12, 5)  3*4=12

# Flatten everything (lose batch dim — rarely wanted)
torch.flatten(x, start_dim=0)          # (120,)   2*3*4*5=120
```

### Visual Breakdown

```
Input shape: (N=2, C=3, H=4, W=5)

Operation                     Result Shape    What happened
─────────────────────────────────────────────────────────────
flatten(x, 0)                 (120,)          All dims flattened
flatten(x, 1)                 (2, 60)         C,H,W → single dim
flatten(x, 2)                 (2, 3, 20)      H,W → single dim
flatten(x, 3)                 (2, 3, 4, 5)    W only → no change
flatten(x, 1, 2)              (2, 12, 5)      C,H → single dim
flatten(x, 0, 1)              (6, 4, 5)       N,C → single dim
flatten(x, 0, 2)              (24, 5)         N,C,H → single dim
```

---

## 6️⃣ Flatten vs Global Average Pooling

### Side-by-Side Comparison

```
Feature maps: (N, 512, 7, 7)

                Flatten                          GAP
                ───────                          ───
Operation:     Reshape to 1D vector            Average each channel spatially
Output shape:  (N, 512 × 7 × 7)               (N, 512)
              = (N, 25088)                      = (N, 512)

FC input:      25,088 features                 512 features
FC params:     25,088 × 1,000 = 25M            512 × 1,000 = 0.5M

Information:   Preserves ALL spatial info       Loses spatial info (averaged)
Flexibility:   Fixed input size required        Any input size works
Parameters:    Massive FC layer needed          Tiny FC layer
Overfitting:   High risk                        Much lower risk
```

```
Flatten approach:                    GAP approach:
┌───────────────────┐               ┌───────────────────┐
│  512 × 7 × 7     │               │  512 × 7 × 7     │
│  feature maps     │               │  feature maps     │
└───────┬───────────┘               └───────┬───────────┘
        │ flatten                           │ adaptive_avg_pool2d(1)
        ▼                                   ▼
┌───────────────────┐               ┌───────────────────┐
│    25,088         │               │      512          │
│    values         │               │      values       │
└───────┬───────────┘               └───────┬───────────┘
        │ Linear(25088, 1000)               │ Linear(512, 1000)
        ▼                                   ▼
┌───────────────────┐               ┌───────────────────┐
│    1,000          │               │      1,000        │
│   class scores    │               │    class scores   │
└───────────────────┘               └───────────────────┘
   ~25M FC params                     ~0.5M FC params
```

### When to Use Each

```
Use Flatten when:
  • Working with small spatial dims (e.g., 1×1 after GAP — then flatten = squeeze)
  • Reproducing classic architectures (VGG, AlexNet)
  • Spatial position matters for the task (rare)

Use GAP when:
  • Building modern architectures (always preferred)
  • Want input size flexibility
  • Need to reduce parameter count
  • Want implicit regularization
```

---

## 7️⃣ Common Errors and Debugging

### Error 1: Shape Mismatch

```python
# ❌ Wrong flatten size for Linear layer
model = nn.Sequential(
    nn.Conv2d(3, 64, 3, padding=1),  # (N, 64, 32, 32)
    nn.MaxPool2d(2, 2),               # (N, 64, 16, 16)
    nn.Flatten(),                      # (N, 16384)
    nn.Linear(8192, 10),               # ← WRONG! expects 8192, got 16384
)

x = torch.randn(1, 3, 32, 32)
try:
    model(x)
except RuntimeError as e:
    print(e)
    # mat1 and mat2 shapes cannot be multiplied (1x16384 and 8192x10)
```

**Fix:** Always verify with a dummy forward pass or calculate manually.

### Error 2: Forgetting to Flatten

```python
# ❌ Feeding 4D tensor to Linear
conv_out = torch.randn(4, 128, 4, 4)
fc = nn.Linear(128, 10)  # even the shape concept is wrong
try:
    fc(conv_out)  # This actually works but does the wrong thing!
except:
    pass
# nn.Linear applies to the LAST dimension, so this treats each spatial
# position independently — probably not what you want!
# It would output shape (4, 128, 4, 10) — applying fc to dim=-1 (W dimension)

# ✅ Correct:
conv_out_flat = torch.flatten(conv_out, 1)  # (4, 2048)
fc_correct = nn.Linear(2048, 10)
output = fc_correct(conv_out_flat)  # (4, 10) ✓
```

### Error 3: Wrong start_dim

```python
x = torch.randn(8, 64, 4, 4)

# ❌ Flattening including batch dimension
wrong = torch.flatten(x, start_dim=0)
print(wrong.shape)  # [8192] — lost batch dimension!

# ✅ Preserve batch dimension
correct = torch.flatten(x, start_dim=1)
print(correct.shape)  # [8, 1024] — batch preserved ✓
```

---

## 8️⃣ Complete Worked Example

```python
import torch
import torch.nn as nn

class FlattenDemoCNN(nn.Module):
    """Demonstrates flatten at different points in the network."""
    
    def __init__(self, input_channels=3, input_size=32, num_classes=10):
        super().__init__()
        
        # Conv backbone
        self.conv1 = nn.Conv2d(input_channels, 32, 3, padding=1)
        self.conv2 = nn.Conv2d(32, 64, 3, padding=1)
        self.conv3 = nn.Conv2d(64, 128, 3, padding=1)
        self.pool = nn.MaxPool2d(2, 2)
        self.relu = nn.ReLU(inplace=True)
        
        # Compute flatten size dynamically
        self._flatten_size = self._get_flatten_size(input_channels, input_size)
        
        # FC layers
        self.fc1 = nn.Linear(self._flatten_size, 256)
        self.fc2 = nn.Linear(256, num_classes)
    
    def _get_flatten_size(self, channels, size):
        """Calculate output size of conv layers using a dummy tensor."""
        x = torch.zeros(1, channels, size, size)
        x = self.pool(self.relu(self.conv1(x)))
        x = self.pool(self.relu(self.conv2(x)))
        x = self.pool(self.relu(self.conv3(x)))
        return x.numel()
    
    def forward(self, x):
        # Conv layers
        x = self.pool(self.relu(self.conv1(x)))
        print(f"  After conv1+pool: {x.shape}") if self.training else None
        
        x = self.pool(self.relu(self.conv2(x)))
        print(f"  After conv2+pool: {x.shape}") if self.training else None
        
        x = self.pool(self.relu(self.conv3(x)))
        print(f"  After conv3+pool: {x.shape}") if self.training else None
        
        # THE FLATTEN STEP
        x = torch.flatten(x, 1)
        print(f"  After flatten:    {x.shape}") if self.training else None
        
        # FC layers
        x = self.relu(self.fc1(x))
        x = self.fc2(x)
        return x

# Test with different input sizes
for size in [32, 28]:
    print(f"\n{'='*50}")
    print(f"Input size: {size}×{size}")
    print(f"{'='*50}")
    model = FlattenDemoCNN(input_channels=3, input_size=size)
    x = torch.randn(2, 3, size, size)
    out = model(x)
    print(f"  Output: {out.shape}")
    print(f"  Flatten size: {model._flatten_size}")

# Output for 32×32:
#   After conv1+pool: torch.Size([2, 32, 16, 16])
#   After conv2+pool: torch.Size([2, 64, 8, 8])
#   After conv3+pool: torch.Size([2, 128, 4, 4])
#   After flatten:    torch.Size([2, 2048])          ← 128×4×4 = 2048
#   Output: torch.Size([2, 10])
#   Flatten size: 2048

# Output for 28×28:
#   After conv1+pool: torch.Size([2, 32, 14, 14])
#   After conv2+pool: torch.Size([2, 64, 7, 7])
#   After conv3+pool: torch.Size([2, 128, 3, 3])
#   After flatten:    torch.Size([2, 1152])          ← 128×3×3 = 1152
#   Output: torch.Size([2, 10])
#   Flatten size: 1152
```

---

## 9️⃣ Flatten in Real Architectures

### LeNet-5 (1998)

```python
class LeNet5(nn.Module):
    def __init__(self):
        super().__init__()
        self.features = nn.Sequential(
            nn.Conv2d(1, 6, 5),        # (1, 28, 28) → (6, 24, 24)
            nn.Tanh(),
            nn.AvgPool2d(2, 2),         # → (6, 12, 12)
            nn.Conv2d(6, 16, 5),        # → (16, 8, 8)
            nn.Tanh(),
            nn.AvgPool2d(2, 2),         # → (16, 4, 4)
        )
        # Flatten: 16 × 4 × 4 = 256
        self.classifier = nn.Sequential(
            nn.Flatten(),               # ← Flatten here
            nn.Linear(16 * 4 * 4, 120),
            nn.Tanh(),
            nn.Linear(120, 84),
            nn.Tanh(),
            nn.Linear(84, 10),
        )
    
    def forward(self, x):
        return self.classifier(self.features(x))
```

### AlexNet (2012)

```python
# AlexNet's transition from conv to FC:
# Last conv output: (N, 256, 6, 6)
# Flatten: 256 × 6 × 6 = 9,216
# FC1: Linear(9216, 4096)  ← ~37.7M params just for this layer!
```

### ResNet (2015) — No Traditional Flatten

```python
# ResNet uses GAP, so "flatten" is just squeezing (1,1) spatial dims
# Last conv output: (N, 512, 7, 7)
# AdaptiveAvgPool2d(1): (N, 512, 1, 1)
# Flatten: (N, 512)  ← much more manageable!
# FC: Linear(512, 1000)
```

---

## 📊 Summary Table

| Property | Details |
|----------|---------|
| **Purpose** | Convert 3D feature maps `(C, H, W)` to 1D vector `(C×H×W)` |
| **Preferred: Module** | `nn.Flatten(start_dim=1)` — for `nn.Sequential` |
| **Preferred: Function** | `torch.flatten(x, start_dim=1)` — for `forward()` |
| **Alternative** | `x.view(x.size(0), -1)` — legacy, requires contiguous |
| **Safe alternative** | `x.reshape(x.size(0), -1)` — works even if non-contiguous |
| **Parameters** | 0 (flatten has no learnable parameters) |
| **Key calculation** | `flatten_size = C × H × W` must match `nn.Linear(flatten_size, ...)` |
| **Batch dim** | Always use `start_dim=1` to preserve the batch dimension |
| **Modern replacement** | `nn.AdaptiveAvgPool2d(1)` + `nn.Flatten()` (GAP) |
| **Debugging** | Use dummy forward pass to verify flatten size |

---

## ❓ Revision Questions

### Q1: Flatten Size Calculation
**A CNN processes `(N, 3, 64, 64)` input through: `Conv2d(3,32,3,p=1)` → `MaxPool(2,2)` → `Conv2d(32,64,3,p=1)` → `MaxPool(2,2)` → `Conv2d(64,128,3,p=1)` → `MaxPool(2,2)` → `Flatten`. What is the flatten output size per sample?**

<details>
<summary>Answer</summary>

Step by step:
- Input: (N, 3, 64, 64)
- Conv(3→32, 3, p=1): (N, 32, 64, 64) — same padding
- MaxPool(2,2): (N, 32, 32, 32) — halved
- Conv(32→64, 3, p=1): (N, 64, 32, 32)
- MaxPool(2,2): (N, 64, 16, 16)
- Conv(64→128, 3, p=1): (N, 128, 16, 16)
- MaxPool(2,2): (N, 128, 8, 8)
- Flatten: (N, 128 × 8 × 8) = (N, **8,192**)

The subsequent `nn.Linear` must have `in_features=8192`.
</details>

### Q2: torch.flatten vs nn.Flatten
**When would you use `nn.Flatten()` instead of `torch.flatten()`? What about `.view()`?**

<details>
<summary>Answer</summary>

- **`nn.Flatten()`**: Use when building models with `nn.Sequential`, since Sequential requires modules (not function calls). It's registered as a submodule and appears in `model.children()`.

- **`torch.flatten()`**: Use inside a custom `forward()` method. It's a function call, cleaner for custom logic, and explicitly shows the start_dim parameter.

- **`.view(x.size(0), -1)`**: Legacy approach. Works identically to flatten but requires the tensor to be **contiguous** in memory. If the tensor has been permuted or transposed, `.view()` will raise an error. Use `.reshape()` instead if contiguity isn't guaranteed.

```python
# nn.Flatten for Sequential
model = nn.Sequential(nn.Conv2d(3, 64, 3), nn.Flatten(), nn.Linear(64, 10))

# torch.flatten for forward()
def forward(self, x):
    x = torch.flatten(self.features(x), 1)
    return self.fc(x)
```
</details>

### Q3: Start Dimension Error
**What happens if you accidentally use `torch.flatten(x, start_dim=0)` instead of `start_dim=1` on a batch of images?**

<details>
<summary>Answer</summary>

With `start_dim=0`, the **batch dimension is also flattened**, collapsing all samples into a single 1D vector:

```python
x = torch.randn(4, 64, 8, 8)  # batch of 4

# Wrong:
wrong = torch.flatten(x, 0)
print(wrong.shape)  # torch.Size([16384])  — 4×64×8×8, lost batch!

# Correct:
correct = torch.flatten(x, 1)
print(correct.shape)  # torch.Size([4, 4096])  — batch preserved
```

This would cause a shape mismatch in the FC layer or, worse, silently produce incorrect results if the FC layer's input size happens to match the total flattened size.
</details>

### Q4: Why Flatten Is a View
**Why is flatten typically a "zero-cost" operation (no data copying)?**

<details>
<summary>Answer</summary>

Flatten is a **view** operation — it reinterprets the same underlying memory with a different shape. Since PyTorch tensors are stored in contiguous row-major order, the elements are already laid out sequentially in memory:

```
Memory:  [a₁, a₂, a₃, ..., aₙ]  ← same for both shapes

Shape 1: (C, H, W)  — 3D interpretation of the same data
Shape 2: (C*H*W,)   — 1D interpretation of the same data
```

No data is moved or copied — only the **metadata** (shape, strides) changes. This makes flatten essentially free in terms of computation and memory. The only exception is when the tensor is non-contiguous (e.g., after transpose), in which case `.reshape()` may need to copy data to make it contiguous.
</details>

### Q5: Flatten Size with Stride
**Input: `(N, 3, 28, 28)`. Layer: `Conv2d(3, 16, 5, stride=2, padding=0)` → `Flatten`. What is the flattened size?**

<details>
<summary>Answer</summary>

Output formula: H_out = ⌊(H_in + 2p − k) / s + 1⌋

H_out = ⌊(28 + 0 − 5) / 2 + 1⌋ = ⌊23/2 + 1⌋ = ⌊11.5 + 1⌋ = 12
W_out = 12 (same)

Conv output: (N, 16, 12, 12)
Flatten: (N, 16 × 12 × 12) = (N, **2,304**)
</details>

---

[← Previous: 04 — Fully Connected Layer](04-fully-connected-layer.md) | [📑 Unit Overview](../README.md) | [Next → 06 — Batch Normalization in CNNs](06-batch-normalization-cnns.md)
