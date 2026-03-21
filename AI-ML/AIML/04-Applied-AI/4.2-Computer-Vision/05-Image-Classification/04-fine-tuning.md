# Fine-tuning Pre-trained Models

## Overview

Fine-tuning takes a pre-trained model and continues training it on your specific dataset. Unlike feature extraction (freezing all layers), fine-tuning updates some or all weights with a small learning rate. This chapter covers practical techniques for effective fine-tuning: learning rate scheduling, data augmentation, regularization, and common pitfalls.

---

## Fine-tuning Best Practices

```
1. Start with frozen backbone, train head for a few epochs
2. Unfreeze layers gradually (from top to bottom)
3. Use discriminative learning rates (lower for earlier layers)
4. Heavy data augmentation to prevent overfitting
5. Use learning rate warmup + cosine annealing
6. Monitor validation loss — stop when it increases

Training schedule:
  Phase 1 (epochs 1-3):   Freeze backbone, train head (lr=1e-3)
  Phase 2 (epochs 4-10):  Unfreeze layer4, reduce lr (lr=1e-4)
  Phase 3 (epochs 11-15): Unfreeze all, very low lr (lr=1e-5)
```

---

## Data Augmentation

```python
from torchvision import transforms

# Strong augmentation for fine-tuning (prevents overfitting)
train_transform = transforms.Compose([
    transforms.RandomResizedCrop(224, scale=(0.8, 1.0)),
    transforms.RandomHorizontalFlip(),
    transforms.RandomRotation(15),
    transforms.ColorJitter(brightness=0.2, contrast=0.2, saturation=0.2, hue=0.1),
    transforms.RandomAffine(degrees=0, translate=(0.1, 0.1)),
    transforms.ToTensor(),
    transforms.Normalize([0.485, 0.456, 0.406], [0.229, 0.224, 0.225]),
    transforms.RandomErasing(p=0.2),  # cutout-like augmentation
])

# Validation: no augmentation (except resize/crop/normalize)
val_transform = transforms.Compose([
    transforms.Resize(256),
    transforms.CenterCrop(224),
    transforms.ToTensor(),
    transforms.Normalize([0.485, 0.456, 0.406], [0.229, 0.224, 0.225]),
])

# Advanced: Albumentations (more augmentation options)
import albumentations as A
from albumentations.pytorch import ToTensorV2

train_aug = A.Compose([
    A.RandomResizedCrop(224, 224),
    A.HorizontalFlip(p=0.5),
    A.ShiftScaleRotate(shift_limit=0.1, scale_limit=0.15, rotate_limit=15),
    A.CoarseDropout(max_holes=8, max_height=16, max_width=16, p=0.3),
    A.Normalize(mean=[0.485, 0.456, 0.406], std=[0.229, 0.224, 0.225]),
    ToTensorV2(),
])
```

---

## Complete Fine-tuning Pipeline

```python
import torch
import torch.nn as nn
import torchvision.models as models
from torch.optim.lr_scheduler import CosineAnnealingLR

# Model setup
model = models.efficientnet_b0(weights=models.EfficientNet_B0_Weights.DEFAULT)
model.classifier[1] = nn.Linear(model.classifier[1].in_features, num_classes)

# Phase 1: Freeze backbone
for param in model.features.parameters():
    param.requires_grad = False

optimizer = torch.optim.AdamW(model.classifier.parameters(), lr=1e-3, weight_decay=0.01)
scheduler = CosineAnnealingLR(optimizer, T_max=10)
criterion = nn.CrossEntropyLoss(label_smoothing=0.1)  # label smoothing helps!

# Training with early stopping
best_val_loss = float('inf')
patience = 3
patience_counter = 0

for epoch in range(20):
    # Unfreeze after epoch 3
    if epoch == 3:
        for param in model.features[-3:].parameters():
            param.requires_grad = True
        optimizer.add_param_group({
            "params": model.features[-3:].parameters(), "lr": 1e-4
        })
    
    model.train()
    for images, labels in train_loader:
        outputs = model(images)
        loss = criterion(outputs, labels)
        optimizer.zero_grad()
        loss.backward()
        torch.nn.utils.clip_grad_norm_(model.parameters(), max_norm=1.0)
        optimizer.step()
    
    scheduler.step()
    
    # Validation
    model.eval()
    val_loss = evaluate(model, val_loader, criterion)
    
    if val_loss < best_val_loss:
        best_val_loss = val_loss
        torch.save(model.state_dict(), "best_model.pth")
        patience_counter = 0
    else:
        patience_counter += 1
        if patience_counter >= patience:
            print("Early stopping!")
            break
```

---

## Common Pitfalls

| Pitfall | Symptom | Solution |
|---------|---------|----------|
| Learning rate too high | Loss explodes, weights diverge | Use 1e-4 to 1e-5 for fine-tuning |
| No augmentation | Overfits quickly | Heavy augmentation |
| Training too long | Val loss increases | Early stopping |
| Forgetting | Pre-trained knowledge lost | Gradual unfreezing, small lr |
| Wrong normalization | Bad predictions | Use ImageNet mean/std |
| Batch size too small | Unstable BN statistics | Use ≥16, or freeze BN layers |

---

## Revision Questions

1. **What is the difference between feature extraction and fine-tuning?**
2. **Why should you use a smaller learning rate when fine-tuning?**
3. **What is gradual unfreezing and why does it help?**
4. **How does label smoothing help during fine-tuning?**
5. **What data augmentation strategies are most effective for image classification?**

---

[Previous: 03-transfer-learning.md](03-transfer-learning.md) | [Next: 05-multi-label.md](05-multi-label.md)
