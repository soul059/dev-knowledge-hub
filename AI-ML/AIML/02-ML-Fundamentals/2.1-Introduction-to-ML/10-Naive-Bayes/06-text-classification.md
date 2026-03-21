# 📘 Chapter 6: Text Classification with Naive Bayes

> **Unit 10: Naive Bayes** | **Section 6 of 7**
> A comprehensive, end-to-end guide to building NLP classification pipelines with Naive Bayes. From raw text to predictions — covering preprocessing, vectorization, model training, spam filtering, sentiment analysis, evaluation, and production-ready code.

---

## 📑 Table of Contents

1. [The Complete NLP Pipeline](#1-the-complete-nlp-pipeline)
2. [Text Preprocessing](#2-text-preprocessing)
3. [Vectorization Methods](#3-vectorization-methods)
4. [Spam Filtering: End-to-End Example](#4-spam-filtering-end-to-end-example)
5. [Sentiment Analysis: End-to-End Example](#5-sentiment-analysis-end-to-end-example)
6. [Pipeline with CountVectorizer + MultinomialNB](#6-pipeline-with-countvectorizer--multinomialnb)
7. [Pipeline with TfidfVectorizer Variant](#7-pipeline-with-tfidfvectorizer-variant)
8. [Model Evaluation](#8-model-evaluation)
9. [Practical Tips & Common Pitfalls](#9-practical-tips--common-pitfalls)
10. [Complete Working Code](#10-complete-working-code)
11. [Summary Table](#11-summary-table)
12. [Revision Questions](#12-revision-questions)

---

## 1. The Complete NLP Pipeline

### 1.1 Architecture Overview

```
    ┌──────────────────────────────────────────────────────────────────────┐
    │                   TEXT CLASSIFICATION PIPELINE                       │
    │                                                                      │
    │  ┌──────────┐   ┌────────────┐   ┌─────────────┐   ┌────────────┐  │
    │  │ Raw Text │──→│ Preprocess │──→│  Vectorize  │──→│  Classify  │  │
    │  │          │   │            │   │             │   │            │  │
    │  │ "Buy NOW │   │ "buy free  │   │ [2,1,0,1,  │   │  Spam:98%  │  │
    │  │  FREE!!" │   │  product"  │   │  0,0,...]  │   │  Ham: 2%   │  │
    │  └──────────┘   └────────────┘   └─────────────┘   └────────────┘  │
    │       │              │                 │                  │          │
    │       │         • Lowercase       • Count or          • Naive      │
    │       │         • Remove punct      TF-IDF              Bayes      │
    │       │         • Tokenize        • n-grams           • Predict    │
    │       │         • Remove stops    • Feature            • Evaluate  │
    │       │         • Stem/Lemma        selection                       │
    │       │                                                              │
    └──────────────────────────────────────────────────────────────────────┘
```

### 1.2 Why Naive Bayes for Text?

```
    ┌─────────────────────────────────────────────────────────┐
    │  Naive Bayes is the GO-TO baseline for text because:   │
    │                                                         │
    │  ✓ Handles high-dimensional sparse data naturally      │
    │  ✓ Training is EXTREMELY fast (single pass)            │
    │  ✓ Scales to millions of documents                     │
    │  ✓ Works well with small training sets                 │
    │  ✓ Competitive accuracy on many text tasks             │
    │  ✓ Easy to interpret (word probabilities per class)    │
    │  ✓ Naturally handles multi-class problems              │
    │  ✓ Incremental learning (partial_fit)                  │
    └─────────────────────────────────────────────────────────┘
```

---

## 2. Text Preprocessing

### 2.1 The Preprocessing Pipeline

```
    Raw text: "I can't believe FREE shipping!! Visit http://spam.com NOW!!!"
    
    Step 1: Lowercase
    → "i can't believe free shipping!! visit http://spam.com now!!!"
    
    Step 2: Remove URLs
    → "i can't believe free shipping!! visit  now!!!"
    
    Step 3: Remove punctuation & special characters
    → "i cant believe free shipping visit now"
    
    Step 4: Tokenize
    → ["i", "cant", "believe", "free", "shipping", "visit", "now"]
    
    Step 5: Remove stop words
    → ["cant", "believe", "free", "shipping", "visit"]
    
    Step 6: Stem/Lemmatize
    → ["cant", "believ", "free", "ship", "visit"]
```

### 2.2 Implementation

```python
import re
import string

def preprocess_text(text):
    """Basic text preprocessing pipeline."""
    # Lowercase
    text = text.lower()
    
    # Remove URLs
    text = re.sub(r'http\S+|www\S+', '', text)
    
    # Remove email addresses
    text = re.sub(r'\S+@\S+', '', text)
    
    # Remove numbers (optional)
    text = re.sub(r'\d+', '', text)
    
    # Remove punctuation
    text = text.translate(str.maketrans('', '', string.punctuation))
    
    # Remove extra whitespace
    text = re.sub(r'\s+', ' ', text).strip()
    
    return text


# Example
raw = "Buy NOW!!! FREE shipping at http://deal.com for $99.99"
print(f"Before: {raw}")
print(f"After:  {preprocess_text(raw)}")
# After:  buy now free shipping at for
```

### 2.3 Advanced Preprocessing with NLTK

```python
import nltk
from nltk.corpus import stopwords
from nltk.stem import PorterStemmer, WordNetLemmatizer
from nltk.tokenize import word_tokenize

# Download required data (first time only)
# nltk.download('punkt')
# nltk.download('stopwords')
# nltk.download('wordnet')

def advanced_preprocess(text):
    """Full NLP preprocessing pipeline."""
    # Lowercase and clean
    text = text.lower()
    text = re.sub(r'http\S+|www\S+', '', text)
    text = re.sub(r'[^a-zA-Z\s]', '', text)
    
    # Tokenize
    tokens = word_tokenize(text)
    
    # Remove stopwords
    stop_words = set(stopwords.words('english'))
    tokens = [t for t in tokens if t not in stop_words and len(t) > 2]
    
    # Lemmatize (or stem)
    lemmatizer = WordNetLemmatizer()
    tokens = [lemmatizer.lemmatize(t) for t in tokens]
    
    return ' '.join(tokens)
```

---

## 3. Vectorization Methods

### 3.1 CountVectorizer (Bag of Words)

```python
from sklearn.feature_extraction.text import CountVectorizer

docs = [
    "I love this movie, it is great",
    "This movie is terrible, I hate it",
    "Great acting and great story",
]

# Basic count vectorization
cv = CountVectorizer()
X = cv.fit_transform(docs)

print("Vocabulary:", cv.get_feature_names_out())
print("Document-Term Matrix:")
print(X.toarray())
```

```
    Vocabulary: ['acting' 'and' 'great' 'hate' 'is' 'it' 'love' 
                 'movie' 'story' 'terrible' 'this']
    
    DTM:
         acting and great hate is it love movie story terrible this
    D1:    0     0    1     0   1  1   1    1     0      0       1
    D2:    0     0    0     1   1  1   0    1     0      1       1
    D3:    1     1    2     0   0  0   0    0     1      0       0
```

### 3.2 TfidfVectorizer

```python
from sklearn.feature_extraction.text import TfidfVectorizer

tfidf = TfidfVectorizer()
X_tfidf = tfidf.fit_transform(docs)

print("TF-IDF Matrix (rounded):")
import numpy as np
print(np.round(X_tfidf.toarray(), 3))
```

### 3.3 Key Vectorizer Parameters

```
    ┌────────────────────────────────────────────────────────────┐
    │  PARAMETER            │ DESCRIPTION            │ TYPICAL  │
    ├────────────────────────┼────────────────────────┼──────────┤
    │  max_features          │ Max vocabulary size     │ 10000    │
    │  ngram_range           │ n-gram range            │ (1,2)    │
    │  stop_words            │ Remove stop words       │'english' │
    │  min_df                │ Min document frequency  │ 2-5      │
    │  max_df                │ Max document frequency  │ 0.95     │
    │  binary                │ Binary counts (0/1)     │ False    │
    │  lowercase             │ Convert to lowercase    │ True     │
    │  strip_accents         │ Remove accents          │'unicode' │
    │  sublinear_tf          │ Apply 1+log(tf)        │ True     │
    │  analyzer              │ 'word' or 'char'       │ 'word'   │
    └────────────────────────┴────────────────────────┴──────────┘
```

### 3.4 N-Grams

```
    Unigrams (1,1):  ["I", "love", "this", "movie"]
    Bigrams (2,2):   ["I love", "love this", "this movie"]
    Uni+Bi (1,2):    ["I", "love", "this", "movie", "I love", 
                       "love this", "this movie"]
    
    Bigrams capture phrases: "not good" vs "not" + "good"
    But increase vocabulary size significantly!
```

---

## 4. Spam Filtering: End-to-End Example

### 4.1 Problem Description

```
    ┌─────────────────────────────────────────────────────────┐
    │  SPAM FILTERING                                        │
    │                                                         │
    │  Input:  Email/SMS text                                 │
    │  Output: Spam (1) or Ham (0)                           │
    │                                                         │
    │  Challenges:                                            │
    │  • Spammers evolve: "fr33", "f.r.e.e", "FREE!!!"     │
    │  • Imbalanced data (usually more ham than spam)        │
    │  • False positives are costly (legit email → spam)     │
    │  • Need to be very fast (real-time filtering)          │
    │                                                         │
    │  Why Naive Bayes?                                       │
    │  • First practical spam filter (Paul Graham, 2002)     │
    │  • Fast enough for real-time email processing          │
    │  • Incremental learning (update with new spam)         │
    │  • Interpretable (which words trigger spam?)           │
    └─────────────────────────────────────────────────────────┘
```

### 4.2 Complete Spam Filter

```python
import numpy as np
from sklearn.feature_extraction.text import CountVectorizer, TfidfVectorizer
from sklearn.naive_bayes import MultinomialNB
from sklearn.model_selection import train_test_split
from sklearn.metrics import classification_report, confusion_matrix
from sklearn.pipeline import Pipeline

# ─── Training Data ──────────────────────────────────────────
emails = [
    # Spam
    "Congratulations! You've won a $1000 gift card. Click here to claim.",
    "FREE prescription medications! Order online now and save big!",
    "URGENT: Your bank account has been compromised. Click to verify.",
    "Win a brand new iPhone 15! Just answer this simple survey.",
    "Make money from home! Earn $5000/week with this simple trick.",
    "Limited time offer: 90% OFF all products! Shop now!",
    "You've been selected for a exclusive cash prize! Act now!",
    "Hot singles in your area want to meet you tonight!",
    "Lose 30 pounds in 30 days! Revolutionary new diet pill!",
    "Dear friend, I need your help transferring $10 million USD.",
    "Buy cheap medications online. No prescription needed!",
    "Congratulations winner! Forward this to claim your reward.",
    "Get rich quick! Investment opportunity of a lifetime!",
    "Free laptop giveaway! Click the link below to enter.",
    "Your PayPal account needs immediate attention. Verify now.",
    # Ham
    "Hi team, the project meeting is rescheduled to 3pm.",
    "Can you review the quarterly report before Friday?",
    "Thanks for the great presentation yesterday.",
    "Please find the attached invoice for last month's services.",
    "The new software update will be deployed next Tuesday.",
    "Let's schedule a call to discuss the client proposal.",
    "Happy birthday! Hope you have a wonderful day.",
    "Reminder: team lunch at the new Italian restaurant tomorrow.",
    "I've updated the documentation. Please review when you can.",
    "The meeting notes from today's standup are attached.",
    "Could you send me the latest sales figures?",
    "Welcome aboard! Your onboarding materials are ready.",
    "The conference room is booked for our 2pm discussion.",
    "Thanks for your help with the database migration.",
    "Just checking in — how's the new feature coming along?",
]
labels = [1]*15 + [0]*15

# Split data
X_train, X_test, y_train, y_test = train_test_split(
    emails, labels, test_size=0.3, random_state=42, stratify=labels
)

# Build pipeline
spam_pipeline = Pipeline([
    ('vectorizer', CountVectorizer(
        stop_words='english',
        max_features=5000,
        ngram_range=(1, 2),
    )),
    ('classifier', MultinomialNB(alpha=0.1)),
])

# Train
spam_pipeline.fit(X_train, y_train)

# Evaluate
y_pred = spam_pipeline.predict(X_test)
print("SPAM FILTER EVALUATION")
print("=" * 50)
print(classification_report(y_test, y_pred, target_names=['Ham', 'Spam']))

# Test on new emails
test_emails = [
    "Free money! Click here to claim your prize now!",
    "Hi, can we move our meeting to Thursday?",
    "URGENT: Verify your account immediately!",
    "The quarterly report looks great, thanks!",
]

predictions = spam_pipeline.predict(test_emails)
probabilities = spam_pipeline.predict_proba(test_emails)

print("\nNew Email Classifications:")
print("-" * 60)
for email, pred, proba in zip(test_emails, predictions, probabilities):
    label = "🚫 SPAM" if pred == 1 else "✉️ HAM"
    conf = max(proba) * 100
    print(f"  {label} ({conf:.1f}%): '{email[:50]}...'")
```

### 4.3 Most Informative Features

```python
# Show which words are most indicative of spam vs ham
feature_names = spam_pipeline.named_steps['vectorizer'].get_feature_names_out()
log_probs = spam_pipeline.named_steps['classifier'].feature_log_prob_

# Difference in log probabilities between spam and ham
log_ratio = log_probs[1] - log_probs[0]  # positive = more spammy

# Top spam words
top_spam_idx = np.argsort(log_ratio)[-10:][::-1]
print("TOP SPAM INDICATORS:")
for idx in top_spam_idx:
    print(f"  '{feature_names[idx]}': log ratio = {log_ratio[idx]:.3f}")

# Top ham words
top_ham_idx = np.argsort(log_ratio)[:10]
print("\nTOP HAM INDICATORS:")
for idx in top_ham_idx:
    print(f"  '{feature_names[idx]}': log ratio = {log_ratio[idx]:.3f}")
```

---

## 5. Sentiment Analysis: End-to-End Example

### 5.1 Problem Description

```
    ┌─────────────────────────────────────────────────────────┐
    │  SENTIMENT ANALYSIS                                     │
    │                                                         │
    │  Input:  Review/tweet text                              │
    │  Output: Positive (1) or Negative (0)                  │
    │                                                         │
    │  Challenges:                                            │
    │  • Sarcasm: "Oh great, another bug"                    │
    │  • Negation: "not bad" = positive                      │
    │  • Context: "unpredictable" (bad for car, good for     │
    │    movie thriller)                                      │
    │  • Mixed sentiment: "food great, service terrible"     │
    │                                                         │
    │  Naive Bayes handles:                                   │
    │  • Direct sentiment words well                         │
    │  • Struggles with negation and sarcasm                  │
    │  • Bigrams help capture "not good" patterns            │
    └─────────────────────────────────────────────────────────┘
```

### 5.2 Complete Sentiment Analyzer

```python
from sklearn.feature_extraction.text import TfidfVectorizer
from sklearn.naive_bayes import MultinomialNB
from sklearn.pipeline import Pipeline
from sklearn.model_selection import cross_val_score
import numpy as np

# Movie reviews (simplified dataset)
reviews = [
    # Positive
    "This movie was absolutely wonderful and entertaining",
    "Brilliant performances by the entire cast, loved it",
    "A masterpiece of storytelling, highly recommended",
    "Beautiful cinematography and excellent direction",
    "One of the best films I've seen this year",
    "Incredible acting and a heartwarming story",
    "Thoroughly enjoyed every minute of this film",
    "Outstanding performances and gripping plot",
    "A delightful and charming movie experience",
    "Superb writing and brilliant execution",
    "I really enjoyed this movie, great entertainment",
    "Amazing special effects and wonderful story",
    "Fantastic movie with great performances",
    "Loved the characters and the storyline",
    "A truly remarkable and inspiring film",
    # Negative
    "Terrible movie, waste of time and money",
    "Boring plot with awful acting throughout",
    "One of the worst films I have ever seen",
    "Poor direction and a confusing storyline",
    "Dull and predictable, not worth watching",
    "Absolutely dreadful, avoid at all costs",
    "The acting was wooden and unconvincing",
    "A complete disaster from start to finish",
    "Painful to watch, terrible script",
    "Disappointing and poorly executed film",
    "I hated every moment of this awful movie",
    "Weak plot and terrible dialogue throughout",
    "Boring and uninspired, a complete letdown",
    "Worst movie of the year without doubt",
    "Poorly written and badly directed film",
]
sentiments = [1]*15 + [0]*15

# Build sentiment pipeline with TF-IDF
sentiment_pipeline = Pipeline([
    ('tfidf', TfidfVectorizer(
        ngram_range=(1, 2),
        max_features=5000,
        stop_words='english',
        sublinear_tf=True,
    )),
    ('clf', MultinomialNB(alpha=0.1)),
])

# Cross-validation
cv_scores = cross_val_score(sentiment_pipeline, reviews, sentiments, cv=5)
print(f"Cross-validation accuracy: {cv_scores.mean():.4f} ± {cv_scores.std():.4f}")

# Train on full data
sentiment_pipeline.fit(reviews, sentiments)

# Test on new reviews
test_reviews = [
    "An amazing movie with stunning visuals",
    "Absolutely terrible, boring from start to end",
    "Decent acting but the plot was weak",
    "A wonderful experience, highly recommended",
]

preds = sentiment_pipeline.predict(test_reviews)
probas = sentiment_pipeline.predict_proba(test_reviews)

print("\nSentiment Predictions:")
print("-" * 60)
for review, pred, proba in zip(test_reviews, preds, probas):
    emoji = "👍" if pred == 1 else "👎"
    label = "Positive" if pred == 1 else "Negative"
    conf = max(proba) * 100
    print(f"  {emoji} {label} ({conf:.1f}%): '{review[:50]}'")
```

---

## 6. Pipeline with CountVectorizer + MultinomialNB

### 6.1 The Standard Pipeline

```python
from sklearn.pipeline import Pipeline
from sklearn.feature_extraction.text import CountVectorizer
from sklearn.naive_bayes import MultinomialNB

# The classic text classification pipeline
text_clf = Pipeline([
    ('vect', CountVectorizer(
        strip_accents='unicode',
        stop_words='english',
        lowercase=True,
        max_features=10000,
        ngram_range=(1, 1),
        min_df=2,
    )),
    ('clf', MultinomialNB(
        alpha=1.0,
        fit_prior=True,
    )),
])

# Usage
text_clf.fit(train_texts, train_labels)
predictions = text_clf.predict(test_texts)
probabilities = text_clf.predict_proba(test_texts)
```

### 6.2 Pipeline Advantages

```
    ┌─────────────────────────────────────────────────────────┐
    │  WHY USE Pipeline?                                      │
    │                                                         │
    │  1. PREVENTS DATA LEAKAGE                               │
    │     Vectorizer fitted only on training data             │
    │     No information from test set leaks into training    │
    │                                                         │
    │  2. CLEAN CODE                                          │
    │     One object for the entire workflow                  │
    │     fit(), predict(), score() work end-to-end           │
    │                                                         │
    │  3. EASY CROSS-VALIDATION                               │
    │     cross_val_score handles train/test splitting        │
    │     correctly at each fold                              │
    │                                                         │
    │  4. GRID SEARCH FRIENDLY                                │
    │     Tune vectorizer AND classifier parameters together  │
    │     Use step__param syntax                              │
    │                                                         │
    │  5. DEPLOYMENT READY                                    │
    │     Serialize one object (pickle/joblib)                │
    │     Load and predict on raw text directly               │
    └─────────────────────────────────────────────────────────┘
```

---

## 7. Pipeline with TfidfVectorizer Variant

### 7.1 TF-IDF Pipeline

```python
from sklearn.pipeline import Pipeline
from sklearn.feature_extraction.text import TfidfVectorizer
from sklearn.naive_bayes import MultinomialNB

# TF-IDF variant (often better performance)
tfidf_clf = Pipeline([
    ('tfidf', TfidfVectorizer(
        strip_accents='unicode',
        stop_words='english',
        lowercase=True,
        max_features=15000,
        ngram_range=(1, 2),      # include bigrams
        min_df=2,
        max_df=0.95,
        sublinear_tf=True,       # use 1 + log(tf)
        norm='l2',               # L2 normalization
    )),
    ('clf', MultinomialNB(
        alpha=0.1,               # less smoothing for TF-IDF
    )),
])
```

### 7.2 Count vs TF-IDF: When to Choose

```
    ┌───────────────────────────────────────────────────────────┐
    │                                                           │
    │  COUNT + MultinomialNB:                                  │
    │  • Simpler, faster                                       │
    │  • True probabilistic model                              │
    │  • Better when word frequency is directly informative    │
    │  • Good baseline                                         │
    │                                                           │
    │  TF-IDF + MultinomialNB:                                 │
    │  • Down-weights common words automatically               │
    │  • Better when documents vary in length                  │
    │  • Often higher accuracy (empirically)                   │
    │  • Technically not a proper multinomial input,            │
    │    but works well in practice                            │
    │                                                           │
    │  Rule of thumb: Start with Count, try TF-IDF,           │
    │  keep whichever gives better CV score.                   │
    └───────────────────────────────────────────────────────────┘
```

---

## 8. Model Evaluation

### 8.1 Key Metrics for Text Classification

```python
from sklearn.metrics import (
    accuracy_score, precision_score, recall_score, 
    f1_score, confusion_matrix, classification_report,
    roc_auc_score
)
import numpy as np

def evaluate_text_classifier(y_true, y_pred, y_proba=None, 
                              class_names=None):
    """Comprehensive evaluation of a text classifier."""
    
    print("=" * 60)
    print("MODEL EVALUATION")
    print("=" * 60)
    
    # Basic metrics
    print(f"\nAccuracy:  {accuracy_score(y_true, y_pred):.4f}")
    print(f"Precision: {precision_score(y_true, y_pred, average='weighted'):.4f}")
    print(f"Recall:    {recall_score(y_true, y_pred, average='weighted'):.4f}")
    print(f"F1 Score:  {f1_score(y_true, y_pred, average='weighted'):.4f}")
    
    if y_proba is not None and len(np.unique(y_true)) == 2:
        print(f"ROC AUC:   {roc_auc_score(y_true, y_proba[:, 1]):.4f}")
    
    # Classification report
    print(f"\nClassification Report:")
    print(classification_report(y_true, y_pred, 
                                target_names=class_names))
    
    # Confusion matrix
    cm = confusion_matrix(y_true, y_pred)
    print(f"Confusion Matrix:")
    if class_names:
        header = "  " + "  ".join(f"{n:>8}" for n in class_names)
        print(f"  Predicted: {header}")
    for i, row in enumerate(cm):
        label = class_names[i] if class_names else f"Class {i}"
        print(f"  {label:>10}: {row}")
```

### 8.2 Understanding the Confusion Matrix

```
    For Spam Detection:
    
                        Predicted
                    Ham         Spam
    Actual  Ham  [  950  │    50  ]     ← 50 false positives (legit → spam)
            Spam [   30  │   970  ]     ← 30 false negatives (spam → inbox)
    
    FALSE POSITIVE (Type I):  Legitimate email classified as spam
      → User MISSES important email! Very costly.
    
    FALSE NEGATIVE (Type II): Spam email classified as legitimate
      → User sees spam in inbox. Annoying but less costly.
    
    For spam filtering: PRECISION is often more important
    (minimize false positives — don't lose real emails!)
```

### 8.3 Hyperparameter Tuning

```python
from sklearn.model_selection import GridSearchCV
from sklearn.pipeline import Pipeline
from sklearn.feature_extraction.text import TfidfVectorizer
from sklearn.naive_bayes import MultinomialNB

# Define pipeline
pipeline = Pipeline([
    ('tfidf', TfidfVectorizer()),
    ('clf', MultinomialNB()),
])

# Parameter grid
param_grid = {
    'tfidf__max_features': [5000, 10000, 20000, None],
    'tfidf__ngram_range': [(1, 1), (1, 2), (1, 3)],
    'tfidf__stop_words': [None, 'english'],
    'tfidf__sublinear_tf': [True, False],
    'tfidf__min_df': [1, 2, 5],
    'clf__alpha': [0.01, 0.1, 0.5, 1.0, 2.0],
}

# Grid search
grid_search = GridSearchCV(
    pipeline, param_grid,
    cv=5,
    scoring='f1_weighted',
    n_jobs=-1,
    verbose=1,
)

grid_search.fit(train_texts, train_labels)

print(f"Best score: {grid_search.best_score_:.4f}")
print(f"Best params: {grid_search.best_params_}")
```

---

## 9. Practical Tips & Common Pitfalls

### 9.1 Tips for Better Performance

```
    ┌─────────────────────────────────────────────────────────────┐
    │  TIPS FOR BETTER NAIVE BAYES TEXT CLASSIFICATION           │
    │                                                             │
    │  1. PREPROCESSING                                          │
    │     • Remove HTML tags, URLs, email addresses              │
    │     • Handle contractions ("can't" → "cannot")             │
    │     • Consider stemming vs lemmatization                   │
    │                                                             │
    │  2. FEATURE ENGINEERING                                    │
    │     • Try bigrams (ngram_range=(1,2))                     │
    │     • Use sublinear_tf=True with TF-IDF                   │
    │     • Set min_df=2-5 to remove very rare words            │
    │     • Set max_df=0.95 to remove very common words         │
    │     • Experiment with max_features (5000-50000)           │
    │                                                             │
    │  3. SMOOTHING                                              │
    │     • Tune alpha! Default 1.0 is often not optimal        │
    │     • Try alpha in [0.01, 0.1, 0.5, 1.0]                 │
    │     • Smaller alpha with TF-IDF (e.g., 0.01-0.1)         │
    │                                                             │
    │  4. EVALUATION                                             │
    │     • Always use cross-validation                          │
    │     • Check precision/recall per class                     │
    │     • Watch for class imbalance                            │
    │                                                             │
    │  5. PRODUCTION                                             │
    │     • Use Pipeline (prevents data leakage)                │
    │     • Serialize model with joblib                         │
    │     • Monitor for concept drift                           │
    └─────────────────────────────────────────────────────────────┘
```

### 9.2 Common Pitfalls

```
    ┌─────────────────────────────────────────────────────────────┐
    │  ❌ COMMON MISTAKES                                        │
    │                                                             │
    │  1. FITTING VECTORIZER ON ALL DATA (data leakage!)        │
    │     Wrong:  vec.fit_transform(ALL_data)                    │
    │     Right:  vec.fit_transform(train) → vec.transform(test)│
    │     Best:   Use Pipeline!                                  │
    │                                                             │
    │  2. IGNORING CLASS IMBALANCE                               │
    │     If 95% ham, 5% spam → model predicts all ham (95% acc)│
    │     Fix: Use F1 score, stratified splits, class_prior     │
    │                                                             │
    │  3. TOO LARGE VOCABULARY                                   │
    │     50,000+ features with 1000 docs → overfitting         │
    │     Fix: max_features, min_df, max_df                     │
    │                                                             │
    │  4. NOT TUNING ALPHA                                       │
    │     Default alpha=1.0 is safe but rarely optimal          │
    │     Fix: GridSearchCV over alpha values                    │
    │                                                             │
    │  5. TRUSTING PROBABILITY OUTPUTS                           │
    │     NB probabilities are poorly calibrated                 │
    │     Fix: Use CalibratedClassifierCV if probs needed       │
    └─────────────────────────────────────────────────────────────┘
```

---

## 10. Complete Working Code

### 10.1 Production-Ready Text Classifier

```python
"""
Complete text classification system using Naive Bayes.
Handles preprocessing, training, evaluation, and prediction.
"""

import numpy as np
import re
import string
from sklearn.feature_extraction.text import TfidfVectorizer, CountVectorizer
from sklearn.naive_bayes import MultinomialNB, BernoulliNB
from sklearn.pipeline import Pipeline
from sklearn.model_selection import (
    train_test_split, cross_val_score, GridSearchCV, StratifiedKFold
)
from sklearn.metrics import (
    classification_report, confusion_matrix, accuracy_score
)


# ═══════════════════════════════════════════════════════════
# 1. DATA PREPARATION
# ═══════════════════════════════════════════════════════════

# Comprehensive email dataset
emails = [
    # ─── SPAM (label=1) ───
    "Congratulations! You won $1000. Click here to claim now!",
    "FREE VIAGRA! Buy online cheap medications no prescription",
    "URGENT: Your bank account compromised verify immediately",
    "Win brand new iPhone! Answer this survey to enter",
    "Make $5000 weekly from home guaranteed income opportunity",
    "Limited time 90% OFF everything! Shop now free shipping",
    "Selected for exclusive cash prize! Claim before deadline",
    "Hot singles in your area want to chat tonight click now",
    "Lose weight fast! New miracle diet pill order today",
    "Dear friend help transfer $10 million USD commission yours",
    "Buy cheap watches online replica designer brands sale",
    "Forward this to claim your lottery reward congratulations",
    "Get rich investment opportunity lifetime returns guaranteed",
    "Free laptop giveaway enter now click the link below",
    "PayPal account suspended verify identity click here urgent",
    "Act now limited offer buy one get three free discount",
    "Your loan approved $50000 low interest apply now click",
    "Secret to making money online revealed free webinar today",
    "Enlarge your portfolio with these hot stock picks free",
    "Claim your free gift card $500 Amazon no purchase needed",
    # ─── HAM (label=0) ───
    "Hi team, project meeting rescheduled to 3pm tomorrow",
    "Can you review the quarterly report before Friday?",
    "Thanks for the great presentation yesterday!",
    "Please find attached invoice for last month services",
    "New software update deployed next Tuesday maintenance",
    "Let's schedule a call to discuss client proposal",
    "Happy birthday! Hope you have a wonderful celebration",
    "Reminder: team lunch at Italian restaurant tomorrow noon",
    "Updated documentation ready for your review",
    "Meeting notes from today's standup attached",
    "Could you send me the latest sales figures please?",
    "Welcome aboard! Your onboarding materials are ready",
    "Conference room booked for 2pm discussion today",
    "Thanks for help with the database migration this week",
    "How's the new feature development coming along?",
    "Please approve the budget for Q3 marketing campaign",
    "The code review comments have been addressed",
    "Attached is the revised project timeline for review",
    "Team building event scheduled for next Friday afternoon",
    "Monthly newsletter: company updates and announcements",
]
labels = [1]*20 + [0]*20

# ═══════════════════════════════════════════════════════════
# 2. PREPROCESSING
# ═══════════════════════════════════════════════════════════

def clean_text(text):
    """Clean and normalize text."""
    text = text.lower()
    text = re.sub(r'http\S+|www\S+', 'URL', text)
    text = re.sub(r'\$\d+[\d,]*', 'MONEY', text)
    text = re.sub(r'\d+%', 'PERCENT', text)
    text = re.sub(r'[^\w\s]', ' ', text)
    text = re.sub(r'\s+', ' ', text).strip()
    return text

emails_clean = [clean_text(e) for e in emails]

# ═══════════════════════════════════════════════════════════
# 3. TRAIN/TEST SPLIT
# ═══════════════════════════════════════════════════════════

X_train, X_test, y_train, y_test = train_test_split(
    emails_clean, labels, test_size=0.25, random_state=42, stratify=labels
)

# ═══════════════════════════════════════════════════════════
# 4. BUILD & COMPARE PIPELINES
# ═══════════════════════════════════════════════════════════

pipelines = {
    'Count + MNB': Pipeline([
        ('vec', CountVectorizer(stop_words='english', ngram_range=(1, 2))),
        ('clf', MultinomialNB(alpha=0.1)),
    ]),
    'TF-IDF + MNB': Pipeline([
        ('vec', TfidfVectorizer(stop_words='english', ngram_range=(1, 2),
                                sublinear_tf=True)),
        ('clf', MultinomialNB(alpha=0.1)),
    ]),
    'Binary + BNB': Pipeline([
        ('vec', CountVectorizer(stop_words='english', ngram_range=(1, 2),
                                binary=True)),
        ('clf', BernoulliNB(alpha=0.1, binarize=None)),
    ]),
}

print("MODEL COMPARISON")
print("=" * 55)
print(f"{'Model':<20} {'CV Accuracy':>15} {'Test Accuracy':>15}")
print("-" * 55)

best_name, best_score = None, 0
for name, pipe in pipelines.items():
    cv_scores = cross_val_score(pipe, X_train, y_train, cv=3, scoring='accuracy')
    pipe.fit(X_train, y_train)
    test_acc = pipe.score(X_test, y_test)
    print(f"{name:<20} {cv_scores.mean():>12.4f}±{cv_scores.std():.3f}"
          f" {test_acc:>14.4f}")
    if cv_scores.mean() > best_score:
        best_score = cv_scores.mean()
        best_name = name

print(f"\nBest model: {best_name}")

# ═══════════════════════════════════════════════════════════
# 5. DETAILED EVALUATION OF BEST MODEL
# ═══════════════════════════════════════════════════════════

best_pipeline = pipelines[best_name]
y_pred = best_pipeline.predict(X_test)

print(f"\n{'='*55}")
print(f"DETAILED EVALUATION: {best_name}")
print(f"{'='*55}")
print(classification_report(y_test, y_pred, target_names=['Ham', 'Spam']))

cm = confusion_matrix(y_test, y_pred)
print("Confusion Matrix:")
print(f"              Pred Ham  Pred Spam")
print(f"  Actual Ham  [{cm[0,0]:>5}    {cm[0,1]:>5}  ]")
print(f"  Actual Spam [{cm[1,0]:>5}    {cm[1,1]:>5}  ]")

# ═══════════════════════════════════════════════════════════
# 6. PREDICTION ON NEW EMAILS
# ═══════════════════════════════════════════════════════════

new_emails = [
    "Free money! Click here to claim your prize now!",
    "Hi, can we reschedule the team meeting to Thursday?",
    "URGENT: Verify your bank account details immediately!",
    "The quarterly earnings report looks great, nice work!",
    "Buy cheap products online 90% discount free shipping",
    "Please review the attached proposal before tomorrow",
]

print(f"\n{'='*55}")
print("NEW EMAIL PREDICTIONS")
print(f"{'='*55}")
for email in new_emails:
    cleaned = clean_text(email)
    pred = best_pipeline.predict([cleaned])[0]
    proba = best_pipeline.predict_proba([cleaned])[0]
    label = "🚫 SPAM" if pred == 1 else "✉️ HAM "
    conf = max(proba) * 100
    print(f"  {label} ({conf:5.1f}%): '{email[:55]}'")

# ═══════════════════════════════════════════════════════════
# 7. FEATURE IMPORTANCE
# ═══════════════════════════════════════════════════════════

feature_names = best_pipeline.named_steps['vec'].get_feature_names_out()
log_probs = best_pipeline.named_steps['clf'].feature_log_prob_
log_ratio = log_probs[1] - log_probs[0]  # Spam vs Ham

print(f"\n{'='*55}")
print("MOST INFORMATIVE FEATURES")
print(f"{'='*55}")

top_spam = np.argsort(log_ratio)[-8:][::-1]
top_ham = np.argsort(log_ratio)[:8]

print("\nTop SPAM words:")
for idx in top_spam:
    print(f"  {feature_names[idx]:<20} (ratio: {np.exp(log_ratio[idx]):.2f}x)")

print("\nTop HAM words:")
for idx in top_ham:
    print(f"  {feature_names[idx]:<20} (ratio: {np.exp(-log_ratio[idx]):.2f}x)")
```

---

## 11. Summary Table

| Concept | Detail | Key Insight |
|---------|--------|-------------|
| **NLP Pipeline** | Text → Preprocess → Vectorize → Classify | End-to-end workflow |
| **Preprocessing** | Lowercase, remove punct, tokenize, stem | Clean text first |
| **CountVectorizer** | Raw word counts → DTM | Bag-of-words baseline |
| **TfidfVectorizer** | TF-IDF weighted features | Down-weights common words |
| **ngram_range** | (1,1)=unigrams, (1,2)=uni+bigrams | Bigrams capture phrases |
| **Pipeline** | Vectorizer + Classifier in one object | Prevents data leakage |
| **GridSearchCV** | Tune vec + clf params together | Find optimal config |
| **alpha tuning** | Try [0.01, 0.1, 0.5, 1.0] | Rarely optimal at default |
| **Spam filtering** | Precision matters most | Don't lose real emails |
| **Sentiment** | Bigrams help ("not good") | Context from word pairs |
| **Feature importance** | Log probability ratios | Interpret the model |
| **Calibration** | CalibratedClassifierCV | Fix probability outputs |

---

## 12. Revision Questions

### Q1: Pipeline Design
**Q:** Why is it critical to use sklearn's Pipeline instead of fitting the vectorizer and classifier separately? Give a specific example of what goes wrong without it.

**A:** Without Pipeline, a common mistake is: `vec.fit_transform(ALL_data)` then split into train/test. This causes **data leakage** — the vectorizer learns vocabulary and IDF weights from test data. Example: if a word appears only in test documents, without Pipeline it gets a feature column (IDF computed from all data). With Pipeline, that word would be unknown at test time (not in vocabulary). Data leakage inflates accuracy metrics — the model appears better than it truly is. Pipeline ensures `vec.fit_transform(X_train)` during training and `vec.transform(X_test)` during evaluation, maintaining strict separation.

---

### Q2: Vectorizer Choice
**Q:** You have a corpus of 10,000 product reviews averaging 200 words each. Would you recommend CountVectorizer or TfidfVectorizer? What specific parameters would you set and why?

**A:** **TfidfVectorizer** — for longer documents, TF-IDF is generally better because it down-weights common words that appear in many reviews (like "product", "good", "the"). Recommended parameters:
- `ngram_range=(1,2)`: capture "not good", "highly recommend" phrases
- `max_features=15000`: limit vocabulary to most common terms
- `stop_words='english'`: remove basic stop words
- `min_df=3`: ignore words appearing in fewer than 3 reviews (typos, unique names)
- `max_df=0.90`: ignore words in >90% of reviews (too common to be useful)
- `sublinear_tf=True`: apply 1+log(tf) to dampen very high counts

---

### Q3: Alpha Tuning Impact
**Q:** A MultinomialNB spam filter with alpha=1.0 has 92% accuracy. After tuning, alpha=0.01 gives 96% accuracy. Explain conceptually why smaller alpha helped and when it could hurt.

**A:** alpha=1.0 adds 1 pseudo-count to every word, which heavily smooths probabilities toward uniform. For common words, this barely matters, but for rare-but-informative words (e.g., "lottery" appearing 2 times in spam), adding 1 pseudo-count to each class significantly dilutes the signal. alpha=0.01 barely perturbs the observed counts, preserving discriminative power. It could hurt when: (1) training data is very small (not enough counts for reliable estimates, smoothing helps generalize), (2) vocabulary is much larger than document count (many zero-count words need smoothing), (3) test data contains many words unseen in training (need non-zero probabilities for them).

---

### Q4: N-gram Selection
**Q:** Why do bigrams help with sentiment analysis but might hurt spam detection? Discuss with specific examples.

**A:** **Sentiment benefits from bigrams:** Negation changes meaning completely — "not good" is negative but unigrams "not" and "good" separately might signal opposite sentiments. "highly recommended" captures stronger positive signal than "highly" + "recommended" separately. **Spam may not benefit:** Spam indicators are often individual words ("free", "win", "urgent", "click"). Bigrams like "click here" aren't much more informative than "click" alone. Adding bigrams roughly squares the vocabulary size, increasing dimensionality without proportional information gain. With limited spam data, the sparser bigram space may lead to overfitting. **Recommendation:** Always try both (1,1) and (1,2) with cross-validation; the optimal choice is task-dependent.

---

### Q5: Handling Imbalanced Data
**Q:** Your spam dataset has 95% ham and 5% spam. A model predicting everything as ham gets 95% accuracy. How would you properly evaluate and improve a Naive Bayes spam filter on this data?

**A:**
- **Evaluation:** Use F1-score (especially for spam class), precision, recall, and AUC-ROC instead of accuracy. Use stratified cross-validation to maintain class ratio in each fold.
- **Improvements:** (1) Set `class_prior=[0.5, 0.5]` in MultinomialNB to prevent the prior from dominating; (2) Use `StratifiedKFold` for cross-validation; (3) Optimize for F1 in GridSearchCV (`scoring='f1'`); (4) Consider the cost of false positives vs false negatives — tune the classification threshold; (5) Use `predict_proba()` and adjust threshold rather than using default 0.5; (6) Report precision and recall separately for the minority (spam) class.

---

### Q6: Feature Importance
**Q:** After training a MultinomialNB spam filter, you examine the top features and find "the" has one of the highest log probabilities for both spam and ham. Why doesn't this hurt classification, and how can you mitigate it?

**A:** "the" has high absolute log probability in both classes because it's common everywhere. But for classification, what matters is the **difference** in log probabilities between classes: log P("the"|spam) - log P("the"|ham) ≈ 0 (similar in both). So "the" contributes approximately equally to both scores, effectively canceling out. It wastes a feature dimension but doesn't hurt the decision. **Mitigation:** (1) `stop_words='english'` removes "the" and similar words; (2) `max_df=0.95` removes words in >95% of documents; (3) TfidfVectorizer with IDF weighting automatically down-weights "the" (IDF ≈ 0 for words in all documents).

---

## 🧭 Navigation

| Previous | Current | Next |
|----------|---------|------|
| [← Chapter 5: Bernoulli Naive Bayes](05-bernoulli-naive-bayes.md) | **Chapter 6: Text Classification** | [Chapter 7: Laplace Smoothing →](07-laplace-smoothing.md) |

### Unit 10 Chapters
1. [Bayes' Theorem Review](01-bayes-theorem-review.md)
2. [The Naive Assumption](02-naive-assumption.md)
3. [Gaussian Naive Bayes](03-gaussian-naive-bayes.md)
4. [Multinomial Naive Bayes](04-multinomial-naive-bayes.md)
5. [Bernoulli Naive Bayes](05-bernoulli-naive-bayes.md)
6. **📍 Text Classification with Naive Bayes** ← You are here
7. [Laplace Smoothing & Practical Considerations](07-laplace-smoothing.md)

---

> *"Naive Bayes turned spam filtering from an unsolved problem into a solved one — Paul Graham's 2002 essay 'A Plan for Spam' started a revolution."*

© 2025 AI/ML Study Notes. All rights reserved.
