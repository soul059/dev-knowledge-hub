# GPT: Generative Pre-trained Transformer

> **Navigation:** [← BERT Applications](../07-BERT-and-Encoder-Models/05-bert-applications.md) | [Autoregressive Language Modeling →](02-autoregressive-language-modeling.md)

---

## Overview

The **Generative Pre-trained Transformer (GPT)** introduced by OpenAI (Radford et al., 2018) is a decoder-only transformer architecture designed for causal (left-to-right) language modeling. Unlike BERT, which uses a bidirectional encoder, GPT generates text autoregressively by predicting the next token given all previous tokens. The model is first **pre-trained** on a large unsupervised text corpus using a language modeling objective and then **fine-tuned** on downstream tasks with minimal architectural changes. GPT demonstrated that a single generative model could achieve strong performance across diverse NLP tasks, laying the foundation for the GPT-2, GPT-3, and GPT-4 family of models.

---

## 1. Decoder-Only Transformer Architecture

GPT uses only the **decoder** portion of the original Transformer (Vaswani et al., 2017), with a critical modification: all attention is **masked** (causal) so that each token can only attend to itself and previous tokens.

### GPT-1 Architecture Specifications

| Parameter            | Value           |
|----------------------|-----------------|
| Transformer layers   | 12              |
| Hidden dimension     | 768             |
| Attention heads      | 12              |
| Head dimension       | 64 (768 / 12)   |
| Feed-forward dim     | 3072 (4 × 768)  |
| Max sequence length  | 512             |
| Vocabulary size      | 40,000 (BPE)    |
| Total parameters     | ~117M           |

### ASCII Diagram: GPT Decoder-Only Architecture

```
                    ┌──────────────────┐
                    │   Output Logits  │   (vocab_size)
                    │   over Vocab     │
                    └────────┬─────────┘
                             │
                    ┌────────▼─────────┐
                    │   Layer Norm     │
                    └────────┬─────────┘
                             │
              ┌──────────────┴──────────────┐
              │        × 12 Layers          │
              │  ┌───────────────────────┐  │
              │  │    Feed-Forward Net   │  │
              │  │  Linear(768→3072)     │  │
              │  │  GELU Activation      │  │
              │  │  Linear(3072→768)     │  │
              │  │  + Residual + LN      │  │
              │  └───────────┬───────────┘  │
              │              │              │
              │  ┌───────────▼───────────┐  │
              │  │  Masked Self-Attention │  │
              │  │  (Causal / Left-to-   │  │
              │  │   Right only)         │  │
              │  │  + Residual + LN      │  │
              │  └───────────┬───────────┘  │
              └──────────────┬──────────────┘
                             │
                    ┌────────▼─────────┐
                    │  Token Embed     │
                    │  + Position Embed│
                    └────────┬─────────┘
                             │
                    ┌────────▼─────────┐
                    │  Input Tokens    │
                    │  [w₁, w₂, ..wₙ] │
                    └──────────────────┘
```

---

## 2. Causal Language Modeling

GPT is trained with a **causal (unidirectional) language modeling** objective: predict the next token given all preceding tokens.

### Mathematical Formulation

Given a sequence of tokens **x = (x₁, x₂, ..., xₙ)**, the training objective maximizes the log-likelihood:

```
L(θ) = Σᵢ log P(xᵢ | x₁, x₂, ..., xᵢ₋₁ ; θ)
```

The probability is computed by:

```
h₀ = W_e · x + W_p                       (token + positional embedding)
hₗ = transformer_block(hₗ₋₁)   for l = 1, ..., 12
P(xᵢ | x<ᵢ) = softmax(hₙ · W_eᵀ)        (tie input/output embeddings)
```

### Causal Mask

The causal mask ensures position `i` can only attend to positions `j ≤ i`:

```
         w₁   w₂   w₃   w₄
   w₁  [  1    0    0    0  ]
   w₂  [  1    1    0    0  ]     0 = masked (future)
   w₃  [  1    1    1    0  ]     1 = visible (past + current)
   w₄  [  1    1    1    1  ]
```

In practice, the mask sets future positions to **−∞** before the softmax, so their attention weights become zero.

---

## 3. GPT vs. BERT: Key Differences

| Feature              | GPT (Decoder)            | BERT (Encoder)              |
|----------------------|--------------------------|-----------------------------|
| Architecture         | Decoder-only             | Encoder-only                |
| Attention direction  | Unidirectional (causal)  | Bidirectional               |
| Pre-training task    | Next token prediction    | Masked LM + NSP            |
| Generation           | Naturally generative     | Not designed for generation |
| Task adaptation      | Fine-tune or prompting   | Fine-tune with task head    |
| Paradigm             | Generative               | Discriminative              |
| Mask type            | Causal (triangular)      | No mask (full attention)    |

```
  GPT (Causal / Left-to-Right):        BERT (Bidirectional):

  Token:  [The] [cat] [sat]            Token:  [The] [MASK] [sat]
            │     │     │                        │      │      │
            ▼     ▼     ▼                        ▼      ▼      ▼
  Attends: [The] [The] [The]           Attends: [The]  [The]  [The]
                  [cat] [cat]                    [MASK] [MASK] [MASK]
                        [sat]                    [sat]  [sat]  [sat]
```

---

## 4. Byte-Pair Encoding (BPE) Tokenization

GPT uses **Byte-Pair Encoding** to create its vocabulary, which handles rare and out-of-vocabulary words by breaking them into subword units.

### BPE Algorithm

```
1. Start with character-level vocabulary
2. Count all adjacent symbol pairs in corpus
3. Merge the most frequent pair into a new symbol
4. Repeat steps 2-3 for k merge operations (k = desired vocab size - base chars)
```

### Worked Example

```
Corpus:  "low lower lowest"

Initial tokens:   l o w </w>   l o w e r </w>   l o w e s t </w>

Step 1: Most frequent pair = (l, o) → merge to "lo"
         lo w </w>   lo w e r </w>   lo w e s t </w>

Step 2: Most frequent pair = (lo, w) → merge to "low"
         low </w>   low e r </w>   low e s t </w>

Step 3: Most frequent pair = (low, e) → merge to "lowe"
         low </w>   lowe r </w>   lowe s t </w>

Final vocabulary includes: l, o, w, e, r, s, t, lo, low, lowe, ...
```

---

## 5. Pre-training and Fine-tuning Pipeline

### Pre-training (Unsupervised)

```
                    ┌─────────────────┐
  Large Corpus ───► │  GPT (Decoder)  │ ───► Minimize -log P(xₜ|x<ₜ)
  (BooksCorpus)     │  12 layers      │
                    └─────────────────┘
```

### Fine-tuning (Supervised)

For downstream tasks, GPT adds a linear classification head on top of the final transformer output at a special token position:

```
  Input:   [START] premise [DELIM] hypothesis [EXTRACT]
                                                  │
                                         ┌────────▼────────┐
                                         │  Linear + Softmax│
                                         │  (task-specific) │
                                         └─────────────────┘
```

The fine-tuning loss combines the task loss and language modeling loss:

```
L_total = L_task + λ · L_LM
```

where λ is a hyperparameter (typically 0.5) that controls auxiliary LM contribution.

---

## 6. Implementation in PyTorch

```python
import torch
import torch.nn as nn
import math


class CausalSelfAttention(nn.Module):
    """Multi-head causal (masked) self-attention."""

    def __init__(self, d_model, n_heads, max_seq_len, dropout=0.1):
        super().__init__()
        assert d_model % n_heads == 0
        self.n_heads = n_heads
        self.head_dim = d_model // n_heads

        self.qkv = nn.Linear(d_model, 3 * d_model)
        self.proj = nn.Linear(d_model, d_model)
        self.dropout = nn.Dropout(dropout)

        # Causal mask: upper triangular = -inf
        mask = torch.triu(torch.ones(max_seq_len, max_seq_len), diagonal=1)
        self.register_buffer("mask", mask.bool())

    def forward(self, x):
        B, T, C = x.size()
        qkv = self.qkv(x).reshape(B, T, 3, self.n_heads, self.head_dim)
        qkv = qkv.permute(2, 0, 3, 1, 4)          # (3, B, heads, T, head_dim)
        q, k, v = qkv[0], qkv[1], qkv[2]

        attn = (q @ k.transpose(-2, -1)) / math.sqrt(self.head_dim)
        attn = attn.masked_fill(self.mask[:T, :T], float('-inf'))
        attn = torch.softmax(attn, dim=-1)
        attn = self.dropout(attn)

        out = (attn @ v).transpose(1, 2).reshape(B, T, C)
        return self.proj(out)


class FeedForward(nn.Module):
    def __init__(self, d_model, d_ff, dropout=0.1):
        super().__init__()
        self.net = nn.Sequential(
            nn.Linear(d_model, d_ff),
            nn.GELU(),
            nn.Linear(d_ff, d_model),
            nn.Dropout(dropout),
        )

    def forward(self, x):
        return self.net(x)


class TransformerBlock(nn.Module):
    def __init__(self, d_model, n_heads, d_ff, max_seq_len, dropout=0.1):
        super().__init__()
        self.ln1 = nn.LayerNorm(d_model)
        self.attn = CausalSelfAttention(d_model, n_heads, max_seq_len, dropout)
        self.ln2 = nn.LayerNorm(d_model)
        self.ff = FeedForward(d_model, d_ff, dropout)

    def forward(self, x):
        x = x + self.attn(self.ln1(x))
        x = x + self.ff(self.ln2(x))
        return x


class GPT(nn.Module):
    """GPT-1 style decoder-only transformer."""

    def __init__(
        self,
        vocab_size=40000,
        d_model=768,
        n_heads=12,
        n_layers=12,
        d_ff=3072,
        max_seq_len=512,
        dropout=0.1,
    ):
        super().__init__()
        self.token_emb = nn.Embedding(vocab_size, d_model)
        self.pos_emb = nn.Embedding(max_seq_len, d_model)
        self.drop = nn.Dropout(dropout)

        self.blocks = nn.ModuleList([
            TransformerBlock(d_model, n_heads, d_ff, max_seq_len, dropout)
            for _ in range(n_layers)
        ])

        self.ln_f = nn.LayerNorm(d_model)
        self.head = nn.Linear(d_model, vocab_size, bias=False)

        # Weight tying: share token embedding and output projection
        self.head.weight = self.token_emb.weight

    def forward(self, idx):
        B, T = idx.size()
        positions = torch.arange(T, device=idx.device).unsqueeze(0)

        x = self.drop(self.token_emb(idx) + self.pos_emb(positions))
        for block in self.blocks:
            x = block(x)
        x = self.ln_f(x)
        logits = self.head(x)                  # (B, T, vocab_size)
        return logits


# --- Instantiate and inspect ---
model = GPT()
total_params = sum(p.numel() for p in model.parameters())
print(f"GPT-1 parameters: {total_params / 1e6:.1f}M")

dummy_input = torch.randint(0, 40000, (2, 128))
logits = model(dummy_input)
print(f"Input shape:  {dummy_input.shape}")
print(f"Output shape: {logits.shape}")
# Output:
# GPT-1 parameters: ~117.0M
# Input shape:  torch.Size([2, 128])
# Output shape: torch.Size([2, 128, 40000])
```

---

## 7. Real-World Applications

| Application               | Description                                                |
|----------------------------|------------------------------------------------------------|
| **Text Generation**        | Story writing, article generation, creative writing        |
| **Code Generation**        | Autocomplete and code suggestion (Codex, GitHub Copilot)   |
| **Chatbots**               | Conversational agents (ChatGPT, customer support bots)     |
| **Summarization**          | Generate concise summaries of long documents               |
| **Translation**            | Generative approach to machine translation                 |
| **Question Answering**     | Generate answers from context or parametric knowledge       |

---

## 8. Summary Table

| Concept                    | Key Point                                                  |
|----------------------------|------------------------------------------------------------|
| Architecture               | Decoder-only transformer (masked self-attention)           |
| Training objective         | Causal language modeling (next token prediction)           |
| Attention direction        | Unidirectional (left-to-right)                             |
| Tokenization               | Byte-Pair Encoding (BPE), ~40K vocab                      |
| Pre-training               | Unsupervised on BooksCorpus                                |
| Fine-tuning                | Task loss + auxiliary LM loss (λ = 0.5)                    |
| Parameters (GPT-1)         | 117M (12 layers, 768 hidden, 12 heads)                     |
| Key innovation             | Generative pre-training + discriminative fine-tuning       |

---

## 9. Revision Questions

1. **Why does GPT use a causal mask instead of full bidirectional attention?** Explain how this connects to the autoregressive generation process.

2. **Write the causal language modeling objective.** What is being maximized, and how does the causal mask enforce the left-to-right dependency?

3. **Compare GPT and BERT architecturally.** When would you choose a decoder-only model over an encoder-only model?

4. **Explain Byte-Pair Encoding (BPE).** Walk through the merge process for a small example vocabulary.

5. **How does GPT's fine-tuning combine task loss and language modeling loss?** Why is the auxiliary LM loss beneficial?

6. **What is weight tying in GPT?** Why is `self.head.weight = self.token_emb.weight` used and what are its benefits?

---

> **Navigation:** [← BERT Applications](../07-BERT-and-Encoder-Models/05-bert-applications.md) | [Autoregressive Language Modeling →](02-autoregressive-language-modeling.md)
