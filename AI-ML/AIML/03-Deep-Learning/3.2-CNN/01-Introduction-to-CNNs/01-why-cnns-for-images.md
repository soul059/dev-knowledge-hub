# 📸 1.1 — Why CNNs for Images?

> **Unit:** 01 – Introduction to CNNs
> **Chapter:** 1 of 5 — Motivation & Visual Data

---

[🏠 Unit Overview](./README.md) · **Next →** [Limitations of FC Networks](./02-limitations-fc-networks.md)

---

## 📋 Chapter Overview

This chapter answers the foundational question: *why do we need a specialised architecture for image data?* We explore how digital images are structured as multi-dimensional tensors, why their spatial layout encodes meaning, how the visual cortex inspired CNNs, and what made them the dominant model after the 2012 ImageNet breakthrough.

**After this chapter you will be able to:**

1. Represent an image as a 3-D tensor (H × W × C).
2. Explain the hierarchical nature of visual features (edges → textures → shapes → objects).
3. Articulate why spatial locality and weight sharing are the two key CNN design principles.
4. Describe the significance of the AlexNet / ImageNet 2012 result.

---

## 1 — Images as 3-D Tensors

A digital image is stored as a grid of **pixels**. Each pixel holds one or more intensity values depending on the number of **channels** (C).

| Image Type | Channels (C) | Tensor Shape |
|---|---|---|
| Grayscale | 1 | H × W × 1 |
| RGB Colour | 3 | H × W × 3 |
| RGBA (with alpha) | 4 | H × W × 4 |
| Hyperspectral | 100+ | H × W × C |

### 1.1 — Tensor Notation

An image **X** is a rank-3 tensor:

```
X ∈ ℝ^(H × W × C)
```

- **H** = height (number of rows of pixels)
- **W** = width (number of columns of pixels)
- **C** = channels (colour depth)

A single pixel at row `i`, column `j` is a vector of length C:

```
X[i, j, :] = [r, g, b]    (for an RGB image)
```

### 1.2 — Concrete Example

A standard ImageNet image is **224 × 224 × 3**:

```
Total scalar values = 224 × 224 × 3 = 150,528
```

### 1.3 — ASCII Diagram: RGB Image Tensor

```
          Width (W = 6)
       ┌──────────────────┐
       │ R R R R R R      │  ← Channel 0 (Red)
       │ R R R R R R      │
Height │ R R R R R R      │
(H=4)  │ R R R R R R      │
       └──────────────────┘
       ┌──────────────────┐
       │ G G G G G G      │  ← Channel 1 (Green)
       │ G G G G G G      │
       │ G G G G G G      │
       │ G G G G G G      │
       └──────────────────┘
       ┌──────────────────┐
       │ B B B B B B      │  ← Channel 2 (Blue)
       │ B B B B B B      │
       │ B B B B B B      │
       │ B B B B B B      │
       └──────────────────┘

   In PyTorch convention the shape is (C, H, W)
   i.e. (3, 4, 6) — channels-first.
```

> **PyTorch vs NumPy convention:**
> - NumPy / PIL / OpenCV: **(H, W, C)** — channels last
> - PyTorch / NCHW: **(C, H, W)** — channels first
> Always be aware of which convention your framework uses.

### 1.4 — PyTorch Code: Loading & Inspecting an Image Tensor

```python
import torch
from torchvision import transforms
from PIL import Image

# Load a sample image
img = Image.open("cat.jpg")  # shape in PIL: (W, H) with C channels

# Convert to PyTorch tensor (channels-first, float [0, 1])
transform = transforms.Compose([
    transforms.Resize((224, 224)),
    transforms.ToTensor(),  # Converts (H,W,C) uint8 → (C,H,W) float32
])

tensor = transform(img)

print(f"Tensor shape : {tensor.shape}")     # torch.Size([3, 224, 224])
print(f"Data type    : {tensor.dtype}")      # torch.float32
print(f"Value range  : [{tensor.min():.2f}, {tensor.max():.2f}]")  # [0.00, 1.00]
print(f"Total values : {tensor.numel()}")    # 150528

# Access a single pixel (row 100, col 150)
pixel = tensor[:, 100, 150]
print(f"Pixel [100,150] RGB: {pixel}")       # tensor([0.87, 0.45, 0.22])
```

---

## 2 — Spatial Structure in Images

Unlike tabular data, the **position** of a value in an image carries meaning. Neighbouring pixels are statistically correlated — they tend to share colour, brightness, and belong to the same object.

### 2.1 — Key Properties of Spatial Structure

| Property | Meaning |
|---|---|
| **Locality** | Nearby pixels are highly correlated. An edge at (50, 50) depends on its neighbours, not on pixel (200, 200). |
| **Stationarity** | The same kind of patterns (edges, corners) can appear anywhere in the image. A vertical edge looks the same whether it is at the top-left or bottom-right. |
| **Compositionality** | Simple features combine hierarchically: edges → textures → parts → objects. |

### 2.2 — ASCII Diagram: Spatial Correlation

```
  Raw image patch (5×5 grayscale values, 0–255):

        120  122  125  200  210
        118  121  123  198  208
        119  120  124  202  212
        117  119  122  199  207
        120  121  125  201  209

  Observation:
  ─────────────────────────────────────
  Left 3 columns ≈ 120 (similar region)
  Right 2 columns ≈ 205 (similar region)
  → There is a VERTICAL EDGE between col 3 and col 4.

  Nearby pixels are correlated; far-apart pixels are less so.
```

---

## 3 — The Feature Hierarchy: Edges → Textures → Shapes → Objects

The human visual cortex processes images **hierarchically**. Early layers detect simple patterns; deeper layers combine them into complex concepts. CNNs learn the same hierarchy.

### 3.1 — Visual Feature Hierarchy

```
Layer Depth     What Is Detected              Example
────────────────────────────────────────────────────────────
Layer 1         Edges & Gradients              ─  │  /  \
Layer 2         Corners & Simple Textures      ┘  ┐  ≡  :::
Layer 3         Texture Patterns & Parts       fur, stripes, wheels
Layer 4         Object Parts                   eyes, ears, tires
Layer 5         Whole Objects                  cat, car, face
────────────────────────────────────────────────────────────
```

### 3.2 — ASCII Diagram: Hierarchical Feature Composition

```
   Pixels → Edges → Textures → Parts → Object

   ░░██░░     ││      ┌──┐     ◉  ◉     🐱
   ░░██░░  →  ││  →   │▓▓│  →  ──┬──  →  CAT
   ░░░░░░     ││      └──┘     /    \
```

### 3.3 — Gabor Filters: The Edge Detectors of Vision

The earliest layers of a CNN learn filters remarkably similar to **Gabor filters** — oriented edge detectors found in the primary visual cortex (V1).

```
Vertical edge filter (3×3):       Horizontal edge filter (3×3):

    -1   0   1                        -1  -1  -1
    -1   0   1                         0   0   0
    -1   0   1                         1   1   1
```

```python
import torch
import torch.nn.functional as F

# A 5×5 input (single channel, batch=1)
image = torch.tensor([
    [120, 122, 125, 200, 210],
    [118, 121, 123, 198, 208],
    [119, 120, 124, 202, 212],
    [117, 119, 122, 199, 207],
    [120, 121, 125, 201, 209],
], dtype=torch.float32).unsqueeze(0).unsqueeze(0)  # (1,1,5,5)

# Vertical edge detection kernel
kernel = torch.tensor([
    [-1, 0, 1],
    [-1, 0, 1],
    [-1, 0, 1],
], dtype=torch.float32).unsqueeze(0).unsqueeze(0)  # (1,1,3,3)

# Apply convolution
edges = F.conv2d(image, kernel, padding=0)
print("Edge map shape:", edges.shape)  # (1, 1, 3, 3)
print("Edge map:\n", edges.squeeze())

# Output:
# tensor([[  6.,  155.,  170.],
#         [  6.,  157.,  172.],
#         [  6.,  157.,  170.]])
# High values on right = strong vertical edge detected there.
```

---

## 4 — Two Key Principles: Spatial Locality & Weight Sharing

CNNs exploit two fundamental properties of visual data:

### 4.1 — Spatial Locality (Local Connectivity)

Each neuron connects only to a **small local region** of the input, not the entire image. This mirrors how our visual neurons have small receptive fields.

```
Fully Connected:                   Locally Connected (Conv):

Input:  ● ● ● ● ● ● ● ● ●       Input:  ● ● ● ● ● ● ● ● ●
         \|/|\|/|\|/|\|/                      ╲│╱   ╲│╱
Neuron:     ●   ●   ●             Neuron:      ●     ●
                                          (each sees only
  (each sees ALL inputs)                   3 nearby inputs)
```

### 4.2 — Weight Sharing (Parameter Sharing)

The same filter (set of weights) slides across the **entire** image. This means:

- A feature detector learned in one part of the image works everywhere.
- Parameter count is drastically reduced.

```
Filter W (3×3) shared everywhere:

  Input Image               Filter W          Output (Feature Map)
  ┌───────────┐             ┌─────┐
  │ x x x x x │             │ w w w│          ┌─────────┐
  │ x x x x x │      *      │ w w w│    →     │ y y y   │
  │ x x x x x │             │ w w w│          │ y y y   │
  │ x x x x x │             └─────┘           │ y y y   │
  │ x x x x x │        Same W used for        └─────────┘
  └───────────┘         every position
```

### 4.3 — Parameter Savings: A Concrete Comparison

For a 224 × 224 × 3 input producing 64 output features:

| Approach | Parameters |
|---|---|
| Fully Connected | 224 × 224 × 3 × 64 = **9,633,792** |
| Conv (3×3 filter) | 3 × 3 × 3 × 64 + 64 = **1,792** |
| **Reduction Factor** | **~5,375×** fewer parameters |

```
FC Parameters:   input_size × output_size
               = (224 × 224 × 3) × 64
               = 150,528 × 64
               = 9,633,792

Conv Parameters: kernel_h × kernel_w × C_in × C_out + bias
               = 3 × 3 × 3 × 64 + 64
               = 1,728 + 64
               = 1,792
```

---

## 5 — Biological Inspiration: The Visual Cortex

### 5.1 — Hubel & Wiesel (1959–1962)

Nobel Prize-winning experiments on cat and monkey visual cortex revealed:

1. **Simple cells** respond to oriented edges in specific locations (≈ conv filters).
2. **Complex cells** respond to edges regardless of exact position (≈ pooling).
3. Cells are organised hierarchically — early cells detect simple features; later cells respond to complex stimuli.

### 5.2 — From Biology to Architecture

```
Visual Cortex                     CNN Architecture
──────────────────────────────────────────────────────
Retina (raw signal)       →       Input image
V1 Simple cells (edges)   →       Conv Layer 1
V1 Complex cells (pooled) →       Pooling Layer 1
V2 (textures)             →       Conv Layer 2
V4 (shapes, parts)        →       Conv Layer 3–4
IT (object recognition)   →       Fully Connected + Softmax
```

### 5.3 — Neocognitron (Fukushima, 1980)

The **Neocognitron** was the first neural network architecture to implement:
- Local receptive fields (S-cells ≈ convolutional neurons)
- Spatial pooling (C-cells ≈ pooling neurons)
- Hierarchical feature extraction

This was the direct precursor to modern CNNs.

---

## 6 — The ImageNet Breakthrough (2012)

### 6.1 — The ImageNet Challenge (ILSVRC)

- **Dataset:** ~1.2 million training images, 1,000 classes
- **Metric:** Top-5 error rate (% of images where correct class not in top 5 predictions)

### 6.2 — AlexNet: The Turning Point

In 2012, **Alex Krizhevsky, Ilya Sutskever, and Geoffrey Hinton** submitted **AlexNet** — a deep CNN that shattered the competition.

| Year | Winner | Top-5 Error | Architecture |
|---|---|---|---|
| 2010 | NEC-UIUC | 28.2% | Hand-crafted features + SVM |
| 2011 | XRCE | 25.8% | Fisher Vectors + SVM |
| **2012** | **AlexNet** | **16.4%** | **Deep CNN (8 layers)** |
| 2013 | ZFNet | 14.8% | Refined CNN |
| 2014 | GoogLeNet | 6.7% | Inception modules |
| 2015 | ResNet | 3.6% | 152-layer residual network |
| Human | — | ~5.1% | — |

> **The 2012 gap** (25.8% → 16.4%) was the largest single-year improvement in ILSVRC history. It proved that deep learning with CNNs was superior to decades of hand-crafted feature engineering.

### 6.3 — AlexNet Architecture Overview

```
Input: 227 × 227 × 3
  │
  ├─ Conv1: 96 filters, 11×11, stride 4  → 55×55×96
  ├─ MaxPool + ReLU
  ├─ Conv2: 256 filters, 5×5, pad 2      → 27×27×256
  ├─ MaxPool + ReLU
  ├─ Conv3: 384 filters, 3×3, pad 1      → 13×13×384
  ├─ Conv4: 384 filters, 3×3, pad 1      → 13×13×384
  ├─ Conv5: 256 filters, 3×3, pad 1      → 13×13×256
  ├─ MaxPool
  ├─ FC6: 4096 neurons + ReLU + Dropout
  ├─ FC7: 4096 neurons + ReLU + Dropout
  └─ FC8: 1000 neurons + Softmax → class probabilities
```

### 6.4 — Key Innovations in AlexNet

| Innovation | Significance |
|---|---|
| **ReLU activation** | Faster training than sigmoid/tanh (no vanishing gradient) |
| **Dropout (0.5)** | Regularisation — prevents overfitting |
| **GPU training** | Trained on 2 × GTX 580 GPUs — made deep nets practical |
| **Data augmentation** | Random crops, flips, colour jittering |
| **Local Response Norm** | Lateral inhibition (later replaced by Batch Norm) |

### 6.5 — PyTorch: Simplified AlexNet-Style Network

```python
import torch
import torch.nn as nn

class SimpleAlexNet(nn.Module):
    """
    Simplified AlexNet for educational purposes.
    Input: (batch, 3, 227, 227)
    Output: (batch, 1000)
    """
    def __init__(self, num_classes=1000):
        super().__init__()
        self.features = nn.Sequential(
            nn.Conv2d(3, 96, kernel_size=11, stride=4),     # → (96, 55, 55)
            nn.ReLU(inplace=True),
            nn.MaxPool2d(kernel_size=3, stride=2),           # → (96, 27, 27)

            nn.Conv2d(96, 256, kernel_size=5, padding=2),    # → (256, 27, 27)
            nn.ReLU(inplace=True),
            nn.MaxPool2d(kernel_size=3, stride=2),           # → (256, 13, 13)

            nn.Conv2d(256, 384, kernel_size=3, padding=1),   # → (384, 13, 13)
            nn.ReLU(inplace=True),

            nn.Conv2d(384, 384, kernel_size=3, padding=1),   # → (384, 13, 13)
            nn.ReLU(inplace=True),

            nn.Conv2d(384, 256, kernel_size=3, padding=1),   # → (256, 13, 13)
            nn.ReLU(inplace=True),
            nn.MaxPool2d(kernel_size=3, stride=2),           # → (256, 6, 6)
        )
        self.classifier = nn.Sequential(
            nn.Dropout(0.5),
            nn.Linear(256 * 6 * 6, 4096),
            nn.ReLU(inplace=True),
            nn.Dropout(0.5),
            nn.Linear(4096, 4096),
            nn.ReLU(inplace=True),
            nn.Linear(4096, num_classes),
        )

    def forward(self, x):
        x = self.features(x)
        x = x.view(x.size(0), -1)  # flatten
        x = self.classifier(x)
        return x

# Verify shapes
model = SimpleAlexNet()
dummy = torch.randn(1, 3, 227, 227)
output = model(dummy)
print(f"Output shape: {output.shape}")  # torch.Size([1, 1000])

# Count parameters
total = sum(p.numel() for p in model.parameters())
print(f"Total parameters: {total:,}")   # ~62 million
```

---

## 7 — Why Not Just Use Fully Connected Networks?

A quick preview of the next chapter — the core limitations:

```
                    FC Network               CNN
                    ──────────               ───
Parameters          Massive (millions)       Compact (shared filters)
Spatial awareness   None (flattened input)   Yes (2D filters)
Translation inv.    No                       Yes (weight sharing)
Overfitting risk    Very high                Lower (fewer params)
Scalability         Poor (> 256×256 fails)   Excellent (any size)
```

---

## 8 — Applications of CNNs

CNNs have become the backbone of nearly all computer vision tasks:

| Application | Description | Example Models |
|---|---|---|
| **Image Classification** | Assign a label to an image | ResNet, EfficientNet |
| **Object Detection** | Locate and classify objects with bounding boxes | YOLO, Faster R-CNN |
| **Semantic Segmentation** | Classify every pixel | U-Net, DeepLab |
| **Face Recognition** | Identify or verify faces | FaceNet, ArcFace |
| **Medical Imaging** | Detect tumours, diseases from scans | Specialised U-Nets |
| **Self-Driving Cars** | Perceive roads, pedestrians, signs | Multi-task CNNs |
| **Style Transfer** | Apply artistic styles to photos | Neural Style Transfer |
| **Super Resolution** | Upscale low-res images | SRCNN, ESRGAN |

---

## 📊 Summary Table

| Concept | Key Idea |
|---|---|
| Image Tensor | Images are 3-D tensors: H × W × C |
| Spatial Structure | Neighbouring pixels are correlated; position matters |
| Feature Hierarchy | Edges → Textures → Parts → Objects (learned automatically) |
| Spatial Locality | Each neuron sees only a small region (receptive field) |
| Weight Sharing | Same filter slides across entire image; reduces parameters massively |
| Biological Basis | Inspired by simple/complex cells in visual cortex (Hubel & Wiesel) |
| ImageNet 2012 | AlexNet cut error from 25.8% to 16.4%; launched the deep learning era |
| CNN Advantage | ~5,000× fewer parameters than FC for same task |

---

## ❓ Revision Questions

### Q1: Tensor Dimensions
An RGB image of size 640 × 480 is loaded into PyTorch. What is the tensor shape in PyTorch's convention?

<details>
<summary>Answer</summary>

PyTorch uses channels-first (NCHW) format.
- Single image: **(3, 480, 640)** — (C, H, W)
- With batch dimension: **(1, 3, 480, 640)** — (N, C, H, W)

Note: H = 480 (height/rows), W = 640 (width/columns), C = 3 (RGB).
</details>

### Q2: Parameter Comparison
A grayscale image of size 28 × 28 is fed into (a) a fully connected layer with 128 outputs, and (b) a convolutional layer with 128 filters of size 5 × 5. Calculate the number of parameters for each (including biases).

<details>
<summary>Answer</summary>

**(a) Fully Connected:**
```
Parameters = input_size × output_size + bias
           = (28 × 28) × 128 + 128
           = 784 × 128 + 128
           = 100,352 + 128
           = 100,480
```

**(b) Convolutional:**
```
Parameters = kernel_h × kernel_w × C_in × C_out + bias
           = 5 × 5 × 1 × 128 + 128
           = 3,200 + 128
           = 3,328
```

**Reduction: ~30× fewer parameters** with convolution.
</details>

### Q3: Feature Hierarchy
In a trained CNN like VGG-16, what kind of features would you expect to see in layers 1, 5, and 13?

<details>
<summary>Answer</summary>

- **Layer 1:** Low-level features — edges, gradients, colour blobs.
- **Layer 5:** Mid-level features — textures, corners, simple shapes.
- **Layer 13:** High-level features — object parts (eyes, wheels, windows), semantic patterns.

Each deeper layer composes features from the layer below it.
</details>

### Q4: AlexNet Impact
Why was the 2012 ImageNet result considered a breakthrough? What was the error rate gap?

<details>
<summary>Answer</summary>

AlexNet achieved **16.4% top-5 error** vs the previous best of **25.8%** — a **9.4 percentage point improvement** (36% relative reduction). This was the largest single-year improvement in ILSVRC history and demonstrated that deep CNNs trained on GPUs could dramatically outperform hand-crafted feature engineering approaches that had dominated computer vision for decades.
</details>

### Q5: Biological Connection
How do the "simple cells" and "complex cells" discovered by Hubel & Wiesel map to CNN components?

<details>
<summary>Answer</summary>

- **Simple cells** → **Convolutional layers**: They detect oriented edges/features at specific locations, like a learned conv filter applied at a specific spatial position.
- **Complex cells** → **Pooling layers**: They respond to features regardless of small shifts in position, providing spatial invariance — exactly what max/average pooling achieves.
</details>

### Q6: Weight Sharing Intuition
A CNN uses a single 3 × 3 filter trained to detect vertical edges. Explain why this filter works for vertical edges at *any* location in the image.

<details>
<summary>Answer</summary>

Because of **weight sharing**, the same 3 × 3 filter (same 9 weights) is applied at every spatial position through the sliding-window (convolution) operation. A vertical edge produces a high-contrast left-to-right intensity change regardless of where it appears. Since the filter weights are identical at every position, a vertical edge at position (10, 10) produces the same activation as one at (200, 200). The filter is learning a **pattern**, not a **location**.
</details>

---

## 🔗 Navigation

| | |
|---|---|
| ← Previous | *This is the first chapter* |
| → Next | [Limitations of FC Networks](./02-limitations-fc-networks.md) |
| 🏠 Unit | [01 – Introduction to CNNs](./README.md) |

---

*© 2025 AIML Study Notes. For educational use.*
