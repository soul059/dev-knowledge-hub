# Feature Maps — The Output of Convolution

> **Chapter 2.4 · Convolution Operation**
> Understand what feature maps are, how they represent detected features, and how
> they form a hierarchy from simple edges to complex objects.

---

## Table of Contents

1. [What Is a Feature Map?](#1-what-is-a-feature-map)
2. [From Input to Feature Map](#2-from-input-to-feature-map)
3. [Multiple Feature Maps = Layer Output](#3-multiple-feature-maps--layer-output)
4. [Feature Hierarchy Across Layers](#4-feature-hierarchy-across-layers)
5. [Interpreting Feature Maps](#5-interpreting-feature-maps)
6. [Visualizing Feature Maps](#6-visualizing-feature-maps)
7. [Worked Example](#7-worked-example)
8. [Python & PyTorch Implementation](#8-python--pytorch-implementation)
9. [Applications](#9-applications)
10. [Summary Table](#10-summary-table)
11. [Revision Questions](#11-revision-questions)

---

## 1. What Is a Feature Map?

A **feature map** (also called an **activation map**) is the output produced when
a single filter is convolved with the input. It is a 2D spatial map that shows
**where** and **how strongly** a particular feature was detected.

```
    Input Image               Filter              Feature Map
    ┌─────────────┐          ┌───────┐           ┌─────────┐
    │             │          │       │           │ ░░░█░░░ │
    │             │   ⊛      │ Edge  │   ═══►   │ ░░░█░░░ │
    │   Photo     │          │Detect │           │ ░░░█░░░ │
    │             │          │       │           │ ░░░█░░░ │
    │             │          └───────┘           │ ░░░█░░░ │
    └─────────────┘                              └─────────┘
                                                 Shows WHERE
                                                 vertical edges
                                                 were detected
    
    ⊛ = convolution operation
    █ = high activation (feature detected)
    ░ = low activation (feature absent)
```

### Key Properties

| Property | Description |
|----------|-------------|
| **Spatial** | Preserves 2D spatial layout of the input |
| **One per filter** | Each filter produces exactly one feature map |
| **Activation values** | High values = feature detected; low/zero = feature absent |
| **Smaller than input** | Unless "same" padding is used |
| **Input to next layer** | Feature maps from layer N become input to layer N+1 |

---

## 2. From Input to Feature Map

### The Convolution Pipeline

```
    ┌──────────┐     ┌──────────┐     ┌──────────┐     ┌──────────┐
    │  Input   │ ──► │ Convolve │ ──► │ Add Bias │ ──► │ Apply    │
    │  (H×W)  │     │ with K   │     │   + b    │     │ ReLU     │
    └──────────┘     └──────────┘     └──────────┘     └──────────┘
                                                             │
                                                             ▼
                                                      ┌──────────┐
                                                      │ Feature  │
                                                      │   Map    │
                                                      │ (H'×W') │
                                                      └──────────┘
```

### Mathematical Definition

For input `I`, filter `K`, and bias `b`:

```
    Feature Map:  F(i,j) = ReLU( Σ Σ I(i+m, j+n) · K(m,n) + b )
                                 m  n
    
    Where:
    - (i, j) is the position in the output
    - (m, n) iterates over the kernel
    - ReLU(x) = max(0, x)  — sets negative values to zero
```

### Step-by-Step Visualization

```
    Input (5×5):          Filter (3×3):        Raw Conv Output (3×3):
    ┌───┬───┬───┬───┬───┐ ┌────┬────┬────┐    ┌─────┬─────┬─────┐
    │10 │20 │30 │40 │50 │ │  1 │  0 │ -1 │    │ -20 │ -20 │ -20 │
    ├───┼───┼───┼───┼───┤ ├────┼────┼────┤    ├─────┼─────┼─────┤
    │10 │20 │30 │40 │50 │ │  1 │  0 │ -1 │    │ -20 │ -20 │ -20 │
    ├───┼───┼───┼───┼───┤ ├────┼────┼────┤    ├─────┼─────┼─────┤
    │10 │20 │30 │40 │50 │ │  1 │  0 │ -1 │    │ -20 │ -20 │ -20 │
    ├───┼───┼───┼───┼───┤ └────┴────┴────┘    └─────┴─────┴─────┘
    │10 │20 │30 │40 │50 │
    ├───┼───┼───┼───┼───┤ Bias b = 25          After Bias + ReLU:
    │10 │20 │30 │40 │50 │                      ┌─────┬─────┬─────┐
    └───┴───┴───┴───┴───┘                      │   5 │   5 │   5 │
                                                ├─────┼─────┼─────┤
    Uniform gradient →                          │   5 │   5 │   5 │
    constant edge response                      ├─────┼─────┼─────┤
                                                │   5 │   5 │   5 │
                                                └─────┴─────┴─────┘
                                                Feature Map (3×3)
```

---

## 3. Multiple Feature Maps = Layer Output

In practice, each convolution layer uses **multiple filters** to detect multiple
features simultaneously. The collection of all feature maps forms the layer's output.

```
    Input                Filter Bank                Layer Output
    ┌──────────┐        ┌──────────┐               ┌──────────┐
    │          │        │ Filter 1 │──────────────► │ Map 1    │
    │          │        ├──────────┤               ├──────────┤
    │  H × W  │   ⊛    │ Filter 2 │──────────────► │ Map 2    │
    │          │        ├──────────┤               ├──────────┤
    │          │        │ Filter 3 │──────────────► │ Map 3    │
    │          │        ├──────────┤               ├──────────┤
    └──────────┘        │   ...    │               │   ...    │
                        ├──────────┤               ├──────────┤
                        │ Filter N │──────────────► │ Map N    │
                        └──────────┘               └──────────┘
    
    Input shape:    H × W × C_in
    Filter bank:    N filters, each K × K × C_in
    Output shape:   H' × W' × N         (N = number of filters)
```

### Concrete Example

```
    Conv Layer 1 of a typical CNN:
    
    Input:  224 × 224 × 3   (RGB image)
    Filters: 64 filters of size 3 × 3 × 3
    Padding: 1, Stride: 1
    
    Output: 224 × 224 × 64  (64 feature maps)
    
    ┌─────────────────────────────────────────────────┐
    │                                                 │
    │  Feature Map 1:  Horizontal edges detected      │
    │  Feature Map 2:  Vertical edges detected        │
    │  Feature Map 3:  Diagonal edges detected        │
    │  Feature Map 4:  Red blobs detected             │
    │  Feature Map 5:  Green-blue transitions         │
    │  ...                                            │
    │  Feature Map 64: Some other low-level feature   │
    │                                                 │
    └─────────────────────────────────────────────────┘
    
    Each 224×224 map shows WHERE that feature appears in the image
```

### Stacking Feature Maps

```
    Individual feature maps            Stacked = Volume
    
    Map 1: ┌──────┐
           │      │                    ┌──────┐
    Map 2: ├──────┤                   ╱      ╱│
           │      │         ═══►     ╱──────╱ │   H' × W' × C_out
    Map 3: ├──────┤                 │      │  │
           │      │                 │      │ ╱
     ...   │ ···  │                 │      │╱
    Map N: ├──────┤                 └──────┘
           │      │
           └──────┘                 This volume becomes the INPUT
                                    to the next convolution layer!
```

---

## 4. Feature Hierarchy Across Layers

One of the most profound discoveries in deep learning: CNNs automatically learn a
**hierarchy of features** from simple to complex.

```
    ┌─────────────────────────────────────────────────────────────────┐
    │                    FEATURE HIERARCHY                           │
    │                                                                │
    │  Layer 1         Layer 2         Layer 3         Layer 4-5     │
    │  ───────         ───────         ───────         ─────────     │
    │                                                                │
    │   ── │ ╱ ╲        ┘ └ ┐ ┌       👁 👃 👄        🐕 🚗 🏠     │
    │                                                                │
    │   Edges &         Corners &       Object          Whole        │
    │   Colors          Textures        Parts           Objects      │
    │                                                                │
    │   Simple ──────────────────────────────────────► Complex       │
    │   Local  ──────────────────────────────────────► Global        │
    │   Generic ─────────────────────────────────────► Task-specific │
    │                                                                │
    └─────────────────────────────────────────────────────────────────┘
```

### Detailed Layer-by-Layer Breakdown

```
    LAYER 1: Edges and Colors
    ┌────────────────────────────────────────────────┐
    │ Feature Map 1:  ──── Horizontal edges          │
    │ Feature Map 2:  │    Vertical edges            │
    │ Feature Map 3:  ╱    45° diagonal edges        │
    │ Feature Map 4:  ╲    135° diagonal edges       │
    │ Feature Map 5:  🔴   Red color blobs           │
    │ Feature Map 6:  🔵   Blue color blobs          │
    │ Feature Map 7:  🟢   Green color blobs         │
    │ Feature Map 8:  ●    Dark spots / high contrast│
    │ ...                                            │
    └────────────────────────────────────────────────┘
    Receptive field: 3×3 to 7×7 pixels
    
    LAYER 2-3: Textures and Patterns
    ┌────────────────────────────────────────────────┐
    │ Feature Map 1:  ┘ └  Corners (combinations     │
    │                      of Layer 1 edges)         │
    │ Feature Map 2:  ≋    Gratings / stripes        │
    │ Feature Map 3:  :::  Dots / stippling          │
    │ Feature Map 4:  ╳    Cross patterns            │
    │ Feature Map 5:  ⌒    Curves / arcs             │
    │ Feature Map 6:  ▓    Textured regions          │
    │ ...                                            │
    └────────────────────────────────────────────────┘
    Receptive field: 10×10 to 40×40 pixels
    
    LAYER 4-5: Object Parts
    ┌────────────────────────────────────────────────┐
    │ Feature Map 1:  Eye shapes                     │
    │ Feature Map 2:  Wheel shapes                   │
    │ Feature Map 3:  Window patterns                │
    │ Feature Map 4:  Nose shapes                    │
    │ Feature Map 5:  Fur textures                   │
    │ Feature Map 6:  Text regions                   │
    │ ...                                            │
    └────────────────────────────────────────────────┘
    Receptive field: 40×40 to 100×100 pixels
    
    LAYER 6+: Complete Objects
    ┌────────────────────────────────────────────────┐
    │ Feature Map 1:  Dog faces                      │
    │ Feature Map 2:  Car fronts                     │
    │ Feature Map 3:  Building facades               │
    │ Feature Map 4:  Flower heads                   │
    │ Feature Map 5:  Human faces                    │
    │ ...                                            │
    └────────────────────────────────────────────────┘
    Receptive field: 100×100+ pixels (nearly whole image)
```

### How the Hierarchy Builds

```
    Layer 1 features:     │  ──  ╱  ╲
                          ↓
    Layer 2 combines:     │ + ── = ┘    (vertical + horizontal = corner)
                          ╱ + ╲ = ╳    (two diagonals = cross)
                          ↓
    Layer 3 combines:     ┘ + └ + ── = ⊏   (corners + edge = shape)
                          ⌒ + ⌒ = ○        (arcs = circle)
                          ↓
    Layer 4 combines:     ○ + colors = 👁   (circle + colors = eye)
                          ○ + texture = ⚙   (circle + spokes = wheel)
                          ↓
    Layer 5 combines:     👁 + 👁 + 👃 + 👄 = 😊  (parts = face)
```

---

## 5. Interpreting Feature Maps

### Activation Values

```
    Feature Map Values:
    
    ┌──────┬──────┬──────┬──────┐
    │  0.0 │  0.0 │  0.9 │  0.0 │    0.0 = Feature NOT detected here
    ├──────┼──────┼──────┼──────┤    0.9 = Feature STRONGLY detected
    │  0.0 │  0.0 │  0.8 │  0.0 │
    ├──────┼──────┼──────┼──────┤    After ReLU, all values ≥ 0
    │  0.1 │  0.0 │  0.7 │  0.1 │
    ├──────┼──────┼──────┼──────┤    High values form a "heat map"
    │  0.0 │  0.0 │  0.0 │  0.0 │    of where the feature exists
    └──────┴──────┴──────┴──────┘
                    ↑
                    Vertical edge detected in column 3
```

### Spatial Correspondence

```
    Input Image:                    Edge Feature Map:
    ┌────────────────────┐         ┌────────────────────┐
    │ ░░░░░░░░██░░░░░░░░ │         │ ░░░░░░░░██░░░░░░░░ │
    │ ░░░░░░░░██░░░░░░░░ │         │ ░░░░░░░░██░░░░░░░░ │
    │ ░░░░░░░░██░░░░░░░░ │    →    │ ░░░░░░░░██░░░░░░░░ │
    │ ░░░░░░░░██░░░░░░░░ │         │ ░░░░░░░░██░░░░░░░░ │
    │ ░░░░░░░░██░░░░░░░░ │         │ ░░░░░░░░██░░░░░░░░ │
    └────────────────────┘         └────────────────────┘
    
    A vertical line in the         The vertical-edge feature map
    input image...                 "lights up" at the same location
```

### Combining Feature Maps for Classification

```
    Image of a cat:
    
    Edge Map:       Texture Map:     Eye Map:        Final Decision:
    ┌──────┐       ┌──────┐        ┌──────┐        ┌──────────────┐
    │╱────╲│       │≋≋≋≋≋≋│        │  ● ● │        │              │
    ││    ││       │≋≋≋≋≋≋│   →    │      │   →    │  CAT: 95%    │
    │╲────╱│       │≋≋≋≋≋≋│        │  ──  │        │  DOG:  3%    │
    └──────┘       └──────┘        └──────┘        │  OTHER: 2%   │
                                                    └──────────────┘
    Outlines        Fur pattern     Cat eyes         Combine all
    detected        detected        detected         evidence
```

---

## 6. Visualizing Feature Maps

### What Visualizations Reveal

```
    Early Layer Feature Maps:           Deep Layer Feature Maps:
    
    ┌──────────────────┐               ┌──────────────────┐
    │ ████████████████ │               │                  │
    │ ████████████████ │               │    ████████      │
    │                  │               │    ████████      │
    │ ████████████████ │               │                  │
    │ ████████████████ │               │         ███████  │
    │                  │               │         ███████  │
    └──────────────────┘               └──────────────────┘
    
    Sharp, edge-like patterns           Blob-like, semantic regions
    Spatially detailed                  Spatially coarse
    Easy to interpret                   Abstract, hard to interpret
```

### Techniques for Feature Map Visualization

| Technique | What It Shows | Complexity |
|-----------|---------------|------------|
| **Direct visualization** | Raw activation values as heatmap | Simple |
| **Gradient-weighted (Grad-CAM)** | Which regions matter for a class | Medium |
| **Deconvolution** | What input pattern activates a neuron | Medium |
| **Feature maximization** | Synthetic input that maximally activates a neuron | Complex |
| **t-SNE of feature maps** | Clustering of feature representations | Complex |

---

## 7. Worked Example

### Complete Feature Map Computation

```
    Input (5×5):                      Three Filters (3×3 each):
    ┌───┬───┬───┬───┬───┐
    │ 0 │ 0 │ 0 │ 0 │ 0 │           F1 (Vertical):   F2 (Horizontal):  F3 (Diagonal):
    ├───┼───┼───┼───┼───┤           ┌────┬────┬────┐ ┌────┬────┬────┐ ┌────┬────┬────┐
    │ 0 │ 0 │ 1 │ 0 │ 0 │           │  1 │  0 │ -1 │ │  1 │  1 │  1 │ │  0 │  0 │  1 │
    ├───┼───┼───┼───┼───┤           ├────┼────┼────┤ ├────┼────┼────┤ ├────┼────┼────┤
    │ 0 │ 0 │ 1 │ 0 │ 0 │           │  1 │  0 │ -1 │ │  0 │  0 │  0 │ │  0 │  1 │  0 │
    ├───┼───┼───┼───┼───┤           ├────┼────┼────┤ ├────┼────┼────┤ ├────┼────┼────┤
    │ 0 │ 0 │ 1 │ 0 │ 0 │           │  1 │  0 │ -1 │ │ -1 │ -1 │ -1 │ │  1 │  0 │  0 │
    ├───┼───┼───┼───┼───┤           └────┴────┴────┘ └────┴────┴────┘ └────┴────┴────┘
    │ 0 │ 0 │ 0 │ 0 │ 0 │
    └───┴───┴───┴───┴───┘           (Input has a vertical line in column 2)
```

**Feature Map 1 (Vertical Edge Detector):**

```
    Position (0,0): 0+0+0 + 0+0-1 + 0+0-1 = -2
    Position (0,1): 0+0+0 + 0+1+0 + 0+1+0 = 2      → Detects LEFT edge of line
    Position (0,2): 0+0+0 + 1+0+0 + 1+0+0 = 2      → Detects ... wait,
                                                        let me recalculate
    
    Position (0,0): patch=[0,0,0; 0,0,1; 0,0,1], filter=[1,0,-1; 1,0,-1; 1,0,-1]
                    = 0+0+0 + 0+0-1 + 0+0-1 = -2
    Position (0,1): patch=[0,0,0; 0,1,0; 0,1,0], filter=[1,0,-1; 1,0,-1; 1,0,-1]
                    = 0+0+0 + 0+0+0 + 0+0+0 = 0    → center of line: no edge
    Position (0,2): patch=[0,0,0; 1,0,0; 1,0,0], filter=[1,0,-1; 1,0,-1; 1,0,-1]
                    = 0+0+0 + 1+0+0 + 1+0+0 = 2    → right edge detected
    
    Feature Map 1:
    ┌─────┬─────┬─────┐
    │  -2 │   0 │   2 │
    ├─────┼─────┼─────┤
    │  -2 │   0 │   2 │
    ├─────┼─────┼─────┤
    │  -2 │   0 │   2 │
    └─────┴─────┴─────┘
    
    After ReLU:
    ┌─────┬─────┬─────┐
    │   0 │   0 │   2 │    ← Detects right side of the vertical line
    ├─────┼─────┼─────┤
    │   0 │   0 │   2 │
    ├─────┼─────┼─────┤
    │   0 │   0 │   2 │
    └─────┴─────┴─────┘
```

**Feature Map 2 (Horizontal Edge Detector):**

```
    Position (0,0): patch=[0,0,0; 0,0,1; 0,0,1], filter=[1,1,1; 0,0,0; -1,-1,-1]
                    = 0+0+0 + 0+0+0 + 0+0-1 = -1
    Position (0,1): patch=[0,0,0; 0,1,0; 0,1,0], filter=[1,1,1; 0,0,0; -1,-1,-1]
                    = 0+0+0 + 0+0+0 + 0-1+0 = -1
    Position (1,1): patch=[0,1,0; 0,1,0; 0,1,0], filter=[1,1,1; 0,0,0; -1,-1,-1]
                    = 0+1+0 + 0+0+0 + 0-1+0 = 0
    
    Feature Map 2 (after ReLU):
    ┌─────┬─────┬─────┐
    │   0 │   0 │   0 │    ← No horizontal edges detected!
    ├─────┼─────┼─────┤       (correct — input has vertical line)
    │   0 │   0 │   0 │
    ├─────┼─────┼─────┤
    │   0 │   0 │   0 │
    └─────┴─────┴─────┘
```

**Result: Three stacked feature maps form the layer output.**

```
    Output Volume:
    
    Map 1 (vert):  Map 2 (horiz):  Map 3 (diag):
    ┌───┬───┬───┐  ┌───┬───┬───┐  ┌───┬───┬───┐
    │ 0 │ 0 │ 2 │  │ 0 │ 0 │ 0 │  │ ? │ ? │ ? │
    ├───┼───┼───┤  ├───┼───┼───┤  ├───┼───┼───┤
    │ 0 │ 0 │ 2 │  │ 0 │ 0 │ 0 │  │ ? │ ? │ ? │
    ├───┼───┼───┤  ├───┼───┼───┤  ├───┼───┼───┤
    │ 0 │ 0 │ 2 │  │ 0 │ 0 │ 0 │  │ ? │ ? │ ? │
    └───┴───┴───┘  └───┴───┴───┘  └───┴───┴───┘
    
    Stacked output shape: 3 × 3 × 3  (H' × W' × C_out)
```

---

## 8. Python & PyTorch Implementation

### Extracting Feature Maps from a CNN

```python
import torch
import torch.nn as nn
import torchvision.models as models
import torchvision.transforms as transforms
from PIL import Image
import matplotlib.pyplot as plt

# Load pretrained VGG16
model = models.vgg16(pretrained=True)
model.eval()

# Hook to capture feature maps
feature_maps = {}

def get_hook(name):
    def hook(module, input, output):
        feature_maps[name] = output.detach()
    return hook

# Register hooks on specific layers
model.features[0].register_forward_hook(get_hook('conv1_1'))   # Layer 1
model.features[5].register_forward_hook(get_hook('conv2_1'))   # Layer 2
model.features[10].register_forward_hook(get_hook('conv3_1'))  # Layer 3
model.features[17].register_forward_hook(get_hook('conv4_1'))  # Layer 4
model.features[24].register_forward_hook(get_hook('conv5_1'))  # Layer 5

# Preprocess image
transform = transforms.Compose([
    transforms.Resize((224, 224)),
    transforms.ToTensor(),
    transforms.Normalize(mean=[0.485, 0.456, 0.406],
                         std=[0.229, 0.224, 0.225]),
])

image = Image.open('sample.jpg')
input_tensor = transform(image).unsqueeze(0)  # (1, 3, 224, 224)

# Forward pass
with torch.no_grad():
    output = model(input_tensor)

# Print feature map shapes
for name, fmap in feature_maps.items():
    print(f"{name}: {fmap.shape}")

# conv1_1: torch.Size([1, 64, 224, 224])   ← 64 maps, full resolution
# conv2_1: torch.Size([1, 128, 112, 112])  ← 128 maps, half resolution
# conv3_1: torch.Size([1, 256, 56, 56])    ← 256 maps, quarter resolution
# conv4_1: torch.Size([1, 512, 28, 28])    ← 512 maps, 1/8 resolution
# conv5_1: torch.Size([1, 512, 14, 14])    ← 512 maps, 1/16 resolution
```

### Visualizing Feature Maps

```python
def visualize_feature_maps(fmaps, layer_name, num_maps=16, cols=4):
    """Visualize the first num_maps feature maps from a layer."""
    fmaps = fmaps.squeeze(0)  # Remove batch dimension
    num_maps = min(num_maps, fmaps.shape[0])
    rows = (num_maps + cols - 1) // cols
    
    fig, axes = plt.subplots(rows, cols, figsize=(cols * 3, rows * 3))
    axes = axes.flatten()
    
    for i in range(num_maps):
        ax = axes[i]
        ax.imshow(fmaps[i].numpy(), cmap='viridis')
        ax.set_title(f'Map {i}')
        ax.axis('off')
    
    # Hide unused subplots
    for i in range(num_maps, len(axes)):
        axes[i].axis('off')
    
    plt.suptitle(f'Feature Maps: {layer_name}', fontsize=16)
    plt.tight_layout()
    plt.savefig(f'feature_maps_{layer_name}.png', dpi=150)
    plt.show()

# Visualize feature maps at each layer
for name, fmap in feature_maps.items():
    visualize_feature_maps(fmap, name)
```

### Building a Simple CNN and Inspecting Feature Maps

```python
import torch
import torch.nn as nn
import torch.nn.functional as F

class SimpleCNN(nn.Module):
    def __init__(self):
        super().__init__()
        self.conv1 = nn.Conv2d(1, 8, kernel_size=3, padding=1)   # 8 feature maps
        self.conv2 = nn.Conv2d(8, 16, kernel_size=3, padding=1)  # 16 feature maps
        self.conv3 = nn.Conv2d(16, 32, kernel_size=3, padding=1) # 32 feature maps
        self.pool = nn.MaxPool2d(2, 2)
        self.fc = nn.Linear(32 * 3 * 3, 10)
    
    def forward(self, x, return_features=False):
        features = {}
        
        x = F.relu(self.conv1(x))       # (B, 8, 28, 28)
        features['conv1'] = x
        x = self.pool(x)                 # (B, 8, 14, 14)
        
        x = F.relu(self.conv2(x))       # (B, 16, 14, 14)
        features['conv2'] = x
        x = self.pool(x)                 # (B, 16, 7, 7)
        
        x = F.relu(self.conv3(x))       # (B, 32, 7, 7)
        features['conv3'] = x
        x = self.pool(x)                 # (B, 32, 3, 3)
        
        x = x.view(x.size(0), -1)       # Flatten
        x = self.fc(x)                   # (B, 10)
        
        if return_features:
            return x, features
        return x

# Create model and run inference
model = SimpleCNN()
dummy_input = torch.randn(1, 1, 28, 28)  # MNIST-like input

output, features = model(dummy_input, return_features=True)

print("Feature map shapes:")
for name, fmap in features.items():
    print(f"  {name}: {fmap.shape}")

# Feature map shapes:
#   conv1: torch.Size([1, 8, 28, 28])
#   conv2: torch.Size([1, 16, 14, 14])
#   conv3: torch.Size([1, 32, 7, 7])
```

### Grad-CAM Visualization

```python
import torch
import torch.nn.functional as F
import numpy as np

def grad_cam(model, input_tensor, target_class, target_layer):
    """
    Generate a Grad-CAM heatmap for a specific class and layer.
    """
    gradients = []
    activations = []
    
    def forward_hook(module, input, output):
        activations.append(output)
    
    def backward_hook(module, grad_in, grad_out):
        gradients.append(grad_out[0])
    
    # Register hooks
    fwd_handle = target_layer.register_forward_hook(forward_hook)
    bwd_handle = target_layer.register_full_backward_hook(backward_hook)
    
    # Forward pass
    model.eval()
    output = model(input_tensor)
    
    # Backward pass for target class
    model.zero_grad()
    one_hot = torch.zeros_like(output)
    one_hot[0, target_class] = 1
    output.backward(gradient=one_hot)
    
    # Compute Grad-CAM
    grad = gradients[0]           # (1, C, H, W)
    act = activations[0]          # (1, C, H, W)
    weights = grad.mean(dim=[2, 3], keepdim=True)  # Global average pooling
    cam = (weights * act).sum(dim=1, keepdim=True)  # Weighted sum
    cam = F.relu(cam)             # Only positive contributions
    cam = cam.squeeze().detach().numpy()
    
    # Normalize to [0, 1]
    cam = (cam - cam.min()) / (cam.max() - cam.min() + 1e-8)
    
    # Clean up hooks
    fwd_handle.remove()
    bwd_handle.remove()
    
    return cam

# Usage:
# heatmap = grad_cam(model, input_tensor, target_class=5, 
#                    target_layer=model.conv3)
```

### Computing Feature Map Statistics

```python
import torch

def feature_map_stats(feature_maps):
    """Compute statistics for each feature map in a batch."""
    # feature_maps shape: (B, C, H, W)
    B, C, H, W = feature_maps.shape
    
    stats = {
        'mean': feature_maps.mean(dim=[2, 3]),    # (B, C)
        'std': feature_maps.std(dim=[2, 3]),       # (B, C)
        'max': feature_maps.amax(dim=[2, 3]),      # (B, C)
        'sparsity': (feature_maps == 0).float().mean(dim=[2, 3]),  # (B, C)
    }
    
    print(f"Feature maps: {C} channels, {H}×{W} spatial")
    print(f"  Mean activation: {stats['mean'].mean():.4f}")
    print(f"  Std activation:  {stats['std'].mean():.4f}")
    print(f"  Max activation:  {stats['max'].max():.4f}")
    print(f"  Avg sparsity:    {stats['sparsity'].mean():.2%}")
    
    return stats

# Usage with the SimpleCNN:
# model = SimpleCNN()
# _, features = model(torch.randn(1, 1, 28, 28), return_features=True)
# for name, fmap in features.items():
#     print(f"\n{name}:")
#     feature_map_stats(fmap)
```

---

## 9. Applications

| Application | Feature Map Usage | Example |
|-------------|-------------------|---------|
| **Object Detection** | Feature maps at multiple scales detect objects of different sizes | YOLO, Faster R-CNN |
| **Semantic Segmentation** | Feature maps are upsampled to produce pixel-level labels | U-Net, DeepLab |
| **Style Transfer** | Feature maps capture content and style separately | Neural style transfer |
| **Feature Pyramid Networks** | Combine feature maps from different layers | FPN for multi-scale detection |
| **Attention Mechanisms** | Feature maps weighted by attention scores | SENet, CBAM |
| **Transfer Learning** | Feature maps from pretrained CNN used as features | Fine-tuning on new tasks |
| **Anomaly Detection** | Unusual feature map activations indicate anomalies | Industrial quality control |

### Feature Maps in Object Detection

```
    Image → CNN Backbone → Feature Maps at Multiple Scales
    
    ┌──────────────────────────┐
    │  Feature Map (56×56)     │  → Detects SMALL objects
    │  High resolution         │     (e.g., distant pedestrians)
    │  Low-level features      │
    └──────────────────────────┘
    
    ┌──────────────┐
    │  Feature Map │             → Detects MEDIUM objects
    │  (28×28)     │                (e.g., cars)
    └──────────────┘
    
    ┌───────┐
    │  Map  │                    → Detects LARGE objects
    │ (14×14)│                      (e.g., buses, buildings)
    └───────┘
```

---

## 10. Summary Table

| Concept | Description |
|---------|-------------|
| **Feature map** | 2D spatial output from one filter's convolution + activation |
| **Activation map** | Synonym for feature map (after ReLU) |
| **Layer output** | Stack of all feature maps = 3D volume (H' × W' × C_out) |
| **Feature hierarchy** | Simple → Complex features across layers |
| **Layer 1** | Edges, colors, simple patterns |
| **Layer 2-3** | Corners, textures, combinations of edges |
| **Layer 4-5** | Object parts (eyes, wheels, windows) |
| **Deep layers** | Whole objects, semantic concepts |
| **Spatial resolution** | Decreases through the network (pooling, stride) |
| **Channel count** | Increases through the network (more features) |
| **Visualization** | Direct display, Grad-CAM, deconvolution, feature maximization |

---

## 11. Revision Questions

**Q1.** A convolution layer has 32 filters applied to an input of size 28×28×1.
What is the shape of the output (assuming stride=1, padding=1, kernel=3×3)?

<details>
<summary>Answer</summary>

```
Spatial: O = (28 - 3 + 2×1)/1 + 1 = 28
Channels: 32 (one per filter)

Output shape: 28 × 28 × 32
```
Each of the 32 filters produces a 28×28 feature map. Stacked together, they form
a 28×28×32 volume.
</details>

**Q2.** Why do feature maps in deeper layers have lower spatial resolution but
more channels? What is the design philosophy?

<details>
<summary>Answer</summary>

Lower spatial resolution (via pooling/stride) progressively discards precise
location information, while increasing channels captures more abstract features.
The philosophy: early layers need spatial precision to detect local features,
while deeper layers need more feature types to represent complex concepts.
This forms an "inverted pyramid" of information representation.
</details>

**Q3.** After ReLU activation, approximately what fraction of a typical feature
map's values are zero? Why?

<details>
<summary>Answer</summary>

Typically 50-80% of values are zero after ReLU. This "sparsity" occurs because
ReLU sets all negative values to zero. Since convolution produces roughly equal
positive and negative outputs (especially with zero-centered kernels), about half
the values become zero. This sparsity is beneficial — it provides efficient
representation and helps with training.
</details>

**Q4.** Explain how feature maps from a pretrained CNN can be used for transfer
learning on a new task.

<details>
<summary>Answer</summary>

Feature maps from early and middle layers capture generic visual features (edges,
textures, shapes) that are useful across tasks. For transfer learning:
1. Use the pretrained CNN as a "feature extractor" — freeze convolutional layers
2. Feed the feature maps into new classification layers
3. Train only the new layers on the new task
4. Optionally fine-tune later conv layers for task-specific features

This works because the feature hierarchy learned on ImageNet transfers well.
</details>

**Q5.** What is the relationship between the number of feature maps and the
number of parameters in a convolution layer?

<details>
<summary>Answer</summary>

```
Parameters = K × K × C_in × C_out + C_out
                                      └── bias
```

More feature maps (C_out) = more parameters. Each additional feature map adds
`K × K × C_in + 1` parameters. For K=3, C_in=64: each new feature map adds
3×3×64 + 1 = 577 parameters.
</details>

**Q6.** Describe the difference between a feature map at Layer 1 and Layer 5 of
VGG16 in terms of: spatial size, number of channels, type of features, and
interpretability.

<details>
<summary>Answer</summary>

| Property | Layer 1 (conv1_1) | Layer 5 (conv5_1) |
|----------|-------------------|-------------------|
| Spatial size | 224×224 | 14×14 |
| Channels | 64 | 512 |
| Features | Edges, colors, simple patterns | Object parts, semantic regions |
| Interpretability | High (visually obvious) | Low (abstract, hard to interpret) |
| Receptive field | 3×3 pixels | ~180×180 pixels |
</details>

---

## Navigation

| | |
|---|---|
| ⬅️ **Previous** | [Stride & Padding](./03-stride-and-padding.md) |
| ➡️ **Next** | [Convolution Arithmetic](./05-convolution-arithmetic.md) |
| 🏠 **Home** | [CNN Overview](../README.md) |

---

> *"Feature maps are the CNN's way of seeing — each map a different lens,
> together forming a complete visual understanding of the world."*
