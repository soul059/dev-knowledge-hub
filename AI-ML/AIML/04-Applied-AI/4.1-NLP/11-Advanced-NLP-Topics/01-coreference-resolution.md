# Coreference Resolution

## Overview

Coreference resolution identifies all expressions in a text that refer to the same real-world entity. For example, in "Marie Curie won the Nobel Prize. **She** was a physicist," the system must determine that "She" refers to "Marie Curie." This is fundamental for text understanding, information extraction, and building coherent NLP pipelines.

---

## What Are Mentions and Coreferences?

```
Text: "Barack Obama was born in Hawaii. He became the 44th 
       president. Obama served two terms in office."

Mentions:
  [Barack Obama]₁  [Hawaii]₂  [He]₁  [the 44th president]₁  
  [Obama]₁  [two terms]₃  [office]₄

Coreference clusters:
  Cluster 1: {Barack Obama, He, the 44th president, Obama}
  Cluster 2: {Hawaii}
  
  All mentions in Cluster 1 refer to the SAME entity

Types of referring expressions:
  - Proper nouns:  "Barack Obama", "Obama"
  - Pronouns:      "He", "She", "It", "They"
  - Noun phrases:  "the 44th president", "the tall man"
  - Demonstratives: "this", "that"
```

---

## Types of Coreference

```
1. Pronominal anaphora (most common):
   "Alice went to the store. She bought milk."
   She → Alice

2. Proper noun coreference:
   "Microsoft released Windows. The company also launched Office."
   The company → Microsoft

3. Cataphora (backward reference):
   "Before she left, Alice locked the door."
   she → Alice (pronoun comes BEFORE the noun)

4. Bridging anaphora (indirect):
   "I entered the room. The ceiling was very high."
   The ceiling is part of the room (not direct reference)

5. Event coreference:
   "The earthquake struck at 3am. The disaster caused widespread damage."
   The disaster → The earthquake
```

---

## Neural Coreference Resolution

```
Modern approach (Lee et al., 2017):

  Step 1: Enumerate all possible spans (mentions)
    "Barack Obama was born in Hawaii"
    → "Barack", "Obama", "Barack Obama", "was", "born", 
       "Hawaii", "born in Hawaii", ...

  Step 2: Score each span as a mention
    mention_score(i) = MLP(span_embedding(i))
    Keep top spans

  Step 3: For each mention, score compatibility with all previous mentions
    pair_score(i, j) = MLP([span_i; span_j; span_i ⊙ span_j; features])
    
    Features: distance, gender match, number match, etc.

  Step 4: Link mentions with highest pair scores
    antecedent(i) = argmax_j pair_score(i, j)

  Step 5: Build clusters from linked mentions
```

---

## Implementation

```python
import spacy

# spaCy with neuralcoref (or use HuggingFace models)
# Alternative: using the allennlp coreference model
from allennlp.predictors.predictor import Predictor

predictor = Predictor.from_path(
    "https://storage.googleapis.com/allennlp-public-models/"
    "coref-spanbert-large-2021.03.10.tar.gz"
)

text = """
Marie Curie was a Polish-French physicist. She was the first woman 
to win a Nobel Prize. Curie discovered polonium and radium. 
Her research on radioactivity changed science forever.
"""

result = predictor.predict(document=text)

# Extract coreference clusters
for cluster in result["clusters"]:
    mentions = []
    for start, end in cluster:
        mention = " ".join(result["document"][start:end+1])
        mentions.append(mention)
    print(f"Cluster: {mentions}")

# Output:
# Cluster: ['Marie Curie', 'She', 'Curie', 'Her']
```

---

## Applications

| Application | How Coref Helps |
|-------------|----------------|
| Information extraction | Merge facts about same entity |
| Summarization | Avoid redundant references |
| Machine translation | Resolve pronouns correctly |
| Question answering | Understand who/what is being asked about |
| Dialogue systems | Track entities across turns |
| Knowledge graph construction | Link mentions to entities |

---

## Evaluation

```
Metrics for coreference:
  MUC:    Counts minimum links to reconstruct clusters
  B³:     Precision/recall over individual mentions
  CEAF:   Best alignment between predicted and gold clusters
  CoNLL:  Average of MUC, B³, CEAF (standard metric)

  State-of-the-art on OntoNotes (CoNLL F1): ~83%
  (SpanBERT-based models)
```

---

## Revision Questions

1. **What is coreference resolution and why is it important?**
2. **What is the difference between anaphora and cataphora?**
3. **How does the span-based neural model enumerate and score mentions?**
4. **Name three NLP tasks that benefit from coreference resolution.**
5. **What is the CoNLL F1 metric for coreference evaluation?**

---

[Previous: ../10-Text-Generation/06-summarization.md](../10-Text-Generation/06-summarization.md) | [Next: 02-semantic-similarity.md](02-semantic-similarity.md)
