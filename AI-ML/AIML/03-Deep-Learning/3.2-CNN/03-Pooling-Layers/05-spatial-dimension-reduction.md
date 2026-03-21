# 5 · Spatial Dimension Reduction

> **Unit 3.2 – Convolutional Neural Networks / 03 – Pooling Layers**
>
> [← Previous: Pooling vs Strided Convolution](./04-pooling-vs-strided-conv.md) | Next: *End of Unit*
>
> [↑ Back to Pooling Layers Overview](./README.md)

---

## 5.1 Chapter Overview

Spatial dimension reduction is one of the most fundamental principles in CNN design. As data flows through the network, **spatial dimensions (H × W) shrink while the number of channels (C) grows**. This creates a hierarchical feature representation:

```
  Input Image                            Feature Vector
  ┌──────────────┐                       ┌───┐
  │              │   Spatial dims ↓      │   │
  │   224×224    │   Channels ↑          │   │
  │   3 channels │   ──────────────►     │512│   (after GAP)
  │              │                       │   │
  │              │                       │   │
  └──────────────┘                       └───┘
   150,528 values                        512 values

  Low-level features ────────────► High-level features
  (edges, colors)                  (objects, concepts)
  Spatially precise                Position invariant
```

This chapter covers **why** spatial reduction is necessary, the **canonical pattern** of doubling channels while halving spatial dimensions, and how to **track tensor shapes** through an entire network.

---

## 5.2 Why Reduce Spatial Dimensions?

### 1. Computational Efficiency

Each convolutional layer's computation scales with spatial size:

```
  FLOPs for one Conv2d layer ≈ 2 × k² × C_in × C_out × H_out × W_out

  Example: Conv(64, 64, 3) on different spatial sizes:
  ┌──────────┬──────────────────┬──────────────────────────┐
  │ Spatial  │  Output Elements │  FLOPs                   │
  ├──────────┼──────────────────┼──────────────────────────┤
  │ 224×224  │  50,176          │  2×9×64×64×50176 = 3.7B  │
  │ 112×112  │  12,544          │  2×9×64×64×12544 = 0.9B  │
  │  56×56   │   3,136          │  2×9×64×64×3136  = 0.2B  │
  │  28×28   │     784          │  2×9×64×64×784   = 58M   │
  │  14×14   │     196          │  2×9×64×64×196   = 14M   │
  │   7×7    │      49          │  2×9×64×64×49    = 3.6M  │
  └──────────┴──────────────────┴──────────────────────────┘

  Reducing spatial size by 2× reduces FLOPs by ~4× !
```

### 2. Growing Receptive Field

Smaller spatial maps with the same kernel size cover a larger proportion of the original input:

```
  Layer 1: 3×3 on 224×224 → sees 3/224 = 1.3% of input
  Layer 3: 3×3 on  56×56  → sees 3/56  = 5.4% of input (after 2 pools)
  Layer 5: 3×3 on  14×14  → sees 3/14  = 21% of input (after 4 pools)
  Layer 7: 3×3 on   7×7   → sees 3/7   = 43% of input (after 5 pools)

  The receptive field in the original image grows MUCH faster
  than the kernel size suggests, because each pooling step
  expands what each spatial position "sees."
```

### 3. Hierarchical Abstraction

```
  SPATIAL SIZE       WHAT THE NETWORK LEARNS
  ─────────────────────────────────────────────────
  224×224 (input)    Raw pixels

  112×112            Edges, color gradients
                     ↕ features span ~6px of input

   56×56             Textures, corners, simple shapes
                     ↕ features span ~24px of input

   28×28             Object parts (eyes, wheels, windows)
                     ↕ features span ~48px of input

   14×14             Object-level features (faces, cars)
                     ↕ features span ~100px of input

    7×7              Scene-level features (whole objects)
                     ↕ features span ~200px of input

    1×1 (GAP)        Global class representation
```

### 4. Memory Management

```
  Activation memory per layer ∝ B × C × H × W

  Example: batch_size=32, dtype=float32 (4 bytes)

  Layer with output (32, 64, 224, 224):
    Memory = 32 × 64 × 224 × 224 × 4 bytes = 412 MB

  Layer with output (32, 512, 7, 7):
    Memory = 32 × 512 × 7 × 7 × 4 bytes = 3.2 MB

  Reduction: 412 MB → 3.2 MB = 129× less activation memory!
```

---

## 5.3 The Canonical Pattern: Double Channels, Halve Spatial

### The Rule

The most common CNN design pattern is:

```
  At each downsampling step:
    • Spatial dimensions:  H × W  →  H/2 × W/2
    • Channel dimensions:  C      →  2C

  This keeps the "information capacity" roughly constant:
    C × H × W  →  2C × H/2 × W/2  =  C × H × W / 2

  Actually loses ~50% of values, but the increased channels
  can encode more ABSTRACT features to compensate.
```

### Visual Pattern

```
  ┌────────────────────────────┐
  │  3 × 224 × 224             │   Input: 150,528 values
  └──────────────┬─────────────┘
                 ▼ Conv + Pool
  ┌────────────────────────────┐
  │  64 × 112 × 112            │   802,816 values
  └──────────────┬─────────────┘
                 ▼ Pool (÷2 spatial, ×2 channels)
  ┌────────────────────────────┐
  │  128 × 56 × 56             │   401,408 values
  └──────────────┬─────────────┘
                 ▼ Pool
  ┌────────────────────────────┐
  │  256 × 28 × 28             │   200,704 values
  └──────────────┬─────────────┘
                 ▼ Pool
  ┌────────────────────────────┐
  │  512 × 14 × 14             │   100,352 values
  └──────────────┬─────────────┘
                 ▼ Pool
  ┌────────────────────────────┐
  │  512 × 7 × 7               │   25,088 values
  └──────────────┬─────────────┘
                 ▼ GAP
  ┌────────────────────────────┐
  │  512 × 1 × 1               │   512 values
  └──────────────┬─────────────┘
                 ▼ FC
  ┌────────────────────────────┐
  │  1000                       │   Class logits
  └────────────────────────────┘
```

### Why This Pattern Works

1. **Constant information bandwidth** — Doubling channels compensates for quartering spatial area.
2. **Feature richness at each level** — More channels = more diverse features at each spatial resolution.
3. **Computational balance** — FLOPs per layer remain roughly constant: `k² × C × 2C × (H/2)² ≈ k² × C × C × H²/2`.

---

## 5.4 Dimension Tracking Through Real Architectures

### VGG-16

```
  Layer                        Output Shape         Params        FLOPs
  ─────────────────────────────────────────────────────────────────────
  Input                        3 × 224 × 224
  Conv3-64                     64 × 224 × 224       1,792         89.9M
  Conv3-64                     64 × 224 × 224       36,928        1.85B
  ► MaxPool 2×2                64 × 112 × 112       0             —
  Conv3-128                    128 × 112 × 112      73,856        925.5M
  Conv3-128                    128 × 112 × 112      147,584       1.85B
  ► MaxPool 2×2                128 × 56 × 56        0             —
  Conv3-256                    256 × 56 × 56        295,168       925.5M
  Conv3-256                    256 × 56 × 56        590,080       1.85B
  Conv3-256                    256 × 56 × 56        590,080       1.85B
  ► MaxPool 2×2                256 × 28 × 28        0             —
  Conv3-512                    512 × 28 × 28        1,180,160     925.5M
  Conv3-512                    512 × 28 × 28        2,359,808     1.85B
  Conv3-512                    512 × 28 × 28        2,359,808     1.85B
  ► MaxPool 2×2                512 × 14 × 14        0             —
  Conv3-512                    512 × 14 × 14        2,359,808     462.4M
  Conv3-512                    512 × 14 × 14        2,359,808     462.4M
  Conv3-512                    512 × 14 × 14        2,359,808     462.4M
  ► MaxPool 2×2                512 × 7 × 7          0             —
  ► GAP (if modernized)        512 × 1 × 1          0             —
  FC → 1000                    1000                  513,000       —
  ─────────────────────────────────────────────────────────────────────
  Total (with GAP):            ~15.5M params

  Pattern: 3 → 64 → 128 → 256 → 512 → 512 (capped at 512)
           224  112   56    28    14     7
```

### ResNet-50

```
  Stage       Output Shape        Channels    Spatial    Downsample Method
  ───────────────────────────────────────────────────────────────────────
  Input       3 × 224 × 224       3           224×224
  Conv1       64 × 112 × 112      64          112×112    stride-2 conv (7×7)
  MaxPool     64 × 56 × 56        64          56×56      max pool 3×3, s=2
  Stage 2     256 × 56 × 56       256         56×56      no spatial change
  Stage 3     512 × 28 × 28       512         28×28      stride-2 conv (first block)
  Stage 4     1024 × 14 × 14      1024        14×14      stride-2 conv (first block)
  Stage 5     2048 × 7 × 7        2048        7×7        stride-2 conv (first block)
  GAP         2048 × 1 × 1        2048        1×1        adaptive avg pool
  FC          1000                 —           —          linear layer
  ───────────────────────────────────────────────────────────────────────

  Channel pattern:  3 → 64 → 256 → 512 → 1024 → 2048
  Spatial pattern:  224 → 112 → 56 → 28 → 14 → 7 → 1

  Note: ResNet uses bottleneck blocks (1×1 → 3×3 → 1×1) that expand
  channels by 4× within each block (e.g., 64 → 256 at the output).
```

### EfficientNet-B0

```
  Stage   Operator            Resolution   Channels   Spatial Reduction
  ─────────────────────────────────────────────────────────────────────
  1       Conv 3×3, s=2       112×112      32         224 → 112 (÷2)
  2       MBConv1, k3         112×112      16         —
  3       MBConv6, k3, s=2    56×56        24         112 → 56  (÷2)
  4       MBConv6, k5, s=2    28×28        40         56 → 28   (÷2)
  5       MBConv6, k3, s=2    14×14        80         28 → 14   (÷2)
  6       MBConv6, k5         14×14        112        —
  7       MBConv6, k5, s=2    7×7          192        14 → 7    (÷2)
  8       MBConv6, k3         7×7          320        —
  9       Conv 1×1 + GAP      1×1          1280       7 → 1     (GAP)
  Head    FC                   —           1000       —
  ─────────────────────────────────────────────────────────────────────

  All downsampling via strided depthwise separable convolutions.
  Channel growth is NOT strictly 2× — compound scaling finds optimal ratios.
```

---

## 5.5 Worked Example — Tracking Shapes Through a Custom CNN

### Problem

Design a CNN for CIFAR-10 (32×32 input) and track all dimensions.

### Architecture Design

```
  Design constraints:
  • Input: 3 × 32 × 32
  • Must reach at least 4×4 spatial before GAP (not too aggressive)
  • Follow double-channels/halve-spatial pattern
  • End with GAP + Linear(→10)
```

### Shape Tracking

```
  Layer                           Output Shape      Reduction    Params
  ──────────────────────────────────────────────────────────────────────
  Input                           3 × 32 × 32
  Conv2d(3, 32, 3, pad=1)        32 × 32 × 32      —            896
  BN + ReLU                       32 × 32 × 32      —            64
  Conv2d(32, 32, 3, pad=1)       32 × 32 × 32      —            9,248
  BN + ReLU                       32 × 32 × 32      —            64
  ► MaxPool(2, 2)                 32 × 16 × 16      ÷2 spatial   0

  Conv2d(32, 64, 3, pad=1)       64 × 16 × 16      ×2 channels  18,496
  BN + ReLU                       64 × 16 × 16      —            128
  Conv2d(64, 64, 3, pad=1)       64 × 16 × 16      —            36,928
  BN + ReLU                       64 × 16 × 16      —            128
  ► MaxPool(2, 2)                 64 × 8 × 8        ÷2 spatial   0

  Conv2d(64, 128, 3, pad=1)      128 × 8 × 8       ×2 channels  73,856
  BN + ReLU                       128 × 8 × 8       —            256
  Conv2d(128, 128, 3, pad=1)     128 × 8 × 8       —            147,584
  BN + ReLU                       128 × 8 × 8       —            256
  ► MaxPool(2, 2)                 128 × 4 × 4       ÷2 spatial   0

  ► AdaptiveAvgPool2d(1)          128 × 1 × 1       GAP          0
  ► Flatten                       128               —            —
  ► Linear(128, 10)               10                classifier   1,290
  ──────────────────────────────────────────────────────────────────────
  Total params:                                                  289,194

  Channel progression:  3  → 32 → 64 → 128
  Spatial progression:  32 → 16 → 8  → 4  → 1
  
  Value counts:  3,072 → 8,192 → 4,096 → 2,048 → 128 → 10
```

---

## 5.6 Python / PyTorch — Shape Tracking Tools

### Manual Shape Tracking with Print Hooks

```python
import torch
import torch.nn as nn

class ShapeTracker(nn.Module):
    """Wraps a model and prints shapes at every layer."""
    def __init__(self, model):
        super().__init__()
        self.model = model
        self._hooks = []
        for name, module in self.model.named_modules():
            if len(list(module.children())) == 0:  # leaf modules only
                hook = module.register_forward_hook(
                    lambda mod, inp, out, n=name: 
                        print(f"  {n:40s} → {str(list(out.shape)):>25s}")
                )
                self._hooks.append(hook)

    def forward(self, x):
        print(f"  {'INPUT':40s} → {str(list(x.shape)):>25s}")
        return self.model(x)

    def remove_hooks(self):
        for h in self._hooks:
            h.remove()
```

### Complete CIFAR-10 CNN with Shape Tracking

```python
import torch
import torch.nn as nn

class CIFAR10CNN(nn.Module):
    def __init__(self, num_classes=10):
        super().__init__()
        self.features = nn.Sequential(
            # Block 1: 3 → 32 channels, 32×32 → 16×16
            nn.Conv2d(3, 32, 3, padding=1),
            nn.BatchNorm2d(32),
            nn.ReLU(inplace=True),
            nn.Conv2d(32, 32, 3, padding=1),
            nn.BatchNorm2d(32),
            nn.ReLU(inplace=True),
            nn.MaxPool2d(2, 2),

            # Block 2: 32 → 64 channels, 16×16 → 8×8
            nn.Conv2d(32, 64, 3, padding=1),
            nn.BatchNorm2d(64),
            nn.ReLU(inplace=True),
            nn.Conv2d(64, 64, 3, padding=1),
            nn.BatchNorm2d(64),
            nn.ReLU(inplace=True),
            nn.MaxPool2d(2, 2),

            # Block 3: 64 → 128 channels, 8×8 → 4×4
            nn.Conv2d(64, 128, 3, padding=1),
            nn.BatchNorm2d(128),
            nn.ReLU(inplace=True),
            nn.Conv2d(128, 128, 3, padding=1),
            nn.BatchNorm2d(128),
            nn.ReLU(inplace=True),
            nn.MaxPool2d(2, 2),
        )
        self.classifier = nn.Sequential(
            nn.AdaptiveAvgPool2d(1),
            nn.Flatten(),
            nn.Linear(128, num_classes),
        )

    def forward(self, x):
        x = self.features(x)
        x = self.classifier(x)
        return x

# Build model and trace shapes
model = CIFAR10CNN()
x = torch.randn(1, 3, 32, 32)

# Method 1: Use ShapeTracker
tracker = ShapeTracker(model)
with torch.no_grad():
    y = tracker(x)
tracker.remove_hooks()

print(f"\nFinal output: {y.shape}")
```

### Programmatic Shape Calculator

```python
def trace_shapes(model, input_shape=(1, 3, 224, 224)):
    """Trace tensor shapes through a model without running it fully."""
    x = torch.randn(*input_shape)
    shapes = [("Input", list(x.shape))]

    for name, module in model.named_modules():
        if len(list(module.children())) == 0:
            x = module(x)
            shapes.append((name, list(x.shape)))

    # Pretty print
    print(f"{'Layer':<45} {'Shape':>25} {'Values':>12}")
    print("─" * 85)
    for name, shape in shapes:
        n_values = 1
        for s in shape:
            n_values *= s
        print(f"{name:<45} {str(shape):>25} {n_values:>12,}")

    return shapes

# Example with VGG-style model
model = nn.Sequential(
    nn.Conv2d(3, 64, 3, padding=1), nn.ReLU(),
    nn.Conv2d(64, 64, 3, padding=1), nn.ReLU(),
    nn.MaxPool2d(2, 2),
    nn.Conv2d(64, 128, 3, padding=1), nn.ReLU(),
    nn.Conv2d(128, 128, 3, padding=1), nn.ReLU(),
    nn.MaxPool2d(2, 2),
    nn.Conv2d(128, 256, 3, padding=1), nn.ReLU(),
    nn.Conv2d(256, 256, 3, padding=1), nn.ReLU(),
    nn.MaxPool2d(2, 2),
    nn.AdaptiveAvgPool2d(1),
    nn.Flatten(),
    nn.Linear(256, 10),
)

with torch.no_grad():
    trace_shapes(model, (1, 3, 32, 32))
```

### Using `torchsummary` for Professional Output

```python
# pip install torchsummary
from torchsummary import summary

model = CIFAR10CNN()
summary(model, (3, 32, 32), device='cpu')

# Output:
# ----------------------------------------------------------------
#         Layer (type)               Output Shape         Param #
# ================================================================
#             Conv2d-1           [1, 32, 32, 32]             896
#        BatchNorm2d-2           [1, 32, 32, 32]              64
#               ReLU-3           [1, 32, 32, 32]               0
#             Conv2d-4           [1, 32, 32, 32]           9,248
#        BatchNorm2d-5           [1, 32, 32, 32]              64
#               ReLU-6           [1, 32, 32, 32]               0
#          MaxPool2d-7           [1, 32, 16, 16]               0
#             Conv2d-8           [1, 64, 16, 16]          18,496
#        BatchNorm2d-9           [1, 64, 16, 16]             128
#              ReLU-10           [1, 64, 16, 16]               0
#            Conv2d-11           [1, 64, 16, 16]          36,928
#       BatchNorm2d-12           [1, 64, 16, 16]             128
#              ReLU-13           [1, 64, 16, 16]               0
#         MaxPool2d-14            [1, 64, 8, 8]               0
#            Conv2d-15           [1, 128, 8, 8]           73,856
#       BatchNorm2d-16           [1, 128, 8, 8]              256
#              ReLU-17           [1, 128, 8, 8]               0
#            Conv2d-18           [1, 128, 8, 8]          147,584
#       BatchNorm2d-19           [1, 128, 8, 8]              256
#              ReLU-20           [1, 128, 8, 8]               0
#         MaxPool2d-21            [1, 128, 4, 4]              0
# AdaptiveAvgPool2d-22            [1, 128, 1, 1]              0
#           Flatten-23                  [1, 128]              0
#            Linear-24                   [1, 10]           1,290
# ================================================================
# Total params: 289,194
# Trainable params: 289,194
```

### Dimension Mismatch Debugging

```python
def debug_shapes(model, x):
    """Debug tool: run input through model layer by layer,
    catching dimension mismatches with helpful error messages."""
    print(f"Input: {x.shape}")
    for i, layer in enumerate(model):
        try:
            x = layer(x)
            print(f"  Layer {i} ({layer.__class__.__name__}): {x.shape}")
        except RuntimeError as e:
            print(f"  ❌ Layer {i} ({layer.__class__.__name__}): FAILED!")
            print(f"     Input shape was: {x.shape}")
            print(f"     Error: {e}")
            return None
    return x

# Example: deliberately wrong model
wrong_model = nn.Sequential(
    nn.Conv2d(3, 64, 3, padding=1),
    nn.MaxPool2d(2),
    nn.Conv2d(64, 128, 3, padding=1),
    nn.MaxPool2d(2),
    nn.Flatten(),
    nn.Linear(100, 10),  # Wrong input size!
)

x = torch.randn(1, 3, 32, 32)
with torch.no_grad():
    debug_shapes(wrong_model, x)
# Will show that Flatten produces [1, 128*8*8] = [1, 8192]
# but Linear expects 100 → dimension mismatch caught!
```

---

## 5.7 Common Spatial Reduction Patterns

### Pattern 1: Aggressive Early Reduction (ResNet)

```
  224 ──[stride-2 conv]──► 112 ──[maxpool]──► 56 ──► ... ──► 7

  Reduces spatial size by 4× in the first two operations.
  Saves massive computation in all subsequent layers.
  The large 7×7 conv at stride 2 captures broad low-level features.
```

### Pattern 2: Gradual Reduction (VGG)

```
  224 ──[pool]──► 112 ──[pool]──► 56 ──[pool]──► 28 ──[pool]──► 14 ──[pool]──► 7

  Each pool halves dimensions. Five pool layers total.
  More conv layers at each resolution = more parameters but richer features.
```

### Pattern 3: Patch Embedding (Vision Transformer)

```
  224 ──[stride-16 conv]──► 14   (VERY aggressive: 16× reduction at once!)

  ViT treats each 16×16 patch as a "token."
  Single stride-16 conv converts the image to a sequence of patch embeddings.
  No further spatial reduction — Transformer processes 14×14 = 196 tokens.
```

### Pattern 4: Feature Pyramid (FPN for Detection)

```
  224 → 112 → 56 → 28 → 14 → 7    (standard forward path)
                ↓     ↓     ↓
               P3    P4    P5       (pyramid features at multiple scales)
                ↑     ↑     ↑
  Lateral connections + upsampling   (top-down path)

  KEEPS features at multiple spatial resolutions.
  Small objects detected at P3 (high res), large at P5 (low res).
```

---

## 5.8 Dimension Formula Quick Reference

### Output Size After Convolution or Pooling

```
            ⌊ input_size + 2 × padding - kernel_size ⌋
  output =  ⎢ ──────────────────────────────────────── ⎥ + 1
            ⌊              stride                       ⌋
```

### Common Cases (Cheat Sheet)

```
  ┌──────────────────────────────────────────┬───────────────────┐
  │ Operation                                │ Spatial Effect     │
  ├──────────────────────────────────────────┼───────────────────┤
  │ Conv(k=3, s=1, p=1)                     │ SAME (no change)   │
  │ Conv(k=3, s=2, p=1)                     │ HALVE (÷2)         │
  │ Conv(k=1, s=1, p=0)                     │ SAME               │
  │ Conv(k=7, s=2, p=3)                     │ HALVE              │
  │ MaxPool(k=2, s=2)                       │ HALVE              │
  │ MaxPool(k=3, s=2, p=1)                  │ HALVE              │
  │ AdaptiveAvgPool2d(1)                    │ → 1×1              │
  │ ConvTranspose(k=2, s=2)                 │ DOUBLE (×2)        │
  │ Upsample(scale_factor=2)               │ DOUBLE             │
  ├──────────────────────────────────────────┼───────────────────┤
  │ Conv(k=3, s=1, p=0)                     │ SHRINK by 2        │
  │ Conv(k=5, s=1, p=0)                     │ SHRINK by 4        │
  │ Conv(k=5, s=1, p=2)                     │ SAME               │
  └──────────────────────────────────────────┴───────────────────┘

  Rule of thumb for "same" padding: p = (k-1)/2  (for odd k)
```

### Multi-Layer Tracking Formula

```
  After N pooling layers with stride 2:

  Final spatial size = input_size / 2^N

  Example:
  Input 224 → after 5 pools → 224 / 2^5 = 224 / 32 = 7

  Input 32 → after 3 pools → 32 / 2^3 = 32 / 8 = 4
```

---

## 5.9 Practical Considerations

### What Happens If You Don't Reduce Spatial Dims?

```
  A 10-layer network on 224×224 with NO pooling:

  Conv(3→64)  at 224×224:   FLOPs ≈   89M
  Conv(64→64) at 224×224:   FLOPs ≈ 1.85B
  Conv(64→64) at 224×224:   FLOPs ≈ 1.85B
  ... (8 more layers)
  Total: ~18.5B FLOPs just for 10 layers!

  WITH pooling every 2 layers (standard):
  Conv(3→64)   at 224×224:  FLOPs ≈   89M
  Conv(64→64)  at 224×224:  FLOPs ≈ 1.85B
  Pool → 112×112
  Conv(64→128) at 112×112:  FLOPs ≈  925M
  Conv(128→128)at 112×112:  FLOPs ≈ 1.85B
  Pool → 56×56
  Conv(128→256)at  56×56:   FLOPs ≈  925M
  Conv(256→256)at  56×56:   FLOPs ≈ 1.85B
  Pool → 28×28
  Conv(256→512)at  28×28:   FLOPs ≈  925M
  Conv(512→512)at  28×28:   FLOPs ≈ 1.85B
  Total: ~11.1B FLOPs — and the network is MUCH deeper / more capable!
```

### Avoiding Common Mistakes

```
  ❌ MISTAKE 1: Too many pools for input size
     Input 32×32 with 6 MaxPool layers → 32/64 < 1 → ERROR!
     Rule: max pools = ⌊log₂(input_size)⌋ - 1
     For 32×32: max 4 pools (32→16→8→4→2), then GAP

  ❌ MISTAKE 2: Forgetting to update Linear layer input size
     After changing pooling, the flatten size changes.
     Always compute: flatten_size = final_channels × final_H × final_W

  ❌ MISTAKE 3: Not matching shortcut dimensions in skip connections
     In ResNets, when spatial dims change, the shortcut needs
     a 1×1 stride-2 conv to match:
     x_shortcut = Conv2d(C_in, C_out, 1, stride=2)(x)

  ✓ TIP: Always trace shapes with a dummy tensor before training!
```

---

## 5.10 PyTorch Shape Tracking — Complete Example

```python
import torch
import torch.nn as nn

class ImageNetCNN(nn.Module):
    """Full ImageNet classifier with explicit shape annotations."""
    def __init__(self, num_classes=1000):
        super().__init__()

        # Stage 1: 224→112 (÷2), 3→64 channels
        self.stage1 = nn.Sequential(
            nn.Conv2d(3, 64, 7, stride=2, padding=3, bias=False),  # 224→112
            nn.BatchNorm2d(64),
            nn.ReLU(inplace=True),
            nn.MaxPool2d(3, stride=2, padding=1),                   # 112→56
        )
        # After stage1: [B, 64, 56, 56]

        # Stage 2: 56→56 (same), 64→128 channels
        self.stage2 = nn.Sequential(
            nn.Conv2d(64, 128, 3, padding=1, bias=False),
            nn.BatchNorm2d(128),
            nn.ReLU(inplace=True),
            nn.Conv2d(128, 128, 3, padding=1, bias=False),
            nn.BatchNorm2d(128),
            nn.ReLU(inplace=True),
        )
        # After stage2: [B, 128, 56, 56]

        # Stage 3: 56→28 (÷2), 128→256 channels
        self.stage3 = nn.Sequential(
            nn.Conv2d(128, 256, 3, stride=2, padding=1, bias=False),  # strided conv!
            nn.BatchNorm2d(256),
            nn.ReLU(inplace=True),
            nn.Conv2d(256, 256, 3, padding=1, bias=False),
            nn.BatchNorm2d(256),
            nn.ReLU(inplace=True),
        )
        # After stage3: [B, 256, 28, 28]

        # Stage 4: 28→14 (÷2), 256→512 channels
        self.stage4 = nn.Sequential(
            nn.Conv2d(256, 512, 3, stride=2, padding=1, bias=False),
            nn.BatchNorm2d(512),
            nn.ReLU(inplace=True),
            nn.Conv2d(512, 512, 3, padding=1, bias=False),
            nn.BatchNorm2d(512),
            nn.ReLU(inplace=True),
        )
        # After stage4: [B, 512, 14, 14]

        # Stage 5: 14→7 (÷2), 512→512 channels
        self.stage5 = nn.Sequential(
            nn.Conv2d(512, 512, 3, stride=2, padding=1, bias=False),
            nn.BatchNorm2d(512),
            nn.ReLU(inplace=True),
        )
        # After stage5: [B, 512, 7, 7]

        # Classifier: GAP + FC
        self.classifier = nn.Sequential(
            nn.AdaptiveAvgPool2d(1),       # [B, 512, 1, 1]
            nn.Flatten(),                   # [B, 512]
            nn.Linear(512, num_classes),    # [B, 1000]
        )

    def forward(self, x):
        x = self.stage1(x)
        x = self.stage2(x)
        x = self.stage3(x)
        x = self.stage4(x)
        x = self.stage5(x)
        x = self.classifier(x)
        return x

# Verify all shapes
model = ImageNetCNN(num_classes=1000)
x = torch.randn(2, 3, 224, 224)

with torch.no_grad():
    s1 = model.stage1(x)
    s2 = model.stage2(s1)
    s3 = model.stage3(s2)
    s4 = model.stage4(s3)
    s5 = model.stage5(s4)
    out = model.classifier(s5)

print("Shape progression:")
print(f"  Input:  {list(x.shape)}")       # [2, 3, 224, 224]
print(f"  Stage1: {list(s1.shape)}")      # [2, 64, 56, 56]
print(f"  Stage2: {list(s2.shape)}")      # [2, 128, 56, 56]
print(f"  Stage3: {list(s3.shape)}")      # [2, 256, 28, 28]
print(f"  Stage4: {list(s4.shape)}")      # [2, 512, 14, 14]
print(f"  Stage5: {list(s5.shape)}")      # [2, 512, 7, 7]
print(f"  Output: {list(out.shape)}")     # [2, 1000]

# Parameter count by stage
for name in ['stage1', 'stage2', 'stage3', 'stage4', 'stage5', 'classifier']:
    stage = getattr(model, name)
    params = sum(p.numel() for p in stage.parameters())
    print(f"  {name}: {params:>10,} params")
```

---

## 5.11 Summary Table

| Aspect | Detail |
|--------|--------|
| **Core Principle** | Reduce spatial dims (H, W) while increasing channels (C) |
| **Canonical Pattern** | Halve spatial, double channels at each stage |
| **Purpose** | Computation efficiency, receptive field growth, hierarchical abstraction |
| **Typical Progression** | 224→112→56→28→14→7→1 (spatial), 3→64→128→256→512→512 (channels) |
| **Methods** | MaxPool, AvgPool, Strided Conv, GAP |
| **Output Size Formula** | ⌊(input + 2×pad - kernel) / stride⌋ + 1 |
| **Memory Impact** | 4× memory reduction per ÷2 spatial reduction |
| **FLOPs Impact** | ~4× FLOPs reduction per ÷2 spatial reduction |
| **Final Step** | GAP to (C, 1, 1) before classifier |
| **PyTorch Debug** | Use forward hooks or `torchsummary.summary()` |

---

## 5.12 Revision Questions

**Q1.** An input tensor of shape `(4, 3, 64, 64)` passes through the following layers. Track the shape at each step:
1. `Conv2d(3, 32, 3, padding=1)`
2. `MaxPool2d(2, 2)`
3. `Conv2d(32, 64, 3, padding=1)`
4. `MaxPool2d(2, 2)`
5. `AdaptiveAvgPool2d(1)`
6. `Flatten()`
7. `Linear(64, 10)`

<details><summary>Answer</summary>

```
1. Conv2d(3→32, k=3, p=1):     (4, 32, 64, 64)   ← same spatial
2. MaxPool2d(2, 2):             (4, 32, 32, 32)   ← spatial halved
3. Conv2d(32→64, k=3, p=1):    (4, 64, 32, 32)   ← channels doubled
4. MaxPool2d(2, 2):             (4, 64, 16, 16)   ← spatial halved
5. AdaptiveAvgPool2d(1):        (4, 64, 1, 1)     ← GAP
6. Flatten():                   (4, 64)            ← remove spatial dims
7. Linear(64, 10):              (4, 10)            ← class logits
```

</details>

**Q2.** Why do CNNs double the number of channels when halving spatial dimensions? What would happen if channels stayed constant?

<details><summary>Answer</summary>

Doubling channels compensates for the information capacity lost by quartering the spatial area (H/2 × W/2 = H×W/4). The total number of values goes from C×H×W to 2C×H/4×W/4 = C×H×W/2 — a gradual reduction. If channels stayed constant, each pooling layer would discard 75% of values, leading to severe information loss. The network would struggle to learn complex features because it lacks channel diversity to represent the growing variety of high-level concepts.

</details>

**Q3.** Calculate the total FLOPs saved by adding a single MaxPool2d(2,2) before a Conv2d(128, 128, 3, padding=1) layer processing a 56×56 feature map, compared to applying the conv at 56×56 first.

<details><summary>Answer</summary>

Without pool (conv at 56×56): FLOPs = 2 × 3² × 128 × 128 × 56 × 56 = 2 × 9 × 16,384 × 3,136 ≈ 924.8M

With pool first (conv at 28×28): FLOPs = 2 × 3² × 128 × 128 × 28 × 28 = 2 × 9 × 16,384 × 784 ≈ 231.2M

Savings: 924.8M - 231.2M = 693.6M FLOPs (75% reduction). Plus the pool itself costs negligible FLOPs (~0.8M).

</details>

**Q4.** A Vision Transformer uses a patch embedding of `Conv2d(3, 768, 16, stride=16)` on a 224×224 image. What is the output shape and how does this compare to VGG's approach?

<details><summary>Answer</summary>

Output: (B, 768, 14, 14) — since ⌊(224-16)/16⌋ + 1 = 14. This produces 14×14 = 196 spatial positions (tokens), each with 768 dimensions.

VGG needs 5 pooling layers to get from 224 to 7 (224→112→56→28→14→7). ViT achieves an even more aggressive reduction (224→14) in a single strided convolution. ViT trades the gradual hierarchical feature building of CNNs for processing all patches at the same resolution through self-attention layers.

</details>

**Q5.** You're designing a CNN for 128×128 medical images. What's the maximum number of MaxPool2d(2,2) layers you can use before GAP, and what spatial size should you target before GAP?

<details><summary>Answer</summary>

Maximum pools: log₂(128) = 7 pools would reach 1×1 (128→64→32→16→8→4→2→1), but that's too aggressive. Best practice is to stop at 4×4 or 2×2 before GAP.

Recommended: 5 pool layers → 128→64→32→16→8→4, then GAP to 1×1. Or 6 pools → 128→...→2, then GAP. Going below 4×4 before the last convolution block risks losing too much spatial detail, especially in medical imaging where fine details matter.

</details>

**Q6.** Explain why aggressive early downsampling (like ResNet's stride-2 conv + maxpool) is computationally beneficial.

<details><summary>Answer</summary>

ResNet reduces 224→112 (stride-2 conv) →56 (maxpool) in just two operations — a 4× spatial reduction before most computation happens. Since FLOPs scale with H²×W², going from 224² to 56² reduces computation by 16×. All subsequent convolutions (which make up >95% of the network's depth) operate on the smaller 56×56 maps or below. Without this early reduction, the first few conv layers at 224×224 would dominate the total compute. The 7×7 stride-2 initial conv has a large receptive field to ensure no low-level features are missed despite the aggressive downsampling.

</details>

---

> [← Previous: Pooling vs Strided Convolution](./04-pooling-vs-strided-conv.md) | Next: *End of Unit*
>
> [↑ Back to Pooling Layers Overview](./README.md)
