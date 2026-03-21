# Image Classification — Standard Pipeline & Practical Implementation

> **Unit 8 · CNN Applications · Topic 1/6**

---

## Table of Contents

1. [Problem Formulation](#1-problem-formulation)
2. [Single-Label vs Multi-Label Classification](#2-single-label-vs-multi-label-classification)
3. [Standard Classification Pipeline](#3-standard-classification-pipeline)
4. [Evaluation Metrics](#4-evaluation-metrics)
5. [ImageNet Challenge & Benchmarks](#5-imagenet-challenge--benchmarks)
6. [Worked Examples](#6-worked-examples)
7. [Practical PyTorch Pipeline](#7-practical-pytorch-pipeline)
8. [Summary Table](#8-summary-table)
9. [Revision Questions](#9-revision-questions)
10. [Navigation](#10-navigation)

---

## 1. Problem Formulation

Image classification assigns one (or more) category labels to an entire image.

```
Input:  Image x ∈ ℝ^(H × W × C)
Output: Class label y ∈ {1, 2, ..., K}  (single-label)
    or: Label vector y ∈ {0,1}^K         (multi-label)

Pipeline:
  ┌─────────┐     ┌───────────┐     ┌──────────┐     ┌──────────┐
  │  Input   │     │  Feature  │     │  Class   │     │  Output  │
  │  Image   │────►│ Extractor │────►│  Head    │────►│  Scores  │
  │ (H×W×3)  │     │  (CNN/ViT)│     │ (FC+Soft)│     │  (K,)    │
  └─────────┘     └───────────┘     └──────────┘     └──────────┘

  Preprocessing → Backbone → Global Pooling → FC → Softmax → Prediction
```

---

## 2. Single-Label vs Multi-Label Classification

```
SINGLE-LABEL: Each image belongs to exactly ONE class
──────────────────────────────────────────────────────
  Image of a cat → P(cat)=0.92, P(dog)=0.05, P(bird)=0.03
  Prediction: argmax → "cat"
  
  Output: softmax(logits)  — sums to 1.0
  Loss:   CrossEntropyLoss
  
  ŷ = softmax(z) where ŷᵢ = exp(zᵢ) / Σⱼ exp(zⱼ)
  L = -Σᵢ yᵢ log(ŷᵢ) = -log(ŷ_true_class)

MULTI-LABEL: Each image can have MULTIPLE labels
──────────────────────────────────────────────────
  Photo of a beach → P(ocean)=0.95, P(sand)=0.90, P(sky)=0.85,
                      P(person)=0.60, P(car)=0.02
  
  Output: sigmoid(logits) per class — each independently [0,1]
  Loss:   BCEWithLogitsLoss (binary CE per class)
  
  ŷᵢ = σ(zᵢ) = 1/(1+exp(-zᵢ))    for each class independently
  L = -Σᵢ [yᵢ log(ŷᵢ) + (1-yᵢ) log(1-ŷᵢ)]

Comparison:
┌──────────────┬───────────────────┬───────────────────┐
│              │ Single-Label      │ Multi-Label       │
├──────────────┼───────────────────┼───────────────────┤
│ Final act.   │ Softmax           │ Sigmoid           │
│ Loss         │ CrossEntropy      │ BCEWithLogits     │
│ Probabilities│ Sum to 1          │ Independent       │
│ Prediction   │ argmax            │ Threshold (>0.5)  │
│ Example      │ ImageNet (1 of K) │ COCO tags         │
└──────────────┴───────────────────┴───────────────────┘
```

---

## 3. Standard Classification Pipeline

```
Complete Pipeline:

1. DATA PREPARATION
   ┌─────────────────────────────────────────────────┐
   │ • Organize: data/train/class_name/image.jpg     │
   │ • Split: train (80%) / val (10%) / test (10%)   │
   │ • Augment: RandomResizedCrop, Flip, ColorJitter │
   │ • Normalize: ImageNet mean/std                   │
   └─────────────────────────────────────────────────┘

2. MODEL SELECTION
   ┌─────────────────────────────────────────────────┐
   │ Pretrained backbone → replace classifier head   │
   │ Small data: Feature extraction (freeze backbone)│
   │ Large data: Fine-tune all layers                │
   └─────────────────────────────────────────────────┘

3. TRAINING
   ┌─────────────────────────────────────────────────┐
   │ Optimizer: AdamW (lr=1e-3 head, 1e-5 backbone) │
   │ Scheduler: Cosine annealing with warmup         │
   │ Loss: CrossEntropy + label smoothing            │
   │ Augmentation: RandAugment + Mixup + CutMix     │
   │ AMP: Mixed precision for speed                  │
   └─────────────────────────────────────────────────┘

4. EVALUATION
   ┌─────────────────────────────────────────────────┐
   │ Metrics: Top-1 accuracy, Top-5 accuracy         │
   │ Confusion matrix, per-class precision/recall    │
   │ Optional: TTA for final evaluation              │
   └─────────────────────────────────────────────────┘

5. DEPLOYMENT
   ┌─────────────────────────────────────────────────┐
   │ Export: TorchScript or ONNX                      │
   │ Quantize: INT8 for edge deployment              │
   │ Serve: TorchServe, Triton, or custom API        │
   └─────────────────────────────────────────────────┘
```

---

## 4. Evaluation Metrics

### Top-1 and Top-5 Accuracy

```
Top-1 Accuracy:
  Correct if the HIGHEST predicted class = true class
  Top-1 = (# correct) / (# total)

Top-5 Accuracy:
  Correct if the true class is in the TOP 5 predictions
  Top-5 = (# correct within top 5) / (# total)

Example:
  True class: "Golden Retriever"
  Predictions: [Labrador(0.35), Golden(0.30), Beagle(0.15), 
                Poodle(0.10), Husky(0.10)]
  
  Top-1: WRONG (Labrador ≠ Golden Retriever)
  Top-5: CORRECT (Golden Retriever is in top 5) ✓

ImageNet (1000 classes):
  Top-1 ≈ 78-90% (depending on model)
  Top-5 ≈ 94-99% (much easier — 5 chances in 1000 classes)
```

### Precision, Recall, F1 (Per-Class)

```
For class c:
  Precision_c = TP_c / (TP_c + FP_c)    — of predicted c, how many correct?
  Recall_c    = TP_c / (TP_c + FN_c)    — of actual c, how many found?
  F1_c        = 2 × P_c × R_c / (P_c + R_c)

Averaging:
  Macro: average over all classes (treats each class equally)
  Weighted: weight by class support (handles class imbalance)

Confusion Matrix (3 classes):
                  Predicted
                  Cat   Dog   Bird
  Actual  Cat  [  85    10     5  ]   Recall = 85/100 = 85%
          Dog  [   8    88     4  ]   Recall = 88/100 = 88%
          Bird [   3     5    92  ]   Recall = 92/100 = 92%
          
  Precision:     85/96  88/103 92/101
               = 88.5%  85.4%  91.1%
```

---

## 5. ImageNet Challenge & Benchmarks

```
ImageNet Large Scale Visual Recognition Challenge (ILSVRC):

Year  Model            Top-5 Error   Key Innovation
2010  Traditional ML    28.2%        Hand-crafted features
2011  Traditional ML    25.8%        
2012  AlexNet           16.4%        Deep learning era begins!
2013  ZFNet             14.8%        Visualization of CNN features
2014  GoogLeNet         6.7%         Inception module
2014  VGGNet            7.3%         Very deep (16-19 layers)
2015  ResNet            3.6%         Residual connections (152 layers)
2016  (Ensemble)        2.99%        
2017  SENet             2.25%        Channel attention

Human performance: ~5.1% top-5 error
→ CNNs surpassed human performance in 2015!

Current SOTA (2024):
  Model                Top-1 Acc    Params
  ConvNeXt V2-H        88.7%       660M
  EVA-02-L (ViT)       90.0%       304M  (with CLIP pretraining)
  InternImage-H        89.6%       1.08B
```

---

## 6. Worked Examples

### Example: Softmax Computation

```
Logits (raw scores): z = [2.0, 1.0, 0.1]

Step 1: Exponentiate
  exp(2.0) = 7.389
  exp(1.0) = 2.718
  exp(0.1) = 1.105

Step 2: Sum
  Σ = 7.389 + 2.718 + 1.105 = 11.212

Step 3: Normalize
  P(class 0) = 7.389 / 11.212 = 0.659
  P(class 1) = 2.718 / 11.212 = 0.242
  P(class 2) = 1.105 / 11.212 = 0.099
  Sum = 1.000 ✓

Step 4: Cross-entropy loss (true class = 0)
  L = -log(0.659) = 0.417
```

### Example: Top-5 Accuracy Computation

```
5 test images, 10 classes:

Image 1: True=3, Top-5 predicted=[3,1,7,2,5]  → Top-1: ✓  Top-5: ✓
Image 2: True=7, Top-5 predicted=[2,7,1,5,3]  → Top-1: ✗  Top-5: ✓
Image 3: True=0, Top-5 predicted=[0,4,2,8,1]  → Top-1: ✓  Top-5: ✓
Image 4: True=5, Top-5 predicted=[1,2,3,4,6]  → Top-1: ✗  Top-5: ✗
Image 5: True=9, Top-5 predicted=[9,8,7,6,5]  → Top-1: ✓  Top-5: ✓

Top-1 accuracy = 3/5 = 60%
Top-5 accuracy = 4/5 = 80%
```

---

## 7. Practical PyTorch Pipeline

```python
import torch
import torch.nn as nn
from torch.utils.data import DataLoader
from torchvision import datasets, transforms, models
from torch.cuda.amp import autocast, GradScaler
import timm

# ============================================================
# 1. Data Preparation
# ============================================================
train_transform = transforms.Compose([
    transforms.RandomResizedCrop(224, scale=(0.08, 1.0)),
    transforms.RandomHorizontalFlip(),
    transforms.RandAugment(num_ops=2, magnitude=9),
    transforms.ToTensor(),
    transforms.Normalize([0.485, 0.456, 0.406],
                         [0.229, 0.224, 0.225]),
    transforms.RandomErasing(p=0.25),
])

val_transform = transforms.Compose([
    transforms.Resize(256),
    transforms.CenterCrop(224),
    transforms.ToTensor(),
    transforms.Normalize([0.485, 0.456, 0.406],
                         [0.229, 0.224, 0.225]),
])

train_dataset = datasets.ImageFolder('data/train', train_transform)
val_dataset   = datasets.ImageFolder('data/val', val_transform)

train_loader = DataLoader(train_dataset, batch_size=64, shuffle=True,
                          num_workers=4, pin_memory=True, drop_last=True)
val_loader   = DataLoader(val_dataset, batch_size=128, shuffle=False,
                          num_workers=4, pin_memory=True)

num_classes = len(train_dataset.classes)
print(f"Classes: {train_dataset.classes}")

# ============================================================
# 2. Model
# ============================================================
model = timm.create_model('efficientnet_b3', pretrained=True,
                           num_classes=num_classes)
device = torch.device('cuda' if torch.cuda.is_available() else 'cpu')
model.to(device)

# ============================================================
# 3. Training Setup
# ============================================================
criterion = nn.CrossEntropyLoss(label_smoothing=0.1)
optimizer = torch.optim.AdamW(model.parameters(), lr=1e-3, weight_decay=0.01)
scheduler = torch.optim.lr_scheduler.CosineAnnealingLR(
    optimizer, T_max=50, eta_min=1e-6)
scaler = GradScaler()

# ============================================================
# 4. Training Loop
# ============================================================
best_acc = 0.0

for epoch in range(50):
    # --- Train ---
    model.train()
    train_loss, train_correct, train_total = 0, 0, 0
    
    for images, labels in train_loader:
        images, labels = images.to(device), labels.to(device)
        optimizer.zero_grad(set_to_none=True)
        
        with autocast(device_type='cuda'):
            outputs = model(images)
            loss = criterion(outputs, labels)
        
        scaler.scale(loss).backward()
        scaler.unscale_(optimizer)
        nn.utils.clip_grad_norm_(model.parameters(), max_norm=1.0)
        scaler.step(optimizer)
        scaler.update()
        
        train_loss += loss.item() * images.size(0)
        train_correct += outputs.argmax(1).eq(labels).sum().item()
        train_total += images.size(0)
    
    # --- Validate ---
    model.eval()
    val_correct, val_top5_correct, val_total = 0, 0, 0
    
    with torch.no_grad():
        for images, labels in val_loader:
            images, labels = images.to(device), labels.to(device)
            with autocast(device_type='cuda'):
                outputs = model(images)
            
            # Top-1
            val_correct += outputs.argmax(1).eq(labels).sum().item()
            # Top-5
            _, top5_pred = outputs.topk(5, dim=1)
            val_top5_correct += top5_pred.eq(
                labels.unsqueeze(1)).any(1).sum().item()
            val_total += images.size(0)
    
    scheduler.step()
    
    train_acc = 100 * train_correct / train_total
    val_acc = 100 * val_correct / val_total
    val_top5 = 100 * val_top5_correct / val_total
    
    if val_acc > best_acc:
        best_acc = val_acc
        torch.save(model.state_dict(), 'best_model.pth')
    
    print(f"Epoch {epoch+1}: Train Acc={train_acc:.1f}%, "
          f"Val Top-1={val_acc:.1f}%, Val Top-5={val_top5:.1f}%")

# ============================================================
# 5. Inference
# ============================================================
model.load_state_dict(torch.load('best_model.pth'))
model.eval()

def predict(image_path):
    from PIL import Image
    image = Image.open(image_path).convert('RGB')
    input_tensor = val_transform(image).unsqueeze(0).to(device)
    
    with torch.no_grad():
        with autocast(device_type='cuda'):
            logits = model(input_tensor)
    
    probs = torch.softmax(logits, dim=1)
    top5_probs, top5_indices = probs.topk(5)
    
    for prob, idx in zip(top5_probs[0], top5_indices[0]):
        print(f"  {train_dataset.classes[idx]}: {prob:.3f}")
```

---

## 8. Summary Table

| Concept | Key Idea |
|---|---|
| **Single-label** | One class per image; softmax + cross-entropy |
| **Multi-label** | Multiple classes per image; sigmoid + BCE |
| **Top-1 accuracy** | Correct if highest prediction = true class |
| **Top-5 accuracy** | Correct if true class in top 5 predictions |
| **Standard pipeline** | Preprocess → backbone → pool → FC → softmax |
| **Label smoothing** | Soft targets prevent overconfidence |
| **ImageNet** | 1000 classes, 1.2M images; standard benchmark |
| **Human top-5 error** | ~5.1%; surpassed by CNNs in 2015 (ResNet) |
| **Practical choices** | EfficientNet/ConvNeXt + AdamW + cosine + AMP |

---

## 9. Revision Questions

1. **Compare single-label and multi-label classification.** What activation function and loss function does each use? Why can't you use softmax for multi-label?

2. **Explain Top-1 and Top-5 accuracy.** Given 5 test images and their top-5 predictions, compute both metrics.

3. **Describe the complete image classification pipeline** from raw images to deployment. What are the key decisions at each stage?

4. **Compute softmax probabilities and cross-entropy loss** for logits `[3.0, 1.0, 0.5, -1.0]` with true class index 0.

5. **How did CNN accuracy on ImageNet evolve** from 2012 to 2020? What was the key innovation at each major improvement?

6. **Design a classification pipeline** for a dataset of 5000 images across 20 classes. Choose model, augmentation, optimizer, scheduler, and justify each choice.

---

## 10. Navigation

| Previous | Up | Next |
|---|---|---|
| [← Mixed Precision Training](../07-Training-CNNs/05-mixed-precision-training.md) | [Unit 8 Index](../08-CNN-Applications/) | [Object Detection →](./02-object-detection.md) |

---

> **References:**  
> - Russakovsky, O. et al. (2015). *ImageNet Large Scale Visual Recognition Challenge.* IJCV.  
> - He, K. et al. (2016). *Deep Residual Learning for Image Recognition.* CVPR.  
> - Wightman, R. et al. (2021). *ResNet Strikes Back: An Improved Training Procedure in timm.*
