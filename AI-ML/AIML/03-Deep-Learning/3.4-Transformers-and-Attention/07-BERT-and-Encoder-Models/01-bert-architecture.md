[← Inference Optimization](../06-Transformer-Training/05-inference-optimization.md) | [Pretraining Objectives →](02-pretraining-objectives.md)

---

# BERT: Bidirectional Encoder Representations from Transformers

BERT (Bidirectional Encoder Representations from Transformers) is a landmark language model introduced by Google AI in 2018 that fundamentally changed how NLP tasks are approached. Unlike previous models that read text either left-to-right or right-to-left, BERT reads the entire sequence of words at once, enabling it to understand the full context of a word based on all of its surroundings. This "bidirectionality" is what gives BERT its power: it learns deep representations that can be fine-tuned for a wide range of downstream tasks — from question answering to sentiment analysis — achieving state-of-the-art results across numerous benchmarks with minimal task-specific architecture changes.

---

## 1. Model Configurations

BERT comes in two standard configurations:

| Parameter            | BERT-base      | BERT-large     |
|----------------------|----------------|----------------|
| Transformer layers   | 12             | 24             |
| Hidden size (d_model)| 768            | 1024           |
| Attention heads      | 12             | 16             |
| Feed-forward size    | 3072           | 4096           |
| Total parameters     | 110M           | 340M           |
| Max sequence length  | 512            | 512            |
| Vocabulary size      | 30,522         | 30,522         |

### Parameter Count Breakdown (BERT-base)

The total parameter count can be estimated as:

```
Embedding parameters:
  Token embeddings:     V × d = 30,522 × 768 = 23.4M
  Position embeddings:  512 × 768           = 0.39M
  Segment embeddings:   2 × 768             = 0.0015M

Per Transformer layer:
  Multi-head attention: 4 × d² = 4 × 768² = 2.36M
  Feed-forward:         2 × d × d_ff = 2 × 768 × 3072 = 4.72M
  Layer norms:          4 × d = 4 × 768    = 0.003M
  Subtotal per layer:   ≈ 7.08M

12 layers total:        12 × 7.08M = 85.0M
Pooler (d × d):         768 × 768 = 0.59M
──────────────────────────────────────────
Grand total:            ≈ 110M parameters
```

---

## 2. Architecture Overview

BERT uses only the **encoder** portion of the original Transformer architecture. Below is a high-level diagram:

```
                        BERT Architecture (BERT-base)
 ═══════════════════════════════════════════════════════════════

  Input:   [CLS] The cat sat on the mat [SEP]

           ↓         ↓    ↓   ↓   ↓   ↓   ↓      ↓
  ┌────────────────────────────────────────────────────────┐
  │              INPUT EMBEDDING LAYER                     │
  │  Token Embed + Segment Embed + Position Embed          │
  └────────────────────────────────────────────────────────┘
           ↓         ↓    ↓   ↓   ↓   ↓   ↓      ↓
  ┌────────────────────────────────────────────────────────┐
  │            Transformer Encoder Block 1                 │
  │  ┌──────────────────────────────────────────────┐      │
  │  │   Multi-Head Self-Attention (12 heads)       │      │
  │  │   Attention(Q, K, V) = softmax(QKᵀ/√dₖ)V    │      │
  │  └──────────────────────────────────────────────┘      │
  │              ↓  (Add & LayerNorm)                      │
  │  ┌──────────────────────────────────────────────┐      │
  │  │   Position-wise Feed-Forward Network         │      │
  │  │   FFN(x) = GELU(xW₁ + b₁)W₂ + b₂           │      │
  │  └──────────────────────────────────────────────┘      │
  │              ↓  (Add & LayerNorm)                      │
  └────────────────────────────────────────────────────────┘
           ↓         ↓    ↓   ↓   ↓   ↓   ↓      ↓
          ...       ...  ... ... ... ... ...     ...
           ↓         ↓    ↓   ↓   ↓   ↓   ↓      ↓
  ┌────────────────────────────────────────────────────────┐
  │            Transformer Encoder Block 12                │
  │  (Same structure as Block 1)                           │
  └────────────────────────────────────────────────────────┘
           ↓         ↓    ↓   ↓   ↓   ↓   ↓      ↓
  ┌────────────────────────────────────────────────────────┐
  │                 OUTPUT LAYER                           │
  │  [CLS]→ pooled output    Token outputs → per-token    │
  └────────────────────────────────────────────────────────┘
```

### Key Architectural Difference: Bidirectional vs. Unidirectional

```
  GPT (Unidirectional / Autoregressive):
  ──────────────────────────────────────
  Token:   The   cat   sat   on    the   mat
  Sees:    ←     ←←    ←←←   ←←←←  ←←←←← ←←←←←←
  Each token can ONLY attend to previous tokens (left context).

  BERT (Bidirectional):
  ─────────────────────
  Token:   The   cat   sat   on    the   mat
  Sees:    ↔↔↔↔↔ ↔↔↔↔↔ ↔↔↔↔↔ ↔↔↔↔↔ ↔↔↔↔↔ ↔↔↔↔↔
  Each token attends to ALL tokens in the sequence (full context).
```

---

## 3. Input Representation

BERT's input is constructed by summing three different embeddings:

```
  Input Representation
  ════════════════════

  Tokens:     [CLS]  I    love  deep  learn  ##ing  [SEP]  It  rocks  [SEP]
               ↓     ↓     ↓     ↓     ↓      ↓      ↓     ↓    ↓      ↓
  Token       E_CLS  E_I  E_lov E_dee E_lea  E_##i  E_SEP  E_It E_roc  E_SEP
  Embed        +      +     +     +     +      +      +      +    +      +
  Segment     E_A    E_A   E_A   E_A   E_A    E_A    E_A    E_B  E_B    E_B
  Embed        +      +     +     +     +      +      +      +    +      +
  Position    E_0    E_1   E_2   E_3   E_4    E_5    E_6    E_7  E_8    E_9
  Embed
               ↓      ↓     ↓     ↓     ↓      ↓      ↓     ↓    ↓      ↓
             ┌────────────────────────────────────────────────────────────────┐
             │           Combined Embedding = Token + Segment + Position     │
             └────────────────────────────────────────────────────────────────┘
```

### Formal Definition

For token at position `i` in segment `s`:

```
Input(i) = TokenEmbed(token_i) + SegmentEmbed(s) + PositionEmbed(i)
```

Where:
- **TokenEmbed** ∈ ℝ^(V × d): maps each vocabulary token to a d-dimensional vector
- **SegmentEmbed** ∈ ℝ^(2 × d): distinguishes sentence A (0) from sentence B (1)
- **PositionEmbed** ∈ ℝ^(512 × d): learned (not sinusoidal) positional encoding

### Special Tokens

| Token   | Purpose                                              |
|---------|------------------------------------------------------|
| `[CLS]` | First token; its output used for classification      |
| `[SEP]` | Separates sentence pairs; marks end of input         |
| `[MASK]`| Replaces tokens during Masked Language Model training|
| `[PAD]` | Padding token to equalize sequence lengths in batch  |
| `[UNK]` | Unknown / out-of-vocabulary token                    |

---

## 4. WordPiece Tokenization

BERT uses **WordPiece** tokenization, a subword method that balances vocabulary size with coverage:

```
Original:     "unbelievable"
WordPiece:    ["un", "##believ", "##able"]

Original:     "embeddings"
WordPiece:    ["embed", "##ding", "##s"]

The '##' prefix indicates a continuation of the previous token.
```

### How WordPiece Works

1. Start with a base vocabulary of individual characters
2. Iteratively merge the most frequent pair of tokens
3. Stop when vocabulary reaches target size (30,522 for BERT)
4. At inference, greedily match the longest subword from left to right

**Advantages:**
- Handles out-of-vocabulary words by breaking them into known subwords
- Compact vocabulary while covering diverse text
- Morphologically meaningful splits (prefixes, suffixes, stems)

---

## 5. Self-Attention in BERT

Each attention head in BERT computes:

```
Attention(Q, K, V) = softmax(QKᵀ / √dₖ) V
```

Where for BERT-base:
- `dₖ = d_model / num_heads = 768 / 12 = 64`
- Q, K, V ∈ ℝ^(seq_len × 64) per head

### Multi-Head Attention

```
MultiHead(Q, K, V) = Concat(head₁, head₂, ..., head₁₂) Wᴼ

where headᵢ = Attention(XWᵢᴷ, XWᵢᵠ, XWᵢⱽ)
```

Parameters per attention layer:
- Wᵢᴷ, Wᵢᵠ, Wᵢⱽ ∈ ℝ^(768 × 64) for each of 12 heads
- Wᴼ ∈ ℝ^(768 × 768)
- Total: 4 × 768 × 768 = 2,359,296 parameters

### GELU Activation Function

BERT uses GELU instead of ReLU in its feed-forward layers:

```
GELU(x) = x · Φ(x) ≈ 0.5x(1 + tanh[√(2/π)(x + 0.044715x³)])

where Φ(x) is the cumulative distribution function of the standard normal.
```

---

## 6. Worked Example: BERT Forward Pass

Consider the input: `"The cat sat"`

```
Step 1: Tokenization
  Input text: "The cat sat"
  Tokens: ["[CLS]", "The", "cat", "sat", "[SEP]"]
  Token IDs: [101, 1996, 4937, 2938, 102]

Step 2: Embedding Lookup (d=768)
  E_token  = TokenEmbed([101, 1996, 4937, 2938, 102])   → (5, 768)
  E_seg    = SegmentEmbed([0, 0, 0, 0, 0])               → (5, 768)
  E_pos    = PositionEmbed([0, 1, 2, 3, 4])               → (5, 768)
  X₀       = E_token + E_seg + E_pos                      → (5, 768)

Step 3: Through 12 Transformer Encoder Layers
  For l = 1 to 12:
    Attention_out = MultiHeadSelfAttention(X_{l-1})         → (5, 768)
    Mid           = LayerNorm(X_{l-1} + Attention_out)      → (5, 768)
    FFN_out       = GELU(Mid · W₁ + b₁) · W₂ + b₂         → (5, 768)
    X_l           = LayerNorm(Mid + FFN_out)                → (5, 768)

Step 4: Output
  X₁₂ shape: (5, 768)
  [CLS] output (X₁₂[0]): used for classification tasks
  All token outputs: used for token-level tasks (NER, QA)
```

---

## 7. Code: Loading BERT with HuggingFace

```python
import torch
from transformers import BertModel, BertTokenizer

# Load pretrained BERT-base
tokenizer = BertTokenizer.from_pretrained("bert-base-uncased")
model = BertModel.from_pretrained("bert-base-uncased")

# Tokenize input
text = "BERT understands bidirectional context."
inputs = tokenizer(text, return_tensors="pt", padding=True, truncation=True)

print("Input IDs:", inputs["input_ids"])
print("Token Type IDs:", inputs["token_type_ids"])  # segment embeddings
print("Attention Mask:", inputs["attention_mask"])

# Forward pass
with torch.no_grad():
    outputs = model(**inputs)

# outputs.last_hidden_state: (batch, seq_len, 768) — all token representations
# outputs.pooler_output:     (batch, 768)          — [CLS] token, pooled
last_hidden = outputs.last_hidden_state
pooled = outputs.pooler_output

print(f"Last hidden state shape: {last_hidden.shape}")
print(f"Pooler output shape:     {pooled.shape}")
```

### Inspecting the Model Architecture

```python
# Count parameters
total_params = sum(p.numel() for p in model.parameters())
trainable_params = sum(p.numel() for p in model.parameters() if p.requires_grad)
print(f"Total parameters:     {total_params:,}")       # ~110M
print(f"Trainable parameters: {trainable_params:,}")

# Inspect layer structure
for name, param in model.named_parameters():
    if "layer.0." in name and "weight" in name:
        print(f"{name}: {param.shape}")
```

### Extracting Token Embeddings

```python
# Get embeddings for individual tokens
tokens = tokenizer.tokenize("unbelievable embeddings")
print(f"WordPiece tokens: {tokens}")
# Output: ['un', '##bel', '##ie', '##va', '##ble', 'em', '##bed', '##ding', '##s']

# Encode and get contextual representations
encoded = tokenizer("unbelievable embeddings", return_tensors="pt")
with torch.no_grad():
    output = model(**encoded)

# Each token now has a 768-dim context-aware representation
for i, token in enumerate(["[CLS]"] + tokens + ["[SEP]"]):
    vec = output.last_hidden_state[0, i, :5]  # first 5 dims
    print(f"  {token:12s} → [{vec[0]:.3f}, {vec[1]:.3f}, {vec[2]:.3f}, ...]")
```

### Sentence Pair Encoding

```python
# Encoding a sentence pair for tasks like NLI or QA
sentence_a = "How old are you?"
sentence_b = "I am 25 years old."

inputs = tokenizer(sentence_a, sentence_b, return_tensors="pt",
                   padding=True, truncation=True, max_length=128)

print("Tokens:", tokenizer.convert_ids_to_tokens(inputs["input_ids"][0]))
print("Segment IDs:", inputs["token_type_ids"][0].tolist())
# Segment 0: [CLS] How old are you ? [SEP]
# Segment 1: I am 25 years old . [SEP]

with torch.no_grad():
    outputs = model(**inputs)

# The [CLS] representation now encodes the relationship between both sentences
cls_embedding = outputs.pooler_output  # (1, 768)
```

---

## 8. Real-World Applications

| Application                  | How BERT Is Used                                      |
|------------------------------|-------------------------------------------------------|
| **Google Search**            | Powers query understanding in Google's search ranking  |
| **Sentiment Analysis**       | Fine-tune [CLS] output for positive/negative labels    |
| **Named Entity Recognition** | Fine-tune per-token outputs for entity labels          |
| **Question Answering**       | Predict answer span start and end positions            |
| **Text Summarization**       | Encode documents for extractive summarization          |
| **Chatbots & Assistants**    | Intent detection and slot filling                      |
| **Legal / Medical NLP**      | Domain-specific BERT variants (BioBERT, LegalBERT)     |

---

## 9. Summary Table

| Concept                | Key Details                                                |
|------------------------|------------------------------------------------------------|
| Architecture           | Encoder-only Transformer (no decoder)                      |
| Bidirectionality       | Attends to all tokens simultaneously (both directions)     |
| Input representation   | Token + Segment + Position embeddings                      |
| Special tokens         | [CLS], [SEP], [MASK], [PAD], [UNK]                        |
| Tokenization           | WordPiece (subword), vocabulary of 30,522                  |
| Activation function    | GELU (Gaussian Error Linear Unit)                          |
| Pretraining tasks      | Masked Language Model + Next Sentence Prediction           |
| BERT-base config       | 12 layers, 768 hidden, 12 heads, 110M params              |
| BERT-large config      | 24 layers, 1024 hidden, 16 heads, 340M params             |
| Max sequence length    | 512 tokens                                                 |

---

## 10. Revision Questions

1. **Why does BERT use bidirectional attention instead of unidirectional like GPT? What advantage does this provide for understanding language?**

2. **Explain the three components that make up BERT's input representation. Why is each one necessary?**

3. **BERT-base has 110M parameters. Walk through how you would estimate this number from the architecture specifications (12 layers, 768 hidden, vocabulary 30,522).**

4. **What is WordPiece tokenization and why does BERT use it instead of word-level tokenization? Give an example of how a word is split.**

5. **Describe the role of the [CLS] token in BERT. How is its output representation used differently from other token outputs?**

6. **Compare the self-attention mechanism in BERT with that in a decoder-based model like GPT. How does the attention mask differ between the two?**

---

[← Inference Optimization](../06-Transformer-Training/05-inference-optimization.md) | [Pretraining Objectives →](02-pretraining-objectives.md)
