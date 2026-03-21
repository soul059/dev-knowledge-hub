[← Fine-tuning BERT](03-fine-tuning-bert.md) | [BERT Applications →](05-bert-applications.md)

---

# BERT Variants: RoBERTa, ALBERT, DistilBERT

Since the original BERT paper, researchers have proposed numerous improvements addressing its limitations in training efficiency, model size, and performance. This chapter covers the four most influential BERT variants: **RoBERTa** (optimized training), **ALBERT** (parameter efficiency), **DistilBERT** (knowledge distillation for speed), and **ELECTRA** (sample-efficient pretraining). Each variant modifies a different aspect of BERT's design — training procedure, architecture, or pretraining objective — while preserving the core encoder-only Transformer foundation. Understanding these variants is essential for selecting the right model for production deployment, where trade-offs between accuracy, speed, and memory are critical.

---

## 1. RoBERTa: A Robustly Optimized BERT

### Key Changes from BERT

RoBERTa (Liu et al., 2019) keeps BERT's architecture **identical** but shows that BERT was significantly undertrained. The improvements are purely in training methodology:

```
  ┌─────────────────────────────────────────────────────────────┐
  │                 RoBERTa vs BERT                             │
  │                                                             │
  │  Change 1: Remove NSP objective entirely                    │
  │    BERT:    L = L_MLM + L_NSP                               │
  │    RoBERTa: L = L_MLM only                                  │
  │                                                             │
  │  Change 2: Dynamic masking                                  │
  │    BERT:    Same mask pattern every epoch (static)           │
  │    RoBERTa: New mask pattern generated each epoch (dynamic)  │
  │                                                             │
  │  Change 3: More training data                               │
  │    BERT:    16GB (BooksCorpus + Wikipedia)                   │
  │    RoBERTa: 160GB (+ CC-News, OpenWebText, Stories)         │
  │                                                             │
  │  Change 4: Larger batch sizes                               │
  │    BERT:    256 sequences                                    │
  │    RoBERTa: 8,192 sequences                                  │
  │                                                             │
  │  Change 5: Longer training                                  │
  │    BERT:    1M steps                                         │
  │    RoBERTa: 500K steps (but with 32× larger batches)        │
  │                                                             │
  │  Change 6: No sentence pairs                                │
  │    BERT:    Input = Sentence A [SEP] Sentence B             │
  │    RoBERTa: Input = Full documents packed to 512 tokens     │
  │                                                             │
  └─────────────────────────────────────────────────────────────┘
```

### Dynamic vs. Static Masking

```
Static Masking (BERT):
  Training data is masked ONCE during preprocessing.
  Epoch 1: "The [MASK] sat on the mat"
  Epoch 2: "The [MASK] sat on the mat"    ← same mask
  Epoch 3: "The [MASK] sat on the mat"    ← same mask

Dynamic Masking (RoBERTa):
  Mask pattern regenerated every time a sequence is fed to the model.
  Epoch 1: "The [MASK] sat on the mat"
  Epoch 2: "The cat [MASK] on the mat"    ← different mask
  Epoch 3: "The cat sat on [MASK] mat"    ← different mask

  Result: Model sees each token in different contexts → better generalization
```

### Why Remove NSP?

```
RoBERTa's ablation study showed:

  Model                           MNLI    SST-2   QNLI    SQuAD
  ────────────────────────────────────────────────────────────────
  BERT (with NSP)                 84.6    92.7    90.5    88.5
  BERT (without NSP)              84.8    92.9    90.8    89.1
  RoBERTa (full documents)        87.6    94.8    92.9    90.4

  NSP provides no benefit (sometimes hurts) because:
  1. Topic prediction is too easy (different documents = different topics)
  2. Full document sequences capture longer coherence naturally
```

---

## 2. ALBERT: A Lite BERT

### Key Innovation: Parameter Reduction

ALBERT (Lan et al., 2020) dramatically reduces parameters while maintaining (and even improving) performance through two techniques:

### Factorized Embedding Parameterization

```
  BERT Embedding:
  ────────────────
  Token → Embedding matrix (V × H)
  V = 30,000, H = 768
  Parameters = V × H = 30,000 × 768 = 23.04M

  ALBERT Factorized Embedding:
  ────────────────────────────
  Token → Small embedding (V × E) → Projection (E × H)
  V = 30,000, E = 128, H = 768
  Parameters = V × E + E × H = 30,000 × 128 + 128 × 768 = 3.94M

  Savings: 23.04M → 3.94M = 83% reduction in embedding parameters!
```

### Mathematical Justification

```
  In BERT, token embeddings live in the same space as hidden states (both ℝ^H).
  But token embeddings are context-independent while hidden states are contextual.

  The embedding space doesn't NEED to be as large as the hidden space.
  Using E << H is more parameter-efficient:

  Original:    V × H         parameters
  Factorized:  V × E + E × H parameters

  When E << H:
    V × E + E × H  <<  V × H
    30000×128 + 128×768 = 3,938,304  <<  30000×768 = 23,040,000
```

### Cross-Layer Parameter Sharing

```
  BERT:  Each of 12 layers has its own parameters
  ┌────────────┐ ┌────────────┐ ┌────────────┐     ┌─────────────┐
  │  Layer 1   │ │  Layer 2   │ │  Layer 3   │ ... │  Layer 12   │
  │  Params A  │ │  Params B  │ │  Params C  │     │  Params L   │
  └────────────┘ └────────────┘ └────────────┘     └─────────────┘
  Total: 12 × 7.08M = 85M layer parameters

  ALBERT: ALL layers share the same parameters
  ┌────────────┐ ┌────────────┐ ┌────────────┐     ┌─────────────┐
  │  Layer 1   │ │  Layer 2   │ │  Layer 3   │ ... │  Layer 12   │
  │  Params A  │ │  Params A  │ │  Params A  │     │  Params A   │
  └────────────┘ └────────────┘ └────────────┘     └─────────────┘
  Total: 1 × 7.08M = 7.08M layer parameters (12× reduction!)

  Sharing options tested:
    - All shared (attention + FFN)  ← default ALBERT
    - Only attention shared
    - Only FFN shared
```

### ALBERT Configurations

| Config       | Layers | Hidden | Embed (E) | Heads | Parameters |
|--------------|--------|--------|-----------|-------|------------|
| ALBERT-base  | 12     | 768    | 128       | 12    | 12M        |
| ALBERT-large | 24     | 1024   | 128       | 16    | 18M        |
| ALBERT-xlarge| 24     | 2048   | 128       | 16    | 60M        |
| ALBERT-xxlarge| 12    | 4096   | 128       | 64    | 235M       |

### SOP vs. NSP

ALBERT replaces NSP with **Sentence Order Prediction (SOP)**:

```
  NSP (BERT):
    Positive: (sentence A, sentence B from same document) → IsNext
    Negative: (sentence A, random sentence from different document) → NotNext
    Problem: Model can cheat by detecting topic mismatch

  SOP (ALBERT):
    Positive: (sentence A, sentence B) in correct order → Correct
    Negative: (sentence B, sentence A) in swapped order → Swapped
    Advantage: Both from same document → model must learn actual coherence
```

---

## 3. DistilBERT: Distilled BERT

### Knowledge Distillation

DistilBERT (Sanh et al., 2019) compresses BERT using **knowledge distillation** — training a smaller "student" model to mimic a larger "teacher" model.

```
  ┌────────────────────────────────────────────────────────────┐
  │              KNOWLEDGE DISTILLATION                        │
  │                                                            │
  │  Teacher (BERT-base): 12 layers, 110M params               │
  │  Student (DistilBERT): 6 layers, 66M params                │
  │                                                            │
  │       Input tokens                                         │
  │         ↓         ↓                                        │
  │  ┌──────────┐  ┌──────────┐                                │
  │  │ Teacher  │  │ Student  │                                │
  │  │ (frozen) │  │(training)│                                │
  │  │ 12 layers│  │ 6 layers │                                │
  │  └────┬─────┘  └────┬─────┘                                │
  │       ↓              ↓                                     │
  │  soft labels    student logits                             │
  │  (teacher's     (student's                                 │
  │   predictions)   predictions)                              │
  │       ↓              ↓                                     │
  │  ┌──────────────────────────────┐                          │
  │  │ Distillation Loss:           │                          │
  │  │ L = α·L_distill + β·L_mlm + γ·L_cos                   │
  │  └──────────────────────────────┘                          │
  └────────────────────────────────────────────────────────────┘
```

### DistilBERT Loss Function

```
L_total = α · L_distill + β · L_mlm + γ · L_cosine

Where:
  L_distill = KL(softmax(z_t/T) || softmax(z_s/T))
    - z_t = teacher logits, z_s = student logits
    - T = temperature (softens probability distributions)
    - Higher T → softer distributions → more knowledge transfer

  L_mlm = standard masked language model loss

  L_cosine = 1 - cos(h_t, h_s)
    - Aligns student and teacher hidden state directions
    - h_t = teacher hidden states, h_s = student hidden states

Temperature scaling:
  softmax(z/T)_i = exp(z_i / T) / Σⱼ exp(z_j / T)

  T = 1: standard softmax (peaked)
  T > 1: softer distribution (reveals "dark knowledge")
  Example: T = 1 → [0.95, 0.03, 0.02]
           T = 5 → [0.45, 0.28, 0.27]  ← reveals teacher's uncertainty
```

### DistilBERT Architecture Choices

```
  Which layers to keep from the teacher?

  Strategy: Take every other layer from BERT-base
  
  BERT:       Layer 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12
  DistilBERT: Layer 1,    3,    5,    7,    9,     11
                ↓         ↓     ↓     ↓     ↓       ↓
              Student layers 1, 2,   3,    4,    5,      6

  - Remove token type embeddings (no segment IDs)
  - Remove the pooler layer
  - Initialize student weights from selected teacher layers
```

### DistilBERT Performance

```
  DistilBERT achieves:
    - 97% of BERT's performance on GLUE benchmark
    - 60% of BERT's size (66M vs 110M parameters)
    - 60% faster inference
    - Suitable for mobile and edge deployment
```

---

## 4. ELECTRA: Efficiently Learning an Encoder

### Replaced Token Detection

ELECTRA (Clark et al., 2020) replaces MLM with a more sample-efficient objective:

```
  ┌──────────────────────────────────────────────────────────────┐
  │                    ELECTRA Architecture                      │
  │                                                              │
  │  Input: "the chef cooked the meal"                           │
  │                                                              │
  │  Step 1: Small Generator (like MLM) replaces some tokens     │
  │    Masked:    "the [MASK] cooked the [MASK]"                 │
  │    Generated: "the artist cooked the soup"                   │
  │               (generator picks plausible replacements)       │
  │                                                              │
  │  Step 2: Discriminator classifies EVERY token as             │
  │          "original" or "replaced"                            │
  │                                                              │
  │    Input to discriminator: "the artist cooked the soup"      │
  │    Labels:                  orig replaced orig  orig replaced│
  │                                                              │
  │  Key advantage: ALL tokens provide training signal           │
  │  (vs. BERT's MLM where only 15% of masked tokens do)        │
  └──────────────────────────────────────────────────────────────┘
```

### ELECTRA Architecture Diagram

```
  Original input:     "the chef cooked the meal"
                           ↓
  ┌──────────────────────────────────────────┐
  │  Mask 15%:  "the [M] cooked the [M]"    │
  └──────────────────────────────────────────┘
                           ↓
  ┌──────────────────────────────────────────┐
  │  Generator (small Transformer)           │
  │  Predicts: [M]→"artist", [M]→"soup"     │
  └──────────────────────────────────────────┘
                           ↓
  Corrupted:     "the artist cooked the soup"
                           ↓
  ┌──────────────────────────────────────────┐
  │  Discriminator (full-size Transformer)   │
  │  Binary classification per token:        │
  │  original or replaced?                   │
  └──────────────────────────────────────────┘
                           ↓
  Predictions:    orig  repl   orig  orig  repl
  Ground truth:   orig  repl   orig  orig  repl

  After pretraining: DISCARD generator, KEEP discriminator
  The discriminator IS the pretrained encoder.
```

### ELECTRA Loss

```
  L_ELECTRA = L_generator + λ · L_discriminator

  L_generator = standard MLM loss (same as BERT's)
  L_discriminator = Σᵢ BCE(D(x_i), is_original_i)

  where:
    D(x_i) = sigmoid(w · h_i) = probability token i is original
    BCE = Binary Cross-Entropy loss
    λ = 50 (discriminator loss weighted higher)

  Sample efficiency:
    BERT:    15% of tokens → training signal per sample
    ELECTRA: 100% of tokens → training signal per sample
             → ~4× more efficient than BERT
```

---

## 5. Comprehensive Comparison Table

| Feature                | BERT-base    | RoBERTa       | ALBERT-base  | DistilBERT   | ELECTRA-base |
|------------------------|-------------|---------------|-------------|-------------|-------------|
| **Parameters**         | 110M        | 125M          | 12M         | 66M         | 110M        |
| **Layers**             | 12          | 12            | 12          | 6           | 12          |
| **Hidden size**        | 768         | 768           | 768         | 768         | 256         |
| **Pretraining obj.**   | MLM + NSP   | MLM only      | MLM + SOP   | Distillation| RTD         |
| **Training data**      | 16GB        | 160GB         | 16GB        | 16GB        | 16GB        |
| **Masking**            | Static      | Dynamic       | Static      | N/A         | Generator   |
| **GLUE score**         | 79.6        | 88.5          | 82.3        | 77.0        | 85.1        |
| **Inference speed**    | 1×          | 1×            | 1×*         | 1.6×        | 1×          |
| **Memory**             | 1×          | 1×            | 0.1×        | 0.6×        | 1×          |
| **Key advantage**      | Pioneering  | Better training| Tiny model | Fast & small| Efficient   |

*ALBERT has same inference cost as BERT (same depth) despite fewer parameters.

---

## 6. When to Use Which Model

```
  Decision Tree for Model Selection
  ══════════════════════════════════

  Need maximum accuracy?
  ├── Yes → RoBERTa (best general performance)
  └── No
      ├── Need fast inference / mobile deployment?
      │   ├── Yes → DistilBERT (60% faster, 97% accuracy)
      │   └── No
      │       ├── Limited GPU memory / many models?
      │       │   ├── Yes → ALBERT (12M params, shared layers)
      │       │   └── No
      │       │       ├── Limited pretraining compute?
      │       │       │   ├── Yes → ELECTRA (4× sample efficient)
      │       │       │   └── No → RoBERTa or BERT-large
      │       │       └──
      │       └──
      └──

  Production Checklist:
  ┌───────────────────────────────────────────────────────────┐
  │  ☐ Latency requirements  → DistilBERT if < 10ms needed  │
  │  ☐ Model size budget     → ALBERT if < 50MB needed      │
  │  ☐ Accuracy requirements → RoBERTa if SOTA needed       │
  │  ☐ Training compute      → ELECTRA if limited GPU time  │
  │  ☐ Domain specificity    → Continue pretraining any base │
  └───────────────────────────────────────────────────────────┘
```

---

## 7. Code: Using Each Variant

### RoBERTa

```python
from transformers import RobertaTokenizer, RobertaForSequenceClassification
import torch

tokenizer = RobertaTokenizer.from_pretrained("roberta-base")
model = RobertaForSequenceClassification.from_pretrained("roberta-base", num_labels=2)

text = "RoBERTa improves upon BERT with better training."
inputs = tokenizer(text, return_tensors="pt", padding=True, truncation=True)

with torch.no_grad():
    outputs = model(**inputs)
    probs = torch.softmax(outputs.logits, dim=-1)
    print(f"Predictions: {probs}")

# Key difference: RoBERTa uses BytePair Encoding (BPE), not WordPiece
tokens = tokenizer.tokenize("unbelievable")
print(f"RoBERTa tokens: {tokens}")  # Uses Ġ prefix for word boundaries
```

### ALBERT

```python
from transformers import AlbertTokenizer, AlbertForSequenceClassification
import torch

tokenizer = AlbertTokenizer.from_pretrained("albert-base-v2")
model = AlbertForSequenceClassification.from_pretrained("albert-base-v2", num_labels=2)

text = "ALBERT uses factorized embeddings and cross-layer sharing."
inputs = tokenizer(text, return_tensors="pt", padding=True, truncation=True)

with torch.no_grad():
    outputs = model(**inputs)
    probs = torch.softmax(outputs.logits, dim=-1)
    print(f"Predictions: {probs}")

# Verify parameter sharing
params = dict(model.named_parameters())
print(f"\nTotal parameters: {sum(p.numel() for p in model.parameters()):,}")
# Verify that ALBERT has drastically fewer unique parameters
# Even though it has 12 layers, parameters are shared across all layers
```

### DistilBERT

```python
from transformers import DistilBertTokenizer, DistilBertForSequenceClassification
import torch

tokenizer = DistilBertTokenizer.from_pretrained("distilbert-base-uncased")
model = DistilBertForSequenceClassification.from_pretrained(
    "distilbert-base-uncased", num_labels=2
)

text = "DistilBERT is smaller and faster than BERT."
inputs = tokenizer(text, return_tensors="pt", padding=True, truncation=True)

with torch.no_grad():
    outputs = model(**inputs)
    probs = torch.softmax(outputs.logits, dim=-1)
    print(f"Predictions: {probs}")

total = sum(p.numel() for p in model.parameters())
print(f"Parameters: {total:,}")  # ~66M (vs BERT's 110M)
```

### ELECTRA

```python
from transformers import ElectraTokenizer, ElectraForSequenceClassification
import torch

tokenizer = ElectraTokenizer.from_pretrained("google/electra-base-discriminator")
model = ElectraForSequenceClassification.from_pretrained(
    "google/electra-base-discriminator", num_labels=2
)

text = "ELECTRA uses replaced token detection for efficient pretraining."
inputs = tokenizer(text, return_tensors="pt", padding=True, truncation=True)

with torch.no_grad():
    outputs = model(**inputs)
    probs = torch.softmax(outputs.logits, dim=-1)
    print(f"Predictions: {probs}")

total = sum(p.numel() for p in model.parameters())
print(f"Parameters: {total:,}")
```

### Speed Comparison Benchmark

```python
import time
import torch
from transformers import (
    BertTokenizer, BertModel,
    DistilBertTokenizer, DistilBertModel,
    AlbertTokenizer, AlbertModel,
)

models_config = {
    "BERT-base": ("bert-base-uncased", BertTokenizer, BertModel),
    "DistilBERT": ("distilbert-base-uncased", DistilBertTokenizer, DistilBertModel),
    "ALBERT-base": ("albert-base-v2", AlbertTokenizer, AlbertModel),
}

text = "Comparing inference speed across BERT variants for benchmarking."
num_runs = 100

for name, (model_name, tok_cls, model_cls) in models_config.items():
    tokenizer = tok_cls.from_pretrained(model_name)
    model = model_cls.from_pretrained(model_name)
    model.eval()

    inputs = tokenizer(text, return_tensors="pt", padding=True, truncation=True)
    params = sum(p.numel() for p in model.parameters())

    # Warmup
    with torch.no_grad():
        for _ in range(10):
            model(**inputs)

    # Benchmark
    start = time.time()
    with torch.no_grad():
        for _ in range(num_runs):
            model(**inputs)
    elapsed = (time.time() - start) / num_runs * 1000

    print(f"{name:15s} | Params: {params/1e6:.1f}M | Latency: {elapsed:.1f}ms")
```

---

## 8. Real-World Applications

| Application              | Best Variant    | Reason                                       |
|--------------------------|-----------------|----------------------------------------------|
| Research / benchmarks    | RoBERTa         | Best accuracy on standard benchmarks          |
| Mobile apps              | DistilBERT      | Small size, fast inference                    |
| Edge / IoT devices       | ALBERT-base     | Smallest parameter count (12M)                |
| Limited training budget  | ELECTRA-small   | 4× more sample-efficient pretraining          |
| Domain adaptation        | RoBERTa         | Strong base for continued pretraining         |
| Real-time APIs           | DistilBERT      | Low latency for high-throughput serving       |
| Multi-task serving       | ALBERT          | Multiple models fit in memory simultaneously  |

---

## 9. Summary Table

| Concept                   | Key Details                                               |
|---------------------------|-----------------------------------------------------------|
| RoBERTa changes           | No NSP, dynamic masking, 10× data, larger batches        |
| ALBERT embedding trick    | V×E + E×H instead of V×H (83% embedding reduction)       |
| ALBERT layer sharing      | All 12 layers share same parameters (12× reduction)       |
| ALBERT sentence task      | SOP (sentence order) instead of NSP                       |
| DistilBERT approach       | Knowledge distillation: student mimics teacher            |
| DistilBERT result         | 60% size, 60% faster, 97% of BERT performance            |
| DistilBERT loss           | L_distill + L_mlm + L_cosine with temperature scaling    |
| ELECTRA objective         | Replaced Token Detection (discriminator task)             |
| ELECTRA efficiency        | 100% of tokens provide training signal (vs BERT's 15%)   |
| Model selection           | Accuracy→RoBERTa, Speed→DistilBERT, Size→ALBERT          |

---

## 10. Revision Questions

1. **List the key training differences between BERT and RoBERTa. Why does removing NSP and using dynamic masking lead to better performance?**

2. **Explain ALBERT's factorized embedding parameterization. Calculate the parameter savings for a vocabulary of 30,000 with E=128 and H=1024.**

3. **How does cross-layer parameter sharing in ALBERT affect model size vs. inference speed? Why doesn't it reduce inference time?**

4. **Describe the knowledge distillation process in DistilBERT. What is the role of temperature in the distillation loss, and why is it important?**

5. **How does ELECTRA's replaced token detection differ from BERT's masked language modeling? Why is ELECTRA more sample-efficient?**

6. **You need to deploy a text classifier on a mobile device with strict latency and size constraints. Which BERT variant would you choose and why? What trade-offs are involved?**

---

[← Fine-tuning BERT](03-fine-tuning-bert.md) | [BERT Applications →](05-bert-applications.md)
