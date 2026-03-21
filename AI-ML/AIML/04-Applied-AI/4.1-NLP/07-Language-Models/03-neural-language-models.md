# Chapter 3: Neural Language Models

> **Unit 4.1 — NLP / 07 – Language Models** | [← Previous: Perplexity](./02-perplexity.md) | [Next → Pretraining & Fine-tuning](./04-pretraining-finetuning.md)

---

## 1. Overview

**Neural language models** replace the discrete count-based probability tables of n-gram models with continuous vector representations and learned functions. The seminal work by Bengio et al. (2003) introduced the **feedforward neural language model**, which maps each word to a dense embedding vector and uses a neural network to predict the next word. This breakthrough solved two fundamental problems of n-gram models: the **curse of dimensionality** (exponential growth of parameters with context size) and the inability to **generalize** across semantically similar words. From feedforward models, the field progressed to **recurrent neural networks (RNNs)** and **LSTMs**, which can condition on arbitrarily long histories — not just a fixed window. These architectures laid the foundation for the modern Transformer-based models that dominate NLP today.

---

## 2. Bengio et al. (2003) — Feedforward Neural Language Model

The first major neural approach to language modeling was proposed in *"A Neural Probabilistic Language Model"* (Bengio, Ducharme, Vincent & Jauvin, 2003). The core ideas are:

1. **Learned word embeddings:** Each word in the vocabulary is mapped to a dense, real-valued vector (the "distributed representation").
2. **Fixed context window:** Like n-grams, the model conditions on the previous (n−1) words — but represents them as continuous vectors instead of discrete symbols.
3. **Neural prediction:** A feedforward network takes the concatenated context embeddings and outputs a probability distribution over the entire vocabulary via softmax.

### 2.1 Architecture Diagram

```
 INPUT (context window of n-1 words)
 ┌──────┐  ┌──────┐  ┌──────┐
 │ wₜ₋₃ │  │ wₜ₋₂ │  │ wₜ₋₁ │         ← word indices
 └──┬───┘  └──┬───┘  └──┬───┘
    │         │         │
    ▼         ▼         ▼
 ┌──────┐  ┌──────┐  ┌──────┐
 │  e₁  │  │  e₂  │  │  e₃  │         ← Embedding lookup (shared matrix C)
 │(d-dim)│  │(d-dim)│  │(d-dim)│         each eᵢ ∈ ℝ^d
 └──┬───┘  └──┬───┘  └──┬───┘
    │         │         │
    └────┬────┴────┬────┘
         │         │
         ▼         ▼
  ┌─────────────────────────┐
  │   Concatenation Layer   │          ← x = [e₁ ; e₂ ; e₃] ∈ ℝ^(3d)
  └───────────┬─────────────┘
              │
              ▼
  ┌─────────────────────────┐
  │     Hidden Layer        │          ← h = tanh(Wx + b)
  │     (h units, tanh)     │            W ∈ ℝ^(h × 3d), b ∈ ℝ^h
  └───────────┬─────────────┘
              │
              ▼
  ┌─────────────────────────┐
  │     Output Layer        │          ← y = Uh + d
  │     (|V| units)         │            U ∈ ℝ^(|V| × h)
  └───────────┬─────────────┘
              │
              ▼
  ┌─────────────────────────┐
  │       Softmax           │          ← P(wₜ = k | context) = exp(yₖ) / Σ exp(yⱼ)
  │  P(wₜ | wₜ₋₃,wₜ₋₂,wₜ₋₁) │
  └─────────────────────────┘
```

### 2.2 Key Equations

```
1. Embedding lookup:     eᵢ = C[wₜ₋ᵢ]              (row lookup in embedding matrix C ∈ ℝ^(|V| × d))

2. Concatenation:        x = [eₙ₋₁ ; eₙ₋₂ ; ... ; e₁]   ∈ ℝ^((n-1)·d)

3. Hidden layer:         h = tanh(W · x + b)         W ∈ ℝ^(H × (n-1)·d)

4. Output logits:        y = U · h + d               U ∈ ℝ^(|V| × H)

5. Softmax:              P(wₜ = k | context) = exp(yₖ) / Σⱼ exp(yⱼ)
```

### 2.3 Why This Was Revolutionary

| N-gram Models                          | Bengio Neural LM                            |
|----------------------------------------|---------------------------------------------|
| Words are atomic, discrete symbols     | Words are dense vectors in continuous space  |
| "cat" and "kitten" are unrelated       | Similar words have similar embeddings        |
| Unseen n-gram → zero probability       | Network can generalize to unseen contexts    |
| Parameters grow as O(V^n)              | Parameters grow as O(V·d + d·H + H·V)       |
| Requires explicit smoothing            | Implicit smoothing via continuous space      |

---

## 3. RNN Language Models

The feedforward neural LM still has a **fixed context window**, just like n-grams. **Recurrent Neural Networks (RNNs)** remove this limitation by maintaining a **hidden state** that is updated at every time step, theoretically allowing the model to condition on the entire preceding sequence.

### 3.1 RNN Architecture — Unrolled View

```
   w₁         w₂         w₃         w₄         w₅
   │          │          │          │          │
   ▼          ▼          ▼          ▼          ▼
┌──────┐  ┌──────┐  ┌──────┐  ┌──────┐  ┌──────┐
│Embed │  │Embed │  │Embed │  │Embed │  │Embed │
│ (C)  │  │ (C)  │  │ (C)  │  │ (C)  │  │ (C)  │
└──┬───┘  └──┬───┘  └──┬───┘  └──┬───┘  └──┬───┘
   │         │         │         │         │
   ▼         ▼         ▼         ▼         ▼
┌──────┐  ┌──────┐  ┌──────┐  ┌──────┐  ┌──────┐
│      │─→│      │─→│      │─→│      │─→│      │
│  h₁  │  │  h₂  │  │  h₃  │  │  h₄  │  │  h₅  │   ← hidden states
│      │  │      │  │      │  │      │  │      │     carry information
└──┬───┘  └──┬───┘  └──┬───┘  └──┬───┘  └──┬───┘     forward in time
   │         │         │         │         │
   ▼         ▼         ▼         ▼         ▼
┌──────┐  ┌──────┐  ┌──────┐  ┌──────┐  ┌──────┐
│Softmax│ │Softmax│ │Softmax│ │Softmax│ │Softmax│
│ P(w₂)│  │ P(w₃)│  │ P(w₄)│  │ P(w₅)│  │P(w₆) │   ← predict next word
└──────┘  └──────┘  └──────┘  └──────┘  └──────┘       at each step

h₀ = 0⃗  (initial hidden state is zero vector)
```

### 3.2 RNN Equations

At each time step t:

```
1. Embedding:    xₜ = C[wₜ]

2. Hidden state: hₜ = tanh(Wₕ · hₜ₋₁ + Wₓ · xₜ + b)
                      ─────────────  ──────────
                      recurrence     new input

3. Output:       yₜ = U · hₜ + d

4. Prediction:   P(wₜ₊₁ | w₁, ..., wₜ) = softmax(yₜ)
```

> **Key Insight:** The hidden state hₜ acts as a **compressed summary** of the entire history w₁, w₂, ..., wₜ. Unlike n-gram or feedforward models, the context window is theoretically **unlimited**.

### 3.3 The Vanishing Gradient Problem

In practice, vanilla RNNs struggle to learn **long-range dependencies**. During backpropagation through time (BPTT), gradients must flow through many time steps:

```
∂L/∂h₁ = ∂L/∂hₜ · ∂hₜ/∂hₜ₋₁ · ∂hₜ₋₁/∂hₜ₋₂ · ... · ∂h₂/∂h₁

Each factor ∂hₖ/∂hₖ₋₁ involves multiplying by Wₕ and the derivative of tanh.

If |eigenvalues of Wₕ| < 1 → gradients shrink exponentially    (VANISH)
If |eigenvalues of Wₕ| > 1 → gradients grow exponentially      (EXPLODE)
```

**Result:** The model "forgets" distant context. For example, in *"The students who passed the difficult exam [were] happy"*, the RNN may fail to connect "students" (plural) with "were" when many words intervene.

---

## 4. LSTM Language Models

**Long Short-Term Memory (LSTM)** networks (Hochreiter & Schmidhuber, 1997) were designed to solve the vanishing gradient problem by introducing a **cell state** — a dedicated memory highway — controlled by learnable **gates**.

### 4.1 Gate Mechanism — Brief Recap

```
┌──────────────────────────────────────────────────────────────────┐
│                        LSTM CELL at time t                       │
│                                                                  │
│  Inputs: xₜ (current embedding), hₜ₋₁ (prev hidden), cₜ₋₁     │
│                                                                  │
│  ┌──────────────┐                                                │
│  │ Forget Gate   │  fₜ = σ(Wf · [hₜ₋₁, xₜ] + bf)              │
│  │  "what to     │  → values ∈ (0,1): how much of cₜ₋₁ to keep │
│  │   discard"    │                                                │
│  ├──────────────┤                                                │
│  │ Input Gate    │  iₜ = σ(Wi · [hₜ₋₁, xₜ] + bi)              │
│  │  "what to     │  c̃ₜ = tanh(Wc · [hₜ₋₁, xₜ] + bc)          │
│  │   add"        │  → iₜ controls how much of candidate c̃ₜ     │
│  │               │    to write into cell state                    │
│  ├──────────────┤                                                │
│  │ Cell Update   │  cₜ = fₜ ⊙ cₜ₋₁ + iₜ ⊙ c̃ₜ                │
│  │  "new memory" │  → additive update (no vanishing gradient!)   │
│  ├──────────────┤                                                │
│  │ Output Gate   │  oₜ = σ(Wo · [hₜ₋₁, xₜ] + bo)              │
│  │  "what to     │  hₜ = oₜ ⊙ tanh(cₜ)                        │
│  │   output"     │  → selectively expose cell state              │
│  └──────────────┘                                                │
│                                                                  │
│  Key: σ = sigmoid, ⊙ = element-wise multiply                    │
└──────────────────────────────────────────────────────────────────┘
```

### 4.2 Why LSTMs Solve Vanishing Gradients

```
Vanilla RNN:   hₜ = tanh(Wₕ · hₜ₋₁ + Wₓ · xₜ)      ← multiplicative, gradients decay

LSTM:          cₜ = fₜ ⊙ cₜ₋₁ + iₜ ⊙ c̃ₜ            ← ADDITIVE update to cell state

The cell state cₜ acts as a "gradient highway":
  ∂cₜ/∂cₜ₋₁ = fₜ    (forget gate, close to 1 when memory should persist)

When fₜ ≈ 1:  gradients flow unchanged across many time steps!
```

> **Key Insight:** The additive cell state update means the LSTM can carry information over hundreds of time steps without gradient degradation, enabling it to model long-range dependencies like subject-verb agreement across clauses.

---

## 5. Advantages over N-gram Models

Neural language models address all the fundamental limitations of n-gram models:

```
┌────────────────────────┬──────────────────────────┬──────────────────────────────┐
│      Problem           │   N-gram Models          │   Neural Language Models     │
├────────────────────────┼──────────────────────────┼──────────────────────────────┤
│ 1. Sparsity            │ Most n-grams have zero   │ Continuous embeddings        │
│                        │ counts → need smoothing  │ → no zero probabilities      │
│                        │                          │ naturally                    │
├────────────────────────┼──────────────────────────┼──────────────────────────────┤
│ 2. Generalization      │ "cat sat" and "kitten    │ Similar words have similar   │
│                        │ sat" are independent     │ embeddings → knowledge       │
│                        │                          │ transfers automatically      │
├────────────────────────┼──────────────────────────┼──────────────────────────────┤
│ 3. Context Length      │ Fixed to (n-1) words;    │ RNN/LSTM: theoretically      │
│                        │ n > 5 impractical        │ unlimited context via        │
│                        │                          │ recurrent hidden state       │
├────────────────────────┼──────────────────────────┼──────────────────────────────┤
│ 4. Storage             │ Must store all n-gram    │ Compact parameters (weight   │
│                        │ counts; grows with V^n   │ matrices); millions, not     │
│                        │                          │ billions of counts           │
├────────────────────────┼──────────────────────────┼──────────────────────────────┤
│ 5. Semantic Awareness  │ No notion of word        │ Embeddings capture           │
│                        │ similarity or meaning    │ semantic relationships       │
│                        │                          │ (king - man + woman ≈ queen) │
└────────────────────────┴──────────────────────────┴──────────────────────────────┘
```

---

## 6. Training Neural Language Models

### 6.1 Objective: Next-Word Prediction

All language models are trained with the same fundamental objective — predict the next word given the preceding context:

```
At each position t in the training corpus:

  Input:   w₁, w₂, ..., wₜ₋₁     (context)
  Target:  wₜ                      (actual next word)
  Output:  P̂(wₜ | w₁, ..., wₜ₋₁)  (model's predicted distribution)

The model learns by adjusting weights to make P̂(wₜ) as high as possible
for the correct next word across all positions in the training corpus.
```

### 6.2 Cross-Entropy Loss

The loss function for language modeling is the **cross-entropy** between the true distribution (one-hot) and the predicted distribution:

```
For a single prediction at position t:

  L_t = −log P̂(wₜ | w₁, ..., wₜ₋₁)

  This is the negative log probability assigned to the CORRECT next word.

Over an entire corpus of N tokens:

            1   N
  L = − ─── · Σ  log P̂(wₜ | w₁, ..., wₜ₋₁)
            N  t=1

Connection to perplexity:   PP = 2^L     (when using log₂)
  ⟹ Minimizing cross-entropy = minimizing perplexity
```

### 6.3 Training Pipeline

```
┌─────────────────────────────────────────────────────────────────┐
│                    TRAINING PIPELINE                            │
│                                                                 │
│  1. Tokenize corpus → [w₁, w₂, w₃, ..., wₙ]                  │
│                                                                 │
│  2. Create (context, target) pairs:                             │
│     ([w₁,...,wₜ₋₁], wₜ) for each position t                   │
│                                                                 │
│  3. Forward pass:                                               │
│     context → embeddings → RNN/LSTM → softmax → P̂(wₜ)         │
│                                                                 │
│  4. Compute loss:                                               │
│     L = −log P̂(wₜ)                                             │
│                                                                 │
│  5. Backward pass:                                              │
│     Compute gradients ∂L/∂θ via backpropagation (BPTT for RNN) │
│                                                                 │
│  6. Update parameters:                                          │
│     θ ← θ − η · ∂L/∂θ     (SGD, Adam, etc.)                   │
│                                                                 │
│  7. Repeat for all positions, all epochs                        │
└─────────────────────────────────────────────────────────────────┘
```

---

## 7. Python Implementation — Simple RNN Language Model in PyTorch

```python
import torch
import torch.nn as nn
import torch.optim as optim

# ─── Model Definition ────────────────────────────────────────────

class RNNLanguageModel(nn.Module):
    """A simple RNN-based language model."""

    def __init__(self, vocab_size, embed_dim, hidden_dim):
        super().__init__()
        self.embedding = nn.Embedding(vocab_size, embed_dim)
        self.rnn = nn.RNN(embed_dim, hidden_dim, batch_first=True)
        self.fc = nn.Linear(hidden_dim, vocab_size)

    def forward(self, x, hidden=None):
        # x: (batch, seq_len) — token indices
        emb = self.embedding(x)              # (batch, seq_len, embed_dim)
        out, hidden = self.rnn(emb, hidden)  # out: (batch, seq_len, hidden_dim)
        logits = self.fc(out)                # (batch, seq_len, vocab_size)
        return logits, hidden


# ─── Data Preparation ────────────────────────────────────────────

corpus = "the cat sat on the mat the cat ate the fish".split()

# Build vocabulary
vocab = sorted(set(corpus))
word2idx = {w: i for i, w in enumerate(vocab)}
idx2word = {i: w for w, i in word2idx.items()}
vocab_size = len(vocab)

# Create input-target pairs: input is tokens[:-1], target is tokens[1:]
indices = [word2idx[w] for w in corpus]
input_seq = torch.tensor(indices[:-1]).unsqueeze(0)   # (1, seq_len)
target_seq = torch.tensor(indices[1:]).unsqueeze(0)   # (1, seq_len)


# ─── Hyperparameters ─────────────────────────────────────────────

EMBED_DIM = 16
HIDDEN_DIM = 32
LEARNING_RATE = 0.01
EPOCHS = 200


# ─── Training Loop ───────────────────────────────────────────────

model = RNNLanguageModel(vocab_size, EMBED_DIM, HIDDEN_DIM)
criterion = nn.CrossEntropyLoss()
optimizer = optim.Adam(model.parameters(), lr=LEARNING_RATE)

for epoch in range(EPOCHS):
    model.train()
    optimizer.zero_grad()

    logits, _ = model(input_seq)
    # logits: (1, seq_len, vocab_size) → reshape for cross-entropy
    loss = criterion(logits.view(-1, vocab_size), target_seq.view(-1))

    loss.backward()
    optimizer.step()

    if (epoch + 1) % 50 == 0:
        print(f"Epoch {epoch+1:3d} | Loss: {loss.item():.4f} | "
              f"Perplexity: {2**loss.item():.2f}")


# ─── Next-Word Prediction ────────────────────────────────────────

model.eval()
test_context = ["the", "cat"]
test_indices = torch.tensor([[word2idx[w] for w in test_context]])

with torch.no_grad():
    logits, _ = model(test_indices)
    probs = torch.softmax(logits[0, -1], dim=0)  # distribution after last token

print(f"\nP(w | '{' '.join(test_context)}'):")
for idx in probs.argsort(descending=True):
    print(f"  {idx2word[idx.item()]:>8s}  {probs[idx].item():.4f}")
```

**Expected output (after training):**
```
Epoch  50 | Loss: 1.2345 | Perplexity: 2.35
Epoch 100 | Loss: 0.5678 | Perplexity: 1.48
Epoch 150 | Loss: 0.2345 | Perplexity: 1.18
Epoch 200 | Loss: 0.1234 | Perplexity: 1.09

P(w | 'the cat'):
       sat  0.4523
       ate  0.4412
      fish  0.0321
       ...
```

> **Note:** In practice, RNN language models are trained on millions of tokens with batching, gradient clipping, dropout, and learning rate schedules. The example above is minimal for clarity.

---

## 8. Summary & Comparison Table

| Aspect                | N-gram LM               | Feedforward Neural LM          | RNN LM                         | LSTM LM                         |
|-----------------------|--------------------------|--------------------------------|---------------------------------|----------------------------------|
| **Year / Origin**     | 1980s+                   | Bengio et al. 2003             | Mikolov et al. 2010            | Sundermeyer et al. 2012         |
| **Context**           | Fixed (n−1) words        | Fixed (n−1) words              | Theoretically unlimited         | Theoretically unlimited          |
| **Representation**    | Discrete counts          | Learned embeddings             | Learned embeddings + hidden     | Learned embeddings + gated cells |
| **Handles Sparsity?** | No — needs smoothing     | Yes — continuous space          | Yes — continuous space           | Yes — continuous space            |
| **Generalization**    | None across words        | Shares via embeddings          | Shares via embeddings + memory  | Shares via embeddings + memory   |
| **Long Dependencies** | ✗ (max n ≈ 5)           | ✗ (fixed window)               | Weak (vanishing gradients)      | ✓ (gated cell state)            |
| **Training**          | Counting + smoothing     | SGD / backprop                 | BPTT                            | BPTT with gates                  |
| **Speed (train)**     | Fast (one-pass counting) | Moderate                       | Slow (sequential)               | Slower (more params per step)    |
| **Speed (inference)** | O(1) table lookup        | Fast (single forward pass)     | Sequential (step-by-step)       | Sequential (step-by-step)        |
| **Perplexity**        | Baseline                 | Better than n-gram             | Better than feedforward         | Best of these four               |
| **Key Weakness**      | Sparsity, no semantics   | Fixed context window           | Vanishing gradients             | Sequential; superseded by Transformers |

---

## 9. Quick Revision Questions

1. **What two problems of n-gram models did Bengio et al. (2003) solve with their neural language model?** *(Answer: (a) The curse of dimensionality / sparsity — by using continuous word embeddings instead of discrete counts; (b) Lack of generalization — similar words share similar embeddings, so knowledge transfers across related contexts.)*

2. **In the feedforward neural LM, what is the role of the embedding matrix C?** *(Answer: C maps each word index to a dense, low-dimensional vector (the word embedding). It is a learnable |V| × d matrix where row C[w] is the embedding for word w.)*

3. **How does an RNN language model differ from the feedforward neural LM in terms of context?** *(Answer: The feedforward model uses a fixed window of (n−1) words as context, while the RNN maintains a hidden state hₜ that is updated at each step, theoretically encoding the entire previous sequence.)*

4. **Explain why vanilla RNNs suffer from vanishing gradients. How do LSTMs address this?** *(Answer: In vanilla RNNs, gradients are repeatedly multiplied by the weight matrix Wₕ during BPTT; if its eigenvalues are < 1, gradients shrink exponentially. LSTMs solve this with an additive cell state update: cₜ = fₜ ⊙ cₜ₋₁ + iₜ ⊙ c̃ₜ. When the forget gate fₜ ≈ 1, gradients flow unchanged through the cell state highway.)*

5. **What is the training objective for neural language models, and how does it relate to perplexity?** *(Answer: The objective is to minimize cross-entropy loss L = −(1/N) Σ log P̂(wₜ | context), which is equivalent to minimizing perplexity since PP = 2^L.)*

6. **Why do neural language models not require explicit smoothing techniques like Kneser-Ney?** *(Answer: Because the softmax output layer always assigns non-zero probability to every word in the vocabulary. The continuous embedding space also allows the model to assign reasonable probabilities to unseen word combinations by leveraging similarity between embeddings.)*

---

> [← Previous: Perplexity](./02-perplexity.md) | [Next → Pretraining & Fine-tuning](./04-pretraining-finetuning.md)

[🔼 Back to Unit Overview](../README.md)
