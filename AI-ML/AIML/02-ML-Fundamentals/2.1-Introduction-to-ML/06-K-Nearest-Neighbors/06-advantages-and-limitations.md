# ⚡ Unit 6, Chapter 6: Advantages, Limitations & Speedup Techniques

> **K-Nearest Neighbors — Part 6 of 6**
>
> KNN is beautifully simple — but simplicity comes with tradeoffs. This final chapter provides a thorough assessment of KNN's strengths and weaknesses, explores data structures that make it faster (KD-Trees, Ball Trees), introduces approximate nearest neighbor methods, and compares KNN against other algorithms to help you decide when to use it.

---

## Table of Contents

1. [Advantages of KNN](#1-advantages-of-knn)
2. [Limitations of KNN](#2-limitations-of-knn)
3. [Computational Complexity Analysis](#3-computational-complexity-analysis)
4. [KD-Tree for Speedup](#4-kd-tree-for-speedup)
5. [Ball Tree for Speedup](#5-ball-tree-for-speedup)
6. [Approximate Nearest Neighbors](#6-approximate-nearest-neighbors)
7. [When to Use KNN vs Other Algorithms](#7-when-to-use-knn-vs-other-algorithms)
8. [Comprehensive Comparison Table](#8-comprehensive-comparison-table)
9. [Best Practices Checklist](#9-best-practices-checklist)
10. [Summary Table](#10-summary-table)
11. [Revision Questions](#11-revision-questions)

---

## 1. Advantages of KNN

### ✅ 1.1 Simplicity & Intuition

```
  KNN is the SIMPLEST ML algorithm to understand and explain:

  "Find the K most similar examples and go with the majority."

  ┌────────────────────────────────────────────────┐
  │  No complex math formulas                     │
  │  No gradient descent                          │
  │  No loss functions to optimize                │
  │  No hyperplane equations                      │
  │                                                │
  │  Just: compute distances → find nearest → vote │
  └────────────────────────────────────────────────┘

  Perfect for:
  - Teaching ML concepts
  - Prototyping / baselines
  - When explainability matters ("your loan was denied because
    5 similar applicants defaulted")
```

### ✅ 1.2 No Training Phase

```
  Training Time Comparison:

  ┌─────────────────────┬─────────────────────────┐
  │  Algorithm          │  Training Time          │
  ├─────────────────────┼─────────────────────────┤
  │  KNN                │  O(1) — just store data │
  │  Linear Regression  │  O(n·d²)                │
  │  SVM                │  O(n² to n³)            │
  │  Random Forest      │  O(T·n·d·log n)         │
  │  Neural Network     │  O(epochs·n·weights)    │
  └─────────────────────┴─────────────────────────┘

  KNN is a "lazy learner" — all computation happens at prediction time.
  This is great when:
  - Data changes frequently (no retraining needed)
  - You need instant model deployment
  - Training time is a constraint
```

### ✅ 1.3 Naturally Handles Non-Linear Boundaries

```
  KNN can model ANY decision boundary shape:

  Linear boundary          Non-linear boundary         Complex boundary
  (easy for most algos)    (hard for linear models)    (KNN handles it!)

  ┌──────────┐             ┌──────────┐                ┌──────────┐
  │● ●  │○ ○ │             │● ●  ╱○ ○ │                │● ○ ● ○ ● │
  │● ●  │○ ○ │             │● ● ╱ ○ ○ │                │ ○ ● ○ ● ○│
  │● ●  │○ ○ │             │● ╱  ○ ○ ○│                │● ○ ● ○ ● │
  │● ●  │○ ○ │             │╱ ○ ○ ○ ○ │                │ ○ ● ○ ● ○│
  └──────────┘             └──────────┘                └──────────┘

  KNN doesn't assume any shape — it adapts to the local data structure.
  No functional form (linear, polynomial, etc.) needs to be specified.
```

### ✅ 1.4 Multi-Class Without Modification

```
  KNN handles any number of classes naturally:

  Binary:     Just count votes for 2 classes
  3-class:    Count votes for 3 classes
  100-class:  Count votes for 100 classes

  No need for One-vs-Rest, One-vs-One, or other multi-class strategies.
  The algorithm is IDENTICAL regardless of number of classes.
```

### ✅ 1.5 Naturally Handles Both Classification & Regression

```
  Same algorithm, different aggregation:
  - Classification: majority vote
  - Regression: average of neighbors

  Most algorithms need fundamentally different implementations for each.
```

### ✅ 1.6 Online Learning

```
  Adding new data is trivial:
  
  Old model:  Store {x₁,...,xₙ}
  New data:   Append x_{n+1}
  New model:  Store {x₁,...,xₙ, x_{n+1}}

  No retraining needed! Just add the new point to storage.
  (Compare: SVM, Neural Networks need complete retraining)
```

---

## 2. Limitations of KNN

### ❌ 2.1 Slow Prediction (High Computational Cost)

```
  For EACH prediction, KNN must:
  1. Compute distance to ALL n training points: O(n·d)
  2. Find K smallest distances:                 O(n log K)

  Total per prediction: O(n·d)

  ┌──────────────────────────────────────────────────────────┐
  │  Example:                                                │
  │  n = 1,000,000 training points                           │
  │  d = 50 features                                         │
  │  m = 10,000 test points                                  │
  │                                                          │
  │  Distances: 1M × 50 × 10K = 500 billion operations!     │
  │                                                          │
  │  Compare: A trained logistic regression needs only       │
  │  50 multiplications + 1 sigmoid per prediction.          │
  └──────────────────────────────────────────────────────────┘
```

### ❌ 2.2 Memory Intensive

```
  KNN stores the ENTIRE training dataset in memory:

  ┌──────────────────────────────────────────────────┐
  │  n = 1,000,000 samples                          │
  │  d = 100 features                               │
  │  8 bytes per float64                             │
  │                                                  │
  │  Memory = 1M × 100 × 8 bytes = 800 MB           │
  │                                                  │
  │  Compare: A logistic regression model stores     │
  │  only 100 weights = 800 bytes!                   │
  └──────────────────────────────────────────────────┘

  KNN memory = O(n·d)
  Most trained models = O(d) or O(d²)
```

### ❌ 2.3 Curse of Dimensionality

```
  (Covered in depth in Chapter 4)

  In high dimensions:
  - All distances converge → can't distinguish neighbors
  - Data becomes sparse → neighbors are far away
  - Need exponentially more data

  Rule of thumb: KNN struggles when d > 20-30
  (unless dimensionality reduction is applied first)
```

### ❌ 2.4 Sensitive to Feature Scale

```
  Without normalization, large-scale features dominate distances.

  Feature A: range [0, 1]         ← ignored
  Feature B: range [0, 100,000]   ← dominates all distances

  ALWAYS scale features before KNN!
  (See Chapter 1 for details)
```

### ❌ 2.5 Sensitive to Noise & Outliers

```
  Noisy labels directly affect predictions:

  ┌──────────────────────────────────────────────────────────┐
  │                                                          │
  │   True pattern:     All points in region R are Class A   │
  │                                                          │
  │   With noise:       One mislabeled point in R is Class B │
  │                                                          │
  │   K=1:  Query near the noisy point → predicts B (wrong!) │
  │   K=3:  Still 2 A's → predicts A (survives noise)        │
  │   K=1:  With outlier as nearest → wrong prediction       │
  │                                                          │
  │   KNN has no mechanism to detect/discard noisy labels.   │
  └──────────────────────────────────────────────────────────┘
```

### ❌ 2.6 Sensitive to Irrelevant Features

```
  KNN uses ALL features equally in distance computation.
  If 90 out of 100 features are irrelevant noise:
  - The noise drowns out the 10 useful features
  - Distances become meaningless
  - Performance drops dramatically

  Solution: Feature selection / dimensionality reduction
  (See Chapter 4)
```

### ❌ 2.7 Imbalanced Classes

```
  With imbalanced classes, KNN biases toward the majority:

  Dataset: 95% Class A,  5% Class B

  K = 10:  In most neighborhoods, ~9-10 neighbors are Class A
           → Almost always predicts A
           → Misses minority class B entirely

  Solutions:
  1. Use smaller K
  2. Use distance-weighted voting
  3. Oversample minority / undersample majority
  4. Adjust class-specific decision thresholds
```

---

## 3. Computational Complexity Analysis

### Detailed Breakdown

```
  ┌──────────────────┬───────────────────┬──────────────────────┐
  │  Operation       │  Brute Force      │  With Tree Structure │
  ├──────────────────┼───────────────────┼──────────────────────┤
  │  Training        │  O(1)             │  O(n·d·log n)        │
  │  (build index)   │  (just store)     │  (build tree)        │
  ├──────────────────┼───────────────────┼──────────────────────┤
  │  Single query    │  O(n·d)           │  O(d·log n) average  │
  │                  │                   │  O(n·d) worst case   │
  ├──────────────────┼───────────────────┼──────────────────────┤
  │  m queries       │  O(m·n·d)         │  O(m·d·log n) avg    │
  ├──────────────────┼───────────────────┼──────────────────────┤
  │  Memory          │  O(n·d)           │  O(n·d)              │
  └──────────────────┴───────────────────┴──────────────────────┘

  Where:
    n = number of training samples
    d = number of dimensions (features)
    m = number of query points
    K = number of neighbors (negligible vs n)
```

### When Brute Force Is Acceptable

```
  Brute force is fine when:
  - n < 10,000 (small datasets)
  - d > 20 (tree structures lose advantage in high dimensions)
  - Only a few predictions needed

  Tree structures help when:
  - n > 10,000 (many training points)
  - d < 20 (low-to-moderate dimensions)
  - Many predictions needed at inference time
```

---

## 4. KD-Tree for Speedup

### What Is a KD-Tree?

A **KD-Tree** (K-Dimensional Tree) is a binary space-partitioning data structure that recursively divides the feature space along alternating axes.

```
  Construction (2D example with 6 points):

  Points: A(2,3), B(5,4), C(9,6), D(4,7), E(8,1), F(7,2)

  Step 1: Split on x-axis (median x = 5.5)
          Left: A(2,3), B(5,4), D(4,7)
          Right: C(9,6), E(8,1), F(7,2)

  Step 2: Split each side on y-axis
          Left-Low (y<4): A(2,3)
          Left-High (y≥4): B(5,4), D(4,7)
          Right-Low (y<2): E(8,1)
          Right-High (y≥2): C(9,6), F(7,2)

  Tree structure:
                    ┌─────────────┐
                    │  x ≤ 5.5?   │
                    └──────┬──────┘
                     ╱           ╲
              ┌──────┐       ┌──────┐
              │y ≤ 4?│       │y ≤ 2?│
              └──┬───┘       └──┬───┘
               ╱    ╲         ╱    ╲
           ┌───┐ ┌────┐  ┌───┐ ┌────┐
           │ A │ │B, D│  │ E │ │C, F│
           └───┘ └────┘  └───┘ └────┘
```

### How KD-Tree Speeds Up Search

```
  Brute force: Check ALL n points.
  KD-Tree:     Prune entire branches that can't contain nearer neighbors.

  Query: Find nearest neighbor of Q(6,3)

  1. Start at root: x ≤ 5.5? → Q has x=6, go RIGHT
  2. At right node: y ≤ 2? → Q has y=3, go RIGHT (high)
  3. Reach leaf: {C(9,6), F(7,2)}
     Best so far: F(7,2), d = √(1+1) = 1.41

  4. Backtrack: Check if LEFT branch could have closer point
     Right-Low: E(8,1), d = √(4+4) = 2.83 → worse, skip

  5. Backtrack more: Check if LEFT subtree could help
     Closest possible distance to left region = |6 - 5.5| = 0.5
     0.5 < 1.41 → YES, we must check left!

  6. Check left subtree... B(5,4): d = √(1+1) = 1.41 (tie!)
     D(4,7): d = √(4+16) = 4.47 → worse
     A(2,3): d = √(16+0) = 4.0 → worse

  Result: F(7,2) or B(5,4) — found nearest without checking all!
  Checked 5 out of 6 points (not much saving here, but for n=1M it's huge)
```

### Complexity

```
  ┌────────────────────┬────────────────┐
  │  Operation         │  Complexity    │
  ├────────────────────┼────────────────┤
  │  Build tree        │  O(n·d·log n)  │
  │  Query (average)   │  O(d·log n)    │
  │  Query (worst)     │  O(n·d)        │
  │  Memory            │  O(n·d)        │
  └────────────────────┴────────────────┘

  Average speedup: from O(n) to O(log n) per query!
  For n = 1,000,000: from ~1M comparisons to ~20 comparisons.
```

### Limitations

```
  ┌──────────────────────────────────────────────────────┐
  │  KD-Trees DEGRADE in high dimensions:                │
  │                                                      │
  │  d ≤ 10-15:  Excellent speedup (O(log n) queries)    │
  │  d = 15-20:  Moderate speedup                        │
  │  d > 20:     Often SLOWER than brute force!           │
  │              (too many branches to prune)             │
  │                                                      │
  │  Rule: If d > ~20, use Ball Tree or brute force.     │
  └──────────────────────────────────────────────────────┘
```

### Using KD-Tree in scikit-learn

```python
from sklearn.neighbors import KNeighborsClassifier

# Automatically uses KD-Tree when beneficial
knn = KNeighborsClassifier(
    n_neighbors=5,
    algorithm='kd_tree',    # Force KD-Tree
    leaf_size=30            # Leaf size (trade-off: tree depth vs brute search)
)

# Or let sklearn choose automatically:
knn_auto = KNeighborsClassifier(
    n_neighbors=5,
    algorithm='auto'        # sklearn picks kd_tree, ball_tree, or brute
)
```

---

## 5. Ball Tree for Speedup

### What Is a Ball Tree?

A **Ball Tree** partitions data into nested hyperspheres (balls) instead of axis-aligned hyperplanes.

```
  KD-Tree: Splits along axes (rectangles)
  Ball Tree: Splits into balls (spheres)

  Ball Tree structure:

                   ┌───────────────┐
                   │  Root Ball    │
                   │  center=(5,4) │
                   │  radius=5.0   │
                   └──────┬────────┘
                    ╱           ╲
           ┌────────────┐  ┌────────────┐
           │ Left Ball  │  │ Right Ball │
           │ c=(2,3)    │  │ c=(8,3)   │
           │ r=2.5      │  │ r=3.0     │
           └─────┬──────┘  └─────┬─────┘
            ╱      ╲          ╱      ╲
         ┌────┐ ┌────┐   ┌────┐ ┌────┐
         │ A,B│ │ D  │   │ E,F│ │ C  │
         └────┘ └────┘   └────┘ └────┘
```

### How Ball Tree Prunes

```
  Query Q, current best distance = d_best

  For a ball with center c and radius r:

  If dist(Q, c) - r > d_best:
     → Entire ball is too far → PRUNE (skip all points in this ball)

  ┌─────────────────────────────────────────────────────┐
  │                                                     │
  │     ╭─────╮                                         │
  │    │  ·   │          Q ·                            │
  │    │  c   │    ← d(Q,c) - r > d_best ?             │
  │    │  ·   │          │                              │
  │     ╰─────╯       d_best ↕                          │
  │     r = radius       · nearest so far              │
  │                                                     │
  │  If YES → no point in this ball can be closer       │
  │  than d_best → skip entire ball!                    │
  └─────────────────────────────────────────────────────┘
```

### Ball Tree vs KD-Tree

```
  ┌────────────────────┬──────────────┬──────────────────┐
  │  Aspect            │  KD-Tree     │  Ball Tree       │
  ├────────────────────┼──────────────┼──────────────────┤
  │  Partition shape   │  Hyperplanes │  Hyperspheres    │
  │                    │  (rectangles)│  (balls)         │
  ├────────────────────┼──────────────┼──────────────────┤
  │  Low dimensions    │  Faster      │  Similar         │
  │  (d < 15)          │  (simpler)   │                  │
  ├────────────────────┼──────────────┼──────────────────┤
  │  High dimensions   │  Degrades    │  Better than     │
  │  (d > 20)          │  rapidly     │  KD-Tree         │
  ├────────────────────┼──────────────┼──────────────────┤
  │  Non-Euclidean     │  Only works  │  Works with ANY  │
  │  metrics           │  with Lp     │  metric          │
  ├────────────────────┼──────────────┼──────────────────┤
  │  Build time        │  O(n·d·log n)│  O(n·d·log n)    │
  ├────────────────────┼──────────────┼──────────────────┤
  │  Query time (avg)  │  O(d·log n)  │  O(d·log n)      │
  ├────────────────────┼──────────────┼──────────────────┤
  │  Clustered data    │  Less        │  Very effective   │
  │                    │  effective   │  (balls fit       │
  │                    │              │  clusters well)   │
  └────────────────────┴──────────────┴──────────────────┘
```

### Using Ball Tree in scikit-learn

```python
from sklearn.neighbors import KNeighborsClassifier

knn = KNeighborsClassifier(
    n_neighbors=5,
    algorithm='ball_tree',     # Use Ball Tree
    leaf_size=30,
    metric='euclidean'         # Ball Tree works with any metric!
)

# Ball Tree with non-standard metric
knn_haversine = KNeighborsClassifier(
    n_neighbors=5,
    algorithm='ball_tree',
    metric='haversine'         # Great for geographic distance!
)
```

---

## 6. Approximate Nearest Neighbors

### Why Approximate?

```
  For very large datasets (millions/billions of points), even
  KD-Trees and Ball Trees become slow. Approximate methods trade
  a small amount of accuracy for massive speedup.

  ┌────────────────────────────────────────────────────────────┐
  │  Exact KNN:        Find the TRUE K nearest neighbors       │
  │                    Time: O(n) worst case per query         │
  │                                                            │
  │  Approximate KNN:  Find K neighbors that are ALMOST the   │
  │                    nearest (with high probability)         │
  │                    Time: O(1) to O(log n) per query!       │
  │                                                            │
  │  Accuracy drop:    Usually < 1-3%                          │
  │  Speedup:          10× to 1000× faster                    │
  └────────────────────────────────────────────────────────────┘
```

### Popular Libraries

```
  ┌──────────────────┬────────────────────────────────────────────┐
  │  Library         │  Description                               │
  ├──────────────────┼────────────────────────────────────────────┤
  │  FAISS           │  Facebook AI. GPU-accelerated. Best for    │
  │  (Facebook)      │  massive-scale (billions of vectors).      │
  │                  │  Used in recommendation, search.           │
  ├──────────────────┼────────────────────────────────────────────┤
  │  Annoy           │  Spotify. Random projection trees.         │
  │  (Spotify)       │  Memory-mapped. Good for serving.          │
  ├──────────────────┼────────────────────────────────────────────┤
  │  HNSW            │  Hierarchical Navigable Small World        │
  │  (hnswlib)       │  graphs. State-of-the-art recall/speed.    │
  ├──────────────────┼────────────────────────────────────────────┤
  │  ScaNN           │  Google. Learned quantization.             │
  │  (Google)        │  Optimized for inner product search.       │
  ├──────────────────┼────────────────────────────────────────────┤
  │  sklearn         │  NearestNeighbors with LSH or trees.       │
  │                  │  Good for moderate scale.                  │
  └──────────────────┴────────────────────────────────────────────┘
```

### Techniques Overview

```
  1. Locality-Sensitive Hashing (LSH):
     Hash similar points to the same bucket.
     Query: hash the query → check only points in same bucket.

     ┌─Bucket A──┐  ┌─Bucket B──┐  ┌─Bucket C──┐
     │ ● ●  ●    │  │ ○ ○  ○    │  │ ▲ ▲  ▲    │
     │   ●  ●    │  │  ○   ○    │  │  ▲   ▲    │
     └───────────┘  └───────────┘  └───────────┘

     Query Q hashes to Bucket B → only check ○ points.

  2. Random Projection Trees:
     Like KD-Trees but split along RANDOM directions
     (not just axis-aligned). Better in high dimensions.

  3. Product Quantization:
     Compress vectors into short codes (e.g., 128D → 8 bytes).
     Compute approximate distances on compressed representations.

  4. Graph-Based (HNSW):
     Build a navigable graph where each point connects to nearby points.
     Navigate the graph from a random start to find nearest neighbors.
```

### Speed Comparison (Typical)

```
  Dataset: 1 million points, 128 dimensions
  Task: Find 10 nearest neighbors

  ┌──────────────────┬─────────────┬──────────────┬──────────┐
  │  Method          │  Time/query │  Recall@10   │  Memory  │
  ├──────────────────┼─────────────┼──────────────┼──────────┤
  │  Brute Force     │  50 ms      │  100%        │  512 MB  │
  │  KD-Tree         │  25 ms      │  100%        │  600 MB  │
  │  Ball Tree       │  20 ms      │  100%        │  600 MB  │
  │  FAISS (IVF)     │  0.5 ms     │  98%         │  200 MB  │
  │  HNSW            │  0.1 ms     │  99%         │  700 MB  │
  │  Annoy           │  0.3 ms     │  97%         │  400 MB  │
  │  FAISS (GPU)     │  0.01 ms    │  99%         │  GPU mem │
  └──────────────────┴─────────────┴──────────────┴──────────┘

  HNSW: 500× faster than brute force with 99% recall!
```

---

## 7. When to Use KNN vs Other Algorithms

### KNN Is a Good Choice When:

```
  ✅ Dataset is small to medium (n < 50,000)
  ✅ Low to moderate dimensions (d < 20, or after reduction)
  ✅ Decision boundary is non-linear
  ✅ No assumptions about data distribution
  ✅ Quick prototyping / baseline model
  ✅ Data changes frequently (add/remove points)
  ✅ Instance-based reasoning needed (explainability)
  ✅ Multi-class classification (no modification needed)
```

### KNN Is a Poor Choice When:

```
  ❌ Dataset is very large (n > 100,000) — slow predictions
  ❌ High-dimensional data (d > 30) — curse of dimensionality
  ❌ Real-time / low-latency predictions needed
  ❌ Limited memory available
  ❌ Many irrelevant features
  ❌ Heavily imbalanced classes
  ❌ Noisy data with many mislabeled points
```

### Algorithm Comparison Matrix

```
  ┌────────────────────┬──────┬──────┬───────┬──────┬─────────┬───────────┐
  │  Criterion         │ KNN  │  LR  │  SVM  │  DT  │   RF    │    NN     │
  ├────────────────────┼──────┼──────┼───────┼──────┼─────────┼───────────┤
  │  Training speed    │ ★★★★★│ ★★★★ │ ★★    │ ★★★★ │ ★★★     │ ★         │
  │  Prediction speed  │ ★    │ ★★★★★│ ★★★★★ │ ★★★★★│ ★★★★    │ ★★★★      │
  │  Memory            │ ★    │ ★★★★★│ ★★★   │ ★★★★ │ ★★★     │ ★★★       │
  │  Non-linear        │ ★★★★★│ ★    │ ★★★★★ │ ★★★★ │ ★★★★★   │ ★★★★★     │
  │  Handles d > 50    │ ★    │ ★★★★ │ ★★★★  │ ★★★  │ ★★★★    │ ★★★★★     │
  │  Handles n > 100K  │ ★    │ ★★★★★│ ★★    │ ★★★★ │ ★★★★    │ ★★★★★     │
  │  Interpretability  │ ★★★★ │ ★★★★★│ ★★    │ ★★★★★│ ★★      │ ★         │
  │  Handles noise     │ ★★   │ ★★★  │ ★★★★  │ ★★   │ ★★★★    │ ★★★       │
  │  No hyperparams    │ ★★★  │ ★★★★ │ ★★    │ ★★★  │ ★★      │ ★         │
  │  Online learning   │ ★★★★★│ ★★★  │ ★     │ ★    │ ★       │ ★★★       │
  └────────────────────┴──────┴──────┴───────┴──────┴─────────┴───────────┘

  LR = Logistic Regression    DT = Decision Tree
  SVM = Support Vector Machine RF = Random Forest
  NN = Neural Network          ★ = worst, ★★★★★ = best
```

### Decision Flowchart

```
  START
    │
    ├── Is n > 100,000?
    │     YES → Consider Random Forest, SVM, or Neural Network
    │     NO  ↓
    │
    ├── Is d > 30?
    │     YES → Can you reduce dimensions?
    │     │       YES → Apply PCA/LDA → Use KNN
    │     │       NO  → Use Random Forest or SVM
    │     NO  ↓
    │
    ├── Is low-latency prediction needed?
    │     YES → Use Logistic Regression, Decision Tree, or SVM
    │     NO  ↓
    │
    ├── Is the boundary likely non-linear?
    │     YES → KNN is a great choice! ✓
    │     NO  → Logistic Regression may be simpler and better
    │
    └── START WITH KNN AS BASELINE, then try other algorithms.
```

---

## 8. Comprehensive Comparison Table

| Feature | KNN | Logistic Reg. | SVM | Decision Tree | Random Forest | Neural Net |
|---------|:---:|:---:|:---:|:---:|:---:|:---:|
| Training time | O(1) | O(n·d²) | O(n²-n³) | O(n·d·log n) | O(T·n·d·log n) | O(epochs·n·W) |
| Prediction time | O(n·d) | O(d) | O(sv·d) | O(depth) | O(T·depth) | O(W) |
| Memory | O(n·d) | O(d) | O(sv·d) | O(nodes) | O(T·nodes) | O(W) |
| Non-linear | ✅ | ❌ | ✅ (kernel) | ✅ | ✅ | ✅ |
| Feature scaling needed | ✅ Must | Helpful | ✅ Must | ❌ | ❌ | ✅ Must |
| Handles missing values | ❌ | ❌ | ❌ | ✅ | ✅ | ❌ |
| Probabilistic output | Approx. | ✅ Native | Via Platt | ❌ | ✅ | ✅ |
| Interpretable | ✅ | ✅ | ❌ | ✅ | ❌ | ❌ |
| Handles imbalance | Poor | Good | Good | Fair | Good | Good |

---

## 9. Best Practices Checklist

```
  ┌────────────────────────────────────────────────────────────┐
  │  KNN IMPLEMENTATION CHECKLIST                              │
  ├────────────────────────────────────────────────────────────┤
  │                                                            │
  │  □ 1. PREPROCESS DATA                                     │
  │     □ Handle missing values (KNN can't handle them!)       │
  │     □ Encode categorical variables                         │
  │     □ Scale features (StandardScaler or MinMaxScaler)      │
  │                                                            │
  │  □ 2. REDUCE DIMENSIONS (if d > 20)                        │
  │     □ Feature selection (SelectKBest, mutual info)         │
  │     □ PCA / LDA                                            │
  │                                                            │
  │  □ 3. CHOOSE K                                             │
  │     □ Start with K = √n (rounded to odd)                   │
  │     □ Use cross-validation to find optimal K               │
  │     □ Use odd K for binary classification                  │
  │                                                            │
  │  □ 4. CHOOSE WEIGHTS                                       │
  │     □ Try weights='distance' (usually ≥ uniform)           │
  │     □ Compare with cross-validation                        │
  │                                                            │
  │  □ 5. CHOOSE METRIC                                        │
  │     □ Default: Euclidean                                   │
  │     □ High-d or sparse: Manhattan                          │
  │     □ Text/NLP: Cosine                                     │
  │     □ Correlated features: Mahalanobis                     │
  │                                                            │
  │  □ 6. CHOOSE ALGORITHM                                     │
  │     □ algorithm='auto' (let sklearn decide)                │
  │     □ d ≤ 15: 'kd_tree' is typically fastest               │
  │     □ d > 15: 'ball_tree' or 'brute'                       │
  │     □ n > 1M: Consider approximate NN (FAISS, HNSW)       │
  │                                                            │
  │  □ 7. EVALUATE                                             │
  │     □ Use cross-validation (not just train/test split)     │
  │     □ Check for overfitting (compare train vs test error)  │
  │     □ Compare with baseline models                         │
  │                                                            │
  └────────────────────────────────────────────────────────────┘
```

### Complete Production-Ready Pipeline

```python
import numpy as np
from sklearn.datasets import load_iris
from sklearn.neighbors import KNeighborsClassifier
from sklearn.model_selection import GridSearchCV, cross_val_score
from sklearn.preprocessing import StandardScaler
from sklearn.decomposition import PCA
from sklearn.pipeline import Pipeline
from sklearn.metrics import classification_report

# ───────────── Build Pipeline ─────────────
pipe = Pipeline([
    ('scaler', StandardScaler()),
    ('pca', PCA()),                     # Optional reduction
    ('knn', KNeighborsClassifier())
])

# ───────────── Hyperparameter Grid ─────────────
param_grid = {
    'pca__n_components': [2, 3, 4],     # Try different dims
    'knn__n_neighbors': [3, 5, 7, 9, 11, 13, 15],
    'knn__weights': ['uniform', 'distance'],
    'knn__metric': ['euclidean', 'manhattan'],
}

# ───────────── Grid Search with CV ─────────────
X, y = load_iris(return_X_y=True)

grid = GridSearchCV(
    pipe, param_grid, cv=10,
    scoring='accuracy', n_jobs=-1,
    return_train_score=True
)
grid.fit(X, y)

# ───────────── Results ─────────────
print(f"Best Parameters:")
for k, v in grid.best_params_.items():
    print(f"  {k}: {v}")
print(f"\nBest CV Accuracy: {grid.best_score_:.4f}")

# Top 5 configurations
results = grid.cv_results_
sorted_idx = np.argsort(results['mean_test_score'])[::-1][:5]
print(f"\nTop 5 Configurations:")
print(f"  {'Rank':>4}  {'CV Accuracy':>12}  {'Params'}")
for rank, idx in enumerate(sorted_idx, 1):
    score = results['mean_test_score'][idx]
    params = results['params'][idx]
    param_str = ', '.join(f"{k.split('__')[1]}={v}" for k, v in params.items())
    print(f"  {rank:4d}  {score:12.4f}  {param_str}")
```

---

## 10. Summary Table

| Category | Detail |
|----------|--------|
| **Pros** | Simple, no training, non-linear boundaries, multi-class, online learning |
| **Cons** | Slow prediction O(n·d), high memory O(n·d), curse of dimensionality, scale-sensitive, noise-sensitive |
| **KD-Tree** | Binary space partition; O(d·log n) queries; best for d < 15 |
| **Ball Tree** | Hypersphere partition; works for any metric; better than KD-Tree for d > 15 |
| **Approximate NN** | FAISS, HNSW, Annoy; 10-1000× speedup; >97% recall; for n > 1M |
| **Best for** | Small-medium datasets, low dimensions, non-linear problems, baselines |
| **Avoid for** | Large n, high d, real-time predictions, noisy/imbalanced data |
| **Always do** | Scale features, choose K via CV, consider dimensionality reduction |

---

## 11. Revision Questions

**Q1.** List three advantages and three disadvantages of KNN. For each disadvantage, suggest a mitigation strategy.

**Q2.** You have a dataset with 500,000 training samples and 10 features. Each KNN prediction currently takes 200ms (brute force). You need predictions under 5ms. Which acceleration technique(s) would you use, and what speedup can you expect?

**Q3.** Explain how a KD-Tree speeds up nearest neighbor search. Draw a simple 2D example with 5 points showing how the tree is constructed and how a query avoids checking all points.

**Q4.** Compare KD-Trees and Ball Trees. In what scenarios would you prefer a Ball Tree over a KD-Tree? Why does KD-Tree performance degrade in high dimensions?

**Q5.** You need to build a recommendation system that finds the 10 most similar users among 100 million user profiles (each with 128 features). Exact KNN is infeasible. Describe the approximate nearest neighbor approach you'd use and the expected trade-offs.

**Q6.** A colleague argues: "KNN is too simple to be useful in production." Counter this argument with three real-world applications where KNN (or its approximate variants) excels, and explain why more complex models aren't always better.

---

<div align="center">

| ← Previous | Current | Next → |
|:-----------:|:-------:|:------:|
| [6.5 KNN Classification & Regression](./05-knn-classification-regression.md) | **6.6 Advantages & Limitations** | [Unit 7: Support Vector Machines](../../07-Support-Vector-Machines/) |

</div>

---

*Unit 6: K-Nearest Neighbors — Chapter 6 of 6*
