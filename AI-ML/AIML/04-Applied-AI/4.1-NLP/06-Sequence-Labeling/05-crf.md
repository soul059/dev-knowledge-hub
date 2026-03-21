# Conditional Random Fields (CRF)

## Overview

Conditional Random Fields (CRFs) are probabilistic models for sequence labeling that consider **label dependencies** — not just what the best label for each token is independently, but what the best **sequence** of labels is. Unlike HMMs, CRFs are discriminative and can use rich overlapping features. CRFs are often used as the final layer in neural NER systems (BiLSTM-CRF).

---

## Why CRF Over Independent Classification?

```
Independent classifier (per-token):
  Token:  "New"   "York"  "is"   "great"
  Pred:   B-LOC   B-PER   O      O        ← Invalid! I-LOC should follow B-LOC

CRF (considers label transitions):
  Token:  "New"   "York"  "is"   "great"  
  Pred:   B-LOC   I-LOC   O      O        ← Valid sequence! ✓

CRF learns that B-LOC → I-LOC is valid but B-LOC → B-PER is unlikely.
```

---

## CRF Formulation

```
P(y|x) = (1/Z(x)) × exp(Σ_t [Σ_k λ_k f_k(y_t, y_{t-1}, x, t)])

Where:
  y = label sequence (B-PER, I-PER, O, ...)
  x = input sequence (tokens)
  f_k = feature function (emission + transition features)
  λ_k = learned weights
  Z(x) = partition function (normalizer)

Key components:
  Emission score:    s(x_t → y_t)     (how likely is label y for token x?)
  Transition score:  t(y_{t-1} → y_t) (how likely is label y after label y'?)
```

---

## Transition Matrix

```
The CRF learns a transition score matrix:

            B-PER  I-PER  B-LOC  I-LOC  O
  B-PER  [  -2.0    3.5   -1.0   -5.0   1.0 ]
  I-PER  [  -1.0    2.0   -2.0   -5.0   2.0 ]
  B-LOC  [  -3.0   -5.0   -2.0    3.5   1.0 ]
  I-LOC  [  -3.0   -5.0   -1.0    2.0   2.0 ]
  O      [   1.0   -5.0    1.0   -5.0   2.0 ]

High values (3.5): B-PER → I-PER, B-LOC → I-LOC (valid continuations)
Low values (-5.0): O → I-PER, B-PER → I-LOC (invalid transitions)
```

---

## BiLSTM-CRF Architecture

```
┌────────────────────────────────────────────┐
│           BiLSTM-CRF for NER              │
│                                            │
│  Token:    The    big    dog    ran         │
│             │      │      │      │         │
│  Embedding: ▼      ▼      ▼      ▼        │
│           ┌──────────────────────────┐     │
│           │   Bidirectional LSTM      │     │
│           └──────────────────────────┘     │
│             │      │      │      │         │
│  Emission:  ▼      ▼      ▼      ▼        │
│           [O:3.1] [O:2.8] [B:3.5] [O:2.9] │
│           [B:0.2] [B:0.1] [I:0.3] [B:0.5] │
│             │      │      │      │         │
│           ┌──────────────────────────┐     │
│           │     CRF Layer             │     │
│           │  (considers transitions)  │     │
│           └──────────────────────────┘     │
│             │      │      │      │         │
│  Output:   O      O    B-ANI    O          │
└────────────────────────────────────────────┘
```

```python
# PyTorch CRF (pip install pytorch-crf)
from torchcrf import CRF
import torch
import torch.nn as nn

class BiLSTM_CRF(nn.Module):
    def __init__(self, vocab_size, embed_dim, hidden_dim, num_tags):
        super().__init__()
        self.embedding = nn.Embedding(vocab_size, embed_dim)
        self.lstm = nn.LSTM(embed_dim, hidden_dim // 2,
                           bidirectional=True, batch_first=True)
        self.fc = nn.Linear(hidden_dim, num_tags)
        self.crf = CRF(num_tags, batch_first=True)
    
    def forward(self, x, tags=None, mask=None):
        embedded = self.embedding(x)
        lstm_out, _ = self.lstm(embedded)
        emissions = self.fc(lstm_out)
        
        if tags is not None:
            # Training: return negative log-likelihood
            loss = -self.crf(emissions, tags, mask=mask)
            return loss
        else:
            # Inference: Viterbi decoding for best sequence
            return self.crf.decode(emissions, mask=mask)
```

---

## CRF vs Softmax (Independent) Layer

| Feature | Softmax (per-token) | CRF |
|---------|:---:|:---:|
| Label dependencies | ❌ Independent | ✅ Considers transitions |
| Invalid sequences | Possible | Prevented by learned transitions |
| Decoding | argmax per token | Viterbi (global best sequence) |
| Training | Cross-entropy per token | Sequence-level likelihood |
| Speed | Faster | Slightly slower |
| NER F1 | ~88-90% | ~91-93% (typical improvement) |

---

## Revision Questions

1. **Why does a CRF layer improve NER over independent per-token classification?**
2. **What does the transition matrix in a CRF represent?**
3. **How does Viterbi decoding find the best label sequence?**
4. **What is the BiLSTM-CRF architecture and why is it effective for NER?**
5. **What types of invalid label sequences can a CRF prevent?**

---

[Previous: 04-bio-tagging.md](04-bio-tagging.md) | [Next: 06-neural-sequence-labeling.md](06-neural-sequence-labeling.md)
