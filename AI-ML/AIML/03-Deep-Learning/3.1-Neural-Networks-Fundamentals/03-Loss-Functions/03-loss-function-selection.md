# 🧭 Loss Function Selection Guide

> **Chapter 3.3 — Choosing the Right Loss Function for Your Task**

| Topic | Details |
|---|---|
| **Subject** | Loss Functions — Decision Framework |
| **Prerequisites** | Chapters 3.1-3.2 |
| **Key Insight** | The loss function encodes what you want the model to optimize — choosing the wrong loss means solving the wrong problem |

---

## 📌 Overview

Choosing the right loss function is one of the most important decisions in building a neural network. The loss function defines the optimization objective — it literally determines what your model learns. A poor choice can lead to a model that optimizes for the wrong thing, is sensitive to outliers, fails on imbalanced data, or produces uncalibrated predictions. This chapter provides a systematic decision framework for selecting the right loss.

---

## 1. The Decision Flowchart

```
                        ┌──────────────────────┐
                        │   What is your task?  │
                        └──────────┬───────────┘
                                   │
                    ┌──────────────┼──────────────┐
                    ▼              ▼              ▼
              ┌──────────┐  ┌──────────────┐  ┌──────────┐
              │Regression│  │Classification│  │  Other   │
              └────┬─────┘  └──────┬───────┘  └────┬─────┘
                   │               │               │
            ┌──────┼──────┐   ┌────┼────┐         │
            ▼      ▼      ▼   ▼    ▼    ▼         ▼
         ┌─────┐┌─────┐┌────┐│   │    │      Ranking/
         │Clean ││Noisy││Both││   │    │      Contrastive/
         │data  ││data ││    ││   │    │      Perceptual
         └──┬──┘└──┬──┘└──┬─┘│   │    │
            │      │      │   │   │    │
            ▼      ▼      ▼   ▼   ▼    ▼
           MSE    MAE   Huber │  │   Multi-
                              │  │   label
                           Binary │
                           class. │
                              │   │
                              ▼   ▼
                           ┌────┐ ┌────────┐
                           │BCE │ │Softmax │
                           │    │ │  + CE  │
                           └────┘ └────────┘
                              │      │
                         Imbalanced? │
                              │      │
                              ▼      ▼
                        Focal Loss / Weighted CE
```

---

## 2. Complete Task-to-Loss Mapping

### 2.1 Regression Tasks

| Scenario | Loss Function | PyTorch | Why |
|---|---|---|---|
| **Standard regression** | MSE | `nn.MSELoss()` | Smooth gradients, penalizes large errors |
| **Outlier-robust regression** | MAE | `nn.L1Loss()` | Linear penalty, robust to outliers |
| **Best of both worlds** | Huber | `nn.HuberLoss(delta=1.0)` | Smooth near zero, robust to outliers |
| **Percentage error matters** | MAPE | Custom | Scale-independent |
| **Log-scale predictions** | MSLE | Custom | Penalizes ratio, not absolute error |
| **Quantile estimation** | Quantile Loss | Custom | Predict specific percentiles |
| **Image reconstruction** | MSE + Perceptual | Custom | Pixel-wise + perceptual quality |

### 2.2 Classification Tasks

| Scenario | Loss Function | PyTorch | Why |
|---|---|---|---|
| **Binary (2 classes)** | Binary CE | `nn.BCEWithLogitsLoss()` | Standard for binary |
| **Multi-class (K classes, mutually exclusive)** | Cross-Entropy | `nn.CrossEntropyLoss()` | Standard for multi-class |
| **Multi-label (multiple classes per sample)** | BCE per label | `nn.BCEWithLogitsLoss()` | Independent binary decisions |
| **Imbalanced classes** | Focal Loss | Custom | Down-weights easy examples |
| **Imbalanced classes** | Weighted CE | `nn.CrossEntropyLoss(weight=w)` | Up-weights rare classes |
| **Noisy labels** | Label Smoothing | `nn.CrossEntropyLoss(label_smoothing=0.1)` | Prevents overconfidence |
| **Ordinal classification** | Ordinal CE | Custom | Respects class ordering |

### 2.3 Specialized Tasks

| Scenario | Loss Function | Use Case |
|---|---|---|
| **Similarity learning** | Contrastive Loss | Learn embeddings where similar items are close |
| **Triplet learning** | Triplet Loss | Learn embeddings with anchor-positive-negative triplets |
| **Ranking** | Margin Ranking Loss | Learn to rank items (search, recommendation) |
| **Image generation** | Adversarial Loss (GAN) | Generator vs. discriminator |
| **Style transfer** | Perceptual Loss | Match features from pre-trained network |
| **Segmentation** | Dice Loss | Overlap between predicted and true masks |
| **Object detection** | CE + Smooth L1 | Classification + bounding box regression |
| **Sequence-to-sequence** | CTC Loss | Alignment-free sequence prediction |

---

## 3. Detailed Decision Guide

### 3.1 Binary vs Multi-class vs Multi-label

```
    BINARY CLASSIFICATION              Example: Spam detection
    ┌───────────────────────┐          "Is this email spam?"
    │  Classes: {0, 1}      │          Output: 1 neuron + sigmoid
    │  Mutually exclusive   │          Loss: BCEWithLogitsLoss
    │  Output: 1 neuron     │
    │  Activation: Sigmoid  │
    └───────────────────────┘

    MULTI-CLASS CLASSIFICATION         Example: Digit recognition
    ┌───────────────────────┐          "Which digit is this? (0-9)"
    │  Classes: {0,1,...,K-1}│          Output: 10 neurons + softmax
    │  Mutually exclusive   │          Loss: CrossEntropyLoss
    │  Output: K neurons    │          Exactly one class per sample
    │  Activation: Softmax  │
    └───────────────────────┘

    MULTI-LABEL CLASSIFICATION         Example: Image tagging
    ┌───────────────────────┐          "What's in this photo?"
    │  Labels: {dog, cat,   │          Tags: [dog=1, outdoor=1, cat=0, car=0]
    │    outdoor, car, ...} │          Output: K neurons + sigmoid (each)
    │  NOT mutually exclusive│          Loss: BCEWithLogitsLoss
    │  Output: K neurons    │          Multiple labels per sample!
    │  Activation: Sigmoid  │
    │  (each independently) │
    └───────────────────────┘
```

### 3.2 When to Use Weighted Loss

```python
# Scenario: Medical diagnosis
# Dataset: 10,000 healthy (class 0), 100 diseased (class 1)
# Ratio: 100:1

# Option 1: Class weights (inversely proportional to frequency)
weights = torch.tensor([1.0, 100.0])  # Weight diseased 100x more
criterion = nn.CrossEntropyLoss(weight=weights)

# Option 2: Balanced weights (auto-computed)
from sklearn.utils.class_weight import compute_class_weight
class_weights = compute_class_weight('balanced', 
                                      classes=np.array([0, 1]),
                                      y=train_labels)
weights = torch.tensor(class_weights, dtype=torch.float32)
criterion = nn.CrossEntropyLoss(weight=weights)

# Option 3: Focal Loss (no manual weight tuning needed)
criterion = FocalLoss(gamma=2.0)

# Option 4: Oversampling minority class (data-level, not loss-level)
from torch.utils.data import WeightedRandomSampler
sample_weights = [100.0 if label == 1 else 1.0 for label in train_labels]
sampler = WeightedRandomSampler(sample_weights, num_samples=len(train_labels))
```

---

## 4. Loss Function Properties at a Glance

```
    ┌────────────────────────────────────────────────────────────────┐
    │             REGRESSION LOSSES COMPARISON                       │
    ├────────────┬──────────┬──────────┬──────────┬─────────────────┤
    │  Property  │   MSE    │   MAE    │  Huber   │ Log-Cosh        │
    ├────────────┼──────────┼──────────┼──────────┼─────────────────┤
    │ Smooth     │  ✅ Yes  │  ❌ No   │  ✅ Yes  │  ✅ Yes         │
    │ Robust     │  ❌ No   │  ✅ Yes  │  ✅ Yes  │  ✅ Moderate    │
    │ Fast conv. │  ✅ Yes  │  ❌ Slow │  ✅ Yes  │  ✅ Yes         │
    │ Interpretable│ ✅ Yes │  ✅ Yes  │  ⚠️ Less │  ❌ Less        │
    │ Hyper-param│  None    │  None    │  δ       │  None           │
    └────────────┴──────────┴──────────┴──────────┴─────────────────┘

    ┌────────────────────────────────────────────────────────────────┐
    │           CLASSIFICATION LOSSES COMPARISON                     │
    ├────────────┬──────────┬──────────┬──────────┬─────────────────┤
    │  Property  │   BCE    │    CE    │  Focal   │ Hinge           │
    ├────────────┼──────────┼──────────┼──────────┼─────────────────┤
    │ Probabilistic│ ✅ Yes │  ✅ Yes  │  ✅ Yes  │  ❌ No          │
    │ Imbalanced │  ❌ Poor │  ❌ Poor │  ✅ Good │  ❌ Poor        │
    │ Calibrated │  ✅ Yes  │  ✅ Yes  │  ⚠️ Less │  ❌ No          │
    │ Hyper-param│  None    │  None    │  γ, α    │  margin         │
    │ Gradient   │  Simple  │  Simple  │  Complex │  Sparse         │
    └────────────┴──────────┴──────────┴──────────┴─────────────────┘
```

---

## 5. Common Mistakes and How to Avoid Them

### 5.1 Mistake: Using MSE for Classification

```
    ❌ WRONG: MSE for classification
    
    True label: y = [0, 0, 1]  (class 2)
    Prediction: ŷ = [0.1, 0.1, 0.8]
    
    MSE = (1/3)[(0-0.1)² + (0-0.1)² + (1-0.8)²] = 0.02
    
    But with ŷ = [0.3, 0.3, 0.4]:
    MSE = (1/3)[(0-0.3)² + (0-0.3)² + (1-0.4)²] = 0.18
    
    Problem: MSE penalizes ALL probabilities being off, not just the true class.
    Cross-entropy only cares about the true class probability.
    
    ✅ CORRECT: Use Cross-Entropy for classification
    CE for ŷ = [0.1, 0.1, 0.8]: -log(0.8) = 0.223
    CE for ŷ = [0.3, 0.3, 0.4]: -log(0.4) = 0.916
```

### 5.2 Mistake: Wrong Activation + Loss Combination

```
    ❌ WRONG combinations:
    • Sigmoid output + CrossEntropyLoss  (CE expects raw logits!)
    • Softmax output + BCELoss           (wrong for multi-class!)
    • Linear output + BCELoss            (need probabilities for BCE!)
    
    ✅ CORRECT combinations:
    • Raw logits + nn.CrossEntropyLoss()      (softmax built-in)
    • Raw logits + nn.BCEWithLogitsLoss()     (sigmoid built-in)
    • Sigmoid output + nn.BCELoss()           (manual sigmoid)
    • No output activation + nn.MSELoss()     (regression)
```

---

## 6. Quick Reference Code

```python
import torch
import torch.nn as nn

# ──── Task → Loss Function Quick Reference ────

# REGRESSION
regression_losses = {
    'standard':    nn.MSELoss(),
    'robust':      nn.HuberLoss(delta=1.0),
    'l1_robust':   nn.L1Loss(),
    'smooth_l1':   nn.SmoothL1Loss(beta=1.0),
}

# BINARY CLASSIFICATION
binary_losses = {
    'standard':    nn.BCEWithLogitsLoss(),           # From logits (recommended)
    'from_probs':  nn.BCELoss(),                     # From probabilities
    'weighted':    nn.BCEWithLogitsLoss(pos_weight=torch.tensor([10.0])),
}

# MULTI-CLASS CLASSIFICATION
multiclass_losses = {
    'standard':    nn.CrossEntropyLoss(),
    'weighted':    nn.CrossEntropyLoss(weight=torch.tensor([1., 2., 5.])),
    'smoothed':    nn.CrossEntropyLoss(label_smoothing=0.1),
    'ignore_pad':  nn.CrossEntropyLoss(ignore_index=-100),   # For NLP padding
}

# MULTI-LABEL CLASSIFICATION (K independent binary decisions)
multilabel_loss = nn.BCEWithLogitsLoss()  # Applied to K-dimensional output

# RANKING
ranking_losses = {
    'margin':      nn.MarginRankingLoss(margin=1.0),
    'triplet':     nn.TripletMarginLoss(margin=1.0),
    'cosine':      nn.CosineEmbeddingLoss(margin=0.5),
}

# Example usage
print("Task → Loss Mapping:")
print(f"  Regression:           nn.MSELoss()")
print(f"  Binary classification: nn.BCEWithLogitsLoss()")
print(f"  Multi-class:          nn.CrossEntropyLoss()")
print(f"  Multi-label:          nn.BCEWithLogitsLoss()")
print(f"  Ranking:              nn.TripletMarginLoss()")
```

---

## 📊 Summary Table

| Scenario | Loss | Output Layer | Activation |
|---|---|---|---|
| **Regression** | MSE / Huber / MAE | 1 neuron | None (linear) |
| **Binary classification** | BCE | 1 neuron | Sigmoid (or built-in) |
| **Multi-class** | Cross-Entropy | K neurons | Softmax (or built-in) |
| **Multi-label** | BCE per label | K neurons | Sigmoid (each) |
| **Imbalanced** | Focal / Weighted CE | As above | As above |
| **Noisy labels** | Label-smoothed CE | K neurons | Softmax |
| **Similarity** | Contrastive / Triplet | Embedding | None |

---

## ❓ Revision Questions

1. **You're building a model to predict housing prices. The dataset has some mansions that are 50x the median price. Which loss function should you use and why?**

2. **You're classifying medical images into 10 disease categories. Only 2% of images belong to the rarest class. What loss function would you choose and what modifications would you make?**

3. **Explain why using MSE for classification is problematic. Give a numerical example showing where CE would give a better gradient signal than MSE.**

4. **When should you use `nn.BCEWithLogitsLoss()` vs `nn.BCELoss()` in PyTorch? What is the numerical stability advantage?**

5. **Design the complete output configuration (neurons, activation, loss) for: (a) predicting age from a photo, (b) classifying an image as cat/dog/bird, (c) tagging a photo with multiple attributes (sunny, outdoor, person, car).**

6. **What is the relationship between cross-entropy loss and maximum likelihood estimation? Why does minimizing CE maximize the likelihood?**

---

## 🧭 Navigation

| Previous | Up | Next |
|---|---|---|
| [← Classification Losses](02-classification-losses.md) | [🏠 Neural Networks Fundamentals](../README.md) | [Custom Loss Functions →](04-custom-loss-functions.md) |

---

*© 2024 AIML Study Notes. Built for deep understanding, not shallow memorization.*
