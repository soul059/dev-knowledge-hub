# Seq2Seq Applications

> **Unit 7, Chapter 2** — Machine translation, text summarization, chatbots, code generation, and image captioning.

---

## 📋 Overview

The sequence-to-sequence framework is one of the most versatile architectures in deep learning, applicable to any task that transforms one sequence into another. This chapter surveys the major applications.

---

## 🌍 Application Map

```
┌─────────────────────────────────────────────────────────────────┐
│                  SEQ2SEQ APPLICATIONS                          │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  Text → Text:                                                  │
│    • Machine Translation   (English → French)                  │
│    • Text Summarization    (Article → Summary)                 │
│    • Paraphrasing          (Sentence → Rephrased)              │
│    • Grammar Correction    (Incorrect → Correct)               │
│    • Question Answering    (Context+Q → Answer)                │
│    • Chatbots/Dialog       (User input → Response)             │
│                                                                 │
│  Text → Code:                                                  │
│    • Code Generation       (Description → Code)                │
│    • SQL Generation        (Natural lang → SQL query)          │
│                                                                 │
│  Audio → Text:                                                 │
│    • Speech Recognition    (Audio → Transcript)                │
│                                                                 │
│  Image → Text:                                                 │
│    • Image Captioning      (Image → Description)               │
│    • OCR                   (Image → Text)                      │
│                                                                 │
│  Text → Audio:                                                 │
│    • Text-to-Speech        (Text → Audio)                      │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 1️⃣ Machine Translation

```
Input:  "The cat sat on the mat"
Output: "Le chat s'est assis sur le tapis"

Key challenges:
  • Different word orders (SOV vs SVO)
  • One-to-many word mappings
  • Gender/number agreement
  • Idioms and cultural expressions

Architecture: Encoder-Decoder + Attention
Metric: BLEU score
```

## 2️⃣ Text Summarization

```
Input:  "Scientists at MIT have developed a new algorithm that
         can predict protein structures with unprecedented accuracy.
         The breakthrough could accelerate drug discovery..."

Output: "MIT researchers created an accurate protein structure
         prediction algorithm that may speed up drug development."

Types:
  • Extractive: Select important sentences
  • Abstractive: Generate new summary text (seq2seq)
Metric: ROUGE score
```

## 3️⃣ Chatbots / Dialog Systems

```
Input:  "What's the weather like today?"
Output: "It's sunny with a high of 72°F."

Architecture: Seq2seq + context window of previous turns
Challenge: Maintaining conversation coherence over multiple turns
```

## 4️⃣ Code Generation

```
Input:  "Write a function that reverses a string"
Output: "def reverse_string(s): return s[::-1]"

Decoder vocabulary includes code tokens + indentation
Challenge: Syntactic validity of generated code
```

## 5️⃣ Image Captioning

```
Input:  [Image of a dog catching a frisbee]
Output: "A brown dog jumping to catch a frisbee in a park"

Architecture:
  CNN Encoder → Feature vector → LSTM Decoder → Caption

  ┌─────┐    ┌─────┐    ┌──────────────────────┐
  │Image│──▶│ CNN  │──▶│  LSTM Decoder         │──▶ Caption
  │     │    │(ResNet)│  │  (one word at a time) │
  └─────┘    └─────┘    └──────────────────────┘
```

---

## 💻 Minimal Example: Seq2Seq for Reversal

```python
import torch
import torch.nn as nn
import torch.optim as optim

# Simple task: reverse a sequence of digits
# Input:  [3, 1, 4, 1, 5]
# Output: [5, 1, 4, 1, 3]

class SimpleSeq2Seq(nn.Module):
    def __init__(self, vocab_size=12, embed_dim=32, hidden_dim=64):
        super().__init__()
        self.encoder_embed = nn.Embedding(vocab_size, embed_dim)
        self.decoder_embed = nn.Embedding(vocab_size, embed_dim)
        self.encoder = nn.LSTM(embed_dim, hidden_dim, batch_first=True)
        self.decoder = nn.LSTM(embed_dim, hidden_dim, batch_first=True)
        self.fc = nn.Linear(hidden_dim, vocab_size)
        self.hidden_dim = hidden_dim
        
    def forward(self, src, trg):
        enc_emb = self.encoder_embed(src)
        _, (h, c) = self.encoder(enc_emb)
        
        dec_emb = self.decoder_embed(trg[:, :-1])
        output, _ = self.decoder(dec_emb, (h, c))
        return self.fc(output)

# Training
SOS, EOS = 10, 11
model = SimpleSeq2Seq()
optimizer = optim.Adam(model.parameters(), lr=0.003)
criterion = nn.CrossEntropyLoss()

for epoch in range(500):
    # Generate batch: random digit sequences
    seq = torch.randint(0, 10, (64, 5))
    rev = seq.flip(1)
    
    # Add SOS/EOS
    src = torch.cat([torch.full((64,1), SOS), seq, torch.full((64,1), EOS)], dim=1)
    trg = torch.cat([torch.full((64,1), SOS), rev, torch.full((64,1), EOS)], dim=1)
    
    output = model(src, trg)
    loss = criterion(output.view(-1, 12), trg[:, 1:].reshape(-1))
    
    optimizer.zero_grad()
    loss.backward()
    optimizer.step()
    
    if epoch % 100 == 0:
        preds = output.argmax(dim=-1)[0]
        print(f"Epoch {epoch}: loss={loss.item():.4f}")
        print(f"  Input:  {seq[0].tolist()}")
        print(f"  Target: {rev[0].tolist()}")
        print(f"  Pred:   {preds.tolist()}")
```

---

## 📊 Application Comparison

| Application | Input | Output | Key Metric | Typical Architecture |
|------------|-------|--------|------------|---------------------|
| Translation | Text (lang A) | Text (lang B) | BLEU | Seq2seq + Attention |
| Summarization | Long text | Short text | ROUGE | Seq2seq + Copy |
| Chatbot | User message | Response | Human eval | Seq2seq + Context |
| Code Gen | NL description | Code | Functional correctness | Seq2seq + Beam |
| Captioning | Image | Text | CIDEr | CNN + LSTM |
| Speech | Audio | Text | WER | Seq2seq + CTC |

---

## 📋 Summary Table

| Concept | Description |
|---------|-------------|
| Versatility | Seq2seq handles any sequence transduction task |
| Key requirement | Paired training data (input-output sequences) |
| Translation | Most studied seq2seq application |
| Summarization | Can be extractive or abstractive |
| Image captioning | Uses CNN encoder + RNN decoder |
| Modern trend | Transformers now outperform RNN seq2seq on most tasks |

---

## ❓ Revision Questions

1. **Name five different applications of the seq2seq framework and describe the input/output for each.**

2. **How is image captioning different from standard seq2seq? What replaces the text encoder?**

3. **What metrics are used for (a) translation, (b) summarization, (c) speech recognition?**

4. **Why is abstractive summarization harder than extractive summarization?**

5. **Design a seq2seq system for grammar correction. What would the training data look like?**

---

## 🧭 Navigation

| Direction | Link |
|-----------|------|
| ⬅️ Previous | [Encoder-Decoder Framework](01-encoder-decoder-framework.md) |
| ➡️ Next | [Beam Search Decoding](03-beam-search-decoding.md) |
| ⬆️ Unit Overview | [README](../README.md) |
