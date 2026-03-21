# Sequence-to-Sequence Tasks with Transformers

[← BART Architecture](02-bart-architecture.md) | [Applications →](04-applications.md)

---

## Overview

Sequence-to-sequence (seq2seq) tasks form the backbone of many practical NLP applications. In these tasks, a model receives a variable-length input sequence and must produce a variable-length output sequence. With the rise of encoder-decoder transformers like T5 and BART, seq2seq models have achieved state-of-the-art performance on machine translation, abstractive summarization, question generation, and data-to-text tasks. This chapter explores the major seq2seq tasks, their formulations, the decoding strategies used to generate outputs (especially beam search), and the evaluation metrics (BLEU, ROUGE, METEOR) that measure quality. We also compare T5 and BART across benchmark datasets and provide hands-on code for running translation and summarization with HuggingFace.

---

## 1. Core Seq2Seq Tasks

### 1.1 Task Taxonomy

```
┌──────────────────────────────────────────────────────────────────┐
│                  SEQUENCE-TO-SEQUENCE TASKS                      │
├──────────────────────┬───────────────────────────────────────────┤
│                      │                                           │
│  MACHINE TRANSLATION │  Source language → Target language         │
│                      │  "How are you?" → "Comment allez-vous?"   │
│                      │                                           │
│  TEXT SUMMARIZATION  │  Long document → Concise summary          │
│                      │  (Abstractive: generates new words)       │
│                      │                                           │
│  QUESTION GENERATION │  Context passage → Relevant question      │
│                      │  "The Eiffel Tower is..." → "What is...?" │
│                      │                                           │
│  DATA-TO-TEXT        │  Structured data → Natural language        │
│                      │  {name: "Café", rating: 5} → "Café is..." │
│                      │                                           │
│  PARAPHRASE GEN.     │  Sentence → Semantically equivalent text  │
│                      │  "It's raining" → "Rain is falling"       │
│                      │                                           │
│  GRAMMAR CORRECTION  │  Erroneous text → Corrected text          │
│                      │  "He go store" → "He goes to the store"   │
│                      │                                           │
└──────────────────────┴───────────────────────────────────────────┘
```

---

## 2. Machine Translation

### 2.1 Problem Formulation

Given source sentence `X = (x₁, x₂, ..., xₘ)` in language A, produce target sentence `Y = (y₁, y₂, ..., yₙ)` in language B.

```
P(Y | X) = ∏_{t=1}^{n} P(y_t | y_{<t}, X)
```

### 2.2 Encoder-Decoder Flow

```
Source: "The cat sits on the mat"
         │
    ┌────▼────┐
    │ ENCODER │  ← Bidirectional self-attention
    │ (6-12   │     over source tokens
    │ layers) │
    └────┬────┘
         │ encoder hidden states H = [h₁, h₂, ..., hₘ]
         │
    ┌────▼────┐
    │ DECODER │  ← Causal self-attention + cross-attention to H
    │ (6-12   │     generates one token at a time
    │ layers) │
    └────┬────┘
         │
    "Le chat est assis sur le tapis"
```

### 2.3 Key Challenges

| Challenge             | Description                                         | Solution                       |
|-----------------------|-----------------------------------------------------|--------------------------------|
| Word order changes    | SVO → SOV between languages                         | Attention mechanism            |
| Vocabulary mismatch   | Different morphology across languages               | Subword tokenization (BPE)     |
| Long-range dependencies | Pronouns referring to distant antecedents          | Multi-head self-attention      |
| Rare words            | Names, technical terms                              | Copy mechanism, BPE fallback   |

---

## 3. Text Summarization

### 3.1 Extractive vs. Abstractive

| Aspect            | Extractive                    | Abstractive                     |
|-------------------|-------------------------------|---------------------------------|
| Method            | Select existing sentences     | Generate new sentences          |
| Analogy           | "Copy & paste"                | "Understand & rewrite"          |
| Grammar           | Always grammatical            | May produce novel phrasings     |
| Flexibility       | Limited by source sentences   | Can combine information         |
| Models            | TextRank, BertSum             | BART, T5, Pegasus               |

### 3.2 Abstractive Summarization Flow

```
Input (500 words) → Tokenize → Encoder → Decoder (autoregressive) → Summary (50 words)
```

---

## 4. Question Generation

### 4.1 Task Definition

Given a context passage and optionally an answer span, generate a relevant question.

```
Context: "Albert Einstein developed the theory of relativity in 1905."
Answer:  "1905"
Output:  "When did Einstein develop the theory of relativity?"
```

### 4.2 T5 Formulation

```
Input:  "generate question: context: Albert Einstein developed the
         theory of relativity in 1905. answer: 1905"
Output: "When did Einstein develop the theory of relativity?"
```

---

## 5. Data-to-Text Generation

### 5.1 Structured Input → Natural Language

```
Input (structured):
  {
    "name": "The Eagle",
    "type": "restaurant",
    "food": "Italian",
    "rating": "5 out of 5",
    "area": "riverside"
  }

Output (natural language):
  "The Eagle is a top-rated Italian restaurant located in the
   riverside area with a perfect 5 out of 5 rating."
```

### 5.2 Linearization for Transformer Input

Structured data is linearized into a text sequence:

```
"name: The Eagle | type: restaurant | food: Italian | rating: 5 out of 5 | area: riverside"
```

This allows standard seq2seq models to process structured data without special architectures.

---

## 6. T5 vs BART: Benchmark Comparison

### 6.1 Performance on Key Benchmarks

| Task / Dataset       | T5-Large (770M) | BART-Large (400M) | Metric   |
|----------------------|-----------------|--------------------|----------|
| CNN/DailyMail (summ.)| R1: 42.50      | R1: 44.16          | ROUGE-1  |
| XSum (summ.)         | R1: 40.90       | R1: 45.14          | ROUGE-1  |
| WMT En→De (transl.) | 28.6            | 27.8               | BLEU     |
| SQuAD 2.0 (QA)      | 88.1            | 88.8               | F1       |
| MNLI (NLI)           | 89.9            | 89.9               | Accuracy |

BART excels at summarization; T5 is competitive across all tasks and scales better.

### 6.2 Architectural Differences Affecting Performance

| Feature                 | T5                          | BART                         |
|-------------------------|-----------------------------|------------------------------|
| Positional encoding     | Relative bias (learned)     | Absolute (learned)           |
| Layer norm              | Pre-norm                    | Post-norm (BART-base)        |
| Pre-training objective  | Span corruption             | Text infilling + others      |
| Pre-training data       | C4 (~750 GB)                | Books + Wikipedia (~160 GB)  |
| Task specification      | Text prefix                 | No prefix (fine-tune per task)|
| Vocab size              | 32,000 (SentencePiece)      | 50,265 (BPE)                 |

---

## 7. Decoding Strategies

### 7.1 Greedy Decoding

Select the highest-probability token at each step:

```
y_t = argmax P(y | y_{<t}, X)
```

**Pros**: Fast, simple.
**Cons**: Misses globally better sequences.

### 7.2 Beam Search

Maintain top-k candidates (beams) at each step:

```
Beam Search Algorithm:
━━━━━━━━━━━━━━━━━━━━
1. Initialize beam with <BOS> token
2. For each step t = 1, 2, ..., T_max:
   a. For each beam candidate b:
      - Compute P(y_t | y_{<t}^b, X) for all vocab tokens
      - Score = log P(y_1^b) + ... + log P(y_t | ...)
   b. Keep top-k candidates by cumulative score
3. Return highest-scoring complete sequence
```

### 7.3 Beam Search Example (k=2)

```
Step 0: [<BOS>]

Step 1:
  <BOS> → "The" (score: -0.3)    ← Beam 1
  <BOS> → "A"   (score: -0.5)    ← Beam 2

Step 2:
  "The" → "cat"    (score: -0.3 + -0.4 = -0.7)   ← Beam 1 ✓
  "The" → "dog"    (score: -0.3 + -0.6 = -0.9)
  "A"   → "small"  (score: -0.5 + -0.3 = -0.8)   ← Beam 2 ✓
  "A"   → "big"    (score: -0.5 + -0.5 = -1.0)

Step 3:
  "The cat"  → "sits" (score: -0.7 + -0.2 = -0.9)  ← Beam 1 ✓
  "A small"  → "cat"  (score: -0.8 + -0.2 = -1.0)  ← Beam 2 ✓

Final: "The cat sits" (best score: -0.9)
```

### 7.4 Length Penalty

To prevent short sequences from dominating:

```
score(Y) = (1 / |Y|^α) × ∑_{t=1}^{|Y|} log P(y_t | y_{<t}, X)

Where α is the length penalty factor:
  α = 0.0  → no penalty (favors short sequences)
  α = 0.6  → moderate penalty (common default)
  α = 1.0  → full normalization by length
```

### 7.5 Decoding Strategy Comparison

```
┌──────────────────┬──────────┬──────────┬───────────────────────┐
│ Strategy         │ Speed    │ Quality  │ Best For              │
├──────────────────┼──────────┼──────────┼───────────────────────┤
│ Greedy           │ Fast     │ Low      │ Real-time, drafts     │
│ Beam Search      │ Medium   │ High     │ Translation, summ.    │
│ Top-k Sampling   │ Fast     │ Variable │ Creative text gen     │
│ Nucleus (top-p)  │ Fast     │ Variable │ Open-ended generation │
│ Beam + length pen│ Medium   │ Highest  │ Production systems    │
└──────────────────┴──────────┴──────────┴───────────────────────┘
```

---

## 8. Evaluation Metrics

### 8.1 BLEU (Bilingual Evaluation Understudy)

Measures n-gram precision between generated and reference text.

```
BLEU = BP × exp(∑_{n=1}^{N} w_n × log p_n)

Where:
  p_n = modified n-gram precision
  w_n = 1/N (uniform weights, typically N=4)
  BP  = brevity penalty = min(1, exp(1 - r/c))
  r   = reference length
  c   = candidate length
```

**Worked Example (BLEU-2):**

```
Reference:  "The cat is on the mat"
Candidate:  "The cat sits on the mat"

p₁ = 5/6 = 0.833 (matching: The, cat, on, the, mat)
p₂ = 3/5 = 0.600 (matching: "The cat", "on the", "the mat")
BP = min(1, exp(1 - 6/6)) = 1.0
BLEU-2 = 1.0 × exp(0.5 × log(0.833) + 0.5 × log(0.600)) ≈ 0.707
```

### 8.2 ROUGE (Recall-Oriented Understudy for Gisting Evaluation)

Measures **recall** of n-grams (how much of the reference is captured).

```
ROUGE-N = (∑ count_match(n-gram)) / (∑ count(n-gram in reference))

Key variants:
  ROUGE-1: Unigram overlap (captures content)
  ROUGE-2: Bigram overlap (captures fluency)
  ROUGE-L: Longest Common Subsequence (captures structure)
```

**ROUGE-L Formula:**

```
LCS = length of longest common subsequence

Recall_lcs    = LCS / len(reference)
Precision_lcs = LCS / len(candidate)

ROUGE-L = F1 = (1 + β²) × Precision × Recall / (β² × Precision + Recall)

(Typically β = 1 for equal weighting → standard F1)
```

**Worked Example (ROUGE):**

```
Reference: "The cat is sitting on the mat"  (7 words)
Candidate: "The cat sits on the mat"        (6 words)

ROUGE-1: Matching 5 of 7 ref unigrams → Recall=0.714, Prec=0.833, F1=0.769
ROUGE-2: Matching 3 of 6 ref bigrams  → Recall=0.500, Prec=0.600, F1=0.545
ROUGE-L: LCS length=5 → Recall=0.714, Prec=0.833, F1=0.769
```

### 8.3 METEOR (Metric for Evaluation of Translation with Explicit ORdering)

```
METEOR considers:
  1. Exact token matches
  2. Stemmed matches     (running → run)
  3. Synonym matches     (big → large)
  4. Word order (chunk penalty)

METEOR = F_mean × (1 - Penalty)

Where:
  F_mean  = (10 × P × R) / (R + 9P)   (recall-weighted harmonic mean)
  Penalty = 0.5 × (chunks / matches)³
  chunks  = number of contiguous matched groups
```

### 8.4 Metric Comparison

```
┌──────────┬──────────────┬──────────────┬────────────────────────┐
│ Metric   │ Focus        │ N-gram Level │ Common Use             │
├──────────┼──────────────┼──────────────┼────────────────────────┤
│ BLEU     │ Precision    │ 1-4 grams    │ Machine translation    │
│ ROUGE    │ Recall       │ 1,2,L        │ Summarization          │
│ METEOR   │ F1 + order   │ unigrams +   │ Translation (robust)   │
│          │              │ synonyms     │                        │
│ BERTScore│ Semantic sim │ Token-level  │ All generation tasks   │
└──────────┴──────────────┴──────────────┴────────────────────────┘
```

---

## 9. Code: Translation and Summarization with HuggingFace

### 9.1 Machine Translation

```python
from transformers import T5ForConditionalGeneration, T5Tokenizer

model_name = "t5-base"
tokenizer = T5Tokenizer.from_pretrained(model_name)
model = T5ForConditionalGeneration.from_pretrained(model_name)

# English to French translation
source_text = "translate English to French: The weather is beautiful today."
input_ids = tokenizer(source_text, return_tensors="pt").input_ids

outputs = model.generate(
    input_ids,
    max_length=50,
    num_beams=4,
    length_penalty=0.6,
    early_stopping=True,
)
translation = tokenizer.decode(outputs[0], skip_special_tokens=True)
print(f"Translation: {translation}")
```

### 9.2 Abstractive Summarization with BART

```python
from transformers import BartForConditionalGeneration, BartTokenizer

model_name = "facebook/bart-large-cnn"
tokenizer = BartTokenizer.from_pretrained(model_name)
model = BartForConditionalGeneration.from_pretrained(model_name)

article = """
Researchers at MIT have developed a new type of solar cell that can
generate electricity even on cloudy days. The technology uses a special
coating that captures diffused light, which makes up a significant
portion of available sunlight in many regions. In laboratory tests,
the new cells achieved 30% higher output than conventional panels
under overcast conditions. The team plans to begin field trials next
year, with commercial production expected within three to five years.
"""

inputs = tokenizer(article.strip(), return_tensors="pt",
                   max_length=1024, truncation=True)

summary_ids = model.generate(
    inputs["input_ids"],
    max_length=80,
    min_length=20,
    num_beams=5,
    length_penalty=1.0,
    no_repeat_ngram_size=3,
    early_stopping=True,
)

summary = tokenizer.decode(summary_ids[0], skip_special_tokens=True)
print(f"Summary: {summary}")
```

### 9.3 Computing Evaluation Scores

```python
# pip install rouge-score nltk
from rouge_score import rouge_scorer
from nltk.translate.bleu_score import sentence_bleu, SmoothingFunction

# ROUGE
scorer = rouge_scorer.RougeScorer(['rouge1', 'rouge2', 'rougeL'], use_stemmer=True)
ref = "MIT researchers developed solar cells that work on cloudy days."
cand = "New solar cells from MIT generate electricity in overcast conditions."
scores = scorer.score(ref, cand)
for key, val in scores.items():
    print(f"  {key}: P={val.precision:.3f} R={val.recall:.3f} F1={val.fmeasure:.3f}")

# BLEU
reference = "The cat is sitting on the mat".split()
candidate = "The cat sits on the mat".split()
smoothing = SmoothingFunction().method1
bleu_4 = sentence_bleu([reference], candidate,
                        weights=(0.25, 0.25, 0.25, 0.25),
                        smoothing_function=smoothing)
print(f"BLEU-4: {bleu_4:.4f}")
```

---

## 10. Real-World Applications

| Task                  | Model Choice        | Dataset           | Industry Application              |
|-----------------------|---------------------|-------------------|------------------------------------|
| Translation           | T5, mBART, NLLB     | WMT, OPUS         | Google Translate, DeepL            |
| News Summarization    | BART-CNN, Pegasus    | CNN/DailyMail     | News aggregators, briefing tools   |
| Scientific Summarization | BART, LED          | arXiv, PubMed     | Research paper summaries           |
| Question Generation   | T5                   | SQuAD             | Educational quiz generators        |
| Data-to-Text          | T5, GPT             | WebNLG, E2E       | Automated report writing           |
| Grammar Correction    | T5, BART             | CoNLL, JFLEG      | Grammarly-style writing assistants |
| Dialogue              | BART, BlenderBot     | PersonaChat       | Customer service chatbots          |

---

## 11. Summary Table

| Aspect                 | Detail                                                |
|------------------------|-------------------------------------------------------|
| **Task Type**          | Variable-length input → variable-length output        |
| **Key Tasks**          | Translation, summarization, QG, data-to-text          |
| **Best Models**        | T5 (unified), BART (summarization), mBART (multilingual)|
| **Decoding**           | Beam search (production), sampling (creative tasks)   |
| **BLEU**               | N-gram precision metric for translation               |
| **ROUGE**              | N-gram recall metric for summarization                |
| **METEOR**             | Harmonic mean with stemming/synonym matching          |
| **Length Penalty**      | α parameter prevents degenerate short outputs         |
| **Key Challenge**       | Balancing fluency, adequacy, and length               |

---

## 12. Revision Questions

1. **Compare extractive and abstractive summarization.** Why are encoder-decoder transformers better suited for abstractive summarization? What advantages does abstractive have over extractive?

2. **Walk through the beam search algorithm with beam width k=3.** Show how candidates are scored and pruned at each step. How does the length penalty factor α affect the final output?

3. **Calculate BLEU-2 for a given reference and candidate.** Include the brevity penalty in your calculation. When can BLEU give misleading scores?

4. **Calculate ROUGE-1, ROUGE-2, and ROUGE-L** for a reference-candidate pair. Why is ROUGE-L based on the longest common subsequence rather than exact n-gram matching?

5. **Compare T5 and BART on summarization benchmarks.** Which model performs better on CNN/DailyMail vs. XSum? What architectural or training differences explain the gap?

6. **How is structured data linearized for seq2seq models?** Give an example of converting a JSON record to a text-to-text input. What challenges arise with complex nested structures?

---
[← BART Architecture](02-bart-architecture.md) | [Applications →](04-applications.md)
