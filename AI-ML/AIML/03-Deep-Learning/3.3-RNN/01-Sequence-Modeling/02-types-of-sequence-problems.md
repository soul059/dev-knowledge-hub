# Types of Sequence Problems

> **Unit 1, Chapter 2** — Classifying sequence-to-sequence mappings: one-to-one, one-to-many, many-to-one, and many-to-many architectures.

---

## 📋 Overview

Sequence problems come in different shapes depending on the relationship between input and output lengths. Understanding these categories is essential for choosing the right architecture. Andrej Karpathy's famous taxonomy identifies **five** core types.

---

## 🗂️ The Five Sequence Problem Types

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    SEQUENCE PROBLEM TAXONOMY                            │
├──────────┬──────────┬──────────┬───────────────┬───────────────────────┤
│ One-to-  │ One-to-  │ Many-to- │  Many-to-Many │  Many-to-Many        │
│ One      │ Many     │ One      │  (Equal)      │  (Unequal / Seq2Seq) │
│          │          │          │               │                       │
│  ┌─┐     │  ┌─┐     │ ┌─┬─┬─┐ │ ┌─┬─┬─┐      │ ┌─┬─┬─┐              │
│  │x│     │  │x│     │ │x│x│x│ │ │x│x│x│      │ │x│x│x│              │
│  └┬┘     │  └┬┘     │ └┬┴┬┴┬┘ │ └┬┴┬┴┬┘      │ └┬┴┬┴┬┘              │
│   │      │   │      │  └─┼─┘  │  │ │ │       │  └──┼──┘              │
│   ▼      │ ┌─┼──┐   │    ▼    │  ▼ ▼ ▼       │     ▼                 │
│  ┌─┐     │ ▼ ▼  ▼   │  ┌─┐   │ ┌─┬─┬─┐      │ ┌─┬─┬─┬─┐            │
│  │y│     │┌─┬─┬─┐   │  │y│   │ │y│y│y│      │ │y│y│y│y│            │
│  └─┘     ││y│y│y│   │  └─┘   │ └─┴─┴─┘      │ └─┴─┴─┴─┘            │
│          │└─┴─┴─┘   │        │               │                       │
│ Image    │ Image     │Sentiment│ POS Tagging   │ Translation           │
│ Classif. │ Captioning│Analysis │ NER           │ Summarization         │
└──────────┴──────────┴──────────┴───────────────┴───────────────────────┘
```

---

## 1️⃣ One-to-One

**Fixed input → Fixed output** (not truly a sequence problem)

```
Input:   ┌──────┐
         │  x   │ ─────▶ f(x) ─────▶ │  y   │
         └──────┘                     └──────┘

Example: Image Classification (single image → single label)
         Tabular data prediction
```

### Characteristics
- Standard feedforward network suffices
- No temporal dimension
- Included for completeness in the taxonomy

### Examples
| Input | Output | Task |
|-------|--------|------|
| Image | Class label | Image classification |
| Features | Prediction | Regression |

---

## 2️⃣ One-to-Many

**Single input → Sequence output**

```
         ┌───┐
Input:   │ x │
         └─┬─┘
           │
     ┌─────┼─────────────────────┐
     ▼     ▼     ▼     ▼     ▼  │
   ┌───┐ ┌───┐ ┌───┐ ┌───┐ ┌───┐
   │y₁ │ │y₂ │ │y₃ │ │y₄ │ │y₅ │
   └───┘ └───┘ └───┘ └───┘ └───┘

   "A"    "dog"  "on"  "the" "beach"

Example: Image → Caption
         Seed → Music sequence
         Class → Text description
```

### Architecture Pattern

```
┌─────┐    ┌─────┐    ┌─────┐    ┌─────┐    ┌─────┐
│ h₀  │───▶│ h₁  │───▶│ h₂  │───▶│ h₃  │───▶│ h₄  │
└──┬──┘    └──┬──┘    └──┬──┘    └──┬──┘    └──┬──┘
   │          │          │          │          │
   ▼          ▼          ▼          ▼          ▼
  y₁         y₂         y₃         y₄         y₅

h₀ is initialized from input x (e.g., CNN features of an image)
```

### Examples
| Input | Output | Task |
|-------|--------|------|
| Image | "A dog playing fetch" | Image captioning |
| Chord | Musical notes | Music generation |
| Genre label | Story | Text generation |
| Noise vector | Molecule sequence | Drug discovery |

---

## 3️⃣ Many-to-One

**Sequence input → Single output**

```
   ┌───┐ ┌───┐ ┌───┐ ┌───┐ ┌───┐
   │x₁ │ │x₂ │ │x₃ │ │x₄ │ │x₅ │
   └─┬─┘ └─┬─┘ └─┬─┘ └─┬─┘ └─┬─┘
     │      │      │      │      │
     ▼      ▼      ▼      ▼      ▼
   ┌───┐  ┌───┐  ┌───┐  ┌───┐  ┌───┐
   │h₁ │─▶│h₂ │─▶│h₃ │─▶│h₄ │─▶│h₅ │
   └───┘  └───┘  └───┘  └───┘  └─┬─┘
                                   │
                                   ▼
                                 ┌───┐
                                 │ y │  ← Single output
                                 └───┘

   "This" "movie" "was" "really" "great" → Positive ✓
```

### Architecture Pattern

Only the **final hidden state** (or a pooled version) produces output:

```
Process entire sequence → Use h_T → Predict y

y = softmax(W_y · h_T + b_y)
```

### Examples
| Input | Output | Task |
|-------|--------|------|
| Movie review text | Positive/Negative | Sentiment analysis |
| ECG signal | Normal/Abnormal | Medical diagnosis |
| Audio clip | Speaker ID | Speaker recognition |
| Stock price history | Up/Down | Stock movement prediction |
| DNA sequence | Gene function | Genomics |

---

## 4️⃣ Many-to-Many (Equal Length)

**Sequence input → Sequence output (same length)**

```
   ┌───┐ ┌───┐ ┌───┐ ┌───┐ ┌───┐
   │x₁ │ │x₂ │ │x₃ │ │x₄ │ │x₅ │
   └─┬─┘ └─┬─┘ └─┬─┘ └─┬─┘ └─┬─┘
     │      │      │      │      │
     ▼      ▼      ▼      ▼      ▼
   ┌───┐  ┌───┐  ┌───┐  ┌───┐  ┌───┐
   │h₁ │─▶│h₂ │─▶│h₃ │─▶│h₄ │─▶│h₅ │
   └─┬─┘  └─┬─┘  └─┬─┘  └─┬─┘  └─┬─┘
     │      │      │      │      │
     ▼      ▼      ▼      ▼      ▼
   ┌───┐ ┌───┐ ┌───┐ ┌───┐ ┌───┐
   │y₁ │ │y₂ │ │y₃ │ │y₄ │ │y₅ │
   └───┘ └───┘ └───┘ └───┘ └───┘

   Input:  "John"  "lives" "in" "New" "York"
   Output:  PER     O       O    LOC   LOC    (NER tags)
```

### Examples
| Input | Output | Task |
|-------|--------|------|
| Words | POS tags | Part-of-speech tagging |
| Words | NER labels | Named entity recognition |
| Video frames | Action labels | Video action labeling |
| Audio frames | Phonemes | Frame-level speech recognition |

---

## 5️⃣ Many-to-Many (Unequal Length / Seq2Seq)

**Sequence input → Sequence output (different lengths)**

```
  ENCODER                              DECODER
  ┌───┐ ┌───┐ ┌───┐ ┌───┐            ┌───┐ ┌───┐ ┌───┐ ┌───┐ ┌───┐
  │x₁ │ │x₂ │ │x₃ │ │x₄ │            │<s>│ │y₁ │ │y₂ │ │y₃ │ │y₄ │
  └─┬─┘ └─┬─┘ └─┬─┘ └─┬─┘            └─┬─┘ └─┬─┘ └─┬─┘ └─┬─┘ └─┬─┘
    │      │      │      │                │      │      │      │      │
    ▼      ▼      ▼      ▼                ▼      ▼      ▼      ▼      ▼
  ┌───┐  ┌───┐  ┌───┐  ┌───┐  context  ┌───┐  ┌───┐  ┌───┐  ┌───┐  ┌───┐
  │h₁ │─▶│h₂ │─▶│h₃ │─▶│h₄ │──────────▶│s₁ │─▶│s₂ │─▶│s₃ │─▶│s₄ │─▶│s₅ │
  └───┘  └───┘  └───┘  └───┘           └─┬─┘  └─┬─┘  └─┬─┘  └─┬─┘  └─┬─┘
                                           │      │      │      │      │
                                           ▼      ▼      ▼      ▼      ▼
                                         ┌───┐  ┌───┐  ┌───┐  ┌───┐  ┌───┐
                                         │y₁ │  │y₂ │  │y₃ │  │y₄ │  │y₅ │
                                         └───┘  └───┘  └───┘  └───┘  └───┘

  Input:  "I" "love" "cats" "<eos>"     "J'" "aime" "les" "chats" "<eos>"
  (English, 4 tokens)                   (French, 5 tokens)
```

### Examples
| Input | Output | Task |
|-------|--------|------|
| English sentence | French sentence | Machine translation |
| Document | Summary | Text summarization |
| Question | Answer | Question answering |
| Speech audio | Text | Speech recognition |

---

## 💻 Python Example: Identifying Problem Types

```python
import torch
import torch.nn as nn

# ─── Many-to-One: Sentiment Classification ───
class SentimentRNN(nn.Module):
    def __init__(self, vocab_size, embed_dim, hidden_dim, num_classes):
        super().__init__()
        self.embedding = nn.Embedding(vocab_size, embed_dim)
        self.rnn = nn.RNN(embed_dim, hidden_dim, batch_first=True)
        self.fc = nn.Linear(hidden_dim, num_classes)
    
    def forward(self, x):
        embedded = self.embedding(x)        # (batch, seq_len, embed_dim)
        output, h_n = self.rnn(embedded)    # h_n: (1, batch, hidden_dim)
        return self.fc(h_n.squeeze(0))      # Use LAST hidden state only

# ─── Many-to-Many (Equal): Sequence Labeling ───
class SequenceLabeler(nn.Module):
    def __init__(self, vocab_size, embed_dim, hidden_dim, num_tags):
        super().__init__()
        self.embedding = nn.Embedding(vocab_size, embed_dim)
        self.rnn = nn.RNN(embed_dim, hidden_dim, batch_first=True)
        self.fc = nn.Linear(hidden_dim, num_tags)
    
    def forward(self, x):
        embedded = self.embedding(x)        # (batch, seq_len, embed_dim)
        output, _ = self.rnn(embedded)      # output: (batch, seq_len, hidden)
        return self.fc(output)              # Output at EVERY time step

# ─── One-to-Many: Text Generation from seed ───
class TextGenerator(nn.Module):
    def __init__(self, vocab_size, embed_dim, hidden_dim):
        super().__init__()
        self.embedding = nn.Embedding(vocab_size, embed_dim)
        self.rnn = nn.RNN(embed_dim, hidden_dim, batch_first=True)
        self.fc = nn.Linear(hidden_dim, vocab_size)
    
    def generate(self, seed_token, max_len=20):
        """Generate sequence from single seed token"""
        tokens = [seed_token]
        x = torch.tensor([[seed_token]])
        h = None
        for _ in range(max_len):
            embedded = self.embedding(x)
            output, h = self.rnn(embedded, h)
            logits = self.fc(output[:, -1, :])
            next_token = logits.argmax(dim=-1).item()
            tokens.append(next_token)
            x = torch.tensor([[next_token]])
        return tokens

print("Many-to-One: sequence → single prediction")
print("Many-to-Many: sequence → sequence (same length)")
print("One-to-Many: single input → generated sequence")
```

---

## 📊 Comparison Table

| Type | Input | Output | Architecture | Example Task |
|------|-------|--------|-------------|--------------|
| One-to-One | Fixed | Fixed | Feedforward | Image Classification |
| One-to-Many | Fixed | Sequence | RNN Decoder | Image Captioning |
| Many-to-One | Sequence | Fixed | RNN → FC | Sentiment Analysis |
| Many-to-Many (=) | Sequence | Same-length Seq | RNN → FC per step | POS Tagging |
| Many-to-Many (≠) | Sequence | Diff-length Seq | Encoder-Decoder | Translation |

---

## 📋 Summary Table

| Concept | Key Point |
|---------|-----------|
| One-to-One | Not truly sequential — standard NN works |
| One-to-Many | RNN generates output autoregressively from single input |
| Many-to-One | RNN reads full sequence, output from final state |
| Many-to-Many (Equal) | Output at every time step, same length as input |
| Many-to-Many (Unequal) | Encoder-decoder architecture needed |
| Output at each step | Use all hidden states (many-to-many) |
| Output at end | Use final hidden state (many-to-one) |

---

## ❓ Revision Questions

1. **Draw the architecture (or describe the data flow) for a many-to-one sequence problem. What hidden state produces the output?**

2. **Why does many-to-many with unequal lengths require an encoder-decoder architecture instead of a single RNN?**

3. **Classify these tasks: (a) email spam detection, (b) English-to-Spanish translation, (c) music generation from a genre label, (d) video frame labeling, (e) DNA sequence → protein function prediction.**

4. **In a many-to-many (equal length) model, outputs are produced at every time step. What is the limitation of this compared to using an encoder-decoder?**

5. **Explain how image captioning combines a CNN and an RNN. Which sequence problem type is it?**

---

## 🧭 Navigation

| Direction | Link |
|-----------|------|
| ⬅️ Previous | [Why Sequence Models](01-why-sequence-models.md) |
| ➡️ Next | [Limitations of Feedforward](03-limitations-of-feedforward.md) |
| ⬆️ Unit Overview | [README](../README.md) |
