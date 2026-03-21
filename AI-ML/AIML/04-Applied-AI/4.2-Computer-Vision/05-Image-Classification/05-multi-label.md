# Multi-label Classification

## Overview

Multi-label classification assigns **multiple labels** to a single image, unlike standard classification where each image gets exactly one label. An image of a beach might be labeled: {ocean, sand, sunset, people}. This is common in real-world applications like photo tagging, medical diagnosis, and content moderation.

---

## Single-label vs Multi-label

```
Single-label:
  Image → exactly 1 class
  Labels: [0, 0, 1, 0, 0]   (one-hot, one 1)
  Loss: CrossEntropyLoss + Softmax
  Output: probabilities sum to 1

Multi-label:
  Image → 0 or more classes
  Labels: [1, 0, 1, 1, 0]   (multi-hot, any number of 1s)
  Loss: BCEWithLogitsLoss (per-class sigmoid)
  Output: independent probability per class

  Key difference:
    Softmax: P(cat) + P(dog) + P(bird) = 1.0  (compete)
    Sigmoid: P(cat)=0.9, P(dog)=0.8, P(bird)=0.1 (independent)
```

---

## Architecture

```
Standard classification:
  CNN → FC → Softmax → [0.1, 0.8, 0.1]  (sum = 1)
                        argmax → class 1

Multi-label classification:
  CNN → FC → Sigmoid → [0.9, 0.1, 0.8, 0.7, 0.2]  (independent)
                        threshold → [1, 0, 1, 1, 0]
                        Labels: {class0, class2, class3}
```

---

## Implementation

```python
import torch
import torch.nn as nn
import torchvision.models as models

class MultiLabelClassifier(nn.Module):
    def __init__(self, num_classes):
        super().__init__()
        self.backbone = models.resnet50(weights=models.ResNet50_Weights.DEFAULT)
        self.backbone.fc = nn.Linear(self.backbone.fc.in_features, num_classes)
        # No softmax! BCEWithLogitsLoss applies sigmoid internally
    
    def forward(self, x):
        return self.backbone(x)

# Setup
num_classes = 20  # e.g., Pascal VOC
model = MultiLabelClassifier(num_classes)
criterion = nn.BCEWithLogitsLoss()  # Binary cross-entropy per class
optimizer = torch.optim.Adam(model.parameters(), lr=1e-4)

# Training
for images, labels in train_loader:
    # labels shape: (batch_size, num_classes) with 0s and 1s
    outputs = model(images)  # (batch_size, num_classes)
    loss = criterion(outputs, labels.float())
    
    optimizer.zero_grad()
    loss.backward()
    optimizer.step()

# Inference
model.eval()
with torch.no_grad():
    logits = model(test_image.unsqueeze(0))
    probs = torch.sigmoid(logits)           # independent probabilities
    preds = (probs > 0.5).int()             # threshold at 0.5
    
    predicted_labels = [classes[i] for i in range(num_classes) if preds[0][i]]
    print(f"Labels: {predicted_labels}")
    # Labels: ['beach', 'sunset', 'people']
```

---

## Evaluation Metrics

```
Per-sample metrics:
  Precision = |predicted ∩ actual| / |predicted|
  Recall    = |predicted ∩ actual| / |actual|
  F1        = 2 × P × R / (P + R)

  Example:
    Actual:    {cat, dog, bird}
    Predicted: {cat, bird, fish}
    
    |predicted ∩ actual| = |{cat, bird}| = 2
    Precision = 2/3 = 0.67
    Recall    = 2/3 = 0.67
    F1        = 0.67

Aggregation:
  Micro:  Pool all predictions, compute globally
  Macro:  Average metrics across classes
  Weighted: Weight by class frequency

mAP (mean Average Precision):
  Standard metric for multi-label (used in Pascal VOC, MS COCO)
  Compute AP (area under P-R curve) per class, then average
```

---

## Handling Imbalanced Labels

| Strategy | Description |
|----------|-------------|
| Weighted BCE | Higher weight for rare classes |
| Focal loss | Down-weight easy examples, focus on hard ones |
| Oversampling | Duplicate rare-label images |
| Class-aware augmentation | Augment more for rare classes |
| Asymmetric loss | Different treatment for positive vs negative |

---

## Revision Questions

1. **How does multi-label differ from multi-class classification?**
2. **Why use Sigmoid + BCE instead of Softmax + CE for multi-label?**
3. **How do you choose the threshold for converting probabilities to labels?**
4. **What is mAP and how is it computed for multi-label classification?**
5. **How do you handle class imbalance in multi-label settings?**

---

[Previous: 04-fine-tuning.md](04-fine-tuning.md) | [Back to README](../README.md)
