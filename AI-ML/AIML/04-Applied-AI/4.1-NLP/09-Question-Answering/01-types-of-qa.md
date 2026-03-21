# Types of Question Answering

## Overview

Question Answering (QA) is the task of automatically answering questions posed in natural language. QA systems vary widely in their input sources, answer formats, and complexity. Understanding the taxonomy of QA is essential for choosing the right approach for each problem setting.

---

## QA Taxonomy

```
Question Answering
├── By Answer Source
│   ├── Reading Comprehension  → answer from given passage
│   ├── Knowledge-Based QA     → answer from knowledge graph
│   ├── Open-Domain QA         → answer from large corpus (web/Wikipedia)
│   └── Conversational QA      → answer considering dialogue history
│
├── By Answer Format
│   ├── Extractive    → span from the source text
│   ├── Abstractive   → generated answer (may not appear in text)
│   ├── Multiple Choice → select from given options
│   ├── Boolean (Yes/No) → binary answer
│   └── List          → multiple items
│
├── By Reasoning Required
│   ├── Factoid       → simple fact lookup ("Who invented penicillin?")
│   ├── Multi-hop     → chain of reasoning across multiple facts
│   ├── Commonsense   → requires world knowledge
│   └── Numerical     → requires computation
│
└── By Domain
    ├── Open-domain   → any topic
    └── Closed-domain → specific area (medical, legal, etc.)
```

---

## Examples by Type

```
Extractive QA:
  Context: "Albert Einstein was born in Ulm, Germany, in 1879."
  Question: "Where was Einstein born?"
  Answer: "Ulm, Germany"  ← extracted directly from text

Abstractive QA:
  Context: "Einstein was born in 1879 and died in 1955."
  Question: "How long did Einstein live?"
  Answer: "76 years"  ← not in text; requires computation

Multi-hop QA:
  Fact 1: "The Eiffel Tower is in Paris."
  Fact 2: "Paris is the capital of France."
  Question: "In which country is the Eiffel Tower?"
  Answer: "France"  ← requires connecting two facts

Boolean QA:
  Context: "Python was created by Guido van Rossum."
  Question: "Was Python created by Linus Torvalds?"
  Answer: "No"
```

---

## Key QA Datasets

| Dataset | Type | Size | Source | Answer Format |
|---------|------|:---:|--------|:---:|
| SQuAD 1.1 | Extractive | 100K | Wikipedia | Span |
| SQuAD 2.0 | Extractive + unanswerable | 150K | Wikipedia | Span or "no answer" |
| Natural Questions | Open-domain | 300K | Google queries | Short/long answer |
| HotpotQA | Multi-hop | 113K | Wikipedia | Span + supporting facts |
| TriviaQA | Open-domain | 95K | Trivia websites | Span |
| BoolQ | Boolean | 16K | Wikipedia | Yes/No |
| DROP | Numerical reasoning | 96K | Wikipedia | Number/date/span |
| CoQA | Conversational | 127K | Various | Free-form |

---

## QA Pipeline vs End-to-End

```
Pipeline Approach (traditional):
  Question → [Question Analysis] → [Document Retrieval] → [Answer Extraction]
                 ↓                      ↓                       ↓
           classify type          find relevant docs      extract answer span

End-to-End (modern):
  Question + Context → [Transformer Model] → Answer
  
  Examples: BERT, RoBERTa, ALBERT fine-tuned on SQuAD
```

---

## Simple Extractive QA with Transformers

```python
from transformers import pipeline

qa_pipeline = pipeline("question-answering", model="deepset/roberta-base-squad2")

context = """
Machine learning is a subset of artificial intelligence that enables
systems to learn and improve from experience without being explicitly
programmed. It focuses on developing algorithms that can access data
and use it to learn for themselves.
"""

questions = [
    "What is machine learning?",
    "What does ML focus on?",
    "Is ML part of AI?"
]

for q in questions:
    result = qa_pipeline(question=q, context=context)
    print(f"Q: {q}")
    print(f"A: {result['answer']} (confidence: {result['score']:.3f})\n")
```

---

## Comparison of QA Approaches

| Approach | Pros | Cons | Best For |
|----------|------|------|----------|
| Extractive | Faithful to source, fast | Can't synthesize | Factoid, reading comprehension |
| Abstractive | Flexible answers | May hallucinate | Complex questions |
| Knowledge-based | Structured, precise | Limited coverage | Factoid with KG |
| Open-domain | Any question | Retrieval errors | General purpose |
| Conversational | Context-aware | History tracking complex | Chatbots, assistants |

---

## Revision Questions

1. **What is the difference between extractive and abstractive QA?**
2. **Give an example of a multi-hop question and explain why it's harder.**
3. **How does SQuAD 2.0 differ from SQuAD 1.1?**
4. **What are the three stages of a traditional QA pipeline?**
5. **When would you prefer knowledge-based QA over open-domain QA?**

---

[Previous: ../08-Machine-Translation/06-multilingual-models.md](../08-Machine-Translation/06-multilingual-models.md) | [Next: 02-reading-comprehension.md](02-reading-comprehension.md)
