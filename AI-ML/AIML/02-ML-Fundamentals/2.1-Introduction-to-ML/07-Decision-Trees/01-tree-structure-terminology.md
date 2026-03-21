# 📘 7.1 — Decision Tree Structure & Terminology

> **Unit 7 · Decision Trees** — Study Notes  
> *Part 1 of 7 — The anatomy of a decision tree, how trees represent decisions, and the language we use to describe them.*

---

## 📑 Table of Contents

1. [What Is a Decision Tree?](#1-what-is-a-decision-tree)
2. [Core Terminology](#2-core-terminology)
3. [ASCII Tree Diagram — Anatomy](#3-ascii-tree-diagram--anatomy)
4. [Binary vs Multi-Way Trees](#4-binary-vs-multi-way-trees)
5. [Tree Depth, Size & Complexity](#5-tree-depth-size--complexity)
6. [How Trees Make Predictions — Traversal](#6-how-trees-make-predictions--traversal)
7. [The "20 Questions" Analogy](#7-the-20-questions-analogy)
8. [Worked Example — Full Traversal](#8-worked-example--full-traversal)
9. [Python — Visualising a Decision Tree](#9-python--visualising-a-decision-tree)
10. [Applications](#10-applications)
11. [Summary Table](#11-summary-table)
12. [Revision Questions](#12-revision-questions)
13. [Navigation](#13-navigation)

---

## 1. What Is a Decision Tree?

A **decision tree** is a supervised learning algorithm that models decisions as a tree-shaped graph of **if-then-else** rules. Each internal node tests a feature, each branch represents an outcome of that test, and each leaf node holds a prediction (class label or numeric value).

Think of it as a **flowchart**:

```
Start
  │
  ▼
[Question 1?]──Yes──▶ [Question 2?]──Yes──▶ Prediction A
  │                        │
  No                       No
  │                        │
  ▼                        ▼
Prediction B           Prediction C
```

Decision trees are used for **classification** (predict a category) and **regression** (predict a number). Their greatest strength is **interpretability** — a non-technical stakeholder can follow the logic from root to leaf and understand *why* a prediction was made.

---

## 2. Core Terminology

### 2.1 Root Node

The **topmost** node of the tree. It contains the very first splitting rule applied to the entire dataset.

- There is exactly **one** root node per tree.
- It has **no parent**.
- It splits the data into two or more subsets.

### 2.2 Internal Nodes (Decision Nodes)

Any node that is **neither the root nor a leaf**. Internal nodes contain a **test condition** on a feature (e.g., `Age ≤ 30`). Each internal node has:

| Property | Description |
|----------|-------------|
| Parent | The node directly above it |
| Children | Two or more nodes directly below it |
| Splitting rule | A condition on one feature |

> **Note:** The root node is technically also a decision node, but we distinguish it because of its unique position.

### 2.3 Leaf Nodes (Terminal Nodes)

Nodes with **no children**. A leaf node contains the **prediction**:

- **Classification tree:** the majority class among the training samples that reached this leaf.
- **Regression tree:** the mean (or median) of the target values of training samples that reached this leaf.

### 2.4 Branches (Edges)

Directed connections from a parent node to a child node. Each branch corresponds to one **outcome** of the parent's test.

```
[Humidity ≤ 70%?]
     /        \
   Yes         No          ← these are the two branches
   /             \
 Leaf A        Leaf B
```

### 2.5 Parent and Child Nodes

| Term | Definition |
|------|-----------|
| **Parent** | A node that has one or more children (it splits further). |
| **Child** | A node directly connected below a parent via a branch. |

Every node except the root has exactly one parent. Every non-leaf node has at least two children.

### 2.6 Subtree

A **subtree** is any node together with all of its descendants (children, grandchildren, …). You can "cut" a subtree and treat it as a standalone tree.

```
         [A]                  Subtree rooted at B:
        /   \
      [B]   [C]                    [B]
     /   \                        /   \
   [D]   [E]                   [D]   [E]
```

### 2.7 Depth (Level)

The **depth** of a node is the number of edges from the root to that node.

| Node | Depth |
|------|-------|
| Root | 0 |
| Root's children | 1 |
| Root's grandchildren | 2 |
| … | … |

The **depth of a tree** (also called **height**) is the maximum depth among all leaf nodes.

### 2.8 Siblings

Nodes that share the **same parent**.

```
     [Parent]
     /      \
  [Sib1]  [Sib2]    ← siblings
```

### 2.9 Degree of a Node

The **number of children** a node has.

- Leaf nodes have degree 0.
- In a binary tree every internal node has degree 2.
- In a multi-way tree internal nodes can have degree > 2.

---

## 3. ASCII Tree Diagram — Anatomy

Below is a fully labelled ASCII diagram of a classification tree that predicts whether someone will play tennis:

```
                        ┌─────────────────────┐
          Depth 0       │   Outlook            │  ◄── ROOT NODE
          (Root)        │   {14 samples}       │
                        └──────┬───────┬───────┘
                               │       │       │
                     Sunny     │ Overcast│  Rainy       ◄── BRANCHES
                               │       │       │
                 ┌─────────┐   │   ┌───┴───┐   │   ┌─────────┐
   Depth 1       │Humidity │   │   │ Play=  │   │   │  Wind   │  ◄── INTERNAL
   (Internal)    │{5 samp.}│   │   │  Yes   │   │   │{5 samp.}│      NODES
                 └──┬───┬──┘   │   │{4/4}   │   │   └──┬───┬──┘
                    │   │      │   └────────┘   │      │   │
              High  │   │Normal│   ◄── LEAF     │Strong│   │Weak
                    │   │      │       NODE      │      │   │
              ┌─────┴┐ ┌┴─────┐                ┌┴─────┐ ┌─┴────┐
  Depth 2     │ No   │ │ Yes  │                │ No   │ │ Yes  │  ◄── LEAF
  (Leaves)    │{2/0} │ │{0/3} │                │{2/0} │ │{0/3} │      NODES
              └──────┘ └──────┘                └──────┘ └──────┘

  Legend:
    {a/b} = a "No" samples / b "Yes" samples reaching this leaf
    {n samples} = total samples at that node
```

### Reading the diagram

| Label | Meaning |
|-------|---------|
| **Root** (`Outlook`) | First question — split the entire dataset by the "Outlook" feature |
| **Internal nodes** (`Humidity`, `Wind`) | Further split a subset |
| **Leaf nodes** (`Yes`, `No`) | Final prediction — no more splitting |
| **Branches** (`Sunny`, `Overcast`, `Rainy`, etc.) | Outcome of the test at the parent |
| **Depth** | Root = 0, next level = 1, leaves here = 2 |

---

## 4. Binary vs Multi-Way Trees

### 4.1 Binary Trees

Every internal node has **exactly 2 children**.

- Used by **CART** (Classification And Regression Trees) — the algorithm behind `sklearn`.
- For a continuous feature: `feature ≤ threshold` → left child, else → right child.
- For a categorical feature with k categories: CART partitions categories into two subsets.

```
     [Age ≤ 30?]
      /       \
    Yes        No
    /            \
 [Leaf]       [Leaf]
```

### 4.2 Multi-Way Trees

Internal nodes can have **more than 2 children** — one branch per category.

- Used by **ID3** and **C4.5** algorithms.
- Natural for nominal features (e.g., `Color ∈ {Red, Green, Blue}` → 3 branches).

```
       [Color?]
      /   |    \
   Red  Green  Blue
    /     |      \
 [L1]   [L2]   [L3]
```

### 4.3 Comparison

| Aspect | Binary Tree | Multi-Way Tree |
|--------|------------|----------------|
| Children per node | Exactly 2 | 2 or more |
| Algorithm examples | CART | ID3, C4.5 |
| Continuous features | `≤ threshold` split | Requires discretisation |
| Categorical features | Subset split | One branch per category |
| Tree depth | Tends to be deeper | Tends to be shallower |
| Implementation | `sklearn` default | Weka, custom |

---

## 5. Tree Depth, Size & Complexity

### 5.1 Key Metrics

| Metric | Definition |
|--------|-----------|
| **Depth (height)** | Longest path from root to any leaf |
| **Number of leaves** | Total leaf nodes (= number of distinct predictions) |
| **Number of nodes** | Internal nodes + leaf nodes |
| **Number of splits** | Internal nodes (each makes one split) |

### 5.2 Relationship Between Depth & Capacity

For a **binary tree** of depth *d*:

```
Maximum number of leaves  = 2^d
Maximum number of nodes   = 2^(d+1) − 1
```

Example:

| Depth | Max Leaves | Max Nodes |
|-------|-----------|-----------|
| 1 | 2 | 3 |
| 2 | 4 | 7 |
| 3 | 8 | 15 |
| 5 | 32 | 63 |
| 10 | 1 024 | 2 047 |
| 20 | 1 048 576 | 2 097 151 |

> **Key Insight:** A tree of depth 20 can partition the feature space into over a million regions. Deep trees are **very expressive** but prone to **overfitting**.

### 5.3 Tree Complexity & Overfitting

```
Complexity ↑   →   Training error ↓   but   Test error ↑  (overfitting)
                     ┌─────────────────────────────────┐
  Error              │  ·  ·                           │
    ↑                │   ·    ·  Test error            │
    │                │    ·      · · · · · · · · · ·   │
    │                │     ·                           │
    │                │  · · · · · · · · · · · · · · ·  │
    │                │         Training error           │
    │                └──────────────────────────────────┘
                           Tree Depth / # Leaves  →
```

Controlling complexity (via pruning or depth limits) is essential — covered in detail in **Part 4**.

---

## 6. How Trees Make Predictions — Traversal

### 6.1 The Traversal Algorithm

To predict for a new sample **x**:

```
function PREDICT(node, x):
    if node is a LEAF:
        return node.prediction
    else:
        test = node.splitting_rule
        if test(x) is TRUE:
            return PREDICT(node.left_child, x)
        else:
            return PREDICT(node.right_child, x)
```

### 6.2 Step-by-Step Traversal Example

Given this tree:

```
          [Income ≤ 50K?]
           /            \
         Yes              No
         /                  \
   [Age ≤ 25?]          [Credit = Good?]
    /       \              /          \
  Yes       No           Yes          No
  /          \            /             \
DENY      REVIEW       APPROVE        REVIEW
```

**Predict for:** `Income = 70K, Age = 40, Credit = Good`

| Step | Node | Test | Result | Go to |
|------|------|------|--------|-------|
| 1 | Root | Income ≤ 50K? | 70K > 50K → **No** | Right child |
| 2 | Internal | Credit = Good? | Good → **Yes** | Left child |
| 3 | Leaf | — | — | **APPROVE** |

**Prediction: APPROVE** ✅

### 6.3 Time Complexity of Prediction

- Each prediction traverses at most *d* nodes (where *d* = tree depth).
- **Time complexity:** O(d) per sample — extremely fast.
- Compared to k-NN which is O(n) per prediction (naively), trees are much faster at inference time.

---

## 7. The "20 Questions" Analogy

Decision trees work exactly like the classic **"20 Questions"** game:

```
Game: "I'm thinking of an animal."

Q1: Is it a mammal?           → Tree: [Feature: Class = Mammal?]
    Yes ──▶
Q2: Does it live in water?    → Tree: [Feature: Habitat = Water?]
    No ──▶
Q3: Is it bigger than a dog?  → Tree: [Feature: Size > Dog?]
    Yes ──▶
Q4: Does it have stripes?     → Tree: [Feature: Pattern = Striped?]
    Yes ──▶
Answer: Tiger!                → Tree: Leaf = Tiger 🐅
```

### Why This Analogy Is Powerful

| 20 Questions Game | Decision Tree |
|-------------------|---------------|
| Each question narrows down possibilities | Each split reduces impurity |
| Good questions eliminate many options | Good splits maximise information gain |
| You try to ask the most informative question first | The algorithm selects the feature with the best split |
| You reach an answer when no more questions needed | You reach a leaf node |
| Asking too many questions is wasteful | A too-deep tree overfits |

### Optimal Strategy

In both the game and the algorithm, the **optimal strategy** is to ask the question that **most evenly splits** the remaining possibilities. This is formalised mathematically through **Information Gain** and **Gini Impurity** (covered in Part 2).

---

## 8. Worked Example — Full Traversal

### Dataset: Should We Play Tennis?

| Day | Outlook  | Temp | Humidity | Wind   | Play? |
|-----|----------|------|----------|--------|-------|
| 1   | Sunny    | Hot  | High     | Weak   | No    |
| 2   | Sunny    | Hot  | High     | Strong | No    |
| 3   | Overcast | Hot  | High     | Weak   | Yes   |
| 4   | Rainy    | Mild | High     | Weak   | Yes   |
| 5   | Rainy    | Cool | Normal   | Weak   | Yes   |
| 6   | Rainy    | Cool | Normal   | Strong | No    |
| 7   | Overcast | Cool | Normal   | Strong | Yes   |
| 8   | Sunny    | Mild | High     | Weak   | No    |
| 9   | Sunny    | Cool | Normal   | Weak   | Yes   |
| 10  | Rainy    | Mild | Normal   | Weak   | Yes   |
| 11  | Sunny    | Mild | Normal   | Strong | Yes   |
| 12  | Overcast | Mild | High     | Strong | Yes   |
| 13  | Overcast | Hot  | Normal   | Weak   | Yes   |
| 14  | Rainy    | Mild | High     | Strong | No    |

### Built Tree (this tree is derived in Part 3)

```
                    [Outlook?]
                   /    |     \
             Sunny  Overcast   Rainy
               /       |         \
        [Humidity?]   Yes       [Wind?]
         /      \               /     \
       High    Normal      Strong    Weak
        /         \          /         \
       No         Yes       No        Yes
```

### Traversal for Day 8: Sunny, Mild, High, Weak

```
Step 1:  Outlook = Sunny   → go left
Step 2:  Humidity = High   → go left
Step 3:  Leaf reached      → Prediction = No  ✅ (matches actual)
```

### Traversal for Day 7: Overcast, Cool, Normal, Strong

```
Step 1:  Outlook = Overcast → go middle
Step 2:  Leaf reached       → Prediction = Yes ✅ (matches actual)
```

---

## 9. Python — Visualising a Decision Tree

```python
import numpy as np
from sklearn.datasets import load_iris
from sklearn.tree import DecisionTreeClassifier, export_text, plot_tree
import matplotlib.pyplot as plt

# --- Load dataset ---
iris = load_iris()
X, y = iris.data, iris.target

# --- Train a shallow tree ---
clf = DecisionTreeClassifier(max_depth=3, random_state=42)
clf.fit(X, y)

# --- Text representation (great for terminals) ---
tree_rules = export_text(clf, feature_names=iris.feature_names)
print("Decision Tree Rules:")
print(tree_rules)

# --- Key structural properties ---
print(f"\nTree depth:       {clf.get_depth()}")
print(f"Number of leaves: {clf.get_n_leaves()}")
print(f"Total nodes:      {clf.tree_.node_count}")

# --- Graphical plot ---
fig, ax = plt.subplots(figsize=(16, 8))
plot_tree(
    clf,
    feature_names=iris.feature_names,
    class_names=iris.target_names,
    filled=True,          # colour by majority class
    rounded=True,
    fontsize=10,
    ax=ax,
)
ax.set_title("Iris Decision Tree (max_depth=3)", fontsize=14)
plt.tight_layout()
plt.savefig("decision_tree_iris.png", dpi=150)
plt.show()

# --- Manual traversal example ---
sample = X[0:1]  # first sample
print(f"\nSample features: {sample}")
print(f"Predicted class:  {iris.target_names[clf.predict(sample)[0]]}")

# Show the path taken through the tree
node_indicator = clf.decision_path(sample)
node_ids = node_indicator.indices
print(f"Nodes visited:    {node_ids}")
```

**Expected output (text tree):**

```
Decision Tree Rules:
|--- petal width (cm) <= 0.80
|   |--- class: setosa
|--- petal width (cm) >  0.80
|   |--- petal width (cm) <= 1.75
|   |   |--- petal length (cm) <= 4.95
|   |   |   |--- class: versicolor
|   |   |--- petal length (cm) >  4.95
|   |   |   |--- class: virginica
|   |--- petal width (cm) >  1.75
|   |   |--- petal length (cm) <= 4.85
|   |   |   |--- class: virginica
|   |   |--- petal length (cm) >  4.85
|   |   |   |--- class: virginica

Tree depth:       3
Number of leaves: 5
Total nodes:      9
```

---

## 10. Applications

| Domain | How Decision Trees Are Used |
|--------|-----------------------------|
| **Medicine** | Diagnosis flowcharts — symptom-based decision paths |
| **Finance** | Credit scoring — approve / deny based on income, history |
| **Marketing** | Customer segmentation and churn prediction |
| **Manufacturing** | Quality control — classify defective vs non-defective parts |
| **Ecology** | Species classification based on habitat features |
| **Law / Compliance** | Rule-based eligibility checks (the tree *is* the rule set) |

---

## 11. Summary Table

| Term | Definition | Example |
|------|-----------|---------|
| **Root node** | Top node; first split | `Outlook?` |
| **Internal node** | Non-leaf, non-root decision node | `Humidity?` |
| **Leaf node** | Terminal node; holds prediction | `Play = Yes` |
| **Branch** | Edge from parent to child | `Sunny`, `Rainy` |
| **Depth** | # edges from root to a node | Root = 0, child = 1 |
| **Parent** | Node directly above | `Outlook` is parent of `Humidity` |
| **Child** | Node directly below | `Humidity` is child of `Outlook` |
| **Subtree** | Node + all its descendants | Subtree rooted at `Wind` |
| **Siblings** | Nodes sharing a parent | `Humidity` and `Wind` |
| **Binary tree** | Max 2 children per node | CART algorithm |
| **Multi-way tree** | >2 children allowed | ID3, C4.5 |
| **Traversal** | Path from root to leaf for prediction | O(d) time |
| **Tree complexity** | Depth / # leaves / # nodes | Deeper → more complex |

---

## 12. Revision Questions

1. **Define** the terms *root node*, *internal node*, *leaf node*, and *branch* in the context of a decision tree. Draw a small ASCII tree labelling each.

2. A binary decision tree has depth 6. What is the **maximum number of leaf nodes** it can have? What is the maximum total number of nodes?

3. Explain the **difference between a binary tree and a multi-way tree**. Which algorithm uses which type?

4. Describe the **traversal algorithm** used to make a prediction with a decision tree. What is its time complexity?

5. How is a decision tree similar to the **"20 Questions" game**? Why does the analogy break down for very deep trees?

6. Given the tree below, predict the output for a sample with `A = 3, B = High`:

```
        [A ≤ 5?]
        /      \
      Yes       No
      /           \
   [B = High?]   Class X
    /       \
  Yes       No
  /           \
Class Y    Class Z
```

---

## 13. Navigation

| Previous | Current | Next |
|----------|---------|------|
| — | **7.1 Tree Structure & Terminology** | [7.2 Splitting Criteria →](02-splitting-criteria.md) |

| Module | Link |
|--------|------|
| 7.1 | **Tree Structure & Terminology** (you are here) |
| 7.2 | [Splitting Criteria](02-splitting-criteria.md) |
| 7.3 | [Building a Decision Tree](03-building-decision-tree.md) |
| 7.4 | [Pruning Techniques](04-pruning-techniques.md) |
| 7.5 | [Advantages & Limitations](05-advantages-and-limitations.md) |
| 7.6 | [Feature Importance](06-feature-importance.md) |
| 7.7 | [Decision Trees for Regression](07-decision-trees-regression.md) |

---

> *"A decision tree is a map of possible outcomes of a series of related choices."* — every ML textbook, paraphrased.
