# 🔍 1.3 — Local Connectivity & Receptive Fields

> **Unit:** 01 – Introduction to CNNs
> **Chapter:** 3 of 5 — How CNNs See the World Through Small Windows

---

[← Previous: Limitations of FC Networks](./02-limitations-fc-networks.md) · [🏠 Unit Overview](./README.md) · [Next → Parameter Sharing](./04-parameter-sharing.md)

---

## 📋 Chapter Overview

Local connectivity is the first of two key innovations that make CNNs feasible (the second is parameter sharing, covered next). Instead of connecting every input to every output, each neuron in a convolutional layer "looks at" only a **small local patch** of the input. This chapter explores receptive fields, how stacking layers expands them, and the precise mathematics behind effective receptive field calculations.

**After this chapter you will be able to:**

1. Define "receptive field" and explain why local regions contain sufficient information for feature detection.
2. Draw how a convolutional neuron connects to a local patch of the input.
3. Calculate the theoretical receptive field of a neuron at any depth in a CNN.
4. Explain how stacking conv layers lets deep neurons "see" larger regions.
5. Distinguish between theoretical and effective receptive fields.

---

## 1 — What Is Local Connectivity?

### 1.1 — Core Idea

In a fully connected layer, every output neuron is connected to **every** input value. In a convolutional layer, each output neuron is connected to only a **small local region** of the input.

This local region is called the **receptive field** of that neuron.

```
Fully Connected (FC):                 Locally Connected (Conv):

Input:  x₁ x₂ x₃ x₄ x₅ x₆ x₇      Input:  x₁ x₂ x₃ x₄ x₅ x₆ x₇
         │╲│╲│╲│╲│╱│╱│╱│                       ╲│╱     ╲│╱
         │ ╲│ ╲│ ╲│╱ │╱ │╱                     ╲│╱     ╲│╱
         │  ╲│  ╲│  ╲│  │╱                      │       │
Output:  ●     ●     ●                Output:   ●       ●

         Each output connects              Each output connects
         to ALL 7 inputs                   to only 3 nearby inputs
         (7 weights each)                  (3 weights each)
```

### 1.2 — Why Local Regions Are Sufficient

Images have **strong local correlations**:

- An edge is defined by the contrast between a pixel and its neighbours (3–5 pixel region).
- A corner is the meeting of two edges (5–7 pixel region).
- A texture is a repeating pattern of local features (7–15 pixel region).

> **Insight:** No single visual feature requires looking at the entire image at once. Features are inherently local — they are defined by relationships between nearby pixels.

---

## 2 — The Receptive Field

### 2.1 — Definition

The **receptive field** of a neuron is the region in the **original input image** that can influence the neuron's activation.

- A neuron in **Layer 1** with a 3×3 filter has a receptive field of **3×3** pixels.
- A neuron in **Layer 2** (also 3×3) sees a 3×3 patch of Layer 1's output, but each of those was computed from a 3×3 patch of the input → its receptive field in the input is **5×5**.

### 2.2 — ASCII Diagram: Receptive Field Growth

```
Layer 0 (Input Image):     ● ● ● ● ● ● ● ● ●
                            ● ● ● ● ● ● ● ● ●
                            ● ● ● ● ● ● ● ● ●
                            ● ● ● ● ● ● ● ● ●
                            ● ● ● ● ● ● ● ● ●
                            ● ● ● ● ● ● ● ● ●
                            ● ● ● ● ● ● ● ● ●

Layer 1 (3×3 conv, s=1):       ○ ○ ○ ○ ○ ○ ○
  RF = 3×3 in input             ○ ○ ○ ○ ○ ○ ○
                                ○ ○ ○ ○ ○ ○ ○
                                ○ ○ ○ ○ ○ ○ ○
                                ○ ○ ○ ○ ○ ○ ○

Layer 2 (3×3 conv, s=1):           □ □ □ □ □
  RF = 5×5 in input                 □ □ □ □ □
                                    □ □ □ □ □

Layer 3 (3×3 conv, s=1):               ■ ■ ■
  RF = 7×7 in input                     ■ ■ ■
                                        ■ ■ ■
```

### 2.3 — Detailed View: How RF Grows from 3×3 to 5×5

```
Input (7×7):
    a  b  c  d  e  f  g
    h  i  j  k  l  m  n
    o  p  q  r  s  t  u
    v  w  x  y  z  A  B
    C  D  E  F  G  H  I
    J  K  L  M  N  O  P
    Q  R  S  T  U  V  W

Layer 1 (3×3 conv, stride=1, no padding):
    Output[0,0] uses: {a,b,c, h,i,j, o,p,q}           ← 3×3 RF
    Output[1,1] uses: {i,j,k, p,q,r, w,x,y}           ← 3×3 RF

Layer 2 (3×3 conv on Layer 1 output):
    Output_L2[0,0] uses L1[0:3, 0:3]
    
    L1[0,0] came from input {a,b,c, h,i,j, o,p,q}
    L1[0,1] came from input {b,c,d, i,j,k, p,q,r}
    L1[0,2] came from input {c,d,e, j,k,l, q,r,s}
    L1[1,0] came from input {h,i,j, o,p,q, v,w,x}
    L1[1,1] came from input {i,j,k, p,q,r, w,x,y}
    ...
    L1[2,2] came from input {q,r,s, x,y,z, E,F,G}
    
    Union of all these input regions:
    
    a  b  c  d  e
    h  i  j  k  l        ← 5×5 region in the original input
    o  p  q  r  s
    v  w  x  y  z
    C  D  E  F  G
    
    Layer 2 neuron's RF = 5×5 ✓
```

---

## 3 — Receptive Field Mathematics

### 3.1 — General Formula

For a network with L convolutional layers, each with kernel size `kₗ`, stride `sₗ`, and dilation `dₗ`, the theoretical receptive field of a neuron in layer L is:

```
RF_L = 1 + Σ(l=1 to L) [(kₗ - 1) × dₗ × ∏(i=1 to l-1) sᵢ]
```

Where:
- `kₗ` = kernel size at layer l
- `sₗ` = stride at layer l
- `dₗ` = dilation at layer l (1 for standard convolution)

### 3.2 — Simplified Formula (Standard Conv, dilation = 1)

```
RF_L = 1 + Σ(l=1 to L) [(kₗ - 1) × ∏(i=1 to l-1) sᵢ]
```

The product term `∏(i=1 to l-1) sᵢ` is called the **jump** — it measures how many input pixels one step at layer l corresponds to.

### 3.3 — Iterative Calculation (Easier in Practice)

Start from layer L and work backwards, or build up from layer 1:

```
j₀ = 1           (jump at input)
RF₀ = 1           (receptive field at input)

For each layer l = 1, 2, ..., L:
    RF_l = RF_(l-1) + (kₗ - 1) × dₗ × j_(l-1)
    j_l  = j_(l-1) × sₗ
```

### 3.4 — Worked Example 1: Three 3×3 Convolutions

```
Layer 1: k=3, s=1, d=1
Layer 2: k=3, s=1, d=1
Layer 3: k=3, s=1, d=1

Step-by-step:
─────────────────────────────────────────
l=0 (input):  j₀=1,   RF₀=1
l=1:          RF₁ = 1 + (3-1)×1×1 = 3     j₁ = 1×1 = 1
l=2:          RF₂ = 3 + (3-1)×1×1 = 5     j₂ = 1×1 = 1
l=3:          RF₃ = 5 + (3-1)×1×1 = 7     j₃ = 1×1 = 1
─────────────────────────────────────────

Result: RF = 7×7

Pattern: For N stacked 3×3 conv layers (stride 1):
  RF = 2N + 1

  3 layers → RF = 7
  5 layers → RF = 11
  10 layers → RF = 21
```

### 3.5 — Worked Example 2: With Stride and Pooling

```
Layer 1: Conv 3×3, s=1
Layer 2: MaxPool 2×2, s=2
Layer 3: Conv 3×3, s=1
Layer 4: MaxPool 2×2, s=2
Layer 5: Conv 3×3, s=1

Step-by-step:
─────────────────────────────────────────────────────
l=0 (input):  j₀=1,   RF₀=1
l=1 (conv):   RF₁ = 1 + (3-1)×1×1 = 3      j₁ = 1×1 = 1
l=2 (pool):   RF₂ = 3 + (2-1)×1×1 = 4      j₂ = 1×2 = 2
l=3 (conv):   RF₃ = 4 + (3-1)×1×2 = 8      j₃ = 2×1 = 2
l=4 (pool):   RF₄ = 8 + (2-1)×1×2 = 10     j₄ = 2×2 = 4
l=5 (conv):   RF₅ = 10 + (3-1)×1×4 = 18    j₅ = 4×1 = 4
─────────────────────────────────────────────────────

Result: RF = 18×18

Key insight: Stride/pooling layers MULTIPLY the growth rate
of the receptive field in subsequent layers.
```

### 3.6 — Worked Example 3: Dilated Convolutions

Dilated (atrous) convolutions expand the receptive field without adding parameters:

```
Layer 1: Conv 3×3, s=1, d=1 (standard)
Layer 2: Conv 3×3, s=1, d=2 (dilated)
Layer 3: Conv 3×3, s=1, d=4 (dilated)

Step-by-step:
──────────────────────────────────────────────────────
l=0 (input):  j₀=1,   RF₀=1
l=1 (d=1):    RF₁ = 1 + (3-1)×1×1 = 3       j₁ = 1×1 = 1
l=2 (d=2):    RF₂ = 3 + (3-1)×2×1 = 7       j₂ = 1×1 = 1
l=3 (d=4):    RF₃ = 7 + (3-1)×4×1 = 15      j₃ = 1×1 = 1
──────────────────────────────────────────────────────

Result: RF = 15×15 (with just 3 layers of 3×3 filters!)

Compare: Standard 3×3 conv × 3 layers → RF = 7×7
         Dilated 3×3 conv × 3 layers   → RF = 15×15  (2.1× larger!)
```

```
Standard 3×3 filter:            Dilated 3×3 filter (d=2):

    ■ ■ ■                          ■ · ■ · ■
    ■ ■ ■                          · · · · ·
    ■ ■ ■                          ■ · ■ · ■
                                   · · · · ·
  Covers 3×3 = 9 pixels           ■ · ■ · ■

                                 Covers 5×5 area, samples 9 pixels
                                 Same parameters, larger RF!
```

---

## 4 — Receptive Field in Practice: Real Architectures

### 4.1 — VGG-16 Receptive Field Calculation

```
VGG-16 conv layers (simplified):
───────────────────────────────────
Block 1: 2× Conv3×3(s=1) + MaxPool2×2(s=2)
Block 2: 2× Conv3×3(s=1) + MaxPool2×2(s=2)
Block 3: 3× Conv3×3(s=1) + MaxPool2×2(s=2)
Block 4: 3× Conv3×3(s=1) + MaxPool2×2(s=2)
Block 5: 3× Conv3×3(s=1) + MaxPool2×2(s=2)

Calculation:
──────────────────────────────────────────────────
l   Type       k   s   d   j      RF
──────────────────────────────────────────────────
0   input      -   -   -   1      1
1   conv3      3   1   1   1      3
2   conv3      3   1   1   1      5
3   pool2      2   2   1   2      6
4   conv3      3   1   1   2      10
5   conv3      3   1   1   2      14
6   pool2      2   2   1   4      16
7   conv3      3   1   1   4      24
8   conv3      3   1   1   4      32
9   conv3      3   1   1   4      40
10  pool2      2   2   1   8      44
11  conv3      3   1   1   8      60
12  conv3      3   1   1   8      76
13  conv3      3   1   1   8      92
14  pool2      2   2   1   16     100
15  conv3      3   1   1   16     132
16  conv3      3   1   1   16     164
17  conv3      3   1   1   16     196
18  pool2      2   2   1   32     212
──────────────────────────────────────────────────

Final RF = 212×212 (covers nearly the entire 224×224 input!)
```

### 4.2 — Why 3×3 Filters Are Preferred Over Larger Ones

Two 3×3 conv layers have the same receptive field as one 5×5 conv layer, but with fewer parameters:

```
Option A: One 5×5 conv (C_in=C_out=C)
  RF = 5×5
  Parameters = 5 × 5 × C × C = 25C²

Option B: Two 3×3 convs (C_in=C_out=C)
  RF = 5×5
  Parameters = 2 × (3 × 3 × C × C) = 18C²

  Savings: 18C²/25C² = 72% of the parameters
  Bonus:   Two non-linearities (ReLU) → more expressive!

For 7×7 RF:
  Option A: One 7×7 conv → 49C² params
  Option B: Three 3×3 convs → 27C² params (55% of params)
```

### 4.3 — PyTorch Code: Computing Receptive Field

```python
def compute_receptive_field(layers):
    """
    Compute receptive field for a sequence of layers.
    
    Each layer is a dict: {'name': str, 'k': int, 's': int, 'd': int}
    k = kernel size, s = stride, d = dilation (default 1)
    
    Returns a list of (layer_name, RF, jump) tuples.
    """
    rf = 1
    j = 1
    results = [("input", rf, j)]
    
    for layer in layers:
        k = layer['k']
        s = layer['s']
        d = layer.get('d', 1)
        
        rf = rf + (k - 1) * d * j
        j = j * s
        
        results.append((layer['name'], rf, j))
    
    return results

# VGG-16 architecture
vgg16_layers = [
    # Block 1
    {'name': 'conv1_1', 'k': 3, 's': 1}, {'name': 'conv1_2', 'k': 3, 's': 1},
    {'name': 'pool1',   'k': 2, 's': 2},
    # Block 2
    {'name': 'conv2_1', 'k': 3, 's': 1}, {'name': 'conv2_2', 'k': 3, 's': 1},
    {'name': 'pool2',   'k': 2, 's': 2},
    # Block 3
    {'name': 'conv3_1', 'k': 3, 's': 1}, {'name': 'conv3_2', 'k': 3, 's': 1},
    {'name': 'conv3_3', 'k': 3, 's': 1}, {'name': 'pool3',   'k': 2, 's': 2},
    # Block 4
    {'name': 'conv4_1', 'k': 3, 's': 1}, {'name': 'conv4_2', 'k': 3, 's': 1},
    {'name': 'conv4_3', 'k': 3, 's': 1}, {'name': 'pool4',   'k': 2, 's': 2},
    # Block 5
    {'name': 'conv5_1', 'k': 3, 's': 1}, {'name': 'conv5_2', 'k': 3, 's': 1},
    {'name': 'conv5_3', 'k': 3, 's': 1}, {'name': 'pool5',   'k': 2, 's': 2},
]

results = compute_receptive_field(vgg16_layers)

print(f"{'Layer':<12} {'RF':>6} {'Jump':>6}")
print("─" * 26)
for name, rf, j in results:
    print(f"{name:<12} {rf:>6} {j:>6}")

# Output:
# Layer            RF   Jump
# ──────────────────────────
# input             1      1
# conv1_1           3      1
# conv1_2           5      1
# pool1             6      2
# conv2_1          10      2
# conv2_2          14      2
# pool2            16      4
# conv3_1          24      4
# conv3_2          32      4
# conv3_3          40      4
# pool3            44      8
# conv4_1          60      8
# conv4_2          76      8
# conv4_3          92      8
# pool4           100     16
# conv5_1         132     16
# conv5_2         164     16
# conv5_3         196     16
# pool5           212     32
```

---

## 5 — Theoretical vs Effective Receptive Field

### 5.1 — The Problem with Theoretical RF

The theoretical RF calculation assumes every pixel in the receptive field contributes **equally** to the output. In practice, this is not true.

### 5.2 — Effective Receptive Field (ERF)

Research by Luo et al. (2016) showed that the **effective receptive field** (the region that actually has significant influence) is much smaller than the theoretical RF, and follows a **Gaussian distribution** centred on the neuron.

```
Theoretical RF (7×7):               Effective RF:

  ■ ■ ■ ■ ■ ■ ■                     · · · · · · ·
  ■ ■ ■ ■ ■ ■ ■                     · · ░ ░ ░ · ·
  ■ ■ ■ ■ ■ ■ ■                     · ░ ▓ ▓ ▓ ░ ·
  ■ ■ ■ ■ ■ ■ ■                     · ░ ▓ █ ▓ ░ ·
  ■ ■ ■ ■ ■ ■ ■                     · ░ ▓ ▓ ▓ ░ ·
  ■ ■ ■ ■ ■ ■ ■                     · · ░ ░ ░ · ·
  ■ ■ ■ ■ ■ ■ ■                     · · · · · · ·

  All 49 pixels treated              Centre pixels (█,▓) contribute
  as equally important                much more than edge pixels (·)
                                      ERF ≈ Gaussian, smaller than RF
```

### 5.3 — Key Findings

| Property | Detail |
|---|---|
| ERF shape | Approximately Gaussian (bell-shaped) |
| ERF size | Grows as **O(√n)** not O(n) for n layers |
| Centre dominance | Centre pixels contribute exponentially more |
| Skip connections | ResNets have larger ERF than VGG (skip connections help) |
| Dilated conv | Can increase ERF without reducing resolution |

### 5.4 — PyTorch Code: Visualising the Effective Receptive Field

```python
import torch
import torch.nn as nn
import numpy as np

def compute_effective_rf(model, input_size=(1, 1, 64, 64)):
    """
    Compute effective receptive field by backpropagating
    from the centre neuron of the output.
    """
    model.eval()
    
    # Create input that requires grad
    x = torch.randn(input_size, requires_grad=True)
    
    # Forward pass
    y = model(x)
    
    # Get centre position of output
    _, _, h_out, w_out = y.shape
    centre_h, centre_w = h_out // 2, w_out // 2
    
    # Backward from centre neuron
    grad_output = torch.zeros_like(y)
    grad_output[0, 0, centre_h, centre_w] = 1.0
    y.backward(gradient=grad_output)
    
    # The gradient w.r.t. input shows which pixels influence the centre output
    erf = x.grad[0, 0].abs().detach().numpy()
    
    return erf

# Simple 3-layer CNN
model = nn.Sequential(
    nn.Conv2d(1, 16, 3, padding=1), nn.ReLU(),
    nn.Conv2d(16, 16, 3, padding=1), nn.ReLU(),
    nn.Conv2d(16, 1, 3, padding=1),
)

erf = compute_effective_rf(model, (1, 1, 64, 64))

# Theoretical RF = 7×7, but effective RF is Gaussian and smaller
print(f"Theoretical RF: 7×7 = 49 pixels")
print(f"Max gradient at centre: {erf[32, 32]:.6f}")
print(f"Gradient at edge of RF: {erf[32, 29]:.6f}")
print(f"Ratio centre/edge: {erf[32, 32] / max(erf[32, 29], 1e-10):.1f}×")

# Show gradient magnitude as simple text heatmap
print("\nEffective RF heatmap (centre 11×11 region):")
region = erf[27:38, 27:38]
region_normalized = region / region.max()
symbols = " ·░▒▓█"
for row in region_normalized:
    line = ""
    for val in row:
        idx = min(int(val * (len(symbols) - 1)), len(symbols) - 1)
        line += symbols[idx] + " "
    print(f"  {line}")
```

---

## 6 — How Local Features Build Global Understanding

### 6.1 — The Compositionality Principle

Local connectivity does NOT mean the network can only see small regions. Through **stacking layers**, each successive layer sees a progressively larger region:

```
Layer 1 neuron:  Sees 3×3 → detects edges
Layer 2 neuron:  Sees 5×5 → detects corners (combinations of edges)
Layer 3 neuron:  Sees 7×7 → detects textures (combinations of corners)
   ...
Layer 10 neuron: Sees 21×21 → detects object parts
   ...
Layer 16 neuron (with pooling): Sees 200×200+ → detects whole objects
```

### 6.2 — ASCII Diagram: Hierarchical Composition

```
                    Layer 3 neuron
                    RF = 7×7 in input
                    Detects: "corner"
                         ┌─┐
                         │■│
                      ┌──┴─┴──┐
                      │       │
              Layer 2 neuron  Layer 2 neuron
              RF = 5×5        RF = 5×5
              Detects:        Detects:
              "horiz edge"    "vert edge"
                 ┌─┐             ┌─┐
                 │■│             │■│
              ┌──┴─┴──┐      ┌──┴─┴──┐
              │       │      │       │
       L1 neurons     L1 neurons
       RF = 3×3       RF = 3×3
       Detect:        Detect:
       gradients      gradients

       ●●●●●●●●●●●●●●●●●●●●●
              Input pixels
```

### 6.3 — Connection to Feature Hierarchy

```
Input Image: A photo of a face

Layer 1 (RF 3×3):    Detects gradients, colour changes
Layer 2 (RF 5×5):    Detects edges (boundaries between regions)
Layer 3 (RF 10×10):  Detects corners, curves
Layer 5 (RF 20×20):  Detects textures (skin, hair)
Layer 8 (RF 50×50):  Detects parts (eye, nose, mouth)
Layer 12 (RF 150×150): Detects face configuration
Layer 16 (RF 212×212): Recognises the specific person

Each layer combines LOCAL features from the previous layer
into increasingly GLOBAL and ABSTRACT representations.
```

---

## 7 — Practical Design Considerations

### 7.1 — How Large Should the RF Be?

The RF should be large enough to encompass the features you want to detect:

| Task | Typical RF Needed | Design Strategy |
|---|---|---|
| Edge detection | 3–5 px | 1–2 conv layers |
| Texture classification | 20–50 px | 5–8 conv layers |
| Object detection (small) | 50–100 px | 10–15 conv layers + pooling |
| Scene understanding | 200+ px | Deep network, dilated conv, or global pooling |

### 7.2 — Strategies to Increase RF

| Strategy | Effect on RF | Trade-off |
|---|---|---|
| More layers | Linear growth (2 per 3×3 layer) | More parameters, deeper network |
| Larger kernels | Faster growth per layer | More parameters per layer |
| Stride > 1 | Multiplies growth of subsequent layers | Reduces spatial resolution |
| Pooling | Similar to stride | Loses fine spatial detail |
| Dilated conv | Exponential growth possible | Can create gridding artifacts |
| Skip connections | Doesn't change theoretical RF | Improves effective RF |

### 7.3 — PyTorch Code: Building Networks with Different RF Strategies

```python
import torch
import torch.nn as nn

# Strategy 1: Stack many 3×3 convolutions
class DeepSmallKernel(nn.Module):
    """7 layers of 3×3 conv → RF = 15×15"""
    def __init__(self):
        super().__init__()
        layers = [nn.Conv2d(1, 32, 3, padding=1), nn.ReLU()]
        for _ in range(5):
            layers += [nn.Conv2d(32, 32, 3, padding=1), nn.ReLU()]
        layers += [nn.Conv2d(32, 1, 3, padding=1)]
        self.net = nn.Sequential(*layers)
    
    def forward(self, x):
        return self.net(x)

# Strategy 2: Use dilated convolutions
class DilatedConv(nn.Module):
    """3 layers of dilated 3×3 conv → RF = 15×15 (same RF, fewer layers!)"""
    def __init__(self):
        super().__init__()
        self.net = nn.Sequential(
            nn.Conv2d(1, 32, 3, padding=1, dilation=1), nn.ReLU(),   # d=1, RF=3
            nn.Conv2d(32, 32, 3, padding=2, dilation=2), nn.ReLU(),  # d=2, RF=7
            nn.Conv2d(32, 1, 3, padding=4, dilation=4),              # d=4, RF=15
        )
    
    def forward(self, x):
        return self.net(x)

# Strategy 3: Use pooling to expand RF
class PoolingExpander(nn.Module):
    """Conv + Pool layers → larger RF per layer"""
    def __init__(self):
        super().__init__()
        self.net = nn.Sequential(
            nn.Conv2d(1, 32, 3, padding=1), nn.ReLU(),    # RF=3
            nn.MaxPool2d(2, 2),                            # RF=4, j=2
            nn.Conv2d(32, 64, 3, padding=1), nn.ReLU(),   # RF=8
            nn.MaxPool2d(2, 2),                            # RF=10, j=4
            nn.Conv2d(64, 1, 3, padding=1),                # RF=18
        )
    
    def forward(self, x):
        return self.net(x)

# Compare parameter counts
for cls in [DeepSmallKernel, DilatedConv, PoolingExpander]:
    model = cls()
    params = sum(p.numel() for p in model.parameters())
    
    # Compute RF using our function from earlier
    print(f"{cls.__name__:20s}: {params:>8,} params")
```

---

## 8 — Local Connectivity as Inductive Bias

### 8.1 — What Is Inductive Bias?

An **inductive bias** is an assumption built into the architecture that helps the model learn more efficiently. Local connectivity encodes the assumption:

> *"Visual features are defined by local spatial relationships between nearby pixels."*

This assumption is almost always true for natural images, which is why CNNs generalise so well.

### 8.2 — When Local Connectivity Fails

| Scenario | Why It Fails | Solution |
|---|---|---|
| Very long-range dependencies | Features depend on distant pixels | Self-attention (Transformers), global pooling |
| Non-image structured data | No spatial locality | FC, Graph Neural Networks |
| Extremely low resolution | Everything is "local" already | FC may suffice (e.g., 4×4 image) |

---

## 📊 Summary Table

| Concept | Key Idea |
|---|---|
| Local Connectivity | Each neuron connects to a small local region, not the entire input |
| Receptive Field (RF) | The region of the input that influences a neuron's output |
| RF Growth | RF grows by (k-1)×d×j per layer (k=kernel, d=dilation, j=jump) |
| Stacking Layers | Deeper neurons have larger RF → can detect larger features |
| Effective RF (ERF) | Centre of RF contributes more; ERF is Gaussian, smaller than theoretical RF |
| 3×3 Preference | Two 3×3 layers match one 5×5 layer's RF with 72% of parameters |
| Dilated Conv | Expands RF exponentially without adding parameters |
| Inductive Bias | Local connectivity encodes the assumption of spatial locality |
| Compositionality | Local features combine into global understanding through depth |

---

## ❓ Revision Questions

### Q1: RF Calculation — Basic
Calculate the receptive field of a network with four 3×3 convolutional layers, all with stride 1 and no dilation.

<details>
<summary>Answer</summary>

Using the iterative formula: RF_l = RF_(l-1) + (k-1) × d × j_(l-1)

```
l=0: RF=1, j=1
l=1: RF = 1 + (3-1)×1×1 = 3,   j = 1×1 = 1
l=2: RF = 3 + (3-1)×1×1 = 5,   j = 1×1 = 1
l=3: RF = 5 + (3-1)×1×1 = 7,   j = 1×1 = 1
l=4: RF = 7 + (3-1)×1×1 = 9,   j = 1×1 = 1

RF = 9×9

Shortcut: For N layers of 3×3 (stride 1): RF = 2N + 1 = 2(4) + 1 = 9 ✓
```
</details>

### Q2: RF with Pooling
Calculate the receptive field: Conv3×3(s=1) → MaxPool2×2(s=2) → Conv3×3(s=1) → MaxPool2×2(s=2).

<details>
<summary>Answer</summary>

```
l=0: RF=1, j=1
l=1 (conv3):  RF = 1 + 2×1×1 = 3,   j = 1×1 = 1
l=2 (pool2):  RF = 3 + 1×1×1 = 4,   j = 1×2 = 2
l=3 (conv3):  RF = 4 + 2×1×2 = 8,   j = 2×1 = 2
l=4 (pool2):  RF = 8 + 1×1×2 = 10,  j = 2×2 = 4

RF = 10×10

Notice how pooling layers multiply the jump, which accelerates
RF growth in subsequent conv layers: the conv after the first pool
adds 4 to RF (vs 2 before pool).
```
</details>

### Q3: Dilated Convolution
How many standard 3×3 conv layers (stride 1) would you need to match the receptive field of three dilated 3×3 conv layers with dilations [1, 2, 4]?

<details>
<summary>Answer</summary>

**Dilated network RF:**
```
l=1 (d=1): RF = 1 + 2×1×1 = 3
l=2 (d=2): RF = 3 + 2×2×1 = 7
l=3 (d=4): RF = 7 + 2×4×1 = 15
```

**Standard 3×3 (d=1) to match RF=15:**
```
RF = 2N + 1 = 15  →  N = 7 layers
```

You would need **7 standard layers** to match what 3 dilated layers achieve. Dilated convolutions are more parameter-efficient for large receptive fields.
</details>

### Q4: 3×3 vs 7×7 Filters
Show mathematically that three 3×3 convolutions have the same receptive field as one 7×7 convolution. Compare parameter counts assuming C channels in and out.

<details>
<summary>Answer</summary>

**Receptive field:**
- Three 3×3 (stride 1): RF = 2(3) + 1 = 7×7 ✓
- One 7×7: RF = 7×7 ✓ — Same!

**Parameters:**
```
Three 3×3: 3 × (3 × 3 × C × C) = 27C²
One 7×7:   1 × (7 × 7 × C × C) = 49C²

Ratio: 27C²/49C² = 55.1%
```

Three 3×3 layers use only **55% of the parameters** and add **two extra non-linearities** (ReLU), making the network more expressive. This is why VGG-16 exclusively uses 3×3 filters.
</details>

### Q5: Effective RF
Why is the effective receptive field typically smaller than the theoretical one? What shape does it approximate?

<details>
<summary>Answer</summary>

The theoretical RF assumes all pixels contribute equally, but in practice:

1. **Centre pixels** are involved in more computation paths than edge pixels (they are "visited" by more overlapping filter applications).
2. Gradients from the centre reach the output through more paths, so they have **higher gradient magnitude**.
3. The contribution falls off approximately as a **Gaussian distribution** — centre pixels have exponentially more influence.

The ERF grows as **O(√n)** rather than O(n), where n is the number of layers. Skip connections (as in ResNets) help enlarge the effective RF closer to the theoretical RF.
</details>

### Q6: Design Question
You're designing a CNN for satellite image analysis where objects of interest span about 100×100 pixels in a 512×512 input. Propose a strategy to ensure your network has sufficient receptive field.

<details>
<summary>Answer</summary>

Target RF ≥ 100×100. Several strategies:

1. **Stack conv + pooling blocks:**
   - 3 blocks of [Conv3×3, Conv3×3, MaxPool2×2] gives RF ≈ 44
   - 5 blocks gives RF ≈ 180 — sufficient ✓

2. **Use dilated convolutions:**
   - Conv3×3 d=[1,2,4,8,16] gives RF = 1 + 2(1+2+4+8+16) = 63
   - Add a few more dilated layers or combine with pooling to reach 100+

3. **Strided convolutions:**
   - Conv3×3(s=2) early on → rapidly expands RF in later layers
   - Conv7×7(s=2) as first layer (like ResNet) → RF=7 with j=2 immediately

4. **Hybrid approach (recommended):**
   - Conv7×7(s=2) + MaxPool3×3(s=2) + 4 blocks of [Conv3×3, Conv3×3, MaxPool2×2]
   - This mirrors ResNet's design and easily achieves RF > 100
</details>

---

## 🔗 Navigation

| | |
|---|---|
| ← Previous | [Limitations of FC Networks](./02-limitations-fc-networks.md) |
| → Next | [Parameter Sharing](./04-parameter-sharing.md) |
| 🏠 Unit | [01 – Introduction to CNNs](./README.md) |

---

*© 2025 AIML Study Notes. For educational use.*
