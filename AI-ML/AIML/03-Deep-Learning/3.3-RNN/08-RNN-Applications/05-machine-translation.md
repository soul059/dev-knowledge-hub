# Machine Translation

> **Unit 8, Chapter 5** — Seq2seq for translation, attention mechanism integration, and BLEU score evaluation.

---

## 📋 Overview

**Machine translation** (MT) was the driving application for seq2seq models and attention. This chapter covers the complete NMT pipeline from encoder-decoder architecture through attention to evaluation with BLEU score.

---

## 🏗️ NMT Architecture with Attention

```
Source: "I love cats"    Target: "J'aime les chats"

   "I"    "love"  "cats"
    │       │       │
    ▼       ▼       ▼
  ┌────┐  ┌────┐  ┌────┐
  │BiLSTM│→│BiLSTM│→│BiLSTM│   ENCODER (bidirectional)
  └──┬──┘  └──┬──┘  └──┬──┘
     h₁      h₂      h₃
     │       │       │
     └───────┼───────┘
             │
        ATTENTION at each decoder step
             │
   ┌─────────┼─────────┐
   │         ▼         │
   │   ┌──────────┐    │
   │   │ Attention │    │
   │   │ Weights   │    │
   │   └────┬─────┘    │
   │        ▼          │
   │   Context c_t     │
   │        │          │
   ▼        ▼          ▼
  ┌────┐  ┌────┐  ┌────┐
  │LSTM │→│LSTM │→│LSTM │   DECODER
  └──┬──┘  └──┬──┘  └──┬──┘
     │       │       │
     ▼       ▼       ▼
   "J'"    "aime"  "les"   "chats"
```

---

## 📊 BLEU Score

```
BLEU (Bilingual Evaluation Understudy):
  Measures n-gram overlap between prediction and reference

  BLEU = BP × exp(Σ (1/N) × log(precision_n))

  precision_n = matching n-grams / total predicted n-grams

  BP (brevity penalty):
    BP = 1                    if pred_len ≥ ref_len
    BP = exp(1 - ref/pred)    if pred_len < ref_len

Example:
  Reference:  "The cat is on the mat"
  Prediction: "The cat sat on the mat"

  1-gram precision: 5/6 (all except "sat")
  2-gram precision: 3/5 ("the cat", "on the", "the mat")
  BLEU-4 uses up to 4-grams

  BLEU range: 0 (no match) to 1 (perfect match)
  Good MT: BLEU > 0.30, Human-level: BLEU ≈ 0.40-0.50
```

---

## 💻 Implementation

```python
import torch
import torch.nn as nn
import torch.nn.functional as F

class AttentionNMT(nn.Module):
    def __init__(self, src_vocab, trg_vocab, embed_dim, hidden_dim):
        super().__init__()
        # Encoder
        self.src_embed = nn.Embedding(src_vocab, embed_dim)
        self.encoder = nn.LSTM(embed_dim, hidden_dim, bidirectional=True, batch_first=True)
        
        # Attention
        self.attn = nn.Linear(hidden_dim * 3, hidden_dim)
        self.v = nn.Linear(hidden_dim, 1, bias=False)
        
        # Decoder
        self.trg_embed = nn.Embedding(trg_vocab, embed_dim)
        self.decoder = nn.LSTM(embed_dim + hidden_dim * 2, hidden_dim, batch_first=True)
        self.fc_out = nn.Linear(hidden_dim * 3 + embed_dim, trg_vocab)
        
        self.enc_proj = nn.Linear(hidden_dim * 2, hidden_dim)
    
    def attention(self, decoder_hidden, encoder_outputs):
        src_len = encoder_outputs.shape[1]
        decoder_hidden = decoder_hidden.unsqueeze(1).repeat(1, src_len, 1)
        energy = torch.tanh(self.attn(torch.cat([decoder_hidden, encoder_outputs], dim=2)))
        attention = self.v(energy).squeeze(2)
        return F.softmax(attention, dim=1)
    
    def forward(self, src, trg):
        # Encode
        embedded_src = self.src_embed(src)
        enc_output, (hidden, cell) = self.encoder(embedded_src)
        
        # Compress bidirectional hidden
        hidden = self.enc_proj(torch.cat([hidden[-2], hidden[-1]], dim=1)).unsqueeze(0)
        cell = self.enc_proj(torch.cat([cell[-2], cell[-1]], dim=1)).unsqueeze(0)
        
        # Decode with attention
        outputs = []
        for t in range(trg.shape[1] - 1):
            embedded_trg = self.trg_embed(trg[:, t]).unsqueeze(1)
            attn_weights = self.attention(hidden.squeeze(0), enc_output)
            context = torch.bmm(attn_weights.unsqueeze(1), enc_output)
            
            dec_input = torch.cat([embedded_trg, context], dim=2)
            dec_output, (hidden, cell) = self.decoder(dec_input, (hidden, cell))
            
            prediction = self.fc_out(torch.cat([dec_output.squeeze(1), context.squeeze(1),
                                                 embedded_trg.squeeze(1)], dim=1))
            outputs.append(prediction)
        
        return torch.stack(outputs, dim=1)

model = AttentionNMT(src_vocab=8000, trg_vocab=10000, embed_dim=256, hidden_dim=512)
src = torch.randint(0, 8000, (4, 15))
trg = torch.randint(0, 10000, (4, 20))
out = model(src, trg)
print(f"Translation output: {out.shape}")  # (4, 19, 10000)
```

---

## 📋 Summary Table

| Concept | Description |
|---------|-------------|
| NMT | Neural Machine Translation with encoder-decoder |
| Encoder | BiLSTM reads source sentence |
| Decoder | LSTM generates target with attention |
| Attention | Dynamic focus on relevant source positions |
| BLEU | N-gram precision metric with brevity penalty |
| Training | Cross-entropy with teacher forcing |

---

## ❓ Revision Questions

1. **Why is bidirectional encoding important for machine translation?**
2. **Calculate BLEU-1 for reference "the cat sat" and prediction "a cat sat".**
3. **How does attention help with word reordering across languages?**
4. **What is the brevity penalty in BLEU and why is it needed?**
5. **Why has the Transformer largely replaced RNN-based NMT systems?**

---

## 🧭 Navigation

| Direction | Link |
|-----------|------|
| ⬅️ Previous | [Named Entity Recognition](04-named-entity-recognition.md) |
| ➡️ Next | [Speech Recognition Basics](06-speech-recognition-basics.md) |
| ⬆️ Unit Overview | [README](../README.md) |
