# Transformer for Translation

## Overview

The Transformer architecture (Vaswani et al., 2017) revolutionized machine translation by replacing recurrence entirely with self-attention. This enables massive parallelization during training and captures long-range dependencies in O(1) path length. Transformers are now the default architecture for all production MT systems.

---

## Why Transformers Dominate Translation

```
RNN-based NMT:                    Transformer NMT:
  Sequential processing             Parallel processing
  O(n) path length                  O(1) path length
  Slow training                     Fast training
  Gradient issues for long seq      Stable gradients

  BLEU: ~28 (WMT'14 En-De)         BLEU: ~28.4+ (same task)
  Training: days on 8 GPUs          Training: 3.5 days on 8 GPUs
```

---

## Architecture for Translation

```
Source: "I love cats"              Target: "J'aime les chats"

  ┌─────────────────┐            ┌──────────────────────┐
  │    ENCODER       │            │      DECODER          │
  │                 │            │                      │
  │  I  love  cats  │            │ <SOS> J' aime les   │
  │  +pos +pos +pos │            │ +pos  +pos +pos +pos│
  │  ↓    ↓    ↓    │            │  ↓    ↓    ↓    ↓   │
  │  ┌────────────┐ │            │ ┌──────────────────┐│
  │  │Self-Attn   │ │            │ │Masked Self-Attn  ││
  │  │+ Add&Norm  │ │            │ │+ Add&Norm        ││
  │  ├────────────┤ │            │ ├──────────────────┤│
  │  │FFN         │ │            │ │Cross-Attention   ││
  │  │+ Add&Norm  │ │   K,V ───▶│ │(Q=dec, K,V=enc)  ││
  │  └────────────┘ │            │ │+ Add&Norm        ││
  │     × 6 layers  │            │ ├──────────────────┤│
  └─────────────────┘            │ │FFN + Add&Norm    ││
                                  │ └──────────────────┘│
                                  │    × 6 layers       │
                                  │         ↓           │
                                  │   Linear + Softmax  │
                                  │         ↓           │
                                  │  J'  aime  les chats│
                                  └──────────────────────┘
```

---

## Training a Transformer for Translation

```python
from transformers import MarianMTModel, MarianTokenizer

# Pre-trained translation model (Helsinki-NLP)
model_name = "Helsinki-NLP/opus-mt-en-fr"
tokenizer = MarianTokenizer.from_pretrained(model_name)
model = MarianMTModel.from_pretrained(model_name)

# Translate
texts = ["I love machine learning.", "The weather is beautiful today."]
inputs = tokenizer(texts, return_tensors="pt", padding=True, truncation=True)
translated = model.generate(**inputs)
results = tokenizer.batch_decode(translated, skip_special_tokens=True)

for src, tgt in zip(texts, results):
    print(f"  {src} → {tgt}")
# I love machine learning. → J'aime l'apprentissage automatique.
# The weather is beautiful today. → Le temps est beau aujourd'hui.
```

---

## Key Transformer Translation Features

| Feature | How It Helps Translation |
|---------|------------------------|
| Self-attention (encoder) | Each source word attends to all other source words |
| Masked self-attention (decoder) | Prevents seeing future target words during training |
| Cross-attention | Decoder queries attend to encoder outputs (like Bahdanau) |
| Multi-head attention | Captures syntax, semantics, alignment simultaneously |
| Positional encoding | Provides word order without recurrence |
| Layer normalization | Stabilizes deep network training |

---

## Model Comparison

| Model | Type | BLEU (WMT En-De) | Speed |
|-------|------|:-:|:-:|
| Phrase-based SMT | Statistical | ~20 | Medium |
| LSTM Seq2Seq | Neural (RNN) | ~25 | Slow |
| LSTM + Attention | Neural (RNN) | ~28 | Slow |
| Transformer | Neural (attention) | ~28.4 | Fast |
| Transformer (big) | Neural (attention) | ~29.3 | Medium |
| mBART / M2M-100 | Multilingual transformer | ~30+ | Medium |

---

## Revision Questions

1. **Why are transformers faster to train than LSTM-based NMT?**
2. **What is the role of cross-attention in the translation decoder?**
3. **Why is masked self-attention needed in the decoder?**
4. **How do you use a pre-trained MarianMT model for translation?**
5. **What are the advantages of transformer-based MT over RNN-based MT?**

---

[Previous: 03-attention-for-translation.md](03-attention-for-translation.md) | [Next: 05-bleu-score.md](05-bleu-score.md)
