# Chapter 2: Identifying Problem Types

## 📋 Chapter Overview
Learn to classify DSA problems by their *type* before solving them. Correct classification is the first step to selecting the right pattern and approach.

---

## 🗂️ The Problem Classification Framework

```
  ┌────────────────────────────────────────────────────────┐
  │              PROBLEM CLASSIFICATION                    │
  │                                                        │
  │   Step 1: What DATA STRUCTURE is involved?             │
  │   ┌────────┬─────────┬───────┬──────────┬────────┐    │
  │   │ Array  │ String  │ Tree  │  Graph   │ Other  │    │
  │   └────────┴─────────┴───────┴──────────┴────────┘    │
  │                                                        │
  │   Step 2: What OPERATION is required?                  │
  │   ┌────────┬─────────┬───────┬──────────┬────────┐    │
  │   │ Search │ Sort    │Count  │ Optimize │Generate│    │
  │   └────────┴─────────┴───────┴──────────┴────────┘    │
  │                                                        │
  │   Step 3: What CONSTRAINTS narrow it down?             │
  │   ┌────────┬─────────┬───────┬──────────┬────────┐    │
  │   │ Sorted │ n≤10^5  │Contig.│ All/Any  │ Edges  │    │
  │   └────────┴─────────┴───────┴──────────┴────────┘    │
  └────────────────────────────────────────────────────────┘
```

---

## 📊 Problem Type Decision Tree

```
                    ┌──────────────────┐
                    │  READ THE PROBLEM │
                    └────────┬─────────┘
                             │
              ┌──────────────┼──────────────┐
              ▼              ▼              ▼
      ┌──────────────┐ ┌─────────┐  ┌────────────┐
      │Array / String│ │Graph /  │  │ Math /     │
      │  Problems    │ │Tree     │  │ Bit Manip  │
      └──────┬───────┘ └────┬────┘  └────────────┘
             │              │
    ┌────────┼────────┐     │
    ▼        ▼        ▼     ▼
 ┌──────┐┌───────┐┌─────┐┌────────┐
 │Sorted││Contig.││Pairs││Traverse│
 │data? ││subarr?││    ││/ Path? │
 └──┬───┘└───┬───┘└──┬──┘└───┬────┘
    │        │       │       │
    ▼        ▼       ▼       ▼
  Binary  Sliding  Two    BFS/DFS
  Search  Window   Ptrs   Backtrack
```

---

## 🏷️ Category 1: Array & String Problems

### Sub-types and Their Patterns:

```
  ARRAY / STRING PROBLEMS
  ┌─────────────────────────────────────────────────┐
  │                                                 │
  │  ┌─────────────────────┐  ┌──────────────────┐ │
  │  │ SUBARRAY problems   │  │ PAIR problems    │ │
  │  │ ─────────────────   │  │ ──────────────── │ │
  │  │ • Max sum subarray  │  │ • Two sum        │ │
  │  │ • Min window substr │  │ • Three sum      │ │
  │  │ • Subarray with sum │  │ • Pair with diff │ │
  │  │                     │  │                  │ │
  │  │ Pattern: Sliding    │  │ Pattern: Two Ptr │ │
  │  │ Window / Prefix Sum │  │ / Hash Map       │ │
  │  └─────────────────────┘  └──────────────────┘ │
  │                                                 │
  │  ┌─────────────────────┐  ┌──────────────────┐ │
  │  │ SEARCH problems     │  │ ORDERING problems│ │
  │  │ ─────────────────   │  │ ──────────────── │ │
  │  │ • Find element      │  │ • Next greater   │ │
  │  │ • Find boundary     │  │ • Sort variants  │ │
  │  │ • Search in rotated │  │ • Merge intervals│ │
  │  │                     │  │                  │ │
  │  │ Pattern: Binary     │  │ Pattern: Stack / │ │
  │  │ Search              │  │ Greedy           │ │
  │  └─────────────────────┘  └──────────────────┘ │
  └─────────────────────────────────────────────────┘
```

### Keyword → Pattern Mapping:

| Keywords in Problem Statement | Likely Pattern |
|------------------------------|----------------|
| "contiguous subarray", "substring" | Sliding Window |
| "sorted array", "two numbers that sum" | Two Pointers |
| "find in sorted", "minimum/maximum possible" | Binary Search |
| "frequency", "anagram", "permutation" | Hash Map |
| "next greater/smaller element" | Monotonic Stack |
| "merge k sorted" | Heap |
| "minimum number of operations" | DP or Greedy |

---

## 🏷️ Category 2: Graph & Tree Problems

```
  GRAPH / TREE PROBLEMS
  ┌─────────────────────────────────────────────────────┐
  │                                                     │
  │  ┌────────────────────┐  ┌────────────────────────┐ │
  │  │ TRAVERSAL problems │  │ CONNECTIVITY problems  │ │
  │  │ ────────────────── │  │ ────────────────────── │ │
  │  │ • Level order      │  │ • Number of islands    │ │
  │  │ • Inorder/Preorder │  │ • Connected components │ │
  │  │ • Zigzag           │  │ • Union / Find         │ │
  │  │                    │  │                        │ │
  │  │ Pattern: BFS/DFS   │  │ Pattern: DFS/BFS or   │ │
  │  │                    │  │ Union-Find             │ │
  │  └────────────────────┘  └────────────────────────┘ │
  │                                                     │
  │  ┌────────────────────┐  ┌────────────────────────┐ │
  │  │ PATH problems      │  │ STATE problems         │ │
  │  │ ────────────────── │  │ ────────────────────── │ │
  │  │ • Shortest path    │  │ • Word ladder          │ │
  │  │ • Path sum         │  │ • Open the lock        │ │
  │  │ • All paths        │  │ • Puzzle solving       │ │
  │  │                    │  │                        │ │
  │  │ Pattern: BFS (short│  │ Pattern: BFS on state  │ │
  │  │ est), DFS (all)    │  │ space / Backtracking   │ │
  │  └────────────────────┘  └────────────────────────┘ │
  └─────────────────────────────────────────────────────┘
```

### Keyword → Pattern Mapping:

| Keywords in Problem Statement | Likely Pattern |
|------------------------------|----------------|
| "shortest path" (unweighted) | BFS |
| "all paths", "all possibilities" | DFS / Backtracking |
| "connected components", "groups" | DFS/BFS or Union-Find |
| "level by level" | BFS |
| "tree depth", "height", "diameter" | DFS |
| "cycle detection" | DFS / Union-Find |
| "topological order" | BFS (Kahn's) / DFS |

---

## 🏷️ Category 3: Optimization & Counting Problems

```
  OPTIMIZATION / COUNTING PROBLEMS
  ┌──────────────────────────────────────────────────┐
  │                                                  │
  │  ┌─────────────────────┐  ┌───────────────────┐ │
  │  │ OPTIMIZATION        │  │ COUNTING          │ │
  │  │ ─────────────────   │  │ ───────────────── │ │
  │  │ • Min cost path     │  │ • Number of ways  │ │
  │  │ • Max profit        │  │ • Count subsets   │ │
  │  │ • Min operations    │  │ • Count paths     │ │
  │  │                     │  │                   │ │
  │  │ Pattern: DP         │  │ Pattern: DP       │ │
  │  │ or Greedy           │  │                   │ │
  │  └─────────────────────┘  └───────────────────┘ │
  │                                                  │
  │  ┌─────────────────────┐  ┌───────────────────┐ │
  │  │ GENERATION          │  │ DECISION          │ │
  │  │ ─────────────────   │  │ ───────────────── │ │
  │  │ • Generate subsets  │  │ • Can partition?  │ │
  │  │ • All permutations  │  │ • Is possible?    │ │
  │  │ • Valid combinations│  │ • True / False    │ │
  │  │                     │  │                   │ │
  │  │ Pattern:            │  │ Pattern: DP /     │ │
  │  │ Backtracking        │  │ Greedy / Search   │ │
  │  └─────────────────────┘  └───────────────────┘ │
  └──────────────────────────────────────────────────┘
```

---

## 🔍 The Constraint Analysis Method

Constraints are your **strongest clue** for pattern selection:

```
  CONSTRAINT → EXPECTED COMPLEXITY → PATTERN

  n ≤ 10       → O(n!)           → Backtracking / Brute Force
  n ≤ 20       → O(2^n)          → Bitmask / Backtracking
  n ≤ 500      → O(n³)           → DP (3 dimensions)
  n ≤ 5,000    → O(n²)           → DP (2 dimensions)
  n ≤ 10^5     → O(n log n)      → Sorting / Binary Search / Heap
  n ≤ 10^6     → O(n)            → Sliding Window / Two Pointers / Hash Map
  n ≤ 10^9     → O(log n)        → Binary Search / Math
  n ≤ 10^18    → O(log n)        → Binary Search / Math / Matrix Exp.
```

### Pseudocode: Constraint Analysis

```
function analyzeConstraints(n):
    if n <= 10:
        return "Try all possibilities (brute force / backtracking)"
    if n <= 20:
        return "Bitmask DP or backtracking with pruning"
    if n <= 500:
        return "O(n³) — possibly 3-nested loops or 3D DP"
    if n <= 5000:
        return "O(n²) — 2D DP or nested loops"
    if n <= 100000:
        return "O(n log n) — sorting, binary search, heap, divide & conquer"
    if n <= 1000000:
        return "O(n) — sliding window, two pointers, hash map, greedy"
    if n > 1000000:
        return "O(log n) — binary search on answer, math"
```

---

## 🧪 Worked Example: Classifying a Problem

**Problem:** "Given a sorted array of integers, find two numbers that add up to a target."

### Step-by-step classification:

```
  Step 1: DATA STRUCTURE
  ┌──────────────────────────────┐
  │ "sorted array of integers"  │ → ARRAY
  └──────────────────────────────┘

  Step 2: OPERATION
  ┌──────────────────────────────┐
  │ "find two numbers"           │ → SEARCH / PAIR
  └──────────────────────────────┘

  Step 3: CONSTRAINTS
  ┌──────────────────────────────┐
  │ "sorted" + "two numbers"     │ → TWO POINTERS
  │ Alternative: HashMap works   │
  │ too, but Two Pointers is     │
  │ optimal (O(1) space)         │
  └──────────────────────────────┘

  VERDICT: Two Pointers Pattern (opposite direction)
```

### Another Example:

**Problem:** "Find the longest substring with at most K distinct characters."

```
  Step 1: DATA STRUCTURE → String (similar to array)
  Step 2: OPERATION → "longest" = optimization, "substring" = contiguous
  Step 3: CONSTRAINTS → "at most K" = variable condition, "substring" = window

  VERDICT: Sliding Window Pattern (variable size)
```

---

## 🧭 Problem Type Quick-Match Chart

| Problem Says... | Think... | Pattern |
|-----------------|----------|---------|
| "Subarray of size K" | Fixed window | Sliding Window |
| "Longest substring with condition" | Variable window | Sliding Window |
| "Sorted array + find" | Binary elimination | Binary Search / Two Ptrs |
| "kth largest/smallest" | Partial sorting | Heap |
| "All subsets/permutations" | Exhaustive search | Backtracking |
| "Minimum cost to reach" | Overlapping subproblems | DP |
| "Number of ways" | Counting paths | DP |
| "Connected components" | Graph traversal | DFS/BFS / Union-Find |
| "Shortest path (unweighted)" | Level-by-level | BFS |
| "Schedule max events" | Local optimal | Greedy |
| "Next greater element" | Maintain order | Monotonic Stack |
| "Group anagrams" | Categorize by key | Hash Map |

---

## 📋 Summary Table

| Concept | Key Takeaway |
|---------|-------------|
| Classification Framework | Data Structure → Operation → Constraints |
| Array/String Problems | Sliding Window, Two Pointers, Binary Search, Hash Map |
| Graph/Tree Problems | BFS, DFS, Backtracking, Union-Find |
| Optimization Problems | DP, Greedy |
| Generation Problems | Backtracking |
| Constraint Analysis | n's size tells you the expected time complexity |
| Keywords | Specific words in the problem map to specific patterns |

---

## ❓ Quick Revision Questions

1. **What are the three steps to classify a DSA problem?**
   > Identify the Data Structure → Determine the Operation → Analyze the Constraints.

2. **If n ≤ 10^6, what time complexity should you target?**
   > O(n) — use patterns like Sliding Window, Two Pointers, or Hash Map.

3. **The problem says "find the longest substring with at most 2 distinct characters." What pattern?**
   > Sliding Window (variable size).

4. **What pattern handles "find all subsets" problems?**
   > Backtracking.

5. **If a problem says "shortest path in unweighted graph," which traversal?**
   > BFS (Breadth-First Search) — it naturally finds shortest paths in unweighted graphs.

6. **Why are constraints the strongest clue for pattern selection?**
   > Constraints (especially n's size) directly tell you the maximum acceptable time complexity, which narrows down which patterns/algorithms can work.

---

[← Previous: Why Patterns Matter](01-why-patterns-matter.md) | [Next: Pattern-Based Thinking →](03-pattern-based-thinking.md)

[← Back to README](../README.md)
