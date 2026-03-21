# Beam Search Decoding

> **Unit 7, Chapter 3** — Greedy vs beam search, beam width, scoring, length normalization, and implementation.

---

## 📋 Overview

During inference, the decoder must choose which token to output at each step. **Greedy decoding** always picks the most likely token, but this can miss globally better sequences. **Beam search** explores multiple candidates simultaneously to find higher-quality outputs.

---

## 📐 Greedy vs Beam Search

```
GREEDY DECODING:
   At each step, pick argmax P(y_t | y_{<t})
   
   Step 1: P("The"=0.4, "A"=0.35, "One"=0.25) → "The"
   Step 2: P("cat"=0.5, "dog"=0.3, "bird"=0.2) → "cat"
   Step 3: P("sat"=0.6, "ran"=0.3, "ate"=0.1) → "sat"
   Result: "The cat sat"  (score: 0.4 × 0.5 × 0.6 = 0.12)

   Problem: Local best ≠ Global best!

BEAM SEARCH (beam_width=2):
   Keep top-2 candidates at each step

   Step 1: Top-2: "The"(0.4), "A"(0.35)
   
   Step 2: Expand each:
     "The cat"(0.4×0.5=0.20), "The dog"(0.4×0.3=0.12)
     "A cat"(0.35×0.45=0.16), "A dog"(0.35×0.4=0.14)
     Top-2: "The cat"(0.20), "A cat"(0.16)
   
   Step 3: Expand each:
     "The cat sat"(0.20×0.6=0.12), "The cat ran"(0.20×0.3=0.06)
     "A cat plays"(0.16×0.5=0.08), "A cat sat"(0.16×0.4=0.064)
     Top-2: "The cat sat"(0.12), "A cat plays"(0.08)
   
   Result: "The cat sat" (but we explored alternatives!)
```

---

## 🧮 Beam Search Algorithm

```
┌────────────────────────────────────────────────────────────────┐
│                  BEAM SEARCH ALGORITHM                         │
│                                                                │
│  Input: Decoder model, beam_width B, max_length T              │
│                                                                │
│  Initialize:                                                   │
│    beams = [(score=0, tokens=[<sos>], hidden_state)]          │
│                                                                │
│  For t = 1 to T:                                              │
│    all_candidates = []                                        │
│    For each beam in beams:                                    │
│      probs = decoder(beam.tokens[-1], beam.hidden)           │
│      For each token in top-B tokens:                         │
│        new_score = beam.score + log(prob[token])             │
│        all_candidates.append((new_score, tokens+[token]))    │
│    beams = top-B candidates from all_candidates              │
│    Remove any beams that generated <eos> → completed         │
│                                                                │
│  Return: highest-scoring completed beam                       │
│                                                                │
│  Note: Use LOG probabilities to avoid numerical underflow     │
│    log P(y₁,...,y_T) = Σ log P(y_t | y_{<t})                │
└────────────────────────────────────────────────────────────────┘
```

---

## 📊 Beam Width Trade-offs

```
┌──────────┬─────────────────┬──────────────────┬─────────────┐
│ Width B  │ Quality         │ Speed            │ Use Case    │
├──────────┼─────────────────┼──────────────────┼─────────────┤
│ B = 1    │ Greedy (lowest) │ Fastest          │ Real-time   │
│ B = 3-5  │ Good            │ Fast             │ Production  │
│ B = 10   │ Very good       │ Moderate         │ Translation │
│ B = 20+  │ Marginal gains  │ Slow             │ Research    │
│ B = ∞    │ Optimal (exact) │ Exponential      │ Impossible  │
└──────────┴─────────────────┴──────────────────┴─────────────┘

Computation: O(B × V × T)  where V = vocabulary size
Memory: O(B × T)

Typical recommendation: B = 4-10 for most tasks
```

---

## 📐 Length Normalization

```
Problem: Beam search prefers SHORTER sequences!

  "cat" → log(0.4) = -0.916
  "the big cat" → log(0.3) + log(0.4) + log(0.5) = -2.813

  Shorter sequence has higher score → beam search biased!

Solution: Normalize by length

  Score_normalized = (1/T^α) × Σ log P(y_t | y_{<t})
  
  where α ∈ [0, 1] is the length penalty:
    α = 0: no normalization (original)
    α = 1: full normalization (per-token average)
    α = 0.6: typical choice (mild penalty)
```

---

## 💻 Implementation

```python
import torch
import torch.nn as nn
import torch.nn.functional as F
from dataclasses import dataclass
from typing import List

@dataclass
class Beam:
    tokens: List[int]
    score: float
    hidden: tuple
    done: bool = False

def beam_search(decoder, encoder_output, beam_width=5, max_len=50,
                sos_idx=1, eos_idx=2, length_penalty=0.6):
    """
    Beam search decoding for seq2seq models
    """
    hidden, cell = encoder_output
    
    # Initialize beams
    beams = [Beam(
        tokens=[sos_idx],
        score=0.0,
        hidden=(hidden, cell)
    )]
    
    completed = []
    
    for _ in range(max_len):
        if not beams:
            break
            
        all_candidates = []
        
        for beam in beams:
            if beam.done:
                completed.append(beam)
                continue
            
            # Decode one step
            input_token = torch.tensor([beam.tokens[-1]])
            h, c = beam.hidden
            prediction, h_new, c_new = decoder(input_token, h, c)
            log_probs = F.log_softmax(prediction.squeeze(), dim=0)
            
            # Get top-B tokens
            top_log_probs, top_indices = log_probs.topk(beam_width)
            
            for log_prob, idx in zip(top_log_probs, top_indices):
                new_beam = Beam(
                    tokens=beam.tokens + [idx.item()],
                    score=beam.score + log_prob.item(),
                    hidden=(h_new, c_new),
                    done=(idx.item() == eos_idx)
                )
                all_candidates.append(new_beam)
        
        # Keep top-B beams
        all_candidates.sort(key=lambda b: b.score, reverse=True)
        beams = [b for b in all_candidates[:beam_width] if not b.done]
        completed.extend([b for b in all_candidates[:beam_width] if b.done])
    
    # Apply length normalization and return best
    if not completed:
        completed = beams
    
    for beam in completed:
        beam.score = beam.score / (len(beam.tokens) ** length_penalty)
    
    completed.sort(key=lambda b: b.score, reverse=True)
    return completed[0].tokens

# Compare greedy vs beam search
print("=== Decoding Comparison ===")
print("Greedy: Always pick top-1 → fast but suboptimal")
print("Beam-5: Keep top-5 candidates → better quality")
```

---

## 📋 Summary Table

| Concept | Description |
|---------|-------------|
| Greedy | Pick argmax at each step — fast but myopic |
| Beam Search | Keep top-B candidates — explores alternatives |
| Beam Width | B — number of parallel hypotheses |
| Log Probability | Sum log probs to avoid underflow |
| Length Normalization | Divide by T^α to avoid short-sequence bias |
| Completed Beams | Hypotheses that generated \<eos\> |
| Typical B | 4-10 for production, higher for research |

---

## ❓ Revision Questions

1. **Why does greedy decoding sometimes produce worse sequences than beam search?**

2. **With beam_width=3 and vocabulary of 1000 words, how many candidates are evaluated at each step?**

3. **Why do we use log probabilities instead of raw probabilities in beam search?**

4. **Explain the length bias problem. How does length normalization address it?**

5. **What is the time complexity of beam search vs greedy decoding?**

6. **Can beam search guarantee finding the optimal (highest probability) sequence? Why or why not?**

---

## 🧭 Navigation

| Direction | Link |
|-----------|------|
| ⬅️ Previous | [Applications](02-applications.md) |
| ➡️ Next | [Limitations](04-limitations.md) |
| ⬆️ Unit Overview | [README](../README.md) |
