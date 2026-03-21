# Data Augmentation for Images — Techniques, Libraries & Best Practices

> **Unit 7 · Training CNNs · Topic 1/5**

---

## Table of Contents

1. [Overview & Motivation](#1-overview--motivation)
2. [Geometric Augmentations](#2-geometric-augmentations)
3. [Color & Photometric Augmentations](#3-color--photometric-augmentations)
4. [Noise & Blur Augmentations](#4-noise--blur-augmentations)
5. [Advanced Augmentations](#5-advanced-augmentations)
6. [Automated Augmentation Policies](#6-automated-augmentation-policies)
7. [torchvision.transforms v2](#7-torchvisiontransforms-v2)
8. [Albumentations Library](#8-albumentations-library)
9. [Test-Time Augmentation (TTA)](#9-test-time-augmentation-tta)
10. [Worked Examples](#10-worked-examples)
11. [Summary Table](#11-summary-table)
12. [Revision Questions](#12-revision-questions)
13. [Navigation](#13-navigation)

---

## 1. Overview & Motivation

Data augmentation artificially increases training set size by applying label-preserving transformations to existing images, reducing overfitting and improving generalization.

```
Why Augmentation Works:

Original: 1 image of a cat          After augmentation: 10+ variations
┌─────────┐                         ┌─────────┐ ┌─────────┐ ┌─────────┐
│         │                         │  flipped │ │ rotated │ │ cropped │
│   🐱    │  ─── Augment ──────►   │   🐱    │ │   🐱   │ │   🐱   │
│         │                         └─────────┘ └─────────┘ └─────────┘
└─────────┘                         ┌─────────┐ ┌─────────┐ ┌─────────┐
Label: "cat"                        │brighter  │ │ noisy  │ │ scaled │
                                    │   🐱    │ │   🐱   │ │   🐱   │
                                    └─────────┘ └─────────┘ └─────────┘
                                    All still labeled "cat"!

Key benefits:
  • Regularization — prevents memorizing training data
  • Invariance — model learns to be robust to transformations
  • Free data — no manual labeling needed
  • Domain adaptation — simulate real-world variations
```

### Augmentation Taxonomy

```
┌────────────────────────────────────────────────────────┐
│              Data Augmentation Methods                  │
├────────────────────┬───────────────────────────────────┤
│ GEOMETRIC          │ Flip, Rotate, Crop, Scale, Affine│
│ COLOR/PHOTOMETRIC  │ Jitter, Normalize, Grayscale     │
│ NOISE/BLUR         │ Gaussian noise, blur, JPEG comp. │
│ ERASING/MASKING    │ CutOut, Random Erasing, GridMask │
│ MIXING             │ Mixup, CutMix                    │
│ AUTOMATED          │ AutoAugment, RandAugment         │
└────────────────────┴───────────────────────────────────┘
```

---

## 2. Geometric Augmentations

### Horizontal/Vertical Flip

```
Original:        H-Flip:          V-Flip:
┌──────────┐    ┌──────────┐    ┌──────────┐
│ A  B  C  │    │ C  B  A  │    │ G  H  I  │
│ D  E  F  │    │ F  E  D  │    │ D  E  F  │
│ G  H  I  │    │ I  H  G  │    │ A  B  C  │
└──────────┘    └──────────┘    └──────────┘

H-flip: x_new(i,j) = x(i, W-1-j)  — mirror left-right
V-flip: x_new(i,j) = x(H-1-i, j)  — mirror top-bottom

Usage: H-flip almost universally used (p=0.5)
       V-flip only if orientation doesn't matter
       ⚠ Don't flip: text recognition, left/right-specific tasks
```

### Random Rotation

```
Rotation by angle θ:

[x']   [cos θ  -sin θ] [x - cx]   [cx]
[y'] = [sin θ   cos θ] [y - cy] + [cy]

Common range: θ ∈ [-15°, +15°] for natural images
              θ ∈ [-180°, +180°] for aerial/medical images

┌────────┐     ┌────────┐     ┌────────┐
│   ──   │     │   /    │     │   |    │
│   ──   │     │  /     │     │   |    │
│   ──   │     │ /      │     │   |    │
└────────┘     └────────┘     └────────┘
  0°             30°             90°
```

### Random Crop / Resized Crop

```
RandomResizedCrop (most common for training):

1. Sample crop area ratio: area ∈ [0.08, 1.0] of original
2. Sample aspect ratio: ratio ∈ [3/4, 4/3]
3. Crop region from image
4. Resize crop to target size (e.g., 224×224)

Original (512×512):
┌──────────────────────┐
│                      │
│   ┌──────────┐       │ Crop 1: large area, centered
│   │          │       │
│   │  Crop 1  │       │
│   │          │       │
│   └──────────┘       │
│                      │
│          ┌────┐      │ Crop 2: small area, off-center
│          │Cr 2│      │
│          └────┘      │
└──────────────────────┘

Both → resize to 224×224

This is the STANDARD augmentation for ImageNet training!
  transforms.RandomResizedCrop(224, scale=(0.08, 1.0))
```

### Affine Transformation

```
General 2D affine transformation:

[x']   [a  b  tx] [x]
[y'] = [c  d  ty] [y]
[1 ]   [0  0   1] [1]

Combines: rotation, scale, shear, translation

Shear transform:
┌──────┐     ╱──────╱
│      │    ╱      ╱    shear_x: rows shifted proportional to y
│      │   ╱      ╱
│      │  ╱      ╱
└──────┘ ╱──────╱

Perspective transform (non-affine):
┌──────┐    ╱────╲
│      │   ╱      ╲    Simulates viewpoint change
│      │  │        │
│      │  │        │
└──────┘   ╲      ╱
             ╲──╱
```

---

## 3. Color & Photometric Augmentations

### Color Jitter

```
ColorJitter(brightness, contrast, saturation, hue):

  Brightness: pixel × (1 + δ),  δ ∈ [-brightness, +brightness]
  Contrast:   pixel = mean + (pixel - mean) × (1 + δ)
  Saturation: interpolate between grayscale and original
  Hue:        shift hue channel in HSV space

Original       +Brightness    +Contrast      -Saturation
┌─────────┐   ┌─────────┐   ┌─────────┐   ┌─────────┐
│ ░▒▓█    │   │ ▒▓██    │   │ ░░██    │   │ ░░▒▒    │
│ ░▒▓█    │   │ ▒▓██    │   │ ░░██    │   │ ░░▒▒    │
└─────────┘   └─────────┘   └─────────┘   └─────────┘

Typical values: ColorJitter(0.4, 0.4, 0.4, 0.1)
```

### Normalization

```
Per-channel normalization (required, not optional):

  x_norm = (x - μ) / σ

ImageNet statistics (standard):
  μ = [0.485, 0.456, 0.406]  (R, G, B means)
  σ = [0.229, 0.224, 0.225]  (R, G, B stds)

This is applied AFTER all augmentations, during preprocessing.
Ensures model sees standardized inputs regardless of augmentation.
```

### Grayscale Conversion

```
RandomGrayscale(p=0.1):

RGB → Gray: Y = 0.299R + 0.587G + 0.114B

Original:          Grayscale:
┌─────────┐       ┌─────────┐
│ R G B   │       │ Y Y Y   │  (replicated to 3 channels)
│ R G B   │  ──►  │ Y Y Y   │
│ R G B   │       │ Y Y Y   │
└─────────┘       └─────────┘

Forces model to use shape/texture rather than color cues
```

---

## 4. Noise & Blur Augmentations

```
Gaussian Noise:
  x_noisy = x + N(0, σ²)
  σ typically 0.01-0.1 of image range
  Simulates sensor noise in cameras

Gaussian Blur:
  Apply Gaussian kernel to smooth image
  K(x,y) = (1/2πσ²) · exp(-(x²+y²)/(2σ²))
  σ ∈ [0.1, 2.0]
  Simulates out-of-focus or motion blur

JPEG Compression:
  Compress image with quality factor q ∈ [40, 100]
  Adds compression artifacts
  Simulates real-world image degradation

Kernel examples (3×3):

Gaussian Blur:        Sharpen:
[1/16  2/16  1/16]   [-1  -1  -1]
[2/16  4/16  2/16]   [-1   9  -1]
[1/16  2/16  1/16]   [-1  -1  -1]
```

---

## 5. Advanced Augmentations

### Mixup (Zhang et al., 2018)

```
Mixup: Linear interpolation of two training samples

  x̃ = λ · xᵢ + (1-λ) · xⱼ     — mixed image
  ỹ = λ · yᵢ + (1-λ) · yⱼ     — mixed label (soft)

  λ ~ Beta(α, α),  typically α = 0.2 or 1.0

Example (λ = 0.6):
  Image A (cat):  60%  ┐
                       ├──► Blended image, label = [0.6 cat, 0.4 dog]
  Image B (dog):  40%  ┘

  ┌─────────┐   ┌─────────┐     ┌─────────┐
  │   🐱    │ + │   🐕    │ ──► │ 🐱+🐕   │  (blended)
  │  (cat)  │   │  (dog)  │     │y=[.6,.4] │
  └─────────┘   └─────────┘     └─────────┘

Benefits:
  • Regularization — prevents overconfident predictions
  • Smoother decision boundaries
  • Better calibration
```

### CutMix (Yun et al., 2019)

```
CutMix: Replace a rectangular region of one image with a patch from another

  x̃ = M ⊙ xᵢ + (1-M) ⊙ xⱼ   — M is a binary mask
  ỹ = λ · yᵢ + (1-λ) · yⱼ     — λ = 1 - (area of cut / total area)

Example (30% of area replaced):
  ┌─────────────┐     ┌─────────────┐     ┌─────────────┐
  │ 🐱 🐱 🐱 🐱│     │ 🐕 🐕 🐕 🐕│     │ 🐱 🐱 🐱 🐱│
  │ 🐱 🐱 🐱 🐱│  +  │ 🐕 🐕 🐕 🐕│ ──► │ 🐱 ┌────┐ 🐱│
  │ 🐱 🐱 🐱 🐱│     │ 🐕 🐕 🐕 🐕│     │ 🐱 │ 🐕🐕│ 🐱│
  │ 🐱 🐱 🐱 🐱│     │ 🐕 🐕 🐕 🐕│     │ 🐱 └────┘ 🐱│
  └─────────────┘     └─────────────┘     └─────────────┘
   cat (100%)          dog (100%)          cat=70%, dog=30%

Superior to Mixup: retains local image statistics (no blending artifacts)
```

### CutOut / Random Erasing

```
CutOut (DeVries & Taylor, 2017):
  Randomly mask out a square region of the image with zeros (or random values)

  ┌────────────┐     ┌────────────┐
  │ 🐱 🐱 🐱 🐱│     │ 🐱 🐱 🐱 🐱│
  │ 🐱 🐱 🐱 🐱│ ──► │ 🐱 ▓▓▓▓ 🐱│  ← masked region
  │ 🐱 🐱 🐱 🐱│     │ 🐱 ▓▓▓▓ 🐱│
  │ 🐱 🐱 🐱 🐱│     │ 🐱 🐱 🐱 🐱│
  └────────────┘     └────────────┘

Random Erasing (Zhong et al., 2020):
  Similar but fills with random values instead of zeros
  More common in practice (torchvision.transforms.RandomErasing)

  Parameters:
    p = 0.5    — probability of erasing
    scale = (0.02, 0.33)  — proportion of image area
    ratio = (0.3, 3.3)    — aspect ratio of erased region

Forces model to not rely on any single discriminative region
```

### GridMask

```
GridMask (Chen, 2020):
  Mask out a structured grid of regions

  ┌──────────────────┐
  │ ░░██░░██░░██░░██ │  ░ = visible
  │ ░░██░░██░░██░░██ │  █ = masked
  │ ██░░██░░██░░██░░ │
  │ ██░░██░░██░░██░░ │  Regular grid pattern
  │ ░░██░░██░░██░░██ │  (unlike random CutOut)
  │ ░░██░░██░░██░░██ │
  └──────────────────┘

Benefits: Avoids masking entire objects (which CutOut can do)
```

---

## 6. Automated Augmentation Policies

### AutoAugment (Cubuk et al., 2019)

```
Searches for optimal augmentation policies using RL:

Policy = 5 sub-policies, each with 2 operations
Each operation: (type, probability, magnitude)

Searched using:
  • Controller: RNN (similar to NAS)
  • Reward: validation accuracy on target dataset
  • Cost: 5000 GPU-hours on CIFAR-10

Example discovered policy for ImageNet:
┌────────────────────────────────────────────────────┐
│ Sub-policy 1: Posterize(p=0.4, m=8) + Rotate(p=0.6, m=9)
│ Sub-policy 2: Solarize(p=0.6, m=5) + AutoContrast(p=0.6)
│ Sub-policy 3: Equalize(p=0.8)     + Equalize(p=0.6)
│ Sub-policy 4: Posterize(p=0.6, m=7)+ Posterize(p=0.6, m=6)
│ Sub-policy 5: Equalize(p=0.4)     + Solarize(p=0.2, m=4)
└────────────────────────────────────────────────────┘

For each training image: randomly pick one sub-policy, apply it
```

### RandAugment (Cubuk et al., 2020)

```
MUCH simpler than AutoAugment — just 2 hyperparameters!

RandAugment(N, M):
  N = number of augmentations to apply (typically 2-3)
  M = magnitude of each augmentation (0-30, typically 9-15)

Algorithm:
  1. Pool of K=14 augmentation operations
  2. Randomly select N operations from the pool
  3. Apply each with magnitude M

  Operations pool:
  ┌──────────────────────────────────────────┐
  │ Identity, AutoContrast, Equalize,       │
  │ Rotate, Solarize, Color, Posterize,     │
  │ Contrast, Brightness, Sharpness,        │
  │ ShearX, ShearY, TranslateX, TranslateY  │
  └──────────────────────────────────────────┘

Why it works:
  • All operations share a SINGLE magnitude parameter
  • Grid search over (N, M) is trivial
  • Comparable performance to AutoAugment
  • No expensive search process!
  
Typical settings:
  • CIFAR-10:  N=2, M=14
  • ImageNet:  N=2, M=9
  • Large models: N=2, M=15 (more augmentation for bigger models)
```

### TrivialAugment

```
Even simpler — ONE random augmentation per image:

TrivialAugment:
  1. Randomly pick one operation from pool
  2. Randomly pick magnitude (uniform distribution)
  3. Apply
  
  That's it! No hyperparameters at all.
  Surprisingly competitive with AutoAugment and RandAugment.
```

---

## 7. torchvision.transforms v2

### Standard Training Pipeline

```python
from torchvision import transforms

# Standard ImageNet training augmentation
train_transform = transforms.Compose([
    transforms.RandomResizedCrop(224, scale=(0.08, 1.0)),
    transforms.RandomHorizontalFlip(p=0.5),
    transforms.ColorJitter(brightness=0.4, contrast=0.4,
                           saturation=0.4, hue=0.1),
    transforms.RandAugment(num_ops=2, magnitude=9),
    transforms.ToTensor(),
    transforms.Normalize(mean=[0.485, 0.456, 0.406],
                         std=[0.229, 0.224, 0.225]),
    transforms.RandomErasing(p=0.25),
])

# Standard validation/test (NO random augmentation)
val_transform = transforms.Compose([
    transforms.Resize(256),
    transforms.CenterCrop(224),
    transforms.ToTensor(),
    transforms.Normalize(mean=[0.485, 0.456, 0.406],
                         std=[0.229, 0.224, 0.225]),
])
```

### torchvision.transforms v2 (Modern API)

```python
from torchvision.transforms import v2

train_transform = v2.Compose([
    v2.RandomResizedCrop(224, scale=(0.08, 1.0), antialias=True),
    v2.RandomHorizontalFlip(p=0.5),
    v2.ColorJitter(brightness=0.4, contrast=0.4,
                   saturation=0.4, hue=0.1),
    v2.RandAugment(num_ops=2, magnitude=9),
    v2.ToDtype(torch.float32, scale=True),  # replaces ToTensor
    v2.Normalize(mean=[0.485, 0.456, 0.406],
                 std=[0.229, 0.224, 0.225]),
    v2.RandomErasing(p=0.25),
])

# v2 advantages:
# • Supports bounding boxes, masks, keypoints (for detection/segmentation)
# • Better performance
# • More consistent API
```

### Mixup / CutMix Implementation

```python
import torch
from torchvision.transforms import v2

# These are applied at BATCH level, not image level
mixup = v2.MixUp(alpha=1.0, num_classes=1000)
cutmix = v2.CutMix(alpha=1.0, num_classes=1000)

# Randomly apply one or the other
mix_cutmix = v2.RandomChoice([mixup, cutmix])

# In training loop:
for images, labels in train_loader:
    images, labels = mix_cutmix(images, labels)
    # labels are now soft labels (one-hot × mixing coefficients)
    outputs = model(images)
    loss = F.cross_entropy(outputs, labels)  # works with soft labels
```

---

## 8. Albumentations Library

Albumentations is a fast augmentation library with a rich set of transforms:

```python
import albumentations as A
from albumentations.pytorch import ToTensorV2

# Powerful augmentation pipeline
train_transform = A.Compose([
    A.RandomResizedCrop(224, 224, scale=(0.08, 1.0)),
    A.HorizontalFlip(p=0.5),
    A.ShiftScaleRotate(shift_limit=0.1, scale_limit=0.2,
                        rotate_limit=15, p=0.5),
    A.OneOf([
        A.GaussNoise(var_limit=(10, 50)),
        A.GaussianBlur(blur_limit=(3, 7)),
        A.MotionBlur(blur_limit=(3, 7)),
    ], p=0.3),
    A.OneOf([
        A.OpticalDistortion(distort_limit=0.3),
        A.GridDistortion(distort_limit=0.3),
        A.ElasticTransform(alpha=1),
    ], p=0.2),
    A.ColorJitter(brightness=0.3, contrast=0.3,
                  saturation=0.3, hue=0.05, p=0.5),
    A.CoarseDropout(max_holes=8, max_height=32, max_width=32,
                     fill_value=0, p=0.3),  # CutOut-like
    A.Normalize(mean=[0.485, 0.456, 0.406],
                std=[0.229, 0.224, 0.225]),
    ToTensorV2(),
])

# Usage in dataset
class AugmentedDataset(torch.utils.data.Dataset):
    def __init__(self, images, labels, transform=None):
        self.images = images
        self.labels = labels
        self.transform = transform

    def __getitem__(self, idx):
        image = self.images[idx]  # numpy array (H, W, C)
        label = self.labels[idx]
        if self.transform:
            augmented = self.transform(image=image)
            image = augmented['image']
        return image, label

    def __len__(self):
        return len(self.images)
```

### Albumentations for Detection/Segmentation

```python
# Transforms that handle bboxes and masks
transform = A.Compose([
    A.RandomResizedCrop(512, 512, scale=(0.5, 1.0)),
    A.HorizontalFlip(p=0.5),
    A.RandomBrightnessContrast(p=0.3),
    A.Normalize(),
    ToTensorV2(),
], bbox_params=A.BboxParams(
    format='pascal_voc',  # or 'coco', 'yolo'
    min_area=100,
    min_visibility=0.3,
))

# Bboxes are automatically transformed with the image!
result = transform(image=image, bboxes=bboxes, labels=bbox_labels)
```

---

## 9. Test-Time Augmentation (TTA)

```
TTA: Apply augmentations at INFERENCE time and average predictions

Standard inference:        TTA inference:
  Image → Model → Pred      Original    → Model → Pred₁ ─┐
                             H-flipped   → Model → Pred₂  │
                             +5° rotated → Model → Pred₃  ├─► Average → Final
                             Crop-TL     → Model → Pred₄  │
                             Crop-BR     → Model → Pred₅ ─┘

Common TTA strategies:
  1. No augmentation (baseline)
  2. + Horizontal flip (2× inference)
  3. + Multi-crop (5-crop or 10-crop) (5-10× inference)
  4. + Multi-scale (resize to multiple sizes)
  
Typical accuracy boost: +0.3% to +1.5%
Cost: 2-10× more inference time
```

### TTA Implementation

```python
import torch
import torch.nn.functional as F
from torchvision import transforms

def tta_predict(model, image, num_classes=1000):
    """Test-Time Augmentation with horizontal flip + multi-crop."""
    model.eval()
    predictions = []

    # Augmented versions
    tta_transforms = [
        transforms.Compose([transforms.Resize(256),
                            transforms.CenterCrop(224)]),
        transforms.Compose([transforms.Resize(256),
                            transforms.CenterCrop(224),
                            transforms.RandomHorizontalFlip(p=1.0)]),
        transforms.Compose([transforms.Resize(256),
                            transforms.FiveCrop(224)]),
    ]

    normalize = transforms.Compose([
        transforms.ToTensor(),
        transforms.Normalize([0.485, 0.456, 0.406],
                             [0.229, 0.224, 0.225]),
    ])

    with torch.no_grad():
        # Center crop + flipped
        for t in tta_transforms[:2]:
            augmented = t(image)
            input_tensor = normalize(augmented).unsqueeze(0)
            pred = F.softmax(model(input_tensor), dim=1)
            predictions.append(pred)

        # Five crops
        crops = tta_transforms[2](image)
        for crop in crops:
            input_tensor = normalize(crop).unsqueeze(0)
            pred = F.softmax(model(input_tensor), dim=1)
            predictions.append(pred)

    # Average all predictions
    avg_pred = torch.stack(predictions).mean(dim=0)
    return avg_pred
```

---

## 10. Worked Examples

### Example 1: Augmentation Effect on Training

```
CIFAR-10, ResNet-18, 200 epochs:

┌────────────────────────────────────┬──────────┬──────────┐
│ Augmentation                       │ Train(%) │ Test(%)  │
├────────────────────────────────────┼──────────┼──────────┤
│ None (only normalize)              │  100.0   │  91.2    │
│ + Random H-Flip                    │   99.8   │  93.1    │
│ + Random Crop (padding=4)          │   99.2   │  94.5    │
│ + Color Jitter                     │   98.5   │  94.9    │
│ + CutOut                           │   97.8   │  95.3    │
│ + RandAugment(N=2, M=14)          │   96.2   │  96.1    │
│ + Mixup + CutMix                   │   93.1   │  96.5    │
└────────────────────────────────────┴──────────┴──────────┘

Observation: 
  • Train accuracy DECREASES with more augmentation (harder task)
  • Test accuracy INCREASES (better generalization)
  • 5.3% improvement from no augmentation to full augmentation
```

### Example 2: Mixup Computation

```
Given: image_a (cat, label=[1,0,0]), image_b (dog, label=[0,1,0])
       λ ~ Beta(0.2, 0.2) → sampled λ = 0.7

Mixed image: x̃ = 0.7 · image_a + 0.3 · image_b
Mixed label: ỹ = 0.7 · [1,0,0] + 0.3 · [0,1,0] = [0.7, 0.3, 0.0]

Loss: CE(model(x̃), ỹ) = -Σᵢ ỹᵢ log(p̂ᵢ)
     = -0.7·log(p̂_cat) - 0.3·log(p̂_dog) - 0.0·log(p̂_other)
     
Equivalent to: 0.7·CE(model(x̃), cat) + 0.3·CE(model(x̃), dog)
```

---

## 11. Summary Table

| Technique | Category | Key Parameters | Typical Settings |
|---|---|---|---|
| **RandomHorizontalFlip** | Geometric | p | p=0.5 |
| **RandomResizedCrop** | Geometric | size, scale | 224, (0.08, 1.0) |
| **RandomRotation** | Geometric | degrees | (-15, 15) |
| **ColorJitter** | Photometric | B, C, S, H | 0.4, 0.4, 0.4, 0.1 |
| **Normalize** | Preprocessing | mean, std | ImageNet stats |
| **GaussianBlur** | Noise | kernel_size | (3, 7) |
| **Mixup** | Mixing | α | α=0.2 or 1.0 |
| **CutMix** | Mixing | α | α=1.0 |
| **CutOut/RandomErasing** | Erasing | p, scale | p=0.25, (0.02, 0.33) |
| **RandAugment** | Automated | N, M | N=2, M=9 |
| **AutoAugment** | Automated | policy | per-dataset |
| **TTA** | Inference | strategies | flip + multi-crop |

---

## 12. Revision Questions

1. **Why is data augmentation an effective regularizer?** Explain using the bias-variance trade-off framework. Does augmentation change the model capacity?

2. **Compare Mixup, CutMix, and CutOut.** How do they differ in how they modify images and labels? Which one is generally best for ImageNet classification and why?

3. **Explain RandAugment** and why it's preferred over AutoAugment. What are its two hyperparameters? How would you tune them?

4. **Write a complete PyTorch augmentation pipeline** for training a ResNet-50 on a custom 10-class dataset with 224×224 images. Include geometric, color, erasing, and normalization transforms.

5. **What is Test-Time Augmentation (TTA)?** Describe a TTA strategy using horizontal flip and 5-crop. How much accuracy improvement can you typically expect? When is TTA worth the extra computation?

6. **When should you NOT use certain augmentations?** Give three examples of task-augmentation mismatches (e.g., vertical flip for satellite images, color jitter for skin disease classification).

---

## 13. Navigation

| Previous | Up | Next |
|---|---|---|
| [← Architecture Search](../06-Modern-CNN-Architectures/06-architecture-search.md) | [Unit 7 Index](../07-Training-CNNs/) | [Transfer Learning →](./02-transfer-learning.md) |

---

> **References:**  
> - Zhang, H. et al. (2018). *mixup: Beyond Empirical Risk Minimization.* ICLR.  
> - Yun, S. et al. (2019). *CutMix: Regularization Strategy to Train Strong Classifiers.* ICCV.  
> - Cubuk, E. et al. (2020). *RandAugment: Practical Automated Data Augmentation with a Reduced Search Space.* NeurIPS.  
> - Müller, S. & Hutter, F. (2021). *TrivialAugment: Tuning-free Yet State-of-the-Art Data Augmentation.* ICCV.
