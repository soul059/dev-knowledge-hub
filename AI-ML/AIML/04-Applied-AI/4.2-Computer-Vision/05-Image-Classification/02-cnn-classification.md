# CNN-based Classification

## Overview

Convolutional Neural Networks are the backbone of modern image classification. From AlexNet (2012) to EfficientNet and Vision Transformers, CNN architectures have evolved dramatically. This chapter covers the key architectures, their innovations, and how to use them.

---

## Evolution of CNN Architectures

```
Year  Model         Depth   Top-5 Error   Key Innovation
2012  AlexNet        8      16.4%         ReLU, dropout, GPU training
2014  VGGNet        16-19   7.3%          Small 3×3 filters, deep
2014  GoogLeNet     22      6.7%          Inception modules (multi-scale)
2015  ResNet        152     3.6%          Skip connections (residual)
2016  DenseNet      264     -             Dense connections
2017  SENet         -       2.3%          Channel attention (squeeze-excite)
2019  EfficientNet  -       2.9%          Compound scaling
2020  ViT           -       ~2%           Transformer for images
```

---

## Key Architecture Patterns

```
VGG pattern (deeper = better):
  [Conv3×3] → [Conv3×3] → [MaxPool] → [Conv3×3] → [Conv3×3] → [MaxPool] → ...
  Simple but 138M parameters!

ResNet pattern (skip connections):
  x → [Conv] → [BN] → [ReLU] → [Conv] → [BN] → (+x) → [ReLU]
       └──────────────────────────────────────────┘
       Skip connection: F(x) + x
  Solves vanishing gradient, enables 152+ layers

Inception pattern (multi-scale):
  ┌── [1×1 Conv] ──┐
  ├── [3×3 Conv] ──┤
  ├── [5×5 Conv] ──┼── Concatenate → output
  └── [MaxPool]  ──┘
  Captures features at multiple scales simultaneously

EfficientNet (compound scaling):
  Scale width × depth × resolution together
  α^φ × β^φ × γ^φ = 2   (constrained optimization)
  Best accuracy/efficiency tradeoff
```

---

## Building a Classifier from Scratch

```python
import torch
import torch.nn as nn

class SimpleCNN(nn.Module):
    def __init__(self, num_classes=10):
        super().__init__()
        self.features = nn.Sequential(
            nn.Conv2d(3, 32, 3, padding=1),
            nn.BatchNorm2d(32),
            nn.ReLU(),
            nn.MaxPool2d(2),
            
            nn.Conv2d(32, 64, 3, padding=1),
            nn.BatchNorm2d(64),
            nn.ReLU(),
            nn.MaxPool2d(2),
            
            nn.Conv2d(64, 128, 3, padding=1),
            nn.BatchNorm2d(128),
            nn.ReLU(),
            nn.AdaptiveAvgPool2d(1),  # global average pooling
        )
        self.classifier = nn.Linear(128, num_classes)
    
    def forward(self, x):
        x = self.features(x)
        x = x.view(x.size(0), -1)
        return self.classifier(x)

# Training loop
model = SimpleCNN(num_classes=10)
criterion = nn.CrossEntropyLoss()
optimizer = torch.optim.Adam(model.parameters(), lr=1e-3)

for epoch in range(10):
    for images, labels in train_loader:
        outputs = model(images)
        loss = criterion(outputs, labels)
        
        optimizer.zero_grad()
        loss.backward()
        optimizer.step()
```

---

## Using Pre-trained Models

```python
import torchvision.models as models

# Load with pre-trained ImageNet weights
resnet = models.resnet50(weights=models.ResNet50_Weights.DEFAULT)
efficientnet = models.efficientnet_b0(weights=models.EfficientNet_B0_Weights.DEFAULT)
vit = models.vit_b_16(weights=models.ViT_B_16_Weights.DEFAULT)

# Model comparison
print(f"ResNet-50:       {sum(p.numel() for p in resnet.parameters())/1e6:.1f}M params")
print(f"EfficientNet-B0: {sum(p.numel() for p in efficientnet.parameters())/1e6:.1f}M params")
print(f"ViT-B/16:        {sum(p.numel() for p in vit.parameters())/1e6:.1f}M params")
```

---

## Architecture Comparison

| Model | Params | Top-1 Acc | FLOPs | Best For |
|-------|:-:|:-:|:-:|---------|
| ResNet-50 | 25M | 76.1% | 4.1G | General purpose |
| EfficientNet-B0 | 5.3M | 77.1% | 0.4G | Mobile/edge |
| EfficientNet-B7 | 66M | 84.3% | 37G | Maximum accuracy |
| ViT-B/16 | 86M | 81.8% | 17.6G | Large datasets |
| ConvNeXt-T | 28M | 82.1% | 4.5G | Modern CNN |

---

## Revision Questions

1. **What innovation did ResNet introduce and why was it important?**
2. **How does an Inception module capture multi-scale features?**
3. **What is compound scaling in EfficientNet?**
4. **Why has global average pooling replaced fully connected layers?**
5. **When would you choose EfficientNet vs ViT?**

---

[Previous: 01-problem-formulation.md](01-problem-formulation.md) | [Next: 03-transfer-learning.md](03-transfer-learning.md)
