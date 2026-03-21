# Text Summarization

## Overview

Text summarization condenses a longer document into a shorter version while preserving the key information. There are two main approaches: **extractive** (selecting important sentences) and **abstractive** (generating new sentences). Modern systems use transformers to produce human-quality summaries.

---

## Extractive vs Abstractive

```
Original text:
  "The Eiffel Tower is a wrought-iron lattice tower on the Champ de 
   Mars in Paris, France. It is named after the engineer Gustave Eiffel, 
   whose company designed and built the tower from 1887 to 1889. The 
   tower is 330 metres tall and was the tallest man-made structure in 
   the world until 1930."

Extractive summary (selects sentences):
  "The Eiffel Tower is a wrought-iron lattice tower on the Champ de 
   Mars in Paris. The tower is 330 metres tall."

Abstractive summary (generates new text):
  "The 330m Eiffel Tower, built by Gustave Eiffel's company between 
   1887-1889, is a famous iron tower in Paris, France."
```

---

## Extractive Summarization

```
Pipeline:
  Document → [Sentence Scoring] → [Selection] → Summary

Scoring methods:
  1. TF-IDF: Score sentences by important term frequency
  2. TextRank: Graph-based (like PageRank for sentences)
  3. BERT embeddings: Cluster sentences, pick centroids

TextRank algorithm:
  ┌─────────────────────────────────┐
  │  1. Split into sentences         │
  │  2. Build similarity graph       │
  │     (cosine similarity of TF-IDF │
  │      or embedding vectors)       │
  │  3. Run PageRank on graph        │
  │  4. Select top-k scored sentences│
  │  5. Order by original position   │
  └─────────────────────────────────┘
```

```python
# Extractive summarization with sumy
from sumy.parsers.plaintext import PlaintextParser
from sumy.nlp.tokenizers import Tokenizer
from sumy.summarizers.text_rank import TextRankSummarizer

text = """
Machine learning is a subset of artificial intelligence that provides 
systems the ability to automatically learn and improve from experience.
The process begins with observations or data, such as examples, direct 
experience, or instruction. It looks for patterns in data and makes 
better decisions based on the examples provided. The primary aim is to 
allow computers to learn automatically without human intervention.
Machine learning algorithms are often categorized as supervised, 
unsupervised, or reinforcement learning.
"""

parser = PlaintextParser.from_string(text, Tokenizer("english"))
summarizer = TextRankSummarizer()
summary = summarizer(parser.document, sentences_count=2)

for sentence in summary:
    print(sentence)
```

---

## Abstractive Summarization

```
Uses encoder-decoder transformers:

  Document → [Encoder] → [Decoder] → Summary

Popular models:
  ┌──────────────┬────────────┬──────────────────┐
  │ Model        │ Parameters │ Key Feature       │
  ├──────────────┼────────────┼──────────────────┤
  │ BART         │ 140M–400M  │ Denoising AE      │
  │ PEGASUS      │ 568M       │ Gap-sentence task  │
  │ T5           │ 220M–11B   │ Text-to-text       │
  │ LED          │ 400M       │ Long docs (16K)    │
  │ GPT-3/4      │ 175B+      │ Few-shot prompting │
  └──────────────┴────────────┴──────────────────┘

PEGASUS innovation:
  Pre-training task: Mask entire important sentences
  Model learns to "generate" missing sentences
  → Directly aligned with summarization!
```

```python
from transformers import pipeline

summarizer = pipeline("summarization", model="facebook/bart-large-cnn")

article = """
Scientists have discovered a new species of deep-sea fish in the 
Pacific Ocean. The fish, named Pseudoliparis swirei, was found at 
a depth of 8,178 meters in the Mariana Trench, making it the 
deepest-dwelling fish ever recorded. The translucent snailfish 
was captured on camera during a research expedition. The discovery 
challenges previous assumptions about the depth limits of fish 
survival. Researchers believe the fish has adapted to the extreme 
pressure through specialized cellular mechanisms. The finding was 
published in the journal Nature Ecology & Evolution.
"""

summary = summarizer(article, max_length=60, min_length=20, do_sample=False)
print(summary[0]["summary_text"])
# "Scientists discovered a new deep-sea fish species at 8,178 meters 
#  in the Mariana Trench, making it the deepest-dwelling fish ever recorded."
```

---

## Long Document Summarization

```
Challenge: Transformers have max input length (512-1024 tokens)
           Documents can be thousands of tokens

Solutions:
  1. Truncation: Just use first 512 tokens (loses info)
  
  2. Chunking + merge:
     Doc → [Chunk 1] → Summary 1 ─┐
         → [Chunk 2] → Summary 2 ─┼→ [Merge] → Final summary
         → [Chunk 3] → Summary 3 ─┘
  
  3. Hierarchical:
     Paragraphs → Paragraph summaries → Document summary
  
  4. Long-context models:
     LED (Longformer Encoder-Decoder): 16,384 tokens
     BigBird: 4,096 tokens with sparse attention
     LLMs (GPT-4): 128K tokens
```

---

## Evaluation Metrics

```
ROUGE (Recall-Oriented Understudy for Gisting Evaluation):

  ROUGE-1: Unigram overlap
  ROUGE-2: Bigram overlap  
  ROUGE-L: Longest Common Subsequence

  Example:
    Reference: "the cat sat on the mat"
    Generated: "the cat is on the mat"

    ROUGE-1:
      Overlap: {the, cat, on, the, mat} = 5
      Precision = 5/7 = 0.71
      Recall    = 5/6 = 0.83
      F1        = 0.77

    ROUGE-2:
      Reference bigrams: {the cat, cat sat, sat on, on the, the mat}
      Generated bigrams: {the cat, cat is, is on, on the, the mat}
      Overlap: {the cat, on the, the mat} = 3
      Precision = 3/5 = 0.60
      Recall    = 3/5 = 0.60
      F1        = 0.60
```

---

## Summary Comparison

| Approach | Faithful | Fluent | Concise | Speed | Best For |
|----------|:---:|:---:|:---:|:---:|---------|
| Extractive | ✓✓ | Medium | Medium | Fast | News, legal docs |
| Abstractive | Medium | ✓✓ | ✓✓ | Slower | Any text |
| Hybrid | ✓✓ | ✓ | ✓ | Medium | Long documents |
| LLM-based | ✓ | ✓✓ | ✓✓ | Slowest | Complex reasoning |

---

## Revision Questions

1. **What is the difference between extractive and abstractive summarization?**
2. **How does TextRank work for extractive summarization?**
3. **Why is PEGASUS particularly good at summarization?**
4. **What is ROUGE and how is it computed?**
5. **How do you handle documents that exceed the model's input length?**

---

[Previous: 05-dialogue-systems.md](05-dialogue-systems.md) | [Back to README](../README.md)
