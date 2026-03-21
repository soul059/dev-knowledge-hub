# Medical Imaging with CNNs — X-ray, CT, MRI Analysis

> **Unit 8 · CNN Applications · Topic 6/6**

---

## Table of Contents

1. [Overview & Modalities](#1-overview--modalities)
2. [U-Net for Medical Segmentation](#2-u-net-for-medical-segmentation)
3. [Class Imbalance Challenges](#3-class-imbalance-challenges)
4. [Limited Labeled Data](#4-limited-labeled-data)
5. [Transfer Learning from ImageNet](#5-transfer-learning-from-imagenet)
6. [3D Convolutions for Volumetric Data](#6-3d-convolutions-for-volumetric-data)
7. [Ethical Considerations](#7-ethical-considerations)
8. [Worked Examples](#8-worked-examples)
9. [PyTorch Implementation](#9-pytorch-implementation)
10. [Summary Table](#10-summary-table)
11. [Revision Questions](#11-revision-questions)
12. [Navigation](#12-navigation)

---

## 1. Overview & Modalities

```
Medical Imaging Modalities:

┌──────────────┬───────────────┬──────────────────────────────────┐
│ Modality     │ Data Format   │ Common Tasks                     │
├──────────────┼───────────────┼──────────────────────────────────┤
│ X-ray        │ 2D grayscale  │ Lung disease, fracture detection │
│ CT scan      │ 3D volume     │ Organ segmentation, tumor detect│
│ MRI          │ 3D volume     │ Brain tumor, cardiac analysis   │
│ Ultrasound   │ 2D/3D video   │ Fetal imaging, cardiac          │
│ Histopathology│ 2D RGB (huge)│ Cancer grading, cell counting  │
│ Retinal      │ 2D RGB        │ Diabetic retinopathy            │
│ Dermoscopy   │ 2D RGB        │ Skin lesion classification      │
└──────────────┴───────────────┴──────────────────────────────────┘

Key Differences from Natural Images:
  ✗ Much LESS data (100s-1000s vs millions)
  ✗ Expert annotation required (expensive, slow)
  ✗ Class imbalance (most pixels are healthy)
  ✗ 3D data (CT, MRI volumes)
  ✗ Different value ranges (Hounsfield units, signal intensity)
  ✓ More structured (anatomy is consistent)
  ✓ Less variation (controlled imaging conditions)
```

---

## 2. U-Net for Medical Segmentation

U-Net was designed specifically for medical image segmentation and remains the dominant architecture.

```
Why U-Net Dominates Medical Imaging:

1. SKIP CONNECTIONS: Precise boundary delineation
   Critical for: tumor boundaries, organ borders

2. WORKS WITH SMALL DATASETS:
   Original paper: trained on 30 images (with augmentation!)

3. SYMMETRIC ARCHITECTURE:
   Encoder-decoder ensures output resolution = input resolution

4. ELASTIC DEFORMATION augmentation:
   Simulates anatomical variability

U-Net Variants for Medical Imaging:
┌──────────────────┬─────────────────────────────────────────┐
│ Variant          │ Key Modification                         │
├──────────────────┼─────────────────────────────────────────┤
│ U-Net            │ Original 2D architecture                │
│ 3D U-Net         │ 3D convolutions for volumetric data     │
│ V-Net            │ 3D + volumetric dice loss + residuals   │
│ Attention U-Net  │ Attention gates in skip connections     │
│ U-Net++          │ Nested skip connections (dense skip)    │
│ nnU-Net          │ Self-configuring U-Net (auto hyperparams)│
│ TransUNet        │ Transformer encoder + U-Net decoder     │
│ Swin-UNETR       │ Swin Transformer + U-Net for 3D        │
└──────────────────┴─────────────────────────────────────────┘

nnU-Net: State-of-the-art framework
  • Automatically determines: architecture, preprocessing,
    augmentation, training schedule
  • Won most medical segmentation challenges (2018-2023)
  • No manual hyperparameter tuning needed!
```

---

## 3. Class Imbalance Challenges

```
Typical class distribution in medical images:

  Brain MRI tumor segmentation:
    Background: ~98%
    Tumor:       ~2%
    
  Retinal vessel segmentation:
    Background: ~88%
    Vessel:     ~12%
    
  Cell detection in histopathology:
    Background: ~95%
    Cells:       ~5%

Problems:
  • Model predicts "all background" → 98% accuracy but useless!
  • Standard CE loss dominated by background gradients
  • Rare class features get drowned out

Solutions:
┌─────────────────────────────────────────────────────────┐
│ 1. WEIGHTED CROSS-ENTROPY                               │
│    w_c = 1 / frequency_c  (or inverse sqrt)            │
│    L = -Σ_c w_c · y_c · log(p_c)                      │
│                                                         │
│ 2. DICE LOSS (naturally handles imbalance)              │
│    L = 1 - 2|A∩B| / (|A|+|B|)                         │
│                                                         │
│ 3. FOCAL LOSS (down-weight easy examples)               │
│    L = -α(1-p)^γ · log(p)                              │
│                                                         │
│ 4. COMBINED LOSS                                        │
│    L = CE + Dice  (most common in practice)            │
│                                                         │
│ 5. OVERSAMPLING positive patches                        │
│    Sample patches centered on lesions more often        │
│                                                         │
│ 6. HARD EXAMPLE MINING                                  │
│    Focus training on misclassified regions              │
└─────────────────────────────────────────────────────────┘
```

---

## 4. Limited Labeled Data

```
Label Scarcity in Medical Imaging:

  ImageNet:        1.2M labeled images
  ChestX-ray14:    ~112K images, noisy labels from NLP
  Brain Tumor (BraTS): ~2000 annotated MRI volumes
  Cell Segmentation:   ~50-500 annotated images typical

Strategies for Limited Data:

1. DATA AUGMENTATION (aggressive)
   ┌───────────────────────────────────────────────┐
   │ Standard: flip, rotate, scale, crop            │
   │ Medical-specific:                               │
   │   • Elastic deformation (simulate tissue)      │
   │   • Intensity augmentation (simulate scanners) │
   │   • Gamma correction                           │
   │   • Additive noise (simulate acquisition noise)│
   │   • Simulated artifacts (motion, aliasing)     │
   └───────────────────────────────────────────────┘

2. SELF-SUPERVISED PRETRAINING
   • Train on unlabeled medical images first
   • Contrastive learning (MoCo, SimCLR on medical images)
   • Masked image modeling (MAE on CT/MRI)

3. SEMI-SUPERVISED LEARNING
   • Use small labeled + large unlabeled dataset
   • Pseudo-labeling, consistency regularization
   • Mean teacher framework

4. ACTIVE LEARNING
   • Model identifies most uncertain samples
   • Expert labels those → most efficient use of annotation budget

5. FEW-SHOT / ZERO-SHOT
   • Meta-learning across tasks
   • Segment anything (SAM) adapted for medical imaging
```

---

## 5. Transfer Learning from ImageNet

```
Surprising finding: ImageNet pretraining helps for medical imaging!

Despite domain gap (natural photos vs X-rays):
  • Early filters (edges, textures) ARE universal
  • Even grayscale medical images benefit
  • Pretrained > random initialization consistently

How to use ImageNet features for medical images:

1. GRAYSCALE IMAGES (X-ray, CT slices):
   • Replicate single channel to 3 channels
   • Or modify first conv layer to accept 1 channel
   
   Option A: image_3ch = image.repeat(1, 3, 1, 1)
   Option B: model.conv1 = nn.Conv2d(1, 64, 7, 2, 3)
             # Initialize: new_weight = old_weight.mean(dim=1, keepdim=True)

2. DIFFERENT RESOLUTION:
   • Medical images often larger (512×512, 1024×1024)
   • Resize to 224/384 or use patch-based approach
   
3. VALUE RANGE:
   • CT: Hounsfield units [-1000, 3000]
   • MRI: arbitrary intensity scale
   • Must normalize appropriately (not just ImageNet stats)
   • Window/level for CT: e.g., lung window [-1200, 600]

Performance comparison:
┌────────────────────────────┬──────────┬──────────┐
│ Setting                    │ Acc/Dice │ Notes    │
├────────────────────────────┼──────────┼──────────┤
│ Random init + small data   │  65-75%  │ Baseline │
│ ImageNet pretrained        │  78-88%  │ +10-15%  │
│ Medical pretrained (self)  │  80-90%  │ Best     │
└────────────────────────────┴──────────┴──────────┘
```

---

## 6. 3D Convolutions for Volumetric Data

```
CT/MRI are 3D volumes: (D × H × W) with D slices

Approach 1: 2D SLICE-BY-SLICE
  Process each slice independently with 2D CNN
  + Simple, can use pretrained 2D models
  - Ignores inter-slice context (axial, sagittal, coronal)

Approach 2: 2.5D (MULTI-SLICE)
  Stack 3 adjacent slices as 3-channel input (like RGB)
  + Some 3D context, still uses 2D architecture
  + Can use ImageNet pretrained models!

Approach 3: FULL 3D CNN
  Use 3D convolutions: nn.Conv3d(in_ch, out_ch, kernel=(3,3,3))
  + Full volumetric context
  - Much more memory and compute
  - Fewer pretrained models available
  
  3D Convolution:
    Input:  (B, C, D, H, W)
    Kernel: (C_out, C_in, kD, kH, kW)
    Output: (B, C_out, D', H', W')

Approach 4: PATCH-BASED 3D
  Extract 3D patches (e.g., 128×128×128) from volume
  Process each patch with 3D U-Net
  Stitch predictions back together
  
  ┌────────────────────────┐
  │ Full CT volume         │
  │ 512 × 512 × 300       │
  │   ┌──────┐             │
  │   │Patch │ 128³        │
  │   │      │             │
  │   └──────┘             │
  │        ┌──────┐        │
  │        │Patch │ (overlap)
  │        └──────┘        │
  └────────────────────────┘
  
  Patches overlap (e.g., 50%) to avoid boundary artifacts
```

### 3D U-Net Architecture

```
3D U-Net: Replace all 2D operations with 3D equivalents

  Conv2d(in, out, 3)     →  Conv3d(in, out, 3)
  MaxPool2d(2)           →  MaxPool3d(2)
  ConvTranspose2d(2)     →  ConvTranspose3d(2)
  BatchNorm2d(out)       →  BatchNorm3d(out)

Memory constraint: 3D volumes are HUGE
  128×128×128 × 64 channels × 4 bytes = 128 MB per feature map!
  
Solutions:
  • Smaller patches (64³ or 96³)
  • Fewer channels (start with 16-32 instead of 64)
  • Mixed precision training (essential!)
  • Gradient checkpointing
```

---

## 7. Ethical Considerations

```
Critical Ethical Issues in Medical AI:

1. BIAS AND FAIRNESS
   ┌─────────────────────────────────────────────────────┐
   │ • Models trained on one demographic may fail on     │
   │   others (skin color affects dermatology AI)        │
   │ • Different scanner vendors produce different       │
   │   image characteristics → domain shift              │
   │ • Underrepresented populations in training data     │
   │ Fix: Diverse datasets, stratified evaluation,       │
   │      domain adaptation, fairness constraints        │
   └─────────────────────────────────────────────────────┘

2. EXPLAINABILITY / INTERPRETABILITY
   ┌─────────────────────────────────────────────────────┐
   │ • Clinicians need to understand WHY the model       │
   │   made a prediction                                 │
   │ • "Black box" models not trusted for critical       │
   │   medical decisions                                 │
   │ Fix: Grad-CAM, attention maps, uncertainty          │
   │      estimation, human-in-the-loop                  │
   └─────────────────────────────────────────────────────┘

3. SAFETY AND VALIDATION
   ┌─────────────────────────────────────────────────────┐
   │ • False negatives (missed disease) can be fatal     │
   │ • False positives cause unnecessary procedures      │
   │ • Must validate on external, independent datasets   │
   │ • Regulatory approval required (FDA, CE marking)    │
   │ Fix: Extensive clinical trials, out-of-distribution │
   │      detection, confidence calibration              │
   └─────────────────────────────────────────────────────┘

4. DATA PRIVACY
   ┌─────────────────────────────────────────────────────┐
   │ • Medical images contain identifiable information   │
   │ • HIPAA, GDPR compliance required                   │
   │ • De-identification may not be sufficient           │
   │ Fix: Federated learning, differential privacy,      │
   │      synthetic data generation                      │
   └─────────────────────────────────────────────────────┘

5. DEPLOYMENT RESPONSIBILITY
   ┌─────────────────────────────────────────────────────┐
   │ • AI should ASSIST clinicians, not replace them     │
   │ • Clear communication of model limitations          │
   │ • Continuous monitoring after deployment             │
   │ • Clear liability framework                         │
   └─────────────────────────────────────────────────────┘
```

---

## 8. Worked Examples

### Example: Handling CT Hounsfield Units

```
CT values in Hounsfield Units (HU):
  Air:    -1000 HU
  Water:      0 HU
  Bone:  +1000 HU
  Metal: +3000 HU

Windowing for different tissues:
  Lung window:   center=-600, width=1500  → range [-1350, 150]
  Soft tissue:   center=40,   width=400   → range [-160, 240]
  Bone:          center=400,  width=1800  → range [-500, 1300]

Preprocessing pipeline:
  1. Load DICOM/NIfTI → raw HU values
  2. Apply window/level:
     clipped = np.clip(hu_image, window_min, window_max)
  3. Normalize to [0, 1]:
     normalized = (clipped - window_min) / (window_max - window_min)
  4. For 2D pretrained model: replicate to 3 channels
  5. Apply ImageNet normalization (or dataset-specific)
```

---

## 9. PyTorch Implementation

### Medical Image Segmentation Pipeline

```python
import torch
import torch.nn as nn
import numpy as np

# Preprocessing for CT images
def preprocess_ct(hu_image, window_center=-600, window_width=1500):
    """Window and normalize CT image from Hounsfield Units."""
    min_val = window_center - window_width // 2
    max_val = window_center + window_width // 2
    
    image = np.clip(hu_image, min_val, max_val)
    image = (image - min_val) / (max_val - min_val)
    return image.astype(np.float32)


# Medical augmentation with albumentations
import albumentations as A
from albumentations.pytorch import ToTensorV2

medical_augmentation = A.Compose([
    A.RandomResizedCrop(256, 256, scale=(0.8, 1.0)),
    A.HorizontalFlip(p=0.5),
    A.ShiftScaleRotate(shift_limit=0.1, scale_limit=0.15,
                        rotate_limit=15, p=0.5),
    A.ElasticTransform(alpha=120, sigma=120*0.05,
                        p=0.3),  # medical-specific
    A.GridDistortion(p=0.2),
    A.RandomBrightnessContrast(brightness_limit=0.2,
                                contrast_limit=0.2, p=0.3),
    A.GaussNoise(var_limit=(10, 50), p=0.2),
    A.Normalize(mean=0.5, std=0.5),
    ToTensorV2(),
])


# Combined loss (CE + Dice)
class CombinedLoss(nn.Module):
    def __init__(self, ce_weight=1.0, dice_weight=1.0):
        super().__init__()
        self.ce = nn.CrossEntropyLoss()
        self.ce_weight = ce_weight
        self.dice_weight = dice_weight

    def dice_loss(self, pred, target, smooth=1.0):
        pred = torch.softmax(pred, dim=1)
        target_oh = torch.zeros_like(pred).scatter_(
            1, target.unsqueeze(1), 1)
        intersection = (pred * target_oh).sum(dim=(2, 3))
        union = pred.sum(dim=(2, 3)) + target_oh.sum(dim=(2, 3))
        dice = (2 * intersection + smooth) / (union + smooth)
        return 1 - dice.mean()

    def forward(self, pred, target):
        return (self.ce_weight * self.ce(pred, target) +
                self.dice_weight * self.dice_loss(pred, target))


# 3D U-Net Block
class DoubleConv3D(nn.Module):
    def __init__(self, in_ch, out_ch):
        super().__init__()
        self.conv = nn.Sequential(
            nn.Conv3d(in_ch, out_ch, 3, padding=1, bias=False),
            nn.InstanceNorm3d(out_ch),  # InstanceNorm better for small batch
            nn.LeakyReLU(0.01, inplace=True),
            nn.Conv3d(out_ch, out_ch, 3, padding=1, bias=False),
            nn.InstanceNorm3d(out_ch),
            nn.LeakyReLU(0.01, inplace=True),
        )
    def forward(self, x):
        return self.conv(x)


class UNet3D(nn.Module):
    def __init__(self, in_channels=1, num_classes=3):
        super().__init__()
        # Encoder (fewer channels due to 3D memory)
        self.enc1 = DoubleConv3D(in_channels, 32)
        self.enc2 = DoubleConv3D(32, 64)
        self.enc3 = DoubleConv3D(64, 128)
        self.pool = nn.MaxPool3d(2)

        self.bottleneck = DoubleConv3D(128, 256)

        # Decoder
        self.up3 = nn.ConvTranspose3d(256, 128, 2, stride=2)
        self.dec3 = DoubleConv3D(256, 128)
        self.up2 = nn.ConvTranspose3d(128, 64, 2, stride=2)
        self.dec2 = DoubleConv3D(128, 64)
        self.up1 = nn.ConvTranspose3d(64, 32, 2, stride=2)
        self.dec1 = DoubleConv3D(64, 32)

        self.out = nn.Conv3d(32, num_classes, 1)

    def forward(self, x):
        e1 = self.enc1(x)
        e2 = self.enc2(self.pool(e1))
        e3 = self.enc3(self.pool(e2))

        b = self.bottleneck(self.pool(e3))

        d3 = self.dec3(torch.cat([self.up3(b), e3], 1))
        d2 = self.dec2(torch.cat([self.up2(d3), e2], 1))
        d1 = self.dec1(torch.cat([self.up1(d2), e1], 1))

        return self.out(d1)

# Test
model = UNet3D(in_channels=1, num_classes=3)
x = torch.randn(1, 1, 64, 64, 64)  # 3D volume
print(model(x).shape)  # (1, 3, 64, 64, 64)
```

---

## 10. Summary Table

| Concept | Key Idea |
|---|---|
| **Medical modalities** | X-ray (2D), CT/MRI (3D), histopathology (huge 2D) |
| **U-Net** | Gold standard for medical segmentation; skip connections |
| **nnU-Net** | Self-configuring U-Net; wins most challenges |
| **Class imbalance** | Tiny lesions vs huge background; use Dice + focal loss |
| **Limited data** | Aggressive augmentation, self-supervised pretrain, semi-supervised |
| **ImageNet transfer** | Works despite domain gap; early features are universal |
| **3D convolutions** | Conv3d for CT/MRI volumes; memory-intensive |
| **Patch-based** | Process overlapping 3D patches to fit in GPU memory |
| **CT preprocessing** | Window/level for tissue type → normalize → model |
| **Ethics** | Bias, explainability, safety, privacy, clinical validation |

---

## 11. Revision Questions

1. **Why is U-Net the dominant architecture** for medical image segmentation? What properties make it suitable for small datasets?

2. **Explain the class imbalance problem** in medical imaging. Compare weighted CE, Dice loss, and Focal loss for handling it.

3. **How can ImageNet pretrained models help** with medical imaging despite the large domain gap? What modifications are needed for grayscale input?

4. **Compare 2D, 2.5D, and 3D approaches** for CT/MRI segmentation. When would you choose each?

5. **List three ethical considerations** for deploying medical AI and propose a mitigation strategy for each.

---

## 12. Navigation

| Previous | Up | Next |
|---|---|---|
| [← Face Recognition](./05-face-recognition.md) | [Unit 8 Index](../08-CNN-Applications/) | [Unit 9: Filter Visualization →](../09-Visualizing-CNNs/01-filter-visualization.md) |

---

> **References:**  
> - Ronneberger, O. et al. (2015). *U-Net: Convolutional Networks for Biomedical Image Segmentation.* MICCAI.  
> - Isensee, F. et al. (2021). *nnU-Net: A Self-Configuring Method for Deep Learning-Based Biomedical Image Segmentation.* Nature Methods.  
> - Litjens, G. et al. (2017). *A Survey on Deep Learning in Medical Image Analysis.* Medical Image Analysis.
