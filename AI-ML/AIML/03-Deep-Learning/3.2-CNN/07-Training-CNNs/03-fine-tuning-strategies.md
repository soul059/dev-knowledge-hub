# Fine-Tuning Strategies — Gradual Unfreezing & Discriminative Learning Rates

> **Unit 7 · Training CNNs · Topic 3/5**

---

## Table of Contents

1. [Overview](#1-overview)
2. [Gradual Unfreezing](#2-gradual-unfreezing)
3. [Discriminative Learning Rates](#3-discriminative-learning-rates)
4. [Freezing Batch Normalization](#4-freezing-batch-normalization)
5. [Fine-Tuning Schedule](#5-fine-tuning-schedule)
6. [Practical Tips & Common Mistakes](#6-practical-tips--common-mistakes)
7. [Worked Examples](#7-worked-examples)
8. [PyTorch Implementation](#8-pytorch-implementation)
9. [Summary Table](#9-summary-table)
10. [Revision Questions](#10-revision-questions)
11. [Navigation](#11-navigation)

---

## 1. Overview

Fine-tuning a pretrained CNN requires careful strategy. Naively unfreezing all layers and training with a single learning rate often **destroys the pretrained features** (catastrophic forgetting).

```
The Problem:

Random head initialization produces LARGE gradients
  ↓
These gradients propagate back through the network
  ↓
Pretrained weights receive LARGE, NOISY updates
  ↓
Carefully learned features are DESTROYED
  ↓
Model must re-learn everything from scratch (defeats transfer learning!)

Solution: Control WHICH layers train and HOW FAST they train
```

---

## 2. Gradual Unfreezing

### The Strategy

```
Phase 1: Train HEAD only (3-5 epochs)
────────────────────────────────────────
┌──────┬──────┬──────┬──────┬──────┬──────┐
│Layer1│Layer2│Layer3│Layer4│Layer5│ HEAD │
│FROZEN│FROZEN│FROZEN│FROZEN│FROZEN│TRAIN │
│  ❄   │  ❄   │  ❄   │  ❄   │  ❄   │  🔥  │
└──────┴──────┴──────┴──────┴──────┴──────┘
  Purpose: Let the head learn reasonable features
           before sending gradients to backbone

Phase 2: Unfreeze LAST layer group (5-10 epochs)
───────────────────────────────────────────────
┌──────┬──────┬──────┬──────┬──────┬──────┐
│Layer1│Layer2│Layer3│Layer4│Layer5│ HEAD │
│FROZEN│FROZEN│FROZEN│FROZEN│TRAIN │TRAIN │
│  ❄   │  ❄   │  ❄   │  ❄   │  🔥  │  🔥  │
└──────┴──────┴──────┴──────┴──────┴──────┘
  Purpose: Adapt high-level features to new task

Phase 3: Unfreeze MORE layers (10-20 epochs)
──────────────────────────────────────────────
┌──────┬──────┬──────┬──────┬──────┬──────┐
│Layer1│Layer2│Layer3│Layer4│Layer5│ HEAD │
│FROZEN│FROZEN│TRAIN │TRAIN │TRAIN │TRAIN │
│  ❄   │  ❄   │  🔥  │  🔥  │  🔥  │  🔥  │
└──────┴──────┴──────┴──────┴──────┴──────┘
  Purpose: Adapt mid-level features

Phase 4: Unfreeze ALL (optional, 10-30 epochs)
──────────────────────────────────────────────
┌──────┬──────┬──────┬──────┬──────┬──────┐
│Layer1│Layer2│Layer3│Layer4│Layer5│ HEAD │
│TRAIN │TRAIN │TRAIN │TRAIN │TRAIN │TRAIN │
│  🔥  │  🔥  │  🔥  │  🔥  │  🔥  │  🔥  │
└──────┴──────┴──────┴──────┴──────┴──────┘
  Purpose: Fine-tune everything with very small LR
  Note: Use discriminative LR (lower for early layers)
```

### Why Gradual Unfreezing Works

```
Gradient stability analysis:

Phase 1 (head only):
  Head learns to produce meaningful gradients
  Backbone receives NO gradient → pristine features preserved
  
Phase 2 (+ last layers):
  Now the head's gradients are REASONABLE (not random noise)
  Last layers receive small, meaningful updates
  Early layers still pristine
  
Phase 3 (+ more layers):
  All unfrozen layers have been partially adapted
  Gradient signal is stable throughout the network
  
Result:
  ┌──────────────────────────────────────────────┐
  │ Feature quality over training:               │
  │                                              │
  │  Quality │    ╱─── Gradual unfreezing        │
  │          │   ╱                               │
  │          │  ╱  ╱── Full fine-tune (risky)    │
  │          │ ╱  ╱                              │
  │          │╱  ╱                               │
  │       ───┼──╱──────────── Frozen (ceiling)   │
  │          │╲                                  │
  │          │ ╲── Naive unfreezing (crash!)     │
  │          └──────────────────────────         │
  │            Epochs                            │
  └──────────────────────────────────────────────┘
```

---

## 3. Discriminative Learning Rates

Assign **different learning rates** to different layer groups, with earlier layers getting smaller LRs.

```
Rationale:
  • Early layers: learn UNIVERSAL features (edges, textures)
    → Small updates needed (features are already good)
    → LR: very small (1e-6 to 1e-5)
    
  • Middle layers: learn DOMAIN-SPECIFIC features
    → Moderate updates needed
    → LR: small (1e-5 to 1e-4)
    
  • Late layers: learn TASK-SPECIFIC features
    → Larger updates needed (features may not transfer well)
    → LR: moderate (1e-4)
    
  • New head: RANDOMLY initialized
    → Largest updates needed
    → LR: normal (1e-3)

Common schemes:

Scheme 1: Layer-group-wise LR (practical)
  ┌────────────┬──────────┐
  │ Layer Group│ LR       │
  ├────────────┼──────────┤
  │ Stem + L1  │ 1e-6     │
  │ Layer 2    │ 1e-5     │
  │ Layer 3    │ 1e-4     │
  │ Layer 4    │ 5e-4     │
  │ Head       │ 1e-3     │
  └────────────┴──────────┘

Scheme 2: Geometric decay (fastai-style)
  base_lr = 1e-3,  factor = 0.1
  Head:    1e-3
  Layer 4: 1e-4   (base_lr × 0.1)
  Layer 3: 1e-5   (base_lr × 0.01)
  Layer 2: 1e-6   (base_lr × 0.001)
  Layer 1: 1e-7   (base_lr × 0.0001)

Scheme 3: LLRD — Layer-wise Learning Rate Decay (for ViT)
  lr_layer_i = base_lr × decay^(L - i)
  
  For ViT with 12 layers, base_lr=1e-4, decay=0.65:
    Layer 12 (top):    1e-4
    Layer 11:          6.5e-5
    Layer 10:          4.2e-5
    ...
    Layer 1 (bottom):  1.3e-6
    Embedding:         8.7e-7
```

---

## 4. Freezing Batch Normalization

### The Problem

```
BatchNorm during training computes running statistics:
  μ_running = (1-momentum) × μ_running + momentum × μ_batch
  σ_running = (1-momentum) × σ_running + momentum × σ_batch

Problem with fine-tuning:
  • Pretrained BN statistics = ImageNet mean/std
  • Your mini-batch statistics ≠ ImageNet statistics
  • Small batches → noisy batch statistics
  
  If BN is in train mode:
    Running stats quickly overwritten by new (noisy) stats
    → Model performance degrades immediately!

Solution: Keep BN in eval() mode during fine-tuning
```

### How to Freeze BN

```python
def freeze_bn(model):
    """Freeze all BatchNorm layers to use pretrained statistics."""
    for module in model.modules():
        if isinstance(module, (nn.BatchNorm2d, nn.BatchNorm1d)):
            module.eval()  # use running mean/var, not batch stats
            # Also freeze the learnable parameters
            for param in module.parameters():
                param.requires_grad = False

# Usage during training:
model.train()
freeze_bn(model)  # BN stays in eval mode even when model is in train mode

# Alternative: Set model to train but override BN
class FrozenBN(nn.Module):
    def train(self, mode=True):
        super().train(mode)
        # Freeze BN layers
        for m in self.modules():
            if isinstance(m, nn.BatchNorm2d):
                m.eval()
```

### When to Freeze BN

```
FREEZE BN when:
  ✓ Batch size < 16 (noisy batch statistics)
  ✓ Fine-tuning a few layers (backbone stats should remain)
  ✓ Very different target domain (BN stats would diverge)
  ✓ Early phases of gradual unfreezing

UNFREEZE BN when:
  ✓ Batch size ≥ 32 (reliable batch statistics)
  ✓ Training all layers from late phases
  ✓ Similar domain to pretraining
  ✓ Long fine-tuning (enough data to recompute good stats)
```

---

## 5. Fine-Tuning Schedule

### Complete Schedule

```
Training Schedule for Transfer Learning:

Epoch    LR (head)    LR (backbone)    Layers Unfrozen     BN Mode
─────    ─────────    ──────────────   ────────────────     ────────
 1-5      1e-3         0 (frozen)      Head only            Eval
 6-10     1e-3         1e-5            + Layer4/5            Eval
11-30     5e-4         5e-6            + Layer3              Train*
31-50     1e-4         1e-6            All layers            Train*

* BN in train mode only if batch_size ≥ 32

Learning Rate Visualization:

LR │
   │ ┌──── Head LR
   │ │                    
1e-3│ ├─────────┐
   │ │         │ 
5e-4│ │         ├────────────────┐
   │ │         │                │
1e-4│ │         │                ├───────────┐
   │ │         │                │           │
1e-5│ │    ┌────┤  Backbone LR  │           │
   │ │    │    ├────────────────┤           │
1e-6│ │    │    │                ├───────────┤
   │ │    │    │                │           │
   └─┴────┴────┴────────────────┴───────────┴── Epoch
    1    5   10                30          50
```

### Cosine Annealing During Fine-Tuning

```
Combine gradual unfreezing with cosine annealing:

LR │
   │  ╱╲
   │ ╱  ╲
   │╱    ╲         ╱╲
   │      ╲       ╱  ╲
   │       ╲     ╱    ╲        ╱╲
   │        ╲   ╱      ╲      ╱  ╲
   │         ╲ ╱        ╲    ╱    ╲___
   │          V          ╲  ╱
   │                      ╲╱
   └────────────────────────────────────── Epoch
   Phase 1     Phase 2       Phase 3

Each phase: warmup to peak LR → cosine decay to min LR
New layers get higher peak LR than already-training layers
```

---

## 6. Practical Tips & Common Mistakes

### Tips

```
1. ALWAYS validate at each phase transition
   → If accuracy drops when unfreezing, reduce LR or go back

2. USE WEIGHT DECAY (but careful with BN)
   → AdamW with wd=0.01 for backbone
   → No weight decay on bias terms and BN parameters

3. LEARNING RATE FINDER
   → Run LR range test at each phase
   → Set max LR to ~10× below the divergence point

4. EARLY STOPPING
   → Monitor validation loss/accuracy
   → Save best checkpoint, not last

5. AUGMENTATION MATTERS
   → More aggressive augmentation for fine-tuning
   → RandAugment + Mixup + CutMix for best results

6. LABEL SMOOTHING
   → CrossEntropyLoss(label_smoothing=0.1)
   → Prevents overconfidence, acts as regularizer
```

### Common Mistakes

```
❌ Using same LR for all layers
   → Pretrained layers receive too-large updates → features destroyed
   
❌ Not freezing BN layers
   → Running statistics corrupted by small batch size
   
❌ Unfreezing everything at once
   → Random head sends noise through entire network
   
❌ Too high LR for fine-tuning
   → Should be 10-100× smaller than training from scratch
   
❌ Not using pretrained preprocessing
   → Must use SAME normalization (mean, std) as pretraining
   
❌ Training too long
   → Small datasets → overfitting quickly
   → Use early stopping!
   
❌ Ignoring data augmentation
   → Essential for preventing overfitting on small datasets
```

---

## 7. Worked Examples

### Example: Gradual Unfreezing Schedule

```
Task: Fine-tune ResNet-50 on 2000 images, 5 classes
Model: ResNet-50 (pretrained ImageNet, 23.5M params)

Layer groups in ResNet-50:
  Group 0: conv1 + bn1 (9.4K params)
  Group 1: layer1 (0.21M params)
  Group 2: layer2 (1.22M params)
  Group 3: layer3 (7.10M params)
  Group 4: layer4 (14.96M params)
  Head:    fc (10K params for 5 classes)

Phase 1 (epochs 1-5): Head only
  Trainable params: 10K (0.04% of total)
  LR: 1e-3
  Expected: 75-85% val accuracy

Phase 2 (epochs 6-15): + Group 4
  Trainable params: 15.0M (63.7%)
  LR: head=1e-3, group4=1e-5
  Expected: 85-92% val accuracy

Phase 3 (epochs 16-30): + Group 3
  Trainable params: 22.1M (93.9%)
  LR: head=5e-4, group4=5e-6, group3=1e-6
  Expected: 90-95% val accuracy

Phase 4 (epochs 31-50): All unfrozen
  Trainable params: 23.5M (100%)
  LR: head=1e-4, group4=1e-6, group3=5e-7, others=1e-7
  Expected: 92-96% val accuracy (with augmentation)
```

---

## 8. PyTorch Implementation

### Complete Gradual Unfreezing with Discriminative LR

```python
import torch
import torch.nn as nn
from torch.optim import AdamW
from torch.optim.lr_scheduler import CosineAnnealingWarmRestarts
from torchvision.models import resnet50, ResNet50_Weights

# ============================================================
# Model Setup
# ============================================================
model = resnet50(weights=ResNet50_Weights.IMAGENET1K_V2)
num_classes = 5

# Replace head
model.fc = nn.Sequential(
    nn.Dropout(0.3),
    nn.Linear(2048, num_classes),
)

# Define layer groups
layer_groups = {
    'stem':   [model.conv1, model.bn1],
    'layer1': [model.layer1],
    'layer2': [model.layer2],
    'layer3': [model.layer3],
    'layer4': [model.layer4],
    'head':   [model.fc],
}

def get_params(modules):
    """Get parameters from a list of modules."""
    params = []
    for m in modules:
        params.extend(list(m.parameters()))
    return params

def freeze_bn(model):
    for m in model.modules():
        if isinstance(m, nn.BatchNorm2d):
            m.eval()
            for p in m.parameters():
                p.requires_grad = False

# ============================================================
# Phase 1: Head Only
# ============================================================
# Freeze everything
for p in model.parameters():
    p.requires_grad = False
# Unfreeze head
for p in model.fc.parameters():
    p.requires_grad = True

optimizer = AdamW(model.fc.parameters(), lr=1e-3, weight_decay=0.01)
criterion = nn.CrossEntropyLoss(label_smoothing=0.1)
device = torch.device('cuda' if torch.cuda.is_available() else 'cpu')
model.to(device)

print("Phase 1: Training head only")
for epoch in range(5):
    model.train()
    freeze_bn(model)
    # ... training loop ...

# ============================================================
# Phase 2: + Layer4
# ============================================================
for p in model.layer4.parameters():
    p.requires_grad = True

optimizer = AdamW([
    {'params': get_params(layer_groups['layer4']), 'lr': 1e-5},
    {'params': get_params(layer_groups['head']),   'lr': 1e-3},
], weight_decay=0.01)

print("Phase 2: + Layer4")
for epoch in range(10):
    model.train()
    freeze_bn(model)
    # ... training loop ...

# ============================================================
# Phase 3: + Layer3
# ============================================================
for p in model.layer3.parameters():
    p.requires_grad = True

optimizer = AdamW([
    {'params': get_params(layer_groups['layer3']), 'lr': 1e-6},
    {'params': get_params(layer_groups['layer4']), 'lr': 5e-6},
    {'params': get_params(layer_groups['head']),   'lr': 5e-4},
], weight_decay=0.01)

scheduler = CosineAnnealingWarmRestarts(optimizer, T_0=5, T_mult=2)

print("Phase 3: + Layer3")
for epoch in range(15):
    model.train()
    freeze_bn(model)
    # ... training loop ...
    scheduler.step()

# ============================================================
# Phase 4: All Layers
# ============================================================
for p in model.parameters():
    p.requires_grad = True

optimizer = AdamW([
    {'params': get_params(layer_groups['stem']),   'lr': 1e-7},
    {'params': get_params(layer_groups['layer1']), 'lr': 5e-7},
    {'params': get_params(layer_groups['layer2']), 'lr': 1e-6},
    {'params': get_params(layer_groups['layer3']), 'lr': 5e-7},
    {'params': get_params(layer_groups['layer4']), 'lr': 1e-6},
    {'params': get_params(layer_groups['head']),   'lr': 1e-4},
], weight_decay=0.01)

print("Phase 4: All layers (discriminative LR)")
for epoch in range(20):
    model.train()
    # BN can be in train mode now (if batch_size >= 32)
    # ... training loop ...
```

### Layer-wise Learning Rate Decay (LLRD) for ViT

```python
import timm

model = timm.create_model('vit_base_patch16_224', pretrained=True,
                            num_classes=10)

def get_llrd_params(model, base_lr=1e-4, decay=0.65):
    """Layer-wise learning rate decay for ViT."""
    param_groups = []
    
    # Embedding layers get lowest LR
    embed_params = []
    for name, param in model.named_parameters():
        if 'patch_embed' in name or 'pos_embed' in name or 'cls_token' in name:
            embed_params.append(param)
    
    num_layers = 12  # ViT-Base has 12 transformer blocks
    
    # Each block gets progressively higher LR
    for i in range(num_layers):
        block_params = []
        for name, param in model.named_parameters():
            if f'blocks.{i}.' in name:
                block_params.append(param)
        if block_params:
            lr = base_lr * (decay ** (num_layers - i))
            param_groups.append({'params': block_params, 'lr': lr})
    
    # Embedding gets lowest LR
    param_groups.insert(0, {
        'params': embed_params,
        'lr': base_lr * (decay ** (num_layers + 1))
    })
    
    # Head gets base_lr
    head_params = [p for n, p in model.named_parameters() if 'head' in n]
    param_groups.append({'params': head_params, 'lr': base_lr})
    
    return param_groups

param_groups = get_llrd_params(model, base_lr=1e-4, decay=0.65)
optimizer = AdamW(param_groups, weight_decay=0.05)

# Print LR per group
for i, group in enumerate(param_groups):
    print(f"Group {i}: lr={group['lr']:.2e}, params={sum(p.numel() for p in group['params']):,}")
```

---

## 9. Summary Table

| Strategy | Key Idea | When to Use |
|---|---|---|
| **Gradual unfreezing** | Unfreeze layers progressively: head → last → ... → all | Always for fine-tuning |
| **Discriminative LR** | Lower LR for early layers, higher for later/head | Always for fine-tuning |
| **LLRD** | Exponential LR decay across layers | ViT fine-tuning |
| **Freeze BN** | Keep BN in eval mode during fine-tuning | Small batch size (<32) |
| **Phase training** | Separate optimizer per phase with different param groups | Structured approach |
| **Cosine annealing** | Smooth LR decay within each phase | During fine-tuning |
| **Label smoothing** | Soft targets (prevent overconfidence) | Small datasets |
| **Weight decay** | L2 regularization on weights (not bias/BN) | Always |

---

## 10. Revision Questions

1. **Explain gradual unfreezing.** Why is it better than unfreezing all layers at once? What happens to pretrained features if you skip Phase 1 (head training)?

2. **Design a discriminative learning rate schedule** for fine-tuning EfficientNet-B0 on 1000 images of 8 classes. Specify LR for each layer group.

3. **Why must BatchNorm layers be frozen** during fine-tuning with small batch sizes? What happens to the running statistics if you don't freeze them?

4. **Compare feature extraction vs gradual unfreezing vs full fine-tuning.** For a dataset with 500 images similar to ImageNet, which strategy gives the best accuracy? What about 50,000 images?

5. **Explain LLRD for Vision Transformers.** Why is exponential decay across layers used? How does it differ from group-wise discriminative LR for CNNs?

6. **You fine-tune a model and val accuracy drops after unfreezing backbone layers.** List three possible causes and their fixes.

---

## 11. Navigation

| Previous | Up | Next |
|---|---|---|
| [← Transfer Learning](./02-transfer-learning.md) | [Unit 7 Index](../07-Training-CNNs/) | [Learning Rate Policies →](./04-learning-rate-policies.md) |

---

> **References:**  
> - Howard, J. & Ruder, S. (2018). *Universal Language Model Fine-tuning for Text Classification.* ACL. (Introduced gradual unfreezing for NLP; widely adopted in CV)  
> - Clark, K. et al. (2020). *Layer-wise Learning Rate Decay.* (Common practice in ViT fine-tuning)
