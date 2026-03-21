# Extractive Question Answering

## Overview

Extractive QA identifies the exact answer span within a given context passage. The model doesn't generate new text — it highlights the substring that answers the question. This approach is faithful to the source, interpretable, and highly effective for factoid questions.

---

## Architecture

```
Input:
  [CLS] Question [SEP] Context passage [SEP]

                    ┌─────────────────────────────┐
                    │   Pre-trained Transformer     │
                    │   (BERT / RoBERTa / ALBERT)   │
                    └──────────┬──────────────────┘
                               │
                    ┌──────────┴──────────────────┐
                    │     Hidden states h_i        │
                    │  for each token in context    │
                    └──────────┬──────────────────┘
                               │
              ┌────────────────┼────────────────┐
              ↓                                 ↓
     ┌────────────────┐              ┌────────────────┐
     │ Start classifier│              │ End classifier  │
     │ W_s · h_i + b_s │              │ W_e · h_i + b_e │
     └────────┬────────┘              └────────┬────────┘
              ↓                                 ↓
     softmax over tokens              softmax over tokens
              ↓                                 ↓
         start index                       end index
              └────────────┬────────────────┘
                           ↓
                    Answer span [start:end]
```

---

## Step-by-Step Inference

```
Context: "Paris is the capital and most populous city of France."
Question: "What is the capital of France?"

Step 1: Tokenize
  [CLS] What is the capital of France ? [SEP] Paris is the capital 
  and most populous city of France . [SEP]

Step 2: Forward pass through BERT
  Get hidden states for all tokens

Step 3: Compute start/end logits
  Token:    Paris  is   the  capital  and  most  populous  city  of  France  .
  Start:    0.92   0.01 0.01 0.03    0.00 0.00  0.00     0.00  0.00 0.00   0.00
  End:      0.03   0.01 0.01 0.01    0.00 0.00  0.00     0.00  0.00 0.00   0.00
  
  Wait — "Paris" alone? Let's try a better example:

  Actually start=0 (Paris), end=0 (Paris)
  Answer: "Paris"  ✓
```

---

## Production-Ready Extractive QA

```python
from transformers import pipeline

# Load a fine-tuned extractive QA model
qa = pipeline("question-answering", model="deepset/roberta-base-squad2")

# Single question
context = """
The Python programming language was created by Guido van Rossum 
and first released in 1991. Python's design philosophy emphasizes 
code readability with the use of significant indentation. It supports 
multiple programming paradigms including procedural, object-oriented, 
and functional programming.
"""

result = qa(question="Who created Python?", context=context)
print(f"Answer: {result['answer']}")       # "Guido van Rossum"
print(f"Score:  {result['score']:.4f}")     # 0.9823
print(f"Start:  {result['start']}")         # character offset
print(f"End:    {result['end']}")

# Multiple questions on same context
questions = [
    "When was Python released?",
    "What does Python emphasize?",
    "What paradigms does Python support?",
]

for q in questions:
    ans = qa(question=q, context=context)
    print(f"Q: {q}\nA: {ans['answer']} ({ans['score']:.2f})\n")
```

---

## Handling Long Documents

```python
def answer_from_long_document(question, document, qa_pipeline, 
                               chunk_size=512, stride=128):
    """Split long document into overlapping chunks and find best answer."""
    words = document.split()
    best_answer = None
    best_score = -1
    
    for i in range(0, len(words), chunk_size - stride):
        chunk = " ".join(words[i:i + chunk_size])
        try:
            result = qa_pipeline(question=question, context=chunk)
            if result["score"] > best_score:
                best_score = result["score"]
                best_answer = result["answer"]
        except Exception:
            continue
    
    return {"answer": best_answer, "score": best_score}
```

---

## Training Tips

| Aspect | Recommendation |
|--------|---------------|
| Base model | RoBERTa-large or DeBERTa-v3 |
| Learning rate | 2e-5 to 5e-5 |
| Batch size | 16–32 |
| Max seq length | 384–512 |
| Doc stride | 128 (overlap between chunks) |
| Epochs | 2–3 (more causes overfitting on SQuAD) |
| Impossible questions | Include unanswerable examples (SQuAD 2.0) |

---

## Extractive QA Limitations

| Limitation | Description | Mitigation |
|------------|-------------|------------|
| Answer must be in text | Can't synthesize or compute | Use abstractive QA |
| Single span only | Some answers are discontinuous | Multi-span extraction |
| Context length limit | Transformers have max length | Sliding window |
| No reasoning | Can't do multi-step logic | Multi-hop models |
| No world knowledge | Only knows what's in context | Add retrieval (RAG) |

---

## Revision Questions

1. **How does extractive QA differ from abstractive QA?**
2. **What are the start and end classifiers in the BERT QA architecture?**
3. **How do you handle documents longer than the model's max sequence length?**
4. **What is the doc stride parameter and why is it important?**
5. **What are the main limitations of extractive QA?**

---

[Previous: 02-reading-comprehension.md](02-reading-comprehension.md) | [Next: 04-open-domain-qa.md](04-open-domain-qa.md)
