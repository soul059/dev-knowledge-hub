# 4. Data Augmentation

> **Unit 7 · Regularization Techniques** — Increasing effective dataset size through transformations

---

## Chapter Overview

**Data augmentation** artificially expands the training dataset by applying label-preserving transformations to existing samples. A flipped cat is still a cat; a slightly rotated digit is still the same digit. By training on these variations, the model becomes **invariant** to the transformations, leading to better generalization. Data augmentation is the most universally applicable regularization technique and is considered **mandatory** for image classification tasks.

---

## 1. Why Data Augmentation Works

### Increasing Effective Dataset Size

```
  Original dataset: 1,000 images
  
  With augmentation (per image):
  • 1 original
  • 1 horizontal flip
  • 4 random crops
  • 2 color variations
  = 8 versions per image
  
  Effective dataset: 1,000 × 8 = 8,000 images
  (though not 8× as diverse as 8,000 unique images)
```

### Teaching Invariances

```
  Without augmentation:                With augmentation:
  
  Model sees cat facing right →        Model sees cat facing right,
  Model learns "cat = this              left, upside down, zoomed,
  specific arrangement of pixels"       dark, bright, cropped →
  → FRAGILE: fails on new poses        → ROBUST: recognizes cats in
                                           any orientation/condition
```

### Regularization Effect

```
  Without augmentation:              With augmentation:
  
  Loss ↑                              Loss ↑
       │╲  ·                              │╲  ·
       │ ╲  ·  ·                          │ ╲  ·
       │  ╲   ·   · · · · ·              │  ╲  ·  ·
       │   ╲──── train                    │   ╲──── ·
       │       ───── val (gap = overfit)  │     ╲──── train
       └──────────→ epoch                 │      ────── val (small gap!)
                                          └──────────→ epoch
```

---

## 2. Image Augmentations

### Common Geometric Transforms

```
  Original:       Horizontal Flip:    Rotation (15°):
  ┌─────────┐     ┌─────────┐        ┌─────────┐
  │  ╱╲     │     │     ╱╲  │        │   ╱╲    │
  │ ╱  ╲    │     │    ╱  ╲ │        │  ╱  ╲╲  │
  │ ╲__╱    │     │    ╲__╱ │        │  ╲__╱ ╲ │
  │ │  │    │     │    │  │ │        │  │  │   │
  │ │  │    │     │    │  │ │        │         │
  └─────────┘     └─────────┘        └─────────┘
  
  Random Crop:    Scale/Zoom:         Shear:
  ┌────┐          ┌─────────┐        ┌─────────┐
  │╱╲  │          │ ┌─────┐ │        │  ╱╲     │
  │╲__╱│          │ │╱╲   │ │        │ ╱  ╲    │
  ││  ││          │ │╲__╱ │ │        │╱╲__╱    │
  └────┘          │ │     │ │        ││  │     │
    ↑             │ └─────┘ │        │         │
  crop to 224     └─────────┘        └─────────┘
```

### Common Photometric Transforms

| Transform | Effect | Typical Range |
|---|---|---|
| **Brightness** | Lighten/darken | ±20% |
| **Contrast** | Increase/decrease contrast | ±20% |
| **Saturation** | Color intensity | ±30% |
| **Hue** | Color shift | ±10° |
| **Gaussian noise** | Add random noise | σ = 0.01–0.05 |
| **Gaussian blur** | Blur the image | kernel 3–7 |
| **JPEG compression** | Add compression artifacts | quality 50–100 |
| **Grayscale** | Remove color | p = 0.1–0.2 |

---

## 3. Standard Augmentation Recipes

### ImageNet / General Image Classification

```python
import torchvision.transforms as T

train_transform = T.Compose([
    T.RandomResizedCrop(224, scale=(0.08, 1.0)),  # random crop & resize
    T.RandomHorizontalFlip(p=0.5),                # 50% chance flip
    T.ColorJitter(brightness=0.4, contrast=0.4, 
                  saturation=0.4, hue=0.1),        # color variations
    T.ToTensor(),
    T.Normalize(mean=[0.485, 0.456, 0.406],        # ImageNet stats
                std=[0.229, 0.224, 0.225]),
])

val_transform = T.Compose([
    T.Resize(256),                                  # resize
    T.CenterCrop(224),                              # deterministic crop
    T.ToTensor(),
    T.Normalize(mean=[0.485, 0.456, 0.406],
                std=[0.229, 0.224, 0.225]),
])
# NOTE: Validation transforms are DETERMINISTIC — no randomness!
```

### CIFAR-10 Recipe

```python
train_transform = T.Compose([
    T.RandomCrop(32, padding=4),         # pad 4 pixels, then random crop
    T.RandomHorizontalFlip(),
    T.ToTensor(),
    T.Normalize((0.4914, 0.4822, 0.4465),
                (0.2470, 0.2435, 0.2616)),
])
```

---

## 4. Advanced Augmentations: Mixup and CutMix

### Mixup (Zhang et al., 2018)

Create new training samples by **linearly interpolating** between two samples:

```
  x̃ = λ · xᵢ + (1-λ) · xⱼ       ← mix inputs
  ỹ = λ · yᵢ + (1-λ) · yⱼ       ← mix labels (soft)
  
  where λ ~ Beta(α, α), typically α = 0.2
```

```
  Image A (cat):     Image B (dog):     Mixup (λ=0.6):
  ┌─────────┐       ┌─────────┐        ┌─────────┐
  │  ╱╲     │       │   ___   │        │  ╱╲__   │
  │ (  )    │  +    │  / _ \  │   =    │ ( _ )   │
  │  ╲╱     │       │ |   | | │        │  ╲╱ |   │
  │ /||\ ▾  │       │  \_|_/  │        │ /|| ▾/  │
  └─────────┘       └─────────┘        └─────────┘
  Label: [1,0]      Label: [0,1]       Label: [0.6, 0.4]
  (cat=1, dog=0)    (cat=0, dog=1)     (60% cat, 40% dog)
```

### CutMix (Yun et al., 2019)

Replace a **rectangular region** of one image with a patch from another:

```
  Image A (cat):     Image B (dog):     CutMix:
  ┌─────────┐       ┌─────────┐        ┌─────────┐
  │  ╱╲     │       │   ___   │        │  ╱╲     │
  │ (  )    │  +    │  / _ \  │   =    │ (  )    │
  │  ╲╱     │       │ |   | | │        │  ╲╱┌───┐│
  │ /||\ ▾  │       │  \_|_/  │        │ /||│_|_/││
  └─────────┘       └─────────┘        └────└───┘─┘
  Label: [1,0]      Label: [0,1]       Label: [0.75, 0.25]
                                        (proportional to area)
```

### PyTorch Implementation

```python
def mixup_data(x, y, alpha=0.2):
    """Apply Mixup augmentation."""
    if alpha > 0:
        lam = torch.distributions.Beta(alpha, alpha).sample()
    else:
        lam = 1.0
    
    batch_size = x.size(0)
    index = torch.randperm(batch_size)
    
    mixed_x = lam * x + (1 - lam) * x[index]
    y_a, y_b = y, y[index]
    
    return mixed_x, y_a, y_b, lam

def mixup_criterion(criterion, pred, y_a, y_b, lam):
    """Mixup loss: weighted combination of losses."""
    return lam * criterion(pred, y_a) + (1 - lam) * criterion(pred, y_b)

# Usage in training loop:
for x, y in train_loader:
    x_mix, y_a, y_b, lam = mixup_data(x, y, alpha=0.2)
    pred = model(x_mix)
    loss = mixup_criterion(criterion, pred, y_a, y_b, lam)
    optimizer.zero_grad(); loss.backward(); optimizer.step()
```

---

## 5. Text Augmentations

### Common NLP Augmentations

| Technique | Example | Use Case |
|---|---|---|
| **Synonym replacement** | "happy" → "joyful" | Sentiment analysis |
| **Random insertion** | "the cat sat" → "the small cat sat" | General NLP |
| **Random deletion** | "the cat sat on mat" → "cat sat mat" | Robustness |
| **Random swap** | "the cat sat" → "cat the sat" | Word order robustness |
| **Back-translation** | EN→FR→EN: "the cat" → "le chat" → "the cat/feline" | Paraphrasing |
| **Token masking** | "the [MASK] sat" | BERT-style pretraining |

```python
# Simple text augmentation
import random

def synonym_replace(sentence, synonyms_dict, n=1):
    """Replace n random words with synonyms."""
    words = sentence.split()
    for _ in range(n):
        candidates = [i for i, w in enumerate(words) if w.lower() in synonyms_dict]
        if candidates:
            idx = random.choice(candidates)
            words[idx] = random.choice(synonyms_dict[words[idx].lower()])
    return ' '.join(words)

# Example
synonyms = {"happy": ["joyful", "glad", "pleased"], 
            "big": ["large", "huge", "enormous"]}
print(synonym_replace("I am happy with the big result", synonyms))
```

---

## 6. Tabular Data Augmentations

### SMOTE (Synthetic Minority Over-sampling)

For **imbalanced** tabular datasets, create synthetic samples of the minority class:

```
  SMOTE Algorithm:
  1. Select a minority sample x
  2. Find its k nearest neighbors (same class)
  3. Pick a random neighbor xₙ
  4. Create synthetic: x_new = x + λ·(xₙ - x), λ ∈ [0,1]
  
  Feature 2 ↑
             │        ●     Original minority samples
             │   ○    ●     ○ = synthetic (interpolated)
             │  ●  ○  
             │     ●   ○
             │   ○   ●
             └──────────────→ Feature 1
```

```python
# Using imbalanced-learn library
# pip install imbalanced-learn

from imblearn.over_sampling import SMOTE

smote = SMOTE(random_state=42)
X_resampled, y_resampled = smote.fit_resample(X_train, y_train)
```

---

## 7. Albumentations Library

**Albumentations** is a fast, flexible augmentation library:

```python
import albumentations as A
from albumentations.pytorch import ToTensorV2

train_transform = A.Compose([
    A.RandomResizedCrop(224, 224, scale=(0.5, 1.0)),
    A.HorizontalFlip(p=0.5),
    A.VerticalFlip(p=0.1),
    A.ShiftScaleRotate(shift_limit=0.1, scale_limit=0.2,
                        rotate_limit=30, p=0.5),
    A.OneOf([
        A.GaussNoise(var_limit=(10, 50)),
        A.GaussianBlur(blur_limit=(3, 7)),
        A.MotionBlur(blur_limit=7),
    ], p=0.3),
    A.ColorJitter(brightness=0.3, contrast=0.3, 
                   saturation=0.3, hue=0.1, p=0.5),
    A.CoarseDropout(max_holes=8, max_height=16, max_width=16,
                     fill_value=0, p=0.3),     # Cutout
    A.Normalize(mean=[0.485, 0.456, 0.406],
                std=[0.229, 0.224, 0.225]),
    ToTensorV2(),
])

# Albumentations works with numpy arrays:
# augmented = train_transform(image=image_np)['image']
```

---

## 8. Test-Time Augmentation (TTA)

Apply augmentations at **inference** and average predictions:

```
  Standard Inference:
  image → model → prediction
  
  TTA (Test-Time Augmentation):
  image           → model → prediction₁ ─┐
  image (flipped) → model → prediction₂ ─┼──→ average → final prediction
  image (crop 1)  → model → prediction₃ ─┤
  image (crop 2)  → model → prediction₄ ─┘
```

```python
def test_time_augmentation(model, image, n_augments=5):
    """Apply TTA and average predictions."""
    model.eval()
    predictions = []
    
    with torch.no_grad():
        # Original
        predictions.append(model(image.unsqueeze(0)))
        
        # Horizontal flip
        predictions.append(model(torch.flip(image, [2]).unsqueeze(0)))
        
        # Random crops (multiple)
        for _ in range(n_augments - 2):
            aug = T.RandomResizedCrop(224, scale=(0.8, 1.0))
            predictions.append(model(aug(image).unsqueeze(0)))
    
    # Average all predictions
    return torch.stack(predictions).mean(dim=0)
```

---

## 9. Guidelines

### What to Augment (and What Not To)

```
  ┌──────────────────────────────────────────────────────────┐
  │  SAFE augmentations (label-preserving):                  │
  │  ✓ Horizontal flip (for most objects)                    │
  │  ✓ Small rotations (±15°)                               │
  │  ✓ Random crops with resize                              │
  │  ✓ Color jitter (brightness, contrast)                   │
  │  ✓ Gaussian noise                                        │
  │                                                          │
  │  DANGEROUS augmentations (may change label):              │
  │  ✗ Vertical flip (for text recognition — "p" vs "d")    │
  │  ✗ Large rotation (for digit recognition — "6" vs "9")  │
  │  ✗ Horizontal flip (for text — "b" vs "d")              │
  │  ✗ Heavy color change (for color-dependent tasks)        │
  └──────────────────────────────────────────────────────────┘
```

### Augmentation Intensity

| Dataset Size | Augmentation |
|---|---|
| Large (>100K) | Light (flip + crop) |
| Medium (10K–100K) | Moderate (flip + crop + color + noise) |
| Small (<10K) | Heavy (all of above + Mixup + CutMix) |
| Very small (<1K) | Maximum + consider transfer learning |

---

## Summary Table

| Concept | Key Idea |
|---|---|
| Core idea | Apply label-preserving transforms to increase data diversity |
| Geometric | Flip, rotate, crop, scale, shear |
| Photometric | Brightness, contrast, saturation, noise |
| Mixup | Interpolate pairs: x̃ = λx₁ + (1-λ)x₂ |
| CutMix | Replace patch of one image with another |
| Text augmentation | Synonym replacement, back-translation |
| Tabular (SMOTE) | Interpolate minority class samples |
| TTA | Augment at test time, average predictions |
| Apply to | Training set only (validation/test = deterministic) |
| PyTorch | `torchvision.transforms`, albumentations |

---

## Revision Questions

1. **Why is data augmentation considered a form of regularization?** How does it reduce overfitting without modifying the loss function?

2. **Explain Mixup mathematically.** Why does mixing labels as well as inputs lead to better calibrated models?

3. **What augmentations would you avoid for digit recognition (0–9)?** Explain with specific examples where the augmentation would change the label.

4. **Compare torchvision.transforms and albumentations.** What are the key differences in API and performance?

5. **Implement CutMix in PyTorch** including the area-proportional label mixing. How do you randomly select the cut region?

6. **What is TTA and when would you use it?** What is the trade-off between computation and accuracy?

---

## Navigation

| Previous | Up | Next |
|---|---|---|
| [← Early Stopping](./03-early-stopping.md) | [Unit 7: Regularization](./README.md) | [Batch Normalization →](./05-batch-normalization.md) |
