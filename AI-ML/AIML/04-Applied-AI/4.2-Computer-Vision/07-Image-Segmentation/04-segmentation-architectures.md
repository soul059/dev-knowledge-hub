# Segmentation Architectures: FCN, U-Net, DeepLab

## Overview

This chapter provides a deeper architectural comparison of the three most influential segmentation models. Each introduced fundamental ideas that shaped all modern segmentation networks.

---

## FCN (Fully Convolutional Network)

```
Long, Shelhamer, Darrell (2015) — The First Deep Segmentation Model

  Key idea: Convert classification CNN to dense prediction

  VGG-16 classifier:
    Input → [Conv layers] → 7×7×512 → FC(4096) → FC(4096) → FC(1000)
    
  FCN: Replace FC with 1×1 convolutions
    Input → [Conv layers] → 7×7×512 → 1×1×4096 → 1×1×4096 → 1×1×C
    
  Then upsample back to input resolution:

  FCN-32s (coarse):
    Pool5 (7×7) → 1×1 conv → 32× upsample → output
    
  FCN-16s (medium):
    Pool5 → 1×1 conv → 2× up ──┐
    Pool4 (14×14) → 1×1 conv ──┼→ fuse → 16× up → output

  FCN-8s (fine):
    Pool5 → 1×1 conv → 2× up ──┐
    Pool4 → 1×1 conv ───────────┼→ fuse → 2× up ──┐
    Pool3 (28×28) → 1×1 conv ──────────────────────┼→ fuse → 8× up → output

  Impact of skip connections:
  
  FCN-32s: ██████████░░░░░░  Blocky, coarse boundaries
  FCN-16s: █████████████░░░  Better boundaries
  FCN-8s:  ██████████████░░  Good boundary detail
  GT:      ████████████████  Ground truth
```

---

## U-Net

```
Ronneberger et al. (2015) — Designed for Medical Image Segmentation

  Symmetric encoder-decoder with skip connections:

         Encoder (contracting)              Decoder (expanding)
  ┌────────────────────────────────────────────────────────────┐
  │                                                            │
  │  Input: 572×572×1                        Output: 388×388×2 │
  │  ┌──────────┐                          ┌──────────┐       │
  │  │conv 64×2 │══════════copy═══════════▶│conv 64×2 │       │
  │  │568×568×64│                          │392×392×64│       │
  │  └────┬─────┘                          └────▲─────┘       │
  │       │ maxpool                              │ up-conv     │
  │  ┌────▼─────┐                          ┌────┴─────┐       │
  │  │conv 128×2│══════════copy═══════════▶│conv 128×2│       │
  │  │280×280   │                          │200×200   │       │
  │  └────┬─────┘                          └────▲─────┘       │
  │       │ maxpool                              │ up-conv     │
  │  ┌────▼─────┐                          ┌────┴─────┐       │
  │  │conv 256×2│══════════copy═══════════▶│conv 256×2│       │
  │  │136×136   │                          │104×104   │       │
  │  └────┬─────┘                          └────▲─────┘       │
  │       │ maxpool                              │ up-conv     │
  │  ┌────▼─────┐                          ┌────┴─────┐       │
  │  │conv 512×2│══════════copy═══════════▶│conv 512×2│       │
  │  │ 64×64    │                          │ 56×56    │       │
  │  └────┬─────┘                          └────▲─────┘       │
  │       │ maxpool                              │ up-conv     │
  │  ┌────▼─────┐                          ┌────┘             │
  │  │conv1024×2│──────────────────────────┘                   │
  │  │ 32×32    │  Bottleneck                                  │
  │  └──────────┘                                              │
  └────────────────────────────────────────────────────────────┘

  Skip connections: CONCATENATE encoder features to decoder
    (not add — this preserves more information)
  
  Why U-Net works so well for medical imaging:
    1. Works with very few training images (data augmentation)
    2. Preserves fine boundaries via skip connections
    3. Tile-based prediction for arbitrary image sizes
    4. Weighted loss at boundaries between touching objects
```

---

## DeepLab Family

```
Chen et al. (2015-2018) — Atrous Convolutions + CRF/ASPP

  DeepLabv1 (2015):
    Modified VGG + atrous convolutions + CRF post-processing
    Key: First to use dilated convolutions for segmentation

  DeepLabv2 (2017):
    ResNet backbone + ASPP (Atrous Spatial Pyramid Pooling)
    
    ASPP: Parallel atrous convolutions at different rates
    ┌──── rate=6  ──── 3×3 conv ────┐
    │                               │
    │──── rate=12 ──── 3×3 conv ────┼──► concat ──► 1×1 conv
    │                               │
    └──── rate=18 ──── 3×3 conv ────┘
    
    Each rate captures context at a different scale

  DeepLabv3 (2017):
    Improved ASPP + batch normalization + image-level features
    Added global average pooling branch to ASPP
    Removed CRF (no longer needed with better features)

  DeepLabv3+ (2018):
    Added decoder module (like U-Net's decoder)
    
    ┌─────────────────────────────────────────┐
    │                                         │
    │  Backbone → ASPP → 1×1 conv            │
    │                      │                  │
    │                    4× up               │
    │                      │                  │
    │  Low-level features ─┼─► concat        │
    │  (from backbone)     │                  │
    │                    3×3 conv             │
    │                      │                  │
    │                    4× up → output       │
    │                                         │
    └─────────────────────────────────────────┘
    
    Simple but effective decoder!
```

---

## Architecture Comparison

| Feature | FCN | U-Net | DeepLabv3+ |
|---------|:---:|:---:|:---:|
| Year | 2015 | 2015 | 2018 |
| Backbone | VGG | Custom | ResNet/Xception |
| Skip connections | Addition | Concatenation | Addition |
| Decoder | Simple upsampling | Symmetric decoder | Light decoder |
| Receptive field | Standard conv | Standard conv | Atrous conv |
| Multi-scale | Skip fusions | Skip fusions | ASPP |
| Post-processing | — | — | — (v1-v2: CRF) |
| Parameters | 134M | 31M | 41M |
| Best for | General | Medical, small datasets | Large-scale |
| mIoU (VOC) | 62.2 | — | 89.0 |

---

## U-Net Variants

```
U-Net++  (2018): Dense skip connections (nested U-Nets)
                 Better feature fusion at multiple depths

Attention U-Net (2018): Attention gates on skip connections
                        Focus on relevant features, suppress noise

V-Net (2016): 3D U-Net for volumetric medical data
              Uses dice loss instead of cross-entropy

nnU-Net (2021): Self-configuring U-Net
                Automatically chooses architecture, preprocessing, training
                State-of-the-art on many medical benchmarks
```

---

## Revision Questions

1. **How did FCN convert classification networks into segmentation networks?**
2. **What is the difference between FCN-8s, FCN-16s, and FCN-32s?**
3. **Why does U-Net use concatenation instead of addition for skip connections?**
4. **How does ASPP capture multi-scale context?**
5. **What makes DeepLabv3+ better than DeepLabv3?**
6. **Why is U-Net particularly effective for medical image segmentation?**

---

[Previous: 03-panoptic-segmentation.md](03-panoptic-segmentation.md) | [Next: 05-mask-rcnn.md](05-mask-rcnn.md)
