# BART: Denoising Sequence-to-Sequence Pre-training

[← T5 Architecture](01-t5-architecture.md) | [Seq2Seq Tasks →](03-seq2seq-tasks.md)

---

## Overview

BART (Bidirectional and Auto-Regressive Transformer), introduced by Lewis et al. (2019) from Facebook AI, is a **denoising autoencoder** for pre-training sequence-to-sequence models. The core idea is straightforward: take an input, corrupt it with an arbitrary noising function, and then train the model to reconstruct the original undamaged text. BART combines the bidirectional encoder of BERT (which can read the full corrupted input) with the autoregressive decoder of GPT (which generates the output left-to-right). This marriage gives BART the ability to both understand context deeply and generate fluent text, making it particularly effective for **text generation** tasks such as summarization, translation, and question answering. BART was pre-trained on a combination of books and Wikipedia data, and its flexibility in corruption strategies allowed the authors to find the optimal noising approach empirically.

---

## 1. BART as a Denoising Autoencoder

### 1.1 High-Level Concept

```
┌──────────────┐      ┌────────────┐      ┌───────────────┐
│ Original     │ ───► │ Noising    │ ───► │ Corrupted     │
│ Text  x      │      │ Function   │      │ Text  x̃       │
└──────────────┘      └────────────┘      └───────┬───────┘
                                                  │
                                                  ▼
                                          ┌───────────────┐
                                          │   ENCODER     │
                                          │ (Bidirectional│
                                          │  Attention)   │
                                          └───────┬───────┘
                                                  │
                                                  ▼
                                          ┌───────────────┐
                                          │   DECODER     │
                                          │ (Autoregressive│
                                          │  Generation)  │
                                          └───────┬───────┘
                                                  │
                                                  ▼
                                          ┌───────────────┐
                                          │ Reconstructed │
                                          │ Text  x̂       │
                                          └───────────────┘

     Training Objective: minimize L = -log P(x | x̃)
```

### 1.2 Mathematical Objective

The pre-training loss is the negative log-likelihood of the original text given the corrupted input:

```
L(θ) = -∑_{t=1}^{T} log P_θ(x_t | x_{<t}, Encoder(x̃))
```

Where:
- `x̃` = corrupted version of original text `x`
- `Encoder(x̃)` = bidirectional encoding of corrupted input
- The decoder generates `x` autoregressively, conditioned on the encoder output

---

## 2. Corruption Strategies

BART's key innovation is exploring **five different noising strategies** during pre-training:

### 2.1 Strategy Overview

```
┌────────────────────────────────────────────────────────────────────┐
│                    BART CORRUPTION STRATEGIES                      │
├────────────────────┬───────────────────────────────────────────────┤
│                    │                                               │
│  1. TOKEN MASKING  │  Random tokens → [MASK]                      │
│                    │  "The cat sat" → "The [MASK] sat"            │
│                    │                                               │
│  2. TOKEN DELETION │  Random tokens removed entirely              │
│                    │  "The cat sat" → "The sat"                   │
│                    │                                               │
│  3. TEXT INFILLING │  Spans replaced by single [MASK]             │
│                    │  "The cat sat on mat" → "The [MASK] mat"     │
│                    │                                               │
│  4. SENTENCE       │  Sentences shuffled randomly                 │
│     PERMUTATION    │  "A. B. C." → "C. A. B."                    │
│                    │                                               │
│  5. DOCUMENT       │  Document rotated at random token            │
│     ROTATION       │  "A B C D E" → "D E A B C"                  │
│                    │                                               │
└────────────────────┴───────────────────────────────────────────────┘
```

### 2.2 Detailed Comparison

| Strategy             | Input Change         | Length Change | Difficulty | Best For         |
|----------------------|----------------------|--------------|------------|------------------|
| Token Masking        | Replace → `[MASK]`   | Same         | Medium     | Understanding    |
| Token Deletion       | Remove token         | Shorter      | Hard       | Alignment tasks  |
| Text Infilling       | Span → single `[MASK]`| Shorter     | Hardest    | Generation ✓     |
| Sentence Permutation | Reorder sentences    | Same         | Medium     | Document tasks   |
| Document Rotation    | Rotate at pivot      | Same         | Easy       | Finding start    |

### 2.3 Text Infilling — The Winning Strategy

Text infilling is the most effective strategy. Span lengths are drawn from a **Poisson distribution** with λ = 3:

```
P(span_length = k) = (λ^k × e^{-λ}) / k!

For λ = 3:
  P(k=0) = 0.050  → insert [MASK] with no deletion (0-length span)
  P(k=1) = 0.149  → replace 1 token
  P(k=2) = 0.224  → replace 2 tokens
  P(k=3) = 0.224  → replace 3 tokens (most common)
  P(k=4) = 0.168  → replace 4 tokens
  P(k=5) = 0.101  → replace 5 tokens
```

### 2.4 Text Infilling Example

```
Original:   "The quick brown fox jumps over the lazy dog by the river"

Span 1 (len=2): tokens 2-3 ("brown fox")     → [MASK]
Span 2 (len=3): tokens 7-9 ("the lazy dog")  → [MASK]

Corrupted:  "The quick [MASK] jumps over [MASK] by the river"
Target:     "The quick brown fox jumps over the lazy dog by the river"
```

The model must figure out:
- **How many tokens** each `[MASK]` represents (unlike BERT where it's always 1:1)
- **Which tokens** were masked

---

## 3. Architecture Details

### 3.1 Model Architecture

BART uses a standard Transformer encoder-decoder:

```
┌──────────────────────────────────────────────────────────┐
│                    BART ARCHITECTURE                      │
│                                                          │
│  ENCODER (6 layers)           DECODER (6 layers)         │
│  ┌────────────────────┐       ┌────────────────────┐     │
│  │ Token Embeddings   │       │ Token Embeddings   │     │
│  │ + Position Embed   │       │ + Position Embed   │     │
│  ├────────────────────┤       ├────────────────────┤     │
│  │                    │       │                    │     │
│  │ Multi-Head         │       │ Masked Multi-Head  │     │
│  │ Self-Attention     │       │ Self-Attention     │     │
│  │ (bidirectional)    │       │ (causal)           │     │
│  │                    │       │                    │     │
│  │ Add & LayerNorm    │       │ Add & LayerNorm    │     │
│  │                    │       │                    │     │
│  │ Feed-Forward       │       │ Cross-Attention    │     │
│  │ Network            │       │ (attend to encoder)│     │
│  │                    │       │                    │     │
│  │ Add & LayerNorm    │       │ Add & LayerNorm    │     │
│  │                    │       │                    │     │
│  │      × 6           │       │ Feed-Forward       │     │
│  └────────────────────┘       │ Network            │     │
│                               │                    │     │
│                               │ Add & LayerNorm    │     │
│                               │                    │     │
│                               │      × 6           │     │
│                               └────────┬───────────┘     │
│                                        │                 │
│                                  ┌─────▼─────┐           │
│                                  │  Linear   │           │
│                                  │ + Softmax │           │
│                                  └───────────┘           │
└──────────────────────────────────────────────────────────┘
```

### 3.2 Configuration (BART-Large)

| Parameter               | Value       |
|--------------------------|-------------|
| Encoder layers           | 12          |
| Decoder layers           | 12          |
| Hidden size (d_model)    | 1024        |
| Attention heads          | 16          |
| Feed-forward size (d_ff) | 4096        |
| Parameters               | ~400M       |
| Vocabulary size          | 50,265      |
| Max position embeddings  | 1024        |
| Positional encoding      | Learned absolute |
| Activation               | GeLU        |

### 3.3 Attention Computation

Encoder self-attention (bidirectional):

```
Attn_enc(Q, K, V) = softmax(QK^T / √d_k) V
```

Decoder self-attention (causal mask applied):

```
Attn_dec(Q, K, V) = softmax(QK^T / √d_k + M_causal) V

where M_causal[i][j] = { 0     if j ≤ i
                        { -∞    if j > i
```

Cross-attention (decoder queries attend to encoder keys/values):

```
CrossAttn(Q_dec, K_enc, V_enc) = softmax(Q_dec × K_enc^T / √d_k) V_enc
```

---

## 4. Pre-Training and Fine-Tuning

### 4.1 Pre-Training Setup

- **Data**: Combination of books corpus and English Wikipedia (~160 GB)
- **Optimizer**: Adam (β₁=0.9, β₂=0.999, ε=1e-8)
- **Learning rate**: Peak 3×10⁻⁵ with linear warmup for 5% of steps
- **Batch size**: 8000 sequences
- **Training steps**: 500,000

### 4.2 Fine-Tuning for Different Tasks

```
┌─────────────────────────────────────────────────────────────────┐
│                    FINE-TUNING STRATEGIES                        │
├──────────────────┬──────────────────────────────────────────────┤
│ CLASSIFICATION   │ Feed same input to encoder & decoder.        │
│                  │ Use final decoder token representation       │
│                  │ with a linear classifier.                    │
│                  │                                              │
│ GENERATION       │ Standard seq2seq fine-tuning.                │
│ (Summarization,  │ Encoder reads input, decoder generates       │
│  Translation)    │ output autoregressively.                     │
│                  │                                              │
│ EXTRACTION       │ Feed input through encoder. Use token        │
│ (Span QA)        │ representations to predict start/end.        │
└──────────────────┴──────────────────────────────────────────────┘
```

---

## 5. Comparison: BART vs BERT vs GPT

```
┌──────────────┬──────────────────┬──────────────────┬──────────────────┐
│              │      BERT        │      GPT         │      BART        │
├──────────────┼──────────────────┼──────────────────┼──────────────────┤
│ Architecture │ Encoder-only     │ Decoder-only     │ Encoder-Decoder  │
│ Attention    │ Bidirectional    │ Unidirectional   │ Bi (enc) + Uni   │
│ Pre-training │ MLM + NSP       │ Language Model   │ Denoising AE     │
│ Generation   │ Poor             │ Excellent        │ Excellent        │
│ Understanding│ Excellent        │ Good             │ Excellent        │
│ Best tasks   │ Classification,  │ Text generation, │ Summarization,   │
│              │ NER, QA          │ creative writing │ translation, QA  │
│ Parameters   │ 110M / 340M     │ 117M / 1.5B     │ 140M / 400M      │
└──────────────┴──────────────────┴──────────────────┴──────────────────┘
```

---

## 6. Worked Example: Summarization with BART

### Input (Corrupted during pre-training)

```
Original:  "Scientists discovered a new species of deep-sea fish in the
            Pacific Ocean. The fish has bioluminescent features."

Corrupted: "Scientists [MASK] deep-sea fish in the [MASK]. The fish [MASK]
            bioluminescent features."
```

### Pre-training Target

```
The model learns to reconstruct the full original text, teaching it to:
  - Understand sentence structure
  - Infer missing information from context
  - Generate fluent text
```

### Fine-tuning for Summarization

```
Input:   "Scientists discovered a new species of deep-sea fish in the
          Pacific Ocean. The fish has bioluminescent features that allow
          it to attract prey in dark waters at depths exceeding 3000m."

Target:  "New bioluminescent deep-sea fish species found in Pacific Ocean."

Decoder generation (autoregressive):
  Step 0: <s>             → "New"        (P=0.72)
  Step 1: <s> New         → "bioluminescent" (P=0.65)
  Step 2: ... New biolum. → "deep"       (P=0.81)
  ...
  Step N:                 → </s>         (P=0.95)
```

---

## 7. Code: Using BART with HuggingFace

### 7.1 Summarization

```python
from transformers import BartForConditionalGeneration, BartTokenizer

model_name = "facebook/bart-large-cnn"
tokenizer = BartTokenizer.from_pretrained(model_name)
model = BartForConditionalGeneration.from_pretrained(model_name)

article = """
The Amazon rainforest, often referred to as the lungs of the Earth,
produces approximately 20 percent of the world's oxygen. Spanning across
nine countries in South America, it is home to an estimated 10 percent
of all species on Earth. However, deforestation has accelerated in recent
years, with satellite data showing significant loss of tree cover.
Environmental groups are calling for stronger international cooperation
to protect this vital ecosystem.
"""

inputs = tokenizer(article.strip(), return_tensors="pt", max_length=1024,
                   truncation=True)
summary_ids = model.generate(
    inputs["input_ids"],
    max_length=80,
    min_length=20,
    num_beams=4,
    length_penalty=2.0,
    early_stopping=True,
)
summary = tokenizer.decode(summary_ids[0], skip_special_tokens=True)
print("Summary:", summary)
```

### 7.2 Simulating BART's Corruption Strategies

```python
import random
import numpy as np

def token_masking(tokens, mask_prob=0.3):
    """Strategy 1: Replace random tokens with [MASK]."""
    result = tokens.copy()
    for i in range(len(result)):
        if random.random() < mask_prob:
            result[i] = "[MASK]"
    return result

def token_deletion(tokens, del_prob=0.3):
    """Strategy 2: Delete random tokens entirely."""
    return [t for t in tokens if random.random() >= del_prob]

def text_infilling(tokens, num_spans=2, lam=3):
    """Strategy 3: Replace random spans with single [MASK]."""
    result = tokens.copy()
    n = len(result)
    for _ in range(num_spans):
        span_len = max(0, np.random.poisson(lam))
        if span_len == 0:
            pos = random.randint(0, len(result))
            result.insert(pos, "[MASK]")
        else:
            start = random.randint(0, max(0, len(result) - span_len))
            end = min(start + span_len, len(result))
            result[start:end] = ["[MASK]"]
    return result

def sentence_permutation(text):
    """Strategy 4: Shuffle sentences."""
    sentences = [s.strip() for s in text.split('.') if s.strip()]
    random.shuffle(sentences)
    return '. '.join(sentences) + '.'

def document_rotation(tokens):
    """Strategy 5: Rotate document at random pivot."""
    pivot = random.randint(1, len(tokens) - 1)
    return tokens[pivot:] + tokens[:pivot]

# Demonstration
text = "The quick brown fox jumps over the lazy dog"
tokens = text.split()

print("Original:     ", text)
print("Token Masking:", ' '.join(token_masking(tokens)))
print("Token Deletion:", ' '.join(token_deletion(tokens)))
print("Text Infilling:", ' '.join(text_infilling(tokens)))
print("Sent Permute: ", sentence_permutation(
    "First sentence. Second sentence. Third sentence"))
print("Doc Rotation: ", ' '.join(document_rotation(tokens)))
```

### 7.3 Fine-Tuning BART for Custom Summarization

```python
import torch
from transformers import BartForConditionalGeneration, BartTokenizer

tokenizer = BartTokenizer.from_pretrained("facebook/bart-base")
model = BartForConditionalGeneration.from_pretrained("facebook/bart-base")

# Example training data
train_data = [
    {
        "input": "The stock market rose sharply today after positive jobs data.",
        "target": "Stock market rises on jobs data."
    },
    {
        "input": "Heavy rainfall caused flooding in several coastal cities.",
        "target": "Flooding hits coastal cities."
    },
]

optimizer = torch.optim.AdamW(model.parameters(), lr=3e-5)
model.train()

for epoch in range(3):
    total_loss = 0
    for item in train_data:
        inputs = tokenizer(item["input"], return_tensors="pt",
                           max_length=128, truncation=True, padding="max_length")
        targets = tokenizer(item["target"], return_tensors="pt",
                            max_length=64, truncation=True, padding="max_length")

        labels = targets.input_ids.clone()
        labels[labels == tokenizer.pad_token_id] = -100

        outputs = model(
            input_ids=inputs.input_ids,
            attention_mask=inputs.attention_mask,
            labels=labels,
        )
        loss = outputs.loss
        loss.backward()
        optimizer.step()
        optimizer.zero_grad()
        total_loss += loss.item()

    print(f"Epoch {epoch + 1}, Avg Loss: {total_loss / len(train_data):.4f}")
```

---

## 8. Real-World Applications

| Application             | Why BART Excels                                 | Example Use Case                      |
|--------------------------|------------------------------------------------|---------------------------------------|
| News Summarization       | Strong abstractive generation                  | CNN/DailyMail summarization           |
| Machine Translation      | Encoder-decoder naturally fits seq2seq         | mBART for multilingual translation    |
| Question Answering       | Deep context understanding + generation        | Generative QA systems                 |
| Dialogue Systems         | Fluent response generation                     | Chatbots, virtual assistants          |
| Text Correction          | Denoising training teaches error correction    | Grammar and spelling correction       |
| Data Augmentation        | Generate paraphrases of training data          | Expanding limited training datasets   |

---

## 9. Summary Table

| Aspect                  | Detail                                             |
|-------------------------|----------------------------------------------------|
| **Paper**               | Lewis et al. (2019), Facebook AI                   |
| **Core Idea**           | Denoising autoencoder (corrupt → reconstruct)      |
| **Architecture**        | Standard Transformer encoder-decoder               |
| **Best Corruption**     | Text infilling (Poisson λ=3 span lengths)          |
| **Positional Encoding** | Learned absolute embeddings                        |
| **Pre-training Data**   | Books corpus + English Wikipedia (~160 GB)          |
| **Model Sizes**         | BART-Base (~140M), BART-Large (~400M)              |
| **Optimizer**           | Adam                                               |
| **Tokenizer**           | Byte-Pair Encoding (BPE), 50,265 vocab             |
| **Key Strength**        | Combines BERT's understanding + GPT's generation   |
| **Loss**                | Cross-entropy on reconstructed original tokens     |

---

## 10. Revision Questions

1. **Explain BART's denoising autoencoder approach.** How does it differ from BERT's masked language modeling? What advantage does full sequence reconstruction provide?

2. **Compare and contrast all five corruption strategies.** Why does text infilling outperform the others? What makes token deletion harder than token masking?

3. **How does BART combine the strengths of BERT and GPT?** Draw the architecture and explain the role of bidirectional encoding vs. autoregressive decoding.

4. **Describe the Poisson distribution used in text infilling.** With λ = 3, what is the probability of a span of length 0? Why is zero-length span corruption useful?

5. **Walk through a fine-tuning example for summarization.** How does the pre-training objective (denoising) transfer to the summarization task? What changes between pre-training and fine-tuning?

6. **Compare BART-Large with T5-Base on summarization benchmarks.** What architectural differences (relative vs. absolute position, pre-norm vs. post-norm) might explain performance gaps?

---

[← T5 Architecture](01-t5-architecture.md) | [Seq2Seq Tasks →](03-seq2seq-tasks.md)
