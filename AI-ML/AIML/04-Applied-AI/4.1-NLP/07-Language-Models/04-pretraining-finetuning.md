# Chapter 4: Pre-training and Fine-tuning Paradigm

> **Unit 4.1 — NLP / 07 – Language Models** | [← Previous: Neural Language Models](./03-neural-language-models.md) | [Next → Transfer Learning](./05-transfer-learning.md)

---

## 1. Overview

The **pre-training and fine-tuning paradigm** represents the most significant shift in NLP methodology since the introduction of neural networks. Before 2018, NLP models were typically trained **from scratch** on each task — a costly process requiring large labeled datasets and task-specific architectures. The new paradigm splits learning into two stages: (1) **pre-training** a large model on massive unlabeled text to learn universal language representations, and (2) **fine-tuning** the pre-trained model on a small task-specific labeled dataset. This approach — pioneered by models like **ELMo** (Peters et al., 2018), **ULMFiT** (Howard & Ruder, 2018), **GPT** (Radford et al., 2018), and **BERT** (Devlin et al., 2019) — democratized high-quality NLP by allowing practitioners to achieve state-of-the-art results with minimal labeled data and modest compute. It mirrors the success of **ImageNet pre-training** in computer vision and has become the standard recipe for virtually all modern NLP systems.

---

## 2. The Paradigm Shift

### 2.1 From Training from Scratch to Pre-train + Fine-tune

The traditional approach required building a task-specific model from the ground up for each new NLP task. This had several critical limitations:

1. **Data hunger:** Supervised models needed thousands to millions of labeled examples per task.
2. **No knowledge sharing:** A sentiment classifier learned nothing that could help a named-entity recognizer.
3. **Expensive:** Each task required full training runs, architecture search, and hyperparameter tuning.
4. **Poor low-resource performance:** Tasks with limited labeled data produced weak models.

The pre-train + fine-tune paradigm solves these problems by **decoupling representation learning from task-specific learning:**

```
 ┌─────────────────────────────────────────────────────────────────────┐
 │                    OLD PARADIGM: Train from Scratch                 │
 │                                                                     │
 │   Task A Data ──▶ [ Random Init Model A ] ──▶ Sentiment Classifier │
 │   Task B Data ──▶ [ Random Init Model B ] ──▶ NER Tagger           │
 │   Task C Data ──▶ [ Random Init Model C ] ──▶ QA System            │
 │                                                                     │
 │   ✗ No knowledge sharing    ✗ Needs lots of labeled data each time │
 └─────────────────────────────────────────────────────────────────────┘

                              vs.

 ┌─────────────────────────────────────────────────────────────────────┐
 │                 NEW PARADIGM: Pre-train + Fine-tune                 │
 │                                                                     │
 │   Unlabeled         ┌────────────────────┐                          │
 │   Corpus    ──────▶ │   PRE-TRAINING     │                          │
 │   (GBs of text)     │   (Self-supervised) │                         │
 │                      └────────┬───────────┘                         │
 │                               │                                     │
 │                    Pre-trained Language Model                        │
 │                               │                                     │
 │              ┌────────────────┼────────────────┐                    │
 │              ▼                ▼                ▼                    │
 │   ┌──────────────┐ ┌──────────────┐ ┌──────────────┐              │
 │   │  FINE-TUNE   │ │  FINE-TUNE   │ │  FINE-TUNE   │              │
 │   │  Task A      │ │  Task B      │ │  Task C      │              │
 │   │  (small data)│ │  (small data)│ │  (small data)│              │
 │   └──────┬───────┘ └──────┬───────┘ └──────┬───────┘              │
 │          ▼                ▼                ▼                        │
 │   Sentiment         NER Tagger       QA System                      │
 │                                                                     │
 │   ✓ Shared knowledge    ✓ Works with little labeled data           │
 └─────────────────────────────────────────────────────────────────────┘
```

### 2.2 Why Pre-training Works

Pre-training on large text corpora forces the model to learn **universal language representations** that capture:

- **Syntax:** Part-of-speech, phrase structure, agreement patterns
- **Semantics:** Word meaning, similarity, analogy relationships
- **Pragmatics:** Discourse coherence, coreference, sentiment cues
- **World knowledge:** Factual associations encoded implicitly in parameters

> **Key Insight:** Language modeling is an ideal pre-training objective because predicting the next word (or a masked word) requires understanding grammar, meaning, facts, and reasoning — all without any human annotation.

```
 What Pre-training Learns (Layer by Layer)
 ══════════════════════════════════════════

 Layer 12 (top)     │  Task-specific features, complex reasoning
 Layer 11           │  Semantic roles, coreference
 Layer 10           │  ↑ More abstract / task-specific
 Layer 9            │  Named entity types, sentiment
 Layer 8            │  Semantic similarity
 Layer 7            │  ───────────────────────────
 Layer 6            │  Long-range dependencies
 Layer 5            │  Syntactic structure (parse trees)
 Layer 4            │  ↓ More general / transferable
 Layer 3            │  Part-of-speech, morphology
 Layer 2            │  Local syntax patterns
 Layer 1 (bottom)   │  Surface features, character n-grams
```

---

## 3. ELMo — Embeddings from Language Models (Peters et al., 2018)

**ELMo** was the first major model to demonstrate that **contextualized word embeddings** — where a word's representation changes based on its surrounding context — dramatically outperform static embeddings (Word2Vec, GloVe) across NLP tasks.

### 3.1 Architecture

ELMo uses a **two-layer bidirectional LSTM** trained as a language model:

```
                        ELMo Architecture
 ═══════════════════════════════════════════════════════

 Input sentence:   "The   bank   refused   the   loan"

                    ┌─────┐ ┌─────┐ ┌─────┐ ┌─────┐ ┌─────┐
 Token Embeddings:  │ The │ │bank │ │refus│ │ the │ │loan │
 (char-CNN)         └──┬──┘ └──┬──┘ └──┬──┘ └──┬──┘ └──┬──┘
                       │      │      │      │      │
          ┌────────────┼──────┼──────┼──────┼──────┼────────────┐
          │            ▼      ▼      ▼      ▼      ▼            │
          │  Forward  ┌──┐  ┌──┐  ┌──┐  ┌──┐  ┌──┐             │
          │  LSTM L1  │h→│──│h→│──│h→│──│h→│──│h→│  ──────▶    │
          │           └──┘  └──┘  └──┘  └──┘  └──┘             │
          │                                                     │
          │  Backward ┌──┐  ┌──┐  ┌──┐  ┌──┐  ┌──┐             │
          │  LSTM L1  │←h│──│←h│──│←h│──│←h│──│←h│  ◀──────    │
          │           └──┘  └──┘  └──┘  └──┘  └──┘             │
          ├─────────────────────────────────────────────────────┤
          │  Forward  ┌──┐  ┌──┐  ┌──┐  ┌──┐  ┌──┐             │
          │  LSTM L2  │h→│──│h→│──│h→│──│h→│──│h→│  ──────▶    │
          │           └──┘  └──┘  └──┘  └──┘  └──┘             │
          │                                                     │
          │  Backward ┌──┐  ┌──┐  ┌──┐  ┌──┐  ┌──┐             │
          │  LSTM L2  │←h│──│←h│──│←h│──│←h│──│←h│  ◀──────    │
          │           └──┘  └──┘  └──┘  └──┘  └──┘             │
          └────────────────────────┬────────────────────────────┘
                                   │
                                   ▼
                    ELMo representation for each token:
                    ELMoₖ = γ · Σⱼ sⱼ · hₖⱼ

                    where sⱼ = softmax weights (task-learned)
                          hₖⱼ = hidden state at layer j for token k
                          γ   = task-specific scaling factor
```

### 3.2 Key Properties

| Property | Detail |
|----------|--------|
| **Contextualized** | "bank" gets different vectors in "river bank" vs. "bank account" |
| **Feature-based** | ELMo vectors are **concatenated** with existing task model inputs — the downstream architecture is unchanged |
| **Multi-layer** | Final representation is a learned weighted sum of all LSTM layers |
| **Pre-training objective** | Bidirectional language modeling: forward LM + backward LM (trained independently) |
| **Parameters** | ~94M (2-layer biLSTM, 4096 hidden units) |

> **Key Insight:** ELMo showed that different LSTM layers capture different linguistic properties — lower layers encode syntax, higher layers encode semantics. The task-specific weighting lets each task pick the right mix.

---

## 4. ULMFiT — Universal Language Model Fine-tuning (Howard & Ruder, 2018)

**ULMFiT** was the first method to propose a **complete fine-tuning recipe** for language models applied to text classification. It introduced three critical techniques that made fine-tuning stable and effective.

### 4.1 Three-Stage Training Process

```
 ┌──────────────────────────────────────────────────────────────────┐
 │                    ULMFiT: Three Stages                          │
 │                                                                  │
 │  STAGE 1: General-Domain Pre-training                            │
 │  ════════════════════════════════════                             │
 │  Corpus: Wikitext-103 (103M tokens)                              │
 │  Model:  3-layer AWD-LSTM                                        │
 │  Task:   Standard LM (predict next word)                         │
 │  Result: General language knowledge                               │
 │                         │                                        │
 │                         ▼                                        │
 │  STAGE 2: Target-Domain Fine-tuning                              │
 │  ══════════════════════════════════                               │
 │  Corpus: Unlabeled target domain text (e.g., IMDb reviews)       │
 │  Task:   Continue LM training on domain text                     │
 │  Technique: Discriminative fine-tuning + STLR                    │
 │  Result: Domain-adapted language model                            │
 │                         │                                        │
 │                         ▼                                        │
 │  STAGE 3: Target-Task Fine-tuning                                │
 │  ════════════════════════════════                                 │
 │  Data:   Labeled task data (e.g., sentiment labels)              │
 │  Task:   Classification with added softmax head                  │
 │  Technique: Gradual unfreezing + discriminative LR               │
 │  Result: Task-specific classifier                                 │
 └──────────────────────────────────────────────────────────────────┘
```

### 4.2 Key Techniques

**1. Discriminative Fine-tuning:** Instead of using a single learning rate for the whole model, each layer gets a **different learning rate**. Lower layers (general features) get smaller learning rates; higher layers (task-specific features) get larger ones.

```
 Layer-wise Learning Rates:
 ══════════════════════════

 Layer 3 (top):    η₃ = η            ← Largest (adapts most)
 Layer 2:          η₂ = η / 2.6      ← Medium
 Layer 1 (bottom): η₁ = η / 2.6²    ← Smallest (preserves general knowledge)

 General rule:  ηₗ₋₁ = ηₗ / 2.6
```

**2. Slanted Triangular Learning Rate (STLR):** A schedule that quickly warms up the learning rate then gradually decays it — allowing the model to rapidly converge to a suitable region then carefully refine.

```
 Learning Rate
      ↑
  η_max ┤        /\
        │       /  \
        │      /    \
        │     /      \
        │    /        \_________
  η_min ┤___/                    \___
        └──────────────────────────────▶ Training Steps
             ↑              ↑
         Warm-up       Gradual Decay
        (short)         (long)

  cut_frac = 0.1   (10% of training for warm-up)
```

**3. Gradual Unfreezing:** During task fine-tuning, layers are unfrozen one at a time starting from the **last layer** (closest to the output), preventing catastrophic forgetting:

```
 Epoch 1:  [FROZEN] [FROZEN] [FROZEN] [TRAIN ✓]  ← Only last layer
 Epoch 2:  [FROZEN] [FROZEN] [TRAIN ✓] [TRAIN ✓]  ← Unfreeze one more
 Epoch 3:  [FROZEN] [TRAIN ✓] [TRAIN ✓] [TRAIN ✓]  ← Unfreeze one more
 Epoch 4:  [TRAIN ✓] [TRAIN ✓] [TRAIN ✓] [TRAIN ✓]  ← All layers training
```

> **Key Insight:** ULMFiT demonstrated that with the right fine-tuning techniques, a pre-trained language model could match or beat task-specific architectures trained on 10–100× more labeled data, achieving a **"ImageNet moment" for NLP.**

---

## 5. BERT — Bidirectional Encoder Representations from Transformers (Devlin et al., 2019)

**BERT** revolutionized NLP by introducing a **deeply bidirectional** pre-training approach using Transformers, outperforming all previous models on 11 NLP benchmarks simultaneously.

### 5.1 Pre-training Objectives

BERT uses two self-supervised objectives:

**Objective 1 — Masked Language Model (MLM):**

```
 Original:    "The cat  sat  on  the  mat"
 Masked:      "The cat [MASK] on  the  mat"

 Task: Predict "sat" using BOTH left ("The cat") AND right ("on the mat") context.

 Masking strategy (for each selected token):
   80% → replace with [MASK]
   10% → replace with random word
   10% → keep unchanged

 Why not 100% [MASK]?
   → Prevents mismatch between pre-training (sees [MASK]) and
     fine-tuning (never sees [MASK]).
```

**Objective 2 — Next Sentence Prediction (NSP):**

```
 Input:  [CLS] Sentence A [SEP] Sentence B [SEP]

 Positive (IsNext):
   A = "The cat sat on the mat"
   B = "It was a fluffy orange tabby"           → Label: IsNext

 Negative (NotNext):
   A = "The cat sat on the mat"
   B = "Stock prices rose 3% on Monday"         → Label: NotNext

 The [CLS] token's final hidden state is used to classify IsNext vs. NotNext.
```

### 5.2 Architecture

```
 ┌─────────────────────────────────────────────────────────────────┐
 │                      BERT Architecture                          │
 │                                                                 │
 │  Input:  [CLS]  The  cat  [MASK]  on  the  mat  [SEP]         │
 │            │     │    │     │     │    │    │     │             │
 │            ▼     ▼    ▼     ▼     ▼    ▼    ▼     ▼             │
 │  ┌──────────────────────────────────────────────────────┐      │
 │  │  Token Embeddings + Segment Embeddings + Position    │      │
 │  └──────────────────────────┬───────────────────────────┘      │
 │                             │                                   │
 │                             ▼                                   │
 │  ┌──────────────────────────────────────────────────────┐      │
 │  │         Transformer Encoder Block × 12 (or 24)       │      │
 │  │                                                      │      │
 │  │   ┌─────────────────────────────────────────────┐    │      │
 │  │   │  Multi-Head Self-Attention (bidirectional)   │    │      │
 │  │   │  ← Each token attends to ALL other tokens → │    │      │
 │  │   └─────────────────────┬───────────────────────┘    │      │
 │  │                         │                            │      │
 │  │   ┌─────────────────────▼───────────────────────┐    │      │
 │  │   │  Feed-Forward Network + LayerNorm + Residual │    │      │
 │  │   └─────────────────────────────────────────────┘    │      │
 │  └──────────────────────────┬───────────────────────────┘      │
 │                             │                                   │
 │                             ▼                                   │
 │    T_[CLS]  T_The  T_cat  T_[MASK]  T_on  T_the  T_mat  T_[SEP]│
 │      │                      │                                   │
 │      ▼                      ▼                                   │
 │   NSP Head              MLM Head                                │
 │  (IsNext?)            (predict "sat")                           │
 └─────────────────────────────────────────────────────────────────┘

 BERT-Base:   L=12, H=768,  A=12, Params=110M
 BERT-Large:  L=24, H=1024, A=16, Params=340M
```

### 5.3 Fine-tuning BERT

Unlike ELMo's feature-based approach, BERT fine-tuning modifies the **entire model end-to-end** by adding a thin task-specific head:

```
 ┌────────────────────┐  ┌────────────────────┐  ┌─────────────────────┐
 │  Classification    │  │  Token Labeling    │  │  Question Answering │
 │  (e.g., sentiment) │  │  (e.g., NER)       │  │                     │
 ├────────────────────┤  ├────────────────────┤  ├─────────────────────┤
 │  [CLS] → Linear   │  │  Each Tᵢ → Linear  │  │  Each Tᵢ → start/  │
 │       → Softmax   │  │       → Softmax    │  │  end span logits    │
 │       → label     │  │       → tag        │  │       → answer span │
 └────────────────────┘  └────────────────────┘  └─────────────────────┘
         ▲                       ▲                       ▲
         │                       │                       │
 ┌───────┴───────────────────────┴───────────────────────┴──────────┐
 │                    BERT (all layers fine-tuned)                   │
 └──────────────────────────────────────────────────────────────────┘
```

---

## 6. GPT — Generative Pre-trained Transformer (Radford et al., 2018)

**GPT** takes a different approach from BERT: it uses a **unidirectional (left-to-right) Transformer decoder** for **autoregressive language modeling**, then fine-tunes on downstream tasks.

### 6.1 Architecture & Pre-training

```
 ┌─────────────────────────────────────────────────────────────────┐
 │                       GPT Architecture                          │
 │                                                                 │
 │  Input:    w₁      w₂      w₃      w₄      w₅                │
 │            │       │       │       │       │                   │
 │            ▼       ▼       ▼       ▼       ▼                   │
 │  ┌──────────────────────────────────────────────────────┐      │
 │  │   Token Embeddings + Positional Embeddings           │      │
 │  └──────────────────────────┬───────────────────────────┘      │
 │                             │                                   │
 │                             ▼                                   │
 │  ┌──────────────────────────────────────────────────────┐      │
 │  │       Transformer Decoder Block × 12                  │      │
 │  │                                                      │      │
 │  │   ┌─────────────────────────────────────────────┐    │      │
 │  │   │  Masked Self-Attention (unidirectional)      │    │      │
 │  │   │  ← Each token attends only to LEFT context → │    │      │
 │  │   └─────────────────────┬───────────────────────┘    │      │
 │  │                         │                            │      │
 │  │   ┌─────────────────────▼───────────────────────┐    │      │
 │  │   │  Feed-Forward Network + LayerNorm + Residual │    │      │
 │  │   └─────────────────────────────────────────────┘    │      │
 │  └──────────────────────────┬───────────────────────────┘      │
 │                             │                                   │
 │                             ▼                                   │
 │            h₁      h₂      h₃      h₄      h₅                │
 │            │       │       │       │       │                   │
 │            ▼       ▼       ▼       ▼       ▼                   │
 │    Predict w₂  Predict w₃  Predict w₄  Predict w₅  Predict w₆ │
 │                                                                 │
 │  Objective: maximize Σ log P(wₜ | w₁, ..., wₜ₋₁)              │
 └─────────────────────────────────────────────────────────────────┘

 GPT-1:   12 layers, 768 hidden, 12 heads, 117M parameters
 Trained on: BooksCorpus (~800M words)
```

### 6.2 GPT vs. BERT: Directionality

```
 BERT (Bidirectional):   "The cat [MASK] on the mat"
                          ←── context ──→ ←── context ──→
                          ALL tokens attend to ALL tokens

 GPT (Unidirectional):   "The cat sat on the mat"
                          ──→  ──→  ──→  ──→  ──→
                          Each token attends only to tokens BEFORE it

 BERT excels at:  Understanding tasks (classification, NER, QA)
 GPT excels at:   Generation tasks (text completion, dialogue, summarization)
```

### 6.3 The GPT Evolution

| Model | Year | Parameters | Training Data | Key Innovation |
|-------|------|------------|---------------|----------------|
| GPT-1 | 2018 | 117M | BooksCorpus (800M words) | Transformer LM + fine-tuning |
| GPT-2 | 2019 | 1.5B | WebText (40GB) | Zero-shot via prompting, no fine-tuning needed |
| GPT-3 | 2020 | 175B | 570GB text | Few-shot in-context learning |
| GPT-4 | 2023 | ~1.8T (est.) | Massive + RLHF | Multimodal, instruction following |

---

## 7. Timeline & Evolution

```
 ════════════════════════════════════════════════════════════════════════
                    Evolution of Pre-training in NLP
 ════════════════════════════════════════════════════════════════════════

  2013 │  Word2Vec (Mikolov)
       │  └─ Static word embeddings, no context
       │
  2014 │  GloVe (Pennington)
       │  └─ Global co-occurrence statistics → static embeddings
       │
  2015 │  ── gap ── (task-specific models dominate)
       │
  2017 │  Transformer (Vaswani) — "Attention Is All You Need"
       │  └─ Self-attention architecture replaces RNNs
       │
  2018 │  ╔══════════════════════════════════════════════╗
  Jan  │  ║  ELMo (Peters et al.)                       ║
       │  ║  └─ Contextualized embeddings from biLSTM   ║
       │  ║  └─ Feature-based: add to existing models   ║
       │  ╠══════════════════════════════════════════════╣
  Jan  │  ║  ULMFiT (Howard & Ruder)                    ║
       │  ║  └─ First complete fine-tuning recipe        ║
       │  ║  └─ Discriminative LR + gradual unfreezing  ║
       │  ╠══════════════════════════════════════════════╣
  Jun  │  ║  GPT-1 (Radford et al.)                     ║
       │  ║  └─ Unidirectional Transformer + fine-tune   ║
       │  ║  └─ Generative pre-training                 ║
       │  ╠══════════════════════════════════════════════╣
  Oct  │  ║  BERT (Devlin et al.)                       ║
       │  ║  └─ Bidirectional Transformer encoder        ║
       │  ║  └─ MLM + NSP objectives                    ║
       │  ╚══════════════════════════════════════════════╝
       │
  2019 │  GPT-2, RoBERTa, ALBERT, DistilBERT, XLNet, T5
       │  └─ Scaling up, removing NSP, distillation, encoder-decoder
       │
  2020 │  GPT-3 (175B) — Few-shot learning, in-context learning
       │
  2022 │  InstructGPT / ChatGPT — RLHF alignment
       │
  2023+│  GPT-4, LLaMA, Gemini — Multimodal, open-source, reasoning
       │
 ════════════════════════════════════════════════════════════════════════
       2018 = the "ImageNet moment" for NLP
 ════════════════════════════════════════════════════════════════════════
```

---

## 8. Python Implementation — Fine-tuning BERT for Text Classification

```python
import torch
from torch.utils.data import DataLoader, Dataset
from transformers import BertTokenizer, BertForSequenceClassification
from transformers import AdamW, get_linear_schedule_with_warmup

# ─── Dataset ─────────────────────────────────────────────────────

class SentimentDataset(Dataset):
    """Simple sentiment classification dataset."""

    def __init__(self, texts, labels, tokenizer, max_len=128):
        self.texts = texts
        self.labels = labels
        self.tokenizer = tokenizer
        self.max_len = max_len

    def __len__(self):
        return len(self.texts)

    def __getitem__(self, idx):
        encoding = self.tokenizer(
            self.texts[idx],
            max_length=self.max_len,
            padding="max_length",
            truncation=True,
            return_tensors="pt",
        )
        return {
            "input_ids": encoding["input_ids"].squeeze(),       # (max_len,)
            "attention_mask": encoding["attention_mask"].squeeze(),
            "labels": torch.tensor(self.labels[idx], dtype=torch.long),
        }


# ─── Sample Data ─────────────────────────────────────────────────

texts = [
    "This movie was absolutely wonderful and moving.",
    "Terrible film, waste of time.",
    "A masterpiece of modern cinema.",
    "Boring plot and bad acting throughout.",
    "I loved every minute of this beautiful story.",
    "The worst movie I have ever seen.",
]
labels = [1, 0, 1, 0, 1, 0]   # 1 = positive, 0 = negative


# ─── Model & Tokenizer ──────────────────────────────────────────

tokenizer = BertTokenizer.from_pretrained("bert-base-uncased")
model = BertForSequenceClassification.from_pretrained(
    "bert-base-uncased",
    num_labels=2,       # binary classification
)

dataset = SentimentDataset(texts, labels, tokenizer)
loader = DataLoader(dataset, batch_size=2, shuffle=True)


# ─── Fine-tuning Hyperparameters ─────────────────────────────────

EPOCHS = 3
LEARNING_RATE = 2e-5                         # typical BERT fine-tuning LR
WARMUP_STEPS = 0

optimizer = AdamW(model.parameters(), lr=LEARNING_RATE, weight_decay=0.01)
total_steps = len(loader) * EPOCHS
scheduler = get_linear_schedule_with_warmup(
    optimizer,
    num_warmup_steps=WARMUP_STEPS,
    num_training_steps=total_steps,
)


# ─── Training Loop ──────────────────────────────────────────────

device = torch.device("cuda" if torch.cuda.is_available() else "cpu")
model.to(device)

for epoch in range(EPOCHS):
    model.train()
    total_loss = 0

    for batch in loader:
        input_ids = batch["input_ids"].to(device)
        attention_mask = batch["attention_mask"].to(device)
        labels_batch = batch["labels"].to(device)

        optimizer.zero_grad()
        outputs = model(
            input_ids=input_ids,
            attention_mask=attention_mask,
            labels=labels_batch,
        )
        loss = outputs.loss
        total_loss += loss.item()

        loss.backward()
        torch.nn.utils.clip_grad_norm_(model.parameters(), max_norm=1.0)
        optimizer.step()
        scheduler.step()

    avg_loss = total_loss / len(loader)
    print(f"Epoch {epoch+1}/{EPOCHS} | Loss: {avg_loss:.4f}")


# ─── Inference ───────────────────────────────────────────────────

model.eval()
test_text = "An incredible and heartwarming story!"

inputs = tokenizer(test_text, return_tensors="pt", truncation=True, max_length=128)
inputs = {k: v.to(device) for k, v in inputs.items()}

with torch.no_grad():
    logits = model(**inputs).logits
    prediction = torch.argmax(logits, dim=1).item()

print(f"\nText: '{test_text}'")
print(f"Prediction: {'Positive' if prediction == 1 else 'Negative'}")
```

**Expected output:**
```
Epoch 1/3 | Loss: 0.7124
Epoch 2/3 | Loss: 0.4532
Epoch 3/3 | Loss: 0.1876

Text: 'An incredible and heartwarming story!'
Prediction: Positive
```

> **Note:** In practice, use a proper train/validation split, larger datasets, and early stopping. The HuggingFace `Trainer` API simplifies this entire pipeline.

---

## 9. Summary & Comparison Table

| Aspect                  | ELMo                          | ULMFiT                         | BERT                            | GPT                             |
|-------------------------|-------------------------------|--------------------------------|----------------------------------|----------------------------------|
| **Year**                | Feb 2018                      | Jan 2018                       | Oct 2018                         | Jun 2018                         |
| **Authors**             | Peters et al.                 | Howard & Ruder                 | Devlin et al.                    | Radford et al.                   |
| **Architecture**        | 2-layer biLSTM                | 3-layer AWD-LSTM               | Transformer Encoder              | Transformer Decoder              |
| **Directionality**      | Bidirectional (shallow)       | Unidirectional (forward LM)    | Deeply bidirectional             | Unidirectional (left-to-right)   |
| **Pre-training Task**   | Forward LM + Backward LM     | Forward LM                     | MLM + NSP                        | Autoregressive LM               |
| **Adaptation Method**   | Feature-based (freeze & extract) | Fine-tune entire model       | Fine-tune entire model           | Fine-tune entire model           |
| **Key Innovation**      | Contextualized embeddings     | Fine-tuning recipe (disc. LR, gradual unfreeze, STLR) | Masked LM for deep bidirectionality | Generative pre-training on Transformers |
| **Parameters**          | ~94M                          | ~24M                           | 110M (Base) / 340M (Large)       | 117M                             |
| **Training Data**       | 1B Word Benchmark             | Wikitext-103                   | BooksCorpus + Wikipedia (3.3B words) | BooksCorpus (800M words)        |
| **Strengths**           | Drop-in improvement for any model | Works with very little labeled data | SOTA on understanding tasks    | Strong generation capability     |
| **Weaknesses**          | Slow (biLSTM), feature-only   | LSTM-based, limited scale      | Cannot generate text naturally   | Misses right-side context        |
| **Best For**            | Adding context to existing models | Low-resource classification  | NLU: classification, NER, QA    | NLG: text generation, completion |

---

## 10. Quick Revision Questions

1. **What is the fundamental difference between the old "train from scratch" paradigm and the "pre-train + fine-tune" paradigm?** *(Answer: In the old paradigm, each task required training a randomly initialized model from scratch on task-specific labeled data. In the new paradigm, a model is first pre-trained on large unlabeled text to learn universal language representations, then fine-tuned on small task-specific labeled data — enabling knowledge transfer and dramatically reducing the need for labeled examples.)*

2. **How does ELMo differ from Word2Vec/GloVe in its word representations, and how is it integrated into downstream tasks?** *(Answer: Unlike Word2Vec/GloVe which produce a single static vector per word, ELMo produces contextualized vectors — the same word gets different representations depending on its surrounding context. ELMo is integrated as a feature-based approach: its vectors are concatenated with existing task model inputs without modifying the downstream architecture.)*

3. **Explain the three key techniques in ULMFiT and why each is important for stable fine-tuning.** *(Answer: (a) Discriminative fine-tuning — different layers get different learning rates (lower layers get smaller LR) to preserve general knowledge while adapting task-specific layers. (b) Slanted triangular learning rates — a schedule that warms up quickly then decays slowly, helping the model converge to a good region then refine. (c) Gradual unfreezing — layers are unfrozen one at a time from top to bottom, preventing catastrophic forgetting of pre-trained knowledge.)*

4. **Why does BERT use Masked Language Modeling instead of standard left-to-right language modeling?** *(Answer: Standard left-to-right LM only allows each token to attend to its left context, preventing deep bidirectional representations. MLM randomly masks tokens and predicts them from both left and right context simultaneously, enabling truly bidirectional encoding. This is critical for understanding tasks where the meaning of a word depends on its full surrounding context.)*

5. **What are the key architectural and philosophical differences between BERT and GPT?** *(Answer: BERT uses a Transformer encoder with bidirectional self-attention and is pre-trained with MLM — it sees all context simultaneously and excels at understanding tasks (classification, NER, QA). GPT uses a Transformer decoder with unidirectional (masked) self-attention and is pre-trained autoregressively — it generates text left-to-right and excels at generation tasks (completion, dialogue, summarization).)*

6. **Why is 2018 often called the "ImageNet moment" for NLP?** *(Answer: Just as ImageNet pre-training in 2012 revolutionized computer vision by enabling transfer learning across vision tasks, 2018 saw ELMo, ULMFiT, GPT, and BERT demonstrate that language model pre-training could be transferred to virtually any NLP task — achieving state-of-the-art results with minimal labeled data and making high-quality NLP accessible without massive task-specific datasets.)*

---

> [← Previous: Neural Language Models](./03-neural-language-models.md) | [Next → Transfer Learning](./05-transfer-learning.md)

[🔼 Back to Unit Overview](../README.md)
