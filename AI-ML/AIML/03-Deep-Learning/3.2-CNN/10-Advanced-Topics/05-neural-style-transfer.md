# Neural Style Transfer

[← Previous: Attention in CNNs](04-attention-in-cnns.md) | [🏠 Back to CNN Overview](../README.md)

---

## Overview

Neural Style Transfer (Gatys et al., 2015) combines the **content** of one image with the **artistic style** of another using CNN feature representations. It revealed that CNNs separately encode content (high-level structure) and style (textures, colors, patterns) — and both can be manipulated independently.

---

## 1. Core Idea

```
┌──────────────┐                              ┌──────────────┐
│              │                              │              │
│   Content    │──── Content Features ────┐   │    Style     │
│   Image      │                          │   │    Image     │
│   (photo)    │                          ▼   │  (painting)  │
│              │                    ┌──────────┤              │
└──────────────┘                    │ Optimize │└──────────────┘
                                    │ Generated│        │
                                    │  Image   │        │
                                    └──────────┘        │
                                          ▲             │
                                          │             │
                                    Style Features ─────┘

Goal: Generated image has CONTENT of photo + STYLE of painting
```

---

## 2. Content Representation

The **content** of an image is captured by feature maps in **deeper layers** of a pretrained CNN (e.g., VGG-19).

**Content Loss:** Match feature maps between generated image and content image.

```
L_content = (1/2) Σᵢⱼ (F^l_ij - P^l_ij)²
```

Where:
- `F^l` = feature map of generated image at layer `l`
- `P^l` = feature map of content image at layer `l`

Typically use `conv4_2` from VGG-19.

---

## 3. Style Representation: Gram Matrix

**Style** is captured by **correlations between feature maps** — the **Gram matrix**.

```
Gram Matrix:  G^l_ij = Σₖ F^l_ik · F^l_jk
```

The Gram matrix captures:
- Which features tend to **co-activate** (texture patterns)
- Feature correlations **independent of spatial location**
- Texture statistics of the image

```
Feature Maps at layer l:        Gram Matrix G:

Map 1: [edge features]          ┌─────────────────┐
Map 2: [color features]   →     │ G₁₁ G₁₂ G₁₃    │  Correlation
Map 3: [texture features]       │ G₂₁ G₂₂ G₂₃    │  between all
                                │ G₃₁ G₃₂ G₃₃    │  pairs of maps
                                └─────────────────┘
                                Size: C × C (channels × channels)
```

**Style Loss:** Match Gram matrices across multiple layers.

```
L_style = Σ_l  wₗ · (1 / (4 · Nₗ² · Mₗ²)) · Σᵢⱼ (G^l_ij - A^l_ij)²
```

Where:
- `G^l` = Gram matrix of generated image at layer `l`
- `A^l` = Gram matrix of style image at layer `l`
- `Nₗ` = number of channels, `Mₗ` = H×W at layer `l`
- `wₗ` = weight for layer `l`

Typically use `conv1_1`, `conv2_1`, `conv3_1`, `conv4_1`, `conv5_1`.

---

## 4. Total Loss

```
L_total = α · L_content  +  β · L_style  +  γ · L_tv
```

- `α` : content weight (preserve structure)
- `β` : style weight (apply artistic style)
- `γ` : total variation weight (smoothness regularization)
- `α/β` ratio controls content vs style balance (typical: 1e-3 to 1e-5)

**Total Variation Loss** (smoothness):
```
L_tv = Σᵢⱼ (x_{i+1,j} - x_{i,j})² + (x_{i,j+1} - x_{i,j})²
```

---

## 5. Optimization-Based Style Transfer

**Procedure:**
1. Load pretrained VGG-19 (freeze weights)
2. Extract content features from content image
3. Extract style features (Gram matrices) from style image
4. Initialize generated image (random noise or copy of content)
5. **Optimize pixel values** of generated image to minimize `L_total`
6. Use L-BFGS or Adam optimizer

```
For each iteration:
    1. Forward pass generated image through VGG
    2. Compute L_content (compare features at conv4_2)
    3. Compute L_style (compare Gram matrices at conv1-5)
    4. Compute L_total = α·L_content + β·L_style
    5. Backward pass → gradients w.r.t. generated image PIXELS
    6. Update generated image pixels
```

### PyTorch Implementation

```python
import torch
import torch.nn as nn
import torch.optim as optim
import torchvision.models as models
import torchvision.transforms as transforms
from PIL import Image

# --- Feature Extractor ---
class StyleContentModel(nn.Module):
    def __init__(self):
        super().__init__()
        vgg = models.vgg19(pretrained=True).features.eval()
        
        # Layer indices for style and content
        self.style_layers = [0, 5, 10, 19, 28]   # conv1_1 to conv5_1
        self.content_layers = [25]                 # conv4_2
        
        self.model = vgg
        for param in self.model.parameters():
            param.requires_grad = False
    
    def forward(self, x):
        style_features = []
        content_features = []
        
        for i, layer in enumerate(self.model):
            x = layer(x)
            if i in self.style_layers:
                style_features.append(x)
            if i in self.content_layers:
                content_features.append(x)
        
        return style_features, content_features

# --- Gram Matrix ---
def gram_matrix(feature_map):
    B, C, H, W = feature_map.shape
    features = feature_map.view(B, C, H * W)
    gram = torch.bmm(features, features.transpose(1, 2))
    return gram / (C * H * W)

# --- Style Transfer ---
def style_transfer(content_img, style_img, num_steps=300, 
                   alpha=1, beta=1e6):
    
    model = StyleContentModel().cuda()
    
    # Extract target features
    with torch.no_grad():
        style_targets, _ = model(style_img)
        style_grams = [gram_matrix(f) for f in style_targets]
        _, content_targets = model(content_img)
    
    # Initialize generated image (start from content)
    generated = content_img.clone().requires_grad_(True)
    optimizer = optim.Adam([generated], lr=0.01)
    
    for step in range(num_steps):
        optimizer.zero_grad()
        
        style_feats, content_feats = model(generated)
        
        # Content loss
        content_loss = sum(
            nn.functional.mse_loss(f, t) 
            for f, t in zip(content_feats, content_targets)
        )
        
        # Style loss
        style_loss = sum(
            nn.functional.mse_loss(gram_matrix(f), g)
            for f, g in zip(style_feats, style_grams)
        )
        
        # Total loss
        loss = alpha * content_loss + beta * style_loss
        loss.backward()
        optimizer.step()
        
        # Clamp pixel values
        with torch.no_grad():
            generated.clamp_(0, 1)
        
        if step % 50 == 0:
            print(f"Step {step}: Content={content_loss.item():.4f}, "
                  f"Style={style_loss.item():.4f}")
    
    return generated
```

---

## 6. Fast Style Transfer (Feed-Forward)

Optimization-based is **slow** (~minutes per image). **Fast style transfer** (Johnson et al., 2016) trains a feed-forward network to apply a specific style instantly.

```
Training (once per style):
┌─────────┐     ┌──────────────┐     ┌──────────┐
│ Content │────►│ Transformer  │────►│Generated │
│ Images  │     │ Network      │     │ Image    │
│ (batch) │     │ (to train)   │     │          │
└─────────┘     └──────────────┘     └──────────┘
                                          │
                                          ▼
                                    VGG Loss Network
                                    (content + style loss)
                                          │
                                          ▼
                                    Backprop to update
                                    Transformer Network

Inference (instant):
Content Image → Transformer Network → Stylized Image
                (single forward pass, ~real-time!)
```

**Transformer Network Architecture:**
```
Input → Conv(9×9) → Conv(3×3)↓ → Conv(3×3)↓ 
    → ResBlock×5 → Deconv(3×3)↑ → Deconv(3×3)↑ 
    → Conv(9×9) → Output
```

---

## 7. Arbitrary Style Transfer

**AdaIN (Adaptive Instance Normalization)** — Huang & Belongie, 2017:

```
AdaIN(content, style) = σ(style) × ((content - μ(content)) / σ(content)) + μ(style)
```

Align the **mean and variance** of content features to match style features. Works for **any style** with a single network.

---

## 8. Applications

| Application | Description |
|---|---|
| **Artistic filters** | Apply painting styles to photos (apps like Prisma) |
| **Video stylization** | Apply consistent style across video frames |
| **Design tools** | Generate artistic variations |
| **Data augmentation** | Stylize training images for domain adaptation |
| **Texture synthesis** | Generate textures from examples |
| **Domain adaptation** | Transfer visual style between domains |

---

## Summary Table

| Method | Speed | Any Style? | Quality |
|---|:---:|:---:|:---:|
| **Optimization-based** (Gatys) | Slow (minutes) | ✅ | ✅ Best |
| **Fast feed-forward** (Johnson) | Real-time | ❌ (one per model) | Good |
| **AdaIN** | Real-time | ✅ | Good |

| Component | CNN Layer | Captures |
|---|---|---|
| **Content** | Deep layers (conv4_2) | High-level structure, objects |
| **Style** | All layers (conv1-5) | Textures, colors, patterns |
| **Gram Matrix** | Correlation of feature maps | Style statistics |

---

## Quick Revision Questions

1. **What does the Gram matrix capture?**
   - Correlations between feature channels — encodes texture/style statistics independent of spatial arrangement

2. **Why use deeper layers for content and multiple layers for style?**
   - Deeper layers capture high-level structure (content); style manifests at all scales (edges to textures), so multiple layers needed

3. **What is optimized in Gatys' method?**
   - The pixel values of the generated image (not network weights) — VGG is frozen

4. **How does fast style transfer achieve real-time speed?**
   - A feed-forward network is trained to apply a specific style in a single forward pass

5. **What is AdaIN?**
   - Adaptive Instance Normalization — aligns mean/variance of content features to style features for arbitrary style transfer

6. **What does the α/β ratio control?**
   - Balance between preserving content structure (high α/β) and applying style (low α/β)

---

[← Previous: Attention in CNNs](04-attention-in-cnns.md) | [🏠 Back to CNN Overview](../README.md)
