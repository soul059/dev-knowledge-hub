# Image Classification: Problem Formulation

## Overview

Image classification assigns a single label to an entire image from a predefined set of categories. It's the most fundamental CV task — "What is in this image?" — and serves as the foundation for object detection, segmentation, and visual understanding.

---

## Problem Definition

```
Input:  Image I (H × W × C tensor)
Output: Class label y ∈ {class_1, class_2, ..., class_K}

Examples:
  Image of a cat     → "cat"
  Image of a car     → "car"
  Chest X-ray        → "pneumonia" or "normal"
  Satellite image    → "forest", "urban", "water"

Mathematically:
  f: ℝ^(H×W×C) → {1, 2, ..., K}
  
  Or with probabilities:
  f: ℝ^(H×W×C) → [0,1]^K    (probability for each class)
  Predicted class = argmax(f(I))
```

---

## Classification Pipeline

```
                    ┌─────────────┐
  Image ──→        │ Preprocessing │
  (224×224×3)      │ resize, norm  │
                    └──────┬──────┘
                           │
                    ┌──────┴──────┐
                    │  Feature     │
                    │  Extractor   │  CNN backbone (ResNet, EfficientNet)
                    │  (backbone)  │
                    └──────┬──────┘
                           │
                    ┌──────┴──────┐
                    │  Classifier  │  Fully connected layers
                    │  (head)      │  + Softmax
                    └──────┬──────┘
                           │
                    ┌──────┴──────┐
                    │  Output      │
                    │  [0.05, 0.9, │  Probabilities for each class
                    │   0.03, 0.02]│  → Predicted: class 2
                    └─────────────┘
```

---

## Loss Function

```
Cross-Entropy Loss (for multi-class):

  L = -Σ_{c=1}^{K} y_c × log(p_c)

  For one-hot labels (only true class matters):
  L = -log(p_true_class)

  Example:
    True label: "cat" (class 0)
    Prediction: [0.7, 0.2, 0.1]  (cat, dog, bird)
    Loss = -log(0.7) = 0.357

    Perfect prediction: [1.0, 0.0, 0.0]
    Loss = -log(1.0) = 0.0
```

---

## Key Datasets

| Dataset | Classes | Images | Image Size | Task |
|---------|:-:|:-:|:-:|------|
| MNIST | 10 | 70K | 28×28 (gray) | Handwritten digits |
| CIFAR-10 | 10 | 60K | 32×32 | Small natural images |
| CIFAR-100 | 100 | 60K | 32×32 | Fine-grained |
| ImageNet | 1000 | 1.2M | ~256×256 | Large-scale |
| Food-101 | 101 | 101K | ~512×512 | Food recognition |

---

## Quick Example

```python
import torch
import torchvision.transforms as transforms
from torchvision import models
from PIL import Image

# Load pre-trained model
model = models.resnet50(weights=models.ResNet50_Weights.DEFAULT)
model.eval()

# Preprocess
transform = transforms.Compose([
    transforms.Resize(256),
    transforms.CenterCrop(224),
    transforms.ToTensor(),
    transforms.Normalize(mean=[0.485, 0.456, 0.406],
                         std=[0.229, 0.224, 0.225]),
])

img = Image.open("cat.jpg")
input_tensor = transform(img).unsqueeze(0)  # add batch dim

# Predict
with torch.no_grad():
    output = model(input_tensor)
    probabilities = torch.softmax(output[0], dim=0)
    top5 = torch.topk(probabilities, 5)

for prob, idx in zip(top5.values, top5.indices):
    print(f"  {prob:.3f}: class {idx.item()}")
```

---

## Evaluation Metrics

| Metric | Formula | Use Case |
|--------|---------|----------|
| Accuracy | correct / total | Balanced datasets |
| Top-5 accuracy | True label in top 5 predictions | ImageNet |
| Precision | TP / (TP + FP) | Per-class performance |
| Recall | TP / (TP + FN) | Missing detections |
| F1 Score | 2 × P × R / (P + R) | Imbalanced datasets |
| Confusion matrix | All predictions tabulated | Error analysis |

---

## Revision Questions

1. **What is the mathematical formulation of image classification?**
2. **What is cross-entropy loss and why is it used for classification?**
3. **What preprocessing steps are needed before feeding images to a CNN?**
4. **What is Top-5 accuracy and why is it used for ImageNet?**
5. **When should you use F1 score instead of accuracy?**

---

[Previous: ../04-Traditional-CV-Methods/05-image-segmentation.md](../04-Traditional-CV-Methods/05-image-segmentation.md) | [Next: 02-cnn-classification.md](02-cnn-classification.md)
