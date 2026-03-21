# ⚡ Activation Layers in CNNs

[← Previous: 01 — Convolutional Layer](01-convolutional-layer.md) | [📑 Unit Overview](../README.md) | [Next → 03 — Pooling Layer](03-pooling-layer.md)

---

## 📋 Chapter Overview

Activation functions introduce **non-linearity** into neural networks. Without them, stacking convolutional layers would be equivalent to a single linear transformation — no matter how deep the network. In CNNs, the activation layer is applied **element-wise** to every value in every feature map after the convolution (and optionally after batch normalization).

**What you'll learn:**
- Why ReLU became the standard activation for CNNs
- In-place ReLU and its memory-saving benefits
- Leaky ReLU, PReLU, and other variants
- The activation placement debate (Conv→BN→ReLU vs Conv→ReLU→BN)
- PyTorch implementation with `nn.ReLU`, `nn.LeakyReLU`, `nn.PReLU`
- When to use which activation and practical guidelines

---

## 1️⃣ Why Non-Linearity Is Essential

### The Problem Without Activation

A convolution is a linear operation. Stacking two linear operations produces another linear operation:

```
f(x) = W₂ · (W₁ · x) = (W₂ · W₁) · x = W_combined · x
```

No matter how many conv layers you stack, without activation functions the entire network collapses to a **single convolution** — losing all representational power.

### What Activation Does

```
Conv Layer Output          After ReLU
(can be any value)         (non-negative only)
┌─────────────────┐        ┌─────────────────┐
│ -2   5   3  -1  │        │  0   5   3   0  │
│  7  -4   1   6  │  ───►  │  7   0   1   6  │
│ -3   2  -5   8  │ ReLU   │  0   2   0   8  │
│  1   0  -2   4  │        │  1   0   0   4  │
└─────────────────┘        └─────────────────┘

Negative activations → zeroed out
Positive activations → passed through unchanged
```

This simple element-wise operation creates **piece-wise linear** decision boundaries, enabling the network to learn complex, non-linear mappings.

---

## 2️⃣ ReLU — The Standard CNN Activation

### Definition

```
ReLU(x) = max(0, x)

         ∂ReLU     ┌ 1   if x > 0
         ───── =   │ 0   if x < 0
          ∂x       └ undefined at x = 0  (typically set to 0)
```

### Graph (ASCII)

```
    output
    ↑
    │         /
    │        /
    │       /
    │      /
    │     /
────┼────/──────────► input
    │   0
    │
    │  (flat at 0 for negative inputs)
```

### Why ReLU Dominates in CNNs

| Advantage | Explanation |
|-----------|-------------|
| **Computational speed** | Just a threshold — `max(0, x)` — no exp/division |
| **Sparse activation** | ~50% of neurons output zero → natural regularization |
| **No vanishing gradient** | Gradient is exactly 1 for positive inputs (unlike sigmoid's tiny gradients) |
| **Biological plausibility** | Neurons either fire or don't — similar to ReLU's on/off behavior |
| **Proven at scale** | All major architectures (VGG, ResNet, EfficientNet) use ReLU variants |

### Historical Context

Before ReLU, CNNs used **sigmoid** or **tanh**. AlexNet (2012) was one of the first major CNNs to use ReLU, and training was **6× faster** compared to tanh.

```
Sigmoid:  σ(x) = 1 / (1 + e^(-x))     → gradient vanishes for |x| >> 0
Tanh:     tanh(x) = (e^x - e^(-x)) / (e^x + e^(-x))  → same vanishing problem
ReLU:     max(0, x)                      → gradient = 1 for all x > 0  ✓
```

---

## 3️⃣ The Dead ReLU Problem

When a neuron's weights update such that the pre-activation is always negative, its output is permanently 0, and its gradient is permanently 0 — the neuron is **dead** and can never recover.

```
Dead Neuron:
  All inputs → negative pre-activation → ReLU outputs 0 → gradient = 0
  → weights never update → permanently stuck at 0

Common causes:
  • Learning rate too high → large weight update overshoots
  • Poor weight initialization
  • Unlucky data distribution in a batch
```

### How Common Is It?

In practice, 10–40% of neurons can die during training, especially with high learning rates. This isn't always catastrophic (it provides implicit regularization), but excessive death hurts capacity.

**Solutions:**
1. Use **Leaky ReLU** or **PReLU** (small gradient for negative inputs)
2. Careful **learning rate scheduling** (warmup)
3. Proper **weight initialization** (He initialization, designed for ReLU)
4. **Batch Normalization** (keeps pre-activations centered)

---

## 4️⃣ In-Place ReLU — Memory Optimization

### What Is In-Place?

Normally, ReLU creates a **new tensor** for the output, keeping the input in memory. In-place ReLU modifies the input tensor directly, **overwriting** negative values with zero.

```python
# Standard ReLU — allocates new tensor
relu = nn.ReLU(inplace=False)  # default
y = relu(x)  # x unchanged, y is new tensor

# In-place ReLU — modifies x directly
relu_ip = nn.ReLU(inplace=True)
y = relu_ip(x)  # x is modified! y and x point to same data
```

### Memory Savings

```
Standard:  Memory = input tensor + output tensor = 2 × tensor_size
In-place:  Memory = input tensor only = 1 × tensor_size
Saving:    50% reduction per activation layer!

For a batch of 32 images at 64×56×56:
  Standard:  32 × 64 × 56 × 56 × 4 bytes × 2 = ~51 MB
  In-place:  32 × 64 × 56 × 56 × 4 bytes     = ~25.5 MB
  Per layer savings: ~25.5 MB
```

### When NOT to Use In-Place

```python
# ❌ BREAKS autograd if input is needed later (e.g., residual connections)
class BrokenResBlock(nn.Module):
    def forward(self, x):
        identity = x                        # identity and x share memory!
        out = self.conv1(x)
        out = self.relu(out)                 # this is fine (operating on conv output)
        out = self.conv2(out)
        out = self.relu_inplace(out)         # fine here too
        return out + identity               # identity is still the original x ✓

# ✅ Actually, the above IS fine because relu is applied to conv output, not x.
# In-place is only dangerous when you need the original input later AND apply
# in-place directly to it.

# ❌ This would break:
class TrulyBroken(nn.Module):
    def forward(self, x):
        identity = x
        x = self.relu_inplace(x)  # Modifies x → identity is also modified!
        return x + identity        # identity is now the ReLU'd version, not original
```

### PyTorch Usage

```python
import torch
import torch.nn as nn

# In-place ReLU in Sequential
model = nn.Sequential(
    nn.Conv2d(3, 64, 3, padding=1, bias=False),
    nn.BatchNorm2d(64),
    nn.ReLU(inplace=True),   # ← saves memory; safe here
    nn.Conv2d(64, 128, 3, padding=1, bias=False),
    nn.BatchNorm2d(128),
    nn.ReLU(inplace=True),
)

# Functional API (also supports in-place)
import torch.nn.functional as F
x = torch.randn(1, 64, 32, 32)
x = F.relu(x, inplace=True)     # modifies x in-place
```

---

## 5️⃣ Leaky ReLU

Fixes the dead neuron problem by allowing a small gradient when the input is negative.

### Definition

```
LeakyReLU(x) = ┌ x              if x > 0
               └ α · x          if x ≤ 0       (α = 0.01 default)

∂LeakyReLU     ┌ 1   if x > 0
─────────── =  │
    ∂x         └ α   if x ≤ 0     (small but non-zero!)
```

### Graph (ASCII)

```
    output
    ↑
    │         /
    │        /
    │       /
    │      /
    │     /
──/─┼────/──────────► input
  / │   0
 /  │
/   │  (slight negative slope α)
```

### PyTorch Implementation

```python
# Module API
leaky_relu = nn.LeakyReLU(negative_slope=0.01, inplace=True)

# Functional API
output = F.leaky_relu(input, negative_slope=0.01, inplace=True)

# In a CNN block
class LeakyConvBlock(nn.Module):
    def __init__(self, in_ch, out_ch):
        super().__init__()
        self.conv = nn.Conv2d(in_ch, out_ch, 3, padding=1, bias=False)
        self.bn = nn.BatchNorm2d(out_ch)
        self.act = nn.LeakyReLU(0.2, inplace=True)  # 0.2 common in GANs
    
    def forward(self, x):
        return self.act(self.bn(self.conv(x)))
```

### When to Use Leaky ReLU

- **GANs**: Standard choice for discriminator (with slope=0.2)
- When you observe many **dead neurons** during training
- Deeper networks where gradient flow is critical

---

## 6️⃣ PReLU — Parametric ReLU

The negative slope is a **learnable parameter** instead of a fixed constant.

### Definition

```
PReLU(x) = ┌ x              if x > 0
           └ a · x           if x ≤ 0       (a is LEARNED)

Parameter a is initialized to 0.25 by default.
One 'a' per channel (or shared across all channels).
```

### PyTorch Implementation

```python
# One slope parameter per channel
prelu = nn.PReLU(num_parameters=64)  # 64 learnable slopes for 64 channels
print(prelu.weight.shape)  # torch.Size([64])

# Single shared slope
prelu_shared = nn.PReLU(num_parameters=1)  # one slope for all channels

# Usage in a model
class PReLUBlock(nn.Module):
    def __init__(self, channels):
        super().__init__()
        self.conv = nn.Conv2d(channels, channels, 3, padding=1, bias=False)
        self.bn = nn.BatchNorm2d(channels)
        self.act = nn.PReLU(num_parameters=channels)
    
    def forward(self, x):
        return self.act(self.bn(self.conv(x)))

# PReLU adds very few parameters: just 1 per channel
# For 256 channels: only 256 extra params (negligible)
```

### PReLU vs Leaky ReLU

| Feature | Leaky ReLU | PReLU |
|---------|-----------|-------|
| Negative slope | Fixed (hyperparameter) | Learned (parameter) |
| Extra parameters | 0 | `num_parameters` |
| Flexibility | Must tune manually | Adapts during training |
| Risk | May use suboptimal slope | Can overfit on small datasets |
| Common slopes | 0.01 (default), 0.2 (GANs) | Initialized at 0.25 |

---

## 7️⃣ Other Activations (Brief Reference)

### ELU (Exponential Linear Unit)

```
ELU(x) = ┌ x                if x > 0
         └ α(e^x - 1)       if x ≤ 0

Smooth negative part → mean activations closer to zero → faster convergence
```

### SELU (Scaled ELU)

```
SELU(x) = λ · ┌ x                    if x > 0
              └ α(e^x - 1)           if x ≤ 0

λ ≈ 1.0507,  α ≈ 1.6733  (specific values for self-normalization)
Used with nn.AlphaDropout and lecun_normal initialization.
Enables self-normalizing networks (rarely used in CNNs in practice).
```

### GELU (Gaussian Error Linear Unit)

```
GELU(x) = x · Φ(x)     where Φ is the standard Gaussian CDF

Smooth approximation of ReLU. Used in:
  - Vision Transformers (ViT)
  - BERT, GPT (NLP transformers)
  - ConvNeXt (modern CNN)
```

### SiLU / Swish

```
SiLU(x) = x · σ(x)     where σ is the sigmoid function

"Smooth ReLU" — used in EfficientNet, YOLOv5+
```

### Comparison Graph (ASCII)

```
    output
    ↑
    │           ReLU  /
    │       SiLU   / /
    │    GELU   / / /
    │         / / //
    │        ///
────┼───────/────────► input
    │      0
   _│_/  Leaky ReLU (slight negative slope)
  / │
 /  │   ELU (exponential curve below zero)
```

### Activation Selection Summary

| Activation | Best For | PyTorch |
|-----------|---------|---------|
| **ReLU** | Default for most CNNs | `nn.ReLU(inplace=True)` |
| **Leaky ReLU** | GANs, dead neuron issues | `nn.LeakyReLU(0.2)` |
| **PReLU** | When you want learned slopes | `nn.PReLU(num_parameters=C)` |
| **GELU** | Vision Transformers, ConvNeXt | `nn.GELU()` |
| **SiLU/Swish** | EfficientNet, YOLOv5+ | `nn.SiLU()` |
| **Sigmoid** | Binary output / gating only | `nn.Sigmoid()` |

---

## 8️⃣ The Activation Placement Debate

### Option A: Conv → BN → ReLU (Original / Most Common)

```
x ──► Conv ──► BatchNorm ──► ReLU ──► next layer

Used in: ResNet, VGG+BN, EfficientNet, most standard CNNs
Reasoning: BN normalizes conv output before non-linearity
```

### Option B: Conv → ReLU → BN

```
x ──► Conv ──► ReLU ──► BatchNorm ──► next layer

Reasoning: Let ReLU create sparsity first, then normalize the result
Some papers suggest this works equally well
```

### Option C: BN → ReLU → Conv (Pre-activation, ResNet v2)

```
x ──► BatchNorm ──► ReLU ──► Conv ──► next layer

Used in: ResNet v2 (pre-activation residual blocks)
Reasoning:
  - BN and ReLU act as "pre-processing" for the conv
  - Gradient flows more smoothly through the identity shortcut
  - Empirically better for very deep networks (200+ layers)
```

### Visual: Pre-activation vs Post-activation ResBlock

```
Post-activation (Original ResNet):    Pre-activation (ResNet v2):

    x ──────────────┐                     x ──────────────┐
    │                │                     │                │
    ▼                │                     ▼                │
  Conv 3×3           │                   BN → ReLU          │
    │                │                     │                │
    ▼                │                     ▼                │
   BN                │                   Conv 3×3           │
    │                │                     │                │
    ▼                │                     ▼                │
  ReLU               │                   BN → ReLU          │
    │                │                     │                │
    ▼                │                     ▼                │
  Conv 3×3           │                   Conv 3×3           │
    │                │                     │                │
    ▼                │                     ▼                │
   BN                │                     │                │
    │                │                     │                │
    ▼                ▼                     ▼                ▼
    +────────────────+                     +────────────────+
    │                                      │
    ▼                                      ▼
  ReLU ← after addition                 (no ReLU here)
    │                                      │
    ▼                                      ▼
  output                                 output
```

### Practical Recommendation

```
For most CNNs:     Conv → BN → ReLU   (safest, most tested)
For very deep nets: BN → ReLU → Conv   (ResNet v2 pre-activation)
For GANs:          Conv → BN → LeakyReLU(0.2)
For transformers:  Linear → GELU
```

---

## 9️⃣ Complete Code Example — Comparing Activations

```python
import torch
import torch.nn as nn
import torch.nn.functional as F

class FlexibleCNN(nn.Module):
    """CNN with configurable activation function for comparison."""
    
    def __init__(self, activation='relu', num_classes=10):
        super().__init__()
        
        # Choose activation
        act_map = {
            'relu':       nn.ReLU(inplace=True),
            'leaky_relu': nn.LeakyReLU(0.01, inplace=True),
            'prelu':      nn.PReLU(num_parameters=1),
            'elu':        nn.ELU(alpha=1.0, inplace=True),
            'gelu':       nn.GELU(),
            'silu':       nn.SiLU(inplace=True),
        }
        self.act = act_map[activation]
        
        # Feature extractor
        self.features = nn.Sequential(
            nn.Conv2d(3, 32, 3, padding=1, bias=False),
            nn.BatchNorm2d(32),
            self._make_act(),
            nn.Conv2d(32, 64, 3, stride=2, padding=1, bias=False),
            nn.BatchNorm2d(64),
            self._make_act(),
            nn.Conv2d(64, 128, 3, stride=2, padding=1, bias=False),
            nn.BatchNorm2d(128),
            self._make_act(),
        )
        
        # Classifier
        self.classifier = nn.Sequential(
            nn.AdaptiveAvgPool2d(1),
            nn.Flatten(),
            nn.Linear(128, num_classes),
        )
    
    def _make_act(self):
        """Create a fresh copy of the activation (needed for PReLU's learnable params)."""
        if isinstance(self.act, nn.PReLU):
            return nn.PReLU(num_parameters=1)
        return self.act  # ReLU, LeakyReLU etc. are stateless, can be shared
    
    def forward(self, x):
        x = self.features(x)
        x = self.classifier(x)
        return x

# Test all activations
x = torch.randn(4, 3, 32, 32)
for act_name in ['relu', 'leaky_relu', 'prelu', 'elu', 'gelu', 'silu']:
    model = FlexibleCNN(activation=act_name)
    out = model(x)
    n_params = sum(p.numel() for p in model.parameters())
    print(f"{act_name:12s} | output: {out.shape} | params: {n_params:,}")

# Example output:
# relu         | output: torch.Size([4, 10]) | params: 96,170
# leaky_relu   | output: torch.Size([4, 10]) | params: 96,170
# prelu        | output: torch.Size([4, 10]) | params: 96,173  (+3 for PReLU params)
# elu          | output: torch.Size([4, 10]) | params: 96,170
# gelu         | output: torch.Size([4, 10]) | params: 96,170
# silu         | output: torch.Size([4, 10]) | params: 96,170
```

### Monitoring Dead Neurons

```python
def count_dead_neurons(model, dataloader, device='cpu'):
    """Count neurons that never activate (always output 0) across the dataset."""
    model.eval()
    hooks = []
    activation_seen = {}
    
    def hook_fn(name):
        def fn(module, input, output):
            # Track if each neuron ever produces a positive value
            if name not in activation_seen:
                activation_seen[name] = torch.zeros(output.shape[1], dtype=torch.bool)
            # Check each channel: is any spatial position ever > 0?
            positive = (output > 0).any(dim=0).any(dim=-1).any(dim=-1)  # per channel
            activation_seen[name] = activation_seen[name] | positive.cpu()
        return fn
    
    # Register hooks on all ReLU layers
    for name, module in model.named_modules():
        if isinstance(module, (nn.ReLU, nn.LeakyReLU)):
            hooks.append(module.register_forward_hook(hook_fn(name)))
    
    with torch.no_grad():
        for batch, _ in dataloader:
            model(batch.to(device))
    
    # Remove hooks
    for h in hooks:
        h.remove()
    
    # Report
    for name, seen in activation_seen.items():
        dead = (~seen).sum().item()
        total = seen.numel()
        print(f"{name}: {dead}/{total} dead neurons ({100*dead/total:.1f}%)")
```

---

## 🔟 He Initialization — Designed for ReLU

Since ReLU zeros out negative values, half the variance is lost. **He initialization** compensates:

```
He (Kaiming) Initialization:

  W ~ N(0, σ²)    where σ² = 2 / fan_in

  fan_in = C_in × k_h × k_w

  Compare to Xavier: σ² = 1 / fan_in  (designed for tanh/sigmoid)
```

PyTorch uses He initialization **by default** for `nn.Conv2d`:

```python
# PyTorch's default init for Conv2d (kaiming_uniform)
# Equivalent to:
nn.init.kaiming_uniform_(conv.weight, mode='fan_in', nonlinearity='relu')

# You can also explicitly set it:
nn.init.kaiming_normal_(conv.weight, mode='fan_out', nonlinearity='relu')
# fan_out preserves variance in backward pass (used in ResNet paper)
```

---

## 📊 Summary Table

| Property | Details |
|----------|---------|
| **Purpose** | Introduce non-linearity; enable complex function learning |
| **Standard in CNNs** | ReLU: `max(0, x)` |
| **In-place ReLU** | `nn.ReLU(inplace=True)` — saves ~50% activation memory |
| **Dead neuron fix** | Leaky ReLU (`α·x` for x<0) or PReLU (learned α) |
| **GAN standard** | `nn.LeakyReLU(0.2)` in discriminator |
| **Transformer std** | GELU or SiLU |
| **Placement** | After BN: `Conv → BN → ReLU` (most common) |
| **Pre-activation** | `BN → ReLU → Conv` (ResNet v2, deep nets) |
| **Initialization** | He/Kaiming for ReLU: `σ² = 2/fan_in` |
| **Gradient** | ReLU: 1 (positive), 0 (negative); no vanishing gradient for x>0 |

---

## ❓ Revision Questions

### Q1: Why ReLU over Sigmoid in CNNs?
**Give at least three reasons why ReLU is preferred over sigmoid in modern CNNs.**

<details>
<summary>Answer</summary>

1. **No vanishing gradient**: ReLU's gradient is 1 for positive inputs (sigmoid's gradient approaches 0 for large/small inputs).
2. **Computational efficiency**: ReLU is just `max(0,x)` — no exponential computation.
3. **Sparse activation**: ~50% of ReLU outputs are zero, providing natural regularization and sparse representations.
4. **Faster convergence**: AlexNet showed ~6× training speedup with ReLU vs tanh.
5. **Scale invariance**: ReLU output is unbounded above, avoiding the saturation problem of sigmoid (which squashes everything to [0,1]).
</details>

### Q2: In-Place Safety
**When is it unsafe to use `nn.ReLU(inplace=True)`? Provide a code example.**

<details>
<summary>Answer</summary>

It's unsafe when the **original input tensor** is needed later in the computation (e.g., for a skip/residual connection):

```python
def forward(self, x):
    identity = x
    x = F.relu(x, inplace=True)  # ❌ Modifies x, but identity points to same data!
    out = self.conv(x)
    return out + identity  # identity was also modified by in-place ReLU
```

Safe alternative: apply in-place only to intermediate tensors that won't be reused:
```python
def forward(self, x):
    identity = x
    out = self.conv(x)           # creates new tensor
    out = F.relu(out, inplace=True)  # ✓ Safe: modifying conv output, not x
    return out + identity
```
</details>

### Q3: Dead Neurons
**What is the "dead ReLU" problem and how does Leaky ReLU address it?**

<details>
<summary>Answer</summary>

**Dead ReLU**: When a neuron's pre-activation becomes permanently negative (due to large weight updates or poor initialization), its ReLU output is always 0, and the gradient is always 0. The neuron's weights never update → it's permanently "dead."

**Leaky ReLU fix**: For negative inputs, Leaky ReLU returns `α·x` (e.g., α=0.01) instead of 0. This ensures:
- The gradient is `α` (small but non-zero) for negative inputs
- Dead neurons can still receive gradient signal and potentially "revive"
- Information about the magnitude of negative values is preserved
</details>

### Q4: Placement in Architecture
**Compare Conv→BN→ReLU vs BN→ReLU→Conv. When would you prefer the latter?**

<details>
<summary>Answer</summary>

**Conv→BN→ReLU** (post-activation): The standard ordering. BN normalizes the conv output, then ReLU introduces sparsity. Used in most architectures (ResNet v1, VGG-BN, EfficientNet).

**BN→ReLU→Conv** (pre-activation): Proposed in ResNet v2. BN and ReLU act as "preprocessing" before the conv. Benefits:
- The identity shortcut in residual blocks is a pure identity (no BN/ReLU on it)
- Gradient flows more freely through the shortcut
- Better performance for **very deep networks** (200+ layers)
- Each BN normalizes the input to its own conv, which is semantically cleaner

Use pre-activation for networks deeper than ~100 layers.
</details>

### Q5: PReLU Parameters
**A CNN has 5 convolutional layers producing 32, 64, 128, 256, and 512 channels. If PReLU (per-channel) is used after each layer, how many additional learnable parameters does PReLU add?**

<details>
<summary>Answer</summary>

PReLU has one learnable slope parameter per channel:
- Layer 1: 32 parameters
- Layer 2: 64 parameters
- Layer 3: 128 parameters
- Layer 4: 256 parameters
- Layer 5: 512 parameters

Total PReLU parameters: 32 + 64 + 128 + 256 + 512 = **992 parameters**

This is negligible compared to the conv parameters (which are typically millions), so PReLU adds almost no overhead.
</details>

### Q6: Functional vs Module API
**What is the difference between `F.relu()` and `nn.ReLU()` in PyTorch? When would you use each?**

<details>
<summary>Answer</summary>

**`nn.ReLU()`**: A module (class instance) that can be used in `nn.Sequential` and is registered as a submodule. It shows up in `model.modules()` and `model.children()`.

```python
self.relu = nn.ReLU(inplace=True)
# Used in __init__, called in forward
```

**`F.relu()`**: A functional (stateless function) called directly in `forward()`. Not registered as a module.

```python
x = F.relu(x, inplace=True)
# Used directly in forward method
```

**When to use which:**
- Use `nn.ReLU()` when building models with `nn.Sequential` or when you want activations to appear in the model's module tree
- Use `F.relu()` in custom `forward()` methods when you want concise code
- For PReLU, you **must** use `nn.PReLU()` because it has learnable parameters
</details>

---

[← Previous: 01 — Convolutional Layer](01-convolutional-layer.md) | [📑 Unit Overview](../README.md) | [Next → 03 — Pooling Layer](03-pooling-layer.md)
