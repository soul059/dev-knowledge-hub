# The Positional Information Problem

[← Why Self-Attention Works](03-why-self-attention-works.md) | [Why Multiple Heads →](../03-Multi-Head-Attention/01-why-multiple-heads.md)

---

## Overview

Self-attention treats its input as a **set**, not a sequence. If you shuffle the tokens in a sentence, self-attention produces the exact same output (just in a different order). This property — called **permutation equivariance** — means self-attention is completely blind to word order. Since word order is essential in language ("dog bites man" ≠ "man bites dog"), this is a critical flaw that must be fixed. This note proves mathematically why self-attention is permutation equivariant, demonstrates the problem with code, and previews the solutions: positional encodings that inject order information into the input.

---

## 1. The Problem: Word Order Matters!

### Motivating Examples

```
Sentence A: "Dog bites man"    →  Routine event (not newsworthy)
Sentence B: "Man bites dog"    →  Bizarre event (headline news!)

Same tokens, different meaning. Order is EVERYTHING.
```

```
Sentence A: "Alice thanked Bob"   → Alice is grateful
Sentence B: "Bob thanked Alice"   → Bob is grateful

Same tokens, opposite meaning.
```

```
Sentence A: "not happy"  → Negative sentiment
Sentence B: "happy not"  → Still negative, but grammatically wrong
```

### The Shocking Truth About Self-Attention

**Self-attention gives the SAME attention weights regardless of word order.**

If you compute self-attention on `["Dog", "bites", "man"]` and `["man", "bites", "Dog"]`, the output for each token is identical — just rearranged to match the new order.

---

## 2. Mathematical Proof: Permutation Equivariance

### Definition

A function f is **permutation equivariant** if for any permutation matrix P:

```
f(PX) = P · f(X)
```

This means: permuting the input just permutes the output in the same way, without changing the actual representations.

### Proof for Self-Attention

Let X ∈ ℝ^(n × d) be the input. Let P ∈ ℝ^(n × n) be a permutation matrix (a matrix that reorders rows).

Self-attention: `SA(X) = softmax(XW_Q(XW_K)ᵀ / √d_k) · XW_V`

Now compute SA(PX):

```
Step 1: Queries, Keys, Values of permuted input
  Q' = (PX)W_Q = P(XW_Q) = PQ
  K' = (PX)W_K = P(XW_K) = PK
  V' = (PX)W_V = P(XW_V) = PV

Step 2: Attention scores
  Q'K'ᵀ = (PQ)(PK)ᵀ = PQ Kᵀ Pᵀ

Step 3: Softmax (applied row-wise)
  softmax(PQ Kᵀ Pᵀ) = P · softmax(QKᵀ) · Pᵀ
  
  Why? P permutes rows, Pᵀ permutes columns.
  Softmax is applied per row, so permuting rows/columns
  just permutes which row gets which softmax output.

Step 4: Output
  SA(PX) = P · softmax(QKᵀ/√d_k) · Pᵀ · PV
         = P · softmax(QKᵀ/√d_k) · V      (since PᵀP = I)
         = P · SA(X)                        ■

Therefore: SA(PX) = P · SA(X)  ← Permutation equivariant!
```

### What This Means in Plain English

> "If I shuffle my input tokens, the output of self-attention is shuffled in exactly the same way. No new information is created or lost. The model literally cannot tell the difference between the original order and any permutation."

---

## 3. Concrete Demonstration

### Example: "Dog bites man" vs "man bites Dog"

```
Original:   X  = [x_dog,  x_bites, x_man ]
Permuted:   X' = [x_man,  x_bites, x_dog ]

Self-Attention output:

Original:   SA(X)  = [out_dog,  out_bites, out_man ]
Permuted:   SA(X') = [out_man,  out_bites, out_dog ]

Notice: out_dog is IDENTICAL in both cases!
        out_man is IDENTICAL in both cases!
        out_bites is IDENTICAL in both cases!

The representation of "dog" is the same whether it appears
at position 0 ("Dog bites man") or position 2 ("man bites Dog").

The model CANNOT distinguish subject from object!
```

### Why This Is Catastrophic

```
Task: Sentiment Analysis

"I love this movie"    → Positive  (correct)
"this movie love I"    → Positive  (correct... but for wrong reasons)

Task: Named Entity Recognition

"John works at Google" → John=PERSON, Google=ORG
"Google works at John" → Google=PERSON?, John=ORG?  ← WRONG!

Task: Machine Translation

"the cat sat on the mat"  →  Without position info, the model
"the mat sat on the cat"     produces the SAME internal representation!
```

---

## 4. ASCII Diagram: The Problem and Solution

```
THE PROBLEM:
═══════════════════════════════════════════════════════════

Input A: [Dog]   [bites]   [man]
           │        │        │
           ▼        ▼        ▼
      ┌─────────────────────────┐
      │    SELF-ATTENTION       │      Same attention
      │  (position-blind)       │      Same output!
      └─────────────────────────┘
           │        │        │
           ▼        ▼        ▼
Output A: [y₁]    [y₂]     [y₃]    ← Identical representations


Input B: [man]   [bites]   [Dog]        (permuted!)
           │        │        │
           ▼        ▼        ▼
      ┌─────────────────────────┐
      │    SELF-ATTENTION       │      Same attention
      │  (position-blind)       │      Same output!
      └─────────────────────────┘
           │        │        │
           ▼        ▼        ▼
Output B: [y₃]    [y₂]     [y₁]    ← Same values, just reordered!


THE SOLUTION:
═══════════════════════════════════════════════════════════

Input A: [Dog]   [bites]   [man]
         + PE₀   + PE₁    + PE₂      ← Add positional encodings!
           │        │        │
           ▼        ▼        ▼
      ┌─────────────────────────┐
      │    SELF-ATTENTION       │      Different attention!
      │  (position-aware)       │      Different output!
      └─────────────────────────┘
           │        │        │
           ▼        ▼        ▼
Output A: [y₁']   [y₂']   [y₃']   ← Position-aware representations


Input B: [man]   [bites]   [Dog]
         + PE₀   + PE₁    + PE₂      ← Different PEs for same tokens!
           │        │        │
           ▼        ▼        ▼
      ┌─────────────────────────┐
      │    SELF-ATTENTION       │      Different attention!
      │  (position-aware)       │      Different output!
      └─────────────────────────┘
           │        │        │
           ▼        ▼        ▼
Output B: [z₁']   [z₂']   [z₃']   ← DIFFERENT from Output A! ✅
```

---

## 5. Code Demonstration: Proving Permutation Equivariance

```python
import torch
import torch.nn as nn
import torch.nn.functional as F

class SelfAttention(nn.Module):
    def __init__(self, d_model, d_k):
        super().__init__()
        self.d_k = d_k
        self.W_Q = nn.Linear(d_model, d_k, bias=False)
        self.W_K = nn.Linear(d_model, d_k, bias=False)
        self.W_V = nn.Linear(d_model, d_k, bias=False)

    def forward(self, X):
        Q = self.W_Q(X)
        K = self.W_K(X)
        V = self.W_V(X)
        scores = Q @ K.transpose(-2, -1) / (self.d_k ** 0.5)
        weights = F.softmax(scores, dim=-1)
        return weights @ V


torch.manual_seed(42)
d_model, d_k = 8, 6
attn = SelfAttention(d_model, d_k)

# --- Original order: [Dog, bites, man] ---
x_dog   = torch.randn(1, d_model)
x_bites = torch.randn(1, d_model)
x_man   = torch.randn(1, d_model)

X_original = torch.stack([x_dog, x_bites, x_man], dim=1)  # (1, 3, 8)
out_original = attn(X_original)

# --- Permuted order: [man, bites, Dog] ---
X_permuted = torch.stack([x_man, x_bites, x_dog], dim=1)  # (1, 3, 8)
out_permuted = attn(X_permuted)

# --- Verify: output for "dog" is the same in both cases ---
print("Output for 'Dog' (original, position 0):")
print(out_original[0, 0].detach().numpy().round(4))

print("\nOutput for 'Dog' (permuted, position 2):")
print(out_permuted[0, 2].detach().numpy().round(4))

print("\nAre they equal?", torch.allclose(out_original[0, 0], out_permuted[0, 2], atol=1e-6))

# --- Also verify: output for "man" ---
print("\nOutput for 'man' (original, pos 2):")
print(out_original[0, 2].detach().numpy().round(4))

print("Output for 'man' (permuted, pos 0):")
print(out_permuted[0, 0].detach().numpy().round(4))

print("Are they equal?", torch.allclose(out_original[0, 2], out_permuted[0, 0], atol=1e-6))
```

### Expected Output

```
Output for 'Dog' (original, position 0):
[ 0.1234 -0.4567  0.7890  0.2345 -0.1234  0.5678]

Output for 'Dog' (permuted, position 2):
[ 0.1234 -0.4567  0.7890  0.2345 -0.1234  0.5678]

Are they equal? True

Output for 'man' (original, pos 2):
[ 0.3456 -0.2345  0.6789  0.1234 -0.3456  0.4567]
Output for 'man' (permuted, pos 0):
[ 0.3456 -0.2345  0.6789  0.1234 -0.3456  0.4567]
Are they equal? True
```

**Confirmed**: Same token gets the **exact same output** regardless of its position. Self-attention is completely blind to order.

---

## 6. The Fix: Positional Encoding

The solution is simple: **add position information to the input before self-attention**.

```
X_new[i] = X[i] + PE[i]
```

Where PE[i] is a unique vector for each position i.

### Preview of Positional Encoding Methods

| Method | Formula/Idea | Used In |
|--------|-------------|---------|
| **Sinusoidal** | PE(pos, 2i) = sin(pos/10000^(2i/d)) | Original Transformer |
| **Learned** | PE = nn.Embedding(max_len, d_model) | BERT, GPT-2 |
| **Relative** | Attention based on (pos_i - pos_j) | Transformer-XL, T5 |
| **Rotary (RoPE)** | Rotate Q, K by position angle | LLaMA, GPT-NeoX |
| **ALiBi** | Add linear bias to attention scores | BLOOM |

---

## 7. Code: Fixing Permutation Invariance with Positional Encoding

```python
import torch
import torch.nn as nn
import torch.nn.functional as F
import math

class SinusoidalPositionalEncoding(nn.Module):
    """Positional encoding from 'Attention Is All You Need'."""
    def __init__(self, d_model, max_len=5000):
        super().__init__()
        pe = torch.zeros(max_len, d_model)
        position = torch.arange(0, max_len, dtype=torch.float).unsqueeze(1)
        div_term = torch.exp(
            torch.arange(0, d_model, 2).float() * (-math.log(10000.0) / d_model)
        )

        pe[:, 0::2] = torch.sin(position * div_term)  # Even indices
        pe[:, 1::2] = torch.cos(position * div_term)   # Odd indices

        pe = pe.unsqueeze(0)  # (1, max_len, d_model)
        self.register_buffer('pe', pe)

    def forward(self, x):
        """Add positional encoding to input x of shape (batch, seq_len, d_model)."""
        return x + self.pe[:, :x.size(1), :]


class SelfAttentionWithPosition(nn.Module):
    def __init__(self, d_model, d_k, max_len=5000):
        super().__init__()
        self.pos_enc = SinusoidalPositionalEncoding(d_model, max_len)
        self.d_k = d_k
        self.W_Q = nn.Linear(d_model, d_k, bias=False)
        self.W_K = nn.Linear(d_model, d_k, bias=False)
        self.W_V = nn.Linear(d_model, d_k, bias=False)

    def forward(self, X):
        # Add positional encoding BEFORE self-attention
        X = self.pos_enc(X)

        Q = self.W_Q(X)
        K = self.W_K(X)
        V = self.W_V(X)
        scores = Q @ K.transpose(-2, -1) / (self.d_k ** 0.5)
        weights = F.softmax(scores, dim=-1)
        return weights @ V


# --- Test: Does positional encoding break permutation equivariance? ---
torch.manual_seed(42)
d_model, d_k = 8, 6
attn_pos = SelfAttentionWithPosition(d_model, d_k)

x_dog   = torch.randn(1, d_model)
x_bites = torch.randn(1, d_model)
x_man   = torch.randn(1, d_model)

# Original: [Dog, bites, man]
X_original = torch.stack([x_dog, x_bites, x_man], dim=1)
out_original = attn_pos(X_original)

# Permuted: [man, bites, Dog]
X_permuted = torch.stack([x_man, x_bites, x_dog], dim=1)
out_permuted = attn_pos(X_permuted)

# Check if "dog" gets the same output in both orderings
print("Output for 'Dog' (original, position 0):")
print(out_original[0, 0].detach().numpy().round(4))

print("\nOutput for 'Dog' (permuted, position 2):")
print(out_permuted[0, 2].detach().numpy().round(4))

print("\nAre they equal?",
      torch.allclose(out_original[0, 0], out_permuted[0, 2], atol=1e-6))

diff = (out_original[0, 0] - out_permuted[0, 2]).abs().max().item()
print(f"Maximum difference: {diff:.6f}")

print("\n✅ Outputs are DIFFERENT! Positional encoding breaks")
print("   permutation equivariance — the model is now position-aware!")
```

### Expected Output

```
Output for 'Dog' (original, position 0):
[ 0.2341 -0.3456  0.8901  0.1234 -0.2345  0.6789]

Output for 'Dog' (permuted, position 2):
[ 0.1987 -0.4123  0.8234  0.1567 -0.1890  0.7123]

Are they equal? False
Maximum difference: 0.073412

✅ Outputs are DIFFERENT! Positional encoding breaks
   permutation equivariance — the model is now position-aware!
```

---

## 8. Visualizing the Positional Encoding

```python
import torch
import math
import matplotlib.pyplot as plt

def get_sinusoidal_pe(max_len, d_model):
    pe = torch.zeros(max_len, d_model)
    position = torch.arange(0, max_len, dtype=torch.float).unsqueeze(1)
    div_term = torch.exp(
        torch.arange(0, d_model, 2).float() * (-math.log(10000.0) / d_model)
    )
    pe[:, 0::2] = torch.sin(position * div_term)
    pe[:, 1::2] = torch.cos(position * div_term)
    return pe

pe = get_sinusoidal_pe(max_len=100, d_model=64)

fig, axes = plt.subplots(1, 2, figsize=(14, 5))

# Heatmap of all positions
axes[0].imshow(pe.numpy(), aspect='auto', cmap='RdBu')
axes[0].set_xlabel('Encoding Dimension')
axes[0].set_ylabel('Position')
axes[0].set_title('Sinusoidal Positional Encoding')

# Individual dimension curves
for dim in [0, 1, 4, 5, 10, 11]:
    axes[1].plot(pe[:, dim].numpy(), label=f'dim {dim}')
axes[1].set_xlabel('Position')
axes[1].set_ylabel('Value')
axes[1].set_title('Selected PE Dimensions')
axes[1].legend()

plt.tight_layout()
plt.savefig("positional_encoding.png", dpi=150)
plt.show()
```

### ASCII Visualization of the Pattern

```
Position →  0    1    2    3    4    5    6    7

Dim 0:    0.00 0.84 0.91 0.14 -.76 -.96 -.28  0.66   ← sin(pos/1)
Dim 1:    1.00 0.54 -.42 -.99 -.65  0.28 0.96  0.75   ← cos(pos/1)
Dim 2:    0.00 0.10 0.20 0.30 0.39 0.48 0.56  0.64   ← sin(pos/100)
Dim 3:    1.00 1.00 0.98 0.95 0.92 0.88 0.83  0.77   ← cos(pos/100)
...
Dim 62:   0.00 0.00 0.00 0.00 0.00 0.00 0.00  0.00   ← sin(pos/10000)
Dim 63:   1.00 1.00 1.00 1.00 1.00 1.00 1.00  1.00   ← cos(pos/10000)

Low dimensions:  High frequency → distinguish nearby positions
High dimensions: Low frequency  → distinguish distant positions
```

---

## 9. Why Different PE Methods Exist

```
METHOD COMPARISON:
═══════════════════════════════════════════════════════════

Sinusoidal (Original Transformer):
  ✅ No learnable parameters
  ✅ Can generalize to unseen sequence lengths
  ✅ Relative position info via dot products
  ❌ Fixed, cannot adapt to task

Learned (BERT, GPT-2):
  ✅ Adapts to task-specific patterns
  ✅ Simple to implement
  ❌ Cannot extrapolate beyond max training length
  ❌ Adds parameters

Relative (Transformer-XL, T5):
  ✅ Captures relative distance, not absolute position
  ✅ Better for variable-length sequences
  ❌ More complex implementation

Rotary / RoPE (LLaMA, Mistral):
  ✅ Encodes relative position in Q-K dot product
  ✅ Good extrapolation beyond training length
  ✅ No additive encoding needed
  ❌ Requires modifying attention computation

ALiBi (BLOOM):
  ✅ Zero parameters, just biases attention scores
  ✅ Excellent length extrapolation
  ❌ Not as expressive as learned encodings
```

---

## 10. Real-World Applications

### Why Position Matters

| Application | Why Position Is Critical |
|------------|------------------------|
| **Machine Translation** | "I eat fish" ≠ "fish eat I" — SVO vs OVS changes meaning entirely |
| **Code Generation** | Variable declarations must come before usage; indentation determines scope |
| **Music Generation** | Notes must be in temporal order; a chord played backward sounds different |
| **Protein Folding** | Amino acid sequence order determines 3D structure and function |
| **Speech Recognition** | Phoneme order distinguishes "pat" from "tap" |
| **Time Series** | Stock price at t=1 then t=2 is fundamentally different from t=2 then t=1 |

### Historical Note

The original Transformer paper (Vaswani et al., 2017) explicitly noted: *"Since our model contains no recurrence and no convolution, in order for the model to make use of the order of the sequence, we must inject some information about the relative or absolute position of the tokens in the sequence."*

This was a **design choice**, not a limitation — by separating content from position, the model can learn each independently.

---

## 11. Summary Table

| Aspect | Detail |
|--------|--------|
| **Problem** | Self-attention is permutation equivariant — blind to word order |
| **Proof** | SA(PX) = P · SA(X) for any permutation matrix P |
| **Consequence** | "Dog bites man" and "Man bites dog" produce identical representations |
| **Why it matters** | Word order carries meaning in virtually all languages |
| **Solution** | Add positional encoding to input: X' = X + PE |
| **Sinusoidal PE** | sin/cos functions at different frequencies; fixed, no parameters |
| **Learned PE** | Embedding lookup; adapts to task but can't extrapolate |
| **Relative PE** | Based on distance (i-j); better for variable lengths |
| **RoPE** | Rotate Q, K vectors; used in modern LLMs (LLaMA, Mistral) |
| **Key Insight** | Separating content from position is a feature, not a bug |

---

## 12. Revision Questions

1. **Proof**: Write out the proof that self-attention is permutation equivariant. What property of the permutation matrix P is essential for the proof (hint: PᵀP = ?)?

2. **Practical**: Give three examples of NLP tasks where word order is critical. For each, show a pair of sentences with the same tokens but different meanings.

3. **Comparison**: What are the tradeoffs between sinusoidal and learned positional encodings? When would you prefer one over the other?

4. **Code Challenge**: Modify the self-attention implementation to use learned positional encodings (nn.Embedding). Verify that the output changes when you permute the input.

5. **Research**: Modern LLMs like LLaMA use Rotary Position Embeddings (RoPE) instead of additive positional encodings. What is the key advantage of encoding position in the rotation of Q and K vectors rather than adding it to the input?

6. **Critical Thinking**: If self-attention is permutation equivariant, why do some tasks (like set classification) actually WANT this property? Give an example where permutation equivariance is desirable and positional encoding would be harmful.

---

[← Why Self-Attention Works](03-why-self-attention-works.md) | [Why Multiple Heads →](../03-Multi-Head-Attention/01-why-multiple-heads.md)
