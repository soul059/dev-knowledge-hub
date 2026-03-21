# RNN Task Variants

> **Unit 2, Chapter 6** — Many-to-one (classification), one-to-many (generation), many-to-many (translation) with PyTorch implementations.

---

## 📋 Overview

RNNs are versatile — by choosing where to read inputs and produce outputs, the same recurrent architecture handles classification, generation, labeling, and translation. This chapter provides **complete PyTorch implementations** for each variant.

---

## 1️⃣ Many-to-One: Sequence Classification

```
   x₁      x₂      x₃      x₄      x₅
    │       │       │       │       │
    ▼       ▼       ▼       ▼       ▼
  ┌───┐   ┌───┐   ┌───┐   ┌───┐   ┌───┐
  │ h₁│──▶│ h₂│──▶│ h₃│──▶│ h₄│──▶│ h₅│
  └───┘   └───┘   └───┘   └───┘   └─┬─┘
                                      │
                                      ▼
                                   ┌─────┐
                                   │  FC  │──▶ ŷ (class label)
                                   └─────┘

Use: Sentiment analysis, spam detection, sequence classification
Key: Only the FINAL hidden state h_T is used for prediction
```

### PyTorch Implementation

```python
import torch
import torch.nn as nn
import torch.nn.functional as F

class ManyToOneRNN(nn.Module):
    """Sequence Classification: input sequence → single label"""
    def __init__(self, vocab_size, embed_dim, hidden_dim, num_classes,
                 num_layers=1, dropout=0.3):
        super().__init__()
        self.embedding = nn.Embedding(vocab_size, embed_dim, padding_idx=0)
        self.rnn = nn.RNN(embed_dim, hidden_dim, num_layers=num_layers,
                          batch_first=True, dropout=dropout if num_layers > 1 else 0)
        self.dropout = nn.Dropout(dropout)
        self.fc = nn.Linear(hidden_dim, num_classes)
    
    def forward(self, x):
        """
        x: (batch_size, seq_len) — token indices
        Returns: (batch_size, num_classes) — class logits
        """
        embedded = self.embedding(x)          # (batch, seq_len, embed_dim)
        output, h_n = self.rnn(embedded)      # h_n: (num_layers, batch, hidden)
        
        # Use LAST layer's final hidden state
        final_hidden = h_n[-1]                # (batch, hidden_dim)
        final_hidden = self.dropout(final_hidden)
        logits = self.fc(final_hidden)        # (batch, num_classes)
        return logits

# Example: Sentiment Analysis
model = ManyToOneRNN(vocab_size=10000, embed_dim=128,
                     hidden_dim=256, num_classes=2)
x = torch.randint(0, 10000, (32, 50))  # batch=32, seq_len=50
logits = model(x)
print(f"Input: {x.shape} → Output: {logits.shape}")  # (32, 2)

# Training
criterion = nn.CrossEntropyLoss()
labels = torch.randint(0, 2, (32,))
loss = criterion(logits, labels)
print(f"Loss: {loss.item():.4f}")
```

---

## 2️⃣ One-to-Many: Sequence Generation

```
   x (seed)
    │
    ▼
  ┌───┐   ┌───┐   ┌───┐   ┌───┐   ┌───┐
  │ h₁│──▶│ h₂│──▶│ h₃│──▶│ h₄│──▶│ h₅│
  └─┬─┘   └─┬─┘   └─┬─┘   └─┬─┘   └─┬─┘
    │       │       │       │       │
    ▼       ▼       ▼       ▼       ▼
   y₁ ──▶  y₂ ──▶  y₃ ──▶  y₄ ──▶  y₅
   (fed back as input to next step)

Use: Text generation, music generation, image captioning
Key: Previous output becomes next input (autoregressive)
```

### PyTorch Implementation

```python
class OneToManyRNN(nn.Module):
    """Sequence Generation: seed input → generated sequence"""
    def __init__(self, vocab_size, embed_dim, hidden_dim):
        super().__init__()
        self.embedding = nn.Embedding(vocab_size, embed_dim)
        self.rnn = nn.RNN(embed_dim, hidden_dim, batch_first=True)
        self.fc = nn.Linear(hidden_dim, vocab_size)
        self.hidden_dim = hidden_dim
    
    def forward(self, x, seq_len):
        """
        x: (batch_size, 1) — seed token
        seq_len: number of tokens to generate
        Returns: (batch_size, seq_len, vocab_size) — logits at each step
        """
        batch_size = x.shape[0]
        h = torch.zeros(1, batch_size, self.hidden_dim)
        
        current_input = x
        outputs = []
        
        for t in range(seq_len):
            embedded = self.embedding(current_input)  # (batch, 1, embed)
            output, h = self.rnn(embedded, h)         # output: (batch, 1, hidden)
            logits = self.fc(output.squeeze(1))        # (batch, vocab_size)
            outputs.append(logits.unsqueeze(1))
            
            # Use predicted token as next input (autoregressive)
            current_input = logits.argmax(dim=-1, keepdim=True)  # (batch, 1)
        
        return torch.cat(outputs, dim=1)  # (batch, seq_len, vocab_size)
    
    @torch.no_grad()
    def generate(self, seed_token, max_len=50, temperature=1.0):
        """Generate text with temperature sampling"""
        self.eval()
        tokens = [seed_token]
        x = torch.tensor([[seed_token]])
        h = torch.zeros(1, 1, self.hidden_dim)
        
        for _ in range(max_len):
            embedded = self.embedding(x)
            output, h = self.rnn(embedded, h)
            logits = self.fc(output.squeeze(1)) / temperature
            probs = F.softmax(logits, dim=-1)
            next_token = torch.multinomial(probs, 1).item()
            tokens.append(next_token)
            x = torch.tensor([[next_token]])
        
        return tokens

# Example: Text Generation
model = OneToManyRNN(vocab_size=5000, embed_dim=64, hidden_dim=128)
seed = torch.randint(0, 5000, (4, 1))  # batch=4, seed token
output = model(seed, seq_len=20)
print(f"Seed: {seed.shape} → Generated: {output.shape}")  # (4, 20, 5000)
```

---

## 3️⃣ Many-to-Many (Equal Length): Sequence Labeling

```
   x₁      x₂      x₃      x₄      x₅
    │       │       │       │       │
    ▼       ▼       ▼       ▼       ▼
  ┌───┐   ┌───┐   ┌───┐   ┌───┐   ┌───┐
  │ h₁│──▶│ h₂│──▶│ h₃│──▶│ h₄│──▶│ h₅│
  └─┬─┘   └─┬─┘   └─┬─┘   └─┬─┘   └─┬─┘
    │       │       │       │       │
    ▼       ▼       ▼       ▼       ▼
   y₁      y₂      y₃      y₄      y₅

Use: POS tagging, NER, sequence labeling
Key: Output at EVERY time step, input and output same length
```

### PyTorch Implementation

```python
class ManyToManyEqualRNN(nn.Module):
    """Sequence Labeling: every input gets a label"""
    def __init__(self, vocab_size, embed_dim, hidden_dim, num_tags):
        super().__init__()
        self.embedding = nn.Embedding(vocab_size, embed_dim, padding_idx=0)
        self.rnn = nn.RNN(embed_dim, hidden_dim, batch_first=True)
        self.fc = nn.Linear(hidden_dim, num_tags)
    
    def forward(self, x):
        """
        x: (batch_size, seq_len) — token indices
        Returns: (batch_size, seq_len, num_tags) — tag logits per position
        """
        embedded = self.embedding(x)        # (batch, seq_len, embed_dim)
        output, _ = self.rnn(embedded)      # output: (batch, seq_len, hidden)
        logits = self.fc(output)            # (batch, seq_len, num_tags)
        return logits

# Example: Named Entity Recognition
model = ManyToManyEqualRNN(vocab_size=10000, embed_dim=100,
                            hidden_dim=128, num_tags=9)  # BIO tagging
x = torch.randint(0, 10000, (16, 30))  # batch=16, seq_len=30
logits = model(x)
print(f"Input: {x.shape} → Tags: {logits.shape}")  # (16, 30, 9)
predictions = logits.argmax(dim=-1)
print(f"Predicted tags: {predictions.shape}")  # (16, 30)
```

---

## 4️⃣ Many-to-Many (Unequal): Encoder-Decoder

```
  ENCODER                              DECODER
   x₁    x₂    x₃                    <s>    y₁    y₂    y₃
    │     │     │                      │     │     │     │
    ▼     ▼     ▼                      ▼     ▼     ▼     ▼
  ┌───┐ ┌───┐ ┌───┐   context       ┌───┐ ┌───┐ ┌───┐ ┌───┐
  │ h₁│▶│ h₂│▶│ h₃│──────────────▶ │ s₁│▶│ s₂│▶│ s₃│▶│ s₄│
  └───┘ └───┘ └───┘                 └─┬─┘ └─┬─┘ └─┬─┘ └─┬─┘
                                       │     │     │     │
                                       ▼     ▼     ▼     ▼
                                      y₁    y₂    y₃   <eos>

Use: Machine translation, summarization, question answering
Key: Encoder compresses input; decoder generates variable-length output
```

### PyTorch Implementation

```python
class Encoder(nn.Module):
    def __init__(self, vocab_size, embed_dim, hidden_dim):
        super().__init__()
        self.embedding = nn.Embedding(vocab_size, embed_dim)
        self.rnn = nn.RNN(embed_dim, hidden_dim, batch_first=True)
    
    def forward(self, x):
        embedded = self.embedding(x)
        output, hidden = self.rnn(embedded)
        return hidden  # Context vector

class Decoder(nn.Module):
    def __init__(self, vocab_size, embed_dim, hidden_dim):
        super().__init__()
        self.embedding = nn.Embedding(vocab_size, embed_dim)
        self.rnn = nn.RNN(embed_dim, hidden_dim, batch_first=True)
        self.fc = nn.Linear(hidden_dim, vocab_size)
    
    def forward(self, x, hidden):
        embedded = self.embedding(x)
        output, hidden = self.rnn(embedded, hidden)
        logits = self.fc(output)
        return logits, hidden

class Seq2Seq(nn.Module):
    """Encoder-Decoder for variable-length sequence transduction"""
    def __init__(self, src_vocab, tgt_vocab, embed_dim, hidden_dim):
        super().__init__()
        self.encoder = Encoder(src_vocab, embed_dim, hidden_dim)
        self.decoder = Decoder(tgt_vocab, embed_dim, hidden_dim)
        self.tgt_vocab = tgt_vocab
    
    def forward(self, src, tgt):
        """Teacher forcing during training"""
        context = self.encoder(src)
        output, _ = self.decoder(tgt[:, :-1], context)  # Shift right
        return output
    
    @torch.no_grad()
    def translate(self, src, max_len=50, sos_token=1, eos_token=2):
        """Autoregressive decoding at inference"""
        self.eval()
        context = self.encoder(src)
        
        current = torch.tensor([[sos_token]])
        hidden = context
        tokens = []
        
        for _ in range(max_len):
            output, hidden = self.decoder(current, hidden)
            next_token = output.argmax(dim=-1)[:, -1:]
            tokens.append(next_token.item())
            if next_token.item() == eos_token:
                break
            current = next_token
        
        return tokens

# Example: Machine Translation
model = Seq2Seq(src_vocab=8000, tgt_vocab=10000,
                embed_dim=128, hidden_dim=256)
src = torch.randint(0, 8000, (4, 12))    # English: batch=4, len=12
tgt = torch.randint(0, 10000, (4, 15))   # French: batch=4, len=15
output = model(src, tgt)
print(f"Source: {src.shape}, Target: {tgt.shape}")
print(f"Output: {output.shape}")  # (4, 14, 10000) — shifted
```

---

## 📊 Comparison Table

| Variant | Input → Output | Key Design | Example Tasks |
|---------|---------------|------------|---------------|
| Many-to-One | Seq → Single | Use final h_T | Sentiment, classification |
| One-to-Many | Single → Seq | Autoregressive generation | Captioning, generation |
| Many-to-Many (=) | Seq → Same-len Seq | Output at every step | POS tagging, NER |
| Many-to-Many (≠) | Seq → Diff-len Seq | Encoder-Decoder | Translation, QA |

---

## 📋 Summary Table

| Concept | Description |
|---------|-------------|
| Many-to-One | Only final hidden state produces output |
| One-to-Many | Previous output fed as next input (autoregressive) |
| Many-to-Many (Equal) | Output at every time step |
| Many-to-Many (Unequal) | Encoder compresses, decoder generates |
| Teacher Forcing | Use ground truth as decoder input during training |
| Autoregressive | Use model's own predictions during inference |

---

## ❓ Revision Questions

1. **For sentiment analysis with a 200-word review, which variant would you use? What is the shape of the final output?**

2. **In one-to-many generation, why is the previous output fed as the next input? What happens if you use random inputs instead?**

3. **Why can't you use a simple many-to-many (equal) model for translation? What problem arises?**

4. **Implement a many-to-one model that uses the AVERAGE of all hidden states instead of just the final one. When might this be better?**

5. **What is teacher forcing? Why is it used during training but not during inference?**

---

## 🧭 Navigation

| Direction | Link |
|-----------|------|
| ⬅️ Previous | [Parameter Sharing](05-parameter-sharing.md) |
| ➡️ Next | [BPTT Algorithm](../03-Backpropagation-Through-Time/01-bptt-algorithm.md) |
| ⬆️ Unit Overview | [README](../README.md) |
