# Language Model Decoding

## Overview

Language model decoding is the process of generating text from a trained language model. Given a prompt or prefix, the model produces a probability distribution over the vocabulary at each step, and a **decoding strategy** determines which token to select. The choice of decoding method dramatically affects output quality, diversity, and coherence.

---

## Autoregressive Generation

```
Language models generate text one token at a time, left to right:

  P(w₁, w₂, ..., wₙ) = Π P(wₜ | w₁, ..., wₜ₋₁)

  Step 1: P(w₁ | prompt)         → select w₁
  Step 2: P(w₂ | prompt, w₁)    → select w₂
  Step 3: P(w₃ | prompt, w₁, w₂) → select w₃
  ...continue until <EOS> or max length

Example:
  Prompt: "The weather today is"
  
  Step 1: P(beautiful|...) = 0.3, P(sunny|...) = 0.25, P(cold|...) = 0.15, ...
          → How to choose? That's the decoding strategy!
```

---

## Greedy Decoding

```
Strategy: Always pick the highest probability token

  P(w | context):
    "beautiful" : 0.30  ← pick this
    "sunny"     : 0.25
    "cold"      : 0.15
    "nice"      : 0.10
    ...

  Pros: Fast, deterministic
  Cons: Repetitive, boring, misses globally optimal sequences

  Example output:
    "The weather today is beautiful. The weather is beautiful. 
     The weather is beautiful."  ← repetition loop!
```

---

## Beam Search

```
Strategy: Keep top-k (beam width) candidates at each step

  Beam width B = 3

  Step 1:  "The weather today is"
           → beautiful (0.30)
           → sunny     (0.25)
           → cold      (0.15)

  Step 2 (expand each beam):
           → beautiful and     (0.30 × 0.20 = 0.060)
           → beautiful today   (0.30 × 0.05 = 0.015)
           → sunny and         (0.25 × 0.22 = 0.055)
           → sunny outside     (0.25 × 0.18 = 0.045)
           → cold and          (0.15 × 0.25 = 0.038)
           → cold outside      (0.15 × 0.20 = 0.030)

  Keep top 3:
           → beautiful and     (0.060)
           → sunny and         (0.055)
           → sunny outside     (0.045)

  Continue until <EOS>...

  Final: Pick beam with highest total log-probability
  
  Pros: Better than greedy, finds higher-probability sequences
  Cons: Still repetitive, computationally expensive (B× slower)
```

---

## Temperature Sampling

```
Strategy: Sample from the distribution, controlling randomness

  logits z = [2.0, 1.5, 0.8, 0.3, -0.5, ...]
  
  Standard softmax (T=1.0):
    P(wᵢ) = exp(zᵢ) / Σ exp(zⱼ)
  
  With temperature T:
    P(wᵢ) = exp(zᵢ/T) / Σ exp(zⱼ/T)

  T < 1.0 → Sharper distribution (more confident, less random)
  T = 1.0 → Original distribution
  T > 1.0 → Flatter distribution (more random, more creative)
  T → 0   → Greedy decoding
  T → ∞   → Uniform random

  Example:
    Token:    beautiful  sunny  cold  rainy  terrible
    T=0.5:    0.65      0.22   0.08  0.04   0.01    (confident)
    T=1.0:    0.30      0.25   0.15  0.10   0.05    (balanced)
    T=2.0:    0.22      0.21   0.18  0.16   0.12    (creative)
```

---

## Implementation

```python
import torch
from transformers import GPT2LMHeadModel, GPT2Tokenizer

model = GPT2LMHeadModel.from_pretrained("gpt2")
tokenizer = GPT2Tokenizer.from_pretrained("gpt2")

prompt = "Artificial intelligence will"
input_ids = tokenizer.encode(prompt, return_tensors="pt")

# Greedy
greedy = model.generate(input_ids, max_length=50, do_sample=False)
print("Greedy:", tokenizer.decode(greedy[0]))

# Beam Search
beam = model.generate(input_ids, max_length=50, num_beams=5, 
                       no_repeat_ngram_size=2)
print("Beam:", tokenizer.decode(beam[0]))

# Temperature Sampling
temp = model.generate(input_ids, max_length=50, do_sample=True,
                       temperature=0.7)
print("Temp=0.7:", tokenizer.decode(temp[0]))

# High temperature (creative)
creative = model.generate(input_ids, max_length=50, do_sample=True,
                           temperature=1.5)
print("Temp=1.5:", tokenizer.decode(creative[0]))
```

---

## Decoding Methods Comparison

| Method | Deterministic | Quality | Diversity | Speed | Use Case |
|--------|:---:|:---:|:---:|:---:|---------|
| Greedy | ✓ | Low | Very low | Fastest | Quick baseline |
| Beam search | ✓ | Medium | Low | Slow | Translation, summarization |
| Temperature | ✗ | Tunable | Tunable | Fast | Creative writing |
| Top-k | ✗ | Good | Good | Fast | Open-ended generation |
| Top-p (nucleus) | ✗ | Best | Best | Fast | Most applications |

---

## Revision Questions

1. **What is autoregressive generation?**
2. **Why does greedy decoding often produce repetitive text?**
3. **How does beam search improve over greedy decoding?**
4. **What does temperature control in text generation?**
5. **What happens when temperature approaches 0 vs infinity?**

---

[Previous: ../09-Question-Answering/05-knowledge-based-qa.md](../09-Question-Answering/05-knowledge-based-qa.md) | [Next: 02-sampling-strategies.md](02-sampling-strategies.md)
