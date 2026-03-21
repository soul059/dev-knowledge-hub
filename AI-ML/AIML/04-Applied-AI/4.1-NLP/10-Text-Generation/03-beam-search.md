# Beam Search

## Overview

Beam search is a heuristic search algorithm that explores multiple candidate sequences in parallel to find high-probability outputs. It maintains a fixed number of "beams" (partial hypotheses) at each generation step, balancing between the exhaustiveness of breadth-first search and the efficiency of greedy search.

---

## Algorithm

```
Input: Language model, prompt, beam width B, max length L

Initialize: beams = [{tokens: [prompt], score: 0.0}]

For each step t = 1 to L:
  candidates = []
  For each beam in beams:
    Get P(next_token | beam.tokens) from model
    For each token w in vocabulary:
      new_score = beam.score + log P(w | beam.tokens)
      candidates.append({tokens: beam.tokens + [w], score: new_score})
  
  Sort candidates by score (descending)
  beams = top B candidates
  
  Move completed beams (ending with <EOS>) to finished list

Return: highest-scoring finished beam
```

---

## Visual Example

```
Prompt: "I like"    Beam width B = 2

Step 1:  "I like" → P(next token)
  "cats"  : log P = -0.8
  "dogs"  : log P = -1.0
  "pizza" : log P = -1.2
  "the"   : log P = -1.5
  
  Keep top 2:
  Beam A: "I like cats"   (score: -0.8)
  Beam B: "I like dogs"   (score: -1.0)

Step 2a: Expand "I like cats" → P(next | "I like cats")
  "and"    : -0.8 + (-0.5) = -1.3
  "because": -0.8 + (-0.9) = -1.7
  "."      : -0.8 + (-0.3) = -1.1  ← EOS

Step 2b: Expand "I like dogs" → P(next | "I like dogs")
  "and"    : -1.0 + (-0.4) = -1.4
  "a lot"  : -1.0 + (-0.6) = -1.6
  "."      : -1.0 + (-0.5) = -1.5  ← EOS

  All candidates: [-1.1, -1.3, -1.4, -1.5, -1.6, -1.7]
  Keep top 2:
  Beam A: "I like cats."    (score: -1.1, FINISHED)
  Beam B: "I like cats and" (score: -1.3, continue)

Step 3: Expand "I like cats and"...
  "dogs"   : -1.3 + (-0.6) = -1.9
  "birds"  : -1.3 + (-1.0) = -2.3

  Best finished: "I like cats." (score: -1.1)
  Continue if not at max length...
```

---

## Length Normalization

```
Problem: Beam search favors shorter sequences
  (more tokens = more negative log-probs accumulated)

Solution: Length-normalized scoring

  Standard score: score = Σ log P(wₜ | w₁..wₜ₋₁)
  
  Length-normalized: score_norm = score / |sequence|^α
  
  Where α is the length penalty:
    α = 0: no normalization (standard)
    α = 1: full normalization (divide by length)
    α = 0.6-0.8: typical sweet spot

  Google's NMT formula:
    lp(Y) = ((5 + |Y|) / (5 + 1))^α
    score = log P(Y) / lp(Y)
```

---

## Implementation

```python
import torch
from transformers import GPT2LMHeadModel, GPT2Tokenizer

model = GPT2LMHeadModel.from_pretrained("gpt2")
tokenizer = GPT2Tokenizer.from_pretrained("gpt2")

prompt = "The future of artificial intelligence"
input_ids = tokenizer.encode(prompt, return_tensors="pt")

# Basic beam search
output = model.generate(
    input_ids,
    max_length=50,
    num_beams=5,            # beam width
    early_stopping=True,
)
print("Basic:", tokenizer.decode(output[0], skip_special_tokens=True))

# With length penalty and n-gram blocking
output = model.generate(
    input_ids,
    max_length=50,
    num_beams=5,
    length_penalty=0.8,      # favor longer sequences slightly
    no_repeat_ngram_size=3,  # prevent 3-gram repetition
    early_stopping=True,
)
print("Tuned:", tokenizer.decode(output[0], skip_special_tokens=True))

# Diverse beam search (multiple diverse outputs)
outputs = model.generate(
    input_ids,
    max_length=50,
    num_beams=6,
    num_beam_groups=3,       # 3 diverse groups of 2 beams each
    diversity_penalty=0.5,
    num_return_sequences=3,
)
for i, out in enumerate(outputs):
    print(f"Beam {i+1}: {tokenizer.decode(out, skip_special_tokens=True)}")
```

---

## Beam Search Variants

| Variant | Modification | Benefit |
|---------|-------------|---------|
| Standard beam | Top-B candidates per step | Better than greedy |
| Length-normalized | Divide score by length^α | Fixes short-sequence bias |
| Diverse beam search | Penalty for similar beams | Multiple distinct outputs |
| Constrained beam | Must include certain tokens | Guided generation |
| Minimum Bayes Risk | Re-rank by expected utility | Better output selection |

---

## When to Use Beam Search

| Task | Recommended | Why |
|------|:-:|-----|
| Machine translation | ✓ | Need high-quality single output |
| Summarization | ✓ | Faithful, concise output |
| Image captioning | ✓ | Descriptive, accurate |
| Open-ended generation | ✗ | Too repetitive, lacks creativity |
| Dialogue / chat | ✗ | Generic, boring responses |
| Creative writing | ✗ | Predictable, safe outputs |

---

## Beam Search vs Sampling

```
Beam Search:                    Sampling (top-p):
  ✓ Higher-probability text      ✓ More diverse/creative
  ✓ Good for structured tasks    ✓ Natural, human-like
  ✗ Repetitive                   ✗ Occasional low-quality
  ✗ Generic/boring               ✗ Non-deterministic
  ✗ Slow (B× computation)        ✓ Fast (single path)
```

---

## Revision Questions

1. **How does beam search differ from greedy search?**
2. **Why does beam search favor shorter sequences and how is this fixed?**
3. **What is diverse beam search and when would you use it?**
4. **Why is beam search preferred for translation but not for chatbots?**
5. **What is the effect of increasing beam width?**

---

[Previous: 02-sampling-strategies.md](02-sampling-strategies.md) | [Next: 04-controllable-generation.md](04-controllable-generation.md)
