# 6. Layer Normalization and Other Normalization Variants

> **Unit 7 · Regularization Techniques** — Normalizing across features instead of across batch

---

## Chapter Overview

BatchNorm normalizes across the **batch dimension** — it requires reasonably sized mini-batches and is batch-dependent. **Layer Normalization** (Ba et al., 2016) normalizes across the **feature dimension** instead, making it independent of batch size and well-suited for **RNNs**, **Transformers**, and scenarios with variable-length sequences. This chapter covers LayerNorm, GroupNorm, InstanceNorm, and provides a comprehensive comparison.

---

## 1. The Normalization Dimension

### Core Difference

The key distinction between normalization methods is **which dimensions** they compute statistics over:

```
  Input tensor: (B, C, H, W)   or   (B, D)
  
  ┌───────────────────────────────────────────────────────┐
  │  BatchNorm:     Normalize across BATCH (B dimension)  │
  │                 Statistics per feature/channel         │
  │                                                       │
  │  LayerNorm:     Normalize across FEATURES (C/D dim)   │
  │                 Statistics per sample                  │
  │                                                       │
  │  InstanceNorm:  Normalize across SPATIAL (H, W)       │
  │                 Statistics per sample per channel      │
  │                                                       │
  │  GroupNorm:     Normalize across GROUPS of channels    │
  │                 Statistics per sample per group        │
  └───────────────────────────────────────────────────────┘
```

### Visual Comparison (3D: Batch × Channel × Spatial)

```
  Tensor shape: (B, C, H×W)
  
  BatchNorm:           LayerNorm:          InstanceNorm:       GroupNorm:
  ┌─────────┐          ┌─────────┐        ┌─────────┐        ┌─────────┐
  │█████████│ C        │█ │ │ │ │ C       │█│ │ │ │ C        │██│ │ │ C  (G=2)
  │█████████│          │█ │ │ │ │         │ │█│ │ │          │██│ │ │
  │█████████│          │█ │ │ │ │         │ │ │█│ │          │ │ │██│
  │█████████│          │█ │ │ │ │         │ │ │ │█│          │ │ │██│
  └─────────┘          └─────────┘        └─────────┘        └─────────┘
  ←── B ──→            ←── B ──→          ←── B ──→          ←── B ──→
  
  █ = dimensions over which statistics are computed
  
  BN: all samples,     LN: all features,  IN: spatial only,  GN: group of
      same feature          same sample        per channel        channels
```

---

## 2. Layer Normalization

### Algorithm

For a single sample with D features:

```
  Input: x ∈ ℝᴰ  (one sample, D features)
  
  1. Compute mean:     μ = (1/D) Σⱼ₌₁ᴰ xⱼ
  2. Compute variance: σ² = (1/D) Σⱼ₌₁ᴰ (xⱼ - μ)²
  3. Normalize:        x̂ⱼ = (xⱼ - μ) / √(σ² + ε)
  4. Scale and shift:  yⱼ = γⱼ · x̂ⱼ + βⱼ
```

### Key Difference from BatchNorm

```
  BatchNorm:                         LayerNorm:
  
  For feature j:                     For sample i:
  μⱼ = mean over batch B            μᵢ = mean over features D
  σ²ⱼ = var over batch B            σ²ᵢ = var over features D
  
  Depends on other samples           Independent of other samples
  in mini-batch                       in mini-batch
  
  ↓                                  ↓
  
  Needs batch size > 1               Works with batch size = 1 ✓
  Statistics change with batch        Statistics are deterministic ✓
  Different train/eval behavior       Same behavior train/eval ✓
```

### Why LayerNorm for Transformers?

```
  Transformer input: (Batch, Sequence_Length, Hidden_Dim)
  
  BatchNorm would normalize across batch:
  ✗ Sequences have different lengths → padding makes stats noisy
  ✗ Auto-regressive models see one token at a time (batch=1)
  ✗ Different positions have different semantic roles
  
  LayerNorm normalizes across hidden_dim:
  ✓ Each token is normalized independently
  ✓ No dependence on batch or sequence length
  ✓ Works perfectly for masked/causal attention
```

---

## 3. LayerNorm in Practice

### For Transformers / Fully Connected

```python
import torch
import torch.nn as nn

# LayerNorm for Transformer hidden states
hidden_dim = 768
layer_norm = nn.LayerNorm(hidden_dim)

# Input: (batch, seq_len, hidden_dim)
x = torch.randn(4, 10, hidden_dim)
y = layer_norm(x)

# Statistics computed over last dimension (768 features) per token
print(f"Input mean per token: {x[0, 0].mean():.4f}")
print(f"Output mean per token: {y[0, 0].mean():.4f}")  # ≈ 0
print(f"Output std per token: {y[0, 0].std():.4f}")   # ≈ 1
```

### Pre-Norm vs Post-Norm in Transformers

```
  Post-Norm (original Transformer):
  
  x → Attention → + → LayerNorm → FFN → + → LayerNorm → output
       ↑          ↑                       ↑
       └──────────┘ residual              └──── residual
  
  Pre-Norm (modern, more stable):
  
  x → LayerNorm → Attention → + → LayerNorm → FFN → + → output
                               ↑                      ↑
                               └── residual x          └── residual
  
  Pre-Norm is more stable for training deep Transformers.
```

---

## 4. Group Normalization

### Concept (Wu & He, 2018)

Divide channels into **G groups** and normalize within each group:

```
  C = 32 channels, G = 8 groups → 4 channels per group
  
  Group 1: channels 0-3   → compute μ₁, σ₁² over these 4 channels
  Group 2: channels 4-7   → compute μ₂, σ₂² over these 4 channels
  ...
  Group 8: channels 28-31 → compute μ₈, σ₈² over these 4 channels
```

### Why GroupNorm?

```
  BatchNorm fails with small batch sizes (B < 8):
  → Noisy statistics, poor training
  
  LayerNorm normalizes ALL channels together:
  → May be too aggressive for conv layers (channels have different roles)
  
  GroupNorm is a MIDDLE GROUND:
  → Groups of related channels normalized together
  → Independent of batch size ✓
  → Respects channel structure ✓
```

### Special Cases

```
  GroupNorm(G=C):   → InstanceNorm  (each channel is its own group)
  GroupNorm(G=1):   → LayerNorm     (all channels in one group)
  
  InstanceNorm ← GroupNorm → LayerNorm
  (G=C)          (1<G<C)    (G=1)
```

### PyTorch

```python
# GroupNorm with 8 groups over 64 channels
gn = nn.GroupNorm(num_groups=8, num_channels=64)
x = torch.randn(16, 64, 32, 32)  # (B, C, H, W)
y = gn(x)
```

---

## 5. Instance Normalization

### Concept (Ulyanov et al., 2016)

Normalize each channel of each sample independently over spatial dimensions:

```
  For sample i, channel c:
  μ_ic = mean over all H×W spatial positions
  σ²_ic = var over all H×W spatial positions
  
  x̂ = (x - μ_ic) / √(σ²_ic + ε)
```

### Primary Use Case: Style Transfer

```
  Style transfer needs to remove instance-specific contrast/brightness:
  
  Photo (high contrast):     After InstanceNorm:
  ┌─────────────┐            ┌─────────────┐
  │ ███  ░░░    │            │ ██   ░░     │
  │ ███  ░░░    │   →        │ ██   ░░     │  ← normalized contrast
  │ ███  ░░░    │            │ ██   ░░     │     per channel
  └─────────────┘            └─────────────┘
  
  Now style can be applied uniformly.
```

### PyTorch

```python
inst_norm = nn.InstanceNorm2d(64)  # 64 channels
x = torch.randn(16, 64, 32, 32)
y = inst_norm(x)
```

---

## 6. Comprehensive Comparison

### Normalization Dimensions

| Method | Normalize Over | Statistics Per | Batch Dependent? |
|---|---|---|---|
| **BatchNorm** | Batch (B) | Feature/Channel | **Yes** |
| **LayerNorm** | Features (C,H,W) | Sample | No |
| **InstanceNorm** | Spatial (H,W) | Sample × Channel | No |
| **GroupNorm** | Group of C, Spatial | Sample × Group | No |

### When to Use Each

```
  ┌──────────────────────────────────────────────────────┐
  │  Task / Architecture          Best Normalization     │
  │  ─────────────────────────    ────────────────────── │
  │  CNN + large batch (≥32)      BatchNorm              │
  │  CNN + small batch (<8)       GroupNorm               │
  │  Transformer / BERT / GPT     LayerNorm              │
  │  RNN / LSTM                   LayerNorm              │
  │  Style Transfer               InstanceNorm           │
  │  GAN discriminator            InstanceNorm/SpectralN │
  │  Reinforcement learning       LayerNorm              │
  │  Object detection             GroupNorm (small batch) │
  └──────────────────────────────────────────────────────┘
```

### Performance vs Batch Size

```
  Accuracy ↑
           │
           │  ── BN (batch=32): best!
  94%      │  ── GN
  93%      │  ─── LN
           │
           │
  90%      │  ── GN ── GN ── GN ── GN    GN: constant regardless of batch
           │  ── LN ── LN ── LN ── LN    LN: constant
           │
  85%      │  ── BN (batch=2): terrible!
           │
           └──────────────────────────────→ Batch Size
            2    4    8    16   32   64
           
  BN performance degrades with small batch
  GN and LN are batch-size independent
```

---

## 7. Python Implementation

### Manual LayerNorm

```python
class ManualLayerNorm(nn.Module):
    """Layer Normalization from scratch."""
    
    def __init__(self, normalized_shape, eps=1e-5):
        super().__init__()
        if isinstance(normalized_shape, int):
            normalized_shape = (normalized_shape,)
        self.normalized_shape = normalized_shape
        self.eps = eps
        
        # Learnable parameters
        self.gamma = nn.Parameter(torch.ones(normalized_shape))
        self.beta = nn.Parameter(torch.zeros(normalized_shape))
    
    def forward(self, x):
        # Compute mean and var over the last len(normalized_shape) dimensions
        dims = tuple(range(-len(self.normalized_shape), 0))
        
        mean = x.mean(dim=dims, keepdim=True)
        var = x.var(dim=dims, keepdim=True, unbiased=False)
        
        # Normalize
        x_norm = (x - mean) / torch.sqrt(var + self.eps)
        
        # Scale and shift
        return self.gamma * x_norm + self.beta

# Verify against PyTorch
torch.manual_seed(42)
x = torch.randn(4, 10, 768)

ln_pytorch = nn.LayerNorm(768)
ln_manual = ManualLayerNorm(768)
ln_manual.gamma.data = ln_pytorch.weight.data.clone()
ln_manual.beta.data = ln_pytorch.bias.data.clone()

out_pytorch = ln_pytorch(x)
out_manual = ln_manual(x)
print(f"Max difference: {(out_pytorch - out_manual).abs().max():.1e}")  # ≈ 0
```

### Comparing All Normalization Methods

```python
import matplotlib.pyplot as plt

torch.manual_seed(42)
x = torch.randn(2, 8, 4, 4)  # (B=2, C=8, H=4, W=4)

norms = {
    'BatchNorm2d': nn.BatchNorm2d(8),
    'LayerNorm': nn.LayerNorm([8, 4, 4]),
    'InstanceNorm2d': nn.InstanceNorm2d(8, affine=True),
    'GroupNorm(G=4)': nn.GroupNorm(4, 8),
    'GroupNorm(G=8)': nn.GroupNorm(8, 8),  # = InstanceNorm
    'GroupNorm(G=1)': nn.GroupNorm(1, 8),  # = LayerNorm
}

print(f"{'Method':<20} {'Output Mean':>12} {'Output Std':>12}")
print('─' * 46)
for name, norm in norms.items():
    norm.train()
    out = norm(x)
    print(f"{name:<20} {out.mean():>12.4f} {out.std():>12.4f}")
```

### Transformer Block with LayerNorm

```python
class TransformerBlock(nn.Module):
    """Pre-Norm Transformer block with LayerNorm."""
    
    def __init__(self, d_model=512, n_heads=8, d_ff=2048, dropout=0.1):
        super().__init__()
        
        # Layer norms
        self.ln1 = nn.LayerNorm(d_model)
        self.ln2 = nn.LayerNorm(d_model)
        
        # Multi-head attention
        self.attention = nn.MultiheadAttention(d_model, n_heads, 
                                                dropout=dropout, batch_first=True)
        
        # Feed-forward
        self.ff = nn.Sequential(
            nn.Linear(d_model, d_ff),
            nn.GELU(),
            nn.Dropout(dropout),
            nn.Linear(d_ff, d_model),
            nn.Dropout(dropout),
        )
    
    def forward(self, x, mask=None):
        # Pre-Norm: LayerNorm BEFORE each sublayer
        
        # Self-attention with residual
        x_norm = self.ln1(x)
        attn_out, _ = self.attention(x_norm, x_norm, x_norm, attn_mask=mask)
        x = x + attn_out
        
        # Feed-forward with residual
        x_norm = self.ln2(x)
        ff_out = self.ff(x_norm)
        x = x + ff_out
        
        return x

# Usage
block = TransformerBlock(d_model=512, n_heads=8)
x = torch.randn(4, 20, 512)  # (batch=4, seq_len=20, d_model=512)
out = block(x)
print(f"Output shape: {out.shape}")  # (4, 20, 512)
```

---

## 8. RMSNorm (Bonus)

A simplified LayerNorm used in LLaMA and other modern LLMs:

```
  Standard LayerNorm: y = γ · (x - μ) / √(σ² + ε) + β
  
  RMSNorm:           y = γ · x / √(mean(x²) + ε)
  
  Difference: No centering (no μ subtraction), no bias β
  Advantage: ~10-15% faster, works equally well in practice
```

```python
class RMSNorm(nn.Module):
    def __init__(self, d_model, eps=1e-6):
        super().__init__()
        self.weight = nn.Parameter(torch.ones(d_model))
        self.eps = eps
    
    def forward(self, x):
        rms = torch.sqrt(x.pow(2).mean(-1, keepdim=True) + self.eps)
        return self.weight * x / rms
```

---

## Summary Table

| Concept | Key Idea |
|---|---|
| LayerNorm | Normalize across features per sample |
| BatchNorm | Normalize across batch per feature |
| InstanceNorm | Normalize across spatial per sample per channel |
| GroupNorm | Normalize across groups of channels per sample |
| LayerNorm formula | y = γ · (x - μ_features) / σ_features + β |
| Batch independent | LayerNorm, GroupNorm, InstanceNorm |
| Best for Transformers | **LayerNorm** |
| Best for CNNs (large B) | **BatchNorm** |
| Best for CNNs (small B) | **GroupNorm** |
| Best for Style Transfer | **InstanceNorm** |
| PyTorch | `nn.LayerNorm(d)`, `nn.GroupNorm(G, C)`, `nn.InstanceNorm2d(C)` |

---

## Revision Questions

1. **Explain the key difference between BatchNorm and LayerNorm** in terms of which dimensions statistics are computed over. Draw the tensor diagrams.

2. **Why is LayerNorm preferred for Transformers and RNNs?** What properties of these architectures make BatchNorm unsuitable?

3. **Show that GroupNorm with G=1 is equivalent to LayerNorm** and GroupNorm with G=C is equivalent to InstanceNorm.

4. **Implement LayerNorm from scratch** and verify it matches `nn.LayerNorm` output.

5. **How does batch size affect each normalization method?** Which methods are robust to batch size = 1?

6. **Compare Pre-Norm and Post-Norm Transformer architectures.** Why has Pre-Norm become more common?

---

## Navigation

| Previous | Up | Next |
|---|---|---|
| [← Batch Normalization](./05-batch-normalization.md) | [Unit 7: Regularization](./README.md) | [Weight Decay →](./07-weight-decay.md) |
