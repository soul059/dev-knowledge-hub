# Sampling Strategies

## Overview

Sampling strategies determine HOW tokens are selected from the probability distribution during text generation. While greedy always picks the max and pure sampling picks randomly according to probabilities, strategies like **top-k** and **top-p (nucleus) sampling** restrict the sampling pool to balance quality and diversity.

---

## Top-k Sampling

```
Strategy: Only sample from the top k most probable tokens

  Full distribution (vocab = 50,000 tokens):
    "beautiful" : 0.30
    "sunny"     : 0.25
    "cold"      : 0.15
    "nice"      : 0.10
    "warm"      : 0.05
    "rainy"     : 0.03
    ... 49,994 more tokens with tiny probabilities ...

  Top-k = 5:
    Keep only top 5, re-normalize:
    "beautiful" : 0.30/0.85 = 0.353
    "sunny"     : 0.25/0.85 = 0.294
    "cold"      : 0.15/0.85 = 0.176
    "nice"      : 0.10/0.85 = 0.118
    "warm"      : 0.05/0.85 = 0.059
    
    Sample from these 5 tokens

  Problem with fixed k:
    - If distribution is peaked: k=50 includes garbage tokens
    - If distribution is flat: k=5 misses valid options
    → k should be ADAPTIVE → that's nucleus sampling
```

---

## Top-p (Nucleus) Sampling

```
Strategy: Sample from the smallest set of tokens whose 
          cumulative probability ≥ p

  Sorted distribution:
    Token        Prob    Cumulative
    "beautiful"  0.30    0.30
    "sunny"      0.25    0.55
    "cold"       0.15    0.70
    "nice"       0.10    0.80  ← cumsum ≥ 0.8, stop here (for p=0.8)
    "warm"       0.05    0.85
    "rainy"      0.03    0.88
    ...

  Top-p = 0.8:
    Keep: {"beautiful", "sunny", "cold", "nice"}
    Re-normalize and sample from these 4 tokens

  Top-p = 0.95: (more diverse)
    Keep more tokens until cumsum ≥ 0.95

  Why p is better than k:
    - Peaked distribution (e.g., after "New"):
      "York" has P=0.9 → nucleus = just {"York"} → safe
    - Flat distribution (e.g., after "I like"):
      Many valid next words → nucleus is larger → more diversity
    - Adapts automatically!
```

---

## Combining Strategies

```
Best practice: Combine temperature + top-p + top-k

  Step 1: Apply temperature to logits
    P(wᵢ) = softmax(zᵢ / T)

  Step 2: Apply top-k filter
    Keep only top k tokens

  Step 3: Apply top-p filter
    From remaining, keep until cumsum ≥ p

  Step 4: Re-normalize and sample

Common configurations:
  ┌────────────────────┬───────┬───────┬───────┐
  │ Use Case           │ Temp  │ Top-k │ Top-p │
  ├────────────────────┼───────┼───────┼───────┤
  │ Code generation    │ 0.2   │ 40    │ 0.90  │
  │ Factual QA         │ 0.3   │ 10    │ 0.85  │
  │ Conversation       │ 0.7   │ 50    │ 0.92  │
  │ Creative writing   │ 0.9   │ 100   │ 0.95  │
  │ Poetry/brainstorm  │ 1.2   │ 200   │ 0.98  │
  └────────────────────┴───────┴───────┴───────┘
```

---

## Implementation

```python
from transformers import GPT2LMHeadModel, GPT2Tokenizer

model = GPT2LMHeadModel.from_pretrained("gpt2")
tokenizer = GPT2Tokenizer.from_pretrained("gpt2")

prompt = "In the future, robots will"
input_ids = tokenizer.encode(prompt, return_tensors="pt")

# Top-k sampling
topk_output = model.generate(
    input_ids, max_length=60, do_sample=True,
    top_k=50, temperature=0.7
)
print("Top-k:", tokenizer.decode(topk_output[0]))

# Top-p (nucleus) sampling
topp_output = model.generate(
    input_ids, max_length=60, do_sample=True,
    top_p=0.92, temperature=0.7
)
print("Top-p:", tokenizer.decode(topp_output[0]))

# Combined
combined = model.generate(
    input_ids, max_length=60, do_sample=True,
    top_k=50, top_p=0.92, temperature=0.8,
    repetition_penalty=1.2,        # penalize repeated tokens
    no_repeat_ngram_size=3,        # no 3-gram repetition
)
print("Combined:", tokenizer.decode(combined[0]))
```

---

## Repetition Penalties

```
Problem: LMs tend to repeat themselves

Solutions:
  1. n-gram blocking:
     If trigram "the cat sat" appeared → block "sat" after "the cat"

  2. Repetition penalty:
     For previously generated token w:
       P'(w) = P(w) / penalty   (penalty > 1 reduces probability)

  3. Frequency penalty:
     P'(w) = P(w) - α × count(w)   (linear reduction by frequency)

  4. Presence penalty:
     P'(w) = P(w) - β × 1[w appeared]   (fixed penalty if seen before)
```

---

## Comparison

| Strategy | Adapts to Distribution? | Quality | Diversity | Repetition Risk |
|----------|:---:|:---:|:---:|:---:|
| Pure sampling | N/A | Low (noisy) | Very high | Low |
| Top-k | ✗ (fixed k) | Good | Medium | Medium |
| Top-p (nucleus) | ✓ (adaptive) | Best | Good | Medium |
| Top-k + Top-p | ✓ | Best | Good | Low |
| + Repetition penalty | ✓ | Best | Best | Very low |

---

## Revision Questions

1. **How does top-k sampling work and what is its main limitation?**
2. **Why is nucleus (top-p) sampling considered better than top-k?**
3. **Give an example where top-k with k=5 would be too restrictive.**
4. **How do repetition penalties help with text generation quality?**
5. **What combination of strategies would you use for code generation vs creative writing?**

---

[Previous: 01-lm-decoding.md](01-lm-decoding.md) | [Next: 03-beam-search.md](03-beam-search.md)
