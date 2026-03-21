# BLEU Score

## Overview

**BLEU** (Bilingual Evaluation Understudy) is the most widely used automatic metric for evaluating machine translation quality. It compares n-gram overlaps between the machine translation (candidate) and one or more human reference translations. BLEU scores range from 0 to 1 (or 0–100 when expressed as percentages).

---

## The Intuition

```
Reference:  "The cat is on the mat"
Candidate1: "The cat is on the mat"        → Perfect match → BLEU ≈ 1.0
Candidate2: "The the the the the the"      → High unigram recall but bad → needs precision
Candidate3: "Le chat est sur le tapis"     → No English overlap → BLEU ≈ 0
```

---

## BLEU Formula

```
BLEU = BP × exp( Σ_{n=1}^{N} w_n × log(p_n) )

Where:
  p_n = modified precision for n-grams of size n
  w_n = weight for each n-gram (typically uniform: 1/N)
  N   = max n-gram size (usually 4)
  BP  = brevity penalty

Modified precision:
  p_n = (# clipped n-gram matches) / (# candidate n-grams)

  "Clipped" means each n-gram match count is capped at the
  max count in any reference translation.

Brevity Penalty:
  BP = 1                     if c > r
  BP = exp(1 - r/c)         if c ≤ r
  
  c = candidate length
  r = effective reference length (closest to candidate)
```

---

## Worked Example

```
Reference:  "the cat sat on the mat"
Candidate:  "the the cat on the mat"

Step 1: Unigram precision (p₁)
  Candidate unigrams: the(3), cat(1), on(1), mat(1) → total = 6
  Matches (clipped):
    "the" appears 3× in candidate, but only 2× in reference → clip to 2
    "cat" → 1 match
    "on"  → 1 match
    "mat" → 1 match
  Clipped matches = 2 + 1 + 1 + 1 = 5
  p₁ = 5/6 = 0.833

Step 2: Bigram precision (p₂)
  Candidate bigrams: "the the"(1), "the cat"(1), "cat on"(1), 
                     "on the"(1), "the mat"(1) → total = 5
  Reference bigrams: "the cat"(1), "cat sat"(1), "sat on"(1),
                     "on the"(1), "the mat"(1)
  Matches: "the cat"(1), "on the"(1), "the mat"(1) = 3
  p₂ = 3/5 = 0.600

Step 3: Brevity Penalty
  c = 6, r = 6 → c = r, so BP = 1.0

Step 4: BLEU (using n=1,2 for simplicity, equal weights)
  BLEU = 1.0 × exp(0.5 × log(0.833) + 0.5 × log(0.600))
       = exp(0.5 × (-0.183) + 0.5 × (-0.511))
       = exp(-0.347)
       = 0.707 → 70.7%
```

---

## Python Implementation

```python
from nltk.translate.bleu_score import sentence_bleu, corpus_bleu
from nltk.translate.bleu_score import SmoothingFunction

# Single sentence BLEU
reference = [["the", "cat", "sat", "on", "the", "mat"]]
candidate = ["the", "the", "cat", "on", "the", "mat"]

score = sentence_bleu(reference, candidate)
print(f"BLEU-4: {score:.4f}")  # BLEU-4 (default: n=1..4)

# With smoothing (handles 0 n-gram matches)
smooth = SmoothingFunction().method1
score_smooth = sentence_bleu(reference, candidate, smoothing_function=smooth)
print(f"BLEU-4 (smoothed): {score_smooth:.4f}")

# Corpus-level BLEU
references_corpus = [[["this", "is", "a", "test"]], 
                      [["another", "sentence"]]]
candidates_corpus = [["this", "is", "a", "test"],
                     ["another", "sentence"]]
corpus_score = corpus_bleu(references_corpus, candidates_corpus)
print(f"Corpus BLEU: {corpus_score:.4f}")
```

---

## BLEU Score Interpretation

| BLEU Score | Quality |
|:---:|---------|
| < 10 | Almost useless |
| 10–19 | Hard to understand |
| 20–29 | Gist is clear |
| 30–39 | Understandable to good |
| 40–49 | High quality |
| 50–59 | Very high quality |
| 60+ | Better than some human translators |

---

## Limitations & Alternatives

| Metric | Strength | Weakness |
|--------|----------|----------|
| **BLEU** | Fast, widely used, corpus-level | Ignores meaning, synonyms, word order |
| **METEOR** | Synonyms, stems, word order | Slower, language-specific |
| **TER** | Edit distance (human intuition) | Doesn't reward good translations |
| **chrF** | Character n-grams, morphology | Less interpretable |
| **BERTScore** | Semantic similarity (embeddings) | Computationally expensive |
| **COMET** | Trained on human judgments | Requires GPU, model-dependent |

---

## Revision Questions

1. **Why is "clipped" precision used instead of raw precision?**
2. **What is the brevity penalty and why is it needed?**
3. **Walk through computing BLEU-2 on a short example.**
4. **What are the main limitations of BLEU as a translation metric?**
5. **When would you prefer BERTScore or COMET over BLEU?**

---

[Previous: 04-transformer-translation.md](04-transformer-translation.md) | [Next: 06-multilingual-models.md](06-multilingual-models.md)
