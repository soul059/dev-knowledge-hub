# Feature Map Visualization — Intermediate Layer Activations

> **Unit 9 · Visualizing CNNs · Topic 2/6**

---

## Table of Contents

1. [Overview](#1-overview)
2. [What Feature Maps Show](#2-what-feature-maps-show)
3. [Hooks in PyTorch](#3-hooks-in-pytorch)
4. [Feature Map Interpretation](#4-feature-map-interpretation)
5. [Spatial Attention from Activations](#5-spatial-attention-from-activations)
6. [Code Example](#6-code-example)
7. [Summary Table](#7-summary-table)
8. [Revision Questions](#8-revision-questions)
9. [Navigation](#9-navigation)

---

## 1. Overview

Feature maps (activations) show **what a specific input image triggers** at each layer of the CNN. Unlike filter visualization (which shows weights), feature map visualization shows the network's **response to a particular image**.

```
Filter Visualization:     Feature Map Visualization:
  "What CAN the filter    "What DOES the filter
   detect?"                detect in THIS image?"
  
  Input-independent        Input-dependent
  Shows learned weights    Shows activations for a specific image
```

---

## 2. What Feature Maps Show

```
For an input image of a cat:

Layer 1 activations (64 channels, 112×112):
  ┌────────┐ ┌────────┐ ┌────────┐ ┌────────┐
  │ Edges  │ │ Edges  │ │ Color  │ │ Texture│
  │  ╱╱╱   │ │  ───   │ │ ████  │ │ ░▒▓█  │
  │ ╱╱╱    │ │  ───   │ │  █    │ │ ░▒▓█  │
  │╱╱╱     │ │  ───   │ │       │ │ ░▒▓█  │
  └────────┘ └────────┘ └────────┘ └────────┘
  Horizontal  Horizontal  Warm      Gradient
  edges       edges       regions   

Layer 3 activations (256 channels, 28×28):
  ┌────────┐ ┌────────┐ ┌────────┐ ┌────────┐
  │   ██   │ │ █    █ │ │        │ │  ████  │
  │  ████  │ │  ████  │ │  ████  │ │ ██  ██ │
  │  ████  │ │        │ │  ████  │ │  ████  │
  │   ██   │ │        │ │        │ │        │
  └────────┘ └────────┘ └────────┘ └────────┘
  Body       Face/head   Fur         Cat
  shape      region      texture     outline

Layer 5 activations (2048 channels, 7×7):
  ┌────┐ ┌────┐ ┌────┐ ┌────┐
  │ █  │ │    │ │ ██ │ │  █ │
  │ ██ │ │  █ │ │    │ │ █  │  Very abstract
  └────┘ └────┘ └────┘ └────┘  Sparse, semantic
  
  High activation → this feature "fires" for the input image
  Zero activation → this feature is not present in the input
```

---

## 3. Hooks in PyTorch

PyTorch **hooks** allow you to intercept activations at any layer without modifying the model.

```python
import torch
import torch.nn as nn

# Forward hook: captures the OUTPUT of a layer
# Signature: hook(module, input, output)

class ActivationExtractor:
    """Extract intermediate activations using hooks."""
    
    def __init__(self, model, target_layers):
        self.activations = {}
        self.hooks = []
        
        for name, module in model.named_modules():
            if name in target_layers:
                hook = module.register_forward_hook(
                    self._make_hook(name)
                )
                self.hooks.append(hook)
    
    def _make_hook(self, name):
        def hook_fn(module, input, output):
            self.activations[name] = output.detach()
        return hook_fn
    
    def remove_hooks(self):
        for hook in self.hooks:
            hook.remove()
    
    def __del__(self):
        self.remove_hooks()


# Usage
from torchvision.models import resnet50, ResNet50_Weights

model = resnet50(weights=ResNet50_Weights.IMAGENET1K_V2)
model.eval()

extractor = ActivationExtractor(model, [
    'conv1', 'layer1', 'layer2', 'layer3', 'layer4'
])

# Forward pass
x = torch.randn(1, 3, 224, 224)
with torch.no_grad():
    output = model(x)

# Access activations
for name, act in extractor.activations.items():
    print(f"{name}: {act.shape}")

# Output:
# conv1: torch.Size([1, 64, 112, 112])
# layer1: torch.Size([1, 256, 56, 56])
# layer2: torch.Size([1, 512, 28, 28])
# layer3: torch.Size([1, 1024, 14, 14])
# layer4: torch.Size([1, 2048, 7, 7])

extractor.remove_hooks()
```

### Hook Types

```python
# FORWARD HOOK: captures output during forward pass
hook = module.register_forward_hook(
    lambda module, input, output: ...)

# BACKWARD HOOK: captures gradients during backward pass
hook = module.register_full_backward_hook(
    lambda module, grad_input, grad_output: ...)

# FORWARD PRE-HOOK: intercepts input before forward
hook = module.register_forward_pre_hook(
    lambda module, input: ...)

# Always remove hooks when done!
hook.remove()
```

---

## 4. Feature Map Interpretation

```
How to interpret feature maps:

1. BRIGHT REGIONS = high activation = feature detected here
   Dark regions = low/zero activation = feature absent

2. SPARSITY increases with depth:
   Conv1: Many channels active (edges everywhere)
   Conv5: Few channels active (specific semantic concepts)

3. SPATIAL CORRESPONDENCE:
   A bright region in a feature map corresponds to
   the same spatial location in the input image
   
   Input image:    Feature map:     Interpretation:
   ┌──────────┐    ┌──────────┐
   │    🐱    │    │    ██    │    "Face-like feature
   │   /  \   │    │   ████   │     detected here"
   │  |    |  │    │    ██    │
   │  (ears)  │    │          │
   └──────────┘    └──────────┘

4. SOME CHANNELS ARE DEAD:
   Always zero regardless of input → that filter learned nothing
   Common in overparameterized networks
```

---

## 5. Spatial Attention from Activations

```
Average feature map across channels → spatial attention map

Given activations A ∈ ℝ^(C × H × W):
  Attention = (1/C) Σ_c A[c, :, :]    ∈ ℝ^(H × W)

This shows WHERE the network is "looking" in the image.

  Input:              Average activation:      Overlay:
  ┌──────────────┐   ┌──────────────┐   ┌──────────────┐
  │     🐱       │   │     ██       │   │     🔴       │
  │    /  \      │   │    ████      │   │    🔴🔴      │
  │   |    |     │   │     ██       │   │     🔴       │
  │   (body)     │   │     ░        │   │     ░        │
  │   ------     │   │              │   │              │
  └──────────────┘   └──────────────┘   └──────────────┘
  
  🔴 = high attention (bright in heatmap)
  ░ = low attention (dark in heatmap)

Max feature map (take max across channels):
  MaxAttention = max_c A[c, :, :]     ∈ ℝ^(H × W)
  
  Shows the STRONGEST response at each spatial location
  regardless of which filter produced it
```

---

## 6. Code Example

```python
import torch
import numpy as np
import matplotlib.pyplot as plt
from torchvision.models import resnet50, ResNet50_Weights
from torchvision import transforms
from PIL import Image

# ============================================================
# 1. Setup
# ============================================================
model = resnet50(weights=ResNet50_Weights.IMAGENET1K_V2)
model.eval()

preprocess = transforms.Compose([
    transforms.Resize(256),
    transforms.CenterCrop(224),
    transforms.ToTensor(),
    transforms.Normalize([0.485, 0.456, 0.406],
                         [0.229, 0.224, 0.225]),
])

# Load image
# image = Image.open('cat.jpg').convert('RGB')
# input_tensor = preprocess(image).unsqueeze(0)

# Dummy input for demonstration
input_tensor = torch.randn(1, 3, 224, 224)

# ============================================================
# 2. Extract activations
# ============================================================
activations = {}

def get_hook(name):
    def hook(module, input, output):
        activations[name] = output.detach()
    return hook

# Register hooks
hooks = []
layers = {
    'conv1': model.conv1,
    'layer1': model.layer1,
    'layer2': model.layer2,
    'layer3': model.layer3,
    'layer4': model.layer4,
}
for name, layer in layers.items():
    hooks.append(layer.register_forward_hook(get_hook(name)))

# Forward pass
with torch.no_grad():
    output = model(input_tensor)

# Remove hooks
for h in hooks:
    h.remove()

# ============================================================
# 3. Visualize feature maps
# ============================================================
def plot_feature_maps(activation, layer_name, num_channels=16):
    """Plot first num_channels feature maps for a layer."""
    act = activation[0].cpu().numpy()  # remove batch dim
    num_channels = min(num_channels, act.shape[0])
    
    ncols = 8
    nrows = (num_channels + ncols - 1) // ncols
    
    fig, axes = plt.subplots(nrows, ncols,
                              figsize=(ncols * 2, nrows * 2))
    for i in range(num_channels):
        ax = axes[i // ncols, i % ncols] if nrows > 1 else axes[i]
        ax.imshow(act[i], cmap='viridis')
        ax.set_title(f"Ch {i}", fontsize=8)
        ax.axis('off')
    
    plt.suptitle(f"{layer_name} Feature Maps ({act.shape})", fontsize=14)
    plt.tight_layout()
    plt.savefig(f'feature_maps_{layer_name}.png', dpi=150)
    plt.close()

for name, act in activations.items():
    print(f"{name}: {act.shape}")
    plot_feature_maps(act, name)

# ============================================================
# 4. Spatial attention maps
# ============================================================
def plot_attention(activation, layer_name):
    """Create spatial attention map from activations."""
    act = activation[0].cpu().numpy()
    
    # Average across channels
    avg_attention = act.mean(axis=0)
    # Max across channels
    max_attention = act.max(axis=0)
    
    fig, (ax1, ax2, ax3) = plt.subplots(1, 3, figsize=(15, 5))
    
    # Original (un-normalized)
    # ax1.imshow(original_image)
    ax1.set_title("Input Image")
    ax1.axis('off')
    
    ax2.imshow(avg_attention, cmap='hot')
    ax2.set_title(f"{layer_name} Average Attention")
    ax2.axis('off')
    
    ax3.imshow(max_attention, cmap='hot')
    ax3.set_title(f"{layer_name} Max Attention")
    ax3.axis('off')
    
    plt.tight_layout()
    plt.savefig(f'attention_{layer_name}.png', dpi=150)
    plt.close()

for name, act in activations.items():
    plot_attention(act, name)

# ============================================================
# 5. Activation statistics per layer
# ============================================================
for name, act in activations.items():
    a = act[0]
    active_channels = (a.abs().sum(dim=(1, 2)) > 0).sum().item()
    sparsity = (a == 0).float().mean().item()
    print(f"{name}: shape={list(a.shape)}, "
          f"active_ch={active_channels}/{a.shape[0]}, "
          f"sparsity={sparsity:.1%}, "
          f"mean={a.mean():.3f}, max={a.max():.3f}")
```

---

## 7. Summary Table

| Concept | Key Idea |
|---|---|
| **Feature maps** | Activations at intermediate layers for a specific input |
| **vs Filter viz** | Filters show WHAT can be detected; feature maps show WHAT IS detected |
| **Hooks** | PyTorch mechanism to capture layer outputs without modifying model |
| **Forward hook** | `module.register_forward_hook(fn)` → captures output |
| **Sparsity** | Deeper layers → sparser activations (fewer features relevant) |
| **Spatial attention** | Average/max across channels → where the network focuses |
| **Dead channels** | Always-zero channels → filter learned nothing useful |

---

## 8. Revision Questions

1. **What is the difference** between filter visualization and feature map visualization? When would you use each?

2. **Write PyTorch code** using hooks to extract the output of `layer3` from a pretrained ResNet-50 for a given input image.

3. **How does activation sparsity** change across layers? Why do deeper layers have sparser activations?

4. **Explain how to create a spatial attention map** from feature maps. What does it tell you about the model's behavior?

5. **A feature map channel is always zero** for all test images. What does this indicate and should you be concerned?

---

## 9. Navigation

| Previous | Up | Next |
|---|---|---|
| [← Filter Visualization](./01-filter-visualization.md) | [Unit 9 Index](../09-Visualizing-CNNs/) | [Grad-CAM →](./03-grad-cam.md) |

---

> **References:**  
> - Zeiler, M. & Fergus, R. (2014). *Visualizing and Understanding Convolutional Networks.* ECCV.  
> - Yosinski, J. et al. (2015). *Understanding Neural Networks Through Deep Visualization.* ICML Workshop.
