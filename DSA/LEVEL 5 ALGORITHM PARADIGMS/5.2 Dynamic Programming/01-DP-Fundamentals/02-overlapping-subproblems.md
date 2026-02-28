# Chapter 2: Overlapping Subproblems

## 📋 Overview
A problem has **overlapping subproblems** when the same smaller subproblems are solved multiple times during the recursive computation. This is the first key property that makes Dynamic Programming applicable and beneficial.

---

## 🧠 Core Concept

```
┌─────────────────────────────────────────────────────────┐
│              OVERLAPPING SUBPROBLEMS                    │
│                                                         │
│  When solving a problem recursively, if the SAME        │
│  subproblem appears MORE THAN ONCE in the recursion     │
│  tree, the subproblems are "overlapping."               │
│                                                         │
│  ┌──────────┐     ┌──────────┐                          │
│  │ Problem  │     │ Problem  │                          │
│  │   A      │     │   B      │                          │
│  └────┬─────┘     └────┬─────┘                          │
│       │                │                                │
│       ▼                ▼                                │
│  ┌─────────┐     ┌─────────┐                            │
│  │  Sub-   │     │  Sub-   │                            │
│  │problem C│◄────│problem C│  ◄── SAME subproblem!      │
│  └─────────┘     └─────────┘                            │
│       ▲                                                 │
│       │                                                 │
│  Solve ONCE, store, and REUSE                           │
└─────────────────────────────────────────────────────────┘
```

---

## 🔍 Visualizing Overlapping Subproblems

### Example 1: Fibonacci Numbers

```
Recursion tree for fib(6):

                            fib(6)
                          /        \
                     fib(5)         fib(4)         ◄─ fib(4) appears twice
                    /     \         /     \
                fib(4)   fib(3)  fib(3)  fib(2)    ◄─ fib(3) appears 3 times
                /    \    /   \   /   \
           fib(3) fib(2) fib(2) fib(1) fib(2) fib(1)
           /   \
       fib(2) fib(1)

Count of subproblem calls:
┌──────────┬────────────┐
│ Subproblem│  # Times  │
├──────────┼────────────┤
│  fib(6)  │     1      │
│  fib(5)  │     1      │
│  fib(4)  │     2      │  ◄── overlapping
│  fib(3)  │     3      │  ◄── overlapping
│  fib(2)  │     5      │  ◄── overlapping
│  fib(1)  │     8      │  ◄── overlapping
│  fib(0)  │     5      │  ◄── overlapping
├──────────┼────────────┤
│  TOTAL   │    25      │  vs only 7 unique subproblems!
└──────────┴────────────┘
```

### Example 2: Climbing Stairs

```
ways(5) = ways(4) + ways(3)

                        ways(5)
                       /       \
                  ways(4)      ways(3)       ◄── computed again below
                 /      \      /      \
            ways(3)  ways(2) ways(2) ways(1)
            /    \
       ways(2) ways(1)

Overlapping: ways(3) computed 2×, ways(2) computed 3×
```

---

## ⚡ Overlapping vs Non-Overlapping

```
┌───────────────────────────────┐  ┌───────────────────────────────┐
│     OVERLAPPING (DP)          │  │  NON-OVERLAPPING (D&C)        │
│                               │  │                               │
│     problem(n)                │  │     merge_sort([5,3,8,1])     │
│      /       \                │  │      /              \         │
│  sub(n-1)  sub(n-2)           │  │  sort([5,3])    sort([8,1])   │
│    / \       / \              │  │   /    \         /    \       │
│ sub(n-2) sub(n-3) sub(n-3)   │  │ [5]    [3]     [8]    [1]    │
│    ▲           ▲              │  │                               │
│    └── SAME ───┘              │  │  Each subproblem is UNIQUE    │
│                               │  │  No repetition occurs         │
│  DP helps! Store & reuse.     │  │  DP won't help here.          │
└───────────────────────────────┘  └───────────────────────────────┘
```

---

## 🧪 How to Detect Overlapping Subproblems

### Method 1: Draw the Recursion Tree
```
Pseudocode:
function solve(n):
    if base_case: return value
    return solve(smaller1) + solve(smaller2)

Draw the tree → Do any nodes repeat?
    YES → Overlapping subproblems exist
    NO  → No overlap, DP may not help
```

### Method 2: Check Function Parameters
```
If the same function is called with the SAME parameters
more than once → overlapping subproblems.

Track calls:
  solve(5) → calls solve(4), solve(3)
  solve(4) → calls solve(3), solve(2)    ◄── solve(3) called AGAIN
  
Parameters (3) repeated → OVERLAPPING!
```

### Method 3: Count Unique States
```
┌────────────────────────────────────────────┐
│  Total recursive calls >> Unique states    │
│                                            │
│  Fibonacci(20):                            │
│    Total calls:    ~21,891                 │
│    Unique states:  21                      │
│    Ratio: 1042:1  → Massive overlap!       │
│                                            │
│  Binary Search(20):                        │
│    Total calls:    ~5                      │
│    Unique states:  5                       │
│    Ratio: 1:1     → No overlap            │
└────────────────────────────────────────────┘
```

---

## 📐 Examples with Different Problems

### Problem: Minimum Cost Climbing Stairs
```
cost = [10, 15, 20]

                     minCost(3)
                    /          \
             minCost(2)      minCost(1)
             /       \          │
        minCost(1)  minCost(0) (base)
            │
         (base)

minCost(1) appears twice → Overlapping!
```

### Problem: Grid Paths (m×n grid)
```
Count paths from (0,0) to (2,2), moving right or down.

                     paths(2,2)
                    /          \
            paths(1,2)       paths(2,1)
            /       \        /       \
      paths(0,2) paths(1,1) paths(1,1) paths(2,0)
                    ▲           ▲
                    └── SAME ───┘

paths(1,1) computed twice → Overlapping!
```

### Problem Without Overlap: Binary Search
```
binarySearch(arr, 0, 15, target)
         │
    binarySearch(arr, 0, 7, target)      ← left half
         │
    binarySearch(arr, 0, 3, target)      ← left-left
         │
    binarySearch(arr, 0, 1, target)      ← found!

Each call is to a DIFFERENT range → No overlap.
DP is NOT useful here.
```

---

## 🔄 Impact of Overlapping Subproblems

### Without Storing (Naive Recursion)
```
Time Complexity Growth:

n    |  Unique States  |  Total Calls (fib)  |  Wasted Work
─────┼─────────────────┼─────────────────────┼──────────────
5    |       6         |        15           |     9
10   |      11         |       177           |   166
20   |      21         |     21,891          |  21,870
30   |      31         |   2,692,537         |  2,692,506
40   |      41         | 331,160,281         | 331,160,240

Wasted work grows EXPONENTIALLY!
```

### With DP (Memoization/Tabulation)
```
n    |  Unique States  |  Total Work  |  Savings
─────┼─────────────────┼──────────────┼──────────────
5    |       6         |      6       |     60%
10   |      11         |     11       |     94%
20   |      21         |     21       |     99.9%
30   |      31         |     31       |     99.999%
40   |      41         |     41       |     99.99999%

Each unique state computed EXACTLY ONCE!
```

---

## 🛠️ Pseudocode: Detecting and Handling Overlap

```
// Step 1: Write naive recursion
function solve(params):
    if base_case:
        return base_value
    return combine(solve(sub1), solve(sub2))

// Step 2: Add a counter to detect overlap
call_count = {}
function solve_with_count(params):
    call_count[params] = call_count.get(params, 0) + 1
    if base_case:
        return base_value
    return combine(solve_with_count(sub1), solve_with_count(sub2))

// Step 3: If overlap detected, add memoization
memo = {}
function solve_memo(params):
    if params in memo:
        return memo[params]       // ← reuse stored result
    if base_case:
        return base_value
    memo[params] = combine(solve_memo(sub1), solve_memo(sub2))
    return memo[params]
```

---

## 💡 Key Insight

```
┌─────────────────────────────────────────────────────────┐
│                                                         │
│  Overlapping subproblems = REDUNDANT COMPUTATION        │
│                                                         │
│  DP eliminates redundancy by:                           │
│                                                         │
│  1. Computing each subproblem ONCE                      │
│  2. Storing the result                                  │
│  3. Looking it up when needed again                     │
│                                                         │
│  This transforms:                                       │
│    Exponential time → Polynomial time                   │
│    O(2ⁿ) → O(n)  for Fibonacci                        │
│    O(3ⁿ) → O(n²) for some grid problems               │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

---

## 📊 Summary Table

| Concept | Description |
|---------|-------------|
| **Overlapping Subproblems** | Same subproblem solved multiple times in recursion |
| **Detection** | Draw recursion tree or track function call parameters |
| **Without DP** | Exponential time due to redundant computation |
| **With DP** | Each subproblem solved once, stored, and reused |
| **Examples** | Fibonacci, Grid paths, Climbing stairs |
| **Non-examples** | Binary search, Merge sort (non-overlapping) |

---

## ❓ Quick Revision Questions

1. **What does "overlapping subproblems" mean? Give an example.**
2. **How would you detect if a recursive solution has overlapping subproblems?**
3. **Does Merge Sort have overlapping subproblems? Why or why not?**
4. **If fib(30) has 31 unique states but makes ~2.7 million recursive calls, what percentage of work is wasted?**
5. **Can a problem have overlapping subproblems but NOT optimal substructure? If so, can DP still help?**
6. **What is the relationship between the number of unique states and the DP table size?**

---

[← Previous: What is DP?](01-what-is-dp.md) | [Next: Optimal Substructure →](03-optimal-substructure.md)

[← Back to README](../README.md)
