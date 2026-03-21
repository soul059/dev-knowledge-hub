# 4. Residual Networks (ResNet)

> **Unit 10 · Building Neural Networks** — The architecture that enabled training networks with 100+ layers

---

## Chapter Overview

**ResNet** (He et al., 2015) is one of the most influential neural network architectures ever created. By introducing **residual blocks** with skip connections, ResNet solved the degradation problem and won the ImageNet competition with a stunning 152-layer network — when the previous best had only 22 layers. ResNet's ideas have since permeated *every* area of deep learning: CNNs, Transformers, diffusion models, and beyond. This chapter covers the ResNet architecture in detail, its variants (basic block, bottleneck block, projection shortcuts), the full family (ResNet-18 through ResNet-152), PyTorch implementation, and the pre-activation variant.

---

## 1. Historical Context

```
  ImageNet Classification Winners:
  
  Year  │ Architecture   │ Depth │ Top-5 Error │ Key Innovation
  ──────┼────────────────┼───────┼─────────────┼──────────────────────
  2012  │ AlexNet        │   8   │   16.4%     │ Deep CNN + GPU training
  2013  │ ZFNet          │   8   │   14.8%     │ Visualization of CNNs
  2014  │ VGGNet         │  19   │    7.3%     │ Small 3×3 filters
  2014  │ GoogLeNet      │  22   │    6.7%     │ Inception modules
  2015  │ ResNet         │ 152   │    3.6%     │ Skip connections ← !!
  ──────┴────────────────┴───────┴─────────────┴──────────────────────
  
  ResNet went from 22 to 152 layers and REDUCED error!
  Human performance on ImageNet: ~5.1% top-5 error
  ResNet-152 beat humans: 3.6% < 5.1%
  
  Paper: "Deep Residual Learning for Image Recognition"
  Authors: Kaiming He, Xiangyu Zhang, Shaoqing Ren, Jian Sun (Microsoft)
  Citations: 190,000+ (one of the most cited CS papers EVER)
```

---

## 2. The Degradation Problem (Revisited)

```
  He et al. showed that simply stacking more layers HURTS:
  
  Training Error ↑
                 │
   10%           │                              ← 56-layer (WORSE)
                 │
    8%           │          ← 20-layer (BETTER)
                 │
    6%           │
                 └──────────────────────────→ Iterations
  
  This is NOT overfitting — the 56-layer network has higher
  TRAINING error than the 20-layer network.
  
  The problem: Optimizing deep plain networks is fundamentally hard.
  
  ResNet's solution: Make the network's default behavior = identity.
  Extra layers can only ADD information, never lose it.
  
  ┌──────────────────────────────────────────────────────────────┐
  │  With ResNet:                                                │
  │  • 34-layer ResNet beats 18-layer ResNet ✓                  │
  │  • 50-layer ResNet beats 34-layer ResNet ✓                  │
  │  • 152-layer ResNet beats 50-layer ResNet ✓                 │
  │  → Deeper IS better with skip connections!                  │
  └──────────────────────────────────────────────────────────────┘
```

---

## 3. ResNet Building Blocks

### 3.1 Basic Block (Used in ResNet-18, ResNet-34)

```
  BASIC BLOCK (2 conv layers):
  
       x (C channels, H×W)
       │
       ├──────────────────────────┐
       │                          │
       ↓                          │ identity
  ┌──────────────┐               │ shortcut
  │ Conv 3×3, C  │               │
  │ stride=1     │               │
  │ padding=1    │               │
  └──────┬───────┘               │
         ↓                        │
  ┌──────────────┐               │
  │ BatchNorm    │               │
  └──────┬───────┘               │
         ↓                        │
  ┌──────────────┐               │
  │    ReLU      │               │
  └──────┬───────┘               │
         ↓                        │
  ┌──────────────┐               │
  │ Conv 3×3, C  │               │
  │ stride=1     │               │
  │ padding=1    │               │
  └──────┬───────┘               │
         ↓                        │
  ┌──────────────┐               │
  │ BatchNorm    │               │
  └──────┬───────┘               │
         ↓                        │
         + ← ── ── ── ── ── ── ─┘   element-wise addition
         ↓
  ┌──────────────┐
  │    ReLU      │
  └──────┬───────┘
         ↓
       output
  
  Parameters: 2 × (C × C × 3 × 3) = 18C² per block
  For C=64: 18 × 4096 = 73,728 parameters
```

### 3.2 Bottleneck Block (Used in ResNet-50, 101, 152)

```
  BOTTLENECK BLOCK (3 conv layers — 1×1, 3×3, 1×1):
  
       x (4C channels, H×W)
       │
       ├──────────────────────────┐
       │                          │
       ↓                          │ identity
  ┌──────────────┐               │ shortcut
  │ Conv 1×1, C  │  "squeeze"   │
  │ stride=1     │  (reduce     │
  │ (reduce ch.) │   channels)  │
  └──────┬───────┘               │
         ↓                        │
  ┌──────────────┐               │
  │ BatchNorm    │               │
  └──────┬───────┘               │
         ↓                        │
  ┌──────────────┐               │
  │    ReLU      │               │
  └──────┬───────┘               │
         ↓                        │
  ┌──────────────┐               │
  │ Conv 3×3, C  │  "transform" │
  │ stride=1     │  (spatial     │
  │ padding=1    │   processing) │
  └──────┬───────┘               │
         ↓                        │
  ┌──────────────┐               │
  │ BatchNorm    │               │
  └──────┬───────┘               │
         ↓                        │
  ┌──────────────┐               │
  │    ReLU      │               │
  └──────┬───────┘               │
         ↓                        │
  ┌──────────────┐               │
  │ Conv 1×1, 4C │  "expand"    │
  │ stride=1     │  (restore    │
  │ (expand ch.) │   channels)  │
  └──────┬───────┘               │
         ↓                        │
  ┌──────────────┐               │
  │ BatchNorm    │               │
  └──────┬───────┘               │
         ↓                        │
         + ← ── ── ── ── ── ── ─┘
         ↓
  ┌──────────────┐
  │    ReLU      │
  └──────┬───────┘
         ↓
       output
  
  Parameters: (4C × C × 1 × 1) + (C × C × 3 × 3) + (C × 4C × 1 × 1)
            = 4C² + 9C² + 4C² = 17C²
  
  Compare to basic block: 2 × (4C × 4C × 3 × 3) = 288C²
  Bottleneck is 17× more parameter efficient for same channel width!
```

### Identity vs Projection Shortcuts

```
  IDENTITY SHORTCUT (when dimensions match):
  y = F(x) + x               ← just add, zero extra parameters!
  
  PROJECTION SHORTCUT (when dimensions change):
  y = F(x) + Wₛx             ← 1×1 conv to match dimensions
  
  When is projection needed?
  • At the start of each "stage" (spatial size halves, channels double)
  
  Stage transitions:
  ┌───────────────────────────────────────────────────────────┐
  │  Stage 1: 56×56, 64 channels   → identity shortcuts     │
  │  ↓ (stride=2)                                            │
  │  Stage 2: 28×28, 128 channels  → projection at start!   │
  │  ↓ (stride=2)                                            │
  │  Stage 3: 14×14, 256 channels  → projection at start!   │
  │  ↓ (stride=2)                                            │
  │  Stage 4: 7×7, 512 channels    → projection at start!   │
  └───────────────────────────────────────────────────────────┘
```

---

## 4. Full ResNet Architecture

### ResNet Family

```
  ┌────────────────────────────────────────────────────────────────┐
  │         │ ResNet-18 │ ResNet-34 │ ResNet-50 │ ResNet-101│ ResNet-152│
  ├─────────┼───────────┼───────────┼───────────┼───────────┼───────────┤
  │Block    │  Basic    │  Basic    │ Bottleneck│ Bottleneck│ Bottleneck│
  │type     │           │           │           │           │           │
  ├─────────┼───────────┼───────────┼───────────┼───────────┼───────────┤
  │conv1    │ 7×7, 64, stride=2                                        │
  │         │ MaxPool 3×3, stride=2                                    │
  ├─────────┼───────────┼───────────┼───────────┼───────────┼───────────┤
  │Stage 1  │ 2 blocks  │ 3 blocks  │ 3 blocks  │ 3 blocks  │ 3 blocks  │
  │(64 ch)  │ 64        │ 64        │ 64,64,256 │ 64,64,256 │ 64,64,256 │
  ├─────────┼───────────┼───────────┼───────────┼───────────┼───────────┤
  │Stage 2  │ 2 blocks  │ 4 blocks  │ 4 blocks  │ 4 blocks  │ 8 blocks  │
  │(128 ch) │ 128       │ 128       │128,128,512│128,128,512│128,128,512│
  ├─────────┼───────────┼───────────┼───────────┼───────────┼───────────┤
  │Stage 3  │ 2 blocks  │ 6 blocks  │ 6 blocks  │23 blocks  │36 blocks  │
  │(256 ch) │ 256       │ 256       │256,256,   │256,256,   │256,256,   │
  │         │           │           │1024       │1024       │1024       │
  ├─────────┼───────────┼───────────┼───────────┼───────────┼───────────┤
  │Stage 4  │ 2 blocks  │ 3 blocks  │ 3 blocks  │ 3 blocks  │ 3 blocks  │
  │(512 ch) │ 512       │ 512       │512,512,   │512,512,   │512,512,   │
  │         │           │           │2048       │2048       │2048       │
  ├─────────┼───────────┼───────────┼───────────┼───────────┼───────────┤
  │Output   │ AdaptiveAvgPool → FC(1000)                               │
  ├─────────┼───────────┼───────────┼───────────┼───────────┼───────────┤
  │Total    │ 8 blocks  │16 blocks  │16 blocks  │33 blocks  │50 blocks  │
  │blocks   │           │           │           │           │           │
  ├─────────┼───────────┼───────────┼───────────┼───────────┼───────────┤
  │Params   │  11.7M    │  21.8M    │  25.6M    │  44.5M    │  60.2M    │
  ├─────────┼───────────┼───────────┼───────────┼───────────┼───────────┤
  │Top-1    │  30.2%    │  26.7%    │  24.0%    │  22.4%    │  21.7%    │
  │error    │           │           │           │           │           │
  └─────────┴───────────┴───────────┴───────────┴───────────┴───────────┘
  
  Note: ResNet-50 has FEWER params than ResNet-34 thanks to bottleneck!
```

---

## 5. PyTorch Implementation

### Basic Block

```python
import torch
import torch.nn as nn

class BasicBlock(nn.Module):
    """ResNet Basic Block (for ResNet-18/34)."""
    expansion = 1  # output channels = input channels × expansion
    
    def __init__(self, in_channels, out_channels, stride=1, downsample=None):
        super().__init__()
        self.conv1 = nn.Conv2d(in_channels, out_channels, kernel_size=3,
                               stride=stride, padding=1, bias=False)
        self.bn1 = nn.BatchNorm2d(out_channels)
        self.relu = nn.ReLU(inplace=True)
        self.conv2 = nn.Conv2d(out_channels, out_channels, kernel_size=3,
                               stride=1, padding=1, bias=False)
        self.bn2 = nn.BatchNorm2d(out_channels)
        self.downsample = downsample  # projection shortcut (if needed)
    
    def forward(self, x):
        identity = x
        
        out = self.conv1(x)
        out = self.bn1(out)
        out = self.relu(out)
        
        out = self.conv2(out)
        out = self.bn2(out)
        
        if self.downsample is not None:
            identity = self.downsample(x)  # match dimensions
        
        out += identity  # ← THE SKIP CONNECTION
        out = self.relu(out)
        return out
```

### Bottleneck Block

```python
class Bottleneck(nn.Module):
    """ResNet Bottleneck Block (for ResNet-50/101/152)."""
    expansion = 4  # output channels = out_channels × 4
    
    def __init__(self, in_channels, out_channels, stride=1, downsample=None):
        super().__init__()
        # 1×1 conv: squeeze channels
        self.conv1 = nn.Conv2d(in_channels, out_channels, kernel_size=1, bias=False)
        self.bn1 = nn.BatchNorm2d(out_channels)
        
        # 3×3 conv: spatial processing
        self.conv2 = nn.Conv2d(out_channels, out_channels, kernel_size=3,
                               stride=stride, padding=1, bias=False)
        self.bn2 = nn.BatchNorm2d(out_channels)
        
        # 1×1 conv: expand channels
        self.conv3 = nn.Conv2d(out_channels, out_channels * self.expansion,
                               kernel_size=1, bias=False)
        self.bn3 = nn.BatchNorm2d(out_channels * self.expansion)
        
        self.relu = nn.ReLU(inplace=True)
        self.downsample = downsample
    
    def forward(self, x):
        identity = x
        
        out = self.relu(self.bn1(self.conv1(x)))    # 1×1 squeeze
        out = self.relu(self.bn2(self.conv2(out)))   # 3×3 process
        out = self.bn3(self.conv3(out))               # 1×1 expand
        
        if self.downsample is not None:
            identity = self.downsample(x)
        
        out += identity  # ← skip connection
        out = self.relu(out)
        return out
```

### Full ResNet Model

```python
class ResNet(nn.Module):
    """Full ResNet architecture."""
    
    def __init__(self, block, layers, num_classes=1000):
        """
        Args:
            block: BasicBlock or Bottleneck
            layers: list of 4 ints, blocks per stage [2,2,2,2] for ResNet-18
            num_classes: number of output classes
        """
        super().__init__()
        self.in_channels = 64
        
        # Initial convolution (7×7, stride 2)
        self.conv1 = nn.Conv2d(3, 64, kernel_size=7, stride=2, padding=3, bias=False)
        self.bn1 = nn.BatchNorm2d(64)
        self.relu = nn.ReLU(inplace=True)
        self.maxpool = nn.MaxPool2d(kernel_size=3, stride=2, padding=1)
        
        # 4 stages of residual blocks
        self.layer1 = self._make_layer(block, 64, layers[0], stride=1)
        self.layer2 = self._make_layer(block, 128, layers[1], stride=2)
        self.layer3 = self._make_layer(block, 256, layers[2], stride=2)
        self.layer4 = self._make_layer(block, 512, layers[3], stride=2)
        
        # Classification head
        self.avgpool = nn.AdaptiveAvgPool2d((1, 1))
        self.fc = nn.Linear(512 * block.expansion, num_classes)
        
        # Weight initialization (Kaiming)
        for m in self.modules():
            if isinstance(m, nn.Conv2d):
                nn.init.kaiming_normal_(m.weight, mode='fan_out', nonlinearity='relu')
            elif isinstance(m, nn.BatchNorm2d):
                nn.init.constant_(m.weight, 1)
                nn.init.constant_(m.bias, 0)
    
    def _make_layer(self, block, out_channels, num_blocks, stride):
        """Create a stage with multiple residual blocks."""
        downsample = None
        
        # Need projection shortcut if stride > 1 or channel mismatch
        if stride != 1 or self.in_channels != out_channels * block.expansion:
            downsample = nn.Sequential(
                nn.Conv2d(self.in_channels, out_channels * block.expansion,
                         kernel_size=1, stride=stride, bias=False),
                nn.BatchNorm2d(out_channels * block.expansion),
            )
        
        layers = []
        # First block: may change dimensions (stride > 1)
        layers.append(block(self.in_channels, out_channels, stride, downsample))
        self.in_channels = out_channels * block.expansion
        
        # Remaining blocks: identity shortcuts
        for _ in range(1, num_blocks):
            layers.append(block(self.in_channels, out_channels))
        
        return nn.Sequential(*layers)
    
    def forward(self, x):
        x = self.maxpool(self.relu(self.bn1(self.conv1(x))))  # 112→56
        
        x = self.layer1(x)  # 56×56
        x = self.layer2(x)  # 28×28
        x = self.layer3(x)  # 14×14
        x = self.layer4(x)  # 7×7
        
        x = self.avgpool(x)  # 1×1
        x = torch.flatten(x, 1)
        x = self.fc(x)
        return x


# Factory functions for different ResNet variants:
def resnet18(num_classes=1000):
    return ResNet(BasicBlock, [2, 2, 2, 2], num_classes)

def resnet34(num_classes=1000):
    return ResNet(BasicBlock, [3, 4, 6, 3], num_classes)

def resnet50(num_classes=1000):
    return ResNet(Bottleneck, [3, 4, 6, 3], num_classes)

def resnet101(num_classes=1000):
    return ResNet(Bottleneck, [3, 4, 23, 3], num_classes)

def resnet152(num_classes=1000):
    return ResNet(Bottleneck, [3, 8, 36, 3], num_classes)


# Test
model = resnet50(num_classes=10)
x = torch.randn(2, 3, 224, 224)
y = model(x)
print(f"Output shape: {y.shape}")  # (2, 10)
print(f"Parameters: {sum(p.numel() for p in model.parameters()):,}")
```

### Using Pretrained ResNet from torchvision

```python
from torchvision.models import resnet50, ResNet50_Weights

# Load pretrained ResNet-50
model = resnet50(weights=ResNet50_Weights.IMAGENET1K_V2)

# Modify for custom number of classes
num_classes = 10
model.fc = nn.Linear(model.fc.in_features, num_classes)

# Freeze feature extractor (optional — for transfer learning)
for param in model.parameters():
    param.requires_grad = False
model.fc.requires_grad_(True)  # only train the new FC layer

print(f"Trainable params: {sum(p.numel() for p in model.parameters() if p.requires_grad):,}")
```

---

## 6. Pre-Activation ResNet

```
  Original ResNet:              Pre-activation ResNet:
  (post-activation)             (He et al., 2016)
  
  x ──┬──────────────┐         x ──┬──────────────┐
      │               │             │               │
      ↓               │             ↓               │
   [Conv]            │          [BN]              │
   [BN]              │          [ReLU]            │
   [ReLU]            │          [Conv]            │
   [Conv]            │          [BN]              │
   [BN]              │          [ReLU]            │
      │               │          [Conv]            │
      +──←────────────┘             │               │
      ↓                             +──←────────────┘
   [ReLU]                          ↓
      ↓                            output
    output
  
  Key difference: BN and ReLU come BEFORE Conv, not after.
  
  Benefits of pre-activation:
  ✓ Cleaner gradient flow (identity shortcut is pure addition)
  ✓ Better regularization effect from BN placement
  ✓ Slightly better performance on very deep networks (1001 layers!)
  ✓ Easier to analyze mathematically
```

```python
class PreActBlock(nn.Module):
    """Pre-activation residual block."""
    expansion = 1
    
    def __init__(self, in_channels, out_channels, stride=1, downsample=None):
        super().__init__()
        self.bn1 = nn.BatchNorm2d(in_channels)
        self.relu = nn.ReLU(inplace=True)
        self.conv1 = nn.Conv2d(in_channels, out_channels, 3, stride, 1, bias=False)
        self.bn2 = nn.BatchNorm2d(out_channels)
        self.conv2 = nn.Conv2d(out_channels, out_channels, 3, 1, 1, bias=False)
        self.downsample = downsample
    
    def forward(self, x):
        identity = x
        
        out = self.bn1(x)
        out = self.relu(out)
        
        if self.downsample is not None:
            identity = self.downsample(out)
        
        out = self.conv1(out)
        out = self.bn2(out)
        out = self.relu(out)
        out = self.conv2(out)
        
        out += identity
        return out  # no final ReLU — it's at the START of next block
```

---

## 7. Summary Table

| Variant | Depth | Block Type | Params | Top-1 Error (ImageNet) | Use Case |
|---------|:-----:|------------|:------:|:---------------------:|----------|
| **ResNet-18** | 18 | Basic | 11.7M | 30.2% | Quick experiments, small datasets |
| **ResNet-34** | 34 | Basic | 21.8M | 26.7% | Medium tasks |
| **ResNet-50** | 50 | Bottleneck | 25.6M | 24.0% | ✓ Most popular, great balance |
| **ResNet-101** | 101 | Bottleneck | 44.5M | 22.4% | Large datasets, high accuracy |
| **ResNet-152** | 152 | Bottleneck | 60.2M | 21.7% | Maximum accuracy, competitions |

---

## 8. Revision Questions

1. **Describe the two types of residual blocks (basic and bottleneck).** How many convolutional layers does each have, and what is the purpose of the 1×1 convolutions in the bottleneck block?

2. **What is a projection shortcut and when is it needed?** Write the PyTorch code for a projection shortcut using a 1×1 convolution.

3. **Explain why ResNet-50 has fewer parameters than ResNet-34 despite being deeper.** What role does the bottleneck architecture play?

4. **Compare original (post-activation) ResNet with pre-activation ResNet.** Where are BatchNorm and ReLU placed in each, and why does pre-activation provide better gradient flow?

5. **Write PyTorch code to load a pretrained ResNet-50, modify it for 5-class classification, and freeze the feature extractor while keeping the final FC layer trainable.**

6. **Why was ResNet such a breakthrough?** Place it in historical context with AlexNet, VGG, and GoogLeNet. What specific problem did it solve that the others couldn't?

---

| [← 03 Skip Connections](03-skip-connections.md) | [Unit 10 Home](README.md) | [05 → Transfer Learning Basics](05-transfer-learning-basics.md) |
|:---|:---:|---:|
