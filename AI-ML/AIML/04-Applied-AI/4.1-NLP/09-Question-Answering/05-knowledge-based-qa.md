# Knowledge-Based Question Answering

## Overview

Knowledge-Based QA (KBQA) answers questions by querying structured knowledge graphs (KGs) such as Wikidata, Freebase, or DBpedia. Instead of reading unstructured text, the system converts natural language questions into formal queries that retrieve precise answers from a graph database.

---

## Knowledge Graphs

```
A Knowledge Graph stores facts as (subject, predicate, object) triples:

  (Albert_Einstein, born_in, Ulm)
  (Albert_Einstein, born_date, 1879-03-14)
  (Albert_Einstein, occupation, Physicist)
  (Albert_Einstein, won, Nobel_Prize_in_Physics)
  (Nobel_Prize_in_Physics, year, 1921)
  (Ulm, located_in, Germany)

  Visually:
    Albert_Einstein ──born_in──→ Ulm ──located_in──→ Germany
         │                                    
         ├──born_date──→ 1879-03-14
         ├──occupation──→ Physicist
         └──won──→ Nobel_Prize_in_Physics ──year──→ 1921

Popular KGs:
  Wikidata:   ~100M entities, ~1.4B triples
  DBpedia:    ~6M entities from Wikipedia infoboxes
  Freebase:   ~46M entities (deprecated, migrated to Wikidata)
  ConceptNet: Commonsense knowledge
```

---

## KBQA Pipeline

```
Question: "Where was Einstein born?"
    │
    ▼
┌──────────────────────┐
│ Entity Linking        │  "Einstein" → Albert_Einstein (Q937 in Wikidata)
└──────────┬───────────┘
           │
           ▼
┌──────────────────────┐
│ Relation Detection    │  "born" → P19 (place of birth)
└──────────┬───────────┘
           │
           ▼
┌──────────────────────┐
│ Query Construction    │  SPARQL: SELECT ?x WHERE { wd:Q937 wdt:P19 ?x }
└──────────┬───────────┘
           │
           ▼
┌──────────────────────┐
│ Query Execution       │  Execute on KG → Q3012 (Ulm)
└──────────┬───────────┘
           │
           ▼
    Answer: "Ulm"
```

---

## Semantic Parsing Approach

```python
# Converting natural language to SPARQL
question = "What movies did Christopher Nolan direct?"

# Semantic parser output:
sparql = """
SELECT ?movie ?movieLabel WHERE {
  ?movie wdt:P57 wd:Q25191 .        # P57 = director, Q25191 = Nolan
  ?movie wdt:P31 wd:Q11424 .        # P31 = instance of, Q11424 = film
  SERVICE wikibase:label { bd:serviceParam wikibase:language "en" }
}
"""

# Using SPARQLWrapper to query Wikidata
from SPARQLWrapper import SPARQLWrapper, JSON

sparql_endpoint = SPARQLWrapper("https://query.wikidata.org/sparql")
sparql_endpoint.setQuery(sparql)
sparql_endpoint.setReturnFormat(JSON)
results = sparql_endpoint.query().convert()

for result in results["results"]["bindings"]:
    print(result["movieLabel"]["value"])
# Inception, The Dark Knight, Interstellar, Tenet, ...
```

---

## Neural KBQA with Embeddings

```
Instead of generating SPARQL, embed questions and KG entities 
in the same vector space:

  Question embedding:   q = encode("Where was Einstein born?")
  Entity embeddings:    e_i = embed(entity_i)    for all entities
  Relation embeddings:  r_j = embed(relation_j)  for all relations

  Score(q, e, r) = f(q, e, r)   → rank candidate answers

  Popular models:
    TransE:   h + r ≈ t           (translation-based)
    TransR:   M_r · h + r ≈ M_r · t
    RotatE:   h ∘ r ≈ t           (rotation in complex space)
    ComplEx:  Re(⟨h, r, t̄⟩)       (complex-valued embeddings)

Embedding approach advantages:
  - No need for explicit SPARQL generation
  - Handles incomplete KGs (missing edges)
  - Can generalize to unseen entity combinations
```

---

## KBQA vs Text-Based QA

| Feature | KBQA | Text-Based QA |
|---------|:---:|:---:|
| Data source | Structured KG | Unstructured text |
| Answer precision | Very high | Depends on extraction |
| Coverage | Limited to KG content | Anything in corpus |
| Multi-hop reasoning | Natural (graph traversal) | Difficult |
| Scalability | Fast (graph queries) | Slower (document retrieval) |
| Handling ambiguity | Entity disambiguation needed | Contextual |
| New information | Requires KG updates | Just add documents |

---

## Multi-Hop KBQA

```
Question: "What country is the birthplace of the inventor of the telephone?"

Step 1: "inventor of the telephone" → Alexander_Graham_Bell
  (Telephone, inventor, Alexander_Graham_Bell)

Step 2: "birthplace of Alexander Graham Bell" → Edinburgh
  (Alexander_Graham_Bell, birthplace, Edinburgh)

Step 3: "country of Edinburgh" → Scotland → United Kingdom
  (Edinburgh, country, United_Kingdom)

Answer: "United Kingdom"

This requires traversing 3 edges in the knowledge graph!
```

---

## Hybrid Systems

```
Modern QA systems combine KBQA and text-based QA:

  Question → ┬─→ [KG Retriever]   → structured answer
             │
             └─→ [Text Retriever]  → extracted answer
                        │
                        ▼
               [Answer Merger / Ranker]
                        │
                        ▼
                   Final Answer

Example: Google Search
  - Knowledge Panel (KBQA): "Albert Einstein, born March 14, 1879"
  - Web Results (Text QA): "Einstein was born in Ulm, Germany..."
```

---

## Revision Questions

1. **What is a knowledge graph and how are facts represented in it?**
2. **Walk through the KBQA pipeline from question to answer.**
3. **How does semantic parsing convert natural language to SPARQL?**
4. **What are KG embedding models and how do they help QA?**
5. **What are the advantages and limitations of KBQA vs text-based QA?**
6. **How do multi-hop questions work in knowledge graphs?**

---

[Previous: 04-open-domain-qa.md](04-open-domain-qa.md) | [Back to README](../README.md)
