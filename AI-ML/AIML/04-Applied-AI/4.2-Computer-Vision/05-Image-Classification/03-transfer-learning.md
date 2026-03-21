# Transfer Learning for Image Classification

## Overview

Transfer learning uses a model trained on a large dataset (ImageNet) as a starting point for a new task. Instead of training from scratch, you leverage learned features — edges, textures, shapes — that transfer well across visual domains. This is the standard approach when you have limited training data.

---

## Why Transfer Learning Works

```
CNN features are hierarchical and generalizable:

  Layer 1-2:  Edges, colors, textures     → Universal (transfer always)
  Layer 3-5:  Shapes, patterns, parts     → Mostly transferable
  Layer 6+:   Object-specific features     → Task-specific (may need retraining)

  ImageNet model → Your task:
  ┌──────────────────────────────────────────────┐
  │ [Edge detectors] → [Texture] → [Parts] → [Objects] → [Cat/Dog/...] │
  │  ^^^^^^^^^^^^      ^^^^^^^^^    ^^^^^^^                             │
  │  Always useful     Usually useful   May need retraining             │
  └──────────────────────────────────────────────┘
```

---

## Two Strategies

```
Strategy 1: Feature Extraction (freeze backbone)
  - Freeze all pre-trained layers
  - Replace only the final classifier head
  - Train only the new head
  - Best when: small dataset, similar domain

Strategy 2: Fine-tuning (unfreeze some/all layers)
  - Start with pre-trained weights
  - Unfreeze last few layers (or all)
  - Train with small learning rate
  - Best when: medium dataset or different domain

Decision guide:
  ┌──────────────────┬───────────────┬──────────────────┐
  │                  │ Similar domain│ Different domain  │
  ├──────────────────┼───────────────┼──────────────────┤
  │ Small dataset    │ Feature extract│ Feature extract   │
  │ (<1K images)     │ (freeze all)  │ (may struggle)    │
  ├──────────────────┼───────────────┼──────────────────┤
  │ Large dataset    │ Fine-tune     │ Fine-tune         │
  │ (>10K images)    │ (last layers) │ (more layers)     │
  └──────────────────┴───────────────┴──────────────────┘
```

---

## Implementation

```python
import torch
import torch.nn as nn
import torchvision.models as models
from torchvision import transforms, datasets
from torch.utils.data import DataLoader

# --- Strategy 1: Feature Extraction ---
model = models.resnet50(weights=models.ResNet50_Weights.DEFAULT)

# Freeze all layers
for param in model.parameters():
    param.requires_grad = False

# Replace classifier head
num_classes = 5  # your number of classes
model.fc = nn.Linear(model.fc.in_features, num_classes)
# Only model.fc parameters are trainable

optimizer = torch.optim.Adam(model.fc.parameters(), lr=1e-3)

# --- Strategy 2: Fine-tuning ---
model = models.resnet50(weights=models.ResNet50_Weights.DEFAULT)

# Freeze early layers, unfreeze later ones
for name, param in model.named_parameters():
    if "layer4" in name or "fc" in name:
        param.requires_grad = True
    else:
        param.requires_grad = False

model.fc = nn.Linear(model.fc.in_features, num_classes)

# Use smaller learning rate for pre-trained layers
optimizer = torch.optim.Adam([
    {"params": model.layer4.parameters(), "lr": 1e-4},
    {"params": model.fc.parameters(), "lr": 1e-3},
])

# Data preparation
transform = transforms.Compose([
    transforms.Resize(256),
    transforms.CenterCrop(224),
    transforms.ToTensor(),
    transforms.Normalize([0.485, 0.456, 0.406], [0.229, 0.224, 0.225]),
])

train_dataset = datasets.ImageFolder("data/train", transform=transform)
train_loader = DataLoader(train_dataset, batch_size=32, shuffle=True)

# Train
criterion = nn.CrossEntropyLoss()
for epoch in range(10):
    for images, labels in train_loader:
        outputs = model(images)
        loss = criterion(outputs, labels)
        optimizer.zero_grad()
        loss.backward()
        optimizer.step()
```

---

## Choosing a Pre-trained Model

| Model | Params | Accuracy | Speed | Best For |
|-------|:-:|:-:|:-:|---------|
| ResNet-18 | 11M | Good | Fast | Quick experiments |
| ResNet-50 | 25M | Very good | Medium | Default choice |
| EfficientNet-B0 | 5M | Very good | Fast | Mobile deployment |
| EfficientNet-B4 | 19M | Excellent | Medium | Production |
| ViT-B/16 | 86M | Excellent | Slow | Large datasets |

---

## Revision Questions

1. **Why do early CNN layers transfer well across tasks?**
2. **When should you freeze the backbone vs fine-tune it?**
3. **Why use a smaller learning rate for pre-trained layers?**
4. **How do you replace the classifier head in a ResNet?**
5. **What pre-trained model would you choose for a mobile app with 500 training images?**

---

[Previous: 02-cnn-classification.md](02-cnn-classification.md) | [Next: 04-fine-tuning.md](04-fine-tuning.md)
