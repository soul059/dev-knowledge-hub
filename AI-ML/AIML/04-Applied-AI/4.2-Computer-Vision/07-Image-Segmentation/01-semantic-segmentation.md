# Semantic Segmentation

## Overview

**Semantic segmentation** assigns a class label to every pixel in an image. Unlike object detection (which draws boxes), segmentation produces pixel-level understanding. Every pixel is classified as one of the predefined categories (e.g., road, car, person, sky), but different instances of the same class are not distinguished.

---

## Concept

```
Input Image              Semantic Segmentation Output
┌──────────────────┐     ┌──────────────────┐
│  🌤️ sky          │     │  SSSSSSSSSSSSSSS │  S = Sky
│                  │     │  SSSSSSSSSSSSSSS │  T = Tree
│  🌳  🏠  🌳     │     │  T HHHH T TTTTT │  H = House
│      ┌──┐       │     │  T HHHH T TTTTT │  R = Road
│ ═════╧══╧═══════│     │  RRRRRRRRRRRRRR │  C = Car
│   🚗             │     │  RCCRRRRRRRRRRR │  P = Person
│        🚶       │     │  RRRRRRRRPRRRRR │
└──────────────────┘     └──────────────────┘

Key: EVERY pixel gets a label. No distinction between individual objects.
     Two people → both labeled "person" (same color/label)
```

---

## Architecture: Encoder-Decoder

```
Most semantic segmentation models follow encoder-decoder:

  Input (H×W×3)
       │
  ┌────▼────┐
  │ Encoder  │  (ResNet, VGG, etc.)
  │ ↓↓↓↓↓   │  Spatial resolution decreases
  │ Features │  Channel depth increases
  └────┬────┘
       │  (H/32 × W/32 × 2048)  — bottleneck
  ┌────▼────┐
  │ Decoder  │  Spatial resolution increases back
  │ ↑↑↑↑↑   │  Upsampling + convolutions
  │         │
  └────┬────┘
       │
  Output (H×W×C)   C = number of classes
  Per-pixel class probabilities
```

---

## Key Architectures

### FCN (Fully Convolutional Network, 2015)

```
First deep learning approach for segmentation

  Replace FC layers in classification CNN with 1×1 convolutions:
  
  VGG-16:  ... → 7×7×512 → FC 4096 → FC 4096 → FC 1000
  FCN:     ... → 7×7×512 → 1×1×4096 → 1×1×4096 → 1×1×C

  Then upsample back to original resolution:
  
  FCN-32s: Single 32× upsampling (coarse)
  FCN-16s: Fuse pool4 + 2× upsample, then 16× up (better)
  FCN-8s:  Fuse pool3 + pool4 + 2× upsample (finest)
  
  Skip connections improve boundary detail!
```

### U-Net (2015)

```
Designed for biomedical image segmentation
  
  Encoder                     Decoder
  ┌────────────┐             ┌────────────┐
  │ 572×572×1  │─────copy────│ 388×388×128│
  │ conv, pool │             │ upconv,cat │
  ├────────────┤             ├────────────┤
  │ 284×284×64 │─────copy────│ 196×196×256│
  │ conv, pool │             │ upconv,cat │
  ├────────────┤             ├────────────┤
  │ 140×140×128│─────copy────│ 100×100×512│
  │ conv, pool │             │ upconv,cat │
  ├────────────┤             ├────────────┤
  │ 68×68×256  │─────copy────│  52×52×1024│
  │ conv, pool │             │ upconv,cat │
  ├────────────┤             └──────┬─────┘
  │ 32×32×512  │────────────────────┘
  │ bottleneck │
  └────────────┘

  Key: Skip connections concatenate encoder features with decoder
       → Preserves fine-grained spatial information
```

### DeepLab (v1-v3+)

```
Innovations:
  1. Atrous (Dilated) Convolutions — larger receptive field without pooling
  
     Standard 3×3:        Atrous 3×3 (rate=2):
     ┌───┬───┬───┐       ┌───┬───┬───┬───┬───┐
     │ × │ × │ × │       │ × │   │ × │   │ × │
     ├───┼───┼───┤       ├───┼───┼───┼───┼───┤
     │ × │ × │ × │       │   │   │   │   │   │
     ├───┼───┼───┤       ├───┼───┼───┼───┼───┤
     │ × │ × │ × │       │ × │   │ × │   │ × │
     └───┴───┴───┘       ├───┼───┼───┼───┼───┤
     receptive: 3×3       │   │   │   │   │   │
                          ├───┼───┼───┼───┼───┤
                          │ × │   │ × │   │ × │
                          └───┴───┴───┴───┴───┘
                          receptive: 5×5 (same params!)

  2. ASPP (Atrous Spatial Pyramid Pooling)
     Multiple atrous convolutions at different rates → capture multi-scale context
     
     Feature map → rate=1  → ─┐
                 → rate=6  → ─┤
                 → rate=12 → ─┼→ concatenate → 1×1 conv → output
                 → rate=18 → ─┤
                 → pool    → ─┘

  3. DeepLabv3+: Encoder-decoder with ASPP
```

---

## Training Details

```
Loss: Cross-entropy per pixel
  L = -(1/N) Σ_pixels Σ_classes y_c × log(p_c)
  
  N = total pixels, y_c = GT one-hot, p_c = predicted probability

Class imbalance handling:
  - Weighted cross-entropy (higher weight for rare classes)
  - Dice loss: L = 1 - (2|P∩G| / (|P|+|G|))
  - Focal loss for hard pixels

Data augmentation:
  - Random crop, flip, scale
  - Color jitter
  - CutOut / CutMix
```

---

## Python: Semantic Segmentation

```python
import torch
import torchvision.models.segmentation as seg

# Pre-trained DeepLabv3+ with ResNet-101
model = seg.deeplabv3_resnet101(pretrained=True)
model.eval()

# Inference
from PIL import Image
from torchvision import transforms

img = Image.open("street.jpg")
preprocess = transforms.Compose([
    transforms.ToTensor(),
    transforms.Normalize(mean=[0.485, 0.456, 0.406],
                         std=[0.229, 0.224, 0.225])
])
input_tensor = preprocess(img).unsqueeze(0)

with torch.no_grad():
    output = model(input_tensor)["out"]
    pred = output.argmax(1).squeeze().numpy()

# Visualize — each class gets a unique color
import matplotlib.pyplot as plt
plt.imshow(pred, cmap="tab20")
plt.title("Semantic Segmentation")
plt.show()
```

---

## Real-World Applications

| Application | Details |
|-------------|---------|
| Autonomous driving | Road, lane, vehicle, pedestrian segmentation |
| Medical imaging | Organ/tumor segmentation from CT/MRI |
| Satellite imagery | Land use, crop, water body mapping |
| Augmented reality | Background removal, scene understanding |
| Robotics | Navigation, obstacle avoidance |

---

## Revision Questions

1. **How does semantic segmentation differ from object detection?**
2. **What is the encoder-decoder architecture and why is it needed?**
3. **How do skip connections in U-Net improve segmentation quality?**
4. **What are atrous convolutions and why are they useful?**
5. **Why is class imbalance a bigger problem in segmentation than classification?**

---

[Next: 02-instance-segmentation.md](02-instance-segmentation.md)
