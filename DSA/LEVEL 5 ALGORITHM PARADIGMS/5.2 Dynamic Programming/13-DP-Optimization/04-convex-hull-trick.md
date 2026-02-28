# Chapter 4: Convex Hull Trick (CHT)

## 📋 Overview
The **Convex Hull Trick** optimizes DP recurrences of the form $dp[i] = \min_j(a[j] \cdot x[i] + b[j])$ — evaluating the minimum of a set of linear functions. It reduces an $O(n^2)$ DP to $O(n \log n)$ or even $O(n)$.

---

## 🧠 Core Problem

### DP Form
```
dp[i] = min over j < i of { dp[j] + cost(j, i) }

When cost(j, i) can be decomposed as:
  cost(j, i) = f(j) · g(i) + h(j) + k(i)

Then:
  dp[i] = min_j { f(j)·g(i) + h(j) } + k(i)
        = min_j { m_j · x + b_j }     where x = g(i)

This is: evaluate minimum of a SET of LINES at point x.
```

### Geometric Interpretation
```
Each j defines a line: y = m_j · x + b_j

We want: for each query x_i, find the line giving minimum y.

The answer always lies on the LOWER CONVEX HULL of lines:

    y ↑
      │     /
      │    /  line 3
      │   / ╱─── line 1
      │  / ╱
      │ /╱───── line 2
      │╱
      └──────────→ x
      
    Lower envelope (minimum):
      ─────╲     ╱─────
             ╲  ╱
              ╲╱  ← intersection points
    
    Only the lines on this envelope matter.
```

---

## 🔍 Step-by-Step Example

### USACO Fencing the Cows (simplified)
```
Minimize dp[i] = min over j<i { dp[j] + a[j] · b[i] }

Where:
  slope m_j = a[j]
  intercept b_j = dp[j]
  query point x = b[i]

Lines added so far:
  j=0: y = a[0]·x + dp[0]
  j=1: y = a[1]·x + dp[1]
  ...

For each i: query the minimum at x = b[i]
Then:       add new line with slope a[i], intercept dp[i]
```

---

## 💻 Pseudocode: Monotone CHT (Li Chao Tree Alternative)

### When slopes are monotone (sorted)
```
// Lines sorted by decreasing slope for minimum queries
// Queries come in increasing order of x

function ConvexHullTrick:
    lines = deque of (slope, intercept) pairs
    
    function addLine(m, b):
        while |lines| >= 2:
            // Check if second-to-last line is now useless
            l1 = lines[|lines| - 2]
            l2 = lines[|lines| - 1]
            l3 = (m, b)
            if intersection(l1, l3).x <= intersection(l1, l2).x:
                lines.removeLast()  // l2 is dominated
            else:
                break
        lines.append((m, b))
    
    function query(x):
        // Remove lines from front that are no longer optimal
        while |lines| >= 2:
            if eval(lines[0], x) >= eval(lines[1], x):
                lines.removeFirst()
            else:
                break
        return eval(lines[0], x)
    
    function eval(line, x):
        return line.slope * x + line.intercept

    function intersection(l1, l2):
        // x where l1 and l2 intersect
        return (l2.b - l1.b) / (l1.m - l2.m)
```

### Condition for removing a line
```
Three lines l1, l2, l3 (in slope order):

l2 is useless if the intersection of l1 and l3
is to the LEFT of the intersection of l1 and l2.

       l1╲   ╱l3
          ╲ ╱
     ───────X──── l2 is above here → remove l2
          ╱ ╲
         ╱   ╲

Cross-multiply to avoid floating point:
(b3 - b1) * (m1 - m2) <= (b2 - b1) * (m1 - m3)
```

---

## 📊 Classic Problem: Breaking Strings

```
dp[i] = min cost to process first i items
dp[i] = min over j { dp[j] + C(j, i) }

If C(j, i) = a[j] · b[i]:  → CHT applies

Example: dp[i] = min_j(dp[j] + prefix[j] · height[i])
  Line for j: y = prefix[j] · x + dp[j]
  Query at x = height[i]
```

---

## 🌐 When CHT Does NOT Directly Apply

```
dp[i] = min_j { dp[j] + (x[i] - x[j])² }

Expand: dp[j] + x[i]² - 2·x[i]·x[j] + x[j]²
      = (-2·x[j]) · x[i] + (dp[j] + x[j]²) + x[i]²
        ↑ slope     ↑ query   ↑ intercept        ↑ constant

m_j = -2·x[j]
b_j = dp[j] + x[j]²
query x = x[i]
add x[i]² at the end

This DOES decompose into lines → CHT applies!
```

---

## 🔧 Variants

### Li Chao Segment Tree
```
Handles arbitrary (non-sorted) queries and insertions.

Structure: segment tree over query x-values
Each node stores the "dominant" line for its interval

Insert: O(log C) where C = range of x values
Query:  O(log C)

Use when:
  - Queries and insertions are interleaved
  - x values are not monotone
  - Online queries
```

### Dynamic CHT (with deletions)
```
Use balanced BST (std::set) of lines.
Insert/delete/query in O(log n).

More complex but handles dynamic line sets.
```

---

## ⚡ Complexity Analysis

```
┌─────────────────────────────────────────────┐
│ Naive DP:         O(n²)                     │
│                                             │
│ CHT (sorted):     O(n)                      │
│   Each line added/removed from deque once   │
│   Amortized O(1) per operation              │
│                                             │
│ CHT (unsorted):   O(n log n)                │
│   Binary search for optimal line            │
│                                             │
│ Li Chao Tree:     O(n log C)                │
│   C = range of query values                 │
│                                             │
│ Space: O(n) for all variants                │
└─────────────────────────────────────────────┘
```

---

## 📊 Applicability Checklist

```
CHT applies when:

□ DP has form: dp[i] = min/max_j { f(j)·g(i) + h(j) } + k(i)
□ The "slope" f(j) and "intercept" h(j) depend only on j
□ The "query" g(i) depends only on i
□ Both slope and query are nicely ordered

CHT variant selection:
  ┌──────────────────────────────────────────────┐
  │ Slopes sorted + queries sorted → O(n) deque  │
  │ Slopes sorted + queries unsorted → O(n log n)│
  │ Arbitrary → Li Chao tree O(n log C)          │
  └──────────────────────────────────────────────┘
```

---

## 📊 Summary Table

| Concept | Description |
|---------|-------------|
| **Form** | dp[i] = min_j { m_j · x_i + b_j } |
| **Insight** | Minimum of linear functions = lower convex hull |
| **Sorted** | O(n) amortized with deque |
| **Unsorted** | O(n log n) with binary search |
| **Li Chao** | O(n log C) segment tree variant |
| **Key step** | Decompose cost(j,i) into slope × query + intercept |

---

## ❓ Quick Revision Questions

1. **What form must a DP recurrence have for CHT to apply?**
2. **Geometrically, what does CHT compute?**
3. **When can you use the O(n) deque-based CHT?**
4. **How do you check if a line is dominated (should be removed)?**
5. **How do you handle dp[i] = min(dp[j] + (x[i]-x[j])²)?**
6. **When would you use a Li Chao tree instead of the deque approach?**

---

[← Previous: Matrix Exponentiation](03-matrix-exponentiation.md) | [Next: Divide & Conquer Optimization →](05-divide-conquer-optimization.md)

[← Back to README](../README.md)
