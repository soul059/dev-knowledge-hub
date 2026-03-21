# 3.4 — Transformers and Attention Mechanisms

> *"Attention Is All You Need"* — Vaswani et al., 2017

## Overview

Transformers have fundamentally reshaped the landscape of deep learning since their introduction in 2017. By replacing recurrent and convolutional architectures with a purely attention-based mechanism, Transformers achieve superior parallelization, capture long-range dependencies effortlessly, and scale to billions of parameters. This module provides a rigorous, ground-up exploration of attention mechanisms, the Transformer architecture, and the family of models it has spawned — from BERT and GPT to Vision Transformers and modern Large Language Models. Across **11 units and 59 lessons**, you will build deep intuition, mathematical fluency, and practical know-how for one of the most consequential innovations in artificial intelligence.

---

## Learning Objectives

By the end of this module you will be able to:

- ✅ **Explain** the intuition behind attention and contrast soft vs. hard attention paradigms
- ✅ **Derive** the scaled dot-product attention formula and compute attention weights by hand
- ✅ **Implement** self-attention and multi-head attention from scratch using matrix operations
- ✅ **Diagram** the complete Transformer encoder-decoder architecture including residual connections, layer normalization, and positional encoding
- ✅ **Compare** sinusoidal, learned, relative, and rotary positional encoding schemes
- ✅ **Describe** pre-training objectives (MLM, NSP, autoregressive LM) and fine-tuning strategies for BERT and GPT model families
- ✅ **Analyze** scaling laws, in-context learning, and prompt engineering techniques for large language models
- ✅ **Evaluate** modern efficiency innovations such as sparse attention, linear attention, and FlashAttention
- ✅ **Apply** parameter-efficient fine-tuning (LoRA, adapters, prefix tuning) and RLHF to adapt LLMs
- ✅ **Design** end-to-end solutions using Retrieval-Augmented Generation (RAG) pipelines

---

## Architecture Overview

```
╔══════════════════════════════════════════════════════════════════════════════════╗
║                      THE  TRANSFORMER  ARCHITECTURE                            ║
║                      ═══════════════════════════════                            ║
║                                                                                ║
║          ENCODER  (×N)                           DECODER  (×N)                 ║
║    ┌─────────────────────┐                 ┌─────────────────────┐             ║
║    │                     │                 │                     │             ║
║    │   ┌─────────────┐   │                 │   ┌─────────────┐   │             ║
║    │   │  Feed-Fwd   │   │                 │   │  Feed-Fwd   │   │             ║
║    │   │  Network    │   │                 │   │  Network    │   │             ║
║    │   └──────┬──────┘   │                 │   └──────┬──────┘   │             ║
║    │       Add & Norm    │                 │       Add & Norm    │             ║
║    │   ───────┼───────   │                 │   ───────┼───────   │             ║
║    │          │          │                 │          │          │             ║
║    │   ┌──────┴──────┐   │                 │   ┌──────┴──────┐   │             ║
║    │   │ Multi-Head  │   │    K, V         │   │ Multi-Head  │   │             ║
║    │   │ Self-Attn   │   │──────────────▶  │   │ Cross-Attn  │   │             ║
║    │   └──────┬──────┘   │                 │   └──────┬──────┘   │             ║
║    │       Add & Norm    │                 │       Add & Norm    │             ║
║    │   ───────┼───────   │                 │   ───────┼───────   │             ║
║    │          │          │                 │          │          │             ║
║    └──────────┼──────────┘                 │   ┌──────┴──────┐   │             ║
║               │                            │   │ Masked      │   │             ║
║               │                            │   │ Multi-Head  │   │             ║
║               │                            │   │ Self-Attn   │   │             ║
║               │                            │   └──────┬──────┘   │             ║
║               │                            │       Add & Norm    │             ║
║               │                            │   ───────┼───────   │             ║
║               │                            └──────────┼──────────┘             ║
║               │                                       │                        ║
║    ┌──────────┴──────────┐                 ┌──────────┴──────────┐             ║
║    │ Positional Encoding │                 │ Positional Encoding │             ║
║    └──────────┬──────────┘                 └──────────┬──────────┘             ║
║    ┌──────────┴──────────┐                 ┌──────────┴──────────┐             ║
║    │  Input Embedding    │                 │  Output Embedding   │             ║
║    └──────────┬──────────┘                 └──────────┬──────────┘             ║
║               │                                       │                        ║
║         ┌─────┴─────┐                          ┌─────┴─────┐                  ║
║         │  Inputs   │                          │  Outputs  │                  ║
║         │ (Source)  │                          │ (Target)  │                  ║
║         └───────────┘                          └───────────┘                  ║
║                                                                                ║
║    ┌────────────────────────────────────────────────────────────────────┐       ║
║    │  SCALED DOT-PRODUCT ATTENTION                                     │       ║
║    │                                                                    │       ║
║    │                    Q · Kᵀ                                          │       ║
║    │  Attention(Q,K,V) = softmax( ─────── ) · V                        │       ║
║    │                               √d_k                                │       ║
║    │                                                                    │       ║
║    │  MULTI-HEAD ATTENTION                                              │       ║
║    │                                                                    │       ║
║    │  MultiHead(Q,K,V) = Concat(head₁, ..., headₕ) · Wᴼ              │       ║
║    │  where headᵢ = Attention(Q·Wᵢᵠ, K·Wᵢᴷ, V·Wᵢⱽ)                 │       ║
║    └────────────────────────────────────────────────────────────────────┘       ║
╚══════════════════════════════════════════════════════════════════════════════════╝
```

---

## Prerequisites

| Prerequisite | Where Covered |
|---|---|
| Linear algebra (matrix multiplication, projections) | [1.3 — Linear Algebra](../../01-Foundations/1.3-Linear-Algebra/README.md) |
| Calculus & backpropagation | [1.4 — Calculus](../../01-Foundations/1.4-Calculus/README.md) |
| Neural network fundamentals | [3.1 — Neural Networks Fundamentals](../3.1-Neural-Networks-Fundamentals/README.md) |
| Recurrent neural networks (context) | [3.3 — RNN](../3.3-RNN/README.md) |
| Python & PyTorch / TensorFlow basics | Assumed |

---

## Table of Contents

### Unit 1 · Attention Mechanism
> 📂 `01-Attention-Mechanism/`

| # | Lesson | Topic |
|---|--------|-------|
| 1 | [01-attention-intuition.md](01-Attention-Mechanism/01-attention-intuition.md) | Why attention? Human visual attention analogy |
| 2 | [02-soft-vs-hard-attention.md](01-Attention-Mechanism/02-soft-vs-hard-attention.md) | Soft vs hard attention comparison |
| 3 | [03-attention-weights.md](01-Attention-Mechanism/03-attention-weights.md) | Alignment scores and attention distribution |
| 4 | [04-query-key-value.md](01-Attention-Mechanism/04-query-key-value.md) | QKV abstraction and weight matrices |
| 5 | [05-attention-score-computation.md](01-Attention-Mechanism/05-attention-score-computation.md) | Dot-product, general, and concat scoring |
| 6 | [06-scaled-dot-product-attention.md](01-Attention-Mechanism/06-scaled-dot-product-attention.md) | Full scaled dot-product formula |
| 7 | [07-additive-attention.md](01-Attention-Mechanism/07-additive-attention.md) | Bahdanau attention mechanism |

---

### Unit 2 · Self-Attention
> 📂 `02-Self-Attention/`

| # | Lesson | Topic |
|---|--------|-------|
| 1 | [01-self-attention-concept.md](02-Self-Attention/01-self-attention-concept.md) | Self-attention concept and intuition |
| 2 | [02-computing-self-attention.md](02-Self-Attention/02-computing-self-attention.md) | Step-by-step computation walkthrough |
| 3 | [03-why-self-attention-works.md](02-Self-Attention/03-why-self-attention-works.md) | Advantages over RNN/CNN architectures |
| 4 | [04-positional-information-problem.md](02-Self-Attention/04-positional-information-problem.md) | Permutation invariance problem |

---

### Unit 3 · Multi-Head Attention
> 📂 `03-Multi-Head-Attention/`

| # | Lesson | Topic |
|---|--------|-------|
| 1 | [01-why-multiple-heads.md](03-Multi-Head-Attention/01-why-multiple-heads.md) | Motivation for multiple attention heads |
| 2 | [02-multi-head-computation.md](03-Multi-Head-Attention/02-multi-head-computation.md) | Full multi-head computation pipeline |
| 3 | [03-head-concatenation.md](03-Multi-Head-Attention/03-head-concatenation.md) | Concatenation and output projection |
| 4 | [04-parameter-efficiency.md](03-Multi-Head-Attention/04-parameter-efficiency.md) | Parameter count analysis |

---

### Unit 4 · Transformer Architecture
> 📂 `04-Transformer-Architecture/`

| # | Lesson | Topic |
|---|--------|-------|
| 1 | [01-attention-is-all-you-need.md](04-Transformer-Architecture/01-attention-is-all-you-need.md) | Original paper overview and key contributions |
| 2 | [02-encoder-architecture.md](04-Transformer-Architecture/02-encoder-architecture.md) | Encoder stack deep dive |
| 3 | [03-decoder-architecture.md](04-Transformer-Architecture/03-decoder-architecture.md) | Decoder stack deep dive |
| 4 | [04-encoder-decoder-attention.md](04-Transformer-Architecture/04-encoder-decoder-attention.md) | Cross-attention between encoder and decoder |
| 5 | [05-layer-normalization.md](04-Transformer-Architecture/05-layer-normalization.md) | Layer normalization theory and placement |
| 6 | [06-residual-connections.md](04-Transformer-Architecture/06-residual-connections.md) | Skip / residual connections |
| 7 | [07-feed-forward-networks.md](04-Transformer-Architecture/07-feed-forward-networks.md) | Position-wise feed-forward networks |

---

### Unit 5 · Positional Encoding
> 📂 `05-Positional-Encoding/`

| # | Lesson | Topic |
|---|--------|-------|
| 1 | [01-why-positional-encoding.md](05-Positional-Encoding/01-why-positional-encoding.md) | Motivation: why position matters |
| 2 | [02-sinusoidal-encoding.md](05-Positional-Encoding/02-sinusoidal-encoding.md) | Sinusoidal formulas and visualization |
| 3 | [03-learned-positional-embeddings.md](05-Positional-Encoding/03-learned-positional-embeddings.md) | Learned positional embeddings |
| 4 | [04-relative-positional-encoding.md](05-Positional-Encoding/04-relative-positional-encoding.md) | Relative positional encoding (Shaw et al.) |
| 5 | [05-rotary-positional-encoding.md](05-Positional-Encoding/05-rotary-positional-encoding.md) | Rotary Position Embedding (RoPE) |

---

### Unit 6 · Transformer Training
> 📂 `06-Transformer-Training/`

| # | Lesson | Topic |
|---|--------|-------|
| 1 | [01-masked-self-attention.md](06-Transformer-Training/01-masked-self-attention.md) | Causal masking for autoregressive decoding |
| 2 | [02-label-smoothing.md](06-Transformer-Training/02-label-smoothing.md) | Label smoothing regularization |
| 3 | [03-warmup-schedules.md](06-Transformer-Training/03-warmup-schedules.md) | Learning rate warmup and decay schedules |
| 4 | [04-training-tricks.md](06-Transformer-Training/04-training-tricks.md) | Practical training tricks and recipes |
| 5 | [05-inference-optimization.md](06-Transformer-Training/05-inference-optimization.md) | KV-cache, quantization, and inference speedups |

---

### Unit 7 · BERT and Encoder Models
> 📂 `07-BERT-and-Encoder-Models/`

| # | Lesson | Topic |
|---|--------|-------|
| 1 | [01-bert-architecture.md](07-BERT-and-Encoder-Models/01-bert-architecture.md) | BERT architecture and design choices |
| 2 | [02-pretraining-objectives.md](07-BERT-and-Encoder-Models/02-pretraining-objectives.md) | Masked Language Modeling (MLM) & Next Sentence Prediction (NSP) |
| 3 | [03-fine-tuning-bert.md](07-BERT-and-Encoder-Models/03-fine-tuning-bert.md) | Fine-tuning BERT for downstream tasks |
| 4 | [04-bert-variants.md](07-BERT-and-Encoder-Models/04-bert-variants.md) | RoBERTa, ALBERT, DistilBERT, and more |
| 5 | [05-bert-applications.md](07-BERT-and-Encoder-Models/05-bert-applications.md) | Practical NLP applications with BERT |

---

### Unit 8 · GPT and Decoder Models
> 📂 `08-GPT-and-Decoder-Models/`

| # | Lesson | Topic |
|---|--------|-------|
| 1 | [01-gpt-architecture.md](08-GPT-and-Decoder-Models/01-gpt-architecture.md) | GPT architecture and design philosophy |
| 2 | [02-autoregressive-language-modeling.md](08-GPT-and-Decoder-Models/02-autoregressive-language-modeling.md) | Autoregressive language modeling objective |
| 3 | [03-gpt2-and-gpt3.md](08-GPT-and-Decoder-Models/03-gpt2-and-gpt3.md) | GPT-2 and GPT-3: scaling and emergent abilities |
| 4 | [04-scaling-laws.md](08-GPT-and-Decoder-Models/04-scaling-laws.md) | Scaling laws (Kaplan et al., Chinchilla) |
| 5 | [05-in-context-learning.md](08-GPT-and-Decoder-Models/05-in-context-learning.md) | In-context learning and few-shot prompting |
| 6 | [06-prompt-engineering.md](08-GPT-and-Decoder-Models/06-prompt-engineering.md) | Prompt engineering techniques |

---

### Unit 9 · Encoder-Decoder Transformers
> 📂 `09-Encoder-Decoder-Transformers/`

| # | Lesson | Topic |
|---|--------|-------|
| 1 | [01-t5-architecture.md](09-Encoder-Decoder-Transformers/01-t5-architecture.md) | T5: Text-to-Text Transfer Transformer |
| 2 | [02-bart-architecture.md](09-Encoder-Decoder-Transformers/02-bart-architecture.md) | BART: Denoising sequence-to-sequence model |
| 3 | [03-seq2seq-tasks.md](09-Encoder-Decoder-Transformers/03-seq2seq-tasks.md) | Summarization, translation, and seq2seq tasks |
| 4 | [04-applications.md](09-Encoder-Decoder-Transformers/04-applications.md) | Practical applications and comparisons |

---

### Unit 10 · Modern Transformer Variants
> 📂 `10-Modern-Transformer-Variants/`

| # | Lesson | Topic |
|---|--------|-------|
| 1 | [01-vision-transformer.md](10-Modern-Transformer-Variants/01-vision-transformer.md) | Vision Transformer (ViT) |
| 2 | [02-swin-transformer.md](10-Modern-Transformer-Variants/02-swin-transformer.md) | Swin Transformer and hierarchical attention |
| 3 | [03-sparse-attention.md](10-Modern-Transformer-Variants/03-sparse-attention.md) | Sparse attention patterns (Longformer, BigBird) |
| 4 | [04-linear-attention.md](10-Modern-Transformer-Variants/04-linear-attention.md) | Linear attention and kernel methods |
| 5 | [05-flash-attention.md](10-Modern-Transformer-Variants/05-flash-attention.md) | FlashAttention: IO-aware exact attention |
| 6 | [06-efficient-transformers.md](10-Modern-Transformer-Variants/06-efficient-transformers.md) | Survey of efficient Transformer architectures |

---

### Unit 11 · Large Language Models
> 📂 `11-Large-Language-Models/`

| # | Lesson | Topic |
|---|--------|-------|
| 1 | [01-llm-landscape.md](11-Large-Language-Models/01-llm-landscape.md) | LLM landscape: open vs. closed models |
| 2 | [02-fine-tuning-llms.md](11-Large-Language-Models/02-fine-tuning-llms.md) | Full fine-tuning of large language models |
| 3 | [03-parameter-efficient-fine-tuning.md](11-Large-Language-Models/03-parameter-efficient-fine-tuning.md) | PEFT: LoRA, adapters, prefix tuning |
| 4 | [04-rlhf-basics.md](11-Large-Language-Models/04-rlhf-basics.md) | Reinforcement Learning from Human Feedback |
| 5 | [05-prompt-engineering-advanced.md](11-Large-Language-Models/05-prompt-engineering-advanced.md) | Advanced prompting: CoT, ToT, self-consistency |
| 6 | [06-rag.md](11-Large-Language-Models/06-rag.md) | Retrieval-Augmented Generation (RAG) |

---

## Suggested Study Plan

```
Week 1 ──────────────────────────────────────────────────────────────────
│  Unit 1: Attention Mechanism          Build intuition and math
│  Unit 2: Self-Attention               From cross- to self-attention
│
Week 2 ──────────────────────────────────────────────────────────────────
│  Unit 3: Multi-Head Attention         Parallel attention heads
│  Unit 4: Transformer Architecture     Full encoder-decoder assembly
│
Week 3 ──────────────────────────────────────────────────────────────────
│  Unit 5: Positional Encoding          Encoding sequential order
│  Unit 6: Transformer Training         Training recipes and tricks
│
Week 4 ──────────────────────────────────────────────────────────────────
│  Unit 7: BERT & Encoder Models        Bidirectional pre-training
│  Unit 8: GPT & Decoder Models         Autoregressive generation
│
Week 5 ──────────────────────────────────────────────────────────────────
│  Unit 9: Encoder-Decoder Models       T5, BART, seq2seq
│  Unit 10: Modern Variants             ViT, efficient attention
│
Week 6 ──────────────────────────────────────────────────────────────────
│  Unit 11: Large Language Models       LLMs, RLHF, RAG
│  🏁 Capstone: Build & fine-tune a Transformer end-to-end
```

**Recommended Order:** Follow the units sequentially (1 → 11). Each unit builds on concepts from the previous one. If you are already comfortable with attention basics, you may skip ahead to Unit 4.

| Your Background | Start At |
|---|---|
| New to attention | Unit 1 — start from scratch |
| Familiar with seq2seq + attention | Unit 2 — self-attention onward |
| Understand Transformers conceptually | Unit 6 — training, then model families |
| Want to work with LLMs right away | Unit 8 — GPT, then Unit 11 |

---

## References

### Foundational Papers

1. **Vaswani, A. et al.** (2017). *Attention Is All You Need*. NeurIPS. [arXiv:1706.03762](https://arxiv.org/abs/1706.03762)
2. **Bahdanau, D., Cho, K., & Bengio, Y.** (2015). *Neural Machine Translation by Jointly Learning to Align and Translate*. ICLR. [arXiv:1409.0473](https://arxiv.org/abs/1409.0473)
3. **Devlin, J. et al.** (2019). *BERT: Pre-training of Deep Bidirectional Transformers for Language Understanding*. NAACL. [arXiv:1810.04805](https://arxiv.org/abs/1810.04805)
4. **Radford, A. et al.** (2018). *Improving Language Understanding by Generative Pre-Training*. OpenAI.
5. **Brown, T. et al.** (2020). *Language Models are Few-Shot Learners*. NeurIPS. [arXiv:2005.14165](https://arxiv.org/abs/2005.14165)
6. **Raffel, C. et al.** (2020). *Exploring the Limits of Transfer Learning with a Unified Text-to-Text Transformer*. JMLR. [arXiv:1910.10683](https://arxiv.org/abs/1910.10683)
7. **Dosovitskiy, A. et al.** (2021). *An Image is Worth 16×16 Words*. ICLR. [arXiv:2010.11929](https://arxiv.org/abs/2010.11929)
8. **Dao, T. et al.** (2022). *FlashAttention: Fast and Memory-Efficient Exact Attention*. NeurIPS. [arXiv:2205.14135](https://arxiv.org/abs/2205.14135)
9. **Hu, E. et al.** (2022). *LoRA: Low-Rank Adaptation of Large Language Models*. ICLR. [arXiv:2106.09685](https://arxiv.org/abs/2106.09685)
10. **Ouyang, L. et al.** (2022). *Training Language Models to Follow Instructions with Human Feedback*. NeurIPS. [arXiv:2203.02155](https://arxiv.org/abs/2203.02155)

### Books & Surveys

- **Jurafsky, D. & Martin, J.H.** — *Speech and Language Processing* (3rd ed.), Chapters 9–10.
- **Tay, Y. et al.** (2022). *Efficient Transformers: A Survey*. ACM Computing Surveys. [arXiv:2009.06732](https://arxiv.org/abs/2009.06732)
- **Lin, T. et al.** (2022). *A Survey of Transformers*. AI Open. [arXiv:2106.04554](https://arxiv.org/abs/2106.04554)

### Online Resources

- [The Illustrated Transformer](https://jalammar.github.io/illustrated-transformer/) — Jay Alammar
- [The Annotated Transformer](https://nlp.seas.harvard.edu/annotated-transformer/) — Harvard NLP
- [Hugging Face Transformers Documentation](https://huggingface.co/docs/transformers)
- [Andrej Karpathy — Let's Build GPT](https://www.youtube.com/watch?v=kCc8FmEb1nY)

---

## Navigation

[← Previous: 3.3 — Recurrent Neural Networks](../3.3-RNN/README.md) | [Next: 4.1 — NLP →](../../04-NLP/4.1-NLP/README.md)

---

<div align="center">

*Part of the [AIML Course](../../README.md) — Module 3: Deep Learning*

</div>
