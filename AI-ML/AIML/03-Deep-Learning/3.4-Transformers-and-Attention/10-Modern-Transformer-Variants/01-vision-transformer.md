# Vision Transformer (ViT)

> **Navigation:**
> Previous: [← Enc-Dec Applications](../09-Encoder-Decoder-Transformers/04-applications.md) | Next: [Swin Transformer →](02-swin-transformer.md)

---

## Overview

The Vision Transformer (ViT), introduced by Dosovitskiy et al. in their landmark 2020 paper
*"An Image Is Worth 16×16 Words,"* demonstrated that a pure transformer architecture — originally
designed for natural language processing — can achieve state-of-the-art results on image
classification when pre-trained on large datasets. ViT breaks an image into a sequence of fixed-size
patches, linearly embeds each patch, and feeds the resulting sequence into a standard transformer
encoder. This simple yet powerful approach challenged the long-held assumption that convolutional
neural networks (CNNs) are essential for computer vision, and opened the door to a unified
architecture for both vision and language tasks.

---

## 1. Motivation: Why Transformers for Vision?

CNNs exploit **translation equivariance** and **locality** through convolutions. These inductive
biases are powerful but limit the model's ability to capture long-range dependencies in early
layers. Transformers, by contrast, use **global self-attention** from the very first layer, enabling
every patch to attend to every other patch immediately.

| Property              | CNN                         | ViT                          |
|-----------------------|-----------------------------|------------------------------|
| Inductive bias        | Strong (locality, equiv.)   | Weak (only sequence order)   |
| Receptive field       | Grows with depth            | Global from layer 1          |
| Data efficiency       | High (works with less data) | Low (needs large pre-train)  |
| Scalability           | Saturates at scale          | Scales well with data/compute|
| Architecture          | Conv → Pool → FC            | Patch Embed → Transformer    |

---

## 2. Image Patches as Tokens

### 2.1 Patch Extraction

Given an input image **x ∈ ℝ^(H × W × C)**, ViT splits it into a grid of non-overlapping patches
of size **P × P**:

```
Number of patches:  N = (H × W) / P²

For a 224 × 224 image with P = 16:
    N = (224 × 224) / (16 × 16) = 196 patches
```

Each patch **xₚ ∈ ℝ^(P² · C)** is flattened into a 1D vector:

```
Patch shape:  (16, 16, 3)  →  flattened: (768,)
```

### 2.2 Linear Projection (Patch Embedding)

Each flattened patch is projected to the model dimension **D** via a learnable linear layer **E**:

```
zₚ = xₚ · E        where E ∈ ℝ^((P²·C) × D)

For ViT-Base:  D = 768
    E ∈ ℝ^(768 × 768)  (since P²·C = 16×16×3 = 768)
```

---

## 3. The [CLS] Token and Position Embeddings

### 3.1 Classification Token

A learnable **[CLS]** token **z_cls ∈ ℝ^D** is prepended to the patch sequence. After the
transformer encoder, the output corresponding to [CLS] is used as the image representation
for classification.

### 3.2 Learned 1D Position Embeddings

Since transformers have no inherent notion of spatial order, **learned 1D position embeddings**
**E_pos ∈ ℝ^((N+1) × D)** are added to encode patch positions:

```
z₀ = [ z_cls ; z₁E ; z₂E ; ... ; z_NE ] + E_pos

where:
    z₀   ∈ ℝ^((N+1) × D)     full input to the transformer
    z_cls ∈ ℝ^(1 × D)          classification token
    zᵢE   ∈ ℝ^(1 × D)          embedded patch i
    E_pos  ∈ ℝ^((N+1) × D)     position embeddings
```

> **Note:** Dosovitskiy et al. found that learned 1D position embeddings perform comparably to
> more complex 2D-aware position embeddings, which is surprising given the 2D nature of images.

---

## 4. ViT Architecture (ASCII Diagram)

```
INPUT IMAGE (224 × 224 × 3)
         │
         ▼
┌─────────────────────────────────────────────────┐
│          PATCH EXTRACTION (16 × 16)             │
│  ┌────┬────┬────┬────┬─── ... ───┬────┐        │
│  │ P1 │ P2 │ P3 │ P4 │          │P196│        │
│  └────┴────┴────┴────┴─── ... ───┴────┘        │
│         196 patches of size (768,)              │
└─────────────────────────────────────────────────┘
         │
         ▼
┌─────────────────────────────────────────────────┐
│       LINEAR PROJECTION  E ∈ ℝ^(768 × 768)     │
│  Each patch (768,) → embedding (768,)           │
└─────────────────────────────────────────────────┘
         │
         ▼
┌─────────────────────────────────────────────────┐
│  PREPEND [CLS] TOKEN + ADD POSITION EMBEDDINGS  │
│                                                 │
│  [CLS] z₁  z₂  z₃  ...  z₁₉₆                  │
│    +    +   +   +         +                     │
│  p₀   p₁  p₂  p₃  ...  p₁₉₆   ← E_pos        │
│                                                 │
│  Result: (197 × 768) input sequence             │
└─────────────────────────────────────────────────┘
         │
         ▼
┌─────────────────────────────────────────────────┐
│         TRANSFORMER ENCODER (L layers)          │
│                                                 │
│  ┌───────────────────────────────────────────┐  │
│  │  Layer Norm                               │  │
│  │  Multi-Head Self-Attention (12 heads)     │  │
│  │  + Residual Connection                    │  │
│  │  Layer Norm                               │  │
│  │  MLP (768 → 3072 → 768)                  │  │
│  │  + Residual Connection                    │  │
│  └───────────────────────────────────────────┘  │
│              × 12 layers (ViT-Base)             │
└─────────────────────────────────────────────────┘
         │
         ▼
┌─────────────────────────────────────────────────┐
│  EXTRACT [CLS] OUTPUT → MLP HEAD → CLASSES     │
│                                                 │
│  z_L[0] ──→ LayerNorm ──→ Linear(768, K)       │
│                              K = num classes    │
└─────────────────────────────────────────────────┘
         │
         ▼
    CLASS PREDICTION
```

---

## 5. Mathematical Formulation

### 5.1 Full Forward Pass

```
Step 1 — Patch Embedding + Position:
    z₀ = [x_cls ; x₁E ; x₂E ; ... ; x_NE] + E_pos

Step 2 — Transformer Encoder (for each layer ℓ = 1, ..., L):
    z'_ℓ  = MSA(LN(z_{ℓ-1})) + z_{ℓ-1}           (attention + residual)
    z_ℓ   = MLP(LN(z'_ℓ))    + z'_ℓ               (feed-forward + residual)

Step 3 — Classification Head:
    y = LN(z_L⁰)                                   (take [CLS] output)
    ŷ = Linear(y)                                   (project to classes)
```

### 5.2 Multi-Head Self-Attention (MSA)

```
Q = z · W_Q,    K = z · W_K,    V = z · W_V

Attention(Q, K, V) = softmax(Q K^T / √d_k) · V

MSA(z) = Concat(head₁, ..., head_h) · W_O
    where head_i = Attention(z·W_Qⁱ, z·W_Kⁱ, z·W_Vⁱ)
```

### 5.3 MLP Block

```
MLP(x) = GELU(x · W₁ + b₁) · W₂ + b₂

    W₁ ∈ ℝ^(D × 4D),  W₂ ∈ ℝ^(4D × D)
    For ViT-Base: 768 → 3072 → 768
```

---

## 6. ViT Model Variants

| Model      | Layers (L) | Hidden (D) | Heads (h) | MLP dim | Params  |
|------------|-----------|------------|-----------|---------|---------|
| ViT-Base   | 12        | 768        | 12        | 3072    | 86M     |
| ViT-Large  | 24        | 1024       | 16        | 4096    | 307M    |
| ViT-Huge   | 32        | 1280       | 16        | 5120    | 632M    |

---

## 7. Worked Example

**Setting:** ViT-Base, image 224×224×3, patch size 16×16.

```
Step 1: Patch extraction
    N = (224 / 16) × (224 / 16) = 14 × 14 = 196 patches
    Each patch: (16 × 16 × 3) = 768 values

Step 2: Linear projection
    E ∈ ℝ^(768 × 768)
    Each patch → 768-dim embedding

Step 3: Prepend [CLS] + add positional embeddings
    Sequence length: 196 + 1 = 197
    Input tensor: (197, 768)

Step 4: Transformer Encoder (12 layers)
    Self-attention per layer:
        Q, K, V ∈ ℝ^(197 × 768)
        Attention weights: (197 × 197) per head
        12 heads, d_k = 768/12 = 64

    FLOPs for one MSA layer (approximate):
        4 × 197 × 768² + 2 × 197² × 768 ≈ 520M FLOPs

Step 5: Classification
    z_L[0] ∈ ℝ^768 → LayerNorm → Linear(768, 1000) → 1000 logits
```

---

## 8. PyTorch Implementation

```python
import torch
import torch.nn as nn

class PatchEmbedding(nn.Module):
    """Split image into patches and embed them."""
    def __init__(self, img_size=224, patch_size=16, in_channels=3, embed_dim=768):
        super().__init__()
        self.img_size = img_size
        self.patch_size = patch_size
        self.num_patches = (img_size // patch_size) ** 2

        # Conv2d acts as a linear projection of flattened patches
        self.projection = nn.Conv2d(
            in_channels, embed_dim,
            kernel_size=patch_size, stride=patch_size
        )

    def forward(self, x):
        # x: (B, C, H, W)
        x = self.projection(x)          # (B, embed_dim, H/P, W/P)
        x = x.flatten(2)                # (B, embed_dim, num_patches)
        x = x.transpose(1, 2)           # (B, num_patches, embed_dim)
        return x


class VisionTransformer(nn.Module):
    """Simplified Vision Transformer (ViT)."""
    def __init__(
        self,
        img_size=224,
        patch_size=16,
        in_channels=3,
        num_classes=1000,
        embed_dim=768,
        depth=12,
        num_heads=12,
        mlp_ratio=4.0,
        dropout=0.1,
    ):
        super().__init__()

        # --- Patch embedding ---
        self.patch_embed = PatchEmbedding(img_size, patch_size, in_channels, embed_dim)
        num_patches = self.patch_embed.num_patches

        # --- [CLS] token ---
        self.cls_token = nn.Parameter(torch.zeros(1, 1, embed_dim))

        # --- Learned 1D position embeddings ---
        self.pos_embed = nn.Parameter(torch.zeros(1, num_patches + 1, embed_dim))
        self.pos_drop = nn.Dropout(dropout)

        # --- Transformer encoder ---
        encoder_layer = nn.TransformerEncoderLayer(
            d_model=embed_dim,
            nhead=num_heads,
            dim_feedforward=int(embed_dim * mlp_ratio),
            dropout=dropout,
            activation='gelu',
            batch_first=True,
            norm_first=True,       # Pre-norm (as in original ViT)
        )
        self.transformer = nn.TransformerEncoder(encoder_layer, num_layers=depth)

        # --- Classification head ---
        self.norm = nn.LayerNorm(embed_dim)
        self.head = nn.Linear(embed_dim, num_classes)

        self._init_weights()

    def _init_weights(self):
        nn.init.trunc_normal_(self.pos_embed, std=0.02)
        nn.init.trunc_normal_(self.cls_token, std=0.02)
        nn.init.trunc_normal_(self.head.weight, std=0.02)

    def forward(self, x):
        B = x.shape[0]

        # Patch embedding: (B, N, D)
        x = self.patch_embed(x)

        # Prepend [CLS] token: (B, N+1, D)
        cls_tokens = self.cls_token.expand(B, -1, -1)
        x = torch.cat([cls_tokens, x], dim=1)

        # Add position embeddings
        x = self.pos_drop(x + self.pos_embed)

        # Transformer encoder
        x = self.transformer(x)

        # Classification: take [CLS] output
        cls_output = self.norm(x[:, 0])
        logits = self.head(cls_output)
        return logits


# --- Quick test ---
if __name__ == "__main__":
    model = VisionTransformer(
        img_size=224, patch_size=16, num_classes=1000,
        embed_dim=768, depth=12, num_heads=12
    )
    img = torch.randn(2, 3, 224, 224)
    logits = model(img)
    print(f"Input shape:  {img.shape}")          # (2, 3, 224, 224)
    print(f"Output shape: {logits.shape}")        # (2, 1000)
    print(f"Parameters:   {sum(p.numel() for p in model.parameters()) / 1e6:.1f}M")
```

---

## 9. ViT vs CNN: Detailed Comparison

```
                   DATA EFFICIENCY vs SCALE

  Accuracy │              ViT ──────────────*
           │            /
           │          /         CNN ────*
           │        /         /
           │  CNN /         /
           │   / ViT      /
           │ /   /       /
           │/  /       /
           │ /       /
           └──────────────────────────────────
            10M    100M      1B       Data Size

Key Insight:
  - CNNs > ViT with small data (strong inductive biases help)
  - ViT > CNNs with large data (fewer biases = more capacity)
```

---

## 10. Real-World Applications

| Application                | Description                                             |
|----------------------------|---------------------------------------------------------|
| **Image Classification**   | ImageNet, CIFAR — ViT-H achieves 90.45% on ImageNet    |
| **Object Detection**       | ViTDet — ViT backbone for detection (DETR-style)       |
| **Medical Imaging**        | ViTs for X-ray, pathology, and retinal image analysis   |
| **Remote Sensing**         | Satellite image classification with ViT backbones       |
| **Self-Supervised Learning** | DINO, MAE — ViT excels with self-supervised pre-training |
| **Multi-Modal Models**     | CLIP — ViT as image encoder paired with text transformer |
| **Video Understanding**    | ViViT, TimeSformer — extending ViT to video sequences   |

---

## 11. Summary Table

| Concept                | Details                                                     |
|------------------------|-------------------------------------------------------------|
| Paper                  | Dosovitskiy et al., *An Image Is Worth 16×16 Words* (2020) |
| Key idea               | Treat image patches as tokens, apply standard transformer   |
| Patch size             | Typically 16×16 (also 14×14, 32×32)                        |
| Position encoding      | Learned 1D embeddings                                       |
| Classification         | [CLS] token → MLP head                                     |
| Pre-norm vs Post-norm  | ViT uses pre-norm (LayerNorm before attention/MLP)          |
| Data requirement       | Needs large-scale pre-training (JFT-300M, ImageNet-21k)    |
| Advantage              | Scales better than CNNs with more data/compute             |
| Limitation             | Poor with small datasets without pre-training               |
| Complexity             | O(N²·D) per layer where N = number of patches               |

---

## 12. Revision Questions

1. **Why does ViT use a [CLS] token?** Explain its role in classification and how it aggregates
   information from all patches through self-attention across multiple layers.

2. **How does patch size P affect ViT's performance and efficiency?** Smaller patches increase N
   quadratically — discuss the trade-off between resolution and computational cost.

3. **Why does ViT require large-scale pre-training while CNNs perform well with less data?**
   Relate your answer to the difference in inductive biases.

4. **Derive the total number of parameters in ViT-Base.** Account for patch embedding, position
   embeddings, [CLS] token, 12 transformer layers, and the classification head.

5. **Compare learned 1D position embeddings to 2D-aware alternatives.** Why did the authors find
   minimal difference? What does this suggest about how ViT learns spatial structure?

6. **How would you adapt ViT for variable-resolution images?** Discuss strategies like
   interpolating position embeddings or using relative position encodings.

---

> **Navigation:**
> Previous: [← Enc-Dec Applications](../09-Encoder-Decoder-Transformers/04-applications.md) | Next: [Swin Transformer →](02-swin-transformer.md)
