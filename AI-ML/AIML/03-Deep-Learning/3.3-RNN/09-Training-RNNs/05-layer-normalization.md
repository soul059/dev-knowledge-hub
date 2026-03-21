# Layer Normalization in RNNs

> **Unit 9, Chapter 5** — Why LayerNorm is preferred over BatchNorm for RNNs, per-time-step normalization, and PyTorch implementation.

---

## 📋 Overview

Normalization techniques stabilize training by controlling the distribution of activations. While **Batch Normalization** revolutionized CNN training, it works poorly with RNNs due to variable sequence lengths and small effective batch sizes per time step. **Layer Normalization** (Ba et al., 2016) normalizes across features within each sample independently, making it the preferred normalization for recurrent architectures.

---

## ❌ Why BatchNorm Fails in RNNs

### BatchNorm Recap

BatchNorm normalizes across the **batch dimension** for each feature:

```
Batch of 4 samples, 3 features:

         Feature 1    Feature 2    Feature 3
Sample 1:   2.1          0.5          -1.3
Sample 2:   1.8          0.7          -0.9
Sample 3:   2.4          0.3          -1.5
Sample 4:   1.9          0.6          -1.1
              ↓            ↓            ↓
           mean=2.05    mean=0.525   mean=-1.2
           std=0.22     std=0.15     std=0.22
              ↓            ↓            ↓
          Normalize    Normalize    Normalize
          each column  each column  each column
```

### Problems with BatchNorm in RNNs

```
Problem 1: Variable Sequence Lengths
──────────────────────────────────────
Batch at time step t=10:

  Sample 1: [h₁₀]     ← still active (length 20)
  Sample 2: [h₁₀]     ← still active (length 15)
  Sample 3: [PAD]      ← already ended (length 8)
  Sample 4: [PAD]      ← already ended (length 6)

BatchNorm statistics corrupted by padding!
Effective batch size shrinks over time steps.

Problem 2: Different Statistics Per Time Step
──────────────────────────────────────
  t=1:  h₁ statistics ← freshly initialized, mostly zeros
  t=5:  h₅ statistics ← moderate activations
  t=20: h₂₀ statistics ← saturated activations

Need SEPARATE BatchNorm parameters for each time step?
  → Defeats the purpose of parameter sharing!

Problem 3: Test-Time Running Statistics
──────────────────────────────────────
Which running mean/var to use at test time?
Different for each time step?
What if test sequence is longer than any training sequence?
  → No running statistics exist for those time steps!
```

---

## ✅ Layer Normalization

### Core Idea

Instead of normalizing across the batch, normalize across the **feature dimension** within each individual sample and time step:

```
BatchNorm: normalizes across samples (↓ column-wise)
LayerNorm: normalizes across features (→ row-wise)

         Feature 1    Feature 2    Feature 3
Sample 1:   2.1          0.5          -1.3   → mean=0.43, std=1.40 → normalize
Sample 2:   1.8          0.7          -0.9   → mean=0.53, std=1.10 → normalize
Sample 3:   2.4          0.3          -1.5   → mean=0.40, std=1.60 → normalize
Sample 4:   1.9          0.6          -1.1   → mean=0.47, std=1.24 → normalize
```

### Mathematical Formulation

For hidden state h at time step t, with D features:

```
Given: h_t = [h₁, h₂, ..., h_D]

Step 1: Compute mean across features
  μ = (1/D) Σᵢ hᵢ

Step 2: Compute variance across features  
  σ² = (1/D) Σᵢ (hᵢ - μ)²

Step 3: Normalize
  ĥᵢ = (hᵢ - μ) / √(σ² + ε)

Step 4: Scale and shift (learnable parameters)
  yᵢ = γ · ĥᵢ + β

where γ (gain) and β (bias) are learnable parameters of size D.
```

### Key Advantage

```
┌──────────────────────────────────────────────────────────────┐
│  LayerNorm operates on EACH SAMPLE INDEPENDENTLY            │
│                                                              │
│  ✅ No dependency on batch size                              │
│  ✅ No dependency on other samples                           │
│  ✅ Same computation at train and test time                   │
│  ✅ Works with any sequence length                            │
│  ✅ Same parameters shared across all time steps              │
└──────────────────────────────────────────────────────────────┘
```

---

## 📐 LayerNorm in RNN: Where to Apply

### Option 1: Pre-Activation LayerNorm (Recommended)

Normalize the pre-activation (before the nonlinearity):

```
Standard RNN:
  h_t = tanh(W_xh · x_t + W_hh · h_{t-1} + b)

With LayerNorm:
  a_t = W_xh · x_t + W_hh · h_{t-1}     ← linear combination
  â_t = LayerNorm(a_t)                     ← normalize
  h_t = tanh(â_t)                          ← nonlinearity
```

### Option 2: Separate Normalization (Ba et al., 2016)

Normalize input and recurrent contributions separately:

```
  a_t = LayerNorm₁(W_xh · x_t) + LayerNorm₂(W_hh · h_{t-1})
  h_t = tanh(a_t)
```

### ASCII Diagram: LayerNorm RNN Cell

```
                    ┌─────────────┐
    x_t ──────────→ │   W_xh · x  │──┐
                    └─────────────┘  │
                                     ├──→ [+] ──→ [LayerNorm] ──→ [tanh] ──→ h_t
                    ┌─────────────┐  │                                        │
    h_{t-1} ──────→ │  W_hh · h   │──┘                                        │
        ↑           └─────────────┘                                            │
        │                                                                      │
        └──────────────────────────────────────────────────────────────────────┘
                                    (next time step)
```

---

## 💻 PyTorch Implementation

### LayerNorm RNN Cell

```python
import torch
import torch.nn as nn

class LayerNormRNNCell(nn.Module):
    """RNN cell with Layer Normalization."""
    
    def __init__(self, input_size, hidden_size):
        super().__init__()
        self.input_size = input_size
        self.hidden_size = hidden_size
        
        # Linear transformations (no bias — LayerNorm has its own)
        self.W_xh = nn.Linear(input_size, hidden_size, bias=False)
        self.W_hh = nn.Linear(hidden_size, hidden_size, bias=False)
        
        # Layer normalization
        self.layer_norm = nn.LayerNorm(hidden_size)
    
    def forward(self, x, h_prev):
        """
        Args:
            x: input at time t [batch, input_size]
            h_prev: hidden state from t-1 [batch, hidden_size]
        Returns:
            h: new hidden state [batch, hidden_size]
        """
        # Linear combination
        a = self.W_xh(x) + self.W_hh(h_prev)
        
        # Layer normalization
        a_norm = self.layer_norm(a)
        
        # Nonlinearity
        h = torch.tanh(a_norm)
        
        return h
```

### LayerNorm LSTM Cell

```python
class LayerNormLSTMCell(nn.Module):
    """LSTM cell with Layer Normalization on each gate."""
    
    def __init__(self, input_size, hidden_size):
        super().__init__()
        self.input_size = input_size
        self.hidden_size = hidden_size
        
        # Combined linear transforms for all 4 gates
        self.W_x = nn.Linear(input_size, 4 * hidden_size, bias=False)
        self.W_h = nn.Linear(hidden_size, 4 * hidden_size, bias=False)
        
        # Separate LayerNorm for each gate
        self.ln_f = nn.LayerNorm(hidden_size)  # forget gate
        self.ln_i = nn.LayerNorm(hidden_size)  # input gate
        self.ln_g = nn.LayerNorm(hidden_size)  # candidate
        self.ln_o = nn.LayerNorm(hidden_size)  # output gate
        
        # LayerNorm for cell state (applied before output gate)
        self.ln_c = nn.LayerNorm(hidden_size)
    
    def forward(self, x, state):
        """
        Args:
            x: [batch, input_size]
            state: tuple of (h, c), each [batch, hidden_size]
        """
        h_prev, c_prev = state
        
        # Compute all gates at once
        gates_x = self.W_x(x)
        gates_h = self.W_h(h_prev)
        gates = gates_x + gates_h
        
        # Split into 4 gates
        f_pre, i_pre, g_pre, o_pre = gates.chunk(4, dim=1)
        
        # Apply LayerNorm to each gate separately
        f = torch.sigmoid(self.ln_f(f_pre))  # forget gate
        i = torch.sigmoid(self.ln_i(i_pre))  # input gate
        g = torch.tanh(self.ln_g(g_pre))      # candidate
        o = torch.sigmoid(self.ln_o(o_pre))  # output gate
        
        # Cell state update
        c = f * c_prev + i * g
        
        # Hidden state (with LayerNorm on cell state)
        h = o * torch.tanh(self.ln_c(c))
        
        return h, c


class LayerNormLSTM(nn.Module):
    """Full LayerNorm LSTM supporting sequences."""
    
    def __init__(self, input_size, hidden_size, num_layers=1):
        super().__init__()
        self.hidden_size = hidden_size
        self.num_layers = num_layers
        
        self.cells = nn.ModuleList([
            LayerNormLSTMCell(
                input_size if i == 0 else hidden_size,
                hidden_size
            ) for i in range(num_layers)
        ])
    
    def forward(self, x, hidden=None):
        """
        Args:
            x: [batch, seq_len, input_size]
        Returns:
            output: [batch, seq_len, hidden_size]
            (h_n, c_n): final hidden and cell states
        """
        batch_size, seq_len, _ = x.size()
        
        if hidden is None:
            h = [torch.zeros(batch_size, self.hidden_size, device=x.device)
                 for _ in range(self.num_layers)]
            c = [torch.zeros(batch_size, self.hidden_size, device=x.device)
                 for _ in range(self.num_layers)]
        else:
            h, c = hidden
        
        outputs = []
        for t in range(seq_len):
            inp = x[:, t, :]
            for layer in range(self.num_layers):
                h[layer], c[layer] = self.cells[layer](inp, (h[layer], c[layer]))
                inp = h[layer]
            outputs.append(h[-1])
        
        output = torch.stack(outputs, dim=1)
        return output, (h, c)

# Usage
model = LayerNormLSTM(input_size=100, hidden_size=256, num_layers=2)
x = torch.randn(32, 50, 100)  # batch=32, seq_len=50, features=100
output, (h, c) = model(x)
print(output.shape)  # [32, 50, 256]
```

---

## 🔬 BatchNorm vs LayerNorm vs GroupNorm for RNNs

```
┌───────────────────────────────────────────────────────────────┐
│                    Normalization Comparison                    │
├──────────────┬──────────────┬──────────────┬─────────────────┤
│              │  BatchNorm   │  LayerNorm   │  GroupNorm      │
├──────────────┼──────────────┼──────────────┼─────────────────┤
│ Normalizes   │ Across batch │ Across feats │ Across feat     │
│ across       │ (per feature)│ (per sample) │ groups          │
├──────────────┼──────────────┼──────────────┼─────────────────┤
│ Batch size   │ Needs large  │ Independent  │ Independent     │
│ dependency   │ batch        │              │                 │
├──────────────┼──────────────┼──────────────┼─────────────────┤
│ Seq length   │ Problematic  │ No issue     │ No issue        │
│ dependency   │              │              │                 │
├──────────────┼──────────────┼──────────────┼─────────────────┤
│ Train/test   │ Different    │ Same         │ Same            │
│ behavior     │ (running avg)│              │                 │
├──────────────┼──────────────┼──────────────┼─────────────────┤
│ RNN suitab.  │ ❌ Poor      │ ✅ Excellent  │ ✅ Good          │
├──────────────┼──────────────┼──────────────┼─────────────────┤
│ CNN suitab.  │ ✅ Excellent  │ ⚠️ OK        │ ✅ Good          │
└──────────────┴──────────────┴──────────────┴─────────────────┘
```

---

## 🧮 Worked Example: LayerNorm Computation

### Setup

```
Hidden state at time t=3:
  h₃ = [2.0, -1.0, 4.0, 1.0]    (D = 4 features)

Step 1: Mean across features
  μ = (2.0 + (-1.0) + 4.0 + 1.0) / 4 = 6.0 / 4 = 1.5

Step 2: Variance
  σ² = [(2.0-1.5)² + (-1.0-1.5)² + (4.0-1.5)² + (1.0-1.5)²] / 4
     = [0.25 + 6.25 + 6.25 + 0.25] / 4
     = 13.0 / 4 = 3.25
  
  σ = √3.25 ≈ 1.803

Step 3: Normalize (ε = 1e-5)
  ĥ₁ = (2.0 - 1.5) / 1.803 =  0.277
  ĥ₂ = (-1.0 - 1.5) / 1.803 = -1.387
  ĥ₃ = (4.0 - 1.5) / 1.803 =  1.387
  ĥ₄ = (1.0 - 1.5) / 1.803 = -0.277

  Verify: mean(ĥ) = 0 ✓, std(ĥ) = 1 ✓

Step 4: Scale and shift (learnable γ, β)
  Assuming γ = [1, 1, 1, 1], β = [0, 0, 0, 0] (initial values):
  y = ĥ (no change with default initialization)
```

---

## ⚡ Performance Impact

### Training Speed Comparison

```
Epochs to reach target perplexity on Penn Treebank:

              Without LN    With LayerNorm
LSTM:            40              28          (30% faster)
GRU:             38              25          (34% faster)  
Deep LSTM:       55              32          (42% faster)

LayerNorm benefit increases with model depth!

Note: Per-step computation is ~5-10% slower due to
normalization overhead, but convergence is much faster,
so total wall-clock time is reduced.
```

### When LayerNorm Helps Most

```
✅ Very helpful:
  • Deep RNNs (3+ layers)
  • Long sequences (100+ time steps)
  • Large hidden sizes (512+)
  • Training from scratch on small data

⚠️ Moderate help:
  • Shallow RNNs (1-2 layers)
  • Short sequences
  • Pre-trained models (fine-tuning)

❌ May not help:
  • Very small models
  • Already well-regularized (heavy dropout)
  • When BatchNorm works (unusual for RNNs)
```

---

## 📋 Summary Table

| Concept | Key Takeaway |
|---------|-------------|
| BatchNorm in RNNs | Fails due to variable lengths, batch dependency, per-step stats |
| LayerNorm | Normalizes across features per sample — ideal for RNNs |
| Where to apply | Pre-activation: before tanh/sigmoid in RNN/LSTM cell |
| LSTM LayerNorm | Apply separately to each gate's pre-activation |
| Cell state LN | Normalize cell state before output gate |
| Parameters | γ (gain) and β (bias), same dimensionality as hidden |
| Train vs test | Identical computation — no running statistics needed |
| Speed impact | ~5-10% slower per step, 30-40% fewer epochs to converge |

---

## ❓ Revision Questions

1. **Explain why BatchNorm fails for RNNs but works well for CNNs.** What specific properties of RNNs cause the failure?

2. **In a LayerNorm LSTM, why do we use separate LayerNorm modules for each gate** rather than one shared LayerNorm?

3. **Given h = [3, -1, 2, 0], compute the LayerNorm output** with γ = [1, 1, 1, 1] and β = [0, 0, 0, 0].

4. **Compare pre-activation LayerNorm with post-activation LayerNorm.** Which is typically used and why?

5. **LayerNorm normalizes to zero mean and unit variance.** Could this be harmful for LSTM cell states that need to maintain magnitudes over time? How is this addressed?

6. **Would you combine LayerNorm with variational dropout?** Are they complementary or redundant?

---

## 🧭 Navigation

| Direction | Link |
|-----------|------|
| ⬅️ Previous | [Dropout in RNNs](04-dropout-in-rnns.md) |
| ⬆️ Unit | [09 Training RNNs](../README.md) |
| ➡️ Next Unit | [Beyond RNNs — Attention Mechanisms](../10-Beyond-RNNs/01-attention-mechanisms.md) |
| 🏠 Home | [RNN Home](../README.md) |

---

*Last updated: 2024*
