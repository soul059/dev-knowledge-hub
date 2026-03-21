# 5. Transfer Learning Basics

> **Unit 10 · Building Neural Networks** — Reusing pretrained features to solve new tasks with limited data

---

## Chapter Overview

Training a deep neural network from scratch requires massive datasets and compute. But what if you only have 500 images of your custom classes? **Transfer learning** lets you leverage features learned by models trained on millions of images (like ImageNet) and adapt them to your specific task. It's the single most practical technique in modern deep learning — used in virtually every real-world computer vision, NLP, and audio project. This chapter covers when and why transfer learning works, the two main strategies (feature extraction and fine-tuning), how domain similarity affects the approach, and complete PyTorch code examples.

---

## 1. The Concept: Reuse Learned Features

```
  TRADITIONAL APPROACH:
  ┌──────────────────────────────────────────────────────────────┐
  │  Your small dataset (500 images)                            │
  │       ↓                                                      │
  │  Train model from SCRATCH (random weights)                  │
  │       ↓                                                      │
  │  Result: TERRIBLE (not enough data to learn good features)  │
  └──────────────────────────────────────────────────────────────┘
  
  TRANSFER LEARNING:
  ┌──────────────────────────────────────────────────────────────┐
  │  ImageNet (1.2 million images, 1000 classes)                │
  │       ↓                                                      │
  │  Pretrained model learns GENERAL features                   │
  │  (edges, textures, parts, objects)                          │
  │       ↓                                                      │
  │  TRANSFER these features to your task                       │
  │       ↓                                                      │
  │  Your small dataset (500 images)                            │
  │       ↓                                                      │
  │  Fine-tune only the last layers                             │
  │       ↓                                                      │
  │  Result: GREAT (leveraging features from 1.2M images!)     │
  └──────────────────────────────────────────────────────────────┘
  
  
  The pretrained model learned a HIERARCHY of features:
  
  ┌────────────┐    ┌─────────────┐    ┌──────────────┐    ┌───────────┐
  │ Layer 1-3  │    │ Layer 4-7   │    │ Layer 8-12   │    │Layer 13+  │
  │            │    │             │    │              │    │           │
  │ Edges      │ →  │ Textures    │ →  │ Object Parts │ →  │ Objects   │
  │ Gradients  │    │ Patterns    │    │ Eyes, wheels │    │ Faces,    │
  │ Colors     │    │ Corners     │    │ Windows      │    │ cars      │
  │            │    │             │    │              │    │           │
  │ UNIVERSAL  │    │ MOSTLY      │    │ SOMEWHAT     │    │ TASK-     │
  │ (always    │    │ UNIVERSAL   │    │ SPECIFIC     │    │ SPECIFIC  │
  │  useful!)  │    │             │    │              │    │           │
  └────────────┘    └─────────────┘    └──────────────┘    └───────────┘
  
  Early features are UNIVERSAL — useful for almost any vision task!
  Late features are SPECIFIC — may need retraining for your task.
```

---

## 2. When to Use Transfer Learning

```
  ┌──────────────────────────────────────────────────────────────┐
  │  USE TRANSFER LEARNING WHEN:                                 │
  │                                                              │
  │  ✓ You have LIMITED DATA (< 10K images)                    │
  │  ✓ A pretrained model exists for a SIMILAR domain          │
  │  ✓ You want to SAVE TIME (skip weeks of training)          │
  │  ✓ You need GOOD PERFORMANCE with limited compute          │
  │  ✓ Your task shares low-level features with pretrained task│
  │                                                              │
  │  DON'T USE TRANSFER LEARNING WHEN:                          │
  │                                                              │
  │  ✗ You have MASSIVE data (100M+ samples)                   │
  │  ✗ Your domain is COMPLETELY DIFFERENT (e.g., medical X-ray│
  │    from a text model)                                       │
  │  ✗ You need the SMALLEST possible model                    │
  │  ✗ Pretrained model is too large for your hardware         │
  └──────────────────────────────────────────────────────────────┘
```

---

## 3. Two Main Strategies

### Strategy 1: Feature Extraction (Freeze Everything)

```
  FEATURE EXTRACTION:
  Use pretrained model as a fixed feature extractor.
  Only train a new classification head.
  
  ┌───────────────────────────────────────────────────┐
  │  Pretrained ResNet-50                             │
  │                                                    │
  │  ┌──────────────────────────────┐ FROZEN          │
  │  │  Conv layers (learned on     │ (no gradient    │
  │  │  ImageNet — don't change!)   │  computation)   │
  │  │                              │                  │
  │  │  Layer 1 → Layer 2 → ...    │                  │
  │  │  → Layer 49                  │                  │
  │  └──────────────┬───────────────┘                  │
  │                 ↓                                   │
  │  ┌──────────────────────────────┐ TRAINABLE        │
  │  │  New FC layer               │ (learn this!)    │
  │  │  2048 → num_classes         │                  │
  │  └──────────────────────────────┘                  │
  └───────────────────────────────────────────────────┘
  
  When to use:
  • Very small dataset (< 1K images)
  • Very similar to ImageNet domain
  • Limited compute
  • Quick baseline
  
  Advantages:
  ✓ Very fast (only train 1 layer)
  ✓ Low risk of overfitting
  ✓ Works with very small datasets
  
  Disadvantages:
  ✗ Limited performance (can't adapt features)
  ✗ Only works if domains are similar
```

### Strategy 2: Fine-Tuning (Unfreeze Gradually)

```
  FINE-TUNING:
  Start with frozen pretrained model, then progressively
  unfreeze layers and train with a small learning rate.
  
  Phase 1: Train head only (same as feature extraction)
  ┌──────────────────────────────────────────────────┐
  │  [FROZEN Conv 1-49]                              │
  │  [TRAINABLE New FC]  ← train for 5-10 epochs    │
  └──────────────────────────────────────────────────┘
  
  Phase 2: Unfreeze last few layers
  ┌──────────────────────────────────────────────────┐
  │  [FROZEN Conv 1-40]                              │
  │  [TRAINABLE Conv 41-49]  ← small LR!            │
  │  [TRAINABLE FC]                                  │
  └──────────────────────────────────────────────────┘
  
  Phase 3: Unfreeze more layers (optional)
  ┌──────────────────────────────────────────────────┐
  │  [FROZEN Conv 1-20]                              │
  │  [TRAINABLE Conv 21-49]  ← very small LR!       │
  │  [TRAINABLE FC]                                  │
  └──────────────────────────────────────────────────┘
  
  When to use:
  • Medium dataset (1K-100K images)
  • Somewhat different from ImageNet domain
  • Have more compute available
  
  Advantages:
  ✓ Better performance (adapted features)
  ✓ Works with less similar domains
  
  Disadvantages:
  ✗ Risk of overfitting if dataset is small
  ✗ Slower training
  ✗ Need to tune LR carefully (too high → destroy pretrained features)
```

### Progressive Unfreezing Schedule

```
  Learning Rate during Fine-tuning:
  
  LR ↑
     │
  1e-3│────── Head LR (new layers)
     │
  1e-4│              ────── Late layers LR (1/10 of head)
     │
  1e-5│                         ────── Early layers LR (1/100 of head)
     │
     └──────┬──────────┬──────────┬──────→ Training Phase
           Phase 1     Phase 2    Phase 3
        (head only) (+ late    (+ early
                     layers)    layers)
  
  KEY RULE: Earlier layers need SMALLER learning rates!
  
  Why? Early features (edges, textures) are already very good.
  Large updates would DESTROY these useful features.
  Only fine adjustments are needed.
```

---

## 4. Domain Similarity Effect

```
  ┌──────────────────────────────────────────────────────────────┐
  │              YOUR DATASET SIZE                               │
  │              Small            Large                          │
  │         ┌─────────────┬──────────────┐                      │
  │  DOMAIN │             │              │                      │
  │ SIMILAR │  Feature    │  Fine-tune   │                      │
  │  to     │  Extraction │  all layers  │                      │
  │  source │  (freeze    │  (you have   │                      │
  │         │  everything)│  enough data)│                      │
  │         ├─────────────┼──────────────┤                      │
  │ DIFFER- │  Fine-tune  │  Fine-tune   │                      │
  │  ENT    │  carefully  │  aggressively│                      │
  │  from   │  (or train  │  or train    │                      │
  │  source │  from       │  from scratch│                      │
  │         │  scratch)   │              │                      │
  │         └─────────────┴──────────────┘                      │
  │                                                              │
  │  DOMAIN SIMILARITY EXAMPLES:                                 │
  │  ────────────────────────                                    │
  │  High: ImageNet → Dog breeds (both natural images)         │
  │  High: ImageNet → Flower classification                    │
  │  Medium: ImageNet → Medical X-ray (both images, diff style)│
  │  Medium: ImageNet → Satellite imagery                      │
  │  Low: ImageNet → Microscopy (very different features)      │
  │  None: ImageNet → Tabular data (completely different!)     │
  └──────────────────────────────────────────────────────────────┘
```

---

## 5. PyTorch Transfer Learning: Complete Example

### Feature Extraction

```python
import torch
import torch.nn as nn
import torch.optim as optim
from torch.utils.data import DataLoader
from torchvision import datasets, transforms, models

# ═══════════════════════════════════════════════
#  1. DATA PREPARATION
# ═══════════════════════════════════════════════

# ImageNet normalization (MUST match pretrained model's training!)
train_transform = transforms.Compose([
    transforms.Resize(256),
    transforms.RandomCrop(224),
    transforms.RandomHorizontalFlip(),
    transforms.ToTensor(),
    transforms.Normalize(mean=[0.485, 0.456, 0.406],   # ImageNet mean
                         std=[0.229, 0.224, 0.225]),     # ImageNet std
])

val_transform = transforms.Compose([
    transforms.Resize(256),
    transforms.CenterCrop(224),
    transforms.ToTensor(),
    transforms.Normalize(mean=[0.485, 0.456, 0.406],
                         std=[0.229, 0.224, 0.225]),
])

# Assume you have data in folders: data/train/class1/, data/train/class2/, ...
train_data = datasets.ImageFolder('data/train', transform=train_transform)
val_data = datasets.ImageFolder('data/val', transform=val_transform)

train_loader = DataLoader(train_data, batch_size=32, shuffle=True, num_workers=4)
val_loader = DataLoader(val_data, batch_size=32, num_workers=4)

num_classes = len(train_data.classes)
print(f"Classes: {train_data.classes}")
print(f"Training samples: {len(train_data)}")

# ═══════════════════════════════════════════════
#  2. LOAD PRETRAINED MODEL
# ═══════════════════════════════════════════════

device = torch.device('cuda' if torch.cuda.is_available() else 'cpu')

# Load pretrained ResNet-50
model = models.resnet50(weights=models.ResNet50_Weights.IMAGENET1K_V2)

# FREEZE all pretrained parameters
for param in model.parameters():
    param.requires_grad = False

# REPLACE final FC layer (1000 classes → your num_classes)
model.fc = nn.Sequential(
    nn.Linear(2048, 512),
    nn.ReLU(),
    nn.Dropout(0.3),
    nn.Linear(512, num_classes)
)

model = model.to(device)

# Count parameters
total = sum(p.numel() for p in model.parameters())
trainable = sum(p.numel() for p in model.parameters() if p.requires_grad)
print(f"Total params: {total:,}")
print(f"Trainable params: {trainable:,}")
print(f"Frozen params: {total - trainable:,}")

# ═══════════════════════════════════════════════
#  3. TRAIN (only the new head)
# ═══════════════════════════════════════════════

# Only optimize trainable parameters
optimizer = optim.Adam(
    filter(lambda p: p.requires_grad, model.parameters()),
    lr=1e-3
)
criterion = nn.CrossEntropyLoss()

for epoch in range(10):
    model.train()
    train_loss = 0
    train_correct = 0
    
    for X, y in train_loader:
        X, y = X.to(device), y.to(device)
        optimizer.zero_grad()
        output = model(X)
        loss = criterion(output, y)
        loss.backward()
        optimizer.step()
        train_loss += loss.item() * X.size(0)
        train_correct += (output.argmax(1) == y).sum().item()
    
    # Validate
    model.eval()
    val_correct = 0
    with torch.no_grad():
        for X, y in val_loader:
            X, y = X.to(device), y.to(device)
            val_correct += (model(X).argmax(1) == y).sum().item()
    
    train_acc = train_correct / len(train_data)
    val_acc = val_correct / len(val_data)
    print(f"Epoch {epoch+1:2d} | Train Acc: {train_acc:.4f} | Val Acc: {val_acc:.4f}")
```

### Fine-Tuning (Progressive Unfreezing)

```python
import torch
import torch.nn as nn
import torch.optim as optim
from torchvision import models

device = torch.device('cuda' if torch.cuda.is_available() else 'cpu')

# Load pretrained model
model = models.resnet50(weights=models.ResNet50_Weights.IMAGENET1K_V2)

# Replace head
num_classes = 5
model.fc = nn.Sequential(
    nn.Linear(2048, 512),
    nn.ReLU(),
    nn.Dropout(0.3),
    nn.Linear(512, num_classes)
)
model = model.to(device)

criterion = nn.CrossEntropyLoss()

# ═══════════════════════════════════════════════
#  PHASE 1: Train head only (5 epochs)
# ═══════════════════════════════════════════════
print("Phase 1: Training head only")
for param in model.parameters():
    param.requires_grad = False
for param in model.fc.parameters():
    param.requires_grad = True

optimizer = optim.Adam(model.fc.parameters(), lr=1e-3)

for epoch in range(5):
    train_one_epoch(model, train_loader, optimizer, criterion, device)
    val_acc = evaluate(model, val_loader, device)
    print(f"  Epoch {epoch+1}: val_acc = {val_acc:.4f}")

# ═══════════════════════════════════════════════
#  PHASE 2: Unfreeze layer4 + head (10 epochs)
# ═══════════════════════════════════════════════
print("\nPhase 2: Unfreezing layer4")
for param in model.layer4.parameters():
    param.requires_grad = True

# Differential learning rates!
optimizer = optim.Adam([
    {'params': model.layer4.parameters(), 'lr': 1e-4},  # lower LR for pretrained
    {'params': model.fc.parameters(), 'lr': 1e-3},       # higher LR for head
])

for epoch in range(10):
    train_one_epoch(model, train_loader, optimizer, criterion, device)
    val_acc = evaluate(model, val_loader, device)
    print(f"  Epoch {epoch+1}: val_acc = {val_acc:.4f}")

# ═══════════════════════════════════════════════
#  PHASE 3: Unfreeze layer3 + layer4 + head (10 epochs)
# ═══════════════════════════════════════════════
print("\nPhase 3: Unfreezing layer3 + layer4")
for param in model.layer3.parameters():
    param.requires_grad = True

optimizer = optim.Adam([
    {'params': model.layer3.parameters(), 'lr': 1e-5},   # smallest LR
    {'params': model.layer4.parameters(), 'lr': 1e-4},   # medium LR
    {'params': model.fc.parameters(), 'lr': 1e-3},       # largest LR
])

for epoch in range(10):
    train_one_epoch(model, train_loader, optimizer, criterion, device)
    val_acc = evaluate(model, val_loader, device)
    print(f"  Epoch {epoch+1}: val_acc = {val_acc:.4f}")


# Helper functions
def train_one_epoch(model, loader, optimizer, criterion, device):
    model.train()
    for X, y in loader:
        X, y = X.to(device), y.to(device)
        optimizer.zero_grad()
        loss = criterion(model(X), y)
        loss.backward()
        optimizer.step()

def evaluate(model, loader, device):
    model.eval()
    correct = total = 0
    with torch.no_grad():
        for X, y in loader:
            X, y = X.to(device), y.to(device)
            correct += (model(X).argmax(1) == y).sum().item()
            total += y.size(0)
    return correct / total
```

---

## 6. Practical Guide: ImageNet Pretrained → Custom Task

```
  STEP-BY-STEP RECIPE:
  
  ┌──────────────────────────────────────────────────────────────┐
  │  1. CHOOSE A PRETRAINED MODEL                               │
  │     • ResNet-50: good default, well-studied                 │
  │     • EfficientNet-B0: good accuracy/speed tradeoff         │
  │     • ViT-B/16: if you have more data (>10K images)        │
  │                                                              │
  │  2. PREPARE YOUR DATA                                        │
  │     • Use SAME preprocessing as pretrained model!           │
  │       → ImageNet: Resize(256), CenterCrop(224)             │
  │       → Normalize(mean=[0.485,0.456,0.406],                │
  │                   std=[0.229,0.224,0.225])                  │
  │     • Add data augmentation for training set                │
  │                                                              │
  │  3. REPLACE THE HEAD                                         │
  │     • ResNet: model.fc = nn.Linear(2048, num_classes)      │
  │     • EfficientNet: model.classifier = nn.Linear(...)      │
  │     • ViT: model.heads = nn.Linear(768, num_classes)       │
  │                                                              │
  │  4. CHOOSE STRATEGY BASED ON DATA SIZE                       │
  │     • < 1K images: Feature extraction (freeze all)          │
  │     • 1K-10K: Fine-tune last 1-2 stages                    │
  │     • 10K-100K: Fine-tune most layers                       │
  │     • > 100K: Fine-tune all or train from scratch           │
  │                                                              │
  │  5. USE SMALL LEARNING RATE                                  │
  │     • Head: 1e-3                                            │
  │     • Pretrained layers: 1e-4 to 1e-5                      │
  │     • Never use LR > 1e-3 for pretrained layers!           │
  │                                                              │
  │  6. MONITOR FOR OVERFITTING                                  │
  │     • Small datasets overfit quickly                        │
  │     • Use early stopping (patience = 5-10)                  │
  │     • Use data augmentation aggressively                    │
  └──────────────────────────────────────────────────────────────┘
```

---

## 7. Transfer Learning Beyond Vision

```
  ┌──────────────────────────────────────────────────────────────┐
  │  DOMAIN           │ PRETRAINED MODEL     │ FINE-TUNE ON     │
  ├───────────────────┼──────────────────────┼──────────────────┤
  │  Computer Vision  │ ResNet (ImageNet)    │ Any image task   │
  │                   │ CLIP (image+text)    │ Zero-shot class. │
  ├───────────────────┼──────────────────────┼──────────────────┤
  │  NLP              │ BERT (Wikipedia)     │ Sentiment, QA    │
  │                   │ GPT (web text)       │ Text generation  │
  │                   │ RoBERTa              │ Classification   │
  ├───────────────────┼──────────────────────┼──────────────────┤
  │  Audio            │ Whisper (speech)     │ Transcription    │
  │                   │ wav2vec (audio)      │ Audio class.     │
  ├───────────────────┼──────────────────────┼──────────────────┤
  │  Medical          │ ResNet (ImageNet)    │ X-ray diagnosis  │
  │                   │ BioGPT (literature)  │ Medical NLP      │
  ├───────────────────┼──────────────────────┼──────────────────┤
  │  Multi-modal      │ CLIP, BLIP           │ Image captioning │
  │                   │ LLaVA                │ Visual QA        │
  └───────────────────┴──────────────────────┴──────────────────┘
```

---

## 8. Common Mistakes

```
  ┌──────────────────────────────────────────────────────────────┐
  │  MISTAKE                          │ FIX                     │
  ├───────────────────────────────────┼─────────────────────────┤
  │  Wrong preprocessing (not        │ Use EXACT same transform│
  │  matching pretrained model)       │ as pretrained training  │
  │                                   │                         │
  │  LR too high for pretrained      │ Use 1e-4 or 1e-5 for   │
  │  layers (destroys features!)     │ pretrained layers       │
  │                                   │                         │
  │  Not freezing enough layers      │ Start frozen, unfreeze  │
  │  with small dataset              │ gradually               │
  │                                   │                         │
  │  Forgetting model.eval() during  │ Always set eval mode    │
  │  inference (BN/dropout active)   │ for inference           │
  │                                   │                         │
  │  Training head with frozen BN    │ BN stats might not match│
  │  statistics from ImageNet        │ your data — fine-tune BN│
  │                                   │                         │
  │  Using random init instead of    │ ALWAYS start from       │
  │  pretrained for small datasets   │ pretrained when possible│
  └───────────────────────────────────┴─────────────────────────┘
```

---

## 9. Summary Table

| Strategy | Data Size | What to Train | Learning Rate | Training Time |
|----------|-----------|---------------|---------------|---------------|
| **Feature Extraction** | < 1K | New head only | 1e-3 (head) | Minutes |
| **Fine-tune last stage** | 1K-5K | Last stage + head | 1e-4 / 1e-3 | Hours |
| **Fine-tune half** | 5K-50K | Later half + head | 1e-5 / 1e-4 / 1e-3 | Hours |
| **Fine-tune all** | 50K-500K | Everything | 1e-5 (all) | Days |
| **From scratch** | 500K+ | Everything | 1e-2 to 1e-3 | Days/weeks |

---

## 10. Revision Questions

1. **Explain the concept of transfer learning.** Why are early layers of a CNN considered "universal features" while later layers are "task-specific"?

2. **Compare feature extraction and fine-tuning as transfer learning strategies.** When would you use each one, and what are the tradeoffs?

3. **What is progressive unfreezing and why is it important?** Explain why earlier layers should use smaller learning rates than later layers.

4. **Write complete PyTorch code to:** (a) load a pretrained ResNet-50, (b) freeze all layers, (c) replace the final FC layer for 5-class classification, (d) train only the new head for 10 epochs.

5. **How does domain similarity between the source and target tasks affect transfer learning strategy?** Give an example of high-similarity and low-similarity transfers and how you'd approach each.

6. **Why must you use the exact same normalization (mean and std) as the pretrained model's training data?** What happens if you normalize differently?

---

| [← 04 Residual Networks](04-residual-networks-resnet.md) | [Unit 10 Home](README.md) | [→ Next Section](../../) |
|:---|:---:|---:|
