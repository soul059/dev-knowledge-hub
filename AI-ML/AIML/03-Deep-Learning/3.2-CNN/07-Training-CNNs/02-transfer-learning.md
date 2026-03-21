# Transfer Learning — Leveraging Pretrained CNNs

> **Unit 7 · Training CNNs · Topic 2/5**

---

## Table of Contents

1. [Overview & Motivation](#1-overview--motivation)
2. [Feature Extraction (Freeze Backbone)](#2-feature-extraction-freeze-backbone)
3. [Fine-Tuning (Unfreeze & Retrain)](#3-fine-tuning-unfreeze--retrain)
4. [When to Transfer — Domain Similarity](#4-when-to-transfer--domain-similarity)
5. [Pretrained Models & Weights](#5-pretrained-models--weights)
6. [torchvision.models & timm Library](#6-torchvisionmodels--timm-library)
7. [Practical Transfer Learning Pipeline](#7-practical-transfer-learning-pipeline)
8. [Worked Examples](#8-worked-examples)
9. [Complete Code Example](#9-complete-code-example)
10. [Summary Table](#10-summary-table)
11. [Revision Questions](#11-revision-questions)
12. [Navigation](#12-navigation)

---

## 1. Overview & Motivation

**Transfer learning** reuses a model trained on a large dataset (e.g., ImageNet) for a new task with limited data. The learned features (edges, textures, shapes) are largely universal.

```
Transfer Learning Concept:

Source Task (ImageNet, 1.2M images, 1000 classes)
┌────────────────────────────────────────────────────┐
│  Conv1 → Conv2 → ... → Conv_N → FC → 1000 classes │
│   ↓       ↓              ↓                          │
│ edges  textures        objects   ← learned features │
└────────────────────────────────────────────────────┘
        │  Transfer these weights!
        ▼
Target Task (Your dataset, maybe 500-5000 images, 10 classes)
┌────────────────────────────────────────────────────┐
│  Conv1 → Conv2 → ... → Conv_N → NEW FC → 10 cls  │
│  ^^^^^^^^^^^^^^^^^^^^^^^^^^^^    ^^^^^             │
│  pretrained (frozen or fine-tuned) new head        │
└────────────────────────────────────────────────────┘

Why it works:
  • Early layers learn UNIVERSAL features (edges, colors, textures)
  • Mid layers learn GENERIC patterns (eyes, wheels, fur)
  • Late layers learn TASK-SPECIFIC features (dog breeds, car types)
  • Only task-specific layers need significant retraining
```

### Transfer Learning Scenarios

```
┌─────────────────────────────────────────────────────────────┐
│                     Target Dataset                           │
│              Small (<10K)           Large (>100K)            │
├────────────┬──────────────────┬───────────────────────────┤
│ Similar    │ Feature extract  │ Fine-tune entire model    │
│ domain     │ (freeze backbone │ (standard fine-tuning,    │
│ (natural   │  train head only)│  larger LR for new head)  │
│ images)    │                  │                           │
├────────────┼──────────────────┼───────────────────────────┤
│ Different  │ Feature extract  │ Fine-tune from earlier    │
│ domain     │ from earlier     │ layers (more layers to    │
│ (medical,  │ layers, or train │ retrain since features    │
│ satellite) │ from scratch     │ are less transferable)    │
└────────────┴──────────────────┴───────────────────────────┘
```

---

## 2. Feature Extraction (Freeze Backbone)

Use the pretrained CNN as a fixed feature extractor:

```
Feature Extraction Pipeline:

Step 1: Load pretrained model
Step 2: FREEZE all convolutional layers (no gradient updates)
Step 3: REPLACE classification head with new one
Step 4: Train ONLY the new head

Pretrained ResNet-50:
┌───────────────────────────────────────────────────────┐
│  Conv1 │ Layer1 │ Layer2 │ Layer3 │ Layer4 │ FC(1000)│
│  ──────────────────────────────────────────  ────────│
│  FROZEN (requires_grad = False)              REPLACED│
│                                              NEW FC  │
│                                              FC(10)  │
└───────────────────────────────────────────────────────┘

Advantages:
  ✓ Very fast training (few parameters to update)
  ✓ Works with very small datasets (<1000 images)
  ✓ No risk of destroying learned features
  ✓ Can precompute features once, train head repeatedly

Disadvantages:
  ✗ Features may not be optimal for target domain
  ✗ Cannot adapt low-level features to new domain
  ✗ Accuracy ceiling compared to fine-tuning
```

### Implementation

```python
import torch
import torch.nn as nn
from torchvision.models import resnet50, ResNet50_Weights

# 1. Load pretrained model
model = resnet50(weights=ResNet50_Weights.IMAGENET1K_V2)

# 2. Freeze ALL parameters
for param in model.parameters():
    param.requires_grad = False

# 3. Replace classification head
num_features = model.fc.in_features  # 2048 for ResNet-50
model.fc = nn.Sequential(
    nn.Linear(num_features, 256),
    nn.ReLU(),
    nn.Dropout(0.3),
    nn.Linear(256, 10),  # 10 classes
)
# New head parameters have requires_grad=True by default

# 4. Only train head parameters
optimizer = torch.optim.Adam(model.fc.parameters(), lr=1e-3)

# Check trainable params
trainable = sum(p.numel() for p in model.parameters() if p.requires_grad)
total = sum(p.numel() for p in model.parameters())
print(f"Trainable: {trainable:,} / {total:,} ({100*trainable/total:.1f}%)")
# Trainable: ~660K / ~23.5M (~2.8%)
```

---

## 3. Fine-Tuning (Unfreeze & Retrain)

Allow the pretrained weights to be updated (partially or fully):

```
Fine-Tuning Approaches:

Approach A: Full Fine-Tuning
  Unfreeze ALL layers, train with small learning rate
  ┌──────────────────────────────────────────────────┐
  │  Conv1 │ Layer1 │ Layer2 │ Layer3 │ Layer4 │ FC │
  │  TRAINABLE (lr=1e-4) ───────────────── (lr=1e-3)│
  └──────────────────────────────────────────────────┘

Approach B: Partial Fine-Tuning
  Freeze early layers, fine-tune later layers + head
  ┌──────────────────────────────────────────────────┐
  │  Conv1 │ Layer1 │ Layer2 │ Layer3 │ Layer4 │ FC │
  │  FROZEN ─────────────── │ TRAINABLE ────────────│
  └──────────────────────────────────────────────────┘

Approach C: Gradual Unfreezing (recommended — see next topic)
  Phase 1: Train head only
  Phase 2: Unfreeze last layer group
  Phase 3: Unfreeze more layers
  ┌──────────────────────────────────────────────────┐
  │ Phase 1: Only FC trainable                       │
  │ Phase 2: Layer4 + FC trainable                   │
  │ Phase 3: Layer3 + Layer4 + FC trainable          │
  │ Phase 4: All trainable (discriminative LR)       │
  └──────────────────────────────────────────────────┘
```

### Key Principles

```
1. SMALL LEARNING RATE for pretrained layers:
   Pretrained weights are already good — large updates destroy them
   Typical: 10-100× smaller than for new head
   
2. LARGER LEARNING RATE for new head:
   Random initialization needs bigger updates
   Typical: lr=1e-3 for head, lr=1e-5 for backbone

3. WARM UP before unfreezing:
   Train head for a few epochs first
   Then unfreeze backbone layers
   This prevents large gradients from destroying pretrained features

4. BATCH NORMALIZATION:
   Keep BN layers in eval mode OR freeze their statistics
   Training BN with small batches produces poor statistics
```

---

## 4. When to Transfer — Domain Similarity

```
Feature Transferability Across Domains:

Layer 1 features (edges, colors): 
  ImageNet ─────► Medical ✓  (edges are edges everywhere)
  ImageNet ─────► Satellite ✓
  ImageNet ─────► Art ✓

Layer 3 features (parts, textures):
  ImageNet ─────► Medical ~  (some textures transfer, some don't)
  ImageNet ─────► Satellite ~
  
Layer 4+ features (object parts):
  ImageNet ─────► Similar natural images ✓
  ImageNet ─────► Medical ✗  (organs ≠ ImageNet objects)
  ImageNet ─────► Satellite ✗  (aerial patterns ≠ ground-level)


Domain Similarity Guide:
┌────────────────────────────────────────────────────────────┐
│                                                            │
│  Source: ImageNet (natural photos)                         │
│                                                            │
│  HIGH SIMILARITY (transfer all layers):                    │
│    • Pet breeds, food recognition, plant species           │
│    • Indoor/outdoor scene classification                   │
│    • Fine-grained natural image classification             │
│                                                            │
│  MEDIUM SIMILARITY (transfer early/mid layers):            │
│    • Artistic images, drawings, sketches                   │
│    • Remote sensing (aerial, satellite)                    │
│    • Document images                                       │
│                                                            │
│  LOW SIMILARITY (transfer only early layers or scratch):   │
│    • Medical imaging (X-ray, MRI, CT)                      │
│    • Microscopy images                                     │
│    • Spectrograms, synthetic images                        │
│                                                            │
│  EVEN medical imaging benefits from ImageNet pretraining!  │
│  (Gabor-like filters and edge detectors are universal)     │
└────────────────────────────────────────────────────────────┘
```

---

## 5. Pretrained Models & Weights

### ImageNet Pretrained Models

```
Popular Pretrained Models:
┌──────────────────┬──────────┬──────────┬──────────┬──────────────┐
│ Model            │ Params(M)│ GFLOPs   │ Top-1(%) │ Best For     │
├──────────────────┼──────────┼──────────┼──────────┼──────────────┤
│ ResNet-18        │  11.7    │  1.8     │  69.8    │ Quick exper. │
│ ResNet-50        │  25.6    │  4.1     │  80.9    │ General      │
│ ResNet-101       │  44.5    │  7.8     │  81.9    │ Higher acc.  │
│ EfficientNet-B0  │   5.3    │  0.4     │  77.7    │ Efficient    │
│ EfficientNet-B4  │  19.0    │  4.2     │  83.4    │ Best tradeoff│
│ ConvNeXt-Tiny    │  28.6    │  4.5     │  82.1    │ Modern CNN   │
│ ViT-B/16         │  86.6    │ 17.6     │  84.5    │ Large data   │
│ Swin-T           │  28.3    │  4.5     │  81.3    │ Dense predict│
└──────────────────┴──────────┴──────────┴──────────┴──────────────┘

How to choose:
  • Small dataset, speed needed → ResNet-18/50
  • Accuracy matters most → EfficientNet-B4 or ConvNeXt
  • Mobile deployment → MobileNetV3, EfficientNet-B0
  • Detection/segmentation → ResNet-50, Swin-T
  • Maximum accuracy (enough data) → ViT-L
```

### Pretraining Paradigms

```
Beyond ImageNet-supervised pretraining:

1. ImageNet-1K supervised (standard)
   • 1.2M images, 1000 classes
   • Most common baseline

2. ImageNet-21K supervised
   • 14M images, 21K classes
   • Better for large models (ViT-L)
   
3. Self-supervised (DINO, MAE, MoCo)
   • No labels needed
   • Often better than supervised for transfer
   • DINO: self-distillation with no labels
   • MAE: mask patches, reconstruct them

4. CLIP (contrastive language-image)
   • 400M image-text pairs
   • Amazing zero-shot transfer
   • Best for multi-modal tasks

Self-supervised ≥ supervised for transfer learning!
(Especially with limited target data)
```

---

## 6. torchvision.models & timm Library

### torchvision.models

```python
from torchvision import models

# Modern API with weights enum
resnet50 = models.resnet50(weights=models.ResNet50_Weights.IMAGENET1K_V2)
efficientnet = models.efficientnet_b0(
    weights=models.EfficientNet_B0_Weights.IMAGENET1K_V1)
vit = models.vit_b_16(weights=models.ViT_B_16_Weights.IMAGENET1K_V1)

# List all available models
print(models.list_models())

# Get transforms associated with weights
preprocess = models.ResNet50_Weights.IMAGENET1K_V2.transforms()
```

### timm Library (PyTorch Image Models)

```python
import timm

# List available models (800+ models!)
print(timm.list_models('*efficientnet*'))
print(timm.list_models('*resnet*'))

# Create model with pretrained weights
model = timm.create_model('efficientnet_b3', pretrained=True, num_classes=10)

# Automatic configuration
data_config = timm.data.resolve_data_config(model.pretrained_cfg)
print(data_config)
# {'input_size': (3, 300, 300), 'mean': (0.485, ...), 'std': (0.229, ...)}

# Get matching transforms
transform = timm.data.create_transform(**data_config, is_training=True)

# Access model components
print(model.default_cfg)  # architecture info
print(model.get_classifier())  # classifier layer
model.reset_classifier(num_classes=5)  # change number of classes

# Feature extraction (get intermediate features)
model = timm.create_model('resnet50', pretrained=True, features_only=True)
features = model(torch.randn(1, 3, 224, 224))
for f in features:
    print(f.shape)
# torch.Size([1, 64, 112, 112])   ← stem
# torch.Size([1, 256, 56, 56])    ← layer1
# torch.Size([1, 512, 28, 28])    ← layer2
# torch.Size([1, 1024, 14, 14])   ← layer3
# torch.Size([1, 2048, 7, 7])     ← layer4
```

---

## 7. Practical Transfer Learning Pipeline

```
Step-by-Step Pipeline:

1. CHOOSE BASE MODEL
   └─ Consider: dataset size, similarity, compute budget, deployment target

2. PREPARE DATA
   └─ Use transforms matching pretrained model (resolution, normalization)

3. MODIFY ARCHITECTURE
   └─ Replace classifier head for your number of classes
   └─ Optionally add dropout, extra layers

4. FREEZE BACKBONE
   └─ Start with all backbone layers frozen

5. TRAIN HEAD (5-20 epochs)
   └─ Higher LR (1e-3), only head parameters
   └─ Validate to get baseline accuracy

6. UNFREEZE BACKBONE (optional, if accuracy insufficient)
   └─ Gradually unfreeze from last layer group
   └─ Use discriminative LR (lower for early layers)

7. FINE-TUNE (10-50 epochs)
   └─ Lower LR (1e-4 to 1e-5 for backbone)
   └─ Cosine annealing or plateau-based scheduler

8. EVALUATE
   └─ Test on held-out data
   └─ Optionally apply TTA
```

---

## 8. Worked Examples

### Example 1: Dataset Size Decision

```
Scenario: 500 images of 5 flower species, pretrained ResNet-50

Decision tree:
  1. Dataset similar to ImageNet? YES (flowers ∈ natural images)
  2. Dataset size? SMALL (500 images = 100 per class)
  
  → Strategy: Feature extraction (freeze backbone, train head only)
  
  Expected accuracy: ~90-95% (leveraging ImageNet features)
  Training from scratch would give: ~50-60% (too little data)
  
  If accuracy < 85%:
  → Try fine-tuning last 1-2 layer groups with very small LR
  → Add stronger augmentation (RandAugment, Mixup)
```

### Example 2: Feature Extraction Timing

```
ResNet-50 feature extraction for 10,000 images:

Option A: Forward pass each epoch
  Per epoch: 10,000 × (forward pass time) = ~2 minutes
  100 epochs = ~200 minutes

Option B: Precompute features (since backbone is frozen!)
  Step 1: Forward 10,000 images through frozen backbone ONCE
          Save 2048-dim feature vectors to disk (~80MB)
          Time: ~2 minutes
  Step 2: Train linear classifier on saved features
          100 epochs × (linear forward+backward) = ~5 minutes
  Total: ~7 minutes (28× faster!)
```

---

## 9. Complete Code Example

```python
import torch
import torch.nn as nn
import torch.optim as optim
from torch.utils.data import DataLoader
from torchvision import datasets, transforms, models
import timm

# ============================================================
# STEP 1: Data Preparation
# ============================================================
# Use transforms matching the pretrained model
train_transform = transforms.Compose([
    transforms.RandomResizedCrop(224),
    transforms.RandomHorizontalFlip(),
    transforms.RandAugment(num_ops=2, magnitude=9),
    transforms.ToTensor(),
    transforms.Normalize([0.485, 0.456, 0.406],
                         [0.229, 0.224, 0.225]),
])

val_transform = transforms.Compose([
    transforms.Resize(256),
    transforms.CenterCrop(224),
    transforms.ToTensor(),
    transforms.Normalize([0.485, 0.456, 0.406],
                         [0.229, 0.224, 0.225]),
])

train_dataset = datasets.ImageFolder('data/train', transform=train_transform)
val_dataset = datasets.ImageFolder('data/val', transform=val_transform)

train_loader = DataLoader(train_dataset, batch_size=32, shuffle=True,
                          num_workers=4, pin_memory=True)
val_loader = DataLoader(val_dataset, batch_size=64, shuffle=False,
                        num_workers=4, pin_memory=True)

num_classes = len(train_dataset.classes)

# ============================================================
# STEP 2: Load Pretrained Model & Modify
# ============================================================
model = timm.create_model('efficientnet_b3', pretrained=True,
                           num_classes=num_classes)

# ============================================================
# STEP 3: Phase 1 — Feature Extraction (Freeze backbone)
# ============================================================
for name, param in model.named_parameters():
    if 'classifier' not in name:
        param.requires_grad = False

optimizer = optim.AdamW(
    filter(lambda p: p.requires_grad, model.parameters()),
    lr=1e-3, weight_decay=0.01
)
criterion = nn.CrossEntropyLoss(label_smoothing=0.1)

device = torch.device('cuda' if torch.cuda.is_available() else 'cpu')
model.to(device)

# Train head for 10 epochs
for epoch in range(10):
    model.train()
    for images, labels in train_loader:
        images, labels = images.to(device), labels.to(device)
        optimizer.zero_grad()
        outputs = model(images)
        loss = criterion(outputs, labels)
        loss.backward()
        optimizer.step()

# ============================================================
# STEP 4: Phase 2 — Fine-Tuning (Unfreeze backbone)
# ============================================================
for param in model.parameters():
    param.requires_grad = True

# Discriminative learning rates
backbone_params = [p for n, p in model.named_parameters()
                   if 'classifier' not in n]
head_params = [p for n, p in model.named_parameters()
               if 'classifier' in n]

optimizer = optim.AdamW([
    {'params': backbone_params, 'lr': 1e-5},   # 100× smaller
    {'params': head_params, 'lr': 1e-3},
], weight_decay=0.01)

scheduler = optim.lr_scheduler.CosineAnnealingLR(optimizer, T_max=40)

# Fine-tune for 40 epochs
best_acc = 0.0
for epoch in range(40):
    # Training
    model.train()
    for images, labels in train_loader:
        images, labels = images.to(device), labels.to(device)
        optimizer.zero_grad()
        outputs = model(images)
        loss = criterion(outputs, labels)
        loss.backward()
        optimizer.step()

    # Validation
    model.eval()
    correct, total = 0, 0
    with torch.no_grad():
        for images, labels in val_loader:
            images, labels = images.to(device), labels.to(device)
            outputs = model(images)
            _, predicted = outputs.max(1)
            total += labels.size(0)
            correct += predicted.eq(labels).sum().item()

    acc = 100. * correct / total
    if acc > best_acc:
        best_acc = acc
        torch.save(model.state_dict(), 'best_model.pth')
    print(f"Epoch {epoch}: Val Acc = {acc:.2f}% (Best: {best_acc:.2f}%)")

    scheduler.step()
```

---

## 10. Summary Table

| Concept | Key Idea |
|---|---|
| **Transfer learning** | Reuse features from model trained on large dataset |
| **Feature extraction** | Freeze backbone, train new classification head only |
| **Fine-tuning** | Unfreeze backbone layers and retrain with small LR |
| **Domain similarity** | Higher similarity → more layers can be transferred |
| **Small target data** | Freeze more layers, simpler head, stronger augmentation |
| **Large target data** | Fine-tune all layers with discriminative LR |
| **Discriminative LR** | Lower LR for early layers, higher for later layers/head |
| **ImageNet pretraining** | Universal baseline; even helps for very different domains |
| **Self-supervised pretrain** | DINO, MAE — often better than supervised for transfer |
| **timm library** | 800+ pretrained models, easy to use API |

---

## 11. Revision Questions

1. **Explain the difference between feature extraction and fine-tuning.** When would you use each approach? What determines the choice?

2. **Why do early CNN layers transfer better** than later layers across domains? What do early, mid, and late layer features typically represent?

3. **You have 200 chest X-ray images** to classify into 3 categories. Describe your complete transfer learning strategy, including model choice, which layers to freeze/unfreeze, learning rates, and augmentation.

4. **What is discriminative learning rate?** Why should pretrained layers use a smaller learning rate than the new head? What's a typical ratio?

5. **Compare torchvision.models and the timm library.** When would you prefer one over the other? How do you get the correct preprocessing transforms for a timm model?

6. **Is ImageNet pretraining useful for medical imaging?** Explain why, given the large domain gap. What evidence supports this?

---

## 12. Navigation

| Previous | Up | Next |
|---|---|---|
| [← Data Augmentation](./01-data-augmentation-images.md) | [Unit 7 Index](../07-Training-CNNs/) | [Fine-Tuning Strategies →](./03-fine-tuning-strategies.md) |

---

> **References:**  
> - Yosinski, J. et al. (2014). *How transferable are features in deep neural networks?* NeurIPS.  
> - He, K. et al. (2019). *Rethinking ImageNet Pre-training.* ICCV.  
> - Wightman, R. (2019). *PyTorch Image Models (timm).* GitHub.  
> - Caron, M. et al. (2021). *Emerging Properties in Self-Supervised Vision Transformers (DINO).* ICCV.
