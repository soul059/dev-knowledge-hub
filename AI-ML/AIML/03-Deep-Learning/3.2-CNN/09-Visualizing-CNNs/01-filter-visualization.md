# Filter Visualization — What Conv1 Filters Learn

> **Unit 9 · Visualizing CNNs · Topic 1/6**

---

## Table of Contents

1. [Overview](#1-overview)
2. [Visualizing Conv1 Filters](#2-visualizing-conv1-filters)
3. [Filter Patterns Across Layers](#3-filter-patterns-across-layers)
4. [Higher Layer Filters](#4-higher-layer-filters)
5. [PyTorch Code to Extract & Display Filters](#5-pytorch-code-to-extract--display-filters)
6. [Worked Example](#6-worked-example)
7. [Summary Table](#7-summary-table)
8. [Revision Questions](#8-revision-questions)
9. [Navigation](#9-navigation)

---

## 1. Overview

Visualizing CNN filters helps us understand **what features the network has learned**. The first convolutional layer is directly interpretable since its filters operate on raw pixel values.

```
Why Visualize Filters?

  1. UNDERSTANDING: What features does the network detect?
  2. DEBUGGING: Are filters learning meaningful patterns?
  3. TRUST: Can we explain what the model "sees"?
  4. COMPARISON: How do different architectures differ?

Filter visualization difficulty by layer:
  Conv1:  EASY    — filters are small, operate on pixels
  Conv2:  HARDER  — operate on Conv1 activations
  Conv3+: VERY HARD — operate on abstract feature maps
                      → Need other techniques (Grad-CAM, etc.)
```

---

## 2. Visualizing Conv1 Filters

```
Conv1 filters are directly interpretable as small images:

For a 7×7×3 filter (RGB input):
  Each filter is a 7×7 patch with 3 color channels
  → Can display as a small 7×7 RGB image

AlexNet Conv1 (96 filters, 11×11×3):
┌─────┬─────┬─────┬─────┬─────┬─────┐
│ ╱   │ ─── │ ╲   │ ║   │ ██ │ ░░ │
│╱    │ ─── │  ╲  │ ║   │ ██ │ ░░ │  Edges at
│     │ ─── │   ╲ │ ║   │ ██ │ ░░ │  various angles
├─────┼─────┼─────┼─────┼─────┼─────┤
│░██░ │ █░█ │ ▒▓▒ │ red │green│blue│
│░██░ │ █░█ │ ▒▓▒ │     │     │    │  Color blobs
│░██░ │ █░█ │ ▒▓▒ │     │     │    │  and gradients
├─────┼─────┼─────┼─────┼─────┼─────┤
│◢◣   │ ◤◥  │ ░▒▓ │ ▓▒░ │ +  │ ◇  │
│◢◣   │ ◤◥  │ ░▒▓ │ ▓▒░ │    │    │  Gabor-like
│◢◣   │ ◤◥  │ ░▒▓ │ ▓▒░ │    │    │  patterns
└─────┴─────┴─────┴─────┴─────┴─────┘

Consistent patterns across different CNN architectures:
  • Edge detectors at various orientations (0°, 45°, 90°, 135°)
  • Color-sensitive filters (red-green, blue-yellow)
  • Gabor-like filters (oriented, frequency-tuned)
  • These resemble V1 neurons in the visual cortex!
```

### Gabor Filter Similarity

```
Gabor filter formula:
  g(x,y) = exp(-(x'² + γ²y'²)/(2σ²)) × cos(2πx'/λ + ψ)
  
  Where:
    x' = x·cosθ + y·sinθ   (rotated coordinates)
    y' = -x·sinθ + y·cosθ
    θ  = orientation
    λ  = wavelength
    σ  = bandwidth
    γ  = aspect ratio
    ψ  = phase offset

CNN Conv1 filters naturally learn Gabor-like features!
This is a strong validation that CNNs discover biologically
plausible visual features without being explicitly programmed.
```

---

## 3. Filter Patterns Across Layers

```
Layer-by-Layer Feature Hierarchy:

Layer 1 (Conv1): Low-level features
┌─────────────────────────────────────────┐
│ Edges: ╱ ╲ ─ │                         │
│ Colors: red, green, blue blobs          │
│ Gabor-like: oriented frequency detectors│
│ Size: 3×3 to 11×11 pixels              │
└─────────────────────────────────────────┘
         │
         ▼
Layer 2-3 (Conv2-3): Mid-level features
┌─────────────────────────────────────────┐
│ Corners: ◣ ◢ ◤ ◥                       │
│ Textures: repeating patterns, grids     │
│ Color combinations: specific hues       │
│ Simple shapes: circles, crosses         │
└─────────────────────────────────────────┘
         │
         ▼
Layer 4-5: High-level features
┌─────────────────────────────────────────┐
│ Object parts: eyes, wheels, faces       │
│ Complex textures: fur, feathers, scales │
│ Semantic patterns: text-like, face-like │
│ These are NOT directly visualizable!    │
└─────────────────────────────────────────┘

This hierarchy is UNIVERSAL across architectures:
  AlexNet, VGG, ResNet, EfficientNet all show similar Conv1 filters
```

---

## 4. Higher Layer Filters

```
Problem: Conv2+ filters operate on feature maps, not pixels

Conv2 filter: 3×3×64 (if Conv1 has 64 output channels)
  → A 3×3 spatial pattern across 64 abstract dimensions
  → Cannot be displayed as a simple image!

Methods to understand higher layers:
  1. Feature map visualization (topic 2) — what activates for a given input
  2. Activation maximization (topic 5) — synthesize input that maximizes filter
  3. Grad-CAM (topic 3) — what regions matter for a class
  4. Deconvolution — project filter back to pixel space (Zeiler & Fergus)

Zeiler & Fergus (2014) — Deconvolution Visualization:
  For each layer, find image patches that most strongly activate each filter
  
  Layer 1: Edges and colors (as expected)
  Layer 2: Corners, contours, and color combinations
  Layer 3: Texture patterns (mesh, stripes, dots)
  Layer 4: Object parts (dog faces, bird legs, wheels)
  Layer 5: Entire objects (dogs, faces, keyboards)
```

---

## 5. PyTorch Code to Extract & Display Filters

```python
import torch
import torch.nn as nn
import numpy as np
import matplotlib.pyplot as plt
from torchvision.models import resnet50, ResNet50_Weights

# Load pretrained model
model = resnet50(weights=ResNet50_Weights.IMAGENET1K_V2)

# ============================================================
# Extract Conv1 filters
# ============================================================
conv1_weights = model.conv1.weight.data.clone()
print(f"Conv1 shape: {conv1_weights.shape}")
# ResNet-50: (64, 3, 7, 7) — 64 filters, each 7×7×3 (RGB)

# ============================================================
# Visualize all 64 Conv1 filters
# ============================================================
def visualize_filters(weights, title="Conv1 Filters", ncols=8):
    """Visualize convolutional filters as images.
    
    Args:
        weights: (num_filters, channels, H, W)
    """
    num_filters = weights.shape[0]
    nrows = (num_filters + ncols - 1) // ncols
    
    fig, axes = plt.subplots(nrows, ncols, figsize=(ncols * 1.5, nrows * 1.5))
    
    for i in range(num_filters):
        ax = axes[i // ncols, i % ncols] if nrows > 1 else axes[i % ncols]
        
        # Get filter and normalize to [0, 1] for display
        filt = weights[i].cpu().numpy()
        
        if filt.shape[0] == 3:  # RGB
            filt = np.transpose(filt, (1, 2, 0))  # (H, W, 3)
            # Normalize per-filter to [0, 1]
            filt = (filt - filt.min()) / (filt.max() - filt.min() + 1e-8)
            ax.imshow(filt)
        else:  # Grayscale or multi-channel → show first channel
            ax.imshow(filt[0], cmap='gray')
        
        ax.set_title(f"F{i}", fontsize=6)
        ax.axis('off')
    
    # Hide empty subplots
    for i in range(num_filters, nrows * ncols):
        axes[i // ncols, i % ncols].axis('off')
    
    plt.suptitle(title, fontsize=14)
    plt.tight_layout()
    plt.savefig('conv1_filters.png', dpi=150, bbox_inches='tight')
    plt.close()

visualize_filters(conv1_weights, "ResNet-50 Conv1 Filters (7×7×3)")


# ============================================================
# Compare filters across different architectures
# ============================================================
from torchvision.models import vgg16, VGG16_Weights, alexnet, AlexNet_Weights

models_to_compare = {
    'AlexNet': alexnet(weights=AlexNet_Weights.IMAGENET1K_V1).features[0],
    'VGG-16': vgg16(weights=VGG16_Weights.IMAGENET1K_V1).features[0],
    'ResNet-50': model.conv1,
}

for name, conv_layer in models_to_compare.items():
    weights = conv_layer.weight.data
    print(f"{name} Conv1: {weights.shape}")
    visualize_filters(weights, f"{name} Conv1 Filters")


# ============================================================
# Filter statistics
# ============================================================
def filter_statistics(weights, name=""):
    """Print statistics about filter weights."""
    print(f"\n{name} Filter Statistics:")
    print(f"  Shape: {weights.shape}")
    print(f"  Mean:  {weights.mean():.4f}")
    print(f"  Std:   {weights.std():.4f}")
    print(f"  Min:   {weights.min():.4f}")
    print(f"  Max:   {weights.max():.4f}")
    
    # Check for dead filters (all near zero)
    norms = weights.flatten(1).norm(dim=1)
    print(f"  Filter L2 norms: min={norms.min():.4f}, "
          f"max={norms.max():.4f}, mean={norms.mean():.4f}")

filter_statistics(conv1_weights, "ResNet-50")


# ============================================================
# Visualize filter frequency content (FFT)
# ============================================================
def visualize_filter_fft(weights, filter_idx=0):
    """Show frequency content of a filter via FFT."""
    filt = weights[filter_idx].mean(0).cpu().numpy()  # average over channels
    fft = np.fft.fftshift(np.abs(np.fft.fft2(filt, s=(64, 64))))
    
    fig, (ax1, ax2) = plt.subplots(1, 2, figsize=(8, 4))
    ax1.imshow(filt, cmap='gray')
    ax1.set_title(f"Filter {filter_idx} (spatial)")
    ax2.imshow(np.log1p(fft), cmap='hot')
    ax2.set_title(f"Filter {filter_idx} (frequency)")
    plt.savefig(f'filter_{filter_idx}_fft.png')
    plt.close()
```

---

## 6. Worked Example

```
ResNet-50 Conv1 Analysis:

Shape: (64, 3, 7, 7)
  64 filters, each 7×7 spatial, 3 channels (RGB)
  Total parameters: 64 × 3 × 7 × 7 = 9,408

Typical filter categories found:
  ┌──────────────────────────────────────────────┐
  │ Category        │ ~Count │ Description        │
  ├──────────────────────────────────────────────┤
  │ Horizontal edges│  ~10   │ ─── patterns       │
  │ Vertical edges  │  ~10   │ │││ patterns       │
  │ Diagonal edges  │  ~10   │ ╱╲ patterns        │
  │ Color filters   │  ~15   │ Red/Green/Blue     │
  │ Gabor-like      │  ~10   │ Oriented + freq    │
  │ Blob detectors  │   ~5   │ Center-surround    │
  │ Other           │   ~4   │ Mixed patterns     │
  └──────────────────────────────────────────────┘

Observation: Consistent across training runs and architectures!
  → These are the "natural basis functions" for image processing
```

---

## 7. Summary Table

| Concept | Key Idea |
|---|---|
| **Conv1 filters** | Directly interpretable as small RGB images |
| **Edge detectors** | Horizontal, vertical, diagonal edges at various orientations |
| **Color filters** | Respond to specific color patterns (red-green, blue-yellow) |
| **Gabor-like** | Oriented, frequency-tuned filters (similar to V1 neurons) |
| **Higher layers** | Not directly visualizable; need activation maximization |
| **Universality** | Different architectures learn similar Conv1 filters |
| **Biological similarity** | CNN filters resemble primate visual cortex (V1) |

---

## 8. Revision Questions

1. **Why are Conv1 filters directly interpretable** but deeper layer filters are not? What changes at deeper layers?

2. **List the common categories of patterns** found in Conv1 filters of trained CNNs. Which of these are similar to biological visual processing?

3. **Write PyTorch code** to extract and visualize the first convolutional layer filters from a pretrained VGG-16 model.

4. **Compare Conv1 filters** of AlexNet (11×11), VGG (3×3), and ResNet (7×7). How does filter size affect what patterns are learned?

5. **What would it mean** if a Conv1 filter had all near-zero weights? How would you detect and diagnose this?

---

## 9. Navigation

| Previous | Up | Next |
|---|---|---|
| [← Medical Imaging](../08-CNN-Applications/06-medical-imaging.md) | [Unit 9 Index](../09-Visualizing-CNNs/) | [Feature Map Visualization →](./02-feature-map-visualization.md) |

---

> **References:**  
> - Zeiler, M. & Fergus, R. (2014). *Visualizing and Understanding Convolutional Networks.* ECCV.  
> - Olah, C. et al. (2017). *Feature Visualization.* Distill.
