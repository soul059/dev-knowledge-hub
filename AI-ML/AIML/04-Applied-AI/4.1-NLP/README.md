# 4.1 Natural Language Processing (NLP)

## Course Overview

Natural Language Processing (NLP) sits at the intersection of linguistics, computer science, and artificial intelligence. This subject covers the complete NLP pipeline — from raw text preprocessing to advanced applications like machine translation, question answering, and text generation. You'll learn both classical approaches (TF-IDF, n-grams, CRFs) and modern deep learning methods (word embeddings, transformers, pre-trained language models).

```
┌─────────────────────────────────────────────────────────────────────┐
│                     NLP LEARNING ROADMAP                            │
│                                                                     │
│  Fundamentals ──▶ Preprocessing ──▶ Representation ──▶ Embeddings  │
│       │                                                    │        │
│       ▼                                                    ▼        │
│  Classification ──▶ Seq Labeling ──▶ Language Models ──▶ MT        │
│                                          │                          │
│                                          ▼                          │
│                              QA ──▶ Text Gen ──▶ Advanced Topics   │
└─────────────────────────────────────────────────────────────────────┘
```

## Learning Objectives

- Understand the NLP pipeline from raw text to actionable insights
- Master text preprocessing and normalization techniques
- Implement text representation methods (BoW, TF-IDF, embeddings)
- Train and evaluate text classification and sequence labeling models
- Build language models and understand transfer learning in NLP
- Apply NLP to machine translation, QA, and text generation tasks

## Table of Contents

### Unit 1: NLP Fundamentals
| # | Topic | File |
|---|-------|------|
| 1 | What is NLP? | [01-what-is-nlp.md](01-NLP-Fundamentals/01-what-is-nlp.md) |
| 2 | NLP Applications | [02-nlp-applications.md](01-NLP-Fundamentals/02-nlp-applications.md) |
| 3 | Challenges in NLP | [03-challenges-in-nlp.md](01-NLP-Fundamentals/03-challenges-in-nlp.md) |
| 4 | NLP Pipeline Overview | [04-nlp-pipeline-overview.md](01-NLP-Fundamentals/04-nlp-pipeline-overview.md) |
| 5 | Text vs Language | [05-text-vs-language.md](01-NLP-Fundamentals/05-text-vs-language.md) |

### Unit 2: Text Preprocessing
| # | Topic | File |
|---|-------|------|
| 1 | Tokenization | [01-tokenization.md](02-Text-Preprocessing/01-tokenization.md) |
| 2 | Lowercasing | [02-lowercasing.md](02-Text-Preprocessing/02-lowercasing.md) |
| 3 | Stopword Removal | [03-stopword-removal.md](02-Text-Preprocessing/03-stopword-removal.md) |
| 4 | Stemming | [04-stemming.md](02-Text-Preprocessing/04-stemming.md) |
| 5 | Lemmatization | [05-lemmatization.md](02-Text-Preprocessing/05-lemmatization.md) |
| 6 | Text Normalization | [06-text-normalization.md](02-Text-Preprocessing/06-text-normalization.md) |
| 7 | Regular Expressions for Text | [07-regular-expressions.md](02-Text-Preprocessing/07-regular-expressions.md) |

### Unit 3: Text Representation
| # | Topic | File |
|---|-------|------|
| 1 | Bag of Words (BoW) | [01-bag-of-words.md](03-Text-Representation/01-bag-of-words.md) |
| 2 | TF-IDF | [02-tf-idf.md](03-Text-Representation/02-tf-idf.md) |
| 3 | N-grams | [03-n-grams.md](03-Text-Representation/03-n-grams.md) |
| 4 | One-Hot Encoding | [04-one-hot-encoding.md](03-Text-Representation/04-one-hot-encoding.md) |
| 5 | Word Embeddings Introduction | [05-word-embeddings-intro.md](03-Text-Representation/05-word-embeddings-intro.md) |
| 6 | Sparse vs Dense Representations | [06-sparse-vs-dense.md](03-Text-Representation/06-sparse-vs-dense.md) |

### Unit 4: Word Embeddings
| # | Topic | File |
|---|-------|------|
| 1 | Word2Vec (CBOW, Skip-gram) | [01-word2vec.md](04-Word-Embeddings/01-word2vec.md) |
| 2 | GloVe | [02-glove.md](04-Word-Embeddings/02-glove.md) |
| 3 | FastText | [03-fasttext.md](04-Word-Embeddings/03-fasttext.md) |
| 4 | Word Embedding Properties | [04-embedding-properties.md](04-Word-Embeddings/04-embedding-properties.md) |
| 5 | Embedding Visualization | [05-embedding-visualization.md](04-Word-Embeddings/05-embedding-visualization.md) |
| 6 | Pre-trained Embeddings | [06-pretrained-embeddings.md](04-Word-Embeddings/06-pretrained-embeddings.md) |
| 7 | Out-of-Vocabulary Handling | [07-oov-handling.md](04-Word-Embeddings/07-oov-handling.md) |

### Unit 5: Text Classification
| # | Topic | File |
|---|-------|------|
| 1 | Problem Formulation | [01-problem-formulation.md](05-Text-Classification/01-problem-formulation.md) |
| 2 | Feature Extraction | [02-feature-extraction.md](05-Text-Classification/02-feature-extraction.md) |
| 3 | Traditional ML Approaches | [03-traditional-ml.md](05-Text-Classification/03-traditional-ml.md) |
| 4 | Deep Learning Approaches | [04-deep-learning-approaches.md](05-Text-Classification/04-deep-learning-approaches.md) |
| 5 | Multi-label Classification | [05-multi-label.md](05-Text-Classification/05-multi-label.md) |
| 6 | Handling Imbalanced Data | [06-imbalanced-data.md](05-Text-Classification/06-imbalanced-data.md) |
| 7 | Evaluation Metrics | [07-evaluation-metrics.md](05-Text-Classification/07-evaluation-metrics.md) |

### Unit 6: Sequence Labeling
| # | Topic | File |
|---|-------|------|
| 1 | POS Tagging | [01-pos-tagging.md](06-Sequence-Labeling/01-pos-tagging.md) |
| 2 | Named Entity Recognition | [02-named-entity-recognition.md](06-Sequence-Labeling/02-named-entity-recognition.md) |
| 3 | Chunking | [03-chunking.md](06-Sequence-Labeling/03-chunking.md) |
| 4 | BIO Tagging Scheme | [04-bio-tagging.md](06-Sequence-Labeling/04-bio-tagging.md) |
| 5 | Conditional Random Fields | [05-crf.md](06-Sequence-Labeling/05-crf.md) |
| 6 | Neural Sequence Labeling | [06-neural-sequence-labeling.md](06-Sequence-Labeling/06-neural-sequence-labeling.md) |

### Unit 7: Language Models
| # | Topic | File |
|---|-------|------|
| 1 | N-gram Language Models | [01-ngram-language-models.md](07-Language-Models/01-ngram-language-models.md) |
| 2 | Perplexity | [02-perplexity.md](07-Language-Models/02-perplexity.md) |
| 3 | Neural Language Models | [03-neural-language-models.md](07-Language-Models/03-neural-language-models.md) |
| 4 | Pre-training and Fine-tuning | [04-pretraining-finetuning.md](07-Language-Models/04-pretraining-finetuning.md) |
| 5 | Transfer Learning in NLP | [05-transfer-learning.md](07-Language-Models/05-transfer-learning.md) |

### Unit 8: Machine Translation
| # | Topic | File |
|---|-------|------|
| 1 | Statistical MT Overview | [01-statistical-mt.md](08-Machine-Translation/01-statistical-mt.md) |
| 2 | Neural MT (Seq2Seq) | [02-neural-mt.md](08-Machine-Translation/02-neural-mt.md) |
| 3 | Attention for Translation | [03-attention-for-translation.md](08-Machine-Translation/03-attention-for-translation.md) |
| 4 | Transformer for Translation | [04-transformer-translation.md](08-Machine-Translation/04-transformer-translation.md) |
| 5 | Evaluation (BLEU Score) | [05-bleu-score.md](08-Machine-Translation/05-bleu-score.md) |
| 6 | Multilingual Models | [06-multilingual-models.md](08-Machine-Translation/06-multilingual-models.md) |

### Unit 9: Question Answering
| # | Topic | File |
|---|-------|------|
| 1 | Types of QA | [01-types-of-qa.md](09-Question-Answering/01-types-of-qa.md) |
| 2 | Reading Comprehension | [02-reading-comprehension.md](09-Question-Answering/02-reading-comprehension.md) |
| 3 | Extractive QA | [03-extractive-qa.md](09-Question-Answering/03-extractive-qa.md) |
| 4 | Open-domain QA | [04-open-domain-qa.md](09-Question-Answering/04-open-domain-qa.md) |
| 5 | Knowledge-based QA | [05-knowledge-based-qa.md](09-Question-Answering/05-knowledge-based-qa.md) |

### Unit 10: Text Generation
| # | Topic | File |
|---|-------|------|
| 1 | Language Model Decoding | [01-lm-decoding.md](10-Text-Generation/01-lm-decoding.md) |
| 2 | Sampling Strategies | [02-sampling-strategies.md](10-Text-Generation/02-sampling-strategies.md) |
| 3 | Beam Search | [03-beam-search.md](10-Text-Generation/03-beam-search.md) |
| 4 | Controllable Generation | [04-controllable-generation.md](10-Text-Generation/04-controllable-generation.md) |
| 5 | Dialogue Systems | [05-dialogue-systems.md](10-Text-Generation/05-dialogue-systems.md) |
| 6 | Summarization | [06-summarization.md](10-Text-Generation/06-summarization.md) |

### Unit 11: Advanced NLP Topics
| # | Topic | File |
|---|-------|------|
| 1 | Coreference Resolution | [01-coreference-resolution.md](11-Advanced-NLP-Topics/01-coreference-resolution.md) |
| 2 | Semantic Similarity | [02-semantic-similarity.md](11-Advanced-NLP-Topics/02-semantic-similarity.md) |
| 3 | Sentiment Analysis | [03-sentiment-analysis.md](11-Advanced-NLP-Topics/03-sentiment-analysis.md) |
| 4 | Topic Modeling (LDA) | [04-topic-modeling.md](11-Advanced-NLP-Topics/04-topic-modeling.md) |
| 5 | Information Extraction | [05-information-extraction.md](11-Advanced-NLP-Topics/05-information-extraction.md) |
| 6 | NLP for Low-Resource Languages | [06-low-resource-nlp.md](11-Advanced-NLP-Topics/06-low-resource-nlp.md) |

## Subject Summary

| Unit | Focus Area | Key Techniques |
|------|-----------|----------------|
| 1 | Fundamentals | NLP pipeline, challenges |
| 2 | Preprocessing | Tokenization, stemming, lemmatization |
| 3 | Representation | BoW, TF-IDF, n-grams |
| 4 | Embeddings | Word2Vec, GloVe, FastText |
| 5 | Classification | ML/DL classifiers, evaluation |
| 6 | Seq Labeling | POS, NER, CRF |
| 7 | Language Models | N-gram, neural LMs, transfer learning |
| 8 | Translation | Seq2Seq, attention, BLEU |
| 9 | QA | Extractive, open-domain, knowledge-based |
| 10 | Generation | Decoding, sampling, dialogue |
| 11 | Advanced | Coreference, sentiment, topic modeling |

---

[Previous Subject: 3.4 Transformers](../../03-Deep-Learning/3.4-Transformers-and-Attention/README.md) | [Next Subject: 4.2 Computer Vision](../4.2-Computer-Vision/README.md)
