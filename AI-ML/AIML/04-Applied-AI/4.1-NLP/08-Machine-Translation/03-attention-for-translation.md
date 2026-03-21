# Attention for Translation

## Overview

Attention mechanisms solved the **information bottleneck** of vanilla Seq2Seq models. Instead of compressing the entire source sentence into a single fixed-size context vector, attention allows the decoder to look at ALL encoder hidden states at each decoding step, focusing on the most relevant source words.

---

## The Bottleneck Problem

```
Without attention:
  Source: "The quick brown fox jumps over the lazy dog" (9 words)
           ↓     ↓     ↓     ↓     ↓     ↓    ↓    ↓    ↓
         [LSTM → LSTM → LSTM → LSTM → LSTM → LSTM → LSTM → LSTM → LSTM]
                                                              ↓
                                                         [context c]  ← single vector!
                                                              ↓
                                                           Decoder

  Everything squeezed into ONE vector → long sentences lose information

With attention:
  Decoder can look at ALL encoder states at every step
  → No bottleneck!
```

---

## Bahdanau Attention (Additive)

```
At each decoder step t:

1. Compute alignment scores:
   e_{t,j} = v^T · tanh(W_1 · s_{t-1} + W_2 · h_j)
   
   s_{t-1} = decoder hidden state (previous step)
   h_j     = encoder hidden state at position j

2. Normalize to get attention weights:
   α_{t,j} = softmax(e_{t,j}) = exp(e_{t,j}) / Σ_k exp(e_{t,k})

3. Compute context vector:
   c_t = Σ_j α_{t,j} · h_j    (weighted sum of encoder states)

4. Use c_t + s_{t-1} to generate next word
```

---

## Attention Visualization

```
Translating "I love cats" → "J'aime les chats"

                     Source words
                   I    love   cats
  Decoder step 1:  
    "J'"         [0.8   0.1    0.1]   ← focuses on "I"

  Decoder step 2:
    "aime"       [0.1   0.8    0.1]   ← focuses on "love"

  Decoder step 3:
    "les"        [0.1   0.1    0.8]   ← focuses on "cats"

  Decoder step 4:
    "chats"      [0.05  0.15   0.8]   ← focuses on "cats"

  Attention creates soft alignment between source and target!
```

---

## PyTorch Attention Layer

```python
import torch
import torch.nn as nn
import torch.nn.functional as F

class BahdanauAttention(nn.Module):
    def __init__(self, hidden_dim):
        super().__init__()
        self.W1 = nn.Linear(hidden_dim, hidden_dim)  # encoder
        self.W2 = nn.Linear(hidden_dim, hidden_dim)  # decoder
        self.v = nn.Linear(hidden_dim, 1)
    
    def forward(self, decoder_hidden, encoder_outputs):
        # decoder_hidden: (batch, hidden)
        # encoder_outputs: (batch, src_len, hidden)
        
        decoder_hidden = decoder_hidden.unsqueeze(1)  # (batch, 1, hidden)
        
        # Alignment scores
        scores = self.v(torch.tanh(
            self.W1(encoder_outputs) + self.W2(decoder_hidden)
        ))  # (batch, src_len, 1)
        
        # Attention weights
        weights = F.softmax(scores.squeeze(-1), dim=1)  # (batch, src_len)
        
        # Context vector
        context = torch.bmm(weights.unsqueeze(1), encoder_outputs)
        # (batch, 1, hidden)
        
        return context.squeeze(1), weights
```

---

## Bahdanau vs Luong Attention

| Feature | Bahdanau (Additive) | Luong (Multiplicative) |
|---------|:---:|:---:|
| Score function | v^T tanh(W₁s + W₂h) | s^T W h (or s^T h) |
| Computation | Slower (MLP) | Faster (dot product) |
| When applied | Before decoder RNN step | After decoder RNN step |
| Typical use | Original attention paper | Simpler, widely used |

---

## Impact of Attention

| Metric | Without Attention | With Attention |
|--------|:-:|:-:|
| BLEU (short sentences) | Good | Good |
| BLEU (long sentences) | Degrades | Maintains quality |
| Interpretability | Black box | Alignment visualization |
| Training stability | OK | Better |

---

## Revision Questions

1. **What is the information bottleneck problem and how does attention solve it?**
2. **Walk through the three steps of computing attention at one decoder step.**
3. **What is the difference between Bahdanau and Luong attention?**
4. **How can attention weights be used to visualize word alignment?**
5. **Why does attention especially help with long source sentences?**

---

[Previous: 02-neural-mt.md](02-neural-mt.md) | [Next: 04-transformer-translation.md](04-transformer-translation.md)
