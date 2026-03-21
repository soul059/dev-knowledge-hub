# GPT-2 and GPT-3: Scaling Up

> **Navigation:** [← Autoregressive LM](02-autoregressive-language-modeling.md) | [Scaling Laws →](04-scaling-laws.md)

---

## Overview

**GPT-2** (Radford et al., 2019) and **GPT-3** (Brown et al., 2020) represent landmark milestones in the scaling of autoregressive language models. GPT-2 demonstrated that a 1.5 billion parameter model trained on diverse internet text could perform **zero-shot task transfer** — solving tasks it was never explicitly trained on — leading to its famous tagline: *"Language Models are Unsupervised Multitask Learners."* GPT-3 pushed this further to 175 billion parameters, showing that with sufficient scale, language models exhibit remarkable **few-shot learning** capabilities, performing tasks from just a handful of examples in the prompt — without any gradient updates. Together, these models established that **scale is a powerful lever** for improving general-purpose language understanding and generation.

---

## 1. The Scaling Progression

```
  GPT-1 (2018)          GPT-2 (2019)          GPT-3 (2020)
  ┌──────────┐          ┌──────────┐          ┌──────────┐
  │  117M    │          │  1.5B    │          │  175B    │
  │  params  │ ──13×──► │  params  │ ─117×──► │  params  │
  └──────────┘          └──────────┘          └──────────┘
   12 layers             48 layers             96 layers
   768 hidden            1600 hidden           12288 hidden
   12 heads              25 heads              96 heads
   512 ctx               1024 ctx              2048 ctx
```

---

## 2. GPT-2: "Language Models are Unsupervised Multitask Learners"

### 2.1 Architecture Changes from GPT-1

| Feature               | GPT-1                    | GPT-2                       |
|------------------------|--------------------------|-----------------------------|
| Parameters             | 117M                     | 1.5B (largest variant)      |
| Layers                 | 12                       | 48                          |
| Hidden dimension       | 768                      | 1600                        |
| Attention heads        | 12                       | 25                          |
| Context length         | 512                      | 1024                        |
| Vocabulary             | 40,000 (BPE)             | 50,257 (Byte-level BPE)     |
| Training data          | BooksCorpus (5GB)        | WebText (40GB)              |
| Layer normalization    | Post-norm                | **Pre-norm** (moved before) |
| Initialization scaling | Standard                 | Scale by 1/√N per layer     |

### 2.2 Key Architectural Modifications

**Pre-Layer Normalization (Pre-LN):**

```
  GPT-1 (Post-LN):                    GPT-2 (Pre-LN):

  x ──► Attention ──► Add ──► LN      x ──► LN ──► Attention ──► Add
        │               ▲                    │                     ▲
        └───────────────┘                    └─────────────────────┘
                                                  (residual)
```

Pre-LN improves training stability, especially for deeper networks.

**Residual Weight Scaling:**

```
Output of each residual block is scaled by 1/√N where N = number of layers.

This prevents the residual stream magnitude from growing with depth:
  h_out = h_in + (1/√N) · sublayer(h_in)
```

### 2.3 GPT-2 Model Variants

| Model           | Layers | Hidden | Heads | Parameters |
|-----------------|--------|--------|-------|------------|
| GPT-2 Small     | 12     | 768    | 12    | 117M       |
| GPT-2 Medium    | 24     | 1024   | 16    | 345M       |
| GPT-2 Large     | 36     | 1280   | 20    | 774M       |
| GPT-2 XL        | 48     | 1600   | 25    | 1.5B       |

### 2.4 Zero-Shot Task Transfer

GPT-2's key insight: **all NLP tasks can be framed as language modeling.**

```
  Task: Translation
  Prompt: "translate English to French: cheese =>"
  Output: "fromage"

  Task: Summarization
  Prompt: "Article: [long text]... TL;DR:"
  Output: "Summary of the article..."

  Task: Question Answering
  Prompt: "Q: What is the capital of France? A:"
  Output: "Paris"
```

No task-specific training data, no fine-tuning — just a well-trained language model completing text.

### 2.5 Byte-Level BPE

GPT-2 switched to **byte-level BPE** from character-level BPE:

```
Character-level BPE (GPT-1):
  - Base vocabulary: Unicode characters (~10K)
  - Can't handle arbitrary byte sequences

Byte-level BPE (GPT-2):
  - Base vocabulary: 256 bytes
  - Can represent ANY text (any language, any encoding)
  - Vocab size: 50,257 tokens
  - No <UNK> tokens ever needed
```

---

## 3. GPT-3: "Language Models are Few-Shot Learners"

### 3.1 Architecture at Scale

| Feature               | GPT-3 (175B)           |
|------------------------|------------------------|
| Parameters             | 175 billion            |
| Layers                 | 96                     |
| Hidden dimension       | 12,288                 |
| Attention heads        | 96                     |
| Head dimension         | 128                    |
| Feed-forward dim       | 49,152 (4 × 12,288)   |
| Context length         | 2,048                  |
| Vocabulary             | 50,257                 |
| Training data          | ~570GB filtered text   |
| Training compute       | ~3,640 petaflop-days   |
| Training cost          | ~$4.6 million (est.)   |

### 3.2 GPT-3 Model Variants

| Model             | Params    | Layers | Hidden  | Heads |
|-------------------|-----------|--------|---------|-------|
| GPT-3 Small       | 125M      | 12     | 768     | 12    |
| GPT-3 Medium      | 350M      | 24     | 1024    | 16    |
| GPT-3 Large       | 760M      | 24     | 1536    | 16    |
| GPT-3 XL          | 1.3B      | 24     | 2048    | 32    |
| GPT-3 2.7B        | 2.7B      | 32     | 2560    | 32    |
| GPT-3 6.7B        | 6.7B      | 32     | 4096    | 32    |
| GPT-3 13B         | 13B       | 40     | 5140    | 40    |
| **GPT-3 175B**    | **175B**  | **96** | **12288**| **96**|

### 3.3 Training Data Composition

```
  Dataset               Tokens (B)    Weight in Mix
  ┌───────────────────┬───────────┬─────────────────┐
  │ Common Crawl       │   410     │     60%         │
  │ WebText2            │    19     │     22%         │
  │ Books1              │    12     │      8%         │
  │ Books2              │    55     │      8%         │
  │ Wikipedia           │     3     │      3%         │
  └───────────────────┴───────────┴─────────────────┘
  Total: ~499B tokens, but model trained on ~300B tokens
  (some datasets sampled more than once, others less)
```

### 3.4 Few-Shot Learning Paradigm

GPT-3 demonstrated three settings for task adaptation **without fine-tuning**:

```
  ┌──────────────────────────────────────────────────────────┐
  │                                                          │
  │  Zero-shot:  "Translate English to French: cheese =>"    │
  │              Task description only, no examples           │
  │                                                          │
  │  One-shot:   "Translate English to French:               │
  │               sea otter => loutre de mer                  │
  │               cheese =>"                                  │
  │              One example provided                         │
  │                                                          │
  │  Few-shot:   "Translate English to French:               │
  │               sea otter => loutre de mer                  │
  │               peppermint => menthe poivrée               │
  │               plush giraffe => girafe en peluche         │
  │               cheese =>"                                  │
  │              Multiple examples provided                   │
  │                                                          │
  └──────────────────────────────────────────────────────────┘
```

### 3.5 Emergent Abilities

As models scale, certain capabilities appear to **emerge** suddenly:

```
  Performance
  ▲
  │                              ●  ← GPT-3 175B
  │                         ●
  │                    ●
  │               ●
  │          ●
  │     ●
  │  ●                          Capabilities that emerge:
  │ ●                           • Arithmetic (3-digit addition)
  │●                            • Word unscrambling
  ├──────────────────────► Scale • Analogical reasoning
    125M  1.3B  13B  175B        • Translation (zero-shot)
```

---

## 4. Comprehensive Version Comparison

| Feature              | GPT-1            | GPT-2            | GPT-3            |
|----------------------|------------------|------------------|------------------|
| Year                 | 2018             | 2019             | 2020             |
| Parameters           | 117M             | 1.5B             | 175B             |
| Layers               | 12               | 48               | 96               |
| Hidden dim           | 768              | 1600             | 12,288           |
| Heads                | 12               | 25               | 96               |
| Context length       | 512              | 1024             | 2048             |
| Training data        | BooksCorpus 5GB  | WebText 40GB     | ~570GB           |
| Vocab size           | 40,000           | 50,257           | 50,257           |
| Tokenizer            | BPE              | Byte-level BPE   | Byte-level BPE   |
| Layer norm            | Post-LN          | Pre-LN           | Pre-LN           |
| Key innovation       | Pre-train+       | Zero-shot        | Few-shot         |
|                      | fine-tune         | transfer         | learning         |
| Task adaptation      | Fine-tuning      | Zero-shot        | Zero/few-shot    |
| Open weights?        | Yes              | Yes              | No (API only)    |

---

## 5. Mathematical Analysis: Parameter Count

### GPT-3 175B Parameter Breakdown

```
Embedding parameters:
  Token embedding:     50,257 × 12,288             =   617,558,016
  Position embedding:   2,048 × 12,288             =    25,165,824

Per transformer layer:
  Multi-head attention:
    Q, K, V projections: 3 × (12,288 × 12,288)     =   452,984,832
    Output projection:   12,288 × 12,288            =   150,994,944
  Feed-forward:
    Linear 1:            12,288 × 49,152            =   603,979,776
    Linear 2:            49,152 × 12,288            =   603,979,776
  Layer norms (×2):      2 × 2 × 12,288            =        49,152
  Subtotal per layer:                               = 1,811,988,480

Total for 96 layers:     96 × 1,811,988,480         = 173,950,893,120

Final layer norm:        2 × 12,288                 =        24,576

Grand total ≈ 174.6B (output head tied with token embedding)
```

---

## 6. Code: Using GPT-2 with HuggingFace

```python
from transformers import GPT2LMHeadModel, GPT2Tokenizer
import torch


def load_gpt2(model_name="gpt2"):
    """Load GPT-2 model and tokenizer from HuggingFace."""
    tokenizer = GPT2Tokenizer.from_pretrained(model_name)
    model = GPT2LMHeadModel.from_pretrained(model_name)
    model.eval()
    return model, tokenizer


def generate_text(model, tokenizer, prompt, max_length=100,
                  temperature=0.8, top_k=50, top_p=0.92,
                  num_return_sequences=1):
    """Generate text using GPT-2 with various decoding strategies."""
    input_ids = tokenizer.encode(prompt, return_tensors="pt")

    with torch.no_grad():
        outputs = model.generate(
            input_ids,
            max_length=max_length,
            temperature=temperature,
            top_k=top_k,
            top_p=top_p,
            do_sample=True,
            num_return_sequences=num_return_sequences,
            pad_token_id=tokenizer.eos_token_id,
        )

    results = []
    for i, output in enumerate(outputs):
        text = tokenizer.decode(output, skip_special_tokens=True)
        results.append(text)
        print(f"\n--- Generation {i+1} ---")
        print(text)

    return results


def compare_model_sizes():
    """Compare different GPT-2 model sizes."""
    model_names = ["gpt2", "gpt2-medium", "gpt2-large", "gpt2-xl"]

    for name in model_names:
        model = GPT2LMHeadModel.from_pretrained(name)
        n_params = sum(p.numel() for p in model.parameters())
        n_layers = model.config.n_layer
        n_heads = model.config.n_head
        d_model = model.config.n_embd
        print(f"{name:12s} | {n_params/1e6:7.1f}M params | "
              f"{n_layers} layers | {d_model} hidden | {n_heads} heads")
        del model


def compute_perplexity(model, tokenizer, text):
    """Compute perplexity of a text under GPT-2."""
    input_ids = tokenizer.encode(text, return_tensors="pt")

    with torch.no_grad():
        outputs = model(input_ids, labels=input_ids)
        loss = outputs.loss  # cross-entropy loss

    perplexity = torch.exp(loss).item()
    return perplexity


# --- Main ---
if __name__ == "__main__":
    model, tokenizer = load_gpt2("gpt2")

    # Generate text
    prompt = "In the year 2050, artificial intelligence"
    print(f"Prompt: {prompt}\n")

    # Greedy decoding
    input_ids = tokenizer.encode(prompt, return_tensors="pt")
    greedy_out = model.generate(input_ids, max_length=60, do_sample=False)
    print("Greedy:", tokenizer.decode(greedy_out[0], skip_special_tokens=True))

    # Nucleus sampling
    generate_text(model, tokenizer, prompt,
                  max_length=60, temperature=0.9, top_p=0.92)

    # Compute perplexity
    test_text = "The cat sat on the mat and watched the birds outside."
    ppl = compute_perplexity(model, tokenizer, test_text)
    print(f"\nPerplexity of test text: {ppl:.2f}")
```

---

## 7. Real-World Applications

| Application                | Model Used      | Why It Matters                               |
|----------------------------|-----------------|----------------------------------------------|
| **ChatGPT**                | GPT-3.5/4       | Conversational AI for millions of users       |
| **GitHub Copilot**         | Codex (GPT-3)   | AI-powered code completion                    |
| **Content Generation**     | GPT-3 API       | Marketing copy, blog posts, emails            |
| **Translation**            | GPT-3           | Competitive zero-shot translation             |
| **Education**              | GPT-3/4         | Tutoring, question generation, explanations   |
| **Code Debugging**         | GPT-4           | Finding and explaining bugs in code           |

---

## 8. Summary Table

| Concept                    | Key Point                                                  |
|----------------------------|------------------------------------------------------------|
| GPT-2 size                 | 1.5B parameters, 48 layers, 1600 hidden                   |
| GPT-3 size                 | 175B parameters, 96 layers, 12,288 hidden                  |
| Scale factor               | GPT-1→2: 13×; GPT-2→3: 117×; GPT-1→3: 1500×              |
| GPT-2 innovation           | Zero-shot task transfer without fine-tuning                |
| GPT-3 innovation           | Few-shot learning via in-context examples                  |
| Pre-LN                     | Layer norm before (not after) sublayers; better stability  |
| Byte-level BPE             | 50,257 vocab; handles any text without UNK tokens          |
| Emergent abilities         | Capabilities that appear only at sufficient scale          |
| Training data scaling      | 5GB → 40GB → 570GB across versions                        |

---

## 9. Revision Questions

1. **What is the key architectural difference between GPT-1 and GPT-2?** Why does Pre-LN help with training deeper networks?

2. **Explain the concept of zero-shot task transfer.** How does GPT-2 perform translation without being explicitly trained on parallel corpora?

3. **What is the difference between zero-shot, one-shot, and few-shot learning in GPT-3?** Provide examples for a sentiment analysis task.

4. **Calculate the approximate parameter count** for a transformer with 48 layers, hidden dim 1600, and vocab 50,257.

5. **Why are GPT-3's weights not publicly released?** Discuss the trade-offs between open and closed model release.

6. **What are emergent abilities?** Give three examples of capabilities that only appear at large scale.

---

> **Navigation:** [← Autoregressive LM](02-autoregressive-language-modeling.md) | [Scaling Laws →](04-scaling-laws.md)
